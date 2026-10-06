# ARM Platform — CI/CD Pipeline

> Every GitHub Actions workflow across all 8 repos, the branch-to-environment mapping, what `arm-deploy-make`'s deploy workflows actually do step by step, and where the pipeline is currently broken or non-operational as written.
>
> [`docs/guides/github-setup.md`](guides/github-setup.md) is the secrets/variables reference — this document does not duplicate it, only points at it. [INFRASTRUCTURE.md](INFRASTRUCTURE.md) covers what each Docker Compose file contains; this document covers how and when those files get invoked.
>
> Verified against every `.github/workflows/*.yml` file in all 8 repos, `deploy/arm-deploy-make/make/{ci,descriptor}.mk`, `repos.conf`, and `git log`/`git merge-base` history in `deploy/arm-deploy-make` and `core/arm-core-be` (**re-verified 2026-08-19**; the production deploy workflow has changed substantially since the 2026-08-10 pass — §4 and §6 note where an earlier finding has since been closed).

---

## 1. The two-track picture

There are **two independent CI/CD tracks** in this platform, and they don't talk to each other:

1. **Per-repo CI** — each of the 7 application repos (everything except `arm-deploy-make`) runs its own lint/test workflow on push/PR to `main`/`dev`, and most also run a `semantic-release`-based "release" workflow (versioning + changelog + GitHub release — **no Docker build, no push, no deploy**).
2. **Centralized deploy** — `deploy/arm-deploy-make`'s own three `deploy-*.yml` workflows are the **only** thing that actually deploys anything. They SSH into a target server and run `make ci-deploy-<env>`, which `git clone`/`pull`s every application repo fresh and builds Docker images directly from that source on the target server — never from an image registry, and never triggered by an application repo's own CI passing.

Pushing to `dev` in `arm-core-be`, for example, runs that repo's own `ci.yml` and `dev-release.yml` — neither of which deploys anything. A deploy only happens when someone pushes to `dev`/`staging`/`main` **in the `arm-deploy-make` repo itself**, or manually triggers one of its `workflow_dispatch` deploy workflows.

---

## 2. Branch strategy

| Branch | `arm-deploy-make` triggers | Deploy environment |
|---|---|---|
| `dev` | `ci.yml` | none — validation only |
| `main` | `deploy-production.yml`, `ci.yml` | `production` (the server) |

**`main` → server is the only automated deploy path.** Pushing to `dev` runs CI and deploys nothing. Until 2026-08-19 there were also `dev` → development and `staging` → staging paths; `deploy-dev.yml` and `deploy-staging.yml` were deleted along with those environments ([DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) §2).

Every application repo's own CI (`ci.yml`, `dev-lint-test.yml`, etc.) triggers on `dev`/`main` push and PR. Note that `arm-core-be`'s `deploy.yaml` still lists a `staging` branch trigger (§5) — that workflow lives in its own repo, was already broken for unrelated reasons, and was not touched by the environment removal.

**Verified, load-bearing gap: `repos.conf` pins every application repo to the `dev` branch, regardless of which environment is deploying.**

```
arm-core-be                  core/arm-core-be                  dev
arm-core-fe                  core/arm-core-fe                  dev
arm-session                 core/arm-session                      dev
arm-app-calendar             apps/arm-app-calendar             dev
arm-service-notification     services/arm-service-notification dev
arm-library-ui-components    library/arm-library-ui-components  dev
arm-app-admin                apps/arm-admin                     dev
```

`ci-deploy-prod` depends on `clone-repos` (§4), which reads this exact file. There is no `main`-branch column and no per-environment override anywhere in `make/repos.mk` or `config.mk`. **Practical effect: deploying to the server clones and builds each application repo's `dev` branch HEAD** — the same source code local development runs, not a promoted or tagged release. A push to `main` in `arm-deploy-make` selects *which deploy runs*; it does not select which application source gets built. Anything merged to an app repo's `dev` is therefore live on the server at the next deploy, with no pre-production environment where it would surface first.

**Second, unrelated drift found in the same file:** `arm-library-ui-components`'s configured `LOCAL_PATH` is `library/arm-library-ui-components`, but the directory actually present in this workspace is `library/arm-ui-library`. Either the GitHub repo was renamed after `repos.conf` was written, or this workspace's `library/` was never populated by `clone-repos` for that entry. `deploy/arm-deploy-make/CLAUDE.md`'s own "Service Paths" table repeats the same stale path (`PATH_LIB_UI`), so this isn't a one-off typo — both the config and its own documentation agree on a path that doesn't match reality.

---

## 3. Workflow inventory (all 8 repos)

| Repo | Workflow | Trigger | What it does |
|---|---|---|---|
| **arm-deploy-make** | `ci.yml` | push/PR: `main`, `dev` | Validates Makefile/compose syntax exists and parses — no app build, no deploy |
| | `deploy-production.yml` | push `main`, `workflow_dispatch` | SSH deploy to the server — the only deploy (§4) |
| **arm-core-be** | `ci.yml` | push/PR: `main`, `dev` | Lint + test |
| | `dev-release.yml`¹ | — | Not confirmed separately from `deploy.yaml`/`deploy-dev.yaml` below — no third lint/release-only file found; core-be's release automation, if any, isn't in a separately named file the way other repos have it |
| | `deploy.yaml` | push: `dev`, `staging`, `main` | **Live, broken** — Kubernetes deploy to a `k8s/` manifest set that doesn't exist in this repo (§5) |
| | `deploy-dev.yaml` | — (entire file commented out) | Dead — safe, never runs |
| **arm-core-fe** | `ci.yml` | push/PR: `main`, `dev` | Lint + test |
| | `e2e.yml` | push/PR: `main`, `dev` | Playwright E2E |
| **arm-session** | `dev-release.yml` | push: `main`, `dev` | Lint + test + `semantic-release` (versioning/changelog only, no deploy) |
| **arm-app-calendar** | `ci.yml` | push/PR: `main`, `dev` | Three jobs: `backend`, `frontend`, `sast` (static analysis) |
| **arm-admin** | — | — | **No `.github/workflows/` directory at all** — zero CI, zero release automation |
| **arm-service-notification** | `dev-release.yml` | push: `main`, `dev` | Lint + test + `semantic-release` |
| | `dev-lint-test.yml` | push/PR: `main`, `dev` | Lint + test (overlaps with `dev-release.yml`'s own lint/test steps) |
| | `deploy-dev.yaml.disabled` | — (`.disabled` suffix) | Dead by naming convention — GitHub Actions only picks up `.yml`/`.yaml` files in `.github/workflows/`, and even that extension wouldn't match; effectively inert either way |
| **arm-ui-library** | `ci.yml` | push/PR: `main`, `dev` | Lint + test |
| | `release.yml` | push: `main`, `dev` | `semantic-release` — this is what actually publishes new versions of `@aracreate/test-arm-ui` (see `docs/guides/npm-publish.md`) |

¹ `arm-core-be` doesn't follow the same `ci.yml` + `dev-release.yml` split every other repo uses — its own `ci.yml` covers lint/test, and there's no separate semantic-release workflow file in this repo the way `arm-session`, `arm-service-notification`, and `arm-ui-library` each have one.

**Verified: `arm-admin` has zero CI/CD of any kind.** No `.github/workflows/` directory exists in `apps/arm-admin` at all — no lint, no test, no release automation. **This has become more consequential, not less:** the admin app deploys to the server ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §7), so its code reaches the only deployed environment with nothing enforcing quality on pushes to its own repo — and, per §2, there is no intermediate environment where an admin regression would surface first.

---

## 4. `arm-deploy-make`'s deploy workflow — job by job

`deploy-production.yml` is the only deploy workflow (361 lines). It does image tagging, health-gated verification, and automatic rollback:

```
validate → connect → prepare → deploy → verify → rollback-on-failure (if: failure())
```

**It declares a `concurrency:` group** (`deploy-production`, `cancel-in-progress: false`). Two overlapping deploys would `rsync` into the same `~/arm-deploy` directory on the server and could write `.last-good-tag` from a still-in-flight run, breaking the assumption that a recorded tag was independently verified. Cancellation is deliberately disabled: queuing behind a live deploy is safer than killing one mid-flight.

1. **`validate`** — checks every required secret is non-empty and that expected compose/make files exist in the checked-out `arm-deploy-make` repo. Does not touch the target server.
2. **`connect`** — SSH key setup, then a bare `ssh ... "echo 'SSH connection OK'"` liveness check.
3. **`prepare`** — the actual `.env` generation flow:
   - `rsync`s the `arm-deploy-make` repo itself to `~/arm-deploy/` on the target server, **excluding** `core/`, `apps/`, `services/`, `library/` (the application source dirs aren't part of this rsync — they get cloned separately by `make ci-deploy-prod`'s own `clone-repos` step once the SSH deploy step runs, per §2/§3 of `make/ci.mk`).
   - Writes `~/arm-deploy/.env` on the server. **This step was rewritten for security and no longer works the way earlier versions of this document described.** Every value is bound through the step's `env:` block and referenced as a real shell variable (`$VAR`), never spliced into the script body as `${{ }}` text; the file is assembled locally with a heredoc and shipped with `scp` rather than reconstructed remotely via `ssh host "printf ... >> .env"`. The distinction is load-bearing: GitHub Actions substitutes `${{ }}` expressions as raw text *before* the runner's shell parses the script, so a secret containing a backtick, `$(...)`, or a quote would be re-parsed as shell syntax — reproduced locally during the repo's own security audit. With the current shape, no secret value ever appears inside command text on either machine. (Also writes `authelia/users_database.yml`, base64-round-tripped because Argon2 hashes contain literal `$` characters.)
4. **`deploy`** — SSHes in and runs `make ci-deploy-prod ROOT=~/arm-deploy`, computing `(secret removed){GITHUB_SHA::7}` and passing it in, then runs `make backup-cron-install` (idempotent, so a server rebuild silently re-installs the nightly backup job). On failure, dumps the last N lines of logs from the key containers before failing the job.
5. **`verify`** — polls up to 30 times at 10-second intervals, checking both `State.Status` **and** `State.Health.Status` for all 20 expected containers (8 app + Kong + Caddy + Authelia + the 10 observability containers). `healthy` and `none` pass; `unhealthy` fails immediately with a log dump; anything still starting stays in the pending set for the next round. Containers that never settle inside the window fail the job.
6. **`rollback-on-failure`** (`if: failure()`) — reads `~/arm-deploy/.last-good-tag` and runs `make rollback TAG=$LAST_GOOD CONFIRM_NO_SCHEMA_ROLLBACK=yes`. It bails out with a clear message if no marker exists (first-ever deploy, or verify has never once passed).

**The `.last-good-tag` marker is written at the end of `verify`, never before** — so a tag can only be recorded if every container it produced was independently confirmed healthy. That ordering is what makes the automated rollback target trustworthy rather than "roll back to the previous thing, which may also have been broken."

**Verified bug, now fully closed — the `verify` steps used to check for admin containers their own compose file never defined.** Commit `2f4ec207` (2026-06-30) added the admin service blocks to `docker-compose.dev.yml` **only**, while adding admin-container verify checks to all three deploy workflows of the time. Production was fixed as a side effect of the descriptor-driven compose work, which gave the admin app real server services and a Kong `admin-api` route ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4, §7). Staging remained broken — its `verify` looked for `arm-admin-backend-staging`/`arm-admin-frontend-staging` while `docker-compose.staging.yml` defined neither, so the next push to `staging` would have failed `verify` outright. **That environment and its workflow were deleted on 2026-08-19, which closed the last live case.** `deploy-production.yml` checks `arm-admin-backend`/`arm-admin-frontend`, both of which the server compose file defines.

---

## 5. `arm-core-be`'s broken parallel deploy path

**Verified, currently live:** `core/arm-core-be/.github/workflows/deploy.yaml` (`name: Multi-Environment Deploy`) triggers on every push to `dev`, `staging`, and `main` — same branches as the repo's own `ci.yml`. It is a **complete Kubernetes deploy pipeline**: Docker build+push to Docker Hub, `kubectl apply` against a cluster via `KUBE_CONFIG`, namespace creation per environment, and `envsubst` templating of `./k8s/core-config.yaml`, `./k8s/deployment.yaml`, `./k8s/service.yaml`, `./k8s/ingress.yaml`.

**None of those `k8s/*.yaml` files exist in this repo** (`ls k8s/` returns nothing). This workflow has no relationship to the Docker Compose + SSH deploy model every other document in this platform describes (`ARCHITECTURE.md`, `INFRASTRUCTURE.md`, this document's own §1) — it appears to predate that model and was never removed. It will fail at the `envsubst < ./k8s/core-config.yaml` step on every single push to `dev`, `staging`, or `main` in `arm-core-be` (assuming it even gets that far — it fails earlier still if `DOCKERHUB_USERNAME`/`DOCKERHUB_PASSWORD`/`KUBE_CONFIG` secrets aren't configured for this repo, which is likely given no Kubernetes cluster is referenced anywhere else in the platform's documentation).

A second file, `deploy-dev.yaml`, sits next to it in the same directory — **every line is commented out**, so it's inert. It's worth knowing it's there only so it isn't mistaken for a working alternative; it isn't running at all, dead or otherwise.

**Practical impact:** this doesn't block real deploys (§1's actual deploy path runs entirely from `arm-deploy-make`, independent of this workflow's pass/fail state), but it does mean `arm-core-be`'s GitHub Actions checks list a permanently-red "Multi-Environment Deploy" run on every push — worth knowing before assuming a red X on a core-be PR means the application code is broken.

---

## 6. Makefile CI/CD targets (`make/ci.mk`)

| Target | Depends on | What it does |
|---|---|---|
| `ci-setup` | `env-check`, `check-docker`, `infra-up`, `infra-wait` | Bring up infra only — used before `ci-test` |
| `ci-test` | `ci-setup`, `install`, `test` | Full local CI test run |
| `ci-build IMAGE_TAG=<sha>` | `env-check`, **`generate-compose-prod`** | `docker compose -f docker-compose.yml build` with the given tag |
| `ci-push IMAGE_TAG=<sha>` | **`generate-compose-prod`** | Logs into Docker Hub (`DOCKER_USERNAME`/`DOCKER_PASSWORD`), pushes the tagged images |
| `ci-deploy-dev` | `env-check`, `check-docker`, `clone-repos`, `infra-up`, `infra-wait` | `docker compose -f docker-compose.dev.yml down` → `build --no-cache --pull` → `up -d` |
| `ci-deploy-prod` | same, `infra-up-prod`/`infra-wait-prod`, **`generate-compose-prod`** | `up -d --build` against `docker-compose.yml` — **no blanket `down`** |
| `rollback TAG=<sha>` | **`generate-compose-prod`** | `docker compose -f docker-compose.yml up -d --no-build` at the given `IMAGE_TAG`, gated behind a `CONFIRM_NO_SCHEMA_ROLLBACK=yes` flag with an explicit warning that this rolls back app images only, not the database schema |

**Every production-facing target now regenerates the compose files first.** `generate-compose-prod` rebuilds `generated/prod/docker-compose.<name>.yml` for all 8 services from their own descriptors before anything is built, pushed, deployed, or rolled back — so a stale generated file can't reach production, and the compose file a rollback starts from is the same one the deploy used. See [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) §8.

**`ci-deploy-prod` deliberately does not `down` the stack first**, because that would take the whole platform offline simultaneously. `up -d --build` recreates only the services whose image or config actually changed, one at a time, gated by the `condition: service_healthy` waits in the generated compose files. The trade-off is documented in `ci.mk` itself: a service *removed* from the manifest leaves its old container running as an orphan, and `--remove-orphans` isn't available as a fix because it would also stop infra containers sharing `arm-infra-network`. Decommissioning a service is a manual cleanup step.

**Correction — `rollback` is now wired up in production. This reverses this document's previous finding.** The earlier pass found `rollback` fully coded but unreachable, because nothing in the pipeline produced an image it could roll back to. Three changes closed that:

1. **Every production deploy gets its own image tag.** `deploy-production.yml` computes `IMAGE_TAG=${GITHUB_SHA::7}` and passes it into `ci-deploy-prod`, instead of everything overwriting `latest` in place. Because `ci-deploy-prod` builds on the target server, the previous deploy's images remain on that server's disk under their own tag — `rollback` starts them with `--no-build` and needs no registry pull at all. This is why the mechanism works without `ci-push` ever running.
2. **`verify` records `.last-good-tag`** on the server, but only after every container is confirmed healthy (§4).
3. **`rollback-on-failure`** runs automatically when `deploy` or `verify` fails, rolling back to that marker.

**One real limit remains: the rollback covers app images, never the database.** The automated job passes `CONFIRM_NO_SCHEMA_ROLLBACK=yes` on the operator's behalf, which is the right call for the common case (a bad image) and the wrong one if the failed deploy ran a non-backward-compatible migration — the workflow emits an explicit `::warning::` about exactly that rather than letting it be silently assumed away. Treat a red deploy that auto-rolled back as "recovered, migration state unverified," not "resolved."

---

## 7. `.env` file generation flow (summary)

```
GitHub Environment secrets/vars (production)
        ↓ bound into the step's env: block (never spliced as ${{ }} script text)
heredoc → /tmp/<env>.env on the runner, umask 077
        ↓ scp
~/arm-deploy/.env on the target server
        ↓
make ci-deploy-<env>            — env-check validates required vars are present
        ↓
generate-compose(-prod)         — REQUIRED_ENV_VARS names checked against this .env
        ↓
docker compose --env-file .env  — every service reads its config from this file
```

This is the same `REQUIRED_CORE_VARS`/`REQUIRED_OPTIONAL_VARS` tiering documented in [`deploy/arm-deploy-make/CLAUDE.md`](../../../deploy/arm-deploy-make/CLAUDE.md) and [ENV-VARS.md](ENV-VARS.md) — `env-check` enforces the same two-tier rule in CI as it does for a local `make arm`. For the actual secret names, generation commands, and the already-documented `openssl rand -hex 16` vs. `-hex 32` drift in `docs/guides/github-setup.md`'s quick-reference section, see that doc and [ENV-VARS.md](ENV-VARS.md) §10 rather than this one — this document only covers how the values get from GitHub into the running containers, not which values are required.

---

## 8. Known pitfalls / gaps (summary)

| Gap | Where | Status |
|---|---|---|
| `repos.conf` pins every application repo to `dev` — the server deploys `dev` HEAD, not a promoted branch | §2 | Verified, load-bearing — affects release-process assumptions platform-wide |
| `arm-library-ui-components`'s configured clone path (`library/arm-library-ui-components`) doesn't match the actual directory (`library/arm-ui-library`) | §2 | Verified, also repeated in `deploy/arm-deploy-make/CLAUDE.md`'s own path table |
| `arm-admin` has zero `.github/workflows/` — no CI, no release automation | §3 | Verified ([INFRASTRUCTURE.md](INFRASTRUCTURE.md) §7) |
| `core-be`'s `deploy.yaml` is a live, currently-broken Kubernetes deploy workflow with no relationship to the platform's actual Docker Compose + SSH model | §5 | Verified — fails on every push to `dev`/`staging`/`main` in that repo |
| Automated rollback reverts app images only; a non-backward-compatible migration from the failed deploy is not undone | §6 | Verified — the workflow warns explicitly rather than assuming it away |

**Closed since this document's previous pass (2026-08-10):**

| Former gap | Closed by |
|---|---|
| `.env` written on the server via `printf '${{ secrets.X }}'` chains, re-parsed by the remote shell | `env:`-bound heredoc + `scp` in all three workflows (§4) |
| Production `verify` checks admin containers its compose file doesn't define | Admin now deployed to production (§4) |
| `rollback` unreachable from any workflow | Per-deploy `IMAGE_TAG`, `.last-good-tag`, `rollback-on-failure` (§6) |
| No protection against overlapping deploys racing on the same server directory | `concurrency:` group on all three workflows (§4) |

---

## 9. Related documents

| Doc | Role |
|---|---|
| [`docs/guides/github-setup.md`](guides/github-setup.md) | Every required GitHub secret/variable, by environment — the authoritative list this doc doesn't duplicate |
| [ENV-VARS.md](ENV-VARS.md) | Canonical env-var reference, including the `hex 16` vs. `hex 32` drift in `github-setup.md`'s quick reference |
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) | What each Docker Compose file actually contains — the thing these workflows start |
| [DEPLOY-ORCHESTRATION.md](DEPLOY-ORCHESTRATION.md) | What `generate-compose-prod` does before every build/push/deploy/rollback, and why production's compose file has no hand-written app blocks |
| [`deploy/arm-deploy-make/CLAUDE.md`](../../../deploy/arm-deploy-make/CLAUDE.md) | Makefile module architecture, `make arm` setup wizard, Makefile coding conventions |
| [`docs/guides/npm-publish.md`](guides/npm-publish.md) | What `arm-ui-library`'s `release.yml` actually publishes to (plain npm, not Verdaccio — see [INFRASTRUCTURE.md](INFRASTRUCTURE.md) §1) |
| [`runbooks/deploy-rollback.md`](runbooks/deploy-rollback.md) | The operational "what do I actually do" companion to this document's §2/§6 findings — the practical git-revert-based rollback procedure |
| [RELEASE-PROCESS.md](RELEASE-PROCESS.md) | Release vs. deploy as two unrelated mechanisms, and the `npmPublish: false` finding for `arm-ui-library` |
