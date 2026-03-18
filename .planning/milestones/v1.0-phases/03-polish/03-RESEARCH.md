# Phase 3: Polish - Research

**Researched:** 2026-03-18
**Domain:** Claude Code plugin metadata, version pinning, CI validation
**Confidence:** HIGH

## Summary

Phase 3 adds professional metadata to both plugin.json and marketplace.json, pins the marketplace plugin reference to a specific git tag, and creates a GitHub Actions CI workflow for validation. All three requirements are well-supported by the Claude Code plugin system -- the official docs explicitly define all needed manifest fields, the `git-subdir` source type supports `ref` pinning, and `claude plugin validate .` is a documented CLI command.

The scope spans two repos: this marketplace catalog repo (marketplace.json + CI workflow) and the-typescript-narrows skill repo (plugin.json metadata + tag convention rename from `skill/` to `plugin/`). The skill repo changes include updating CLAUDE.md, the release command, and the release workflow to use the new `plugin/v*` tag convention.

**Primary recommendation:** Execute in two logical waves: (1) metadata enrichment + tag convention rename + version pinning, (2) CI workflow creation. Both are low-risk mechanical changes.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Add metadata to BOTH plugin.json (in the-typescript-narrows repo) AND marketplace.json (in this repo)
- Plugin is self-describing; marketplace enriches for discovery
- Author: object with name ("sethlivingston") and url (GitHub profile URL)
- Homepage: https://github.com/sethlivingston/the-typescript-narrows (skill repo)
- License: MIT (matches marketplace repo)
- Keywords: minimal set of 3-5 -- typescript, style-guide, code-quality, ai-coding, claude-code
- Keep existing description in marketplace.json for standalone catalog readability
- Repository field in plugin.json points to the skill repo
- Pin marketplace.json plugin entry to a git tag via the `ref` field in the source
- Use NEW tag convention: `plugin/v{semver}` (replacing old `skill/v{semver}` convention)
- Fully qualified ref in marketplace.json: `refs/tags/plugin/v1.0.0`
- Create `plugin/v1.0.0` tag on current HEAD of the-typescript-narrows repo as part of this phase
- Update all docs, scripts, and CLAUDE.md in the skill repo to reference the `plugin/` tag convention instead of `skill/`
- GitHub Actions workflow in this marketplace repo
- Scope: `claude plugin validate .` only -- no lint, no install test
- Triggers: push to main + PRs targeting main
- Install Claude CLI via `npm install -g @anthropic-ai/claude-code` (no version pin -- always latest)
- Standard Node.js setup action for npm availability

### Claude's Discretion
- Exact workflow YAML structure and job naming
- Node.js version for CI runner
- Whether to add a status badge to README
- Exact wording of any metadata descriptions
- Order of fields in plugin.json and marketplace.json

### Deferred Ideas (OUT OF SCOPE)
- Scoped marketplace naming (`@sethlivingston/...`) -- user chose not to discuss, dropped from Phase 3 scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| POLSH-01 | Plugin entries include metadata (author, homepage, repository, license, keywords) | Official plugin manifest schema confirms all fields are supported; official marketplace docs confirm plugin entries can include any plugin.json field |
| POLSH-02 | Plugin entries pin to a specific ref or sha for version stability | `git-subdir` source type supports `ref` field for branch/tag pinning; official docs confirm format |
| POLSH-03 | GitHub Actions workflow validates marketplace on push/PR | `claude plugin validate .` is a documented CLI command; `@anthropic-ai/claude-code` is the npm package |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `@anthropic-ai/claude-code` | latest | CLI for `claude plugin validate .` in CI | Official Anthropic package; user decided no version pin |
| `actions/checkout@v4` | v4 | Git checkout in GitHub Actions | Standard GitHub-provided action |
| `actions/setup-node@v4` | v4 | Node.js setup for npm availability | Standard GitHub-provided action |

### Supporting
No additional libraries needed. This phase is metadata + configuration only.

## Architecture Patterns

### Files to Modify (Marketplace Repo)

```
claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json        # Add metadata fields + ref pinning to plugin entry
└── .github/
    └── workflows/
        └── validate.yml        # NEW: CI workflow
```

### Files to Modify (Skill Repo: the-typescript-narrows)

```
the-typescript-narrows/
├── plugin/the-typescript-narrows/
│   └── .claude-plugin/
│       └── plugin.json         # Add author, homepage, repository, license, keywords
├── CLAUDE.md                   # Update skill/v* references to plugin/v*
├── .claude/commands/
│   └── release.md              # Update skill/v* references to plugin/v*
└── .github/workflows/
    └── release-skill.yml       # Rename file + update skill/v* to plugin/v*
```

### Pattern 1: Plugin Manifest Metadata
**What:** Enrich plugin.json with all standard metadata fields
**When to use:** Any plugin that will be distributed via marketplace

The official plugin manifest schema (from docs) supports these metadata fields:

```json
{
  "name": "the-typescript-narrows",
  "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
  "version": "1.0.0",
  "author": {
    "name": "sethlivingston",
    "url": "https://github.com/sethlivingston"
  },
  "homepage": "https://github.com/sethlivingston/the-typescript-narrows",
  "repository": "https://github.com/sethlivingston/the-typescript-narrows",
  "license": "MIT",
  "keywords": ["typescript", "style-guide", "code-quality", "ai-coding", "claude-code"]
}
```

Source: [Plugins reference - Plugin manifest schema](https://code.claude.com/docs/en/plugins-reference)

### Pattern 2: Marketplace Entry with Metadata + Ref Pinning
**What:** Enrich marketplace.json plugin entry with metadata and pin to tag
**When to use:** Any marketplace plugin entry for production stability

```json
{
  "name": "the-typescript-narrows",
  "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
  "author": {
    "name": "sethlivingston",
    "url": "https://github.com/sethlivingston"
  },
  "homepage": "https://github.com/sethlivingston/the-typescript-narrows",
  "license": "MIT",
  "keywords": ["typescript", "style-guide", "code-quality", "ai-coding", "claude-code"],
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/sethlivingston/the-typescript-narrows.git",
    "path": "plugin/the-typescript-narrows",
    "ref": "refs/tags/plugin/v1.0.0"
  }
}
```

Source: [Plugin marketplaces - Plugin sources](https://code.claude.com/docs/en/plugin-marketplaces)

Note: The `repository` field is omitted from the marketplace entry because it describes the plugin source repo, which is already captured by the `source.url` field. The `repository` field belongs in plugin.json where it's self-describing.

### Pattern 3: GitHub Actions Validation Workflow
**What:** CI workflow that validates marketplace manifest on every push/PR
**When to use:** Any marketplace repo that needs to resist accidental breakage

```yaml
name: Validate Marketplace

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install Claude Code CLI
        run: npm install -g @anthropic-ai/claude-code

      - name: Validate marketplace
        run: claude plugin validate .
```

Source: [Plugins reference - CLI commands](https://code.claude.com/docs/en/plugins-reference)

### Pattern 4: Tag Convention Rename in Skill Repo
**What:** Replace `skill/v*` tag convention with `plugin/v*` across all files
**When to use:** One-time migration for the-typescript-narrows repo

Files affected and the changes needed:
1. **CLAUDE.md** line 47: `skill/v{semver}` -> `plugin/v{semver}`
2. **`.claude/commands/release.md`**: All `skill/v*` references -> `plugin/v*` (lines 20, 22, 37)
3. **`.github/workflows/release-skill.yml`**: Tag trigger `skill/v*` -> `plugin/v*`, version extraction `skill/v` -> `plugin/v`, tag listing `skill/v*` -> `plugin/v*`; consider renaming file to `release-plugin.yml`

### Anti-Patterns to Avoid
- **Duplicating version in both plugin.json and marketplace.json:** Official docs warn that plugin.json always wins silently, so set version in ONE place only. Since plugin.json already has `"version": "1.0.0"`, don't add version to the marketplace entry.
- **Using short ref format:** Use `refs/tags/plugin/v1.0.0` (fully qualified) not `plugin/v1.0.0` to avoid ambiguity with branch names.
- **Adding `repository` to marketplace entry:** The marketplace entry already has `source.url` pointing to the repo. The `repository` field is for plugin.json where the plugin describes itself.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Manifest validation | Custom JSON schema checks | `claude plugin validate .` | Official validator catches schema errors, required fields, naming conventions |
| CI workflow | Complex multi-step pipeline | Single validate step | User explicitly scoped CI to validation only |

## Common Pitfalls

### Pitfall 1: Version Conflicts Between plugin.json and marketplace.json
**What goes wrong:** Setting version in both places causes confusion -- plugin.json silently wins
**Why it happens:** Marketplace entry schema allows version field
**How to avoid:** Set version only in plugin.json (already has `"version": "1.0.0"`)
**Warning signs:** `claude plugin validate .` may not catch this; users see unexpected version

### Pitfall 2: Forgetting to Create the Tag Before Pinning
**What goes wrong:** marketplace.json references `refs/tags/plugin/v1.0.0` but the tag doesn't exist yet
**Why it happens:** Tag creation and manifest update are done in different repos
**How to avoid:** Create and push tag on the-typescript-narrows BEFORE updating marketplace.json ref field
**Warning signs:** `claude plugin install` fails with "ref not found"

### Pitfall 3: Workflow File Rename Without Updating References
**What goes wrong:** Renaming `release-skill.yml` to `release-plugin.yml` without checking if anything references the old filename
**Why it happens:** GitHub Actions workflows are triggered by tag patterns, not filename references, but there may be documentation links
**How to avoid:** Search for `release-skill` references before renaming
**Warning signs:** Broken documentation links

### Pitfall 4: Node.js Version Too Old for Claude Code CLI
**What goes wrong:** Claude CLI may require a recent Node.js version
**Why it happens:** `@anthropic-ai/claude-code` is actively developed and may use modern Node.js features
**How to avoid:** Use Node.js 22 (current LTS) in the CI workflow
**Warning signs:** `npm install -g @anthropic-ai/claude-code` fails or `claude plugin validate` crashes

### Pitfall 5: Author Field Format
**What goes wrong:** Using `"author": "sethlivingston"` (string) instead of object format
**Why it happens:** npm package.json uses string format, but Claude plugin schema uses object
**How to avoid:** Always use object format `{"name": "...", "url": "..."}`; official docs show object format only
**Warning signs:** Validation may reject string format

## Code Examples

### Current marketplace.json (to be enriched)
```json
{
  "name": "sethlivingston-plugins",
  "owner": {
    "name": "sethlivingston"
  },
  "plugins": [
    {
      "name": "the-typescript-narrows",
      "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
      "source": {
        "source": "git-subdir",
        "url": "https://github.com/sethlivingston/the-typescript-narrows.git",
        "path": "plugin/the-typescript-narrows"
      }
    }
  ]
}
```

### Current plugin.json (to be enriched)
```json
{
  "name": "the-typescript-narrows",
  "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
  "version": "1.0.0"
}
```

### Tag Creation Command
```bash
cd ../the-typescript-narrows
git tag plugin/v1.0.0
git push origin plugin/v1.0.0
```

Current HEAD of the-typescript-narrows: `b3e105597e32c13304e9dce335565af0900b5b71`

Existing tags: `eslint-plugin/v1.0.0`, `skill/v1.0.0`

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `skill/v*` tag convention | `plugin/v*` tag convention | Phase 3 (this phase) | Aligns tag naming with Claude Code plugin terminology |
| Unpinned plugin sources | Ref-pinned sources | Phase 3 (this phase) | Version stability for marketplace consumers |
| No CI validation | `claude plugin validate .` in CI | Phase 3 (this phase) | Prevents accidental manifest breakage |

## Open Questions

1. **Does `claude plugin validate .` require an API key or auth?**
   - What we know: It's a local validation command that checks JSON structure and schema
   - What's unclear: Whether it phones home or requires ANTHROPIC_API_KEY in CI
   - Recommendation: Test locally first; if it needs auth, the CI workflow may need a secret. Most likely it does NOT need auth since it's schema validation only.

2. **Should the old `skill/v1.0.0` tag be deleted from the skill repo?**
   - What we know: User wants all docs updated to `plugin/` convention; old tag exists
   - What's unclear: Whether to delete old tag or leave it for backward compatibility
   - Recommendation: Leave old tag in place (deleting published tags is bad practice); create new `plugin/v1.0.0` tag on same commit

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Claude Code CLI (`claude plugin validate .`) |
| Config file | `.claude-plugin/marketplace.json` (the manifest IS the testable artifact) |
| Quick run command | `claude plugin validate .` |
| Full suite command | `claude plugin validate .` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| POLSH-01 | Plugin entries include metadata | smoke | `claude plugin validate .` | N/A (validates manifest) |
| POLSH-02 | Plugin entry pins to specific ref | manual-only | Verify `ref` field present in marketplace.json; install test would require network | N/A |
| POLSH-03 | CI workflow validates on push/PR | manual-only | Push to branch and verify workflow runs; inspect `.github/workflows/validate.yml` exists | Wave 0 |

### Sampling Rate
- **Per task commit:** `claude plugin validate .`
- **Per wave merge:** `claude plugin validate .` + manual review of marketplace.json ref field
- **Phase gate:** Full validation green + CI workflow file reviewed

### Wave 0 Gaps
- [ ] `.github/workflows/validate.yml` -- the CI workflow itself (created as part of POLSH-03)

## Sources

### Primary (HIGH confidence)
- [Plugins reference](https://code.claude.com/docs/en/plugins-reference) - Complete plugin manifest schema with all metadata fields
- [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - Marketplace schema, plugin sources, git-subdir with ref pinning
- [Official marketplace](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) - Real-world examples of plugin entries with metadata

### Secondary (MEDIUM confidence)
- Current state of marketplace.json and plugin.json in both repos (direct file reads)
- Current state of skill repo files referencing `skill/v*` convention (grep results)

### Tertiary (LOW confidence)
- Whether `claude plugin validate .` requires auth in CI (needs testing)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - official docs clearly define all fields and supported values
- Architecture: HIGH - file changes are mechanical and well-scoped across two repos
- Pitfalls: HIGH - common issues are well-documented in official troubleshooting guides

**Research date:** 2026-03-18
**Valid until:** 2026-04-18 (stable domain; Claude Code plugin schema is unlikely to change breaking)
