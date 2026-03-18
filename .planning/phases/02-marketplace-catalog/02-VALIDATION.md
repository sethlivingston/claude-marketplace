---
phase: 2
slug: marketplace-catalog
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 2 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Claude Code CLI (no automated test framework — static manifest project) |
| **Config file** | none |
| **Quick run command** | `claude plugin validate .` |
| **Full suite command** | `claude plugin validate . && claude plugin marketplace add sethlivingston/claude-marketplace --scope project` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `claude plugin validate .`
- **After every plan wave:** Run `claude plugin validate . && claude plugin marketplace add sethlivingston/claude-marketplace --scope project`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 02-01-01 | 01 | 1 | MKTPL-01 | smoke | `claude plugin validate .` | N/A (CLI) | ⬜ pending |
| 02-01-02 | 01 | 1 | MKTPL-02 | smoke | `claude plugin validate .` | N/A (CLI) | ⬜ pending |
| 02-01-03 | 01 | 1 | MKTPL-03 | smoke | `claude plugin validate .` | N/A (CLI) | ⬜ pending |
| 02-01-04 | 01 | 1 | PLUG-03 | smoke | `claude plugin validate .` | N/A (CLI) | ⬜ pending |
| 02-01-05 | 01 | 1 | MKTPL-04 | e2e | `claude plugin marketplace add sethlivingston/claude-marketplace` | N/A (CLI) | ⬜ pending |
| 02-01-06 | 01 | 1 | PLUG-04 | manual | Install + invoke `/the-typescript-narrows` | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

Existing infrastructure covers all phase requirements. The Claude Code CLI itself is the test tool — no test framework installation needed.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| User can install and invoke `/the-typescript-narrows` skill | PLUG-04 | Requires interactive Claude Code session to verify skill invocation | 1. `claude plugin install the-typescript-narrows` 2. Start Claude Code 3. Type `/the-typescript-narrows` 4. Verify skill loads |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
