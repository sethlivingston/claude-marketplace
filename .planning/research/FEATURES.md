# Feature Research

**Domain:** Personal Claude Code plugin marketplace (thin catalog pointing to external plugin repos)
**Researched:** 2026-03-18
**Confidence:** HIGH

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Valid `marketplace.json` schema | Claude Code will reject the marketplace entirely without it. The schema is well-documented and validated by `claude plugin validate .` | LOW | Must include `name`, `owner`, and `plugins` array. Use kebab-case names, avoid reserved Anthropic names. |
| At least one installable plugin | An empty marketplace is useless. Users need something to install on first visit. | LOW | `the-typescript-narrows` is the first plugin. Source via `git-subdir` pointing to the skill repo's subdirectory. |
| Plugin descriptions | Users browse the Discover tab and need to understand what each plugin does before installing. Descriptions show inline. | LOW | Set `description` on each plugin entry in marketplace.json. Keep concise (one sentence). |
| Working plugin installation | After `/plugin install plugin-name@marketplace-name`, the plugin must actually work -- skills load, commands fire, no errors in the Errors tab. | MEDIUM | Requires the target repo to have correct plugin structure: `.claude-plugin/plugin.json`, skills in `skills/` subdirectory. This is the main structural work. |
| Plugin validation passes | `claude plugin validate .` must pass cleanly. Users and CI rely on this. | LOW | Validates JSON syntax, schema conformance, kebab-case names, source paths, and frontmatter in skill/agent/command files. |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Opinionated skill quality | Seth's plugins encode strong, well-researched opinions (like TypeScript narrowing best practices). The marketplace is a curated collection, not a dumping ground. Quality over quantity. | LOW (organizational, not technical) | This is a content differentiator, not a code feature. Each skill should have thorough `SKILL.md` with references. |
| `strict: false` marketplace-side control | Allows the marketplace to define plugin component exposure without requiring the source repo to restructure. Useful when the skill repo serves multiple purposes beyond being a Claude plugin. | LOW | Set `strict: false` on plugin entries where the marketplace should be the authority for component definitions. Avoids conflicts with any `plugin.json` in the source repo. |
| Version pinning with `ref` and `sha` | Pin plugins to specific commits or tags for reproducibility. Users get a known-good version, not whatever happens to be on `main`. | LOW | Add `ref` (branch/tag) and optionally `sha` (exact commit) to the `source` object. Prevents breaking changes from propagating automatically. |
| Plugin metadata richness | `author`, `homepage`, `repository`, `license`, `keywords`, `category`, `tags` make plugins discoverable and professional in the Discover tab. | LOW | All optional fields but they make the marketplace feel polished. `homepage` can link to the skill repo README. |
| Multiple complementary plugins | A curated set of plugins that work well together (e.g., TypeScript narrowing + other TS best practices). The marketplace becomes a "style guide as code" toolkit. | MEDIUM | Each new plugin requires a properly structured skill repo. The marketplace.json update itself is trivial. |
| CI validation via GitHub Actions | Automated `claude plugin validate .` on every push/PR ensures the marketplace never ships broken. | LOW | Use a simple GitHub Actions workflow. The `ivan-magda/claude-code-plugin-template` repo has a `validate-plugins.yml` example. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Hosting plugins inside the marketplace repo | Seems simpler than maintaining separate repos. Everything in one place. | Couples plugin development to marketplace releases. Plugins can't be versioned independently. Bloats the marketplace repo. Claude copies plugins to cache -- external references (like `../shared-utils`) break. | Keep plugins in their own repos. Use `git-subdir`, `github`, or `url` source types. The marketplace is a thin catalog only. |
| Auto-update enabled by default | Users always get the latest. | For a personal marketplace, breaking changes in a skill could silently change behavior for all users. Third-party marketplaces have auto-update OFF by default (correctly). | Let users opt-in to auto-update. Pin versions with `ref`/`sha` for stability. |
| Community contributions / PRs | Grow the marketplace through community input. | PROJECT.md explicitly scopes this as a personal marketplace. Reviewing community PRs adds maintenance burden. Quality control becomes harder. | Keep personal. If a community marketplace is wanted later, create a separate repo for it. |
| npm/pip source types | Broader distribution options. | Adds complexity for no gain at this scale. Git-based sources are simpler and sufficient. PROJECT.md marks this out of scope. | Stick with `git-subdir` and `github` source types. |
| CI/CD automation for plugin updates | Automatically sync marketplace when plugin repos update. | Over-engineering for a personal marketplace with a handful of plugins. Manual updates are fine. PROJECT.md marks this out of scope. | Manually update `marketplace.json` when plugin versions change. Consider CI only for validation (not deployment). |
| Wrapping third-party plugins | Re-distributing other people's plugins through your marketplace. | License concerns, maintenance burden, version drift. The official marketplace already handles curation at scale. | Only distribute your own plugins. Link to third-party plugins in documentation if desired. |

## Feature Dependencies

```
[Valid marketplace.json]
    +-- requires --> [Correct schema structure]
    +-- requires --> [At least one plugin entry]
                        +-- requires --> [Plugin repo with valid structure]
                                            +-- requires --> [.claude-plugin/plugin.json in source repo]
                                            +-- requires --> [skills/ directory with SKILL.md files]

[Working plugin installation]
    +-- requires --> [Valid marketplace.json]
    +-- requires --> [Plugin repo with valid structure]
    +-- requires --> [Correct source type config (git-subdir)]

[Version pinning]
    +-- enhances --> [Working plugin installation]

[CI validation]
    +-- enhances --> [Valid marketplace.json]

[Plugin metadata richness]
    +-- enhances --> [Plugin descriptions]
```

### Dependency Notes

- **Working plugin installation requires Plugin repo with valid structure:** This is the core dependency. The marketplace.json can be perfect, but if the skill repo at `sethlivingston/the-typescript-narrows` doesn't have the right directory layout (`.claude-plugin/plugin.json`, `skills/the-typescript-narrows/SKILL.md`), installation will fail or the skill won't load.
- **`strict: false` can bypass plugin.json requirement:** If set, the marketplace entry defines all component paths, and the source repo must NOT have its own `plugin.json` declaring components (or it will conflict). This is an either/or choice per plugin.
- **CI validation enhances marketplace.json:** Not required for functionality, but catches breakage before users hit it. Add after the core marketplace works.

## MVP Definition

### Launch With (v1)

Minimum viable product -- what's needed to validate the concept.

- [x] Valid `.claude-plugin/marketplace.json` with correct schema -- the marketplace exists
- [ ] `the-typescript-narrows` plugin entry with `git-subdir` source pointing to the skill repo
- [ ] Skill repo restructured to valid plugin layout (`.claude-plugin/plugin.json`, `skills/` directory)
- [ ] `claude plugin validate .` passes for both marketplace and plugin
- [ ] Users can add marketplace and install the plugin end-to-end

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Plugin metadata enrichment (`author`, `homepage`, `keywords`, `license`) -- once the basic flow works
- [ ] Version pinning with `ref`/`sha` -- once the plugin has a tagged release worth pinning
- [ ] GitHub Actions CI for validation -- once there's enough to validate (more than one plugin, or after the first breakage)
- [ ] Second plugin added to the marketplace -- validates the "catalog" pattern at n>1

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] `strict: false` mode for plugins where the source repo shouldn't need Claude plugin structure -- only if a specific plugin needs it
- [ ] Release channels (stable vs latest refs) -- only if user base grows enough to warrant it
- [ ] Team distribution via `extraKnownMarketplaces` in `.claude/settings.json` -- only if recommending to teams

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Valid marketplace.json | HIGH | LOW | P1 |
| Plugin repo restructuring | HIGH | MEDIUM | P1 |
| Plugin descriptions | HIGH | LOW | P1 |
| Validation passes | HIGH | LOW | P1 |
| End-to-end install works | HIGH | MEDIUM | P1 |
| Metadata enrichment | MEDIUM | LOW | P2 |
| Version pinning | MEDIUM | LOW | P2 |
| CI validation workflow | MEDIUM | LOW | P2 |
| Second plugin | MEDIUM | MEDIUM | P2 |
| strict:false mode | LOW | LOW | P3 |
| Release channels | LOW | MEDIUM | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | Official Anthropic Marketplace | ivan-magda/template | dashed/claude-marketplace | Our Approach |
|---------|-------------------------------|---------------------|--------------------------|--------------|
| Plugin count | 46+ (curated, multi-author) | 2 (hello-world + dev toolkit) | Personal skills collection | Start with 1, grow organically with own plugins only |
| Plugin types | Skills, agents, hooks, MCP, LSP | Skills, commands | Skills | Skills first (matches the-typescript-narrows). Add other types as needed. |
| Source types | Relative paths (plugins in repo) | Relative paths | Local paths | `git-subdir` (plugins in external repos). Thin catalog pattern. |
| CI/CD | Internal review process | GitHub Actions validation | None | Add GitHub Actions validation after MVP |
| Documentation | Extensive official docs | 8 guides in docs/ | README only | README with usage instructions. Link to official docs for concepts. |
| strict mode | true (plugins self-describe) | true (default) | N/A | Start with true. Use false only if a specific plugin needs it. |

## Sources

- [Official marketplace docs: Create and distribute](https://code.claude.com/docs/en/plugin-marketplaces) -- HIGH confidence, authoritative
- [Official docs: Discover and install plugins](https://code.claude.com/docs/en/discover-plugins) -- HIGH confidence, authoritative
- [Anthropic official plugin directory](https://github.com/anthropics/claude-plugins-official) -- HIGH confidence, reference implementation
- [Plugin template by ivan-magda](https://github.com/ivan-magda/claude-code-plugin-template) -- MEDIUM confidence, community best practice
- [Personal marketplace example by dashed](https://github.com/dashed/claude-marketplace) -- MEDIUM confidence, similar use case

---
*Feature research for: Claude Code plugin marketplace*
*Researched: 2026-03-18*
