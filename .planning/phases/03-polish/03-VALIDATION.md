---
phase: 3
slug: polish
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 3 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Claude Code CLI (`claude plugin validate .`) |
| **Config file** | `.claude-plugin/marketplace.json` |
| **Quick run command** | `claude plugin validate .` |
| **Full suite command** | `claude plugin validate .` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `claude plugin validate .`
- **After every plan wave:** Run `claude plugin validate .`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 03-01-01 | 01 | 1 | POLSH-01 | smoke | `claude plugin validate .` | N/A | ⬜ pending |
| 03-01-02 | 01 | 1 | POLSH-02 | manual | `grep "ref" .claude-plugin/marketplace.json` | N/A | ⬜ pending |
| 03-01-03 | 01 | 2 | POLSH-03 | manual | `test -f .github/workflows/validate.yml` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `.github/workflows/validate.yml` — CI workflow file (created as part of POLSH-03 task)

*Existing infrastructure covers metadata validation via `claude plugin validate .`*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Ref pinning resolves correctly | POLSH-02 | Requires network access to resolve git tag | Verify `plugin/v1.0.0` tag exists on the-typescript-narrows, then check `ref` field in marketplace.json matches `refs/tags/plugin/v1.0.0` |
| CI workflow triggers on push/PR | POLSH-03 | Requires actual GitHub push to verify | Push a commit or open a PR and verify workflow runs in Actions tab |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
