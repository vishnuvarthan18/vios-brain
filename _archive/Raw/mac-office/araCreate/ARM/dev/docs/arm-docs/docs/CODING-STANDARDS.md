# ARM Platform — Coding Standards & Conventions

This document is the in-code counterpart to [TEMPLATE-CONFORMANCE.md](TEMPLATE-CONFORMANCE.md), which covers repo-level structure, file headers, and naming — deliberately not duplicated here. This document covers the architectural patterns and conventions observed consistently *inside* the code itself: NestJS module structure, error handling, guard/repository/facade patterns, and where consistency actually breaks down across the platform's 5 backends and 3 frontends.

**Everything below reflects patterns actually observed across the codebase, verified during this documentation pass — this is a description of the platform's real conventions, not an aspirational style guide.** Where consistency doesn't actually hold, that's stated plainly rather than papered over.

---

## 1. TypeScript configuration

Every backend uses the same curated subset of strict-mode flags — not a blanket `"strict": true`, but a specific, identical list across `core-be`, `calendar-be`, and `admin-be` (this is the NestJS CLI's own default generated `tsconfig.json`, not a platform-specific hardening):

```json
"strictNullChecks": true,
"noImplicitAny": true,
"strictBindCallApply": true,
"noFallthroughCasesInSwitch": true,
"forceConsistentCasingInFileNames": true
```

`module`/`moduleResolution: "nodenext"` — this is why `.js` extensions appear on relative imports in compiled backend code (`import { X } from './y.js'`) even though the source files are `.ts` — a `nodenext`-specific requirement, not a typo pattern to "fix."

---

## 2. NestJS backend structure

All 4 NestJS backends (`core-be`, `calendar-be`, `admin-be`, `notification-service`) follow the same file-role convention, confirmed identically across every one of them:

| Suffix | Role |
|---|---|
| `*.module.ts` | Wires providers/controllers/imports for one feature area |
| `*.controller.ts` | HTTP routes only — thin, delegates to a service |
| `*.service.ts` | Business logic |
| `*.repository.ts` | Database access, isolated from service logic (seen in `calendar-be`'s `event.repository.ts`/`sync-config.repository.ts`/`sync-history.repository.ts`, `admin-be`'s `users.repository.ts`) |
| `*.dto.ts` | Request/response shape + `class-validator` decorators |
| `*.entity.ts` (TypeORM) / `*.schema.ts` (Mongoose) | Persistence model |
| `*.guard.ts` | Auth/authorization checks |
| `*.spec.ts` | Jest unit test, colocated with the file it tests |

**Global route prefix**: every backend sets `app.setGlobalPrefix('v1')` — confirmed directly in `core-be`, `calendar-be`, and `admin-be`'s `main.ts`. A route documented anywhere in this platform's reference docs without a `/v1/` prefix should be treated as suspect — this exact class of error was caught and fixed in [`calendar/WEBHOOKS.md`](calendar/WEBHOOKS.md) §4 during this documentation pass itself.

### The facade pattern is used repeatedly, on purpose

Several services across the platform are deliberately thin — pure delegation to more focused services, not business logic themselves. Confirmed examples: `calendar-be`'s `EventsService` (exactly 110 lines, delegates to `SyncQueryService`/`BlockerService`/`WebhookProcessorService`/`SyncOrchestratorService`/`SyncLifecycleService` — [`calendar/ARCHITECTURE.md`](calendar/ARCHITECTURE.md) §3), `CalendarService` (resolves a provider via the registry and delegates), `AccountService`. This is a recognized, intentional pattern on this platform, not incidental thinness — when a controller needs to touch more than one concern, prefer adding a facade over growing the controller or one service.

---

## 3. Auth guards

Every non-session service backend uses the same dual-path guard pattern (Kong-trusted `X-User-Id` header vs. direct JWT verification), just named differently per repo (`KongJwtGuard` in `core-be`/`admin-be`, `AuthGuard` in `calendar-be`). Full detail, including the `TRUST_PROXY_HEADERS` security constraint behind it, is in [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) §3 — don't reimplement or hand-roll a different guard pattern for a new backend; follow this one.

**Not consistent across backends**: `ValidationPipe` strictness. [SECURITY.md](SECURITY.md) found this varies between the four backends — check that document before assuming every DTO gets the same validation rigor by default.

---

## 4. Error handling

NestJS's built-in HTTP exception classes (`NotFoundException`, `BadRequestException`, `ForbiddenException`, `HttpException` with an explicit status) are the standard — confirmed as the consistent pattern throughout `calendar-be`'s API-wrapping services (`google-calendar-api.service.ts`, `microsoft-calendar-api.service.ts`) and `admin-be`'s `TypeOrmExceptionFilter` (catches `QueryFailedError` → maps to `400`). Prefer a specific NestJS exception class over a bare `throw new Error(...)` or a hand-rolled response shape — the platform's exception filters and global error handling assume the NestJS exception hierarchy.

---

## 5. Logging

**Never use `console.error`/`console.log`** — this is an explicit, stated rule in at least one service's own `CLAUDE.md` (`calendar-be`: "Do not use `console.error` — use the NestJS `Logger`"), and the pattern holds platform-wide. All 5 backends configure `nestjs-pino`'s `LoggerModule.forRoot()` **identically** — same `redact.paths`, same correlation-ID propagation (`x-correlation-id`/`x-request-id` header, or a generated UUID), same `pino-pretty` dev transport — verified field-by-field in [MONITORING.md](MONITORING.md) §2. Don't introduce a different logging setup for a new service; copy the existing `LoggerModule.forRoot()` config verbatim.

---

## 6. Testing

`.spec.ts` files colocated with the code they test, run via Jest (NestJS's default). **No platform-wide coverage threshold is enforced or documented anywhere** — this document does not invent one; treat "what % coverage is expected" as genuinely unanswered rather than assuming an unstated standard exists.

**Pre-commit enforcement is not platform-wide.** Only `services/arm-service-notification` has a `.husky` pre-commit hook with `lint-staged` configured — confirmed by a repo-wide search, nothing equivalent exists in `core-be`, `calendar-be`, `admin-be`, `core-fe`, `calendar-fe`, or `admin-fe`. Linting/testing elsewhere happens at CI time only (see [CICD.md](CICD.md) for the actual per-repo workflow inventory), not as a local commit gate. Don't assume a pre-commit hook will catch a lint violation before it reaches CI unless you're specifically in `arm-service-notification`.

---

## 7. React / frontend conventions

Absolute imports via a `@/` alias (confirmed in `calendar-fe`'s `vite.config.ts` `resolve.alias`) rather than long relative paths. Two of the three frontends (`core-fe`, `calendar-fe`) share the `@aracreate/test-arm-ui` design-token UnoCSS preset; `admin-fe` uses UnoCSS's own `(secret removed)()` directly instead — a real, documented divergence, not a standard to enforce retroactively (see [MODULE-FEDERATION.md](MODULE-FEDERATION.md) §6 and [ADR-006](adr/006-unocss-mfe-scoping.md)). No router of its own in either Module Federation remote — routing lives entirely in the host shell (`core-fe`); a new remote should follow this pattern, not bring its own `BrowserRouter` into the federated path (see [MODULE-FEDERATION.md](MODULE-FEDERATION.md) §8's "Adding a new MFE remote" checklist for the full sequence).

---

## Related documents

- [TEMPLATE-CONFORMANCE.md](TEMPLATE-CONFORMANCE.md) — repo structure, file headers, naming conventions (the complementary, repo-level half of this document)
- [AUTH-ARCHITECTURE.md](AUTH-ARCHITECTURE.md) §3 — the auth guard pattern in full
- [MONITORING.md](MONITORING.md) §2 — the shared pino logging configuration in full
- [SECURITY.md](SECURITY.md) — `ValidationPipe` inconsistency and other cross-backend security posture findings
- [MODULE-FEDERATION.md](MODULE-FEDERATION.md) — frontend/MFE-specific conventions in full
- [CICD.md](CICD.md) — what actually runs lint/test, and when (CI vs. pre-commit)
