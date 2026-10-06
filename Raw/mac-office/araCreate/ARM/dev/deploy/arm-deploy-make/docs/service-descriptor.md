# Service Descriptor Contract

Any repo orchestrated by `arm-deploy-make` exposes a `make descriptor`
target. It prints the keys below as `KEY=value` lines, one per line, to
stdout — nothing else on stdout (no banners, no blank lines). This is
captured verbatim by `orchestrator describe`, so anything extra on stdout
is treated as part of the last key's value.

This is **not** the same thing as `services.conf`, which `orchestrator/`
already calls "the manifest" (see `orchestrator/README.md`). That file still
owns NAME/PATH/INSTALL_CMD/DEV_CMD for `install`/`dev`. The descriptor below
is a separate, additive contract for facts a repo states about itself that
drive real compose generation in both local dev and production.

## Fixed keys (always present)

| Key | Allowed values | Notes |
|---|---|---|
| `KIND` | `core` \| `app` \| `service` \| `util` | What kind of thing this repo is. `util` is a shared dependency consumed by other `core`/`app`/`service` entries via their own `DEPENDS_ON_LOCAL`/`DEPENDS_ON_PROD` — not exercised by any repo yet. |
| `INSTALL_CMD` | any shell command | Run with the repo root as CWD. |
| `DEV_CMD` | any shell command | Run with the repo root as CWD. |
| `DB_REQUIRED` | `yes` \| `no` | Whether this service needs a database at all. |
| `DB_ENGINE` | `postgres` \| `mongo` \| `none` | Must be `none` iff `DB_REQUIRED=no`. |
| `DB_LOCATION` | `shared` \| `external` \| `none` | Must be `none` iff `DB_REQUIRED=no`. |
| `DB_HOST` | `host:port`, or omitted | Required (and must be non-empty) only when `DB_LOCATION=external`. Omit entirely otherwise. |
| `DEPENDS_ON_LOCAL` | space-separated service NAMEs, or empty | Local-dev dependencies. Other services (by their `services.conf` NAME) that must be healthy before this one starts in `make local-dev`. |
| `DEPENDS_ON_PROD` | space-separated service NAMEs, or empty | Production dependencies — **often a different set from `DEPENDS_ON_LOCAL`**, not just different resolved names. e.g. local dev's `core-fe` waits on `session`+`calendar-fe`+`admin-fe` for Module Federation hot-reload; prod's `core-frontend` (a static build) only waits on `core-be`. |
| `HEALTH_PATH` | an HTTP path, e.g. `/v1/health/ready` | Polled by the container healthcheck. Must mean the same thing in both environments (verified true for all 8 current services). |
| `PORT` | an integer | The container-internal port the app's HTTP listener binds to **in local dev**. For prod backends this is the same port; for prod frontends it's unused — see "Frontends are structurally different" below. |
| `FE_BE` | `frontend` \| `backend` | Selects which template category this service uses, combined with `KIND` (see "Compose generation" below). |
| `LANGUAGE` | free text, e.g. `typescript` | Not validated beyond non-empty. Not consumed by `generate-compose` — informational/future use only. |
| `REQUIRED_ENV_VARS` | space-separated env var NAMEs, or empty | **Names only, never values or defaults** — secrets stay out of the service repo entirely. Each name must exist as a key in `arm-deploy-make`'s central `.env` — checked by `orchestrator generate-compose`, not `orchestrator describe`. Same names are used in both environments; only use this for a var whose *name* and *presence* (not value shape) is identical in local dev and prod. |
| `PROD_NAME` | e.g. `core-backend` | The compose **service key** in production (`services:\n  <PROD_NAME>:`) — also what `depends_on:`/Kong reference. Prod service keys don't follow any single naming convention across the platform (`core-backend`, `arm-session`, `notification-service`, ...) so this must be declared explicitly, not derived. |
| `PROD_CONTAINER_NAME` | e.g. `arm-core-backend` | The actual `container_name:` in production — **usually** `arm-` + `PROD_NAME`, but not mechanically derived (kept as its own explicit field since `session`'s service key is irregularly already `arm-session`, which would double-prefix under a "just add arm-" rule). Shows up in `docker ps`/logs/monitoring. |
| `PROD_IMAGE` | e.g. `arm-core-backend` | The image name in `image: <PROD_IMAGE>:${IMAGE_TAG:-latest}`. |
| `HOST_PORT` | an integer, or empty | Host-side port mapping for local dev (`"<HOST_PORT>:<PORT>"`). Empty means no `ports:` key at all (not every service needs host access, and this varies even within one KIND+FE_BE group). |
| `PROD_HOST_PORT` | an integer, or empty | Same, for production. Most backends are empty (Kong-only, no direct host access); `session` is a real exception (has its own host port in prod, unlike `core-be`/`calendar-be`/`admin-be`). |

## Variable-count keys (zero or more)

| Key | Format | Notes |
|---|---|---|
| `EXTRA_ENV_LOCAL_<N>` | `KEY: value` (N = 1, 2, 3, ...) | Local-dev-only environment content that doesn't fit `REQUIRED_ENV_VARS`' names-only shape — internal service URLs, renamed vars (e.g. `JWT_SECRET: ${JWT_ACCESS_SECRET}`), defaulted vars (`${VAR:-default}`), or plain literals. Each line becomes one `environment:` (or, for frontends, `build.args:`) entry, verbatim. |
| `EXTRA_ENV_PROD_<N>` | `KEY: value` (N = 1, 2, 3, ...) | Same idea, production values — often genuinely different shape from the `_LOCAL` counterpart, not just a different literal (e.g. `admin-be`'s DB host changes from `arm-postgres-dev` to `arm-postgres-prod`; `session`'s `CORE_BE_URL` changes from `http://arm-core-be-local:3000` to `http://core-backend:3000`). |

Numbering must start at 1 and be contiguous per scope (`EXTRA_ENV_LOCAL_1`,
`EXTRA_ENV_LOCAL_2`, ...) — order in the file is preserved into the
generated compose block. A service that needs none for a given scope just
omits those lines entirely.

**Escaping note for Makefile authors:** a value containing `${VAR}` (meant
to reach the generated compose file literally, for Docker Compose to
interpolate later — not to be expanded by `make` or the shell that runs the
recipe) needs `\$${VAR}` in the Makefile source: `$$` because `make` itself
eats one `$`, and `\$` because the shell running `echo` would otherwise
expand `${VAR}` from its own environment before it's ever written to the
descriptor. `echo "(secret removed): \$${POSTGRES_PASSWORD}"`
is correct; `$${POSTGRES_PASSWORD}` alone silently produces an empty value
instead of the literal string — a real bug caught during this contract's
rollout, not a hypothetical.

## Validation rules

- All fixed keys must be present exactly once (`DB_HOST` only required when `DB_LOCATION=external`).
- `KIND` must be one of the four listed values.
- `DB_REQUIRED=yes` requires `DB_ENGINE` in `{postgres, mongo}` and `DB_LOCATION` in `{shared, external}`.
- `DB_REQUIRED=no` requires `DB_ENGINE=none` and `DB_LOCATION=none`.
- `DB_LOCATION=external` requires `DB_HOST` to be present and non-empty.
- `FE_BE` must be `frontend` or `backend`.
- `LANGUAGE`, `PROD_NAME`, `PROD_CONTAINER_NAME`, `PROD_IMAGE` must all be non-empty.
- Each name in `REQUIRED_ENV_VARS` must exist as a key in `arm-deploy-make`'s central `.env` — checked by `orchestrator generate-compose`, not by `orchestrator describe`.

## Example

```
KIND=app
INSTALL_CMD=pnpm install --frozen-lockfile
DEV_CMD=pnpm start:dev
DB_REQUIRED=yes
DB_ENGINE=mongo
DB_LOCATION=shared
DEPENDS_ON_LOCAL=core-be
DEPENDS_ON_PROD=core-be
HEALTH_PATH=/v1/health/ready
PORT=4001
FE_BE=backend
LANGUAGE=typescript
REQUIRED_ENV_VARS=JWT_ACCESS_SECRET GOOGLE_CLIENT_ID GOOGLE_CLIENT_SECRET
PROD_NAME=calendar-backend
PROD_CONTAINER_NAME=arm-calendar-backend
PROD_IMAGE=arm-calendar-backend
HOST_PORT=4001
PROD_HOST_PORT=
EXTRA_ENV_LOCAL_1=CORE_API_URL: http://arm-core-be-local:3000
EXTRA_ENV_LOCAL_2=MONGODB_URI: ${MONGODB_URI:-mongodb://arm-mongodb-dev:27017/arm-calendar}
EXTRA_ENV_PROD_1=CORE_API_URL: http://core-backend:3000
EXTRA_ENV_PROD_2=MONGODB_URI: ${MONGODB_URI}
```

## FE+BE-in-one-repo convention

Services that ship both a frontend and backend half in one repo
(`arm-app-calendar`, `arm-admin`) do **not** use a `descriptor-be`/
`descriptor-fe` split on a single repo-root Makefile — `services.conf`
already paths each half to its own subdirectory (e.g. `calendar-be` →
`apps/arm-app-calendar/src/backend`, `calendar-fe` →
`apps/arm-app-calendar/src/frontend`), exactly like core-be/core-fe are two
independent repos. So each subdirectory gets its own real `Makefile` with a
plain `descriptor:` target — no new convention or orchestrator code needed.

## Compose generation

`orchestrator generate-compose` (`orchestrator/lib/generate-compose.sh`)
validates a service's descriptor, checks its `REQUIRED_ENV_VARS` names
exist in the env file, and fills a template to produce one generated
compose file. Two things distinguish this from a naive per-service
templating scheme:

**Templates are shared per `KIND`+`FE_BE`, not per literal service name.**
`orchestrator/templates/<kind>-<fe_be>.yml.tmpl` (and
`orchestrator/templates/prod/<kind>-<fe_be>.yml.tmpl` for production) is
looked up from the descriptor's own `KIND`/`FE_BE` values — e.g.
`app-backend.yml.tmpl` is the *one* template used to generate both
`calendar-be`'s and `admin-be`'s compose blocks. There are 5 categories
today (10 template files total — 5 local + 5 prod):

| Template | Local-dev consumers | Prod consumers |
|---|---|---|
| `service-backend` | notification | notification |
| `core-backend` | core-be, session | core-be, session |
| `core-frontend` | core-fe | core-fe |
| `app-backend` | calendar-be, admin-be | calendar-be, admin-be |
| `app-frontend` | calendar-fe, admin-fe | calendar-fe, admin-fe |

Everything that's genuinely common within a group (build shape, healthcheck
structure, network list, `CHOKIDAR_USEPOLLING`) lives directly in the
template. Everything that differs *even within* one group — internal
service URLs, DB wiring (Mongo vs dual-Postgres), renamed/defaulted vars —
lives in each service's own `EXTRA_ENV_LOCAL_<N>`/`EXTRA_ENV_PROD_<N>`
lines instead.

**`--env-scope local|prod`** controls everything that differs by
*environment* rather than by service: which `EXTRA_ENV_*`/`DEPENDS_ON_*`
fields are read, whether `{{NAME}}` resolves to the plain manifest name
(local, wrapped as `arm-{{NAME}}-local` by the local templates) or
`PROD_NAME` (prod, used as-is), and how `depends_on:` targets resolve —
local entries are always `arm-<dep>-local`; prod entries require fetching
each dependency's *own* descriptor to read its `PROD_NAME` (prod service
keys don't follow any single naming convention, so this can't be
string-built the way local dev's can).

`make local-dev` runs `generate-compose` automatically for all 8 services;
`make generate-compose-prod` (wired into `ci-deploy-prod`/`ci-build`/
`ci-push`/`rollback`) does the same against the prod template set. Neither
`docker-compose.local.dev.yml` nor `docker-compose.yml` (production) has
any hand-written app-service blocks left — both are just an `include:`
list plus, in production's case, Kong (which isn't a descriptor-driven app
repo, so stays hand-written).

### Frontends are structurally different from backends

Three things don't transfer cleanly from backends to frontends, all
handled by giving frontends their own template category rather than
special-casing within a shared one:

- **`PORT` means something different per environment for frontends.** In
  local dev it's the live Vite dev server's own port. In production,
  frontends build static assets served by nginx on a fixed internal port
  80 — prod frontend templates hardcode `80` for the container side and
  ignore the descriptor's `PORT` entirely (`HOST_PORT`/`PROD_HOST_PORT`
  still supply the host-side number).
- **Config is build args, not runtime `environment:`.** Prod frontends are
  a built artifact, not a live process reading env vars — `ENV_BLOCK`/
  `EXTRA_ENV_BLOCK` go under `build.args:` instead, which sits one indent
  level deeper than `environment:` does in backend templates (`args:` is
  nested inside `build:`). Frontend prod templates use the `_ARGS`
  variants of the block placeholders (`{{ENV_BLOCK_ARGS}}`,
  `{{EXTRA_ENV_BLOCK_ARGS}}` — 8-space child indent, not 6) for exactly
  this reason; using the plain `{{ENV_BLOCK}}`/`{{EXTRA_ENV_BLOCK}}` there
  produces invalid YAML (`build.args must be a mapping`) once
  `REQUIRED_ENV_VARS` is non-empty, or a blank-line-broken mapping when
  it's empty — both caught during this contract's rollout, not
  hypothetical.
- **`VITE_SESSION_URL`'s real production access pattern is unresolved,
  platform-wide.** None of the three prod frontends pass it as a build
  arg, despite their own Dockerfiles declaring it as an expected `ARG` —
  the frontend code falls back to `http://localhost:5001` baked into the
  built JS bundle, correct only if the browser is on the same host as the
  server. This is a real gap to investigate across all three frontends
  together, not a one-off fix for whichever service happens to be worked
  on next.

### Known engine pitfalls (both caught during this contract's rollout)

- **A descriptor value landing in both a template's hardcoded section and
  a `REQUIRED_ENV_VARS`/`EXTRA_ENV_*` line produces a silently-broken
  duplicate YAML key** — the last one wins, so a value can look correct in
  the template source and still be wrong at runtime. `docker compose
  config` does not catch this (it accepts the duplicate silently) —
  diffing the actual `environment:`/`args:` keys of a freshly generated
  file is the only reliable check.
- **Block placeholders that can legitimately be empty must be
  conditionally omitted, key and all** — not just their content. A
  service with no `DEPENDS_ON_*` needs the entire `depends_on:` key gone,
  not `depends_on:` followed by nothing (invalid YAML); a service with
  empty `REQUIRED_ENV_VARS` needs `{{ENV_BLOCK}}` to produce zero output
  lines, not one blank line (which breaks the parent mapping in some
  positions, e.g. `build.args:`). `generate-compose.sh` builds each block
  as either the fully-formed multi-line string (key included, where
  applicable) or empty, and only `printf`s it when non-empty.
