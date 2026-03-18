# Claude Marketplace

## What This Is

A personal Claude Code plugin marketplace hosted on GitHub at `sethlivingston/claude-marketplace`. It serves as a thin catalog that lists Seth's plugins and points to their source repositories. Coders add the marketplace with `/plugin marketplace add sethlivingston/claude-marketplace` and install individual plugins from it.

## Core Value

Coders can discover and install Seth's Claude Code plugins from a single, well-structured marketplace.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

(None yet — ship to validate)

### Active

<!-- Current scope. Building toward these. -->

- [ ] Marketplace repo with valid `.claude-plugin/marketplace.json`
- [ ] The-typescript-narrows listed as first plugin using `git-subdir` source
- [ ] The-typescript-narrows skill repo structured as a valid Claude Code plugin (`.claude-plugin/plugin.json`, `skills/` directory)
- [ ] Users can add marketplace via `/plugin marketplace add sethlivingston/claude-marketplace`
- [ ] Users can install the-typescript-narrows plugin and use the `/the-typescript-narrows` skill
- [ ] Marketplace passes `claude plugin validate .`

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
- The skill repo currently lacks plugin structure (no `.claude-plugin/plugin.json`, skills not in `skills/` subdirectory)
- The `git-subdir` source type allows sparse cloning of just the plugin subdirectory from a larger repo
- Using `strict: false` in the marketplace entry may allow defining plugin metadata entirely from the marketplace side

## Constraints

- **Plugin structure**: The skill repo needs restructuring to match Claude Code's expected plugin layout (`skills/` subdirectory, `.claude-plugin/plugin.json`)
- **Source type**: Must use `git-subdir` since the plugin lives in a subdirectory of the `the-typescript-narrows` repo
- **Naming**: Marketplace names cannot use reserved Anthropic names; must be kebab-case

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Thin catalog approach | Plugins stay in their own repos; marketplace is just a pointer | — Pending |
| Personal marketplace only | No community contributions needed at this scale | — Pending |
| git-subdir source for first plugin | Skill lives in a subdirectory of another repo | — Pending |

---
*Last updated: 2026-03-18 after initialization*
