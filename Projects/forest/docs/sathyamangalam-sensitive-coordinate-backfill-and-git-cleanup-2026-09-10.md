# Sensitive-coordinate backfill + git cleanup — 10 Sep 2026 — DONE

## The bug
5 taxon rows in harvest-engine's D1 had `is_sensitive = 0` when they should have been 1: Elephas maximus indicus, Melursus ursinus ursinus, Panthera pardus fusca, Gyps bengalensis (x2 — one exact match, one with author citation). These were created before the trinomial/authorship name-parsing fix (`src/lib/species-name.js`) landed in an earlier session — that fix only applies to NEW taxon rows going forward, `findOrCreateTaxon` never revisits `is_sensitive` on existing rows. Result: 13 occurrence rows (5 elephant, 2 sloth bear, 2 leopard, 4 vulture sightings) had their exact GPS coordinates copied to `public_lat`/`public_lon` in D1 instead of being blurred.

## Real-world impact: none, caught in time
`atlas.db` (what the live site actually reads) was NEVER exposed — the separate `sync_d1_to_atlas.py` script's own `SensitivityOracle` safety net independently re-derives sensitivity and catches exactly this case, so these 13 records were already coarsened on the site side before this fix. Confirmed by checking atlas.db directly: all showed round, blurred coordinates already. This was a real bug in D1 with zero public exposure, not a live leak.

## Fix applied
New script: `harvest-engine/scripts/backfill-sensitive-taxa.mjs` (dry-run by default, `--apply` to write). Two-step, reusing the real production trigger instead of reimplementing the coarsening math:
1. `UPDATE taxon SET is_sensitive = 1 WHERE id IN (...)` for the 5 affected ids.
2. `UPDATE occurrence SET lat = lat WHERE taxon_id IN (...)` — a value-no-op that still fires SQLite's `trg_occurrence_coarsen_update` trigger (AFTER UPDATE OF lat), which recomputes `public_lat`/`public_lon` correctly using the same logic already verified in `scripts/verify-coarsening.sql`.

Dry run confirmed exactly 5 taxa / 13 occurrences (matching the sync script's earlier finding precisely) before applying. Applied in production, all 13 verified "OK coarsened" afterward.

## Also fixed today (same session): news_event sync gap
See `site-sync-and-deploy-2026-09-10.md` for full detail — `sync_d1_to_atlas.py` never extracted/loaded `news_event`, so articles from the fixed news streams (mongabay-india, toi-coimbatore, thehindu-tn) could never reach the live site. Added `sync_news_events()`.

## Git cleanup
Found 11 site-build scripts that existed only on Vishnu's Mac, never committed: `apply_schema_migration_2026-09-07.py`, `build_comparables_html.py`, `build_credits.py`, `build_history_pages.py`, `build_place_pages.py`, `build_species_pages.py`, `comparables_template.html`, `make_single_file.py`, `site_common.py`, `sync_d1_to_atlas.py`, `tier_documents.py` — including `sync_d1_to_atlas.py` itself, the very script central to today's fixes. Core pipeline scripts (`export_from_db.py`, `build_dist.py`, `devserve.py`, `deploy-prod.sh`) were already tracked; these 11 newer ones were not. All 11 plus the new `backfill-sensitive-taxa.mjs` committed and pushed as `8af71d8`.

## Full sequence today, in order
1. Deployed harvest-engine (17 fix commits + wrangler.toml email-binding fix) — `harvest-engine-deploy-2026-09-10.md`
2. Investigated shodhganga rewrite, deprioritized — `shodhganga-rewrite-investigation-2026-09-10.md`
3. Researched openalex/CORE, decided to enable both — `openalex-and-core-research-2026-09-10.md`
4. Synced D1 -> atlas.db -> exports -> dist -> deployed to sathyamangalam.online, found and fixed the news_event sync gap — `site-sync-and-deploy-2026-09-10.md`
5. Found and fixed the sensitive-taxon is_sensitive backfill gap (this doc)
6. Committed 11 previously-untracked scripts

## Still open
- Register free OpenAlex API key (Vishnu's action item).
- Build new CORE stream.
- Resolve the 3 data conflicts on the live site (tiger/elephant/leopard counts — two values each on file).
- bhl (no key), wdpa (dead end, left as-is), Email Routing for DLQ alerts (skipped, free to do anytime).
- Remember: sync -> export -> build -> deploy must be re-run manually after any future harvest-engine change that should reach the live site.
