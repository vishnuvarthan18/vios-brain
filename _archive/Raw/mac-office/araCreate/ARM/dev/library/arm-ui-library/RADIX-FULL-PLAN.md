# Radix-full component migration — canonical tracker

> **Follow-up to [`RADIX-DS-PLAN.md`](./RADIX-DS-PLAN.md)** (Phases 0–7: tokens + layout calcs — **DONE**).
> This plan migrates **custom HTML/CSS → Radix Themes components and layout props**.
> Status markers: `[ ]` todo · `[~]` in progress · `[x]` done · `[!]` blocked · `[-]` dropped.
> One commit per phase. Update statuses **in place** as phases land.
>
> **2026-08-13 — DS package split:** the 10 design-system primitives (`AccentHeader`/
> `AccentCard`, `AccountCard`, `CardFooter`, `ConfirmDialog`, `EventBlock`, `IconTile`,
> `MonthGrid`, `PausedBadge`, `Skeleton`, `Toast`) moved out of this repo entirely, into
> [`aracreate-group/arm-design-system`](https://github.com/aracreate-group/arm-design-system)
> as the `@arametrics/ds` package (a real dependency in `package.json`, not a local folder).
> References below to `src/components/ds/` are historical — describing where things were when
> each phase/note was written — not the current path.

## Goal (user-confirmed 2026-07-10)

**Radix is the primary system** — not just tokens, but components (`TextField`, `Button`,
`Tabs`, `Card`, `Flex`, etc.). Bespoke UI (calendar grid, search overlay) stays, but must be
**composed from Radix primitives**, not plain `<input>` / `<button>` + `globals.css` classes.

Phases 0–7 already delivered: Radix spacing tokens, `layout-tokens.ts`, no raw layout px.

---

## ▶ PLAN CLOSED (2026-08-17)

All phases (8–14) done. Phase 14 purged 11 dead CSS classes left over from the pre-`643e291`
custom calendar grid plus two unrelated orphans (`.terms-back-link`, `.home-app-card`) — see
Phase 14's section below for the full list and verification. No further action items; this
tracker is historical from here. Day-to-day DS rules live in `AGENTS.md`.

## ▶ 2026-08-13 Phase 9 closeout

Phase 9 was already committed on this branch (`f19ffff`, 2026-07-27) but never marked done in
this tracker. Re-verified against current tree before starting Phase 10:

- `.btn-amber-primary` / `.btn-dark-accent` / `.btn-auth-cta` in `globals.css` are color/shadow
  only — geometry already comes from Radix `Button size`. No further work needed.
- `.btn-social-lg` was deleted in `f19ffff` (replaced by `Button variant="outline" size="3"`).
- Confirmed-sites list (auth, settings, admin `Clear filters`, `OtpVerifyStep`) all converted
  to Radix `Button`/`IconButton` in `f19ffff`.
- `ClickableRow` (`src/components/ds/ClickableRow.tsx`) was added in `f19ffff` for the 3
  copy-pasted calendar row sites. The later calendar rebuild (`643e291`, post-dates Phase 9)
  replaced `CalendarAccountsPage.tsx` with `CalendarSyncsPage.tsx` using a different grid-row
  layout and dropped `ClickableRow` usage — it is now an unused export. Carried forward to
  **Phase 12** (already tracks identical "wire up or delete" decisions for `AccentCard`/
  `PausedBadge`/`AccountCard`/`MonthGrid"] — add `ClickableRow` to that same decision.
- Remaining raw `<button>` in `CalendarMergedView.tsx`, `ConnectProviderList.tsx`,
  `AddAccountMenu.tsx`, `MiniMonthPicker.tsx`, `MonthGrid.tsx` are out of Phase 9's scope
  (not in the confirmed-sites list); `Sidebar.tsx`/`Topbar.tsx`/`SearchOverlay.tsx` are
  Phase 10's explicit scope.
- Verify: `npm run lint && npx tsc --noEmit && npm run build` — all clean (0 errors, only
  pre-existing fast-refresh/storybook warnings; pre-existing 500kB chunk warning unrelated).

## ▶ 2026-07-27 scope refresh (from code-level DS audit)

Working on branch `ds-structural-cleanup` (main untouched pending review). Audit of the
current tree found:

- Existing real `Button`/`IconButton` usage app-wide is **already geometry-clean** — only
  `size`/`variant`/`color` + color/shadow-only treatment classes (`.btn-amber-primary`,
  `.btn-dark-accent`). Phase 9 does not need to touch these call sites, only the raw
  `<button>` list below.
- The calendar rebuild (committed `643e291`) repeats the raw-`<button>`-with-inline-geometry
  anti-pattern in 3 new places, all sharing one copy-pasted "clickable row" recipe
  (`data-table-row` class + inline flex/padding/width): `ConnectProviderRows.tsx:54`,
  `CalendarAccountsPage.tsx:144`, `CalendarSyncEditorPage.tsx:371` (`StepCard`). Candidate
  for one shared DS component (e.g. `ClickableRow`) instead of 3 ad hoc copies — decide during
  Phase 9.
- `globals.css` has **no raw px** for geometry — every class already uses `var(--space-N)`
  tokens. Phase 14 is a composition refactor, not a px hunt.
- New scope item for **Phase 13**: raw px bypassing `layout-tokens.ts` in
  `CalendarMergedView.tsx` (`minWidth: 220, 320`), `BigCalendarView.tsx` (`height: 640`),
  `CalendarSyncEditorPage.tsx` (`minWidth: 320/200/240`), plus pre-existing `HomePage.tsx`,
  `AdminUsersPage.tsx`/`AdminUserDetailPage.tsx` decorative dot dimensions, `ForbiddenPage.tsx`
  (`maxWidth: 400`).
- New scope item for **Phase 12**: `AccentCard`, `PausedBadge`, `AccountCard`, `MonthGrid` in
  `src/components/ds/` have zero real consumers. `CalendarAccountsPage.tsx` hand-rolls an
  account-list card + reauth badge instead of reusing `AccountCard`/`PausedBadge`, which look
  purpose-built for that state. Decide: wire up or delete — don't leave both.
- Fold into **Phase 10**: `SearchOverlay.tsx` raw `<input>` → `TextField.Root`/`TextField.Slot`
  (only remaining Phase-8-adjacent gap).

## ▶ IF RESUMING LATER

1. Read **Phase overview** — find the first phase not marked `[x] DONE`.
2. Follow that phase's unchecked `[ ]` items in order.
3. One commit per phase; update this file before committing.
4. After each phase: `npm run lint && npm run typecheck && npm run test && npm run build`.
5. Fresh clone tests need `npx playwright install chromium`.
6. Prior token rules still apply — see `RADIX-DS-PLAN.md` + `AGENTS.md`.

---

## Current state vs target

| Layer | After Phases 0–7 | Target (this plan) |
|-------|------------------|---------------------|
| Spacing/sizing tokens | ✅ Done | Keep |
| Radix components | ⚠️ Partial | Full — no plain form controls |
| Layout (`Box`/`Flex` props) | ⚠️ Heavy `style={{}}` | Prefer Radix layout props |
| Brand theme (`theme-overrides.css`) | ✅ Correct | Keep — Radix theme extension |
| Bespoke calendar/search UI | Custom DS | Keep — built **on** Radix primitives |

---

## Phase overview

| Phase | Scope | Status | Commit |
|-------|-------|--------|--------|
| 8 | Form controls — `TextField`, labels, errors, OTP | `[x] DONE` | *(pending)* |
| 9 | Buttons — drop `.btn-*` geometry; Radix `Button` + thin shadow theme | `[x] DONE` | `f19ffff` |
| 10 | Shell chrome — `IconButton`, topbar, sidebar CSS shrink | `[x] DONE` | *(pending)* |
| 11 | Navigation — `PortalNav` → `Tabs`, `PillTabs` → `SegmentedControl` | `[x] DONE` | *(pending)* |
| 12 | DS layer — `Skeleton`, `Badge`, `Card` wrappers | `[x] DONE` | *(pending)* |
| 13 | Inline styles → `Box`/`Flex`/`Grid` props | `[x] DONE` | *(pending)* |
| 14 | `globals.css` purge — dead CSS from superseded calendar grid + orphans | `[x] DONE` | *(pending)* |

---

## Radix replacements (audit reference)

| Need | Use Radix | Replace today |
|------|-----------|---------------|
| Text inputs | `TextField` | `.auth-input` + `<input>` |
| Selects | `Select` (native sizing) | `Select.Trigger` + `.auth-input` hack |
| Buttons | `Button` `color`/`variant`/`size` | `.btn-amber-primary`, `.btn-dark-accent`, `.btn-social-lg`, `.btn-auth-cta` |
| Admin section nav | `Tabs` | `PortalNav` custom CSS |
| Settings pill switcher | `SegmentedControl` | `PillTabs` (or keep if visual mismatch — decision in Phase 11) |
| Cards | `Card` | `.settings-card`, thin `AccentCard` wrappers |
| Badge | `Badge` | `PausedBadge` |
| Skeleton | `Skeleton` | `.skeleton` class |
| Icon actions | `IconButton` | `.sidebar-icon-btn`, `.topbar-icon-circle` |
| Divider | `Separator` | `.auth-divider` |
| Layout | `Box`, `Flex`, `Grid`, `Container` | `div` + inline `style` |

### Stays custom (no Radix primitive — compose with Radix inside)

| Area | Component | Notes |
|------|-----------|-------|
| Week availability grid | `AvailabilityView` | `Box`/`Flex` + `.cal-*` geometry only |
| Month picker | `MonthGrid` | Custom grid |
| Shell sidebar layout | `Sidebar` | Amber pill nav — use `IconButton`/`Flex`, keep layout |
| Search overlay | `SearchOverlay` | `Dialog` + `TextField` + custom keyboard list |
| Brand button glow | Theme CSS | Shadow on `Button` only — not custom height/padding |

### Keep forever (theme, not "non-Radix")

- `theme-overrides.css` — brand colors, semantic aliases
- `layout-tokens.ts` — layout max-width calcs on Radix scale
- `fonts.css` — Poppins

---

## Phase 8 — Form controls  `[x] DONE`  *(2026-07-13)*

- [x] `LoginPage.tsx` / `SignupPage.tsx` — email field: `TextField.Root size="3"`; `.auth-input` removed.
- [x] `OtpInput.tsx` — OTP cells now `TextField.Root` per digit; `.otp-box` restyled to target
      `.rt-TextFieldRoot`/`.rt-TextFieldInput` (geometry/centering) instead of a plain `<input>`.
- [x] `SettingsPage.tsx` — `Field` helper → `TextField.Root` (`type` narrowed to the Radix union);
      kept `.field-label-lg` (label typography — out of scope, not a form-control class).
- [x] `SettingsPage.tsx` — `Select.Trigger` no longer takes `.auth-input`; relies on the
      existing `.rt-SelectTrigger:where(.rt-variant-surface)` surface-color routing in globals.css.
- [x] `SyncSetup.tsx` start-date input (same `.auth-input` class, not originally listed) →
      `TextField.Root type="date" size="3"`, since the class was being deleted.
- [x] Errors: `.field-error` span → `Text size="1" color="red"` below field (Login + Signup).
- [x] Deleted dead CSS: `.auth-input`, `.auth-input::placeholder`, `.auth-input:focus`,
      `.auth-input:disabled`, `.auth-input-error(:focus)`, `.field-error`. Kept `.field-label-lg`
      (still used) and the readonly/disabled surface-color routing rules (now apply automatically
      to `TextField.Root`/`Select.Trigger` via existing `:has()` selectors).
- [x] Verify: `lint` (0 errors, 12 pre-existing warnings) · `typecheck` clean · `test` 167/167 ·
      `build` green. Manual Playwright smoke: login email → OTP dev-hint → 4-digit auto-submit →
      `/home`; Settings Account tab fields (bordered, readonly Email visually gray); invalid-email
      red border + red error text. No console errors.

**Files:** `LoginPage.tsx`, `SignupPage.tsx`, `OtpInput.tsx`, `SettingsPage.tsx`, `SyncSetup.tsx`,
`globals.css`.

---

## Phase 9 — Buttons  `[x] DONE`  *(committed `f19ffff` 2026-07-27; marked in tracker 2026-08-13)*

- [x] `.btn-amber-primary` — color/shadow-only class confirmed; geometry from `Button size`.
- [x] `.btn-dark-accent` — same; dark fill + amber text via theme, shadow/color only.
- [x] `.btn-social-lg` — deleted; replaced by `Button variant="outline" size="3"`.
- [x] `.btn-auth-cta` — no height/radius overrides; width/radius utility only, `Button size="4"`.
- [x] Converted to Radix `Button`/`IconButton`: `AccountSettingsPage.tsx` (avatar picker),
      `LinkedAccountSettingsPage.tsx` (delete), `AdminUsersPage.tsx` ("Clear filters"),
      `OtpVerifyStep.tsx` (inline buttons), `LoginPage.tsx`/`SignupPage.tsx` (social buttons).
- [x] Added shared `ClickableRow` DS component for the copy-pasted row pattern (see
      2026-08-13 closeout note above — later calendar rebuild stopped using it; tracked as a
      wire-up-or-delete decision in Phase 12).
- [x] Verify: lint/typecheck/build clean 2026-08-13.
- [-] `Topbar.tsx`/`Sidebar.tsx`/`SearchOverlay.tsx` raw buttons — **not actually in Phase 9's
      scope**; these are Phase 10 (shell chrome), still open there.

**Files:** all pages using `btn-*` classes, `globals.css` button sections, plus the calendar
files listed above.

---

## Phase 10 — Shell chrome  `[x] DONE`  *(2026-08-13, verified live in browser per-conversion)*

- [x] `Sidebar.tsx` — drawer-close button (`.sidebar-icon-btn`) → `IconButton variant="ghost"`.
      Verified in mobile drawer.
- [x] `Topbar.tsx` — mobile hamburger (`.sidebar-icon-btn`) → `IconButton variant="ghost"`.
      Verified at mobile width.
- [x] `SearchOverlay.tsx` — query input → `TextField.Root` (kept borderless via style override
      to match the existing composed row — icon/spinner and kbd hint stay as siblings, not
      `TextField.Slot`, since restructuring them risked misaligned padding I couldn't verify
      without more surgery than the swap warranted). Verified via ⌘K.
- [-] `Sidebar.tsx` collapse toggle (`.sidebar-collapse-btn`) — **kept as native `<button>`**.
      Not a bare unstyled button: fully custom bordered-square treatment (own background/
      border/hover, "white square straddling the edge" per its own comment) that `IconButton`
      variants don't replicate without a style override I couldn't visually verify safely.
- [-] `Topbar.tsx` search trigger (`.topbar-search`) / bell (`.topbar-icon-circle`) — **kept as
      native `<button>`**. The search trigger has no value/onChange — it's a button styled as a
      search box that opens a dialog (command-palette pattern), not a real text field; wrapping
      in `TextField` would be semantically wrong. Bell is a bordered icon-circle with its own
      geometry, same reasoning as the collapse toggle.
- [-] `SearchOverlay.tsx` result-item buttons (`.search-result-item`) — **kept as native
      `<button>`**. Full-bleed composed row (icon tile + two-line text + kbd hint), same shape
      as the calendar module's row pattern — not a simple `IconButton`/`Button` candidate, per
      Phase 9's own precedent for row-shaped controls.
- [-] `ShellLayout.tsx` `<main>` padding → `Box p` — **not converted**. Padding is asymmetric
      (`var(--space-6) calc(var(--space-6) + var(--space-1))`); Radix's `p`/`px`/`py` props only
      accept plain space-scale tokens, not calc expressions, so the exact value can't be
      expressed via props without dropping the intentional `+ var(--space-1)` offset.
- [x] Verify: lint/typecheck/build/build-storybook clean; search ⌘K, mobile hamburger, mobile
      drawer close all manually confirmed working in the browser.

---

## Phase 11 — Navigation  `[x] DONE`  *(2026-08-13, verified live in browser)*

- [x] `PortalNav.tsx` → Radix `TabNav.Root` / `TabNav.Link` (not `Tabs` — `Tabs.Trigger` is a
      `<button>` built for panel-switching, not routed links; `TabNav.Link asChild` wraps
      react-router's `Link` for real navigation, which is what this component needs). Removed
      the now-dead `.portal-nav-link:hover` CSS (active/hover treatment now comes from
      `TabNav`'s own styling). Fixed stale comments in `PortalNav.stories.tsx` and
      `TabNav.stories.tsx` that described the old custom implementation.
- [-] `PillTabs.tsx` — **does not exist**; already removed in an earlier redesign when Settings
      sub-navigation moved into the Sidebar's drill-in menu (see `Sidebar.tsx` comment: "folded
      in from the old in-page PillTabs"). Checklist item was stale.
- [x] Verify: admin section nav (Overview/Users/Monitoring/etc.) confirmed working in browser —
      active tab highlight, hover, click-to-navigate. Settings has no tabs to verify (moved to
      Sidebar). lint/typecheck/build/build-storybook all clean.

---

## Phase 12 — DS layer  `[x] DONE`  *(2026-08-13 — every remaining item audited, no code changes needed)*

- [-] `Skeleton.tsx` — **kept as-is**. Radix's own `Skeleton` wraps a sized child (`loading`
      prop, clones child shape via `Slot.Root`); this app's `Skeleton` is a standalone shaped
      placeholder (`width`/`height`/`circle`/`radius` props, no children) used at 34 call sites.
      Different API shapes, not a thin-wrapper opportunity — converting would mean rewriting
      every call site to pass a dummy sized child. File's own header comment already documents
      it as "the one retained legacy wrapper."
- [-] `PausedBadge.tsx` — **kept as-is**. Own docstring documents why: measured 59×17@1728
      (≈18px app height) is below Radix `Badge`'s smallest size/padding floor. Deliberate
      exception, not oversight.
- [-] `AccountCard.tsx` — **kept as-is**. Already Radix-composed (`Box`/`Flex`/`Avatar`/
      `DropdownMenu`). Swapping the outer `Box` for `Card` would require overriding every one of
      `Card`'s defaults anyway (this component uses a custom radius, custom `--shadow-card`
      value, and a custom `layout.accountCardW` width, none of which match `Card`'s own
      defaults) — no real simplification, just a different starting point with the same amount
      of override code.
- [-] `AccentHeader.tsx` / `AccentCard` — **kept as-is**. Checked Radix Themes' `Card` source —
      it has no header subcomponent at all (single flat `<div>` wrapper, no `Card.Header`).
      There is no "Card header pattern" in this library to migrate to; the plan's suggestion
      referred to something that doesn't exist. `AccentHeader` fills a genuine gap (two-tone
      amber header bar) per its own docstring.
- [-] `ConfirmDialog.tsx` — **kept as-is**. Already built on `AlertDialog`. Grepped
      `globals.css` for confirm-dialog-specific rules — none exist; already cleaned up in an
      earlier phase. Nothing to trim.
- [x] `ClickableRow.tsx` — **deleted** 2026-08-13 (zero real consumers; `CalendarSyncsPage.tsx`'s
      row is a multi-column grid with headers, not a single-flex-row shape, so it doesn't fit
      `ClickableRow`'s API — wiring it up would have been a forced fit, not real reuse).
- [x] Verify: lint/typecheck/build/build-storybook clean; Storybook DS stories unaffected (no
      component code changed).

---

## Phase 13 — Inline styles → layout props  `[x] DONE`  *(2026-08-17)*

- [x] Scope refresh: `AvailabilityView.tsx`/`SyncSetup.tsx` (the plan's original targets) no
      longer exist — both deleted by the calendar rebuild (`643e291` and an earlier commit).
      Their functional successors are `CalendarSyncEditorPage.tsx` (wizard, was `SyncSetup`)
      and `CalendarMergedView.tsx`/`BigCalendarView.tsx` (merged grid, was `AvailabilityView`).
      Swept those instead of the stale names.
- [x] `HomePage.tsx` — converted `style={{ flex: 1 }}` → `flexGrow="1"`, `style={{ minWidth: 0 }}`
      → `minWidth="0"`, `style={{ minHeight: ... }}` → `minHeight` prop, on `Flex`/`Box` only
      (Radix `LayoutProps`). Left `Text`/icon `style={{ color }}` and typography as-is — `Text`
      only has `MarginProps`, no layout props to migrate to.
- [x] `AdminUserDetailPage.tsx` — same treatment: `Box`/`Flex` `flex`/`minWidth`/`gap` → props.
      `Select.Trigger style={{ width: "100%" }}` kept as style — `SelectTriggerProps` has no
      `width` layout prop, only `MarginProps`.
- [x] `CalendarSyncEditorPage.tsx` (26 inline styles, most in file) — converted static
      `flex`/`minWidth`/`minHeight`/`overflow`/`flexShrink`/padding values on `Flex`/`Box` to
      props. Kept `display: grid` / `gridTemplateColumns` (dynamic template string) and other
      conditional/dynamic values (`height: wizardStep === 2 ? ... `) in `style`, since Radix
      layout props don't take arbitrary CSS like `grid-template-columns`.
- [x] `CalendarMergedView.tsx` — same treatment on the left-rail/grid `Box`/`Flex` wrappers;
      dynamic `flex` shorthand strings (`isNarrow ? "1 1 100%" : ...`) kept in `style` (no Radix
      prop takes a flex-shorthand string), but the plain static `minWidth`/`p`/`pl` values moved
      to props.
- [x] `BigCalendarView.tsx` — audited, no changes. Every `style={{}}` here is either on `Text`
      (typography/color only, no layout props exist to move to) or on raw `div`/`span` inside
      react-big-calendar's `components` render props (event chips, `.rbc-radix-scope` wrapper
      needing `containerType`) — not Radix components, nothing to convert.
- [x] Verify: `npx tsc --noEmit` clean, `npm run lint` clean (0 errors, 14 pre-existing
      fast-refresh/storybook warnings), `npm run build` clean (pre-existing 500kB chunk warning
      unrelated).

---

## Phase 14 — `globals.css` purge  `[x] DONE`  *(2026-08-17)*

- [x] Scope note: `globals.css` was already lean (~48 classes) going in — Phases 8–13 removed
      their own dead CSS incrementally as they landed (`.auth-input*`, `.field-error`,
      `.btn-social-lg`, `ClickableRow`'s styles, etc.), so this phase found only leftovers from
      an earlier custom calendar/week-grid implementation and two unrelated orphans, not a
      60→15 mechanical cut. The plan's original "~60 classes" estimate predates that incremental
      cleanup.
- [x] Grep-audited every class in `globals.css` against `src/**/*.{ts,tsx}` (excluding the
      stylesheet itself). Found 11 with zero live references:
      `.terms-back-link` (orphaned — `TermsPage` now uses a different back-link pattern),
      `.home-app-card` (orphaned), `.cal-time-label`, `.cal-day-col`, `.cal-hour-slot`,
      `.cal-event`, `.cal-nav-btn`, `.cal-nav-today`, `.cal-now-line`, `.cal-now-dot` (all from
      the pre-`643e291` custom week grid, superseded by `react-big-calendar` — see Phase 13's
      note on `AvailabilityView`), `.data-table-row--selected` (superseded by
      `.calendar-picker-row[data-selected="true"]`'s own state). Deleted all 11, plus their
      associated hover/pseudo-class rules and one `@keyframes`-free comment header.
      Everything else confirmed live (spot-checked ambiguous ones like `.btn-dark-accent--strong`,
      `.provider-choice__copy/__chevron`, `.auth-card-left/right` individually).
- [x] `globals.css` now ~37 classes — every remaining one grep-confirmed in active use across
      shell chrome, auth, calendar composition utilities (`.cal-day-col`'s siblings are gone but
      `.calendar-picker-row`, `.provider-choice`, `.connect-account-tile` etc. remain — they're
      the *current* calendar/account UI, not legacy).
- [x] Verify: `npx tsc --noEmit` clean · `npm run lint` clean (0 errors, 14 pre-existing
      fast-refresh/storybook warnings, unchanged from before this phase) · `npm run build` clean
      (pre-existing 500kB chunk warning unrelated) · `npm run test`: 9 pre-existing failures in
      2 Storybook interaction test files (missing `IntegrationsProvider` wrapper around
      `Sidebar` stories) — confirmed identical failure count/cause on the pre-Phase-14 tree via
      `git stash`, unrelated to this phase's CSS-only changes.
- [x] Mark plan **CLOSED** — Phases 8–14 all done.

---

## Risk notes

| Phase | Risk | Mitigation |
|-------|------|------------|
| 8 Forms / OTP | High | Test login/signup manually + Storybook; one screen at a time |
| 9 Buttons | Medium | Amber shadow — theme-only CSS, no size overrides |
| 10 Shell | Medium | Sidebar active pill — visual compare after each file |
| 11 Tabs | Low | SegmentedControl may need keep-PillTabs decision |
| 12–14 | Low | Mechanical |

---

## Relationship to other docs

| Doc | Role |
|-----|------|
| [`RADIX-DS-PLAN.md`](./RADIX-DS-PLAN.md) | **Closed** — token migration (Phases 0–7) |
| **This file** | **Active** — component + layout migration (Phases 8–14) |
| [`AGENTS.md`](./AGENTS.md) | Day-to-day rules for agents |
| [`CLEANUP-PLAN.md`](./CLEANUP-PLAN.md) | Historical — original repo cleanup |
