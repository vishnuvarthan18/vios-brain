# ARM Platform — Module Federation Architecture

> How the shell (`core/arm-core-fe`) loads the `calendar` and `admin` remotes at runtime: exposes/consumes wiring, shared-dependency negotiation, UnoCSS cross-MFE scanning, and how CSS is kept from leaking between apps.
>
> [ARCHITECTURE.md](ARCHITECTURE.md) §6 has the one-table summary. This document is the deep dive — annotated config, the reverse host→remote consumption pattern, and known drift/pitfalls verified against source.
>
> Verified against `core/arm-core-fe/vite.config.ts`, `core/arm-core-fe/uno.config.ts`, `core/arm-core-fe/src/routes/app.router.tsx`, `apps/arm-app-calendar/src/frontend/vite.config.ts`, `apps/arm-app-calendar/src/frontend/uno.config.ts`, `apps/arm-app-calendar/src/frontend/App.tsx`, `apps/arm-admin/src/frontend/vite.config.ts`, `apps/arm-admin/src/frontend/uno.config.ts`, and the three repos' `package.json` files (re-verified 2026-08-27).

---

## 1. Topology

| Role | Federation name | Local port | Exposes | Consumes from `core` |
|---|---|---|---|---|
| **Host** | `core` | `3000` | `./toast`, `./theme-store` | — |
| **Remote** | `calendar` | `3001` | `./Page-1`, `./SyncDetail`, `./SyncView`, `./MergedView`, `./AuthCallback` | `core/theme-store` |
| **Remote** | `admin` | `10000` | `./AdminApp` | `core/toast` |

`core/arm-core-fe` is the only host — it owns the router (`BrowserRouter`/`HashRouter`), navbar, and global auth state. Both remotes are pure UI slices with no router of their own; they render inside routes the shell defines (`/calendar/*`, `/admin/*`).

All three apps build with `@module-federation/vite` (not the older `@originjs/vite-plugin-federation` — the plugin name shows up correctly in every `vite.config.ts`, but `apps/arm-admin/src/frontend/CLAUDE.md`'s wording could be read as implying otherwise; confirmed as `@module-federation/vite` in that repo's own `package.json` and `vite.config.ts`).

---

## 2. Host → remote registration (`core-fe/vite.config.ts`)

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
      entry: env.VITE_CALENDAR_REMOTE_URL || "http://localhost:3001/calendar/remoteEntry.js",
      entryGlobalName: "calendar",
      shareScope: "default",
    },
    admin: {
      type: "esm",
      name: "admin",
      entry: env.VITE_ADMIN_REMOTE_URL || "http://localhost:10000/admin/remoteEntry.js",
      entryGlobalName: "admin",
      shareScope: "default",
    },
  },
  shared: { /* see §4 */ },
})
```

The shell lazy-imports both remotes in `src/routes/app.router.tsx`:

```ts
import('calendar/Page-1')        → CalendarAccountPage   // /calendar/syncs
import('calendar/SyncDetail')    → CalendarSyncPage      // /calendar/syncs/new, /calendar/syncs/:id
import('calendar/MergedView')    → CalendarMergedPage    // /calendar/view
import('calendar/SyncView')      → CalendarSyncViewPage  // /calendar/view/:targetCalendarId
import('calendar/AuthCallback')  → CalendarAuthCallback  // /calendar/account, /calendar/auth/callback
import('admin/AdminApp')         → AdminApp              // /admin/*
```

Each is wrapped in `timedLazy()` (`core-fe/src/utils/load-remote-module.ts`) rather than React's bare `lazy()` — it races the dynamic `import()` against a 15s timeout so a remote whose dev server is down fails visibly instead of hanging the route forever. The `/calendar/account` and `/calendar/auth/callback` routes sit **outside** the protected layout so the OAuth popup's redirect lands before the session guard runs.

`/calendar` itself is a redirect to `/calendar/syncs`. The `/calendar/syncs*` and `/calendar/view*` paths are the canonical design-system ones; older `/calendar/sync*` paths persist in call sites inside the calendar repo, which is why the shell still carries both.

Remote entry URLs are env-driven (`VITE_CALENDAR_REMOTE_URL`, `VITE_ADMIN_REMOTE_URL`), defaulting to the local-dev ports above. Note the path segment: each remote sets `base` (`/calendar/`, `/admin/`), so its `remoteEntry.js` is served *under* that base, not at the server root.

### 2.4 A runtime override exists for `calendar` — live, but currently a no-op

**Correcting prior versions of this document and its dependents** ([DATABASE-DESIGN.md](DATABASE-DESIGN.md) §2.4, [core-be/API.md](core-be/API.md) §4, [troubleshooting/mfe-not-loading.md](troubleshooting/mfe-not-loading.md)), which asserted core-fe never calls `core-be`'s registry API and cited this section number for a claim it never actually contained. Verified directly against source instead: `core-fe/src/lib/remote-registry.ts`'s `initRemoteRegistry()` calls `GET /v1/registry/calendar` and, if a same-origin `remoteUrl` comes back, calls this same `registerRemotes()` runtime API to override the config above before any `calendar/*` lazy import resolves. It's invoked from `ProtectedRoute` on every authenticated session, not dead code.

The reason it has no observable effect today: nothing anywhere in this workspace ever calls `POST /v1/registry/register`, so the backing `registry` table is always empty and the read always comes back empty — the build-time `entry` above is what actually resolves in every environment that exists. Full detail, including the trust-origin check and why `admin` has no equivalent override, is in [`core/arm-core-be/REGISTRY.md`](../../../core/arm-core-be/REGISTRY.md).

---

## 3. Remote exposes (`calendar-fe` / `admin-fe` `vite.config.ts`)

**calendar-fe** (`apps/arm-app-calendar/src/frontend/vite.config.ts`) — federation name `calendar`, `remotes: {}` (declares no static remotes of its own — see §3.1 for how it still reaches `core/theme-store`):

```ts
exposes: {
  "./Page-1": "./src/features/calendar/components/calendar-account-page",
  "./SyncDetail": "./src/features/calendar/components/sync-detail-page",
  "./SyncView": "./src/features/calendar/components/sync-view-page",
  "./MergedView": "./src/features/calendar/components/calendar-merged-page",
  "./AuthCallback": "./src/features/calendar/pages/auth-callback",
}
```

**admin-fe** (`apps/arm-admin/src/frontend/vite.config.ts`) — federation name `admin`, also no `remotes` key:

```ts
exposes: {
  "./AdminApp": "./src/AdminApp.tsx",
}
```

Each build inlines its CSS into a single named file via `cssCodeSplit: false` + a custom `assetFileNames` — `calendar-assets/calendar-styles.css` for calendar-fe, `admin-assets/admin-styles.css` for admin-fe. Both remotes set a matching `base` (`/calendar/`, `/admin/`) so their built asset paths resolve correctly when served behind the shell.

### 3.1 Reverse consumption: remotes importing from the host without declaring it

Neither remote lists `core` in its own `remotes` config, yet both dynamically `import()` a `core/*` module at runtime:

- `calendar-fe`'s `use-theme.ts:34` — `import("core/theme-store")`
- `admin-fe`'s `lib/toast.ts` — `import("core/toast")`

This works because `@module-federation/vite`'s runtime federation resolves `core/*` module IDs against whatever host has already registered itself as `core` in the browser's shared federation runtime — which is guaranteed to have happened, since these remotes are only ever mounted *inside* the shell that just loaded them. No static `remotes.core` entry is needed for this to resolve.

Both call sites treat this as best-effort:
- Both `vite.config.ts` files mark the module `external` (`build.rollupOptions.external`) and `optimizeDeps.exclude`, so Vite doesn't try to bundle or pre-resolve it.
- Both wrap the `import()` in a `.catch()` (calendar) or rely on a dev-only stub plugin (admin) that no-ops the module when running standalone, so `pnpm dev` on port 3001/10000 works without the shell present.
- A local `stubMfRemotesPlugin()` in each `vite.config.ts` substitutes a stub implementation (`useThemeStore = undefined` for calendar, a no-op `toast` for admin) for standalone dev builds only — production/federated builds never use the stub.

**Practical implication:** if you rename or remove `./toast` or `./theme-store` from `core-fe`'s `exposes`, both remotes fail this import silently (falls back to their standalone-mode behavior) rather than throwing — a genuine cross-repo contract with no compile-time check.

---

## 4. Shared dependency negotiation

All three `vite.config.ts` files declare `shared` singletons, each pulled from that app's own `package.json` `dependencies` via `requiredVersion: dependencies.react` etc.:

| Package | core-fe | calendar-fe | admin-fe |
|---|---|---|---|
| `react` | `singleton, ^19.2.0` | `singleton, ^19.1.1` | `singleton, ^19.1.1` |
| `react-dom` | `singleton, ^19.2.0` | `singleton, ^19.1.1` | `singleton, ^19.1.1` |
| `react-router-dom` | `singleton, ^7.18.2` | `singleton, ^7.18.2` | `singleton, ^7.18.2` |
| `@radix-ui/themes` | `singleton, ^3.3.0` | `singleton, ^3.3.0` | `singleton, ^3.3.0` |
| `@aracreate/test-arm-ui` | `singleton, 3.1.4` | `singleton, 3.1.5` | `singleton, 3.1.4` |
| `react-toastify` | `singleton` (no version pin) | `singleton` (no version pin) | not shared |
| `zustand` | `singleton, ^5.0.12` | `singleton` (no version pin) | not shared |

**Version drift, verified:** two pins disagree across apps.

- `react`/`react-dom` — `core-fe` pins `^19.2.0`, both remotes pin `^19.1.1`.
- `@aracreate/test-arm-ui` — `calendar-fe` pins the exact version `3.1.5`, while `core-fe` and `admin-fe` both pin `3.1.4`. These are **exact** pins, not ranges, so the two `requiredVersion` values are mutually unsatisfiable by design.

Because `singleton: true` tells the runtime "there can only be one instance," and the host loads first, the shell's version wins the negotiation for every app in the session — `19.2.0` for React, `3.1.4` for the UI library. The remotes' `requiredVersion` becomes advisory, not enforced. Neither has caused a visible bug, but it means **a remote's own `package.json` version is not what runs behind the shell.** For `@aracreate/test-arm-ui` in particular this is a live footgun: a calendar component written against a `3.1.5`-only export will typecheck and pass standalone dev on `:3001`, then fail at runtime once federated into a shell serving `3.1.4`. Bump the host first, or keep the three pins equal.

**Not shared:** `react-toastify` and `zustand` are absent from `admin-fe`'s `shared` block entirely — admin doesn't use either package, so this is intentional, not a gap. Admin *does* share `@aracreate/test-arm-ui` and `@radix-ui/themes` — it consumes the library's components even though it builds its utility classes with `presetWind4()` rather than the library's UnoCSS preset (see §6).

**Rule for adding a new shared dependency:** if a new package needs to be a true singleton across host and remote (e.g. a new state library both use), add it to `shared` in every `vite.config.ts` that consumes it — a dependency shared by the host but not declared in a remote's own `shared` block loads as a separate, non-singleton instance in that remote, defeating the point.

---

## 5. CSS isolation strategy

### 5.1 The `.calendar-scope` wrapper is gone — do not reintroduce it

Older revisions of this document, and of the root `CLAUDE.md`, required the shell to wrap the calendar remote's `SyncView` in a `<div className="calendar-scope">`. That rule existed for exactly one reason: FullCalendar injected reset CSS at global scope on import, and without a wrapper it silently broke the shell's layout.

**FullCalendar has been removed from the calendar app.** The grid is now the design system's `BigCalendarView` (built on react-big-calendar), which ships scoped styles. `fullcalendar` survives only as a stale entry in `apps/arm-app-calendar/src/frontend/package-lock.json` — nothing under `src/` imports it. With the cause gone, the wrapper was removed from both rendering paths:

- `core-fe/src/routes/app.router.tsx` — the federated path; routes now mount the exposed components directly inside `RemoteErrorBoundary`, with no scope div.
- `calendar-fe/src/App.tsx` — the standalone-only path; its routes sit in a plain `<div className="h-full w-full">`.

Nothing in either repo references `.calendar-scope` any more. Re-adding it would be cargo-cult: it would scope styles against a library that is no longer there.

### 5.2 `calendar-styles.css` is standalone-only — verified, and by design

`calendar-styles.css` (`apps/arm-app-calendar/src/frontend/src/styles/calendar-styles.css`) is imported **only** in `App.tsx`. The MF-exposed components import `virtual:uno.css` but not `calendar-styles.css`. Since `App.tsx` is never part of the federation path — the shell imports the individual page components directly — **`calendar-styles.css` never loads when calendar-fe runs embedded in the shell.**

This is intentional per the file's own header comment, not an oversight:

```css
/*
 * When running inside core via module federation, ALL CSS variables
 * (--bg, --text, --primary, etc.) are already defined on :root / .dark / .light
 * by core's index.css. Do NOT redefine them here — that would override core's
 * theme and break dark/light switching.
 *
 * These rules only apply when running standalone (port 3001) where core's
 * index.css is not loaded.
 */
```

Every rule left in the file is now a standalone-mode fallback for variables the host would otherwise supply. The FullCalendar `--fc-*` theme overrides that this section used to flag as a real gap ("brand theming never applies when federated") **are gone along with FullCalendar** — that gap is closed by deletion, not by a fix.

### 5.3 The calendar remote's other three stylesheets

`calendar-styles.css` is one of four files under `apps/arm-app-calendar/src/frontend/src/styles/`, and only it is standalone-only:

| File | Loaded via | Applies when federated? |
|---|---|---|
| `index.css` | `main.tsx` (`@/styles/index.css`) | Standalone only — `main.tsx` is the standalone entry point. Carries the Poppins `@font-face` declarations. |
| `calendar-styles.css` | `App.tsx` | No — see §5.2 |
| `calendar-ds.css` | imported by the components that need it | Yes |
| `theme-overrides.css` | imported by the components that need it | Yes |

Two of these are deliberate duplication worth knowing about before you "clean up" either:

- **`theme-overrides.css` is a verbatim copy of `core-fe/src/styles/theme-overrides.css`.** The design-system components reference `var(--amber-*)` directly and the library ships only a subset of those tokens, so the full brand scale lives in each app. When federated the host has already declared them (identical values, so the second declaration is a no-op); this copy is what makes standalone `:3001` render in brand colours instead of stock Radix. **If the two files diverge, the shell and the calendar app render two different yellows.**
- **`calendar-ds.css` supplies classes the shared library's own components render but don't ship styles for** — `ConnectProviderList` emits `provider-choice*`, `AddAccountMenu` (`variant="tile"`) emits `connect-account-tile`. Without it those components render unstyled (provider rows collapse to a vertical stack). Its own header notes the proper fix is upstream, in the library's `tokens.css`.

### 5.4 `virtualExposes` vs. `App.tsx` entry point

There is no `virtualExposes` config in any of the three `vite.config.ts` files — `@module-federation/vite` exposes are declared directly via the `exposes` object (§3), each pointing at a specific component file, not a virtual re-export module. The practical distinction that matters is the one in §5.2: **`App.tsx`/`main.tsx` (imported nowhere by federation) vs. the individual page components (what `exposes` actually points at)** — anything imported only by those two entry points is standalone-dev-only and invisible to the shell.

---

## 6. UnoCSS cross-MFE scanning

Each app runs its own UnoCSS instance (the `UnoCSS()` Vite plugin), scoped to a `content.pipeline.include` glob list.

**core-fe** cross-scans calendar source so that Tailwind/UnoCSS utility classes used inside calendar components (which core doesn't otherwise know about at build time) still generate the right CSS when calendar is loaded into the shell. This is configured in **two places that disagree**:

| File | `include` path |
|---|---|
| `core-fe/vite.config.ts` (UnoCSS Vite plugin) | `"../calendar/src/**/*.{ts,tsx}"` |
| `core-fe/uno.config.ts` (standalone UnoCSS config) | `"../../apps/arm-app-calendar/src/frontend/src/**/*.{ts,tsx}"` |

**Verified drift:** `core/arm-core-fe/vite.config.ts`'s path resolves (relative to `core/arm-core-fe/`) to `core/calendar/src/**` — **that directory does not exist** in this workspace (confirmed: `core/calendar` is not present; the calendar frontend lives at `apps/arm-app-calendar/src/frontend/src`). Only `uno.config.ts`'s path is correct. Because `unocss/vite`'s `content.pipeline.include` is additive to whatever UnoCSS's own file-watcher/scanner otherwise discovers (and Vite still transitively processes calendar's federated chunks through its own build pipeline once bundled), this stale glob has not visibly broken styling — but it means the *explicit* cross-MFE scan config in `vite.config.ts` is currently a no-op. Anyone debugging "a calendar utility class isn't generating in the shell build" should fix this path first (`"../../apps/arm-app-calendar/src/frontend/src/**/*.{ts,tsx}"`, matching `uno.config.ts`) before assuming UnoCSS itself is broken.

**calendar-fe** and **admin-fe** each scan only their own `src/**/*.{ts,tsx}` — since they ship their own bundled CSS file (§3) rather than relying on the shell to generate their utility classes, there's no cross-scan need in either direction.

**Preset divergence:** `core-fe` and `calendar-fe` both use `acPreset()` from `@aracreate/test-arm-ui/preset` — the shared design-token preset. `admin-fe` uses `(secret removed)()` (UnoCSS's own (secret removed) preset) directly, with no reference to the shared preset. This means admin's utility classes are not guaranteed to resolve to the same design tokens (colors, spacing) as core/calendar — worth knowing if a visual mismatch between `/admin/*` and the rest of the shell ever comes up. This is presumably deliberate (`apps/arm-admin/src/frontend/CLAUDE.md` doesn't flag it as a gap), not a documented gap, but no explanation for the choice exists anywhere in either app's `CLAUDE.md`.

---

## 7. Local dev workflow

| Service | Command (from its own repo) | Port |
|---|---|---|
| core-fe (host) | `pnpm dev` | `3000` |
| calendar-fe (remote) | `pnpm dev` | `3001` |
| admin-fe (remote) | `pnpm dev` | `10000` |

Via the platform Makefile, `make arm-run` / `make local-dev` starts all three (plus every backend) together in Docker Compose — see root `CLAUDE.md`'s port map and `docs/guides/local-dev-setup.md` for the full sequence. Running a single frontend with plain `pnpm dev` outside Compose works too: both remotes support **standalone mode** (see §3.1 and §5.2 — theme/toast calls no-op via the stub plugin, `App.tsx`/`main.tsx`'s own routing and styles take over) so you can iterate on `calendar-fe` or `admin-fe` alone at `localhost:3001` / `localhost:10000` without the shell running. To see a remote change reflected inside the shell, the shell must be pointed at that remote's dev server — the default `VITE_CALENDAR_REMOTE_URL` / `VITE_ADMIN_REMOTE_URL` values already target `localhost:3001` / `localhost:10000`, so running both `core-fe` and the remote's `pnpm dev` locally is sufficient; no env override needed unless the remote runs on a non-default port.

---

## 8. Adding a new MFE remote — checklist

1. **Scaffold the remote app** with its own `vite.config.ts`, using `@module-federation/vite`'s `federation()` plugin. Pick a unique `name` (this becomes the module-id prefix hosts import from, e.g. `myapp/SomeComponent`).
2. **Declare `exposes`** for every component the shell will lazy-load. Point each at a specific file — not a barrel/index re-export, matching the existing three remotes' pattern.
3. **Declare `shared`** for every dependency this remote has in common with the host (React, React Router, and anything else marked `singleton: true` in `core-fe/vite.config.ts` — see §4). Pull `requiredVersion` from the remote's own `package.json` `dependencies`, matching the existing pattern — but see §4's version-drift note: the host's version wins regardless of what you pin here.
4. **Register the remote in `core-fe/vite.config.ts`**: add an entry to `federation({ remotes: { ... } })` with `type: "esm"`, `name`, an `entry` URL driven by a new `VITE_<NAME>_REMOTE_URL` env var (default to the new remote's local dev port), and `shareScope: "default"`.
5. **Add routes in `core-fe/src/routes/app.router.tsx`** that `import()` the exposed components and mount them, following the existing `lazy(() => import('remoteName/Component'))` pattern used for `calendar` and `admin`.
6. **If the remote needs host-provided state or utilities** (theme, toast, etc.), follow §3.1's reverse-consumption pattern: dynamically `import("core/whatever")`, mark it `external` in `build.rollupOptions.external` and `optimizeDeps.exclude`, and add a dev-only stub via a local `stubMfRemotesPlugin()` so standalone `pnpm dev` still works without the shell.
7. **Decide on CSS strategy up front** — either ship a single bundled CSS file the way calendar-fe/admin-fe do (`cssCodeSplit: false` + custom `assetFileNames`, with any host-supplied-variable fallbacks imported only by a standalone-only entry point — see §5.2) or scan the new remote's source from `core-fe/uno.config.ts` the way calendar is (cross-scanned) if the shell needs to generate the remote's utility classes itself. Don't do both by accident — re-read §6's drift note before copying `core-fe/vite.config.ts`'s (currently broken) cross-scan path.
8. **Add the new remote's URL env var** to `ENV-VARS.md` and the root `CLAUDE.md` port map.
9. **Verify in Compose**: add the new service to `docker-compose.local.dev.yml` with its published host port, then confirm `make local-dev-rebuild SERVICE=<new-remote>` and a full `make arm-run` both bring the shell up with the new remote loading correctly.

---

## 9. Known pitfalls (summary)

| Pitfall | Where | Status |
|---|---|---|
| `core-fe/vite.config.ts`'s UnoCSS cross-scan path for calendar resolves to a nonexistent directory (`core/calendar/src`) | §6 | Verified broken, low visible impact — `uno.config.ts` has the correct path |
| `@aracreate/test-arm-ui` exact pins disagree — `calendar-fe` on `3.1.5`, host and admin on `3.1.4` | §4 | Verified. Singleton negotiation gives every app the host's `3.1.4`, so a calendar component using a `3.1.5`-only export passes standalone dev and fails only once federated |
| Host/remote `react`/`react-dom` version pins disagree (`19.2.0` vs `19.1.1`) | §4 | Verified, silently resolved by singleton negotiation (host wins) — no known breakage |
| `theme-overrides.css` is duplicated verbatim between `core-fe` and `calendar-fe` with no sync check | §5.3 | By design (standalone brand colours), but divergence shows up as two different yellows |
| `calendar-ds.css` patches in classes the shared library renders but doesn't ship styles for | §5.3 | Workaround in place; the real fix is upstream in the library's `tokens.css` |
| Remotes reach `core/toast` / `core/theme-store` with no static `remotes.core` declaration and no compile-time contract check | §3.1 | By design (runtime federation), but a rename of either host export breaks both remotes silently |
| `admin-fe` doesn't use the shared `@aracreate/test-arm-ui` UnoCSS preset the other two apps use | §6 | Observed, undocumented as either intentional or a gap — note it *does* share the package as a federation singleton |
| The `.calendar-scope` wrapper requirement | §5.1 | **Resolved by removal** — FullCalendar is gone from the calendar app, the wrapper is gone from both rendering paths. Do not reintroduce |

---

## 10. Related documents

| Doc | Role |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) §6 | One-table MF summary, part of the platform-wide map |
| [CLAUDE.md](../../../CLAUDE.md) | MF exposes list, workspace layout, and the note that the CSS isolation rule is obsolete |
| [`core/arm-core-fe/CLAUDE.md`](../../../core/arm-core-fe/CLAUDE.md) | Host-side source structure, route map, auth state |
| [`core/arm-core-fe/README.md`](../../../core/arm-core-fe/README.md) | Host-side how-to — adding a new remote, Settings/Linked Accounts page architecture |
| [`apps/arm-app-calendar/CLAUDE.md`](../../../apps/arm-app-calendar/CLAUDE.md) | Calendar remote's backend + frontend context |
| [`calendar/ARCHITECTURE.md`](calendar/ARCHITECTURE.md) §9 | Calendar remote's MF exposes, standalone-dev stubbing, SSE-based live updates |
| [`apps/arm-admin/src/frontend/CLAUDE.md`](../../../apps/arm-admin/src/frontend/CLAUDE.md) | Admin remote's source structure, auth fallback |
| [`troubleshooting/mfe-not-loading.md`](troubleshooting/mfe-not-loading.md) | Symptom-first triage guide built on this document's §9 pitfalls |
| [ADR-002](adr/002-module-federation.md), [ADR-006](adr/006-unocss-mfe-scoping.md) | Why Module Federation and this UnoCSS scoping strategy were chosen, and what alternatives were considered |
