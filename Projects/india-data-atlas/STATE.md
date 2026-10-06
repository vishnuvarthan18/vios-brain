---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: India Data Atlas (India Data Platform)

Full details: [[Projects/india-data-atlas/SUMMARY]] · Log: [[Projects/india-data-atlas/LOG]] · Dev: [[Projects/india-data-atlas/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Active. Backend runs on its own on the [[Companies/OVHcloud]] VPS.
- All code in one private monorepo `vishnuvarthan18/india-platform` (local `~/india-monorepo`); 10 old repos kept for now.
- All 16 engine jobs run every 3 hours since 2026-10-02 (earlier: daily, Overpass weekly). Server load near zero.
- Collecting fine: Mountains & Geo, Water, Forests & Land, Tribal & Culture, Laws.
- Problems: Living Species runs show failed (~10 days, cause unknown); Protected Areas collected on only 2 of 14 days; Extinct Species never built.
- Data (2026-09-21): 20,552 entities, 68,597 facts, 14,572 taxa.
- New ops console (Next.js + shadcn/ui, black and white) live at `https://ops.vidivu.in` since 2026-10-01 (port 8020); old Python console on 8010 as fallback.
- CI/CD live on [[Tools/GitHub]] Actions: auto-deploy after checks, health check on exact commit, auto-rollback. Engines deploy the same way. (earlier: no CI/CD)
- Offsite backup: R2 bucket + credentials ready (2026-09-25); backup script not extended yet.
- No public API, no public website yet.

## Next steps
1. Fix Living Species failures and Protected Areas gaps.
2. Finish offsite backup to R2 (script, pruning, heartbeat, one test restore).
3. Watch disk use with 3-hourly jobs.
4. Write `RUNBOOK.md`; retire old repos and old console when the monorepo is proven.
5. Audit engine code for the D-71 `else 2` bug.
6. Extinct Species: build or drop.
7. GIS Phase 0: install [[Tools/QGIS]], sign up for [[Tools/Google Earth Engine]] and free data portals.

## Blockers
- On Vishnu: rotate data.gov.in key; register WDPA/IUCN/GeoNames/OpenTopography/GFW keys; OSM vs WDPA; India Code/Kanoon; bulk-dataset capacity call + India-egress retest.

## Key places
- Console: `https://ops.vidivu.in`
- VPS login user `ubuntu` (see SUMMARY key facts)
- Repo: `vishnuvarthan18/india-platform`; local `~/india-monorepo`
- Docs: `docs/` folder in this project (copies of the claude.ai project docs)
