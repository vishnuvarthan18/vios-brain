# CI/CD, cleanup & real offsite backup — plan

_Written 2026-09-25, after auditing all 10 repos on the Mac (`~/india-platform/`) and the VPS's actual running state._

## Audit findings (what's actually true today)

**Good — no urgent security fire:**
- No secrets ever committed to any repo's git history (checked `.env`, `_key`, `secret`, `credentials.json` patterns across the 4 most likely repos).
- `.env` and `.venv` are properly gitignored everywhere they appear.
- The `Claude outputs` folder accidentally committed to `india-data-core` on 2026-09-23 is now untracked and gitignored — just a harmless local leftover.
- Repo structure is consistent: `Dockerfile`, `docker-compose.yml`, `requirements.txt`, a package dir, `scripts/`, `ops/systemd/` — across all 9 engine-shaped repos. `india-data-platform` is intentionally different (it's the original core repo: `DECISIONS.md`, `PLAN.md`, `harvest-engine/`, `site/`, `exports/`).
- `india-extinct-engine/data` is a 4KB example file, not committed bulk data. Not bloat.

**Two real gaps:**
1. **Backup is not offsite.** `india-data-core/ops/scripts/pg_backup.sh` runs nightly via `core-pg-backup.timer`, dumps Postgres with `pg_dump`, keeps 7 days locally, and pushes to a `pg-backups` MinIO bucket with 90-day retention — but that bucket is a container on the *same* VPS, same disk. A dead disk or a lost VPS takes out the primary and the "backup" together. The raw-archive/media bucket (`core_miniodata`, ~600MB) has no offsite copy at all.
2. **No CI/CD anywhere.** Zero `.github/workflows` files in any of the 10 repos. Every deploy is a manual SSH session — which is exactly how tonight's session hit wrong-shell and wrong-directory mistakes three times over.

One existing decision to respect: a comment in `pg_backup.sh` says *"Never by GitHub Actions cron (PLAN.md §3.4)"* — scheduling stays on VPS systemd timers, not GitHub Actions cron. CI/CD below is for build/deploy on push, not for replacing the harvest schedule.

## Phase 1 — Fix the real single point of failure (do first)

Extend the existing `pg_backup.sh` pattern rather than replacing it:
- After the local MinIO upload succeeds, add a second push to a true offsite target for both buckets (`pg-backups` and the raw archive/media bucket).
- Recommended target: Cloudflare R2 (`rclone` with an S3-compatible remote) — free up to 10GB, no egress fees, API built for many files and for automation, and no personal-login-token expiry problem the way Google Drive has.
- Keep the same retention shape: prune old offsite copies after e.g. 90 days for DB dumps, 30-60 days for raw archive.
- Verify by restoring one dump from the offsite copy into a scratch database once, so "we have backups" is proven, not assumed.
- Report success/failure to the same heartbeat table the rest of the platform uses, so a broken offsite push shows red in the ops console, not silence.

### Progress log

- **2026-09-25:** R2 bucket `india-data-atlas-raw` created (single-bucket scope, Object Read & Write only, IP-restricted to the VPS). Account API token `india-data-atlas-backup` created, then rolled twice during setup (once as a precaution, once because an Access Key ID was accidentally pasted into chat and had to be treated as compromised). Final credentials saved to `~/core-infra/.env` on the VPS (`R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET=india-data-atlas-raw`) — all values entered directly at the VPS terminal, never through chat. Connectivity verified with `mc ls` against the bucket via Docker — connected successfully, no errors. **Credentials are live and working.** Next: extend `pg_backup.sh` to push to this bucket, add retention pruning, wire up heartbeat, and do one test restore.
- Open question still unanswered by the owner: what created the three pre-existing R2 buckets seen in the account (`blastdesk-downloads`, `forest-raw-pdfs`, `harvest-engine-raw`) — unrelated to this project as far as we know, but worth confirming they're not orphaned/forgotten.

## Phase 2 — Minor repo cleanup (low risk, can run alongside Phase 1)

- Remove the local, already-gitignored `Claude outputs/` folder from `india-data-core` on disk (cosmetic only).
- Decide `india-extinct-engine`'s fate (build or drop) — this is open item 12 from `open-items-next-session.md`, unrelated to CI/CD but worth closing while touching repos.
- No structural cleanup is actually needed — the repos are already consistent.

## Phase 3 — CI/CD (build + deploy on push)

Given the VPS is only 2 vCores / 4GB RAM, the simplest reliable shape is: GitHub Actions runs on GitHub's own runners (free, doesn't touch VPS resources) and its only job is to SSH into the VPS and drive the same commands we've been running by hand.

Per repo, on push to `main`:
1. Run tests/lint if the repo has any (most don't yet — add later, not blocking).
2. SSH into the VPS using a **dedicated deploy key**, not your personal key — generated fresh, its public half added to a restricted `authorized_keys` entry (or a separate limited deploy user) on the VPS, its private half stored as an encrypted GitHub Actions secret.
3. Run, in order: `git pull --rebase`, `docker compose build`, apply any new DB migration, `docker compose up -d`, then hit a health-check endpoint.
4. On success, log the deploy (repo, commit SHA, timestamp) — this is the "deploy history" the ops console already has a place for (open item 14) but nothing writes to yet.
5. On failure, stop before restarting containers, so a bad build never replaces a working one.

This also directly fixes tonight's actual failure mode: no more hand-typed multi-repo SSH loops that silently run in the wrong shell.

## Phase 4 — Rollback + runbook

- Tag the built Docker image with the git commit SHA on every deploy, keep the last 5.
- A rollback is: re-point `docker compose` at the previous SHA's image and restart — no rebuild needed, so it's fast under pressure.
- Write a one-page runbook: how to roll back a bad deploy, how to restore Postgres from an offsite dump, how to restore the raw archive. This belongs in `india-data-core/README.md` or a new `RUNBOOK.md`, not only in this project doc.

## Order of execution

1. Phase 1 (offsite backup) — closes the actual risk behind "if anything fails." **In progress — credentials done, script extension next.**
2. Phase 3 (CI/CD) — closes the actual risk behind tonight's repeated manual mistakes.
3. Phase 4 (rollback runbook) — cheap once Phase 3 exists, since it just documents and tags what Phase 3 already does.
4. Phase 2 (cosmetic cleanup) — whenever, no urgency.

## Decisions needed from the owner before building

- Offsite backup target: confirm R2 (recommended) vs Google Drive vs something else. **Decided: R2. Done.**
- Whether to create a new restricted deploy-only Linux user on the VPS for CI (recommended) vs reusing the `ubuntu` user with a second key. **Decided: dedicated deploy-only user. Not yet built.**
- Whether Extinct Species engine gets built or dropped (Phase 2, not blocking).
