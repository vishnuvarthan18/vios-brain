# Troubleshooting — MFE Not Loading

Symptom-first triage for Module Federation failures — a remote (`calendar` or `admin`) not appearing in the shell, loading unstyled, or throwing a federation error in the console. [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) is the architecture reference this doc maps symptoms onto — it already documents 5 verified pitfalls in detail (§9); this document is the "what do I actually check" companion.

**Correction to a common assumption before you start**: `core-be` does have a `GET /v1/registry` endpoint (`src/registry/`), and `core-fe` genuinely does call it (`ProtectedRoute` → `initRemoteRegistry()`, on every authenticated session) — this document previously said otherwise, which was wrong; corrected 2026-08-12. What's still true, for a different reason: nothing anywhere writes to the `registry` table, so that call always comes back empty and never actually changes anything. Remote URLs are, in every environment that exists today, the **static env vars** (`VITE_CALENDAR_REMOTE_URL`, `VITE_ADMIN_REMOTE_URL`), baked in at build/container-start time, defaulting to `http://localhost:3001/calendar/remoteEntry.js` / `http://localhost:10000/admin/remoteEntry.js`. If you're chasing "why doesn't the shell know where the remote is," don't look at the registry endpoint — check the env var. Full mechanism detail: [`core/arm-core-be/REGISTRY.md`](../../../../core/arm-core-be/REGISTRY.md).

---

## 1. Quick triage table

| Symptom | Most likely cause | Jump to |
|---|---|---|
| Blank area / error boundary where the remote should be | Remote dev server not running, or env var pointing at the wrong URL | §2 |
| Console shows a shared-module/version error | Singleton negotiation conflict (rare — see §3 for why it usually *doesn't* error) | §3 |
| Remote loads, but layout or theming looks wrong | The standalone-only CSS gotcha, or a shared-library version mismatch | §4 |
| A calendar/admin utility class doesn't generate in the shell's build | UnoCSS cross-scan path — broken for calendar, N/A for admin | §5 |
| Toast notifications or theme switching silently no-op inside a remote | Reverse host→remote import (`core/toast`, `core/theme-store`) failed silently | §6 |
| Works at `localhost:3001`/`:10000` standalone, breaks embedded in the shell | Standalone entry point (`App.tsx`) vs. federated exposes are genuinely different code paths | §7 |

---

## 2. Remote fails to load entirely

**Check the env var first**: `VITE_CALENDAR_REMOTE_URL` / `VITE_ADMIN_REMOTE_URL` on the shell (`core-fe`) must point at a URL that's actually serving `remoteEntry.js`. In local dev this defaults to `http://localhost:3001/calendar/remoteEntry.js` / `http://localhost:10000/admin/remoteEntry.js` (note the path segment — each remote sets a `base`, so `remoteEntry.js` is not at the server root) — if the remote's dev server isn't running (or `make local-dev`'s health cascade hasn't gotten it healthy yet — see [restart-services.md](../runbooks/restart-services.md) §2's startup-order diagram), the shell has nothing to load.

**Diagnostic**: open the browser's Network tab and look for the `remoteEntry.js` request specifically — a failed/404/connection-refused request there confirms the remote isn't reachable at the configured URL, which is a deployment/env-var problem, not a code problem. If `remoteEntry.js` loads successfully but the specific exposed component still fails, the problem is in that remote's `exposes` config (a typo'd path, or the component file itself throwing on import) rather than reachability.

**The route is wrapped in `RemoteErrorBoundary`**, and the dynamic import is wrapped in `timedLazy()`, which fails the import after 15s rather than hanging the route indefinitely — so a genuinely broken remote should render a caught error state, not crash the whole shell. If the *entire shell* goes blank rather than just the remote's mount point, the failure is happening somewhere the error boundary doesn't cover (e.g. during the dynamic `import()` itself before the boundary's render path engages) — worth distinguishing "the remote's component threw" from "the import() promise itself rejected," since they surface differently.

---

## 3. Shared-dependency / version errors

Per [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §4, `react`/`react-dom` are pinned to different versions between the host (`^19.2.0`) and both remotes (`^19.1.1`) — and this is a **known, currently-harmless** drift, not something to chase as the cause of a real bug. Because all three declare `singleton: true`, the host's version wins the negotiation and the remotes' own pinned version is advisory only — the remotes run on `19.2.0` in practice, whatever their own `package.json` says. If you see a shared-singleton console warning specifically about this pairing, it's expected and not actionable on its own.

**If a *new* shared-dependency error appears** (a package not in the table above, or a genuine breaking-version conflict): confirm the package is declared in `shared` in **every** `vite.config.ts` that uses it — per §4's own rule, a dependency shared by the host but not declared in a remote's `shared` block loads as a separate, non-singleton instance in that remote, which is a common way to introduce exactly this class of error when adding a new shared library.

---

## 4. Remote loads but looks wrong — CSS/styling issues

This is the most common "MFE not loading correctly" symptom in practice, and it's mostly already-documented, verified behavior rather than a mystery to debug from scratch:

- **Don't look for a `.calendar-scope` wrapper — it no longer exists.** It was required only because FullCalendar reset CSS globally; FullCalendar has been removed from the calendar app and the wrapper was deleted from both `core-fe/src/routes/app.router.tsx` and calendar-fe's own `App.tsx`. If a doc, comment, or older branch tells you to add it back, that instruction is stale — see [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §5.1.
- **A style that works at `:3001` doesn't apply when embedded**: check *which file* it's in. `calendar-styles.css` is imported only by `App.tsx`, and `index.css` only by `main.tsx` — both are standalone-only entry points, never part of the federation path, so nothing in either file loads behind the shell. Styles that must apply when federated belong in `calendar-ds.css` or `theme-overrides.css`, which the components themselves import. Table of all four files: [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §5.3.
- **A design-system component renders unstyled or with the wrong brand colour**: the shared UI library is a federation singleton and the **host's** version wins. `calendar-fe` pins `@aracreate/test-arm-ui` at `3.1.5` while `core-fe` and `admin-fe` pin `3.1.4`, so a calendar component written against a `3.1.5`-only style or export looks correct standalone and wrong (or throws) once federated. Compare the three `package.json` pins before debugging the CSS itself — [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §4.
- **Two different yellows between the shell and the calendar area**: `theme-overrides.css` is duplicated verbatim in `core-fe` and `calendar-fe` on purpose (standalone mode needs its own copy). If they've drifted, the federated path shows the host's copy and standalone shows calendar's — diff the two files.
- **Whole shell theme breaks (colors wrong platform-wide) after touching calendar CSS**: the opposite failure mode — check that nothing was added to `calendar-styles.css` that redefines root CSS variables (`--bg`, `--text`, `--primary`, etc.). The file's own header comment explains these must never be redefined there, since `core-fe`'s `index.css` already owns them and calendar's federated components inherit them.

---

## 5. A utility class doesn't generate in the shell's build

Applies specifically to **calendar** — admin ships its own bundled CSS and doesn't rely on the shell's UnoCSS scan (§6 of [MODULE-FEDERATION.md](../MODULE-FEDERATION.md)).

**This is very likely the known broken cross-scan path**, not a new bug: `core-fe/vite.config.ts`'s UnoCSS plugin config points at `"../calendar/src/**/*.{ts,tsx}"`, which resolves to `core/calendar/src` — **a directory that doesn't exist**. The correct path (used correctly in `core-fe/uno.config.ts`, just not in `vite.config.ts`) is `"../../apps/arm-app-calendar/src/frontend/src/**/*.{ts,tsx}"`. Because Vite's build pipeline still transitively processes calendar's federated chunks, this hasn't caused *total* class-generation failure historically — but if a specific new utility class used only in calendar source isn't appearing in the shell's generated CSS, fix this path first before assuming UnoCSS itself is misbehaving.

---

## 6. Toast / theme calls silently doing nothing inside a remote

Both remotes reach into the host at runtime without a static `remotes.core` declaration (`import("core/toast")` in admin, `import("core/theme-store")` in calendar) — see [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) §3.1. This works because the remote is only ever mounted inside a shell that's already registered itself as `core` in the browser's federation runtime — but it has **no compile-time contract check**. If someone renames or removes `./toast` or `./theme-store` from `core-fe`'s `exposes`, both remotes fail this specific import **silently** — they fall back to their standalone-mode stub behavior (a no-op toast, `useThemeStore = undefined`) rather than throwing an error anywhere visible. If toast notifications or theme sync stop working inside an embedded remote with no console error at all, this reverse-import contract is the first thing to check — grep `core-fe/vite.config.ts`'s `exposes` for the exact module names both remotes depend on.

---

## 7. Works standalone, breaks (or looks different) embedded in the shell

`calendar-fe`'s `App.tsx` and `admin-fe`'s `main.tsx` are **standalone-dev-only entry points** — never part of what federation actually exposes (§3, §5.3 of [MODULE-FEDERATION.md](../MODULE-FEDERATION.md)). The shell imports the individual exposed page components directly (`calendar-account-page.tsx`, `sync-view-page.tsx`, etc. — see the `exposes` map), not `App.tsx`. **Anything only imported or wrapped inside `App.tsx` is invisible to the federated path** — this is exactly what causes §4's CSS gotcha, and it can affect anything else added to `App.tsx` in the future (a new provider, a new global wrapper) without realizing it won't reach the embedded version. When a remote behaves differently standalone vs. embedded and it isn't the CSS issue already covered, check whether the differing behavior lives in `App.tsx`/`main.tsx` specifically before assuming it's a federation runtime bug.

---

## 8. Diagnostic checklist

1. **Network tab**: confirm `remoteEntry.js` loads from the expected URL (not 404, not connection-refused).
2. **Confirm the env var**: `VITE_CALENDAR_REMOTE_URL`/`VITE_ADMIN_REMOTE_URL` on the shell container — especially after any port/URL change to a remote.
3. **Confirm the remote's dev server directly** — hit `http://localhost:3001` (or `:10000`) on its own; if it doesn't respond standalone, federation was never going to work regardless of shell config.
4. **Check `make local-dev-ps`** (or equivalent) — per [restart-services.md](../runbooks/restart-services.md), `core-fe` has no healthcheck of its own and both remotes' healthchecks only confirm Vite is serving *something*, not that the app is functional — a "healthy"/"running" status doesn't rule out a broken remote.
5. **Console errors** — a federation-specific error (module resolution, shared-scope conflict) points at config; a React error from inside the component itself points at the component's own code, unrelated to federation.

---

## Related documents

- [MODULE-FEDERATION.md](../MODULE-FEDERATION.md) — the architecture reference this doc triages against, including 5 verified pitfalls in its own §9
- [runbooks/restart-services.md](../runbooks/restart-services.md) — healthcheck strength per frontend service, relevant to step 4 above
- [calendar/ARCHITECTURE.md](../calendar/ARCHITECTURE.md) §9 — calendar-fe's exposes, standalone-dev stubbing detail
- [DATABASE-DESIGN.md](../DATABASE-DESIGN.md) §2.4 — background on why `core-be`'s `registry` table is always empty despite the endpoint being live and read, referenced in this doc's opening correction
- [`core/arm-core-be/REGISTRY.md`](../../../../core/arm-core-be/REGISTRY.md) — full mechanism detail: read path, trust boundary, why the write path is never invoked
