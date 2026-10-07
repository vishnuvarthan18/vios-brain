# BFF → Session Rename — Phased Task Sheet

> Execution plan for renaming `bff` → `session` across the ARM workspace.
> Decisions and the authoritative old→new mapping live in
> [BFF-RENAME-INVENTORY.md](BFF-RENAME-INVENTORY.md) §1 and §1A. **Do not re-derive
> names here** — every task below resolves against that mapping.
>
> Created 2026-08-25. **Phases 0-6 executed** (P4-1 blocked on repo rename) — see [BFF-RENAME-P0-RECORD.md](BFF-RENAME-P0-RECORD.md).
> Phases 1-8 not started; no source code modified.

---

## Shape of the work

Per inventory §1.1 this is a **flag day**: decision 4 (rename Redis prefixes with no
dual-read) forces a logout at deploy, so there are no compatibility shims. Phases 1–6
are prepared on branches in parallel and **all deploy together in Phase 7**.

```
P0 Pre-flight ✅ ─┬─→ P1 arm-session internals ─→ P2 arm-session wire surface ──┐
                  │                                                            │
                  ├─→ P5 arm-tool-cli ─────────────────────────────────────────┤
                  │                                                            ├─→ P8 Cutover ─→ P9 Verify
                  ├─→ P6 Docs ─────────────────────────────────────────────────┤
                  │                                                            │
                  └─→ P3 Consumer repos ──→ P4 Deploy & infra ─────────────────┘
                                                                               │
   P7 Cutover readiness (human / server / decisions) ───────────────────────────┘
   └─ runs in parallel with P1-P6; MUST land before P8
```

**8 repos in scope.** Kong, Authelia and `arm-ui-library` have **zero** references and
are untouched.

| Repo | Phase(s) | Weight |
|---|---|---|
| `arm-core-bff` → `arm-session` | 1, 2, 4 | 463 lines |
| `arm-deploy-make` | 4 | 132 |
| `arm-core-be` | 3 | 102 |
| `arm-core-fe` | 3 | 96 |
| `arm-app-calendar` | 3 | 94 |
| `arm-tool-cli` | 5 | 52 |
| `arm-admin` | 3 | 50 |
| `arm-docs` | 6 | 786 |

---

## Phase 0 — Pre-flight ✅ complete

**Goal:** rollback anchors and branches, so code work can begin. No code.
**Record:** [BFF-RENAME-P0-RECORD.md](BFF-RENAME-P0-RECORD.md)

| ID | Task | Status |
|---|---|---|
| P0-1 | Capture rollback anchors (git SHA + branch per repo) | ✅ Done — 8 repos recorded |
| P0-2 | Create `refactor/rename-bff-to-session` in all 8 repos | ✅ Done — `git branch`, not checked out; nothing committed or pushed |
| P0-3 | Capture the local baseline (`make local-dev-ps`, `make test`) | ✅ Done — ⚠️ `make test` **fails today**: 10 pre-existing `calendar-be` failures, `notification` never runs |
| P0-4 | Draft the user notice | ✅ Drafted — scheduling parked to [P7-2](#phase-7--cutover-readiness) |

**Exit:** ✅ branches cut, rollback SHAs recorded, baseline captured and its problems known.

> The window booking, the two server-side captures and the baseline decisions were
> **parked into [Phase 7](#phase-7--cutover-readiness)**. None of them block Phases 1–6.

---

## Phase 1 — `arm-session` internals ✅ complete

**Goal:** merge `src/bff/` into `src/session/` and rename every identifier.
**Done:** 2026-08-25 on `refactor/rename-bff-to-session`. Uncommitted.
**Result:** build ok · **11 suites / 92 tests pass** (identical to baseline) · lint 0 errors.

| ID | Task | Status |
|---|---|---|
| P1-1 | Merge `bff.module.ts` into the existing `SessionModule` | ✅ One module: controller + guard + session/admin/proxy services + metric providers; `exports` unchanged |
| P1-2 | `BffController` → `SessionController` | ✅ → `src/session/session.controller.ts` |
| P1-3 | `BffGuard` → `SessionGuard` | ✅ → `src/session/session.guard.ts` |
| P1-4 | `BffAugmentedRequest` → `SessionAugmentedRequest` | ✅ → `src/session/session-request.interface.ts` |
| P1-5 | `BffSession` → `Session` | ✅ Prefix dropped, not translated |
| P1-6 | Move `proxy/` and `dto/` | ✅ → `src/session/{proxy,dto}/` |
| P1-7 | Move and rename all 6 spec files | ✅ |
| P1-8 | Rename locals `bffReq` → `sessionReq`, `.bffSession`/`.bffSessionId` → `.session`/`.sessionId` | ✅ No shadowing — the property sits on the request, the local `session` is separate |
| P1-9 | Delete `src/bff/` | ✅ Gone; `app.module.ts` imports `SessionModule` only |
| P1-10 | Fix the stale self-path comment in `session.controller.spec.ts:6` | ✅ *(added during execution)* |

**Exit:** ✅ `pnpm build`, `pnpm test`, `pnpm lint` all pass; **no `Bff*`/`bff*` identifiers
remain in `src/`**.

> ⚠️ The original exit criterion said "`grep -ri bff src/` returns nothing". **That was
> wrong** — route prefixes (`/bff/*`, `@Controller('bff')`), metric names, Redis key
> prefixes, `x-bff-secret`, `BFF_INTERNAL_SECRET`, the pino label and prose comments are
> all **Phase 2** scope and still present by design. The corrected criterion is
> identifiers only.

**Follow-ups noted, not actioned (out of P1 scope):**
- `core/arm-bff/CLAUDE.md` still documents the old `src/bff/` layout and `BffSession` — **P6-9**.
- `session.controller.throttle.spec.ts:55` carries a `(BFF-09)` task ref in a `describe`
  string. Pre-existing and against convention; flagged, not removed.

---

## Phase 2 — `arm-session` wire surface ✅ complete

**Goal:** everything the outside world observes. Still one repo.
**Done:** 2026-08-25 on `refactor/rename-bff-to-session`. Uncommitted.
**Result:** build ok · **11 suites / 92 tests pass** · lint 0 errors · route registration verified against a booted app.

| ID | Task | Status |
|---|---|---|
| P2-1 | `@Controller('bff')` → `@Controller('session')` | ✅ |
| P2-2 | Collapse `/bff/session/activate` → `/session/activate` | ✅ `@Post('session/activate')` → `@Post('activate')`; verified `/session/session/activate` returns 404 |
| P2-3 | Rename the internal header to `x-session-secret` | ✅ **Task was wrong — see note** |
| P2-4 | CORS handling of the internal header | ✅ **Task was wrong — see note** |
| P2-5 | `BFF_INTERNAL_SECRET` → `SESSION_INTERNAL_SECRET`; `SESSION_SECRET` → `SESSION_COOKIE_SECRET` | ✅ `env.validation.ts`, `session.service.ts`, `session.controller.ts`, `.env.example`. `BFF_PUBLIC_URL` is **not used in this repo** |
| P2-6 | Redis prefixes → `session:data:` / `session:handoff:` / `session:refresh-lock:` | ✅ No `session:session:` |
| P2-7 | Metrics → `session_miss_total`, `session_proxy_duration_seconds` | ✅ Providers, `@InjectMetric`, and specs |
| P2-8 | pino `service: 'bff'` → `'session'` | ✅ |
| P2-9 | Descriptor: `HEALTH_PATH`, `PROD_NAME`/`PROD_CONTAINER_NAME`/`PROD_IMAGE`, `REQUIRED_ENV_VARS` | ✅ All now `arm-session` / `/session/health` |
| P2-10 | `package.json` name → `arm-session` | ✅ |
| P2-11 | Rewrite in-repo code comments referring to "the BFF" | ✅ *(added)* |
| P2-12 | `Makefile` header → `make dev-session` / `make test-session` | ✅ *(added, matches P4-6)* |

### ⚠️ Two tasks were specified wrongly — corrected during execution

**P2-3 had the data flow backwards.** `arm-session` **never sends** the internal secret.
It only **verifies** it inbound on `/session/internal/create-session`
(`session.controller.ts:606`) and **strips** it from proxied requests
(`proxy/proxy.service.ts:55`). Outbound calls authenticate with
`Authorization: Bearer {accessToken}`, not the shared secret.

> **This invalidates Phase 3's premise that `arm-session` is a sender.** It is a
> *verifier*. `arm-core-be` is the sender to `/session/internal/create-session`.
> **Phase 3 must first trace who actually calls the `SessionSecretGuard`-protected
> routes on core-be, calendar-be and admin-be** — it is not `arm-session`.
> See [P3-0](#3-0--trace-the-real-callers-before-editing).

**P2-4 asked for the opposite of the design.** CORS deliberately **excludes** the
internal header — `ALLOWED_REQUEST_HEADERS` in `config/cors.options.ts` lists only
`content-type`, `cache-control`, `pragma`, `x-correlation-id`, and
`session.cors.spec.ts:163` asserts the header is **absent** from the preflight
response. The real task was updating that comment and the negative assertion, which
is what was done. **Do not add `x-session-secret` to `allowedHeaders`.**

**Exit:** ✅ build, tests, lint pass. **No `bff` remains in `src/`** except one repo-path
comment (`session.controller.spec.ts:6`) that **P4-3** sweeps with the directory rename.

**Deferred to Phase 6 (this repo's own docs, still describing `/bff/*` and `src/bff/`):**
`README.md`, `CLAUDE.md`, `TASK_SHEET.md`, `src/readme.md`, `docs/readme.md`,
`tests/readme.md`, `scripts/motd`. Added as **P6-12**.

**Flagged, not actioned:** `tests/app.e2e-spec.ts` is leftover Nest scaffolding — it
asserts `GET /` returns `Hello World!`, a route this service has never had. Pre-existing
and unrelated to the rename; it would fail if `pnpm test:e2e` were run.

---

## Phase 3 — Consumer repos ✅ complete

**Goal:** every service that talks to the session service switches names.
**Done:** 2026-08-25 on `refactor/rename-bff-to-session` in 4 repos. Uncommitted.

| Repo | Build | Tests |
|---|---|---|
| `arm-core-be` | ✅ | ✅ 35 suites / 301 tests — **matches baseline** |
| `arm-app-calendar` (be) | ✅ | ✅ 5 failed / 59 passed · 10 failed / 600 passed — **exactly the baseline 10, same test names** |
| `arm-app-calendar` (fe) | ⚠️ pre-existing failure | ✅ 7 files / 97 tests |
| `arm-admin` (be) | ✅ | ✅ 7 suites / 71 tests |
| `arm-admin` (fe) | ✅ | ✅ 4 files / 23 tests |
| `arm-core-fe` | ⚠️ pre-existing failure | ✅ 13 files / 73 tests |

### 3-0 — Trace result (the corrected sender/verifier map)

Phase 2 showed `arm-session` is a **verifier**, not a sender. The full map:

| Sender | Verifier | Route(s) |
|---|---|---|
| `arm-core-be` `auth.controller.ts:233,391` | **`arm-session`** | `POST /session/internal/create-session` *(done in P2)* |
| `arm-app-calendar` (be) `core-auth.service.ts:42` | **`arm-core-be`** `SessionSecretGuard` | `auth.controller:289,443`, `registry.controller:32` |
| ⚠️ **operator / external — no in-code caller** | **`arm-app-calendar`** (be) `InternalSecretGuard` | `POST /webhook/renew-all` |
| — none — | `arm-admin` (be) | **none** |

**Two dead-but-required env vars found:**
- `arm-admin` (be) **requires `BFF_INTERNAL_SECRET` at startup but never reads it** — no guard, no sender. Renamed to `SESSION_INTERNAL_SECRET` to avoid a startup failure after the `.env` rename, but it is dead config.
- `arm-core-be`'s descriptor **requires `SESSION_SECRET` but never uses it** — not in `src/`, not in `.env.example`. Renamed to `SESSION_COOKIE_SECRET` for the same reason. Also dead.

> Both were renamed mechanically to preserve current behaviour. Removing them is a
> separate decision.

### Work done

| ID | Task | Status |
|---|---|---|
| P3-0 | Trace real senders and verifiers | ✅ Map above |
| P3-1 | `BffSecretGuard` → `SessionSecretGuard`, file renamed | ✅ `src/auth/guards/session-secret.guard.ts` (+ spec) |
| P3-2 | core-be verifies **and sends** `x-session-secret` | ✅ Guard + `auth.controller.ts:233,391` |
| P3-3 | core-be env: `BFF_INTERNAL_SECRET`/`BFF_URL`/`BFF_PUBLIC_URL` → `SESSION_*` | ✅ + `SESSION_SECRET` → `SESSION_COOKIE_SECRET` in the descriptor |
| P3-4 | core-be routes it calls: `/session/internal/create-session`, `/session/oauth/callback` | ✅ |
| P3-5 | calendar-be verifier + sender switch to `x-session-secret` | ✅ `internal-secret.guard.ts`, `core-auth.service.ts` |
| P3-6 | calendar-be env | ✅ |
| P3-7 | calendar-fe `VITE_SESSION_URL`, `/session/*` | ✅ |
| P3-8 | calendar-fe env + Docker + Makefile | ✅ |
| P3-9 | admin-be env | ✅ |
| P3-10 | admin-fe `SESSION_BASE_URL`, `/session/proxy/admin/v1` | ✅ |
| P3-11 | admin-fe env + Docker + Makefile + tests | ✅ |
| P3-12 | core-fe `bff-routes.ts` → `session-routes.ts`, `BFF_ROUTES` → `SESSION_ROUTES` | ✅ |
| P3-13 | core-fe barrel export + consumers | ✅ |
| P3-14 | core-fe `SESSION_BASE_URL`, `VITE_SESSION_URL` | ✅ |
| P3-15 | core-fe env + Docker + Makefile + tests | ✅ |
| P3-16 | Rewrite prose comments across all 4 repos | ✅ *(added)* |
| P3-17 | core-fe descriptor `DEPENDS_ON_LOCAL=bff` → `session` | ✅ *(added, matches P4-4)* |

### 🚨 Blocking discovery — OAuth redirect URIs are externally registered

calendar-be's OAuth callback URLs contain the route prefix:

```
OAUTH_CALLBACK_URL            = http://localhost:5001/bff/calendar/oauth/callback
MICROSOFT_OAUTH_CALLBACK_URL  = http://localhost:5001/bff/calendar/oauth/callback/microsoft
```

Both are now `/session/calendar/oauth/callback[/microsoft]` in code and `.env.example`.
**But these URLs are registered with the identity providers.** Unless the new paths are
added as authorised redirect URIs *before* cutover, Google and Microsoft will reject
every login and add-account attempt with `redirect_uri_mismatch`.

Added as **P7-10 / P7-11**. This is an external dependency with lead time — treat it as
the long pole.

### Baseline gap closed

`make test` ran **backend jest suites only** — no frontends, and **no admin at all**.
It also never typechecked anything, and `vitest` transpiles without typechecking, so a
frontend type error passed `pnpm test` and surfaced only at `pnpm build`.

That blind spot was hiding three real errors on `dev`, all in `.test.tsx` files:

| Repo | Error | Fix |
|---|---|---|
| `arm-app-calendar` (fe) | `calendar-account-page.test.tsx:89` TS6133 — `historyMap` unused | renamed to `_historyMap` (leading `_` is exempt under `noUnusedParameters`) |
| `arm-core-fe` | `remote-error-boundary.test.tsx:16` TS2786 — `ThrowingChild` infers `void`, not a valid JSX component | annotated `: never`, which is assignable to `ReactNode` |
| `arm-core-fe` | `home-page.test.tsx:105` TS2375 — `firstName: undefined` under `exactOptionalPropertyTypes` | destructure the key out instead of assigning `undefined` |

All three frontends now build. **Both fixes and the coverage change are unrelated to the
rename** — see the branch note below.

#### `make test` now covers all 8 services + typecheck

Added to `deploy/arm-deploy-make/make/test.mk`:

| Target | Covers |
|---|---|
| `test-admin` | admin-be jest — **was not in `make test` at all** |
| `test-core-fe` | core-fe vitest — new |
| `test-calendar-fe` | calendar-fe vitest — new |
| `test-admin-fe` | admin-fe vitest — new |
| `typecheck` | `tsc -b --noEmit` across all 3 frontends — new |

`test` now depends on all of them. The `typecheck` target was verified non-vacuous: a
deliberately reintroduced type error made it print the error and exit non-zero.

> ⚠️ **Branch placement.** The `test.mk` change sits in `arm-deploy-make`'s **`main`**
> working tree (uncommitted), and the three type fixes sit on the **rename branches** of
> `arm-core-fe` and `arm-app-calendar`. None of it is rename work. Commit these
> separately from the rename so they can land — or be reverted — independently.

---

## Phase 4 — Deploy & infrastructure ✅ complete (1 blocked)

**Done:** 2026-08-25 on `refactor/rename-bff-to-session`. Uncommitted.
**Verified:** `make test-self` 27/27 · `make env-check` passes · `make descriptor-check
NAME=session` valid · `make generate-compose` emits `docker-compose.session.yml` with
`arm-session-local` and `/session/health` · `make test` still exits **0**.

| ID | Task | Status |
|---|---|---|
| **P4-1** | Rename GitHub repo `arm-core-bff` → `arm-session` | 🚫 **BLOCKED — needs your authorization.** See below |
| P4-2 | `repos.conf` repo name **and** path | ✅ |
| P4-3 | Move `core/arm-bff` → `core/arm-session` | ✅ 32 uncommitted changes carried intact |
| P4-4 | `services.conf` key `bff` → `session` | ✅ |
| P4-5 | `PATH_BFF` → `PATH_SESSION` | ✅ |
| P4-6 | 6 make targets renamed | ✅ `session`, `dev-session`, `install-session`, `logs-session`, `test-session`, `test-session-cov` |
| P4-7 | `make help` text | ✅ |
| P4-8 | Containers → `arm-session` / `arm-session-local` | ✅ |
| P4-9 | Container-name exception in `generate-compose.sh` | ✅ **Comment only — see note** |
| P4-10 | Compose includes → `docker-compose.session.yml` | ✅ + removed the orphaned generated `docker-compose.bff.yml` files |
| P4-11 | `VITE_SESSION_URL` in the FE template | ✅ |
| P4-12 | Template comments | ✅ |
| P4-13 | Caddy `handle /session/*` → `arm-session:5000` | ✅ |
| P4-14 | Health-check URLs | ✅ |
| P4-15 | Prometheus `job_name: session`, target `arm-session:5000` | ✅ |
| P4-16 | Grafana alert rule regex | ✅ |
| P4-17 | Grafana dashboard regex, title, metric | ✅ |
| P4-18 | CI `CONTAINERS` list | ✅ |
| P4-19 | `.env.example`, `env-init`, `REQUIRED_CORE_VARS` | ✅ |
| P4-20 | 4 env test fixtures | ✅ |
| P4-21 | `docs/service-descriptor.md` | ✅ |
| P4-22 | Root workspace: **both** `.env` files, root `docker-compose.yml`, `.claude/settings.json` | ✅ |
| P4-23 | Complete `SESSION_SECRET` → `SESSION_COOKIE_SECRET` here | ✅ *(added — see note)* |

### 🚫 P4-1 is blocked and needs you

`gh` is authenticated as `kishor-aracreate`, so the rename is technically possible from
here — but renaming a shared org repo is outward-facing and breaks every existing clone's
remote until GitHub's redirect is relied on. **Not done unilaterally.** Everything else in
P4 already points at `arm-session`, so this is the last step and can be done at any time
before P8.

### Notes on two tasks that were not what the plan assumed

**P4-9 — the "exception" is a comment, not code.** `generate-compose.sh` reads `PROD_NAME`
and `PROD_CONTAINER_NAME` from each descriptor generically; nothing special-cases the
session service. **The irregularity also survives the rename**: `PROD_NAME` is `arm-session`
(the prod *compose service key*, which `depends_on:` and Caddy reference), so it is still
not `arm-` + service key. Normalising `PROD_NAME` to `session` is a separate change with
its own blast radius and was **not** done. The comment now says so.

**P4-23 was missing from the plan entirely.** Decision 6 renamed `SESSION_SECRET` →
`SESSION_COOKIE_SECRET`, and P2/P3 applied it in `arm-session` and `arm-core-be` — but the
deploy repo still *generated and validated* `SESSION_SECRET`. `arm-session` calls
`getOrThrow('SESSION_COOKIE_SECRET')`, so **it would have failed to boot**. Caught by
`make env-check`. `AUTHELIA_SESSION_SECRET` was deliberately left alone.

### Two findings

**There are two separate `.env` files.** `ENV_FILE := $(MAKEFILE_DIR).env` — `make` and
compose read `deploy/arm-deploy-make/.env` (5402 bytes), **not** the workspace-root `.env`
(2037 bytes). Both existed with overlapping keys and both needed the rename. This is a
standing drift risk worth collapsing — added as **P10-12**.

**Word-boundary trap, twice.** `\bbff_proxy_duration_seconds\b` does not match
`bff_proxy_duration_seconds_bucket` (a `_` follows), so the Grafana dashboard query was
silently missed on the first pass. The same rule is what safely protected
`AUTHELIA_SESSION_SECRET` and `VITE_BFF_URL` from unintended matches. A suffix sweep
(`bff_[a-z_]*`) confirmed nothing else was missed.

---

## Phase 5 — `arm-tool-cli` generators ✅ complete

**Goal:** stop scaffolding new services with stale references.
**Done:** 2026-08-25 on `refactor/rename-bff-to-session`. Uncommitted.
**Result:** build ok · **115 of 116 tests pass** (the 1 failure is pre-existing — see below)
· **verified end to end by scaffolding a real service**.

### P7-8 resolved — the branch base did not matter

`arm-tool-cli` sat on `fix/dead-exports`, one commit (`ae5402e`) ahead of `dev`. That
commit touches only `constants/*`, `scaffolder/create-app.ts` and
`conformance/repo-files.ts` — and **`repo-files.ts` contains no `bff` reference on
either branch**. All 9 P5 target files are byte-identical across the two bases, so the
choice was moot for this phase. Work proceeded on the `dev`-based branch; `ae5402e`
remains untouched on `fix/dead-exports`.

| ID | Task | Status |
|---|---|---|
| P5-1 | Hardcoded path → `core/arm-session/src/session/proxy/proxy.service.ts` | ✅ `emit-registration.ts`, `build-registration.ts` |
| P5-2 | Emitted instruction: `SESSION_INTERNAL_SECRET` | ✅ |
| P5-3 | Port map `[5001, "bff"]` → `[5001, "session"]` | ✅ + the `already used by session` collision message and its test |
| P5-4 | Generated README text → `/session/proxy/<name>/v1` | ✅ |
| P5-5 | Generated FE config `BFF_BASE_URL` → `SESSION_BASE_URL` | ✅ + `VITE_BFF_URL` → `VITE_SESSION_URL` |
| P5-6 | Generated BE guard comments | ✅ |
| P5-7 | Descriptor `requiredEnvVars` | ✅ |
| P5-8 | Update assertions | ✅ `conformance.test.ts`, `registration.test.ts` |
| P5-9 | Confirm branch base | ✅ Moot — see above |
| P5-10 | `docker-files.ts` — generated `ARG`/`ENV VITE_SESSION_URL` | ✅ *(added — not in the original list)* |
| P5-11 | `ui-components.ts` CLI output text | ✅ *(added)* |
| P5-12 | `README.md` registration-steps path | ✅ *(added)* |

### Functional verification

Scaffolded a throwaway app (`--kind app --type backend --db PostgreSQL`) from the built
CLI and grepped the output:

```
grep -rn -i "bff" <generated service>   →   (none)
```

The generated service emits `session` naming throughout — `REGISTRATION.md` points at
`core/arm-session/src/session/proxy/proxy.service.ts`, the FE constant is
`SESSION_BASE_URL` → `/session/proxy/<name>/v1`, the Dockerfile declares
`VITE_SESSION_URL`, and the backend descriptor requires `SESSION_INTERNAL_SECRET`.

### ⚠️ Pre-existing failure, not caused by the rename

`integration.test.ts › should create a React service project successfully` **hangs**
until the timeout (siblings finish in ~10 ms). Verified identical on pristine `dev` via
`git stash`. Added as **P10-10**.

> This surfaced a further coverage gap: **`arm-tool-cli` is in neither `repos.conf` nor
> `make test`**, so nothing in the workspace runs its suite. Added as **P10-11**.

**Deferred to Phase 6:** `arm-tool-cli`'s `TASK_SHEET.md` and the stale
`workspace-registration.patch` artefact still mention `bff`.

---

## Phase 6 — Documentation ✅ complete

**Done:** 2026-08-25. Uncommitted. **83 markdown files** updated across all 8 repos.
**Verified:** `make test-self` 27/27 · `make env-check` passes · `make test` exits **0** ·
zero encoding damage across every `.md` in the workspace.

| ID | Task | Status |
|---|---|---|
| P6-1 | `BFF-SESSION-FLOW.md` → `SESSION-FLOW.md` | ✅ `git mv` + all inbound links |
| P6-2 | `architecture/bff-architecture.md` → `session-architecture.md` | ✅ `git mv` + all inbound links |
| P6-3 | `_sidebar.md` + `meta/documentation-inventory.md` | ✅ |
| P6-4 | The 9 heaviest files | ✅ |
| P6-5 | Remaining 46 files | ✅ |
| P6-6 | `REDIS-ARCHITECTURE.md` key namespaces | ✅ matches P2-6 |
| P6-7 | `MONITORING.md` metric names | ✅ matches P2-7 |
| P6-8 | `CICD.md` repos.conf row | ✅ |
| P6-9 | Root `CLAUDE.md` + 7 per-repo `CLAUDE.md` | ✅ |
| P6-10 | Superseding ADR | ✅ **[ADR-009](adr/009-rename-bff-to-session.md)** + indexed in `adr/readme.md` |
| P6-11 | notification's task-sheet prose line | ✅ |
| P6-12 | `arm-session`'s own repo docs (deferred from P2) | ✅ `README.md`, `CLAUDE.md`, `TASK_SHEET.md`, `src/readme.md`, `docs/readme.md`, `tests/readme.md`, `scripts/motd` |
| P6-13 | `arm-tool-cli` `TASK_SHEET.md` + `README.md` (deferred from P5) | ✅ *(added)* |

### Deliberately left saying "bff"

| What | Why |
|---|---|
| `adr/001-standalone-bff.md` — filename and decision text | ADRs record decisions at a point in time. ADR-009 supersedes the *naming*; the original is not rewritten. **Its link targets were repaired** so they still resolve |
| Task IDs `BFF-01`…`BFF-19` | They identify work already done; renaming breaks cross-references for no gain |
| Auth0 and WunderGraph reference links | External articles about the BFF *pattern* — correct as written |
| `BFF-RENAME-*.md` (these three docs) | They document the rename and must name both sides |
| `docs/superpowers/plans/` | Historical plan records, out of scope |

### ⚠️ Broad prose substitution caused collateral damage — all repaired

Replacing the bare token `BFF` → `session service` across 83 files also hit things that
were **not** the service name. Every case was found and repaired, but the pattern is worth
recording:

| Damage | Example | Repair |
|---|---|---|
| Task IDs | `BFF-18` → `session service-18` | Restored to `BFF-18` |
| Filenames | `BFF-SESSION-FLOW.md` → `session service-SESSION-FLOW.md` | Restored, then renamed properly |
| External article titles | "The Backend For Frontend Pattern" → "The session service Pattern" | Restored |
| Headings/labels | "BFF Proxy Duration p95" → "session service Proxy Duration p95" | → "Session Proxy Duration p95" |
| Hyphenated compounds | "BFF-side" → "session service-side" | → "session-service-side" |

**Lesson for any future mass rename here: substitute the *specific* forms first
(`the BFF`, `BFF proxy`, `BFF's`) and only then the bare token — and re-grep for
`<newname>-\d+` and `<newname> [A-Z]` afterwards to catch IDs and titles.**

A second, separate trap: `perl -CSD` with a UTF-8 literal in the replacement mangled an
em-dash to `â\x80\x94` in one file. A full-workspace decode check now confirms **0 files
with encoding damage**.

---

## Phase 7 — Cutover readiness

**Goal:** the human, server-side and decision work parked out of Phase 0.
**Blocks on:** nothing — every item can be actioned today.
**Blocks:** Phase 8. ⚠️ **This phase must complete *before* cutover, not after.**

> A rollback snapshot taken after the cutover is worthless, and the window cannot be
> announced retroactively. "Last phase" here means *last before the deploy* — these
> items are deliberately placed immediately ahead of Phase 8, not after Phase 9.

### 7a — Human / scheduling

| ID | Task | Owner |
|---|---|---|
| P7-1 | Book the maintenance window | you |
| P7-2 | Fill in and schedule the user notice — draft is [P0 record §4](BFF-RENAME-P0-RECORD.md#4-drafted-user-notice-p0-2) | you |

### 7b — Server-side capture (no access from the workspace)

| ID | Task | Command |
|---|---|---|
| P7-3 | Record image tags for all 8 server containers | [P0 record §5](BFF-RENAME-P0-RECORD.md#5-commands-you-need-to-run-on-the-server) |
| P7-4 | Snapshot the server `.env` | [P0 record §5](BFF-RENAME-P0-RECORD.md#5-commands-you-need-to-run-on-the-server) |
| P7-5 | Paste both results into §1 of the P0 record | — |

### 7c — Baseline decisions

| ID | Task | Why it matters |
|---|---|---|
| ~~P7-6~~ | ~~Decide: fix the 10 `calendar-be` failures, or carry them~~ | ✅ **Decided 2026-08-25: carry them.** Moved to [Phase 10](#phase-10--carried-debt). ⚠️ One — `AuthController — Microsoft add-account` — sits in the flow P3-7 touched, so P9-5 must exercise Microsoft add-account **manually**; the automated test cannot confirm it |
| ~~P7-7~~ | ~~Establish a `notification` test baseline~~ | ✅ **Resolved 2026-08-25: 61 tests passing.** `make test` no longer aborts early, so it is now reached and covered |
| ~~P7-8~~ | ~~Confirm `arm-tool-cli`'s branch base~~ | ✅ **Moot** — all 9 P5 files are identical on both bases; `ae5402e` touches no `bff` reference. Proceeded on `dev` |
| P7-10 | 🚨 Register `/session/calendar/oauth/callback` in **Google Cloud Console** (OAuth client → Authorised redirect URIs) | **Blocks cutover.** Without it Google login and add-account fail with `redirect_uri_mismatch` |
| P7-11 | 🚨 Register `/session/calendar/oauth/callback/microsoft` in **Microsoft Entra** (App registration → Redirect URIs) | **Blocks cutover.** Same failure mode for Microsoft |
| ~~P7-12~~ | ~~Decide: fix the 3 pre-existing frontend `tsc -b` errors~~ | ✅ **Resolved 2026-08-25 — fixed, see [§Baseline gap closed](#baseline-gap-closed)** |
| P7-9 | Commit or stash `arm-core-be`'s uncommitted source WIP | Gates **Phase 3a**, not cutover — resolve early |

**Exit:** window booked, notice scheduled, server snapshots pasted into the P0 record,
all four decisions made.

---

## Phase 8 — Cutover

**Goal:** one coordinated deploy in the booked window.
**Blocks on:** P1–P6 merged and reviewed, **and P7 complete**.

| ID | Task | Notes |
|---|---|---|
| P8-1 | Final review across all 8 branches | — |
| P8-2 | Send the "starting now" notice | — |
| P8-3 | Merge all 8 repos to `main` | — |
| P8-4 | Rename the server `.env` keys — values preserved | `BFF_*`→`SESSION_*`, `SESSION_SECRET`→`SESSION_COOKIE_SECRET` |
| P8-5 | `make env-check` on the server | Must pass before deploying |
| P8-6 | Dispatch `deploy-production.yml` from `arm-devops-makefile` `main` | All images together |
| P8-7 | Watch container health | All 8 up |

**Rollback:** redeploy the P7-3 image tags and rename the `.env` keys back. Secret
*values* never changed, so no re-issue is needed.

---

## Phase 9 — Post-cutover verification

**Blocks on:** P8.

| ID | Check |
|---|---|
| P9-1 | `make env-check` passes |
| P9-2 | `make test` exits **0** — all 8 services green plus frontend typecheck (1,329 tests). Any failure is a real regression |
| P9-3 | `make local-dev-ps` — all 8 healthy |
| P9-4 | Login at `localhost:3000` completes; session survives a refresh |
| P9-5 | Google **and** Microsoft add-account popups complete — still worth doing manually because P7-10/11 change the provider-side redirect URI |
| P9-6 | `/session/proxy/{core,calendar,admin}/v1` all return 200 — confirms all 4 header verifiers agree |
| P9-7 | `make edge-up` — Caddy routes `/session/*` |
| P9-8 | Grafana `service-health` renders both panels on the new metric names |
| P9-9 | Loki queries on `service="session"` return logs |
| P9-10 | `grep -ri bff` returns **only** the false positives in inventory §4 |
| P9-11 | *(optional)* `SCAN`/`DEL` orphaned `bff:*` Redis keys to reclaim memory |
| P9-12 | Close the window; confirm users can log back in |

---

## Phase 10 — Carried debt

**Goal:** everything deliberately *not* fixed as part of the rename.
**Blocks on:** nothing. Runs after cutover, or whenever there is room.
**Nothing here blocks the rename.**

### 10a — The 10 `calendar-be` failures ✅ fixed

**Decision reversed 2026-08-25: fixed, not carried.** All 10 now pass; `calendar-be` is
**64 suites / 611 tests, all green**.

Every one was the same root cause: **the implementation evolved and the test doubles or
assertions were never updated**. None was a production bug — verified by reading the
implementation in each case, not by adjusting tests until they went green.

| ID | Test | Root cause | Fix |
|---|---|---|---|
| P10-1a | `getGoogleToken` — not authenticated | Impl **self-heals**: an unauthenticated account that still holds a refresh token gets one refresh attempt. Fixture had a refresh token, so a refresh fired | Narrowed the test to the no-refresh-token case its name describes, **and added the missing sibling test** for the self-heal branch |
| P10-1b | `refreshFromCoreAuth` — no token | Impl reads the account up front for a **provider guard** before calling CoreAuth | Assert the *write* never happens (`encrypt` not called), not that the lookup never happens |
| P10-1c | `refreshFromCoreAuth` — error path | Catch path **re-reads** the account to decide whether to mark it unauthenticated → 2 lookups | Same: assert no write |
| P10-2 | `MicrosoftService.generateAddAccountUrl` | `MicrosoftService` gained a `TokenEncryptionService` dependency; **`state` is now an encrypted, TTL-bearing payload**, not the bare `userId` | Provided the missing DI mock; assert the ciphertext is passed through and the plaintext carries `{ userId, exp }` |
| P10-3 | `AuthController` — Microsoft add-account | `exchangeAddAccountCode` returns `{ alreadyLinked?: boolean }`; the mock returned `undefined`, so the controller threw reading `.alreadyLinked` | Mock resolves the real shape; assert `{ ok: true, alreadyLinked: false }` |
| P10-4a | `previewCalendar` — no SyncConfigs | Impl now also returns `events: []` | Added `events: []` to the expectation |
| P10-4b | `previewCalendar` — PENDING default | Mock `eventRepo` lacked `findByCalendarId`, which the impl now calls | Added it to the mock |
| P10-5a | `handleWebhook` — Microsoft dispatch | Microsoft carries only `subscriptionId`, so the impl looks up by **`findByResourceId`**, not Google's `channelId`+`resourceId` pair | Added the method to the mock; assert the resourceId-only lookup |
| P10-5b/c | `pollTargetCalendar` ×2 | Mock `SyncGroupMetaRepository` was `{}`; impl calls `findByTargetCalendarId`. Assertion also expected the **`[ BLOCKER ]` title prefix**, which the toggles work removed — `CLAUDE.md` documents that nothing writes it any more | Mocked the repo; assert `summary: 'Busy'` |

> **P10-3's caveat is lifted.** It sat in the Microsoft add-account flow P3-7 touched and
> was masking any regression there. It now genuinely covers that path, so **P9-5 no
> longer depends on manual verification alone** — though exercising the real popup at
> cutover is still worth doing, since P7-10/11 change the provider-side redirect URI.

> **One coverage gain:** the self-heal branch in `refreshTokenIfExpiredForAccount` had no
> test at all. P10-1a added one.

### 10b — Dead configuration found during P3

| ID | Task |
|---|---|
| P10-6 | `arm-admin` (be) requires `SESSION_INTERNAL_SECRET` at startup but **never reads it** — no guard, no sender. Renamed mechanically in P3-9 to avoid a startup failure; decide whether to drop it |
| P10-7 | `arm-core-be`'s descriptor requires `SESSION_COOKIE_SECRET` but it appears in neither `src/` nor `.env.example`. Renamed mechanically in P3-3; decide whether to drop it |

### 10c — Pre-existing breakage, unrelated to the rename

| ID | Task |
|---|---|
| P10-8 | `arm-session` `tests/app.e2e-spec.ts` is leftover Nest scaffolding asserting `GET /` returns `Hello World!` — a route this service has never had. Fails if `pnpm test:e2e` is run; not wired into `make test` |
| P10-9 | `PATH_LIB_UI` in `make/config.mk` points at `library/arm-library-ui-components`; the directory on disk is `library/arm-ui-library`, and no target references the variable |
| P10-10 | `arm-tool-cli` `integration.test.ts › should create a React service project successfully` hangs until timeout. Verified pre-existing on `dev` |
| P10-11 | `arm-tool-cli` is in neither `repos.conf` nor `make test` — nothing in the workspace runs its 116 tests |
| P10-12 | Two `.env` files with overlapping keys: workspace root and `deploy/arm-deploy-make/`. Only the latter is read by `make`/compose. Collapse to one |
| P10-13 | `PROD_NAME=arm-session` is still not `arm-` + service key, so `PROD_CONTAINER_NAME` remains a separate explicit field. Normalising it would touch prod compose keys, `depends_on:` and Caddy |

---

## Full test baseline after the coverage fix

`make test` now runs every suite and reports at the end rather than stopping at the
first failure.

| Suite | Result |
|---|---|
| core-be | ✅ 301 |
| **arm-session** | ✅ 92 |
| calendar-be | ✅ **611** — all 10 failures fixed |
| admin-be | ✅ 71 |
| core-fe | ✅ 73 |
| calendar-fe | ✅ 97 |
| admin-fe | ✅ 23 |
| notification | ✅ 61 — **first ever baseline** |
| frontend typecheck | ✅ all 3 OK |
| **Total** | **1,329 tests · 0 failing** |

`make test` now exits **0** with every suite passing and the frontend typecheck clean —
the first fully green run in this workspace.

---

## Task count by phase

| Phase | Tasks | Status / can start |
|---|---:|---|
| P0 Pre-flight | 4 | ✅ complete |
| P1 `arm-session` internals | 10 | ✅ complete |
| P2 `arm-session` wire surface | 12 | ✅ complete |
| P3 Consumer repos | 18 | ✅ complete |
| P4 Deploy & infra | 23 | ✅ complete (P4-1 blocked) |
| P5 `arm-tool-cli` | 12 | ✅ complete |
| P6 Docs | 13 | ✅ complete |
| P7 Cutover readiness | 12 | **now** ⚠️ P7-10/11 external lead time; P7-8/12 done |
| P8 Cutover | 7 | after P1–P7 |
| P9 Verification | 12 | after P8 |
| P10 Carried debt | 13 | 10a ✅ done; 10b/10c open |
| **Total** | **137** | |
