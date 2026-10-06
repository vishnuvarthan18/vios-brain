# 11 — arm-cli: The Platform CLI

> Phases 0-7 for arm-cli — from scaffolder to platform tool. No Node.js on servers. Compose generation stays in arm-deploy-make.

---

## Responsibility Split

```
Developer laptop                           Servers (S1-S4)
────────────────                           ────────────────
arm-cli scaffold ───── descriptor ─────→   orchestrator generate-compose
arm-cli validate ───── descriptor ─────→   (lightweight bash re-check)
arm-cli check ──────── SSH ────────────→   docker ps / redis-cli / pg_isready
                                           make ci-deploy-prod
                                           docker compose up -d
```

| arm-cli (TypeScript) | arm-deploy-make (bash) |
|---|---|
| Scaffold new repos | Generate compose from descriptors |
| Validate repos & descriptors | Run compose (up/down/build) |
| Check server health (SSH, read-only) | Deploy, rollback |
| Port allocation | Infra management (backup, restore) |
| Template conformance | Kong/Caddy rendering |

---

## Phase 0: Make CLI Runnable

**Goal:** `arm-cli --help` works after `pnpm build && pnpm link --global`.

Fix build scripts (tsup, tsc-alias, copy-templates).

**Verify:** `arm-cli --version` prints `1.0.0`.

---

## Phase 1: Delete Generic Infrastructure Generators

**Goal:** Remove ~3,300 lines of dead code — CI/CD generators, Docker/Kafka/Redis infra generators. ARM's infra is arm-deploy-make's job.

**Verify:** Scaffolded repos have no CI/CD files, no docker-compose, no infra config.

---

## Phase 2: Fix Descriptors for Multi-Server

**Goal:** Generated descriptors use `${VAR:-default}` for infra hostnames.

### The Bug

`arm-tool-cli/src/core/descriptor/build-descriptor.ts:65`:

```typescript
// Broken — ignores .env override:
{ key: "DB_HOST", value: "arm-postgres-prod" }

// Fixed — .env can set DB_HOST=10.0.1.1:
{ key: "DB_HOST", value: "${DB_HOST:-arm-postgres-prod}" }
```

Also: `DB_PORT` → `${DB_PORT:-5432}` (PgBouncer uses 6432).

`localDbEnv` stays unchanged (Docker DNS, no override needed).

MongoDB `${MONGODB_URI}` is already correct.

### Conformance Rule

> Every `EXTRA_ENV_PROD_*` referencing a shared infra hostname MUST use `${VAR:-default}`.

| Env var | Required pattern |
|---|---|
| `DB_HOST` | `${DB_HOST:-arm-postgres-prod}` |
| `DB_PORT` | `${DB_PORT:-5432}` |
| `REDIS_HOST` | `${REDIS_HOST:-arm-redis-prod}` |
| `KAFKA_BROKERS` | `${KAFKA_BROKERS:-arm-kafka1-prod:9092,...}` |
| `MONGODB_URI` | `${MONGODB_URI}` |

---

## Phase 3: ARM-Conformant Templates

**Goal:** Generated React/NestJS code matches arm-admin and arm-app-calendar structure and conventions.

---

## Phase 4: Emit Registration Patch

**Goal:** arm-cli emits `workspace-registration.patch` + checklist (not auto-edit). Patch adds one row to `services.conf` + env vars to `.env.example`.

---

## Phase 5: `arm-cli validate`

**Goal:** Workspace-wide validation. Replaces scattered bash validation in arm-deploy-make with proper TypeScript checks.

```bash
arm-cli validate              # all repos
arm-cli validate core-be      # one repo
arm-cli validate --fix        # auto-fix trivial issues
```

### What It Checks

| Check | What |
|---|---|
| Descriptor fields | Run `make descriptor`, parse, validate against contract |
| Multi-server pattern | `EXTRA_ENV_PROD_*` uses `${VAR:-default}` for infra hosts |
| Port collisions | No two services claim the same port |
| Env var completeness | Every `REQUIRED_ENV_VARS` exists in `.env.example` |
| Structure | Dockerfile, Makefile (`descriptor` target), package.json exist |
| Template conformance | SPDX headers, versioning, naming per TEMPLATE-CONFORMANCE.md |

### Output

```
arm-cli validate

  core-be         ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env
  arm-session         ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env
  calendar-be     ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env
  admin-be        ✓ descriptor  ✓ ports  ✓ structure  ✓ conformance  ✓ env
  notification    ✓ descriptor  ✓ ports  ✓ structure  ✗ conformance  ✓ env
                    └── missing SPDX header in 3 files

  8/8 repos checked, 1 warning, 0 errors
```

### What Stays in arm-deploy-make

`make env-check` stays for CI (lightweight, no Node dependency). `generate-compose.sh` keeps its basic safety-net check. Thorough validation = `arm-cli validate`.

---

## Phase 6: `arm-cli check`

**Goal:** Server health dashboard from developer laptop via SSH. Read-only. Replaces the manual monthly checklist.

```bash
arm-cli check                 # everything
arm-cli check health          # container status all servers
arm-cli check backups         # last backup time, S3 sync, drill status
arm-cli check firewall        # ufw rules diff against expected
arm-cli check certs           # TLS cert expiry
arm-cli check disk            # disk usage all servers
arm-cli check redis           # memory, eviction policy
arm-cli check postgres        # connections vs max_connections
arm-cli check logs            # recent errors from Loki
```

### How It Works

SSHes into each server, runs standard commands (`docker ps`, `redis-cli`, `pg_isready`, `df -h`, `ufw status`), parses output, prints dashboard.

### Server Config

```yaml
# arm-tool-cli/servers.yml (gitignored)
servers:
  db:      { host: 10.0.1.1, user: deploy }
  app:     { host: 10.0.1.2, user: deploy }
  observe: { host: 10.0.1.3, user: deploy }
  backup:  { host: 10.0.1.4, user: deploy }
```

### Output

```
arm-cli check

  S1 (DB)       10.0.1.1   ✓ postgres 87/250  ✓ mongodb ok  ✓ kafka ok    ✓ disk 34%
  S2 (App)      10.0.1.2   ✓ 8/8 services     ✓ redis 45%   ✓ cert 58d   ✓ disk 28%
  S3 (Observe)  10.0.1.3   ✓ prometheus       ✓ grafana     ✓ loki        ✓ disk 41%
  S4 (Backup)   10.0.1.4   ✓ last backup 6h   ✓ S3 sync 6h  ✓ drill 3d   ✓ disk 22%
```

### Does NOT Do

- Deploy (GitHub Actions owns this)
- Rollback (`make rollback` via CI)
- Mutate server state (all read-only)
- Replace Grafana alerts (24/7 automated; this is on-demand)

---

## Phase 7: End-to-End Validation

**Goal:** Prove the full lifecycle.

1. `arm-cli scaffold --kind service --db PostgreSQL --port 4300 -y`
2. `arm-cli validate` — passes
3. Apply registration patch → `make generate-compose-prod` — succeeds
4. Set `DB_HOST=10.0.1.1` → compose file shows override
5. `arm-cli check` — servers healthy
6. Register arm-cli in `repos.conf` at `tools/arm-tool-cli`
7. Document in ONBOARDING.md

---

## Command Map

```
arm-cli
├── scaffold                    # Phase 1-4
│   ├── --kind app|service
│   ├── --db PostgreSQL|MongoDB|None
│   ├── --port <N>
│   └── --frontend-port <N>
│
├── validate                    # Phase 5
│   ├── [service-name]
│   └── --fix
│
└── check                       # Phase 6
    ├── health
    ├── backups
    ├── firewall
    ├── certs
    ├── disk
    ├── redis
    ├── postgres
    └── logs
```
