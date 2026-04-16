# Coding Standards

Team51 follows **WordPress Coding Standards** for all PHP and JavaScript code, with a few team-specific additions.

## PHP

### PHPCS ruleset

We use the **`WordPress-Extra`** ruleset as our PHPCS standard. This is stricter than the base `WordPress` ruleset and includes additional rules for code clarity and best practices.

The Team51 PHPCS configuration is maintained in `a8cteam51/team51-configs` and referenced in each project's `.phpcs.xml`:

```xml
<rule ref="vendor/a8cteam51/team51-configs/quality-tools/phpcs.xml.dist"/>
```

### Additional PHP quality tools

As of 2025, new projects also use:

- **PHPMD** (PHP Mess Detector) — detects code smells, unused variables, overly complex methods.
- **PHPStan** — static analysis for type safety, undefined variables, and incorrect method calls.
- **PHP Syntax Check** — ensures no syntax errors across supported PHP versions.

All of these run automatically via **GitHub Actions** on every pull request.

### PHP style rules

- **Indentation**: tabs, not spaces.
- **Brace style**: opening brace on the same line for functions and control structures.
- **Naming**:
  - Functions: `snake_case` (e.g., `a8csp_get_event_title()`)
  - Classes: `PascalCase` (e.g., `A8CSpecialProjects\CustomEvents\Plugin`)
  - Constants: `UPPER_SNAKE_CASE` (e.g., `A8CSP_CUSTOM_EVENTS_VERSION`)
  - Hooks: `snake_case` with namespace prefix (e.g., `a8csp_custom_events/event_title`)
- **String quoting**: single quotes for simple strings, double quotes only when interpolation is needed.
- **Yoda conditions**: required (e.g., `if ( true === $value )`).
- **Strict comparisons**: use `===` and `!==` instead of `==` and `!=`.

### PHPDoc

All functions, classes, and methods must have PHPDoc comments:

```php
/**
 * Retrieves the display title for an event.
 *
 * @since 1.0.0
 *
 * @param int $post_id The event post ID.
 *
 * @return string The event display title.
 */
function a8csp_get_event_title( int $post_id ): string {
    // ...
}
```

### Namespace conventions

New plugins use PHP namespaces:

```php
namespace A8CSpecialProjects\CustomEvents;
```

Theme functions typically remain in the global namespace with a prefix:

```php
function theme_slug_setup() { ... }
```

## JavaScript

### ESLint

We use **ESLint** with the `@wordpress/eslint-plugin` configuration for JavaScript and React code.

### JS style rules

- **Indentation**: tabs.
- **Semicolons**: required.
- **Quoting**: single quotes.
- **Variables**: use `const` by default, `let` when reassignment is needed, never `var`.
- **Functions**: prefer arrow functions for callbacks; named functions for top-level declarations.
- **Modules**: use ES module syntax (`import`/`export`) in block code compiled by `@wordpress/scripts`.

### Block JavaScript

For custom Gutenberg blocks:

- Use the `@wordpress/scripts` build process.
- Follow the [Block Editor Handbook](https://developer.wordpress.org/block-editor/) conventions.
- Register blocks in `src/blocks/{block-name}/` with `block.json`, `edit.js`, `save.js`, and `style.scss`.

## CSS / SCSS

- **Indentation**: tabs.
- **Naming**: use BEM-style class names or WordPress block conventions (`wp-block-{name}`).
- **Variables**: define in `theme.json` when possible; use SCSS variables for build-time values.
- **Nesting**: avoid deep nesting (max 3 levels).
- **Specificity**: keep selectors as low-specificity as possible; avoid `!important`.

## Editor configuration

All projects should include an `.editorconfig` file (provided by the project scaffold):

```ini
root = true

[*]
indent_style = tab
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

## Running checks locally

```bash
# PHP Coding Standards
composer run phpcs

# PHP Mess Detector
composer run phpmd

# PHPStan
composer run phpstan
```

> **Note**: npm script names for JavaScript linting and asset builds vary by project. Check the project's `package.json` for available scripts.
