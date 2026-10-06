# Master project log / index — updated 25 Aug 2026

Running index of every doc in the project, by date, so future sessions know where things stand without re-reading everything. Sathyamangalam Atlas docs and other tracks (Tamil idea explicitly excluded — not saved anywhere) both included since this is the whole-project index.

## Today's session (25 Aug 2026) — what happened

1. Reviewed session handoff and project-position docs to establish current state.
2. External reviewer critiqued the technical blueprint: coverage numbers unreliable, empty `claim` table, `request_hash` reporting bug, gazetteer under-target, project naming collision (Atlas vs Record).
3. Ran competitive research (5 projects: ALA, NBN Atlas, GBIF, Minnesota Biodiversity Atlas, India Biodiversity Portal) — found scope mismatch (we combine 5 domains, they do 1), institutional vs solo gap, no precedent for the `claim`-table approach, no shared data standard adopted.
4. Wrote and saved a deep-research brief for a separate Cowork session: full worldwide classified list (continent → country → state) of comparable "document a place/population comprehensively" projects, plus how they work, how they fund themselves, and 4 follow-up angles (disputed-claims UX, Darwin Core, realistic timelines, contradictions).
5. Received and logged (not yet reviewed/decided) agent output on: Item 6 pixelArea/geometry verification (30.5% land / 7.4% loss ratio), NDVI annulus test result (original hypothesis wrong for 6 of 7 years, 2026 is an unexplained anomaly), and the `request_hash` audit (elevation was the only affected stream, now fixed and instrumented).
6. **Standing decision: paused.** Not making further decisions or building until the deep-research comparable-projects report comes back. Two things await Vishnu's call: production migration/deploy (needs dashboard password), and the NDVI-anomaly / "drier forest" framing editorial decision.
7. Separately, sketched a Tamil documentation-project idea in chat only — explicitly NOT saved to project memory per instruction, to be taken to a new project/chat.

## Document index by date

### 13–14 Aug 2026
- `sathyamangalam/harvester-status-2026-08-13.md`
- `sathyamangalam/data-bundle-integration-2026-08-14.md`
- `sathyamangalam/git-pipeline-2026-08-14.md`
- `sathyamangalam/session-checkpoint-2026-08-14.md`
- `senna-regrowth/verification-repo-2026-08-14.md` — separate project (GEE Senna/invasive-species script), not the Atlas
- `senna-regrowth/primer-satellite-invasive-mapping.md`

### 18 Aug 2026
- `sathyamangalam/record-outreach-2026-08-18.md` — outreach log to Field Director, DDs, PCCF, NGOs; also where the "Record" naming shows up alongside "Atlas"

### 21 Aug 2026
- `sathyamangalam/harvest-engine-session-handoff-2026-08-21.md`
- `sathyamangalam/record-db-harvester-migration-plan-2026-08-21.md`
- `claude/claude-science-wildlife-genomics-research-2026-08-21.md`

### 23 Aug 2026
- `sathyamangalam/project-position-2026-08-23.md` — the phase-by-phase plan-vs-actual assessment (source coverage 56.8% vs Atlas content coverage ~35-40%; gazetteer at ~4% flagged as biggest gap; `claim` table flagged as empty differentiator)
- `sathyamangalam/session-record-2026-08-23.md`
- `sathyamangalam/landsat-ndvi-series-2026-08-23.md`
- `sathyamangalam/completeness-pass-2026-08-23.md`
- `sathyamangalam/cross-stage-verification-2026-08-23.md`
- `sathyamangalam/stage6-gee-senna-coordination.md`

### 24 Aug 2026
- `sathyamangalam/credentials-and-wdpa-finding-2026-08-24.md` — all 5 API keys resolved except WDPA (ruled out)

### 25 Aug 2026
- `sathyamangalam/session-handoff-2026-08-25.md` — most recent full handoff before this session (polygon resolved: 791.62 km² OSM relation 4192204; two-databases-diverged flagged as the top blocker; 7 of 11 site pages are stubs)
- `sathyamangalam/reviewer-findings-and-bug-list-2026-08-25.md` — this session's running bug list (7 items) + competitive research summary
- `sathyamangalam/deep-research-brief-comparable-projects-2026-08-25.md` — the brief sent out for deep research, awaiting report back
- `sathyamangalam/agent-output-item6-ndvi-requesthash-2026-08-25.md` — latest agent output, logged not reviewed
- `sathyamangalam/master-project-log-index-2026-08-25.md` — this doc

### Undated / standing reference docs (no date in filename — check content if precise date needed)
- `sathyamangalam/harvest-engine-verification-playbook.md`
- `sathyamangalam/path-to-70-percent-coverage.md`
- `sathyamangalam/source-atlas.md`
- `sathyamangalam/harvest-plan.md`
- `sathyamangalam/atlas-website-dossier.md`
- `claude/wildlabs-profile-setup.md`
- `claude/current-plan-locked-aug-2026.md`
- `website need to resech ` (untitled/typo filename as stored)
- `claude/market-context-aug-2026.md`
- `claude/no-government-track-plan.md`
- `claude/six-track-master-plan.md`
- `claude/forest-life-options-beyond-scientist.md`
- `claude/government-scientist-master-plan.md`
- `claude/jump-now-90-day-plan.md`
- `claude/scientist-money-plan.md`
- `claude/scientist-degree-path.md`
- `forest-tech-career-context-and-findings.md` — appears to be about Vishnu's own career, separate from the Atlas
- `parts` (untitled/typo filename)
- `these are the websites need to reseach more ` (untitled/typo filename)

### Project files
- `Conservation Tech Entry Routes for a CS Graduate in Erode_ A Nilgiris FieldFirst Playbook.pdf`

## Current overall status (as of this session)

**Standing pause:** no further building or deciding on the Atlas until the deep-research comparable-projects report is back (Task 1: worldwide list by continent/country/state; Task 2: how these projects work; Task 3: how they fund/sustain; Task 4: disputed-claims UX, Darwin Core, timelines, contradictions).

**Open decisions blocking progress (carried forward, unchanged):**
1. Which database is canonical — D1 vs atlas.db
2. When/whether to launch given the page-readiness gate
3. Production migration 0005 not yet applied — needs dashboard password
4. NDVI 2026 anomaly / "drier forest" framing — editorial call pending
5. `claim` table implementation — still unbuilt
6. Project naming (Atlas vs Record) — still unresolved
7. Gazetteer under-querying bug — root cause not yet fixed

**Ruled out / closed, do not relitigate:**
- WDPA_API_KEY — ruled out, India withholds ~900 PAs
- Outreach timing to officials — confirmed not an issue by Vishnu
- request_hash bug — audited, elevation was the only affected stream, now fixed and instrumented
