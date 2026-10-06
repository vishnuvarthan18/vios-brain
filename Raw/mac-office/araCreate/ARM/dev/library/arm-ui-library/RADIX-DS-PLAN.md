# Radix-native design system plan — canonical tracker

> Working document for migrating arm-ui to **Radix tokens only** (audited 2026-07-10).
> Status markers: `[ ]` todo · `[~]` in progress · `[x]` done · `[!]` blocked · `[-]` dropped.
> One commit per phase so each is revertable. Update statuses **in place** as phases land.

## Design direction (user-confirmed 2026-07-10)

| Principle | Rule |
|-----------|------|
| **Source of truth** | Radix Themes scale — `size` props, `var(--space-*)`, `var(--radius-*)`, `var(--font-size-*)`, semantic color tokens |
| **Figma role** | Visual reference for look & feel only — **not** a measurement spec. No ÷1.2, no export PNG pixel diffing |
| **Snapping** | When replacing a raw px value, use the **nearest** Radix step. Small drift is OK |
| **Guardrail** | Reject any swap that moves more than **one space step** (~4–8px) unless the user explicitly approves |
| **Buttons** | Geometry from Radix `size` prop only; CSS classes = color/shadow treatment (`.btn-amber-primary`, `.btn-dark-accent`) |
| **Typography** | Radix `size` on Text/Heading; never raw `font-size` in components or new CSS |
| **New CSS** | No new bespoke px classes. If a pattern repeats, tokenize with `var(--space-*)` / `var(--radius-*)` |

### Allowed exceptions (not converted)

- `1px` borders and hairlines
- `2px` focus outlines and `outline-offset`
- `box-shadow` / `rgba()` / `blur()` values (not on the space scale)
- Percentages, `100vh`, `100%`, `max-width` breakpoints
- SVG `viewBox` and icon intrinsic dimensions where Radix has no equivalent

---

## ✅ PLAN CLOSED — 2026-07-10 (reopened Phase 7, closed again same day)

All phases done and committed. Reopen only if new UI work adds raw px outside allowed exceptions.

## ▶ NEXT ACTION

**None** — plan closed. Token work complete (Phases 0–7).

**Next work:** [`RADIX-FULL-PLAN.md`](./RADIX-FULL-PLAN.md) — Radix **component** migration (Phases 8–14).

## ▶ IF RESUMING LATER

1. Read this file's **Phase overview** table — find the first phase not marked `[x] DONE`.
2. Jump to that phase's section and follow its unchecked `[ ]` items in order.
3. Do **not** re-ask decisions in the **Decisions** table (all ✅ are final).
4. One commit per phase; update this file's overview table + phase checkboxes before committing.
5. After each phase: `npm run lint && npm run typecheck && npm run build` (Phase 6 also runs `npm run test`).
6. Browser tests need `npx playwright install chromium` on a fresh clone.

---

## Phase overview

| Phase | Scope | Status | Commit |
|-------|-------|--------|--------|
| 0 | Close cleanup plan + rewrite AGENTS.md rules | `[x] DONE` | `6cd6c8c` |
| 1 | `globals.css` — Figma-measured classes → Radix tokens | `[x] DONE` | `6403c63` |
| 2 | Shell components — SearchOverlay, Sidebar, Topbar, PortalNav | `[x] DONE` | `afdf013` |
| 3 | Auth — AuthCard, OtpInput, inputs, social buttons, BrandPanel | `[x] DONE` | `9be2f01` |
| 4 | Calendar — AvailabilityView, MonthGrid, SyncSetup, AccountManager | `[x] DONE` | `ba5674b` |
| 5 | Pages — admin tables, Home, Settings, Terms, OAuth | `[x] DONE` | `9cf7262` |
| 6 | Stale comments + final gate | `[x] DONE` | `301aa6a` |
| 7 | Remaining px → layout tokens + missed spots | `[x] DONE` | *(this commit)* |

**Out of scope (unchanged from cleanup):** `React.lazy` route splitting · CI wiring · Figma PNG screenshot diffing.

---

## Decisions

| # | Question | Decision |
|---|----------|----------|
| ① | Old cleanup Phase 2 (revert snap sites to Figma px) | ✅ **Dropped** — user wants Radix-only, not Figma px (2026-07-10) |
| ② | Visual verification method | ✅ **Look-and-feel smoke pass** in browser/Storybook — not pixel diff vs Figma exports |
| ③ | Auth CTA height (52px today) | **Default:** `size="4"` button (48px) or `var(--space-8)` — nearest step, 4px shorter. Flag if it feels wrong in Phase 3 |
| ④ | OTP boxes (65×65 today) | **Default:** `var(--space-9)` (64px) — 1px drift, already proven in snap pass |
| ⑤ | Topbar search width (267px today) | **Default:** `width: 100%; max-width: var(--space-9)` or flex-based — drop fixed 267px |
| ⑥ | `.metric-card-value` (28px today) | **Default:** Heading `size="6"` or `var(--font-size-6)` — check Radix step map in Phase 1 |

---

## Phase 0 — Rules & cleanup close  `[x] DONE`  *(2026-07-10, commit 6cd6c8c)*

- [x] Update `CLEANUP-PLAN.md`: decision ② → **bless Radix tokens**; Phase 2 → `[-] DROPPED`; visual gate → `[-] DROPPED` (Figma px diff not our verify method).
- [x] Rewrite `AGENTS.md` § Design system rules:
  - Remove Figma ÷1.2 measurement requirement and PNG screenshot diff verify step.
  - Add Radix-only principles from the table above.
  - Keep typography table, button size conventions, DS component list.
  - Point agents at this file (`RADIX-DS-PLAN.md`) for migration work.
- [x] Verify: `npm run lint && npm run typecheck` (no code changes expected).

## Phase 1 — `globals.css` tokenization  `[x] DONE`  *(2026-07-10, commit 6403c63)*

Convert Figma-measured bespoke classes to Radix tokens. **Do not change color/shadow treatment** — geometry only.

| Class / area | Current px | Target (nearest Radix) | Notes |
|--------------|-----------|------------------------|-------|
| `.sidebar-icon-btn` | 28×28 | `var(--space-5)` (24) or `var(--space-6)` (32) | Pick closer feel; 28→24 is 4px |
| `.sidebar-collapse-btn` | 24×24 | `var(--space-5)` | exact match |
| `.topbar-search` | h 40 ✓, w 267, min-w 150 | h keep `var(--space-7)`, w → flex/max-width tokens | decision ⑤ |
| `.auth-input` | h 48, pad 0 16, r 9 | h `var(--space-8)`, pad `0 var(--space-4)`, r `var(--radius-3)` | consider Radix TextField size instead |
| `.btn-social-lg` | h 44, gap 10, r 8 | h `var(--space-7)` or `var(--space-8)`, gap `var(--space-3)`, r `var(--radius-2)` | |
| `.otp-box` | 65×65, r 17 | `var(--space-9)`, r `var(--radius-4)` or `var(--radius-5)` | decision ④ |
| `.btn-auth-cta` | h 52, r 9 | Radix `size="4"` + drop height override | decision ③ |
| `.settings-card` | pad 40, mt 18 | pad `var(--space-8)` or `var(--space-9)`, mt `var(--space-5)` | |
| `.settings-avatar-picker` | 74×74 | `var(--space-9)` (64) — 10px is >1 step; **flag user** or use `calc(var(--space-9) + var(--space-2))` = 72 | needs judgment in implementation |
| `.linked-account-row` | r 14, pad 10 14 | r `var(--radius-4)`, pad `var(--space-3) var(--space-4)` | |
| `.linked-account-delete` | 47×47 | `var(--space-8)` (48) | 1px drift |
| `.metric-card-value` | 28px font | `var(--font-size-6)` or Heading size | decision ⑥ |
| `.terms-back-link` | 15px font | `size="3"` Text | |
| `.auth-divider` | gap 12, margin 24 | `var(--space-3)`, `var(--space-6)` | |
| `.field-label-lg` | margin-bottom 8 | `var(--space-2)` | |
| Mobile `.auth-card-right` | pad 40 28 | `var(--space-8) var(--space-6)` | |

- [x] Strip Figma measurement comments; replace with brief token rationale where non-obvious.
- [x] Verify: lint + typecheck + build.

---

## Phase 2 — Shell components  `[x] DONE`  *(2026-07-10, commit afdf013)*

- [x] `SearchOverlay.tsx` — replace inline px with `var(--space-*)`.
- [x] `Sidebar.tsx` — nav layout constants → Radix tokens.
- [x] `Topbar.tsx` — padding/gap → tokens; remove Figma comments.
- [x] `PortalNav.tsx` — tab padding/margin → tokens.
- [x] Verify: lint + typecheck.

## Phase 3 — Auth  `[x] DONE`  *(2026-07-10, commit 9be2f01)*

- [x] `AuthCard.tsx`, `OtpInput.tsx`, `LoginPage.tsx`, `SignupPage.tsx`, icon components.
- [x] Verify: login + signup OTP flow (Storybook/browser tests).

## Phase 4 — Calendar  `[x] DONE`  *(2026-07-10, commit ba5674b)*

- [x] `AvailabilityView.tsx`, `MonthGrid.tsx`, `SyncSetup.tsx`, `SyncStatus.tsx`, `AccountManager.tsx`, `CalendarEmpty.tsx`.
- [x] Verify: calendar stories pass.

## Phase 5 — Pages  `[x] DONE`  *(2026-07-10, commit 9cf7262)*

- [x] Admin pages, `HomePage.tsx`, `SettingsPage.tsx`, `TermsPage.tsx`, `CalendarOAuthPage.tsx`.
- [x] DS: `PausedBadge`, `ConfirmDialog`, `AccountCard`, `AccentHeader`, `PillTabs`.
- [x] Verify: spot-check admin users + home + settings.

## Phase 6 — Final gate  `[x] DONE`  *(2026-07-10)*

- [x] Grep audit: no `÷1.2` / `snapped to nearest` stale comments remain in `src/`.
- [x] Remaining raw px are allowed exceptions (1px borders, shadows, dialog max-widths, layout constants).
- [x] `npm run lint && npm run typecheck && npm run test && npm run build` all green (167/167 tests).
- [x] Look-and-feel: Storybook browser suite covers auth, shell, calendar, admin flows.
- [x] Plan **CLOSED**.

## Phase 7 — Strict token cleanup  `[x] DONE`  *(2026-07-10)*

- [x] Add `src/lib/layout-tokens.ts` — shared `calc(var(--space-*) …)` layout widths.
- [x] Convert missed spots: `OtpVerifyStep`, `ShellLayout`, `PausedBadge`, Settings `FIELD_W`/`COL_GAP`, dialog max-widths, grid columns, account card sizes, sync layout flex mins.
- [x] Verify: lint + typecheck + test (167/167) + build green.

---

## Token quick-reference (Radix Themes default scale)

Use this when picking "nearest step":

| Token | px |
|-------|-----|
| `--space-1` | 4 |
| `--space-2` | 8 |
| `--space-3` | 12 |
| `--space-4` | 16 |
| `--space-5` | 24 |
| `--space-6` | 32 |
| `--space-7` | 40 |
| `--space-8` | 48 |
| `--space-9` | 64 |

**Snapping rule:** `|raw − token| ≤ 4px` → snap freely. Between two steps → pick visually closer. If both are >4px away from raw → stop and flag (e.g. 74px avatar: 64 vs 72 vs 80).

---

## Relationship to `CLEANUP-PLAN.md`

The cleanup plan (Phases 0–6) is **done**. This document superseded its blocked Phase 2 and
optional Figma visual gate. Keep `CLEANUP-PLAN.md` as historical record.

**Token migration (Phases 0–7):** closed — see phase table above.

**Component migration (Phases 8–14):** tracked in [`RADIX-FULL-PLAN.md`](./RADIX-FULL-PLAN.md).
