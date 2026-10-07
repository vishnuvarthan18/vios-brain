---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: India Data Atlas (India Data Platform, first named "Ecotourism Atlas")

## 1. What this project is
- **Goal:** A citation-backed data platform for India's nature and culture: protected areas, forests, mountains, water, species (living and extinct), tribal communities and languages, and laws. Every fact keeps its source, date, licence and confidence.
- **Why:** [[People/Vishnu]] loves forests, animals, tribal culture and mountains. No money plan yet. Long-term project, no fixed "done" line.
- **History:** Started (Aug 2026) as "Ecotourism Atlas" — a wiki of all Indian protected areas. Not a booking or travel-guide site. Scope then grew into an India-wide data platform.
- **Early design (Aug 2026, "Ecotourism Atlas"):** cited data atlas for all Indian protected areas (national parks, sanctuaries, tiger/biosphere/conservation/community reserves). [[Tools/Scrapy]] harvest engine (one spider per source, YAML descriptors) on [[Tools/GitHub]] Actions; raw files to [[Tools/Cloudflare]] R2, facts to Cloudflare D1 with per-fact citations. Own repo, own Cloudflare account, own D1 database — kept fully separate from the sister Sathyamangalam atlas ([[Projects/forest/SUMMARY]]). Never deployed; replaced by the OVH VPS stack in Sep 2026. Google export also lists an "ecotourism repo" next to sathyamangalam.online (probably this, not sure).
- **Sister projects (kept separate on purpose):** [[Projects/forest/SUMMARY]] (one tiger reserve, on [[Tools/Cloudflare]]) and [[Projects/semmozhi/SUMMARY]] ([[Tools/Scrapy]] + [[Tools/GitHub]] Actions).
- **How it works (6 layers):**
  1. Database — [[Tools/PostgreSQL]] + PostGIS. The single source of truth.
  2. Engines — 8 separate data collectors (harvest → normalize → load). One repo each.
  3. Core API — the only writer to the database. Checks source, licence, dedup and `publish_precision`.
  4. Public API — postponed (not built).
  5. Public website / web app — not built. Plan: a GIS map first.
  6. Admin ops console — built and live at `https://ops.vidivu.in` (new Next.js console since 2026-10-01).
- **8 engines:** Living Species, Mountains & Geography, Tribal & Culture, Water Systems, Protected Areas, Forests & Land, Laws & Management, Extinct Species.
- **Sensitive data rule:** sacred groves, traditional knowledge and FRA claims are only stored and shown at district level. A database trigger enforces it; the console also hides exact points on the way out.
- **Server:** [[Companies/OVHcloud]] VPS (US branch, Oregon), 2 vCore / 4 GB RAM / 40 GB NVMe, about $6.31/month. Jobs run by [[Tools/systemd]] timers. Services run in [[Tools/Docker]]. Budget limit about Rs 1000 a month.

## 2. Status now (as of 2026-10-06)
- **Active.** Last work: 2026-10-06 (all code committed and pushed). Last chat: 2026-10-02 (hosting comparison).
- **Code:** all in one private monorepo `vishnuvarthan18/india-platform` (local `~/india-monorepo`) since 2026-09-29. The 10 old repos are kept for now.
- **Backend runs on its own.** All 16 engine jobs run **every 3 hours** since 2026-10-02 (earlier: daily, Overpass weekly). Server load near zero.
- **Data (2026-09-21):** 20,552 entities, 68,597 facts, 14,572 taxa, 11,478 raw archive files, 118 harvest runs, ~1.4% rejected rows.
- **Engines:**
  - Collecting fine: Mountains & Geography (6,196 entities), Water Systems (624, NWDP), Forests & Land (FSI, mangroves, wetlands, desertification, WorldCover), Tribal & Culture (830 languages + Census ST population for 585 districts), Laws & Management (e-Gazette job only).
  - Living Species — 14,572 [[Companies/GBIF]] taxa + 5,480 plant taxa from [[Companies/Kew]] WCVP/IPNI; but runs show **failed** for ~10 days (cause unknown).
  - Protected Areas — 602 entities, 405 with real shapes ([[Tools/OpenStreetMap]]); data collected on only 2 of 14 days.
  - Extinct Species — never built (repo exists). Build or drop is Vishnu's call.
- **Ops console:** new console (Next.js + shadcn/ui, black and white) live at `https://ops.vidivu.in` since 2026-10-01 (container ops-web, port 8020), with an Uptime page and simple mobile menu. Old Python console still runs on 8010 as fallback. Behind [[Tools/Cloudflare]] + [[Tools/nginx]] Basic Auth + own login.
- **CI/CD: live** on [[Tools/GitHub]] Actions since 2026-10-01 — checks, auto-deploy over SSH, health check on the exact commit, auto-rollback. Engines deploy the same way. (earlier: none, manual SSH deploys)
- **Offsite backup:** [[Tools/Cloudflare]] R2 bucket and credentials ready and tested (2026-09-25). Backup script not yet extended (not sure if done since).
- **GIS learning + web map:** planned (zero spend, 6 phases). Not started as far as notes show (not sure).
- **Blocked on Vishnu:** rotate data.gov.in key; register WDPA/IUCN/GeoNames/OpenTopography/GFW keys; OSM vs WDPA; India Code/Kanoon call; bulk-dataset capacity call + India-egress retest.

## 3. Next steps
1. Find why Living Species runs fail and why Protected Areas collects so rarely.
2. Finish offsite backup: extend `pg_backup.sh` to push DB dumps + raw archive to R2, add pruning, report to heartbeat, do one test restore.
3. Watch disk use with jobs every 3 hours (monthly sources now re-download often).
4. Write `RUNBOOK.md` (rollback is now automatic in CI).
5. Retire the 10 old repos and the old Python console once the monorepo is proven.
6. Audit the engine code for the D-71 `else 2` exit-code bug.
7. Vishnu's own items: register WDPA, IUCN v4, GeoNames, OpenTopography, GFW keys; rotate `DATA_GOV_IN_API_KEY`; decide OSM vs WDPA; decide India Code / Indian Kanoon; capacity call + India-egress retest.
8. Decide build or drop for Extinct Species.
9. Start GIS Phase 0: install [[Tools/QGIS]], sign up for [[Tools/Google Earth Engine]], NASA Earthdata, Copernicus, Bhoonidhi.
10. Optional: outside uptime monitoring (e.g. Uptime Kuma on [[Companies/PikaPods]]) — suggested, not decided.

## 4. Decisions
- 2026-08-26 (not sure exact day) — Start a 3rd project "Ecotourism" as a separate repo; do NOT merge with [[Projects/forest/SUMMARY]] — keep projects independent. #decision
- 2026-08-26 (not sure) — Data only, wiki style. No booking, no hotels, no travel guide for now — tourism is a later goal. #decision
- 2026-08-26 (not sure) — Build with [[Tools/Scrapy]] (Python), not [[Tools/Cloudflare]] Workers/TypeScript — Vishnu must be able to read and trust the code. #decision
- 2026-08-26 (not sure) — No AI in data collection, cleaning or checking — cost, and only inspectable code is trusted. #decision
- 2026-08-26 (not sure) — Every fact must have source_url, retrieved_at, licence, confidence. #decision
- 2026-08-26 (not sure) — Collect images, but only CC0 / CC-BY / CC-BY-SA, one per occurrence, small size — save storage. #decision
- 2026-08-26 (not sure) — Local names need a language code; "unknown" if not known, never guessed. #decision
- 2026-08-27 — Threats branch: only usable source is NTCA tiger mortality (tiger reserves only); encroachment source left open — little structured data exists. #decision
- 2026-09-05 — eBird and data.gov.in API keys added as [[Tools/GitHub]] Actions secrets and a local `.env` (never committed). #decision
- 2026-09-07 (not sure) — Scope grows to India-wide platform with 8 engines, one core API, one shared [[Tools/PostgreSQL]] + PostGIS DB on an [[Companies/OVHcloud]] VPS. Engines never write to the DB directly. #decision
- 2026-09-06 — Move from Cloudflare D1/R2 plan to own VPS ([[Companies/OVHcloud]]) with Postgres/PostGIS + MinIO; Hetzner was sold out, Hostinger needed 24-month prepay. #decision
- 2026-09-06 — Seed-list sources fetched monthly with a 30-day cooldown per source. #decision
- 2026-09-07 (not sure) — Schedule with [[Tools/systemd]] timers on the VPS, not [[Tools/GitHub]] Actions — Actions drifts and auto-disables after 60 days. No DAG framework. #decision
- 2026-09-07 (not sure) — Fetch → parse → load, raw archive first (raw bytes to [[Tools/MinIO]] before parsing). #decision
- 2026-09-07 (not sure) — Postgres and MinIO bound to 127.0.0.1 only (D-2) — Docker rules bypass ufw. #decision
- 2026-09-07 (not sure) — Sacred groves, traditional knowledge, FRA claims capped at district precision by a DB trigger. #decision
- 2026-09-07 (not sure) — Use [[Companies/Kew]] WCVP via ChecklistBank + IPNI instead of POWO site (POWO blocked by bot check). #decision
- 2026-09-07 (not sure) — Respect deliberate blocks (PBDB robots.txt, Supreme Court/NGT CAPTCHA, TKDL) — do not route around. #decision
- 2026-09-07 — Next 3 months = backend only, full data build on all 8 engines, no website or money work yet — Vishnu has no end-use plan; motive is love for forests, animals, tribes, mountains. #decision
- 2026-09-07 — Stay on [[Tools/OpenStreetMap]] for PA shapes; WDPA path stays dormant; do not build against India Code / Indian Kanoon (until Vishnu decides). #decision
- 2026-09-08 — Main repo renamed `ecotourism` → `india-data-platform`. All 9 repos pushed to [[Tools/GitHub]]. GitHub Organization move deferred (cosmetic). #decision
- 2026-09-08 — Deploy by running [[Tools/Claude Code]] directly on the VPS — avoids sandbox network limits. #decision
- 2026-09-08 — Fix Census TLS by installing the missing CA cert, never by turning verification off. #decision
- 2026-09-08 — Add a separate Public API layer (versioned, rate-limited, precision-aware) to the plan; host the public site away from the 4 GB VPS. #decision
- 2026-09-16 — Test monthly jobs by starting them by hand (`systemctl start`) instead of waiting weeks — harvests are idempotent. #decision
- 2026-09-16 — D-71: a "partial" run exits 0, not 2 — FRA J&K source table has a permanent flaw; the signal stays in the run record. #decision
- 2026-09-16 — D-70: staleness window comes from each job's cadence, not a flat 36 hours — two-thirds of jobs were always "stale". #decision
- 2026-09-16 — Ops console: [[Tools/FastAPI]] + Jinja, hand-written CSS, vendored [[Tools/Leaflet]], no framework, no CDN — must work with no outbound network. #decision
- 2026-09-16 — Console reads Postgres directly with a restricted read-only role, not via core API. #decision
- 2026-09-16 — Console hides exact points of restricted data on map and CSV too — an export leaves the platform once forwarded. #decision
- 2026-09-16 — Alerts stored as episodes (opened/resolved); notify only on change. #decision
- 2026-09-16 — Actions go through a separate host service (`ops-control`) with a root-owned allowlist; no shell in the web app. #decision
- 2026-09-17 — "Colour means something": grey UI, red/amber/green only for health facts. #decision
- 2026-09-17 — Login lockout: 5 fails = 5 min lock, even with correct password; no proxy headers trusted. #decision
- 2026-09-17 — Key rotation refuses line breaks (fixed a real privilege escalation). #decision
- 2026-09-17 — All 10 repos moved into `~/india-platform/`; console lifted out of `india-data-platform`. #decision
- 2026-09-17 — Rebase, never force-push; always `git fetch` before local work — the VPS also pushes to the same branches. #decision
- 2026-09-17 — Console gets its own [[Tools/GitHub]] repo, branch `main`. #decision
- 2026-09-21 — Console made public at `ops.vidivu.in` via [[Tools/Cloudflare]] (Full Strict SSL, origin cert) + [[Tools/nginx]] Basic Auth as a 2nd login layer. Changes the earlier "SSH tunnel only" plan. #decision
- 2026-09-21 — Console runs read-only first (`CONTROL_TOKEN` empty) — safer first deploy. #decision
- 2026-09-21 — Do not retry geo-peaks a 3rd time back-to-back — avoid hammering Wikidata. #decision
- 2026-09-21 — No public API for now; web app reads Postgres via a read-only role, hosted apart from the VPS via secure tunnel. #decision
- 2026-09-21 — Web app is GIS-first: map first, other subjects as layers/panels. Suggested stack: PostGIS → Martin/pg_tileserv → [[Tools/MapLibre]] (to confirm). #decision
- 2026-09-21 — Best academic angle: Geoinformatics. Flagship project: India Protected Area Health Index (one score per PA from 8 satellite signals). #decision
- 2026-09-22 — Zero spend: only free courses, tools and data. Learn by doing on our own parks. 6-phase GIS plan. #decision
- 2026-09-22 — Space engine stores only per-park results, never raw imagery, on the VPS. #decision
- 2026-09-23 — (later replaced by every 3 hours) All engine jobs daily (Vishnu's choice), except Overpass grid walk stays weekly — public server throttles (504/429). #decision
- 2026-09-23 — geo-peaks: retry with backoff (30/60/120s on 429/5xx/timeouts) + reordered SPARQL query. #decision
- 2026-09-25 — Offsite backup target = [[Tools/Cloudflare]] R2 (not Google Drive) — free 10 GB, no egress fees, no token expiry. #decision
- 2026-09-25 — CI uses a dedicated deploy-only Linux user (not built yet). CI is for build/deploy only; harvest schedule stays on systemd. #decision
- 2026-09-25 — Work order: offsite backup → CI/CD → rollback runbook → cleanup. #decision
- 2026-09-25 — R2 access key rolled after an Access Key ID was pasted in chat; all secrets now typed only at the VPS terminal. #decision
- 2026-09-29 — One monorepo instead of 10 repos (one place, one CI); old repos kept until it works. #decision
- 2026-09-29 — JavaScript (Next.js + shadcn/ui) for the UI; Python stays for engines, core API and harvesting. Console design: black and white, name only (no logo). #decision
- 2026-10-01 — Auto-deploy on push to main after checks pass, with auto-rollback. #decision
- 2026-10-01 — All engines run every 3 hours, all day (replaces the 2026-09-23 daily choice). #decision
- 2026-10-02 — Stay on [[Companies/OVHcloud]], not [[Companies/PikaPods]] — PikaPods can't run custom code, systemd timers or SSH. (Claude's advice; Vishnu did not reply — not sure if agreed.) #decision

## 5. Timeline
- 2026-08-26 (not sure) — Project started as "Ecotourism Atlas". Data hierarchy defined (identity, zones, hydrology, flora, fauna, threats, people/tribe, corridors).
- 2026-08-27 — Flora/fauna normalization done and verified (GBIF, eBird, iNaturalist) on 3 test reserves: Sathyamangalam, Bandipur, Mudumalai.
- 2026-08-28 — First scaffold pushed: 20-table D1 schema, 3 Scrapy spiders, NTCA (58 tiger reserves), WII gazette, 353 water bodies via Overpass. Two silent bugs found and fixed. First GitHub Actions run failed (exit code 1, cause unknown).
- 2026-09-02 — Found the Ecotourism site was never deployed (no hosting; D1/R2 never set up). eBird + data.gov.in keys set in `.env`.
- 2026-09-05 — eBird + data.gov.in keys working. Branch 1 test run started on live GBIF.
- 2026-09-06 — Overnight build: core stack live on the VPS, 9 timers; 8,255 entities, 44,129 facts; 602 protected areas (405 with polygons). Seed-list normalizer: 607 reserves from NTCA + WII.
- 2026-09-07 — POWO/WCVP plant enrichment (5,480 taxa); PARIVESH fixed; slug migration tool; new harvesters (wetlands, passes, Census ST, Elephant Reserves). Vishnu says he has no end-use plan; decides 3 months of backend-only build. Next-phase plan written.
- 2026-09-08 — SSH "problem" solved (login is `ubuntu`, not root). Found 9 repos, all pushed to GitHub. Big deploy: wetlands, desertification, passes/ranges, Overpass peaks, Census ST, FRA J&K, Elephant Reserves. 6 new timers. Full architecture plan written.
- 2026-09-09 to 09-15 — Observation pause; jobs fire on their own.
- 2026-09-16 — Pause closed, all 16 jobs verified. D-71 fixed. Ops console built (9 sections, 56 files). D-70 fixed in migration 0019.
- 2026-09-17 — Console redesigned, 184 tests, contrast audit (121 → 0 failures), security hardening, privilege-escalation bug fixed. Repos moved into `~/india-platform/`.
- 2026-09-20/21 — geo-peaks fails twice (Wikidata 500 + timeout).
- 2026-09-21 — Console pushed to GitHub and deployed live at `ops.vidivu.in`. Migrations 0019–0021 applied. Data snapshot: 20,552 entities. Use cases, academic subjects, GIS-first web app and space-data flagship researched.
- 2026-09-22 — GIS master plan (zero spend, 6 phases).
- 2026-09-23 — All jobs switched to daily (except Overpass). geo-peaks retry fix works (454 peaks). Migration 0022 applied.
- 2026-09-25 — Repo audit: no secrets leaked; backup not offsite; no CI/CD. R2 bucket + credentials set up and tested.
- 2026-09-29 — 10 repos merged into monorepo `india-platform`; new Next.js + shadcn/ui console built and deployed on port 8020.
- 2026-10-01 — Uptime page added; ops.vidivu.in switched to the new console; auto CI/CD for web and engines; water engine deployed by pipeline.
- 2026-10-02 — All engine timers every 3 hours; Engines page counts fixed for Laws and Living Species. Chat: SSH details, PikaPods vs OVH comparison (stay on OVH).
- 2026-10-05 — Simpler mobile menu in the ops console.
- 2026-10-06 — Everything committed and pushed.

## 6. Key facts
- **Owner:** [[People/Vishnu]] — GitHub `vishnuvarthan18`, works on a Mac (`mac-2-lan`, Apple Silicon). Non-technical; wants plain English.
- **Server:** [[Companies/OVHcloud]] US branch (OVH US LLC, `auth.us.ovhcloud.com`), VPS-1 2027, Oregon USA, Ubuntu 26.04. IP `40.160.137.239`, host `vps-e8d92c83.vps.ovh.us`. Login: `ssh ubuntu@40.160.137.239` (root login off).
- **On the VPS:** repos in `~` without the `india-` prefix (`~/geo-engine`, `~/culture-engine`, `~/core-infra` = `india-data-core`, `~/india-ops-console`, ...). DB: `docker exec -i core-postgres psql -U india -d india_data`. Console tunnel fallback: `ssh -L 8010:127.0.0.1:8010 ubuntu@40.160.137.239`.
- **Console:** `https://ops.vidivu.in` (domain vidivu.in on [[Tools/Cloudflare]]). [[Tools/nginx]] → new console `127.0.0.1:8020` (old Python console on 8010 as fallback).
- **Dev history:** [[Projects/india-data-atlas/DEV-LOG]] (Claude Code sessions, personal Mac). Chats index: INDEX (archived: Projects/india-data-atlas/chats/INDEX.md). Project docs copied to `docs/`.
- **Monorepo (since 2026-09-29, private):** `vishnuvarthan18/india-platform`, local `~/india-monorepo`. Folders: `core/`, `protected-areas/`, `engines/`, `ops-console/`, `web/` (new console), `docs/`.
- **Old repos (kept until monorepo is proven; [[Tools/GitHub]] under `github.com/vishnuvarthan18/`):** `india-data-platform` (was `ecotourism`), `india-data-core`, `india-ops-console`, `india-species-engine`, `india-forest-engine`, `india-water-engine`, `india-culture-engine`, `india-geo-engine`, `india-extinct-engine`, `india-laws-engine`.
- **Local folders:** `~/india-monorepo` (monorepo), `~/india-platform/` (old 10 repos). Old early path: `/Users/vishnuvarthanv/Ecotourism`.
- **Tools:** Next.js + shadcn/ui (new console), [[Tools/PostgreSQL]] + PostGIS, [[Tools/MinIO]], [[Tools/Docker]], [[Tools/systemd]], [[Tools/FastAPI]], [[Tools/Leaflet]], [[Tools/nginx]], [[Tools/Scrapy]] (early phase), [[Tools/Claude Code]], [[Tools/Claude]], [[Tools/GitHub]], [[Tools/Cloudflare]] (DNS, R2).
- **GIS tools planned:** [[Tools/QGIS]], [[Tools/Google Earth Engine]], [[Tools/MapLibre]], Martin, ESA SNAP, EnMAP-Box, NetLogo, OpenDroneMap.
- **Main data sources:** [[Companies/data.gov.in]] (covers 6 of 8 engines), [[Companies/GBIF]], [[Companies/Wikidata]], [[Tools/OpenStreetMap]] / Overpass, [[Companies/Kew]] WCVP + IPNI, Glottolog, Census of India, FSI ISFR, NWDP, ESA WorldCover, Global Mangrove Watch, MoEFCC, NTCA, e-Gazette.
- **Satellite data planned:** Landsat, Hansen GFC, GEDI, FIRMS, Black Marble, GRACE-FO, GPM ([[Companies/NASA]]); Sentinel-1/2, WorldCover ([[Companies/ESA]]); Bhoonidhi, NISAR S-band ([[Companies/ISRO]]); ALOS-2.
- **Hosting compared:** [[Companies/PikaPods]] — not fit for the platform; maybe for Uptime Kuma (~$2.50/month).
- **Secrets live only in** `.env` / `harvest-secrets.env` files on the VPS. None saved here.
- **Useful facts:** heartbeat job_keys differ from systemd unit names; after any hot-fix run `docker compose build`; run harvests by hand via the systemd service (it loads env); Mac device-bridge shell can't commit or push — Vishnu runs git in Terminal; use `git --no-optional-locks status` for read-only checks.
- **Visual pages:** platform article https://claude.ai/code/artifact/b6ba74f3-a47a-45b0-9cc6-798d179bf957 · subjects https://claude.ai/artifact/K1w2So9CW4x12GYUzi68vc · space data https://claude.ai/artifact/6EzvxSyNTVpawGAGpfoAdB · platform (GIS) https://claude.ai/artifact/Gigs4dZRp1RknS73mdJDQZ

## 7. Files and documents
Project files (claude.ai project "India Data Atlas"):
- `claude/next-phase-plan.md` — 2026-09-07 handoff for the DevOps agent: slug migration, PARIVESH resource ID, harvest wrapper, Phase G engines.
- `claude/session-2026-09-08-vps-recovery-and-deploy.md` — SSH fix, 9 repos to GitHub, big deploy (D-66–D-71).
- `claude/full-platform-architecture-plan.md` — the 6 layers, what is missing (staging, CI/CD, backups, legal, licensing).
- `claude/session-2026-09-16-pause-close-and-ops-console.md` — pause closed, D-71, console built.
- `claude/project-status-and-plan.md` — status by engine, source-atlas findings, build order.
- `claude/session-2026-09-17-console-redesign-and-hardening.md` — redesign, 184 tests, security, performance.
- `claude/ops-dashboard-plan.md` — console design and section status.
- `claude/session-2026-09-17-repo-cleanup-and-consolidation.md` — repo move to `~/india-platform/`, GitHub sync.
- `claude/open-items-next-session.md` — main "what's left" list (updated 2026-09-21).
- `claude/academic-project-subjects.md` — 9 academic subjects the data supports.
- `claude/webapp-gis-first-plan.md` — GIS-first web app decisions.
- `claude/space-data-gis-flagship.md` — PA Health Index from satellite data.
- `claude/gis-master-plan.md` — zero-spend 6-phase learn-and-build plan.
- `claude/session-2026-09-23-all-jobs-daily.md` — all jobs daily, geo-peaks fix.
- `claude/cicd-cleanup-backup-plan.md` — offsite backup, CI/CD, rollback plan + progress log.
In repos:
- `DECISIONS.md` (in each repo; D-numbers), `PLAN.md`, README files, `RUNBOOK.md` (planned).
- `india-data-core/ops/scripts/pg_backup.sh`, `make-all-daily.sh`.
- `india-ops-console/scripts/run-tests.sh`, `pin-base-image.sh`, `control/control_service.py`, migrations 0019–0022.

## 8. Open questions and problems
- **No end-use or money plan yet.** Options discussed: public map site, research tool, open dataset, story site; money via ads, affiliate, data licensing, grants (none chosen).
- **5 items only Vishnu can do** (API keys, rotation, OSM vs WDPA, India Code/Kanoon, bulk data + India-egress retest). India-WRIS/MoTA may look down only because the VPS is in the USA.
- **Backup is not offsite yet** — local MinIO is on the same disk as the DB.
- **No staging, few tests in engine code.** CI/CD is now live (2026-10-01); earlier manual SSH deploys caused wrong-shell mistakes.
- **Living Species failing (~10 days) and Protected Areas collecting rarely** — cause unknown.
- **Console is public on the internet** — the earlier plan said SSH tunnel only. Protected by Cloudflare + Basic Auth + own login, but `control_service.py` (old console, runs as root) has no outside review.
- Monthly sources now download the same data every 3 hours — wastes disk; watch it. (Overpass weekly choice was replaced by the every-3-hours switch — check public Overpass throttling.)
- geo-peaks: Wikidata query still slow; timeout setting not confirmed.
- D-71 `else 2` bug not checked in 7 engine repos.
- D-28 slug migration gap and PARIVESH stale resource ID (from 2026-09-07 plan) — not sure if done.
- Grasslands and coral reefs: no usable India source (permanent gap for now). Springs, waterfalls, estuaries: near-zero data.
- Mail (Aug–Sep 2026) mentions ops.vidivu.in "India Data Platform" and the india-platform repo — same project.
- Three unknown R2 buckets in the Cloudflare account (`blastdesk-downloads`, `forest-raw-pdfs`, `harvest-engine-raw`) — what made them?
- Extinct Species: build or drop? Laws: blocked.
- Legal before any public site: source licences/attribution, Terms, Privacy Policy (DPDP Act), Hindi/regional languages scope.
- `occurrence`, `timeseries`, `entity_relation` tables are still empty.
- GIS plan (2026-09-22) still lists "deploy the ops console" as needed, but it was deployed 2026-09-21 — the GIS doc is out of date.
- Early Cloudflare D1/R2 + GitHub Actions setup (Aug) was replaced by the VPS — when exactly, not sure.

## 9. All chats in this project
- Flora and fauna normalization pipeline with species deduplication — 2026-08-27
- Sathyamangalam Atlas pipeline fixes and coverage expansion — 2026-08-27 (belongs to [[Projects/forest/SUMMARY]])
- We have a problem — 2026-08-28
- Progress update — 2026-08-31
- Server password retrieval — 2026-09-07
- Moving project to viOS (12) — 2026-10-02
- Pikapods vs OVHcloud comparison — 2026-10-02
- Note: most build work (2026-09-02 to 2026-10-06) was done in [[Tools/Claude Code]] / agent sessions — see [[Projects/india-data-atlas/DEV-LOG]].
