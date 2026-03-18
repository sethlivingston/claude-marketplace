# Phase 2: Marketplace Catalog - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Create `.claude-plugin/marketplace.json` in the marketplace repo and validate end-to-end installation of the-typescript-narrows plugin. Users should be able to add the marketplace and see the plugin available for install. This phase works in the marketplace repo (sethlivingston/claude-marketplace).

</domain>

<decisions>
## Implementation Decisions

### Marketplace identity
- Name: `sethlivingston-plugins`
- Owner: `sethlivingston`
- Description: short and functional, e.g. "Seth Livingston's Claude Code plugins"
- User wanted npm-style scoped name (`@sethlivingston/claude-marketplace`) but unclear if Claude Code supports that format — deferred investigation to Phase 3

### Plugin entry content
- Minimum fields only: name, source (no duplicated description — let Claude Code read it from the plugin's own plugin.json)
- Source type: `git-subdir` pointing to repo `sethlivingston/the-typescript-narrows`, subdirectory `plugin/the-typescript-narrows`
- No ref pinning — use default branch. Version pinning deferred to Phase 3 (POLSH-02)
- No keywords or extra metadata — deferred to Phase 3 (POLSH-01)

### End-to-end validation
- Two-step: run `claude plugin validate .` first, then live install test
- Push marketplace.json to GitHub, then test with `claude plugin marketplace add sethlivingston/claude-marketplace` (test the real remote flow)
- Success = `/the-typescript-narrows` skill appears in available skills after install
- If anything fails, debug and fix in-phase — this is the whole point of Phase 2

### Repo extras
- Minimal README.md — brief description, refer readers to marketplace.json for plugin list. No duplication of info. Few people will visit this repo directly
- Minimal .gitignore — standard ignores (.DS_Store, node_modules, etc.)
- MIT License
- CLAUDE.md — project conventions for Claude Code

### Claude's Discretion
- Exact README wording
- CLAUDE.md content and conventions
- .gitignore contents
- marketplace.json field ordering
- How to structure the validate-then-test workflow

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Plugin structure (from Phase 1)
- `../the-typescript-narrows/plugin/the-typescript-narrows/.claude-plugin/plugin.json` — Plugin manifest with name/description to reference (not duplicate)
- `.planning/phases/01-plugin-structure/01-CONTEXT.md` — Phase 1 decisions on plugin layout, naming, git-subdir path

### Claude Code plugin system
- No external plugin spec docs — use `claude plugin validate .` output as authoritative structure check
- Use `claude plugin marketplace --help` and `claude plugin install --help` for CLI interface

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — marketplace repo is currently empty (only .git and .planning)

### Established Patterns
- Phase 1 established minimal manifest approach (only required fields)
- Commit convention: `type(scope): description`
- Naming consistency: `the-typescript-narrows` everywhere

### Integration Points
- `.claude-plugin/marketplace.json` is the main deliverable — Claude Code reads this when users add the marketplace
- Plugin source points to `sethlivingston/the-typescript-narrows` repo's `plugin/the-typescript-narrows` subdirectory
- After push to GitHub, users interact via `claude plugin marketplace add sethlivingston/claude-marketplace`

</code_context>

<specifics>
## Specific Ideas

- User considered npm-style scoped naming (`@sethlivingston/claude-marketplace`) — worth investigating feasibility in Phase 3
- README should be minimal and not duplicate marketplace.json content — just point readers to the JSON
- This is a personal marketplace with few direct visitors — optimize for function over presentation

</specifics>

<deferred>
## Deferred Ideas

- Scoped marketplace naming (`@sethlivingston/...`) — investigate in Phase 3
- Rich metadata (keywords, author, homepage) — Phase 3 (POLSH-01)
- Version pinning to specific ref/sha — Phase 3 (POLSH-02)

</deferred>

---

*Phase: 02-marketplace-catalog*
*Context gathered: 2026-03-18*
