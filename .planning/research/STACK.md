# Stack Research

**Domain:** Claude Code plugin marketplace (thin JSON catalog)
**Researched:** 2026-03-18
**Confidence:** HIGH

## Recommended Stack

This is not a traditional application -- it is a Git-hosted JSON catalog with no runtime, no build step, and no dependencies. The "stack" is the file structure conventions defined by Claude Code's plugin system, plus validation tooling.

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| JSON | N/A | Marketplace and plugin manifest format | Required by Claude Code plugin system. `marketplace.json` and `plugin.json` are the only artifacts that matter. |
| Git / GitHub | N/A | Hosting and distribution | Claude Code fetches marketplaces via `git clone`. GitHub is the recommended host per official docs. Users add with `/plugin marketplace add owner/repo`. |
| Claude Code CLI | >= 1.0.33 | Validation and testing | `claude plugin validate .` validates marketplace structure, JSON syntax, and schema compliance. This is the only build/test tool needed. |
| Markdown (SKILL.md) | N/A | Skill definitions with YAML frontmatter | Each skill is a `SKILL.md` file with `---` frontmatter (name, description) followed by instructions. Required format for skills. |

### Marketplace File Structure

This is the actual "stack" -- the directory layout that Claude Code expects:

```
claude-marketplace/
  .claude-plugin/
    marketplace.json       # The catalog (required)
  README.md                # Documentation for users
```

The marketplace repo itself is intentionally thin. Plugins live in their own repos and are referenced via `source` entries in `marketplace.json`.

### Plugin File Structure (in separate repos)

Each plugin referenced by the marketplace must follow this layout:

```
plugin-repo/
  .claude-plugin/
    plugin.json            # Plugin manifest (required)
  skills/
    skill-name/
      SKILL.md             # Skill definition
      references/          # Supporting files (optional)
  commands/                # Slash commands (optional)
  agents/                  # Agent definitions (optional)
  hooks/                   # Event handlers (optional)
  .mcp.json                # MCP server config (optional)
```

Critical rule: `commands/`, `agents/`, `skills/`, and `hooks/` go at the plugin root, NOT inside `.claude-plugin/`. Only `plugin.json` goes inside `.claude-plugin/`.

### marketplace.json Schema

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `$schema` | string | No | Schema URL for validation: `https://anthropic.com/claude-code/marketplace.schema.json` |
| `name` | string | Yes | Marketplace identifier, kebab-case. Users see this in install commands. |
| `owner.name` | string | Yes | Maintainer name |
| `owner.email` | string | No | Contact email |
| `metadata.description` | string | No | Brief marketplace description |
| `metadata.version` | string | No | Marketplace version |
| `metadata.pluginRoot` | string | No | Base directory for relative source paths |
| `plugins` | array | Yes | List of plugin entries |

### Plugin Entry Schema (inside marketplace.json)

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `name` | string | Yes | Plugin identifier, kebab-case |
| `source` | string or object | Yes | Where to fetch the plugin |
| `description` | string | No | Brief plugin description |
| `version` | string | No | Plugin version (prefer setting in plugin.json for external sources) |
| `strict` | boolean | No | Default `true`. Set `false` to define plugin entirely from marketplace side. |
| `category` | string | No | For organization |
| `tags` | array | No | For searchability |
| `keywords` | array | No | For discovery |

### Source Types for This Project

For the `the-typescript-narrows` plugin living in a subdirectory of another repo, use `git-subdir`:

```json
{
  "name": "the-typescript-narrows",
  "source": {
    "source": "git-subdir",
    "url": "sethlivingston/the-typescript-narrows",
    "path": "skill/the-typescript-narrows"
  },
  "description": "Opinionated TypeScript narrowing patterns",
  "strict": false
}
```

Use `strict: false` because the plugin structure is defined from the marketplace side, allowing the marketplace entry to control what gets exposed without requiring a `plugin.json` in the source repo (though ideally the source repo should be restructured to include one).

### Supporting Libraries

None. This project has zero runtime dependencies. No `package.json`, no `node_modules`, no build tools.

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `claude plugin validate .` | Validate marketplace structure | Run from marketplace root. Checks JSON syntax, schema, and structure. |
| `claude --plugin-dir ./path` | Test plugins locally | Load plugins for testing without installing. |
| `/plugin marketplace add ./path` | Test marketplace locally | Add local marketplace for end-to-end testing. |
| `/plugin install name@marketplace` | Test plugin installation | Verify plugins install and work correctly. |
| `/reload-plugins` | Reload after changes | Pick up changes without restarting Claude Code. |

## Installation

```bash
# No installation needed. This is a JSON catalog repo.
# To validate:
claude plugin validate .

# To test locally:
# (from within a Claude Code session)
# /plugin marketplace add ./path/to/claude-marketplace
# /plugin install the-typescript-narrows@seth-plugins
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| `git-subdir` source | `github` source | When the plugin is the entire repo (not a subdirectory). Simpler config, no `path` field needed. |
| `strict: false` | `strict: true` (default) | When the plugin repo already has its own `.claude-plugin/plugin.json` with correct component definitions. Preferred long-term. |
| Thin catalog (plugins in separate repos) | Fat catalog (plugins bundled in marketplace repo) | When you want self-contained distribution. Simpler for small plugin counts but harder to version independently. |
| GitHub hosting | GitLab, Bitbucket, self-hosted | When your team uses a different git provider. Claude Code supports any git host. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| `package.json` / npm | This is not a Node.js project. Adding npm adds unnecessary complexity for a JSON catalog. | Plain JSON files validated by `claude plugin validate`. |
| JSON Schema validators (ajv, etc.) | Over-engineering. Claude Code's built-in validator already checks the schema. | `claude plugin validate .` |
| CI/CD pipelines | Out of scope per PROJECT.md. Manual updates are fine at this scale. | Manual validation and git push. |
| Reserved marketplace names | Names like `claude-code-marketplace`, `anthropic-plugins`, `agent-skills` are blocked by Anthropic. | Use a personal name like `seth-plugins` or `seth-marketplace`. |
| `../` in source paths | Claude Code explicitly rejects paths containing `..` in relative sources. | Paths relative to marketplace root starting with `./`. |
| Putting skills inside `.claude-plugin/` | Common mistake. Claude Code expects skills at the plugin root level, not inside `.claude-plugin/`. | `plugin-root/skills/skill-name/SKILL.md` |

## Stack Patterns by Variant

**If adding more skill-only plugins:**
- Use `git-subdir` source if the skill lives in a subdirectory of another repo
- Use `github` source if the skill has its own dedicated repo
- Set `strict: false` if the source repo lacks `.claude-plugin/plugin.json`

**If restructuring the-typescript-narrows repo to be a proper plugin:**
- Add `.claude-plugin/plugin.json` to the repo
- Move skills into `skills/` directory at repo root
- Switch marketplace entry to `strict: true` (or omit, since it defaults to true)
- Switch source type from `git-subdir` to `github` if the entire repo becomes the plugin

**If adding plugins with hooks or MCP servers:**
- Use `github` source type (full repo clone needed for complex plugins)
- Ensure hooks go in `hooks/hooks.json` at plugin root
- MCP config goes in `.mcp.json` at plugin root

## Version Compatibility

| Component | Minimum Version | Notes |
|-----------|-----------------|-------|
| Claude Code CLI | 1.0.33 | Required for `/plugin` commands and validation |
| Git | 2.25+ | Sparse checkout (used by `git-subdir`) requires Git 2.25+ |

## Sources

- [Create and distribute a plugin marketplace - Claude Code Docs](https://code.claude.com/docs/en/plugin-marketplaces) -- Complete marketplace schema, source types, validation, hosting (HIGH confidence)
- [Create plugins - Claude Code Docs](https://code.claude.com/docs/en/plugins) -- Plugin structure, manifest schema, skills format, testing workflow (HIGH confidence)
- [anthropics/claude-plugins-official marketplace.json](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) -- Real-world example of marketplace structure with 100+ plugins (HIGH confidence)

---
*Stack research for: Claude Code plugin marketplace (thin JSON catalog)*
*Researched: 2026-03-18*
