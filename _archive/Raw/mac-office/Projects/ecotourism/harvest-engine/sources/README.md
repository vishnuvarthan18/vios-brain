# Source descriptors

One YAML file per source. Spiders are generic and read settings from these
files at run time — adding a new source means adding a YAML file here, not
editing spider code.

## Schema

```yaml
name: string              # unique source name; used as the D1 source.name
description: string
kind: api | html | pdf | csv
spider: string            # which generic spider class handles this kind
base_url: string
license: string
terms_url: string
rate_limit_ms: integer    # minimum delay between requests
auth: none | api_key | none-but-header
robots_txt_verified: string   # date this was last checked, per source, by hand
respect_robots_txt: bool  # optional, default true. Only set false when
                           # robots_txt_verified documents WHY (e.g. the
                           # site's robots.txt blanket-disallows its own
                           # sanctioned API path) — api_source.py reads
                           # this and sets Scrapy's dont_obey_robotstxt
                           # per-request; every other source stays subject
                           # to the global ROBOTSTXT_OBEY=True.
notes: string

requests:                 # one or more request templates this source issues
  - name: string
    url: string            # may contain {placeholders} filled from reserve table or params
    method: GET | POST
    params: {}

auth_header:               # optional — for APIs needing a header-based key (e.g. eBird)
  name: string              # header name, e.g. X-eBirdApiToken
  value: string              # MUST be "env.VAR_NAME" — never a literal secret in the YAML

url_params_from_env:       # optional — for APIs needing a query-param key (e.g. data.gov.in)
  placeholder_name: ENV_VAR_NAME   # {placeholder_name} in requests[].url is filled from this env var

parser: string              # optional — for seed_list.yaml sources, names a function in
                             # seed_list.PARSERS that extracts structured records from the response
user_agent_override: string # optional — some .gov.in sites reject Scrapy's default UA (see
                             # ntca-tiger-reserves.yaml notes)
```

## Milestone 1 sources (protected area seed list)

- `ntca-tiger-reserves.yaml` — National Tiger Conservation Authority's live
  Tiger Reserve list (name, state, notification years, core/buffer/total
  area, gazette PDF) — the richest single source, but Tiger Reserves only.
- `wii-gazette-notifications.yaml` — WII's legacy per-state gazette
  notification database (National Parks, Wildlife Sanctuaries, Conservation
  Reserves, Community Reserves), one request per state/UT (35 total).
- `parivesh.yaml` — data.gov.in dataset (MoEFCC-sourced, Aug 2023) for
  cross-checking state-wise PA counts; requires `DATA_GOV_IN_API_KEY` (see
  `.env.example`) and its column names are unverified until fetched with a
  real key — see the file's own notes before writing a normalizer against it.

**Two sources were removed** after verification on 2026-08-27:
`wii-protected-areas.yaml` (wii.gov.in/protected_area_network and
/protected-area-network are both content-free department pages, no data)
and `moefcc-notifications.yaml` (moef.gov.in/notifications was never
confirmed to host a real notification list). `wiienvis.nic.in`, which
appears in search results and is even linked from WII's current site, is
confirmed **dead** (NXDOMAIN) — do not use it anywhere.

WDPA is explicitly excluded (blocked for India) — do not add a wdpa.yaml.

## Species-data sources (GBIF, eBird, iNaturalist)

- `gbif.yaml` — GBIF occurrence search, per-reserve bbox/polygon query
- `ebird.yaml` — eBird recent observations, per-reserve centroid + radius;
  requires `EBIRD_API_KEY` (see `.env.example`)
- `inaturalist.yaml` — iNaturalist observations search, per-reserve bbox

Run via `scripts/run_api_spider.py <source_name>`, which iterates every
reserve currently in D1 (requires Milestone 1's seed list to have already
loaded reserve bboxes — see `reserve.bbox_min_lon/lat/max_lon/lat`,
added/backfilled in `db/migrations/0003_reserve_bbox_and_normalize_tracking.sql`)
and runs the `api_source` spider once per reserve — one reserve failing
doesn't stop the others.

After fetching, `scripts/normalize_flora_fauna.py` turns the raw GBIF/
eBird/iNaturalist JSON sitting in R2 into typed `species` /
`species_reserve` / `occurrence` / `media` rows — species deduplicated by
`scientific_name` across reserves, occurrences linked to every reserve
whose bbox contains the occurrence's coordinates (an occurrence in an
overlapping buffer zone links to more than one reserve, by design), and
media rows written only for images whose license is CC0/CC-BY/CC-BY-SA/
CC-BY-NC/CC-BY-NC-SA (no -ND, no missing/all-rights-reserved license) —
see that script's own docstring for the full policy. Tested end-to-end
against a local D1/R2 stand-in (real fetched GBIF/iNaturalist responses;
no EBIRD_API_KEY in this environment so eBird was tested against a fixture
built from the API's documented response shape) — not yet run against
real provisioned D1/R2, which don't exist yet (see top-level README).

## Zones (Branch 2)

No new source file — `scripts/normalize_zones.py` re-parses the raw HTML
already fetched by `ntca-tiger-reserves.yaml` (same `source` rows the seed
list uses) into `zone` rows. See the top-level README's "Zones" section for
what is/isn't sourceable this way.

## Language tagging for local names (ENVIS, TKDL, gazetteers, community sources)

`local_place_name.language` and `species_cultural_name.language` are `NOT
NULL` in the schema — every row must carry a language. When writing a
normalizer for a source that produces local/regional names:

- If the source states the language explicitly (e.g. a gazetteer entry
  labelled "(Tamil)", a TKDL record with its own language field), pass that
  code through `harvest_engine.language_tagging.resolve_language_code()`.
- If the source does NOT state the language, do not infer or guess one from
  context (e.g. "this reserve is in Karnataka so it must be Kannada" is not
  good enough) — pass `None` and the row gets tagged `'unknown'`.
- Run `scripts/check_unknown_languages.py` after a harvest to see the count
  of `'unknown'` rows awaiting manual review; it's wired into
  `.github/workflows/harvest.yml` already and always reports, never blocks.
