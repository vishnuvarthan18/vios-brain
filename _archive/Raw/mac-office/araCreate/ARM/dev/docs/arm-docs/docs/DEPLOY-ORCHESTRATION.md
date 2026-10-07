# ARM Platform — Deploy Orchestration

> How `deploy/arm-deploy-make` turns 8 independently-owned repos into a running stack: the `services.conf` manifest, each repo's self-declared **descriptor**, the `orchestrator` engine, and the shared templates that generate Compose service blocks for both environments.
>
> [INFRASTRUCTURE.md](INFRASTRUCTURE.md) covers *what each environment contains* — the components, networks, ports. This document covers *where those service definitions come from*, which since the descriptor rollout is no longer "someone hand-wrote them in a compose file."
>
> Verified against `services.conf`, `orchestrator/` (`bin/orchestrator`, `lib/*.sh`, `templates/**`), `make/{descriptor,kong,install,local-dev,ci,config}.mk`, `kong/kong.yml.tmpl`, `docker-compose.local.dev.yml`, `docker-compose.yml`, `docker-compose.edge.local.yml`, and each service's own `make descriptor` output (2026-08-19). The authoritative key-by-key contract lives in the deploy repo itself: [`deploy/arm-deploy-make/docs/service-descriptor.md`](../../../deploy/arm-deploy-make/docs/service-descriptor.md) — this document is the platform-level map through it, not a duplicate.

---

## 1. The shape of the thing

Three separate contracts, often confused with each other:

| Contract | File | Owns | Consumed by |
|---|---|---|---|
| **Manifest** | `services.conf` (deploy repo) | Where each service lives and how to install/run it outside Docker — `NAME`, `PATH`, `INSTALL_CMD`, `DEV_CMD` | `orchestrator install` / `orchestrator dev` → `make install`, `make dev-<service>` |
| **Descriptor** | `make descriptor` target in **each service repo** | What the service *is* — kind, ports, health path, DB engine, dependencies, prod naming, env vars | `orchestrator describe` / `generate-compose` → `make local-dev`, `make ci-deploy-prod` |
| **Template** | `orchestrator/templates/<kind>-<fe_be>.yml.tmpl` (+ `templates/prod/`) | The Compose shape common to a whole *category* of service | `orchestrator generate-compose` |

The manifest is a deploy-repo file that describes other repos. The descriptor is the inverse: each repo describes **itself**, and the deploy repo asks it. That inversion is the whole point — adding a service to production no longer means hand-writing a compose block in `arm-deploy-make`.

```text
service repo's Makefile          deploy repo
┌──────────────────┐             ┌──────────────────────────────────────┐
│ make descriptor  │──KEY=value──▶ orchestrator describe   (validate)    │
│  KIND=app        │             │ orchestrator generate-compose         │
│  PORT=4001       │             │   ├─ picks templates/<KIND>-<FE_BE>   │
│  FE_BE=backend   │             │   ├─ --env-scope local | prod         │
│  ...             │             │   └─ checks REQUIRED_ENV_VARS vs .env │
└──────────────────┘             └───────────────┬──────────────────────┘
                                                 ▼
                                  generated/docker-compose.<name>.yml
                                  generated/prod/docker-compose.<name>.yml
                                                 │  include:
                                                 ▼
                          docker-compose.local.dev.yml / docker-compose.yml
```

---

## 2. Two environments, both generated

There are exactly two environments, and every app service block in both is generated:

| Environment | Compose file | App service blocks |
|---|---|---|
| **Local** | `docker-compose.local.dev.yml` | **Generated** — an `include:` list of `generated/docker-compose.<name>.yml`, nothing else but networks |
| **Server** | `docker-compose.yml` | **Generated** — an `include:` list of `generated/prod/...`. Kong, Caddy, Authelia and the observability stack have hand-written *service blocks* here, but Kong's declarative *config* is generated too — see §2a |

A descriptor change therefore propagates to both environments automatically. There is nowhere left for a service definition to drift by hand.

**This was not true until 2026-08-19.** Two further environments existed — a shared CI/dev host (`docker-compose.dev.yml`, 758 lines) and staging (`docker-compose.staging.yml`, 640 lines) — whose app services were hand-written inline, so a descriptor change reached local and production only and drifted silently in the other two. Both files were deleted rather than migrated. The reasons:

- They were the *only* environments the engine did not generate, so deleting them took coverage to 100% without writing any new generation logic.
- Staging had silently lost its admin service while `kong/kong.yml.tmpl` still routed `/api/admin` to it — exactly the drift the hand-written files made possible.
- The observability stack existed in three near-identical copies (dev, staging, prod). It is now one.

The identifiers stay `prod` — `COMPOSE_PROD`, `generated/prod/`, `--env-scope prod`, `ci-deploy-prod`, and the `PROD_NAME` / `PROD_IMAGE` / `PROD_HOST_PORT` / `PROD_CONTAINER_NAME` descriptor keys. Those keys are declared in each service's own repo, so renaming them to `server` would mean changing eight repositories for no functional gain. **"Server" is the word for humans; "prod" is the identifier in code.**

Adding a pre-production environment back later is a template directory plus an `--env-scope` value — not another hand-written compose file.

## 2a. Kong's declarative config is generated too

`generated/` holds one more thing besides compose blocks: **`generated/kong.yml`**, rendered from
`kong/kong.yml.tmpl` by `make generate-kong-config` (`make/kong.mk`). Both compose files mount the
rendered file, and `ci-deploy-prod`, `up`, and `edge-up` each run the render first, so no path can start
Kong without it.

The reason is the same class of problem this whole document is about — a config that *looks* authored
but has to be produced. **Kong interpolates nothing in declarative config**: `${VAR}` is loaded as
literal text, and `{vault://env/...}` does not work either, because `jwt_secrets.secret` is not a
referenceable field in DB-less mode (both verified against `kong:3.7-ubuntu`, 2026-08-19). The file
previously shipped as `kong/kong.yml` carrying `secret: (secret removed) and Kong used that
placeholder text as the actual HMAC key — see [SECURITY.md](SECURITY.md) §11 `SEC-007` and
[INFRASTRUCTURE.md](INFRASTRUCTURE.md) §4.1.

The render fails closed rather than emitting something subtly wrong: it aborts if `JWT_ACCESS_SECRET` is
empty, aborts if any `{{...}}` placeholder is left unsubstituted (deleting the partial output), writes
under `umask 077`, and substitutes with bash parameter expansion rather than `sed` — the secret is base64
and routinely contains `/`, which breaks an `s///` command. Same reasoning as
`orchestrator/lib/generate-compose.sh`.

## 2b. Running the server's edge locally

`make edge-up` starts Kong and Caddy in front of the *already running* local stack, using the same
`generated/kong.yml` and `Caddyfile.prod` the server mounts — no forked config. Local containers are
joined to `arm-edge-local` under network **aliases** matching the server's compose service names
(`core-backend`, `calendar-backend`, `admin-backend`, `arm-session`), and they already listen on exactly the
internal ports those configs expect. It publishes only 80/443/8000/8100, so it runs alongside
`make local-dev`; `make edge-down` detaches cleanly.

This exists because the edge otherwise had no pre-server exercise at all — and it earned its keep
immediately, surfacing both `SEC-006` and `SEC-007` on its first run.

Note it does **not** run the whole server stack locally, which is not feasible: `docker-compose.yml`
wants ports 3000/3001/10000 (taken by the local stack), its service blocks point at `arm-*-prod` infra
hostnames declared in eight repos' descriptors, and `Caddyfile.prod` needs a real domain for ACME.

**Never hand-edit anything under `generated/`.** Both directories are gitignored and rewritten from scratch on every `make local-dev` / `make ci-deploy-prod`. An edit there survives exactly until the next run.

---

## 3. The manifest — `services.conf`

Tab-delimited, four columns, `#` comments ignored:

```
NAME  PATH  INSTALL_CMD  DEV_CMD
```

`PATH` is relative to the workspace `ROOT`. `INSTALL_CMD`/`DEV_CMD` are run with that path as CWD and are not tied to any package manager — `notification` delegates to its own `make install`/`make dev` while everything else runs `pnpm` directly.

| NAME | PATH |
|---|---|
| `core-be` | `core/arm-core-be` |
| `session` | `core/arm-session` |
| `core-fe` | `core/arm-core-fe` |
| `calendar-be` | `apps/arm-app-calendar/src/backend` |
| `calendar-fe` | `apps/arm-app-calendar/src/frontend` |
| `admin-be` | `apps/arm-admin/src/backend` |
| `admin-fe` | `apps/arm-admin/src/frontend` |
| `notification` | `services/arm-service-notification` |

These `NAME` values are the platform's canonical short ids — `DEPENDS_ON_LOCAL`/`DEPENDS_ON_PROD` in every descriptor reference them, `make local-dev-rebuild SERVICE=<name>` takes them, and the local container name is always `arm-<NAME>-local`.

**FE+BE-in-one-repo convention:** `arm-app-calendar` and `arm-admin` each ship two halves in one git repo, and the manifest simply paths each half to its own subdirectory. Each subdirectory has its own real `Makefile` with its own `descriptor:` target — there is no `descriptor-be`/`descriptor-fe` split, and the orchestrator needs no special case for it.

---

## 4. The descriptor — what each repo declares about itself

`make descriptor` prints `KEY=value` lines to stdout and **nothing else** — no banners, no blank lines. The output is captured verbatim, so a stray `echo` becomes part of the previous key's value.

Grouped by what they drive (full table, allowed values, and validation rules: [`docs/service-descriptor.md`](../../../deploy/arm-deploy-make/docs/service-descriptor.md)):

| Group | Keys |
|---|---|
| **Identity / role** | `KIND` (`core`\|`app`\|`service`\|`util`), `FE_BE` (`frontend`\|`backend`), `LANGUAGE` |
| **Run commands** | `INSTALL_CMD`, `DEV_CMD` |
| **Data** | `DB_REQUIRED`, `DB_ENGINE` (`postgres`\|`mongo`\|`none`), `DB_LOCATION` (`shared`\|`external`\|`none`), `DB_HOST` (only when `external`) |
| **Runtime** | `PORT`, `HEALTH_PATH`, `HOST_PORT`, `PROD_HOST_PORT` |
| **Ordering** | `DEPENDS_ON_LOCAL`, `DEPENDS_ON_PROD` |
| **Production naming** | `PROD_NAME`, `PROD_CONTAINER_NAME`, `PROD_IMAGE` |
| **Config** | `REQUIRED_ENV_VARS`, `EXTRA_ENV_LOCAL_<N>`, `EXTRA_ENV_PROD_<N>` |

Four of these carry non-obvious meaning:

- **`DEPENDS_ON_LOCAL` and `DEPENDS_ON_PROD` are genuinely different sets, not the same set under different names.** Local `core-fe` waits on `session` + `calendar-fe` + `admin-fe` because Module Federation hot-reload needs the remotes' dev servers alive; production `core-frontend` is a static build and waits only on `core-be`.
- **The three `PROD_*` naming keys are separate on purpose.** Production service keys follow no single convention (`core-backend`, `arm-session`, `notification-service`), so the compose service key, the `container_name`, and the image name are each declared rather than derived. `session` is the reason: its service key is already `arm-session`, which an "add `arm-` prefix" rule would turn into `arm-arm-session`.
- **`REQUIRED_ENV_VARS` is names only — never values, never defaults.** Secrets stay out of service repos entirely. Each listed name must exist as a key in `arm-deploy-make`'s central `.env`; that check runs in `generate-compose`, not `describe`. Use it only for vars whose *name and presence* are identical in local dev and prod.
- **`EXTRA_ENV_*_<N>` is the escape hatch for everything else** — internal service URLs, renamed vars (`JWT_SECRET: ${JWT_ACCESS_SECRET}`), defaulted vars, literals. Each line becomes one `environment:` entry verbatim (or one `build.args:` entry for frontends). Numbering starts at 1 and must be contiguous per scope; file order is preserved into the output.

**Escaping trap for whoever writes the Makefile:** a value meant to reach the generated compose file as a literal `${VAR}` (for Compose to interpolate later) must be written `\$${VAR}` in the Makefile — `$$` because `make` eats one `$`, and `\$` because the recipe's shell would otherwise expand it from its own environment first. Writing `$${POSTGRES_PASSWORD}` silently yields an empty value rather than the literal string. This was a real bug during rollout, not a hypothetical.

Validate one repo's descriptor without generating anything:

```sh
make descriptor-check NAME=calendar-be
```

---

## 5. Templates are shared per role, not per service

`orchestrator/templates/<KIND>-<FE_BE>.yml.tmpl` is looked up from the descriptor's own values. There are 5 categories, 10 files (5 local + 5 prod):

| Template | Consumers |
|---|---|
| `core-backend` | `core-be`, `session` |
| `core-frontend` | `core-fe` |
| `app-backend` | `calendar-be`, `admin-be` |
| `app-frontend` | `calendar-fe`, `admin-fe` |
| `service-backend` | `notification` |

Anything genuinely common to a whole category lives in the template — build shape, healthcheck structure, network list, `ulimits`, resource limits, `CHOKIDAR_USEPOLLING`. Anything that differs *within* a category lives in that service's own `EXTRA_ENV_*` lines instead: `calendar-be` and `admin-be` share `app-backend.yml.tmpl` despite one wiring MongoDB and the other two Postgres databases.

Placeholders the engine fills: `{{NAME}}`, `{{BUILD_CONTEXT}}`, `{{PORT}}`, `{{HEALTH_PATH}}`, `{{ENV_BLOCK}}`, `{{EXTRA_ENV_BLOCK}}`, `{{ARGS_BLOCK}}` (§7), `{{DEPENDS_ON_BLOCK}}`, `{{PORTS_BLOCK}}`, plus `{{PROD_IMAGE}}` and `{{PROD_CONTAINER_NAME}}` in the prod set.

`KIND=util` is defined in the contract but unused — no repo declares it today.

---

## 6. `--env-scope` — what differs by environment rather than by service

One flag controls every environment-dependent difference:

| | `--env-scope local` | `--env-scope prod` |
|---|---|---|
| Env lines read | `EXTRA_ENV_LOCAL_<N>` | `EXTRA_ENV_PROD_<N>` |
| Dependency set | `DEPENDS_ON_LOCAL` | `DEPENDS_ON_PROD` |
| Service key | `arm-{{NAME}}-local` | `PROD_NAME`, as-is |
| `depends_on:` targets | `arm-<dep>-local`, string-built | each dependency's **own** descriptor is fetched to read its `PROD_NAME` |
| Host port | `HOST_PORT` | `PROD_HOST_PORT` |
| Template set | `templates/` | `templates/prod/` |

Prod dependency resolution is the one place the engine has to read *other* services' descriptors: since prod service keys follow no convention, `depends_on: core-backend` cannot be derived from the string `core-be`.

---

## 7. Frontends are structurally different from backends

Three things don't transfer from backends, which is why frontends get their own template category rather than conditionals inside a shared one:

- **`PORT` means different things per environment.** Locally it's the live Vite dev server's port. In production the frontend is static assets served by nginx on a fixed internal port `80` — the prod frontend templates hardcode `80` and ignore `PORT` entirely. `HOST_PORT`/`PROD_HOST_PORT` still supply the host side.
- **Config is build args, not runtime env.** A built frontend doesn't read env vars at runtime, so prod frontends put their config under `build.args:` — nested inside `build:`, one indent level deeper than a backend's `environment:`. Prod frontend templates use a single `{{ARGS_BLOCK}}` placeholder that carries the same content as `{{ENV_BLOCK}}` + `{{EXTRA_ENV_BLOCK}}` at 8-space child indent **and includes the `args:` key itself**, so it can be omitted whole when a service declares neither `REQUIRED_ENV_VARS` nor `EXTRA_ENV_PROD_<N>`. Emitting the 6-space blocks there instead produces invalid YAML (`build.args must be a mapping`), and emitting a bare `args:` with nothing under it is equally invalid — both were hit during rollout.

  > The deploy repo's own `docs/service-descriptor.md` still describes this as two separate `{{ENV_BLOCK_ARGS}}`/`{{EXTRA_ENV_BLOCK_ARGS}}` placeholders. That was the earlier design; the templates and `generate-compose.sh` now use the single `{{ARGS_BLOCK}}`. Trust the templates.
- **`VITE_SESSION_URL` has no resolved production story, platform-wide.** None of the three prod frontends pass it as a build arg even though their Dockerfiles declare it as an expected `ARG`, so the built bundle falls back to `http://localhost:5001` — correct only when the browser is on the same host as the server. This is an open cross-frontend gap, not a per-service oversight, and the descriptor rollout did not resolve it.

---

## 8. Where it's wired into `make`

| Target | Runs | Output |
|---|---|---|
| `make local-dev` | `generate-compose` first, then brings the stack up | `generated/` (8 files) |
| `make generate-compose` | all 8 services, `--env-scope local` | `generated/` |
| `make generate-compose-prod` | all 8 services, `--env-scope prod`, prod templates | `generated/prod/` |
| `make ci-deploy-prod` | depends on `generate-compose-prod` | `generated/prod/` |
| `make ci-build` / `ci-push` / `rollback` | all depend on `generate-compose-prod` | `generated/prod/` |
| `make descriptor-check NAME=<name>` | `orchestrator describe`, validation only | nothing written |
| `make install` / `make dev-<service>` | `orchestrator install` / `dev` via the **manifest**, not the descriptor | nothing written |

Every image-producing or image-starting production target regenerates first, so a stale `generated/prod/` can't be deployed by accident.

The engine itself is standalone bash with no build step and no Node dependency — deliberate, since it runs before any application repo has been cloned or installed. It also contains no ARM-specific values: the service list, paths, and commands all arrive via `--manifest`/`--root`, so it can be extracted into its own repo later without a rewrite. Self-tests live in `tests/test-generate-compose.sh`, run by `make test`.

---

## 9. Known engine pitfalls

Both were caught during rollout, and neither is caught by `docker compose config`:

- **A value that appears in both a template's hardcoded section and a `REQUIRED_ENV_VARS`/`EXTRA_ENV_*` line produces a duplicate YAML key.** Compose accepts it silently and the last one wins, so the template can look correct while the running container isn't. Diffing the actual `environment:`/`args:` keys of a freshly generated file is the only reliable check.
- **Optional blocks must be omitted key and all, not left empty.** A service with no dependencies needs the whole `depends_on:` key gone (a bare `depends_on:` with nothing under it is invalid YAML); a service with empty `REQUIRED_ENV_VARS` needs `{{ENV_BLOCK}}` to emit zero lines, not one blank line — which breaks the parent mapping when it lands under `build.args:`. The engine builds each block as either a fully-formed multi-line string (key included) or the empty string, and only emits it when non-empty.

---

## 10. What is still hand-written, and why

| Thing | Why |
|---|---|
| **Kong** (in `docker-compose.yml`) | Not a descriptor-driven app repo — it has no repo in `repos.conf` and nothing to run `make descriptor` against |
| **Caddy, Authelia, observability stack** | Same reason — infrastructure containers from upstream images, not built from workspace repos |
| **`docker-compose.infra*.yml`** | Databases and Kafka — never in scope for the service descriptor |

---

## 11. Known gaps (summary)

| Gap | Where | Status |
|---|---|---|
| `VITE_SESSION_URL` isn't passed as a build arg by any prod frontend; the bundle falls back to `http://localhost:5001` | §7 | Verified, open, cross-frontend |
| `KIND=util` is specified but unexercised — no repo declares it | §4 | Verified, harmless |
| `orchestrator/README.md`'s Subcommands section documents only `install`/`dev`, omitting `describe` and `generate-compose` | deploy repo | Verified doc drift in the deploy repo itself |
| `docs/service-descriptor.md` documents `{{ENV_BLOCK_ARGS}}`/`{{EXTRA_ENV_BLOCK_ARGS}}`; the templates and engine use a single `{{ARGS_BLOCK}}` | §7 | Verified doc drift in the deploy repo itself |

---

## 12. Related documents

| Doc | Role |
|---|---|
| [`deploy/arm-deploy-make/docs/service-descriptor.md`](../../../deploy/arm-deploy-make/docs/service-descriptor.md) | The authoritative contract — every key, allowed values, validation rules, worked example |
| [`deploy/arm-deploy-make/orchestrator/README.md`](../../../deploy/arm-deploy-make/orchestrator/README.md) | The engine's own manifest-format and CLI reference |
| [INFRASTRUCTURE.md](INFRASTRUCTURE.md) | What each environment actually contains once these files are generated |
| [CICD.md](CICD.md) | Where `generate-compose-prod` sits in the deploy pipeline |
| [ENV-VARS.md](ENV-VARS.md) | The central `.env` that `REQUIRED_ENV_VARS` names are validated against |
| [`deploy/arm-deploy-make/CLAUDE.md`](../../../deploy/arm-deploy-make/CLAUDE.md) | Makefile module architecture and conventions |
