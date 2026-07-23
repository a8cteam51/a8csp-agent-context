---
name: block-editor-development
description: >
  Conventions and patterns for building custom Gutenberg blocks, block patterns,
  and editor extensions in Team51 projects. Activate when creating or modifying
  blocks, working with @wordpress/scripts, InnerBlocks, block.json, or editor UI.
---

# Block / Editor Development

## Overview

Team51 builds custom Gutenberg blocks for partner sites. Blocks live inside plugins (not themes) unless they are purely presentational block patterns or template parts. All block code uses the `@wordpress/scripts` toolchain and follows WordPress Block Editor Handbook conventions.

## Block Registration

### File structure

Each block lives in its own directory under `src/blocks/`:

```
src/blocks/{block-name}/
├── block.json        ← Block metadata (required)
├── edit.js           ← Editor component
├── save.js           ← Frontend save (or null for dynamic blocks)
├── index.js          ← Registration entry point
├── style.scss        ← Shared styles (editor + frontend)
├── editor.scss       ← Editor-only styles (optional)
└── view.js           ← Frontend interactivity script (optional)
```

### block.json conventions

- **`name`**: `a8csp/{block-name}` (matches the plugin namespace).
- **`textdomain`**: matches the plugin slug (e.g., `a8csp-custom-events`).
- **`category`**: use a custom category registered by the plugin, or a core category (`text`, `media`, `design`, `widgets`, `embed`).
- **`apiVersion`**: always `3` for new blocks.
- **`supports`**: declare only the supports the block actually needs. Prefer opting in over leaving defaults.

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "a8csp/event-card",
  "version": "1.0.0",
  "title": "Event Card",
  "category": "a8csp-events",
  "textdomain": "a8csp-custom-events",
  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./style-index.css",
  "render": "file:./render.php",
  "supports": {
    "html": false,
    "align": [ "wide", "full" ],
    "color": { "background": true, "text": true },
    "spacing": { "margin": true, "padding": true },
    "typography": { "fontSize": true }
  }
}
```

### Registration in PHP

Register the block from the plugin's main class or hooks file:

```php
add_action( 'init', function () {
    $blocks_dir = plugin_dir_path( __FILE__ ) . 'build/blocks/';
    foreach ( glob( $blocks_dir . '*/block.json' ) as $block_json ) {
        register_block_type( dirname( $block_json ) );
    }
} );
```

## Static vs Dynamic Blocks

| Type | When to use | `save.js` | Server render |
|---|---|---|---|
| **Static** | Content is stored in post HTML and doesn't change based on context | Returns JSX markup | No |
| **Dynamic** | Content depends on live data (queries, current user, options) | Returns `null` | Yes — use `render` in block.json or `render_callback` |

**Prefer dynamic blocks** for anything that queries posts, options, or user data. Static blocks are appropriate for simple layout or text content.

### Dynamic block rendering

Use a `render.php` file referenced in block.json:

```php
<?php
/**
 * Renders the Event Card block.
 *
 * @param array    $attributes Block attributes.
 * @param string   $content    Inner block content.
 * @param WP_Block $block      Block instance.
 */
$event_id = $attributes['eventId'] ?? 0;
if ( ! $event_id ) {
    return;
}

$title = esc_html( get_the_title( $event_id ) );
$date  = esc_html( get_post_meta( $event_id, '_event_date', true ) );

printf(
    '<div %s><h3>%s</h3><time>%s</time></div>',
    get_block_wrapper_attributes(),
    $title,
    $date
);
```

## InnerBlocks

Use `InnerBlocks` when a block needs to contain other blocks (layout wrappers, sections, columns).

- **Always constrain allowed blocks** via `allowedBlocks` unless the block is a generic container.
- Use `template` and `templateLock` to provide default structure.
- Use `useInnerBlocksProps` for proper editor rendering.

```jsx
import { useBlockProps, useInnerBlocksProps } from '@wordpress/block-editor';

const ALLOWED_BLOCKS = [ 'core/heading', 'core/paragraph', 'core/image' ];
const TEMPLATE = [
    [ 'core/heading', { level: 3, placeholder: 'Section title' } ],
    [ 'core/paragraph', { placeholder: 'Section content…' } ],
];

export default function Edit() {
    const blockProps = useBlockProps();
    const innerBlocksProps = useInnerBlocksProps( blockProps, {
        allowedBlocks: ALLOWED_BLOCKS,
        template: TEMPLATE,
    } );

    return <div { ...innerBlocksProps } />;
}
```

## Block Patterns and Variations

### Block patterns

Prefer block patterns when the need is a reusable layout composed entirely of existing blocks (no custom logic). Register in PHP:

```php
register_block_pattern(
    'a8csp/hero-with-cta',
    array(
        'title'       => __( 'Hero with CTA', 'a8csp-theme' ),
        'description' => __( 'Full-width hero section with heading, text, and button.', 'a8csp-theme' ),
        'categories'  => array( 'a8csp-layouts' ),
        'content'     => '<!-- wp:group {"align":"full","layout":{"type":"constrained"}} -->...',
    )
);
```

Patterns live in the theme (presentational) or plugin (functional). Register a custom pattern category for discoverability.

### Block variations

Use variations when a core or custom block needs alternate presets (different default attributes, inner blocks, or icon). Registered in JavaScript:

```js
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation( 'core/group', {
    name: 'a8csp-card',
    title: 'Card',
    attributes: { className: 'is-style-card' },
    innerBlocks: [
        [ 'core/heading', { level: 3 } ],
        [ 'core/paragraph' ],
    ],
    scope: [ 'inserter' ],
} );
```

## Build Toolchain

- Use **`@wordpress/scripts`** for compilation, linting, and hot reload.
- Source lives in `src/`, build output goes to `build/` (gitignored).
- Common scripts in `package.json`:

```json
{
  "scripts": {
    "build": "wp-scripts build",
    "start": "wp-scripts start",
    "lint:js": "wp-scripts lint-js",
    "lint:css": "wp-scripts lint-style"
  }
}
```

- Multiple block entry points are auto-discovered from `src/blocks/*/index.js` in `@wordpress/scripts` v27+.
- For single-entry plugins, the entry point is `src/index.js`.

## Editor UX Conventions

- **Block toolbar first, Inspector sidebar second.** Put the most-used controls in the toolbar; put detailed settings in the sidebar.
- **Use `Placeholder` components** for blocks that require configuration before they display content (e.g., selecting a post, entering a URL).
- **Provide meaningful previews** in the editor. Avoid blocks that show only a placeholder or icon in the editor and render fully only on the frontend.
- **Respect `theme.json`** — use block supports for colors, spacing, and typography instead of custom controls that duplicate what the theme already provides.

## Customizing Taxonomy UI in the Editor

- **`meta_box_cb` does not work in the block editor.** Core registers every taxonomy meta box with `__back_compat_meta_box => true`, and the block editor skips those entirely. Any taxonomy with `show_in_rest => true` gets the native sidebar panel instead — checkboxes for hierarchical taxonomies, a token field for flat ones. A PHP-only radio/select `meta_box_cb` silently renders as checkboxes; do not propose it, and flag it in review if you see it.
- **Custom taxonomy UI (radio buttons, a select, anything else) IS possible** — never tell a partner it isn't. Use the `editor.PostTaxonomyType` JS filter to swap the panel component for that taxonomy slug (e.g. a `RadioControl` or `SelectControl` writing term IDs via `editPost`), enqueued with `enqueue_block_editor_assets` scoped to the relevant post type. This is the officially documented mechanism (the filter wraps `HierarchicalTermSelector` / `FlatTermSelector` via `withFilters`).
- **Keep `show_in_rest => true`.** It is required for the taxonomy to appear in the Query Loop block's Filters panel (which keys off `publicly_queryable` in REST visibility) and does not conflict with a custom panel.
- **Enforce single-term invariants server-side.** Quick Edit and Bulk Edit keep core's checkbox checklist no matter what the editor panel shows, so an "exactly one term" rule needs a `save_post` / `set_object_terms` guard, not just UI.

## WPCOM Compatibility

- Block themes work well on both Pressable and WPCOM Simple.
- Blocks that rely on filesystem writes, server-side cron, or WP-CLI will not work on WPCOM Simple.
- Test block rendering on WPCOM staging before deploying — the WPCOM editor adds its own blocks and styles that may conflict.
- Avoid bundling large JS dependencies; use `@wordpress/*` packages (already available in the editor runtime) via `wp-scripts` dependency extraction.

## Boundaries

- For plugin scaffolding and structure, see `PLUGIN-DEVELOPMENT.md`.
- For PHP and JS coding standards, see `coding-standards.md`.
- For security in editor contexts (sanitization, escaping block output), see `security.md`.
