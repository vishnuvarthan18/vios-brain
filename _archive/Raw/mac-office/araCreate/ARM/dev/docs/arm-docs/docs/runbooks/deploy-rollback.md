# Runbook — Deploy Rollback

**The short version, upfront**: the server now rolls itself back. A failed deploy or a failed health check triggers `rollback-on-failure`, which restores the last image tag that was independently verified healthy — usually before anyone opens this runbook. What you have to do by hand depends on where you are:

| Situation | What to do |
|---|---|
| **Deploy failed and auto-rollback ran** | Confirm it succeeded, then check migration state (§5) — the images went back, the schema did not |
| **The server is bad but the deploy "succeeded"** (auto-rollback never fired) | `make rollback TAG=<sha>` by hand (§1) — the images are on the server |
| **The bad change is in `arm-deploy-make` itself** | Git revert on `main`, then redeploy (§4 step 3) |

> **This runbook previously said the opposite** — that `make rollback` was non-operational and git revert was the only real path. That was accurate when written and is no longer true. §2 explains what changed.

Verified against `deploy/arm-deploy-make/make/ci.mk`, `repos.conf`, and `deploy-production.yml` (re-verified 2026-08-19, after dev and staging were removed), cross-checked against [CICD.md](../CICD.md) §4/§6 — this runbook is the operational "what do I actually do" companion to that document's "how it's built" account.

---

## 1. The coded mechanism — `make rollback TAG=<sha>`

```sh
make rollback TAG=<git-sha> CONFIRM_NO_SCHEMA_ROLLBACK=yes
```

Runs `docker compose -f docker-compose.yml up -d --no-build` with `IMAGE_TAG=<sha>` — starts every production service from a **previously built and pulled** image at that tag, without rebuilding. Requires `CONFIRM_NO_SCHEMA_ROLLBACK=yes`; omitting it prints an explicit warning and refuses to proceed:

> This rolls back APP IMAGES ONLY — it does not touch the database. If any migration ran between `<TAG>` and now, the old app code at `<TAG>` may not be compatible with the current schema.

This is a genuinely good safety gate — schema/code version mismatches are exactly the kind of thing that turns a rollback into a second incident.

**Prerequisite: images tagged `<sha>` must exist on the target server.** In production they do, as a side effect of every normal deploy (§2). Run the target from the deploy directory on the production host:

```sh
cd ~/arm-deploy
make rollback TAG=<sha> CONFIRM_NO_SCHEMA_ROLLBACK=yes ROOT=~/arm-deploy
```

To find a tag to roll back to: `cat ~/arm-deploy/.last-good-tag` for the last verified-healthy deploy, or `docker images | grep arm-` for everything still on disk. Tags are 7-character commit SHAs.

---

## 2. How production got a working rollback

Three pieces, all in `deploy-production.yml`:

1. **Per-deploy image tags.** The `deploy` job computes `IMAGE_TAG=${GITHUB_SHA::7}` and passes it to `make ci-deploy-prod`, rather than every deploy overwriting `latest`. Because production builds on the target server, the previous deploy's images stay on that server's disk under their own tag — `rollback` starts them with `--no-build`, so no registry and no `ci-push` are involved at all. This is what closed the gap this runbook previously described.
2. **A verified marker.** `verify` polls all 20 expected containers for both running status and health (up to 30 × 10s), and only then writes the tag to `~/arm-deploy/.last-good-tag`. A marker therefore cannot name a deploy that was never confirmed healthy.
3. **Automatic recovery.** `rollback-on-failure` runs `if: failure()` after `deploy`/`verify` and rolls back to that marker, passing `CONFIRM_NO_SCHEMA_ROLLBACK=yes` on the operator's behalf. If no marker exists — first-ever deploy, or verify has never passed — it stops with an explicit "manual intervention required" rather than guessing.

**What this does not do:** it never touches the database (§5), and it exists in production only.

Two consequences worth internalising before an incident:

- **A green pipeline that ends in a rollback is still an incident.** The workflow emits a `::warning::` about unreverted migrations; treat the state as "recovered, migration compatibility unverified."
- **Rollback restores images, not source.** The bad commit is still on `dev`, and the next deploy will build it again. Auto-rollback buys time to fix forward or revert (§4) — it isn't the end of the response.

---

## 3. What a deploy actually does — and why git revert is still the durable fix

`ci-deploy-prod` clones every application repo fresh via `clone-repos`, which reads `repos.conf`. **`repos.conf` pins every application repo's `LOCAL_PATH` to the `dev` branch.** So a server deploy clones and builds each application repo's **`dev` branch HEAD**, not a promoted or tagged release — even though the deploy itself is triggered by a push to `main` in `arm-deploy-make`. That asymmetry is worth holding in mind: `arm-deploy-make`'s `main` selects *which deploy runs*, while the application code that gets built is whatever is on each app repo's `dev` at that moment.

**Practical consequence**: because every deploy is a fresh build from a branch HEAD, not a promoted artifact, **reverting the bad commit(s) on that branch and re-running the deploy workflow gets the good code running again** — the next deploy simply builds the reverted code. No image tags, no registry, no promotion step required. An image rollback (§2) buys time; the revert is the *durable* fix that has to follow it, since the rollback leaves the bad commit on the branch untouched.

**The consequence to watch**: since the app repos build from `dev` HEAD, anything merged to `dev` is live on the server at the next deploy — there is no pre-production environment where it would surface first. That is the direct trade-off of running two environments; see [DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md) §2.

---

## 4. Practical rollback procedure

1. **Identify the last-known-good commit** on the affected application repo's `dev` branch. On the server, `cat ~/arm-deploy/.last-good-tag` gives the last SHA of `arm-deploy-make` that deployed and verified cleanly; note that this is the *deploy repo's* SHA, not the application repo's — nothing records which application-repo commit each deploy built, so identifying that still means correlating deploy times against `git log` or the GitHub Actions run history.
2. **Revert, don't force-reset.** `git revert <bad-sha>` (or a range) and push a new commit — `dev` is a shared branch other deploys and CI depend on; a `git reset --hard` + force-push rewrites history other people's local branches and any in-flight PRs are based on. Use revert commits.
3. **If the break is in `arm-deploy-make` itself** (a bad Makefile/compose change, not an application repo) — revert on `main`, the branch that deploys to the server (per [CICD.md](../CICD.md) §2's branch table).
4. **Let the push trigger the deploy workflow** (`deploy-production.yml` on push to `main`, or trigger manually via `workflow_dispatch` in GitHub Actions if the push already happened and the workflow needs a manual kick).
5. **Verify** using [restart-services.md](restart-services.md) §3's health-check reference — a rollback deploy goes through the exact same `ci-deploy-prod` path as a normal deploy, so the same healthcheck-strength caveats apply (the session service and frontends give weaker signal than the backends with real dependency checks).

---

## 5. Database schema — check before rolling back application code

The `rollback` target's warning is correct to flag this even though the target itself isn't reachable: rolling back application code without considering schema state can break things worse than the original incident.

| Service | Migration tooling | Rollback story |
|---|---|---|
| `arm-core-be` | TypeORM migrations | Has a `migration:revert` script (`package.json`) — TypeORM's standard down-migration mechanism. Usable if the migration's `down()` method was actually implemented (not verified per-migration here — check the specific migration file before relying on it). |
| `apps/arm-admin` (backend) | TypeORM migrations, `migrationsRun: true` on startup | **No `migration:revert` script found in `package.json`.** Reverting a schema change here means writing and running a manual down-migration or a hand-written SQL fix — there's no one-command path. |
| `apps/arm-app-calendar` (backend) | Additive, idempotent startup backfills (`StartupMigrationService`, see [ARCHITECTURE.md](../calendar/ARCHITECTURE.md), [SYNC-FLOW.md](../calendar/SYNC-FLOW.md) §3) | Not applicable in the traditional sense — these migrations only ever add data to fields that don't exist yet; they don't alter MongoDB's schema-less structure and re-run safely on every boot. Rolling back application code doesn't require rolling these back. |

**If a migration ran as part of the deploy being rolled back, check whether the previous application code can actually run against the new schema before rolling back code alone** — this is exactly the scenario `CONFIRM_NO_SCHEMA_ROLLBACK` exists to force a human to think about. **Automated rollback answers that question for you, with "yes", every time**: `rollback-on-failure` passes the flag unattended because a bad image is the common case and fast recovery beats waiting for a human. That's the right default and a real hazard — after any auto-rollback, this section is the first thing to check, not an optional follow-up.

---

## 6. What NOT to do

- **Don't treat a successful auto-rollback as incident resolution.** The bad commit is still on the branch and the migration it ran is still applied (§2, §5).
- **Don't force-push over `dev` or `main`** as a way to "undo" a bad commit — these are shared branches that trigger deploys and that other people's work depends on. Use `git revert`.
- **Don't assume a revert on `dev` is inert until you choose to ship it** — per §3, the server builds each app repo's `dev` HEAD on the next deploy.
- **Don't skip the schema-compatibility check** (§5) — the underlying risk (old code, new schema) is exactly as real when rolling back via git revert + redeploy as via image tags.

---

## Related documents

- [CICD.md](../CICD.md) §4, §6 — the production deploy's tag/verify/rollback loop and the `repos.conf` branch-pinning gap, in the context of the whole pipeline
- [restart-services.md](restart-services.md) §3 — health-check verification after a rollback deploy, including per-service healthcheck strength caveats
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) — schema/migration detail behind §5's table
- `docs/guides/github-setup.md` — secrets required for `ci-push` (`DOCKER_USERNAME`/`DOCKER_PASSWORD`), needed only if the tag-based path is ever backed by a registry rather than the server's local image store
- [RELEASE-PROCESS.md](../RELEASE-PROCESS.md) — the same `repos.conf` branch-pinning finding, from the forward-release direction rather than rollback
