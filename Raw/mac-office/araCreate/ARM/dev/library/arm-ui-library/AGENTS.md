# arm-ui

Plain TypeScript React SPA — not Next.js. Built with Vite, routed with `react-router-dom` (plain `<Routes>`/`<Route>`, no data-router/loaders), styled with Radix Themes.

- Entry: `index.html` → `src/main.tsx` → `src/App.tsx` (route tree).
- Routes: `src/routes/RequireAuth.tsx`, `RequireGuest.tsx`, and `RequireRole.tsx` are the auth/guest/role guards; `src/layouts/` holds the shell/admin/calendar layout wrappers; `src/pages/` holds one file per route.
- `@/*` resolves to `src/*` (see `tsconfig.json` and `vite.config.ts`).
- Auth state (`src/lib/auth-context.tsx`) persists across hard reloads via `src/lib/api/session-storage.ts` (localStorage-backed mock session, restored on boot with a loading state in `RequireAuth`). Role/integrations/calendar state (`src/lib/admin-context.tsx`, `integrations-context.tsx`, `calendar-context.tsx`) is still plain in-memory — reload resets those, client-side nav (`<Link>`, `navigate()`) does not.
- All auth operations go through `src/lib/api/auth.ts` — a mock API client with a typed `AuthErrorCode` taxonomy (see file header) standing in for a real backend. Never add ad-hoc `setTimeout`/error-string logic to a page or to `auth-context.tsx`; extend the mock client instead so a real backend swap later only touches that one file.
- Storybook uses `@storybook/react-vite`; stories needing router context use the `withRouter` decorator in `stories/app/_mocks.tsx` (wraps in `MemoryRouter`), not any Next-specific mocking.
- Scripts: `npm run dev` / `build` / `preview` are Vite; `npm run storybook` / `build-storybook` are Storybook.

## Design system rules — ALWAYS follow when building or changing UI

**Active migration tracker:** [`RADIX-FULL-PLAN.md`](./RADIX-FULL-PLAN.md) — component +
layout migration (Phases 8–14). Token work is complete in [`RADIX-DS-PLAN.md`](./RADIX-DS-PLAN.md)
(Phases 0–7, closed). **Calendar app spec:** [`CALENDAR-APP-SPEC.md`](./CALENDAR-APP-SPEC.md).
Read the active plan's `▶ NEXT ACTION` before UI work.

**Source of truth:** Radix Themes scale — `size` props on Text/Heading/Button, and
`var(--space-*)`, `var(--radius-*)`, `var(--font-size-*)`, semantic color tokens.
Figma exports in `../ui-ux/figma/exports/` are a **look-and-feel reference only** — not a
measurement spec. No ÷1.2 math, no raw Figma px, no pixel-diff screenshot verification.

**Snapping:** when replacing a value, use the nearest Radix step. Small drift (≤ one space
step) is OK; do not make large jumps without user approval.

**Typography** — Poppins everywhere (`--font-poppins`), `data-mono` class for
monospace/meta. Use Radix `size` props, never raw `font-size`:

| Use | Radix size |
|---|---|
| Topbar/page title | Heading `5` medium |
| Card titles / account names | `4`–`5` medium |
| Body, nav items, inputs | `3` |
| Secondary text, field labels | `2` |
| Badges, emails, meta | `1` |

Weight: use `medium` for emphasis (titles, active states) — never `bold`, per the
Vercel-density pass (2026-07-27). Radix's own `weight="bold"` reads heavier than
this app's reference.

**Buttons** — geometry comes from the Radix `size` prop ONLY. Never set
`height`, `paddingInline`, or `font-size` on a button inline or in CSS:
- Standard CTA (Add New Account, Sync, Cancel): `size="3"` + `btn-amber-primary`
  (amber) or `btn-dark-accent` (dark). These classes style color/shadow only.
- Dialog / compact actions: `size="2"`.
- Settings Save: `size="4"`.
- Auth screens: `size="4"` + `btn-auth-cta` (width/radius utility only — no height override).

**Components** — compose from `@arametrics/ds` (github:aracreate-group/arm-design-system,
a real dependency in `package.json` — not a local folder) before writing anything new:
`AccentCard`/`AccentHeader` (amber-header cards), `AccountCard` (account tiles), `CardFooter`,
`ConfirmDialog`, `EventBlock`, `IconTile`, `MonthGrid`, `PausedBadge`, `Skeleton`, `Toast`.
Plain Radix primitives for the rest. `src/components/calendar/`, `auth/`, `admin/`, `shell/`
stay in this repo — they're product features (use app contexts/routing), not design-system
primitives, and were deliberately excluded from the split.
New bespoke CSS classes in `globals.css` must use Radix tokens only — no raw px for
spacing, sizing, or typography (1px borders, shadows, and focus outlines are fine).
Component layout widths should use `src/lib/layout-tokens.ts` (`layout.*` calcs).

**Verify** — after UI changes, run `npm run lint && npm run typecheck && npm run build`.
Spot-check affected screens in dev for look-and-feel (not Figma pixel diff).
Auth is passwordless (no password anywhere): on /login enter
`user@arametrics.io`, click Continue, then type the 4-digit code shown in
the "Dev hint" callout (the mock sends no real email).
