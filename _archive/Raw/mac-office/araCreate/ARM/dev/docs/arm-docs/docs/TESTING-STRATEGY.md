# ARM Platform — Testing Strategy

This document describes how testing actually works across ARM's 8 repos today: the test pyramid each service really has (not an aspirational one), NestJS unit/integration conventions, React component test conventions, the one real E2E suite that exists, and — critically — which of these suites CI actually runs. Several services have meaningfully more test infrastructure than their CI pipeline exercises, and two frontends have none at all; both are stated plainly below rather than assumed away.

See [CODING-STANDARDS.md](CODING-STANDARDS.md) §6 for the adjacent, already-established facts this document builds on: `.spec.ts` colocated with source, no platform-wide coverage threshold, and pre-commit enforcement limited to `arm-service-notification`. This document doesn't repeat those, it goes one level deeper — layers, conventions, and CI wiring.

---

## 1. The test pyramid, per service

There is no single platform-wide pyramid — each service has its own shape, and three services have no pyramid above "unit" at all.

| Service | Unit | Integration | E2E |
|---|---|---|---|
| `arm-core-be` | 28 `.spec.ts` (Jest) | — | 3 specs, `TestingModule`-only (not real DB — see §3) |
| `arm-session` | 7 `.spec.ts` (Jest) | — | 1 spec, unmodified NestJS boilerplate, **broken** (see §3) |
| `arm-app-calendar` (calendar-be) | 56 `.spec.ts` (Jest) | Yes — real MongoDB + Redis via Testcontainers | Yes — same Testcontainers lifecycle, real HTTP via supertest |
| `arm-admin` (admin-be) | 4 `.spec.ts` (Jest) | — | 1 spec, unmodified NestJS boilerplate, **broken** (see §3) |
| `arm-service-notification` | 3 `.spec.ts` (Jest) | — | 1 spec, real logic behind mocked `MailService`/`DlqService` (only genuinely useful "e2e" spec of the four boilerplate-named ones) |
| `arm-core-fe` | 8 `.test.tsx` (Vitest + Testing Library) | — | Yes — Playwright, 2 specs (`login`, `home`) |
| `apps/arm-app-calendar` frontend (calendar-fe) | **none** | — | — |
| `apps/arm-admin` frontend (admin-fe) | **none** | — | — |
| `arm-ui-library` | 51 `.test.tsx` (Vitest + Testing Library + `jest-axe`) | — | — (component-level only; see §4) |

**Reading this table:** `calendar-be` is the only backend with a real integration layer, and the only service on the platform with a true, isolated E2E suite. `core-fe` is the only frontend with browser-level E2E coverage. Every number above is a repo-relative fact, not a platform target — there is no stated "every service should have N unit tests" policy anywhere.

---

## 2. NestJS unit test conventions

All 4 NestJS backends (`core-be`, `calendar-be`, `admin-be`, `arm-service-notification`) use the same shape, run via `pnpm test` (Jest, `testRegex: ".*\\.spec\\.ts$"`, colocated with source per [CODING-STANDARDS.md](CODING-STANDARDS.md) §2):

- `Test.createTestingModule({...}).compile()` from `@nestjs/testing`, with every dependency of the unit under test replaced by a hand-written mock object (`jest.fn()` per method), not the real provider. `AuthService`'s spec is representative: `UsersService`, `AuthRepository`, `JwtService`, `RedisService`, `NotificationProducerService`, `GoogleAuthService`, and `MicrosoftAuthService` are all plain object mocks, not `useValue`-wrapped real instances.
- No database, no network, no real dependency ever runs inside a `.spec.ts` file on this platform. If a test needs a real Mongo/Postgres/Redis connection, it belongs in `calendar-be`'s integration layer (§3), not here — that boundary is drawn consistently across all 56 of `calendar-be`'s own unit specs, which mock `MongooseModel`/`RedisService` the same way `core-be` mocks TypeORM repositories.
- **Mocking is always explicit, never `jest.mock()` module auto-mocking** — every backend spec constructs its own mock object literal and wires it in via the `TestingModule` providers array. Follow this pattern for a new backend spec rather than introducing automocking, which nothing else in the platform uses.

---

## 3. NestJS integration/E2E — real per-repo divergence

This is the platform's biggest inconsistency in testing conventions, and it's worth stating precisely rather than glossing as "everyone has integration tests":

### `calendar-be` — the only real integration/E2E layer

`calendar-be` alone has `test:integration` and `test:e2e` npm scripts backed by **Testcontainers** (`@testcontainers/mongodb`, `@testcontainers/redis`), each with its own Jest config:

- **Integration** (`jest.config.integration.ts`): `testMatch: src/**/*.integration-spec.ts`, files colocated with the unit specs they extend (e.g. `webhook-renewal.service.integration-spec.ts` next to `webhook-renewal.service.spec.ts`).
- **E2E** (`jest.config.e2e.ts`): `testMatch: tests/**/*.e2e-spec.ts`, real HTTP via `supertest` against a booted Nest app.
- Both share one container lifecycle: `tests/setup/containers.ts` (Jest `globalSetup`) starts a real `mongo:7` and `redis:7-alpine` container **once** for the whole run and writes connection details to a state file; `tests/setup/containers-teardown.ts` tears them down. `--runInBand` keeps everything in one Node process so `process.env` set by `globalSetup` is visible to every test file. 60s per-test timeout — module compilation plus real Mongo driver topology discovery needs the headroom.
- This is real, working infrastructure — and per §5, **CI never runs it.** `pnpm test` (unit only) is the only test command calendar-be's own `ci.yml` invokes.

### `core-be`, `admin-be`, `arm-session` — "e2e" that isn't

All three ship a `tests/*.e2e-spec.ts` + `tests/jest-e2e.json`, run via `test:e2e`, but none of them touch a real database:

- **`core-be`**: its e2e specs (`app.e2e-spec.ts`, `auth-otp-throttling.e2e-spec.ts`, `throttling.e2e-spec.ts`) boot a real `INestApplication` via `supertest`, but every service dependency (`AuthService`, `UsersService`, `GoogleAuthService`, etc.) is still a hand-written mock passed through the `TestingModule` — the file header for the auth suite says so explicitly: *"e2e suite exercising AuthController/UsersController against a real, fully-mocked TestingModule."* This is closer to a controller-integration test than a true end-to-end test; it verifies HTTP wiring (guards, pipes, status codes, cookie handling) with the business logic swapped out, not real persistence.
- **`arm-session` and `admin-be`**: their `tests/app.e2e-spec.ts` is the **unmodified NestJS CLI scaffold** — `imports: [AppModule]`, then `GET / → expect 200 'Hello World!'`. Neither `AppModule` actually registers an `AppController` (confirmed: no `app.controller.ts` exists in either repo, and neither module's `controllers:` array references one) — this test would fail if it ever ran. Per §5, it never does: neither repo's CI invokes `test:e2e`, so this dead scaffold has sat unexercised and unnoticed. Don't treat `test:e2e` as meaningful coverage for either service; treat it as a leftover from `nest new` that needs deleting or replacing, not extending.
- **`arm-service-notification`**: also scaffold-named (`app.e2e-spec.ts`), but unlike the two above it was actually rewritten — it exercises `NotificationController` directly (not via HTTP) against mocked `MailService`/`DlqService`, asserting welcome-email, OTP-email, and DLQ-routing-on-failure behavior. Functionally this is another unit test with an e2e filename, not a true end-to-end test, but at least it tests real logic.

**Practical implication:** "does this service have e2e tests" is not a yes/no question on this platform — check which of the three shapes above a given repo actually has before assuming coverage.

---

## 4. React component test conventions (Vitest + Testing Library)

Two of the three frontend surfaces have component tests; two have none (see §1).

- **`core-fe`** (8 test files) and **`arm-ui-library`** (51 test files) both use Vitest + `@testing-library/react` + `@testing-library/user-event`, `jsdom` environment.
- **Mocking pattern in `core-fe`**: `vi.mock()` for module-level mocks (e.g. `react-router-dom`'s `useNavigate`, wrapped via `importOriginal` to keep the rest of the module real), `vi.hoisted()` for mocks referenced inside a `vi.mock()` factory (e.g. the shared `toast` module), and **MSW** (`msw` is a devDependency) for mocking network calls rather than stubbing `fetch` directly. Routing-dependent components are rendered inside `<MemoryRouter>` with explicit `initialEntries`, not the real browser router.
- **`arm-ui-library`** additionally runs `jest-axe` accessibility assertions (`axe(container)`) in its component specs — this is the only place on the platform automated accessibility testing happens. It also ships a **second, separate Vitest config** (`vitest.config.ts`, distinct from `vitest.components.config.ts`) that runs Storybook's own interaction tests via `@storybook/addon-vitest` in a real headless Chromium (`@vitest/browser` + `playwright` provider) — two different test runs, two different purposes: `test` (npm script) runs component specs in `jsdom`; `test:storybook` runs story-level interaction tests in a real browser. Don't conflate the two when adding a new component — a `.test.tsx` file belongs to the former, a `.stories.tsx` interaction play function to the latter.
- **`calendar-fe` and `admin-fe` have zero test tooling** — no Vitest, no Testing Library, no test script in `package.json`, confirmed by dependency inspection (neither `vitest` nor `@testing-library/*` appears in either repo's `package.json` at all). This isn't an oversight limited to one repo; it's consistent across both Module Federation remotes other than the shell. Any new component test convention for either remote would be introduced from scratch, not extended from an existing pattern — there's nothing to copy from within the same repo, only from `core-fe` or `arm-ui-library` across the workspace.

---

## 5. E2E test approach

**Only `core-fe` has a real, automated browser E2E suite** — Playwright, `e2e/` directory, 2 specs (`login.spec.ts`, `home.spec.ts`), configured in `playwright.config.ts`:

- Chromium only, `fullyParallel: true`, CI-only retries (2) and single worker.
- `trace: 'retain-on-failure'`, `screenshot: 'only-on-failure'`, `video: 'retain-on-failure'` — full failure artifacts, not just a pass/fail signal.
- `webServer` block starts `pnpm dev` automatically and waits for `localhost:3000`; locally it reuses an already-running dev server, in CI it always starts fresh.
- Runs in a **separate** workflow (`e2e.yml`) from the repo's own `ci.yml` — see §6 for why that split matters. `e2e.yml` sets `VITE_BASE_URL`/`VITE_SESSION_URL` pointing at `localhost:4000`/`5000` and a calendar remote URL, but the workflow itself brings up no backend, no session service, and no calendar-fe container — these Playwright specs exercise `core-fe`'s own UI against whatever those URLs resolve to in the runner, which is nothing unless the specs themselves are written to tolerate failed network calls. Worth verifying directly (open `e2e/login.spec.ts`/`home.spec.ts`) before assuming this suite proves a real cross-service login flow in CI, rather than just UI behavior with mocked or absent responses.

**No other service has an automated E2E/browser suite and no documented manual test checklist exists anywhere in the workspace either.** For `calendar-fe`, `admin-fe`, and every cross-service flow not covered by `core-fe`'s two Playwright specs (Google/Microsoft OAuth end-to-end, calendar sync, admin user management), verification today is manual and ad hoc — `make arm-run` + click through the UI at `localhost:3000`, per [ONBOARDING.md](ONBOARDING.md). This is a real gap, not an oversight in this document: there is nothing to link to because nothing exists.

---

## 6. CI test run — what actually executes on push/PR

This is the section most worth reading before assuming "tests pass in CI" means what it sounds like. Full workflow inventory is [CICD.md](CICD.md); this table extracts only the test-relevant commands, verified directly against each repo's `.github/workflows/*.yml`:

| Repo | Workflow | Trigger | Commands run |
|---|---|---|---|
| `arm-core-be` | `ci.yml` | push/PR `main`, `dev` | `pnpm lint` → `pnpm test` → `pnpm test:e2e` (the mocked-`TestingModule` kind, §3) |
| `arm-session` | `dev-release.yml` | push `main`, `dev` (not PR) | `pnpm test` (unit only — the broken `test:e2e` scaffold, §3, is never invoked) |
| `arm-app-calendar` (calendar-be) | `ci.yml` | push/PR `main`, `dev` | `pnpm audit` → `pnpm lint` → `pnpm test` (unit only — **integration and E2E, both real and working with Testcontainers, never run in CI**) |
| `arm-admin` | — | — | **No workflow exists.** Confirmed: no `.github/workflows/` directory in the repo at all. Neither the 4 unit specs nor the broken e2e scaffold run anywhere automated — not in per-repo CI (none exists) and not in the workspace-level `make test` either (see below). |
| `arm-service-notification` | `dev-lint-test.yml` + `dev-release.yml` (overlapping) | push/PR `main`, `dev` | `pnpm lint` → `pnpm test` (unit only) |
| `arm-core-fe` | `ci.yml` | push/PR `main`, `dev` | `pnpm lint` → `pnpm exec tsc --noEmit` — **`pnpm test` (the 8 Vitest unit specs) is never invoked by this workflow.** |
| `arm-core-fe` | `e2e.yml` (separate workflow) | push/PR `main`, `dev` | Playwright install → `pnpm test:e2e` |
| `arm-ui-library` | `ci.yml` | push/PR `main`, `dev` | `pnpm type-check` → `pnpm build` → `pnpm test` (component specs) |

**Net effect, stated plainly:**
- `core-fe`'s own component-level unit tests (Vitest) never run in any CI workflow — only lint, typecheck, and the separate Playwright suite do. The 8 `.test.tsx` files are exercised locally/manually only.
- `calendar-be`'s integration and E2E suites — its most substantial test investment on the platform — never run in CI. Only its unit specs do.
- `admin-be` has no automated testing whatsoever, at any layer, in any environment.

### Workspace-level `make test`

`deploy/arm-deploy-make/make/test.mk`'s `test` target (used by `ci-test` → `ci-setup + install + test`) runs:

```
test-core test-session test-calendar test-notification
```

Each sub-target is `cd <service> && pnpm test` (unit only, matching each repo's own `ci.yml` command). **`arm-admin` and `arm-ui-library` are not included in this target at all** — confirmed by reading `test.mk` directly, there is no `test-admin` target. This workspace-level command reflects the same per-repo gaps as the table above; it does not add coverage the per-repo CI workflows lack, it only orchestrates the same `pnpm test` commands across the 4 services that were wired in when this Makefile module was written.

---

## 7. Coverage thresholds per service

**None exist, anywhere on the platform.** No `jest.config`/`package.json` `jest` block sets `coverageThreshold`, and no Vitest config sets `test.coverage.thresholds` — confirmed by grep across every backend and frontend config. This matches [CODING-STANDARDS.md](CODING-STANDARDS.md) §6's already-stated finding; restated here because it's the direct answer to this document's own suggested-sections outline, not assumed to exist just because `test:cov`/`--coverage` scripts are wired up in most `package.json` files. Having a `test:cov` script is not the same as a threshold being enforced — every one of those scripts just prints a report; nothing fails the build if coverage drops.

---

## Related documents

- [CODING-STANDARDS.md](CODING-STANDARDS.md) §6 — the adjacent, already-established baseline facts (colocated specs, no coverage threshold, pre-commit scope) this document builds on
- [CICD.md](CICD.md) — full per-repo workflow inventory (this document extracts only the test-relevant subset)
- [calendar/ARCHITECTURE.md](calendar/ARCHITECTURE.md) — the module boundaries `calendar-be`'s facade-pattern services are unit-tested against
- [MODULE-FEDERATION.md](MODULE-FEDERATION.md) — why `calendar-fe`/`admin-fe` are separate Vite apps with no router of their own, relevant context for why they'd need their own test setup rather than inheriting `core-fe`'s
