# arm-core-be — Registry Module

`src/registry/` — a service/MFE self-registration and lookup API, intended to let `core-fe`'s Module Federation host discover remote entry URLs at runtime instead of only at build time.

**Correcting prior documentation in this workspace**: [DATABASE-DESIGN.md](../../docs/arm-docs/docs/DATABASE-DESIGN.md) §2.4, [core-be/API.md](../../docs/arm-docs/docs/core-be/API.md) §4, and [troubleshooting/mfe-not-loading.md](../../docs/arm-docs/docs/troubleshooting/mfe-not-loading.md) all previously stated that `core-fe` never calls this API from any live code path, citing a `MODULE-FEDERATION.md` §2.4 that doesn't actually exist in that document. That claim is false, verified directly against source: `core-fe` **does** call the registry's read endpoint, from a live, always-mounted component. What's true instead, and more precisely stated below: the read path is wired in, but the table it reads from is never populated by anything in this workspace — so today it behaves as a no-op, not because it's disconnected, but because nobody ever writes to it. All three docs above have been corrected in place; this document is the source of truth going forward.

---

## 1. What actually exists

`RegisterModule` (`src/registry/register.module.ts` — the class is `RegisterModule`, the folder and every other file in it is `registry.*`; that naming mismatch is real, not a typo in this doc) is wired into `AppModule.imports`. It provides:

| Endpoint | Guard | Purpose |
|---|---|---|
| `POST /v1/registry/register` | `@Public()` + `BffSecretGuard` | Upsert-by-name: registers or overwrites a service/MFE's `url`/`remoteUrl`. Body: `{ name, type: 'backend'\|'mfe', url?, remoteUrl? }`. |
| `GET /v1/registry` | `KongJwtGuard` (any authenticated user) | Paginated list of every registered entry. |
| `GET /v1/registry/type/:type` | `KongJwtGuard` | Same, filtered by `type`. |
| `GET /v1/registry/:name` | `KongJwtGuard` | Single entry lookup by exact name. |

Backed by one Postgres table, `registry` (`arm_core` database — see [DATABASE-DESIGN.md](../../docs/arm-docs/docs/DATABASE-DESIGN.md) §2.4 for the full column schema): `id`, `name` (unique), `type` (`'backend'` | `'mfe'`), `url`, `remote-url`, `registered-at`, `updated-at`.

---

## 2. The read path is live and wired in — verified

`core-fe/src/lib/remote-registry.ts` exports `initRemoteRegistry()`, which calls `RegistryService.getServiceByName("calendar")` (→ `GET /bff/proxy/core/v1/registry/calendar`) and, if a `remoteUrl` comes back, calls Module Federation's own `registerRemotes()` runtime API to override the `calendar` remote's entry URL before any lazy import of it resolves.

This function is called from `core-fe/src/components/auth/common/protected-route.tsx:53`, inside a `useEffect` gated only on `userId` being known (guarded against React StrictMode's dev double-invoke via a `useRef`). `ProtectedRoute` wraps the entire authenticated shell — this runs **once per session, for every logged-in user**, not behind a flag and not dead code by any reasonable definition of the term.

**Trust boundary**: `isTrustedOrigin()` in the same file only accepts a `remoteUrl` whose *origin* matches the build-time `VITE_CALENDAR_REMOTE_URL`'s origin — deliberate, per the code's own comment, because `POST /registry/register` has no authorization beyond the shared `X-BFF-Secret` (not admin-scoped, not user-scoped), so a value read back from it can't be trusted to redirect the shell to an arbitrary domain. A registry entry can only override the *path* on the already-trusted host, not the host itself.

**Only `calendar` is handled.** There is no equivalent override for the `admin` remote anywhere in `core-fe` — `admin`'s entry URL is 100% the build-time `VITE_ADMIN_REMOTE_URL`, with no registry read path touching it at all, live or dead.

---

## 3. The write path is never called — this is why it behaves as a no-op

Grepped for every possible caller across all 8 repos: `registryService.register(...)` (the backend service method) is only ever called from `registry.controller.ts` itself. `RegistryService.registerService(...)` (the matching frontend API-client method, `core-fe/src/api/services/registry-service.ts`) is defined but never imported or called from any component, page, or startup hook — it exists as dead client code even though the module it belongs to (`remote-registry.ts`) is not. No backend self-registers itself at startup. No deploy script, seed migration, or documented ops procedure populates the table either.

**Net effect**: the `registry` table is empty in every environment this workspace can produce on its own. `GET /v1/registry/calendar` always returns nothing useful, `initRemoteRegistry()`'s `remoteUrl` check never passes, and Module Federation always falls through to its build-time default (`VITE_CALENDAR_REMOTE_URL`, per [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) §2). This is exactly why earlier documentation observed "not used" and generalized that to "dead code" — the *observed behavior* was right, the *mechanism* claim wasn't. If someone ever did `POST` a `calendar`-type entry with a same-origin `remoteUrl` (manually, with the right `X-BFF-Secret` — nothing automates this today), the shell would pick it up on the next authenticated session and actually use it.

---

## 4. Adding a live consumer, if this is ever meant to be used for real

None of the following exists today — this is what would need to happen, not a description of current behavior:

1. Something needs to call `POST /v1/registry/register` for `calendar` (and, if wanted, `admin` — which would first need its own `initRemoteRegistry`-equivalent read call added to `core-fe`, since none exists) whenever that remote's real deployed URL is known — e.g. as a deploy-pipeline step after `calendar-fe`/`admin-fe` finish building, using `BFF_INTERNAL_SECRET` as the `X-BFF-Secret`.
2. Until that's built, treat `VITE_CALENDAR_REMOTE_URL`/`VITE_ADMIN_REMOTE_URL` as the only remote-discovery mechanism that actually functions — this matches [troubleshooting/mfe-not-loading.md](../../docs/arm-docs/docs/troubleshooting/mfe-not-loading.md)'s practical triage advice ("check the env var, not the registry"), which remains correct even though its stated reasoning has been corrected.

---

## 5. `README.md`'s "placeholder for future org/role management" claim is also wrong

This repo's own `README.md` describes `src/registry/` as a "placeholder org/role management" module. Checked against `git log -- src/registry` (`feat(registry): add BffSecretGuard...`, `feat: add notification producer, registry module...`) and the entity schema itself (`name`, `type: 'backend'|'mfe'`, `url`, `remoteUrl` — nothing resembling a role or permission) — there is no trace anywhere of registry ever being org/role-related. That line in `README.md` has been corrected to match this document.

---

## Related documents

- [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) §2 / §2.4 — the build-time remote-registration config this table would override if it were ever populated
- [DATABASE-DESIGN.md](../../docs/arm-docs/docs/DATABASE-DESIGN.md) §2.4 — full `registry` table column schema
- [core-be/API.md](../../docs/arm-docs/docs/core-be/API.md) §4 — full endpoint reference with guard/DTO detail
- [troubleshooting/mfe-not-loading.md](../../docs/arm-docs/docs/troubleshooting/mfe-not-loading.md) — practical triage guidance this document's finding doesn't change
