# Pitfalls Research

**Domain:** Claude Code plugin marketplace (personal, thin-catalog style)
**Researched:** 2026-03-18
**Confidence:** HIGH (sourced primarily from official docs and confirmed bug reports)

## Critical Pitfalls

### Pitfall 1: Plugin name collides with marketplace name

**What goes wrong:**
When the `name` field in a plugin's `.claude-plugin/plugin.json` matches the marketplace name, installation fails with an obscure `EXDEV: cross-device link not permitted` error on Linux. The installer stages files at `cache/<plugin.json name>` then renames to `cache/<marketplace>/<plugin>/<version>`. When names collide, the destination nests inside the staging path, triggering a cross-filesystem rename failure.

**Why it happens:**
It is natural to name both the marketplace and its primary plugin the same thing. For example, naming the marketplace `claude-marketplace` and a plugin `claude-marketplace` seems logical but breaks installation.

**How to avoid:**
- Use a distinct marketplace name (e.g., `seth-plugins` or `sethlivingston-marketplace`) that cannot collide with any plugin name.
- Validate by checking: does any plugin entry's `name` field match the marketplace `name` field? If yes, rename one of them.
- Run `claude plugin validate .` after any name changes.

**Warning signs:**
- Plugin installs fail with `EXDEV` errors.
- Marketplace name and any plugin name are identical strings.

**Phase to address:**
Phase 1 (initial marketplace setup). Choose the marketplace name carefully from the start.

---

### Pitfall 2: strict mode conflict kills plugin loading

**What goes wrong:**
When `strict` is `true` (the default) and the plugin's own `plugin.json` declares components (commands, agents, hooks, skills), the marketplace entry can supplement but not contradict. When `strict` is `false`, the marketplace entry is the entire definition -- but if the plugin also has a `plugin.json` that declares components, the plugin fails to load silently due to the conflict.

**Why it happens:**
Developers set `strict: false` in the marketplace entry to control the plugin definition from the marketplace side, but forget that the source plugin repo also has a `plugin.json` declaring its own components. Or they restructure the source plugin to add components but forget the marketplace is using `strict: false`.

**How to avoid:**
- If the source repo has its own `.claude-plugin/plugin.json` with component declarations (commands, agents, skills, hooks): use `strict: true` (or omit, since true is default).
- If you want full marketplace-side control via `strict: false`: ensure the source repo's `plugin.json` declares NO components. It can have `name`, `description`, `version` only.
- For this project: since the-typescript-narrows skill repo will have its own `plugin.json` defining the skills directory, use `strict: true` and let the marketplace entry supplement with metadata only.

**Warning signs:**
- Plugin installs but skills/commands do not appear when running `/plugin`.
- No error messages (silent failure).
- Check the `/plugin` Errors tab for loading issues.

**Phase to address:**
Phase 1 (marketplace and plugin structure setup). Decide the strict mode strategy before writing either `plugin.json` or `marketplace.json`.

---

### Pitfall 3: Plugin directory structure does not match Claude Code expectations

**What goes wrong:**
Claude Code expects a specific directory layout: `.claude-plugin/plugin.json` at the plugin root, with `skills/` subdirectory containing skill directories each with a `SKILL.md`. If the skill repo has a flat structure (e.g., `SKILL.md` at the repo root, or skills not nested under `skills/<skill-name>/`), the plugin loads but skills are not discovered.

**Why it happens:**
The project context explicitly notes that the-typescript-narrows skill repo "currently lacks plugin structure (no `.claude-plugin/plugin.json`, skills not in `skills/` subdirectory)." The skill was written before the plugin system existed and needs restructuring.

**How to avoid:**
- Restructure the source repo to match the expected layout:
  ```
  skill/the-typescript-narrows/       (git-subdir path target)
    .claude-plugin/
      plugin.json
    skills/
      the-typescript-narrows/
        SKILL.md
        references/
          ...guide files...
  ```
- Run `claude plugin validate .` from the plugin subdirectory after restructuring.
- Test the full flow: add marketplace, install plugin, verify `/the-typescript-narrows` skill appears.

**Warning signs:**
- `claude plugin validate .` reports missing files or invalid structure.
- Plugin installs but skill command does not appear in `/plugin` Installed tab.
- SKILL.md frontmatter parsing warnings in the Errors tab.

**Phase to address:**
Phase 1 (source repo restructuring). This is the first thing to do before the marketplace can work at all.

---

### Pitfall 4: git-subdir path points to wrong directory level

**What goes wrong:**
The `path` field in a `git-subdir` source must point to the directory that contains `.claude-plugin/plugin.json`. If it points one level too high or too low, the sparse clone succeeds but the plugin fails to load because the expected files are not at the root of the cloned content.

**Why it happens:**
The-typescript-narrows plugin lives at `skill/the-typescript-narrows` in the source repo. It is easy to accidentally point to `skill/` (too high) or `skill/the-typescript-narrows/skills/` (too low). The sparse clone does not validate that a valid plugin exists at the target path.

**How to avoid:**
- The `path` in marketplace.json must point to the directory that directly contains `.claude-plugin/`:
  ```json
  {
    "source": {
      "source": "git-subdir",
      "url": "sethlivingston/the-typescript-narrows",
      "path": "skill/the-typescript-narrows"
    }
  }
  ```
- Verify by mentally checking: does `<path>/.claude-plugin/plugin.json` exist in the source repo?
- Test locally by cloning the repo and running `claude plugin validate` from that subdirectory.

**Warning signs:**
- Plugin installation fails with "plugin.json not found" or similar.
- Plugin installs but has zero skills/commands.
- Git clone times out (wrong path causing full clone attempt).

**Phase to address:**
Phase 1 (marketplace.json authoring). Get the path right on the first attempt.

---

### Pitfall 5: Files outside plugin directory boundary are not accessible after installation

**What goes wrong:**
When users install a plugin, Claude Code copies the plugin directory to a cache location at `~/.claude/plugins/cache`. Paths that reference files outside the plugin's directory using `../` will not work because those files are not copied. This means a plugin cannot reference shared utilities, configs, or files from parent directories.

**Why it happens:**
The plugin system is designed for self-contained plugins. The `git-subdir` sparse clone only fetches the specified subdirectory. Any files outside that subdirectory boundary are simply not available at runtime.

**How to avoid:**
- Ensure ALL files the plugin needs (SKILL.md, references, configs) are inside the plugin subdirectory.
- Do not reference files from the parent repo using relative paths like `../../`.
- If files must be shared across plugins, use symlinks (which are followed during copying) -- but for a personal marketplace with one plugin, just keep everything self-contained.

**Warning signs:**
- Skills reference files that exist in the source repo but are missing in the installed plugin.
- "File not found" errors when using installed plugin features.
- Works when testing locally via `./` path but fails when installed via marketplace.

**Phase to address:**
Phase 1 (source repo restructuring). Ensure the plugin subdirectory is fully self-contained before publishing.

---

### Pitfall 6: Marketplace uses a reserved name

**What goes wrong:**
Claude Code blocks marketplace names that are reserved for official Anthropic use. The following names (and variations that impersonate official marketplaces) are blocked: `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `life-sciences`. Names like `official-claude-plugins` or `anthropic-tools-v2` are also blocked.

**Why it happens:**
Developers choose descriptive names that accidentally overlap with reserved names. The name `claude-marketplace` might or might not be blocked (it is not explicitly listed but could be caught by impersonation detection).

**How to avoid:**
- Use a personally-scoped name: `seth-plugins`, `sethlivingston-tools`, or similar.
- Avoid any name containing `claude`, `anthropic`, or `official` as prefixes.
- Run `claude plugin validate .` to catch reserved name violations before publishing.

**Warning signs:**
- `claude plugin validate .` rejects the marketplace name.
- `/plugin marketplace add` fails with a reserved name error.

**Phase to address:**
Phase 1 (marketplace setup). Choose the name once and do not change it, since users reference it in install commands like `/plugin install X@marketplace-name`.

---

### Pitfall 7: Version set in both marketplace entry and plugin.json

**What goes wrong:**
When `version` is specified in both the marketplace entry in `marketplace.json` AND in the plugin's own `plugin.json`, the plugin manifest always wins silently. The marketplace version is ignored. This causes confusion when you update the marketplace version but the installed plugin still reports the old version from its `plugin.json`.

**Why it happens:**
It seems thorough to specify the version in both places. The official docs explicitly warn: "When possible, avoid setting the version in both places. The plugin manifest always wins silently."

**How to avoid:**
- For `git-subdir` sourced plugins (like this project): set the version in the plugin's own `plugin.json` and omit it from the marketplace entry.
- Or, if using `strict: false` with no `plugin.json` components: set version only in the marketplace entry.
- Pick one source of truth and stick with it.

**Warning signs:**
- Auto-updates skip because Claude Code sees the same version from `plugin.json` even though marketplace.json version changed.
- Plugin version in `/plugin` UI does not match what is in `marketplace.json`.

**Phase to address:**
Phase 1 (initial configuration). Establish the version strategy early.

---

### Pitfall 8: SKILL.md frontmatter is malformed

**What goes wrong:**
SKILL.md files require valid YAML frontmatter between `---` delimiters. If the frontmatter has YAML syntax errors (wrong indentation, missing colons, unquoted special characters), the skill loads with no metadata. The `description` field in frontmatter is what makes the skill discoverable and triggers it. Without valid frontmatter, the skill may install but never activate.

**Why it happens:**
YAML is finicky. Common mistakes: using tabs instead of spaces, forgetting to quote strings with colons, or having trailing whitespace. The existing SKILL.md may have frontmatter written for a different system that does not match Claude Code's expected format.

**How to avoid:**
- Use the minimal required frontmatter format:
  ```markdown
  ---
  description: Brief description of what this skill does
  ---
  ```
- Run `claude plugin validate .` which checks frontmatter parsing.
- Test the skill by installing and verifying it appears as a slash command.

**Warning signs:**
- `claude plugin validate .` reports YAML frontmatter parse warnings.
- Skill installs but does not appear as a slash command.
- Skill appears but never triggers contextually.

**Phase to address:**
Phase 1 (source repo restructuring). Fix frontmatter during the restructuring step.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Omitting `ref` or `sha` pin in git-subdir source | Simpler marketplace.json, always gets latest | Users get different plugin versions depending on when they install; breaking changes in source repo immediately affect all users | Acceptable during solo development with one plugin; add pins before sharing widely |
| Using `strict: false` to avoid restructuring source repo | Can define plugin entirely from marketplace side without touching source repo | Two places to maintain component definitions; easy to accidentally add plugin.json components in source repo later and silently break loading | Only acceptable if the source repo will never have its own plugin.json components |
| Hardcoding version "1.0.0" and never updating | One less thing to maintain | Auto-updates stop working (Claude Code skips updates when version does not change) | Never -- always bump version on changes, even if just to "1.0.1" |
| Skipping `claude plugin validate` before publishing | Saves a few seconds | Users hit cryptic errors during install; issues harder to diagnose remotely | Never -- always validate |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| git-subdir with GitHub shorthand | Using `"url": "sethlivingston/the-typescript-narrows"` without specifying it is a GitHub shorthand | Use full URL `"url": "https://github.com/sethlivingston/the-typescript-narrows.git"` or verify that GitHub shorthand (`owner/repo`) is supported for git-subdir `url` field (docs confirm it is) |
| Plugin cache after changes | Updating marketplace.json or plugin source but users still see old version | Users must run `/plugin marketplace update` and then `/reload-plugins`. Bump the version in plugin.json to trigger update detection |
| Private repo authentication | Assuming git credential helpers work for background auto-updates | Background auto-updates require `GITHUB_TOKEN` or `GH_TOKEN` in environment. Manual install/update uses credential helpers. For a public personal marketplace, this is not an issue |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| git-subdir from large repo without ref pin | Slow clone on first install; git timeout after 120s | Pin to a ref or sha; keep the source repo reasonably sized | When source repo exceeds a few hundred MB of git history |
| Many plugins in one marketplace without categories | Users cannot find what they want in `/plugin` Discover tab | Add `category` and `tags` fields to each plugin entry | When marketplace grows beyond 5-10 plugins (not an immediate concern for this project) |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Publishing a marketplace that points to repos you do not control | Third party could change the plugin source to include malicious hooks or MCP servers | Only point to repos you own. For personal marketplace, this is inherently safe since all repos are yours |
| Not pinning plugin sources with `sha` | Source repo could be compromised; HEAD of branch could change to malicious code | Pin with `sha` for production use. For personal marketplace with your own repos, ref pinning is sufficient |
| Including hooks that run shell commands without `${CLAUDE_PLUGIN_ROOT}` | Hooks might execute from wrong directory or reference system commands that vary | Always use `${CLAUDE_PLUGIN_ROOT}` for paths in hook commands |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Choosing a long or hard-to-type marketplace name | Users must type `/plugin marketplace add sethlivingston/claude-marketplace` and `/plugin install X@long-marketplace-name` | Keep the marketplace name short and memorable. The GitHub repo can be `claude-marketplace` but the marketplace `name` in marketplace.json should be concise (e.g., `seth-plugins`) |
| No description on marketplace or plugins | Users see bare names in Discover tab with no context about what plugins do | Always set `metadata.description` on marketplace and `description` on each plugin entry |
| Skill name does not match what users expect to type | User expects `/typescript-narrows` but skill is named `/the-typescript-narrows` or vice versa | Skill command name comes from the skill directory name. Choose directory names that match what users will type |

## "Looks Done But Isn't" Checklist

- [ ] **marketplace.json**: Has `name`, `owner.name`, and at least one plugin entry -- verify with `claude plugin validate .`
- [ ] **Plugin entry**: Has both `name` and `source` fields -- validate JSON syntax (trailing commas break parsing)
- [ ] **Source repo plugin structure**: Has `.claude-plugin/plugin.json` at the plugin root AND `skills/<name>/SKILL.md` -- verify the directory exists at exactly the path specified in git-subdir
- [ ] **SKILL.md frontmatter**: Has valid YAML with at least `description` field -- verify by checking the plugin Errors tab after installation
- [ ] **Skill references**: All files referenced by SKILL.md (like `references/` guides) are inside the plugin subdirectory boundary -- test by installing from marketplace, not just local path
- [ ] **End-to-end test**: Actually run `/plugin marketplace add`, `/plugin install`, and the skill command -- do not assume validation passing means it works
- [ ] **Version field**: Set in exactly one place (plugin.json OR marketplace entry, not both) -- check both files

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Name collision (marketplace vs plugin) | LOW | Rename marketplace `name` in marketplace.json. Users who already added must remove and re-add |
| Wrong strict mode causing silent load failure | LOW | Change `strict` field in marketplace.json, bump version, users run `/plugin marketplace update` |
| Wrong git-subdir path | LOW | Fix `path` in marketplace.json, users run `/plugin marketplace update` and reinstall |
| Source repo not restructured correctly | MEDIUM | Restructure repo, push changes, bump plugin version, users reinstall. May require clearing plugin cache (`rm -rf ~/.claude/plugins/cache/<marketplace>/<plugin>`) |
| Reserved marketplace name | LOW | Rename in marketplace.json. If already shared, users must remove old marketplace and add new one |
| Broken SKILL.md frontmatter | LOW | Fix YAML, push to source repo, bump version |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Name collision | Phase 1: Marketplace setup | `claude plugin validate .` passes; names are distinct |
| strict mode conflict | Phase 1: Marketplace setup | Plugin loads with skills visible in `/plugin` Installed tab |
| Directory structure mismatch | Phase 1: Source repo restructuring | `claude plugin validate .` passes from plugin subdirectory |
| Wrong git-subdir path | Phase 1: Marketplace.json authoring | Plugin installs successfully from marketplace |
| Files outside boundary | Phase 1: Source repo restructuring | All referenced files exist after cache-based installation |
| Reserved name | Phase 1: Marketplace setup | `claude plugin validate .` passes |
| Version dual-source | Phase 1: Initial configuration | Version appears correctly in `/plugin` UI |
| Malformed frontmatter | Phase 1: Source repo restructuring | Skill appears as slash command after installation |

## Sources

- [Create and distribute a plugin marketplace - Claude Code Docs](https://code.claude.com/docs/en/plugin-marketplaces) -- official docs (HIGH confidence)
- [Discover and install prebuilt plugins - Claude Code Docs](https://code.claude.com/docs/en/discover-plugins) -- official docs (HIGH confidence)
- [Official marketplace.json - anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json) -- real-world example (HIGH confidence)
- [BUG: category and source keys leak into cached plugin.json - Issue #26555](https://github.com/anthropics/claude-code/issues/26555) -- confirmed bug (HIGH confidence)
- [DOCS: Plugin name collision with marketplace name - Issue #24389](https://github.com/anthropics/claude-code/issues/24389) -- confirmed issue (HIGH confidence)
- [BUG: Marketplace installation doesn't clone git submodules - Issue #17293](https://github.com/anthropics/claude-code/issues/17293) -- known limitation (MEDIUM confidence)

---
*Pitfalls research for: Claude Code plugin marketplace (personal, thin-catalog)*
*Researched: 2026-03-18*
