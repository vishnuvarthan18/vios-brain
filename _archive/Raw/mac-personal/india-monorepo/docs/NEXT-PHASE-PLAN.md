# Next Phase Plan — Deepen the 6 live engines

_Handoff written 2026-09-07. This run is deliberately scoped to work that does NOT depend on any of the five user-reserved items — the goal is to make real progress while those decisions are pending, not to sit idle. Execute in order, but skip ahead if a given item turns out to be blocked for a reason not listed below (log it and move on). Log every decision to `DECISIONS.md`, same convention as prior runs. Do not wait for confirmation — only pause for genuinely destructive/irreversible actions._

**Out of scope for this run — these are on the user, leave them alone:**
- Rotating `DATA_GOV_IN_API_KEY`
- Any new API key registrations (WDPA, IUCN v4, GeoNames, OpenTopography, GFW) — if a task below turns out to need one of these, stop that specific task, log it as blocked-on-user, and move to the next item
- The OSM-vs-WDPA geometry provider decision (stay on OSM)
- India Code / Indian Kanoon build-or-not (do not build against either; Laws & Management stays unstarted)
- Capacity call on the large bulk rasters/datasets (Copernicus DEM, HydroRIVERS/HydroLAKES, JRC GSW, ESA WorldCover full rasters) — metadata/inventory work on these is fine, bulk ingestion is not
- Re-testing India-WRIS/MoTA from an Indian egress point

## 1. Forests & Land — add the ecosystem types beyond forest

The original scope explicitly included deserts, grasslands, wetlands, mangroves, and coral/coastal-marine — only forest cover (FSI ISFR) and mangroves (Global Mangrove Watch) are built so far.

- Grasslands and coral reefs were flagged in the source atlas as having **no usable India source at all** — re-verify that's still true (a quick re-check, not a full re-survey) before permanently skipping them. If still true, leave as documented gaps, don't force it.
- Deserts and wetlands: check data.gov.in and any other already-approved keyless sources for usable datasets. Build what's reachable.

## 2. Mountains & Geography — close the named-peaks/passes gap

The source atlas flagged Wikidata and Overpass as re-testable (both came back reachable from the VPS in an earlier run — confirm still true) and as the real fix for the "named peaks/passes" gap beyond the 380 Wikidata peaks already ingested. Expand peak/pass/range coverage using these two sources if still reachable.

## 3. Tribal & Culture — add depth beyond language classification

Currently only Glottolog (language classification) is built. The original scope also asked for population numbers, livelihoods, and legal land rights (with the explicit exception: publish at district-aggregate precision only, never precise coordinates or individual/village-level data — this is already a schema requirement via `publish_precision`).

- Census 2011 is documented as the newest granular ST population data available — build against it for population figures, respecting the aggregation rule.
- Check for any keyless source on land rights / FRA claims at district-aggregate level (MoTA's FRA reports were flagged as blocked when tested from outside the VPS — retest from the VPS itself, separate from the India-egress question which is about India-WRIS/MoTA's original block, not a new one).

## 4. Water Systems — round out remaining source types

NWDP and the core datasets are done. Check HydroLAKES-derived lake polygons and JRC Global Surface Water's extent-history data (both documented as permissive-licensed and keyless) — these were named as build targets in the source atlas but not confirmed built yet. Springs/waterfalls/estuaries are a documented real gap (NWDP itself returns near-zero) — don't force it, just confirm still true.

## 5. Protected Areas — MoEFCC Elephant Reserves

The source atlas named the MoEFCC Elephant Reserves Atlas PDF as the only source for elephant reserves (not an IUCN category, so WDPA/OSM won't have it). If not yet built, this is a bounded, keyless addition.

## 6. Verification pass (last, every run)

- Zero heartbeat alerts, zero stale sources, zero empty-successful runs.
- All touched repos committed.
- Append a status block to `DECISIONS.md`: what got added per engine, what got confirmed-still-blocked and why, updated entity/fact counts.
