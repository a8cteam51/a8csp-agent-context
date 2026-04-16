---
name: rest-api-development
description: >
  WordPress REST API development patterns for Team51 projects following
  WordPress-native conventions. Activate when registering custom endpoints,
  building API integrations, handling REST authentication, permission callbacks,
  schema validation, or WP_REST_Response/WP_Error patterns.
---

# REST API Development

## Overview

Team51 uses the WordPress REST API for custom endpoints, third-party integrations, and decoupled frontend features. All API work follows WordPress-native patterns — `register_rest_route()`, `WP_REST_Controller`, proper permission callbacks, and schema validation. No external routing frameworks.

## Namespace Conventions

- **Namespace**: `a8csp/v{version}` (e.g., `a8csp/v1`).
- **Routes**: lowercase, plural nouns, hyphen-separated (e.g., `/events`, `/event-categories`).
- **Versioning**: increment the version number for breaking changes. Non-breaking additions do not require a new version.
- **Partner-specific namespaces**: `a8csp-{partner-slug}/v1` when endpoints are unique to a single partner project.

```php
register_rest_route( 'a8csp/v1', '/events', array( ... ) );
register_rest_route( 'a8csp/v1', '/events/(?P<id>\d+)', array( ... ) );
```

## Endpoint Registration

### Basic registration

```php
add_action( 'rest_api_init', function () {
    register_rest_route(
        'a8csp/v1',
        '/events',
        array(
            array(
                'methods'             => WP_REST_Server::READABLE,
                'callback'            => 'a8csp_get_events',
                'permission_callback' => 'a8csp_events_read_permission',
                'args'                => a8csp_get_events_args(),
            ),
            array(
                'methods'             => WP_REST_Server::CREATABLE,
                'callback'            => 'a8csp_create_event',
                'permission_callback' => 'a8csp_events_write_permission',
                'args'                => a8csp_create_event_args(),
            ),
            'schema' => 'a8csp_get_event_schema',
        )
    );
} );
```

### Use REST constants

| Constant | HTTP Method |
|---|---|
| `WP_REST_Server::READABLE` | GET |
| `WP_REST_Server::CREATABLE` | POST |
| `WP_REST_Server::EDITABLE` | POST, PUT, PATCH |
| `WP_REST_Server::DELETABLE` | DELETE |
| `WP_REST_Server::ALLMETHODS` | All methods |

## Permission Callbacks

**Every endpoint must have an explicit `permission_callback`.** This is both a WordPress requirement and a security mandate.

```php
function a8csp_events_read_permission() {
    return true; // Public endpoint — intentional
}

function a8csp_events_write_permission() {
    return current_user_can( 'edit_posts' );
}
```

Rules:

- **Never omit `permission_callback`** — WordPress emits a `_doing_it_wrong` notice and the endpoint is insecure.
- **Never use `__return_true`** without a comment justifying that the endpoint is intentionally public.
- **Use capability checks** (`current_user_can()`) for protected endpoints.
- **For per-object permissions**, check capabilities against the specific resource:

```php
function a8csp_update_event_permission( WP_REST_Request $request ) {
    $post = get_post( $request['id'] );
    if ( ! $post ) {
        return new WP_Error( 'rest_not_found', __( 'Event not found.', 'a8csp-custom-events' ), array( 'status' => 404 ) );
    }
    return current_user_can( 'edit_post', $post->ID );
}
```

## Argument Validation and Sanitization

Define `args` for every endpoint with `validate_callback` and `sanitize_callback`:

```php
function a8csp_get_events_args() {
    return array(
        'per_page' => array(
            'type'              => 'integer',
            'default'           => 10,
            'minimum'           => 1,
            'maximum'           => 100,
            'sanitize_callback' => 'absint',
            'validate_callback' => 'rest_validate_request_arg',
        ),
        'category' => array(
            'type'              => 'string',
            'sanitize_callback' => 'sanitize_text_field',
            'validate_callback' => 'rest_validate_request_arg',
        ),
        'status' => array(
            'type'    => 'string',
            'default' => 'publish',
            'enum'    => array( 'publish', 'draft', 'future' ),
        ),
    );
}
```

- **Always provide both `validate_callback` and `sanitize_callback`** (or use `rest_validate_request_arg` with type/enum constraints).
- Use WordPress REST schema types: `string`, `integer`, `number`, `boolean`, `array`, `object`.
- Use `enum` for fixed value sets. Use `minimum`/`maximum` for numeric bounds.
- Use `required => true` for mandatory parameters.

## Schema

Define a JSON Schema for each resource. This enables auto-discovery, documentation, and validation:

```php
function a8csp_get_event_schema() {
    return array(
        '$schema'    => 'http://json-schema.org/draft-04/schema#',
        'title'      => 'event',
        'type'       => 'object',
        'properties' => array(
            'id'    => array(
                'description' => __( 'Unique identifier.', 'a8csp-custom-events' ),
                'type'        => 'integer',
                'context'     => array( 'view', 'edit' ),
                'readonly'    => true,
            ),
            'title' => array(
                'description' => __( 'Event title.', 'a8csp-custom-events' ),
                'type'        => 'string',
                'context'     => array( 'view', 'edit' ),
                'required'    => true,
            ),
            'date'  => array(
                'description' => __( 'Event date in ISO 8601 format.', 'a8csp-custom-events' ),
                'type'        => 'string',
                'format'      => 'date-time',
                'context'     => array( 'view', 'edit' ),
            ),
        ),
    );
}
```

Use `context` to control which fields appear in different views (`view` = public, `edit` = authenticated).

## Response Formatting

### Success responses

Return `WP_REST_Response` with explicit status codes:

```php
function a8csp_get_events( WP_REST_Request $request ) {
    $events = get_posts( array(
        'post_type'      => 'event',
        'posts_per_page' => $request['per_page'],
    ) );

    $data = array_map( 'a8csp_prepare_event_response', $events );

    $response = new WP_REST_Response( $data, 200 );
    $response->header( 'X-WP-Total', count( $data ) );

    return $response;
}
```

### Error responses

Return `WP_Error` with a machine-readable code, human-readable message, and HTTP status:

```php
return new WP_Error(
    'a8csp_event_not_found',
    __( 'The requested event does not exist.', 'a8csp-custom-events' ),
    array( 'status' => 404 )
);
```

Error code conventions:

- Prefix with the plugin namespace: `a8csp_{error_description}`.
- Use standard HTTP status codes in the `data` array.
- Provide actionable error messages.

## WP_REST_Controller Pattern

For complex resources, extend `WP_REST_Controller` to get a structured class with standard methods:

```php
class A8CSP_Events_Controller extends WP_REST_Controller {
    protected $namespace = 'a8csp/v1';
    protected $rest_base = 'events';

    public function register_routes() { ... }
    public function get_items( $request ) { ... }
    public function get_item( $request ) { ... }
    public function create_item( $request ) { ... }
    public function update_item( $request ) { ... }
    public function delete_item( $request ) { ... }
    public function get_item_schema() { ... }
    public function get_item_permissions_check( $request ) { ... }
    public function get_items_permissions_check( $request ) { ... }
}
```

Use this pattern when a resource has full CRUD operations. For simple one-off endpoints (webhooks, utility actions), standalone `register_rest_route()` is fine.

## Caching REST Responses

REST responses are cached by the platform page cache (Pressable Varnish, WPCOM cache). For public endpoints:

- Set appropriate `Cache-Control` headers for cacheable responses.
- Use `ETag` or `Last-Modified` headers for conditional requests when data freshness matters.
- For authenticated or user-specific responses, the platform cache typically won't cache (due to cookies). No special handling needed.

For server-side caching of expensive queries behind an endpoint, use transients:

```php
function a8csp_get_events( WP_REST_Request $request ) {
    $cache_key = 'a8csp_rest_events_' . md5( wp_json_encode( $request->get_params() ) );
    $data      = get_transient( $cache_key );

    if ( false === $data ) {
        $data = expensive_events_query( $request );
        set_transient( $cache_key, $data, 5 * MINUTE_IN_SECONDS );
    }

    return new WP_REST_Response( $data, 200 );
}
```

## Authentication

WordPress REST API authentication methods available to Team51:

| Method | Use case |
|---|---|
| **Cookie + nonce** | Frontend JS on the same domain (logged-in users). Use `wp_create_nonce( 'wp_rest' )` and send as `X-WP-Nonce` header. |
| **Application Passwords** | External integrations, CI/CD pipelines, third-party services. |
| **WPCOM OAuth** | WPCOM Simple sites using the WordPress.com REST API. |

- Never implement custom token-based auth when WordPress-native methods suffice.
- For public endpoints, authentication is not required but the `permission_callback` must still be defined.

## WPCOM Considerations

- The WPCOM REST API (`/wp/v2/...` and `/wpcom/v2/...`) is available on WPCOM Simple sites. Custom endpoints deployed via GitHub Deployments work on both platforms.
- WPCOM adds its own REST API extensions. Check for namespace collisions.
- The Jetpack REST API proxy allows accessing site REST endpoints through WordPress.com authentication. Be aware this adds latency.

## Boundaries

- For input sanitization and output escaping specifics, see `security.md`.
- For PHP coding standards and naming conventions, see `coding-standards.md`.
- For caching patterns beyond REST, see `performance-patterns` skill.
- For plugin structure and where to place endpoint code, see `PLUGIN-DEVELOPMENT.md`.
