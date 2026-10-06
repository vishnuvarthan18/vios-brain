# Session Record — 23 Aug 2026

Full chronological record. Companion docs: `harvest-engine-verification-playbook.md` (method), `path-to-70-percent-coverage.md` (live tally), `cross-stage-verification-2026-08-23.md` and `completeness-pass-2026-08-23.md` (findings).

## Where this session started and ended

**Started:** "7 of 8 stages done and independently verified; Stage 6 has 3 open data-quality questions."

**Ended:** Stage 6's three questions closed. The "7 of 8 done" claim shown to be wrong — verified for correctness, never for coverage. Database backed up (it had no backup). Two new GEE layers built. Coverage 52.3% → 56.8%, with a defined path to 70%.

## Sequence

### 1. Stage 6's three questions — all closed

- **NDVI 0.407** — not a bug. The bbox-dilution theory was *refuted by the agent itself*: the densest patch anywhere in the box (76% tree cover) peaked at 0.644, so dilution cannot explain a ceiling that low. Real cause, cross-checked against the management plan's own text: Sathyamangalam is dry thorn / dry mixed deciduous / semi-evergreen, not dense evergreen. The 0.6–0.9 expectation is calibrated to evergreen.
- **Rainfall 8 of 31 days** — genuine CHIRPS publication latency (~22 days), not a query bug. Clean cutoff at 31 July, zero for every probed date in August. July 24–31 = exactly 8 days.
- **Forest loss 3350.27** — flagged as fractional, therefore not a native-resolution count. Investigation confirmed `scale: 30` explicit, no `bestEffort`, `maxPixels 1e9`, `UMD/hansen/global_forest_change_2023_v1_11`. Benign boundary partial-pixel weighting. **But the ×900 m² conversion was still wrong** — see below.

### 2. Integrity pass (A–F)

Confirmed clean: commit `9d46872` is what it claims · Stage 7's four ruled-out sources all have documented registry files · sensitive-species coarsening correctly enforced, zero leaks (4 rows, 0.045° grid, 5,000 m precision) · `rows_written` overcounting bug has not recurred at any of 18 call sites · idempotency live (`job_run` 81: 0 written, 2 skipped) · historical volumes 1807–1915 match registry exactly · Stage 1 reconciles (4,311 + 881 + 198 = 5,390).

Found: stale `sources/gee.json` · no structured scope metadata · Stage 1↔8 taxonomy unreconciled with OCR corruption (`Elephus maximus`, `Macaca radiate`, `Hysteris indica`, `Viverrricula india`) · `Panthera tigris` in Stage 8 with zero occurrence rows · `place_ids_json` unimplemented at 0 of 1,183 rows.

### 3. 🔴 No remote D1 — data existed in one unbacked-up file

`wrangler.toml` still has `database_id = "REPLACE_WITH_D1_DATABASE_ID"`. Everything lived only in `.wrangler/state/v3/d1/`, gitignored, destroyed by a workspace clean. **Now dumped** to `db-backups/harvest-engine-d1-local-2026-08-23.sql` (7.58 MB), restore-verified across all 13 tables. Risk closed.

Consequence: "sync D1 to the live site" presumed a remote D1 to sync *from*. Real path is provision remote → load dump → then sync.

### 4. Completeness pass — the status correction

Only Stage 5 is complete against its own spec. Roughly 23 of 44 named sources across Stages 1–6 delivering data at the time of the pass.

| Stage | Delivering / named |
|---|---|
| 1 Biodiversity | 3 / 6 |
| 2 Literature | 7 / 9 |
| 3 Gazetteer | 2 / 6 |
| 4 Government/legal | **1 / 9** |
| 5 Historical | **6 / 6** |
| 6 Remote sensing | 4 / 8 (now 6 / 8) |
| 7 Media | 1 news_event total |
| 8 Management plan | 4 / 29 appendices |

Three kinds of gap, needing different responses: **blocked on a free credential** (WDPA, LGD, BHL) · **never built** (IUCN — key present, stream never written; IBP; Xeno-canto; Landsat; ERA5-Land; IMD; SRTM; moef/Parivesh; 25 appendices) · **structurally un-backfillable** (eBird 30-day cap; FIRMS NRT 10-day cap; Stage 7 RSS has no date parameter and can only accumulate forward).

### 5. Stage 6 AlphaEarth — resolved, *not* a defect

The master prompt contains no disturbance/regrowth deliverable in any wording. The five carried-over senna decisions (64-band embeddings, 0.973 measured ceiling, 2022 cohort cutoff, 200 m/10 m dual resolution, 2017–2024 window) are **dormant guidance for future work, not unmet requirements**. Senna species discrimination confirmed not shipped, which is correct.

### 6. Forest-loss area — corrected

`pixelArea()` returned 2,935,701.59 m² against the flat multiply's 3,015,243.53 m² — 2.64% *smaller*, opposite the predicted direction. Mechanism: implied mean pixel area 876.26 m². `scale: 30` on an EPSG:4326 image makes Earth Engine derive its grid at the equator (30 ÷ 111,320 ≈ 0.00026949°); at 11.5°N those pixels are ~29.81 × 29.40 m ≈ 876.4 m², matching to within 0.02%.

**The figure is 2.936 km² (293.6 ha), not 3.02.** All-time (8.17 km²) needs the same recompute; the 36.9% ratio survives since the error cancels.

### 7. Two GEE layers built

**SRTM/DEM** (`job_run` 82–83) — `USGS/SRTMGL1_003`. Elevation 187–2,098 m, mean 787; slope 0–79.57°, mean 11.15. Established the metadata pattern and backfilled it onto all four pre-existing rows.

**ERA5-Land** (`job_run` 86–87) — `ECMWF/ERA5_LAND/DAILY_AGGR`, chosen over HOURLY despite 1–2 days more lag because it ships pre-aggregated daily min/mean/max. Latency measured first by binary search: DAILY_AGGR 8 days, HOURLY 6–7. Mean 23.92 °C / min 20.01 / max 29.26 over 2026-08-08→15, `image_count` 7, native scale 11,132 m. Added `data_latency_days` and backfilled `22` onto the CHIRPS row.

### 8. The bbox problem, compounding

The DEM result is diagnostic about the bounding box itself: 1,911 m of relief topping 2,000 m means it spans reservoir plain to montane plateau — at least three ecological zones, including ground that is not dry deciduous at all.

So NDVI 0.407 blends vegetation types with different ceilings; forest loss counts drivers that differ by zone; temperature averages 12–13 °C of lapse-rate spread over only ~21 ERA5-Land grid cells.

**`WDPA_API_KEY` is therefore blocking publication of every spatial figure, not merely high-leverage.**

## Current state

- Coverage **25 of 44 = 56.8%**. Target 31 = 70.5%. Remaining +6.
- Stage 6 at 6 of 8 sources; only Landsat and IMD outstanding, IMD deliberately excluded.
- **Landsat prompt issued; run in flight at session end.** Expect ~41 GEE round-trips where every prior layer made one — it may exceed a single Worker invocation. Per-year idempotency was specified, which makes a timeout recoverable by re-running. A runtime addendum (commit-as-you-go, 100–200 m reduction scale for the scale-robust mean, report wall-clock) was drafted but **not included in the issued prompt**.

## Open items

**Blocking:** register `WDPA_API_KEY` (also `DATA_GOV_IN_API_KEY`, `BHL_API_KEY`).

**Verification:**
1. Reconcile the two ERA5 means — 297.43 K probe implies 24.28 °C, stored value is 23.92 °C (implying ~297.07 K). Probably different windows; unstated.
2. Confirm `image_count` 7 over an 8-day window is `filterDate` end-exclusivity, not a missing day.
3. `slope_max = 79.57°` not publishable — store a 95th percentile alongside; DEM maxima are artifact-dominated.
4. **`historical_passage` may be truncating mid-content** — passage id 39 was cut mid-number. Audit across all 1,182 rows; it may be clipping exactly the figures that make passages citable.

**Carried, unfixed:** Stage 1↔8 taxonomy reconciliation (needs a design decision) · `place_ids_json` implementation · Stage 7's Hindu TN filter missing bare `tiger`/`elephant` terms (dropped two relevant stories from a 60-item feed) · recompute all-time forest loss with `pixelArea()` · ~21-cell limitation into the ERA5 registry.

**Unverified:** FIRMS NRT-vs-archive distinction (stated from general API knowledge) · whether moef/Parivesh was ever considered · whether Overpass/Census/Bhuvan failures are local-network-only · the `year=1400` document row.

## Sequencing from here

1. Land Landsat → 26 of 44 (59%).
2. Register the three free keys — they arrive on someone else's schedule, so start that clock regardless of build order.
3. IUCN and Xeno-canto (no blockers) → 28.
4. Whatever the keys unblock → 31 = 70%.
5. **Move cron automation up.** For Stage 7 and FIRMS it is not an optimisation — it is the only mechanism by which those stages ever accumulate data, since neither can look backward. Every day without it is coverage permanently lost.
6. Remote D1 provisioning and the `atlas.db` / Turso sync — after coverage is honest, not before.
