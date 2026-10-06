# BFF Rename — Reference Inventory

> Working reference for renaming the `bff` concept across the ARM workspace.
> **Nothing has been changed yet.** This document is the map, not the work.
>
> Survey date: 2026-08-25. Regenerate with the commands in [Appendix A](#appendix-a--regenerating-this-inventory).
>
> **Execution plan:** [BFF-RENAME-TASKS.md](BFF-RENAME-TASKS.md) — 101 tasks across 9 phases.

---

## 1. Decisions (finalised 2026-08-25)

| # | Decision | Choice |
|---|---|---|
| 1 | Name | **`arm-session`** — service key `session`, env prefix `SESSION_`, routes `/session/*` |
| 2 | Scope | **All 7 layers**, including layer 5 (runtime state) |
| 3 | `arm-tool-cli` | **In scope, same pass** |
| 4 | Layer 5 | **Renamed, no dual-read migration** — forced logout accepted |
| 5 | `src/bff/` | **Merged into `src/session/`** as one NestJS feature module |
| 6 | Inter-service secret | **`SESSION_INTERNAL_SECRET`**, and existing `SESSION_SECRET` → **`SESSION_COOKIE_SECRET`** |

### 1.1 Consequence of decisions 2 + 4 — this is a flag day

Renaming the Redis prefixes without a dual-read window signs out **every logged-in
user** at deploy and drops **in-flight OAuth handoffs**. Since that downtime is being
accepted anyway, the compatibility shims originally proposed for layers 3–4
(dual-prefix routing, dual header acceptance) buy very little: they exist to avoid
exactly the disruption that decision 4 already accepts.

**Therefore the plan below is a single coordinated cutover in a maintenance window,
not a staged rollout.** All repos ship together. See [§5](#5-execution-order).

Required before the window:
- User-facing notice that re-login will be required
- A rollback plan — the previous images plus the old `.env` (secrets are unchanged in
  value, only in name, so rollback is a rename-back, not a re-issue)
- Confirmed maintenance slot

> If you would rather avoid the forced logout, the only change needed is to revisit
> decision 4 — everything else in this document stands.

### 1.2 Names ruled out

- `arm-gateway` — "gateway" already denotes Kong, the API edge behind Caddy
  (`Caddyfile.prod:35`). A second "gateway" is worse than the current jargon.
- Any `/api/*` route prefix — owned by Caddy → Kong. `/bff/*` exists specifically to
  *bypass* Kong (`(secret removed):68`).

---

## 1A. The mapping

Authoritative old → new table. Everything below in this document resolves to this.

### Repo, directory, infrastructure

| Old | New |
|---|---|
| GitHub repo `arm-core-bff` | `arm-session` |
| Directory `core/arm-bff` | `core/arm-session` |
| `package.json` name `arm-bff` | `arm-session` |
| Container `arm-bff` (server) | `arm-session` |
| Container `arm-bff-local` | `arm-session-local` |
| Image `arm-bff` | `arm-session` |
| Orchestrator service key `bff` | `session` |
| `PATH_BFF` | `PATH_SESSION` |
| `docker-compose.bff.yml` (generated) | `docker-compose.session.yml` |
| Descriptor `PROD_NAME`/`PROD_CONTAINER_NAME`/`PROD_IMAGE` | `arm-session` |

> Removes the hardcoded exception documented at `generate-compose.sh:21-22` — the
> container name now derives from the service key like every other service.

### Make targets

| Old | New |
|---|---|
| `bff` | `session` |
| `dev-bff` | `dev-session` |
| `install-bff` | `install-session` |
| `logs-bff` | `logs-session` |
| `test-bff` | `test-session` |
| `test-bff-cov` | `test-session-cov` |

### Routes

| Old | New |
|---|---|
| `@Controller('bff')` | `@Controller('session')` |
| `/bff/health` | `/session/health` |
| `/bff/login/email/request-otp` | `/session/login/email/request-otp` |
| `/bff/login/email/verify` | `/session/login/email/verify` |
| `/bff/logout` | `/session/logout` |
| `/bff/me` | `/session/me` |
| `/bff/calendar/add-account[/microsoft]` | `/session/calendar/add-account[/microsoft]` |
| `/bff/calendar/oauth/callback[/microsoft]` | `/session/calendar/oauth/callback[/microsoft]` |
| `/bff/oauth/callback` | `/session/oauth/callback` |
| `/bff/proxy/*path` | `/session/proxy/*path` |
| `/bff/internal/create-session` | `/session/internal/create-session` |
| `/bff/session/activate` | `/session/activate` ⚠ **not** `/session/session/activate` |

### Env vars and header

| Old | New |
|---|---|
| `BFF_INTERNAL_SECRET` | `SESSION_INTERNAL_SECRET` |
| `SESSION_SECRET` | `SESSION_COOKIE_SECRET` ⚠ *pre-existing var, renamed for clarity* |
| `BFF_PUBLIC_URL` | `SESSION_PUBLIC_URL` |
| `BFF_URL` | `SESSION_URL` |
| `BFF_BASE_URL` | `SESSION_BASE_URL` |
| `VITE_BFF_URL` | `VITE_SESSION_URL` |
| `PROD_BFF_INTERNAL_SECRET` | `PROD_SESSION_INTERNAL_SECRET` |
| `PROD_BFF_PUBLIC_URL` | `PROD_SESSION_PUBLIC_URL` |
| `BFF_ROUTES` (FE constant) | `SESSION_ROUTES` |
| Header `X-BFF-Secret` | `X-Session-Secret` |

**Unchanged:** `SESSION_ID` (cookie), `SESSION_TTL_SECONDS`, `SESSION_MAX_AGE_MS`.

### Code identifiers

| Old | New |
|---|---|
| `BffController` | `SessionController` |
| `BffGuard` | `SessionGuard` |
| `BffModule` | *merged into existing* `SessionModule` |
| `BffSession` | `Session` |
| `BffAugmentedRequest` | `SessionAugmentedRequest` |
| `BffSecretGuard` (core-be) | `SessionSecretGuard` |
| `bffSession` / `bffSessionId` | `session` / `sessionId` ⚠ *check for shadowing* |
| `bffReq` / `bffRes` | `sessionReq` / `sessionRes` |
| `bffUrl` / `bffPublicUrl` | `sessionUrl` / `sessionPublicUrl` |

### Files

| Old | New |
|---|---|
| `src/bff/bff.controller.ts` | `src/session/session.controller.ts` |
| `src/bff/bff.guard.ts` | `src/session/session.guard.ts` |
| `src/bff/bff.module.ts` | *merge into* `src/session/session.module.ts` |
| `src/bff/bff-request.interface.ts` | `src/session/session-request.interface.ts` |
| `src/bff/bff.cors.spec.ts` | `src/session/session.cors.spec.ts` |
| `src/bff/bff.session-security.spec.ts` | `src/session/session-security.spec.ts` |
| `src/bff/bff.controller*.spec.ts` | `src/session/session.controller*.spec.ts` |
| `src/bff/bff.guard.spec.ts` | `src/session/session.guard.spec.ts` |
| `src/bff/proxy/` | `src/session/proxy/` |
| `src/bff/dto/` | `src/session/dto/` |
| `core-be src/auth/guards/bff-secret.guard.ts` | `session-secret.guard.ts` |
| `core-fe src/lib/constants/api-routes/bff-routes.ts` | `session-routes.ts` |
| `docs BFF-SESSION-FLOW.md` | `SESSION-FLOW.md` |
| `docs architecture/bff-architecture.md` | `architecture/session-architecture.md` |

### Runtime state (layer 5 — no migration)

| Old | New |
|---|---|
| `bff:session:` | `session:data:` ⚠ **not** `session:session:` |
| `bff:handoff:` | `session:handoff:` |
| `bff:refresh-lock:` | `session:refresh-lock:` |
| `bff_session_miss_total` | `session_miss_total` |
| `bff_proxy_duration_seconds` | `session_proxy_duration_seconds` |
| pino `service: 'bff'` | `service: 'session'` |
| Prometheus `job_name: bff` | `job_name: session` |
| Grafana `job=~"...\|bff\|..."` | `job=~"...\|session\|..."` |

`REDIS-ARCHITECTURE.md` documents these key namespaces and must be updated to match.

---

## 2. Scale

**1,815 matching lines across 253 files**, after removing false positives.

| Area | Lines | Notes |
|---|---:|---|
| `docs/arm-docs` | 786 | Prose + diagrams. 55 markdown files |
| `core/arm-bff` | 463 | The service itself. Self-contained |
| `deploy/arm-deploy-make` | 132 | Make, compose, Caddy, Kong, observability, CI |
| `core/arm-core-be` | 102 | `BffSecretGuard`, auth controller, env validation |
| `core/arm-core-fe` | 96 | `BFF_ROUTES`, auth client, config |
| `apps/arm-app-calendar` | 94 | FE call sites + BE internal-secret guard |
| `arm-tool-cli` | 52 | **Code generator — emits BFF wiring into new services** |
| `apps/arm-admin` | 50 | FE constants + BE env validation |
| `CLAUDE.md` (root) | 27 | Architecture rules |
| `.claude/settings.json` | 7 | Allowlisted curl commands against `/bff/*` |
| `.env` (workspace) | 3 | Live local secrets |
| `services/arm-service-notification` | 2 | Comments only |
| `docker-compose.yml` (root) | 1 | Comment only |

`arm-service-notification` and `arm-ui-library` are effectively untouched.

---

## 3. The seven layers

Ordered cheapest and safest first. Layers 1–2 are independently shippable.
Layers 3–5 require a coordinated rollout across repos.

### Layer 1 — Internal code identifiers (`core/arm-bff` only)

Contained entirely within one repo. No cross-repo contract. Purely mechanical.

| Identifier | Count |
|---|---:|
| `BffGuard` | 86 |
| `BffController` | 51 |
| `BffSession` | 50 |
| `BffAugmentedRequest` | 21 |
| `BffModule` | 3 |
| `BffSessionService` | 2 |
| `bffSession` / `bffSessionId` | 39 |
| `bffReq` / `bffRes` | 29 |
| `bffUrl` / `bffPublicUrl` | 10 |

**Files to rename (`git mv`):**

```
core/arm-bff/src/bff/                        → merge into src/session/
core/arm-bff/src/bff/bff.controller.ts
core/arm-bff/src/bff/bff.controller.spec.ts
core/arm-bff/src/bff/bff.controller.login.spec.ts
core/arm-bff/src/bff/bff.controller.throttle.spec.ts
core/arm-bff/src/bff/bff.cors.spec.ts
core/arm-bff/src/bff/bff.guard.ts
core/arm-bff/src/bff/bff.guard.spec.ts
core/arm-bff/src/bff/bff.module.ts
core/arm-bff/src/bff/bff.session-security.spec.ts
core/arm-bff/src/bff/bff-request.interface.ts
```

Note `BffSecretGuard` (34) is **not** here — it lives in `core-be`. See Layer 4.

**Risk:** low. Compiler-verified. Tests catch mistakes.

---

### Layer 2 — Service, repo, and infrastructure names

| Token | Where |
|---|---|
| GitHub repo `arm-core-bff` | `deploy/arm-deploy-make/repos.conf:38`, `docs/arm-docs/docs/CICD.md:38`, git remote |
| Directory `core/arm-bff` | `repos.conf`, `services.conf`, `make/config.mk:163` (`PATH_BFF`) |
| `package.json` name `arm-bff` | `core/arm-bff/package.json` |
| Container `arm-bff` (server) | `Caddyfile.prod:69`, `docker-compose.yml`, `deploy-production.yml:310` |
| Container `arm-bff-local` | `make/local-dev.mk:42,84,149,210` |
| Descriptor `PROD_NAME` / `PROD_CONTAINER_NAME` / `PROD_IMAGE` = `arm-bff` | `core/arm-bff/Makefile:42-44` |
| Orchestrator service key `bff` | `services.conf:20`, `orchestrator/lib/generate-compose.sh:21-22` |
| Generated compose `docker-compose.bff.yml` | `docker-compose.local.dev.yml:43`, `docker-compose.yml:28` |
| Prometheus `job_name: bff` | `observability/prometheus.prod.yml:11` |

**Make targets to rename:**

```
make/apps.mk:55      bff:
make/dev.mk:28       dev-bff:
make/install.mk:46   install-bff:
make/utils.mk:61     logs-bff:
make/test.mk:74      test-bff:
make/test.mk:82      test-bff-cov:
make/config.mk:163   PATH_BFF
```

Also `make/utils.mk:11`, `make/dev.mk:17`, `make/install.mk:16`, `make/test.mk:11-12`,
`make/apps.mk:14` (`.PHONY` lists) and the `make help` text at `Makefile:59,132,142,149,163`
and `make/local-dev.mk:112,145,155`.

> **`generate-compose.sh:21-22` already documents `arm-bff` as a "real, pre-existing
> irregularity"** — every other service's container name is derived from its service key,
> but the BFF's was hardcoded. A rename is the moment to remove that special case.

**Risk:** medium. Container-name changes must land in Caddy, compose, and the
deploy workflow's `CONTAINERS` list simultaneously, or health checks fail.

> **Prometheus `job_name` change breaks dashboard/alert continuity.** `job=~"...|bff|..."`
> appears in `observability/grafana/provisioning/alerting/rules.yaml:18` and
> `observability/grafana/dashboards/service-health.json:18`. Historical series keep the
> old label value. Either keep `job_name: bff` permanently, or accept a discontinuity
> and update both files.

---

### Layer 3 — Route prefix `/bff/*`

**668 occurrences.** The single largest category.

| Repo | Occurrences |
|---:|---:|
| `core/arm-bff` | 319 |
| `docs/arm-docs` | 155 |
| `core/arm-core-fe` | 35 |
| `apps/arm-app-calendar` | 33 |
| `arm-tool-cli` | 11 |
| `apps/arm-admin` | 11 |
| `deploy/arm-deploy-make` | 10 |
| `core/arm-core-be` | 10 |
| `.claude/settings.json` | 7 |
| `CLAUDE.md` | 4 |
| `.env` | 2 |

**Source of truth:** `core/arm-bff/src/bff/bff.controller.ts:43` — `@Controller('bff')`.

Every route under it:

```
GET   /bff/health
POST  /bff/login/email/request-otp
POST  /bff/login/email/verify
POST  /bff/logout
GET   /bff/me
GET   /bff/calendar/add-account
GET   /bff/calendar/add-account/microsoft
GET   /bff/calendar/oauth/callback
GET   /bff/calendar/oauth/callback/microsoft
GET   /bff/oauth/callback
ALL   /bff/proxy/*path
POST  /bff/internal/create-session
POST  /bff/session/activate
```

**Frontend definition points (change these, not the call sites):**

- `core/arm-core-fe/src/lib/constants/api-routes/bff-routes.ts` — the `BFF_ROUTES` constant (12 hits)
- `core/arm-core-fe/src/api/clients/auth-client.ts` (10)
- `apps/arm-admin/src/frontend/src/lib/constants.ts`
- `apps/arm-app-calendar/src/frontend/src/features/calendar/services/api.ts`

**Edge and infra:**

- `deploy/arm-deploy-make/Caddyfile.prod:68` — `handle /bff/*`
- `deploy/arm-deploy-make/docker-compose.edge.local.yml:29`
- `core/arm-bff/Makefile:37` — descriptor `HEALTH_PATH=/bff/health`
- `deploy/arm-deploy-make/make/utils.mk:24-25` and `make/test.mk:38` — health-check URLs

**Risk:** high. Browser and edge must agree. Per decision 2+4 there is **no
compatibility window** — the controller switches prefix and every consumer switches
with it, in the same maintenance window. Watch `/bff/session/activate`, which becomes
`/session/activate`, not `/session/session/activate`.

---

### Layer 4 — Env vars and the internal header

| Token | Count | Kind |
|---|---:|---|
| `BFF_INTERNAL_SECRET` | 105 | Shared secret, **4 services must match** |
| `VITE_BFF_URL` | 54 | Frontend build-time var |
| `BFF_PUBLIC_URL` | 31 | BFF's own public origin |
| `BFF_ROUTES` | 19 | FE constant (not env) |
| `BFF_BASE_URL` | 16 | FE runtime config |
| `BFF_URL` | 20 | core-be + calendar-fe |
| `X-BFF-Secret` / `x-bff-secret` | 91 | HTTP header |
| `PROD_BFF_INTERNAL_SECRET`, `PROD_BFF_PUBLIC_URL` | 3 | Descriptor keys |
| `PATH_BFF` | 5 | Make variable |

**`X-BFF-Secret` verifiers — all must change together:**

| Service | File |
|---|---|
| `arm-bff` (sender) | `src/bff/proxy/proxy.service.ts`, `src/bff/bff.controller.ts` |
| `arm-core-be` | `src/auth/guards/bff-secret.guard.ts` ← **file also needs renaming** |
| `arm-app-calendar` (be) | `src/common/guards/internal-secret.guard.ts`, `src/common/core-auth/core-auth.service.ts` |
| `arm-admin` (be) | `src/config/env.validation.ts` (validates the var; guard is `kong-jwt.guard.ts`) |

**Env files carrying `BFF_*` (9):**

```
.env                                            ← live local secrets
core/arm-bff/.env.example
core/arm-core-be/.env.example
core/arm-core-fe/.env.example
core/arm-core-fe/src/environments/.env.example
apps/arm-admin/src/backend/.env.example
apps/arm-admin/src/frontend/.env.example
apps/arm-app-calendar/src/frontend/.env.example
deploy/arm-deploy-make/.env.example
```

Plus 4 test fixtures: `deploy/arm-deploy-make/tests/fixtures/env/{valid,placeholder,missing-core,bad-hex-key}.env`

**Validation schemas that will reject the old/new name:**

```
core/arm-bff/src/config/env.validation.ts
core/arm-core-be/src/config/env.validation.ts
apps/arm-admin/src/backend/src/config/env.validation.ts
apps/arm-app-calendar/src/backend/src/common/config/env.ts
deploy/arm-deploy-make/make/env.mk:206        ← env-init generates BFF_INTERNAL_SECRET;
                                                 env-check diffs .env against .env.example
```

**Risk: highest.** A partial rollout means the secret header stops matching and
**every proxied request 401s**. `make env-check` will also fail loudly mid-migration.
Per decision 2+4 there is **no fallback read and no dual header acceptance** — all
four verifiers and all seven env-carrying repos cut over together in the window.
Note `SESSION_SECRET` → `SESSION_COOKIE_SECRET` is an *additional* rename beyond the
`BFF_*` set (decision 6), and its value is preserved.

---

### Layer 5 — Runtime state (⚠ read before touching)

These are **not** code references. Changing them mutates live data contracts.

| Item | Location | Consequence of renaming |
|---|---|---|
| `bff:session:` | `core/arm-bff/src/session/session.service.ts:12` | **Every logged-in user is signed out.** Existing Redis keys orphaned |
| `bff:handoff:` | `session.service.ts:13` | In-flight OAuth handoffs break during deploy |
| `bff:refresh-lock:` | `session.service.ts:15` | Brief double-refresh window across the cutover |
| `bff_session_miss_total` | `src/bff/bff.guard.ts:25`, `bff.module.ts:24` | Metric history discontinuity |
| `bff_proxy_duration_seconds` | `src/bff/proxy/proxy.service.ts:107`, `bff.module.ts:28` | Same; also breaks `service-health.json:67` |
| `service: 'bff'` (pino) | `src/app.module.ts:38` | Loki log queries filtering on this label break |

**Already safe — do not change:**

- Session cookie is `SESSION_ID` (`src/bff/bff.guard.ts:33`) — already neutral.

**Decision 4: rename all of it, no dual-read migration.** This is the single highest
blast-radius item in the document and the reason the whole rename is a flag day
([§1.1](#11-consequence-of-decisions-2--4--this-is-a-flag-day)).

At cutover: every user is signed out, in-flight OAuth handoffs are dropped, and
Grafana panels lose series continuity. Old `bff:*` keys are orphaned rather than
deleted and expire on their own TTL; optionally `SCAN`/`DEL` them afterwards.

Watch the prefix mapping: `bff:session:` → `session:data:`, **not** `session:session:`.

---

### Layer 6 — Documentation (786 lines, 55 files)

**Files to rename:**

```
docs/arm-docs/docs/BFF-SESSION-FLOW.md
docs/arm-docs/docs/architecture/bff-architecture.md
docs/arm-docs/docs/adr/001-standalone-bff.md      ← historical ADR; see note
```

`_sidebar.md` and `meta/documentation-inventory.md` link to these and must be updated.

**Heaviest files:**

| File | Hits |
|---|---:|
| `meta/miro-diagrams.md` | 98 |
| `architecture/bff-architecture.md` | 91 |
| `meta/documentation-inventory.md` | 67 |
| `BFF-SESSION-FLOW.md` | 53 |
| `calendar/GOOGLE-OAUTH.md` | 49 |
| `ARCHITECTURE.md` | 36 |
| `ENV-VARS.md` | 33 |
| `AUTH-ARCHITECTURE.md` | 24 |
| `SECURITY.md` | 22 |

Remaining 46 files have ≤21 hits each.

Also: root `CLAUDE.md` (27) and the per-repo `CLAUDE.md` files in `arm-bff`,
`arm-core-be`, `arm-core-fe`, `arm-admin/src/{backend,frontend}`, `arm-app-calendar`,
`arm-deploy-make`.

> **ADR note:** `adr/001-standalone-bff.md` records a decision made at a point in
> time. Convention is not to rewrite ADR history — add a superseding ADR recording
> the rename instead, and leave 001 as written.

---

### Layer 7 — `arm-tool-cli` (the code generator) ⚠ easy to miss

`arm-tool-cli` scaffolds new ARM services and **emits BFF wiring into the files it
generates**. If it is not updated, every future service is born with stale references.

| File | What it emits |
|---|---|
| `src/core/registration/emit-registration.ts:119` | Literal path `core/arm-bff/src/bff/proxy/proxy.service.ts` |
| `src/core/registration/emit-registration.ts:125` | Instruction that `BFF_INTERNAL_SECRET` must match |
| `src/core/registration/build-registration.ts:147` | Same hardcoded path |
| `src/core/registration/ports.ts:14` | `[5001, "bff"]` port map |
| `src/core/conformance/readme-files.ts:57,58,123,180,181,223,229,230` | Generated README text with `/bff/proxy/<name>/v1` |
| `src/core/conformance/frontend-arm-files.ts` | `BFF_BASE_URL` in generated FE config (10 hits) |
| `src/core/conformance/backend-arm-files.ts` | Generated BE guard comments (8 hits) |
| `src/core/descriptor/build-descriptor.ts:111` | `requiredEnvVars: [..., "BFF_INTERNAL_SECRET"]` |
| `src/utils/__tests__/conformance.test.ts` | Assertions on the above |

`arm-tool-cli` is **not** listed in `repos.conf` — confirm whether it is in scope
and how it is versioned before starting.

---

## 4. False positives — do not touch

These matched `bff` case-insensitively but are unrelated:

| Pattern | Example | Where |
|---|---|---|
| Hex colours | `#F7FBFF`, `#E14BFF` | `core/arm-core-fe/src/assets/landing-page/desktop.svg` |
| Base64 integrity hashes | `LJsabFFvV`, `5orIbff80` | every `pnpm-lock.yaml` |
| Generated secrets | random base64 in `.env` | `.env` |
| Unrelated skill docs | — | `docs/superpowers/`, `ref/`, `aracreate-template-codebase/` |
| Browser console logs | — | `.playwright-mcp/*.log` (throwaway) |

Excluding these dropped the count from 2,300 lines / 341 files to 1,815 / 253.

---

## 5. Execution order

Per [§1.1](#11-consequence-of-decisions-2--4--this-is-a-flag-day) this is a **single
coordinated cutover in a maintenance window**, not a staged rollout. Steps 1–7 are all
prepared on branches first, then deployed together in step 8.

| # | Step | Layers | Where |
|---|---|---|---|
| 0 | Book the window; send the re-login notice; confirm rollback images | — | ops |
| 1 | Merge `src/bff/` into `src/session/`; rename identifiers and files | 1 | `arm-session` |
| 2 | Change `@Controller('bff')` → `@Controller('session')`; fix `/session/activate` | 3 | `arm-session` |
| 3 | Rename Redis prefixes, metrics, pino label | 5 | `arm-session` |
| 4 | Rename env vars + `X-Session-Secret` in all 4 backends and 3 frontends | 4 | all app repos |
| 5 | Rename repo, dir, package, containers, images, make targets, descriptor, Caddy, Prometheus, Grafana, CI | 2, 5 | `arm-deploy-make` + GitHub |
| 6 | Update `arm-tool-cli` generators | 7 | `arm-tool-cli` |
| 7 | Rewrite docs; rename the 3 doc files; add superseding ADR | 6 | `arm-docs` |
| 8 | **Cutover:** `make env-init` on the server for renamed vars, deploy all images together | — | server |
| 9 | Post-cutover verification ([§6](#6-verification-checklist)) | — | server |

**Ordering constraints that matter:**

- Step 4 must cover **all four** `X-Session-Secret` verifiers in the same deploy. Any
  service left on `X-BFF-Secret` rejects every proxied request.
- Step 5's container rename must land in `Caddyfile.prod`, the compose files, **and**
  `deploy-production.yml:310`'s `CONTAINERS` list simultaneously.
- `SESSION_SECRET` → `SESSION_COOKIE_SECRET` keeps its **value**. Only the key name
  changes, so rollback is a rename-back, not a secret re-issue.
- Redis keys under the old prefixes are orphaned, not deleted. They expire on their own
  TTL (`SESSION_TTL_SECONDS`, default 30 days). Optionally `SCAN`/`DEL` `bff:*` after
  the window to reclaim memory.

---

## 6. Verification checklist

- [ ] `make env-check` passes
- [ ] `make test` passes (`test-session`, `test-core`, `test-calendar`, `test-notification`)
- [ ] `make local-dev-ps` shows all 8 services healthy
- [ ] Browser login at `localhost:3000` completes and survives a refresh
- [ ] Google **and** Microsoft add-account popups complete
- [ ] `make edge-up` — Caddy routes the new prefix
- [ ] Grafana `service-health` dashboard still renders both panels
- [ ] Every user is re-login-able and a fresh session survives a refresh
- [ ] `grep -ri bff` returns **only** false positives from [§4](#4-false-positives--do-not-touch) —
      decision 2 means there are no intentional survivors

---

## 7. Collision watch-list

Resolved by decisions 5 and 6, but these are the specific traps during the edit:

| Trap | Why | Correct result |
|---|---|---|
| `src/session/` already exists | `src/bff/` merges in rather than renames onto it | One `SessionModule` absorbing `bff.module.ts`'s providers |
| `SESSION_SECRET` vs `SESSION_INTERNAL_SECRET` | Adjacent, indistinguishable names | `SESSION_COOKIE_SECRET` (signs the cookie) vs `SESSION_INTERNAL_SECRET` (service-to-service) |
| `/bff/session/activate` | Naive prefix swap yields `/session/session/activate` | `/session/activate` |
| `bff:session:` | Naive swap yields `session:session:` | `session:data:` |
| `BffSession` → `SessionSession` | Same class of error on types | `Session` — drop the prefix, don't translate it |
| `bffSession` local variables | Renaming to `session` may shadow an outer `session` | Check each site; `sessionData` where shadowed |
| `SESSION_KEY_PREFIX` constant | Already exists in `session.service.ts:12` | Unchanged — only its *value* changes |
| `SESSION_ID` cookie | Already neutral | **Unchanged.** Do not touch |

## Appendix A — regenerating this inventory

```sh
cd /Users/kishor/Work/01-araCreate/arm

EX="--exclude-dir=node_modules --exclude-dir=.git --exclude-dir=dist \
    --exclude-dir=build --exclude-dir=generated --exclude-dir=coverage \
    --exclude=pnpm-lock.yaml --exclude=CHANGELOG.md"

# All matches, minus known-irrelevant trees
grep -rn $EX -iE "bff" . 2>/dev/null \
  | grep -vE "^(\.playwright-mcp|docs/superpowers|ref/|aracreate-template-codebase)" \
  | grep -vE "^[^:]*\.(svg|png|jpg)" \
  | grep -vE "^[^:]*pnpm-lock\.yaml" \
  > /tmp/bff-raw.txt

wc -l /tmp/bff-raw.txt                       # total lines
cut -d: -f1 /tmp/bff-raw.txt | sort -u | wc -l   # total files

# Env-var / header token census
grep -ohE "[A-Z0-9_]*BFF[A-Z0-9_]*" /tmp/bff-raw.txt | sort | uniq -c | sort -rn

# Code identifier census
grep -ohE "\bBff[A-Za-z0-9_]*" /tmp/bff-raw.txt | sort | uniq -c | sort -rn

# Route prefix distribution by repo
grep "/bff/" /tmp/bff-raw.txt | cut -d: -f1 \
  | sed -E 's|^(docs/arm-docs)/.*|\1|; s|^([a-z]+/[^/]+)/.*|\1|' \
  | sort | uniq -c | sort -rn

# Paths whose names contain bff
find . -iname "*bff*" -not -path "*/node_modules/*" -not -path "*/.git/*" \
  -not -path "*/dist/*" -not -path "*/.playwright-mcp/*" | sort
```
