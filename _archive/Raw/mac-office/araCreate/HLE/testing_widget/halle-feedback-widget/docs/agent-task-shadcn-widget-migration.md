# Task brief — Migrate the widget to React + shadcn/ui with our brand tokens

**Status: queued, not yet handed to the dev agent. Vishnu approved this
21 Sept 2026, having been told plainly that this drops the widget's
15KB size rule (React + shadcn alone is 69-116 KB before our own code —
see `docs/agent-task-shadcn-admin-migration.md`'s companion decision notes
and the Claude project's `claude/decision-admin-shadcn-rebuild.md` for the
full numbers and what is knowingly being given up). Do this alongside the
admin migration (`docs/agent-task-shadcn-admin-migration.md`), same brand
tokens.**

## Goal

Rebuild the widget's UI (the button, panel, and screens a tester sees) in
React using shadcn/ui components, themed with our brand tokens, instead of
today's plain JavaScript/hand-built DOM and CSS. This replaces the
"vanilla JS, no framework, under 15KB" approach with a shared component
system across the whole project.

## What is changing vs. what is not

**Changing:** the framework and component library used to build the
widget's UI (plain JS/DOM → React + shadcn), and the widget's total size
(under 15KB → 100KB+, accepted trade-off).

**NOT changing, still hard rules — verify these explicitly hold once
rebuilt, do not assume the migration preserves them automatically:**

- **Shadow DOM isolation.** The widget must still render inside a Shadow
  DOM so the client site's CSS cannot reach in and the widget's CSS cannot
  leak out onto their page. React can render into a Shadow root, but this
  needs deliberate setup (styles injected inside the shadow root, not just
  a global `<style>` tag) — check this against a hostile host page the same
  way `docs/BUILD-SPEC.md`'s isolation test did before.
- **Fail silently.** Every entry point still wrapped so a widget error
  never breaks the client's page, and declares no globals except one
  namespaced object.
- **Accessibility floors:** 16px minimum font size, 44px minimum tap
  target (56px for the widget's own big option buttons), 4.5:1 contrast —
  same as before, `computed-styles.spec.ts` (or its equivalent after the
  rebuild) should still check these.
- **Token-gated launcher**, fails closed and silently on an invalid token,
  shows the expired-link notice on an expired one — unchanged behaviour.
- **No live-page freehand drawing** — pen still freezes a full-size
  picture first, matching the industry pattern already researched
  (`docs/research-how-the-industry-solved-this.md`). This task is a UI
  rebuild, not a UX change — don't reopen this decision.
- **English only, five fixed answer sentences, no keyboard path to element
  selection** — all unchanged product decisions, not part of this task.

## Scope — OUT, do not touch as part of this task

- **The screenshot capture pipeline itself** (`capture_screenshot`, the
  modern-screenshot integration, and anything inside the currently-broken
  live path described in `docs/session-handover.md`'s top banner —
  blank/mostly-empty images, 8-10s capture time, stuck "Loading"). That is
  a separate, already-written-up bug, queued right after this migration.
  If the React rebuild happens to touch the same files, be careful not to
  fix or mask that bug as a side effect — leave the underlying capture
  logic's behaviour as-is, migrate only the surrounding UI shell.
- The element-picker logic (`@medv/finder` or equivalent) and marker-pen
  drawing logic itself — these can be wrapped in React, but their behaviour
  is not part of this task. The already-queued marker-pen fixes
  (`docs/agent-task-marker-and-capture-speed.md` Part A) come after this.
- The backend API, database, and admin dashboard (separate task brief:
  `docs/agent-task-shadcn-admin-migration.md`).
- Widget hosting/deployment mechanics (still a versioned `/v1.js` on the
  same CDN/object storage, still built via the existing pipeline — only
  swap in whatever build step React/shadcn needs, e.g. bundling JSX).

## Brand tokens to wire in (same as the admin task, source:
## `docs/halle-design-system-draft.md`)

| Token | Value | Use |
|---|---|---|
| Navy (primary) | `#29308A` | Launcher/panel background, primary buttons |
| Text | `#2A2924` | Body text |
| White | `#FFFFFF` | Backgrounds, text on navy |
| Border Gray | `#EBEBEB` | Input/panel borders |
| Error | `#D93025` | Marker pen stroke, element outline box |
| Font | Helvetica Neue, system-ui fallback (no external font load — client machines may not have it) | All widget text |
| Radius | 8px (inputs), 12px (panels/buttons) | Matches current widget |
| Spacing | 8 · 16 · 24 (px) | Matches current widget's applied scale |

## Suggested approach

1. Confirm the build pipeline: the widget currently builds via esbuild into
   one file (`WIDGET_API_ORIGIN=... npm run build --workspace
   halle-feedback-widget-embed`). Decide whether esbuild can still bundle
   React/JSX directly (it can, with the right loader config) or whether a
   different bundler step is needed — keep the "one versioned `/v1.js`
   file, no separate app shell" delivery model.
2. Set up React to mount inside the Shadow DOM root, with shadcn's
   Tailwind-based styles scoped to that shadow root only (this is the part
   most likely to go wrong — verify carefully against a test host page with
   aggressive global CSS, the same kind of test used originally).
3. Rebuild each of the six widget screens (launcher, mode menu, pointer
   overlay, review screen, sent confirmation, expired-link notice) as React
   components using shadcn primitives (buttons, dialogs/sheets, etc.)
   themed with the tokens above.
4. Re-run the existing widget test suite (`computed-styles.spec.ts` and the
   rest of the 34 widget tests referenced in
   `docs/halle-design-system-draft.md`'s status block) and fix or rewrite
   any that assumed the old DOM structure.
5. Re-verify Shadow DOM isolation against a hostile host page — this was
   tested before (`docs/BUILD-SPEC.md`) and must still pass.
6. Measure and report the real final `/v1.js` size after this change, so
   it's on record rather than assumed from the library-only numbers above.

## Acceptance

- All six widget screens work end to end (button → point at element →
  freeze picture → draw → review → send), rebuilt in React/shadcn, visually
  matching the brand tokens.
- Shadow DOM isolation confirmed still holds.
- Accessibility floors (font size, tap target, contrast) still pass.
- Existing widget test suite green (updated as needed for the new
  structure, not deleted to make it pass).
- Final built `/v1.js` size measured and reported to Vishnu — expected to
  be well over the old 15KB budget; this is expected and accepted, not a
  failure, but Vishnu should see the real number once it exists.
- The screenshot capture bug (top banner in `docs/session-handover.md`)
  is untouched by this task — still open, still next in line after this.

## After this task

The live screenshot bug is next (see `docs/session-handover.md`). Do not
start that work as part of this task.
