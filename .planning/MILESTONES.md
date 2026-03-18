# Milestones

## v1.0 Claude Marketplace MVP (Shipped: 2026-03-18)

**Phases completed:** 3 phases, 4 plans
**Timeline:** 1 day (2026-03-18)
**Commits:** 33
**Delivered:** A working Claude Code plugin marketplace catalog with one published plugin

**Key accomplishments:**

1. Restructured the-typescript-narrows into a valid Claude Code plugin with `.claude-plugin/plugin.json` and `skills/` directory
2. Created marketplace catalog (`marketplace.json`) with `git-subdir` source referencing the plugin
3. Validated end-to-end: marketplace add → plugin discovery → installation → skill invocation
4. Enriched plugin and marketplace metadata (author, homepage, license, keywords)
5. Pinned plugin ref to `plugin/v1.0.0` tag for version stability
6. Added GitHub Actions CI workflow for automated marketplace validation

**Archives:** [ROADMAP](milestones/v1.0-ROADMAP.md) | [REQUIREMENTS](milestones/v1.0-REQUIREMENTS.md) | [AUDIT](milestones/v1.0-MILESTONE-AUDIT.md)

---
