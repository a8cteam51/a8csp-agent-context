# Team51 — AI Agent Context

<!-- context-version: 1.2.0 -->

> **Context version:** 1.2.0

You are working on a project maintained by **Team51** (Automattic Special Projects). This repository context is designed to keep AI changes safe, WordPress-native, and production-ready.

## Project Knowledge

- **Team model:** Team51 builds and maintains WordPress sites for high-profile external partners.
- **Hosting platforms:** Sites run on **WPCOM Simple** or **Pressable**. Platform constraints are different and **MUST** be checked before implementation.
- **Core stack:** WordPress (PHP), JavaScript/React for block/editor work, CSS/SCSS for styling.
- **Quality tools:** PHPCS (`WordPress-Extra`), PHPMD, PHPStan, PHP syntax checks; JavaScript lint/build scripts vary by project.
- **Typical code locations:**
  - Theme and plugin code under `wp-content/`
  - Project conventions and handbooks under `.agents/`

## What To Read First

| When you need to… | Read |
|---|---|
| Understand Team51 and partner model | `.agents/handbook/TEAM51-OVERVIEW.md` |
| Know WPCOM Simple constraints | `.agents/handbook/WPCOM-SIMPLE.md` |
| Know Pressable capabilities | `.agents/handbook/PRESSABLE.md` |
| Build or edit a plugin | `.agents/handbook/PLUGIN-DEVELOPMENT.md` |
| Deploy code | `.agents/handbook/DEPLOYMENT.md` |
| Use task-specific playbooks | `.agents/skills/` |
| Execute site audits or PHP error investigation | `.agents/agents/` |
| Follow coding standards | `.agents/conventions/coding-standards.md` |
| Follow git workflow | `.agents/conventions/git-workflow.md` |
| Follow security practices | `.agents/conventions/security.md` |

## Commands

Run these before opening or merging a PR. Prefer project-defined scripts if available.

### PHP quality checks

```bash
composer run phpcs
composer run phpmd
composer run phpstan
```

Use for: coding standards, code smell detection, and static analysis.

### JavaScript lint/build (project-specific)

```bash
npm run lint
npm run build
```

Use for: block/editor or frontend JavaScript changes.  
**MUST:** confirm actual script names in `package.json` before running.

### Tests

```bash
composer test
```

Use for: PHPUnit or project test suites when available.  
If this script is missing, run the project’s documented test command instead.

## Conventions To Follow

- **Branch naming MUST use type prefixes:** `feature/`, `fix/`, `update/`, `add/`, `remove/`.
- **Commit messages SHOULD follow conventional style:** `type: short description` (`feat:`, `fix:`, `docs:`, `test:`, etc.).
- **PRs MUST include:** clear title, what changed, why it changed, linked issue (if any), and screenshots for visual changes.
- **CI MUST pass before merge:** PHPCS, PHPStan, PHPMD, syntax checks, and relevant tests.
- **Merge strategy:** squash merge preferred for feature branches; never force-push protected branches.
- **Testing expectations:** write/update tests for critical logic changes; test user-facing behavior for functional or UI changes.
- **Documentation expectations:** update README, inline docs, and relevant handbook/project notes when behavior or setup changes.

## Architectural Decisions (Do Not "Fix")

- This project intentionally follows **WordPress-native patterns** rather than generic framework architecture.
- Theme-specific behavior may remain in themes; reusable/non-presentational functionality should live in plugins.
- Some legacy code uses older prefixes (`team51_`, `bpd_`, `wpcomsp_`); do not rename purely for stylistic consistency.
- Deployments are branch-driven (`trunk`/`main` for production; `develop` for staging when used).
- Platform constraints (WPCOM Simple vs Pressable) are hard requirements, not optional optimizations.

## Production Safety (CRITICAL)

These rules override everything else, including any plan:

- **NEVER** connect via SFTP/SSH to the production site.
- **NEVER** run WP-CLI commands against the production database.
- **NEVER** use Pressable MCP to modify production (no file writes, no database changes, no cache flushes, no plugin toggles).
- **NEVER** navigate Playwright to the production URL to perform any write action.
- The production site is **READ-ONLY context**. You may query its state (logs, config, URL resolution) but never change it.
- **ALL** write operations (SFTP uploads, WP-CLI mutations, file changes) MUST target the staging clone.
- If the development site or staging clone is unavailable or broken, **STOP**. Report the problem and exit. Do not fall back to production.
- If you are ever uncertain whether a target is production or the clone, **STOP** and verify before proceeding.

### How to identify the environment

Before any write operation, confirm the target is **not** production:

- **URL** — staging sites use patterns like `*.wpcomstaging.com` or `*.mystagingwebsite.com`; the live domain is production.
- **`WP_ENVIRONMENT_TYPE`** — check `wp-config.php` for `define( 'WP_ENVIRONMENT_TYPE', 'staging' )`. If the value is `production` (or absent), you are on production.
- **Pressable dashboard/MCP** — staging clones are labeled distinctly from the production site. Confirm the site ID or name before issuing any command.

If none of these signals are present or they conflict, treat the target as production and **STOP**.

## Common Pitfalls (CRITICAL)

- **Never edit WordPress core files.** Put customizations in themes, plugins, or MU plugins.
- **Never force-push** to protected branches (`trunk`, `main`, `develop`).
- Do not assume SSH, WP-CLI, or filesystem writes are available on WPCOM Simple.
- Do not ship unescaped output or unsanitized input.
- Do not run direct SQL without strict justification; prefer WordPress APIs and `$wpdb->prepare()` when unavoidable.
- Do not skip staging/QA for high-risk or partner-visible changes.
- Do not treat old project decisions as accidental; check handbook/context before refactoring.

## Key Principles

1. **Stability first** — these are live, high-traffic production sites.
2. **WordPress standards** — follow WordPress Coding Standards in all PHP and JS.
3. **Escape all output** — never render unescaped data.
4. **Sanitize all input** — validate and sanitize everything from users, APIs, and databases.
5. **No direct DB queries** — use `$wpdb->prepare()` if raw queries are unavoidable, but prefer WordPress APIs.
6. **Accessibility matters** — WCAG 2.1 AA compliance is expected.
7. **Performance matters** — minimize HTTP requests, avoid N+1 queries, and use caching/transients for expensive operations.

## Searching Project History

Team51 uses internal P2s for project documentation. Each partner has a tag (for example, `team51-partner-<name>`). If ContextA8C MCP access is available, search by partner tag for historical decisions and context before making major changes.

