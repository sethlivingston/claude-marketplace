---
phase: 1
slug: plugin-structure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Claude Code CLI (plugin validate) |
| **Config file** | none — built into `claude` CLI |
| **Quick run command** | `cd plugin/the-typescript-narrows && claude plugin validate .` |
| **Full suite command** | Same as quick run (single validation command) |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd plugin/the-typescript-narrows && claude plugin validate .`
- **After every plan wave:** Run `cd plugin/the-typescript-narrows && claude plugin validate .`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 01-01-01 | 01 | 1 | PLUG-01 | smoke | `cd plugin/the-typescript-narrows && claude plugin validate .` | N/A — uses CLI | ⬜ pending |
| 01-01-02 | 01 | 1 | PLUG-02 | smoke | `cd plugin/the-typescript-narrows && claude plugin validate .` | N/A — uses CLI | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

None — validation uses the built-in `claude plugin validate` CLI command. No test framework or fixtures needed.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Path references updated across repo | PLUG-02 | grep-based check, not plugin validate | `grep -r "skill/" ../the-typescript-narrows --include="*.md" --include="*.yml" --include="*.mjs"` returns no stale references |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
