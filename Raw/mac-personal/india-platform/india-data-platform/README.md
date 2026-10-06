# India Data Platform — Protected Areas Engine

The first of the platform's eight engines (see `PLAN.md`). It harvests
India's protected areas — national parks, wildlife sanctuaries, tiger
reserves, conservation and community reserves — and writes them to the
shared core at [`india-data-core`](../india-data-core).

## Where things run

| | |
|---|---|
| Core stack (Postgres+PostGIS, MinIO, core API) | VPS `40.160.137.239`, `~/core-infra/` |
| This engine | VPS, `~/pa-engine/`, containerised |
| Scheduling | **systemd timers on the VPS**, never GitHub Actions cron (`PLAN.md` §3.4) |
| CI | GitHub Actions, `.github/workflows/ci.yml` — tests only, no `schedule:` trigger |

## The three stages

Acquire → Parse → Load, kept genuinely separate (`PLAN.md` §3.3):

```
scripts/run_harvest.py         asks the core registry what is DUE, then fetches.
                               Raw bytes go to MinIO through the core API BEFORE
                               anything parses them.
scripts/normalize_reserves.py  replays those archived bytes into entity + fact
                               rows. Re-runnable without re-fetching — a parser
                               bug is fixed by re-parsing, not by hitting a
                               government website again.
scripts/backfill_geometry.py   fills in boundaries for anything still missing one.
```

`scripts/run_spider.py` runs one source and is the piece that reports
honestly: it pre-flights required API keys against the registry, catches
spider construction errors, and treats a run that scraped nothing — or that
archived an HTTP 200 whose body contains zero records — as a failure. See
"Bugs fixed" below for why each of those checks exists.

## Cadence is data, not code

There is no hard-coded source list and no flat cooldown here any more. Each
source carries a `schedule_tier` and an independent `staleness_ceiling` in
the core registry, and `run_harvest.py` just asks `GET /v1/sources/due`.
Changing how often a source is fetched is an UPDATE, not a redeploy.

The ceiling is checked separately from the tier on purpose: a source that is
due *only* because of its ceiling is reported as `past_staleness_ceiling`,
which means the tier schedule has stopped firing — a bug, not just a fetch.

## Bugs fixed during the migration

- **Reserves had no geometry at all.** Nothing ever wrote centroid, boundary
  or bbox, so all 602 reserves were NULL and the map had zero pins.
  `backfill_geometry.py` fills them from OpenStreetMap (ODbL — commercial use
  and redistribution both permitted). The WDPA path `PLAN.md` specifies is
  implemented too and runs as soon as a key exists.
- **`parivesh` reported OK while being completely broken.** `scrapy crawl`
  exits 0 when a spider raises in `__init__`, so a source that had never
  fetched anything looked healthy. Now: a registry-driven pre-flight on
  required keys, construction errors caught, and post-run stats inspected.
  With that in place the source turned out to be returning
  `{"status":"error","message":"Meta not found","records":[]}` — see
  `DECISIONS.md` D-16.
- **WII dedup never worked.** WII's Drupal pages carry a per-request session
  nonce, so hashing the raw HTML reported "changed" on every single fetch.
  Change detection now hashes the **extracted fields**
  (`harvest_engine/content_hash.py`). Raw-body hashing still runs one level
  down, where it belongs: the core API dedups identical *payloads*.
- **Credentials were being stored in the database.** data.gov.in takes its
  key as a query parameter, and the fetched URL is kept as provenance. The
  core API now redacts secret-bearing query parameters before storing.

## Known gaps, deliberately not "fixed"

- **Telangana has no WII source page**, so its two tiger reserves (Kawal,
  Amrabad) can never cross-merge with a WII row. Real and permanent.
- **10 WII rows carry a bare `CR` suffix**, ambiguous between
  `conservation_reserve` and `community_reserve`. The page cannot decide it,
  so they are skipped and logged by name rather than guessed. Resolving them
  means reading the gazette PDFs.
- **200 of 602 reserves have no OSM match.** Matching is name+state and
  deliberately conservative: an ambiguous name is skipped, never guessed. A
  wrong boundary looks like data; a NULL is visibly missing.

## Running it by hand

```bash
cd ~/pa-engine
set -a; . ~/core-infra/engine-keys.env; set +a

docker compose run --rm harvest python scripts/run_harvest.py --dry-run
docker compose run --rm harvest python scripts/run_harvest.py --source ntca-tiger-reserves
docker compose run --rm harvest python scripts/normalize_reserves.py
docker compose run --rm harvest python scripts/backfill_geometry.py --source osm --mode polygon

sudo systemctl start pa-harvest.service      # the whole pipeline, as the timer runs it
journalctl -u pa-harvest -f
```

## Legacy

`scripts/normalize_seed_list.py` and the other `normalize_*.py` scripts still
target the retired Cloudflare D1 schema. They are kept because
`normalize_reserves.py` **imports their extraction and identity logic** —
the parsers, `STATE_ALIASES`, the WII type-suffix rules — rather than copying
it. That logic was written and tested against the live tables and is the
valuable part; only the write target changed.
