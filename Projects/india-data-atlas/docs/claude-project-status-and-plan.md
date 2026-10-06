# Ecotourism Atlas → India Data Platform — Status & Plan

_Last updated: 2026-09-16. **The "nothing is deployed" warning that headed this doc for a week is resolved** — the deepening work was deployed later on 2026-09-08 and all 16 jobs were verified running on 2026-09-16. The backend is on autopilot. The new frontier is the ops console (built, not deployed) and the five standing user decisions._

Plain-language article covering the whole platform end to end: https://claude.ai/code/artifact/b6ba74f3-a47a-45b0-9cc6-798d179bf957

## Current status (2026-09-16)

**Backend: running on its own, verified.** Six of eight engines live, 16 systemd jobs, nightly Postgres backups, heartbeat every 5 minutes. On 2026-09-16 every one of the 16 jobs was confirmed to actually fire and finish cleanly — three had fired on their own schedule since the pause, three were triggered by hand (their next natural run is in October, so waiting would have proven nothing; all harvests are idempotent).

**Console: built, not deployed.** `india-ops-console` — all nine sections of `ops-dashboard-plan.md`, 56 files, 2 commits, tested against a throwaway Postgres with PostGIS and the real schema. Currently at `~/india-data-platform/india-ops-console`; move it to `~/india-ops-console`. No session has yet had a route to the VPS to deploy it.

**The real blockers are now decisions, not code.** See "Five things that need the user" below.

### Engines in production

| Engine | Status | Jobs |
|---|---|---|
| Living Species | 14,572 GBIF taxa + 5,480 WCVP/IPNI-enriched plant taxa | 2 (weekly) |
| Mountains & Geography | 6,196 entities incl. passes and ranges | 4 (mixed) |
| Tribal & Culture | 830 languages + Census ST population, 585 districts across 30 states | 3 (monthly) |
| Water Systems | 624 entities | 1 (daily) |
| Protected Areas | 602 entities, 405 with real geometry | 1 (daily) |
| Forests & Land | FSI ISFR, mangroves, wetlands, desertification | 4 (monthly) |
| Laws & Management | e-Gazette only — blocked on the India Code / Kanoon call | 1 (daily) |
| Extinct Species | Not started — repo exists, needs a build-or-drop decision | 0 |

**Infrastructure:** VPS live, ~20% disk used, real data footprint ~280MB, zero known open bugs in what's deployed.

## Five things that need the user

1. **Register 5 API keys** — WDPA, IUCN v4, GeoNames, OpenTopography, GFW. Unblocks the most future work.
2. **Rotate `DATA_GOV_IN_API_KEY`** — it covers 6 of 8 engines, so treat any leak as platform-wide.
3. **OSM vs WDPA** for Protected Areas geometry — parked on OSM, never resolved.
4. **India Code / Indian Kanoon build-or-not** — this is the whole Laws & Management engine.
5. **Capacity call on bulk datasets + an India-egress retest** for India-WRIS/MoTA. The VPS is in Oregon; that alone may be why those sources look unreachable.

These are now also rows in the console's own open-items checklist, so they stop living only in a doc.

## Next dev priorities

1. Deploy the ops console (both phases) and verify it against the live database.
2. Audit the other seven engine repos for the D-71 exit-code bug (`grep -rn 'else 2' ~/*/scripts/*.py`).
3. Extinct Species and Laws & Management build-or-drop decisions.
4. Public API layer, then the public-facing website — which does not exist at all yet.

## Recent history

### 2026-09-16 — pause closed, D-71 found, console built

Full record in `session-2026-09-16-pause-close-and-ops-console.md`. Headlines:

- **D-71:** `harvest_fra_jk.py` returned exit code 2 on a `partial` run, so systemd marked it failed every month for a permanent flaw in the source table (Jammu districts sum to 5,063/5,101 against stated subtotals of 5,157/5,195). Partial now exits 0; the signal is preserved in the run record and heartbeat. Fixed on the VPS and committed.
- **D-70 fixed** in the console's migration 0019: per-job cadence replaces the flat 36-hour staleness window. 8 of 16 jobs are monthly and 4 weekly, so two-thirds of the fleet was permanently mis-flagged. Measured on a fleet fixture: 6 alerts → 2, both genuine.
- **Correction:** `forest-desertification` is monthly, not daily. Earlier docs were wrong.
- **Operational lesson:** editing a script on the VPS is not enough — the code is `COPY`-ed into the Docker image, so `docker compose build` is required after any hot-fix.

### 2026-09-08 — deepening pass, then deploy

Built and validated 7 migrations (0012–0018) and 9 harvester/normalizer scripts: wetlands, desertification, Wikidata mountain passes/ranges, Overpass grid-walk, Census ST population, J&K FRA titles, MoEFCC Elephant Reserves. A first session had SSH blocked by its own harness and could not deploy; a later session that day restored access, applied everything, installed 6 new timers, and ran a full verification pass with zero genuine alerts.

---

## Background (architecture and source findings that still hold)

### What this project is

Started as "Ecotourism Atlas" — a reserve-centric, citation-backed data atlas for India's officially designated Protected Areas. **Scope expanded** into a full India-wide data platform: forests, mountains, water, all species (living and extinct), tribal communities and history, laws and management. Long-term, no-rush, no clean "100% done" line by design.

Repos live at `~/<repo>` on the user's Mac (device `mac-2-lan`): `india-data-platform` (main), `india-data-core`, and one per engine.

### Architecture

**"Super-app" model:** 8 separate engines, each its own repo, all writing through **one core API** into **one shared Postgres+PostGIS database** — engines never write directly to the DB. The core validates real source, confidence, no missing fields, dedup, licence, and `publish_precision` (full / district_aggregate / withhold) on every fact. All six provenance fields are NOT NULL columns, so a record missing any of them cannot be stored at all — the API's validation is a friendlier error on top of a constraint that would have refused it anyway.

A database trigger caps sacred groves, traditional knowledge and FRA claims at district precision regardless of what any engine tries. **As of 2026-09-16 the console applies the matching rule on the way out**: restricted entities plot at a district representative point and CSV exports drop their coordinates.

### Infrastructure

OVHcloud VPS-1 2027, `40.160.137.239`, Ubuntu 26.04 LTS, 2 vCore / 4 GB RAM / 40 GB NVMe, ~₹505/month. Login is `ssh ubuntu@...`, not root. Postgres+PostGIS and MinIO via Docker Compose, bound to 127.0.0.1 only — Docker's iptables rules run before ufw's INPUT chain, so a 0.0.0.0 bind would be internet-reachable even with ufw denying everything (D-2). Reach services over an SSH tunnel.

### Scheduling decisions (closed)

- **Systemd timers on the VPS, not GitHub Actions** — Actions dispatch drift and 60-day auto-disable on inactive public repos make it unsuitable. Actions is CI/tests only.
- **No DAG/workflow framework** — N mostly-independent per-source jobs need a clock, not a coordinator.
- **Fetch → parse → load, raw-archive-first:** acquire raw bytes and metadata to MinIO before parsing; parse is pure and re-runnable; load is a validated upsert via the core API. Long jobs are detached systemd units, immune to SSH drops.
- **Per-source scheduling:** coarse cadence tier plus jitter, paired with an absolute staleness ceiling that does not depend on the scheduler being correct.

### Data source atlas — key findings

- `api.data.gov.in` covers 6 of 8 engines (small tabular extracts, not geodata).
- GBIF's India records are 92% eBird + 2.5% iNaturalist — ingest GBIF once only.
- India's own taxonomic/PA authorities are largely inaccessible — taxonomy and geometry inherit from international aggregators (GBIF, WDPA/OSM, WCVP/Kew, iDigBio).
- **NWDP is the best source in the survey** — 70 agencies, no key needed. Springs, waterfalls and estuaries are a genuine near-zero gap.
- **Deliberate policy blocks, not to route around:** PBDB robots.txt, Supreme Court/NGT CAPTCHA, TKDL patent-office restriction.
- FSI ISFR available via data.gov.in JSON directly — no OCR needed.
- **POWO's site is Cloudflare-bot-gated** — use Kew's WCVP via ChecklistBank, plus IPNI for authorship.
- **Grasslands and coral reefs have no usable India source at all** — checked twice, treat as a stable permanent gap pending a research partnership.
- **HydroLAKES and JRC Global Surface Water are bulk-download-only**, no lighter API.
- **censusindia.gov.in has an incomplete TLS cert chain** — worked around (same class of fix as e-Gazette's).
- Full per-source notes and licensing table live in each repo's `DECISIONS.md`, and are now rendered and searchable in the console.

### Build order

PLAN.md §4: Phase A (server) → B (Postgres+PostGIS+MinIO) → C (core repo/API) → D (API keys) → E (migrate PA engine) → F (Water Systems/NWDP) → G (remaining 6 engines). **Phases E and F live. Phase G: 6 of 8 engines live and verified running. Extinct Species and Laws & Management await decisions, not work.**
