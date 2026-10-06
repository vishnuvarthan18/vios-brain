# DECISIONS.md

Decision + status log for the unattended overnight build run against `PLAN.md`.

- **Run started:** 2026-09-06
- **Operator:** unattended agent run (no interactive confirmation available)
- **Rule in force:** where `PLAN.md` is ambiguous, make the most reasonable call, log it here with reasoning, and continue. Stop only for destructive/irreversible actions not already covered by the plan.

Conventions used below:
- **D-n** = a decision made because the plan was ambiguous or under-specified.
- **S-n** = a status update at the end of a phase.
- **B-n** = a blocker that could not be resolved unattended (logged, worked around, run continues).

---

## Pre-flight

**Verified before starting Phase A:**
- `ssh ubuntu@40.160.137.239` — passwordless login confirmed working.
- Server identity: host `vps-e8d92c83`, Ubuntu 26.04 LTS, 3814 MB RAM, 38 GB disk (2.2 GB used), load ~0.18. Matches the spec in PLAN.md §1.
- Local repo `/Users/vishnuvarthanv/Ecotourism` on branch `main`, 4 commits, working tree has uncommitted changes to `.github/workflows/harvest.yml` and `harvest-engine/scripts/run_seed_list_harvest.py`, plus untracked `PLAN.md` and `harvest-engine/scripts/normalize_seed_list.py`.

---

## Phase A — Server base setup

**D-1 — Ran `apt full-upgrade` and rebooted once, beyond the plan's literal `apt upgrade -y`.**
PLAN.md A.1 says `sudo apt update && sudo apt upgrade -y`. That ran clean (17:37:58–17:39:41 UTC, ~160 packages including systemd, openssh-server, libc6, openssl), but plain `upgrade` held back the kernel set (`linux-image-virtual`, `linux-headers-*`, `linux-virtual` 7.0.0-14.14 → 7.0.0-31.31, which are flagged `resolute-security`) because they pull in new packages, and the box then reported `*** System restart required ***`.
**Decision:** run `apt full-upgrade` to take the held-back security kernel, then reboot once.
**Reasoning:** (a) leaving a 17-revision-old kernel with pending security updates on a public-IP server that is about to host the project's only database is the wrong default; (b) a reboot is not destructive or irreversible — nothing is installed on the box yet, there is no data to lose, and this is the cheapest possible moment to take the downtime. Doing the same reboot after Phase B puts Postgres data on the volume would be strictly riskier. Logged rather than silently skipped because it is a (small) widening of the plan's literal step.

**Note (not a decision, for the record):** SSH refused connections for ~2 minutes mid-upgrade. Cause was the `openssh-server` 1:10.2p1-2ubuntu3.2 → 3.6 restart, confirmed from `/var/log/apt/history.log`. Not a fault; noted so a future reader doesn't chase it.

**D-2 — All container ports bind to `127.0.0.1`, not `0.0.0.0`.**
PLAN.md A.4 says "nothing else exposed publicly by default" and B.1 says bind the MinIO console to localhost. Generalised to *every* published port (Postgres 5432, MinIO 9000/9001, core API 8000).
**Reasoning:** Docker's own iptables `DOCKER` chain is evaluated before ufw's `INPUT` rules, so a `0.0.0.0`-published container port is reachable from the internet *even with ufw set to deny incoming* — a well-known and frequently-exploited footgun. Binding to `127.0.0.1` in the compose port mapping is the fix that does not depend on ufw being correct. Reaching these services from the Mac is done over an SSH tunnel, which the project already has key auth for. When the core API eventually needs to be public, that gets a deliberate reverse proxy + ufw rule, not a widened bind.

**S-1 — PHASE A COMPLETE.**
- `apt update` + `apt upgrade` + `apt full-upgrade` + `autoremove`: done, 0 packages now upgradable.
- Rebooted onto kernel `7.0.0-31-generic`; `/var/run/reboot-required` clear.
- Docker Engine **29.8.0**, Compose plugin **v5.5.1**, containerd **v2.3.4**, installed from `download.docker.com` for `resolute` (the official repo does publish an Ubuntu 26.04 suite — verified `dists/resolute/Release` → HTTP 200 before adding it, so no need to pin to `noble`).
- `ubuntu` added to the `docker` group; verified in a *fresh* SSH session (`id -nG` → `ubuntu docker`) rather than assumed.
- Timezone: already `Etc/UTC` on the OVH image; explicitly set to `UTC` anyway per A.3.
- ufw: reset, default deny in / allow out, `22/tcp` allowed, enabled. Verified still reachable afterwards.
- `docker run --rm hello-world` succeeded **without sudo**. Storage driver `overlayfs`, disk 3.4G/38G used.

---

## Phase B — Core infrastructure (Postgres + PostGIS + MinIO)

**D-3 — One `docker-compose.yml` in a new `india-data-core` repo, deployed to `~/core-infra/`.**
Phase B.1 says write the compose file in `~/core-infra/` on the VPS; Phase C.3 says add the core API as a third service *in that same file*; Phase C says the core repo may live on the Mac or the VPS. Rather than have an untracked compose file on the server and a repo that does not contain its own deployment, there is **one** compose file, versioned in `/Users/vishnuvarthanv/india-data-core` on the Mac, `rsync`ed to `~/core-infra/` on the VPS. `~/core-infra/` is therefore a deployment of the repo, not a separate hand-maintained thing. Secrets (`.env`, `engine-keys.env`) exist **only** on the VPS at mode 0600 and are gitignored.

**D-4 — Postgres 17 + PostGIS 3.5 (`postgis/postgis:17-3.5`).**
The plan says "e.g. postgis/postgis:16-3.4 or newer available tag". `18-3.6` was also available and was checked. Chose 17: PG 18 is one major old at this point but 17 is the version whose extension/tooling ecosystem is fully settled, and this database is meant to run unattended for months. PG 17 is supported to 2029, so nothing is bought by taking the newer major today. Confirmed running: `PostgreSQL 17.5`, `PostGIS 3.5 USE_GEOS=1 USE_PROJ=1 USE_STATS=1`.

**D-5 — MinIO pinned to `RELEASE.2025-09-07T16-13-09Z`, and buckets created with `mc`, not the console.**
`minio/minio:latest` currently resolves to exactly that release — MinIO's community line stopped moving after the AIStor transition, so `latest` is effectively frozen. Pinned it explicitly anyway so a future `docker pull` cannot silently move the storage layer. Phase B.3 offers "the MinIO console or the `mc` CLI"; used `mc`, because these community releases ship only a stub web console and because bucket creation should be a re-runnable script (`ops/scripts/bootstrap_minio.sh`), not a click-path.
**Flagged, not acted on:** the community MinIO line is frozen, so it will not receive future security fixes. This is a *future* migration question (Garage and SeaweedFS are the S3-compatible successors worth evaluating), not a reason to deviate from a plan that names MinIO explicitly. Noted here so it is a known cost rather than a surprise.

**S-2 — PHASE B COMPLETE.**
- `~/core-infra/docker-compose.yml` up with `postgres` + `minio`, both reporting **healthy**.
- Named volumes `core_pgdata` and `core_miniodata` (not bind mounts, per B.1).
- Secrets generated on the VPS with `openssl rand`, written to `~/core-infra/.env` at mode 0600. Never transmitted to or stored on the Mac.
- Buckets `raw-archive` (object versioning **enabled** — raw payloads are the evidence layer and must not be silently replaced) and `pg-backups` created.
- Backups (B.4) done as a systemd timer rather than cron, since §3.4 wants systemd timers and §C.4 asks for exactly this job. `core-pg-backup.timer` runs 02:40 UTC nightly, `Persistent=true` so a missed night runs at next boot. **Verified by running it for real**: 94 KiB dump → `pg-backups` bucket → `pg-backup` heartbeat recorded `success`.

## Phase C — Core repo scaffold

**D-6 — A generic `entity` + `entity_fact` core rather than eight per-domain fact tables.**
Phase C.1 allows "a generic `fact` or per-domain fact tables — follow the existing PA engine's `reserve`/`reserve_fact` pattern as the template". Chose the generic form: a protected area, a river, a peak and a statute differ in their *attributes*, not in their identity or spatial shape, and attributes are exactly what `entity_fact` already holds per-source. Eight near-identical tables would mean eight copies of the same six provenance columns and eight upsert paths to keep correct.
**Not made fully generic:** `taxon`/`occurrence` and `timeseries_series`/`timeseries_point` are separate tables. A taxon's identity is nomenclatural (a GBIF usage key), not spatial, and NWDP telemetry is far too high-volume for a per-field fact table. Forcing those two into `entity_fact` would have been generic in the wrong direction.

**D-7 — The publish-precision limits are a database trigger, not API-layer validation.**
PLAN.md §5/§G.6 forbid full-precision publication of sacred grove coordinates, traditional medicinal knowledge, and individual/village-level FRA claimant data. Implemented as a `restricted_entity_type` table plus a `BEFORE INSERT OR UPDATE` trigger on `entity`. Enforcing it only in the API would mean all eight future engines have to remember it and every future write path has to re-implement it; in the database, a buggy engine cannot cross the line even by accident. **Verified**: `sacred_grove` at `full` is refused (rejection reason `precision_violation`), the same row at `district_aggregate` is accepted.

**D-8 — Sandbox-blocked sources seeded `untested`, never `blocked`.**
§3.9 lists 15 sources blocked by the *research sandbox's* filtering proxy rather than by the source. Seeding any of them `blocked` would bake the sandbox's artefact into the platform's own registry. They are `untested` with `status_checked_at` NULL, and `POST /v1/sources/{key}/status` is the only thing that can move them — i.e. only a real probe from this VPS.

**D-9 — Per-source schedule tiers now replace the PA engine's flat 30-day cooldown.**
§3.6 and §2 both call for this. Each of the 40 seeded sources carries a `schedule_tier` AND an independent `staleness_ceiling`. The ceiling is evaluated separately from the tier in `/v1/sources/due`, and a source due *only* because of its ceiling is reported with `due_because = past_staleness_ceiling` — that distinction is the signal that the tier schedule has stopped firing, which is the Nutch adaptive-scheduler bug class §3.6 warns about. Notable retiering: `ntca-tiger-reserves` 30d → quarterly (a legal register that gained 3 reserves in 2 years), `ntca-tiger-mortality` 30d → weekly (an incident stream, not a register), `wii-gazette-notifications` 30d → annual (an archive of already-final documents).

**Two real bugs found and fixed during Phase C verification** (recorded because both were caught only by testing against the live stack, not by reading the code):
1. `could not determine data type of parameter $1` — Postgres cannot infer a parameter's type inside `ST_GeomFromGeoJSON($1)`. Every geometry parameter is now explicitly cast (`%s::text`, `%s::float8`). Also confirmed on the live server that the PostGIS constructors used are all strict on NULL, which let the guarding `CASE WHEN` wrappers be dropped entirely.
2. **Per-record isolation was not actually per-record.** The first implementation ran a whole batch inside one transaction and called `rollback()` on a bad record — which would have discarded every *good* record already accepted in that batch. Rewritten to use an autocommit connection with `with conn.transaction():` around each record, so each one commits or rolls back alone. **Verified**: a 2-fact batch with one bad entity_uid now returns `accepted: 1, rejected: 1` and the good fact is present in the table afterwards.

**S-3 — PHASE C COMPLETE.**
- Repo `/Users/vishnuvarthanv/india-data-core` created, committed, deployed to `~/core-infra/`.
- Schema applied: `0001_core.sql` + `0002_seed_engines_and_sources.sql`. 8 engines, **40 sources** seeded with per-source licence, cadence, ceiling and status.
- Core API containerised (non-root uid 10001, healthcheck) and running as the third compose service. All of `/healthz`, `/readyz`, `/docs` live.
- Per-engine API keys minted for all 8 engines, SHA-256-only in the database, plaintext written once to `~/core-infra/engine-keys.env` (0600).
- **End-to-end verified against the live stack:** unauthenticated write → 401; polygon entity upsert → centroid `POINT(77.25 11.25)`, bbox and SRID 4326 all derived server-side, area 3020 km² (correct for a 0.5°×0.5° box at 11°N); a fact missing `license`/`publish_precision` → 422 with `reason_code=missing_required_provenance` **and a row in `ingest_rejection` carrying the payload**; the precision trigger refusing/accepting as designed; per-record isolation proven.
- Two systemd timers installed and enabled: `core-pg-backup.timer` (nightly 02:40 UTC) and `core-api-heartbeat.timer` (every 5 min against a 15-min silence tolerance). `/v1/ops/alerts` now reports zero heartbeat alerts.
- Smoke-test rows deleted afterwards; the database holds only the seeded registry.

---

## Phase D — Free API key registration

**B-1 — BLOCKER (worked around, run continued): five of the seven registrations cannot be completed unattended.**
WDPA/Protected Planet, IUCN Red List v4, GeoNames, OpenTopography and Global Forest Watch all require a human to complete a web signup: an email address to confirm, terms-of-use to accept as a legal person, and in several cases a CAPTCHA. I will not create accounts or accept licence terms on someone's behalf — for IUCN and WDPA in particular the terms carry real commercial-use restrictions that the account holder is agreeing to.
**What I did instead**, so that filling each key in is the *only* remaining step:
- Created `~/core-infra/harvest-secrets.env` (mode 0600, VPS only, gitignored) with a documented, commented placeholder for each key, the exact signup URL, and the licence restriction that applies to it.
- The source registry already records which env var each source expects (`source.api_key_env_var`), so no code changes are needed once a value is pasted in.
**The one Phase E consequence:** the geometry backfill in Phase E.2 depends on `WDPA_API_KEY`. See S-5 for how that was handled.

**D-10 — PLAN.md Phase D's claim about the data.gov.in key is wrong; no re-registration is needed.**
Phase D says "the repo currently references `DATA_GOV_IN_API_KEY` from the public sample key; replace with a real one." Checked: the value in the repo's `.env` is **not** the documented public sample key (`579b464db66ec23bdd000001cdd3946e44ce4aad7209ff7b23ac571b`), it is a distinct 56-char key, and it returns HTTP 200 from both the Mac and the VPS against a real `api.data.gov.in` resource. Treated as already provisioned; `data-gov-in` marked `active`. The existing `EBIRD_API_KEY` was also verified working from the VPS (HTTP 200).

**D-11 — India Code found reachable but WAF-blocked; deliberately NOT harvested in this run.**
PLAN.md §G.8 records India Code as "currently down on both its domains — check periodically whether it's back". Checked, and the plan's premise is wrong: `www.indiacode.nic.in` resolves via Akamai and returns **HTTP 200 to a plain browser User-Agent**. It returns **403 to any User-Agent naming this project** — tested four variants: honest UA → 403, `Mozilla/5.0 (compatible; india-data-platform/0.1; +mailto:…)` → 403, bare `Mozilla/5.0` → 200, full Chrome UA → 200. So it is an Akamai WAF UA rule, not an outage.
**Decision: record the finding, do not build the spider.** Reasoning: getting in requires a UA that conceals what the client is, and §3.7's whole posture is that these sources should see who is asking. There *is* precedent in this repo for a browser UA — `sources/ntca-tiger-reserves.yaml` already sets one, documented as "required, not optional" because NTCA returns 406 otherwise — so this is not a bright line the project has never crossed. But that precedent was a deliberate human call, the same call is a policy question rather than a technical one, and §G.8 makes e-Gazette the primary feed *upstream of India Code anyway*, so nothing in the plan is blocked by leaving it. Marked `blocked` in the registry with the full diagnosis. **This one wants a human decision.**

**D-12 — e-Gazette is up, but serves a broken TLS chain; left `untested` rather than called working.**
`egazette.gov.in` resolves and responds (HTTP 302 behind the TLS failure), but presents **only its leaf certificate** — `CN=egazette.gov.in`, Let's Encrypt issuer `CN=YR2`, valid to 2026-10-19 — and omits the intermediate, so every standard client fails with `unable to get local issuer certificate`. The fix is to supply/pin the LE `YR2` intermediate in the harvester's HTTP client. It is explicitly **not** to disable certificate verification: this is the primary legal feed for the Laws engine and unverified TLS on a legal source is not acceptable. Left `untested` because it is not yet actually fetchable — calling it `active` would be a lie the scheduler would then act on.

**S-4 — PHASE D COMPLETE (to the extent possible unattended), and the §3.9 re-test done as a deliverable.**
Provisioned + verified from the VPS: `DATA_GOV_IN_API_KEY`, `EBIRD_API_KEY`. Confirmed needing no key: GBIF, NWDP CKAN, Copernicus DEM, USGS. Awaiting a human: 5 keys (B-1).

**§3.9 re-test results — 15 sandbox-"blocked" sources probed from this VPS with an identified client. Most were never blocked:**

| Source | Verdict from this VPS |
|---|---|
| iNaturalist | **200 — not blocked** (stays retired only because GBIF already has its records) |
| Overpass | **200 — not blocked** (`/api/status`) |
| Wikidata SPARQL | **200 — not blocked** (live query) |
| ChecklistBank | **200 — not blocked** |
| NCBI E-utilities | **200 — not blocked** |
| FAOSTAT | **200 — not blocked** |
| CPCB | **301 — reachable** (redirect, not a block) |
| Xeno-canto | host **200**; `/api/2` gone (404), `/api/3` now **401 — needs a key**. Not blocked, just re-versioned behind registration |
| eBird | 403 without a key, **200 with the real key** — never blocked |
| India-WRIS | **genuinely unreachable** — DNS resolves (164.100.85.36) but TCP connect times out at 25s on both hostnames. Real block from this host, likely outbound geo/IP filtering since the OVH VPS is not on an Indian IP. Worth one retest from an Indian egress before calling it permanent |
| CWC flood forecasting | still `untested` — not probed in this pass |
| India Code | up, **WAF-403 on our UA** (see D-11) |
| e-Gazette | up, **broken TLS chain** (see D-12) |
| NWDP | **200, 558 datasets** — and it is as good as the Atlas says |
| GBIF | **200**, no key |

**Registry now: 11 active · 22 untested · 2 blocked · 2 permanent_gap · 3 retired.**

**D-13 — One correction to the Atlas's NWDP findings, recorded in the registry.**
PLAN.md §F.1/§F.3 numbers were checked against the live CKAN catalogue rather than trusted. Confirmed exactly: `river`=291, `flood`=473, `glacial`=1. Confirmed as genuine zero-result absences: `spring`=0, `waterfall`=0. **But `estuary` returns 2 datasets, not zero** — §F.3 lists estuaries alongside springs and waterfalls as a real absence, and that part is wrong. Corrected in the `nwdp-ckan` source note so Phase F does not skip two usable datasets.

---

## Phase E — Migrate the Protected Areas engine

**D-14 — Closed the geometry gap with OpenStreetMap, not WDPA. WDPA is implemented but dormant.**
PLAN.md §2/§E.2 names WDPA (`with_geometry=true`) as the fix for all 602 reserves having NULL geometry. WDPA needs a self-serve key that could not be registered unattended (B-1), so following the plan literally would have left the gap open for all of Phase E — and the geometry gap is the thing blocking both the map and every bbox-driven species query.
**Decision:** make OSM/Overpass the working geometry source now, keep the WDPA code path implemented and ready behind `--source wdpa`.
**Reasoning:** OSM was confirmed reachable from this VPS during the §3.9 re-test (HTTP 200), needs no key, and on the licensing merits is arguably the *better* primary source: OSM is **ODbL-1.0 — commercial use and redistribution both permitted** under share-alike, whereas **WDPA forbids both**. A platform whose output is meant to be publishable is poorly served by building its base geometry layer on a source that forbids redistribution. WDPA still earns its place as a cross-check once a key exists — the two disagreeing about a boundary is worth knowing, and per-fact provenance means both can be stored side by side. Registered as its own source row `osm-protected-areas` (migration 0003), separate from `overpass-osm`, because the two are different queries, cadences and engines on a shared host.

**D-15 — Rewrote the polygon fetch from per-object to batched, mid-run, after it proved to be a bad citizen.**
The first polygon implementation issued one Overpass query per protected area. Against 402 objects on a free, donated-capacity service that produced a wall of HTTP 429s and 504s, **six polygons in twenty minutes**, and a projected runtime in days. That is exactly the behaviour PLAN.md §3.7 tells this project not to have. Stopped it and rewrote the pass to resolve matches locally first, then fetch geometry in batched `id:` queries — **7 requests instead of 402**, with a 5s gap between them. Logged because it is a real reversal, made on evidence, not a detail.

**D-16 — `parivesh` has never worked, and the fix is what proved it.**
The plan describes the bug as "reports OK even when it fails at startup due to missing `DATA_GOV_IN_API_KEY` handling". Fixed in `scripts/run_spider.py` with three checks replacing the exit code: a registry-driven pre-flight on `requires_api_key`/`api_key_env_var`, spider construction inside `CrawlerProcess` so a raising `__init__` is caught, and post-run stats inspection. **Verified**: missing key → exit 1 with a clear message; a `blocked` source → exit 1; a working source → exit 0.
**But with the key present it still reported success while achieving nothing.** The archived payload is 190 bytes: `{"message":"Meta not found","status":"error","total":0,"records":[]}`. The data.gov.in resource id in `sources/parivesh.yaml` does not exist. So the source has never returned a single record, and the original bug was hiding a second one underneath it.
**Fixed that too, generically:** `api_source.py` now counts records in JSON responses and detects API-level error payloads, and `run_spider.py` fails a run where every counted response returned zero records. An archived HTTP 200 is a *fetch*, not a *harvest*. `parivesh` left `untested` with the diagnosis recorded — fixing it needs someone to find the correct resource id, which is a data question, not a code one.

**D-17 — SECURITY: API keys were being written to the database. Fixed at the core API, and the stored value scrubbed.**
Found while inspecting the parivesh archive: data.gov.in takes its credential as a **query parameter**, and the fetched URL is stored as provenance in `raw_archive_ref.request_url` — so the live `DATA_GOV_IN_API_KEY` was sitting in the database in plaintext, and therefore in every nightly backup. Protected Planet (`?token=`) would have done the same.
**Fixed in the core API rather than in each engine**, so no engine can leak a key by forgetting: `redact_url()` rewrites any query parameter named `api-key`, `api_key`, `apikey`, `key`, `token`, `access_token`, `auth`, `password`, `secret`, `signature` or `sig` to `REDACTED` before the row is written. Unit-checked against four URL shapes. The one already-stored credential was scrubbed with an `UPDATE`. **Note for the owner: that key was briefly stored in plaintext and is in any backup taken before 2026-09-06T18:40Z — rotating `DATA_GOV_IN_API_KEY` is cheap and would be the tidy thing to do.**

**D-18 — Deterministic entity slugs; the old collision loop is gone.**
The D1 normalizer's `_unique_slug()` tried `bandipur`, then `bandipur-tiger-reserve`, then `bandipur-karnataka`, then numeric suffixes — up to 100 SELECTs per reserve, with a result that depended on insertion order, so the same input could produce different slugs on different runs. The slug is now a pure function of the identity tuple the old code already matched on: `slugify(name)-slugify(type)-slugify(state)` → `bandipur-tiger-reserve-karnataka`. Same identity rule, no lookups, genuinely idempotent, and the old "ambiguous match, >1 row" branches become unreachable because a uid is unique by construction.

**D-19 — Kept `normalize_seed_list.py` and the other D1-era scripts.**
They target the retired D1 schema and do not run, but `normalize_reserves.py` **imports** their extraction and identity logic — the parsers, `STATE_ALIASES`, `TYPE_SUFFIX_MAP`, `split_wii_name_and_type`, `canonical_state`. That logic was written and verified against the live tables and is the valuable part; only the write target changed, which is exactly what PLAN.md §E.1 asks for ("a write-target change, the scraping/normalizing logic … should not need a rewrite"). Deleting them to tidy up would have meant copying the logic, i.e. forking it.

**S-5 — PHASE E COMPLETE.** Verified end to end against the live stack.

| E.n | Result |
|---|---|
| E.1 repoint at the core API | Done. `RawStoragePipeline`(R2) + `D1LoaderPipeline`(D1) replaced by one `CoreRawArchivePipeline`. **The engine now holds no database or object-store credentials at all** — one per-engine API key is its whole authority. |
| E.2 geometry gap | **602 → 402 reserves with a centroid** (was 0). Polygon pass running. 200 unmatched, with per-reason counts recorded; matching is name+state and deliberately refuses to guess. |
| E.3 systemd, not Actions cron | `pa-harvest.timer` daily 03:20 UTC, `Persistent=true`. `.github/workflows/harvest.yml` (monthly cron + Cloudflare deploy) **deleted**, replaced by `ci.yml` with tests only — zero `schedule:` triggers remain in the repo. |
| E.4 parivesh + WII bugs | Both fixed, plus a third found underneath parivesh (D-16) and a security bug found while verifying (D-17). |
| E.5 full harvest verification | **All 5 active sources SUCCESS.** |

**Data actually landed:**
- **602 protected areas** — which matches the plan's own "all 602 current reserves" exactly, an independent confirmation the migration lost nothing.
- **1,572 facts**, zero rejections.
- **33 cross-source merges** (WII National Park / Wildlife Sanctuary rows correctly merged into their NTCA tiger reserve rather than duplicating it).
- **10 records correctly skipped**, not guessed: the ambiguous bare-`CR` suffix (conservation vs community reserve) and unrecognised suffixes, each logged by name for manual review.
- Raw payloads in MinIO first, every time; `harvest_run` rows opened and closed per source.

**WII dedup fix, concretely:** change detection now hashes the extracted records (`harvest_engine/content_hash.py`), not the raw HTML. Unit-checked: stable under dict key reordering, sensitive to a value change, and sensitive to row reordering. Raw-body hashing still runs one level down in the core API, where it correctly answers the different question of whether the *payload* is byte-identical.

---

## Phase F — Water Systems engine

**D-20 — The NWDP licence inventory is treated as the deliverable, not a preliminary.**
PLAN.md §F.1 says to check each dataset's licence tag individually because they are inconsistent. That instruction does more work than it looks: until the inventory exists, nothing built on NWDP can honestly state what it may do with any given record. So the catalogue pass makes every dataset an `entity` carrying **its own** licence, and a dataset with no licence tag is recorded as `UNSTATED — treat as All Rights Reserved until confirmed` rather than quietly upgraded to "open".
**The plan was right to insist.** Across all 558 datasets: **541 Other (Open) · 7 Other (Non-Commercial) · 5 UNSTATED · 4 "License not specified" · 1 casing variant.** So **17 of 558 (3%) are not freely open**, including 7 that forbid commercial use outright. A blanket "NWDP is open" assumption would have mislabelled every one of them.

**D-21 — The engine verifies the Atlas's numbers on every run instead of trusting them.**
`ATLAS_EXPECTATIONS` in `water_engine/nwdp.py` holds the Atlas's per-keyword counts, and the catalogue pass re-runs them against the live portal each time. Cheap, and it turns a stale figure into a visible discrepancy rather than a silent wrong assumption.

**S-6 — PHASE F COMPLETE (NWDP), bulk sources registered but not ingested.**
- **558 datasets harvested, 3,881 facts, zero fetch failures.** Raw catalogue listing archived to MinIO first.
- Atlas counts **confirmed exactly**: `river`=291, `flood`=473, `glacial`=1.
- Genuine absences **confirmed and recorded** as `coverage_gap` entities with `is_permanent_gap`: `spring`=0, `waterfall`=0 (PLAN.md §F.3).
- **Correction (D-13): `estuary` returns 2 datasets, not zero.** §F.3 lists estuaries alongside springs and waterfalls as a real absence; that part is wrong, and Phase F would have skipped two usable datasets on its authority.
- HydroRIVERS/HydroLAKES and JRC Global Surface Water: **registered with licences recorded and confirmed reachable (HTTP 200), deliberately not ingested.** The global HydroRIVERS geodatabase alone is ~2 GB, and this is a 40 GB disk shared with Postgres and MinIO. Ingesting it is a capacity decision that wants the regional Asia extract and a streaming reader, and it is the owner's call rather than something to do unattended at 3am. Recorded in the engine README so it is a known next step, not a forgotten one.
- India-WRIS: **genuinely unreachable from this VPS** (DNS resolves, TCP times out). Marked `blocked` with the diagnosis; likely outbound geo-filtering, worth one retest from an Indian egress.

---

## Phase G — Remaining engines

Built in the plan's stated order. Every engine is its own repo (§3.1), containerised, on its own systemd timer, and holds **no database or object-store credentials** — one per-engine API key is its entire authority.

**D-22 — §G.3's eBird/iNaturalist exclusion needed three queries, not a filter, and the first implementation faked it.**
The plan says to exclude those two GBIF dataset keys from any species-coverage metric. My first `species_facet_for_wkt` **documented** a "negated datasetKey predicate" and did not implement one — the parameter simply was not there. GBIF's simple `/occurrence/search` API has **no NOT operator**; repeating `datasetKey` ORs values together, and negation exists only in the asynchronous Downloads predicate API. Rewritten to do it subtractively: query all, query eBird-only, query iNaturalist-only, and take `net = all − eBird − iNat`, dropping any species whose net count is zero.
**The exclusion matters enormously.** Verified live: those two datasets are **62,235,817 of 65,579,568 India occurrences — 94.9%**, matching the plan's "95%" almost exactly. In sample reserves the effect is Kanha 999 → 357 species and 1,386,449 → 14,284 occurrences. Without it the metric measures birdwatcher attention, not biodiversity knowledge. Every `taxon_entity` row records `excluded_dataset_keys` so the figure is auditable rather than asserted.

**D-23 — India Code and Indian Kanoon: both of the plan's §G.8 premises are now wrong.**
- **India Code** is described as "currently down on both its domains". It is **up** — HTTP 200 to a plain browser UA, 403 to any UA naming this project. An Akamai WAF rule, not an outage (D-11).
- **Indian Kanoon** is described as having "no CAPTCHA on its free interface, unlike the actual court sites". It now sits behind a **Cloudflare managed challenge**: `/`, `/search/` and even `/robots.txt` all return the "Just a moment…" interstitial requiring JavaScript and cookies.
Both are marked `blocked` with the full diagnosis rather than harvested. Getting past either needs a client that conceals what it is, and §3.7's posture is that these sources should see who is asking. **Both want a human decision, not an unattended one.**

**D-24 — e-Gazette: diagnosed and fixed properly, with verification intact.**
e-Gazette is §G.8's primary feed, and the Atlas had it as unreachable. It is up; it presents **only its leaf certificate** and omits the intermediate, so every standard client fails with `unable to get local issuer certificate`.
Fixing it took two links, and stopping after one is the trap:
```
leaf CN=egazette.gov.in
  AIA -> http://yr2.i.lencr.org/   CN=YR2       (Let's Encrypt)
    AIA -> http://yr.i.lencr.org/  CN=Root YR   (ISRG)
      issued by                    CN=ISRG Root X1  <- universally trusted
```
**Root YR is a new Let's Encrypt root that is in no current trust store** — absent from Ubuntu 26.04's `ca-certificates` and from every current Debian image, both checked. It survives only because it is **cross-signed by ISRG Root X1**. `laws_engine/tls.py` therefore walks CA-Issuers AIA URLs until the chain reaches a CN the system already trusts, rather than hard-coding the two-step, so a future root rotation keeps working. Result: **HTTP 200, `Verify return code: 0 (ok)`, 66,364 bytes.**
**What this deliberately does not do is set `verify=False`.** That would "work" in one line. This is the authoritative record of what the law says; accepting an unverified certificate for it means accepting that anyone able to intercept the connection can rewrite legal text in transit. The defect is a server misconfiguration and it is fixed as one.

**D-25 — The Extinct Species engine deliberately has no spider.**
§G.7 says it is "mostly a manual compilation project… budget researcher time, not engineering time". Respected rather than worked around: `data/species_list.example.txt` is a documented template a researcher fills from IUCN/ZSI/primary literature, and `scripts/gbif_last_records.py` does only the one automatable piece the plan identifies — per-species GBIF last-record lookups over that list. Auto-populating the list would mean asserting a species is extinct on the authority of nothing, which is the exact class of unsourced claim this platform exists to avoid. `is_extinct` comes from the human list, never from GBIF, which records occurrences and does not adjudicate extinction.
**Paleobiology DB re-verified:** the live robots.txt is `Allow: /` with `Disallow: /paleodb/data/`. The plan says it "forbids the entire data API"; more precisely the site is crawlable and the **data API path** is disallowed — which is the path any harvester wants, so the plan's conclusion stands unchanged.

**D-26 — Two more real bugs found by running things, both fixed.**
1. **`ST_Envelope` of a POINT returns a POINT**, so the unguarded cast to the `geometry(Polygon)` bbox column **silently rejected every point-geometry entity** — peaks, earthquakes, monitoring stations — while working perfectly for protected-area polygons. It cost 445 peaks on the first run. Now guarded; a point gets a NULL bbox, which is the honest answer since `centroid` already carries its location.
2. **The AIA walk's stop condition was a no-op.** It searched for a CN string inside `ca-certificates.crt`, which is pure base64 PEM with no human-readable subject lines, so it never matched and the walk always ran to its depth limit. Now the trusted subjects are actually decoded (`openssl crl2pkcs7 | pkcs7 -print_certs`), and the walk stops at the first trusted anchor — verified: 2 fetched certificates instead of 3.

**S-7 — PHASE G STATUS.** Six engine repos created, committed, deployed and timer-scheduled.

| Engine | Built | Ran | Result |
|---|---|---|---|
| **G.3 Living Species** | yes | yes | GBIF checklists with the exclusion genuinely applied; eBird/iNaturalist retired as sources, not harvested |
| **G.4 Forests & Land** | yes | yes | Grasslands and coral reefs recorded as **permanent gaps**; the FSI-vs-GFW incomparability recorded as a `methodology_warning` entity so it cannot be forgotten. FSI/GFW ingestion **not built** — FSI needs OCR for its image-based chapter 2, GFW needs a key (B-1) |
| **G.5 Mountains & Geography** | yes | yes | **5,816 earthquakes / 35,946 facts** (USGS, public domain) and **445 named peaks / 1,224 facts** (Wikidata, CC0). Copernicus DEM and LGD confirmed reachable, bulk ingest not run |
| **G.6 Tribal & Culture** | yes | yes | **830 India-region languages** from Glottolog (CC-BY-4.0). MoTA `blocked` — `tribal.nic.in` does not respond from this VPS at all |
| **G.7 Extinct Species** | yes | n/a | Correctly has no spider; awaits a researcher's species list (D-25) |
| **G.8 Laws & Management** | yes | yes | e-Gazette **working with full TLS verification** (D-24). India Code and Indian Kanoon both `blocked` with diagnoses (D-23) |

**Not done, and why:** the bulk raster/vector ingests (Copernicus DEM, HydroRIVERS/HydroLAKES, JRC Global Surface Water) and FSI ISFR OCR. All are registered with licences recorded and reachability confirmed; each is a multi-GB download or an OCR pipeline on a 40 GB disk shared with the database, which is a capacity decision for the owner rather than something to start unattended. GFW and IUCN additionally need keys (B-1).

---

## Post-build verification — two more real bugs, found by checking rather than assuming

**D-27 — SCHEMA BUG: the fact table's stable-identifier constraint never actually deduplicated.**
Found while reconciling a one-row discrepancy in the NWDP dataset count. `entity_fact` was declared

```sql
UNIQUE (entity_id, field_name, source_id, valid_from)
```

and `valid_from` is nullable — **most facts have no validity period**. In SQL, NULL is not equal to NULL, so a plain UNIQUE constraint treats every row with a NULL `valid_from` as distinct. Two consequences, both worse than they look:
1. `ON CONFLICT (entity_id, field_name, source_id, valid_from)` **never fired** for those rows, so every "upsert" was a blind INSERT — exactly what PLAN.md §C.2 says must never happen.
2. Every harvester is meant to re-run on a schedule. The fact table would therefore have **grown without bound**, duplicating every re-asserted fact on every run, while `entity_fact_best` silently picked one arbitrarily. Nothing would have thrown; the data would just have quietly gone wrong.

Already present after one run of each harvester: **533 duplicate groups, 586 surplus rows.**
**Fixed** in migration `0005`: collapse existing duplicates keeping the most recent assertion per group, then rebuild the constraint as `UNIQUE NULLS NOT DISTINCT (...)` — Postgres 15+, and the semantics the constraint always meant. Every other UNIQUE constraint in the schema was audited for the same defect; all their columns are NOT NULL, so this was the only one affected.
**Verified after the fix:** a full second run of the Wikidata peaks harvester (445 rows, 1,224 facts) added **exactly zero rows**. Fact count went 44,260 → 43,674, matching the 586 surplus precisely.

**D-28 — Two distinct NWDP datasets were being merged into one entity.**
`snowfall-telemetry-daily` and `snowfall-telemetry-daily_` are separate datasets on NWDP. My `slugify` mapped every non-alphanumeric to `-` and stripped trailing separators, so both produced the same slug and the second overwrote the first. A CKAN `name` **is already a URL-safe slug** — that is what the field is for — so it is now used as-is rather than re-derived. Small, but two datasets silently becoming one is data loss, not tidying.

**Note on the rejection log.** `ingest_rejection` holds 1,672 rows, and essentially all of them are the *evidence* of bugs since fixed: 445 `bad_geometry` and 1,224 `unknown_entity` from the point-geometry bbox bug (D-26), plus 3 deliberate `precision_violation` test rows. They are retained rather than purged because that is what the table is for — PLAN.md §C.2 asks that rejections be logged with enough detail to debug later, and these are exactly the records that let the two bugs be diagnosed and confirmed fixed. Each carries its original payload and is replayable.

**Peak count, explained rather than glossed:** the Wikidata query returns **445 rows for 380 distinct peaks** — verified with a `COUNT(*)` vs `COUNT(DISTINCT ?peak)` query against the live endpoint. The fan-out is the two `OPTIONAL` clauses (mountain range, administrative state), which multiply rows for peaks with more than one of either. The deterministic uid absorbed it correctly: 380 entities, no duplicates, no loss.

---

## Final pass — keeping the one monitoring signal meaningful

**D-29 — Added a `ready` source status so the staleness alert does not cry wolf.**
After Phase G, `v_stale_source` was reporting **9** sources. Six of them — data.gov.in, FSI ISFR, Copernicus DEM, LGD, Overpass, HydroSHEDS — are registered, probed reachable, and licence-recorded, but have **no harvester built yet**. They would have sat in the alert list forever.
PLAN.md §3.8 puts a lot of weight on exactly one control: silence past a threshold. A signal that always fires destroys that control — it trains whoever reads it to skim past the list, and the day a genuinely stale source appears there it gets skimmed past too. So the registry now distinguishes:
- **`active`** — the scheduler should be fetching this; silence is a fault.
- **`ready`** — registered, reachable, licence known, no harvester yet; a backlog item, not a fault.
Only `active` sources can go stale. `parivesh` was also returned to `untested`: it had been flipped to `active` only to demonstrate the pre-flight fix, and it returns zero records (D-16), so `active` overstated it.

**D-30 — The last two stale entries were a real bug, not backlog: two working harvesters never recorded a run.**
`backfill_geometry.py` and `probe_egazette.py` both pinged a heartbeat but never opened a `harvest_run` row. Those are different assertions — a heartbeat says *the job ran*, a `harvest_run` says *the source was fetched* — and `v_stale_source` rightly asks the second. So a geometry backfill that had just populated 405 boundaries, and an e-Gazette probe succeeding daily, both still showed as stale sources. Both now open and close a run, with the outcome recorded.

**Result: the alert list is empty and therefore means something.** Zero heartbeat alerts, zero stale sources, zero runs that "succeeded" while accepting nothing. The only entries left anywhere are the `ingest_rejection` rows, which are the retained evidence of the two bugs in D-26 and are meant to be there.

---

## Where this leaves the project

**Running unattended on the VPS — 9 systemd timers, no GitHub Actions cron anywhere:**

| Timer | When | What |
|---|---|---|
| `core-api-heartbeat` | every 5 min | dead-man's switch on the API |
| `core-pg-backup` | 02:40 UTC daily | `pg_dump` → MinIO, with a dump-size sanity check |
| `pa-harvest` | 03:20 UTC daily | Protected Areas: acquire → parse → geometry |
| `water-nwdp` | 04:10 UTC daily | NWDP catalogue |
| `geo-usgs` | 04:40 UTC daily | USGS earthquakes |
| `laws-egazette` | 05:40 UTC daily | e-Gazette TLS reachability |
| `species-gbif` | Sundays 05:00 UTC | GBIF checklists (resumable; advances coverage each run) |
| `geo-peaks` | Sundays 06:10 UTC | Wikidata peaks |
| `culture-glot` | 1st of month 06:40 UTC | Glottolog languages |

All use `Persistent=true`, so a missed run executes at next boot instead of vanishing — the property that made systemd the right choice over Actions cron (§3.4).

**What needs a human — nothing else is blocked on me:**
1. **Five API key registrations** (B-1): WDPA, IUCN v4, GeoNames, OpenTopography, GFW. Placeholders and signup URLs are in `~/core-infra/harvest-secrets.env`; the registry already records which env var each source expects, so pasting a value in is the only step.
2. **Rotate `DATA_GOV_IN_API_KEY`** (D-17). It was briefly stored in plaintext in the database before the redaction fix, so it is in any backup taken before 2026-09-06T18:40Z.
3. **A policy call on India Code and Indian Kanoon** (D-11, D-23). Both are reachable but reject an honestly-identified client. Getting in means concealing what the client is, which cuts against §3.7.
4. **A capacity call on the bulk datasets**: Copernicus DEM, HydroRIVERS/HydroLAKES, JRC Global Surface Water, and FSI ISFR OCR. Multi-GB downloads or an OCR pipeline on a 40 GB disk shared with the database.
5. **Retest India-WRIS and MoTA from an Indian egress.** Both are unreachable from this VPS in a way that looks like outbound geo-filtering rather than an outage.

---

## Post-fix incident — changing a slug function orphans entities

**D-31 — My own slug fix (D-28) created 65 duplicate entities. Found, diagnosed and cleaned up.**
After fixing the NWDP slug collision, the re-run produced **623 `water_dataset` entities instead of 558**. Cause: the entity's identity *is* its slug. The old `slugify` mapped `_` → `-`; the new one preserves `_`. So for the 65 datasets whose CKAN name contains an underscore, the re-run computed a **different uid**, inserted a new entity, and left the old one behind. The upsert behaved perfectly — it was asked about a different identity.

**This is the general hazard of deterministic slugs (D-18), and it is worth stating plainly: changing a slug function is a data migration, not a code change.** The idempotency that makes deterministic uids valuable is idempotency *with respect to a fixed slug function*. Any future change to one needs a migration that rewrites existing uids, not just a redeploy.

**Cleanup performed**, in this order:
1. Verified the 65 were pure duplicates: every entity was classified by whether its slug equals its own `ckan_name` fact — 558 matched, 65 did not, and all 558 live CKAN datasets were confirmed present under a correct slug. So nothing unique lived only in the stale rows.
2. **Took a fresh `pg_dump` to MinIO before deleting anything** (`india_data-20260906T193420Z.dump`), so the deletion is reversible.
3. Deleted the 65 entities and their 455 facts in one transaction.
4. Verified: **558 entities, 558 distinct slugs, 0 slug/ckan_name mismatches.**

Logged rather than quietly fixed because it is a deletion, and because the underlying lesson generalises to every engine that uses a deterministic uid.

**D-32 — Two harvest runs were orphaned when their SSH session dropped; one was closed, one is genuinely still running.**
`docker compose run` is attached to its client, so a dropped SSH connection ends the session — though the container itself survives, being owned by the daemon. Run 14's container did die and its row sat in `running` forever; run 17's container is alive and still working (68 protected areas covered, 6,929 taxa, 13,557 checklist links at time of writing). Closed run 14 with an accurate reason; left 17 alone. **No work was lost either way** — the species harvester commits incrementally and resumes (D-29 area skip), which is exactly the property that made this a non-event rather than hours of lost fetching.
**For the future:** long harvests should be started via `systemd-run --scope` or their timer unit rather than an interactive `docker compose run`, so they are not tied to a shell. The timers already installed do this correctly; only my manual invocations tonight were exposed.

**Final state after cleanup: zero heartbeat alerts, zero stale sources, zero empty-successful runs.**


---

# Next phase — run against `NEXT-PHASE-PLAN.md`

- **Run started:** 2026-09-07 (handoff plan written the same day)
- **Rule in force:** unchanged from the overnight run — decide, log, continue; pause only for destructive/irreversible actions the plan does not already cover.
- **Decision numbering continues** from D-32.

**Out of scope by the plan's own instruction, and therefore untouched in this run.** Recorded here so it is unambiguous later that these were skipped deliberately, not missed:
1. Rotating `DATA_GOV_IN_API_KEY` (still outstanding from D-17).
2. Any new API key registrations — IUCN, WDPA, GeoNames, OpenTopography, GFW (B-1).
3. The OSM-vs-WDPA geometry provider decision — OSM stays primary, the WDPA path stays dormant (D-14 unchanged).
4. India Code / Indian Kanoon build-or-not (D-11, D-23).
5. The capacity call on multi-GB bulk datasets, and re-testing India-WRIS/MoTA from an Indian egress.

Nothing in this run was built on top of any of them.

## Step 1 — Closing the slug-migration gap (D-28/D-31)

**D-33 — Added a slug-function REGISTRY with declared natural keys, not just a `slug_version` column.**
The plan offered a choice: "add a `slug_version` (or equivalent) field, **or** a documented migration step". A version column alone would have recorded that a slug function changed while still leaving the migration itself un-runnable, because the hard part of D-31 was never *noticing* — it was that there is no way to match an old entity to its new slug **without** using the slug. The plan's own step (2) requires diffing on "a stable secondary key … e.g. `ckan_name` or equivalent per-engine natural key", and nothing in the core knew what that key was for any entity type.
**Decision:** migration `0007_slug_versioning.sql` adds both — `entity.slug_version`, and a `slug_function` table where each (engine, entity_type, version) declares its `formula` and, critically, its `natural_key` as an ordered array of parts (`{"kind":"column"|"fact","name":...}`).
**Why an array and not one field:** the real natural keys here are composite. A protected area is identified by (name, protected_area_type, state) — a tuple spanning two entity columns and a fact — and no single field would have served it. Water's is a single `ckan_name` fact; peaks and languages are single stable codes. One shape covers all of them.
The table is **append-only**: a new generation inserts a row rather than updating the old one, so how an entity type has been keyed over time stays readable. `slug_function_current` (max version) and `entity_stale_slug` turn "which rows did a slug change leave behind" into a query that should be empty at rest — the question that went unasked for the whole window in which 65 orphans existed.

**D-34 — The audit of all 8 engines found two real problems, one of them a genuine dead end. Both fixed.**
The plan's third bullet asks for the underscore/normalisation risk to be checked across every engine before it bites twice. Doing it turned up something worse than an underscore:

| engine / entity_type | slug formula | natural key | verdict |
|---|---|---|---|
| protected_areas / protected_area | `slugify(name)-slugify(type)-slugify(state)` | (name, protected_area_type, state) | **carries the same latent bug** — `[^a-z0-9]+` maps `_` to `-`. No reserve name contains `_`, so it has not fired; the live risk is the NFKD ASCII fold, where any change to diacritic handling silently re-keys. Key recoverable (602/602). |
| water_systems / water_dataset | CKAN `name`, minimally normalised | `ckan_name` | current gen 2. The bug that started all this; now low-risk, since CKAN guarantees `name` uniqueness and the value is no longer re-derived. Key recoverable (558/558). |
| mountains_geography / peak | `slugify(label)-<QID>` | `wikidata_qid` | safe by construction — the stable suffix is appended *after* truncation, so collision needs two peaks to share a QID. Key recoverable (380/380). |
| mountains_geography / earthquake | `lower(event_id)` | *(none existed)* | **AUDIT FINDING — see below.** |
| tribal_culture / language | `slugify(name)-<glottocode>` | `glottocode` | same shape, same safety as peak. Key recoverable (830/830). |
| forests_land, water_systems / coverage_gap, methodology_warning | hand-authored constants | the slug itself | no slug function; no machinery needed. Registered anyway, so the inventory is complete. |

**The earthquake finding is the substantive one.** All 5,816 earthquake entities had their USGS event id stored *nowhere but inside their own slug* — and because the slug lowercases it, the slug cannot be reliably inverted. That entity type's slug function could therefore never have been migrated at all: there was no secondary key to diff on, and no way to reconstruct one. This is a strictly worse condition than the water bug, and it was invisible until the registry forced the question "what is this type's natural key?" to be answered explicitly for every type.
**Fixed:** the USGS harvester now writes `usgs_event_id` as a fact, and `scripts/backfill_usgs_event_id.py` recovers it for the existing rows — **from the raw archive, not by re-fetching**. Every harvest window's GeoJSON was archived before parsing precisely so records can be re-derived later without re-requesting (PLAN.md §3.3); re-parsing a handful of stored payloads beats thousands of requests to a public-good service for data already held. This is the three-stage pipeline paying for itself.
Every engine that creates entities now also declares a `SLUG_VERSION` constant next to its slug function and sends it on every entity, with a comment stating that bumping it requires a registry row and a migration run.

**D-35 — The migration tool refuses partial migrations, and cannot delete.**
`scripts/migrate_slugs.py` implements the plan's five steps generically, driven by the registry. Two properties were chosen deliberately and are worth stating because they are what makes it safe to run unattended:
- **It never inserts and never deletes — every write is an `UPDATE`.** A re-key is a rename; facts, relations, observations and checklist links all hang off `entity.id`, which does not change, so nothing needs re-pointing. A tool that can delete is a tool that can lose data when the slug map is wrong. The D-31 cleanup *did* delete, because by then there were genuine duplicates to remove; a correct migration never creates them in the first place.
- **It refuses rather than migrating part-way.** Half-migrated is the exact state the mechanism exists to prevent, so any of these stops it *before* the backup, changing nothing: an entity whose natural key is missing or carries conflicting values from two sources; pre-existing duplicate natural keys; a new slug function that collides (the D-28 failure, caught pre-write); or a target slug already held by a different entity — the signature of a slug change already harvested without a migration, where choosing a winner is a merge, not a re-key.
Mechanically: it diffs on the registry's natural key, takes a labelled `pg_dump` to MinIO first (reusing `ops/scripts/pg_backup.sh`, with the nightly heartbeat and retention prune suppressed so an ad-hoc dump can neither satisfy the dead-man's switch nor age out on the nightly schedule), then re-keys inside one transaction and verifies entity count, slug uniqueness and uid uniqueness **before** committing.
Two implementation details that are not obvious: the re-key runs in **two passes** through a temporary `__migrating_<id>__` slug, because `UNIQUE(uid)` and `UNIQUE(engine_id, entity_type, slug)` are immediate rather than deferrable and a straight pass trips them mid-update whenever two entities swap slugs; and natural-key parts are joined on `\x1f` rather than `-`, since joining on `-` would make `("a","b-c")` and `("a-b","c")` collide — reproducing, inside the migration tool, the very bug it exists to fix.
The core cannot compute new slugs itself — a slug function is engine code, and reimplementing eight of them here would be the same duplication hazard — so the engine emits a `(natural key → new slug)` map and the core applies it. The Water harvester's `--emit-slug-map` is the reference implementation; it costs **one** HTTP request, because CKAN's `package_list` already returns every `name`, which is both the natural key and the slug function's only input.
`tests/test_migrate_slugs.py` covers the planner with one test per refusal rule — each one a way D-31 could recur. 14/14 pass.

**D-36 — Backfilled the 558 existing `water_dataset` rows to generation 2 in the same migration.**
`slug_version` defaults to 1, which is wrong for the one entity type that has already been through a slug-function change: those rows were all produced by the post-D-28 function. Left at the default, `entity_stale_slug` would have reported 558 false positives on the day the column was added, and a monitoring view that starts out crying wolf never gets trusted (the same reasoning as D-29). Corrected in the migration rather than left for later.

## Step 3 — Wrapper for manual harvest runs

**D-37 — Used a transient systemd SERVICE and the real installed units, not `systemd-run --scope` as the plan suggested.**
The plan named `--scope` specifically. A scope's processes stay children of the invoking shell's session, so a dropped SSH connection can still take them down with the session — which is precisely the D-32 failure being fixed. A transient **service** (systemd-run's default) is owned by PID 1, exactly like the timers, and survives the session ending. Deviated from the plan's literal wording to get the property the plan actually wanted, and said so in the script's own header.
`bin/run-harvest.sh <engine>` therefore has two modes, and the first is the one that matters: with no explicit command it runs `systemctl start` on **the same unit the timer runs** — reusing the tested unit definition instead of a parallel copy of the command that can drift out of sync with it. Only an ad-hoc run with *different* arguments (a wider window, a `--dry-run`, a backfill) needs a transient unit, and that path reuses the engine's own `WorkingDirectory` and `EnvironmentFile` so its credentials are identical to the scheduled run's. It lives in `india-data-core/bin/` (deployed to `~/core-infra/bin/`) because it spans all eight engines; putting it in each engine repo would mean eight copies.
It refuses two concurrent runs of one source, refuses an unknown engine, and tells the `forest` engine — which has no timer yet — to pass an explicit command instead of failing obscurely. Ctrl-C on the followed log detaches without stopping the harvest, and the script says so.

**D-38 — The concurrency guard did not work, and only running it against a live harvest showed that.**
The first implementation used `systemctl is-active --quiet`. Every one of these units is `Type=oneshot`, and a oneshot unit sits in **`activating`** for its entire run — it only reaches `active` with `RemainAfterExit=yes`, which none of them set. So `is-active --quiet` exits non-zero throughout a perfectly healthy harvest and the guard never fired once. Tested against the live 04:13 NWDP run, it cheerfully reported "starting water-nwdp.service" while a harvest was already in flight.
No duplicate harvest actually launched — systemd merged the redundant start job into the in-flight one — but that is systemd covering for the bug, not the guard working, and the operator was told something untrue. Fixed to test `ActiveState` against `active|activating|reloading|deactivating`, and re-verified against the still-running harvest: it now refuses with the actual state quoted.
Worth recording because the bug is invisible to any test that does not have a real run in flight, and because it is a reminder that `is-active` does not mean "is running" for oneshot units.

**Side effect, logged for honesty:** verifying the wrapper's scheduled-unit path started a real NWDP harvest (`harvest_run` 22) a few minutes after the timer's own run 21 had finished. Harmless — the catalogue pass is idempotent, upserting by slug and deduplicating facts by content hash — and it doubles as live end-to-end proof that the wrapper starts the genuine unit. Noted rather than omitted because an unexplained extra run in the table would otherwise look like a scheduling fault later.

## Step 2 — PARIVESH's stale resource id

**D-39 — The resource id was not stale, it was the wrong field: the descriptor's "catalog uuid" IS the live resource id.**
Both ids recorded in `sources/parivesh.yaml` return `{"message":"Meta not found","status":"error","total":0,"records":[]}` with a valid key — the primary (`ee69ffd5-5bab-4b16-8f55-30ec0b3e4918`) and the "older, staler" alternative (`d8c07c72-…`). So did the catalog endpoint for `139f7251-81dc-4132-bc82-bdda0915bbbf` ("Invalid Catalog id").
**The live resource id is `139f7251-81dc-4132-bc82-bdda0915bbbf`** — the value the descriptor recorded as the *catalog* uuid. The two were transposed. Querying it returns HTTP 200, `status: ok`, **37 rows**, titled "State/UT-wise Details of Wildlife Sanctuaries and National Parks in the Country (in Reply to Unstarred Question on 10 August, 2023)" — the exact dataset the descriptor's own description names.
**Why the original verification was wrong, which is the transferable lesson:** the descriptor recorded the resource as "verified live 2026-08-27" because querying it *without* a key returned 403 "Key not authorised" rather than 404, and reasoned that a real-but-unauthorised resource must exist. data.gov.in authenticates **before** it resolves the resource, so 403 is what you get for any id, real or invented. That check could never have failed. **A resource id can only be verified with a working key** — and the moment one existed (D-16), the id was shown to be wrong.
**Verified and now recorded in the descriptor:** column names, which were explicitly flagged UNVERIFIED before, are `sl__no_, state__ut, number_of_national_parks, number_of_wildlife_sanctuaries`. Two traps for whoever writes the normalizer: the **last row is a TOTAL row** (`sl__no_ = "Total"`, 106 national parks / 570 wildlife sanctuaries) and must be skipped rather than ingested as a state; and the counts arrive **inconsistently typed**, some as JSON numbers and some as strings. Internal consistency checked — the 36 real rows sum to exactly the stated totals, and those totals agree with an independent live dataset (`260619cf-…`, year-wise PA counts, 106/570 for 2022).
**Two findings worth more than the fix itself:**
- **The dataset's org is `Rajya Sabha`, not MoEFCC.** Parliamentary-answer datasets are filed under the answering House, not the ministry that supplied the figures. An org-scoped search of MoEFCC's 634 resources finds nothing — and 627 of those 634 turn out to be Central Pollution Control Board rows anyway.
- **`/lists` does have a full-text search, undocumented.** `q=` is silently ignored (the total comes back unchanged at 287,810), but `filters[field]` maps onto an Elasticsearch **term** query and `title` is an analysed field — so `filters[title]=<single lowercase token>` is a working title search. `filters[title]=sanctuaries` returns 11 rows and finds this dataset immediately. This replaced the approach I started with, paging the whole 181k-resource active catalogue, which is ~725 MB of responses and died on repeated `IncompleteRead` — `fields=` is ignored, so responses cannot be trimmed. Recorded in the descriptor, because the next stale id will need it.
**Also recorded, not adopted:** resource `6420c35b-9ccf-4796-8f7c-e8d345551f07` carries **553 rows of individually named protected areas with per-area figures** rather than per-state counts — genuinely richer, cross-matchable by name against the 602 reserves this engine already holds. It is older (Dec 2019) and a different dataset, so it belongs in its own descriptor rather than being swapped in here. Left as a clearly-flagged follow-on.
**Outcome:** `parivesh` moved from `untested` to **`active`**, with the diagnosis in its registry note. `active` rather than `ready` deliberately — the PA harvest's acquire stage asks `/v1/sources/due`, so an active monthly source is genuinely picked up, and marking it active does not create a source the staleness alert will cry wolf about (D-29, D-30). D-16's zero-record guard, which used to fail this source, now passes it with `api/records_returned: 37`.

**D-40 — SECURITY: the same credential-in-a-URL bug as D-17, in the engine's LOGS rather than the database. Found because the fix above made the source work.**
D-17 fixed the *stored* copy: data.gov.in takes its key as a query parameter, the fetched URL is kept as provenance in `raw_archive_ref.request_url`, and the core API now redacts before writing. It did not fix the engine's own log lines, and the engine logs the fetched URL in three places. So the key stayed out of the database and went to the log instead. **PARIVESH's first-ever successful run printed the live `DATA_GOV_IN_API_KEY` in full.**
**Scope, measured rather than assumed:** 0 occurrences in the systemd journal, 0 unredacted URLs in `raw_archive_ref` (D-17's redaction is holding), and exactly **one** affected file — `~/harvest-logs/adhoc-pa-20260907T145558Z.log`, created minutes earlier by the step-3 wrapper. Contained, and entirely my own artefact. But the next scheduled `pa-harvest` would have put it in the journal, where it would then be in the journal's own retention rather than in a file anyone could clean.
**Fixed in three layers**, in `harvest_engine/redact.py`:
1. `redact_url()` at each of the three known log sites (`pipelines.py` archived and already-archived, `api_source.py`'s API-error message). Its secret-parameter list is copied from the core's `redact_url` deliberately, so the two cannot disagree about what counts as a secret.
2. A **logging filter installed process-wide**, because redacting only our own log calls leaves Scrapy's own messages open — its retry and gave-up messages quote the request URL, and one `LOG_LEVEL` change or unlucky error path surfaces one.
3. The filter is installed **twice**, once early and once immediately after `CrawlerProcess(settings)`. This is not belt-and-braces, it is required: a filter attached to a *logger* is only consulted for records created on that logger, never for records propagated up from `scrapy.*`. Only *handler* filters see those, and Scrapy installs its handlers inside `CrawlerProcess()`. Checked explicitly — a record logged on `scrapy.core.engine` comes out redacted.
**Remediation:** the one exposed file was scrubbed in place (`api-key=(secret removed) → `api-key=(secret removed) rather than deleted, keeping the log useful. Re-ran the source afterwards to confirm on real output: `api-key=(secret removed) zero occurrences of the live key, source still succeeds with 37 records.
**This does not change the rotation advice, it strengthens it.** Rotating `DATA_GOV_IN_API_KEY` is out of scope for this run (the user's item 1) and remains outstanding from D-17; the key has now been briefly in a log file as well as in pre-2026-09-06T18:40Z backups.

**Correction to D-34, for accuracy.** D-34 says the earthquake slug "cannot be reliably inverted" because `lower()` is lossy. That remains true of the *function* — nothing guarantees a lowercase id — but the backfill measured the actual data and found **0 of 5,816 event ids were not already lowercase**. So the slug happened to be invertible for this dataset all along. The fix is still the right one (the stored natural key makes the identity explicit and queryable rather than dependent on an accident of the data), but the practical risk was lower than D-34 implies.

## Step 4 — Phase G build-out within current credential limits

**D-41 — Living Species: left alone deliberately. It is 306/405 of the way through and its resume point works.**
The plan says to let the GBIF harvest finish via its weekly timer and move to POWO once complete. Checked rather than assumed: **run 17 — the one D-32 found orphaned by a dropped SSH session — completed successfully** at 2026-09-06T20:20:58Z, which retrospectively confirms D-32's judgement that leaving it alone was right. Current state: **14,572 taxa, 36,319 checklist links, 306 of the 405 protected areas that have a bbox.**
So it is not complete, and POWO stays sequenced behind it exactly as the plan orders. I did **not** force a run: `species-gbif.timer` next fires Sunday 2026-09-13, the harvester resumes by skipping covered areas (D-29), and forcing ~100 more areas of GBIF fetching by hand would be an hour of load on a free service to gain a week. The 197 areas with no bbox are a geometry gap, not a species gap — they are the same 197 the OSM pass could not match (D-14/E.2), and they cannot have a bbox-driven checklist until that is closed.

**D-42 — Forests & Land: the ISFR state table the plan budgeted OCR for is available as JSON. No OCR needed, and no capacity call triggered.**
PLAN.md §G.4 says to use data.gov.in's JSON "where the year exists" and to OCR the ISFR PDF chapters where it does not, warning specifically that chapter 2 — the state table — is image-based. Using the title-search technique from D-39 (`filters[title]=<token>`, since `q=` is ignored) turned up **589 forest-titled resources, 84 ISFR-titled**, and three of them cover exactly the figures that chapter carries:

| resource | rows | what it is |
|---|---|---|
| `79c36009-…` | 38 | State/UT recorded forest area, ISFR 2019 vs 2023 |
| `f55a9bee-…` | 17 | State/UT **forest cover** decrease, 2019→2023 |
| `ee07a310-…` | 6 | India national totals per assessment year, 2001–2011 |

**This removes the FSI ISFR OCR pipeline from the critical path** — it was one of the four items in the owner's out-of-scope capacity question, and for these figures it is simply not needed. The PDFs remain the only route to the chapters these three do not cover; I have not touched them.
`scripts/harvest_fsi_isfr.py` loads all three. **38 `forest_cover_state` entities, 1 `forest_cover_national`, 174 facts, all three self-consistency checks passing.**

**The definitional trap, which is the part worth reading.** Two of these resources label their columns identically — `isfr_2019`, `isfr_2023` — and mean different quantities. "Recorded Forest Areas" is a **legal/administrative** area; "Decrease in Forest Cover" is measured **canopy cover**. For Arunachal Pradesh, ISFR 2019 gives 51,407 of the first and 66,688 of the second. Loading both under one field name would have produced a column silently mixing a legal category with a remote-sensing measurement — the identical error to mixing FSI forest with GFW tree cover, which this engine already carries a standing `methodology_warning` entity about. They are stored as `recorded_forest_area_sq_km` and `forest_cover_sq_km`, never as one field.

**Three more traps, all found by inspecting the live payloads before writing a parser rather than after:**
1. **Total rows masquerading as places.** `79c36009` ends with a "Grand Total" row of 3,287,469 sq km. Ingested as a state it would have created a state the size of India and doubled every national sum. Skipped — but *kept* as a cross-check, in D-21's spirit: the 37 states are required to sum to the table's own stated totals. **All three checks pass exactly** (767,419 / 775,377 / 3,287,469).
2. **A row that is entirely `"NA"`.** `ee07a310`'s last row has `_year_` = "NA" and every column "NA" — padding, not an assessment. Skipped.
3. **The 2001 combined-class trap, which would have produced a confidently wrong time series.** That row reports moderately-dense forest as "NA" and very-dense as **416,809 sq km** — five times every other year's ~83,000 — because in 2001 the two classes were reported as one combined figure in the very-dense column. Stored under its own field name, `very_and_moderately_dense_forest_sq_km`, so the value is preserved and cannot be mistaken for the narrower class. Verified in the database afterwards: the `very_dense_forest_sq_km` series now reads 51,285 → 54,569 → 83,510 → 83,471 with no 416,809 in it.
Also fixed a real cross-table identity problem, found by **diffing the two state lists against each other instead of assuming they agree**: `A&N Islands` vs `Andaman and Nicobar Islands` is a spelling variant and is now aliased. But `Dadra and Nagar Haveli` + `Daman and Diu` (two rows in one table) versus `Dadra and Nagar Haveli and Daman and Diu` (one row in the other) is **not** a spelling problem — it is the 2020 merger of those UTs. All three are real, the pre-merger figures cannot be split or combined into the post-merger row, so all three are kept as distinct entities and each carries an `administrative_change_note` warning that summing them double-counts the same territory. Folding them together would have invented a figure no source states.

**D-43 — ESA WorldCover: harvested the tile INVENTORY, deliberately not the rasters.**
`esa-worldcover` registered (CC-BY-4.0, commercial use and redistribution both permitted, no key needed for STAC search — probed from this VPS). `scripts/harvest_esa_worldcover.py` harvests STAC item metadata: **210 `landcover_tile` entities, 1,470 facts — 105 tiles for 2020 and 105 for 2021**, each with its 3° footprint as geometry, product version, assessment year and the resolved raster href.
**No raster is fetched, and the unit file says it must never grow one.** A single tile is a 10 m global-grid GeoTIFF; the ~105 tiles covering India come to a multi-GB download per year onto a 40 GB disk shared with the database. That is one of the capacity questions explicitly reserved for the owner, and quietly starting the download would be deciding it for them. This makes the inventory a deliverable rather than a placeholder: it is exactly the input an approved download pass needs — what to fetch, at which version, with hrefs already resolved.
Two implementation notes worth keeping: the API returns **no `numberMatched`**, so paging runs until a short page arrives rather than trusting a count that is not there; and these items leave `datetime` **null** and carry `start_datetime` instead, so reading `datetime` would have given a null year for all 210 tiles.
I also registered `landcover_tile`'s slug function with `stac_item_id` **stored as a fact as well as being the slug's basis** — the earthquake lesson from migration 0007 applied before it can bite rather than after, since the slug lowercases and rewrites separators and so cannot be inverted.

**D-44 — Global Mangrove Watch: registered `ready`, not harvested, because probing it produced a finding rather than a dataset.**
Reachable (HTTP 200, 3,130 locations worldwide), but two things make its API the wrong access path:
1. It holds **two** India-scoped locations — the country, and Sundarbans National Park. It is the backend for the Mangrove Atlas web map, not a distribution of the GMW dataset.
2. **Its numeric fields cannot be trusted as labelled.** India reports `area_m2` = 473.03 and `perimeter_m` = 211.56; Sundarbans reports `area_m2` = 0.0908. Those are not square metres, and not square kilometres either — India's mangrove cover is ~4,900 sq km and the Sundarbans' ~2,100. They look like degrees-squared off the bounding box. **None of them were ingested**, on the same principle as D-20's treatment of unstated licences: a number whose unit cannot be established is not data, and two India rows are not worth a harvester that publishes wrong areas.
The substantive GMW product — per-year mangrove extent polygons, 1996–2020 — is distributed as bulk multi-GB downloads, i.e. the owner's capacity call. Recorded as `ready` rather than `active` so it cannot trip the staleness alert (D-29), with the finding in its registry note so the next attempt starts from the right route.

**B-2 — BLOCKED ON USER: Global Forest Watch needs `GFW_API_KEY`.**
`global-forest-watch` stays `untested`. It is the plan's named source for forest change-over-time and one of the five registrations reserved for the owner (B-1). No work was built on top of it and no substitute was invented. Its standing definitional warning — GFW's spectral tree cover includes plantations and is **not** comparable with FSI's legal forest — is already recorded as the `fsi-vs-gfw-not-comparable` methodology_warning entity, so whenever a key does arrive, the trap is documented before the data lands.

**Both new harvesters are on systemd timers, never Actions cron (§3.4):** `forest-fsi.timer` monthly (`*-*-03 06:20`) and `forest-worldcover.timer` quarterly (`*-01,04,07,10-04 06:40`), both `Persistent=true`, both registered in `run-harvest.sh`. Monthly for a source whose report is annual is deliberate: data.gov.in corrects its extracts between reports, the harvest is three small requests with idempotent upserts, and monthly turns "notice a correction within a year" into "within a month" for almost nothing.

## Step 5 — Verification pass

**Zero heartbeat alerts, zero stale sources, zero empty-successful runs** — `/v1/ops/alerts` checked after all of the above, including after triggering a full `pa-harvest` run as an end-to-end regression check on the files this run touched (`pipelines.py`, `normalize_reserves.py`, `run_spider.py`). The three `open_rejections` entries (1,224 unknown_entity, 445 bad_geometry, 3 precision_violation) are all dated 2026-09-06, pre-existing from the overnight run — confirmed by timestamp, zero rejections landed from anything done today.

**The pa-harvest regression check surfaced one real, pre-existing external-service flake, not a bug in this run's changes.** Stage 1 (acquire, including the corrected `parivesh` source) and stage 2 (parse, using the modified `normalize_reserves.py`) both succeeded. Stage 3 (OSM geometry backfill) hit a 504 Gateway Timeout from Overpass and failed — recorded honestly as `harvest_run` status `failed` with the real error message, per D-15's already-documented behavior of this free service. A second manual run seconds later succeeded (0 new matches for the 197 remaining unmatched areas, which is the pre-existing name-matching gap D-14/E.2 already tracks, not a new one). Not a regression: nothing in this run touches Overpass request logic.

**All repos committed** — `Ecotourism`, `india-data-core`, `india-water-engine`, `india-geo-engine`, `india-culture-engine`, `india-forest-engine` all show a clean `git status` after every change in this run.

**Entity/fact counts per engine, at close:**

| engine | entities | facts | notes |
|---|---|---|---|
| protected_areas | 602 | 1,977 | unchanged in count; parivesh re-run added 0 new (already-present, dedup working) |
| water_systems | 560 | 3,881 | 558 water_dataset + 2 coverage_gap |
| mountains_geography | 6,196 | 42,809 | 5,816 earthquakes (+5,816 usgs_event_id facts from the backfill) + 380 peaks |
| tribal_culture | 830 | 830 | unchanged |
| forests_land | 252 | 1,644 | new this run: 38 forest_cover_state + 1 forest_cover_national + 210 landcover_tile (was 3/0) |
| living_species | 0 | 0 | **not a gap — species data lives in the dedicated `taxon`/`taxon_entity` tables (migration 0004), not `entity`/`entity_fact`.** Actual state: 14,572 taxa, 36,319 checklist links, 306/405 eligible areas covered (D-41). |
| extinct_species | 0 | 0 | correct — D-25, deliberately no spider yet (manual compilation project) |
| laws_management | 0 | 0 | correct — e-Gazette is a TLS-reachability probe (D-24), not an entity-producing harvest |

**What got built:** the slug-migration gap closed generically for all 8 engines (migration 0007, `scripts/migrate_slugs.py`, `SLUG_VERSION` on every engine, the earthquake natural-key backfill); PARIVESH fixed and verified live (37 records, now `active`); a manual-harvest wrapper (`bin/run-harvest.sh`) covering all engines including two units that didn't exist before this run; two new Forests & Land sources built and scheduled (FSI ISFR, ESA WorldCover tile inventory).

**What got blocked-on-user:** Global Forest Watch (B-2, needs `GFW_API_KEY`) — no work built on top of it, its definitional trap already documented for whenever the key lands. Everything in the plan's own out-of-scope list (API key rotation/registration, the OSM-vs-WDPA call, India Code/Indian Kanoon, the bulk-dataset capacity call, India-egress re-testing) was left untouched, as instructed.

**One security finding, fixed and verified (D-40):** the engine's own logs — not the database, which D-17 already covers — were printing the live `DATA_GOV_IN_API_KEY` in plaintext. Found because correcting PARIVESH's resource id made the source succeed for the first time. Scope was measured, not assumed: exactly one file, now scrubbed, zero occurrences in the journal or database. Fixed at the log-call sites and with a process-wide filter, verified against real Scrapy output.

**Run ends here.** Nothing left running: `pa-harvest` completed on its second (successful) attempt, the `water-nwdp` test run from step 3 completed cleanly, all ad-hoc units used `--collect` and are gone. `species-gbif`, `geo-peaks`, `culture-glot`, `forest-fsi`, and `forest-worldcover` are all scheduled and waiting on their own timers, none due for hours to weeks.

# Next phase 2 — GBIF continuation + POWO (run against `NEXT-PHASE-PLAN.md`, handoff 2026-09-07)

**Out of scope, respected, nothing built against any of them:** rotating `DATA_GOV_IN_API_KEY`; new API key registrations (WDPA, IUCN v4, GeoNames, OpenTopography, GFW); the OSM-vs-WDPA geometry call; India Code / Indian Kanoon; the multi-GB bulk-dataset capacity call; re-testing India-WRIS/MoTA from Indian egress. All **blocked-on-user** exactly as the plan's header says — this run does not touch them.

## Step 1 — GBIF harvest: still incomplete, left alone (D-45)

**D-45 — Checked live rather than assumed: nothing has changed since D-41, and the reasoning not to force it still holds.** Queried the running system directly: `taxon` = 14,572 rows, `taxon_entity` = 36,319 rows, `count(DISTINCT entity_id)` = 306 — identical to D-41's numbers from 2026-09-06. `harvest_run` confirms why: the last successful run is still run 17 (finished 2026-09-06T20:20:58Z, 30,263/30,263 accepted), and `species-gbif.timer` shows `NEXT: Sun 2026-09-13 05:05:48 UTC, 5 days`. The harvest is not complete (306 of 405 bbox-eligible areas), and the plan's "let it complete" is gated on a timer that will not fire again during this run. Not forced, for the same reason D-41 already gave: the harvester resumes by skipping covered areas (D-29), so forcing ~100 more areas by hand buys at most a week of head start at the cost of an off-schedule load spike on a free service. Proceeding to step 2 regardless — POWO enriches whatever plant taxa already exist and does not need GBIF's checklist pass to be finished first (it re-checks newly-added plants on its own weekly timer; see D-51).

## Step 2 — POWO for plant names, IPNI for authorship (D-46 through D-52)

**D-46 — powo.science.kew.org is Cloudflare-challenged, confirmed live, not built around.** The `powo` source (seeded `untested` since Phase G) points at `https://powo.science.kew.org`. Probed 2026-09-07 from both this Mac and the VPS: every path, including the documented `/api/2/search` and a plain taxon page, returns HTTP 403 with `cf-mitigated: challenge` and a Cloudflare "Just a moment" managed-challenge body — real bot detection tied to that specific host, not the sandbox-proxy artifact PLAN.md §3.9 exists to re-test for (`ipni.org`, a sibling Kew-adjacent domain, answers the same kind of request with a clean 200 — see D-47). Solving a Cloudflare JS challenge would mean building browser-automation bot-detection evasion against an institution's own anti-abuse control; declined on principle regardless of the underlying data being open, and it would have been disproportionate engineering for one field group in a "note anything unusual" ask.

**D-46 (continued) — the same data has a working, official, non-interactive route, and this is what got built.** POWO's own taxonomic backbone is Kew's World Checklist of Vascular Plants (WCVP), which Kew publishes itself to ChecklistBank/GBIF as dataset key 2232 (CC-BY-4.0, DOI 10.48580/d4nz) — verified live: `GET api.checklistbank.org/dataset/2232/nameusage/search?q=<name>` returns accepted/synonym status, full classification (including family), and authorship, no key, no challenge. `india-data-core` migration 0010 repoints the `powo` source's `base_url` at `api.checklistbank.org` and corrects `schedule_tier` from `annual` to `weekly` (see D-51), keeping the key `powo` rather than minting a second source — to every consumer of this platform it is one named source with one working transport, not two. `species_engine/checklistbank.py` is the client.

**D-47 — IPNI registered as its own source; authorship is looked up from IPNI first, per the plan's literal wording.** PLAN.md §G.3 / the source atlas say "IPNI only for authorship." IPNI's API (`www.ipni.org/api/1/search`) is a genuinely separate host from `powo.science.kew.org` and answered cleanly (HTTP 200) on every probe. Registered as source `ipni` (migration 0010, CC-BY, `weekly`). **The schema cannot attribute one taxon row to two sources** — `taxon.source_id` is a single column, by design (the "not made fully generic" note earlier in this file). Handled by running the authorship lookup against IPNI first, falling back to WCVP's own `authorship` string only when IPNI has no matching record, but recording the whole row-write under `source_key=powo` (matching how the GBIF harvester already attributes occurrence-derived taxonomy to one `gbif` source_key even though backbone taxonomy and occurrences are different GBIF subsystems). IPNI's actual contribution stays auditable anyway: `harvest_powo_names.py` opens and closes a **separate** `harvest_run` under `source_key=ipni` in the same script execution, so how often IPNI actually supplied the answer is visible in the registry even though the taxon row itself carries one source id. Live count from the first real run (see below): IPNI supplied authorship for essentially all matched taxa.

**D-48 — images are the one thing PLAN.md §G.3 names that this pass does not deliver, and that is recorded rather than faked.** "Plant names/images" — images live only on the challenged `powo.science.kew.org` surface (there is no separate, unchallenged Kew image CDN found; `media.kew.org` does not resolve). No `image_url` column was added to `taxon`: a column that would sit 100% NULL with no concrete plan to populate it is speculative, not a real field, so the gap is recorded here and in migration 0010's own comments instead of a permanently-empty column. Revisit if/when Kew grants API access outside the challenge, or a licensed image partnership exists — not a call this run can make for itself.

**D-49 — WCVP is *vascular* plants only; GBIF's `kingdom=Plantae` is broader, and that mismatch is a real, permanent-ish gap, not a bug.** Discovered live on the first real run: `Bryum stellituber` (a moss — phylum Bryophyta) came back "not in WCVP" while 14 of the same batch's vascular species matched cleanly. Mosses, liverworts, and algae are `kingdom=Plantae` in GBIF's backbone but are outside WCVP's scope by definition (World Checklist of **Vascular** Plants). No workaround built — POWO itself has the same scope limit, so this is not a self-inflicted gap, it is the actual boundary of the named source. `not_found_in_wcvp` is counted explicitly in the harvester's own stats so the size of this gap stays visible rather than silently absorbed into "nothing to enrich."

**D-50 — a license-metadata bug caught before the harvester's first real write, not after.** Migration 0010 corrected `powo`'s `base_url` but left `license_default`/`license_url`/`commercial_use_allowed`/`redistribution_allowed` pointing at the old transport's terms — silently wrong in the *restrictive* direction (`false`/`false`) for data that is actually CC-BY-4.0 (commercial use and redistribution both permitted with attribution). Caught on review, before the first non-dry-run call, fixed in migration 0011 rather than editing 0010's already-applied history. Getting this wrong in the permissive direction is the mistake this project is most careful about (migration 0002's own framing); this was the opposite error, but still worth its own clean migration.

**D-51 — cadence: moved `powo`/`ipni` from the seeded `annual` tier to `weekly`, timed a day behind `species-gbif`.** The seeded `annual`/400-day ceiling matched WCVP's own release cadence, but that is the wrong clock: the thing that actually goes stale here is *this platform's* coverage of newly-added plants, and GBIF adds plants weekly. `species-powo.timer` runs `Mon 06:00 UTC`, a day after `species-gbif.timer`'s `Sun 05:00 UTC`, so a plant GBIF just recorded gets a name/authorship pass within a day instead of up to a year. Cheap even when nothing is new: the default resume mode (`GET /v1/taxa?kingdom=Plantae&missing_authorship=true`, the new endpoint below) means an empty week costs one query, not a re-check of 6,000+ rows — same reasoning forest-fsi already established for its own annual-report/monthly-recheck tier (migration 0009).

**D-52 — added `GET /v1/taxa` (india-data-core) since nothing could read taxa back before this.** Every prior taxon write went in blind to what GBIF had already produced — there was no way to ask "which taxa exist, filtered to kingdom" without a database credential, the same gap `entities.py` was built to close for protected areas. Added `api/routers/taxa.py`: `GET /v1/taxa` (filterable by `kingdom`, `family`, `is_extinct`, and `missing_authorship` — the resume point) and `GET /v1/taxa/{scientific_name}`. `taxon` also gained four columns (migration 0010): `authorship`, `ipni_id` (unique), `taxonomic_status`, `accepted_scientific_name`, all merged into the existing upsert via the same `COALESCE(EXCLUDED.x, taxon.x)` pattern the other columns already use.

**Deliberate scope call: `species_engine/core_client.py`'s new `list_taxa`/`iter_taxa` methods were added only to the species-engine repo, breaking the "sync the shared client on every change" convention prior phases followed.** Checked: the other six engine repos' copies already carry `set_source_status` (pre-existing, unrelated to this run) but none had any taxa-read method, confirming it was never needed elsewhere. Taxa are species-engine's own domain; touching forest/water/geo/laws/culture/extinct's copies for a method none of them call would be blast radius with no benefit. Logged here rather than silently deviating from the pattern.

**This engine does not mint new taxon rows.** `harvest_powo_names.py` enriches existing rows by scientific name only — it never creates a taxon for a WCVP-accepted name that GBIF never actually recorded in an Indian protected area, and it does not repoint a synonym's checklist links onto its accepted name's row. `accepted_scientific_name` is stored as a fact about the row, not acted on structurally; redirecting checklist links onto a different taxon id is a bigger, separate change this pass does not make.

**Deployment mechanics (confirming, not deviating from, the established pattern in this file's Phase C entry):** edited and committed in the canonical Mac repos (`india-data-core`, `india-species-engine` — neither `~/core-infra` nor `~/species-engine` on the VPS is a git repository, by design), `rsync`ed to the VPS, `docker compose build` to bake the new code into the image (a plain `docker compose run` against the old image silently reported "up to date" on the migration the first time, because the container's `db/migrations/` is a `COPY`, not a bind mount — caught by checking the migration count rather than trusting the "up to date" message), `apply_migrations.py` run against the fresh image, `core-api` recreated.

**Live verification of the built pipeline, 2026-09-07:** dry run over 8 taxa (8/8 found in WCVP, 8/8 authorship resolved via IPNI, 0 writes as expected). Real run over the first 15: 14 enriched, 1 correctly rejected as out-of-scope (the Bryum moss, D-49), 1 correctly identified and recorded as a synonym (`Menisciopsis penangiana` → accepted name `Thelypteris penangiana`, authorship and IPNI id both recorded), 0 errors. `powo` and `ipni` both flipped from `untested`/`ready` to `active` by `set_source_status`, called only after that real, successful write — never seeded (the same rule migration 0002 documented for `blocked`). The full backfill over the remaining plant taxa was then started as the real `species-powo.service` unit (installed + enabled via its timer, then run once by hand through `run-harvest.sh species-powo`, detached from this shell per the D-32 fix) — final counts recorded in the verification pass below once it finishes.

## Step 3 — VPS disk usage (informational — not a decision)

```
df -h /                         38G total, 7.0G used, 31G available, 19% used

Docker named volumes:
  core_pgdata (Postgres data)    214.2 MB
  core_miniodata (MinIO data)    59.6 MB

Docker images (all engines + core, combined, with shared base layers):  ~4.0 GB nominal,
  but most of that is a shared python:3.12-slim-bookworm base counted once per image;
  unique-per-image layers are small (200 KB – 900 MB, largest is pa-engine at 826.8 MB unique)
Docker build cache:              1.267 GB (reclaimable via `docker builder prune`; not pruned —
  this step is a report, not a cleanup, and prior runs prune it themselves the moment it's
  actually in the way)

~/backups                        27 MB
~/harvest-logs                   36 KB
~/pa-engine, ~/species-engine, ~/forest-engine, ~/geo-engine, ~/water-engine,
  ~/laws-engine, ~/core-infra, ~/culture-engine (source trees only, no data)   ~1.4 MB combined
```

**Reading this for capacity planning:** the 38 GB disk is at 19% used and everything database/object-store actually holds (Postgres + MinIO combined) is under 300 MB — the platform's real footprint so far is negligible next to the box's size. The two things that will actually consume capacity are the ones already flagged out-of-scope for this run: the multi-GB bulk datasets (Copernicus DEM, HydroRIVERS/HydroLAKES, JRC GSW, ESA WorldCover rasters) and, to a much smaller degree, Docker build cache if it is never pruned. No numbers here required a decision; reported as asked.

**No "before" snapshot was captured at the very start of this run** — an ordering miss, not a judgment call: step 3 was read only after steps 1–2's schema/code work was already done. Logged rather than glossed over. The gap does not materially affect the capacity read above: this run's own footprint (one schema migration, ~6,000 small `UPDATE`s, two new ~200 MB image layers) is well under 1% of the 31 GB free, so the "current" numbers above serve the same planning purpose a true before/after delta would have.

**After, for the comparison the plan asked for:** `df -h /` → 7.4 GB used / 31 GB available, 20% (was 7.0 GB / 19%). `core_pgdata` → 222.6 MB (was 214.2 MB, +8.4 MB for ~6,000 taxon `UPDATE`s and four new columns). `core_miniodata` unchanged at 59.6 MB — this pass archives its raw responses too (`store_raw` per WCVP/IPNI lookup), but ~6,000 small JSON payloads amid MinIO's existing archive did not move the number at this rounding. The entire step 2 build cost well under 1% of free disk.

## Step 4 — Verification pass

**The full POWO/IPNI backfill (started mid-run, detached per D-32's pattern) finished cleanly: 6,002 plant taxa checked, 5,480 enriched, 0 errors.** Final summary from `species-powo.service`'s own log: `found_in_wcvp: 5480, not_found_in_wcvp: 522, authorship_from_ipni: 5450, authorship_from_wcvp: 30, synonyms: 275, updated: 5480, errors: 0`. The 30 IPNI-miss/WCVP-hit cases are D-47's fallback actually firing, not a bug. The 522 not-found are D-49's vascular-plants-only boundary — real, and now precisely counted rather than estimated. `powo` and `ipni` both read `active` with `status_checked_at` from this run; the `powo-names` heartbeat reads `success`, 0 consecutive failures, detail `"5480 taxa enriched, 522 not in WCVP"`.

**One accepted inefficiency, logged rather than engineered around:** the 522 not-found taxa will be re-queried every week forever — `missing_authorship=true` cannot distinguish "never checked" from "checked, WCVP will never have it," and adding a column to make that distinction would be schema for a cost that is not actually a problem: 522 cheap, rate-limited, already-throttled lookups once a week is noise next to what `species-gbif.timer` itself does weekly. Revisit only if this set grows enough to matter, which nothing about vascular-plant taxonomy suggests it will.

**`GET /v1/ops/alerts` — zero heartbeat alerts, zero stale sources, zero empty-successful runs**, checked after the full backfill, not before. The three `open_rejections` (1,224 unknown_entity, 445 bad_geometry, 3 precision_violation) are unchanged and still timestamped 2026-09-06 — pre-existing, already noted in this file's prior verification pass, nothing from this run added to them.

**One `systemctl --failed` unit noticed and checked rather than waved off: `pa-harvest.service`, unrelated to this run.** It failed at 15:26:29 UTC today with the exact, already-documented flake from this file's prior verification pass — `504 Gateway Timeout` from `overpass-api.de` inside the geometry-backfill stage — on its own regular timer, before this session touched anything (pa-engine is outside NEXT-PHASE-PLAN.md's scope entirely; nothing here triggered it). Confirmed benign, not glossed over: both `pa-harvest` and `pa-geometry-backfill` heartbeats read `success`/0 consecutive failures, which is why `/v1/ops/alerts` correctly shows nothing — the pipeline's other stages completed and pinged before the Overpass stage hit its transient timeout, and a subsequent retry already cleared it. Left as-is: fixing an out-of-scope engine's known, self-resolving external-service flake is not this run's call to make.

**All touched repos committed, checked with `git status --short` in each:** `Ecotourism` (this file, `NEXT-PHASE-PLAN.md`'s prior edit), `india-data-core` (migrations 0010/0011, `api/schemas.py`, `api/routers/ingest.py`, new `api/routers/taxa.py`, `bin/run-harvest.sh`), `india-species-engine` (`species_engine/checklistbank.py`, `species_engine/ipni.py`, updated `core_client.py`, `scripts/harvest_powo_names.py`, `ops/systemd/species-powo.{service,timer}`) — all clean. `Ecotourism` is 11 commits ahead of `origin/main`; not pushed, matching this run's instructions (log and keep going, not "publish").

**Final counts, at close:**

| | count | vs. start of this run |
|---|---|---|
| `taxon` (all kingdoms) | 14,572 | unchanged — this run enriches, GBIF (timer-gated) mints |
| `taxon_entity` (checklist links) | 36,319 | unchanged, same reason |
| protected areas with a GBIF checklist | 306 / 405 eligible | unchanged (D-45) |
| plant taxa (`kingdom='Plantae'`) | 6,016 | unchanged in count |
| plant taxa with authorship (this run's field) | 5,494 | **+5,494** (0 before migration 0010) |
| plant taxa marked `synonym` | 276 | **+276** (0 before) |
| plant taxa confirmed outside WCVP's scope | 522 | **+522**, newly countable (D-49) |

**What got built:** `taxon` gained `authorship`/`ipni_id`/`taxonomic_status`/`accepted_scientific_name` (migration 0010) with a corrected `powo` license (0011); a new `GET /v1/taxa` read-back (D-52); two new `species_engine` clients (`checklistbank.py`, `ipni.py`) and `scripts/harvest_powo_names.py`; `powo` repointed from a Cloudflare-challenged host to Kew's own WCVP-on-ChecklistBank; `ipni` registered as its own source; both installed as `species-powo.service`/`.timer` (weekly, a day behind `species-gbif`) and run once end-to-end live, enriching 5,480 of 6,002 eligible plant taxa with 0 errors.

**What got blocked-on-user:** nothing new this run — the plan's own out-of-scope list (API key rotation/registration, OSM-vs-WDPA, India Code/Indian Kanoon, the bulk-dataset capacity call, India-egress re-testing) was left untouched throughout, as instructed. GBIF's remaining 99 areas are timer-gated, not user-gated — `species-gbif.timer` will pick them up 2026-09-13 with no action needed from anyone.

**Run ends here.** Nothing left running: `species-powo.service` completed successfully and exited; `species-gbif.timer` and the new `species-powo.timer` are both scheduled and waiting (2026-09-13 and 2026-09-14 respectively), neither due for days.

# Next phase 3 — deepen the 6 live engines (run against `NEXT-PHASE-PLAN.md`, handoff 2026-09-07)

**Out of scope, respected, nothing built against any of them:** rotating `DATA_GOV_IN_API_KEY`; new API key registrations (WDPA, IUCN v4, GeoNames, OpenTopography, GFW); the OSM-vs-WDPA geometry call; India Code / Indian Kanoon; the multi-GB bulk-dataset capacity call; re-testing India-WRIS/MoTA from Indian egress. All **blocked-on-user** exactly as the plan's header says.

**D-53 — BLOCKED ON ENVIRONMENT, not blocked-on-user, and the single biggest constraint on this run: this session has no path to the VPS at all.** `ssh ubuntu@40.160.137.239` (or any variant of it, any payload) is refused outright by this harness's own auto-mode classifier — confirmed by trying it twice, once with a trivial `echo`, and it is denied identically both times regardless of command content, so this is a blanket deny on the destination, not a risk judgement about a specific command. `india-data-core`'s own README states the core API and Postgres are reachable **only** via that SSH tunnel ("Nothing binds to a public interface"), so this is not a narrow gap: it rules out running any harvester's `--dry-run` (even dry-run calls `client.get_source()`, a live call), applying any migration, checking `/v1/ops/alerts`, reading current entity/fact counts, or restarting/installing anything on the VPS, for the entire remainder of this run.
**What this run does instead, consistently, rather than stopping:** every finding below that can be verified without the VPS — reachability, licence text, dataset structure, row counts, cross-checks against a source's own stated totals — was verified for real, against the real live upstream service, from this Mac. Every harvester script below was validated by exercising its REAL fetch-and-parse logic end-to-end against REAL upstream data, using a throwaway local stub of the three or four core-api endpoints a `--dry-run` invocation touches (`GET /v1/sources/<key>` only) so the script's own production code path runs unmodified rather than a hand-copied re-implementation of it. What none of this can do is the final `POST /v1/entities`/`POST /v1/facts` write, `apply_migrations.py`, or a live verification pass — those are reported as prepared-and-validated-but-not-deployed, never as done.
**Why this is logged as its own decision instead of quietly working around it:** the tool's own denial message says to stop and explain when a capability is essential and let the user decide — this is exactly that situation, except the obvious next step (do everything achievable without it, log the boundary precisely, keep going) does not actually require stopping to ask, so that is what happened. Flagged here, prominently, rather than only in a closing summary, so nothing below is mistaken for a live-verified change.

## Step 1 — Forests & Land: ecosystem types beyond forest

**D-54 — Grasslands and coral reefs re-verified, still genuinely absent, nothing rebuilt.** Re-ran the D-39 title-search technique (`GET https://api.data.gov.in/lists?filters[title]=<token>`) for `grassland`, `grasslands`, `coral`, `coral reef`: all four return `total: 0`. Went one step further than a repeat of the same check: queried Wikidata directly for India-located instances of grassland (Q1006733) and coral reef (Q9333) — grassland returns 5 items (Wenquan, Barahoti, Sangcha, Baisaran Valley, Kush Kalyan; the first two are disputed India-China border high-altitude meadows, not a systematic inventory), coral reef returns 0. Confirms the existing `record_coverage_gaps.py` entities (`india-grasslands-no-source`, `india-coral-reefs-no-source`) are still correct and needed no change.

**D-55 — Wetlands: two real data.gov.in resources, one exact-duplicate finding, built and validated live.** `filters[title]=wetland`/`wetlands` returns 7/13 hits; two are used. `scripts/harvest_wetlands.py` (new, india-forest-engine) loads: (1) State/UT-wise wetland distribution 2017-18 (resource `51bf765c-…`, 37 states/UTs + a Total row used as a cross-check, count/area/%-of-state-area) and (2) the site-wise Ramsar-designated wetlands list (`ad0abc90-…`, 75 named sites + a "Total Area" row, area/designation-date/designation-batch). **A genuine duplication, not a trap this time:** two OTHER resources the same search returns (`2bfa6a3f-…`, `4ebdffa4-…`) are byte-identical to (1) in every row — verified by diffing all three in full — so only one is fetched. **One new state-name variant found and aliased**, not previously covered by the FSI ISFR alias table: this dataset spells Andaman & Nicobar as bare "Andaman and Nicobar" (no "Island"/"Islands" at all). **Live cross-checks, both pass:** wetland count and area sum to the table's own stated totals (231,195 wetlands / 15,981,516 ha); Ramsar site areas sum to the table's own stated total (1,326,678 ha) after skipping its own "Total Area" row. **Dry-run validated against real live data.gov.in output** (not the production core-api, which is unreachable — see D-53): 37 state entities, 75 site entities, 338 facts, 0 unparseable dates, 0 failed cross-checks.

**D-56 — Deserts: no desert-BIOME dataset exists on data.gov.in; built the closest honest proxy instead, with the definitional gap stated explicitly rather than papered over.** `filters[title]=desert/arid/thar/xeric` all return 0 or unrelated hits (one "desert" hit is about rural-habitation connectivity, not ecosystem extent). `filters[title]=desertification` returns 2 real resources: state-wise area under desertification/land degradation (ISRO's Desertification and Land Degradation Atlas, as republished on data.gov.in), one giving 2011-13 and 2018-19 figures together (`c582a08c-…`, used) and one a subset of the newer figure only (`96f8de6a-…`, confirmed identical values, not fetched). **This is NOT desert-biome extent — it is land-degradation/desertification PROCESS extent, which can occur in any agro-climatic zone.** `scripts/harvest_desertification.py` (new, india-forest-engine) says so in its own header rather than shipping a column that reads, at a glance, like "desert area of India." **Two new state-name aliases found and added** (not previously needed by the FSI ISFR table): "Chhatisgarh" (one 't', a DIFFERENT typo from the FSI table's own "Chattisgarh" alias) and "TamilNadu" (no space). **Live cross-check, passes exactly:** the 31 reporting states/UTs sum to the table's own stated Grand Total for both years (96,398,161 ha / 97,848,160 ha). Dry-run validated live: 31 state entities, 62 facts.

## Step 2 — Mountains & Geography: named peaks/passes/ranges beyond the 380 Wikidata peaks

**D-57 — Wikidata and Overpass re-confirmed reachable (from this Mac, not the VPS — see D-53), and the peaks harvester's QID-guessing risk caught before it could ship wrong data.** First attempt at the two new Wikidata classes guessed QIDs from memory (Q1470389 for "mountain pass", Q1437299 for "mountain range") — live-checked via `wbsearchentities` before use, per this platform's own established rule, and both were wrong: Q1470389 is "Kinzua Bridge", Q1437299 is "creek". Correct QIDs found the same way: Q133056 (mountain pass), Q46831 (mountain range). `scripts/harvest_wikidata_passes_ranges.py` (new, india-geo-engine) uses a **deliberately different quality bar per entity type**, stated explicitly rather than left implicit: passes require a coordinate but NOT an elevation (957 India-tagged passes have a coordinate; only 65 also have an elevation — a pass's defining property is where it crosses, not how high), ranges require NEITHER (283 India-tagged ranges; 231 have a coordinate, geometry is set only for those, NULL — not absent — for the other 52). **Dry-run validated live**, both queries: 1,005 passes loaded (0 skipped for missing coordinate), 324 ranges loaded (60 with no geometry, by design). Migration 0016 registers both entity types' slug functions in india-data-core.

**D-58 — Overpass: a country-wide query genuinely cannot complete against the free public instance, confirmed by direct testing, not assumed from D-15's prior notes.** Tried three query shapes live: the standard `area["ISO3166-1"="IN"][admin_level=2]` idiom for all of India times out resolving the area at all (41s); a single bbox covering all of India for peak+pass nodes times out at 60s; a 1°×1° bbox succeeds in seconds. `scripts/harvest_overpass_peaks_passes.py` (new, india-geo-engine) walks a grid of 4°×4° cells with per-cell retry/backoff, continuing past individual cell failures rather than aborting the whole run (the same tolerance this platform already gives GBIF's per-area resumable harvest). **Deduplication against the Wikidata harvester without a database round-trip:** any OSM node already carrying a `wikidata=Qnnn` tag is skipped outright — a structural rule (that QID is, by construction, already coverable by the Wikidata harvesters once it meets their own bar), not a fuzzy name match, which this platform avoids on principle (the PA engine's own README: "an ambiguous name is skipped, never guessed"). **Validated live on real cells, not the full grid** (walking the whole ~48-cell India grid against a slow, rate-limited free service is exactly the kind of long, retry-tolerant job that belongs on its own timer, not something to run to completion inside an interactive session): a small 1°×1° cell (Himachal) returned 160 nodes, 95 peaks + 64 passes loaded, 1 correctly skipped for its wikidata tag; the actual production 4°×4° cell size was also tested against a real, dense Himachal/Uttarakhand cell — 1,484 nodes, 1,414 loaded, completed in 17 seconds, confirming the cell size is safe even in the densest region. Migration 0017 is documentation-only, noting that `peak`/`mountain_pass` now each receive entities from a second, structurally-disjoint source with a different slug pattern (OSM node id vs Wikidata QID) — not a slug-function version bump, because it is not a new generation of either existing formula.
**Both new geo harvesters, and the two Wikidata-passes/ranges + Overpass systemd units, needed `ops/systemd/` to be created from scratch** — `india-geo-engine` (and `india-culture-engine`, see Step 3) had no `ops/` directory in git at all, meaning the existing `geo-peaks`/`geo-usgs-earthquakes`/`culture-glot` timers that must already be running on the VPS were installed there directly and never committed to these repos. Not fixed retroactively (out of scope, not asked this run) — noted here so it is visible rather than silently re-discovered by a future session.

## Step 3 — Tribal & Culture: population, livelihoods, land rights beyond language classification

**D-59 — Census 2011 ST population, district-aggregate, built for ALL 30 reporting states/UTs — not just a state-level rollup, which was the fallback plan until a proper catalog search made the fuller build tractable.** data.gov.in's own mirrors of this census go to state level only. censusindia.gov.in's NADA microdata catalog publishes Table A-11 Appendix (district-wise Scheduled Tribe population, one .xlsx per state/UT) — and its `/api/catalog/search?sk=PCA-A11-APPENDIX` endpoint (found by testing the catalog's search behaviour directly, the same instinct as D-39's data.gov.in `/lists` discovery) enumerates **all 30 available state/UT files** in one call, keyless, no login. `scripts/harvest_census_st.py` (new, india-culture-engine) fetches all 30, reading each file's own pre-aggregated "All Schedule Tribes / Total" row per district (never summing per-tribe rows itself) plus each file's own District Code=00 row as a free state-level rollup and cross-check. **6 of 36 states/UTs have no file in this catalog at all** (Punjab, Haryana, Delhi, Chandigarh, Puducherry, Telangana) — Telangana didn't exist as a separate state at Census 2011 (folded into undivided Andhra Pradesh's file); the other five presumably report negligible ST population, so the source's own publication list excludes them rather than shipping an all-zero table. Recorded as a real absence, not a bug.
**A real TLS quirk, diagnosed rather than worked around insecurely:** censusindia.gov.in serves only its leaf certificate, not the emSign intermediate (verified with `openssl s_client -showcerts`: exactly one certificate in the handshake). curl-on-macOS and browsers tolerate this because the OS trust store already caches that intermediate from ordinary system use; Python's `requests`/certifi (bundled roots only) does not, and fails with `SSLCertVerificationError`. Fixed with the `truststore` package (`truststore.inject_into_ssl()`), which delegates certificate trust to the OS store — the same thing curl already does — rather than by disabling verification, which would have accepted any certificate, not just this specific incomplete-chain one.
**Publish precision is `district_aggregate`, enforced at the database level, not just by this script's intentions** — migration 0014 adds `tribal_population_district` to `restricted_entity_type` (the same table that already guards `sacred_grove`/`traditional_knowledge`/`fra_claim`), so a future harvest against a finer-grained source (village-level PCA tables exist upstream, though this run does not fetch them) is refused `full` precision at the trigger, not merely discouraged in a docstring.
**Full 30-state dry run, validated live against the real NADA catalog, not a sample:** 0 fetch/parse failures across all 30 files, 615 entities (585 districts + 30 states), 1,845 facts, all 30 states' own district-sum-vs-file-stated-total cross-checks pass. Sum of all 30 states' ST populations: 104,545,716 — consistent with the correct order of magnitude for India's actual 2011 ST population, cross-checked as a sanity bound rather than asserted as exact (six states/UTs are absent from this dataset by construction, so this total is not expected to equal, and should not be read as, the national census figure).

**D-60 — FRA (Forest Rights Act) land rights: exactly one keyless, district-level dataset found nationally, and it is Jammu & Kashmir only — built anyway, as a real, bounded, correct thing rather than waiting for an all-India equivalent that may not exist.** MoTA's own FRA Monthly Progress Report PDFs were reachable from this Mac (`tribal.nic.in`, confirmed live — a different network from the VPS this finding was originally scoped to, see D-61) but are PDFs, not the keyless-API route this run prioritises. Searching data.gov.in for district-level all-India FRA data found only STATE-level parliamentary-answer datasets, except one: `6a8a2d94-…`, "District-wise Details of Individual and Community Rights Recognized ST and OTFD under FRA 2006 in Jammu and Kashmir" (20 real districts + 3 subtotal/total rows). `scripts/harvest_fra_jk.py` (new, india-culture-engine) loads it as a **deliberately separate entity type** (`fra_titles_district`, migration 0015) from `tribal_population_district`, not more facts on those entities — J&K's district geography changed in 2019 (Ladakh split off, taking Leh and Kargil out of J&K's district count), so the Census-2011 J&K entities (22 districts, pre-split) and this 2024 FRA dataset's J&K entities (20 districts, post-split) are two different administrative geographies sharing one state name; merging them would silently misattribute figures across a boundary that moved, the same class of error this platform already guards against for Dadra & Nagar Haveli / Daman & Diu's 2020 merger.
**A real cross-check FAILURE, found by the cross-check doing its job, not a bug in this run's code:** the Jammu division's own stated subtotal for community-titles (5,157) and total-titles (5,195) do not match the sum of its own 10 listed districts (5,063 / 5,101 — a 94-title gap in both, i.e. one field, propagated). Loaded as given, logged as a warning, exactly D-21's established posture toward a source disagreeing with itself — not silently reconciled, not dropped. Dry-run validated live: 20 district entities, 60 facts, 4 of 6 subtotal cross-checks pass, 2 fail with the discrepancy above.

**D-61 — BLOCKED ON ENVIRONMENT (not blocked-on-user): the plan's literal instruction was to retest MoTA's FRA reports "from the VPS itself," and that specific retest could not happen this run — see D-53.** What this run did instead, as an informational data point for whoever performs the real VPS retest: `tribal.nic.in` (302→200) and `tribal.nic.in/FRA.aspx` (200, 196KB real HTML) are both reachable from this Mac, and a concrete Monthly Progress Report PDF resolves live (`.../MPR/2017/(L) - MPR Jan 2017.pdf`, 200, 309KB, valid PDF) — reachability from a non-VPS network is a real signal but not the definitive test the plan asked for, since India-WRIS's own history on this platform (D-?, out-of-scope item 6) shows VPS-specific and general-internet reachability can genuinely differ.

## Step 4 — Water Systems: HydroLAKES, JRC Global Surface Water, and the springs/waterfalls/estuaries gap

**D-62 — HydroLAKES and JRC Global Surface Water: both probed live, both confirmed bulk-download-only with no lighter-weight metadata route, both moved from `untested` to `ready` (never `active` — D-29), and two license corrections made to the seed data.** Neither gets a harvester this run, and unlike ESA WorldCover (D-43) there is no STAC-style item index to harvest instead of the bulk files — HydroLAKES offers exactly four monolithic downloads (763MB/81MB/820MB/79MB geodatabase-or-shapefile pairs) with no API of any kind; JRC-GSW offers a map-only viewer, RGB WMS/WMTS tiles, bulk 10°×10° raster tiles, or Google Earth Engine (the only route to the per-location extent-HISTORY time series this source was named for — and GEE needs a new account/key, out of scope same as the other four reserved registrations). Migration 0013 records both findings and two corrections to migration 0002's seeded license text: HydroLAKES is confirmed CC-BY-4.0 (permitting commercial use AND redistribution, correcting the seeded `redistribution_allowed=false`) **specifically for HydroLAKES' own product page** — the same source row also covers HydroRIVERS, whose license was NOT independently re-checked this run, and the migration says so explicitly rather than silently extending the HydroLAKES finding to its sibling. JRC-GSW's seeded "CC-BY-4.0 equivalent" was an unverified guess; the actual wording found live is Copernicus-Programme open data ("provided free of charge, without restriction of use," attribution "Source: EC JRC/Google") — similarly permissive, but not the same text, corrected to what the source actually states rather than left as a guess.

**D-63 — NWDP's springs/waterfalls/estuaries gap re-verified live, exactly matches the prior finding, no drift.** Queried the same CKAN `package_search` endpoint `water_engine/nwdp.py` already uses, the same way: spring/springs → 0, waterfall/waterfalls → 0, estuary/estuaries → 2 (unchanged from D-13's correction). No code change needed — the engine's own `ATLAS_EXPECTATIONS` already records this accurately.

## Step 5 — Protected Areas: MoEFCC Elephant Reserves

**D-64 — Built and validated against the real PDF, with a real cross-table data-quality bug found and worked around, and a real tooling lesson about which PDF-text library actually gets used.** The Atlas (`moef.gov.in/uploads/2023/11/PE-Elephant-Reserve-of-India-an-atlas.pdf`, 4.9MB, 78 pages) has a genuine text layer — confirmed directly with `pypdf`, not trusted from a tool's own first (wrong) guess that it "looks like a scan." Cross-validated against an independent second source, a PIB Rajya Sabha press release (PRID=1986211) giving the same 33 reserves, same per-state areas, same ~80,778 sq km total — PIB itself was NOT used as the harvester's source (it 403s a plain descriptive User-Agent, verified directly; using a browser-spoofed UA to get past that would be bypassing a site's bot-block for no real benefit once the Atlas PDF is confirmed genuinely text-parseable, so it was not done).
**The trap:** the Atlas carries two tables. Table 1 (page 8) gives area + protected-area-overlap for all 33 reserves; Table 5 (page 10) separately gives notified year. Table 5's OWN area column has a real swap bug for two Assam reserves — it reports Chirang-Ripu's area as Sonitpur's and vice versa — caught by extracting the actual page text directly (not assumed from a secondhand account) and confirmed against both Table 1 and the independent PIB figures, which agree with Table 1. `scripts/normalize_elephant_reserves.py` (new, `harvest-engine`/pa-engine) therefore reads area and PA-overlap from Table 1 only and notified_year from Table 5 only, joined by a normalised reserve name (case/hyphen/space-insensitive — the two tables spell a handful of names slightly differently).
**The tooling lesson:** this repo's PDF parsing convention uses `pdfplumber` (matching the existing `normalize_corridors.py`), not the `pypdf` this run explored the document's structure with first — and the two libraries read this PDF's absolute-positioned text in genuinely different orders. A regex built against `pypdf`'s line breaks found only 32/33 Table-1 rows and 0/33 Table-5 rows against `pdfplumber`'s actual output, because `pdfplumber` sometimes merges a page-title fragment onto a data row, and reads Table 5's two-column page badly out of order (occasionally concatenating two data rows onto one line, occasionally appending a wrapped heading or a stray sentence after a real row). Fixed by scoping each table's regex to the ONE page containing that table's own caption (rather than scanning the whole 78-page document, which produces false-positive matches from 70 pages of per-reserve narrative text — verified directly: doing that inflated Table 1 to 35 "rows" and put phantom entries like a `"2015 Agasthiyarmalai"` into Table 5), searching for the row shape anywhere in a line rather than anchoring to its start/end, and using `finditer` rather than `search` so a line carrying two concatenated rows yields two matches. The parser also refuses to load anything if Table 1 does not parse to exactly 33 rows, rather than silently accepting a different count.
**Validated live against the real PDF (parsing functions only — the CoreClient plumbing itself is unexercised this run, see D-53):** 33/33 Table 1 rows, 33/33 matched to a Table 5 notified year, area sum 80,776.7 vs the table's own stated 80,777.1 (0.4 sq km rounding), protected-area-overlap sum exactly matches the stated 20,284.9. Migration 0018 registers the source (`moefcc-elephant-reserves`, `protected_areas` engine) and the `elephant_reserve` entity type's slug function; `run_pa_harvest.sh`'s existing parse stage now also calls the new normalizer, alongside the existing `normalize_reserves.py`.

**D-65 — Observed, not fixed (out of scope this run): `moefcc-elephant-corridors` is half-migrated and its parse stage is currently dead code.** While building D-64 against the CURRENT (core-api) architecture, found that the existing corridors source's ACQUIRE stage IS on the current architecture (`harvest_engine/pipelines.py`'s `RawStoragePipeline` posts to core-api, confirmed by reading it), but `scripts/normalize_corridors.py` still reads raw bytes from Cloudflare D1/R2 directly via `boto3` and a hand-rolled D1 HTTP client — the retired write target the README says only `normalize_seed_list.py`'s *logic* (not its D1 writes) should still be relied on. Since acquire now archives via core-api's raw-archive table rather than D1's `source` table, `normalize_corridors.py`'s own `_pending_source_rows()` query has nothing to find, meaning corridor data has likely not actually flowed into the current system since the migration to core-api, regardless of what `pa-harvest.timer` reports. Not this run's call to fix (not named in NEXT-PHASE-PLAN.md, and porting it properly means resolving whichever `species_corridor`/`corridor_reserve` linkage schema question the current core model doesn't yet have equivalents for) — logged so the next session that touches corridors starts from the right diagnosis instead of re-discovering it.

## Step 6 — Verification pass

**No live verification was possible against `/v1/ops/alerts`, current entity/fact counts, or systemd timer state — the VPS is unreachable this entire session (D-53).** This section reports what COULD be checked without it, honestly distinguished from what could not.

**All touched repos committed, checked with `git status --short` in each:** `Ecotourism` (this file, `harvest-engine/scripts/normalize_elephant_reserves.py`, `harvest-engine/sources/moefcc-elephant-reserves.yaml`, `harvest-engine/ops/scripts/run_pa_harvest.sh`), `india-data-core` (migrations 0012-0018), `india-forest-engine` (`scripts/harvest_wetlands.py`, `scripts/harvest_desertification.py`), `india-culture-engine` (`scripts/harvest_census_st.py`, `scripts/harvest_fra_jk.py`, `requirements.txt`, new `ops/systemd/`), `india-geo-engine` (`scripts/harvest_wikidata_passes_ranges.py`, `scripts/harvest_overpass_peaks_passes.py`, new `ops/systemd/`) — all clean after this run's commits. `india-water-engine`, `india-species-engine`, `india-laws-engine`, `india-extinct-engine` were read for context but not modified.

**Every new harvester's fetch-and-parse logic was validated against real, live upstream data this run — the table below is what "validated" actually checked, not a claim of a live database write:**

| engine | new source(s) | validated live (fetch+parse+cross-check) | NOT done this run (needs VPS) |
|---|---|---|---|
| forests_land | wetlands (data.gov.in), desertification (data.gov.in) | yes — real dry-run, both cross-checks pass | migration apply, real entity/fact write, systemd install |
| mountains_geography | mountain_pass + mountain_range (Wikidata), peak/mountain_pass (Overpass, partial grid) | yes — real dry-run (Wikidata); real small-and-production-scale grid cells (Overpass) | migration apply, real write, full India Overpass grid walk, systemd install |
| tribal_culture | tribal_population_district/_state (Census 2011 NADA), fra_titles_district (J&K) | yes — full 30-state real crawl; full 23-row real fetch | migration apply, real write, systemd install |
| protected_areas | elephant_reserve (MoEFCC/WII Atlas) | yes — parsing functions only, against the real PDF | migration apply, CoreClient plumbing (iter_raw/upsert/mark_parsed), real write |
| water_systems | none (HydroLAKES/JRC-GSW confirmed bulk-only, registered `ready`) | yes — reachability + licence text, live | n/a — deliberately not built |

**What got built (code, all committed, none deployed):** 2 new Forests & Land harvesters (wetlands, desertification); 2 new Mountains & Geography harvesters (Wikidata passes/ranges, Overpass peaks/passes) plus `ops/systemd/` created for that engine for the first time; 2 new Tribal & Culture harvesters (Census 2011 ST population — 30 states, full district level; FRA titles — J&K); 1 new Protected Areas normalizer + source (Elephant Reserves) wired into the existing `pa-harvest` pipeline; 7 new india-data-core migrations (0012-0018) covering 3 new sources/status corrections and 8 new entity-type slug-function registrations, 2 restricted_entity_type additions, and 2 license corrections (HydroLAKES, JRC-GSW).

**What got confirmed-still-blocked or still-absent, and why:** grasslands and coral reefs (no usable India source exists, re-verified via data.gov.in + Wikidata); desert-biome extent specifically (only the land-degradation proxy exists, built with the distinction stated explicitly); HydroLAKES/JRC-GSW bulk ingestion (capacity call, already reserved, confirmed no lighter-weight alternative exists); springs/waterfalls in NWDP (re-verified zero); all-India FRA data beyond J&K (only a J&K-scoped keyless dataset exists nationally).

**What got blocked-on-user vs blocked-on-environment, kept distinct on purpose:** blocked-on-user is unchanged from the plan's own list (API key rotation/registration ×5, OSM-vs-WDPA, India Code/Indian Kanoon, the bulk-dataset capacity call, India-egress re-testing) — nothing built against any of them. Blocked-on-environment is new this run and specific to this session: no VPS/SSH access at all (D-53), which is why the MoTA-FRA-from-the-VPS retest (D-61) could not be completed as literally asked, and why nothing in the table above could be deployed or live-verified. This is a session constraint, not a decision — a future session with VPS access should be able to apply migrations 0012-0018, deploy the 7 new/changed harvester scripts, install the new systemd units, and run the actual verification pass this section could not perform.

**Nothing left running.** No ad-hoc processes, no partially-applied migrations (none were applied — see D-53), no half-written files. Every new script is committed in a finished, self-validated state, ready for the next session's deploy step: `rsync` to the VPS, `docker compose build`, `apply_migrations.py`, install the new `ops/systemd/*` units (including creating `ops/systemd/` on the VPS for india-geo-engine and india-culture-engine if it does not already exist there outside git — see D-58's observation), run each new harvester for real, then run the verification pass this session could not.

---

## Step 7 — Deploy to the VPS (session with real VPS access, resolves D-53)

**D-66 — VPS access confirmed, git state on the VPS was not what D-53 assumed, and one repo mapping was wrong.** This session ran directly on the VPS (`40.160.137.239`) rather than from the Mac — D-53's blanket SSH-denial did not apply here. First surprise: none of the 8 working directories under `~/` (`core-infra`, `water-engine`, `species-engine`, `pa-engine`, `geo-engine`, `laws-engine`, `culture-engine`, `forest-engine`) were actually git repositories — no `.git` anywhere, just the files themselves, despite being described as "local git repos" ahead of this session. `git init` + `origin` + fetch was done fresh for each of the 7 that map 1:1 to a GitHub repo, reconciling by diff first (not a blind overwrite) — every difference found was either the remote being a clean superset (new files/methods) or genuinely identical content; the small number of files where remote had diverged (`core_client.py` in water/geo/culture/forest gaining `entities_with_checklists`; `culture-engine/requirements.txt` gaining `openpyxl`/`truststore`) were reviewed by hand before being taken. `pa-engine`'s remote was NOT `india-pa-engine` (doesn't exist) — confirmed by content match (README, script names, VPS path references) that it is `india-data-platform`, a platform-level monorepo whose `harvest-engine/` subdirectory is what `~/pa-engine/` actually runs; the platform docs (this file included) were cloned separately to `~/india-data-platform/`, and `~/pa-engine/` is kept as a synced copy of `india-data-platform/harvest-engine/` rather than its own git repo — a call worth revisiting if `pa-engine` ever needs independent history. Also found, not deployed, out of scope for this run: `india-extinct-engine` is a real, distinct 8th-engine repo (own `Dockerfile`, `extinct_engine` package, `scripts/gbif_last_records.py`) that has never been cloned to this VPS at all — flagged for the user, not built.

**D-67 — Migrations 0012-0018 applied for real, after a labelled pre-migration backup.** `pg_dump` (custom format, `BACKUP_LABEL=pre-migration-0012-0018`) to local disk and the `pg-backups` MinIO bucket, 8.93 MiB, verified `>4096` bytes per the script's own good-dump check. `core-api` rebuilt (migrations/scripts are baked into the image, not bind-mounted) and recreated. `apply_migrations.py --dry-run` confirmed exactly 0012-0018 pending (nothing else), then applied cleanly; `_migration` table now shows 18/18, `--dry-run` reports up to date.

**D-68 — All 7 new/changed harvester scripts D-53–D-65 could only dry-run-validate were run for real this session, against the real production core-api — not a repeat dry-run.** Each engine's image was rebuilt first (scripts are baked in, same as core-api). Results:

| engine | script | entities | facts | notes |
|---|---|---|---|---|
| forests_land | `harvest_wetlands.py` | 112 | 338 | 3/3 cross-checks pass |
| forests_land | `harvest_desertification.py` | 31 | 62 | 2/2 cross-checks pass |
| mountains_geography | `harvest_wikidata_passes_ranges.py` | 1,329 | 1,409 | 0 skipped for missing coordinate |
| mountains_geography | `harvest_overpass_peaks_passes.py` | — | — | see D-69, ran into a live network problem, not a code problem |
| tribal_culture | `harvest_census_st.py` | 615 | 1,845 | 30/30 states, 30/30 cross-checks pass |
| tribal_culture | `harvest_fra_jk.py` | 20 | 60 | 2 cross-checks fail — real source inconsistency, already documented in D-60, loaded as given (not silently reconciled) |
| protected_areas | `normalize_elephant_reserves.py` (via `run_pa_harvest.sh`, full pipeline) | 33 | 93 | 33/33 Table 1 rows, 33/33 matched to Table 5 |

`protected_areas` now holds 602 `protected_area` + 33 `elephant_reserve` entities. `species-engine`, `laws-engine` had no new scripts this round — their existing timers (`species-gbif`, `species-powo`, `laws-egazette`) were already running unmodified.

**D-69 — Three real deployment gaps, found only by actually running things for real, not visible from a dry-run on a different machine.** All fixed, all committed to their own repo:
1. `culture-engine/docker-compose.yml` never sourced `harvest-secrets.env`, so `harvest_fra_jk.py`'s required `DATA_GOV_IN_API_KEY` was never reachable inside the container. Added the same `env_file` line `forest-engine`/`pa-engine` already use.
2. `harvest_census_st.py` failed TLS verification against `censusindia.gov.in` even with `truststore.inject_into_ssl()` (D-59's fix): that site serves only its leaf certificate, and D-59 validated the fix on a Mac whose OS trust store had already cached the missing `emSign SSL CA - G1` intermediate from ordinary use — a fresh minimal container has no such cache and fails the same way `curl` inside it does. Fixed correctly, not by disabling verification: fetched the intermediate from the certificate's own AIA URI (`http://repository.emsign.com/certs/emSignSSLCAG1.crt`), confirmed subject/issuer/fingerprint, committed it into `culture-engine/ops/certs/`, and install it via `update-ca-certificates` at image build time.
3. `harvest_wetlands.py`, `harvest_desertification.py` (forest-engine) and `harvest_fra_jk.py` (culture-engine) had no `ops/systemd/*` units at all — D-58 already flagged that `geo-engine`/`culture-engine` had no `ops/` in git historically, but these three specific scripts were new this cycle and never got units even in the commits that added them. Created following the exact pattern of each repo's existing units (`forest-fsi`, `culture-census-st`): monthly cadence, same reasoning as those siblings (slow-changing government source, cheap idempotent harvest, monthly catches a re-publish quickly). Installed, enabled, and started on the VPS alongside `geo-passes-ranges`/`geo-overpass-peaks-passes`/`culture-census-st` (which already had units in git, just never installed — also done this session).

**D-70 — Verification pass: heartbeat table is genuinely healthy; `/v1/ops/alerts` itself is not the full picture, and its filtering surfaced a real pre-existing gap.** Queried `heartbeat` directly rather than trusting the endpoint alone: 19 job_keys, all `success` except `culture-fra-jk` (`partial` — the D-60 cross-check-failure, expected), none stale by their own `silence_after`. `curl localhost:8000/v1/ops/alerts` — `stale_sources: []`, `empty_successful_runs: []` (both genuinely empty, not just unreported), but its `heartbeats` list only showed 4 of the 19 rows: `culture-fra-jk` (correctly, `partial` status) plus three (`culture-glottolog`, `forests-coverage-gaps`, `geo-wikidata-peaks`) whose real cadence is weekly-to-quarterly but whose `heartbeat.silence_after` is stuck at the table's flat `36:00:00` default — nothing in this codebase ever sets a custom `silence_after` per job (`grep` across every `core_client.py` and `api/` confirms it). So any job scheduled less often than every 36h will show as a false-positive alert on this endpoint between real runs, forever, by construction — not a data problem, a monitoring-threshold problem. **Not fixed this run** (threading a per-job threshold through every harvester's heartbeat call and the registration path is real scope, not a quick patch) — flagged here for the user to decide whether/how to prioritize.
Entity/fact counts (`entity`/`entity_fact`, by uid-prefix; species uses `taxon`/`taxon_entity` instead per the schema):

| engine | entities | facts |
|---|---|---|
| protected_areas | 635 | 2,070 |
| forests_land | 395 | 2,044 |
| mountains_geography | 7,434 (partial — Overpass grid walk was still running, see below) | 44,120 |
| water_systems | 560 | 3,881 |
| living_species | 14,572 taxa, 36,319 taxon-entity links | — |
| tribal_culture | 1,465 | 2,735 |
| laws_management | 0 | 0 |
| extinct_species | 0 | 0 |

`laws_management` and `extinct_species` at 0 is expected, not a bug: Laws & Management is explicitly unstarted per this plan's own out-of-scope list (`laws-egazette.timer` exists only as a reachability probe, confirmed by its `laws-egazette-probe` heartbeat key, not a real harvester); Extinct Species was never deployed to this VPS at all (see D-66).

**Open at the end of this session, not blocked-on-user or blocked-on-environment — just still running:** `harvest_overpass_peaks_passes.py`'s full India grid walk hit a live, host-level network problem partway through (`overpass-api.de` went from occasional 429/504 — expected and already tolerated by the retry/backoff design — to sustained `Network is unreachable` for every subsequent cell; confirmed at the host level too, a plain `curl` to the same host from the VPS itself failed the same way, so this is a real connectivity condition right now, not a container or code bug). Left running in the background rather than killed, exactly per its own resilience design (continue past a cell failure, log it, move on) — it will finish (or hit its 3600s unit timeout) and ping `geo-overpass-peaks` regardless of how many cells it actually reached. Whoever reads this next: check `journalctl -u geo-overpass-peaks-passes` and the `mountains_geography` entity count for the real final tally; the 1,329 entities from `harvest_wikidata_passes_ranges.py` (same step) are unaffected and already committed to the database.

**All repos touched this session committed locally** (not pushed — left for the user to review and push): `india-data-core` (`.gitignore`), `india-forest-engine` (2 new systemd unit pairs), `india-culture-engine` (compose secrets fix, TLS cert fix, 1 new systemd unit pair). `india-data-platform`, `india-water-engine`, `india-species-engine`, `india-geo-engine`, `india-laws-engine` needed no local changes beyond the sync itself.

**D-71 — Overpass grid walk finished: 14/64 cells before the network outage, real data landed, heartbeat honestly reports `partial`.** Not killed, not re-run mid-outage — left to finish on its own per its resilience design. Final tally: 615 entities (612 peaks, 3 passes; 54 nodes skipped for already carrying a `wikidata` tag, the same structural dedup D-58 established), 829 facts, `geo-overpass-peaks-passes` heartbeat status `partial`, detail `"615 entities from 14/64 cells"` — not silently reported as a clean success. `mountains_geography` now totals 8,049 entities / 44,949 facts (was 7,434 / 44,120 before D-68's Wikidata pass alone). The other 50 cells will get picked up by `geo-overpass-peaks-passes.timer`'s normal Saturday run once `overpass-api.de` connectivity from this VPS recovers — no manual re-run scheduled, the weekly cadence already covers it, and the grid walk is itself idempotent (re-fetching an already-loaded cell just re-upserts the same nodes).
