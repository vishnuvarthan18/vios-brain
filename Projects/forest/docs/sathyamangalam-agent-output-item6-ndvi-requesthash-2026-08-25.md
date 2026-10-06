# Agent output — Item 6 pixelArea, NDVI annulus, request_hash audit (25 Aug 2026)

Raw output from the harvest-engine agent, logged verbatim per Vishnu's instruction — not yet reviewed or decided on. We are paused on making further decisions until the deep-research report is back. Do not act on this until Vishnu says so.

## Item 6 — pixelArea first, on bbox

Prediction held to 0.001 km²:
- MEASURED 7.9596 km² · predicted 7.959
- 3-year 2.9357 km² · was 3.015
- implied cell 876.30 m² all-time · 876.26 m² window

Same implied cell area in both windows, so −2.64% is a property of the grid, not of where loss happens. The 36.9% ratio survives: 36.88% by area, 36.88% by count — now by construction rather than cancellation, because forest-loss-all-time is a stored layer instead of a debug-call number.

Near-repeat of the layer's original defect, caught before landing: first denominator returned 9,101 for a 2,584 km² bbox. `count(lossyear)=9,101` vs `count(datamask)=2,949,208` — Hansen's lossyear band is masked where no loss occurred, so "cells with loss" had been mislabelled as "cells examined," off by 324×.

Geometry run as its own commit. Pre-registered below the 30.5% area share; measured 7.4%. The sanctuary holds 30.5% of the land and 7.4% of the recorded loss. Ratio moves with geometry: 36.88% bbox, 34.25% core.

## NDVI — the annulus confirmed it, the archive overturned it

Direct test worked: predicted 0.4245 / measured 0.4244, reconstruction exact to 2.6e-5. The archive test does **not** support the original hypothesis:

| year | core | annulus | gap |
|---|---|---|---|
| 2019–2021, 2024, 2025 | 0.44–0.55 | 0.42–0.46 | −0.006 to −0.115 (core greener) |
| 2026 | 0.3614 | 0.4130 | +0.0516 |

In every valid prior year the core is greener in this window. Original prediction was right for 2019–2025; 2026 is the reversal. (2022–23 excluded as instrument-invalid — QA60 entirely unpopulated those years, mask silently did nothing. 2026's QA60 is populated, checked first.)

Three explanations tested for 2026, none close it: window choice (no), stricter SCL masking (no — 0.3996 vs 0.56–0.63 elsewhere), rainfall (25% below 7-year mean, but 2020 was drier with the second-highest NDVI). Drop is largest at p95 (−0.203) — neither a disturbance nor typical drought signature.

**Coverage bug caught:** `nd_count = 264,567` had been in the record all along; divided by 905,866 cells the core contains = a mean over 29.2% of the sanctuary. 2025's was 82.4%. Within-2026 comparison is coverage-matched and stands; across-year comparison is not. Count rule now enforced in code (`layer-review.js` refuses to write a layer without both count and denominator) — which immediately surfaced that this reserve's published temperature is a mean over 21 ERA5 cells.

## request_hash — audit result

New rule: no separate list — the request body is every input, so `gee-request.js` returns body and hash together, can't disagree. Test perturbs all 366 constants in the graph rather than naming parameters. Source-level guard caught the Landsat loop still re-serializing.

**Audit finding: elevation is alone.** 21 runs wrote nothing while skipping; 19 reported success, all 19 genuine. 3 flagged by commit cross-reference resolve to a robots refactor and a counter fix. GEE values reproduce exactly against a live recompute today. Stated limit: hash can't cover parsers, so a filter fix doesn't propagate to an already-fetched source — no confirmed instance, none rule-out-able, because which row a skip matched was never recorded. **It is now recorded.**

`skipped_duplicate` is now its own status; implementing it surfaced two more instances of the same lying pattern: a stream returning no status was recorded as success even with errors, and a throw erased the counts — one run wrote seven layers and recorded `failed, rows_written=0`.

## Two things needing Vishnu's decision

1. **Production is not updated.** Migration 0004 applied there; 0005 is not — must apply before deploying or the annulus insert hits the 0003 trigger. Triggering the production run needs the dashboard password (agent doesn't have it). Everything here is verified against live Earth Engine and a full local run.

2. **Editorial call, agent steers away from the framing originally offered.** "The protected forest is drier than the irrigated farmland" is false for six of seven measured years. The agent's recommended clean, well-controlled result to publish instead: **30.5% of the land, 7.4% of the loss.** Whether the 2026 NDVI anomaly gets published as an open question, or gets its own dedicated investigation (agent thinks it deserves one — largest unexplained signal found) — undecided, Vishnu's call.

## Status
Logged only. Paused on deciding anything until the deep-research report on comparable projects comes back — avoiding decisions made in a rush.
