# ARM Platform — Local Development Setup

Full hot-reload stack in Docker. All services run with `src/` volume mounts — edit any file and the change is live in under a second. No Caddy, no Kong.

> **Where the service definitions live:** `docker-compose.local.dev.yml` contains no service blocks — it `include:`s eight generated files, each built from that service's own repo descriptor at `make local-dev` time. To change how a service runs locally (its port, env, dependencies), edit that repo's `descriptor:` Makefile target, not the compose file. See [DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md).

---

## What you get

| Service | Local URL | Hot Reload |
| --- | --- | --- |
| Core Frontend | `http://localhost:3000` | Vite HMR |
| Calendar Frontend | `http://localhost:3001` | Vite HMR |
| ARM session service | `http://localhost:5001` | NestJS watch |
| Core Backend | `http://localhost:4000` | NestJS watch |
| Calendar Backend | `http://localhost:4001` | NestJS watch |
| Notification Service | — | NestJS watch (Kafka consumer only) |

> **Note:** The session service is exposed on port **5001** (not 5000). Port 5000 is the container-internal port; `5001` is the host-side port. All browser-side API calls go to `http://localhost:5001`.

---

## Architecture overview

```text
Browser
  ├── http://localhost:3000  →  arm-core-fe-local      (Vite dev server)
  │                              └── MFE loads from http://localhost:3001/calendar/remoteEntry.js
  ├── http://localhost:3001  →  arm-calendar-fe-local   (Vite dev server)
  └── http://localhost:5001  →  arm-session-local           (session + proxy)
                                  ├── http://arm-core-be-local:3000     (internal)
                                  └── http://arm-calendar-be-local:4001 (internal)

Infrastructure (shared Docker network: arm-infra-network)
  PostgreSQL 16   arm-postgres-dev:5432
  MongoDB 7       arm-mongodb-dev:27017
  Redis 7         arm-redis-dev:6379
  Kafka 3.8       arm-kafka-dev:9092   (KRaft — no Zookeeper)
```

**Service startup order** (enforced by Docker health checks):

```text
arm-core-be-local (healthy)
  └── arm-session-local (healthy)
        └── arm-core-fe-local
  └── arm-calendar-be-local (healthy)
        └── arm-calendar-fe-local (healthy)
              └── arm-core-fe-local
```

The frontends are last to start. `make local-dev` returns immediately but it takes 60–90 seconds for all services to become reachable while health checks pass in order.

---

## Prerequisites

- **Docker Desktop** — installed and running
- **pnpm** — required for native dev mode and `make install`; auto-installed by `make install` if missing
- **Git** — SSH or HTTPS access to the GitHub org (`aracreate-group`)
- **Google Cloud project** — OAuth 2.0 credentials (Client ID + Secret)
- **SMTP credentials** — for the notification service (signup emails, password reset)

---

## Step 1 — Navigate to the deploy directory

All `make` commands run from here:

```bash
cd deploy/arm-deploy-make
```

Every command in this document assumes you are in `deploy/arm-deploy-make/`. You can also run any command from the repo root with:

```bash
make -C deploy/arm-deploy-make <target>
```

---

## Step 2 — Create and fill in `.env`

```bash
cp .env.example .env
```

Open `.env` and fill in every blank value. The table below lists what is required.

### Generate secrets

Run these once and paste the output into `.env`:

```bash
# JWT — one each
openssl rand -base64 32   # → JWT_ACCESS_SECRET
openssl rand -base64 32   # → JWT_REFRESH_SECRET

# Session and session service
openssl rand -hex 32      # → SESSION_COOKIE_SECRET
openssl rand -hex 32      # → SESSION_INTERNAL_SECRET

# Google token encryption — MUST be exactly 64 hex characters (32 bytes for AES-256)
openssl rand -hex 32      # → GOOGLE_TOKEN_ENCRYPTION_KEY
```

### Required `.env` values

| Variable | What to put |
| --- | --- |
| `POSTGRES_PASSWORD` | any strong password |
| `MONGO_ROOT_PASSWORD` | any strong password |
| `MONGODB_URI` | `mongodb://(secret removed)@arm-mongodb-dev:27017/arm-calendar?authSource=admin` |
| `REDIS_PASSWORD` | any strong password |
| `JWT_ACCESS_SECRET` | `openssl rand -base64 32` |
| `JWT_REFRESH_SECRET` | `openssl rand -base64 32` |
| `SESSION_COOKIE_SECRET` | `openssl rand -hex 32` |
| `SESSION_INTERNAL_SECRET` | `openssl rand -hex 32` |
| `GOOGLE_CLIENT_ID` | Google Cloud Console → Credentials |
| `GOOGLE_CLIENT_SECRET` | Google Cloud Console → Credentials |
| `GOOGLE_TOKEN_ENCRYPTION_KEY` | `openssl rand -hex 32` — must be **exactly 64 hex characters** |
| `SMTP_HOST` | your SMTP server hostname |
| `SMTP_USER` | your SMTP username / sender address |
| `SMTP_PASS` | your SMTP password |

**MONGODB_URI note:** Replace `<MONGO_ROOT_PASSWORD>` with the same value you set in `MONGO_ROOT_PASSWORD`. The hostname `arm-mongodb-dev` is the Docker container name — do not change it.

**Google OAuth note:** The callback URLs for local dev are set automatically by the Docker Compose file. Add `http://localhost:4000/v1/auth/google/callback` and `http://localhost:4001/auth/google/callback` to your Google Cloud Console Authorised redirect URIs.

### Verify your env before starting

```bash
make env-check
```

This prints `OK` for every required variable or `MISSING` for anything not set. Fix all `MISSING` values before proceeding.

---

## Step 3 — Clone all repositories

```bash
make clone-repos
```

Reads `repos.conf` and clones 6 repositories from `GITHUB_BASE_URL` (defaults to `https://github.com/aracreate-group`) into the workspace:

```text
arm/
  core/
    arm-core-be               ← Core backend   (NestJS, PostgreSQL, Redis, Kafka)
    arm-core-fe               ← Core frontend  (React, Vite, Module Federation host)
    arm-session                   ← session service            (NestJS, session proxy)
  apps/
    arm-app-calendar/
      apps/
        backend               ← Calendar backend  (NestJS, MongoDB, Redis)
        frontend              ← Calendar frontend (React, Vite, MFE remote)
  services/
    arm-service-notification  ← Email service  (NestJS, Kafka consumer)
  library/
    arm-library-ui-components ← UI component library
```

If a repo already exists locally, this command pulls the latest instead of re-cloning.

**Check repo status at any time:**

```bash
make repos-status    # shows branch, and how many commits behind remote each repo is
make update-repos    # pull latest for all repos (alias for clone-repos)
```

---

## Step 4 — Start infrastructure

```bash
make infra-up
```

Starts four containers on the `arm-infra-network` Docker network:

| Container | Image | Port |
| --- | --- | --- |
| `arm-postgres-dev` | postgres:16 | `127.0.0.1:5432` |
| `arm-mongodb-dev` | mongo:7 | `127.0.0.1:27017` |
| `arm-redis-dev` | redis:7 | `127.0.0.1:6379` |
| `arm-kafka-dev` | apache/kafka:3.8 | `127.0.0.1:9092` (KRaft, no Zookeeper) |

All four are bound to loopback rather than published on all interfaces, so `psql`/`mongosh`/`redis-cli` work from your machine exactly as before, but nothing else on your network can reach them.

Then wait for all containers to pass their health checks:

```bash
make infra-wait
```

This polls each service and blocks until all four are healthy. It also creates the required Kafka topics (`send_email`, `send_welcome_email`, `send_password_reset_email`, `notification.dlq`). On first run with a cold Docker cache, Kafka can take up to 30 seconds — this is normal.

```text
  Waiting for PostgreSQL ......... ready
  Waiting for MongoDB ............ ready
  Waiting for Redis .............. ready
  Waiting for Kafka (may take ~30s) ...... ready
  Creating Kafka topics .......... ready
  All infrastructure is healthy and ready
```

Verify at any time:

```bash
make infra-status    # show container status (should all be "healthy")
```

> **Data persistence:** Infrastructure volumes are never deleted by `make local-dev-down` or `make clean-apps`. Your database data survives container restarts. To wipe data completely: `make clean-volumes FORCE=true`.

---

## Step 5 — Build the local dev images

```bash
make local-dev-build
```

Builds all 6 Docker images using the `Dockerfile.local.dev` files in each service repo. This runs `pnpm install` inside each container and sets up the file watchers.

Run this **once on first setup**, and again any time you:

- Add or update a package in any `package.json`
- Change `pnpm-lock.yaml` in any service

> This takes 5–15 minutes on the first run (Docker pulls Node base images and installs all dependencies). Subsequent rebuilds are faster due to layer caching.

---

## Step 6 — Start the stack

```bash
make local-dev
```

Starts all 6 services. Services start in dependency order (core-be → session → calendar-be → calendar-fe → core-fe). Wait 60–90 seconds for all health checks to pass, then open <http://localhost:3000>.

```text
Stack is up — hot reload active on all services

  Core frontend    →  http://localhost:3000  (Vite HMR)
  Calendar MFE     →  http://localhost:3001  (Vite HMR)
  arm-session          →  http://localhost:5001
  core-backend     →  http://localhost:4000
  calendar-backend →  http://localhost:4001

  Edit any file under src/ and changes apply instantly.
```

**Hot reload behaviour:**

- **Backends** — `nest --watch` restarts the NestJS process on any `src/` file change (< 1 second)
- **Frontends** — Vite HMR replaces modules in the browser without a full page reload

**Why no Kong or Caddy:**
Both are intentionally absent in local dev. Auth guards in both backends have a JWT fallback that activates when the `X-User-Id` header (normally injected by Kong) is absent. That fallback is the active code path in local dev. The session service calls backends directly using their Docker container names.

---

## Day-to-day commands

### Logs

```bash
make local-dev-logs          # tail all service logs
make local-dev-logs-be       # backends only (core-be, calendar-be, session, notification)
make local-dev-logs-fe       # frontends only (core-fe, calendar-fe)
```

For individual infrastructure logs:

```bash
make infra-logs              # all infra logs
make infra-logs-postgres
make infra-logs-mongodb
make infra-logs-redis
make infra-logs-kafka
```

### Status

```bash
make local-dev-ps            # show all local-dev container status + health
make infra-status            # show infra container status
make health                  # curl all service /health endpoints and print OK/FAIL
```

### Control

```bash
make local-dev-restart       # stop + start (no image rebuild)
make local-dev-down          # stop all app containers (infra keeps running)
make infra-down              # stop infra (data preserved)
```

---

## Re-running after the first setup

On subsequent days (infra data is still on disk):

```bash
make infra-up        # start infra containers (instant if already built)
make infra-wait      # confirm all healthy
make local-dev       # start the app stack
```

---

## Updating dependencies

If you modify `package.json` or `pnpm-lock.yaml` in any service, rebuild only that service's image:

```bash
make local-dev-down
make local-dev-build
make local-dev
```

Or rebuild everything from scratch (no cache):

```bash
make local-dev-down
make generate-compose      # refresh generated/ before building against it
docker compose -f docker-compose.local.dev.yml build --no-cache
make local-dev
```

(`make local-dev` runs `generate-compose` itself; the explicit call is only needed when invoking `docker compose` directly, since `docker-compose.local.dev.yml` `include:`s files that may not exist yet on a fresh clone.)

---

## Alternative: Native pnpm dev (single service)

If you are working on a single service and want faster iteration without Docker build overhead, run it natively. Infrastructure must still be running in Docker.

**Requirements:** `pnpm` installed locally, infra running (`make infra-up && make infra-wait`).

```bash
make dev-core-be        # core-backend   → http://localhost:4000
make dev-session            # arm-session        → http://localhost:5000
make dev-core-fe        # core-frontend  → http://localhost:3000
make dev-calendar-be    # calendar-be    → http://localhost:4001
make dev-calendar-fe    # calendar-fe    → http://localhost:3001
make dev-notification   # notification   → (Kafka consumer, no HTTP)
```

Each command runs `pnpm start:dev` (backends) or `pnpm dev` (frontends) inside the repo directory with full hot reload.

**Typical workflow:** run the service you are editing natively, keep all others in Docker.

> **Note:** When running the session service natively, it listens on port `5000` (not 5001 — that mapping only exists in Docker). Update `VITE_SESSION_URL` in your frontend `.env` accordingly if running both natively.

---

## Running tests

```bash
# Run all unit tests (no infra needed)
make test

# Run all tests with infra spun up automatically
make test-all

# Individual services
make test-core           # core-backend
make test-session            # arm-session
make test-calendar       # calendar-backend
make test-notification   # notification service

# With coverage reports
make test-core-cov
make test-session-cov
make test-calendar-cov
make test-notification-cov

# Smoke test (hits /health on every running service)
make smoke-test
```

---

## First-time setup checklist

```text
[ ] Docker Desktop is running
[ ] cd deploy/arm-deploy-make
[ ] cp .env.example .env
[ ] Fill in all required values (see Step 2 table above)
[ ] make env-check              — confirm no MISSING variables
[ ] make clone-repos
[ ] make infra-up
[ ] make infra-wait             — wait until all 4 services show "ready"
[ ] make local-dev-build        — takes 5–15 min first time
[ ] make local-dev
[ ] Wait 60–90 seconds for health checks to cascade
[ ] Open http://localhost:3000
```

Or use the one-command setup (does clone + install + infra automatically):

```bash
make setup    # env-check + clone-repos + install + infra-up + infra-wait
```

Then run `make local-dev-build` and `make local-dev` to start the stack.

---

## Troubleshooting

### Container exits immediately on startup

```bash
make local-dev-ps        # check which container is not running
make local-dev-logs      # read the error from the failing service
```

Common causes:

| Symptom | Cause | Fix |
| --- | --- | --- |
| `MISSING: JWT_ACCESS_SECRET` | `.env` incomplete | Run `make env-check` and fill in all missing values |
| `MongoServerError: Authentication failed` | `MONGODB_URI` password doesn't match `MONGO_ROOT_PASSWORD` | Update `MONGODB_URI` in `.env` — the password must be identical |
| `Error: Invalid key length` | `GOOGLE_TOKEN_ENCRYPTION_KEY` is not exactly 64 hex chars | Regenerate: `openssl rand -hex 32` — count must be 64 |
| `ECONNREFUSED arm-postgres-dev:5432` | Infra not running | `make infra-up && make infra-wait` |

### Frontend shows blank page or Module Federation error

The calendar MFE (`localhost:3001`) must be running before the core frontend (`localhost:3000`) loads it. Check:

```bash
make local-dev-ps               # confirm arm-calendar-fe-local is Up and healthy
make local-dev-logs-fe          # look for Vite errors in either frontend
```

If calendar-fe is still starting up (health check pending), wait another 30 seconds and refresh.

### Core frontend is unreachable after stack starts

The core frontend only starts after `arm-session-local` and `arm-calendar-fe-local` are both healthy. This can take up to 90 seconds on first start. Watch the startup:

```bash
make local-dev-ps     # repeat every 15s until arm-core-fe-local shows "Up"
```

### Hot reload not firing on macOS

All services set `CHOKIDAR_USEPOLLING=true` to work around the Docker Desktop filesystem notification limitation on macOS. If changes still do not apply:

```bash
make local-dev-restart    # stop + restart all containers (no rebuild)
```

### Port already in use

```bash
lsof -i :3000    # or :3001, :4000, :4001, :5001
```

Stop the conflicting process, or change that service's host-side port. **The port is not in `docker-compose.local.dev.yml`** — that file is just an `include:` list. Each service's host port comes from `HOST_PORT` in its own repo's `descriptor:` Makefile target, and the compose files under `generated/` are rewritten from it on every `make local-dev`. Editing a generated file works until the next run and no longer; edit the descriptor instead. See [DEPLOY-ORCHESTRATION.md](../DEPLOY-ORCHESTRATION.md).

### Infra containers not found by app services

App containers connect to infra via the external `arm-infra-network`. If that network was deleted:

```bash
make infra-down
make infra-up
make infra-wait
make local-dev-restart
```

### Kafka topics missing

If notification emails are not being sent and you see `UnknownTopicOrPartitionException` in logs:

```bash
make infra-wait    # re-runs the topic creation step safely (idempotent)
```

### Reset everything and start fresh

```bash
make local-dev-down            # stop app containers
make infra-down                # stop infra
make clean-volumes FORCE=true  # ⚠ DELETES ALL DATABASE DATA
make infra-up
make infra-wait
make local-dev-build
make local-dev
```

---

## Complete `make` reference

### First-time setup

```bash
make setup              # env-check + clone-repos + install + infra-up + infra-wait
make env-check          # verify all required .env variables are set
```

### Repositories

```bash
make clone-repos        # clone or pull all repos
make update-repos       # pull latest for all repos
make repos-status       # show branch and sync status per repo
```

### Infrastructure

```bash
make infra-up           # start Postgres, MongoDB, Redis, Kafka
make infra-down         # stop infra (data preserved)
make infra-restart      # stop + start infra
make infra-wait         # block until all infra is healthy + topics created
make infra-status       # show container status
make infra-logs         # tail all infra logs
make infra-logs-postgres
make infra-logs-mongodb
make infra-logs-redis
make infra-logs-kafka
```

### Local dev (Docker hot reload)

```bash
make local-dev          # start full hot-reload stack
make local-dev-down     # stop stack (infra untouched)
make local-dev-restart  # stop + start (no rebuild)
make local-dev-build    # rebuild all images (run after package.json changes)
make local-dev-logs     # tail all service logs
make local-dev-logs-fe  # frontend logs only
make local-dev-logs-be  # backend logs only
make local-dev-ps       # show container status
```

### Native pnpm dev (single service)

```bash
make dev-core-be        # core-backend   :4000
make dev-session            # arm-session        :5000
make dev-core-fe        # core-frontend  :3000
make dev-calendar-be    # calendar-be    :4001
make dev-calendar-fe    # calendar-fe    :3001
make dev-notification   # notification (Kafka consumer)
```

### Install

```bash
make install            # pnpm install in all services
make install-core-be
make install-core-fe
make install-session
make install-calendar-be
make install-calendar-fe
make install-notification
```

### Testing

```bash
make test               # all unit tests (no infra needed)
make test-all           # infra-up + infra-wait + all tests
make smoke-test         # curl /health on all running services
make test-core
make test-session
make test-calendar
make test-notification
make test-core-cov      # with coverage report
make test-session-cov
make test-calendar-cov
make test-notification-cov
```

### Health and status

```bash
make health             # curl all service /health endpoints
make ps                 # show all running containers (infra + apps)
```

### Cleanup

```bash
make clean-apps                  # remove app containers (infra safe)
make clean                       # remove all containers + networks
make clean-volumes FORCE=true    # ⚠ delete all DB data
make clean-images                # remove built ARM Docker images
```
