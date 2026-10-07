# Ecotourism Atlas

Reserve-centric, citation-backed data atlas for all Indian protected areas —
National Parks, Wildlife Sanctuaries, Tiger Reserves, Biosphere Reserves,
Conservation Reserves, and Community Reserves. Every fact is sourced and
cited at the fact level. No booking, no tourism logistics, no state-level
general culture content.

This is a standalone project — separate GitHub repo, separate Cloudflare
account/deployment, separate D1 database from any sibling project (e.g.
sathyamangalam-atlas). Do not merge into or share infrastructure with
sibling repos.

## Architecture

```
harvest-engine/   Scrapy (Python) harvesting engine — one spider per source,
                   driven by YAML descriptors in sources/, scheduled via
                   GitHub Actions. Raw fetches go to Cloudflare R2 first
                   (raw-first, hashed by SHA-256 for idempotency);
                   normalized facts load into Cloudflare D1 with mandatory
                   per-fact provenance (source_url, retrieved_at, license,
                   confidence).
harvest-engine/tests/  Local D1/R2 stand-ins (real sqlite3 + a local-directory
                   S3-compatible server) so every spider/normalize script can
                   be run and verified against real fetched data before real
                   Cloudflare infra is provisioned. See tests/README.md;
                   `python tests/run_branch_test.py <branch>` runs one
                   branch's harvest + normalize end-to-end.
site/              Static frontend — Leaflet map over India, reserve detail
                   pages built from D1 data. No live API: the map reads a
                   static GeoJSON/JSON export generated from D1 at deploy
                   time (harvest-engine/scripts/export_geojson.py).
exports/           Generated GeoJSON/JSON output (build artifact, gitignored).
```

## Design rules

- **Raw-first**: every fetched response is stored in R2, unmodified, before
  any parsing or normalization happens.
- **Idempotent by hash**: raw content is deduped by SHA-256 before
  re-storing or re-processing.
- **Fail-one-continue-all**: one spider/source/record failing never halts
  the others — enforced at both the spider level (errback logging) and the
  orchestration level (`scripts/run_seed_list_harvest.py` runs each source
  in its own subprocess).
- **Config-driven sources**: adding a source means adding a YAML file under
  `harvest-engine/sources/`, not editing spider code. See
  `harvest-engine/sources/README.md`.
- **Fact-level provenance**: every row in D1 carries `source_id` (which
  itself carries `source_url`, `retrieved_at`, `license`) plus its own
  `confidence` rating. See `harvest-engine/db/migrations/`.

## Data hierarchy

RESERVE is the root entity. IDENTITY, ZONES, HYDROLOGY, FLORA, FAUNA,
THREATS, PEOPLE/TRIBE (reserve-specific only), and CORRIDORS all key off
`reserve_id`. Species, occurrences, threats, cultural names, and corridors
are cross-cutting join tables (many-to-many), not nested blobs — a species
page assembles reserves/occurrences/threats/local-names/corridors/media via
foreign keys. Full schema: `harvest-engine/db/migrations/0002_full_hierarchy.sql`.

## Milestone 1 — protected area seed list

Sourced from NTCA (Tiger Reserves) + WII's legacy gazette notification
database (National Parks, Wildlife Sanctuaries, Conservation Reserves,
Community Reserves) + a data.gov.in dataset for cross-checking (WDPA is
blocked for India and is not used). This backbone (`reserve` table: name,
type, state, coordinates, legal status) must be loaded into D1 before any
flora/fauna/culture harvesting begins — every later branch attaches via
`reserve_id`.

Status: sources verified live and parsers written + tested end-to-end
against real fetched pages (2026-08-27) —
`harvest-engine/sources/ntca-tiger-reserves.yaml` (58 Tiger Reserves) and
`harvest-engine/sources/wii-gazette-notifications.yaml` (35 state/UT
requests). See `harvest-engine/sources/README.md` for what was ruled out
along the way (two dead/content-free source URLs, one dead domain). Still
open: `scripts/normalize_seed_list.py` to turn parsed records into
`reserve`/`reserve_fact` rows doesn't exist yet, and Cloudflare R2/D1 need
provisioning — env vars are documented in `harvest-engine/.env.example`
but nothing is created yet.

## Species-data sources — GBIF, eBird, iNaturalist

`harvest-engine/sources/gbif.yaml`, `ebird.yaml`, `inaturalist.yaml` +
`scripts/run_api_spider.py` (iterates every reserve's bbox from D1, one
subprocess per reserve, fail-one-continue-all). eBird requires
`EBIRD_API_KEY` (see `.env.example`); a missing/revoked key fails that
source's run loudly rather than silently proceeding unauthenticated.

Status: sources are fetchable (raw response → R2, `source` row → D1), and
`harvest-engine/scripts/normalize_flora_fauna.py` now turns that raw JSON
into typed `species`/`species_reserve`/`occurrence`/`media` rows — species
deduplicated across reserves by scientific name, media stored only for
unambiguously-licensed images (downloaded into R2, not linked by bare
external URL — see `harvest-engine/sources/README.md` for the license
policy and reserve-linking rule). Tested end-to-end against a local D1/R2
stand-in with real fetched GBIF/iNaturalist responses (no EBIRD_API_KEY in
this environment, so eBird was tested against a fixture built from the
API's documented shape, not a live fetch). Still `workflow_dispatch`-only
in `.github/workflows/harvest.yml` (not the daily cron) pending a run
against real provisioned D1/R2, which don't exist yet — see Milestone 1
status above; flip to `schedule:` once that dry run is confirmed clean.

Also added in this pass: `reserve.bbox_min_lon/bbox_min_lat/bbox_max_lon/
bbox_max_lat` columns (`harvest-engine/db/migrations/0003_reserve_bbox_and_normalize_tracking.sql`)
— `scripts/run_api_spider.py` and the GBIF/eBird/iNaturalist YAML
descriptors already depended on these columns for per-reserve queries, but
no earlier migration had created them, so that fetch step could not
actually run against real D1. Backfilled from `centroid_lat/centroid_lon`
with a fixed ~0.15° pad as a placeholder box; replacing that with real
boundary-derived bboxes is separate follow-up work, not done here.

## robots.txt override fix (affects eBird, iNaturalist, Overpass)

Found and fixed while verifying Zones/Hydrology against live sources
2026-08-27: Scrapy's global `ROBOTSTXT_OBEY = True`
(`harvest_engine/settings.py`) had no per-source override anywhere in the
codebase, but iNaturalist (`Disallow: /*?` — blocks every query-string
URL, confirmed live) and eBird (`Disallow: /` for the whole host, confirmed
live) both blanket-disallow their own sanctioned API surface in robots.txt.
This meant `RobotsTxtMiddleware` was silently filtering out every
iNaturalist/eBird request before it was ever sent — **neither source had
actually fetched live data through the real spider pipeline**, despite the
README previously describing iNaturalist as tested against "real fetched
... responses." (GBIF's robots.txt has no such conflict — verified, no
change needed there.)

Fixed by adding an explicit, documented `respect_robots_txt: false` YAML
field (default `true` — every source stays obedient unless it opts out),
read by `harvest_engine/spiders/api_source.py` and passed to Scrapy per
request via `meta["dont_obey_robotstxt"]`. Set on `sources/inaturalist.yaml`
and `sources/ebird.yaml` (each with its live-verified justification in
`robots_txt_verified`) and on `sources/overpass.yaml` (Hydrology — see
below). Verified working end-to-end: a real live iNaturalist fetch for
Bandipur's bbox now succeeds (HTTP 200, stored to R2/D1) where it
previously would have been silently dropped.

## Zones (core/buffer areas)

`scripts/normalize_zones.py` re-parses the raw NTCA Tiger Reserve table
HTML already fetched for the seed list (`sources/ntca-tiger-reserves.yaml`
— no new source) into `zone` rows (`zone_type='core'`/`'buffer'`,
`area_sq_km`; see `db/migrations/0004_zone_area.sql`).

Verified live 2026-08-27 that this is the only structured (non-scanned)
zone data available from current sources: the NTCA table has per-reserve
core/buffer area numbers but only for Tiger Reserves; the actual zone
*boundary descriptions* and *entry rules/permitted activities* text exists
only in each reserve's gazette notification PDF (e.g.
`ntca.gov.in/assets/uploads/notification/tigerreserve/Bandipur.pdf`),
confirmed to be scanned images with no extractable text layer (checked via
pdfplumber on Bandipur.pdf and Mudumalai.pdf — zero characters extracted).
OCR is out of scope (no AI-based collection/cleaning in this project, no
OCR pipeline exists) — flagging this as a manual-curation gap rather than
silently leaving `zone.boundary_geojson`/`entry_rules` blank with no
explanation. Non-Tiger-Reserve zones (National Parks, Wildlife
Sanctuaries, Conservation/Community Reserves, Biosphere Reserves) are not
covered by any source found so far.

Tested end-to-end against a local D1/R2 stand-in (`harvest-engine/tests/`)
with a real live fetch of the NTCA page — 6 zone rows (core+buffer for all
3 reference reserves) created correctly, idempotent on re-run (see
`harvest-engine/tests/README.md`). Also fixed in this pass: `RawFetchItem`
(`harvest_engine/items.py`) was missing the `content_sha256`/`r2_key`/
`is_duplicate` fields that `RawStoragePipeline` sets on every item — this
bug meant no spider had ever actually written a row to D1 through the real
pipeline before it was caught here (only unit-level/isolated testing had
been done previously, not a real end-to-end run through the full item
pipeline).

## Hydrology (rivers, water bodies)

`sources/overpass.yaml` + `scripts/normalize_hydrology.py` — Overpass API
(OpenStreetMap), queried per-reserve bbox for water features (rivers,
streams, lakes/ponds/reservoirs, wetlands, springs) via
`scripts/run_api_spider.py overpass`, same per-reserve-subprocess pattern
as gbif/ebird/inaturalist. Maps OSM tags to the existing
`water_body.water_type` schema values — see the normalize script's
docstring for the full mapping and what's deliberately left NULL
(geojson/full geometry — only a representative center point is fetched, to
keep load down on a shared, rate-limited public instance).

Verified live 2026-08-27: real fetch against all 3 reference reserves
returned 353 water_body rows total (325 for Bandipur, 23 for Mudumalai, 5
for Sathyamangalam) — this large spread reflects genuinely uneven OSM
mapping density between adjacent parks, not a bug (cross-checked: e.g.
Sathyamangalam is mapped in OSM as a proper `boundary=protected_area`
relation, while Bandipur/Mudumalai are not, despite being comparably major
tiger reserves — see `sources/overpass.yaml` notes and
`harvest-engine/tests/fixtures.py`). Tested end-to-end against a local
D1/R2 stand-in, idempotent on re-run.

## Threats (poaching + human-wildlife conflict)

`scripts/normalize_threats.py` sources two of the spec's three threat
categories, each from its own live source:

- **poaching** — `sources/ntca-tiger-mortality.yaml` (new
  `ntca_tiger_mortality` parser in `harvest_engine/spiders/seed_list.py`),
  NTCA's live Tiger Mortality page (`ntca.gov.in/tiger-mortality/`), a
  real per-incident table (date/state/reserve/inside-or-outside/whether a
  wildlife-crime seizure was made) covering 2021 through the present.
  Only rows where "Whether seizure" = Yes become a threat row — the
  source records mortality + seizure status, not cause of death, so a
  seizure=No mortality is not treated as poaching (would be fabricating a
  claim the source doesn't make). Verified live 2026-08-27: 841
  deduplicated mortality records (2021-2026), 62 with a confirmed
  seizure — including one for Sathyamangalam (2023-02-19) and one for
  Bandipur (2026-08-12), both present in the reference-reserve test set.
- **human_wildlife_conflict** — `sources/sathyamangalam-exgratia.yaml`
  (new `sathyamangalam_exgratia` parser), Sathyamangalam Tiger Reserve's
  own official site (`sathytiger.tn.gov.in/exg`), a real yearly ex-gratia
  compensation table (human deaths/injuries, crop/livestock/property
  damage, settled amounts, beneficiaries). This is reserve-and-year
  aggregate data, not per-incident, so one threat row is created per
  (reserve, year) rather than per individual case. Verified live
  2026-08-27: 4 years (2022-2023 through 2025-2026) for Sathyamangalam —
  the ONLY structured, per-reserve, live HWC source found so far;
  Bandipur has no equivalent public site (bandipurtigerreserve.com is an
  unregistered/parked domain) and Mudumalai's own site
  (mudumalaitigerreserve.com) is a client-side-rendered Vue SPA whose
  real content isn't visible to a plain HTTP fetch (this project has no
  headless-browser rendering capability).

Both tested end-to-end against a local D1/R2 stand-in, idempotent on
re-run.

**Encroachment is not sourced yet** — no structured (tabular/API),
per-reserve live source was found as of 2026-08-27 (NTCA's own
"human-tiger-interactions" page is policy/SOP text only; PARIVESH,
Karnataka's e-Parihara, and Tamil Nadu's EDAF systems all either lack a
public API or are access-restricted to forest officials — see
`scripts/normalize_threats.py`'s docstring for the full list checked).
Flagged as a manual-curation gap, same pattern as the Zones
boundary/entry-rules text gap above.

## People/Tribe — manual curation needed, not automated

No branch built for this milestone. Researched 2026-08-27: no structured,
reserve-attributable, live source was found for tribal/indigenous
community data tied to a specific reserve (tested against Bandipur,
Mudumalai, Sathyamangalam). What exists:
- NTCA's per-reserve "brief note" PDFs (e.g.
  `ntca.gov.in/assets/uploads/briefnote/bandipur.pdf`) are real,
  text-extractable, but too thin — generic mentions of "settlements"/
  "border villages," no named communities, no population figures.
  NTCA's fuller Tiger Conservation Plan / gazette notification documents
  (which likely do have a socio-economic/settlements chapter) are scanned
  images with no extractable text, same problem as the Zones gap.
- Real ethnographic/traditional-ecological-knowledge content for
  communities in this exact landscape (Soliga, Jenu Kuruba, etc.) exists,
  but only as scattered individual peer-reviewed journal papers (ATREE,
  Frontiers, Springer, antrocom) — each would need to be individually
  found, read, and hand-curated as its own citation; no queryable,
  reserve-indexed database aggregates them.
- TKDL (Traditional Knowledge Digital Library) is live but is a
  patent-prior-art search tool, national in scope, not reserve-specific.
- ENVIS Centre on Medicinal Plants (envis.frlht.org, medicinalplants.nic.in)
  — both domains are dead (DNS failure, confirmed 2026-08-27).

This branch needs manual/document curation (hand-reading brief notes +
individually sourcing academic papers per community), not a scrape
target — flagging to the project owner per the build spec's own
instruction to do so, same posture as Sathyamangalam's precedent for
non-automatable content.

## Corridors

`sources/moefcc-elephant-corridors.yaml` (`kind: pdf`, no dedicated
per-source spider parser — `scripts/normalize_corridors.py` re-parses the
raw PDF from R2, same pattern as normalize_zones.py/normalize_threats.py)
— MoEFCC/Project Elephant's "Elephant Corridors of India 2023" report, a
real 183-page, text-extractable (not scanned) PDF with a consistent
per-corridor structured block (Connectivity, State, Indicative
length/width, Geo coordinates, etc.) for every named corridor nationwide.
`projectelephant.gov.in` itself is dead; this PDF is hosted on
`moef.gov.in` instead.

Extracts `corridor.name`/`description` (the "Connectivity" field's own
text)/`length_km`/`width_km` (new columns —
`db/migrations/0005_corridor_description.sql`) for all 81 corridor
entries found. `corridor_reserve` links are created ONLY for a
reserve.name explicitly present as a substring of the Connectivity text —
NOT inferred from landscape-level language like "Mudumalai – Bandipur –
Wayanad – Sathyamangalam complex" (which appears as ecological context on
several entries, not a literal two-reserve connectivity claim). No
corridor entry found explicitly names two of this project's 3 reference
reserves as its own two endpoints — each of the 3 real matching corridors
names exactly one reference reserve plus a non-reference-reserve forest
(Kaniyanpura-Moyar → Bandipur; Edayarhalli-Guthiyalathur → Sathyamangalam;
Mudumalai-Mukuruthi → Mudumalai), so each is stored with exactly one
corridor_reserve link — the corridor's other endpoint is a real,
documented gap for a human to resolve by reading the source PDF, not
guessed at. `species_corridor` links the corridor to the `species` row for
Elephas maximus when one exists (populated by FLORA/FAUNA harvesting) —
skipped, not fabricated, if that row doesn't exist yet.

Verified live 2026-08-27: 81 corridors extracted, all 3 expected
reference-reserve links found. Tested end-to-end against a local D1/R2
stand-in (including confirming species_corridor linking works once an
Elephas maximus species row is present), idempotent on re-run.

## Language tagging

`species_cultural_name.language` and `local_place_name.language` are
mandatory (`NOT NULL`) ISO 639-1 codes, or the literal `'unknown'` when a
source doesn't state the language — never guessed, never NULL. See
`harvest-engine/harvest_engine/language_tagging.py` and
`scripts/check_unknown_languages.py` (wired into the harvest workflow,
advisory-only) for the enforcement + review-surfacing mechanism.

## Local development

```bash
cd harvest-engine
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in R2/D1 credentials, then `source .env` or use direnv

python scripts/apply_migrations.py
scrapy crawl seed_list -a source=ntca-tiger-reserves
python scripts/export_geojson.py
```

## Notes for contributors

- Spiders must define `async def start(self):`, not `def start_requests(self):`
  — Scrapy >=2.13 (this project pins `scrapy>=2.11,<3`, so a fresh install
  can land on either) calls `Spider.start()` as the crawl entry point and
  never looks for `start_requests` at all. Getting this wrong doesn't raise
  an error — the spider silently makes zero requests and the crawl "finishes"
  immediately with 0 items scraped, which is easy to miss if you don't check
  `item_scraped_count` in the stats. All three spiders in this repo already
  use the correct hook; keep new ones consistent.

## Out of scope

No booking/safari logistics, no public live API, no state-level tourism or
general culture content unless tied to a specific reserve, no merging with
sibling repos/databases, no crawler framework other than Scrapy.
