# ARM Admin — Frontend

The admin Module Federation remote for the ARM platform. React 19 + Vite + TanStack Query + UnoCSS. Loaded by `core/arm-core-fe` (the shell) at runtime via the MF plugin — it is not a standalone app in production, but runs standalone on port `10000` for development. Provides the admin portal UI: user management (list, edit, disable/enable, delete, grant/revoke admin) and an audit log viewer.

## Stack

| Layer | Tool |
| --- | --- |
| Framework | React 19 + Vite |
| Module Federation | `@module-federation/vite` |
| Data fetching | TanStack Query |
| HTTP client | axios |
| Styling | UnoCSS (Wind preset — Tailwind-compatible utility classes) |
| Language | TypeScript |
| Package manager | pnpm |

## Architecture

```text
src/
├── AdminApp.tsx                  ← MF entry point — exposed as ./AdminApp
│                                    QueryClientProvider + AdminAuthProvider + AdminRouter
├── main.tsx                      ← Standalone dev only — wraps AdminApp in BrowserRouter
├── api/
│   ├── client.ts                 ← adminClient: axios instance → /bff/proxy/admin/v1
│   ├── users.ts                  ← list, getById, update, disable, enable, delete, grantAdmin, revokeAdmin
│   └── audit.ts                  ← list
├── components/
│   ├── admin-auth-provider.tsx   ← Calls GET /admin/me on mount; renders AccessDenied on 403
│   └── access-denied.tsx         ← Shown when the user is authenticated but not an admin
├── layouts/
│   └── admin-layout.tsx          ← Header + Tabs nav (Users / Audit) + <Outlet />
├── pages/
│   ├── users/
│   │   ├── users-page.tsx        ← Paginated list, search, disable/enable/delete
│   │   └── user-detail-page.tsx  ← Edit form (firstName, lastName, email, picture)
│   └── audit/
│       └── audit-page.tsx        ← Paginated audit log, action-type filter
├── routes/
│   └── index.tsx                 ← <Routes> only — no Router wrapper (lives in the host's BrowserRouter)
└── lib/
    ├── constants.ts               ← BFF_BASE_URL, ADMIN_API_BASE
    └── toast.ts                   ← Lazily imports core/toast (MF); no-ops in standalone dev
```

### Module Federation

This is a **remote**, loaded by `core/arm-core-fe` (the host):

- **Exposed as:** `./AdminApp` → `src/AdminApp.tsx`
- **Entry file:** `remoteEntry.js`, served on port `10000` in local dev
- **Federation name:** `admin` (`vite.config.ts`, via `@module-federation/vite`)
- **Mounted at:** `/admin/*` in `core/arm-core-fe/src/routes/app.router.tsx`
- **Production build:** `base: '/admin/'`; CSS is emitted as a single `admin-assets/admin-styles.css` file (`cssCodeSplit: false`) so the host doesn't have to worry about chunked admin styles

**No Router inside this remote.** `AdminApp` renders `<Routes>` directly — no `BrowserRouter`, `HashRouter`, or `MemoryRouter` — because it lives inside the host's own `BrowserRouter`. Route paths in `src/routes/index.tsx` are relative, not absolute: the host mounts admin at `<Route path="/admin/*">`, which strips `/admin/` from the context before these nested `<Routes>` run. The outer layout route is pathless (`<Route element={<AdminLayout />}>`); children are `users`, `users/:id`, `audit`. A `<Navigate to="/admin/users">` still uses the full absolute path, because navigation targets the browser URL, not a route segment. For **standalone dev**, `main.tsx` wraps `AdminApp` in its own `BrowserRouter` so `pnpm dev` works without the host present.

**Shared singletons:** `react`, `react-dom`, and `react-router-dom` are declared `singleton: true` (pinned to this package's own `dependencies` versions) in `vite.config.ts` — they must not be duplicated with the host. Adding a new shared dependency that the host also uses means adding it to `shared` in both `vite.config.ts` files, not just this one.

**Standalone-dev stubbing:** a local `stubMfRemotesPlugin()` in `vite.config.ts` intercepts imports of MF-only modules (currently `core/toast`) during standalone dev and swaps in a no-op stub, so `pnpm dev` doesn't fail trying to resolve a remote the host would normally provide. If you add a second host-provided import, add its stub here too.

## API Client and Auth

`adminClient` (`src/api/client.ts`) is an axios instance:

- `baseURL`: `ADMIN_API_BASE` = `${VITE_BFF_URL}/bff/proxy/admin/v1`
- `withCredentials: true` — sends the `SESSION_ID` httpOnly cookie automatically

Every call goes through the BFF proxy; the BFF forwards to the admin backend (port `10001`) with the user's JWT bearer token injected. Nothing here calls the admin backend directly.

**The client has a global response interceptor with two hard redirects**, not just error propagation:

- `401` → `window.location.href = '/auth/login'` (session is gone; bounce to login)
- `403` **except on the `/admin/me` call itself** → `window.location.href = '/home'` (not an admin; bounce out of the admin portal entirely)

The `/admin/me` exclusion exists because that specific call's `403` is handled in place by `AdminAuthProvider` (renders `<AccessDenied />` inline) rather than hard-navigating away — if the interceptor redirected on that one too, the in-place access-denied UI would never get a chance to render.

### Two layers of admin-access enforcement

1. **Primary:** `core/arm-core-fe`'s sidebar hides the Admin nav entry entirely when `isAdmin === false` in the shell's auth store.
2. **Secondary (this remote):** `AdminAuthProvider` calls `GET /admin/me` once on mount, before rendering any admin page:
   - `200` → admin confirmed, renders `AdminContext.Provider` around the app (the resolved `AdminMe` is available anywhere via `useAdminContext()`)
   - `403` → authenticated but not an admin → renders `<AccessDenied />`
   - Any other error (network failure, `5xx`) → **fails open**, renders children anyway — an infra blip shouldn't lock a legitimate admin out of the UI

The secondary guard is what actually stops a non-admin who navigates to `/admin/*` directly by URL, since the sidebar hiding only stops discovery through the UI.

## Route Map

| Path | Component | Notes |
| --- | --- | --- |
| `/admin` | redirect | → `/admin/users` |
| `/admin/users` | `UsersPage` | Paginated list + search |
| `/admin/users/:id` | `UserDetailPage` | Edit form + metadata |
| `/admin/audit` | `AuditPage` | Read-only log + filter |

## Data Fetching

TanStack Query, with these query keys:

- `['users', { search, page }]` — user list
- `['users', id]` — single user detail
- `['audit', { action, page }]` — audit log

Mutations (disable, enable, delete, update) all call `queryClient.invalidateQueries({ queryKey: ['users'] })` on success to keep the list fresh. `grantAdmin`/`revokeAdmin` invalidate only `['users', id]` — the detail page they run from.

## Environment Variables

| Variable | Default | Notes |
| --- | --- | --- |
| `VITE_PORT` | `10000` | Dev server port (also `strictPort: true` — fails rather than silently picking another port) |
| `VITE_BFF_URL` | `http://localhost:5001` | BFF base URL — must be the **host** port, not a container port |

## Commands

```sh
pnpm install
pnpm dev        # standalone dev server on :10000
pnpm build      # production build, emitted under base: '/admin/'
pnpm lint
pnpm preview    # preview the production build locally
```

Standalone (`pnpm dev`) is useful for iterating on admin pages in isolation, but the `AdminApp` you actually ship only gets exercised end-to-end when the shell (`core/arm-core-fe`) loads it as a remote — verify changes there too before considering a change done.

## What NOT to Do

- Don't add a `BrowserRouter`, `HashRouter`, or `MemoryRouter` inside this remote — the host provides the router context, and a nested router would break navigation.
- Don't call the admin backend directly. All calls go through `adminClient` → the BFF proxy.
- Don't import from `core/arm-core-fe` source files directly. Only consume MF-exposed modules (e.g. `core/toast`) if needed, and add a dev-mode stub for any new one.
- Don't skip `AdminAuthProvider` on a new top-level page — it's the secondary defense against direct-URL access by a non-admin.
- Don't add a separate BFF for admin. `arm-bff` is the single BFF for the entire platform.

For AI-agent-specific conventions and constraints (kept in sync with this guide), see [CLAUDE.md](./CLAUDE.md).
