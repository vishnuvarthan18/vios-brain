# Agent log

Newest at the top. One entry per queue item.
Any STOP reason is the first line of the file.

---

## T4-10..T4-14 · THE CUTOVER — manual marking removed  2026-09-19 (cloud session)

WHAT:     The last four items, including the irreversible one.

          T4-10  the 09:00-10:00 IST register window, enforced where a lead
                 marks, plus absent and not-yet-marked as different colours
          T4-13  a default timer on the task create form, copied onto the
                 release when it is opened
          T4-14  the student's own card shows when full points end
          T4-12  CUTOVER: 2026-09-19-g-scoring-cutover.sql

          Deleted: web/src/pages/Marking.jsx, web/src/pages/Board.jsx
          POST /api/mentor/score now answers 410 and names the Points screen.

TESTS:    39 + 377 + 50 back end, 21 browser, 0 failures, after the cutover.
          tests/releases.js 39/0. tests/tasks.js 45/4 — the same four as
          before, unchanged. The `g` migration was applied, applied twice, and
          refused correctly on a database without `f`.

OLD PATH: This is the entry where "what does the old path still do" is the
          whole job, so it was checked rather than reasoned about.
          · v_leaderboard was REBUILT over the automatic calculation with the
            SAME COLUMN NAMES. Every screen, export and report reading it gets
            the new numbers without being edited, and nothing can be left
            behind quietly reading the old world. That is the trick that made
            this a small change instead of a large one.
          · The four recalc triggers are dropped. Their FUNCTIONS are kept —
            they are what the archive would be read back with.
          · teams.project_points / quiz_points / total_points are zeroed and
            carry a COMMENT saying they are dead. Not dropped: a dropped
            column takes every query mentioning it down too. Zeroed because a
            stale 21.2 sitting in total_points is worse than an obvious 0 —
            someone will believe the 21.2.
          · `scores` keeps its rows. scores_legacy is a copy with a date, not
            a move.

FOUND:    Three, all before they shipped.

          1. THE WINDOW WAS ABOUT TO BE HUNG OFF A JSON ARRAY. The register's
             response is an array; an extra property on it is silently dropped
             by JSON.stringify. The page would have shown no window at all and
             nothing anywhere would have errored. It has its own endpoint.

          2. THAT ENDPOINT WAS REGISTERED AFTER /api/attendance/:day, so
             "/window" was read as a day, Number("window") is NaN, and it
             answered 400 — a route that exists, is registered and can never
             be reached. Same shape as the Journey bug. Moved above, with a
             comment saying it has to stay there.

          3. Pill had no "bad" tone, so tone="bad" fell through to the grey
             default — which is the same grey as everything else, so ABSENT
             and NOT-YET-MARKED would have rendered identically. That is the
             exact distinction T4-10 exists to draw, and an unknown tone is
             not an error, so nothing would have said a word.

ASSUMED:  The comparison was run on the FIXTURE, not on a real dump: 53 teams,
          990 points before, 4197 after, and NOT ONE TEAM WENT DOWN. The rise
          is attendance and the daily survey, which have always been recorded
          and have never been worth anything. That is the expected shape, but
          a fixture is not the real roster and T4-11 stays open until the same
          query has been run against a real dump.

          tests/tasks.js keeps its one obsolete assertion, failing, with a
          comment saying why. Deleting a test in the same breath as removing
          the feature it guarded is how a real regression gets out.

NOTE:     NOT DEPLOYED, and the deploy rule has not moved. On the live site
          every team still reads its old total and marking still exists.
          Before this goes anywhere near the server: take a dump, and run the
          comparison on real data.

NEXT:     Nothing queued in Track 4. The open items are the two secrets, the
          quiz questions, and T4-11 on a real dump.

---

## T4-06..T4-09 · SCORING v3 — the three screens  2026-09-19 (cloud session)

WHAT:     The admin adjustments screen, the live leaderboard, and the points
          timer on the release board.

          New:  web/src/pages/Adjust.jsx
                web/src/pages/BoardLive.jsx
                web/check-scoring.mjs
          Changed: web/src/pages/Open.jsx (the timer on each venue card),
                web/src/App.jsx, web/src/lib/nav.js,
                src/server.js (release_id + points_minutes on the release
                board payload — five per-venue blocks, all of them)

TESTS:    21 browser checks, all passing, including a REAL WRITE: add seven
          points with no reason typed, see the total move, undo it, see the
          total come back exactly and the row still listed as undone.
          Back end re-run after the server.js change: 39 + 370 + 50, no
          failures. tests/releases.js 39/0 and tests/tasks.js 45/4 — the same
          four that were already failing before any of this (see
          known-issues.md), unchanged.

OLD PATH: The old Board is untouched and still in the nav. Both are listed,
          deliberately: "Leaderboard" reads the old stored totals that every
          other screen uses, "Live board" reads the automatic calculation.
          They will disagree, and showing both is the honest way to say so
          until cutover. Both entries go when the old scoring does.
          The release board's existing behaviour is unchanged — the payload
          gained two fields and nothing else moved.

FOUND:    Two bugs, both caught by LOOKING at a screenshot, neither of which
          threw an error or failed any assertion that existed at the time.

          1. THE LIVE BOARD RANKED ON A NUMBER IT WAS NOT SHOWING. Ranking is
             by points per member by default — the teams are different sizes.
             The row showed the TOTAL. So the column read 131, 94, 121, 119
             down the page and the only sensible conclusion from looking at it
             is that the board is broken. Every number on screen was correct.
             The headline is now whichever number the rank was made of, with
             the other underneath. There is now a check for it: the big number
             on each row must go down the page.

          2. A CLOSED RELEASE SHOWED A DEADLINE THAT WAS NOT RUNNING. A closed
             row still carries opened_at from last time, so the timer printed
             "until 11:42 PM" beside a red Closed badge. Arithmetically right,
             completely misleading. It now reads "20 min from opening" until
             the thing is actually open.

          Also: a custom timer value — 20 minutes, say — matched no preset, so
          nothing on the row looked chosen and the timer read as unset. It
          gets its own chip now.

ASSUMED:  The timer went on the RELEASE BOARD, not the create forms. The
          release is where it belongs — it starts at opening, and the two
          venues open at different times, so one number on the item cannot be
          right for both. A default on the create form is still worth having
          and is queued as T4-13; it is a convenience, not the mechanism.
          Auto-cycling venues in projector mode was NOT built (T4-08 is marked
          accordingly). The venue is picked before going full screen.

NOTE:     src/public/v3/ is gitignored, so the built front end is NOT in the
          commit. Whoever pulls this must run `npm run build` in web/ or the
          three screens will not be there.

NEXT:     T4-10, the attendance window where a lead marks. Then the cutover,
          which stays blocked on Vishnu looking at /api/v3/scoring/compare.

---

## T4-01..T4-05 · SCORING v3 — automatic marks, built and tested  2026-09-19 (cloud session)

WHAT:     Automatic scoring, the live-leaderboard API, and the one admin screen
          that can change a number. Plan in docs/scoring-v3-plan.md. Nothing is
          switched over: every screen in the app still reads the old totals.

          New:  src/db/migrations/2026-09-19-f-scoring-v3.sql (+ -down)
                src/routes/scoring-v3.js   — 8 routes under /api/v3/
                tests/scoring.js           — the rules
                tests/scoring-routes.js    — the writes, every role, signed in
          Changed: scripts/seed/generate.js, tests/harness/routes.js,
                tests/harness/session-suite.js, src/server.js (one mount line)

TESTS:    459 assertions, 0 failures, at 209 students / 53 teams.
            tests/scoring.js          39 pass
            tests/harness/session-suite.js  370 pass  (was 294/16 before)
            tests/scoring-routes.js   50 pass
          Migration applied, run twice, rolled back, rolled back twice, and
          re-applied — clean each time.
          NOT RUN: behaviour.js, releases.js, tasks.js. Those are Playwright
          suites and this container has no chromium headless-shell. They must
          be run on the Mac before this is trusted. Nothing here touches the
          front end, so they should be unaffected — but "should be" is not a
          test result and is recorded as such.

OLD PATH: Deliberately untouched, and checked rather than assumed.
          teams.project_points / quiz_points / total_points, the recalc
          triggers, v_leaderboard and /api/mentor/score all still exist and
          still behave exactly as before. The migration is add-only: four new
          columns, three new tables, six new functions, four new views.
          GET /api/v3/scoring/compare exists to put old and new side by side
          before anyone decides to switch.

FOUND:    Five things, all worth more than the feature.

          1. THE OBVIOUS LEADERBOARD TOOK 8.9 SECONDS. Written first as
             SELECT team_auto_points(t.id) FROM teams — 53 function calls each
             re-scanning the same tables. A board that refreshes itself cannot
             take nine seconds. Rewritten set-based as v_team_day_points_v3:
             8.9s -> 25ms. Both versions are kept, and tests/scoring.js asserts
             they agree for EVERY team, because two ways of adding up points
             that disagree is the exact bug this rewrite exists to kill.

          2. THE ROUTE AUDIT WAS BLIND TO src/routes/*.js. It read server.js
             alone, so routes mounted from a module were invisible in BOTH
             directions — they never triggered "no row in the table", and a row
             naming one would have failed as "not found in server.js". Fixed;
             it now reads server.js plus every file in src/routes/.
             That immediately exposed 10 routes that had NEVER been asserted
             against any role: the 8 survey routes (which routes.js had
             predicted in a comment — "eight more that would arrive silently
             uncovered at the rebase" — and it was right), /api/drive/status
             and /api/profile/completion. All 10 now have rows.

          3. TWO OF THOSE 10 WERE GUESSED WRONG, and the suite said so.
             /api/drive/status is admin-only and /api/profile/completion is
             students-only, both enforced inside the handler where no grep of
             the middleware can see it. Guessing "auth" and being answered 403
             is the harness doing its job.

          4. THE FIXTURE HAD NO SURVEYS AT ALL. Eight survey routes could only
             ever be called with an id of 0 and answered 404. A route that has
             only ever returned "no such thing" is a route nobody has tested.
             generate.js now seeds four daily surveys, the final round, and
             deliberately partial answering (so "a half-answered survey scores
             nothing" has rows behind it, not just a unit test).

          5. THE DOWN MIGRATION IS NOT A NO-OP. It drops the points_minutes
             columns, so down-then-up leaves them EMPTY — no deadlines, every
             late hand-in silently becomes full points, and the totals go UP.
             Noticed because the leaderboard moved after a rollback cycle and
             it took a while to see why. Written into the down file as data
             loss, with the backup commands.

          Also: scripts/fake-data.js was written and then DELETED. T3-A1 was
          marked TODO in the work queue but scripts/seed/generate.js has been
          built for days. The queue was stale and a duplicate seeder was about
          to be committed on the strength of it. The real one was extended
          instead. Same failure as ux-fixes-applied.md: a document that stopped
          matching the code, believed because it was written down.

ASSUMED:  Four decisions taken rather than blocking on them overnight. All are
          one settings row or one line, and all are listed in the plan §7:
          ranking per member; no daily cap; teams may go negative; an admin may
          still mark attendance after 10:00.
          Also assumed: hand-in points are per team per activity ONCE, even for
          a per-student task. That is what Vishnu said, and it is the thing
          most likely to be read as a bug later, so it has its own test.

NEXT:     The three screens — admin adjustments, live leaderboard, the timer
          control on the create forms. Then, and only then, the cutover:
          archive scores into scores_legacy, recompute, delete manual marking.

---

## LANE B · HANDOVER — the split ends here  2026-09-19 23:10 IST

Lane B stands down. Five items DONE: A1, A2, A1b, A4, A5a. Everything is
committed and pushed to `v3-dev`. Nothing is half-finished and nothing is
uncommitted.

This entry is for whoever runs the single lane from here. It is the part that
is not obvious from the diff.

---

### 1. The things that will bite at the rebase

**The generator does not fill Lane A's survey tables.** `surveys`,
`survey_questions` and `survey_answers` all cascade from `students`, so when
the generator empties `students` their rows vanish **without an error**. The
fixture then looks healthy while carrying no survey data at all, and any
survey test run against it quietly measures nothing.

It now warns. Run the generator against a database with those tables and it
names all three. Adding them to `scripts/seed/generate.js` is the fix, and it
is a small one — the Day-9-empty-quiz precedent shows the shape.

**Eight survey GET routes would have arrived untested.** The route table in
`tests/harness/routes.js` is *iterated*, so a route missing from it is not
tested and nothing said so. It was complete for the 36 routes on `v3-dev`,
which is exactly why the gap was invisible. The audit now runs both ways and a
new route **fails the run** until someone records who may reach it. After the
rebase, expect eight failures naming the survey routes; that is the guard
working, and the fix is eight rows in the table, not a change to the guard.

**`2026-09-18-c-project-open-per-dept` must not be re-run.** See
`docs/migration-ledger.md`. It and `19-b` both rebuild `v_team_projects` and
create nothing else; `18-c`'s text has no `group_id` and the live view has it,
so re-running `18-c` silently replaces the current view with the older one and
drops the column. `src/server.js:444` and `:1542` read it with `SELECT *`, so
it would just stop arriving. On an existing database: skip 13, run 14 → 15.

---

### 2. What A1b found beyond the 500

The 500 itself is **Q4** and is the item on the critical path: task hand-in by
text or Drive link fails with `42P10`, because `ON CONFLICT (task_id, team_id)`
cannot infer a partial unique index. Two sites —
`src/server.js:1374` and `src/routes/drive-uploads.js:389`. One line each:
`ON CONFLICT (task_id, team_id) WHERE NOT per_student DO UPDATE`. Verified on a
copy: `tests/tasks.js` goes from 9 passing to 43.

Four other things came out of that item and are **not** the 500:

1. **`tests/tasks.js:283` is stale and will fail even after the fix.** It
   asserts the unique *constraint* exists by counting
   `pg_constraint WHERE contype = 'u'`. The uniqueness is a partial *index*
   now, so the count is zero against a correct database. Three of the five
   residual failures are this kind — the test being out of date, not the code
   being wrong. Do not "fix" the schema to satisfy it.

2. **A uniformly healthy fixture silently disables tests.** `releases.js` has
   four checks that look for a quiz with no questions and skip themselves —
   `if (rows.length)` — when the fixture has none. Every quiz had questions, so
   four real checks were passing without running. Day 9's quiz is now
   deliberately empty. **Worth grepping for other `if (rows.length)` guards
   before trusting any suite's green.**

3. **`tests/lib.js` and `src/server.js` disagree on the default staff
   password** — `'test-staff-pw'` against `'changeme'`. A browser suite run
   with neither the env var nor a `.env` fails at sign-in for a reason the
   output does not explain.

4. **`src/db/schema.sql` is stale against the live database** and must not be
   used to build a fixture database. Live has 23 tables and 13 views;
   `schema.sql` has 15 and 5. `teams.*_points` are NUMERIC live and INT there.
   Build scratch databases with
   `pg_dump --schema-only --no-owner --no-privileges -d bootcamp_local`.

---

### 3. The state of the harness

**Green:** `tests/harness/session-suite.js` — **265 checks, 0 failed**, at 209
students and 53 teams. `tests/harness/sabotage.js` — **13 proofs, 0 failed**.
`node tests/harness/run-suite.js releases` — 39 green.

**Two scratch databases**, both rebuildable and neither precious:
`bootcamp_seed` (the generator's own target) and `bootcamp_harness` (what the
harness reseeds on every run). **`bootcamp_local` holds the real dump and the
guard refuses to write to it** — that refusal is sabotage-proved.

**What the harness is isolated by:** its own database, its own port (3131), and
its own local-only fixture staff password set for the server it starts. It
never reads the real `.env`, so it cannot leak the real password.

**It refuses to run** if something is already listening on 3131. That is not
decoration: a leftover server made a working fix look ineffective, because the
requests reached stale code. If a run dies with `HARNESS INVALID`, free the
port rather than working around it.

**Writes are not covered.** The route table is GET-only. Write flows need rows
created and cleaned per case, and a write-flow suite is the obvious next piece
of the harness. It is not claimed as done.

**A fifth guard tier exists that no grep will show.** `/api/my-team`,
`/api/my-projects`, `/api/profile` and `/api/posts` carry `auth` as middleware
and then turn staff away *inside* the handler. `/api/me` contains the same
`kind !== 'student'` string in a nearby branch and is open to all four roles.
**This matters for B4's "prove it with a grep"** — a grep of the middleware
list will be wrong about these four. Two of them also answer a refusal with
**400**, which is what an input validator returns; they are asserted with a
separately named helper so the oddity stays visible.

---

### 4. On the guard lessons, from the agent that kept tripping them

All four fired today, and the fourth fired last.

- **Sabotage at the caller's level.** Proof 2's first version went red with
  `MODULE_NOT_FOUND` — this worktree has no `node_modules` of its own — so the
  suite failed without testing a single permission. Red for the wrong reason.
- **Sabotage the load-bearing line.** The seed guard's refusal, not its message.
- **A passing sabotage is a finding.** Proof 4 **passed** when it should have
  gone red. The cause was real: the static audit read `src/server.js` at its
  canonical path while the suite exercised a sabotaged *copy* — it was auditing
  a file nobody was running. Had I accepted the pass, the repo would carry a
  coverage guard that inspects the wrong file. It now reads `HARNESS_SERVER`.
- **Assert WHY it went red.** Both of the above were caught only by the
  "fails for the right reason" assertion, not by the red/green itself.

The ownership rule also earned its place. The sabotage proofs need a broken
`src/server.js`, and that file is not this lane's — so they build a throwaway
tree of symlinks with one file replaced. `git status src/` is clean after every
run. **That constraint is what produced the `HARNESS_SERVER` seam, and that
seam is what made Proof 4's defect visible at all.** Working around the rule
would have hidden it.

---

### 5. What I did not do, deliberately

- **Did not fix the task-submit 500.** `src/server.js` and `src/routes/` are
  not this lane's. Q4 has the diagnosis, the two line numbers, the one-line
  fix and the evidence it is sufficient.
- **Did not do A5b**, though it is nearly empty — see Q5. The order said stop
  after A5a, and it is now the single lane's call.
- **Did not touch `tests/lib.js`, `tests/readme.md` or any existing suite.**
  `releases.js` and `tasks.js` run against the fixture without being edited.
- **Did not deploy, ssh, or touch the production database.** No task in Phase A
  came close to needing any of them.

---

## LANE B · T3-A5a  DONE  2026-09-19 22:20 IST

WHAT:     Reader audit of `v_student_progress`. **READ-ONLY. Nothing fixed,
          nothing changed, no migration written** — A5a is the audit, A5b is
          the fix and it waits for `survey` to merge.

TESTS:    None to run: this is an audit. Evidence is a repo-wide grep plus the
          live view definition from `bootcamp_local`.

          The view has **33 columns**, two of which — `has_photo` and
          `has_education` — the JS weights dropped.

          **Every reader of the view, with file and line:**

          | # | File | Line | What it does |
          | --- | --- | --- | --- |
          | 1 | `src/server.js` | 3430 | `SELECT * FROM v_student_progress` — the only application reader |

          That is the complete list. One reader, one line.

          `/api/admin/progress` (admin-only) does `SELECT *` and hands the rows
          straight to the front end, so the real question is what the screen
          uses. `src/public/app.js:2546` consumes it and renders **eight**
          columns:

              has_goal · has_goal_3y · has_goal_5y · has_resume_v1 ·
              name · posts · roll_no · team_code

          **`has_photo` and `has_education` are not among them.** A repo-wide
          grep for either name across `src/`, `scripts/` and `tests/` returns
          nothing outside the migration and schema files that define them.

          Non-readers, checked and excluded:
          - `src/routes/profile-completion.js` — the authoritative weights.
            Reads `student_profiles` **directly**; zero mentions of the view.
            Six weights, no photo, no education, matching the brief.
          - `scripts/migrate-cvs.js` — zero mentions of the view.
          - `tests/migrate-cvs.js:222` — creates the `student_profiles` table
            with `photo_url`/`education` as a fixture, because the migration it
            tests recreates the view. That is the base table, not the view.
          - `src/db/*.sql` and `src/db/migrations/*.sql` — these **define** the
            view, they do not read it.

OLD PATH: This is the old path, and the news is good. The stale columns are
          genuinely orphaned: the JS weights dropped them, the view kept them,
          and nothing ever read them from the view. The danger the brief
          warned about — a view still answering a question the new weights no
          longer ask — did not materialise into a wrong number on a screen,
          because no screen asks the view for completeness at all. Completeness
          comes from `profile-completion.js`, which reads the table.

ASSUMED:  Nothing. The audit is a grep over a fixed tree plus the live view
          definition, and both are quoted above.

FOUND:    **A5b is smaller than `docs/phase-a-decisions.md` expects, and step 2
          of its plan has no work in it.**

          Phase A decisions A5 says, in order: (1) find every reader, (2) fix
          the readers that use it to decide completeness, (3) mark the view,
          (4) write up anything that genuinely needs the columns.

          Step 1 is done: one reader. **Step 2 is empty** — no reader uses the
          view to decide completeness, because the only reader is a `SELECT *`
          feeding a screen that ignores both columns. Step 4 is empty too:
          nothing needs them for anything.

          So A5b reduces to **step 3 alone** — a comment header on the view
          definition saying the two columns are deprecated, kept until Phase B,
          and are not a completeness signal. That is a one-file documentation
          change to a migration, and it does not touch `src/server.js`.

          **Which means A5b may not need to wait for `survey` to merge.**
          `docs/lanes.md` splits A5 because fixing the readers means editing
          `src/server.js` — but there are no readers to fix. Not acted on:
          the order says stop after A5a, and changing a migration file is not
          obviously inside this lane's ownership list either. Flagged for
          Vishnu as the cheapest item left in Phase A.

          Also worth recording: `src/db/schema.sql`'s copy of the view has
          **neither** `has_photo` nor `has_education`, while the live view has
          both. `schema.sql` is stale against the database generally — noted
          under A1 — and the view is one more instance. Anyone reading
          `schema.sql` to learn the view's shape gets 21 columns instead of 33.

NEXT:     **Stopping here, as instructed.** A3 (module split) and A5b both need
          Lane A's `survey` branch merged first. Queue items A1, A2, A1b, A4
          and A5a are DONE.

---

---

## LANE B · T3-A4  DONE  2026-09-19 21:50 IST

WHAT:     `docs/migration-ledger.md` — the real run order for all 15
          migrations, the marker each one was verified by, and the two that
          cannot be replayed blindly. Plus a pointer at the top of
          `src/db/migrations/readme.md`, which is this lane's file, saying it
          lists 9 of 15 and where the rest are.

TESTS:    Documentation only. **No migration changed, nothing run against any
          database.** Every check was a SELECT against local `bootcamp_local`.

          13 of 15 confirmed applied by a strong marker — a column, table or
          sequence the file creates. The two view-only ones were told apart by
          reading `pg_get_viewdef('v_team_projects')` and looking for a phrase
          only the later one introduces, which is the method Track 1 used.

          Also verified rather than assumed: every one of the 15 is a single
          BEGIN/COMMIT (checked, all 15); `18-c` contains no `group_id` at all
          (so the regression below is a fact, not an inference); and
          `v_team_projects` is read by two live routes with `SELECT *`.

OLD PATH: `src/db/migrations/readme.md` documents 9 and stops at 17 Sep. Its
          run order is correct as far as it goes and is carried into the ledger
          unchanged as items 1–9. What it is missing is the six from 18–19 Sep,
          and the fact that its "safe to run twice" claim does not extend to
          them — see the finding below.

ASSUMED:  That a marker's presence means its migration ran. It means the
          statement creating that marker ran; a file that died halfway would
          still show a marker placed early in the file. None of these is
          written that way — all 15 are one transaction — so the two agree
          here, but the ledger states the distinction rather than hiding it.

FOUND:    Three things, none fixed.

          1. **`2026-09-18-c-project-open-per-dept` must not be re-run on a
             live database.** It and `2026-09-19-b-project-view-by-group` both
             drop and recreate `v_team_projects` and create nothing else.
             `18-c`'s text has no `group_id`; the live view has it, so `19-b`
             is what the database holds. Running `18-c` now would silently
             replace the current view with the older one and drop `group_id`
             from it. `src/server.js:444` and `:1542` both read the view with
             `SELECT *`, so the column would just stop arriving, with no error
             anywhere. This is the one real hazard in the set: the run order
             cannot be replayed blindly, because two files fight over one view.

          2. **12 of 15 migrations have no `down`.** Rule 12 says every
             migration has a working one. Only `a-closed-by-default`,
             `a-quiz-per-student` and `a-tasks` do. All 15 are additive, so
             the risk is not data loss — it is that there is no written way
             back. Reversing `19-a-project-groups` by hand means knowing to
             drop a column, an index and a sequence **and** to restore the
             previous `chk_releases_item_id`, whose earlier text now survives
             only inside `18-b-project-formats`. Not fixed: writing downs for
             twelve already-applied migrations is Phase B work, and doing it
             tonight would mean editing files that have already run in
             production.

          3. **Item 4's marker is weak and is marked ⚠️ rather than ✅.**
             `a-closed-by-default` seeds a Day 1 attendance row in `releases`
             and creates nothing else — but an admin opening Day 1 attendance
             by hand creates the same row. The four rows present locally are
             equally consistent with the migration having run and with it never
             having run. It is additive and cheap to re-run, but re-running
             settles nothing either way. What settles it is a migration table,
             which is a Phase B argument, not a tonight one.

          The absence of any migration-tracking table is the root of all three.
          Every "is this applied?" answer in this repo is an inference from
          objects, and two of the fifteen cannot be answered at all.

NEXT:     T3-A5a — the `v_student_progress` reader audit. READ-ONLY, report
          only, no fixes.

---

---

## LANE B · T3-A1b  DONE  2026-09-19 21:15 IST

WHAT:     The `releases` and `tasks` suites now run against the generator.
          `tests/harness/run-suite.js` reseeds the scratch database, starts a
          server against it, finds the fixture's admin, and runs the suite with
          `BASE_URL`/`PGDATABASE`/`ADMIN_EMAIL` pointed at it — reseeding
          between suites, because each one writes rows.

          **Neither suite was edited.** Both already sign in through the real
          login and already read their fixtures out of the database. What they
          lacked was a database with the right shapes and a server pointed at
          it, which is what the runner supplies.

TESTS:    `releases.js` — **39 passing, 0 failing** at 209 students / 53 teams,
          including the venue-leak checks: opening a quiz for EEE does not
          expose it to ECE.

          `tasks.js` — 9 passing, then a true red on a real server bug (below).
          With the one-line fix applied to a throwaway copy it reaches **43
          passing**, which is the evidence that the bug is the whole blocker.
          Left red: the fix is not this lane's to make.

          Re-ran after the fixture change: session-suite still 229/0, sabotage
          still 10/0, generator still deterministic.

GUARDS:   One new guard, sabotage-proved as it was written.

          The harness now refuses to run if anything is already listening on
          its port. This is not theoretical: a leftover server from an earlier
          run held 3131, the suite attached to it, and a **working fix was
          reported as ineffective** because the requests never reached the
          patched code. A run against the wrong server has to stop, not carry
          on. Proved by putting a squatter on the port and watching the suite
          exit 1 with "HARNESS INVALID: something is already listening on
          http://127.0.0.1:3131", then freeing it and watching 229/0 return.

          The first version probed the port *after* spawning and raced its own
          child; it failed with a confusing sign-in error rather than the clear
          one. Moved before the spawn.

OLD PATH: `tests/releases.js` and `tests/tasks.js` both default `ADMIN_EMAIL`
          to Vishnu's real address, which no generated fixture contains, and
          both default `PGDATABASE` to `bootcamp_test`. Run bare, they fail at
          the first sign-in. Both honour the environment variables, so the
          runner supplies them and the files stay untouched.

          `tests/lib.js` is the shared helper for the browser suites and signs
          in properly, so those suites are not the untrustworthy kind. Not
          changed: `tests/lib.js` is not in this lane's ownership list.

ASSUMED:  The runner reseeds between suites rather than after each one, so the
          last suite's rows are left in `bootcamp_harness` for inspection. It
          is a scratch database that every run rebuilds, so nothing is lost.

FOUND:    **A real bug in the live code, written up as Q4 for Vishnu.**

          `POST /api/tasks/:id/submit` returns 500 for every task whose
          `submission_type` is `text` or `drive`. `server.js:1374` uses
          `ON CONFLICT (task_id, team_id)`, and there is no longer a plain
          UNIQUE to infer — the `a-quiz-per-student` migration replaced it with
          two PARTIAL unique indexes split on `per_student`. Postgres will not
          use a partial index as an arbiter unless the statement repeats the
          predicate, so it raises 42P10.

          **It reproduces against `bootcamp_local`, loaded from the real dump**
          — so it is the live schema, not a fixture artefact. Nobody has hit it
          because all 56 `task_submissions` rows in the dump are `per_student`,
          i.e. the photo-upload path, which does not reach this statement.

          `src/routes/drive-uploads.js:389` has the identical statement and its
          comment still cites the UNIQUE the migration removed. Same defect,
          not reproduced only because that path needs a file upload.

          Not fixed: `src/server.js` and `src/routes/` are Lane A's for the
          whole of Track 2, and rule 19 says a finding is written down rather
          than fixed. The fix is one line in each site, verified on a copy.

          Also: `tests/tasks.js:283` asserts the unique **constraint** exists by
          counting `pg_constraint WHERE contype='u'`. Stale for the same reason
          — the uniqueness is an index now — so it fails against a correct
          database. Three of the five residual failures are the test being out
          of date, not the code being wrong.

          This is the second time the partial-index discrepancy has surfaced:
          it was logged under A1 from reading the schema, and here it turned
          into a 500 in a real request. It is the same root cause.

          One fixture change was needed, and it is worth stating plainly.
          `releases.js` has four checks that look for a quiz with NO questions
          and skip themselves when there is none — `if (rows.length)`. The
          generator gave every quiz five to eight, so those four silently
          passed. Day 9's quiz is now deliberately empty. A fixture that is
          uniformly healthy turns real checks into no-ops.

NEXT:     T3-A4 — the migration ledger.

---

---

## LANE B · T3-A2  DONE  2026-09-19 19:40 IST

WHAT:     Session test harness. Every test signs in through the real login and
          carries the session; there is no way to make a request without one.
          `tests/harness/` — `index.js` (reseed, start, sign in, tear down),
          `session.js` (cookie-carrying client), `server.js` (isolated server),
          `roles.js` (four roles from the fixture), `routes.js` (the route
          table), `expect.js`, `session-suite.js`, `sabotage.js`, `readme.md`.

TESTS:    `session-suite.js` — **229 checks, 0 failed**, at 209 students and
          53 teams, every one through a real signed-in session. 37 read routes
          × the roles that may reach them and every role that must not, plus
          anonymous refused on all 37.

          Also: the four roles hold real cookies, a lead signs in AS a lead,
          a student with no team gets an answer rather than a 500 on four
          screens that assume a team, and logout really ends the session.

GUARDS:   **`sabotage.js` — 10 checks, 0 failed.** Three guards, each broken
          at the level it claims to protect, each asserting red-when-broken,
          green-when-restored AND that it went red for the RIGHT reason:

          1. Seed guard: with the scratch-name check removed, `bootcamp_local`
             is accepted. The guard is what refuses it.
          2. Session suite: `/api/admin/students` loses `require_admin`, and
             the suite fails naming that route and the mentor role.
          3. The harness itself: `auth` stops requiring a cookie, and the
             harness refuses to run at all with HARNESS INVALID rather than
             reporting results that would be meaningless.

          The "right reason" assertion earned its place immediately. Proof 2
          was vacuous on the first run: the sabotaged server died with
          MODULE_NOT_FOUND — this worktree has no `node_modules` of its own
          and resolves from the main checkout — so the suite went red without
          testing a single permission. Exactly the 19 Sep near-miss shape: red
          for a reason that is not the hole that was cut. Fixed by linking
          node_modules into the throwaway tree.

          Proof 3 then failed for the opposite reason: the guard fired
          correctly but throws, so HARNESS INVALID lands on stderr and the
          check was only reading stdout. The test was wrong, not the code.

          **No Lane A file was edited, at any point, including transiently.**
          `docs/lanes.md` makes that a stop-work event whatever the reason, and
          writing a file then restoring it is still writing it. Proofs 2 and 3
          need a broken `src/server.js`, so they build a throwaway tree of
          symlinks with only that one file replaced, and point the harness at
          it through `HARNESS_SERVER`. `git status src/` is clean after a run.

OLD PATH: The existing suites (`tests/flows.js`, `behaviour.js`, `redesign.js`,
          `onboarding.js`) drive a real browser through `tests/lib.js`, which
          signs in properly — so they are not the untrustworthy kind. They are
          left alone: they target `localhost:3099` and a database the runner
          supplies, where this harness starts its own server on 3131 against a
          scratch database. Both can run; they must not run at once, since the
          browser suites write real rows. `tests/readme.md` already says the
          suite is for a scratch database. Not changed — `tests/readme.md` is
          not in this lane's ownership list, and A1b is the item that makes
          the `releases` and `tasks` suites run against the generator.

ASSUMED:  A local-only fixture staff password is set for the server the
          harness starts, rather than reading the real one from `.env`.
          `docs/phase-a-decisions.md` A2 permits exactly this and asks that it
          be said in the log: **said here.** It is not a secret — it exists
          only in `tests/harness/env.js`, only for a server the harness starts,
          only against a scratch database. Nothing in this lane reads or prints
          the real `STAFF_PASSWORD`. If a `.env` is ever added to this
          worktree, the harness still wins, because a real environment
          variable beats the file.

          Only GET routes are in the table. Writes need rows created and
          cleaned per case; enumerating them here would mean POSTing to
          `/api/admin/teams` eighty times a run. A write-flow suite is the
          natural next piece and is not claimed as done.

FOUND:    Three things noticed, none fixed:

          1. **`student` is a fifth guard tier that no grep will show.**
             `/api/my-team`, `/api/my-projects`, `/api/profile` and
             `/api/posts` carry `auth` as middleware and then turn staff away
             inside the handler. A middleware-level audit reads them as "any
             session". Found by calling every route as every role — `/api/me`
             contains the same `kind !== 'student'` string in a nearby branch
             and is in fact open to all four. Anything that reasons about
             permissions by grepping the middleware list will be wrong about
             these four. Relevant to B4's "prove it with a grep".

          2. **Two routes answer a refusal with 400.** `/api/my-team` and
             `/api/my-projects` return 400 "Students only" where the other two
             return 403. 400 says "your request was malformed" and is what an
             input validator returns, so a permission check that accepts 400
             as proof of a guard would also pass a route that simply failed to
             parse its parameters. Asserted with a separately named helper so
             the oddity stays visible instead of being smoothed over.

          3. `tests/lib.js` defaults `STAFF_PASSWORD` to `'test-staff-pw'`
             while `src/server.js` defaults to `'changeme'`. A browser suite
             run with neither the env var nor a `.env` fails to sign staff in,
             for a reason the output does not explain. Not this lane's file.

NEXT:     T3-A1b — make the `releases` and `tasks` suites run against the
          generator.

---

---

## LANE A · T2-09 – T2-18  DONE  2026-09-19 19:10 IST  — TRACK 2 COMPLETE

WHAT:     The four proof reports, the chase list, the venue-leak suite, the
          four-role session suite, the old-path check, the admin Surveys
          screen, nine screenshots, and the deploy command set below.
          Branch `survey`, 26 commits ahead of main, pushed.

TESTS:    266 checks across seven suites, every one through a real signed-in
          session against a real dump:

            survey              101
            survey-sessions      52
            task-upsert          40
            screens              25
            survey-old-path      24
            survey-venue-leak    20
            survey-routes         4

          Existing suites unchanged from the baseline taken before any
          survey work: flows 21/1, behaviour 15/0, redesign 33/0,
          onboarding 1/0, completion 34/0. The one flows failure is the
          unopened projects, which predates this and is not mine.

OLD PATH: Checked in three states -- no survey at all, one closed for this
          venue, one open -- and the first two are indistinguishable from
          the app as it was. Team points byte-identical after 824 answers,
          leaderboard order unchanged, and task_submissions, attendance,
          daily_posts and quiz_attempts row counts untouched. Verified
          against teams that actually HAVE points, because every team
          reading 0.0 satisfies "unchanged" whatever the survey does.

FOUND:    Six items I had marked DONE were not. T2-03 and T2-08 through
          T2-12 had their APIs built and no screen: there was no way to
          reach the loader, the results or the proof report without curl.
          T2-17 is what caught it, and nothing else would have. An API is
          not a screen and the queue said screen.

          Six false greens, all mine, all in one feature, by an agent that
          knew the pattern:

            1. sabotage at an intermediate, not the caller
            2. sabotage of a plausible-looking line, not the load-bearing one
            3. an assertion satisfied by degenerate data (every number 1)
            4. real volume seeded AROUND the route under test
            5. a query compared against a fresh read of itself
            6. a suite reporting 0 pass / 0 fail and reading as success

          The last one recurred after being fixed: the scratch-database
          guard silently zeroed five suites a second time. It now prints
          FAIL and exits non-zero, so a refusal can never read as a pass.

ASSUMED:  The admin Surveys screen sits beside the quiz loader in the nav
          and follows its shape. The brief says not to invent a new place
          for it while the v3 nav is being rebuilt.

RISK:     Nothing here is live. Every task remains a photo upload until the
          cutover, so the task-submit fix exists in the code and cannot
          fire. The survey is closed for both venues until an admin opens
          it, and a survey with no questions cannot be opened at all.

NEXT:     Vishnu reviews docs/screens/, especially 02. Then `survey` merges
          to main, `v3-dev` rebases, and one lane starts A3.

---

## T2-18 — THE DEPLOY COMMAND SET.  DO NOT RUN THIS YET.

**No deploy until the whole build is finished.** This is written down so it
exists when that moment comes, and because writing it now is how the ownership
step stops being forgotten. It has been rehearsed against a local copy of the
19 Sep dump; it has not been run against anything else.

Payload commit: **574614e** on `survey` (26 commits ahead of `main`).
Migration: `src/db/migrations/2026-09-19-c-surveys.sql`, additive, safe to run
twice, with a working `down` beside it.

### Before anything

```sh
# 1. A fresh dump, and CHECK ITS ROW COUNTS -- not merely that it is valid gzip.
ssh hetzner 'sudo -u postgres pg_dump bootcamp | gzip > /var/backups/bootcamp/pre-survey-$(date +%F-%H%M).sql.gz'
ssh hetzner 'ls -lh /var/backups/bootcamp | tail -3'
ssh hetzner "zcat /var/backups/bootcamp/pre-survey-*.sql.gz | grep -c '^INSERT INTO public.students' "
```

A dump that is 19K, or that counts zero students, is not a backup. Stop if
either is true.

```sh
# 2. Nobody is working. Not "probably nobody" -- look.
ssh hetzner "sudo -u postgres psql -d bootcamp -tAc \
  \"SELECT COUNT(*) FROM students WHERE last_login > now() - interval '15 minutes'\""
```

### The payload

```sh
# 3. Built with git archive of a NAMED COMMIT. Never rsync from a working
#    tree -- rsync ships whatever is on disk, including another lane's
#    uncommitted edits.
COMMIT=574614e
rm -rf /tmp/deploy-payload && mkdir -p /tmp/deploy-payload
git archive "$COMMIT" | tar -x -C /tmp/deploy-payload

# 4. Ship it. The excludes are not optional: --delete is on both lines and
#    uploads/ holds student CVs that are in NO database dump.
rsync -az --delete \
  --exclude node_modules --exclude .git --exclude .env \
  --exclude uploads --exclude .archives --exclude logs \
  /tmp/deploy-payload/ hetzner:/opt/bootcamp-dashboard/
```

### Migration, ownership and restart — ONE STEP

These three are one step. A gap between them is an outage: the database is
correct, every object is owned by `postgres`, and the app -- connecting as
`bootcamp` -- returns 500 on every route. This took the live app down twice
during v2. Run them together, in this order, without pausing to check
anything in between.

```sh
# 5a. The migration.
ssh hetzner 'sudo -u postgres psql -v ON_ERROR_STOP=1 -d bootcamp \
  -f /opt/bootcamp-dashboard/src/db/migrations/2026-09-19-c-surveys.sql'

# 5b. Reassign ownership of everything it created.
ssh hetzner "sudo -u postgres psql -d bootcamp <<'SQL'
DO \$\$
DECLARE r RECORD;
BEGIN
  FOR r IN SELECT tablename FROM pg_tables WHERE schemaname='public' AND tableowner<>'bootcamp' LOOP
    EXECUTE format('ALTER TABLE public.%I OWNER TO bootcamp', r.tablename);
  END LOOP;
  FOR r IN SELECT viewname FROM pg_views WHERE schemaname='public' AND viewowner<>'bootcamp' LOOP
    EXECUTE format('ALTER VIEW public.%I OWNER TO bootcamp', r.viewname);
  END LOOP;
  FOR r IN SELECT c.relname FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace
            WHERE n.nspname='public' AND c.relkind='S' AND pg_get_userbyid(c.relowner)<>'bootcamp' LOOP
    EXECUTE format('ALTER SEQUENCE public.%I OWNER TO bootcamp', r.relname);
  END LOOP;
  FOR r IN SELECT p.proname, pg_get_function_identity_arguments(p.oid) args
             FROM pg_proc p JOIN pg_namespace n ON n.oid=p.pronamespace
            WHERE n.nspname='public' AND pg_get_userbyid(p.proowner)<>'bootcamp' LOOP
    EXECUTE format('ALTER FUNCTION public.%I(%s) OWNER TO bootcamp', r.proname, r.args);
  END LOOP;
END \$\$;
SQL"

# 5c. Restart. The running process still holds what it loaded before the
#     migration, so until this runs the fix has not reached the app.
ssh hetzner 'sudo systemctl restart bootcamp'
```

### Verify — watch the log ACROSS the restart, not just that it came back

```sh
# 6. Nothing left owned by postgres. This must print NOTHING.
ssh hetzner "sudo -u postgres psql -d bootcamp -tA \
  -c \"SELECT tablename FROM pg_tables WHERE schemaname='public' AND tableowner<>'bootcamp'\" \
  -c \"SELECT viewname FROM pg_views WHERE schemaname='public' AND viewowner<>'bootcamp'\""

# 7. The three tables exist and are empty.
ssh hetzner "sudo -u postgres psql -d bootcamp -tAc \
  \"SELECT (SELECT COUNT(*) FROM surveys) || ' / ' || (SELECT COUNT(*) FROM survey_questions)\""

# 8. Points did not move. Compare against the same query run BEFORE step 5a.
ssh hetzner "sudo -u postgres psql -d bootcamp -tAc \
  \"SELECT SUM(total_points) || ' over ' || COUNT(*) || ' teams' FROM teams\""

# 9. The log, across the restart. The one v2 outage was found this way and
#    missed the time it was not done.
ssh hetzner 'sudo journalctl -u bootcamp --since "5 minutes ago" | tail -40'

# 10. A real sign-in, as a real student, through the real login.
```

### Rolling back — about 90 seconds

The migration is additive: nothing is dropped, renamed or rewritten, so the
**previous code runs unchanged against the new schema**. That makes the
rollback a code rollback, and the down migration is not part of it.

```sh
git archive <previous-main-commit> | tar -x -C /tmp/rollback-payload
rsync -az --delete --exclude node_modules --exclude .git --exclude .env \
  --exclude uploads --exclude .archives --exclude logs \
  /tmp/rollback-payload/ hetzner:/opt/bootcamp-dashboard/
ssh hetzner 'sudo systemctl restart bootcamp'
```

Only run the down migration if the tables themselves are the problem, and
read it first: it refuses while any answer exists and takes `-v force=1` to
destroy them. Those answers are the only record of what a student knew before
a topic was taught, and that cannot be reconstructed afterwards.

### What stays off after this deploy

- **Every task remains a photo upload** until the cutover. The task-submit
  500 fix ships in the code and cannot fire until a text or Drive task is
  created.
- **No survey is open.** A survey is closed for both venues until an admin
  opens it on the Open tab, and one with no questions cannot be opened at all.
- **The final round asks for a confirmation** naming the real question count.
  Opening it early is unrecoverable.

### Merge debt this deploy does not resolve

`docs/merge-debt.md` M1 through M4. M1 is the one to watch: `run.sh` lists
nine chase queries and query 10 lives on this branch, so after the merge the
chase set silently produces nine CSVs instead of ten. No error, just a list
that is quietly missing.

---

## LANE A · LIVE FIX — task hand-in 500  DONE  2026-09-19 18:45 IST

WHAT:     POST /api/tasks/:id/submit returned 500 for every text and
          Drive-link hand-in; drive-uploads.js had the identical defect.
          Found by Lane B during A1b, correctly refused there because
          src/server.js is Lane A's. Fixed ahead of T2-16 so it ships with
          the Track 2 deploy.

CAUSE:    task_submissions has no plain UNIQUE (task_id, team_id). The
          per-student migration replaced it with two PARTIAL unique indexes,
          and Postgres cannot infer a partial index from a bare column list.
          `ON CONFLICT (task_id, team_id)` matched nothing and raised
          "there is no unique or exclusion constraint matching the ON CONFLICT
          specification".

          Reproduced against the real dump before changing anything.

FIX:      Name the predicate, and branch on the task's mode -- which index
          applies is decided by tasks.per_student, copied onto the row by
          trg_sync_submission_mode. A fix naming only the team predicate
          would leave every per-student task broken, and both live tasks are
          per-student.

          Also fixed a second defect found while reading: the "already
          marked" check asked by team even for per-student tasks, so one
          marked teammate would have blocked every other member's own
          hand-in.

TESTS:    tests/task-upsert.js -- 40 checks, both modes x both submission
          types, through the real route with real sessions. It asserts the
          mode actually changes behaviour: a teammate gets their OWN row in
          per-student mode and replaces the single team row otherwise. A fix
          using one index for both passes everything else and fails there.

          tasks.js goes 7 -> 9 on a clean database loaded from the dump.
          Lane B reported 9 -> 43; I could not reproduce 43 and did not
          claim it. The remaining 5 failures are a FIXTURE mismatch, not
          this bug: the suite asserts `creating a task does not open it` by
          counting task releases and expecting 0, and the real dump carries
          9. Verified by reading the rows. Not mine to fix -- tests/ is
          shared and that assertion belongs to whoever owns the fixture.

SABOTAGE: Both sites, and the reason asserted rather than just the redness,
          per Lane B's rule. Restoring the bare column list in server.js
          turns 6 checks red AND puts "no unique or exclusion constraint
          matching the ON CONFLICT specification" in the server log -- the
          actual cause, not a coincidental failure. The drive-uploads site
          was proved the same way directly against the database, since its
          route needs Drive credentials this agent must never hold.

OLD PATH: The comment at the drive-uploads site said "step 4 puts
          UNIQUE (task_id, team_id) on it", which was true when written and
          false since the per-student migration. The stale comment is what
          made the defect invisible to review -- it asserted the very
          constraint that had been removed.

ASSUMED:  A per-student task's "already marked" check is per student. The
          alternative -- one marked member locking the team -- contradicts
          the whole point of per-student mode.

FOUND:    tests/tasks.js is not idempotent and assumes an empty releases
          table. Against a real dump it cannot distinguish a genuine break
          from its own fixture assumptions, which is why this bug survived
          in a suite that was already failing.

NEXT:     T2-16, the old-path check.

---

## LANE B · T3-A1  DONE  2026-09-19 18:05 IST

WHAT:     Deterministic fake-data generator: 209 students, 53 teams, two
          venues, nine days, from a seed. `scripts/seed/` — `generate.js`,
          `guard.js`, `random.js`, `names.js`, `readme.md`.
          Branch `v3-dev`, own worktree `.worktrees/lane-b`.

TESTS:    Run at full volume against `bootcamp_seed` (a scratch database
          created from `bootcamp_local`'s schema, schema only, no data).
          209 students / 53 teams / 6 teamless / 53 leads, 20 tasks,
          423 task hand-ins, 248 projects, 1068 quiz attempts,
          304 assessment attempts. Runs in ~2s.

          Determinism: same seed twice gives a byte-identical fingerprint
          across students, teams, attendance, releases and quiz_attempts,
          INCLUDING primary keys; seed 7 differs. First attempt failed this —
          sequences were not reset, so the logical roster matched but every
          id shifted. Fixed by resetting every sequence in `public`.

          PII: zero of the 209 generated names, roll numbers, phones or
          emails collide with the real dump, compared by hash. Nothing in
          `names.js` is copied from the roster; phones sit in a documentation
          range that is never allocated.

GUARDS:   `guard.js` refuses any target that is not loopback AND not marked
          scratch. Verified it refuses `bootcamp_local` (holds the real dump),
          `bootcamp` (the app database) and a remote host. **Not yet proved by
          sabotage** — that is written as a harness check under A2, per the
          standing rule that a guard is not a control until watched failing.

OLD PATH: `src/db/schema.sql` is stale against the live database and should
          not be used to build a fixture database. Live has 23 tables and 13
          views; schema.sql has 15 and 5 — no `tasks`, `task_submissions`,
          `releases`, `assessment_*`, `attendance_audit`. `teams.*_points` are
          NUMERIC live but INT in schema.sql, and `v_student_progress` in
          schema.sql lacks `has_photo`/`has_education`, which the migrations
          add. The generator therefore targets a database built from a
          `pg_dump --schema-only` of the loaded dump, not from schema.sql.
          This is a documentation problem, not a code one; recorded for A4.

ASSUMED:  Fixture venue/team split mirrors the real one exactly (EEE 14 teams
          / 55 students, ECE 39 / 154) because the brief states it. If wrong,
          only volume shifts; no test logic depends on the exact split.
          Team dept and student dept always agree in the fixture. The brief
          says nothing enforces this in real data and asks for mismatches to
          be reported as a count — manufacturing them here would make the
          fixture disagree with the rule the app is being built to, so the
          mismatch case belongs in a test that inserts it deliberately.
          Scratch-name marker set to seed|fake|fixture|dev|scratch|harness.
          If a real database is ever named that way, the guard would allow it;
          `bootcamp_local` and `bootcamp` are explicitly excluded by not
          matching.

FOUND:    Two things noticed, neither fixed (out of scope, logged only):

          1. `grade_quiz_attempt()` ends with
             `submitted_at = COALESCE(submitted_at, now())`. Calling it on an
             unsubmitted attempt silently marks it submitted. It bit the
             generator — "opened the quiz but handed in nothing", which is
             chase list 4, emptied itself to zero. Worked around in the
             fixture by grading only submitted attempts. If any server route
             calls this function on a live in-progress attempt, that route
             closes a student's quiz early. Not investigated; `server.js` is
             Lane A's for the whole of Track 2.

          2. `task_submissions` has no plain `UNIQUE (task_id, team_id)` as
             the brief states. It is a pair of PARTIAL unique indexes split on
             `per_student`: `idx_task_sub_team (task_id, team_id) WHERE NOT
             per_student` and `idx_task_sub_student (task_id, submitted_by)
             WHERE per_student AND submitted_by IS NOT NULL`. Correct in
             spirit, but anything written against the brief's wording will be
             wrong for per-student tasks. Zero duplicate groups in real data.

NEXT:     T3-A2 — the session test harness, and the sabotage proof for the
          seed guard.

---

---

## LANE A · T2-04 – T2-08  DONE  2026-09-19 17:20 IST

WHAT:     The Open tab row, the student card and answering screen, the final
          round end to end, and today's results with names behind every count.
          Plus two guards: the three-state rule and route agreement.
          Branch `survey`, pushed.

TESTS:    66 checks in tests/survey.js, every one through a real signed-in
          session for an admin and a student. tests/survey-routes.js: 4.
          Browser-verified at 390px: the card is first and tagged Required,
          other items stay usable, both buttons lock on the first tap, the
          answer survives a refresh, no horizontal scroll, no page errors.

          Results at a realistic volume -- ECE answered, EEE not started --
          read 35 yes / 116 no / 55 not asked of 206, which is the case the
          three-state rule exists for.

GUARDS:   Both sabotage-tested per the standing rule.

          Three states: stripping not_asked from the RESPONSE turns four
          checks red. The first attempt, stripping it from the initialiser,
          PASSED -- a later line put the property back -- which is now the
          worked example in standing-authorisation.md.

          Route agreement: removing the pages_for() entry turns the
          silent-bug check red; removing the route turns the other red.

OLD PATH: The Open tab needed no edit -- it draws whatever /api/admin/releases
          returns, so matching the quiz's shape gave the two venue columns,
          the open/close buttons and "Open for both" for free. Verified in a
          browser rather than assumed.

          flows and redesign unchanged from baseline (21/1, 33/0) after the
          home-page change. The one flows failure is still the unopened
          projects, not the survey.

ASSUMED:  The final round shows on the Open tab every day rather than only on
          day 9, now with a confirm naming the real question count. Opening it
          early is unrecoverable, so the guard is a tap rather than hiding the
          row -- hunting for it on the last day is the worse failure.

FOUND:    tests/gates.js and its Makefile line were written by me BEFORE
          docs/lanes.md assigned that file to Lane B. Committed and pushed on
          `survey` in e9ae524 and earlier. Not touched since reading the lanes
          doc. v3-dev was cut from cb7ac5d, which predates it, so Lane B will
          find the file arriving underneath them at the rebase. Written up as
          Q3 in questions-for-vishnu.md with the four calibrated budgets, so
          Lane B can take them rather than re-deriving the numbers.

          The survival split corrected my own audit twice: venue literals are
          37 not 6, and 24 of those die with app.js, so B2's real scope is 13.
          Activity-type gates do not shrink at all -- all 17 are server-side.

NEXT:     T2-09 to T2-12, the four proof reports.

---

## FOUND: enumerating gates  2026-09-19 16:40 IST

Four bugs in two days were the same bug. Writing down all of them on purpose,
because finding them by accident has cost a day.

### The shape

A concept is gated by a list of its allowed values. A new value is added. The
list is not. The new value then fails at ONE of its gates while passing the
others, so the feature half-works -- which is worse than not working, because
the failure appears far from the change and only under the exact conditions
that reach that one gate.

The four found so far:

| # | Gate that was not updated | How it showed up |
| --- | --- | --- |
| 1 | `set_release()`'s `ALLOWED` array | isOpenFor had its survey branch and was right. Every release row was refused anyway: "Not something that can be opened" |
| 2 | `chk_releases_item_id` CHECK | Would have accepted item_type='survey' and then refused every row carrying an item_id -- first visible when staff opened a survey in front of a room |
| 3 | `pages_for()` in app.js | The route existed, the page function existed, the API returned correct data, and `go('survey')` silently redirected home |
| 4 | v2's `is_lead` / `is_team_lead` | The original: one name in one place, its old name read in another. 156 of 209 students locked out of the quiz screen |

Three of those four are in the SAME feature, added by me, in one day. That is
the measure of the problem: knowing about the pattern did not stop me walking
into it three more times.

### Every enumeration of an activity type in this codebase

Adding a sixth activity type today means editing all of these. The count is
the point.

**Server -- src/server.js**

| Line | What it enumerates |
| --- | --- |
| 293-340 | `isOpenFor()` -- one `if (item_type === ...)` branch per type, then a final `return item_type === 'attendance'` that silently catches everything unlisted |
| 1662 | `const DEPTS = ['EEE', 'ECE']` -- the venue enumeration, same shape, different concept |
| 1689-1878 | `GET /api/admin/releases` builds `items[]` with one hand-written block per type: quiz, survey, task, project, then a loop over the three id-less types |
| 1856 | `for (const type of ['attendance', 'pre_assessment', 'post_assessment'])` |
| 1859 | the label map `{ attendance: ..., pre_assessment: ..., post_assessment: ... }` |
| 1895 | `set_release()`'s `ALLOWED` array -- **bug 1** |
| 1904 | which types must carry an `item_id` -- the JS half of **bug 2** |
| 1917 | quiz-specific open validation (>= MIN_QUIZ_QUESTIONS) |
| 1938 | survey-specific open validation (has questions) |
| 1957 | task-specific open validation (venue matches) |
| 2003 | `if (item_type === 'quiz')` -- writes back the legacy `quizzes.is_open` flag |
| 2016 | `if (item_type === 'project')` -- writes back `projects.is_open` |

**Front end -- src/public/app.js**

| Line | What it enumerates |
| --- | --- |
| 135 | `ICON` map -- a page with no entry falls back to the admin icon |
| 465 | `pages_for()` student list -- **bug 3** |
| 441-453 | `pages_for()` staff list, plus the two conditional splices for assess and survey |
| 546-565 | the `{ home: page_home, ... }[page]` route table |

**Database CHECK constraints** (from pg_constraint on a loaded dump)

| Table | Constraint | Enumerates |
| --- | --- | --- |
| releases | `releases_item_type_check` | all 7 activity types |
| releases | `chk_releases_item_id` | which 4 carry an item_id -- **bug 2** |
| releases | `releases_dept_check` | ECE, EEE |
| students / teams / tasks | `*_dept_check` | ECE, EEE -- three copies of one fact |
| tasks | `tasks_submission_type_check` | image, drive, text, file, none |
| projects | `chk_projects_submission_type` | the same five, second copy |
| projects | `projects_status_check` | assigned, in_progress, submitted, scored |
| assessment_questions / assessment_attempts | `*_kind_check` | pre, post -- two copies |
| surveys | `chk_survey_round` | daily, final |
| survey_answers | `chk_answer_round` | daily, final -- second copy |
| attendance | `chk_attendance_source` | self, admin |
| student_profiles | 5 URL checks | the same Drive-or-local-path rule, five times |

**Chase scripts** (branch `chase-lists`, scripts/chase/)

`01-no-task-handin.sql` matches `item_type = 'task'`, `02` matches
`'project'`, `06` matches `'attendance'`, and `00-flags.sql` names four types.
Each hard-codes the one type it cares about, so a renamed type breaks them
silently -- they would return zero rows and read as "nobody is outstanding".

### The count

**Roughly 30 places** enumerate a closed set that a new activity type,
venue, submission type or round has to be added to. Of those, **12 would have
to change to add one new activity type**, spread across three languages and
two branches, with no test that fails if one is missed.

Venue is the same problem waiting: `DEPTS` in JS, three `*_dept_check`
constraints, and every `CROSS JOIN (VALUES ('ECE'),('EEE'))` in the chase
scripts. A third venue is a bigger edit than a third activity type.

### Why this is Track 3 Phase B's evidence

B3 makes `activities` one table with one shape for task, project, quiz,
assessment and survey. B4 makes `releases.is_open_for(activity, venue)` the
only gate, with "prove it with a grep" as the acceptance test. B2 makes
`venues` a table instead of a dept enum.

The list above is what those three items are worth: today N is about 30, and
12 of them move together for one new type. The v3 target is 1.

The grep that proves B4 is done is the one that finds this list empty:

    grep -rnE "item_type *(===|=) *'" src/
    grep -rnE "'(ECE|EEE)'" src/

Both should return nothing but the venues table and the activities table.

NEXT:     T2-07 -- the final round's admin flow.

---

## T2-02, T2-03  DONE  2026-09-19 16:05 IST

WHAT:     The survey's release gate, the admin loader, and the student's own
          form. Commits on branch `survey`: isOpenFor's survey branch, then
          set_release + loader + student routes + tests/survey.js. Pushed.

TESTS:    tests/survey.js -- 25 checks, all passing, every one through a real
          signed-in session for BOTH an admin and a student. The venue check
          is made from the student's session, not the admin's.

          Baseline for the four suites the handover claims: 71 checks,
          70 pass, 1 fail. The handover's number is exactly right.

          The one failure is flows.js "today's project is marked". The test is
          correct and the data is the finding: 106 day-2 projects exist and
          none is open, so no card is raised.

          releases (20 pass / 19 fail) and tasks (7/7) fail substantially.
          Checked properly rather than assumed: each was run on its own
          freshly-loaded database, WITH and WITHOUT the survey migration, and
          the results are identical both ways. These pre-date this work. The
          cause is fixtures -- releases needs quizzes with questions and the
          dump has zero across all nine. They are written for a seeded
          database, not a real dump.

OLD PATH: This is where the item earned its keep. isOpenFor() had its survey
          branch from T2-02 and was correct, but set_release()'s ALLOWED list
          did not carry 'survey', so every release row for a survey was
          refused with "Not something that can be opened". The gate worked
          and the admin could never reach it. Reading the code did not find
          this; signing in and trying to open one did. Exactly the v2 pattern
          -- the new code correct, the layer underneath it not.

          Also widened set_release's item_id rule: a survey release names one
          survey, the way a quiz release names one quiz. Without it a survey
          would have been allowed through with item_id null and then failed
          the chk_releases_item_id constraint added in T2-01.

          Checked that nothing else reads releases.item_type as a closed set:
          the only other reader is isOpenFor, which now has its branch.

ASSUMED:  The final round is served to a student as just another open survey,
          found by round rather than through a question, because it owns no
          question rows. If the final round ever needs to be openable
          alongside a daily round on the same screen, the ordering in
          /api/survey/today (round DESC, then day) decides which wins -- it
          currently prefers the final round. Local, reversible, one ORDER BY.

FOUND:    Three of my own test failures were the test being wrong, not the
          code, which is the rule the working-rules file names. In order: the
          admin students endpoint returns only active students and carries no
          is_active flag, so filtering on one found nobody; survey rows
          persisted between runs, so the second run reported a dozen failures
          that were one stale row; and the releases API field is `open`, not
          `is_open`, so my test wrote no release row at all and it read
          exactly like a broken gate. Each fixed in the test.

          tests/survey.js therefore deletes rows by design, to start from a
          known state. It clears ONLY the three survey tables and their
          release rows, and refuses to run unless PGDATABASE ends in _test or
          _local. It cannot reach real data.

          Method error worth recording: backgrounding the server with `( ... &)`
          does not propagate env vars, so .env won and the staff password did
          not match. Three suites read as broken and were fine. A false red,
          caused by me, found by checking rather than reporting.

          devDependencies cannot reach production: setup-server.sh and
          update.sh both run `npm ci --omit=dev`. Playwright was already in
          devDependencies at ^1.63.0 with Chromium installed; nothing was
          added. Both files read, neither edited. Carried into T2-18.

NEXT:     T2-04 -- the Open tab's survey row, per venue, shaped like a quiz.

---

## T2-01  DONE  2026-09-19 15:45 IST

WHAT:     The survey schema -- surveys(day, round), survey_questions,
          survey_answers(round) with UNIQUE (survey_question_id, student_id,
          round), plus a working down migration. Branch `survey` off `main`,
          commits d79c011 and eafe9cc, pushed.

TESTS:    Run against a scratch copy of the 19 Sep dump, not an empty
          database. Eight constraint tests, each expecting a refusal and
          getting one:

            final survey carrying a day            refused
            daily survey without a day             refused
            second daily survey for the same day   refused
            second final round                     refused
            questions attached to the final round  refused
            opening a survey with no questions     refused
            valid daily survey                     accepted
            one final round                        accepted

          Down migration actually run, both ways: it refuses while answers
          exist ("holds 1 rows... Re-run with -v force=1"), and with force it
          drops all three tables, all three functions, and restores both
          releases constraints. Teams and points identical before and after
          (53 teams, 0.0 points). Up is safe to run twice -- the second run is
          NOTICEs only.

OLD PATH: releases is the old path here, and it had TWO constraints naming
          item_type, not one: releases_item_type_check lists the allowed
          types, and chk_releases_item_id decides which of them carry an
          item_id. My first draft matched the catalogue by pattern, hit two
          rows, and failed outright. Widening only the first would have been
          worse than failing -- the type would have been accepted and then
          every survey release row refused, an error appearing the first time
          staff tried to open a survey in front of a room.

          isOpenFor() in src/server.js has a `task` branch returning false
          with the comment "tasks arrives in the next step". No survey branch
          exists yet, so a survey release row currently falls through to
          `return item_type === 'attendance'` and reads false. That default is
          correct -- closed -- and T2-02 adds the branch.

ASSUMED:  Answers are insert-only, per survey-spec.md section 8 ("first tap is
          locked"). The UNIQUE is therefore the whole enforcement and the
          route writes ON CONFLICT DO NOTHING. If answers ever need to be
          editable the constraint stays correct and only the route changes --
          no migration needed.

          The final round's "openable" test is that at least one question
          exists ANYWHERE, since its form is every daily question in the
          program. A daily round's test is its own questions.

FOUND:    A bug in my own down migration, found by checking rather than
          assuming: it re-added the narrowed constraint as
          chk_releases_item_type while the widened releases_item_type_check
          was still present, leaving two overlapping checks. The narrow one
          silently wins, so a re-run of the up migration appeared to succeed
          and would then have refused every survey release. Fixed to drop both
          names and restore the originals; up -> down -> up -> down now leaves
          exactly one constraint with its original name and definition.

          No migration in this repo had a `down` before this one. The
          convention is now a companion `-down.sql` file. Reconciling
          src/db/migrations/readme.md is T3-A4, not this item.

          Branch correction: T2-01 was first committed to `chase-lists`, which
          is Track 1's branch. The queue and brief both say Track 2 is
          `survey` off `main`. Corrected by creating `survey` off `main` and
          cherry-picking -- no stash, no reset, nothing discarded.
          `chase-lists` still holds identical copies of those two commits;
          they are harmless there and removing them would mean rewriting a
          pushed branch.

NEXT:     T2-02 -- release rows per survey per venue, through isOpenFor.

---

## T1-01 .. T1-14  DONE  2026-09-19 15:40 IST

WHAT:     scripts/chase/ -- nine read-only chase queries, a runner writing one
          CSV per query with a venue column, a verifier that proves every count
          a second way, and scripts/load-local-dump.sh. Committed as 4bd7c9c,
          fc8ac28, a37722e on branch chase-lists.

TESTS:    Run at real volume: 206 chaseable students (209 less three seeded
          test accounts), 53 teams, both venues.

          - All nine lists run for every day 1-9. 81 runs, slowest 7.6 ms,
            against a 2000 ms budget.
          - Every count cross-checked by a second query written a different
            way -- team-count x membership against per-student NOT EXISTS,
            EXCEPT against NOT EXISTS, FILTER aggregates against WHERE joins,
            arithmetic complements against direct counts. All nine agree on
            days 1, 2, 3 and 8.
          - Seven invariants hold: attendance partitions the room exactly
            (202 present + 4 absent + 0 unmarked = 206 on day 2); lists 5 and 6
            never overlap; lists 3 and 4 never overlap; no test or inactive
            student is chaseable; every chaseable student has a venue; profile
            percent stays within 0-100; a self-pasted Drive CV is never chased.
          - Query 1's 139 hand-verified independently of both queries:
            151 ECE students less 12 members of the 3 teams that handed in.
          - Queries 3 and 4 had no real data to test against -- the dump holds
            nine quizzes with zero questions. Proved in a rolled-back
            transaction with six questions, a release for ECE only, and 30
            attempts of which 10 submitted: 121 ECE / 0 EEE never opened,
            20 opened-not-submitted. Exactly as predicted. ROLLBACK verified
            to leave quiz_attempts at 0.
          - Read-only proved twice: no write keyword in any query file, and
            the database refuses an UPDATE inside the session the runner uses.
          - Row counts identical after 81 runs. No leftover objects.

OLD PATH: Checked, and it mattered more than anywhere else in this item.

          v_student_progress still exposes has_photo and has_education after
          src/routes/profile-completion.js dropped both from the weights. A
          list 7 built from the view -- the obvious way to build it -- would
          have named 200 of 206 students for a photo and an education line the
          profile page no longer asks for, every day, with no way to clear
          them. Query 7 reads the JS weights instead. Verified by running the
          app's own completion() against all 206 students for day 2 and day 8:
          zero disagreement with the SQL, including the Day 8 switch where
          resume_v2 starts counting and the total moves from 85 to 100.

          Also checked: the app writes attendance through two paths only, the
          lead's ON CONFLICT DO NOTHING and the admin's ON CONFLICT DO UPDATE.
          Neither ever deletes a row, which is what makes lists 5 and 6 two
          genuinely different lists rather than one asked twice.

          Also checked: submissions is still the project hand-in table. The
          project-formats migration widened its constraint rather than
          replacing it, and store_project_file() still writes it.

ASSUMED:  Three seeded test accounts (TEST0001-3, ECE-T99-TESTTEAM) are
          excluded from every list. They are is_active and would otherwise
          appear on all nine lists every day. If they are real students the
          exclusion is wrong -- 00-flags.sql reports the count excluded so it
          can never swallow a real student silently. Excluding them is what
          makes the ECE cohort read 151, matching the documented figure.

          The day passed to run.sh is the room's day. The computed default is
          printed for comparison rather than trusted, because CURRENT_DATE is
          UTC on the server and IST on this Mac -- a list pulled near midnight
          would otherwise name a different set of students.

FOUND:    Answering the ops-findings question directly: submissions = 0 and
          scores = 0 do NOT mean the queries read the wrong table. All 159
          project rows are status='assigned' with is_open=false, and every
          release row for both day-2 project groups is is_open=f. No project
          has ever been opened, so nobody could have handed one in and nobody
          could have scored one. List 2 correctly returns zero rows. This is
          a "nothing was opened" finding, not a schema one.

          task_submission_orphans holds 50 unclaimed Drive files from 27 teams
          against Day 1's task 8 -- real student work, detached by the
          overwrite bug, counted in 00-flags.sql and chased by nobody.

          quiz_questions is 0 across all nine quizzes, so no quiz can be
          opened by any route and lists 3 and 4 are empty for every real day.

          daily_posts is 7 for 206 students two days in. List 9 is therefore
          near-total, and posts_open is carried on every row so the caller can
          see whether the students could have posted at all.

          personal_email is empty for all 209 students, so that column is
          blank in every CSV. Worth knowing before anyone tries to mail a list.

          36 CVs are handed in and not yet copied to Drive, not the 76 the
          cv-drive-links migration mentions. scripts/migrate-cvs.js has run
          since. These are a count in 00-flags.sql and never names.

          208 of 209 students have a student_profiles row. The one without is
          TEST0003, a seeded test account, so no real student is missing a
          profile row and 00-flags.sql correctly reports 0. chased still LEFT
          JOINs the profile table, so a real student who never opened the
          profile page would appear in lists 7 and 8 rather than vanishing
          from them -- which an inner join would have done silently.

NEXT:     Track 1 complete. Not starting Track 2: see questions-for-vishnu.md.
