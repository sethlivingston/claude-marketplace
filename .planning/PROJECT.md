# Claude Marketplace

## What This Is

A personal Claude Code plugin marketplace hosted on GitHub at `sethlivingston/claude-marketplace`. It serves as a thin catalog that lists Seth's plugins and points to their source repositories. Coders add the marketplace with `/plugin marketplace add sethlivingston/claude-marketplace` and install individual plugins from it.

## Core Value

Coders can discover and install Seth's Claude Code plugins from a single, well-structured marketplace.

## Requirements

### Validated

- ✓ Marketplace repo with valid `.claude-plugin/marketplace.json` — v1.0
- ✓ The-typescript-narrows listed as first plugin using `git-subdir` source — v1.0
- ✓ The-typescript-narrows skill repo structured as a valid Claude Code plugin — v1.0
- ✓ Users can add marketplace and discover plugins — v1.0
- ✓ Users can install the-typescript-narrows and use the skill — v1.0
- ✓ Marketplace passes `claude plugin validate .` — v1.0
- ✓ Plugin and marketplace metadata enriched (author, homepage, license, keywords) — v1.0
- ✓ Plugin ref pinned to `plugin/v1.0.0` tag — v1.0
- ✓ CI validation workflow on push/PR — v1.0

### Active

- [ ] Second plugin added to validate catalog pattern at n>1 (EXPN-01)
- [ ] Release channels for plugins (stable vs latest refs) (EXPN-02)
- [ ] Team distribution via `extraKnownMarketplaces` (EXPN-03)

### Out of Scope

- Community contributions — this is a personal marketplace
- Plugin hosting within the marketplace repo — plugins live in their own repos
- npm/pip distribution — Git-based sources only for now
- Offline mode — marketplace requires GitHub access by design

## Context

Shipped v1.0 with 76 LOC across JSON, YAML, and Markdown deliverables (33 commits, 1 day).
Tech stack: Claude Code plugin system, GitHub Actions, `git-subdir` source type.
One plugin published: the-typescript-narrows (pinned to `plugin/v1.0.0`).

## Constraints

- **Source type**: Must use `git-subdir` since the plugin lives in a subdirectory of the `the-typescript-narrows` repo
- **Naming**: Marketplace names cannot use reserved Anthropic names; must be kebab-case

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Thin catalog approach | Plugins stay in their own repos; marketplace is just a pointer | ✓ Good |
| Personal marketplace only | No community contributions needed at this scale | ✓ Good |
| git-subdir source for first plugin | Skill lives in a subdirectory of another repo | ✓ Good |
| HTTPS URLs for plugin sources | GitHub shorthand defaults to SSH, which fails without SSH keys | ✓ Good |
| plugin/v* tag convention | Replaced skill/v* for semantic clarity; old tag preserved | ✓ Good |
| Unpinned Claude Code CLI in CI | No version field in marketplace.json; CI uses latest CLI | ⚠️ Revisit |
| Omitted $schema from marketplace.json | Validation passes without it; minor tech debt | ⚠️ Revisit |

---
*Last updated: 2026-03-18 after v1.0 milestone*
