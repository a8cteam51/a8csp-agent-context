# Pressable

Pressable is Automattic's managed WordPress hosting platform. It is Team51's **preferred platform** for most partner sites, especially those requiring custom themes, plugins, or developer tools.

## When we use Pressable

- Partner sites that need custom themes or plugins
- Sites requiring staging environments
- Projects where SSH/SFTP access is needed
- WooCommerce or other complex WordPress setups
- High-traffic sites that benefit from Pressable's infrastructure

## Key capabilities

### SSH and SFTP access
- Full SSH access to the site filesystem.
- SFTP for file transfers.
- SSH keys are managed centrally by Team51 via the Team51 CLI.

### WP-CLI access
- Full WP-CLI support via SSH.
- Use for database operations, plugin management, cache flushing, and more.

### Staging environments
- Pressable supports **multiple staging sites** per production site (unlike WPCOM Simple which allows only one).
- Staging sites can be created via the Pressable control panel (MPCP) or the Team51 CLI.
- Staging environments are useful for testing deployments, theme changes, and plugin updates before going live.
- **Important**: some Pressable features (like rewrite rules) are shared between staging and production. Flushing rewrite rules on a sandbox can affect production.

### GitHub integration
- Repos can be connected to Pressable for automated deployments.
- GitHub Deployments and DeployHQ are both used (the team is migrating toward GitHub Deployments).
- Deploy keys are managed automatically when connecting a repo.

### Caching
- Pressable has its own **server-side caching layer** (Varnish/nginx-based).
- Cache can be purged via the Pressable control panel, WP-CLI, or the Team51 CLI.
- Be mindful of caching when testing changes — purge the cache if you're not seeing updates.
- Use cache-busting version strings on CSS and JS assets — see [Asset versioning](../conventions/coding-standards.md#asset-versioning).

## Pressable control panel (MPCP)

- **My Pressable Control Panel** is the web dashboard for managing Pressable sites.
- Access error logs, PHP logs, and deployment history here.
- Site cloning, staging creation, and domain management are available in MPCP.

## Error logs and debugging

- PHP error logs are accessible via MPCP and Kibana.
- Enable `WP_DEBUG` on staging sites for detailed error output.
- Use `WP_DEBUG_LOG` to write errors to `wp-content/debug.log`.
- Production sites should **never** have `WP_DEBUG` set to `true`.

## Environment differences

| Aspect | Production | Staging |
|---|---|---|
| Caching | Aggressive server-side caching | Caching active but can be bypassed |
| Indexing | Search engines can index | `noindex` by default |
| Email | Sends real emails | May send real emails — use caution |
| Data | Live data | Clone of production at creation time |
| Deploys | From `trunk` branch | From `develop` branch (typically) |

## Site management

- All Team51 Pressable sites live under the **Special Projects Team Pressable account** (except rare cases where the partner has their own account).
- The Team51 CLI provides commands for creating sites, managing SSH keys, rotating passwords, and deploying code.
- Jetpack is installed and connected on all Pressable sites for backups, security scanning, and performance monitoring.

## Development tips

- Always work on a staging site first; never push untested code to production.
- Use SSH to inspect logs, run WP-CLI commands, and debug issues.
- Be aware of the Pressable caching layer when testing frontend changes.
- Coordinate deployments with the team via `#team51-deployers` in Slack.
- Use the Team51 Safety Net plugin on Pressable sites for additional protection on development and staging environments.
