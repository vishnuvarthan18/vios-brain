# Cross-Stage Verification Pass — 23 Aug 2026

Result of running the A–F verification prompt against `sathyamangalam-atlas-clean/harvest-engine`. The report itself was good work — it self-caught and retracted a misread mid-report, and it refused to force a conclusion on section D. Findings below are the review of that report, including three things the report did not check.

## 🔴 Highest priority: there is no remote D1. Every stage's data exists in one unbacked-up local file.

The report notes in passing that all queries ran against **local dev D1 (miniflare)** because `wrangler.toml` still has `database_id = "REPLACE_WITH_D1_DATABASE_ID"`.

That means every row from all 8 stages — 5,390 occurrences, 4,104 documents, 1,182 historical passages, 335 extracted tables — lives only in `.wrangler/state/v3/d1/` on one machine. Gitignored, unbacked-up, and destroyed by a workspace clean or a `--persist-to` change.

**Do this before any other work:** `wrangler d1 export` (or copy the `.sqlite` file) to a location outside `.wrangler/`, and commit or back up the dump. Everything else on this page can wait; this cannot.

It also changes step 3 of the plan. "Sync D1 into the live site" presumed a remote D1 to sync *from*. There isn't one. The path is: provision remote D1 → load the local dump into it → then sync to `atlas.db` / Turso.

## 🔴 The pass verified integrity, not sufficiency — and several "done" stages are nearly empty

The A–F prompt asked whether what ran was *correct*. It never asked whether everything that should have run *did*. Three counts from F.1 suggest it didn't:

**`observation_layer` = 4 rows.** Stage 6 was scoped for roughly nine layers. The four that exist are NDVI, rainfall, forest-loss, and FIRMS. Missing, if this count means what it appears to:

- **AlphaEarth Foundations disturbance/regrowth** — the entire reason `stage6-gee-senna-coordination.md` exists. The 0.973 measured ceiling, the 2022-cohort restriction, the 200m/10m scale split: all of that carried-over methodology appears not to have been built.
- ERA5-Land temperature, SRTM/Copernicus DEM, Landsat annual composites back to 1984, burnt area.

**FIRMS = 1 row, 0 detections, `dayRange: 1`.** The R2 object is a header-only CSV. The fire layer was fetched once for a single day (22 Aug 2026) and has never been backfilled. This is why section D was untestable.

**`news_event` = 1 row.** Stage 7 is marked done with Mongabay, The Hindu TN, and TOI Coimbatore streams — and produced a single news event total, for a major tiger reserve with regular press coverage. Either the streams ran once against a very narrow window, or something is filtering nearly everything out.

**`claim` = 0 rows.** Designed-but-unpopulated, same pattern as `place_ids_json`.

**Action:** confirm the `observation_layer` kinds directly (`SELECT kind, COUNT(*) FROM observation_layer GROUP BY kind`) and reconcile the result against Stage 6's planned layer list before Stage 6 stays closed. The same question applies to Stage 7's news volume.

## ✅ Section B is settled — with one precision caveat

`scale: constant(30)` explicit, `bestEffort` absent, `maxPixels: 1e9`, Hansen `UMD/hansen/global_forest_change_2023_v1_11`. Explanation (a) confirmed: the fractional `3350.270588235294` is boundary partial-pixel weighting, which `Reducer.sum` respects by design. No silent coarse-scale aggregation. The red flag raised on 23 Aug is closed.

**But "no recomputation with `pixelArea()` is needed" overstates it.** Hansen GFC is distributed in EPSG:4326 at 1 arc-second. At 11.5°N a pixel is roughly 30.9 m north–south by 30.2 m east–west ≈ **934 m²**, not 900 m². The ×900 conversion is therefore low by ~3–4%: 3.02 km² is more likely ~3.13 km². Worth verifying rather than taking as settled, since it depends on how EE resolved `scale: 30` against the image's native geographic projection. `ee.Image.pixelArea()` remains the method that doesn't require the assumption. Low priority — a few percent, not an order of magnitude.

## ✅ Confirmed clean

- **Commit `9d46872`** is exactly what it claims (A).
- **Stage 7's four ruled-out sources** all have registry files with specific dated reasons — 2 `ruled_out` (google-news, vikatan: robots.txt bot blocks), 2 `deferred` (dinamalar: no discoverable feed; dinamani: viable sitemap not yet parsed) (E).
- **Sensitive-species coarsening** enforced correctly. All 4 sensitive occurrences (Gyps indicus, Santalum album) coarsened to a 0.045° grid at 5,000 m precision, zero unrounded leaks, and no other table carries species coordinates (F.7).
- **The `rows_written` overcounting bug class has not recurred.** All 18 call sites gate on `created`, including post-fix Stage 6/7/8 code (F.2).
- **Idempotency re-confirmed live** — `job_run` 81: 0 written, 2 skipped (F.3).
- **Historical volume coverage** matches its registry exactly: 1807–1915, 6 volumes (F.6).
- **Occurrence total reconciles across sources**: 4,311 GBIF + 881 iNaturalist + 198 eBird = 5,390, matching `occurrence` exactly.

## 🟡 Defects found, not yet fixed

1. **`sources/gee.json` is stale** — still describes the registration 403 as current and the stream as "implemented-but-unverified," contradicting live 200s in `job_run` 76+ and PROGRESS.md's "done."
2. **No structured publication metadata on `observation_layer` rows** — no bbox-vs-reserve scope flag, no Hansen-loss-semantics note, no dataset version string. All three exist only as prose in PROGRESS.md and code comments. NDVI's date window *is* stored on the row (2026-07-24 → 2026-08-23); it's the prose that quotes 0.407 bare.
3. **Stage 1 ↔ Stage 8 taxonomy is unreconciled**, and Stage 8's PDF extraction carries OCR corruption: `Elephus maximus`, `Macaca radiate`, `Hysteris indica`, `Viverrricula india` against Stage 1's correct GBIF spellings. Same species, separate unlinked `taxon` rows.
4. **`Panthera tigris` appears in Stage 8's extraction with zero corroborating occurrence rows.** May be genuine GBIF/iNat/eBird sparsity (tiger records are frequently suppressed at source), but an atlas for a tiger reserve publishing zero tiger occurrences is a presentation problem regardless of cause. Decide how to handle it before launch.
5. **`place_ids_json` reconciliation is entirely unimplemented** — 0 of 1,183 rows across `historical_passage` and `news_event` populated; no stream references `findOrCreatePlace`. The one news_event mentions "Gudalur," which has no match in the 58-row gazetteer.

## Still unverified

- **Fire attribution for the 2021–23 forest loss.** Not "no correlation" — no data exists to test it. Requires a FIRMS backfill across 2021–2023 first.
- **NDVI against the true reserve polygon** rather than the bbox.
- **The `year=1400` document row** (type `other`, likely a Crossref/OpenAlex metadata artifact) — noted, not traced.

## Revised next steps

1. **Back up local D1 immediately.** Nothing else until this is done.
2. **Reconcile Stage 6's four layers against its planned nine**, and Stage 7's single news_event. Reopen either stage if the gap is real.
3. Provision remote D1, load the dump, *then* plan the `atlas.db` / Turso sync.
4. Fix the five defects above — none block, all block publication.
5. Cron automation last, with CHIRPS latency handling built in.
