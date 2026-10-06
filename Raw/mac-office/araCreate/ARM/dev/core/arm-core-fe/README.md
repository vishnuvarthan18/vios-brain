# ARM Core Frontend

The Module Federation host shell for the ARM platform. Every user starts here — it owns the router, navbar, global auth state, and the authenticated layout that every other frontend (calendar, admin, and any future app) mounts into as a remote.

- Loads the `calendar` and `admin` remotes at runtime via Module Federation, and exposes shared singletons (`toast`, `theme-store`) back to them
- Owns the landing page and all auth pages — email-OTP login, Google/Microsoft SSO, OAuth callback
- Owns the Settings page, including the Linked Accounts tab — the UI entry point for connecting/reconnecting/unlinking Google and Microsoft calendar accounts (see [Settings page](#settings-page) below)
- Global auth state (Zustand `auth-store`); session validated against the BFF (`GET /bff/me`) in `ProtectedRoute`
- Also ships as an Electron desktop build (`electron/`) alongside the web build

**What it does not own:** any calendar-specific or admin-specific business logic — those live in their own remotes (`calendar/*`, `admin/AdminApp`) and are only ever consumed here through their Module Federation exposed entry points, never imported from source directly.

## Stack

| Layer | Tool |
| --- | --- |
| Framework | React 19 + Vite |
| Module Federation | `@module-federation/vite` (host) |
| State | Zustand |
| Styling | UnoCSS, `@aracreate/test-arm-ui` |
| Desktop | Electron |
| Language | TypeScript |
| Package manager | pnpm |
| Unit tests | Vitest (colocated `*.test.ts`/`.tsx`) |
| E2E tests | Playwright (`e2e/`) |

## Architecture

```text
Browser (localhost:3000)
  → arm-core-fe (host: router, navbar, auth state)
      → loads calendar/* remotes  (arm-app-calendar, :3001)
      → loads admin/AdminApp      (arm-admin frontend, :10000)
      → all API calls go through the BFF proxy (/bff/proxy/*)
```

- `src/routes/` — `BrowserRouter` (web) or `HashRouter` (Electron), top-level route table, lazy-loaded MF remote imports
- `src/store/auth-store.ts` — Zustand auth state; `src/hooks/use-auth-sync.ts` — cross-tab logout/revalidation
- `src/api/clients/auth-client.ts` — two axios instances (`NoAuthClient` for the anonymous health check, `AuthClient` for everything else via the BFF, `withCredentials: true`)
- `src/shared/toast.ts`, `src/store/theme-store.ts` — exposed to remotes via Module Federation

Tokens never reach this app — the BFF owns the session; see the workspace-level [CLAUDE.md](../../CLAUDE.md) for the full BFF architecture, and this repo's own [CLAUDE.md](./CLAUDE.md) for the full route map, Module Federation wiring, and auth state details.

## Module Federation

This app is the **host**. The full cross-platform picture (shared-dependency negotiation, UnoCSS cross-MFE scanning, `.calendar-scope` CSS isolation, verified config drift) is in [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) — this section is the host-specific how-to.

### Adding or updating a remote

Remotes are registered in `vite.config.ts`'s `federation()` plugin call:

```ts
federation({
  name: "core",
  filename: "remoteEntry.js",
  exposes: {
    "./toast": "./src/shared/toast.ts",
    "./theme-store": "./src/store/theme-store.ts",
  },
  remotes: {
    calendar: {
      type: "esm",
      name: "calendar",
      entry: env.VITE_CALENDAR_REMOTE_URL || "http://localhost:3001/remoteEntry.js",
      entryGlobalName: "calendar",
      shareScope: "default",
    },
    admin: { /* same shape, VITE_ADMIN_REMOTE_URL, default :10000 */ },
  },
  shared: { /* singletons — see below */ },
})
```

To add a new remote:

1. Add an entry to `remotes` above — pick a `name`, an `entry` URL driven by a new `VITE_<NAME>_REMOTE_URL` env var (default to that remote's local dev port).
2. Add lazy-loaded routes in `src/routes/app.router.tsx` that `import('remoteName/Component')`, following the pattern already used for `calendar/*` and `admin/AdminApp`.
3. Add the new remote's default URL to [ENV-VARS.md](../../docs/arm-docs/docs/ENV-VARS.md) and the root [CLAUDE.md](../../CLAUDE.md) port map.
4. If the new remote shares a dependency with this host (React, Zustand, etc.), add it to `shared` here **and** to the remote's own `vite.config.ts` `shared` block — a dependency declared on only one side loads as a separate, non-singleton instance.

### Shared singletons

`react`, `react-dom`, `react-router-dom`, `react-toastify`, `zustand`, `@aracreate/test-arm-ui` — all `singleton: true`. **Verified drift:** this host currently pins `react`/`react-dom` to `^19.2.0` while both remotes pin `^19.1.1` — the host's version wins at runtime regardless of what a remote's own `package.json` says, since `singleton: true` means "exactly one instance across the federated app." Not a bug in practice (source-compatible minor versions), but worth knowing before assuming a remote's locally-installed React version is what actually runs behind this shell. Full detail in [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) §4.

### UnoCSS cross-MFE scanning

This shell's UnoCSS instance needs to know about calendar-fe's source files too, so utility classes used inside `calendar/*` components (which this repo doesn't otherwise know about at build time) still generate the right CSS once the remote is loaded in. Configured via `content.pipeline.include` in **two places that currently disagree**:

- `vite.config.ts`'s `UnoCSS()` plugin call — `"../calendar/src/**/*.{ts,tsx}"`
- `uno.config.ts` (standalone config) — `"../../apps/arm-app-calendar/src/frontend/src/**/*.{ts,tsx}"`

**Verified: `vite.config.ts`'s path is stale and resolves to a directory that doesn't exist** (`core/calendar/src` — the calendar frontend actually lives at `apps/arm-app-calendar/src/frontend/src`). Only `uno.config.ts`'s path is correct. This hasn't visibly broken styling because Vite's own build pipeline still processes the federated calendar chunks, but if a calendar-only utility class ever fails to generate in a production shell build, check this path first before assuming UnoCSS itself is broken. Full detail and fix direction in [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) §6.

### CSS isolation rule

When `CalendarSyncViewPage` is rendered inside the shell's `HomeLayout` (route `/calendar/sync/view/:targetCalendarId`), it's wrapped in a `.calendar-scope` div in `app.router.tsx`. FullCalendar's theme-variable overrides key off this class — without the wrapper, calendar-fe falls back to FullCalendar's unstyled default theme. See [MODULE-FEDERATION.md](../../docs/arm-docs/docs/MODULE-FEDERATION.md) §5 for the verified nuance that these overrides currently only load in calendar-fe's standalone dev entry point, not the federated path this shell actually uses.

### Electron support

`BUILD_TARGET=electron` switches Vite's `base` to `./` and uses `HashRouter` instead of `BrowserRouter`. The Electron build lives in `electron/`. This affects all route paths — if you add routes, they must work with hash routing too.

## Settings page

`/settings` (`SettingsPage` → `Settings` → `SettingsForm`, `src/components/home/form/settings-form.tsx`) is a two-tab page built on `@aracreate/test-arm-ui`'s `Tabs`:

| Tab | Component | What it does |
| --- | --- | --- |
| **Account Settings** (default) | `AccountProfileForm` | Edit `firstName`/`lastName` (email shown, not editable here) — `UserService.updateUser()`, updates both local form state and `auth-store.currentUser` on success |
| **Linked Accounts** | `LinkedAccountsTab` | Lists connected Google/Microsoft calendar accounts, with Add/Reconnect/Unlink actions |

**The Linked Accounts tab is lazily mounted, not just hidden** — `{activeTab === "linked" && <LinkedAccountsTab />}` — so `useLinkedAccounts()`'s account fetch only fires once the user actually opens that tab, not on every Settings page load.

Linked Accounts is the browser-side entry point for the OAuth flows documented in [`apps/arm-app-calendar/GOOGLE-OAUTH.md`](../../docs/arm-docs/docs/calendar/GOOGLE-OAUTH.md) and the unlink cascade in [`apps/arm-app-calendar/DELETE-CASCADE.md`](../../docs/arm-docs/docs/calendar/DELETE-CASCADE.md):

- **Add Account** — a dropdown (`Connect Google` / `Connect Microsoft`) opens `BFF_ROUTES.CALENDAR_ADD_ACCOUNT` or `CALENDAR_ADD_ACCOUNT_MICROSOFT` in a popup via `useOAuthPopup`.
- **Reconnect** — calls `SettingsService.getReconnectUrl(email)` to get the reconnect URL, then opens it in the same popup mechanism.
- **Unlink** — routes through `UnlinkConfirmDialog` (explicit confirmation) before calling the unlink action; this is what triggers the ordered delete cascade on the calendar-be side.

## API Clients

Two axios instances in `src/api/clients/auth-client.ts` — `NoAuthClient` (core-be direct, anonymous health check only) and `AuthClient` (everything else, via the BFF, session cookie sent automatically). Full behavior including the 401-retry-then-redirect interceptor is in [CLAUDE.md](./CLAUDE.md#api-clients).

## Auth State

`auth-store.ts` (Zustand) — `userId`, `isAuthenticated`, `currentUser`, `isAdmin`. Session validation happens in `ProtectedRoute`, not the `use-auth` hook. Cross-tab logout/revalidation via `useAuthSync` in `HomeLayout`. Full detail in [CLAUDE.md](./CLAUDE.md#auth-state) and the platform-wide flow in [AUTH-ARCHITECTURE.md](../../docs/arm-docs/docs/AUTH-ARCHITECTURE.md).

## Commands

```sh
make install    # install dependencies
make setup      # create .env from .env.example if missing
make dev        # run locally with hot reload (vite)
make build      # type-check + build to dist/
make test       # unit tests (vitest)
make test-e2e   # e2e tests (playwright)
make lint       # eslint (check only, used in CI)
make lint-fix   # eslint --fix
make release    # cut a semantic release (not yet wired — see TASK_SHEET.md)
make clean      # remove build artefacts
```

`make help` (default) prints the full target list.

**Not currently wrapped by a `make` target — run directly with `pnpm`:**

```sh
pnpm preview       # serve the production build (dist/) on :3000, after `pnpm build`
pnpm build:run     # build then preview in one step
pnpm electron:dev  # run the Electron desktop shell against the Vite dev server
pnpm electron:build # build the Electron desktop package
```

In the full ARM workspace, this app normally runs via `make local-dev` from `deploy/arm-deploy-make/` — see the workspace [CLAUDE.md](../../CLAUDE.md) for the standard startup flow. This repo's own `Makefile` is a standalone entry point for working in this repo directly.

## Conventions

Repo conventions (file headers, naming, versioning, Makefile targets) follow [aracreate-template-codebase](../../aracreate-template-codebase). Template-conformance work for this repo is tracked in `TASK_SHEET.md`.

## License

Proprietary — see [LICENSE](./LICENSE), Copyright (C) 2026, araCreate Group.
