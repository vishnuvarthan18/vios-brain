# Session summary — 8 Sep 2026: project timeline explainer + "will the new engine fail again" discussion

This session was conversational (no building), covering three things Vishnu asked for. Saved so future sessions have it without re-deriving.

## 1. Full project timeline, explained in plain English

- **Aug 10, 2026 — start.** Vishnu wanted forest/mountain work using tech skills, no exams. Research surfaced one strong wedge: nobody keeps an updated map of invasive plants (Senna spectabilis) regrowing after clearance in Sathyamangalam/Nilgiris reserves.
- **Mid-late Aug — grew into the Sathyamangalam Atlas.** A database + website meant to be a complete sourced reference for every plant/animal/place in the landscape, not just counts. Built a "harvest engine" pulling from iNaturalist, GBIF, OSM, government docs, etc., plus NDVI/satellite work on Senna regrowth.
- **Aug 23-25 — reality check.** Outside review found coverage numbers unreliable, empty `claim` table, gazetteer at ~4%, naming confusion (Atlas vs Record). Paused building, commissioned deep research on comparable worldwide projects.
- **Sep 6 — scope locked.** Defined "done" precisely: every plant/animal/place needs its own full page with facts, sources, sightings, population trends, threats, honest gap-labeling — not just a database count. Big blockers: gazetteer only 93 of up to 2,000 places, species not categorized, no occurrence page, no population-over-time structure, no photo pipeline.
- **Sep 6-7 — recent build work.** Fixed sensitive-species coordinate exposure (dev only, not yet deployed). Applied 22-table schema migration on dev. Tiered documents by reliability. Widened places 43→81 (capped, needs data.gov.in API key). Known bug: crocodile/king cobra miscategorized.
- **Sep 7-8 overnight — detail pages build.** Species detail page template (tested on 5 species), category browser rewired to new schema, place detail page template (tested on 3 places), bibliography/sources page. All on dev/staging only.

## 2. What the finished Atlas will look like (explained when asked)

A Wikipedia-style site scoped to Sathyamangalam only: separate browsable sections per category (Birds, Mammals, Insects, Reptiles, Trees, Plants, Fungi, Fish, Amphibians). Every species page has bilingual names, real sighting locations/dates (not "found in India"), sourced population-trend timeline, relationships (predator/prey/habitat), threats, photos with attribution, local/tribal names where documented, and an honest completeness label instead of hidden blanks. Every place page is a hub linking every species/document/history tied to it. A `claim` table holds disputed/competing figures instead of silently picking one number. Explicitly out of scope: new fieldwork, interviews, oral history — everything comes from already-existing documents/datasets.

## 3. "Will a new engine just fail the same way again?" — the answer given

The old harvest-engine failure wasn't a bad idea, it was a silent bug: `requestHash`-based dedup had no time window, so streams reported "success" for over a week while not actually calling most APIs (crossref, iNaturalist, GBIF, EuropePMC, IA-advancedsearch all frozen since Aug 27-29; only unpaywall was genuinely still working, draining to zero). The health-check guard couldn't catch it because it required `rowsSkippedDuplicate === 0`, the exact counter the bug inflated.

Separately, on Sep 6 there was a process incident: the dashboard Worker got deployed to production without Vishnu's explicit go-ahead (violated the standing no-deploy-without-explicit-turn-by-turn-approval rule). Public site was NOT touched; only the backend dashboard, and only locally-merged git (never pushed to origin). Rule was restated and tightened afterward.

Answer given: a rebuild will repeat the same failure class unless these specific fixes (already identified in `harvest-engine-dedup-freeze-and-pause-2026-09-05.md`, none deployed yet) go in first:
1. Max-age refetch window per source (critical — closes the freeze bug).
2. `max_batch_size` 5→1 (shrinks blast radius of any redelivery bug).
3. Fix the health-check guard so it can't read "0 duplicates" when duplicates are actually happening.
4. Make retry logic actually retry failed HTTP responses, not just thrown errors.
Plus the standing deploy-approval rule stays in force regardless of which engine runs.

Vishnu did not yet say whether to proceed with applying these fixes or planning a new engine from scratch — that decision is still open going into the next session.

## Open decisions carried forward (unchanged from before this session)
- API key for data.gov.in (places growth).
- When to deploy sensitive-species fix to production.
- When to fix crocodile/king cobra category bug.
- dev/main branch reconciliation.
- Whether to fix-forward the existing harvest engine (9 ranked fixes, none shipped) vs. build something new — raised this session, not resolved.
