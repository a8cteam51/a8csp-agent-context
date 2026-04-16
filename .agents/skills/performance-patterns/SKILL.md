---
name: performance-patterns
description: >
  WordPress performance optimization patterns for Team51 projects. Activate when
  working on caching, query optimization, asset loading, image performance, database
  queries, transients, object caching, lazy loading, or page speed improvements.
---

# Performance Patterns

## Overview

Team51 sites are high-traffic, production WordPress installations on Pressable and WPCOM Simple. Performance is a key principle: minimize HTTP requests, avoid N+1 queries, and cache expensive operations. This skill covers WordPress-native performance patterns applicable across both platforms.

## Query Optimization

### Avoid N+1 queries

Never loop through posts and issue individual queries per post. Use a single `WP_Query` with appropriate arguments.

```php
// BAD — N+1: one query per post for meta
$posts = get_posts( array( 'post_type' => 'event', 'numberposts' => 50 ) );
foreach ( $posts as $post ) {
    $date = get_post_meta( $post->ID, '_event_date', true ); // 50 extra queries
}

// GOOD — prime the meta cache in one query
$posts = get_posts( array(
    'post_type'              => 'event',
    'numberposts'            => 50,
    'update_post_meta_cache' => true,
) );
foreach ( $posts as $post ) {
    $date = get_post_meta( $post->ID, '_event_date', true ); // served from cache
}
```

### Restrict query fields

When you only need IDs or a count, tell WordPress:

```php
// Only need IDs
$ids = get_posts( array(
    'post_type' => 'event',
    'fields'    => 'ids',
) );

// Only need a count
$query = new WP_Query( array(
    'post_type'      => 'event',
    'posts_per_page' => 1,
    'no_found_rows'  => true, // skips SQL_CALC_FOUND_ROWS
    'fields'         => 'ids',
) );
$found = $query->found_posts;
```

### `no_found_rows`

Set `'no_found_rows' => true` on any `WP_Query` that does not need pagination. This eliminates the expensive `SQL_CALC_FOUND_ROWS` call.

### Meta query performance

`meta_query` arguments cause `JOIN` operations that scale poorly. Mitigation strategies:

- Store frequently-queried values as taxonomy terms instead of post meta.
- Use `'meta_key'` + `'orderby' => 'meta_value'` only when necessary; prefer taxonomy-based filtering.
- If raw SQL is unavoidable, add indexes and use `$wpdb->prepare()`.

## Caching

### Transients

Use transients for expensive operations that don't need real-time freshness (API responses, complex aggregations, computed data).

```php
$cache_key = 'a8csp_featured_events_' . md5( wp_json_encode( $args ) );
$result    = get_transient( $cache_key );

if ( false === $result ) {
    $result = expensive_query_or_api_call( $args );
    set_transient( $cache_key, $result, HOUR_IN_SECONDS );
}

return $result;
```

**Transient guidelines:**

- Key names must be 172 characters or fewer (WordPress limit).
- Use descriptive prefixed keys: `a8csp_{feature}_{identifier}`.
- Choose TTL based on data freshness needs — not a default value.
- Invalidate explicitly when the underlying data changes (hook into `save_post`, `updated_option`, etc.).
- On Pressable and WPCOM, transients backed by object cache don't hit the database — they're fast.

### Object cache

Both Pressable and WPCOM provide persistent object caches (Memcached). WordPress core functions (`get_post()`, `get_post_meta()`, `get_option()`) already use the object cache. Leverage this:

- **Don't bypass the cache** by using direct `$wpdb` queries for data that core functions already cache.
- **Use `wp_cache_*` functions** for custom cached data that doesn't fit the transient model:

```php
$data = wp_cache_get( 'a8csp_stats_summary', 'a8csp' );
if ( false === $data ) {
    $data = compute_stats_summary();
    wp_cache_set( 'a8csp_stats_summary', $data, 'a8csp', 300 );
}
```

- Use cache groups (`'a8csp'`) to namespace and enable group-level invalidation.

### Page cache awareness

Pressable and WPCOM both apply server-side full-page caching (Varnish/nginx). Implications:

- **Logged-out visitors** see cached pages. Dynamic content for anonymous users must use JavaScript (e.g., client-side fetches to the REST API) or fragment caching.
- **Cache-busting**: Use versioned asset URLs (`wp_enqueue_style( ..., filemtime(...) )`). Don't rely on query strings alone — some CDNs strip them.
- **Vary headers**: If content varies by cookie or header, ensure proper `Vary` headers so the page cache serves the correct variant.
- **Cache purge**: After deployments or major content changes, purge via the platform tools (Pressable MPCP, WPCOM admin, or Team51 CLI).

## Asset Loading

### Enqueue strategy

- **Only load assets where needed.** Use conditional checks:

```php
add_action( 'wp_enqueue_scripts', function () {
    if ( ! is_singular( 'event' ) ) {
        return;
    }
    wp_enqueue_style( 'a8csp-event-single', ... );
    wp_enqueue_script( 'a8csp-event-single', ... );
} );
```

- **Use block-level asset loading.** When assets belong to a block, declare them in `block.json` (`editorScript`, `style`, `viewScript`). WordPress loads them only when the block is present.

### Script loading attributes

Use `wp_script_add_data()` or the `strategy` argument (WordPress 6.3+) for non-blocking script loading:

```php
wp_enqueue_script( 'a8csp-analytics', $url, array(), $ver, array(
    'strategy' => 'defer',
) );
```

- **`defer`**: preferred for scripts that don't need to run during parsing.
- **`async`**: use for independent scripts (analytics, third-party embeds).
- Never defer/async scripts that other scripts depend on.

### CSS optimization

- Inline critical CSS for above-the-fold content when page speed is a hard requirement.
- Use `wp_enqueue_block_style()` (WordPress 5.9+) to load block-specific stylesheets only when the block is used.
- Avoid loading the entire theme stylesheet on pages that don't need it.

## Image Performance

### Responsive images

WordPress generates `srcset` and `sizes` attributes automatically for images inserted through the media library. Ensure custom image output preserves this:

```php
echo wp_get_attachment_image( $id, 'large' ); // includes srcset
```

When outputting custom `<img>` tags, add `srcset` and `sizes` manually or use `wp_get_attachment_image_srcset()`.

### Lazy loading

WordPress adds `loading="lazy"` to images and iframes by default (since 5.5). Best practices:

- **Don't lazy-load above-the-fold images.** Use `fetchpriority="high"` and `loading="eager"` for the LCP image.
- Use `wp_img_tag_add_loading_optimization_attrs()` to control behavior programmatically.
- For custom image output, add `loading="lazy"` and `decoding="async"` attributes.

### Image sizing

- Register custom image sizes when the theme or plugin needs specific dimensions:

```php
add_image_size( 'a8csp-card-thumb', 400, 300, true );
```

- Avoid generating unnecessary sizes. Remove unused default sizes via `intermediate_image_sizes` filter if they add bloat.

## Database Performance

- Prefer WordPress API functions over direct `$wpdb` queries — they use the object cache.
- When raw queries are unavoidable, use `$wpdb->prepare()` and ensure relevant columns are indexed.
- Avoid `LIKE '%term%'` queries on large tables — they cannot use indexes.
- Use `wp_defer_term_counting()` and `wp_defer_comment_counting()` during bulk operations.
- For bulk imports or migrations, wrap in a transaction and disable autocommit if the plugin controls the table.

## Platform-Specific Notes

### Pressable
- Persistent object cache (Memcached) is always active.
- Varnish page cache sits in front of WordPress. Respect cache headers.
- PHP OPcache is enabled — no action needed, but avoid `eval()` or dynamic includes that defeat it.

### WPCOM Simple
- Object cache and page cache are managed by the platform — you cannot configure them.
- No filesystem caching (no write access). Transients and object cache are the only options.
- Third-party CDN configuration is limited. Use WPCOM's built-in CDN (Photon) for images.

## Boundaries

- For security considerations in caching and data handling, see `security.md`.
- For coding standards (naming, formatting), see `coding-standards.md`.
- For platform capabilities and constraints, see `PRESSABLE.md` and `WPCOM-SIMPLE.md`.
