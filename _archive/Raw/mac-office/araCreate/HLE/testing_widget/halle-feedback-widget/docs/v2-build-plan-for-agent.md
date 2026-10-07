# V2 BUILD PLAN — for the coding agent

**Written by Claude, 8 September 2026. Corrected against the real repo the
same evening — read §0.1 before anything else.** This is the single document
that turns `widget-v2-spec.md` and `admin-v2-spec.md` into something an agent
can build without stopping to ask what to do. Every decision that could block
the agent has been made below. Where a real judgement call remains, it is
marked **OPEN** and the agent is told exactly what to do about it (almost
always: make the documented default choice and note it, don't stop).

**Precedence.** This document, `widget-v2-spec.md` and `admin-v2-spec.md`
override `docs/build-plan.md` and `docs/BUILD-SPEC.md` wherever they
disagree. Those two are the historical record of v1, not instructions for
this build. `docs/agent-rules.md` (12 nevers, 7 always) still applies in
full — this document adds to it, it does not replace it.

**The agent's job in this run is code only.** Do not rewrite, restructure or
"tidy" any document in `docs/`. The two exceptions, and the only two files in
`docs/` this run may create or append to, are `docs/v2-blocked.md` and
`docs/v2-overnight-log.md`.

---

## 0.1 Corrections to this plan — read first

An earlier draft of this plan told the agent that screenshot capture and
storage were not built and had to be written fresh. **That was wrong.** The
repo was inspected directly on the evening of 8 September and the draft was
corrected before it reached the agent. What is actually true:

| Earlier draft said | Reality in the repo |
| --- | --- |
| Screenshot capture not built | **Built and committed** — `5056f35 feat: add M6b — screenshot capture, privacy stripping and consent`. `src/widget/src/capture.ts`, loaded as a separate lazy chunk, 304 lines of Playwright tests in `tests/widget/capture.spec.ts` |
| Picture storage not built | **Built and committed** — `e9795c4 feat: add M6a — screenshot storage and signed uploads`. `src/web/lib/storage/*`, `POST /api/v1/uploads`, ~1,900 lines with tests |
| Build it as PNG | It is **WebP**. `capture.ts` produces `image/webp` at quality 0.8, `lib/api/webp.ts` checks the RIFF/WEBP magic bytes on upload, and the storage key validator only accepts `.webp` |
| Store at `storage/reports/<project_id>/<report_id>.png` | The real, validated shape is `reports/<project_id>/<report_id>.webp` — enforced by one regex in `src/web/lib/storage/keys.ts`. Do not invent a second layout |
| Column is `image_path`, not null | The column is **`reports.screenshot_key`**, already present, **nullable**, and it stays nullable — `widget-v2-spec.md §11` explains why |
| No retention in this plan | Retention is **already built** — `RETENTION_DAYS = 90` in `src/web/lib/retention.ts`, with a sweep in `src/web/lib/db/retention.ts` and `scripts/retention.mts`. Do not remove it and do not change the number |
| Capture the whole page vs viewport was open | Already built as **the visible window only**. Nothing to decide |
| Milestones are M6–M9 | **Renumbered M7–M10.** M6a and M6b are taken |

**The rule this establishes for the run:** if this plan says something does
not exist, check the repo before building it. Where this plan and the repo
disagree about what is already built, **the repo wins** — log the
disagreement in `docs/v2-blocked.md` and carry on with what is there.

**Status this plan assumes, verified against the repo 8 Sept evening:**
M0–M5 built, tested (437 tests) and committed. M6a and M6b built and
committed. Two more commits on top: `19eb46b docs: correct build-plan.md
against the M6a/M6b implementation` and `1bd0add build: add make demo`.
There is also **staged but uncommitted work in the index** (tunnel scripts,
`v1.js` / `capture.js` routes, `widget-asset.ts`, password scripts,
`docs/live-test-plan.md`, `Makefile`) that is **not part of this run** — see
§7.

---

## 0.2 How to run this

**This is an overnight run.** How to execute it — continuous or
stop-between-milestones, what to do when genuinely blocked, commit
authorization, logging — is set by `v2-overnight-run.md`, not by this
section. Read that document first. This document is the technical content
for the four milestones; it does not itself decide the run's pace.

| # | Milestone | Depends on |
| --- | --- | --- |
| M7 | Database — schema changes | nothing |
| M8 | Widget v2 — the tester-facing flow | M7 |
| M9 | Picture flow changes on top of M6a/M6b | M7 (can run alongside M8 — different files) |
| M10 | Admin v2 — queue, picture viewer, filters, simplified login | M7, M8, M9 |

---

## 1 Pre-answered decisions — read this before starting anything

| Question an agent might otherwise stop on | Answer |
| --- | --- |
| Is screenshot capture / storage built already? | **Yes — M6a and M6b.** Adapt it, never rebuild it. See §0.1 and §4 |
| Is the picture a PNG or WebP? | **WebP.** Unchanged. Do not convert the pipeline to PNG |
| Where do pictures live? | `reports/<project_id>/<report_id>.webp` on local disk, via the existing storage module. **No cloud storage, no S3, no third-party service** — "everything local" is a standing decision |
| Which column holds the picture? | **`reports.screenshot_key`**, existing, nullable, stays nullable |
| Does every report have a picture? | The tester is never asked and cannot decline (`widget-v2-spec.md §11`). But a capture that **fails** must still let the report through — `agent-rules.md §1.11`. So: always attempted, occasionally absent, never blocking |
| Is there a consent screen? | **Not any more.** Remove the step and its four config strings. Keep every line of privacy stripping |
| Retention? | **Already built, 90 days.** Leave it alone. Do not delete, do not retune |
| Do roles/permissions get removed or just disabled? | **Removed.** Delete the code, the column, the checks. Not commented out, not left dormant |
| Does the `issues` table stay, simplified? | **No.** It is replaced by a `status` column directly on `reports`. Drop the `issues` table, its transition table, its event log, its comments table |
| Does tester-page assignment get touched? | **No.** Leave the testers table and whatever assigns pages to testers exactly as it is |
| Are `answer_id` and `option_order` dropped from `reports`? | **No.** Kept, nullable, simply never written to again |
| One login — one shared account, or many accounts with no roles? | **Many accounts, no roles.** Keep the existing accounts/session mechanism from M3. Drop the role column and every permission check tied to it |
| Does the exact page stay on a report, or just its template? | **Exact page stays.** Template is only how the list is organised. `admin-v2-spec.md §6` |
| Are screenshot-mode reports grouped? | **No. Never.** Every report is its own row, always |
| What names a report in the list? | Page name + number. E.g. "Contact page #14" |
| Whole page or visible window in the picture? | **Visible window.** Already built that way. Not a question |
| Is there real tester data in the local database? | No. Testing hasn't started — blocked on hosting and the real page URLs. Treat the local database as disposable. Still take a `pg_dump` before any destructive migration, as a habit |
| Who can see a stored picture? | Only through an authenticated route. Never a public URL |

---

## 2 M7 — Database

### 2.1 `reports` table — new columns

```
comment          text        not null
mode             text        not null    check (mode in ('pointer','screenshot'))
markup           jsonb       null        -- marker-pen strokes, coordinate form
meta             jsonb       not null    -- see §2.3
status           text        null        check (status in ('bug','deleted','fixed','closed'))
                                          -- null = still in the queue, not yet looked at
```

**`screenshot_key` already exists and is not touched by this migration.** It
is nullable and stays nullable (§1). Do not add `image_path`; do not rename
`screenshot_key`.

`status` is null on arrival — that null *is* "in the queue". Do not add a
separate `'queue'` value; a report with `status is null` and one with
`status = 'bug'` are the two states that matter for the queue view and the
tracked-item view respectively (see §5.1).

### 2.2 `reports` table — columns that stop being written

`answer_id` and `option_order` stay in the schema, both nullable already (or
made nullable now if they weren't). No writes to either column anywhere in
the new code path. Do not drop them.

### 2.3 The `meta` JSON shape

Written once, at submission, never updated. Matches
`widget-v2-spec.md §5` exactly:

```json
{
  "page_url": "string",
  "referrer": "string | null",
  "screen_size": "string, e.g. '1512 × 982 px'",
  "viewport": "string, e.g. '1502 × 760 px'",
  "device_type": "phone | tablet | desktop",
  "browser": "string, name + version",
  "os": "string",
  "input": "touch | mouse / trackpad",
  "language": "string, e.g. 'en'",
  "captured_at": "ISO 8601 timestamp",
  "console_errors": [
    { "at": "HH:MM:SS", "text": "string" }
  ]
}
```

`console_errors` is capped at 5 entries client-side before it is ever sent —
the server does not need to enforce the cap, but should not choke if a
malformed client sends more; truncate to 5 server-side defensively.

### 2.4 `pages` table — new column

```
template   text   not null   check (template in ('home','product_category','product_detail','contact','not_found'))
```

Backfill: the existing `pages` table (built in M5, "pages with bulk import")
has real rows already, even if only 3 have real URLs (the rest are
placeholders). Backfill `template` for every existing row using the URL
pattern table in `admin-v2-spec.md §6` (`/` → home,
`/products-selection/…` → product_category, `/products/…` →
product_detail, `/contact` → contact, anything else that 404s → not_found).
This is a one-time data migration, not new logic — the runtime detection
method (tag-first, URL-fallback) only matters for pages added after this
migration, via the bulk-import path. Wire the bulk-import code to read a
template hint if the import file provides one, and fall back to the same
URL-pattern matching otherwise.

**Do not invent the real page URLs.** A placeholder row gets a template by
the same rules as any other row; it does not get a guessed address.

### 2.5 Dropped entirely

- `issues` table (and its transition/state-history table, if it's a separate
  table rather than a column).
- Issue comments table.
- Issue event log table — **check first** whether this event log is used for
  anything other than issues (e.g. a general audit log). If it's
  issues-only, drop it. If it's shared, keep the table and just stop writing
  issue-specific events to it.
- Role column on the accounts/users table, and any permissions/role-check
  table.
- Team-list table, if it's separate from the general accounts table.
- Issue-assignment column/table (who a bug is assigned to).

**Do not drop:** the testers table, whatever implements tester-page
assignment, the accounts/session tables themselves (only the role column
inside them), the string-editor tables, the pages table (only gains a
column), **anything in the storage or retention path**.

### 2.6 Acceptance for M7

- A fresh migration applies cleanly to a reset local database.
- Existing `pages` rows all have a non-null `template` after migration.
- No code anywhere still references the dropped tables/columns — a full-repo
  search for their names turns up nothing outside the migration history
  itself.
- `answer_id` and `option_order` remain queryable and nullable; a manual
  `select` against a freshly migrated, empty `reports` table succeeds.
- `screenshot_key` is untouched, still nullable, and the existing M6a upload
  tests still pass unchanged.

---

## 3 M8 — Widget v2 (tester-facing)

### 3.1 Source of truth

`widget-v2-spec.md` in full, plus the working mock —
`widget-prototype-v2-flow-mock.html` in the Claude project — for the
interaction and motion design specifically (§4 of the spec references it
directly). The mock is plain browser JavaScript; it is a reference for
*behaviour*, not a file to copy wholesale into the production widget's build
system. Bundling, Shadow DOM wrapping and the size budget already built in
M2/M6b all still apply.

### 3.2 What changes from the widget as built

- Remove: the five-answer question step, the "where did you look for it?"
  step, randomised option order, the separate confirm-your-target step, **and
  the consent screen** (`widget-v2-spec.md §11`) with its four config strings
  `consentQ`, `consentSub`, `btnConsentInclude`, `btnConsentExclude` and
  their string-editor rows.
- Keep and reuse exactly as built: element pointing, hover outline,
  capture-phase click interception, element fingerprinting (selector, text,
  tag, box coordinates), Shadow DOM isolation, silent-failure behaviour,
  the size gate, the lazy capture chunk and its loader, and **all of
  `capture.ts`'s privacy path**.
- New: the mode chooser (two icon+word buttons), the docked corner panel
  with fixed header/scrollable body/fixed footer (`widget-v2-spec.md §4`),
  the one shared comment-and-picture screen, the marker pen (both modes),
  the `meta` capture including the console-error listener, the "What else we
  send with this" disclosure.

### 3.3 Payload sent to the reports endpoint

```json
{
  "page_id": "existing field, unchanged",
  "token": "existing field, unchanged",
  "mode": "pointer | screenshot",
  "comment": "string, min 1 character after trim",
  "target": {
    "selector": "string",
    "text": "string",
    "box": { "x": 0, "y": 0, "w": 0, "h": 0 }
  },
  "markup": [[[0,0],[1,1]]],
  "meta": { "...": "as in §2.3" }
}
```

`target` is present only in pointer mode; omit the key entirely in
screenshot mode rather than sending `null` — simpler for the API layer to
validate with one schema branch on `mode`.

**The picture is not in this payload.** It travels by the existing M6a
upload sequence, unchanged — see §4.2. Keep whatever field the built code
already uses to tie an upload to its report; do not invent a new
`image_ref` contract on top of it.

### 3.4 Client-side validation

- `comment`: required, trimmed, minimum 1 character. Send button disabled
  until satisfied.
- Everything else in the payload is captured automatically; there is no
  other tester-facing required field.

### 3.5 Acceptance for M8

- Both paths (pointer, screenshot) reach the one shared screen and can send
  a complete report.
- No consent screen appears anywhere in either path.
- The panel docks to the launcher's corner, never centres on the screen, on
  desktop and mobile viewport sizes.
- Send is disabled with an empty comment, and a whitespace-only comment is
  rejected.
- The marker pen draws, undoes, and clears correctly with both mouse and
  touch input (Playwright touch emulation, matching the existing approach).
- A thrown JS error on the host page during the session appears in
  `meta.console_errors` on the next report submitted, capped at 5.
- The existing widget suites still pass for everything that didn't change —
  Shadow DOM isolation, capture-phase click, size budget, and the whole of
  `tests/widget/capture.spec.ts` **except** the consent assertions, which
  are updated rather than deleted wholesale (a deleted test is invisible; an
  updated one still proves the flow).
- Gzip size still inside the existing budgets, same split as before: mode
  chooser and comment box in the main bundle, marker pen in the lazy capture
  chunk.

---

## 4 M9 — Picture flow changes on top of M6a/M6b

**This milestone does not build capture, upload or storage. Those exist.**
It changes *when* a picture is taken and *what is drawn into it*, and it
removes the consent step's effect on the picture path. Land it as its own
commit — this is the privacy-sensitive part of the project and it gets
reviewed on its own.

### 4.1 What already exists and must not be rewritten

Read these files before writing a line:

- `src/widget/src/capture.ts` — the lazy-loaded capture module. Produces a
  WebP blob of the **visible window**, double-captures for Safari, returns
  `null` on any failure instead of throwing. Its `strip_clone()` blanks every
  `input` value and attribute, every `textarea` value **and** child text,
  every `[contenteditable]`, and every `[data-fb-block]`, on a detached
  clone attached under `document.documentElement` — never on the live page.
  The header comment explains why each of those four cases needs a different
  mechanism. **Do not simplify it, do not replace blanking with placeholder
  characters, do not move the clone.**
- `src/widget/src/loader.ts` — how the chunk is fetched at runtime without
  `import.meta.url`. Leave the mechanism alone.
- `src/web/app/api/v1/uploads/route.ts`, `src/web/lib/storage/*`,
  `src/web/lib/api/webp.ts`, `src/web/lib/api/uploads-schema.ts` — the
  signed-slot upload, the magic-byte check, the one legal key shape.
- `src/web/lib/retention.ts`, `src/web/lib/db/retention.ts`,
  `src/web/scripts/retention.mts` — the 90-day sweep.

An open question already logged in `docs/blocked.md` and still open: a
`<select>`'s chosen option is deliberately **not** blanked. Leave it that
way; it is not this run's decision.

### 4.2 What changes

1. **Capture trigger.** Pointer mode: capture on the click that selects the
   element. Screenshot mode: capture the moment the mode is chosen. Both
   before the shared screen renders, so the tester sees the real picture.
2. **Element box burned in** (pointer mode): draw the selected element's box
   onto the captured image before upload, using the fingerprint coordinates
   already captured in M2.
3. **Marker strokes burned in** (both modes): composite the strokes onto the
   image before upload, and also store them as coordinates in
   `reports.markup`. Order matters — the tester's strokes go on top of the
   element box.
4. **Consent removed from the picture path.** No consent screen, no
   `consented` session field, no branch that sends a report deliberately
   without a picture. The only path to a report without a picture is a
   capture that returned `null`.
5. **A failed capture still sends the report.** `screenshot_key` null,
   nothing shown to the tester, one line in the widget's own silent-failure
   path. This is `agent-rules.md §1.11` and it does not change.

### 4.3 Acceptance for M9

- A full pointer-mode report and a full screenshot-mode report each produce
  a viewable WebP on disk at the one legal key shape, with the box and the
  strokes visibly burned in.
- Forcing the capture function to return `null` still produces a sent
  report, with `screenshot_key` null and nothing shown to the tester.
- The picture is not retrievable by an unauthenticated request, to the
  storage path or to the serving route.
- The existing WebKit capture test still passes.
- **The privacy test still fails when stripping is skipped.** Take the
  existing privacy test, deliberately disable `strip_clone()`, and prove the
  test goes red. If it stays green, the test is the bug — fix the test and
  say so in the log. This is the single most important check in the run.

---

## 5 M10 — Admin v2

### 5.1 The two lists

Not one list with a status filter bolted on — two distinct views, because
they have different actions available (`admin-v2-spec.md §1`, §4):

- **Queue** — every report where `status is null`. Actions per row: **Bug**,
  **Delete**.
- **Tracked items** — every report where `status in ('bug','fixed','closed')`.
  Actions per row: **Fix it**, **Close it** (available regardless of current
  state — clicking either just sets `status` to that value; there is no
  "already fixed, can't re-open" restriction unless testing surfaces a real
  need for one — don't add a state machine back in by accident).

Deleted reports (`status = 'deleted'`) do not appear in either list. Keep
the row in the database (nothing in this project is hard-deleted —
consistent with the append-only convention already established for
`reports` in M0); just filter it out of both views.

A report whose `screenshot_key` is null shows a plain "no picture was taken"
placeholder in the list and in the viewer. Never a broken image.

### 5.2 Grouping by template

Both lists group by `pages.template` first, per `admin-v2-spec.md §6`. Order
of groups: Home, Product Category, Product Detail, Contact, 404/Not Found —
that fixed order, not alphabetical, not by count. Within a group, sort
newest-first by `created_at`. Each report row is named `<page name> #<n>` —
`<n>` is a per-project running number, not per-template and not per-page
(simplest to implement as a sequence already used elsewhere in the schema,
or the row's own primary key rendered short, if no such sequence exists —
agent's choice, not worth a design decision).

### 5.3 The picture viewer

- Opens from a click on a report's thumbnail in either list.
- Shows the full-size image, the comment text, the mode, (pointer mode
  only) the element text/selector, and a collapsed-by-default technical
  section mirroring the tester-side "What else we send with this" —
  same grouping (this page / your device / errors), reusing the `meta`
  JSON directly rather than re-deriving it.
- Zoom: minimum viable is click-to-toggle between fit-to-screen and 100%;
  scroll-wheel/pinch zoom is a nice-to-have, not required.
- Next/previous: moves to the adjacent report **within the current
  filtered/sorted list** (respecting whatever filters are active — §5.4),
  not the whole unfiltered table. Keyboard arrow keys work in addition to
  on-screen buttons.
- The relevant action buttons (Bug/Delete or Fix it/Close it, depending on
  which list opened the viewer) are present in the viewer itself, so a
  reviewer doesn't have to close it to act.
- Images are served only through the authenticated route. The viewer does
  not get its own unguarded path.

### 5.4 Filters and search

- Filters: template (5 values), mode (pointer/screenshot), date range.
  Status is implicit in which list is open (§5.1), not a filter within it.
- Search: free-text match against `comment` only. A case-insensitive
  substring match is sufficient — the tester panel is 10–30 people and the
  site is 49 pages; this does not need full-text search infrastructure.
- Filters and search combine (AND, not OR) and persist into the picture
  viewer's next/previous scope (§5.3).

### 5.5 Login simplification

- Keep the existing session/login mechanism from M3 as-is.
- Remove the role column from the accounts table (§2.5) and every
  permission check that reads it.
- Every screen in the admin app becomes visible to every logged-in account.
  There is no screen, filter, or field that used to require a specific role
  and now needs a replacement gate — it simply becomes ungated.

### 5.6 CSV export

Both existing exports (per-report, per-issue — the second no longer applies
since there's no separate issues table) collapse into one export, from
`reports`, with these columns: `id`, `page_url`, `template`, `mode`,
`comment`, `target_text`, `target_selector`, `status`, `created_at`. Do not
dump the `meta` JSON as one column — flatten `browser`, `os`, `device_type`
and `screen_size` into their own columns; leave `console_errors` and the
rest of `meta` out, since anyone who needs that detail opens the viewer.
Keep the existing CSV formula-injection guard — that was a real bug caught
by the M3 quality gate, and the new export path must not reintroduce it.

### 5.7 Acceptance for M10

- Logging in as any account (no role concept left to test against) reaches
  every screen.
- A report submitted by the widget appears in the Queue, grouped under the
  correct template, within one refresh/poll cycle.
- Bug/Delete and Fix it/Close it each update `status` correctly and move the
  row to/out of the correct list immediately.
- The picture viewer opens, shows the correct image and metadata, and
  next/previous correctly stays within the current filter/search scope.
- A report with a null `screenshot_key` renders correctly in both lists and
  in the viewer.
- Filters and search combine correctly (tested with at least: template +
  mode together; search text that matches nothing shows an empty, not
  broken, list).
- CSV export opens cleanly in a spreadsheet with no formula-injection
  warning on a comment deliberately crafted to start with `=`, `+`, `-`, or
  `@`.
- An unauthenticated request for a report image is rejected.

---

## 6 What this plan deliberately does not cover

- Hosting, the real 49 page URLs, the client's Webflow plan, German —
  unrelated blockers, tracked in `session-handover.md §4`, not touched by
  this build.
- Retention policy beyond what is already built (90 days) — the *number* is
  a real conversation, but not this run's.
- Whether a report can be "un-deleted" from the queue view — not asked for,
  don't build it speculatively.
- Notifications, reminders, or any signal that a test round has finished.

## 7 Rules specific to this plan, on top of `docs/agent-rules.md`

- **Code only.** Do not edit any document in `docs/` except
  `docs/v2-blocked.md` and `docs/v2-overnight-log.md`. If a doc is wrong,
  log it — do not fix it.
- **Leave the pre-existing staged work in the index alone.** At the start of
  this run `git status` already shows staged changes that are not part of
  M7–M10 (tunnel scripts, `v1.js` / `capture.js` routes, `widget-asset.ts`,
  password scripts, `docs/live-test-plan.md`, `Makefile`). Do not commit
  them, do not unstage them, do not revert them. For every milestone commit,
  stage **only the exact paths you created or changed for that milestone**,
  by path — never `git add -A`, `git add .`, or `git commit -a`. If a file
  you must change is already in that staged set, keep going but log it in
  `docs/v2-blocked.md`.
- Do not invent the real page URLs, or a URL pattern for a template that
  isn't in `admin-v2-spec.md §6`.
- M9 lands as its own commit with nothing else mixed in — privacy-sensitive
  code gets reviewed on its own.
- No `Co-Authored-By` trailer, ever. Commit authorization for this run is
  set by `v2-overnight-run.md`.
- If a Playwright test for M8 or M9 needs a browser feature the existing
  setup doesn't exercise (canvas compositing, touch emulation), that's
  expected — the existing suites are the pattern to extend, not a ceiling.

## 8 Open items carried forward, not blocking this build

1. The retention *number* (90 days, already built and running).
2. Whether pinch/scroll zoom is worth adding to the viewer beyond the
   click-to-toggle minimum in §5.3 — try the minimum first.
3. Whether a `<select>`'s chosen option should be blanked in the capture —
   already logged in `docs/blocked.md`, left as built.
4. Whether a tester can retake a picture; icon shapes; exact wording —
   `widget-v2-spec.md §8`, all non-blocking.
