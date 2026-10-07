---
tags: project
---
# DEV-LOG: India Data Atlas / India Data Platform

Coding history from Claude Code sessions on the personal Mac. Started as the "Ecotourism Atlas" (protected areas only, folder ~/Ecotourism), grew into the India Data Platform. Main page: [[Projects/india-data-atlas/SUMMARY]]

## What was built
- **Goal:** an India-wide environmental and cultural data platform: 8 independent data "engines" feeding one shared core database through one validating core API. No launch deadline; build for months.
- **The 8 engines:** Protected Areas, Forests & Land, Mountains & Geography, Water Systems, Living Species, Extinct Species, Tribal & Culture, Laws & Management.

## Repos
- Monorepo (since 2026-09-29, private): **vishnuvarthan18/india-platform** (local ~/india-monorepo). Holds all 10 old repos with full history (86 commits).
- The 10 old repos (kept until monorepo proven; local ~/india-platform/):
  - india-data-platform — Protected Areas engine + public map site (was ~/Ecotourism)
  - india-data-core — shared database, API, storage
  - india-ops-console — private admin dashboard (Python)
  - india-forest-engine, india-geo-engine, india-water-engine, india-species-engine, india-extinct-engine, india-culture-engine, india-laws-engine
- Monorepo folders: `core/`, `protected-areas/`, `engines/` (culture, extinct, forest, geo, laws, species, water), `ops-console/`, `web/` (new console), `docs/` (PLAN, DECISIONS, NEXT-PHASE-PLAN, DEPLOY).

## Architecture
- **Server:** OVHcloud VPS-1, Ubuntu 26.04, 2 vCore / 4 GB RAM / 40 GB NVMe, about Rs 505 a month. Budget limit about Rs 1000 a month. Login user ubuntu.
- **Core (india-data-core):** Postgres 17 + PostGIS 3.5, MinIO for raw payloads, FastAPI core API. Engines never write to the DB directly; core API does auth, validation, dedup, licence checks. Data model: `entity` + `entity_fact` (one row per field, source, validity; conflicting values kept, best picked at read time), plus taxon/occurrence and timeseries tables. All bound to 127.0.0.1.
- **Harvest pattern:** Acquire, Parse, Load kept separate. Raw bytes archived to MinIO before parsing, so parsers can re-run without re-fetching. Cadence lives in the core registry (`schedule_tier`, `staleness_ceiling`).
- **Scheduling:** systemd timers on the VPS, never GitHub Actions cron. Since 2026-10-02 all 16 engine jobs run every 3 hours.
- **Ops console:** old Python console on port 8010; new console (`web/`, Next.js + shadcn/ui, black and white, dark) as container ops-web on port 8020. Public at https://ops.vidivu.in through nginx (Cloudflare cert, browser password + console login). Switched to the new console on 2026-10-01; backup of old nginx file kept on server.
- **CI/CD (GitHub Actions):** per-folder checks (web, ops-console, protected-areas, core and engines). Web: type-check, lint, build, privacy test on real PostGIS, Docker build; then auto-deploy over SSH with a health check on the exact commit and auto-rollback. Engines: auto-deploy after checks, builds image on server, import test, records a baseline (water first deployed 2026-10-01).
- **Privacy:** precision rules hide sensitive spots (e.g. sacred groves) on maps.
- **Geometry:** OpenStreetMap, not WDPA (WDPA needs a key and forbids redistribution).

## Timeline (newest first)
- 2026-10-06 — "Save all": everything already committed and pushed.
- 2026-10-05 — Simple mobile menu (plain "Menu" button) for the ops console.
- 2026-10-02 — All 16 engine timers switched to every 3 hours (via new systemd override files). Fixed Engines page counting for Laws and Living Species.
- 2026-10-01 — New Uptime page (engine health graphs). ops.vidivu.in switched from old console (8010) to new console (8020). Auto CI/CD for web and engines; water engine deployed by pipeline.
- 2026-09-29 — Merged 10 repos into monorepo vishnuvarthan18/india-platform. Cleaned docs into docs/. Built new ops console with Next.js + shadcn/ui (dark), with login, all 9 sections plus Atlas inventory tab. Black and white only, no "i" logo. Deployed to server on port 8020 via GitHub Actions + deploy key.
- 2026-09-07 — NEXT-PHASE runs: slug registry + migrate tool; PARIVESH fixed (37 records); manual harvest wrapper (bin/run-harvest.sh); POWO blocked by Cloudflare challenge, used Kew WCVP on ChecklistBank + IPNI instead (5,480 of 6,002 plant taxa enriched); new GET /v1/taxa. GBIF at 306/405 areas, 14,572 taxa.
- 2026-09-07 — Second run (no VPS access): wetlands + desertification harvesters, Wikidata passes (1,005) and ranges (324), Census 2011 ST population (585 districts), FRA titles J&K, MoEFCC Elephant Reserves (33). Not deployed then.
- 2026-09-06 — Overnight build (PLAN.md phases A to G): core stack live on VPS, 9 systemd timers. 8,255 entities, 44,129 facts, 4,892 taxa. 602 protected areas (405 with real polygons, was 0). Fixed upsert dedup bug, API key leaking into DB, ST_Envelope point bug.
- 2026-09-06 — Seed-list normalizer (normalize_seed_list.py) dry run: 645 records from NTCA (59) + WII (586); after fixes 607 reserves. State-name fixes (Chhattisgarh, Madhya Pradesh, Odisha). Seed harvest cron daily to monthly; 30-day cooldown per source.
- 2026-09-02 — Set up .env with eBird and data.gov.in keys (secret, not saved). Found site was never deployed: no hosting, Cloudflare D1/R2 never set up, normalizer missing, no reserves.geojson.

## Decisions
- 2026-09-06 — Move from Cloudflare D1/R2 plan to own VPS (OVH) with Postgres/PostGIS + MinIO; OVH chosen because Hetzner stock sold out and Hostinger needed 24-month prepay. #decision
- 2026-09-06 — Use systemd timers, not GitHub Actions cron. #decision
- 2026-09-06 — Use OSM geometry, not WDPA. Do not build against India Code / Indian Kanoon (they need a disguised client). #decision
- 2026-09-06 — Seed-list sources fetched monthly with 30-day cooldown. #decision
- 2026-09-29 — One monorepo instead of 10 separate repos (one place, one CI). Old repos kept until it works. #decision
- 2026-09-29 — JavaScript (Next.js + shadcn/ui) for the UI; keep Python for engines, core API and harvesting. #decision
- 2026-09-29 — Console design: black and white only, name only (no logo). #decision
- 2026-10-01 — Auto-deploy on push to main after checks pass, with auto-rollback. #decision
- 2026-10-01 — All engines run every 3 hours, all day. #decision

## State at last session (2026-10-06)
- ops.vidivu.in shows the new console, with Uptime page and real data.
- All 16 engine jobs run every 3 hours; server load near zero.
- Healthy: Mountains & Geo, Water, Forests & Land, Tribal & Culture, Laws (mostly).
- Not healthy: Living Species (runs marked failed ~10 days, cause not found), Protected Areas (data on only 2 of 14 days), Extinct Species (never built).
- All code pushed; nothing unsaved.

## Open items
- Find why Living Species runs fail; fix Protected Areas harvest.
- Decide: build or drop Extinct Species engine.
- Rotate DATA_GOV_IN_API_KEY (was stored in plaintext once; keys also pasted in chat).
- Register API keys: WDPA, IUCN v4, GeoNames, OpenTopography, GFW.
- Capacity call on big bulk datasets (Copernicus DEM, HydroRIVERS/LAKES, JRC GSW, ESA WorldCover).
- Retest India-WRIS and MoTA from an Indian network.
- Archive the 10 old repos once the monorepo is proven.
- Public map site still has no reserves.geojson locally (export needs VPS DB).
- Contact pop-up request: unclear which site (not done).

## Session index
- personal-mac__india-data-platform__2026-10-01_dc2343c6 (archived: Projects/india-data-atlas/claude-code/personal-mac__india-data-platform__2026-10-01_dc2343c6.md) — 2026-10-01 to 10-06 — Uptime page, 3-hour schedules, Engines counts, mobile menu, save all
- personal-mac__india-data-platform__2026-09-29_c34327dc (archived: Projects/india-data-atlas/claude-code/personal-mac__india-data-platform__2026-09-29_c34327dc.md) — 2026-09-29 to 10-01 — monorepo, new shadcn console, CI/CD, deploy, ops.vidivu.in switch
- personal-mac__Ecotourism__2026-09-07_2f09bcea (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-07_2f09bcea.md) — 2026-09-07 — next-phase steps 1 to 6 without VPS access
- personal-mac__Ecotourism__2026-09-07_5cefb471 (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-07_5cefb471.md) — 2026-09-07 — GBIF check, POWO/WCVP enrichment, disk usage
- personal-mac__Ecotourism__2026-09-07_405ea857 (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-07_405ea857.md) — 2026-09-07 — slug migration, PARIVESH fix, harvest wrapper
- personal-mac__Ecotourism__2026-09-06_af1630bb (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-06_af1630bb.md) — 2026-09-06 — overnight build phases A to G on VPS
- personal-mac__Ecotourism__2026-09-06_fd56665b (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-06_fd56665b.md) — 2026-09-06 — seed-list dry run, normalizer fixes, state names, cooldown
- personal-mac__Ecotourism__2026-09-02_44017208 (archived: Projects/india-data-atlas/claude-code/personal-mac__Ecotourism__2026-09-02_44017208.md) — 2026-09-02 — .env keys, why site is not live, file paths
