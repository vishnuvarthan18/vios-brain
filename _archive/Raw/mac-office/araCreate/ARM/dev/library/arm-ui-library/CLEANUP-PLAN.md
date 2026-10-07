# ARM Cleanup Plan — canonical tracker

> Working document for the full-project cleanup (audited 2026-07-10).
> Status markers: `[ ]` todo · `[~]` in progress · `[x]` done · `[!]` blocked (needs a decision
> or external input — do not start) · `[-]` dropped.
> One commit per phase so each is revertable. Update statuses **in place** as phases land —
> don't re-derive this plan from scratch, and don't re-ask the user questions already answered
> in the decisions table below.

## ✅ PLAN CLOSED — 2026-07-10

All actionable phases are **done and committed**. This tracker is a historical record.

**Follow-up work** completed in [`RADIX-DS-PLAN.md`](./RADIX-DS-PLAN.md) — Radix-native design
system migration (closed 2026-07-10).

| Phase | Status | Commit |
|-------|--------|--------|
| 0 Git safety net | `[x] DONE` | `bdcd223` / `22c0ddf` / `7576fb8` |
| 1 Bug fixes | `[x] DONE` | `bafa957` |
| 2 Radix snap revert | `[-] DROPPED` | — (decision ②: bless Radix tokens; see RADIX-DS-PLAN.md) |
| 3 Mock-API consolidation | `[x] DONE` | `5a7aed5` |
| 4 Dead code & docs | `[x] DONE` | `43f7f27` |
| 5 Typography sweep | `[x] DONE` | `3e14646` |
| 6 Tooling & scripts | `[x] DONE` | `0d8f6fd` |
| Final gate (tooling) | `[x] DONE` | lint · typecheck · test (167/167) · build |
| Final gate (visual) | `[-] DROPPED` | Figma PNG diff superseded by look-and-feel smoke pass in RADIX-DS-PLAN.md |

**Deferred / not in scope:** `React.lazy` route splitting · CI wiring.

## ▶ IF RESUMING LATER

This plan is **closed**. All new UI work continues in [`RADIX-DS-PLAN.md`](./RADIX-DS-PLAN.md).
Read that file's `▶ NEXT ACTION` section and follow it exactly — do not re-derive work
from scratch or re-ask decisions already marked ✅ in either plan's tables.

| # | Question | Options | Decision |
|---|----------|---------|----------|
| ① | `../archive/` + `../ui-ux/T9T file.zip` | commit / delete (rec: delete archive, keep zip — only offline Figma backup, read quota spent) | ✅ **delete both** — archive (2026-07-10), `T9T file.zip` (2026-07-10) |
| ② | "Radix snap" sites | revert to measured Figma px / bless token snap everywhere | ✅ **bless Radix tokens everywhere** — no Figma px; Phase 2 dropped; see `RADIX-DS-PLAN.md` (2026-07-10) |
| ③ | Unused DS components `CalendarViewToggle`, `PageHeading` | delete / keep as DS inventory | ✅ **delete** (2026-07-10) |
| ④ | Dead buttons (Google/Microsoft, Invite user, Export CSV, avatar picker) | wire cheap mock (rec for Export CSV) / disable + "coming soon" (rec for rest) / leave | ✅ **mixed: Export CSV = cheap mock; rest = disable + "coming soon"** (2026-07-10) |

---

## Phase 0 — Git safety net  `[x] DONE`  *(2026-07-10, commits bdcd223 / 22c0ddf / 7576fb8)*

Everything else is unsafe until the work is committed. Only commit in history is
"Initial commit from Create Next App"; the whole Vite app is untracked.

- [x] Rewrite root `ARM/.gitignore` for Vite: unanchored `node_modules/`, plus `dist/`,
      `storybook-static/`, `*.tsbuildinfo`, `coverage/`, `.DS_Store`, `*storybook.log`.
      Drop Next entries (`/.next/`, `next-env.d.ts`, `.vercel`).
      Also added `**/.claude/settings.local.json` (local permission grants, both levels).
- [x] Delete leftovers: `ARM/.next/`, stray `.DS_Store` files. Decision ① applied:
      `archive/` deleted (393MB); `ui-ux/T9T file.zip` deleted later (2026-07-10).
- [x] Commit 1: old Next skeleton removal (pending `D` deletions) + new .gitignore. → `bdcd223`
- [x] Commit 2: add `arm-ui/` (156 files; README + AGENTS.md fixes folded in so the
      tree never enters history stale). → `22c0ddf`
- [x] Commit 3: add `ui-ux/` (design spec that AGENTS.md rules depend on). → `7576fb8`
- [x] Rewrite `arm-ui/README.md` — now documents Vite 8 / react-router 7 / Radix /
      port 5173 / passwordless auth; stale Next.js 16 content gone.
- [x] Patch AGENTS.md drift: add `RequireGuest` to guards list (DS list fixed in Phase 4).
- [x] Verify: `git status` clean ✓; fresh-clone `npm install && npm run build` ✓
      (only warning = known 575KB chunk, tracked in Phase 6).

## Phase 1 — Bug fixes  `[x] DONE`  *(2026-07-10, commit bafa957)*

- [x] **B1** `AvailabilityView.tsx` — event tint/border now
      `color-mix(in srgb, <token> 12%/33%, transparent)`; verified non-transparent
      computed background in "Show titles" mode.
- [x] **B2** `AccountManager.tsx` — grid back to `repeat(auto-fill, 240px)`;
      computed style verified `240px 240px …`.
- [x] **B3** `CalendarOAuthPage.tsx` — `acct` validated (NaN→0, clamped 0..len-1);
      `?acct=-1/abc/99` all render.
- [x] **B4** `/calendar-oauth` moved inside `RequireAuth` (kept outside ShellLayout —
      full-screen consent page); unauthenticated visit redirects to /login.
- [x] **B5** Dead `restoring` state + spinner branches + lying comments removed from
      `auth-context.tsx`, `RequireAuth.tsx`, `RequireGuest.tsx`.
- [x] **B6** AdminUsersPage unreachable error phase deleted (incl. now-unused
      Callout/ExclamationTriangleIcon imports); phase union is `"loading" | "success"`.
- [x] **B7** OTP auto-verify now event-driven from `handleCodeChange`/`handleOtpChange`
      (verify fn takes the code as an argument — no stale-state effect). CalendarPage
      eslint-disable moved to the deps line where the warning actually fires.
- [x] **B8** Sparkline gradient id from `useId()` (colons stripped — they break
      `url(#…)`); MetricCard delta color driven by new `goodDirection` field on
      `PlatformMetric` (symmetric green/red/flat-gray — no label matching).
- [x] Verify: eslint 0 errors (12 pre-existing fast-refresh/storybook warnings),
      `tsc --noEmit` clean; 10/10 Playwright checks green at 1440×950 (login OTP →
      connect ×2 → source select → sync → merged availability; admin users; reload).

## Phase 2 — Resolve the "Radix snap" damage  `[-] DROPPED`  *(decision ②: bless Radix tokens — superseded by RADIX-DS-PLAN.md)*

A mechanical pass replaced measured Figma px with nearest space tokens (~10 sites,
grep `snapped\|clamped` in src/). Assuming revert-to-measured:

- [ ] `src/components/auth/AuthCard.tsx:30,44` — restore `calc(100vh - 54px)` (54 = 2×27px page padding; 48 makes sticky panel 6px too tall).
- [ ] `src/components/admin/PortalNav.tsx:60` — tab underline overlap back to `-1px` (now −4px).
- [ ] `src/components/ds/MonthGrid.tsx:31-33` — cells 56/44, gap 6 (now 64/48/8 → grid ~60px too wide). Chevrons back to 18px (`:109,126`).
- [ ] `src/components/calendar/AvailabilityView.tsx` — SLOT_H 56 (`:21`), min event height 22 (`:119`), today-circle margin 2px (`:404`), time-label offset −7px (`:436`).
- [ ] `src/components/ds/PillTabs.tsx:62` — padding 6px.
- [ ] `src/components/auth/OtpInput.tsx:46` — gap 10px.
- [ ] `src/pages/admin/AdminAuditPage.tsx:40` — icon 22px; `SearchOverlay.tsx:311` icon 13px.
- [ ] Fix mixed-unit icon props (`width="var(--space-4)" height="16px"`) in AccountManager/SyncSetup/CalendarEmpty.
- [ ] KEEP documented deliberate deviations: topbar search/bell sizing, SHELL_HEADER_H (comments explain a real decision).
- [ ] Verify: Playwright screenshots (settings, calendar setup, month grid, auth) vs `../ui-ux/figma/exports/` PNGs.

## Phase 3 — Mock-API consolidation  `[x] DONE`  *(2026-07-10)*

Extend the `lib/api/auth.ts` pattern ("swap one file for a real backend") to everything:

- [x] New `src/lib/api/users.ts` (admin CRUD, sessions, audit), `api/calendar.ts`
      (accounts, sync incl. the 2200ms), `api/profile.ts` (settings save, linked accounts).
      All fake latency moves here (+ shared `api/latency.ts`).
- [x] Kill every page-level fake `setTimeout`: HomePage 900ms, AdminUsersPage 800ms,
      AdminUserDetailPage 700/450/400/500ms, SettingsPage 500ms, CalendarOAuthPage 800/400ms,
      calendar-context `startSync` 2200ms.
- [x] AdminUserDetailPage: drop the local `sessions` state copy + render-time sync hack —
      read from admin-context as single source of truth.
- [x] Move inline mock data → mock.ts: hardcoded phone `(removed)`, fake linked-account
      domains (SettingsPage), "21 Jun 2026" label (HomePage); one exported `MOCK_TODAY`
      instead of duplicate `new Date(2026, 5, 22)` in AvailabilityView + SyncSetup.
- [x] OTP endpoints return `expiresAt`; Login/Signup stop hardcoding 5 min (drift vs `OTP_TTL_SEC`).
- [x] Extract shared `OtpVerifyStep` component (dev-hint callout, countdown, resend button
      w/ duplicated 12-line style, "Use a Different Email") — used by Login + Signup.
- [x] Settings save updates the auth user (sidebar name follows rename). Language field →
      proper Select.
- [x] Verify: all flows; grep proves zero fake-latency `setTimeout` outside `lib/api/` + contexts
      (remaining page timers = resend cooldown, toast dismiss, signup success pause, search debounce).

## Phase 4 — Dead code, stale docs, dev gating  `[x] DONE`  *(2026-07-10)*

- [x] Delete stubs `src/components/calendar/SourceManager.tsx`, `MergedView.tsx`.
- [x] Apply decision ③ to `CalendarViewToggle` + `PageHeading` (used nowhere; update
      `ds/index.ts` + AGENTS.md DS list either way).
- [x] Purge ~12 dead CSS classes in globals.css (password-era & retired-layout):
      `.btn-eye-toggle`, `.strength-bar`, `.topbar-avatar-btn`, `.profile-chip-trigger`,
      `.shell-bottom-action`, `.btn-shortcut`, `.auth-back-link`, `.auth-text-link`,
      `.auth-accent-link`, `.admin-table-row`, `.auth-mobile-logo`, `.btn-social` (verify),
      `.field-label` (verify), `.sr-only` (verify). Kept `.btn-social-lg` + `.field-label-lg`.
- [x] Fix lying comments: theme-overrides.css:40 `slate`→`gray`; auth-context `signInAs`
      "Topbar"→Sidebar; calendar-context deleted Next path `app/(shell)/calendar/layout.tsx`.
- [x] Gate "Dev: simulate provider error" (CalendarOAuthPage) behind `import.meta.env.DEV`
      (sidebar role switcher already is).
- [x] Apply decision ④ to dead buttons (Google/Microsoft, Invite user, Export CSV, avatar
      picker, notifications bell).
- [x] TermsPage: "Last updated June 2024" vs `CURRENT_TERMS_VERSION = "2026-06-01"` — align.

## Phase 5 — Typography sweep  `[x] DONE`  *(2026-07-10)*

- [x] Replace raw inline `font-size` (10–28px) with Radix `size` props or existing
      `ds-section-label` / `data-mono` classes. Offenders: HomePage, Sidebar, SearchOverlay,
      PortalNav, AdminAuditPage, AdminUserDetailPage, CalendarOAuthPage, AvailabilityView,
      Login/Signup resend buttons (via `OtpVerifyStep`).
- [x] Genuinely new sizes → documented class in globals.css with Figma measurement comment
      (`.metric-card-value` 28px, `.search-overlay-input`; tokenized `.data-mono` /
      `.cal-time-label` / `.topbar-search-placeholder`).
- [x] Verify: `tsc --noEmit` + `eslint` + `build` green; visual pass deferred to final gate
      Playwright screenshots.

## Phase 6 — Tooling & scripts  `[x] DONE`  *(2026-07-10, commit 0d8f6fd)*

- [x] package.json: `"typecheck": "tsc --noEmit"`, `"test": "vitest run"`, explicit
      `"lint": "eslint ."`.
- [x] `vitest.config.ts` merges app Vite config so `@/` alias + React plugin resolve in
      Storybook browser tests; fixed router decorators in SyncSetup + Sidebar stories.
- [x] Storybook vitest suite: **167/167** passing (requires `npx playwright install chromium`).
- [ ] Deferred/optional: `React.lazy` route splitting for admin + calendar (575KB single JS chunk, 707KB CSS); CI.

## Final gate  `[x] DONE` *(tooling)* · `[ ] optional` *(visual)*
ok
- [x] lint + typecheck + test + build all green (eslint 0 errors, 12 pre-existing warnings).
- [x] Storybook vitest: 167/167 passing (`npm run test`; needs Playwright chromium).
- [ ] Full Playwright screenshot pass vs Figma exports (1440×950, client-side nav,
      passwordless login per AGENTS.md) — **optional; not run in this cleanup pass**.
- [x] Each phase = one commit; history tells the cleanup story.

---

## Audit reference (2026-07-10 findings, condensed)

- **A. Repo:** nothing committed; .gitignore trap (root-anchored patterns); `.next/` leftover; stale README.
- **B. Bugs:** B1 invalid CSS color concat · B2 64px card grid · B3 OAuth param crash ·
  B4 oauth route unguarded · B5 dead `restoring` · B6 broken error/retry · B7 2 lint errors ·
  B8 sparkline id collision + label-string logic.
- **C. Snap damage:** ~10 sites where measured Figma px got replaced by space tokens (worst: 240→64).
- **D. Consistency:** fake setTimeouts in pages (violates own AGENTS.md principle) · raw font-sizes ·
  dead files/components/CSS · stale comments · mock data in pages · Login/Signup duplication ·
  dead buttons · ungated dev button · missing test/typecheck scripts.
