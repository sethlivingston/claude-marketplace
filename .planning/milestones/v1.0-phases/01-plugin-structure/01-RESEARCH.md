# Phase 1: Plugin Structure - Research

**Researched:** 2026-03-18
**Domain:** Claude Code plugin system -- manifest structure, directory layout, validation
**Confidence:** HIGH

## Summary

Phase 1 restructures the-typescript-narrows skill repo into a valid Claude Code plugin. The work happens entirely in the skill repo (at `../the-typescript-narrows/`), not in the marketplace repo. The core tasks are: (1) rename `skill/` to `plugin/`, (2) create `.claude-plugin/plugin.json` inside the plugin root, (3) move skill files into the correct `skills/the-typescript-narrows/` subdirectory within the plugin root, and (4) update all references to the old `skill/` path across the repo.

The Claude Code plugin system is well-documented with a clear, minimal manifest schema. The `name` field is the only required manifest field. The plugin directory structure has strict rules: `.claude-plugin/plugin.json` contains the manifest; all component directories (`skills/`, `commands/`, `agents/`, etc.) must be at the plugin root, NOT inside `.claude-plugin/`. Validation via `claude plugin validate <path>` is the authoritative check.

**Primary recommendation:** Create a minimal `plugin.json` with name, description, and version. Restructure files with `git mv` to preserve history. Validate with `claude plugin validate .` from within the plugin root directory.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Plugin root is `plugin/the-typescript-narrows/` (rename parent directory from `skill/` to `plugin/`)
- Inner directory name stays `the-typescript-narrows` -- matches repo name, skill name, and invocation command
- The marketplace (Phase 2) will use `git-subdir` pointing to `plugin/the-typescript-narrows/`
- Minimum viable plugin.json: name, description, version only
- Name: `the-typescript-narrows` (matches skill name in SKILL.md frontmatter)
- Version: `1.0.0` (matches latest skill tag `skill/v1.0.0`)
- Additional metadata (author, homepage, repository, license, keywords) deferred to Phase 3 (POLSH-01)
- Use `git mv` to preserve file history through the restructuring
- LICENSE and README.md move to plugin root (`plugin/the-typescript-narrows/`)
- SKILL.md and references/ move into `skills/the-typescript-narrows/` inside the plugin root
- Update ALL references to `skill/` across the repo -- CLAUDE.md, repo README.md, scripts, and any other files that reference the old path
- This restructuring is the exemplary template for future plugins -- correctness over convenience
- Run `claude plugin validate .` from inside `plugin/the-typescript-narrows/` (the actual plugin root)
- Validation only -- no test install in this phase (that's Phase 2, PLUG-04)

### Claude's Discretion
- Description text for plugin.json (should be concise, derived from SKILL.md)
- Order of operations for the git mv commands
- Whether to update the skill release tag convention after rename

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| PLUG-01 | The-typescript-narrows skill repo has `.claude-plugin/plugin.json` manifest | Plugin manifest schema documented below; only `name` is required; user locked name/description/version fields |
| PLUG-02 | The-typescript-narrows skill files are in `skills/the-typescript-narrows/` subdirectory | Plugin directory structure documented below; skills must be at plugin root in `skills/<name>/SKILL.md` format |
</phase_requirements>

## Standard Stack

This phase involves no libraries or dependencies. It is a file restructuring and manifest creation task using:

| Tool | Purpose | Why Standard |
|------|---------|--------------|
| `git mv` | Rename/move files preserving history | Standard git operation for tracked file moves |
| `claude plugin validate` | Validate plugin structure | Official Claude Code CLI command, authoritative validator |

No `npm install` needed. The plugin is pure markdown with a JSON manifest.

## Architecture Patterns

### Target Plugin Directory Structure

After restructuring, the plugin root at `plugin/the-typescript-narrows/` must look like:

```
plugin/the-typescript-narrows/          # Plugin root
├── .claude-plugin/                      # Manifest directory
│   └── plugin.json                      # Plugin manifest (name, description, version)
├── skills/                              # Skills directory (at plugin root, NOT in .claude-plugin/)
│   └── the-typescript-narrows/          # Skill directory (name matches plugin name)
│       ├── SKILL.md                     # Skill definition with frontmatter
│       └── references/                  # Supporting reference files
│           ├── async-promises.md
│           ├── control-flow.md
│           ├── discriminated-unions.md
│           ├── error-handling.md
│           ├── functions.md
│           ├── generics.md
│           ├── immutability.md
│           ├── iteration.md
│           ├── modules.md
│           ├── naming.md
│           ├── null-handling.md
│           ├── resource-management.md
│           ├── tsconfig-advanced.md
│           ├── type-declarations.md
│           └── type-safety.md
├── LICENSE                              # Moves from skill dir to plugin root
└── README.md                            # Moves from skill dir to plugin root
```

### Current Structure (Before)

```
skill/the-typescript-narrows/
├── LICENSE
├── README.md
├── SKILL.md
└── references/
    └── (15 .md files)
```

### Pattern: Minimal plugin.json Manifest

**What:** JSON manifest with only required + locked fields
**When to use:** MVP plugin with metadata deferred to later phase

```json
{
  "name": "the-typescript-narrows",
  "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
  "version": "1.0.0"
}
```

Source: Official docs at https://code.claude.com/docs/en/plugins-reference confirm `name` is the only required field. `description` and `version` are optional metadata. The description above is derived from the SKILL.md frontmatter (concise version).

### Anti-Patterns to Avoid

- **Putting components inside .claude-plugin/:** Only `plugin.json` goes in `.claude-plugin/`. The `skills/` directory MUST be at the plugin root. This is explicitly called out in official docs as a common mistake.
- **Using absolute paths in manifest:** All paths in `plugin.json` must be relative and start with `./`.
- **Forgetting to update path references:** The repo has many files referencing the old `skill/` path. Missing even one creates inconsistency.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Plugin validation | Manual directory checks | `claude plugin validate .` | Authoritative validator catches manifest schema errors, missing directories, structural issues |
| File moves with history | `cp` + `rm` | `git mv` | Preserves git history for blame and log |

**Key insight:** The `claude plugin validate` command is the single source of truth for plugin correctness. Do not assume the structure is correct based on documentation alone -- always validate.

## Common Pitfalls

### Pitfall 1: Components Inside .claude-plugin/
**What goes wrong:** Skills, commands, agents placed inside `.claude-plugin/` directory instead of at plugin root
**Why it happens:** Intuition says "plugin stuff goes in the plugin directory"
**How to avoid:** Only `plugin.json` goes in `.claude-plugin/`. Everything else (skills/, commands/, etc.) goes at the plugin root.
**Warning signs:** `claude plugin validate` reports "No commands found" or skills not discovered

### Pitfall 2: Incomplete Path Reference Updates
**What goes wrong:** Old `skill/` paths remain in some files, causing broken references or confusion
**Why it happens:** The repo has many files referencing `skill/` -- easy to miss some
**How to avoid:** Comprehensive grep for `skill/` across the entire repo after restructuring
**Warning signs:** Any remaining `skill/` references that should be `plugin/`

### Pitfall 3: git mv Ordering Issues
**What goes wrong:** Trying to `git mv` into a directory that doesn't exist yet, or moving a parent directory before its contents are reorganized
**Why it happens:** The restructuring requires both renaming the parent AND reorganizing the internal structure
**How to avoid:** Plan the operation order carefully. Rename parent first (`skill/` to `plugin/`), then reorganize internals.
**Warning signs:** git errors about destination not existing

### Pitfall 4: Plugin Name Mismatch
**What goes wrong:** Plugin name in `plugin.json` doesn't match skill directory name or SKILL.md frontmatter name
**Why it happens:** Multiple sources of the name that can drift
**How to avoid:** Use `the-typescript-narrows` everywhere -- it's already consistent in the SKILL.md frontmatter
**Warning signs:** Skill not discoverable after installation, wrong namespacing in UI

### Pitfall 5: Running Validation from Wrong Directory
**What goes wrong:** `claude plugin validate .` run from repo root instead of plugin root
**Why it happens:** Natural to validate from repo root
**How to avoid:** Must `cd` into `plugin/the-typescript-narrows/` (the actual plugin root) before running validate
**Warning signs:** "No manifest found" error when manifest actually exists

## Code Examples

### plugin.json Manifest
```json
{
  "name": "the-typescript-narrows",
  "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
  "version": "1.0.0"
}
```
Source: Schema from https://code.claude.com/docs/en/plugins-reference

### git mv Operations (Recommended Order)

```bash
# Step 1: Rename parent directory skill/ -> plugin/
git mv skill plugin

# Step 2: Create the skills subdirectory structure inside plugin root
mkdir -p plugin/the-typescript-narrows/skills/the-typescript-narrows

# Step 3: Move SKILL.md and references/ into the skills subdirectory
git mv plugin/the-typescript-narrows/SKILL.md plugin/the-typescript-narrows/skills/the-typescript-narrows/SKILL.md
git mv plugin/the-typescript-narrows/references plugin/the-typescript-narrows/skills/the-typescript-narrows/references

# Step 4: LICENSE and README.md stay at plugin root (already there after step 1)
# plugin/the-typescript-narrows/LICENSE -- already in correct location
# plugin/the-typescript-narrows/README.md -- already in correct location

# Step 5: Create .claude-plugin directory and plugin.json
mkdir -p plugin/the-typescript-narrows/.claude-plugin
# Then write plugin.json to plugin/the-typescript-narrows/.claude-plugin/plugin.json
```

### Validation Command
```bash
cd plugin/the-typescript-narrows && claude plugin validate .
```

## Files Requiring Path Updates

Comprehensive inventory of files in the skill repo that reference `skill/` and need updating to `plugin/`:

### Source code and scripts
| File | References | Notes |
|------|------------|-------|
| `scripts/generate-traceability.mjs` (line 113) | `join(ROOT, 'skill', 'the-typescript-narrows', 'references')` | Change `'skill'` to `'plugin'` and update path to include `skills/` subdirectory |

### Documentation and configuration
| File | References | Notes |
|------|------------|-------|
| `CLAUDE.md` (line 10) | `skill/` in repo layout | Update to `plugin/` |
| `CLAUDE.md` (line 47) | `skill/v{semver}` tag convention | Decision: keep or update tag convention (Claude's discretion) |
| `README.md` | References to `skill/` directory | Update to `plugin/` |
| `plugin/the-typescript-narrows/README.md` | Install instructions reference `skill/the-typescript-narrows/` | Update to plugin install instructions |

### Claude commands
| File | References | Notes |
|------|------------|-------|
| `.claude/commands/add-opinion.md` (lines 155, 185) | `skill/the-typescript-narrows/references/` and `skill/the-typescript-narrows/SKILL.md` | Update paths to `plugin/the-typescript-narrows/skills/the-typescript-narrows/` |
| `.claude/commands/release.md` (lines 20, 22, 37) | `skill/v*` tag references | Decision: keep tag convention or update (Claude's discretion) |

### GitHub workflows
| File | References | Notes |
|------|------------|-------|
| `.github/workflows/release-skill.yml` (lines 5, 18, 23, 29) | `skill/v*` tags and `skill/` in git log path | Tags are convention (may keep); git log path filter needs update |

### Files to skip (planning docs, historical)
The `.planning/` directory contains many historical references to `skill/` but these are historical records and should NOT be updated.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `commands/` directory for skills | `skills/<name>/SKILL.md` structure | Claude Code plugin system launch | Skills use dedicated directory with SKILL.md, commands/ is legacy |
| No plugin system | `.claude-plugin/plugin.json` manifest | 2025 Claude Code plugins | Structured plugin discovery and marketplace distribution |

## Open Questions

1. **Tag convention after rename**
   - What we know: Currently uses `skill/v{semver}` tags. The directory is renaming from `skill/` to `plugin/`.
   - What's unclear: Should tags change to `plugin/v{semver}`? This would break existing tag references.
   - Recommendation: This is marked as Claude's discretion. Recommend keeping `skill/v*` tags for backward compatibility in this phase -- changing tags is a separate concern that could be addressed in Phase 3 (POLSH). The workflow file references to `skill/v*` tags are tag-pattern-based, not directory-path-based.

2. **Description text for plugin.json**
   - What we know: SKILL.md frontmatter has a long description. Plugin descriptions should be concise.
   - Recommendation: Use a shortened version: "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point" (derived from the SKILL.md subtitle, not the verbose frontmatter description).

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Claude Code CLI (plugin validate) |
| Config file | none -- built into `claude` CLI |
| Quick run command | `cd plugin/the-typescript-narrows && claude plugin validate .` |
| Full suite command | Same as quick run (single validation command) |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| PLUG-01 | Plugin has valid `.claude-plugin/plugin.json` manifest | smoke | `cd plugin/the-typescript-narrows && claude plugin validate .` | N/A -- uses CLI |
| PLUG-02 | Skill files in `skills/the-typescript-narrows/` subdirectory | smoke | `cd plugin/the-typescript-narrows && claude plugin validate .` | N/A -- uses CLI |

### Sampling Rate
- **Per task commit:** `cd plugin/the-typescript-narrows && claude plugin validate .`
- **Per wave merge:** Same command
- **Phase gate:** `claude plugin validate .` passes with no errors

### Wave 0 Gaps
None -- validation uses the built-in `claude plugin validate` CLI command. No test framework or fixtures needed.

## Sources

### Primary (HIGH confidence)
- Official Claude Code plugins reference: https://code.claude.com/docs/en/plugins-reference -- manifest schema, directory structure, validation, all component specs
- `claude plugin validate --help` -- CLI confirmation of validate command
- Live `claude plugin validate .` output -- confirmed error format and expected manifest locations

### Secondary (MEDIUM confidence)
- Skill repo files read directly: CLAUDE.md, SKILL.md, README.md, workflows, scripts -- current state of the repo confirmed

### Tertiary (LOW confidence)
- None -- all findings verified against primary sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no libraries, just git and CLI commands
- Architecture: HIGH -- official docs clearly specify plugin directory structure
- Pitfalls: HIGH -- official docs explicitly call out the .claude-plugin/ anti-pattern; path inventory compiled from actual grep of repo

**Research date:** 2026-03-18
**Valid until:** 2026-04-18 (plugin system is stable, 30-day validity)
