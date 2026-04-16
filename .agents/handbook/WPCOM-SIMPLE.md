# WPCOM Simple

WordPress.com Simple sites run directly on Automattic's WordPress.com infrastructure. They are the most constrained hosting option Team51 uses.

## When we use WPCOM Simple

- Internal Automattic sites
- Very simple partner sites that can use existing WordPress.com themes
- Projects with low/no maintenance requirements
- Sites that do not need third-party plugins or custom PHP beyond what WPCOM allows

## Key constraints

### No SSH access
You cannot SSH into a WPCOM Simple site. There is no filesystem access outside of the WordPress admin and the WPCOM API.

### Limited plugin access
- **No custom plugin uploads** unless explicitly allowed by WordPress.com.
- Only plugins available in the WordPress.com marketplace can be activated.
- Some plugin functionality is replicated by built-in WPCOM features (e.g., SEO, forms, social sharing).
- Team51-developed plugins can be added as MU plugins on Team51-flagged sites via special arrangement.

### No WP-CLI access
WP-CLI is not available on Simple sites. Any operations requiring WP-CLI must use the WPCOM REST API or the WordPress admin.

### Limited PHP configuration
- No access to `php.ini` or `.htaccess`.
- No custom cron jobs — use `wp_cron` or the WPCOM cron system.
- No custom rewrite rules outside of what WordPress provides (note: WPCOM adds overhead to rewrites that doesn't exist on self-hosted sites).

### Theme restrictions
- Custom themes must be compatible with the WordPress.com environment.
- Block themes are preferred and work best on Simple.
- Classic themes may have limitations with certain template hierarchies.

## WordPress.com-specific APIs and features

- **WPCOM REST API** — use for programmatic site management.
- **Jetpack integration** — Simple sites have Jetpack features built in (stats, backups, security).
- **WordPress.com Block Editor** — full Gutenberg support with WPCOM-specific blocks.
- **Global Styles** — theme.json-based styling works well on Simple.
- **GitHub Deployments** — WPCOM now supports deploying theme/plugin code from GitHub repos.

## Deployment

- Code is deployed via **GitHub Deployments** integration (WPCOM connects directly to the GitHub repo).
- The `trunk` branch typically maps to the production site.
- See `DEPLOYMENT.md` for the full workflow.

## Common gotchas

1. **SVG uploads** — not natively supported. Use the Safe SVG plugin if available, or inline SVGs in theme code.
2. **Permalink quirks** — WPCOM adds layers onto rewrite rules that don't exist locally. Test permalink structures on the actual site.
3. **Caching** — WPCOM has its own aggressive caching layer. You cannot clear it programmatically the same way you would on Pressable.
4. **Plugin auto-updates** — managed by WordPress.com; you don't control update timing.
5. **Content freeze during migrations** — coordinate with the partner to avoid content changes during site moves.
6. **Local development differences** — local environments (Studio, Local, VVV) won't replicate all WPCOM Simple constraints. Test on a WPCOM staging site when possible.
7. **Staging** — WPCOM allows only **one staging site per production site**. Plan accordingly.

## Development tips

- Start development on a WPCOM Business or Creator plan site to validate compatibility early.
- Use the WPCOM REST API for any automation instead of relying on filesystem access.
- Avoid code that depends on filesystem writes (`file_put_contents`, custom log files, etc.).
- Keep themes lean — avoid bundling large libraries; use WordPress core APIs instead.
- Test thoroughly on the actual WPCOM environment; local dev won't catch all incompatibilities.
