# Runbook — Database Backup and Restore

**The tooling already exists and works** — this was previously listed as "not started" in the documentation inventory, which was only true of this runbook document itself. `deploy/arm-deploy-make/make/backup.mk` and its two scripts are real, tested (the restore drill specifically proves backups are restorable, not just that the backup step ran), and only target **production** — verified against the scripts directly, 2026-08-11.

**Update 2026-08-17:** three of the four gaps §4 originally listed are now closed — off-host sync, scheduling, and a MongoDB restore drill all exist. §1–§4 below are updated to match; only the retention/PITR distinction is new content, not a correction of what was there before.

---

## 1. Taking a backup

```sh
make backup-postgres   # arm_core + arm_admin, both dumped from arm-postgres-prod
make backup-mongodb    # arm-calendar, dumped from arm-mongodb-prod
make backup-sync       # sync backups/ to S3-compatible object storage — skips gracefully if BACKUP_S3_BUCKET is unset
make backup-prune      # delete local backups older than BACKUP_RETENTION_DAYS (default 14)
make backup-all        # postgres + mongodb + sync + prune, in that order
```

| | Postgres | MongoDB |
|---|---|---|
| Mechanism | `pg_dump` per database, piped through `gzip` | `mongodump --archive --gzip`, run inside the container then `docker cp`'d out |
| Output | `backups/postgres/<db>-<UTC timestamp>.sql.gz` — one file per database (`arm_core`, `arm_admin`) | `backups/mongodb/arm-calendar-<UTC timestamp>.archive` |
| Credentials | `POSTGRES_USER` from `.env` (default `admin`) | `MONGO_ROOT_PASSWORD` from `.env` (required — script fails fast if unset), `MONGO_ROOT_USER` defaults to `arm_admin` |
| Atomicity | Writes to `<file>.sql.gz.partial`, only `mv`s to the final name on success — an interrupted dump never leaves a file that looks complete under its real name | No equivalent `.partial` pattern — relies on `set -euo pipefail` stopping the script before `docker cp` runs if `mongodump` itself fails |

**Both scripts hardcode `arm-postgres-prod`/`arm-mongodb-prod`** — they only work against the production compose stack's container names. Running `make backup-postgres` against the local stack does nothing useful; there's no `ENV` parameter to retarget them.

`backups/` is gitignored — every backup lands on local disk on whatever machine ran the command first. **`make backup-sync` copies it off-host** to (secret removed) object storage (Hetzner Object Storage by default — `scripts/backup-sync-s3.sh`, run via the official `amazon/aws-cli` Docker image, no host-level dependency) once `(secret removed)`/`(secret removed)`/`(secret removed)`/`(secret removed)` are set in `.env`. Left unset, `backup-sync` prints a warning and exits 0 rather than failing `backup-all` — off-host sync is opt-in-but-recommended, not a hard requirement to keep backing up locally. Note this is still logical/point-in-time dumps, not continuous PITR (WAL archiving via pgBackRest/wal-g) — a real RPO gap if more than a day of data loss matters, and a separate, bigger undertaking from what closed here.

**Scheduling:** `make backup-cron-install` adds a daily 02:00 crontab entry running `make backup-all` (idempotent — checks before adding, safe to call on every deploy; `deploy-production.yml`'s `deploy` job does exactly that). `make backup-cron-status` / `make backup-cron-remove` inspect/undo it. Logs land in `backups/cron.log`.

---

## 2. Automated restore drills — Postgres `arm_core` and MongoDB `arm-calendar`

```sh
make restore-postgres-drill
make restore-mongodb-drill
```

`restore-postgres-drill` finds the most recent `arm_core-*.sql.gz` in `backups/postgres/`, restores it into a **throwaway** `arm_core_restore_drill` database on the *same running* `arm-postgres-prod` instance, counts the restored public tables as a sanity check, then drops the throwaway database.

`restore-mongodb-drill` (new) does the equivalent for MongoDB: finds the most recent `arm-calendar-*.archive` in `backups/mongodb/`, uses `mongorestore --nsFrom/--nsTo` to restore it into a throwaway `arm-calendar_restore_drill` database on the *same running* `arm-mongodb-prod` instance (namespace-remapped at restore time, not restored into and then copied out of the live `arm-calendar` — the live database is never touched by the restore itself), counts the restored collections, then drops the throwaway database.

**Neither touches its live database** — verification tools, not restore procedures for an actual incident (§3 covers that).

Run these periodically (§1's cron job runs `backup-all`, not the drills — nothing schedules the drills themselves yet) as the actual proof that backups are usable, not just that the backup step completed without error.

---

## 3. Manual restore — for an actual incident, or for what the drills don't cover

**No automated drill exists for `arm_admin`** — despite `backup-postgres` backing up both Postgres databases, only `arm_core` gets a tested restore path (mirroring `restore-postgres-drill`'s pattern for `arm_admin` would close this the same way `restore-mongodb-drill` closed the MongoDB gap). If you need to actually restore, or need to verify `arm_admin` specifically, do it by hand:

**Postgres — restoring `arm_core` or `arm_admin` for real** (adapt `restore-postgres-drill`'s pattern, targeting the real database name — **this overwrites live data, confirm you mean to before running**):
```sh
gunzip -c backups/postgres/arm_core-<timestamp>.sql.gz | \
  docker exec -i arm-postgres-prod psql -U <POSTGRES_USER> -d arm_core
```
Consider dropping and recreating the target database first (as the drill does) if the restore needs to be a clean replace rather than an overlay onto existing data — a plain `psql` replay onto a non-empty database can conflict with existing rows/constraints depending on what changed since the backup.

**MongoDB — restoring `arm-calendar` for real** (adapt `restore-mongodb-drill`'s pattern, but restore directly into `arm-calendar` — no `--nsFrom`/`--nsTo` remap — **this overwrites live data, confirm you mean to before running**):
```sh
docker cp backups/mongodb/arm-calendar-<timestamp>.archive arm-mongodb-prod:/tmp/restore.archive
docker exec arm-mongodb-prod mongorestore \
  --username <MONGO_ROOT_USER> --password <MONGO_ROOT_PASSWORD> \
  --authenticationDatabase admin \
  --archive=/tmp/restore.archive --gzip \
  --drop
docker exec arm-mongodb-prod rm /tmp/restore.archive
```
`--drop` replaces existing collections with the backup's contents rather than merging — appropriate for a real restore, but confirm that's actually what's wanted before running it against live data.

**Restoring from the off-host copy** (server lost or backups/ otherwise gone locally): pull the archive back down first, then follow the same restore steps above.

```sh
docker run --rm \
  -e (secret removed)<(secret removed)> -e (secret removed)<(secret removed)> \
  -v "$(pwd)/backups:/backups" amazon/aws-cli:2.24.13 \
  s3 sync "s3://<BACKUP_S3_BUCKET>/backups" /backups --endpoint-url <BACKUP_S3_ENDPOINT>
```

---

## 4. What this doesn't cover — real, currently-open gaps

- **No PITR.** These remain point-in-time logical dumps (`pg_dump` / `mongodump`), not continuous WAL archiving (pgBackRest/wal-g). Worst case, up to a day of data loss (the gap between the last nightly backup and an incident) — acceptable for many setups, not for an RPO tighter than 24h. A bigger, separate undertaking from what closed here.
- **No retention/cleanup on the off-host copy.** `backup-prune` only prunes local disk (`BACKUP_RETENTION_DAYS`, default 14) — the S3 bucket accumulates every synced backup forever unless a lifecycle rule is configured on the bucket itself. That's deliberately not this repo's job; set an S3 lifecycle policy on `BACKUP_S3_BUCKET` directly.
- **No automated drill for `arm_admin`**, and **no schedule for the drills themselves** — `backup-cron-install` schedules `backup-all` (taking + syncing + pruning backups), not `restore-postgres-drill`/`restore-mongodb-drill` (proving they're restorable). Running the drills periodically is still a manual, on-call responsibility.
- **Container names are hardcoded to the server stack.** There's no equivalent tooling for backing up the local databases via these same targets.

Off-host storage, scheduling, and the MongoDB drill (all originally listed here as gaps) are closed as of 2026-08-17 — see §1/§2 above.

---

## Related documents

- [INFRASTRUCTURE.md](../INFRASTRUCTURE.md) §9 — where this tooling was first verified, volume/persistence strategy context
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) — schema reference for what's actually inside `arm_core`/`arm_admin`/`arm-calendar`
- [restart-services.md](restart-services.md) — restarting services after a restore, and the healthcheck-strength caveats that apply the same way post-restore
