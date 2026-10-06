# Path to 70% Source Coverage — plan and progress

**Start: 23 of 44 named sources across Stages 1–6 = 52.3%.**
**Now: 25 of 44 = 56.8%** (SRTM/DEM `job_run` 82, ERA5-Land `job_run` 86).
**Target: 31 of 44 = 70.5%. Remaining: +6.**

## What "counts" — fixed rule, so the number is not gameable

A source counts only if it (a) delivers a non-trivial row count appropriate to what that source actually holds for this reserve, (b) has a registry file in `sources/` documenting its query and limits, and (c) has a successful `job_run`. A genuinely-zero result from a real, correctly-queried source counts as covered and must be documented as such.

## Progress

| # | Source | Tier | Status |
|---|---|---|---|
| 1 | SRTM / Copernicus DEM | GEE | ✅ **Done** — `USGS/SRTMGL1_003`, `job_run` 82–83 |
| 2 | ERA5-Land temperature | GEE | ✅ **Done** — `ECMWF/ERA5_LAND/DAILY_AGGR`, `job_run` 86–87 |
| 3 | Landsat composites | GEE | Next — the last GEE layer in this push |
| 4 | WDPA | Credential | **Blocking** — awaiting key; unblocks the reserve polygon |
| 5 | BHL | Credential | Awaiting key; stream exists, `job_run` failed on missing key |
| 6 | LGD | Credential | Needs key + `resource_id` placeholder replaced |
| 7 | IUCN Red List | New stream | `IUCN_API_KEY` already in `.dev.vars`; stream never written |
| 8 | Xeno-canto | New stream | Open API, no key — least effort on the list |
| 9 | India Biodiversity Portal | New stream | Verify a usable API exists before committing effort |

Stage 6 is now 6 of 8 named sources. Only Landsat and IMD remain, and IMD is deliberately excluded.

## Layer results

**SRTM/DEM** — elevation 187–2,098 m, mean 787 m; slope 0–79.57°, mean 11.15°. Established the metadata pattern (`dataset_version`, `geometry_scope`, `reduction_scale`, `units`), backfilled onto all four pre-existing rows. `sources/gee.json` stale-403 language corrected.

**ERA5-Land** — mean 23.92 °C, min 20.01, max 29.26 over 2026-08-08→15, `image_count` 7, native scale 11,132 m. Measured latency: DAILY_AGGR 8 days, HOURLY 6–7 days; chose DAILY_AGGR since it ships pre-aggregated daily min/mean/max. Added `data_latency_days` as a fifth metadata field and backfilled `22` onto the CHIRPS rainfall row — the hard-won latency finding is now queryable rather than prose, which is what lets a scheduled run distinguish expected lag from failure.

## Cross-layer validation: the two new layers corroborate each other

The 1915 Coimbatore gazetteer (`historical_passage` id 1128) gives 77.6 °F = 25.33 °C annual mean. Live ERA5-Land bbox mean is 23.92 °C — 1.4 °C cooler.

That gap is *expected*, and the SRTM layer explains it. Coimbatore station sits near ~411 m; the bbox mean elevation is 787 m. At a standard ~6.5 °C/km lapse rate, 376 m of additional elevation predicts ~2.4 °C of cooling, giving ~22.9 °C. Observed 23.92 °C lands within about 1 °C of that.

So the temperature layer is consistent with the elevation layer and with a 111-year-old independent measurement, once elevation is accounted for. This is the first genuine cross-layer validation in the project and the pattern worth repeating: use one layer to predict what another should show, then check.

## The bbox problem keeps compounding

The elevation range (1,911 m of relief, topping 2,000 m) means the bbox reaches from the Bhavanisagar plain into montane plateau — at least three ecological zones, including ground that is not dry deciduous forest at all. Every bbox-scoped figure inherits this:

- **NDVI 0.407** blends vegetation types with genuinely different ceilings.
- **Forest loss** counts stand-replacement across zones with different drivers.
- **Temperature 23.92 °C** averages across roughly 12–13 °C of lapse-rate spread — and does so over only ~21 ERA5-Land grid cells, since 0.1° pixels barely resolve a 0.63° × 0.34° box.

`WDPA_API_KEY` is therefore **blocking publication of any spatial figure**, not merely high-leverage.

## Open defects

1. **Reconcile the two ERA5 means.** The report quotes a live raw probe of 297.43 K and a stored 23.92 °C. 297.43 − 273.15 = 24.28 °C, not 23.92; the stored value implies a raw of ~297.07 K. Almost certainly two different query windows, but that is unstated and must be confirmed — the alternative is a subtly wrong conversion.
2. **Confirm `image_count` 7 over an 8-day window** is `filterDate` end-exclusivity, not a missing day. Document it either way, or a future scheduled run will read it as a gap.
3. **`slope_max = 79.57°` is not publishable as-is** — ~163 m rise over a 30 m pixel. Store a 95th-percentile slope alongside; DEM maxima are artifact-dominated.
4. **`historical_passage` may be truncating mid-content.** Passage id 39 was cut mid-number and had to be recovered from raw R2. Audit the truncation rule across all 1,182 rows — it may be clipping exactly the figures that make passages citable.
5. Carried over, unfixed: `~21 grid cells` limitation should be in the registry alongside the elevation-averaging note.

## Deliberately not in the 70% push

IMD (no clean public API, CHIRPS already covers rainfall) · moef.gov.in/Parivesh (scraping, no API, no registry stub — valuable but slow, do after 70%) · Census and Bhuvan (failed on unreachable hosts; retry from a deployed Worker first) · the 25 unextracted management-plan appendices (real content value, but not "named sources" — track separately).

## Honest note

Stage 4 sits at 1 of 9 and is the substantively weakest area for a conservation atlas, but every remaining target there is scraping with no API. Reaching 70% via GEE layers and species APIs is legitimate — and it still leaves the legal and government record thin. The number should not substitute for that judgement.
