# Architecture Research

**Domain:** Claude Code Plugin Marketplace (personal, thin catalog)
**Researched:** 2026-03-18
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Marketplace Repository                        │
│  (sethlivingston/claude-marketplace)                             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  .claude-plugin/marketplace.json                          │    │
│  │  - name, owner, plugins[]                                 │    │
│  │  - Each plugin entry: name, source, description           │    │
│  └────────────────────────┬─────────────────────────────────┘    │
│                           │                                      │
│              (source references point outward)                   │
└───────────────────────────┼──────────────────────────────────────┘
                            │
              ┌─────────────┴─────────────────┐
              │                               │
              ▼                               ▼
┌──────────────────────┐        ┌──────────────────────────┐
│  Plugin Repo A       │        │  Plugin Repo B           │
│  (external GitHub)   │        │  (external GitHub)       │
│                      │        │                          │
│  └─ subdirectory/    │        │  └─ .claude-plugin/      │
│     └─ .claude-      │        │     └─ plugin.json       │
│        plugin/       │        │  └─ skills/              │
│        └─plugin.json │        │     └─ skill-name/       │
│     └─ skills/       │        │        └─ SKILL.md       │
│        └─ skill/     │        │                          │
│           └─SKILL.md │        │                          │
└──────────────────────┘        └──────────────────────────┘
              │                               │
              └───────────┬───────────────────┘
                          │
                          ▼
              ┌──────────────────────┐
              │  Claude Code Client  │
              │                      │
              │  1. Registers mktpl  │
              │  2. Clones/fetches   │
              │  3. Caches plugins   │
              │     at ~/.claude/    │
              │     plugins/cache/   │
              │  4. Loads skills     │
              └──────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Implementation |
|-----------|----------------|----------------|
| `marketplace.json` | Catalog of available plugins with source pointers | Single JSON file at `.claude-plugin/marketplace.json` |
| Plugin entry (in marketplace) | Declares plugin name, source location, description, and optional overrides | Object in the `plugins[]` array |
| Plugin repo (external) | Houses actual plugin code: skills, commands, agents, hooks | Separate GitHub repository with `.claude-plugin/plugin.json` |
| `plugin.json` | Plugin manifest declaring name, version, description, and component paths | JSON file at `<plugin-root>/.claude-plugin/plugin.json` |
| `SKILL.md` | Skill definition with YAML frontmatter and instructions | Markdown file at `<plugin-root>/skills/<skill-name>/SKILL.md` |
| Claude Code runtime | Marketplace registration, plugin fetching, caching, skill loading | Built into Claude Code client |

## Recommended Project Structure

### Marketplace Repository

```
claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json     # The catalog -- this IS the marketplace
├── .planning/               # Project planning (not shipped)
│   └── ...
├── CLAUDE.md                # Instructions for Claude when working on this repo
└── README.md                # Human-readable docs for the marketplace
```

### Plugin Repository (each external plugin)

```
the-typescript-narrows/            # Plugin lives in a subdirectory of this repo
├── skill/
│   └── the-typescript-narrows/    # The plugin root (git-subdir target)
│       ├── .claude-plugin/
│       │   └── plugin.json        # Plugin manifest
│       └── skills/
│           └── the-typescript-narrows/
│               ├── SKILL.md       # Skill entry point
│               └── references/    # Supporting files loaded on demand
│                   ├── guide-1.md
│                   └── guide-2.md
```

### Structure Rationale

- **Marketplace repo is minimal:** It contains only `marketplace.json` and documentation. No plugin code lives here. This is the "thin catalog" approach described in PROJECT.md.
- **Plugin repos are self-contained:** Each plugin has its own `.claude-plugin/plugin.json` and follows the standard plugin directory layout. They can be developed, tested, and validated independently.
- **`git-subdir` source type:** Because the first plugin (`the-typescript-narrows`) lives in a subdirectory of a larger repo, the marketplace uses `git-subdir` to sparse-clone only the plugin directory, minimizing bandwidth.

## Architectural Patterns

### Pattern 1: Thin Catalog Marketplace

**What:** The marketplace repo contains zero plugin code. It is purely a JSON catalog that points to external plugin sources via `git-subdir`, `github`, or `url` source types.

**When to use:** Personal or small-team marketplaces where plugins already have their own repos. This is the right choice for this project.

**Trade-offs:**
- Pro: No code duplication, each plugin evolves independently
- Pro: Marketplace repo stays tiny and easy to maintain
- Con: Users depend on external repos being accessible
- Con: No atomic updates across marketplace + plugin

### Pattern 2: Strict vs Non-Strict Plugin Entries

**What:** The `strict` field controls whether `plugin.json` in the plugin repo is authoritative (`strict: true`, default) or whether the marketplace entry defines everything (`strict: false`).

**When to use:** Use `strict: true` (default) when the plugin repo has a well-formed `plugin.json`. Use `strict: false` when the marketplace needs to define plugin metadata because the source repo does not have its own `plugin.json`, or when the marketplace operator wants full control over component exposure.

**Trade-offs:**
- `strict: true`: Plugin author controls the manifest; marketplace supplements
- `strict: false`: Marketplace operator controls everything; plugin repo is raw files. However, if the plugin repo also declares components in `plugin.json`, the plugin fails to load (conflict)

**Recommendation for this project:** Use `strict: true` (default). Structure the plugin repo with a proper `plugin.json` so it can also work standalone outside the marketplace.

### Pattern 3: Progressive Skill Loading

**What:** Claude Code uses a two-phase loading model. First, skill descriptions (~100 tokens each) are loaded into context so Claude knows what is available. Full skill content (<5k tokens recommended) loads only when the skill is invoked or deemed relevant. Supporting files (references, templates) load only when Claude reads them.

**When to use:** Always -- this is how Claude Code works. Design skills with this in mind.

**Trade-offs:**
- Pro: Efficient context usage -- dozens of skills can coexist
- Con: Skill descriptions must be written carefully to trigger correct matching
- Keep `SKILL.md` under 500 lines; put detailed content in supporting files

## Data Flow

### Marketplace Registration and Plugin Installation

```
User runs: /plugin marketplace add sethlivingston/claude-marketplace
    │
    ▼
Claude Code clones marketplace repo
    │
    ▼
Reads .claude-plugin/marketplace.json
    │
    ▼
Registers marketplace in ~/.claude/plugins/known_marketplaces.json
    │
    ▼
User runs: /plugin install the-typescript-narrows@seth-plugins
    │
    ▼
Claude Code reads plugin entry from marketplace.json
    │
    ▼
Resolves source: { source: "git-subdir", url: "...", path: "..." }
    │
    ▼
Sparse-clones only the plugin subdirectory from the source repo
    │
    ▼
Copies plugin to ~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/
    │
    ▼
Plugin is now available -- skills appear in / menu
```

### Skill Invocation Flow

```
User types /the-typescript-narrows (or Claude auto-detects relevance)
    │
    ▼
Claude Code loads SKILL.md content into context
    │
    ▼
Claude follows instructions in SKILL.md
    │
    ▼
If SKILL.md references supporting files (e.g., references/guide.md)
    │
    ▼
Claude reads those files on demand using Read tool
    │
    ▼
Claude applies the skill's guidance to the user's code
```

### Plugin Update Flow

```
User runs: /plugin marketplace update
    │
    ▼
Claude Code pulls latest marketplace.json
    │
    ▼
Compares plugin versions (marketplace version vs cached version)
    │
    ▼
If version changed: re-fetches plugin from source, updates cache
```

### Key Data Flows

1. **Marketplace discovery:** User adds marketplace URL -> Claude Code clones repo -> reads `marketplace.json` -> stores reference in `known_marketplaces.json`.
2. **Plugin installation:** User selects plugin -> Claude Code resolves source -> fetches plugin files -> copies to versioned cache.
3. **Skill activation:** Session starts -> skill descriptions loaded into context budget -> user/Claude invokes skill -> full SKILL.md loaded -> supporting files loaded on demand.

## Build Order (Dependencies)

Components must be built in this order due to dependencies:

```
Phase 1: Plugin repo structure
    │     (the-typescript-narrows must be a valid plugin first)
    │
    ▼
Phase 2: Marketplace repo structure
    │     (marketplace.json points to the now-valid plugin)
    │
    ▼
Phase 3: Validation and testing
          (validate both, test install flow end-to-end)
```

**Rationale:**
1. **Plugin repo first** because the marketplace is just a pointer. You cannot validate the marketplace if the plugin it points to is not properly structured. The skill repo currently lacks plugin structure (no `.claude-plugin/plugin.json`, skills not in `skills/` subdirectory per PROJECT.md).
2. **Marketplace repo second** because `marketplace.json` references the plugin. The source URL, path, and plugin name must match the actual structure created in Phase 1.
3. **Validation last** because `claude plugin validate .` checks the entire chain: marketplace schema, plugin references, and component structure.

## Anti-Patterns

### Anti-Pattern 1: Embedding Plugin Code in the Marketplace

**What people do:** Copy plugin files directly into the marketplace repository.
**Why it is wrong:** Creates code duplication. Plugin updates require changes in two places. Defeats the purpose of a thin catalog.
**Do this instead:** Use source references (`github`, `git-subdir`, `url`) to point to external repos.

### Anti-Pattern 2: Using `strict: false` When the Plugin Has Its Own `plugin.json`

**What people do:** Set `strict: false` in the marketplace entry while the plugin repo also has a `plugin.json` that declares components.
**Why it is wrong:** Claude Code detects the conflict and the plugin fails to load entirely.
**Do this instead:** Use `strict: true` (default) and let `plugin.json` in the plugin repo be authoritative. The marketplace entry can supplement with additional metadata.

### Anti-Pattern 3: Referencing Files Outside the Plugin Directory

**What people do:** Use `../shared-utils` or other paths that escape the plugin root.
**Why it is wrong:** Plugins are copied to a cache directory at install time. Only files within the plugin directory are copied. External references break silently.
**Do this instead:** Keep all plugin files self-contained within the plugin root. If sharing is needed, use symlinks (which are followed during copy).

### Anti-Pattern 4: Oversized SKILL.md Files

**What people do:** Put all reference material directly in SKILL.md.
**Why it is wrong:** SKILL.md loads fully into context when invoked. Large files waste context budget and can push past the skill character budget (2% of context window).
**Do this instead:** Keep SKILL.md under 500 lines. Put detailed reference material in supporting files (e.g., `references/`) that Claude reads on demand.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| GitHub | `git-subdir` source in `marketplace.json` | Sparse-clones only the target subdirectory. Plugin repo must be public (or user must have git credentials configured). |
| Claude Code CLI | `claude plugin validate .` | Validates marketplace and plugin structure. Run from marketplace root. |
| Claude Code client | `/plugin marketplace add`, `/plugin install` | End-user commands that consume the marketplace. |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Marketplace <-> Plugin repo | JSON source reference (one-way) | Marketplace points to plugin; plugin knows nothing about marketplace |
| Plugin repo <-> Claude Code | File structure convention | Claude Code expects `.claude-plugin/plugin.json` + `skills/` directory |
| SKILL.md <-> Supporting files | File path references in markdown | Claude reads supporting files on demand via Read tool |

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1-5 plugins | Single marketplace.json, manual version management, no automation needed |
| 5-20 plugins | Consider adding `metadata.pluginRoot` to DRY up source paths. Still manual. |
| 20+ plugins | Add categories and tags for discoverability. Consider CI to auto-validate on push. |

### Scaling Priorities

1. **First concern (not a bottleneck):** Plugin count. At personal scale this will not be an issue. The official Anthropic marketplace has 72+ plugins in a single JSON file without problems.
2. **Second concern:** Version management. If plugins update frequently, tracking versions manually becomes tedious. The `ref` and `sha` fields in source entries provide pinning, but there is no built-in automation. A GitHub Action running `claude plugin validate .` on PRs would be the first automation to add.

## Sources

- [Create and distribute a plugin marketplace - Claude Code Docs](https://code.claude.com/docs/en/plugin-marketplaces) (HIGH confidence -- official docs)
- [Extend Claude with skills - Claude Code Docs](https://code.claude.com/docs/en/skills) (HIGH confidence -- official docs)
- [Anthropic official marketplace.json](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) (HIGH confidence -- official reference implementation)
- [Demo marketplace template](https://github.com/mrlm-xyz/demo-claude-marketplace) (MEDIUM confidence -- community example)

---
*Architecture research for: Claude Code Plugin Marketplace*
*Researched: 2026-03-18*
