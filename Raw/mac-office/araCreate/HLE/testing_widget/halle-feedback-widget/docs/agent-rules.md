# AGENT RULES

**Read this at the start of every session, before touching code.**

For whoever writes the code, human or AI. Most of the rules below look
arbitrary. None of them are — each one is the outcome of research or of a
decision already taken. If a rule seems wrong, raise it. Do not route around it.

**Revised 7 September 2026** — scope locked. English only, no keyboard path.
See [build-plan.md §9](build-plan.md#9-decisions-on-record) for what that costs.

**Amended 21 September 2026** — §1.4 and §1.9 are overridden for the widget's
React/shadcn rebuild. This was Vishnu's deliberate decision, taken *after*
seeing what the same migration actually cost in the admin dashboard, not an
oversight and not the agent routing around a rule. See
[§1.4](#1-never) and [§1.9](#1-never) for what replaced them, and
§5 below for the decision and its price.

**Amended 21 September 2026 (later)** — the standing "no server upgrade,
ever, for this test" rule is overridden. System libraries a real headless
browser needs are now installed on the production server. See §5 below.

| Part | Covers |
| --- | --- |
| [1 Never](#1-never) | The twelve hard stops |
| [2 Always](#2-always) | The seven habits |
| [3 Out of scope](#3-out-of-scope) | What not to build, however tempting |
| [4 Working method](#4-working-method) | Milestones, commits, dependencies |
| [5 Overrides on record](#5-overrides-on-record) | Rules deliberately set aside, and why |

## 1 Never

1. **Never update or delete a row in `reports`.** Append-only means append-only.
   That table is the client's sign-off evidence. Resolution state belongs in
   `issues`.
2. **Never put a client-specific value in the widget** — no wording, no colour,
   no page list, no URL. It carries a public key and fetches its own
   configuration. This still holds now that there is only one language: the
   strings come from the API, not from the source.
3. **Never use `localStorage`, `sessionStorage` or cookies for the tester
   token.** German §25 TDDDG treats device storage as needing consent
   *regardless of whether the data is personal*. Writing anything to the device
   forces a cookie banner onto the client's site. The token lives in the URL
   query string and in a closure, nowhere else.
4. ~~**Never add a dependency to `src/widget/`.** Zero, permanently.~~
   **Overridden 21 September 2026 for the React/shadcn rebuild** (see §5).
   React, shadcn/ui and Tailwind may be added to `src/widget/`, and
   `modern-screenshot` remains in place for the screenshot module. Every
   *other* dependency still needs asking about first — the override is for
   the named rebuild, not a general opening of the door.
5. **Never use `html2canvas`** (no release in four years, its own README says
   not for production, cannot render `box-shadow` or `object-fit` — product
   photos come out stretched) **or `getDisplayMedia`**, or any API that throws a
   browser permission dialog. The dialog cannot be styled or skipped, and it
   ends the session for an elderly tester.
6. **Never load an external font, and never ask the host site for a CSP
   concession.** All three competitors fail at least one of these. We demand
   nothing from the host.
7. **Never write a query without an org and project scope.** One helper
   function, used everywhere. If a query *can* be written without a tenant
   scope, the helper is wrong.
8. **Never reword a tester-facing string.** Every one was chosen against
   research on elderly and non-native readers. "Too small or too faint" is about
   size and must stay distinct from "I did not understand the words", which is
   about meaning. The copy is the product.
9. ~~**Never put a CSS framework in the widget**~~, and **never let CSS cross
   the Shadow DOM boundary in either direction**. Webflow ships one large
   global stylesheet with aggressive element selectors; without encapsulation
   the widget breaks.

   **Overridden in part, 21 September 2026** (see §5): Tailwind may now be
   used in the widget. The Shadow DOM half of this rule is **not** overridden
   and is now the load-bearing one — Tailwind's styles must be injected
   *inside* the shadow root, never as a global `<style>` tag, and isolation
   must be re-proven against a hostile host page rather than assumed.
10. **Never declare a global** other than one namespaced object.
11. **Never show a tester an error.** Fail silently, retry in memory, and show
    the thank-you screen anyway. A failed report is our problem, not theirs.
12. **Never group issues destructively.** Every report stays visible under its
    issue, with its own screenshot.

## 2 Always

1. **Wrap every widget entry point in `try/catch` and fail silent.** The client
   site must be byte-for-byte unaffected when the widget breaks.
2. **Re-read the element at click time.** Never trust the hover result — sticky
   headers and scroll animations move things between hover and click.
3. **Reject Webflow's generated classes** (`/^w-/`, `/^w--/`) when building
   selectors. They are unstable. The element fingerprint stores the element's
   *text*, not only a selector, because a designer renaming a style changes the
   class and not the words.
4. **Intercept clicks in the capture phase** with `preventDefault()`,
   `stopPropagation()` and `stopImmediatePropagation()`.
5. **Build to the accessibility basics that do not need a keyboard path** —
   44 px minimum targets (56 px for the widget's own options), 16 px minimum
   font size, visible focus states, 4.5:1 contrast, `prefers-reduced-motion`
   honoured. **Do not build the keyboard element-selection path** — it is out of
   scope, and **do not put a WCAG conformance claim anywhere**, because without
   it there is not one to make.
6. **Write a migration for every schema change.** No hand edits to the
   database, in any environment.
7. **Write the acceptance test before calling a milestone done.**

## 3 Out of scope

Do not build these. If you think one is needed, ask first.

**Dropped by decision on 7 September** — these are not oversights, do not add
them back:

- German, and any language or locale mechanism. English only.
- The keyboard path to element selection.
- The "this page was fine" button. Problems only. `reports.outcome` exists but
  only ever holds `'problem'` — do not write `'page_ok'`, and do not call
  anything in the UI a "coverage" grid.
- Multiple targets per report.
- "Where did you look for it?"
- Read aloud, and the bigger-text button.

**Never in scope:**

- Signup pages, billing, plans, payment
- A multi-project or multi-customer switcher UI
- White-label, a public API, webhooks
- Kanban boards, sprints, time tracking, burndown charts
- Emails to testers of any kind, including reminders
- Screen recording, console log capture, network request capture
- Drawing or annotation tools
- Severity or priority questions put to the tester

## 4 Working method

**Milestones.** Work the milestones in [build-plan.md §7](build-plan.md#7-milestones)
in order. Do not start the next until the acceptance list for the current one
passes. Each milestone ships and is testable on its own.

**Dependencies.** Ask before adding one to any workspace. Never to the widget,
except `modern-screenshot` for the screenshot module, and ask first.

**Speed.** The schedule is five days and assumes questions get answered within
minutes. So batch your questions, ask them all at once, and keep working on
whatever is not blocked while you wait. Do not stop the whole task for one
answer.

**Commits.** Follow
[git-conventions](https://github.com/aracreate-group/aracreate-conventions/blob/main/git/git-conventions.md).
The three that get missed most:

- `<type>: <lowercase imperative>`, no articles, no trailing full stop, under
  ~72 characters.
- **No `Co-Authored-By` trailers**, and no names, emails or tokens in a message.
  These repos carry single authorship.
- One concern per commit, staged by path — not a blanket `git add`.

**Committing and pushing each need their own explicit instruction, given at the
time.** Approval to make a change is not approval to commit it. Prepare the
work, draft the message to a file, say what is ready, and wait.

## 5 Overrides on record

A rule listed here was **deliberately set aside by Vishnu**, with the cost
known in advance. None of these is an oversight, and none of them is a
precedent for setting aside any other rule. If a rule is not in this section,
it stands exactly as written.

### 21 September 2026 — §1.4 and §1.9, for the widget's React/shadcn rebuild

**Decision.** Rebuild the widget's UI in React with shadcn/ui and Tailwind,
themed with the brand tokens, matching the admin dashboard. Briefed in
[agent-task-shadcn-widget-migration.md](agent-task-shadcn-widget-migration.md).

**What it sets aside.** §1.4 (zero dependencies in `src/widget/`) and the
CSS-framework half of §1.9. The Shadow DOM half of §1.9 stands and becomes
more important, not less.

**Why it is on record rather than simply done.** The agent raised the conflict
instead of proceeding, as the preamble to this document requires, and Vishnu
chose to sequence the admin migration first so the decision could be taken
against real numbers rather than estimates.

**What it costs, measured on the admin migration (19–21 September):**

- **Size.** The widget's built `/v1.js` was ~25KB. React and shadcn alone are
  69–116KB before any of our own code, so the widget is expected to land well
  over 100KB — a 4–5× increase on a script that loads on a client's
  production site. The old "under 15KB" budget is gone, knowingly.
- **Dependencies.** The admin gained 7 runtime packages plus Tailwind and
  PostCSS as build tooling. The widget will carry its own equivalent set.
- **Build.** A Tailwind/PostCSS step, and JSX bundling in the widget's
  esbuild pipeline.
- **A cascade-layer trap, found the hard way.** Unlayered CSS beats layered
  CSS regardless of specificity, so pre-existing rules silently overrode
  Tailwind utilities on any element they matched — in the admin this held
  buttons below the 44px tap-target floor with the correct class applied, and
  it was invisible in `npm run dev` because the dev server serves an
  unprocessed intermediate stylesheet. Verify styling against a production
  build, not the dev server.

**What is NOT overridden, and must be re-proven rather than assumed:**

- **Shadow DOM isolation** (§1.9, second half). Styles inside the shadow
  root, never a global `<style>` tag; re-test against a hostile host page.
- **Fail silently** (§1.1, §2.1) and **one namespaced global** (§1.10).

### 21 September 2026 (later) — the standing "no server upgrade" rule

**Decision.** Install the system libraries a real headless browser
(Chromium/Playwright) needs on the production server
(`212.227.213.174` / `feedback.arametrics.app`), overriding the earlier
closed decision to never touch that server for this purpose.

**What it sets aside.** The rule recorded in the Claude project
(`claude/session-handover.md` §4/§9, `claude/PROJECT-INDEX.md`) that closed
this question: no server upgrade, ever, for the capture-accuracy test —
because the box is shared with an unrelated project (JupyterHub), has no
swap space, and RAM has been observed as low as 109MB free.

**Why it is on record rather than simply done.** Two no-new-server attempts
were tried first and both fell short: tuning the existing capture library
(the positioning bug is upstream in `modern-screenshot`, issue #104, open
and unfixed — see
[agent-task-tune-in-browser-capture.md](agent-task-tune-in-browser-capture.md)),
and swapping to html2canvas (fixes that bug but introduces two new,
unfixable ones of its own, plus a 4x gzipped size increase — see
[agent-task-html2canvas-full-check.md](agent-task-html2canvas-full-check.md)).
Vishnu was told the risks plainly — shared box, no swap, ~19 packages
including desktop/session software — and chose to proceed rather than pay
for a new server. No VPS snapshot was taken first; that was also his
explicit call.

**What it actually cost, measured on 21 September 2026:**

- **Packages.** Exactly 19 new packages via `apt-get install`, matching the
  dependency chain predicted in advance: `libnspr4`, `libnss3`,
  `libatk1.0-0`, `libatk-bridge2.0-0`, `libxdamage1`, `libxkbcommon0`,
  `libasound2`, `libatspi2.0-0`, plus their own transitive dependencies
  (`dbus-user-session`, `at-spi2-core`, `gsettings-desktop-schemas`, alsa
  and dconf packages, `xkb-data`). ~4MB downloaded, ~22.5MB installed.
- **Browser binary.** Chrome Headless Shell (~114MB), downloaded separately
  via `npx playwright install chromium` — the system libraries alone are
  not the browser itself.
- **Disk.** 103GB → 102GB free (of 118GB) — negligible.
- **RAM.** 533MB → 466MB free after cleanup (of 3.8GB, no swap); briefly
  dipped to 239MB mid-install. Tight, as warned, but did not starve either
  service.
- **Both other services verified healthy before and after**, not just
  "still running": `halle-feedback` (HTTP 200, locally and externally,
  before and after) and JupyterHub (HTTP 200 on its real proxied path,
  `/jupyter/hub/login` — a bare `/` or `/hub/login` check misleadingly
  404s, that is normal for this JupyterHub's config, not a fault; one
  active user session with 9–10 live Jupyter kernels throughout, no drop
  attributable to the install).
- **Confirmed working**: a real headless Chromium instance launched
  end-to-end on the server and screenshotted both a static page and a real
  external URL (`halle-dev.webflow.io`) — the exact failure recorded in
  `docs/test-chrome-headless-shell-21-sept.md` no longer reproduces.

**What is NOT done yet.** The capture pipeline is not wired to use this.
This override covers the server having a working browser available to it —
nothing more. See
[agent-task-install-browser-libs-on-prod.md](agent-task-install-browser-libs-on-prod.md).
- **No device storage for the tester token** (§1.3).
- **Accessibility floors** (§2.5): 16px text, 44px targets, 56px for the
  widget's own big option buttons, 4.5:1 contrast.
- **No tester-facing string may be reworded** (§1.8).
- Everything in §3 stays out of scope. This is a UI rebuild, not a UX change.

### 2 October 2026 — admin-v2-spec.md §1.4, "one login for everyone, no roles"

**Decision.** Add one flag, `users.is_admin`, so logins can be managed from
the dashboard on a **Users** screen that only admins can open. Briefed in
[users-admin-spec.md](users-admin-spec.md).

**What it sets aside.** Part of admin-v2-spec.md §1.4. Every other screen
and action stays the same for every login; there are no roles beyond admin
or not, and no permission checks anywhere else.

**Why.** Vishnu chose it over "every login can manage logins", which would
have let any login, the client's included, disable or create any other.

**What it costs.** One more check on the Users page and every one of its
actions (`lib/auth/require-admin.ts`); one migration (0014); the first
admin on each server is made by hand with `make user-admin EMAIL=...`.

### 2 October 2026 — snake_case everywhere, and the only exceptions

**Decision.** Every name this project owns is snake_case, as the araCreate
conventions require (repo §3.3): variables, functions, parameters, props,
object properties, and JSON keys — including the widget↔server contract
(`tester_token`, `upload_url`, …) and the saved widget strings
(`launcher_short`, `btn_back`, …, migration 0015). Vishnu's call: 100%.

**The only exceptions are names this project does not own,** where another
name would simply not work:

- Names a library or the platform reads: React and DOM props (`className`,
  `defaultValue`), Radix/esbuild/Playwright/drizzle/Next.js options
  (`sideOffset`, `entryPoints`, `webServer`, `withTimezone`, `notNull`,
  `searchParams`), browser APIs (`childList`, `currentScript`, `timeZone`),
  cookie options (`httpOnly`, `maxAge`, `sameSite`). Read one into a local
  name, and the local name is snake_case: `{ className: class_name }`.
- React hooks, which must be `useX` for the rules-of-hooks lint
  (`useSidebarCollapsed`).
- React components and TypeScript types, which are PascalCase, and
  constants, which are UPPER_SNAKE_CASE.

**What it cost.** A type-aware rename across src/ and tests/, a migration
for the stored string keys, and one coordinated deploy: the server, the
widget bundle and migration 0015 must go live together, or the live widget
and the server stop understanding each other.
