# Task brief — Migrate the admin dashboard to React + shadcn/ui with our brand tokens

**Status: queued, not yet handed to the dev agent. Vishnu approved this
21 Sept 2026 and chose to do it BEFORE the live screenshot bug fix (a
deliberate, confirmed reversal of the usual order — see
`claude/decision-admin-shadcn-rebuild.md` in the Claude project, mirrored
here as context for the agent).**

## Goal

Rebuild the admin dashboard's UI using shadcn/ui components, themed with our
own brand colors, fonts and spacing — instead of the current hand-built,
one-off CSS. This should make every future admin screen change faster and
more consistent, and give us a proper base of accessible, well-tested
components (buttons, tables, dialogs, dropdowns, forms, inputs, badges,
tabs) instead of writing each one from scratch.

## Scope — IN

- Everything under the admin app (`src/web/app`): login, Overview, Queue,
  Tracked items, All reports, Report detail, Pages, Page detail, Testers,
  Wording, and the sidebar/nav shell.
- Installing and configuring shadcn/ui + Tailwind CSS in the existing
  Next.js app.
- Defining our brand tokens (colors, fonts, spacing, radius — see table
  below) as the theme shadcn/Tailwind use, so every component picks them up
  automatically rather than needing per-component overrides.
- Replacing existing hand-built UI pieces (buttons, tables, status badges,
  form inputs, dialogs/modals, dropdowns) with shadcn equivalents wired to
  our tokens.

## Scope — OUT, do not touch

- **The widget** (`src/widget/`) — being migrated separately, see
  `docs/agent-task-shadcn-widget-migration.md`. Do not mix the two tasks up
  or share incomplete work between them.
- **The screenshot capture pipeline** (`capture_screenshot`, the modern-
  screenshot integration, the live bug described in
  `docs/session-handover.md`'s top banner). Do not fix, touch, or
  refactor anything in the capture path as part of this task — that is a
  separate, already-written-up bug, coming right after this task.
- The database schema, API routes' actual logic/behaviour (only their
  rendering/UI may change), and auth logic.
- Any backend/server deployment config, runbook.md, or the production
  server itself. This is a code-only, dev-branch task until Vishnu says
  otherwise.

## Brand tokens to wire in (source: `docs/halle-design-system-draft.md`,
## already applied once to the admin's plain CSS — carry the same values
## into the new shadcn theme, don't reinvent them)

| Token | Value | Use |
|---|---|---|
| Navy (primary) | `#29308A` | Primary buttons, nav, headings, links |
| Text | `#2A2924` | Body text |
| White | `#FFFFFF` | Backgrounds, text on navy |
| Border Gray | `#EBEBEB` | Input/table/card borders |
| Light Blue | `#B5E0FA` | Highlight / "has activity" accents |
| Pale Blue | `#D3EDFC` | Secondary accent |
| Success | `#1B7A34` (darkened from `#1E8E3E` for 4.5:1 contrast — do not use the lighter value) | "Fixed" status, confirmations |
| Error | `#D93025` | "Bug" status, form errors |
| Font | Helvetica Neue, falling back to system-ui stack (same as today — no new font loading) | All admin text |
| Radius | 8px (inputs), 12px (cards/buttons/panels) | Matches current admin |
| Spacing scale | 8 · 16 · 24 · 32 · 40 · 48 (px) | Matches current admin |

Keep the existing status-badge mapping: Bug = Error, Fixed = Success,
Closed/Deleted = neutral gray.

## Suggested approach

1. Install Tailwind CSS + shadcn/ui into `src/web` following shadcn's
   standard Next.js App Router setup.
2. Set up the Tailwind theme / CSS variables using the token table above —
   this is the step that makes every shadcn component "ours" instead of
   shadcn's stock look, so get it right before styling individual screens.
3. Migrate screen by screen, starting with the ones easiest to verify
   (Login, then Overview), ending with the ones with the most custom
   markup (Queue, Report detail — these have the report grid and image
   viewer).
4. Keep the existing `status-badge` concept but rebuild it as a shadcn
   `Badge` variant using the tokens above.
5. Do not change any screen's actual information or behaviour — this is a
   visual/component-library migration, not a redesign of what each screen
   shows. (The separate "16 screens" pass, still queued after this, is
   where actual screen-by-screen redesign happens.)
6. Run the existing test suite and the visual/gate checks after each
   screen, the same way past admin UI passes in this project were verified
   (lint/build/test/size, then a visual check per screen).

## Acceptance

- All ten admin screens + sidebar render using shadcn components themed
  with our brand tokens — no shadcn default blue/gray anywhere.
- Existing test suite still green.
- No change to the widget, the capture pipeline, the database, or backend
  logic.
- Vishnu can see and confirm each screen looks right, screen by screen,
  the same way past UI passes in this project were reviewed.

## After this task

The live screenshot bug (`docs/session-handover.md` top banner) is next.
Do not start that work as part of this task, and do not let this task
block on it — they are sequenced back-to-back, not in parallel.
