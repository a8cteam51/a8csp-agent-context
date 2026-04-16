# Git Workflow

## Repository hosting

All Team51 project repositories live on **github.a8c.com** under the `a8cteam51` organization.

## Branch naming

Use descriptive branch names with a type prefix:

| Prefix | Use for |
|---|---|
| `feature/` | New features or enhancements |
| `fix/` | Bug fixes |
| `update/` | Dependency updates, content changes, minor improvements |
| `add/` | Adding new files, components, or integrations |
| `remove/` | Removing features or deprecated code |

Examples:
- `feature/partner-newsletter-signup`
- `fix/mobile-nav-overflow`
- `update/phpcs-config`
- `add/custom-events-plugin`

## Main branches

| Branch | Purpose |
|---|---|
| `trunk` (or `main`) | Production-ready code. Deploys to the live site. |
| `develop` | Integration branch. Deploys to the staging site. |

- **`trunk`** is the default and protected branch.
- **`develop`** is used for staging; not all projects have a `develop` branch.
- Feature branches are created from `trunk` (or `develop` if one exists).

## Pull request process

1. **Create a feature branch** from `trunk` (or `develop`).
2. **Make your changes** and commit with clear messages.
3. **Push the branch** to GitHub.
4. **Open a pull request** targeting `trunk` (or `develop`).
5. **CI checks run automatically** — PHPCS, PHPStan, PHPMD, syntax checks.
6. **Request review** if the change is significant. For low-risk changes (copy updates, minor CSS fixes), self-review and merge is acceptable.
7. **Merge the PR** using **squash merge** (preferred) to keep the commit history clean.

### PR guidelines

- Write a clear PR title that summarizes the change.
- Include a description of **what** changed and **why**.
- Link to the relevant Linear issue or GitHub issue if one exists.
- Include screenshots for visual changes.
- Ensure all CI checks pass before merging.

## Commit messages

We prefer **conventional commit style** messages:

```
type: short description

Longer explanation if needed. Explain the "why" not just the "what".

Refs: #123
```

Common types:
- `feat:` — new feature
- `fix:` — bug fix
- `update:` — enhancement to existing feature
- `refactor:` — code restructuring without behavior change
- `style:` — formatting, whitespace, missing semicolons (no code change)
- `docs:` — documentation changes
- `test:` — adding or updating tests
- `chore:` — build process, dependency updates, config changes

Examples:
- `feat: add newsletter signup block pattern`
- `fix: resolve mobile navigation overflow on iOS Safari`
- `update: bump PHPCS config to latest team51-configs`

## Merge strategy

- **Squash merge** is preferred for feature branches — it collapses all branch commits into a single clean commit on the target branch.
- **Regular merge** (no squash) is used for release branches or when preserving individual commit history matters.
- **Never rebase or force-push** `trunk` or `develop`.

## When to target `develop` vs `trunk`

- **Target `develop`** when the project uses a staging workflow and you want changes tested on staging before production.
- **Target `trunk` directly** for:
  - Hotfixes that need to go live immediately
  - Projects that don't use a `develop` branch
  - Very low-risk changes (copy, config)

## Syncing `develop` and `trunk`

When `develop` has been tested and is ready for production:

1. Open a PR from `develop` to `trunk`.
2. Use a **regular merge** (not squash) to preserve the commit history from develop.
3. After merging, the production deployment triggers automatically.

## GitHub Actions

Every Team51 project should have these GitHub Actions workflows:

- **PHP Coding Standards** (`php-coding-standards-pull_request.yml`) — runs PHPCS on every PR.
- **PHP Syntax Check** — validates PHP syntax across supported versions.
- **PHPMD** — runs PHP Mess Detector.
- **PHPStan** — runs static analysis.

These are all provided by the Team51 Project Scaffold and Plugin Scaffold.

## `.gitignore` conventions

The project scaffold provides a standard `.gitignore`. Key entries:

- `vendor/` — Composer dependencies (installed via CI or locally)
- `node_modules/` — npm dependencies
- `.env` — environment-specific config (never commit secrets)
- `wp-content/uploads/` — user uploads (not tracked in Git)
- `wp-content/themes/*/` with exceptions for the project theme — ensures only the project theme is tracked
