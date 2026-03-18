# Phase 1: Plugin Structure - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Restructure the-typescript-narrows skill repo into a valid, self-contained Claude Code plugin. The skill currently lives at `skill/the-typescript-narrows/` and needs `.claude-plugin/plugin.json`, a `skills/` subdirectory, and must pass `claude plugin validate .`. This phase works in the skill repo, not the marketplace repo.

</domain>

<decisions>
## Implementation Decisions

### Plugin root location
- Plugin root is `plugin/the-typescript-narrows/` (rename parent directory from `skill/` to `plugin/`)
- Inner directory name stays `the-typescript-narrows` — matches repo name, skill name, and invocation command
- The marketplace (Phase 2) will use `git-subdir` pointing to `plugin/the-typescript-narrows/`

### Manifest contents
- Minimum viable plugin.json: name, description, version only
- Name: `the-typescript-narrows` (matches skill name in SKILL.md frontmatter)
- Version: `1.0.0` (matches latest skill tag `skill/v1.0.0`)
- Additional metadata (author, homepage, repository, license, keywords) deferred to Phase 3 (POLSH-01)

### File reorganization
- Use `git mv` to preserve file history through the restructuring
- LICENSE and README.md move to plugin root (`plugin/the-typescript-narrows/`)
- SKILL.md and references/ move into `skills/the-typescript-narrows/` inside the plugin root
- Update ALL references to `skill/` across the repo — CLAUDE.md, repo README.md, scripts, and any other files that reference the old path
- This restructuring is the exemplary template for future plugins — correctness over convenience

### Validation approach
- Run `claude plugin validate .` from inside `plugin/the-typescript-narrows/` (the actual plugin root)
- Validation only — no test install in this phase (that's Phase 2, PLUG-04)

### Claude's Discretion
- Description text for plugin.json (should be concise, derived from SKILL.md)
- Order of operations for the git mv commands
- Whether to update the skill release tag convention after rename

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Skill repo structure
- `../the-typescript-narrows/CLAUDE.md` — Repo layout, conventions, release process (references `skill/` directory that will be renamed)
- `../the-typescript-narrows/skill/the-typescript-narrows/SKILL.md` — Skill frontmatter with name/description (source for plugin.json fields)
- `../the-typescript-narrows/skill/the-typescript-narrows/README.md` — Skill-level README that moves to plugin root

### Claude Code plugin system
- No external plugin spec docs referenced — use `claude plugin validate .` output as the authoritative structure check

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `SKILL.md` frontmatter contains name and description — reuse for plugin.json fields
- Existing `LICENSE` and `README.md` in skill directory — relocate to plugin root

### Established Patterns
- Repo uses prefixed git tags: `skill/v{semver}` and `eslint-plugin/v{semver}`
- Commit convention: `type(scope): description` (e.g., `feat(skill): ...`)
- Monorepo with independent artifacts: eslint-plugin, skill (soon plugin), docs

### Integration Points
- CLAUDE.md repo layout section references `skill/` — needs update to `plugin/`
- Repo root README.md references `skill/` in layout — needs update
- `.github/workflows/` may reference skill paths — check during implementation
- `scripts/generate-traceability.mjs` — check for skill path references

</code_context>

<specifics>
## Specific Ideas

- User wants this to be the exemplary template for future plugins — every path, name, and reference must be correct
- Naming consistency is a priority: the-typescript-narrows everywhere (directory, skill name, plugin name, repo name)

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-plugin-structure*
*Context gathered: 2026-03-18*
