# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — Claude Marketplace MVP

**Shipped:** 2026-03-18
**Phases:** 3 | **Plans:** 4 | **Sessions:** ~3

### What Was Built
- Valid Claude Code plugin structure for the-typescript-narrows
- Marketplace catalog (`marketplace.json`) with git-subdir source referencing the plugin
- End-to-end flow: marketplace add → plugin discovery → installation → skill invocation
- Metadata enrichment (author, homepage, license, keywords) for plugin and marketplace
- Version pinning via `plugin/v1.0.0` tag
- GitHub Actions CI for automated marketplace validation

### What Worked
- Linear dependency chain (plugin structure → marketplace → polish) kept phases clean with no rework
- Single-day execution for entire milestone — small, focused scope paid off
- HTTPS URL decision caught early (Phase 2) avoided SSH-only breakage for users
- Audit passed cleanly on first run — requirements were well-scoped

### What Was Inefficient
- Roadmap progress table got out of sync (Phase 2 showed "Planned" after completion) — manual tracking drift
- Nyquist validation was partial across all phases — VALIDATION.md files created but never fully completed
- Phase 2 SUMMARY frontmatter didn't extract one-liners via gsd-tools (format mismatch)

### Patterns Established
- `plugin/v*` tag convention for plugin releases (replacing `skill/v*`)
- HTTPS URLs for all plugin source references (never GitHub shorthand)
- CI validation as a gate for marketplace changes

### Key Lessons
1. For catalog-only repos, deliverable LOC is tiny but the integration testing matters most — E2E validation was the real work
2. Tag convention changes should happen early (we did it in Phase 3 but could have decided in Phase 1)
3. Nyquist validation should be run during execution, not deferred — partial compliance is noise

### Cost Observations
- Model mix: ~70% opus, ~30% sonnet (estimation)
- Sessions: ~3
- Notable: Entire milestone completed in a single day due to small, well-defined scope

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Sessions | Phases | Key Change |
|-----------|----------|--------|------------|
| v1.0 | ~3 | 3 | Initial project — established plugin marketplace pattern |

### Top Lessons (Verified Across Milestones)

1. (Awaiting second milestone for cross-validation)
