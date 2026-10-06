# India Data Platform — Build Plan (DevOps Agent Instructions)

**Read this whole file before doing anything. Execute phases in order — do not skip ahead. Stop and report back at the end of each phase before starting the next one, unless told otherwise.**

**This is an unattended overnight run. Never stop to ask a question. If something is ambiguous or under-specified, make the most reasonable decision yourself, write it down with your reasoning in a `DECISIONS.md` file in this repo, and keep going. Only stop if you hit an action that is destructive/irreversible (e.g. deleting data, wiping a volume, force-pushing over history) and is not already explicitly covered by this plan — log it in `DECISIONS.md` and pause there only.**

Last written: 2026-09-06. This plan is the product of: (1) a live-verified survey of 136 India environmental/tribal/legal data sources ("India Data Source Atlas"), (2) a multi-part research pass on large-scale harvester architecture (scheduling, failure handling, change detection, avoiding blocks, data validation, monitoring, capacity/cost, tooling). Both are summarized inline where they drive a decision below.

---

## 0. What this project is

A full India-wide environmental/cultural data platform, built as a "super-app" of **8 independent engines** feeding **one shared core database**, via **one core API** that validates everything before it's written. Started as a protected-areas-only "Ecotourism Atlas" (still exists, still working — see Phase 2), now expanded to:

1. Protected Areas Engine (exists — this repo, `harvest-engine/`)
2. Forests & Land Engine (forest, deserts, grasslands, wetlands, mangroves, coral reefs)
3. Mountains & Geography Engine (peaks, ranges, glaciers, elevation)
4. Water Systems Engine (rivers, lakes, watersheds, groundwater)
5. Living Species Engine (all taxa, full checklists, not just sighted)
6. Extinct Species Engine (separate from #5 — historical/museum sources, mostly manual)
7. Tribal & Culture Engine (population, language, history, land rights, sacred groves)
8. Laws & Management Engine (who manages what, legal protections, land records)

No launch deadline. Build for months, refine over time. Do not block on total completeness — it doesn't exist for this scope.

---

## 1. Infrastructure already provisioned — DO NOT re-provision

- **Server**: OVHcloud VPS-1 2027, live and paid for.
  - IP: `40.160.137.239`
  - OS: Ubuntu 26.04 LTS
  - Specs: 2 vCore / 4 GB RAM / 40 GB NVMe / unlimited traffic (500 Mbps)
  - Login user: `ubuntu` (NOT `root` — root login is disabled on this image)
  - SSH key auth is already set up from `~/.ssh/id_ed25519` on the Mac at `/Users/vishnuvarthanv/Ecotourism`'s owner's machine. Passwordless `ssh ubuntu@40.160.137.239` is confirmed working.
  - No commitment / pay-as-you-go billing. Renews ~$5.35/month (~₹505), inside a hard ₹500–1000/month infra budget.
- **Local repo**: `/Users/vishnuvarthanv/Ecotourism` on the owner's Mac (git repo, already has commits, remote `origin` configured). Contains the existing, working Protected Areas harvest engine (see Phase 2 below for its current state).
- **Budget ceiling**: ~₹1000/month total infra spend. Do not provision anything (managed DB, managed queue, paid monitoring, proxies) that isn't free or trivially cheap without asking first.

**Why OVH and not Hetzner/Hostinger**: Hetzner was the original pick on cost/spec grounds (CX23, 2vCPU/4GB/20TB) but its Cost-Optimized tier was sold out fleet-wide at signup time — not a permanent rejection, just unavailable stock. Hostinger's cheapest plan clearing the ₹1000 ceiling required a 24-month prepay and still landed at ₹599+tax/month. OVH VPS-1 hit the ₹500 target with no prepay commitment. If cost/performance ever needs revisiting, Hetzner Cost-Optimized (CX23/CAX11) restocking is worth a periodic check — it remains the best price-to-spec match found, just unavailable at signup time.

---

## 2. Existing Protected Areas engine — what's already built (read before touching it)

Located in this same repo, `harvest-engine/`. **This is a working, tested pipeline. Understand it before changing it.**

- Scrapy-based (`scrapy==2.18.0`, see `harvest-engine/requirements.txt` for the full dependency set — boto3, shapely, pymupdf/pdfplumber for PDF parsing, pyyaml for source configs).
- `harvest-engine/sources/*.yaml` — one YAML config per source (NTCA, WII gazette, PARIVESH, GBIF, eBird, iNaturalist, Overpass, MoEFCC elephant corridors, tiger mortality, Sathyamangalam ex-gratia). This is exactly the "acquisition as config, extraction as code" pattern the architecture research independently validated — keep it.
- `harvest-engine/harvest_engine/spiders/` — three spider types: `api_source.py` (generic API-driven), `bulk_csv.py`, `seed_list.py` (the NTCA/WII gazette scraper).
- `harvest-engine/scripts/normalize_seed_list.py` — turns raw scraped HTML into `reserve`/`reserve_fact` rows. Handles state-name aliasing (Chattisgarh/Chhattisgarh etc. via `STATE_ALIASES`), cross-source matching, type-suffix regex.
- `harvest-engine/scripts/run_seed_list_harvest.py` — orchestrates a harvest run, with a per-source cooldown gate (`SOURCE_COOLDOWN_DAYS`, currently a flat 30 days for all seed-list sources).
- `harvest-engine/db/migrations/*.sql` — the current schema (reserves, facts, zones, corridors, hydrology, threats, flora/fauna). This targeted Cloudflare D1 originally.
- `.github/workflows/harvest.yml` — currently runs the harvest on a monthly GitHub Actions cron (`17 3 1 * *`) and has an unexecuted deploy job targeting Cloudflare Pages/D1/R2.
- **Known bugs/gaps, already diagnosed, not yet fixed:**
  - No code writes `centroid_lat`/`centroid_lon`, `boundary_geojson`, or `bbox` for reserves. All 602 current reserves have NULL geometry — the map has zero pins. Fix: pull polygons from WDPA (`api.protectedplanet.net/v3`, free API key, `with_geometry=true`) — see Phase 6.
  - `parivesh` spider reports "OK" even when it fails at startup due to missing `DATA_GOV_IN_API_KEY` handling (`scrapy crawl` exits 0 on a spider start error). Fix this in Phase 6 when the source-registry pattern is formalized.
  - WII's Drupal pages carry per-request session nonces that defeat the existing SHA-256 raw-content dedup. The cooldown gate is currently the only real defense. Proper fix (per the change-detection research): hash the **extracted fields**, not the raw HTML — do this when the source-registry/change-detection layer is built in Phase 6.
  - Telangana has no WII source page at all (2 tiger reserves — Kawal, Amrabad — can never cross-merge with a WII row). This is a real, permanent data gap, not a bug. Leave it flagged in the schema, don't try to fix it.
  - `SOURCE_COOLDOWN_DAYS` is one flat number for every source. Should be tiered per source's real update cadence (see the source-atlas per-source "Updates" column, Phase 6).

**Do not delete or rewrite this engine's logic wholesale.** It gets migrated (Phase 6), not replaced.

---

## 3. Non-negotiable architecture decisions (already made, backed by research — do not relitigate)

1. **8 separate engine repos, 1 core repo, 1 shared Postgres+PostGIS database.** Engines never write to the DB directly — they call the core API, which validates then writes.
2. **Every fact/record carries: `source`, `retrieved_at`, `confidence`, `license`, `publish_precision`.** The last two are new requirements from the source-atlas research: real per-source and even per-record licensing conflicts exist (GBIF is CC0/CC-BY/CC-BY-NC-SA *per record*; WDPA, GADM, and IUCN forbid commercial use/redistribution; some data — sacred grove coordinates, FRA claimant records — should never be published at street-level precision, only aggregated to district). `publish_precision` should be an enum: `full`, `district_aggregate`, `withhold`.
3. **Three-stage pipeline inside every engine**: **Acquire** (raw bytes + full response metadata — status, headers, ETag, Last-Modified, fetch timestamp — written to object storage before any parsing, keyed by source+partition+content-hash) → **Parse** (pure function, raw blob → records, versioned, re-runnable without re-fetching) → **Load** (validated upsert by stable ID, via the core API). This is both what the existing PA engine already approximates (raw HTML → R2 → normalize) and what the architecture research converged on independently (Common Crawl's WARC/WAT/WET split, pupa/Open States' scrape/import split). Formalize it everywhere.
4. **Scheduling runs on the VPS's own systemd timers — never GitHub Actions cron.** This is not a style preference; it's evidence-backed: GitHub's own staff confirmed (2026-06-04) that scheduled-cron drift has gotten "&gt;30% worse in 2ish months," with real delays of 1–3+ hours as of mid-2026, and this delay is upstream of runner allocation (it affects self-hosted runners triggered by Actions cron too). Scheduled workflows on public repos also auto-disable after 60 days of inactivity — a direct hazard for a project meant to run untouched for months. GitHub Actions is fine to keep for CI/tests; just not as the scheduler.
5. **No DAG orchestrator (Airflow/Dagster/Temporal).** This system's shape is "N mostly-independent per-source jobs," not a dependency graph over a time partition — the shape those tools are built for. Use systemd timers per source, or a lightweight task queue (Celery/RQ on Redis) if/when the number of sources makes per-source systemd units unwieldy. Do not introduce Airflow, Dagster, Temporal, or similar at this project's scale — the research is explicit that they cost more than they return for this workload shape (concrete example: Dagster's managed pricing bills per-asset-materialization×frequency, which is actively hostile to a many-small-sources harvester).
6. **Coarse per-source schedule tiers, not adaptive/learned scheduling.** Bucket sources into a handful of cadences (e.g. 15-min / 6-hour / daily / monthly / quarterly / annual) matching their real-world update frequency (this is already recorded per-source in the India Data Source Atlas's "Updates" column — reuse it, don't re-derive it). Always pair a cadence with an absolute staleness ceiling independent of the schedule logic (this is the fix for a known, real bug class in Nutch's adaptive scheduler — don't reinvent adaptive scheduling from scratch).
7. **No headless browsers fleet-wide, no residential proxies, ever (for this project).** Government/scientific sources want requesters *identified* (declared User-Agent + contact email, registered API keys), not hidden — proxy rotation works against what these sources actually want and buys nothing here. Headless browsers cost 10–100× more in compute/bandwidth than plain HTTP fetching; use Playwright narrowly, only for the specific India sources that are confirmed JS-only (PARIVESH, INGRES, DILRMP, Bhu-Naksha), never as a default.
8. **Monitoring: a dead-man's-switch/heartbeat is the single highest-value control**, not elaborate dashboards. Every scheduled job pings a heartbeat on success; silence past a threshold alerts. This is what catches the dangerous failure mode: a scraper that returns zero rows without ever throwing an error. Use a self-hosted option (Healthchecks.io is open-source and self-hostable) to stay inside budget.
9. **Re-test, don't trust, any source marked "Blocked" in the Source Atlas.** 15 sources (iNaturalist, eBird, ChecklistBank, Xeno-canto, NCBI, Overpass, Wikidata, India-WRIS, CWC flood forecasting, CPCB, FAOSTAT) were blocked by the *research sandbox's* filtering proxy, not by the source itself. Verify each from this VPS with an honest, identified client before writing any of them off.

---

## 4. Phase-by-phase build order

### Phase A — Server base setup

Run on the VPS (`ssh ubuntu@40.160.137.239`):

1. Update the system: `sudo apt update && sudo apt upgrade -y`
2. Install Docker Engine + Docker Compose plugin (official Docker install script or apt repo — use the official Docker docs for Ubuntu 24.04/26.04, since the pre-installed version may be outdated). Add the `ubuntu` user to the `docker` group so commands don't need `sudo` every time: `sudo usermod -aG docker ubuntu` (then log out/in once).
3. Set the server's timezone to UTC explicitly (`sudo timedatectl set-timezone UTC`) — all scheduling and timestamps in this project should be UTC internally; convert to IST only at display time.
4. Set up basic firewall (`ufw`): allow SSH (22), and later whatever ports the core API/dashboard need — nothing else exposed publicly by default.
5. Confirm Docker works: `docker run hello-world`.

### Phase B — Core infrastructure (Postgres + PostGIS + MinIO)

All via Docker Compose on the VPS, in a new directory e.g. `~/core-infra/`.

1. Write a `docker-compose.yml` with two services:
   - **`postgres`**: use the official `postgis/postgis` image (a Postgres image with PostGIS pre-installed — pick a current stable major version, e.g. postgis/postgis:16-3.4 or newer available tag). Persist data to a named Docker volume, not a bind mount, unless there's a reason to want the files directly visible. Set a strong password via environment variable, stored in a `.env` file (never committed to git).
   - **`minio`**: official `minio/minio` image, running in server mode, with persisted storage on a named volume. Set root user/password via `.env`. Expose the S3 API port and the web console port (bind console only to localhost or behind auth — don't expose the MinIO console to the public internet without a reason).
2. Bring it up: `docker compose up -d`. Confirm both containers are healthy (`docker compose ps`, `docker compose logs`).
3. Create the initial MinIO bucket(s) needed for raw-payload archival (e.g. `raw-archive`), either via the MinIO console or the `mc` CLI.
4. **Do not skip**: set up a basic backup habit even before real data exists — e.g. a simple `pg_dump` cron/systemd-timer job writing to a local file or a second MinIO bucket. This is cheap now and expensive to retrofit later.

### Phase C — Core repo scaffold

Create a new repo (e.g. `india-data-core`), separate from this one, either on the VPS directly or on the Mac and pushed up — whichever the DevOps agent's setup makes easier, but it should end up runnable on the VPS via Docker.

1. **Schema** (Postgres + PostGIS): design tables for at minimum —
   - `source` (id, name, engine, base_url, schedule_tier, license_default, notes)
   - a generic `fact` or per-domain fact tables — follow the existing PA engine's `reserve`/`reserve_fact` pattern as the template, but make sure every fact-level table has: `source_id`, `retrieved_at`, `confidence`, `license`, `publish_precision`, `content_hash`.
   - `raw_archive_ref` — pointer table linking a fact back to its raw payload's location in MinIO (bucket + key), not the raw bytes themselves.
   - Use PostGIS geometry columns (`geometry(Polygon, 4326)` etc.) wherever spatial data is involved — reserves, protected area boundaries, water bodies, mountain ranges.
2. **Core API**: a small service (pick a lightweight, well-supported framework appropriate to whatever language the team is most comfortable maintaining long-term — Python/FastAPI is a reasonable default given the existing Python/Scrapy investment) exposing endpoints for engines to submit records. It must, per-request:
   - Authenticate the calling engine (per-engine API key).
   - Validate required fields are present (schema-level check).
   - Enforce dedup/upsert by stable identifier (never blind-insert).
   - Reject or flag records with missing `license`/`publish_precision`.
   - Log rejections with enough detail to debug later — don't silently drop bad records.
3. Containerize the core API (Dockerfile) and add it as a third service in the `docker-compose.yml` from Phase B, on the same Docker network as Postgres and MinIO so they can reach each other by service name.
4. Set up systemd timers (not GitHub Actions) for anything the core repo itself needs to run on a schedule (e.g. a nightly `pg_dump` backup job).

### Phase D — Register free API keys (do this in parallel with B/C, no dependency)

Register now, store as environment variables / secrets on the VPS (never commit real keys to git):

- `data.gov.in` — register a personal key (the repo currently references `DATA_GOV_IN_API_KEY` from the public sample key; replace with a real one).
- WDPA / Protected Planet (`api.protectedplanet.net`) — free self-serve key. **License note: no commercial use, no redistribution** — tag accordingly in the `source` table.
- IUCN Red List API v4 — free but restricted registration. **License note: commercial use is strictly forbidden; scraping-like misuse can get a token revoked** — respect published rate limits exactly.
- GeoNames — free username-based key.
- OpenTopography — free registration.
- GBIF needs no key for read access.

### Phase E — Migrate the Protected Areas engine (first real engine on the new core)

This proves the core API/DB/MinIO stack actually works end to end before building anything new.

1. Point the existing `harvest-engine/` at the new core API instead of the old D1/R2 targets — this is a write-target change, the scraping/normalizing logic itself is already tested and should not need a rewrite.
2. Fix the centroid/geometry gap: add a step that pulls polygon geometry from WDPA (`with_geometry=true`) and populates what the schema needs, replacing the current NULL centroid/boundary/bbox situation.
3. Move its scheduling off the GitHub Actions monthly cron and onto a systemd timer on the VPS, with the source-registry pattern from Phase C.
4. Fix, while touching this code anyway: the `parivesh` spider's silent-failure bug, and the WII nonce/dedup issue (switch to hashing extracted fields, not raw HTML).
5. Verify: run a full harvest against the new stack, confirm reserves land in Postgres with real geometry, confirm raw payloads land in MinIO first.

### Phase F — Second engine: Water Systems

Chosen to go second specifically because the India Data Source Atlas found the National Water Data Portal (`nwdp.nwic.gov.in`, open CKAN, no key required) to be the single best source in the entire 136-source survey — high value, and a good stress-test of the core API against a very different data shape (time-series telemetry, not static polygons) than the PA engine gave it.

1. Build acquisition against NWDP first (291 river datasets, 309 water-quality, 473 flood, 64 dam/reservoir, 1 glacial-lakes GeoJSON layer — check each dataset's license tag individually, they're inconsistent per-dataset per the Atlas).
2. Add HydroRIVERS/HydroLAKES (HydroSHEDS, static bulk downloads) for global-consistent river/lake topology, and JRC Global Surface Water for 40-year extent history — genuinely complementary to NWDP, not duplicative.
3. Known gaps to flag in the schema rather than chase: springs, waterfalls, and estuaries returned zero results on NWDP's own catalog — real absence, not a fetch failure.

### Phase G — Remaining engines, in this order (per the Atlas's per-engine findings)

3. **Living Species** — build against GBIF once (it already contains iNaturalist's 1.6M and eBird's 60.6M India records — do not also pull those APIs separately, that duplicates 95% of the corpus). Exclude those two dataset keys from any species-coverage metric. Add POWO for plant names/images, IUCN v4 (with the registered key) for conservation status.
4. **Forests & Land** — FSI ISFR via `data.gov.in`'s JSON extracts where the year exists, OCR the PDF chapters where it doesn't (chapter 2, the state-table chapter, is image-based — budget for OCR, not text parsing). Global Forest Watch API for change-over-time (note: its spectral "tree cover" definition includes plantations — **not comparable** to FSI's legal forest definition; never mix them in one metric). Grasslands and coral reefs: no usable India source exists at all — record as a permanent gap, don't build a spider for it.
5. **Mountains & Geography** — Copernicus DEM (public S3, no key) for terrain, LGD for the administrative gazetteer, USGS for earthquakes (public domain, free — use instead of India's paid NCS catalog). Re-test Wikidata SPARQL and Overpass from this VPS for named peaks — both were sandbox-blocked in the research, not necessarily actually blocked.
6. **Tribal & Culture** — Glottolog (clean CC-BY-4.0 JSON/RDF) for language classification. MoTA FRA reports for the rest — re-test the individual PDFs from this VPS, they were blocked in the research sandbox. **Hard rule, not optional**: sacred grove coordinates, traditional medicinal knowledge, and any individual/village-level FRA claimant data must be published at `district_aggregate` precision at most, never `full` — this is both a legal and ethical line drawn explicitly in the Atlas.
7. **Extinct Species** — mostly a manual compilation project, not a scraping project; budget researcher time, not engineering time. The one automatable piece is per-species GBIF occurrence pulls for last-record dates, once a hand-built species list exists. Don't build against the Paleobiology Database — its own robots.txt forbids the entire data API by policy.
8. **Laws & Management** — e-Gazette as the primary feed (upstream of India Code itself, which is currently down on both its domains — check periodically whether it's back). Indian Kanoon for practical judgment search access (no CAPTCHA on its free interface, unlike the actual court sites).

---

## 5. Things to explicitly NOT do

- Do not provision Cloudflare D1/R2 for anything — that plan is fully superseded.
- Do not schedule anything via GitHub Actions cron (see §3.4) — CI/tests only.
- Do not introduce Airflow, Dagster, Temporal, Kubernetes, or Kafka at this scale — none of them are justified by this workload (single small VPS, ~136 known sources, mostly slow-changing).
- Do not use residential proxies or rotate IPs to evade rate limits — identify honestly instead; every relevant source here explicitly prefers this.
- Do not run headless browsers as the default fetch method — HTTP-only unless a source is confirmed JS-only.
- Do not treat a source in the Atlas marked "Blocked" as dead without re-testing it from this VPS first.
- Do not publish sacred-grove coordinates, traditional medicinal knowledge, or individual/village-level FRA data at full precision.
- Do not exceed the ~₹1000/month total infra budget without checking first — that includes not casually adding managed services, paid monitoring tiers, or proxy bandwidth.

---

## 6. Reference material

The full India Data Source Atlas (136 sources, per-category breakdowns, licensing details, "which one to build against" per engine) and the full harvester-architecture research (scheduling theory, failure handling, change detection, avoiding blocks, data validation, monitoring, capacity/cost curves, tooling comparisons) were both reviewed in full to produce this plan. If a decision in this plan seems under-justified, the reasoning is almost certainly in one of those two source documents — ask for the relevant excerpt rather than re-deriving it from scratch.
