# india-data-core

The shared core of the India environmental/cultural data platform: one
Postgres+PostGIS database, one MinIO object store for raw payloads, and one
validating write API that every engine goes through.

Built to `PLAN.md` in the `india-data-platform` repo. The architectural decisions
here are not re-litigated locally — see that file (§3) for why there is no
DAG orchestrator, why scheduling is systemd timers rather than GitHub
Actions cron, and why there are no proxies or headless browsers by default.

## Why engines don't touch the database

Eight engines writing straight to Postgres means eight implementations of
authentication, field validation, dedup-on-stable-id, licence enforcement and
rejection logging — seven of which will be subtly wrong. Everything goes
through `core-api` instead, which does all five in one place.

## Layout

```
db/migrations/     Postgres + PostGIS schema, applied in filename order
api/               FastAPI service
  routers/         health · registry+heartbeat · runs · raw archive · ingest
ops/scripts/       pg_backup.sh · bootstrap_minio.sh
ops/systemd/       core-pg-backup.{service,timer}
scripts/           apply_migrations.py · create_engine_key.py
docker-compose.yml postgres · minio · core-api
```

## The data model in one paragraph

`entity` is any named/spatial thing an engine produces — a protected area, a
river, a peak, a statute. Its attributes live in `entity_fact`, one row per
(entity, field, source, validity period), each carrying its own `source_id`,
`retrieved_at`, `confidence`, `license`, `publish_precision` and
`content_hash`. Three sources may assert three different areas for the same
reserve; all three are kept and none overwrites another, with resolution
happening at read time in the `entity_fact_best` view. `raw_archive_ref`
points at the unmodified bytes in MinIO. `taxon`/`occurrence` and
`timeseries_series`/`timeseries_point` exist separately because a taxon's
identity is nomenclatural rather than spatial, and telemetry is far too
high-volume for the fact table.

`content_hash` on a fact is the hash of the **extracted fields**, never of
raw HTML. That is deliberate: WII's Drupal pages carry a per-request session
nonce, so raw-body hashing reports "changed" on every single fetch.

## Guarantees enforced in the database, not just the API

- All six provenance columns are `NOT NULL` on every fact-level table.
- `publish_precision` is an enum, and a trigger refuses `full` for
  `sacred_grove`, `traditional_knowledge` and `fra_claim` entity types — the
  legal/ethical line in `PLAN.md` §5 is a constraint, not a convention.
- Every write path upserts on a stable identifier; there is no blind insert.
- Rejections go to `ingest_rejection` with the offending payload, including
  ones rejected by schema validation before they reach a router.

## Operating it

```bash
cd ~/core-infra
docker compose ps
docker compose logs -f core-api

# schema
docker compose exec -T core-api python scripts/apply_migrations.py

# mint an engine key (printed once)
docker compose exec -T core-api python scripts/create_engine_key.py protected_areas "pa-engine"

# what needs attention
curl -s localhost:8000/v1/ops/alerts | python3 -m json.tool

# what is due to be harvested
curl -s localhost:8000/v1/sources/due | python3 -m json.tool
```

Nothing binds to a public interface. To reach the API or Postgres from a
laptop, tunnel:

```bash
ssh -N -L 8000:127.0.0.1:8000 -L 5432:127.0.0.1:5432 ubuntu@40.160.137.239
```

## Backups

`ops/systemd/core-pg-backup.timer` runs a nightly `pg_dump` at 02:40 UTC into
the `pg-backups` MinIO bucket, keeps 7 days locally and 90 remotely, refuses
to accept a suspiciously small dump as a good one, and pings the `pg-backup`
heartbeat. `Persistent=true` means a missed night runs at next boot instead
of being skipped.

## Changing a slug function

An entity's identity **is** its slug: `uid` is `<engine>:<entity_type>:<slug>`
and the API upserts on `uid`. So changing an engine's slug function does not
rename entities — the next harvest computes different uids, inserts new rows,
and abandons the old ones. This is not hypothetical: the Water engine's
underscore fix turned 558 datasets into 623 overnight (D-28, D-31). The upsert
was working perfectly; it was asked about a different identity.

**Changing a slug function is a data migration.** The steps, in order:

1. **Register the new generation** in a migration: `INSERT INTO slug_function`
   with the new `version`, the `formula`, and the `natural_key` — the parts
   that identify an entity *independently of its slug*. Nothing else works
   without this; the natural key is what the diff runs on.
2. **Bump `SLUG_VERSION`** in the engine, next to the slug function itself,
   and make sure every entity dict sends it.
3. **Emit a slug map** from the engine — `(natural key → new slug)` for every
   entity of that type. The Water harvester's `--emit-slug-map` is the
   reference implementation; it costs one HTTP request.
4. **Dry-run the migration**: `python scripts/migrate_slugs.py --slug-map
   /tmp/x.json`. It prints what it would re-key and refuses outright on any
   condition that makes a safe re-key impossible — an unrecoverable or
   ambiguous natural key, pre-existing duplicates, or a new function that
   collides. A refusal changes nothing.
5. **Apply**: add `--apply`. It takes a labelled `pg_dump` to MinIO first,
   re-keys in place inside one transaction, and verifies the entity count is
   unchanged and slugs are unique before committing.
6. **Deploy the engine** and let it harvest normally.

Doing 6 before 5 is the mistake that caused the incident.

Two properties make this safe to run unattended: it **never inserts and never
deletes** — every write is an `UPDATE` of an existing row, so a wrong map
cannot lose data — and it **refuses partial migrations**, because
half-migrated is the state the whole mechanism exists to prevent. Entities the
engine no longer emits are reported as orphan candidates and left alone;
deciding to delete one is a separate, human call.

```bash
# which rows are keyed by a superseded slug function (should be empty at rest)
psql -c 'SELECT * FROM entity_stale_slug'
# the registry, current generation per entity type
psql -c 'SELECT * FROM slug_function_current'
# the planner's refusal rules
python tests/test_migrate_slugs.py
```

Every entity type in the database is registered in `slug_function`, including
those whose slugs are hand-authored constants (`is_computed = false`). An
entity type *missing* from that table is an unaudited one, which is a
different and worse thing than one audited and found to need no machinery.
