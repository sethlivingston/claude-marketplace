# Project Research Summary

**Project:** claude-marketplace (personal Claude Code plugin marketplace)
**Domain:** Git-hosted JSON catalog for Claude Code plugin distribution
**Researched:** 2026-03-18
**Confidence:** HIGH

## Executive Summary

This project is not a traditional software application. It is a thin JSON catalog repository that Claude Code uses to discover and install plugins. There is no runtime, no build step, no dependencies, and no compilation — just a `marketplace.json` file that points to external plugin repos and a `README.md` for humans. The entire "stack" is the file structure conventions enforced by Claude Code's plugin system, validated by `claude plugin validate .`. Experts build this by keeping the marketplace repo minimal (catalog only) and hosting each plugin in its own repo, using `git-subdir` source references for plugins that live in subdirectories of larger repos.

The recommended approach is to restructure the `the-typescript-narrows` skill repo first (adding `.claude-plugin/plugin.json` and organizing skills into the expected `skills/` directory layout), then write `marketplace.json` to point at it, then validate end-to-end. The key insight from architecture research is the hard build-order dependency: the plugin source must be valid before the marketplace can reference it meaningfully. Attempting to write `marketplace.json` before fixing the skill repo layout is the most common early mistake.

The primary risk is the source repo's current invalid structure — PROJECT.md explicitly notes it lacks `.claude-plugin/plugin.json` and the expected directory layout. All eight documented pitfalls cluster into Phase 1: choose a non-colliding, non-reserved marketplace name; decide strict mode strategy before writing either JSON file; pin the `git-subdir` path to exactly the directory containing `.claude-plugin/`; and ensure `SKILL.md` has valid YAML frontmatter. These are all preventable with one thorough Phase 1 pass and a final `claude plugin validate .` run before publishing.

## Key Findings

### Recommended Stack

This project has zero runtime dependencies. The "stack" is a set of JSON file format conventions: `marketplace.json` at `.claude-plugin/marketplace.json` in the marketplace repo, and `plugin.json` at `.claude-plugin/plugin.json` in each plugin's root directory. Skills are Markdown files (`SKILL.md`) with YAML frontmatter. The only tooling required is the Claude Code CLI (>= 1.0.33) for `claude plugin validate .` and Git >= 2.25 for sparse checkout support (required by `git-subdir`). No `package.json`, no linter, no bundler.

**Core technologies:**
- JSON (`marketplace.json`, `plugin.json`): The entire distribution mechanism — required by Claude Code, no alternatives
- Git / GitHub: Hosting and distribution; Claude Code fetches via `git clone` / sparse checkout
- Claude Code CLI >= 1.0.33: The only validation and testing tool; `claude plugin validate .` is the build step
- Markdown with YAML frontmatter (`SKILL.md`): Skill definitions; two-phase loading (description into context budget, full content on invocation)
- `git-subdir` source type: Sparse-clones only the plugin subdirectory from a larger repo — required for `the-typescript-narrows` which lives in a subdirectory

### Expected Features

All P1 features are structural, not behavioral — they are about correctly forming files and directories, not writing code.

**Must have (table stakes):**
- Valid `marketplace.json` with `name`, `owner`, and `plugins[]` — Claude Code rejects the marketplace entirely without it
- At least one installable plugin (`the-typescript-narrows`) — an empty marketplace is useless
- Plugin descriptions on each marketplace entry — required for Discover tab
- Working end-to-end installation — `marketplace.json` can be valid but the plugin still fails if the source repo lacks correct structure
- `claude plugin validate .` passing cleanly — users and CI depend on this

**Should have (competitive):**
- Rich plugin metadata (`author`, `homepage`, `keywords`, `license`, `category`, `tags`) — makes the marketplace feel professional
- Version pinning with `ref`/`sha` in `git-subdir` source — reproducibility for users
- GitHub Actions CI running `claude plugin validate .` on push — catches breakage before users hit it
- Second plugin added — validates the catalog pattern at n > 1

**Defer (v2+):**
- `strict: false` mode — only needed if a specific plugin should not have its own `plugin.json`
- Release channels (stable vs latest refs) — only if user base grows
- Team distribution via `extraKnownMarketplaces` — only if recommending to teams

### Architecture Approach

The architecture is a thin-catalog pattern: the marketplace repo contains zero plugin code and functions purely as a JSON index. Each plugin lives in its own external repository and is referenced via source type (`git-subdir` for plugins in repo subdirectories, `github` for plugins that are entire repos). Claude Code's runtime handles everything at consumption time: reading `marketplace.json`, sparse-cloning plugin sources, copying to a versioned cache at `~/.claude/plugins/cache/`, and loading skill descriptions into the context budget. The marketplace operator never ships code — only pointers.

**Major components:**
1. `marketplace.json` (`.claude-plugin/marketplace.json`) — the catalog; maps plugin names to source locations, descriptions, and metadata
2. Plugin repo structure (external) — `.claude-plugin/plugin.json` manifest + `skills/<name>/SKILL.md`; must be self-contained within the `git-subdir` path boundary
3. `SKILL.md` with YAML frontmatter — the actual deliverable; description loads into context budget, full content loads on invocation
4. Claude Code runtime — fetches, caches, and loads plugins; the marketplace operator has no control over this layer

### Critical Pitfalls

All eight documented pitfalls are Phase 1 concerns. The most dangerous are silent failures where the plugin installs but nothing works.

1. **Plugin name collides with marketplace name** — name them distinctly from the start (e.g., marketplace `seth-plugins`, plugin `the-typescript-narrows`); collision causes `EXDEV` errors on Linux
2. **Plugin directory structure does not match Claude Code expectations** — restructure source repo to `.claude-plugin/plugin.json` + `skills/<name>/SKILL.md` before writing `marketplace.json`; missing structure means skills never load
3. **`strict` mode conflict kills plugin loading silently** — if the plugin repo has `plugin.json` with component declarations, use `strict: true` (default); using `strict: false` with a component-declaring `plugin.json` causes silent load failure
4. **`git-subdir` path points to wrong directory level** — the `path` field must point to the directory that directly contains `.claude-plugin/`; one level too high or too low yields zero skills with no error
5. **Malformed `SKILL.md` frontmatter** — skill installs but never appears as a slash command; validate with `claude plugin validate .` and check the Errors tab after install
6. **Using a reserved marketplace name** — avoid any name containing `claude`, `anthropic`, or `official` as prefixes; `claude plugin validate .` will catch this

## Implications for Roadmap

The architecture research is explicit about build order: plugin repo structure must precede marketplace catalog, which must precede validation. This drives a natural two-phase structure, with an optional polish phase after the core works.

### Phase 1: Plugin Repo Restructuring

**Rationale:** The marketplace cannot work until the source plugin is a valid Claude Code plugin. This is the blocker for everything else. PROJECT.md confirms the skill repo currently lacks the required structure.

**Delivers:** A properly structured plugin at `skill/the-typescript-narrows/` in the source repo, with `.claude-plugin/plugin.json` and `skills/the-typescript-narrows/SKILL.md` (and supporting `references/` files) all self-contained within the `git-subdir` target directory.

**Addresses:** Working plugin installation, plugin validation passes, table-stakes features

**Avoids:**
- Pitfall 3 (directory structure mismatch)
- Pitfall 5 (files outside plugin boundary)
- Pitfall 8 (malformed SKILL.md frontmatter)

### Phase 2: Marketplace Catalog Setup

**Rationale:** With a valid plugin in hand, `marketplace.json` is a simple JSON file — but several naming and configuration decisions must be made correctly on the first pass to avoid breaking published install commands later.

**Delivers:** A valid `.claude-plugin/marketplace.json` with correct marketplace name, plugin entry pointing to the restructured source repo via `git-subdir`, descriptions, and passing `claude plugin validate .`. Users can add the marketplace and install the plugin end-to-end.

**Addresses:** Valid marketplace.json, plugin descriptions, end-to-end installation

**Avoids:**
- Pitfall 1 (name collision between marketplace and plugin)
- Pitfall 4 (wrong git-subdir path)
- Pitfall 6 (reserved marketplace name)
- Pitfall 7 (version set in both places)
- Pitfall 2 (strict mode conflict — decide strategy here)

### Phase 3: Polish and Stability

**Rationale:** Once the core flow works end-to-end, add metadata enrichment and validation automation to make the marketplace feel professional and prevent future breakage.

**Delivers:** Rich plugin metadata (`author`, `homepage`, `keywords`, `license`), version pinning with `ref` in `git-subdir` source, and a GitHub Actions workflow running `claude plugin validate .` on push.

**Addresses:** Plugin metadata richness, version pinning, CI validation (all P2 features)

**Avoids:** Unpinned sources that could silently break users when the skill repo changes

### Phase Ordering Rationale

- Plugin repo first because `marketplace.json` source references must point to a structurally valid plugin; validating the marketplace validates the full chain
- All pitfall-avoidance happens in Phases 1 and 2; by the time Phase 3 starts, the core is working and changes are low-risk
- Phase 3 is deliberately optional — the marketplace is fully functional after Phase 2; polish adds professionalism but not correctness

### Research Flags

Phases with standard, well-documented patterns (skip deeper research-phase):
- **Phase 1:** Plugin structure is fully documented in official Claude Code docs; the directory layout and `plugin.json` schema are unambiguous
- **Phase 2:** `marketplace.json` schema is fully documented; `git-subdir` source type is documented with examples
- **Phase 3:** GitHub Actions, version pinning, and metadata fields are all documented; no novel patterns required

No phases need `/gsd:research-phase` during planning. All questions are answered by the official documentation consulted in this research cycle.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Sourced from official Claude Code docs and the official Anthropic plugin marketplace repo; schema is authoritative |
| Features | HIGH | Sourced from official docs + two real-world community marketplace examples; feature set is simple and well-defined |
| Architecture | HIGH | Official docs describe the thin-catalog pattern explicitly; data flow verified against real marketplace.json from anthropics/claude-plugins-official |
| Pitfalls | HIGH | Eight pitfalls sourced from official docs, confirmed GitHub issues (26555, 24389), and explicit doc warnings |

**Overall confidence:** HIGH

### Gaps to Address

- **Exact `plugin.json` schema for the-typescript-narrows:** The plugin manifest fields (`name`, `description`, `version`, `skills`) are documented but the exact component declaration syntax should be verified against `claude plugin validate .` output during Phase 1 execution. This is a verification concern, not a knowledge gap.
- **GitHub shorthand vs full URL in git-subdir:** Docs confirm `owner/repo` shorthand works for `git-subdir` `url` field, but full HTTPS URL (`https://github.com/owner/repo.git`) is the safe fallback if shorthand fails. Test during Phase 2.
- **Git submodule limitation:** A confirmed GitHub issue (#17293) notes that `git-subdir` does not follow git submodules. Relevant only if `the-typescript-narrows` repo uses submodules (unlikely but worth checking during Phase 1).

## Sources

### Primary (HIGH confidence)
- [Create and distribute a plugin marketplace - Claude Code Docs](https://code.claude.com/docs/en/plugin-marketplaces) — marketplace schema, source types, validation, hosting
- [Create plugins - Claude Code Docs](https://code.claude.com/docs/en/plugins) — plugin structure, manifest schema, skills format, testing workflow
- [Extend Claude with skills - Claude Code Docs](https://code.claude.com/docs/en/skills) — SKILL.md format, progressive loading model
- [Discover and install plugins - Claude Code Docs](https://code.claude.com/docs/en/discover-plugins) — installation flow, user-facing commands
- [anthropics/claude-plugins-official marketplace.json](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) — official reference implementation with 72+ plugins

### Secondary (MEDIUM confidence)
- [Plugin template by ivan-magda](https://github.com/ivan-magda/claude-code-plugin-template) — community best practice, GitHub Actions validation example
- [Personal marketplace by dashed](https://github.com/dashed/claude-marketplace) — similar personal-marketplace use case
- [Demo marketplace template](https://github.com/mrlm-xyz/demo-claude-marketplace) — community example

### Tertiary (confirmed bugs, HIGH confidence for specific issues)
- [BUG: category and source keys leak into cached plugin.json - Issue #26555](https://github.com/anthropics/claude-code/issues/26555)
- [DOCS: Plugin name collision with marketplace name - Issue #24389](https://github.com/anthropics/claude-code/issues/24389)
- [BUG: Marketplace installation doesn't clone git submodules - Issue #17293](https://github.com/anthropics/claude-code/issues/17293)

---
*Research completed: 2026-03-18*
*Ready for roadmap: yes*
