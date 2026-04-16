# Security Practices

Security is non-negotiable on Team51 projects. We work on high-profile, live production sites. Every line of code must be written with security in mind.

## Core principles

1. **Never trust user input** — validate, sanitize, and escape everything.
2. **Defense in depth** — apply multiple layers of protection.
3. **Least privilege** — only grant the permissions that are needed.
4. **Fail securely** — when something goes wrong, default to denying access.

## Output escaping

**Always escape data when outputting it.** No exceptions.

| Context | Function |
|---|---|
| HTML content | `esc_html()` |
| HTML attributes | `esc_attr()` |
| URLs | `esc_url()` |
| JavaScript strings | `esc_js()` |
| Textarea content | `esc_textarea()` |
| Translated strings with HTML | `wp_kses()` or `wp_kses_post()` |

```php
// CORRECT
echo '<a href="' . esc_url( $link ) . '">' . esc_html( $title ) . '</a>';

// WRONG — unescaped output
echo '<a href="' . $link . '">' . $title . '</a>';
```

### Late escaping

Escape as **late as possible** — at the point of output, not when storing or retrieving data. This ensures data is always escaped for its specific context.

## Input sanitization

**Always sanitize data received from users, APIs, or external sources.**

| Data type | Function |
|---|---|
| Plain text | `sanitize_text_field()` |
| Textarea | `sanitize_textarea_field()` |
| Email | `sanitize_email()` |
| URL | `esc_url_raw()` |
| Integer | `absint()` or `intval()` |
| Filename | `sanitize_file_name()` |
| HTML content | `wp_kses_post()` |
| Array of values | sanitize each element individually |

```php
// CORRECT
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );

// WRONG — unsanitized input used directly
$title = $_POST['title'];
```

### wp_unslash()

WordPress adds slashes to `$_GET`, `$_POST`, and `$_REQUEST` data. Always use `wp_unslash()` before sanitizing superglobal input.

## Nonce verification

**All forms and state-changing AJAX requests must use nonces.**

### Creating nonces

```php
// In a form
wp_nonce_field( 'a8csp_save_event', 'a8csp_event_nonce' );

// For a URL
$url = wp_nonce_url( $action_url, 'a8csp_delete_event' );

// For AJAX
wp_create_nonce( 'a8csp_ajax_action' );
```

### Verifying nonces

```php
// In form processing
if ( ! isset( $_POST['a8csp_event_nonce'] )
    || ! wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['a8csp_event_nonce'] ) ), 'a8csp_save_event' )
) {
    wp_die( 'Security check failed.' );
}

// In AJAX handlers
check_ajax_referer( 'a8csp_ajax_action', 'nonce' );
```

## Capability checks

**Always verify the current user has the required capability before performing actions.**

```php
// CORRECT
if ( ! current_user_can( 'edit_posts' ) ) {
    wp_die( 'You do not have permission to perform this action.' );
}

// For post-specific checks
if ( ! current_user_can( 'edit_post', $post_id ) ) {
    wp_die( 'You do not have permission to edit this post.' );
}
```

Common capabilities:
- `manage_options` — administrator-level actions
- `edit_posts` — content editing
- `upload_files` — media uploads
- `edit_theme_options` — customizer and widget changes

## Database queries

### Use `$wpdb->prepare()` for all raw queries

```php
// CORRECT
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->posts} WHERE post_type = %s AND post_status = %s",
        'event',
        'publish'
    )
);

// WRONG — SQL injection vulnerability
$results = $wpdb->get_results(
    "SELECT * FROM {$wpdb->posts} WHERE post_type = '{$type}'"
);
```

### Prefer WordPress API functions

Before writing raw SQL, check if a WordPress function exists:

- `WP_Query` / `get_posts()` for post queries
- `get_post_meta()` / `update_post_meta()` for post meta
- `get_option()` / `update_option()` for options
- `get_terms()` for taxonomy queries
- `get_users()` for user queries

## Sensitive data handling

- **Never hardcode credentials** (API keys, passwords, tokens) in code.
- Store secrets in **1Password** and access via environment variables or WordPress options (encrypted if possible).
- Use **`wp_remote_get()`** / **`wp_remote_post()`** for HTTP requests — never `file_get_contents()` or raw `curl`.
- Never log sensitive data (passwords, tokens, PII) to debug files.
- The **Safety Net plugin** protects staging and development sites from accidentally exposing production data.

## File operations

- **Never use `eval()`** or `preg_replace()` with the `e` modifier.
- Validate file types before processing uploads.
- Use WordPress functions for file operations (`wp_upload_dir()`, `wp_handle_upload()`).
- Never allow user input to determine file paths without strict validation.

## REST API endpoints

When registering custom REST API endpoints:

```php
register_rest_route(
    'a8csp/v1',
    '/events',
    array(
        'methods'             => 'GET',
        'callback'            => 'a8csp_get_events',
        'permission_callback' => function () {
            return current_user_can( 'read' );
        },
    )
);
```

- **Always include a `permission_callback`** — never set it to `__return_true` unless the endpoint is intentionally public.
- Sanitize all input parameters.
- Escape all output data.

## Security checklist for PRs

Before merging any PR, verify:

- [ ] All user input is sanitized
- [ ] All output is escaped for its context
- [ ] Nonces are used for all state-changing operations
- [ ] Capability checks are in place for all privileged actions
- [ ] No raw SQL without `$wpdb->prepare()`
- [ ] No hardcoded secrets or credentials
- [ ] No `eval()` or dangerous PHP functions
- [ ] REST API endpoints have appropriate permission callbacks
