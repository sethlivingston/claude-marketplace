# Roadmap: Claude Marketplace

## Overview

This project delivers a personal Claude Code plugin marketplace in three phases: first restructure the skill repo into a valid plugin, then create the marketplace catalog and validate end-to-end installation, then add polish (metadata, version pinning, CI). The hard dependency is structural -- the marketplace cannot reference a plugin that does not yet have valid plugin structure.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Plugin Structure** - Restructure the-typescript-narrows into a valid Claude Code plugin
- [ ] **Phase 2: Marketplace Catalog** - Create marketplace.json and validate end-to-end installation
- [ ] **Phase 3: Polish** - Add metadata, version pinning, and CI validation

## Phase Details

### Phase 1: Plugin Structure
**Goal**: The-typescript-narrows skill repo is a valid, self-contained Claude Code plugin
**Depends on**: Nothing (first phase)
**Requirements**: PLUG-01, PLUG-02
**Success Criteria** (what must be TRUE):
  1. The skill repo has a `.claude-plugin/plugin.json` manifest with correct name, description, and version
  2. Skill files live at `skills/the-typescript-narrows/SKILL.md` within the plugin subdirectory
  3. `claude plugin validate .` passes when run against the plugin directory
**Plans**: 1 plan

Plans:
- [ ] 01-01-PLAN.md — Restructure files, create manifest, update references, validate plugin

### Phase 2: Marketplace Catalog
**Goal**: Users can add the marketplace and install the-typescript-narrows plugin end-to-end
**Depends on**: Phase 1
**Requirements**: MKTPL-01, MKTPL-02, MKTPL-03, MKTPL-04, PLUG-03, PLUG-04
**Success Criteria** (what must be TRUE):
  1. `.claude-plugin/marketplace.json` exists with a valid name, owner, and plugins array
  2. The plugin entry uses `git-subdir` source pointing to the correct subdirectory of the skill repo
  3. User can run `/plugin marketplace add sethlivingston/claude-marketplace` successfully
  4. User can install the-typescript-narrows and invoke the `/the-typescript-narrows` skill
  5. `claude plugin validate .` passes on the marketplace repo with no errors
**Plans**: TBD

Plans:
- [ ] 02-01: TBD

### Phase 3: Polish
**Goal**: The marketplace feels professional and resists accidental breakage
**Depends on**: Phase 2
**Requirements**: POLSH-01, POLSH-02, POLSH-03
**Success Criteria** (what must be TRUE):
  1. Plugin entries include author, homepage, repository, license, and keywords metadata
  2. The `git-subdir` source pins to a specific ref or sha for version stability
  3. A GitHub Actions workflow runs `claude plugin validate .` on push and PR
**Plans**: TBD

Plans:
- [ ] 03-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Plugin Structure | 0/1 | Not started | - |
| 2. Marketplace Catalog | 0/? | Not started | - |
| 3. Polish | 0/? | Not started | - |
