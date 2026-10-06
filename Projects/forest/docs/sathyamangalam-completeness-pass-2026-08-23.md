# Completeness Pass — 23 Aug 2026

**Headline: the "7 of 8 stages done and independently verified" status was wrong, and this pass is what found it.** The stages were verified for *correctness* — the code runs, writes accurately, dedupes properly, doesn't fabricate. They were never checked for *coverage*. Against their own master-prompt source lists, only Stage 5 is genuinely complete.

Database backed up first (blocking step): `db-backups/harvest-engine-d1-local-2026-08-23.sql`, 7.58 MB, restored into a fresh SQLite file and row-count-matched on all 13 tables. That risk is closed.

## Source coverage against the master prompt

| Stage | Named sources | Delivering data | Notes |
|---|---|---|---|
| 1 Biodiversity | 6 | 3 | GBIF 4,311 / iNat 881 / eBird 198. IBP, Xeno-canto, IUCN never built |
| 2 Literature | 9 | 7 | Shodhganga, BHL blocked (BHL_API_KEY absent) |
| 3 Gazetteer | 6 | 2 | Only Overpass (40) + Wikidata (18). WDPA, LGD, Census, Bhuvan all zero |
| 4 Government/legal | 9 | 1 | All 20 rows from NTCA alone. moef.gov.in/Parivesh has no registry stub at all |
| 5 Historical texts | 6 | 6 | **Complete.** All 6 volumes produced passages |
| 6 Remote sensing | 8 | 4 | Missing Landsat, ERA5-Land, IMD, SRTM/Copernicus DEM |
| 7 Media | 3 streams | 1 row | See below — architectural, not a bug |
| 8 Management plan | 29 appendices | 4 | 25 never scoped; 5 of 11 map pages undecoded |

Roughly 23 of 44 named sources across Stages 1–6 are producing data. `harvest-plan.md` estimated 62–68% coverage achievable through automation; the build is currently well under that.

## Three kinds of gap — they need different responses

**1. Blocked on a missing free credential.** `WDPA_API_KEY`, `DATA_GOV_IN_API_KEY` (LGD), `BHL_API_KEY`. Cheap to unblock, disproportionate payoff.

**2. Never built.** IUCN Red List (key *is* present in `.dev.vars` — the stream was simply never written), India Biodiversity Portal, Xeno-canto, Landsat, ERA5-Land, IMD, SRTM/DEM, moef/Parivesh, 25 management-plan appendices. Engineering time, no blockers.

**3. Structurally un-backfillable by the current architecture.** These cannot be fixed by running harder:

- **eBird** uses `obs/geo/recent` — a hard 30-day lookback. All 198 rows are a rolling snapshot, not historical coverage. Undisclosed in PROGRESS.md.
- **FIRMS** uses the NRT `area/csv` endpoint — 10-day rolling cap, no date parameter. Cannot serve 2021–2023. Historical fire needs the separate Archive Download product (free, but a different format and a new parser).
- **Stage 7 news** has no date-window parameter in any stream. RSS serves only what is currently in the feed. Stage 7 can only ever accumulate *forward from now*.

## 🔑 Highest-leverage single action: get `WDPA_API_KEY`

`reserve.boundary_geojson` is **NULL**. There is no reserve polygon in the database — only the bbox. Every geographic figure the Atlas has (NDVI, forest loss, and anything spatial that follows) is computed over a ~2,585 km² box containing farmland, Bhavanisagar reservoir and settlements, and none of it can be made reserve-scoped until a polygon exists.

The chain: WDPA key → extend `wdpa.js` to call the detail endpoint and store real geometry (currently it only stores a text note that a match was found) → change `gee-expression.js`'s `polygonGeometry()` to load that GeoJSON instead of building a rectangle from the four bbox corners. One free registration unblocks the single dependency that makes every spatial number publishable rather than approximate.

## ✅ Stage 6 AlphaEarth question — resolved, and it is *not* a defect

The master prompt's Stage 6 does not include disturbance or regrowth detection in any wording; the only hit for "disturbance/regrowth" is the "coordinate with, don't duplicate" line naming the separate senna project. So the five AlphaEarth decisions (64-band embeddings, 0.973 measured ceiling, 2022 cohort cutoff, 200m/10m dual resolution, 2017–2024 window) are **dormant guidance for future work, not unmet requirements**. Their absence is correct.

The earlier "Stage 6 is not done because AlphaEarth is missing" call was wrong — it came from treating `stage6-gee-senna-coordination.md` as a requirements spec when the master prompt is. Retracted.

Senna species discrimination confirmed **not shipped** (zero matches for "senna" anywhere in code, registries, or dashboard), which is correct — the coordination doc says it is non-functional and must not ship.

## ✅ The forest-loss pixel-area discrepancy — explained

The recompute returned `2,935,701.59 m²` via `pixelArea()` against `3,015,243.53 m²` from the flat ×900 multiply — **2.64% smaller**, when the earlier estimate predicted 3–4% *larger*. The agent correctly flagged the contradiction rather than reconciling it by assumption. The mechanism:

Implied mean pixel area = 2,935,701.59 ÷ 3350.2706 = **876.26 m²**.

`scale: 30` on an EPSG:4326 image makes Earth Engine convert 30 m to degrees at the **equator**: 30 ÷ 111,320 ≈ 0.00026949°. At 11.5°N those pixels measure roughly 29.81 m north–south (0.00026949 × 110,618 m/deg) by 29.40 m east–west (0.00026949 × 109,085 m/deg) = **876.4 m²** — matching the implied value to within 0.02%.

So the reduction never happened on Hansen's native 1-arc-second grid (~934 m² at this latitude) *or* on a true 30 m grid (900 m²). It happened on an equator-derived degree grid that is smaller than both. The earlier 934 m² estimate assumed native resolution and was wrong.

**`pixelArea()` is the correct figure: 2.936 km² (293.6 ha), not 3.02 km².** It is self-consistent — the pixel count and the pixel area come from the same grid. The flat multiply paired a count from an 876 m² grid with a 900 m² assumption.

The all-time figure (8.17 km²) needs the same recompute. The **36.9% ratio survives** unchanged if both numbers were computed the same way, since the error cancels.

## Defects and gaps found

1. **Stage 7 relevance-filter gap** — The Hindu TN's filter dropped two genuinely relevant stories from a 60-item feed ("Tiger responsible for two fatal attacks in Gudalur to be captured", "Three elephant deaths in 10 days in Nilgiris") because it lacks bare `tiger`/`elephant` terms. TOI's filter has them, which is the only reason the one stored row exists. Concrete and cheap to fix.
2. **`summary_25w` is NULL** on the sole news_event — traced to source, not a code bug (TOI's `<description>` is a thumbnail-link wrapper with empty CDATA; `truncateToWords()` correctly returns null). Still means the master prompt's ≤25-word summary requirement is unmet for that row.
3. **`claim` table: 0 rows** — designed-but-never-instructed. Zero mentions in the master prompt, no stream writes to it. Not a failed stage.
4. **`sources/gee.json` still stale** (carried over, unfixed) — describes the registration 403 as current.
5. **No structured scope metadata on `observation_layer` rows** (carried over, unfixed).
6. **Minor internal inconsistency in the report itself**: the defect list says "Stage 6: 3 of 6 named sources missing" while its own table shows four (Landsat, ERA5-Land, IMD, SRTM/DEM).

## Still unverified

- FIRMS NRT-vs-archive distinction stated from general API knowledge, not re-checked against NASA's live docs.
- Whether moef.gov.in/Parivesh was considered and dropped or never looked at — no trace either way.
- Whether Overpass/Census/Bhuvan failures are local-network-only or would also fail from a deployed Worker.
- The `year=1400` document row (Crossref/OpenAlex metadata artifact), still untraced.

## Revised next steps

1. **Register for `WDPA_API_KEY`** — highest leverage single action. Then `DATA_GOV_IN_API_KEY` and `BHL_API_KEY` while at it.
2. **Correct the project status** — replace "7 of 8 done and verified" with per-stage source coverage. Done in the handoff doc.
3. **Move cron automation up the priority list.** It was step 5. For Stage 7 and FIRMS it is not an optimisation — it is the *only* mechanism by which those stages ever accumulate data, since neither can look backward. Every day without it is a day of coverage permanently lost.
4. **Build the WDPA polygon chain**, then recompute NDVI and forest loss reserve-scoped.
5. Recompute the all-time forest-loss figure with `pixelArea()`.
6. Fix the Hindu TN filter terms.
7. Remote D1 provisioning and the `atlas.db` / Turso sync — after coverage is honest, not before.
