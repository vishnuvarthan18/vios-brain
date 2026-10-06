# Harvest Engine — Session Handoff (updated 23 Aug 2026, evening)

## ⚠️ Status corrected: the "7 of 8 stages done and verified" claim was wrong

The stages were verified for **correctness** — code runs, writes accurately, dedupes, doesn't fabricate. They were never checked for **coverage**. The completeness pass on 23 Aug found that against their own master-prompt source lists, **only Stage 5 is genuinely complete**. Roughly 23 of 44 named sources across Stages 1–6 are producing data.

Full detail: `sathyamangalam/completeness-pass-2026-08-23.md`.
Integrity findings: `sathyamangalam/cross-stage-verification-2026-08-23.md`.

Working copy: `sathyamangalam-atlas-clean`. Remote: `https://github.com/vishnuvarthan18/sathyamangalam-atlas.git`.

## Stage ledger — correctness vs coverage

| Stage | Code correct | Sources delivering | Status |
|---|---|---|---|
| 0 Foundation | ✅ `e152604` | n/a | Done |
| 1 Biodiversity | ✅ `cd3ad2b` | 3 of 6 | Partial — IBP, Xeno-canto, IUCN never built |
| 2 Literature | ✅ | 7 of 9 | Partial — Shodhganga, BHL blocked |
| 3 Gazetteer | ✅ | 2 of 6 | Partial — WDPA, LGD, Census, Bhuvan all zero |
| 4 Government/legal | ✅ | 1 of 9 | Partial — NTCA only; moef/Parivesh has no stub |
| 5 Historical texts | ✅ | 6 of 6 | **Complete** |
| 6 Remote sensing | ✅ `9d46872` | 4 of 8 | Partial — Landsat, ERA5-Land, IMD, SRTM/DEM missing |
| 7 Media | ✅ | 1 news_event | Structurally limited — RSS only, no backfill possible |
| 8 Management plan | ✅ `85307bc` | 4 of 29 appendices | Partial |

Correctness verification stands and is not in question: idempotency confirmed live (`job_run` 81: 0 written, 2 skipped), the `rows_written` overcounting bug has not recurred at any of 18 call sites, sensitive-species coarsening is correctly enforced with zero leaks, and Stage 1's occurrence total reconciles exactly (4,311 + 881 + 198 = 5,390).

## Database backup — risk closed

All data previously existed only in one unbacked-up local miniflare file (`wrangler.toml` still has `database_id = "REPLACE_WITH_D1_DATABASE_ID"`; there is no remote D1). Now dumped to `db-backups/harvest-engine-d1-local-2026-08-23.sql`, 7.58 MB, restore-verified with row counts matched on all 13 tables.

## The single highest-leverage action: get `WDPA_API_KEY`

`reserve.boundary_geojson` is **NULL** — there is no reserve polygon, only a bbox. Every spatial figure in the Atlas is computed over a ~2,585 km² box including farmland, Bhavanisagar reservoir and settlements, and none can be made reserve-scoped until the polygon exists. One free registration unblocks the chain: key → extend `wdpa.js` to fetch real geometry (it currently stores only a text note) → point `gee-expression.js`'s `polygonGeometry()` at it.

## Stage 6 — closed correctly, with two corrections to earlier claims

- **AlphaEarth is not a missing requirement.** The master prompt's Stage 6 contains no disturbance/regrowth deliverable in any wording. The five carried-over senna decisions are dormant guidance for future work. The earlier "Stage 6 is not done because AlphaEarth is missing" call treated a coordination doc as a requirements spec and was wrong. Retracted. Senna species discrimination confirmed not shipped, which is correct.
- **Forest loss is 2.936 km² (293.6 ha), not 3.02 km².** `pixelArea()` recompute returned 2,935,701.59 m². Implied mean pixel area is 876.26 m², because `scale: 30` on an EPSG:4326 image makes Earth Engine derive a degree grid at the equator (30 ÷ 111,320 ≈ 0.00026949°), which at 11.5°N measures ~29.81 × 29.40 m = 876.4 m² — matching to within 0.02%. Neither 900 m² nor the native 1-arc-second ~934 m² applies. The all-time 8.17 km² figure needs the same recompute; the 36.9% ratio survives unchanged.

NDVI (0.407) and the CHIRPS 8-of-31 latency finding both stand as previously resolved.

## Three kinds of gap, needing different responses

1. **Blocked on a free credential** — `WDPA_API_KEY`, `DATA_GOV_IN_API_KEY`, `BHL_API_KEY`.
2. **Never built** — IUCN (key *is* present; stream was never written), India Biodiversity Portal, Xeno-canto, Landsat, ERA5-Land, IMD, SRTM/DEM, moef/Parivesh, 25 management-plan appendices.
3. **Structurally un-backfillable** — eBird `obs/geo/recent` has a hard 30-day lookback (all 198 rows are a rolling snapshot, undisclosed); FIRMS NRT `area/csv` caps at 10 days and cannot serve 2021–2023 (needs the separate Archive Download product); Stage 7's RSS streams have no date parameter at all and can only accumulate forward from now.

## Revised next steps

1. **Register for `WDPA_API_KEY`**, plus the other two free keys.
2. **Move cron automation up.** It was step 5. For Stage 7 and FIRMS it is not an optimisation — it is the only mechanism by which those stages ever accumulate data, since neither can look backward. Every day without it is coverage permanently lost.
3. Build the WDPA polygon chain, then recompute NDVI and forest loss reserve-scoped.
4. Recompute the all-time forest-loss figure with `pixelArea()`.
5. Fix The Hindu TN's relevance filter (missing bare `tiger`/`elephant` terms — it dropped two genuinely relevant stories from a 60-item feed).
6. Fix carried-over defects: stale `sources/gee.json`, missing structured scope metadata on `observation_layer`.
7. Remote D1 provisioning and the `atlas.db` / Turso sync — after coverage is honest, not before.
8. Unautomatable regardless: physical archive visits, the STR Field Director relationship, original photography.

## Vishnu's stated bar: 99.99% success, zero bugs

The right target remains "no bug ships unverified." That standard held — but this pass shows it needs a second half: **no stage is called done without checking what it actually collected, not only whether it collected correctly.** A pipeline can be flawless and still be empty.

## Don't re-litigate

Architecture (100% Cloudflare), naming, build order, working-directory fix (`-clean` not `-main`), and the correctness verification of all stages — settled. What is open is coverage, not correctness.
