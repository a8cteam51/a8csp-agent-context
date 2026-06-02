# a8csp-agent-context

Shared AI agent context for Team51 WordPress project repositories.

This repository gives coding agents such as Claude Code, Cursor, Codex, and
GitHub Copilot a compact source of Team51 conventions, platform constraints,
WordPress patterns, and production-safety rules. It is a documentation/context
repository, not a WordPress theme, plugin, or deployable application.

## Repository Contents

| Path | Purpose |
| --- | --- |
| `AGENTS.md` | Primary entry point for agents. Read this first for Team51 project context, production-safety rules, and links to the rest of the repository. |
| `CLAUDE.md` | Claude Code shim that points back to `AGENTS.md`. |
| `.agents/handbook/` | Team51 operating guides for WPCOM Simple, Pressable, deployment workflows, plugin development, and the Team51 partner model. |
| `.agents/conventions/` | Coding standards, git workflow, and security guidance for Team51 WordPress work. |
| `.agents/skills/` | Reusable agent skills for accessibility, block editor development, performance patterns, REST API development, and the WordPress agent-skills submodule. |
| `.agents/tool-configs/.cursorrules` | Cursor entry-point shim that tells Cursor to load this context. |
| `.gitmodules` | Defines the `.agents/skills/wordpress` submodule. |

## Agent Entry Points

Start with `AGENTS.md`. It includes the context version, required production
safety rules, the read-first map, and the command expectations agents should use
inside downstream Team51 project repositories.

Tool-specific entry points are intentionally thin:

- `CLAUDE.md` contains `@AGENTS.md` for Claude Code.
- `.agents/tool-configs/.cursorrules` tells Cursor to load `AGENTS.md`,
  handbook files, skills, and conventions.

## Skills

Local skills live under `.agents/skills/{skill-name}/SKILL.md`:

| Skill | Focus |
| --- | --- |
| `accessibility` | WCAG 2.1 AA, semantic HTML, keyboard behavior, ARIA, forms, media, and block accessibility. |
| `block-editor-development` | Gutenberg block structure, `block.json`, dynamic blocks, InnerBlocks, patterns, variations, and `@wordpress/scripts` conventions. |
| `performance-patterns` | Query optimization, caching, asset loading, image performance, and Pressable/WPCOM cache behavior. |
| `rest-api-development` | WordPress REST route registration, permission callbacks, schema validation, responses, authentication, and caching. |
| `wordpress` | External WordPress agent skills provided as a git submodule from `WordPress/agent-skills`. |

After cloning, initialize the submodule before relying on the `wordpress` skill:

```sh
git submodule update --init --recursive
```

## Runtime And Validation

This repository tracks Markdown context files and a git submodule pointer only.
It does not define a Composer project, npm package, lockfile, GitHub Actions
workflow, deployment config, or markdown lint command.

For README or context-only changes, run:

```sh
git diff --check
```

Commands such as `composer run phpcs`, `composer run phpstan`, `composer test`,
`npm run lint`, and `npm run build` are documented here as expectations for
downstream Team51 project repositories. They are not available in this context
repository unless a future change adds the corresponding manifests.

## Maintenance Notes

- Keep `AGENTS.md` as the primary read-first file.
- Add new local skills as `.agents/skills/{skill-name}/SKILL.md`.
- Keep handbook files under `.agents/handbook/` and cross-link them from
  `AGENTS.md` when they become required read-first material.
- Keep conventions under `.agents/conventions/` and make security guidance easy
  for agents to discover.
- Update tool shims such as `CLAUDE.md` and `.agents/tool-configs/.cursorrules`
  when the entry-point structure changes.
- Do not document generated, vendor, build, or dependency directories unless
  they are intentionally tracked.

## Contributing

1. Create a branch with a Team51 git-workflow prefix such as `feature/`,
   `fix/`, `update/`, `add/`, or `remove/`.
2. Add or edit the relevant Markdown context files.
3. Keep files concise and easy for agents to parse.
4. Use clear headings, short lists, and examples where they help.
5. Avoid internal jargon unless it is explained in the same file.
6. Run `git diff --check`.
7. Open a pull request for review.
