# Where the Atlas Actually Stands — 23 Aug 2026

Assessment of the whole project against `harvest-plan.md`'s own coverage math, not just the Harvest Engine build.

## Two numbers that must not be conflated

- **Source coverage: 25 of 44 = 56.8%.** How many named sources are wired up and delivering. This is what recent sessions have been tracking.
- **Atlas content coverage: roughly 35–40%** on the plan's own 15%→82% scale. This is what the site is actually worth to a reader.

These are not the same thing, and the second is the one that matters. **Reaching 70% source coverage does not produce 70% Atlas coverage** — a source counts as "delivering" whether it returns 40 rows or 4,000.

The ~35–40% figure is an estimate with wide error bars, derived by weighting each phase's planned coverage points by the fraction of its expected volume actually delivered. Treat it as directional, not precise.

## Phase-by-phase: planned volume vs actual

| Phase | Plan's expectation | Actual | Read |
|---|---|---|---|
| 1 Foundation | schema, provenance, orchestrator | ✅ built | Solid |
| 2 **Gazetteer** | **800–2,000 toponyms** | **58 places** | **~4% of target — the biggest gap in the project** |
| 3 Biodiversity | 20k–60k records, 1,500–3,000 taxa | 5,390 records, **1,654 taxa** | Records far below; **taxa inside the planned range** |
| 4 Literature | 400+ entries | 4,104 documents | **Exceeded, by an order of magnitude** |
| 5 Government/legal | 9 targets | 20 instruments, NTCA only | ~1 of 9 |
| 6 History mining | several thousand words | 1,182 passages, 6 volumes | Close to target |
| 7 Management plan | 29 appendices, +8 points | 4 appendices | ~1 of 8 points delivered |
| 8 Remote sensing | 9 layers | 6 of 8 named sources | Good, but FIRMS holds 0 detections |
| 9 Field/archive/relationship | "start day one" | outreach begun 18 Aug | Long lead times, cannot be compressed later |

## The three biggest gaps are not the ones we've been working on

### 1. The gazetteer is at ~4% and it is not blocked on anything

`harvest-plan.md` calls Stream 2 **"the highest-value stream in the entire plan and it is the easiest,"** worth the single largest jump in the phasing (15% → 34%, nineteen points). It expected 800–2,000 named toponyms.

There are **58 places**. Overpass returned 40; Wikidata 18.

The plan's own Overpass query asks for place nodes, natural peaks/springs/water, waterways, tracks and unclassified highways, places of worship, and protected-area relations across the full bbox. That query does not return 40 features for a 2,585 km² landscape — it returns hundreds to low thousands. **This is almost certainly an under-querying bug, not a data-availability limit and not a credential block.**

Fixing it is plausibly worth more Atlas coverage than all six remaining sources in the 70% push combined. It should be the next thing looked at, ahead of Landsat's successors.

Also unstarted from this stream: Census 2011 village directory join, LGD codes, the management plan's 40+ named streams and 25+ ponds reconciled against OSM names — which the plan notes is itself a publishable data-quality finding.

### 2. The `claim` table is empty, and it is the project's stated differentiator

`harvest-plan.md`, on the data model: *"`claim` is the important one... The site renders both and says so. **That single design decision is what makes this a reference work rather than another blog.**"*

`claim` has **0 rows**. The completeness pass confirmed no stream writes to it and the master prompt never mentions it. The schema exists; nothing populates it.

The plan names the disputes it was built for: 793.49 vs 917.27 km² core area, tiger counts across methods, fee schedules. Those contradictions are exactly what the Atlas is supposed to surface rather than silently resolve. Right now it has no mechanism to.

### 3. Management plan extraction: 4 of 29 appendices, worth 8 coverage points

The plan rates this the biggest single-document win in the project. Village and beat inventories, budget and staff tables, stream registers — 25 appendices unattempted. This is content the site cannot get any other way.

## What is genuinely strong

- **Provenance and correctness discipline.** Verified this session: idempotency, no `rows_written` overcounting recurrence across 18 call sites, honest zero-result reporting, self-corrected agent errors.
- **Sensitive-species coarsening is enforced in code, not policy** — exactly as the plan demanded. Confirmed: 4 sensitive occurrences, 0.045° grid, 5,000 m precision, zero leaks.
- **Literature over-delivered** — 4,104 documents against a 400-entry target.
- **Legal and ethical lines held** — Academia.edu downloaded manually as a human, robots.txt overrides documented per-source, ruled-out sources carry dated reasons.
- **Remote sensing metadata is now the best-documented part of the database** — `dataset_version`, `geometry_scope`, `reduction_scale`, `units`, `data_latency_days` on every row.

## The structural constraints that no amount of building removes

- **`reserve.boundary_geojson` is NULL.** Every spatial figure is a bbox average over terrain spanning 1,911 m of elevation and at least three ecological zones. The plan's own `STR_WKT` constant was meant to be "fetched once from Protected Planet" in Phase 1. It never was. Blocks publication of every geographic statistic.
- **Three sources can only look forward, never back** — eBird (30-day cap), FIRMS NRT (10-day cap), all Stage 7 RSS (no date parameter). Every day without a scheduler is coverage permanently lost.
- **Phase 9 has the longest lead times in the project** and the least ability to be accelerated later: Gass Forest Museum, Tamil Nadu Archives, the Field Director relationship, original photography. The plan says start day one for exactly this reason.

## Recommended reordering

1. **Diagnose the gazetteer.** 58 places against an 800–2,000 target, on the stream the plan calls easiest and highest-value. Nothing else has this ratio.
2. **`WDPA_API_KEY`** — unblocks the reserve polygon and every spatial figure.
3. **Turn on Cron.** Not an optimisation for the three forward-only sources; it is the only mechanism by which they ever accumulate.
4. Finish Landsat, then IUCN and Xeno-canto (no blockers).
5. **Implement `claim` writes** — the differentiator the plan is built around.
6. Scope the remaining 25 management-plan appendices.
7. Remote D1, then the `atlas.db` / Turso sync.

## The honest headline

**The engine is built, correct, and trustworthy. The tank is about a third full.** The engineering discipline in this project is genuinely high — the verification culture caught real bugs, retracted its own wrong theories, and refused to fabricate. That is rarer and harder than it looks.

What has not happened at the same standard is checking whether each stream actually *collected what it was built to collect*. Stage 3 is the clearest case: correct code, honest reporting, 4% of the intended data.
