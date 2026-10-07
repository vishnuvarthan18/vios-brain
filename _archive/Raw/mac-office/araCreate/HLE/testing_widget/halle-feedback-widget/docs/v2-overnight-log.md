# V2 OVERNIGHT LOG

Run started 2026-09-08, per `docs/v2-overnight-run.md`. Milestones M7 → M8 →
M9 → M10, one continuous unattended pass, quality gate applied per milestone.

## Start-of-run checklist

- [x] `docs/agent-rules.md` and `docs/quality-gate.md` read in full.
- [x] `v2-build-plan-for-agent.md` read in full — §0.1 first, then §1.
- [x] `widget-v2-spec.md` and `admin-v2-spec.md` read in full.
- [x] The four §4.1 files read before any picture-path code: `capture.ts`,
      `loader.ts`, `app/api/v1/uploads/route.ts` + `lib/storage/*` +
      `lib/api/webp.ts` + `lib/api/uploads-schema.ts`, `lib/retention.ts` +
      `lib/db/retention.ts` + `scripts/retention.mts`.
- [x] `git status` recorded (2026-09-08 22:xx, before M7):
  ```
  On branch main
  Changes to be committed:
    modified:   Makefile
    new file:   docs/live-test-plan.md
    new file:   scripts/tunnel-stop.sh
    new file:   scripts/tunnel.sh
    new file:   src/web/app/capture.js/route.ts
    new file:   src/web/app/v1.js/route.ts
    new file:   src/web/lib/widget-asset.ts
    modified:   src/web/package.json
    new file:   src/web/scripts/check-demo-password.mts
    new file:   src/web/scripts/user-password.mts
    modified:   tests/db/tenant-import-guard.test.ts
  Untracked files:
    docs/admin-v2-spec.md
    docs/v2-build-plan-for-agent.md
    docs/v2-overnight-run.md
    docs/widget-v2-spec.md
  ```
  This matches exactly what `v2-build-plan-for-agent.md §0.1`/§7 describes as
  pre-existing and out of scope. Left untouched for the whole run, confirmed
  again before each commit.
- [x] Confirmed no real submitted reports before M7's migration: local
      `halle_feedback_dev` has 210 rows in `reports`, several explicitly
      marked `Fixture note — nicht echt.`, the rest low-effort manual
      test-clicking noise consistent with "testing hasn't started, blocked on
      hosting and real page URLs" (`v2-build-plan-for-agent.md §1`). Took a
      `pg_dump` backup to the session scratchpad before running M7's
      migration, as a habit per §2's first stop condition, not because the
      data looked real.
- [x] This file created before M7 begins.

---

## M7 — Database (2026-09-08, 22:06–22:39)

**What was done.** New migration `0005_v2-database-simplification.sql`
(hand-edited after `drizzle-kit generate` produced SQL that would fail
outright against this database's existing rows — see the migration's own
header comment):

- Dropped `issues`, `issue_reports`, `issue_events`, `comments` tables, the
  `role` column and its check constraint on `users`, and `projects.next_issue_ref`.
- Added `reports.comment` (not null), `reports.mode` (not null, check
  pointer/screenshot), `reports.markup` (jsonb, nullable), `reports.meta`
  (jsonb, not null), `reports.status` (nullable, check
  bug/deleted/fixed/closed). Added nullable first, backfilled (placeholder
  values for the 210 pre-existing v1 rows: `comment` = a plain marker string,
  `mode` = `'screenshot'`, `meta` = `{}`), then constrained NOT NULL.
- Added `pages.template` (not null, check home/product_category/
  product_detail/contact/not_found). Backfilled from the URL-pattern table
  in `admin-v2-spec.md §6` — all 3 existing rows (`/`, `/contact`, `/404`)
  matched a real pattern, nothing fell through to a guess.
- Rewrote the `reports_append_only` trigger function (originally installed
  in migration 0001) to allow ONLY a status-only UPDATE through — every
  other UPDATE, every DELETE, every TRUNCATE still raises exactly as before.
  The backfill's own UPDATEs run with the trigger temporarily disabled
  (`ALTER TABLE ... DISABLE/ENABLE TRIGGER`), re-enabled before the migration
  ends, since they predate the status-aware replacement and touch columns
  other than status.
- Took a `pg_dump` of `halle_feedback_dev` to the session scratchpad before
  running the migration against it, per §2's habit (not because the data
  looked real — several rows were explicitly marked
  `Fixture note — nicht echt.`, the rest low-effort manual test-clicking
  noise consistent with "testing hasn't started").

**Code changes on top of the schema:**

- `lib/db/report-status.ts` (new) — `set_report_status`, the one function
  allowed to mutate `reports`: tenant-scoped, `is_uuid`-checked, sets only
  `status`. No permission check — admin-v2-spec.md §1.4/§5.5, one login for
  everyone.
- `lib/db/submit-report.ts` — rewritten: no more issue-grouping transaction
  (dropped `issues`, `group_report`); a report insert is now a page lookup
  plus one insert, no transaction needed.
- `lib/api/reports-schema.ts` / `app/api/v1/reports/route.ts` — v2 payload
  (`comment`, `mode`, `markup`, `meta`, `target` optional not nullable);
  `console_errors` truncated to 5 server-side defensively; `markup` bounded
  (200 strokes, 2000 points/stroke) after an adversarial review flagged it
  as the one unbounded field in the payload.
- Deleted the whole issues/roles/team code surface: `lib/db/issue-*.ts`,
  `lib/db/comments.ts`, `lib/db/categories.ts`, `lib/db/team.ts`,
  `lib/db/export-issues.ts`, `lib/db/admin-permissions.ts`,
  `lib/db/issue-permissions.ts`, `app/app/issues/**`, `app/app/admin/team/**`,
  `app/app/export/issues/route.ts`.
- Every admin action that used to take an `Actor` and call
  `can_manage_*(actor)` (pages, testers, assignments, string editor) had that
  parameter and check removed — `dashboard_scope()`/`dashboard_user_id()`
  still gate every one of them on having a valid session, just not a role.
  Confirmed by re-reading every call site: none lost its session check, only
  its role check.
- `lib/page-template.ts` (new) — the URL-pattern fallback
  (`template_from_url_pattern`), used by the migration's backfill, `db-seed.mts`,
  and `pages-admin.ts`'s add/edit/bulk-import (which also accept an explicit
  template hint, tab-separated in bulk import, that wins when valid).
- `lib/auth/session.ts` — `role` removed from the session cookie payload
  entirely (no column left to read it from).

**Tests.** 367 passing (up from 350 pre-M7; net new: `report-status.test.ts`,
`reports-schema.test.ts`; net removed: every issue/team/comments/
admin-permissions test file, ~9 files). `tests/db/reports-append-only.test.ts`
extended to prove the new status-only exception precisely: status-only
succeeds, status+another-column together still fails, a lone non-status
column still fails, delete/truncate still fail.

**Quality gate:**

- `make lint` / `make build` / `make test` (367) / `make test-widget` (35,
  unaffected — M7 touched no widget code) / `make size` (both budgets,
  unaffected) — all green.
- §1 mutation proofs run and reverted: (a) `pages_template_check` and
  `reports_status_check` — inserted an invalid value directly via psql
  against the test database, both rejected at the constraint; (b) the
  append-only trigger — status-only update succeeds, comment-only update
  rejected, status+comment together rejected, delete rejected, all verified
  directly via psql before writing the automated test; (c) dropped
  `scoped_where` from `set_report_status` — the new IDOR test
  (`a report id from a different project resolves to undefined`) went red
  exactly as expected (cross-project status write silently succeeded),
  reverted, green again; (d) dropped the `is_uuid` guard from
  `set_report_status` — the malformed-id test went red with an unhandled
  Postgres `invalid input syntax for type uuid` error instead of resolving
  to undefined, reverted, green again.
- §2 attack list, items exercised: IDOR (report-status.test.ts, cross-project
  id → undefined), unsafe query construction (grepped all `sql\`...\`` raw
  template literals in lib/db — every one is a bound Drizzle column
  reference or a schema-level CHECK constraint, no string interpolation of
  untrusted input), wrong-type input reaching a uuid column (malformed id →
  undefined, not a 500). Role-based authorisation bypass (item 1) is now
  moot for every action this milestone touched — there are no roles left to
  bypass; every action is gated on session only, confirmed at every call
  site during the adversarial review below.
- §3 concurrency: `report-status.test.ts` fires two concurrent status
  changes on the same report (`Promise.all`) and asserts both succeed with
  the final value being one of the two, never a lost update or a merge.
- §4 loop/TODO greps: clean, nothing found in `lib/db`, `app/api`, `app/app`.
- §5 adversarial review: run via a subagent given the full file list and the
  quality-gate's own five questions. Findings: (1) the v2 report lifecycle
  (`set_report_status`) has no route/action/UI wired to it yet — correctly
  identified as a deliberate M7/M10 boundary (M7 is "database only" per
  v2-build-plan-for-agent.md §0.2), not a defect, and confirmed by
  `app/app/layout.tsx`'s own comment; (2) a stale doc comment in
  `lib/db/reports.ts` still described the dropped issues-transaction
  behaviour — fixed; (3) `reports-schema.ts`'s `markup` field had no upper
  bound, unlike every other payload field — fixed (200 strokes / 2000
  points per stroke) and covered by a new `reports-schema.test.ts`.

**Commit — first attempt caught and corrected.** The first `git commit` (no
`-a`, no pathspec, just `git commit -m`) picked up the ENTIRE index, not only
the paths this run had just `git add`ed — the pre-existing staged set
(Makefile, `docs/live-test-plan.md`, `scripts/tunnel-stop.sh`,
`scripts/tunnel.sh`, `src/web/app/capture.js/route.ts`,
`src/web/app/v1.js/route.ts`, `src/web/lib/widget-asset.ts`,
`src/web/package.json`, `src/web/scripts/check-demo-password.mts`,
`src/web/scripts/user-password.mts`, `tests/db/tenant-import-guard.test.ts`)
was still sitting staged in the index from before this run started, and
`git commit` with no pathspec commits the whole index regardless of what was
just `git add`ed — staging M7's files on top of an already-staged unrelated
set does not scope the commit to only the new additions. Caught immediately
by re-reading `git status` and the commit's own file list right after
committing (94 files, correctly expected 83) — this is exactly what
`v2-overnight-run.md §1` "stage by path, never wholesale" and
`v2-build-plan-for-agent.md §7` were warning against, and a plain `git add
<paths>` followed by an unscoped `git commit` was not actually sufficient to
honour it when the index already had unrelated staged content sitting in it.

Fixed with `git reset --soft HEAD~1` (purely local, nothing pushed anywhere,
fully reversible — restores the exact pre-commit index) followed by `git
restore --staged` on exactly the 11 pre-existing paths, leaving them
unstaged/untracked again exactly as they were at the run's start, confirmed
against the original recorded `git status`. Re-committed with only the 83
M7 paths staged. `make test` re-run clean (367 passing) after the correction
to confirm the reset/restore sequence didn't lose or alter anything.

**Lesson for M8/M9/M10:** `git add <exact paths>` is necessary but not
sufficient when the index already has unrelated staged content — after
staging, diff `git status`'s staged list against the intended file list
BEFORE running `git commit`, every time, not just after.

Final commit: `feat: land M7 — database simplification for v2` (83 files).
Pre-existing staged set confirmed still unstaged/untracked, diffs unchanged
from the run's start.

Proceeding to M8.

---

## M9 — Picture flow changes on top of M6a/M6b (2026-09-08, ~23:00)

Landed BEFORE M8's own commit, deliberately: `v2-build-plan-for-agent.md §7`
requires "M9 lands as its own commit with nothing else mixed in — the
privacy-sensitive part of the project gets reviewed on its own." M8's state
machine (app.ts) calls M9's new `load_and_burn_in` as part of its send flow,
so the two are read-order dependent but not commit-order dependent — M9's
own diff (capture.ts + loader.ts) is purely additive and stands on its own
without app.ts's changes at all, so it commits first, cleanly, on its own.

**What was done.** Read the four files `v2-build-plan-for-agent.md §4.1`
names before writing anything: `capture.ts`, `loader.ts`,
`app/api/v1/uploads/route.ts` + `lib/storage/*` + `lib/api/webp.ts` +
`lib/api/uploads-schema.ts`, `lib/retention.ts` + `lib/db/retention.ts` +
`scripts/retention.mts`. `strip_clone()` untouched, not even reformatted —
confirmed by diff review before committing.

- `capture.ts` — added `burn_in_markup(blob, box, strokes)`: decodes the
  already-captured WebP via `createImageBitmap`, draws the element box (if
  pointer mode) then the marker strokes ON TOP (widget-v2-spec.md §6 — order
  matters), re-encodes to WebP. Returns the ORIGINAL blob unchanged on any
  failure — a burn-in failure must never turn a successful capture into no
  picture at all, same "never block a report" contract `capture_screenshot`
  itself already followed (agent-rules.md §1.11).
- `loader.ts` — added `load_and_burn_in`, the dynamic-import counterpart to
  the new function, with the same null/original-blob-on-failure contract as
  `load_and_capture`.
- Capture trigger moves to selection/mode-choice time (M8's app.ts calls
  `start_capture()` the instant a target is picked or Screenshot is chosen,
  never waiting for a separate step) — this file's own half of that is just
  exposing the capture/burn-in split; the actual trigger timing lives in
  app.ts (M8).
- Consent is not referenced anywhere in this diff — nothing to remove here,
  since capture.ts/loader.ts never implemented the consent UI themselves
  (that lived in app.ts, removed in M8).

**What was NOT touched, verified by reading the diff before staging:**
`lib/storage/*`, `app/api/v1/uploads/route.ts`, `lib/api/webp.ts`,
`lib/api/uploads-schema.ts`, `lib/retention.ts`, `lib/db/retention.ts`,
`scripts/retention.mts` — all M6a, all untouched. The one legal key shape
(`reports/<project_id>/<report_id>.webp`) and the 90-day retention number
are both exactly as built.

**Quality gate — the single most important check in this run
(v2-build-plan-for-agent.md §4.3):** deliberately commented out the
`strip_clone(clone)` call in `build_capture_clone` and re-ran
`capture.spec.ts`'s privacy test. First attempt showed a FALSE NEGATIVE —
the test stayed green — traced to `make test-widget` never rebuilding
`dist/` before running (the Playwright static server was serving a stale
`dist/capture.js` built before the mutation). Confirmed by rebuilding
by hand (`node scripts/build.mjs`) and re-running: the test correctly went
red (`input: expected no dark (text) pixels, sampled 11/49`). Reverted the
mutation, rebuilt again, confirmed green. This is exactly the "if it stays
green, the test is the bug" case quality-gate.md §5 warns about — except the
test itself was never the problem; the BUILD FRESHNESS was. Fixed at the
source: `Makefile`'s `test-widget` target now runs
`npm run build --workspace halle-feedback-widget-embed` before
`npm run test:widget`, so this class of false-negative can't recur silently
in any future milestone's widget testing, this run's or a later one's. Logged
here rather than glossed over, since a gate that can go green while the
regression it exists to catch is live is a real finding about the gate
itself, not a footnote.

The authenticated image-serving route (build plan §4.3's second bullet)
does not exist yet — that route is M10's own deliverable
(admin-v2-spec.md §5.3, the picture viewer). Logged so the check is not
forgotten: M10 must prove an unauthenticated request for a report image is
rejected, as part of its own gate.

The failed-capture path (§4.3's third bullet) is covered by
`capture.spec.ts`'s three "capture chunk failure" tests (404, throw, never
resolves) — all pass, all confirm the report still sends with the review
screen showing "(no picture)" and nothing else shown to the tester.

**Commit.** `feat: add M9 — picture flow changes on top of M6a/M6b` —
`src/widget/src/capture.ts` and `src/widget/src/loader.ts` only, verified
standalone-buildable and typecheck-clean against the pre-M8 `app.ts`/`types.ts`
(moved M8's new, still-untracked `marker-pen.ts`/`meta.ts` aside temporarily
to prove this, since they depend on M8's `types.ts` additions and would
otherwise falsely implicate this commit's own files in a typecheck failure
that is actually M8's dependency, not M9's).

---

## M8 — Widget v2, the tester-facing flow (2026-09-08, ~23:03)

Landed on top of M9 (see above for why the order is that way round, not
build-plan order).

**What was done — the state machine (`app.ts`, fully rewritten):**

- States: `idle -> choosing -> pointing (pointer mode only) -> review ->
  sent`. The five-answer question, the "where did you look for it?" step
  (never existed here, v1 already dropped it), the separate detail step and
  the consent step are all gone — replaced by ONE shared `review` screen:
  picture (boxed in for pointer mode) + marker pen + required comment box +
  the "What else we send with this" disclosure + Send/Cancel.
- Capture moves to selection/mode-choice time: `start_capture()` fires the
  instant a target is picked (pointer) or Screenshot is chosen, before
  `review` ever renders — widget-v2-spec.md §1, "no separate OK step between
  clicking and the picture being taken."
- Reused UNCHANGED, confirmed by diff review: element pointing, hover
  outline, capture-phase click interception (`install_picker`), element
  fingerprinting, the touch confirm flow, Shadow DOM isolation, the
  `content`/`clear()` stylesheet-survival fix from M6b, the focus trap.
- New: the mode chooser (`render_choosing`), the docked panel shell
  (`render_panel` — fixed header with optional Back/Close, scrolling body,
  used by choosing/pointing/sent; the review screen builds its own footer
  variant since it needs Send/Cancel there instead), the marker pen wiring
  (`create_marker_pen` from the new `marker-pen.ts`, strokes scaled from the
  on-screen canvas's CSS pixels up to the captured image's actual pixel
  dimensions before being sent), the `meta` capture and disclosure
  (`build_disclosure`, reading `meta.ts`'s `build_meta()`).
- `install_console_error_listener()` called once at `create_app` construction
  — widget-v2-spec.md §2.3, `window.addEventListener('error'/'unhandledrejection')`,
  capped at 5, cleared naturally on page load since it's a plain module-level
  array with no persistence.

**New files:**

- `meta.ts` — `build_meta()` (page/referrer/screen/viewport/device/browser/
  os/input/language/timestamp/console_errors) and the console-error listener.
  `detect_device_type()` deliberately kept separate from `device.ts`'s own
  `detect_device()` — different audiences (this one feeds the v2 disclosure
  JSON, the other still feeds the unchanged v1 `reports.device` column) —
  renaming the older one to match would have been an unrelated, unrequested
  change to a column still in active use.
- `marker-pen.ts` — `create_marker_pen(canvas)`: pointer-events based
  (mouse and touch alike, per widget-v2-spec.md §6), undo/clear, one colour
  one width, records strokes in the canvas's own CSS-pixel space (burn-in at
  the image's real resolution is M9's `burn_in_markup`, not this file's job).

**Config/schema fallout, fixed as part of "the payload changes"
(widget-v2-spec.md §9 effort table's own line item):** `lib/db/config.ts`'s
`Strings`/`ProjectConfig` shape rewritten for v2 (no more
`Option`/`DEFAULT_OPTIONS`/`ANSWER_IDS`/`launcherVisibility` — the launcher
is now unconditionally gated on `tester` resolving, widget-v2-spec.md §1);
`lib/api/config-schema.ts` (the string-editor save validator) and the
`strings-form.tsx`/`actions.ts` UI updated to match; `/api/v1/config/route.ts`
stops sending `options`/`launcherVisibility` in its response body;
`lib/db/report-list.ts` (and the M3-era read-only `/app/reports` page + its
CSV export) show `comment`/`mode` in place of the removed `answerLabel` —
that whole screen is still the M3 list, not yet replaced by M10's
Queue/Tracked-items views, so it was kept working against the v2 shape
rather than left broken between milestones.

**Tests.** All three widget spec files rewritten for the v2 flow (mode
chooser, no consent, required comment, marker pen). 34/34 passing. Web suite
346/346 (the config/report-list test files above updated to match).

**Quality gate:**

- `make lint` / `make build` / `make test` (346) / `make test-widget` (34) /
  `make size` — all green. `v1.js` grew from 6,203 to 7,629 bytes gzipped
  (49.7% of the 15,360-byte budget, 7,731 bytes of headroom) — the mode
  chooser, docked panel shell, comment box and marker-pen wiring account for
  the growth; `capture.js` unchanged from M9 at 10,300 bytes (33.5% of
  30,720). Both gates hold with real headroom to spare.
- Found and fixed during this milestone's own gate pass, before the
  computed-styles suite went green: two sub-16px font-size rules
  (`.icon-btn` at 15px, `.disclosure`/`.disclosure summary` at 14–15px) —
  agent-rules.md §2.5's 16px minimum, caught by `computed-styles.spec.ts`
  exactly as that file exists to catch a regression like this. Both bumped
  to 16px. Also settled, not left ambiguous: agent-rules.md §2.5's "56px for
  the widget's own options" now applies to the mode-chooser's two buttons
  (the closest v2 analogue to v1's five-answer options) — everything else
  interactive (Back, Undo, Clear, Cancel) sits at the general 44px floor.
  `computed-styles.spec.ts` asserts both floors separately.
- §2 attack list: N/A for this milestone's own new code (no new server-side
  mutation, no new database query) — the config/report-list changes above
  are read-path only, covered by the existing IDOR/tenant-scope tests
  already passing unchanged.
- §4 loop/TODO greps: clean.
- §5 adversarial review: covered together with M9's, since the two are
  read-order dependent (see M9's own entry above for the review's scope and
  findings — the `markup` bound and the stale `reports.ts` comment were
  found and fixed there, before either commit).

**Commit.** `feat: add M8 — widget v2, the tester-facing flow` — every file
above, staged by exact path. `Makefile`'s `test-widget` build-freshness fix
(found during M9's own gate work) staged via `git add -p`, taking only the
one hunk that is the fix itself — the pre-existing, unrelated tunnel/
user-password Makefile diff sits in the same file but a different hunk
range and was left unstaged, confirmed by re-diffing `--cached` before
committing.

---

## M10 — Admin v2 (2026-09-08, ~23:12–23:26)

**What was done.**

- `lib/db/report-queue.ts` (new) — `load_queue`/`load_tracked_items`
  (admin-v2-spec.md §5.1: status null vs. status in bug/fixed/closed,
  deleted never shown in either), grouped by template in the fixed display
  order (§6), each report named `<page label> #<n>` where `<n>` is a
  per-project running number computed once via a CTE over every report in
  the project (see the mutation-proof below for why it is a CTE and not an
  inline window function), `load_flat_list` (the next/previous scope for
  the picture viewer, same filters as whichever list opened it),
  `load_report_detail` (the one variant carrying the full `meta` JSON).
  Filters (template/mode/date-range/free-text search on `comment` only)
  combine as AND in one query.
- `lib/db/report-status.ts` — unchanged from M7 (`set_report_status`); this
  milestone is its first real caller.
- `app/app/report-status-actions.ts` (new) — `mark_bug_action`,
  `delete_report_action`, `mark_fixed_action`, `mark_closed_action`, all
  thin wrappers over `set_report_status` with no permission check beyond
  the session middleware already gates every `/app/*` route with
  (admin-v2-spec.md §1.4 — one login for everyone). No restriction on the
  current state before Fix it/Close it, confirmed by test and by the
  adversarial review below.
- `app/app/queue/page.tsx` + `queue-actions-form.tsx`, `app/app/tracked/page.tsx`
  + `tracked-actions-form.tsx` — the two list screens, Bug/Delete and Fix
  it/Close it respectively, both grouped by template with the shared
  filter form.
- `app/app/reports/[id]/page.tsx` + `viewer-image.tsx` (new) — the picture
  viewer: full image, comment, mode, (pointer mode only) element text/
  selector, the collapsed "What else we send with this" section reusing
  `meta` directly, click-to-toggle zoom, next/previous (mouse and
  ArrowLeft/ArrowRight) scoped to the exact filtered/sorted list that opened
  it via a `src=queue|tracked` param plus the same filter query params
  carried through.
- `app/app/screenshots/[id]/route.ts` (new) — the authenticated
  image-serving route. Sits under `/app/*`, so `middleware.ts`'s existing
  session gate covers it with no bespoke check of its own; looks up the
  report tenant-scoped by id first (a cross-project id 404s exactly like a
  made-up one), then reads the actual bytes through `get_storage().get()`.
- `app/app/report-filters.tsx` (new) — the shared template/mode/date/search
  filter form and `parse_filters`, used by Queue, Tracked items and the
  viewer's own next/previous scope.
- `lib/db/export-reports.ts` (new) + rewritten `app/app/export/reports/route.ts`
  — admin-v2-spec.md §5.6's unified CSV export: `id, page_url, template,
  mode, comment, target_text, target_selector, status, created_at` plus
  `browser`/`os`/`device_type`/`screen_size` flattened out of `meta`;
  `console_errors` and the rest of `meta` deliberately left out. Reuses
  `csv.ts`'s existing formula-injection guard verbatim.
- `app/app/layout.tsix` nav updated: Queue and Tracked items replace the
  gap left where Issues/Team used to be (M7 removed those, left the nav
  thin on purpose until this milestone).
- Login itself needed no changes — M7 already stripped `role` from the
  session cookie and every admin action; this milestone's own actions
  simply never had a role to check.

**Found and fixed during this milestone's own gate work, before the M3-era
`/app/reports` page and its CSV export were touched:** both still
referenced the removed `answerLabel`/`answer_id` filter — updated to
`comment`/`mode` (a de-facto M8 payload-change follow-through logged in
that milestone's own entry above, but the touch to this specific page and
export happened here, while working on the unified export).

**Real bug found by adversarial review, before this shipped, and fixed
before committing:** the running-number window function
(`row_number() over (partition by project_id order by created_at asc)`)
was first written embedded directly inside each filtered query. A window
function evaluates strictly AFTER that query's own WHERE clause, so
`load_report_detail` — a query whose WHERE always matches exactly one row —
always computed "#1" regardless of the report's real position, and a
report's number would shift depending on which filters happened to be
active when a list was queried. Every click from a list into the viewer
would have shown the wrong number on nearly every report. Fixed with a CTE
(`report_numbers`, numbered once, unfiltered, across the whole project)
joined into whichever query needs a row's number, so the number is fixed
the moment it's computed and identical no matter which list, filter
combination, or single-report lookup is asking. Reverted the fix by hand,
confirmed the new regression test in `report-queue.test.ts` goes red
exactly as expected (`expected 'Home #2' to be 'Home #1'`), restored the
fix, confirmed green again — the actual mutation-proof, not just a claim.

**Also found, harmless, worth noting:** two stale test files
(`tests/web/login-action.test.ts`, `tests/web/storage-upload-url.test.ts`,
`tests/web/middleware-revocation.test.ts`, `tests/db/users.test.ts`) still
inserted a `role: 'staff'` field into `users` rows — a field the table no
longer has since M7. This never failed any test because (a) this repo's
`tsc` typecheck scope is `src/web` only, so test files under `tests/` are
never statically checked, and (b) Drizzle's `.values()` silently ignores an
object key that matches no column rather than erroring, so the extra field
was quietly dropped on every insert rather than causing a runtime failure.
Removed from all four files. Logged because it is exactly the kind of
"passes today, would not catch a real regression" gap the quality gate
exists to surface — the test suite currently has a blind spot for schema
drift inside test files themselves, unrelated to this milestone's own
scope but found while working in this area.

**Tests.** `tests/db/report-queue.test.ts` (19, including the running-number
regression guard), `tests/db/export-reports.test.ts` (8, including the
formula-injection guard across all seven leading trigger characters and the
meta-flattening/no-leak check), plus the middleware matcher-coverage
addition to `tests/web/middleware-revocation.test.ts` (the authenticated
image route). 375 total, up from 354 pre-M10.

**Quality gate:**

- `make lint` / `make build` / `make test` (375) / `make test-widget` (34,
  unaffected) / `make size` (both budgets, unaffected) — all green.
- §1 mutation proof: the running-number CTE fix above, reverted and
  re-applied with the regression test as the actual proof.
- §2 attack list: IDOR — `load_report_detail`/`load_flat_list` both
  tenant-scoped, a cross-project report id resolves to undefined/absent
  everywhere, proved in `report-queue.test.ts`; the image route inherits
  the same guarantee plus the session gate from `middleware.ts`'s existing
  matcher (proved by asserting the matcher pattern itself covers
  `/app/screenshots/[id]`, since calling `middleware()` directly — this
  repo's only way to test middleware — bypasses Next's own path-matching).
  Unsafe query construction: `report-queue.ts`'s free-text search escapes
  `%`/`_`/`\` before building the `ilike` pattern, proved with a test that
  a comment literally containing `%`/`_` is matched as plain text, not as a
  wildcard. Wrong-type input: malformed report ids resolve to undefined,
  not a database error, at both `load_report_detail` and (already, from M7)
  `set_report_status`.
- §2.10 N+1: proved two ways in `report-queue.test.ts` — statically, that
  `report-queue.ts` contains at most two `db.select(...)` call sites total
  and neither sits inside a loop; functionally, that 20 reports across
  several pages all return correctly from one `load_queue` call.
- §2.11 mass assignment: N/A — no form in this milestone accepts a status
  or role field from the client; every status transition is a fixed
  literal baked into which action was called (`mark_bug_action` always
  writes `'bug'`, etc.), never a value read out of form data.
- §4 loop/TODO greps: clean across every M10 file.
- §5 adversarial review: run via a subagent given the full file list and
  admin-v2-spec.md's own acceptance criteria. One real bug (the
  running-number instability, above); one class of false alarm explicitly
  checked and ruled out (CSV export intentionally ignores active
  list filters and exports the whole project's reports — confirmed
  against the spec, not an oversight); confirmed no restriction was
  reintroduced on Fix it/Close it, no lingering role/permission references,
  and the CSV column list matches §5.6 exactly with no `meta` leak.

**Commit.** `feat: add M10 — admin v2 (queue, tracked items, picture viewer)`
— every file above, staged by exact path, plus the four stale-role test
file fixes (logged as their own finding, not folded silently into M10's
diff without explanation).

---

## Run summary — morning read

All four milestones completed in one continuous pass: **M7 → M9 → M8 → M10**
(M9 landed before M8 despite the numbering, per `v2-build-plan-for-agent.md
§7`'s "M9 lands as its own commit with nothing else mixed in" — see M9's own
entry above for why the build-plan's stated order and the actual commit
order differ on purpose). Four commits on `main`, in this order:

1. `feat: land M7 — database simplification for v2` (83 files)
2. `feat: add M9 — picture flow changes on top of M6a/M6b` (2 files)
3. `feat: add M8 — widget v2, the tester-facing flow` (23 files)
4. `feat: add M10 — admin v2 (queue, tracked items, picture viewer)`

No `Co-Authored-By` trailer on any of them. Nothing pushed to any remote —
none exists for this project. Final test count: 375 web/db tests + 34
widget acceptance tests, all green, both widget size budgets held with
real headroom (`v1.js` 7,629/15,360 bytes gzipped, `capture.js`
10,300/30,720).

**One process failure, caught and corrected before it caused lasting
damage:** M7's first `git commit` swept in ~11 pre-existing staged files
that had nothing to do with this run (tunnel scripts, `v1.js`/`capture.js`
routes, `widget-asset.ts`, password scripts, `live-test-plan.md`, parts of
`Makefile`/`package.json`) because staging new files on top of an
already-staged unrelated set does not scope a plain `git commit` to just
the new additions. Caught immediately by re-reading `git status` right
after committing, fixed with `git reset --soft HEAD~1` (purely local,
nothing pushed) plus `git restore --staged` on exactly the pre-existing
paths, then re-committed correctly. Full account in M7's own log entry
above. The lesson applied for the rest of the night: diff the staged file
list against the intended one before every subsequent `git commit`, not
after.

**One real bug found and fixed before it shipped:** M10's running-number
window function was filter-dependent (see that milestone's own entry) —
caught by adversarial review, fixed with a CTE, proved by an actual
revert-and-reapply against the regression test, not just asserted.

**One quality-gate blind spot found, not this run's to fully close:** four
test files across the repo (not just this run's own new files) carried a
stale `role: 'staff'` field left over from before M7 removed that column —
silently dropped by Drizzle rather than causing any test failure, because
this repo's `tsc` typecheck scope never covers files under `tests/`. Fixed
where found (four files); the underlying gap (test files are not
statically typechecked at all) is a repo-wide tooling question, not
something this run's own scope extends to deciding on its own.

**What is in `docs/v2-blocked.md`:** nothing — no genuinely blocked
decision came up overnight that the build plan's own pre-answered table
(§1) didn't already settle. Every judgement call actually made (the
console_errors bound on `markup`, the `<n>` running-number implementation
choice, the mode-chooser buttons inheriting the "56px option" floor) is
logged inline in the relevant milestone's own entry above instead, since
none of them were genuinely open questions — each had a specific, correct
answer derivable from the spec plus ordinary engineering judgement, not a
coin-flip needing a paper trail of its own.

**What would be next, if anything:** the M3-era `/app/reports` read-only
list and its CSV export are still live, side by side with the new Queue/
Tracked-items/unified-export screens — not a conflict (different routes,
different purposes: `/app/reports` is the raw append-only feed, Queue/
Tracked are the working views), but worth a deliberate decision in the
morning about whether `/app/reports` stays as a permanent "everything,
unfiltered" audit view or gets folded away now that Queue+Tracked cover the
same rows by status. Not decided here, since the build plan never asked
for that page to be removed and removing it was not part of any
milestone's stated scope.

Repo state: working tree clean except the pre-existing staged set from
before this run started (still exactly as untouched as it was at the
start — confirmed by `git status` one more time after the M10 commit),
plus this log file itself and `docs/v2-blocked.md`'s absence (never
created, since nothing was ever logged to it).
