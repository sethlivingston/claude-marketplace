# Phase 3: Polish - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Add professional metadata to plugin entries, pin marketplace references to specific versions, and set up CI validation. The marketplace should feel polished and resist accidental breakage. No new plugins or capabilities — polish what exists.

</domain>

<decisions>
## Implementation Decisions

### Metadata content
- Add metadata to BOTH plugin.json (in the-typescript-narrows repo) AND marketplace.json (in this repo)
- Plugin is self-describing; marketplace enriches for discovery
- Author: object with name ("sethlivingston") and url (GitHub profile URL)
- Homepage: https://github.com/sethlivingston/the-typescript-narrows (skill repo)
- License: MIT (matches marketplace repo)
- Keywords: minimal set of 3-5 — typescript, style-guide, code-quality, ai-coding, claude-code
- Keep existing description in marketplace.json for standalone catalog readability (don't remove it even though plugin.json also has it)
- Repository field in plugin.json points to the skill repo

### Version pinning
- Pin marketplace.json plugin entry to a git tag via the `ref` field in the source
- Use NEW tag convention: `plugin/v{semver}` (replacing old `skill/v{semver}` convention)
- Fully qualified ref in marketplace.json: `refs/tags/plugin/v1.0.0`
- Create `plugin/v1.0.0` tag on current HEAD of the-typescript-narrows repo as part of this phase
- Update all docs, scripts, and CLAUDE.md in the skill repo to reference the `plugin/` tag convention instead of `skill/`

### CI workflow
- GitHub Actions workflow in this marketplace repo
- Scope: `claude plugin validate .` only — no lint, no install test
- Triggers: push to main + PRs targeting main
- Install Claude CLI via `npm install -g @anthropic-ai/claude-code` (no version pin — always latest)
- Standard Node.js setup action for npm availability

### Claude's Discretion
- Exact workflow YAML structure and job naming
- Node.js version for CI runner
- Whether to add a status badge to README
- Exact wording of any metadata descriptions
- Order of fields in plugin.json and marketplace.json

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Current marketplace state
- `.claude-plugin/marketplace.json` — Current marketplace manifest to update with metadata and ref pinning
- `.planning/phases/02-marketplace-catalog/02-CONTEXT.md` — Phase 2 decisions on marketplace identity, plugin entry structure

### Plugin structure (skill repo)
- `../the-typescript-narrows/plugin/the-typescript-narrows/.claude-plugin/plugin.json` — Current minimal plugin manifest to enrich with metadata
- `../the-typescript-narrows/CLAUDE.md` — Repo conventions, tag references to update from skill/ to plugin/
- `.planning/phases/01-plugin-structure/01-CONTEXT.md` — Phase 1 decisions on plugin layout, naming

### Requirements
- `.planning/REQUIREMENTS.md` — POLSH-01 (metadata), POLSH-02 (version pinning), POLSH-03 (CI validation)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `marketplace.json` already has working plugin entry structure — extend with metadata fields
- `plugin.json` in skill repo has name/description/version — extend with author/homepage/license/keywords/repository

### Established Patterns
- Commit convention: `type(scope): description`
- HTTPS URLs for git sources (not SSH shorthand) — confirmed in Phase 2
- Minimal manifest approach from Phase 1 — now expanding deliberately
- Monorepo tag convention: `{artifact}/v{semver}` (changing artifact prefix from `skill` to `plugin`)

### Integration Points
- `.github/workflows/` directory needs creation in marketplace repo
- Skill repo tag convention change affects: CLAUDE.md, any release scripts, repo README
- marketplace.json `source.ref` field is the connection point for version pinning

</code_context>

<specifics>
## Specific Ideas

- User explicitly wants all docs, commands, and scripts in the skill repo updated to use `plugin/` tag convention — not just the tag itself
- Tag the current HEAD of the-typescript-narrows as `plugin/v1.0.0` during this phase
- Fully qualified ref format (`refs/tags/plugin/v1.0.0`) chosen for unambiguity

</specifics>

<deferred>
## Deferred Ideas

- Scoped marketplace naming (`@sethlivingston/...`) — user chose not to discuss, dropped from Phase 3 scope

</deferred>

---

*Phase: 03-polish*
*Context gathered: 2026-03-18*
