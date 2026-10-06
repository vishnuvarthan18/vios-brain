# Web app — GIS first (decided 2026-09-21)

## Decisions
- **No public API for now.** The web app reads Postgres directly through a read-only role. The public API layer from full-platform-architecture-plan.md is postponed.
- **One web app, one place, all subjects.** Sections per subject (see academic-project-subjects.md).
- **GIS is the first focus.** The map comes first, and the other sections are layers or panels on it.
- Host it separately from the 4 GB VPS, and connect to the database through a secure tunnel.
- Keep the precision rules: sacred groves, traditional knowledge and FRA show at district level only.

## GIS phase 1 — layers from data already live
1. Protected areas: 602, of which 405 have real shapes (OSM)
2. Peaks, passes and ranges: 6,196
3. Water bodies: 624 (NWDP)
4. Forest layers: FSI district cover, mangroves, wetlands, desertification
5. District maps: ST population (585 districts), language counts
6. Species density per protected area (GBIF)

## GIS phase 2 — analysis
- Ecotourism suitability score (AHP weights over layers 1–6)
- Forest inside vs outside protected areas (buffer comparison)
- OSM vs WDPA boundary comparison (needs the WDPA key)

## Suggested stack (to confirm)
PostGIS → vector tile server (Martin or pg_tileserv) → MapLibre GL front end. Reuse the ops console's map and precision filter code where it fits.
