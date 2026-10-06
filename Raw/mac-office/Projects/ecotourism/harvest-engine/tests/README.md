# Local D1/R2 stand-ins

Real Cloudflare R2/D1 are not provisioned yet (see top-level README). These
stand-ins let every harvest/normalize script run completely unmodified
against real local infrastructure instead of hand-mocked unit tests:

- `d1_stand_in.py` — a tiny HTTP server reproducing the D1 HTTP API's
  request/response shape (`POST /query`), backed by real sqlite3. Point
  `D1_HTTP_API_BASE_URL` at it (e.g. `http://127.0.0.1:8787`) and
  `scripts/apply_migrations.py` / any `d1_query`/`d1_execute` call works
  exactly as it would against real D1.
- `r2_http_stand_in.py` — a minimal S3-compatible HTTP server (PUT/GET
  object only, path-style addressing, no signature verification) backed by
  a local directory. Point `R2_ENDPOINT_URL` at it and any real
  `boto3.client("s3", ...)` call in production code (RawStoragePipeline,
  normalize_flora_fauna.Normalizer, or a future branch's normalizer) works
  completely unmodified — including inside a `scrapy crawl` subprocess,
  where in-process mocking can't reach.

Both are test-only scaffolding, never imported by production code
(`harvest_engine/`, `scripts/*.py` construct `boto3.client("s3", ...)` and
talk to `D1_HTTP_API_BASE_URL` themselves, unaware these are stand-ins) —
`run_branch_test.py` just points the real code's env vars at these local
servers instead of real Cloudflare R2/D1. Production code never imports
anything from `tests/`.

## Running a branch test end-to-end

```bash
cd harvest-engine
source .venv/bin/activate
python tests/run_branch_test.py <branch>   # e.g. zones, hydrology, threats, people, corridors
```

Each branch test:
1. Starts a fresh local D1 stand-in (sqlite3 file under a temp dir) and
   applies all `db/migrations/*.sql`.
2. Seeds exactly the 3 reference reserves (Sathyamangalam, Bandipur,
   Mudumalai) with real bbox/centroid data, matching the fixture set used
   across every branch so results are comparable branch to branch.
3. Runs the real spider(s) for that branch's source(s) against the live
   real source (not a fixture) — raw responses land in the local R2
   stand-in exactly as they would in real R2.
4. Runs the real `scripts/normalize_*.py` for that branch against the
   local stand-ins.
5. Prints row counts + a few sample rows per table touched, and runs
   `scripts/check_unknown_languages.py`-style assertions where relevant
   (e.g. zero rows with a NULL `source_id`/`confidence`/`retrieved_at` —
   the provenance rule is structural, so a test failure here means the
   schema itself was violated, not just a script bug).

Nothing here talks to real Cloudflare R2/D1 — that dry run happens only
after all 6 branches are built and locally verified (see top-level README).
