---
phase: 02-marketplace-catalog
plan: 01
subsystem: infra
tags: [claude-code, marketplace, plugin-catalog, git-subdir]

# Dependency graph
requires:
  - phase: 01-plugin-structure
    provides: Valid plugin.json for the-typescript-narrows
provides:
  - Working marketplace manifest at .claude-plugin/marketplace.json
  - End-to-end plugin discovery and installation via marketplace
  - Repo boilerplate (README, LICENSE, .gitignore, CLAUDE.md)
affects: [03-polish-and-expand]

# Tech tracking
tech-stack:
  added: [claude-plugin-marketplace]
  patterns: [git-subdir source references to external plugin repos]

key-files:
  created:
    - .claude-plugin/marketplace.json
    - README.md
    - LICENSE
    - .gitignore
    - CLAUDE.md
  modified: []

key-decisions:
  - "Used HTTPS URL (https://github.com/sethlivingston/the-typescript-narrows.git) instead of GitHub shorthand for plugin source -- SSH cloning fails for users without SSH keys"
  - "owner.email omitted from marketplace.json -- validation passes without it"
  - "No version pinning (ref/sha) in plugin source -- deferred to Phase 3"

patterns-established:
  - "Plugin source URLs: Always use full HTTPS URLs, not GitHub shorthand"
  - "Marketplace structure: .claude-plugin/marketplace.json at repo root"

requirements-completed: [MKTPL-01, MKTPL-02, MKTPL-03, MKTPL-04, PLUG-03, PLUG-04]

# Metrics
duration: ~15min
completed: 2026-03-18
---

# Phase 02 Plan 01: Marketplace Manifest Summary

**Marketplace manifest with git-subdir plugin catalog, validated locally and verified end-to-end with remote install of the-typescript-narrows**

## Performance

- **Duration:** ~15 min (across two sessions with checkpoint)
- **Started:** 2026-03-18
- **Completed:** 2026-03-18
- **Tasks:** 2
- **Files modified:** 5

## Accomplishments
- Created marketplace.json with sethlivingston-plugins catalog containing the-typescript-narrows plugin entry
- Pushed to GitHub and verified marketplace add, plugin install, and skill invocation all work end-to-end
- Established HTTPS URL pattern for plugin sources after discovering SSH shorthand fails for users without SSH keys

## Task Commits

Each task was committed atomically:

1. **Task 1: Create marketplace manifest and repo boilerplate** - `3fb1741` (feat)
2. **Task 2: Push to GitHub and verify end-to-end installation** - checkpoint approved (no additional code commit needed)

Additional fix commit:
- **HTTPS URL fix** - `8b3a917` (fix) - Changed git-subdir source URL from GitHub shorthand to full HTTPS URL

## Files Created/Modified
- `.claude-plugin/marketplace.json` - Marketplace manifest with plugin catalog (the main deliverable)
- `README.md` - Minimal repo description with usage instructions
- `LICENSE` - MIT license
- `.gitignore` - Standard ignores (.DS_Store, node_modules, *.log)
- `CLAUDE.md` - Project conventions for Claude Code

## Decisions Made
- Used full HTTPS URL for plugin source instead of GitHub shorthand -- SSH cloning fails for users without SSH keys configured
- Omitted owner.email from marketplace.json -- validation passes without it, avoids exposing email
- No version pinning (ref/sha) in plugin source entries -- deferred to Phase 3 per user decision

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Changed plugin source URL from GitHub shorthand to full HTTPS URL**
- **Found during:** Task 2 (Push to GitHub and verify end-to-end installation)
- **Issue:** `claude plugin marketplace add` failed because the git-subdir source URL `sethlivingston/the-typescript-narrows` resolved to SSH clone, which fails for users without SSH keys
- **Fix:** Changed URL to `https://github.com/sethlivingston/the-typescript-narrows.git`
- **Files modified:** .claude-plugin/marketplace.json
- **Verification:** Marketplace re-added successfully, plugin installed and skill invocable
- **Committed in:** 8b3a917

---

**Total deviations:** 1 auto-fixed (1 bug fix)
**Impact on plan:** Essential fix for correct operation. No scope creep.

## Issues Encountered
- GitHub shorthand URL in git-subdir source resolved to SSH protocol, which fails for users without SSH keys. Fixed by switching to full HTTPS URL. This is now an established pattern for future plugin entries.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Marketplace is live and functional on GitHub
- Plugin discovery and installation verified end-to-end
- Ready for Phase 3 (polish and expand): version pinning, additional plugin entries, marketplace documentation enhancements

## Self-Check: PASSED
- All 5 created files exist (.claude-plugin/marketplace.json, README.md, LICENSE, .gitignore, CLAUDE.md)
- Commit 3fb1741 exists (Task 1)
- Commit 8b3a917 exists (HTTPS URL fix)

---
*Phase: 02-marketplace-catalog*
*Completed: 2026-03-18*
