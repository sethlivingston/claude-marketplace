# Claude Marketplace

## What This Is

A personal Claude Code plugin marketplace hosted on GitHub at `sethlivingston/claude-marketplace`. It serves as a thin catalog that lists Seth's plugins and points to their source repositories. Coders add the marketplace with `/plugin marketplace add sethlivingston/claude-marketplace` and install individual plugins from it.

## Core Value

Coders can discover and install Seth's Claude Code plugins from a single, well-structured marketplace.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- [x] Marketplace repo with valid `.claude-plugin/marketplace.json` — Validated in Phase 2: marketplace-catalog
- [x] The-typescript-narrows listed as first plugin using `git-subdir` source — Validated in Phase 2: marketplace-catalog
- [x] The-typescript-narrows skill repo structured as a valid Claude Code plugin (`.claude-plugin/plugin.json`, `skills/` directory) — Validated in Phase 1: plugin-structure
- [x] Users can add marketplace via `/plugin marketplace add sethlivingston/claude-marketplace` — Validated in Phase 2: marketplace-catalog
- [x] Users can install the-typescript-narrows plugin and use the `/the-typescript-narrows` skill — Validated in Phase 2: marketplace-catalog
- [x] Marketplace passes `claude plugin validate .` — Validated in Phase 2: marketplace-catalog
- [x] Plugin and marketplace metadata enriched (author, homepage, license, keywords) — Validated in Phase 3: polish
- [x] Plugin ref pinned to `plugin/v1.0.0` tag — Validated in Phase 3: polish
- [x] CI validation workflow runs `claude plugin validate .` on PRs — Validated in Phase 3: polish

### Active

<!-- Current scope. Building toward these. -->

(All active requirements validated)

### Out of Scope

- Community contributions — this is a personal marketplace
- Plugin hosting within the marketplace repo — plugins live in their own repos
- npm/pip distribution — Git-based sources only for now
- CI/CD automation — manual updates are fine at this scale

## Context

- Claude Code plugin marketplaces are defined by a `.claude-plugin/marketplace.json` file at the repo root
- Plugins can be sourced from GitHub repos, git URLs, git subdirectories, npm, or relative paths
- The first plugin (`the-typescript-narrows`) currently lives at `sethlivingston/the-typescript-narrows` in the `skill/the-typescript-narrows` subdirectory
- The skill has a `SKILL.md` with frontmatter and `references/` directory with detailed TypeScript opinion guides
- The skill repo has been restructured as a valid Claude Code plugin with `.claude-plugin/plugin.json` and `skills/` subdirectory
- The `git-subdir` source type allows sparse cloning of just the plugin subdirectory from a larger repo
- Tag convention migrated from `skill/v*` to `plugin/v*`; current release: `plugin/v1.0.0`

## Constraints

- **Plugin structure**: The skill repo needs restructuring to match Claude Code's expected plugin layout (`skills/` subdirectory, `.claude-plugin/plugin.json`)
- **Source type**: Must use `git-subdir` since the plugin lives in a subdirectory of the `the-typescript-narrows` repo
- **Naming**: Marketplace names cannot use reserved Anthropic names; must be kebab-case

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Thin catalog approach | Plugins stay in their own repos; marketplace is just a pointer | Confirmed |
| Personal marketplace only | No community contributions needed at this scale | Confirmed |
| git-subdir source for first plugin | Skill lives in a subdirectory of another repo | Confirmed |
| HTTPS URLs for plugin sources | GitHub shorthand defaults to SSH, which fails without SSH keys | Confirmed |

---
*Last updated: 2026-03-18 after Phase 3 completion — all milestone phases complete*
