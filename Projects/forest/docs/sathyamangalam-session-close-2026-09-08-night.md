# Session close — 8 Sep 2026, night

Read-me-first pointer for the morning. Everything below is already saved in the project.

## What's running overnight
The deploy-prep run for the sensitive-species safety fix. See
`sathyamangalam/deploy-safety-fix-plan-2026-09-08.md` for the full plan.
This one is scoped narrowly — ONLY the sensitivity-registry fix and
nomenclature matcher fix, cherry-picked onto a clean branch off `main`,
NOT a merge of the rest of `dev`. It stops at a pre-flight summary
(Step 4) and does NOT touch production. Step 5 (the actual deploy) is
deliberately held back until Vishnu reviews the summary and says go.

Results should land at `sathyamangalam/deploy-safety-fix-results-[date].md`.

## What to do first in the morning
1. Read this file.
2. Read the results doc once it exists.
3. Decide on Step 5 — approve or hold the actual production deploy.

## Full state of the project going into this
- Structural build complete on the test copy: Land, Life, History, People
  all exist as real, honest, sourced detail pages (species: 2,664 pages;
  places: 92 pages; history: 83 pages; people: honest empty state).
  Committed on dev as of `9c4a6fb`.
- Schema migration applied and committed (`dcc5cc5`) — 22 tables, verified
  safe, tests passing.
- Sensitive-species fix built, tested on staging, NOT yet live in
  production — this is what tonight's run is working toward safely.
- Document tiering applied on dev (34 A / 11 B / 370 C / 1,424
  unclassified).
- Gazetteer widened to 81 places; real growth blocked on
  DATA_GOV_IN_API_KEY.

## Still open, not urgent tonight
1. DATA_GOV_IN_API_KEY — only Vishnu can get this.
2. Small fixes: crocodile/king cobra mislabeling, tree category, missing
   family field, tier-C citability tension, stale report path.
3. The 48 photo-contributor dataset — open editorial question, no
   decision made yet.
4. `rm main` — stray 0-byte file still breaking git commands.
5. dev/main branch reconciliation (30+ commits, non-mechanical) — its own
   future planning session, not a quick task.
6. Outreach Tier 2 emails — was paused on "site not presentable enough,"
   arguably no longer true now that all 4 sections exist. Worth
   reconsidering once the safety fix is live.

## Reading order in the morning
1. This file
2. `sathyamangalam/deploy-safety-fix-results-[date].md` (once it exists)
3. `sathyamangalam/deploy-safety-fix-plan-2026-09-08.md` for the full plan it followed
4. `sathyamangalam/history-people-run-results-2026-09-08.md` for last night's build
