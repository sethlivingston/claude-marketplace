# Requirements: Claude Marketplace

**Defined:** 2026-03-18
**Core Value:** Coders can discover and install Seth's Claude Code plugins from a single, well-structured marketplace

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Marketplace Catalog

- [ ] **MKTPL-01**: Marketplace repo has valid `.claude-plugin/marketplace.json` with name, owner, and plugins array
- [ ] **MKTPL-02**: Each plugin entry has a concise description explaining what it does
- [ ] **MKTPL-03**: Marketplace passes `claude plugin validate .` with no errors
- [ ] **MKTPL-04**: User can add marketplace via `/plugin marketplace add sethlivingston/claude-marketplace`

### Plugin Structure

- [ ] **PLUG-01**: The-typescript-narrows skill repo has `.claude-plugin/plugin.json` manifest
- [ ] **PLUG-02**: The-typescript-narrows skill files are in `skills/the-typescript-narrows/` subdirectory
- [ ] **PLUG-03**: Marketplace references the-typescript-narrows via `git-subdir` source pointing to the plugin subdirectory
- [ ] **PLUG-04**: User can install the-typescript-narrows and use the `/the-typescript-narrows` skill

### Polish

- [ ] **POLSH-01**: Plugin entries include metadata (author, homepage, repository, license, keywords)
- [ ] **POLSH-02**: Plugin entries pin to a specific ref or sha for version stability
- [ ] **POLSH-03**: GitHub Actions workflow validates marketplace on push/PR

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Expansion

- **EXPN-01**: Second plugin added to marketplace to validate catalog pattern at n>1
- **EXPN-02**: Release channels (stable vs latest refs) for plugins with growing user base
- **EXPN-03**: Team distribution via `extraKnownMarketplaces` in `.claude/settings.json`

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Plugins hosted in marketplace repo | Marketplace is catalog-only; all plugins live in external repos |
| Community contributions | Personal marketplace, no review burden needed |
| npm/pip distribution | Git-based sources are sufficient at this scale |
| Auto-update configuration | Default off for third-party marketplaces is correct |
| strict: false mode | Plugin repos should self-describe with their own plugin.json |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| MKTPL-01 | — | Pending |
| MKTPL-02 | — | Pending |
| MKTPL-03 | — | Pending |
| MKTPL-04 | — | Pending |
| PLUG-01 | — | Pending |
| PLUG-02 | — | Pending |
| PLUG-03 | — | Pending |
| PLUG-04 | — | Pending |
| POLSH-01 | — | Pending |
| POLSH-02 | — | Pending |
| POLSH-03 | — | Pending |

**Coverage:**
- v1 requirements: 11 total
- Mapped to phases: 0
- Unmapped: 11 ⚠️

---
*Requirements defined: 2026-03-18*
*Last updated: 2026-03-18 after initial definition*
