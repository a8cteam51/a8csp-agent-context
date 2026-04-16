# Plugin Development

## When to create a plugin

Create a custom plugin when:

- The functionality is **not theme-specific** (it should persist if the theme changes).
- The feature involves **custom post types, taxonomies, REST API endpoints, or admin tools**.
- The code is **reusable across multiple partner sites**.
- The partner needs **WooCommerce customizations** or integrations with third-party services.

Keep functionality in the theme when:

- It is **purely presentational** (template modifications, style overrides).
- It only makes sense in the context of the current theme (block patterns, template parts).

## Plugin scaffold

Use the **Team51 Plugin Scaffold** (`a8cteam51/team51-plugin-scaffold`) for all new plugins. It provides:

- Standardized folder structure
- Composer dependencies with PHPCS, PHPMD, and PHPStan pre-configured
- GitHub Actions for automated code quality checks
- PHPUnit test setup with examples

## Plugin structure

A typical Team51 plugin follows this structure:

```
plugin-name/
├── .github/
│   └── workflows/           ← GitHub Actions (PHPCS, PHPStan, tests)
├── assets/
│   ├── css/
│   └── js/
├── src/
│   ├── Plugin.php           ← Main plugin class
│   ├── Hooks.php            ← Hook registrations
│   └── ...                  ← Feature-specific classes
├── tests/
│   └── ...                  ← PHPUnit tests
├── vendor/                  ← Composer dependencies (gitignored)
├── composer.json
├── package.json             ← If the plugin includes blocks or frontend assets
├── plugin-name.php          ← Plugin bootstrap file
├── uninstall.php            ← Cleanup on uninstall
└── README.md
```

## Naming conventions

- **Plugin slug**: `a8csp-{feature-name}` (e.g., `a8csp-custom-events`)
- **PHP namespace**: `A8CSpecialProjects\{FeatureName}` (e.g., `A8CSpecialProjects\CustomEvents`)
- **Function prefix**: `a8csp_{feature_name}_` for non-namespaced functions
- **Hook prefix**: `a8csp_{feature_name}/` for custom hooks
- **Text domain**: matches the plugin slug (e.g., `a8csp-custom-events`)

> **Note**: Some older plugins use the `team51_`, `bpd_`, or `wpcomsp_` prefix. New plugins should use `a8csp_`.

## Hooks and filters

- **Prefer filters over actions** when the goal is to modify data.
- **Namespace all custom hooks** using the plugin slug: `apply_filters( 'a8csp_custom_events/event_title', $title )`
- **Document hooks** with PHPDoc comments so other developers (and AI agents) can discover them.
- **Use WordPress core hooks** before creating custom ones — check if a core hook already exists for your use case.

Example:

```php
/**
 * Filters the event display title.
 *
 * @param string $title   The event title.
 * @param int    $post_id The event post ID.
 */
$title = apply_filters( 'a8csp_custom_events/event_title', $title, $post_id );
```

## Team51 shared plugins

Before building new functionality, check if a shared plugin already exists:

- **Safety Net** — protects development and staging environments from accidental data leaks.
- **Simple Events** — lightweight event management.
- **Plugin Autoupdate Filter** — controls auto-update behavior for specific plugins.
- **Background Tasks** (`a8csp-background-tasks`) — manages async background processing.

These are maintained in the Team51 GitHub organization.

## Quality assurance

All plugins must pass these automated checks (configured in the scaffold):

1. **PHPCS** — WordPress-Extra ruleset with Team51 configs
2. **PHPMD** — PHP Mess Detector for code quality
3. **PHPStan** — static analysis for type safety and bug detection
4. **PHP Syntax Check** — ensures no syntax errors across supported PHP versions
5. **PHPUnit** — unit tests (write tests for critical functionality)

## WPCOM Simple compatibility

If a plugin needs to run on WPCOM Simple:

- Avoid filesystem writes.
- Do not depend on WP-CLI or SSH access.
- Test on an actual WPCOM site — local environments won't catch all incompatibilities.
- Plugins may need to be deployed as MU plugins via special arrangement with WordPress.com.

## Activation and deactivation

- Use `register_activation_hook` for setup tasks (creating database tables, setting default options, flushing rewrite rules).
- Use `register_deactivation_hook` for cleanup (removing scheduled events, flushing rewrite rules).
- Implement `uninstall.php` to remove all plugin data when a user deletes the plugin.
