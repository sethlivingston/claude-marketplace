# Phase 2: Marketplace Catalog - Research

**Researched:** 2026-03-18
**Domain:** Claude Code plugin marketplace manifest (`.claude-plugin/marketplace.json`)
**Confidence:** HIGH

## Summary

This phase creates the marketplace manifest (`.claude-plugin/marketplace.json`) in the claude-marketplace repo and validates end-to-end plugin installation. The Claude Code plugin system uses a well-defined JSON manifest format for marketplaces, with the official Anthropic marketplace (`anthropics/claude-plugins-official`) serving as the canonical reference.

The marketplace.json format supports multiple source types including `git-subdir`, which is exactly what we need to reference the-typescript-narrows plugin living in a subdirectory of its own repo. The official marketplace contains multiple `git-subdir` examples, including some using GitHub shorthand (`owner/repo` instead of full URLs). The validate command (`claude plugin validate .`) provides authoritative structural validation.

**Primary recommendation:** Create a minimal marketplace.json following the official schema pattern, with a single `git-subdir` plugin entry pointing to `sethlivingston/the-typescript-narrows` at path `plugin/the-typescript-narrows`. Validate locally, push, then test the full remote flow.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Marketplace name: `sethlivingston-plugins`
- Owner: `sethlivingston`
- Description: short and functional, e.g. "Seth Livingston's Claude Code plugins"
- Plugin entry: minimum fields only (name, source); no duplicated description from plugin.json
- Source type: `git-subdir` pointing to repo `sethlivingston/the-typescript-narrows`, subdirectory `plugin/the-typescript-narrows`
- No ref pinning (use default branch)
- No keywords or extra metadata
- Two-step validation: `claude plugin validate .` first, then live install test
- Push to GitHub then test with `claude plugin marketplace add sethlivingston/claude-marketplace`
- Success = `/the-typescript-narrows` skill appears after install
- Minimal README.md, .gitignore, MIT License, CLAUDE.md
- User considered npm-style scoped name but deferred to Phase 3

### Claude's Discretion
- Exact README wording
- CLAUDE.md content and conventions
- .gitignore contents
- marketplace.json field ordering
- How to structure the validate-then-test workflow

### Deferred Ideas (OUT OF SCOPE)
- Scoped marketplace naming (`@sethlivingston/...`) -- investigate in Phase 3
- Rich metadata (keywords, author, homepage) -- Phase 3 (POLSH-01)
- Version pinning to specific ref/sha -- Phase 3 (POLSH-02)
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| MKTPL-01 | Marketplace repo has valid `.claude-plugin/marketplace.json` with name, owner, and plugins array | Official marketplace.json structure documented with exact field format; owner is `{name, email}` object |
| MKTPL-02 | Each plugin entry has a concise description explaining what it does | **Note:** User decided minimum fields only, no duplicated description. However, official examples ALL include description per plugin. Validate whether description is required by the validator |
| MKTPL-03 | Marketplace passes `claude plugin validate .` with no errors | Validate command confirmed working; tested on empty repo to see error format |
| MKTPL-04 | User can add marketplace via `/plugin marketplace add sethlivingston/claude-marketplace` | CLI confirmed: `claude plugin marketplace add <source>` accepts GitHub shorthand |
| PLUG-03 | Marketplace references the-typescript-narrows via `git-subdir` source pointing to plugin subdirectory | Multiple `git-subdir` examples found in official marketplace with exact field structure |
| PLUG-04 | User can install the-typescript-narrows and use the `/the-typescript-narrows` skill | `claude plugin install <plugin>` confirmed with `--scope` option |
</phase_requirements>

## Standard Stack

No libraries needed -- this phase creates static JSON and text files only.

### Tools Used
| Tool | Purpose | Why |
|------|---------|-----|
| `claude plugin validate .` | Validate marketplace manifest structure | Authoritative validation -- the only source of truth for correctness |
| `claude plugin marketplace add` | Add marketplace from GitHub | Tests the real user flow |
| `claude plugin install` | Install plugin from marketplace | End-to-end verification |

## Architecture Patterns

### Required File Structure
```
claude-marketplace/
  .claude-plugin/
    marketplace.json        # Main deliverable
  .gitignore
  CLAUDE.md
  LICENSE
  README.md
```

### marketplace.json Schema (from official example)

The official Anthropic marketplace uses this top-level structure:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "marketplace-name",
  "description": "Short description",
  "owner": {
    "name": "Owner Name",
    "email": "email@example.com"
  },
  "plugins": [...]
}
```

**Key finding:** `owner` is an object with `name` and `email` fields, not a plain string. The CONTEXT.md says owner is `sethlivingston` (a string), but the actual format requires an object. The planner needs to handle this -- likely `{"name": "sethlivingston"}` with email optional (needs validation).

### git-subdir Source Format

From the official marketplace, `git-subdir` entries look like this:

```json
{
  "name": "plugin-name",
  "description": "What the plugin does",
  "source": {
    "source": "git-subdir",
    "url": "owner/repo",
    "path": "path/to/plugin/root"
  }
}
```

**Verified examples from official marketplace:**

1. **Railway** (GitHub shorthand + ref + sha):
```json
{
  "source": "git-subdir",
  "url": "railwayapp/railway-skills",
  "path": "plugins/railway",
  "ref": "main",
  "sha": "d52f3741a6a33a3191d6138eb3d6c3355cb970d1"
}
```

2. **Semgrep** (full URL, no ref):
```json
{
  "source": "git-subdir",
  "url": "https://github.com/semgrep/mcp-marketplace.git",
  "path": "plugin"
}
```

3. **AWS plugins** (full URL + ref, no sha):
```json
{
  "source": "git-subdir",
  "url": "https://github.com/awslabs/agent-plugins.git",
  "path": "plugins/aws-serverless",
  "ref": "main"
}
```

**For our case:** Use GitHub shorthand `sethlivingston/the-typescript-narrows` with path `plugin/the-typescript-narrows`. No ref or sha (per user decision).

### MKTPL-02 Tension: Description Field

The user decided "minimum fields only, no duplicated description." However, MKTPL-02 requires "each plugin entry has a concise description." Every plugin in the official marketplace includes a description field.

**Resolution:** The description in marketplace.json is likely required by the validator. Include it. This is not "duplication" -- it's the marketplace-facing description (what users see when browsing), which can differ from the plugin.json description (what users see after install). Keep it concise as the user requested.

### Anti-Patterns to Avoid
- **Putting plugin source files in the marketplace repo:** This is a catalog-only repo. Plugins live in their own repos.
- **Using `url` source type instead of `git-subdir`:** The plugin is in a subdirectory, so `git-subdir` is required to point to the right root.
- **Omitting the `$schema` field:** The official marketplace includes it. Include for consistency and potential future validation.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| JSON validation | Manual checking | `claude plugin validate .` | Authoritative validator catches structural errors |
| Install testing | Assumptions about correctness | `claude plugin marketplace add` + `claude plugin install` | Only real flow proves it works |

## Common Pitfalls

### Pitfall 1: Owner Format Mismatch
**What goes wrong:** Using `"owner": "sethlivingston"` (string) instead of `"owner": {"name": "sethlivingston"}` (object)
**Why it happens:** CONTEXT.md says owner is `sethlivingston`, but the schema expects an object
**How to avoid:** Use the object format `{"name": "sethlivingston"}`. Run validate to confirm.
**Warning signs:** Validation error on owner field

### Pitfall 2: Wrong git-subdir Path
**What goes wrong:** Pointing to the repo root instead of the plugin subdirectory, or getting the path wrong
**Why it happens:** Confusion between repo path and plugin root
**How to avoid:** Path must be `plugin/the-typescript-narrows` (the directory containing `.claude-plugin/plugin.json`)
**Warning signs:** Install succeeds but plugin has no skills; or install fails with "no manifest found"

### Pitfall 3: Forgetting to Push Before Testing Remote Flow
**What goes wrong:** Testing `claude plugin marketplace add sethlivingston/claude-marketplace` before pushing marketplace.json to GitHub
**Why it happens:** Natural to want to test locally first
**How to avoid:** Validate locally with `claude plugin validate .`, then push, then test remote add
**Warning signs:** "marketplace not found" or "no manifest" errors from the add command

### Pitfall 4: Plugin Name Mismatch
**What goes wrong:** Plugin name in marketplace.json doesn't match plugin name in the plugin's own plugin.json
**Why it happens:** Typo or inconsistency
**How to avoid:** Plugin name in marketplace entry must be `the-typescript-narrows` (matching plugin.json)
**Warning signs:** Plugin installs but can't be found by name, or validation warnings

### Pitfall 5: Missing Description When Required
**What goes wrong:** Omitting the description field per "minimum fields" decision, but validator requires it
**Why it happens:** Tension between user's minimal approach and schema requirements
**How to avoid:** Include a concise description. Test with validate. If truly optional, can remove later
**Warning signs:** Validation error on missing description

## Code Examples

### Target marketplace.json (Confidence: HIGH)

Based on official marketplace patterns:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "sethlivingston-plugins",
  "description": "Seth Livingston's Claude Code plugins",
  "owner": {
    "name": "sethlivingston"
  },
  "plugins": [
    {
      "name": "the-typescript-narrows",
      "description": "Opinionated TypeScript guidelines for AI-assisted development -- one opinion per decision point",
      "source": {
        "source": "git-subdir",
        "url": "sethlivingston/the-typescript-narrows",
        "path": "plugin/the-typescript-narrows"
      }
    }
  ]
}
```

**Notes:**
- `$schema` included for consistency with official marketplace
- `owner.email` omitted (may or may not be required -- validate will tell us)
- Description reuses plugin.json description text (same source of truth, not duplication)
- GitHub shorthand for URL (confirmed working in official examples: `railwayapp/railway-skills`, `zapier/zapier-mcp`, etc.)

### Validation Workflow

```bash
# Step 1: Validate locally
cd /Users/sethlivingston/dev/sethlivingston/claude-marketplace
claude plugin validate .

# Step 2: Push to GitHub
git add .claude-plugin/marketplace.json
git push origin main

# Step 3: Add marketplace
claude plugin marketplace add sethlivingston/claude-marketplace

# Step 4: Install plugin
claude plugin install the-typescript-narrows

# Step 5: Verify skill is available
# Invoke /the-typescript-narrows in Claude Code
```

### Minimal .gitignore

```gitignore
.DS_Store
node_modules/
*.log
```

## State of the Art

| Aspect | Current State | Source |
|--------|--------------|--------|
| Marketplace format | JSON manifest at `.claude-plugin/marketplace.json` | Official marketplace (HIGH) |
| Source types | `url`, `git-subdir`, `github`, local path (string) | Official marketplace examples (HIGH) |
| GitHub shorthand | `owner/repo` works for git-subdir URL field | Official examples: railway, zapier, neon, legalzoom (HIGH) |
| Schema URL | `https://anthropic.com/claude-code/marketplace.schema.json` (returns 404 but used in official manifest) | Official marketplace (HIGH) |
| Owner format | Object with `name` and `email` fields | Official marketplace (HIGH) |
| Validate command | `claude plugin validate <path>` -- works on both plugins and marketplaces | CLI help + tested (HIGH) |

## Open Questions

1. **Is `owner.email` required?**
   - What we know: Official marketplace has it. Our user hasn't specified one.
   - What's unclear: Whether the validator requires it
   - Recommendation: Try without it first; add if validator complains

2. **Is plugin `description` required in marketplace entries?**
   - What we know: Every official marketplace entry has one. User wanted minimum fields.
   - What's unclear: Whether validator enforces it
   - Recommendation: Include it. The description serves a different purpose (marketplace browsing) than plugin.json description (post-install). Use the same text from plugin.json for consistency.

3. **Does marketplace `add` work with the repo still having only .planning/ and .claude-plugin/?**
   - What we know: The add command fetches from GitHub and reads the manifest
   - What's unclear: Whether extra files/directories cause issues
   - Recommendation: Should be fine -- the command looks for `.claude-plugin/marketplace.json` specifically

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Claude Code CLI (no automated test framework -- this is a static manifest project) |
| Config file | none |
| Quick run command | `claude plugin validate .` |
| Full suite command | `claude plugin validate . && claude plugin marketplace add sethlivingston/claude-marketplace --scope project` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| MKTPL-01 | Valid marketplace.json with name, owner, plugins | smoke | `claude plugin validate .` | N/A (CLI) |
| MKTPL-02 | Plugin entry has description | smoke | `claude plugin validate .` | N/A (CLI) |
| MKTPL-03 | Passes validate with no errors | smoke | `claude plugin validate .` | N/A (CLI) |
| MKTPL-04 | User can add marketplace | e2e | `claude plugin marketplace add sethlivingston/claude-marketplace` | N/A (CLI) |
| PLUG-03 | git-subdir source pointing to plugin subdirectory | smoke | `claude plugin validate .` | N/A (CLI) |
| PLUG-04 | User can install and use the-typescript-narrows | manual-only | Install + invoke `/the-typescript-narrows` in Claude Code | N/A |

### Sampling Rate
- **Per task commit:** `claude plugin validate .`
- **Per wave merge:** Full validate + marketplace add test
- **Phase gate:** All 6 requirements verified (4 automated, 1 semi-automated, 1 manual)

### Wave 0 Gaps
None -- no test infrastructure needed. The Claude Code CLI itself is the test tool. The `.claude-plugin/marketplace.json` file IS both the deliverable and the test input.

## Sources

### Primary (HIGH confidence)
- `anthropics/claude-plugins-official` marketplace.json via GitHub API -- authoritative reference for marketplace format, field structure, source types, and `git-subdir` examples (83 plugins analyzed)
- `claude plugin validate .` CLI output -- tested on empty repo, confirms expected manifest location
- `claude plugin marketplace --help` -- confirms add/list/remove/update subcommands
- `claude plugin install --help` -- confirms `--scope` option and `plugin@marketplace` syntax
- Phase 1 plugin.json at `the-typescript-narrows/plugin/the-typescript-narrows/.claude-plugin/plugin.json` -- confirmed structure

### Secondary (MEDIUM confidence)
- Schema URL `https://anthropic.com/claude-code/marketplace.schema.json` -- referenced in official marketplace but returns 404 (schema may be validated internally by Claude Code)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- based on official marketplace example with 83 plugins
- Architecture: HIGH -- marketplace.json format confirmed from authoritative source
- Pitfalls: HIGH -- derived from actual format analysis and identified tensions with user decisions

**Research date:** 2026-03-18
**Valid until:** 2026-04-18 (stable -- marketplace format unlikely to change frequently)
