# Team51 Overview

## What is Team51?

Team51 is **Automattic's Special Projects team**. We build and maintain WordPress websites for high-profile partners — publishers, brands, organizations, and public figures.

## The partner model

- **Partners are external clients**, not Automattic employees.
- Each partner has a dedicated GitHub repository, P2 tag, and Slack channel.
- We handle everything from initial build to ongoing maintenance.
- Communication with partners happens through Zendesk, Slack Connect, or email — never through internal-only channels.

## What we build

- Custom WordPress themes (block themes preferred for new builds)
- Custom plugins for partner-specific functionality
- Site migrations (from other platforms to WordPress)
- Performance optimizations and ongoing support

## Hosting platforms

All Team51 sites live on one of two Automattic-owned platforms:

| Platform | Best for | Key traits |
|---|---|---|
| **WPCOM Simple** | Internal sites, simple partner sites without custom plugins | No SSH, limited plugin access, WordPress.com infrastructure |
| **Pressable** | Most partner sites needing custom themes/plugins | SSH access, staging environments, GitHub deploys |

See `WPCOM-SIMPLE.md` and `PRESSABLE.md` for platform-specific details.

## Our priorities

1. **Stability** — sites are live and often high-traffic; breaking changes are unacceptable.
2. **Performance** — fast load times, efficient queries, proper caching.
3. **Accessibility** — WCAG 2.1 AA compliance.
4. **Security** — defense in depth; escape, sanitize, verify.
5. **Maintainability** — clean, documented, standards-compliant code that other developers can pick up.

## Key tools

- **GitHub** (github.a8c.com/a8cteam51) — all project repos live here.
- **Team51 CLI** — internal CLI tool for managing sites, deployments, and infrastructure across Pressable and WPCOM.
- **1Password** — shared credentials and secrets.
- **Linear** — issue tracking and project management.
- **P2** — internal blogs for project documentation and async communication.
- **PHPCS** — automated code quality checks on every PR via GitHub Actions.

## Shared resources

- **Team51 Project Scaffold** (`a8cteam51/team51-project-scaffold`) — standard folder structure and build scripts for new projects.
- **Team51 Plugin Scaffold** (`a8cteam51/team51-plugin-scaffold`) — starter structure for custom plugins.
- **Special Projects Blocks Monorepo** (`a8cteam51/special-projects-blocks-monorepo`) — shared custom blocks reused across projects.
- **Team51 Patterns** — approved block patterns shared across partner sites.
