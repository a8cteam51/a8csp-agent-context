# a8csp-agent-context

Shared AI agent context for Team51's WordPress project repositories.

## What is this?

This repo provides AI coding agents (Claude Code, Cursor, Codex, Copilot) with the knowledge they need to work effectively on Team51 projects. It contains markdown files describing our conventions, platform constraints, and workflows.

## Contents

| Folder | Purpose |
|---|---|
| `.agents/agents/` | Agent runbooks — executable procedures (flat `.md` files) |
| `.agents/handbook/` | Team51 workflows, platform guides, plugin conventions |
| `.agents/conventions/` | Coding standards, git workflow, security practices |
| `.agents/skills/` | Agent skills — reusable domain knowledge (one subdirectory per skill with `SKILL.md`) |
| `.agents/tool-configs/` | Entry-point shims for specific AI tools (Cursor) |
| `CLAUDE.md` | Claude Code entry point (references `AGENTS.md`) |
| `AGENTS.md` | Root entry point — agents read this first |

### Skills

The `.agents/skills/` directory uses one subdirectory per skill, each containing `SKILL.md`:

| Skill | Description |
|---|---|
| `managed-site-tools/` | Team51 CLI MCP tools for WordPress.com, Pressable, Jetpack, GitHub |
| `block-editor-development/` | Gutenberg blocks, block.json, InnerBlocks, @wordpress/scripts |
| `performance-patterns/` | Caching, query optimization, asset loading, image performance |
| `accessibility/` | WCAG 2.1 AA, ARIA, keyboard nav, screen readers |
| `rest-api-development/` | Custom REST endpoints, permission callbacks, schema validation |
| `wordpress/` | [WordPress/agent-skills](https://github.com/WordPress/agent-skills) (submodule) |

### Agents

The `.agents/skills/wordpress/` directory is a **git submodule**. After cloning this repo, run:

```bash
git submodule update --init --recursive
```

## Contributing

1. Create a feature branch: `feature/add-new-skill`
2. Add or edit markdown files following the existing structure
3. Keep files concise — agents have context-window limits
4. Use clear headings and bullet lists — easier for agents to parse
5. Include examples where helpful
6. Avoid internal jargon without explanation
7. Open a PR for review


## Repository structure

```
a8csp-agent-context/
├── AGENTS.md                                  ← root entry point for agents
├── CLAUDE.md                                  ← Claude Code entry point (@AGENTS.md)
└── .agents/
    ├── agents/                                ← runbooks (flat .md files)
    │   ├── site-auditor.md
    │   └── php-error-investigator.md
    ├── conventions/                           ← coding standards, git workflow, security
    ├── handbook/                              ← platform guides, plugin conventions
    ├── skills/                                ← skills (subdir + SKILL.md per skill)
    │   ├── accessibility/
    │   ├── block-editor-development/
    │   ├── managed-site-tools/
    │   ├── performance-patterns/
    │   ├── rest-api-development/
    │   └── wordpress/                         ← WordPress agent skills (git submodule)
    └── tool-configs/                          ← Cursor shims
```

## TODO

The following skills files are planned but not yet written:

- [ ] Static .cursorrules
- [ ] `.agents/handbook/theme-development.md` — Theme conventions: block themes, theme structure, theme.json patterns, build process, custom CSS strategy, accessibility and performance guidelines.
- [ ] Document npm script names (`build`, `watch`, `build:blocks`, etc.) — verify against the actual Team51 Project Scaffold `package.json` and add to the relevant handbook files.
