# Work queue

The agent works this file from the top. See `docs/unattended-operation.md`.

**Status values:** `TODO` · `DOING` · `DONE` · `BLOCKED` · `FAILED`
The agent updates this file and commits it after every item.

**Only add items whose decisions are already made.**

---

## Track 1 — chase lists · branch `chase-lists` · **DONE**

All 14 items complete. 17 checks pass, pushed. See `docs/agent-log.md`.

Usage:

```sh
./scripts/load-local-dump.sh ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
./scripts/chase/run.sh 2
./scripts/chase/verify.sh 2
```

---

## Track 2 — daily survey · branch `survey` off `main`

**Spec: `docs/survey-spec.md`. It supersedes the survey section of the brief.**

| # | Item | Status |
| --- | --- | --- |
| T2-01 | Migration: `surveys` (id, day, round, title), `survey_questions`, `survey_answers` with `UNIQUE (survey_question_id, student_id, round)`. Working `down` | DONE |
| T2-02 | Release rows — one per survey per venue, through `isOpenFor`. A survey with zero questions cannot be opened | DONE |
| T2-03 | Admin loader — paste-many, one question per line, parsed list shown before saving | DONE |
| T2-04 | Open tab — survey row with a per-venue control, identical in shape to a quiz | DONE |
| T2-05 | Student card — first item in My work, tagged **Required**, does **not** lock other items | DONE |
| T2-06 | Answering — Yes/No, **saves the instant it is tapped**, saved state per question, survives a refresh | DONE |
| T2-07 | **Final round** — admin opens it once; its form is every `survey_questions` row for the program, ordered by day then position. Creates no new question rows | DONE |
| T2-08 | Daily results — Yes/No per question, by venue, every count linking to its named list with phones and CSV | DONE |
| T2-09 | Proof report 1 — per question: "Day 3: 12 of 206 knew → End: 190 know, +86%", ordered by gain | DONE |
| T2-10 | Proof report 2 — one headline: "Across N topics, X% before → Y% after" | DONE |
| T2-11 | Proof report 3 — per student, on their own profile | DONE |
| T2-12 | Proof report 4 — every view above split by venue, EEE and ECE side by side | DONE |
| T2-13 | "Has not answered today's survey" list, added to the chase set | DONE |
| T2-14 | **Venue leak test** — opening for EEE must not expose it to ECE | DONE |
| T2-15 | Session tests — every route, every role, signed in, at 209 students | DONE |
| T2-16 | Old-path check — My work still behaves correctly with the new card present | DONE |
| T2-17 | **Screenshots** — Playwright, 390px, every new screen, saved to `docs/screens/`. Student card, answering, admin loader, Open tab, daily results, each proof report. So the look can be judged without reading code | DONE |
| T2-18 | Write the deploy command set into the log. **Do not run it** | DONE |

`personal_email` is empty for all 209 students. Say so on any screen offering to
contact them. Phone only.

---

## Track 3 Phase A — foundation · branch `v3-dev` off `main`

Start when Track 2 is `DONE` or `BLOCKED`. No user-visible change in any of
these. **Every decision is pre-answered in `docs/phase-a-decisions.md` — read it
before starting, and do not stop to ask.**

Order is deliberate: the small, safe items are banked first, the module split
runs last.

| # | Item | Status |
| --- | --- | --- |
| T3-A1 | Fake-data generator — 209 students, 53 teams, two venues, nine days. Seeded and deterministic. Invented names and numbers only | **DONE** — `scripts/seed/generate.js`. This row said TODO for days after the file existed, and a duplicate seeder was nearly committed on the strength of it. Extended 19 Sep with surveys and the points timers |
| T3-A2 | Session test harness — every test signs in for real, four roles, every route, plus one role that must be refused. At 209 students | **DONE** — `tests/harness/`. 370 pass, 0 fail as of 19 Sep |
| T3-A4 | Migration ledger — reconcile readme (9) against the directory (16), record the run order and how each was verified. Documentation only | TODO |
| T3-A5 | `v_student_progress` — find every reader, fix the completeness readers to use the JS weights, mark the two columns deprecated. **Do not drop them** | TODO |

### T3-A3 — module split, one module per item, last

Only after A2 is done. Behaviour does not change; the same assertions are green
before and after each one. Commit and push each module alone.

| # | Module | Status |
| --- | --- | --- |
| T3-A3-01 | `auth` | TODO |
| T3-A3-02 | `students` | TODO |
| T3-A3-03 | `teams` | TODO |
| T3-A3-04 | `venues` | TODO |
| T3-A3-05 | `releases` | TODO |
| T3-A3-06 | `activities` | TODO |
| T3-A3-07 | `submissions` | TODO |
| T3-A3-08 | `scoring` | TODO |
| T3-A3-09 | `attendance` | TODO |
| T3-A3-10 | `reports` | TODO |
| T3-A3-11 | `storage` | TODO |
| T3-A3-12 | `programs`, and `server.js` reduced to wiring only | TODO |

If a module will not move cleanly: stop that module, log why, take the next one.
Never force it, never refactor to make it fit.

---

## Track 3 Phase B — data model · branch `v3-dev`

Start only when Phase A is `DONE` or `BLOCKED`. **Order matters: B0 first, then
B2 before B3.**

**B2's real scope is 13 server-side venue literals**, not 37. The other 24 live
in `src/public/app.js`, which Phase C1/C2 deletes and replaces with React —
fixing them in B2 is work that is thrown away. The survival test applies to
every budget: *does this file survive v3?* If not, it is not Phase B's problem.

By that test the activity-type count does **not** shrink: all 17 gates are in
`server.js`, none in `app.js`, so every one has to be folded into
`is_open_for()` by hand. Page routes, by contrast, are 10 of 11 in `app.js` and
die with it.

**The acceptance test for B0, B2 and B3 is the conformance test, T3-B0.** None
of them is done until it passes for every type and every venue.

| # | Item | Status |
| --- | --- | --- |
| T3-B0 | **Conformance test.** A registry of activity types; for each one, drive the FULL lifecycle through a real signed-in session: create it · open it for venue A only · confirm venue B cannot see it · hand it in · score it · confirm it shows on the student's screen and in the reports. The test iterates the registry, so adding a type adds a row — and if any of the ~17 gates was missed, that row fails. Written FIRST, against today's code, and expected to fail for types that are genuinely broken | TODO |
| T3-B1 | `programs` table, one row, backfilled everywhere | TODO |
| T3-B2 | `venues` table replaces the dept enum, backfilled. Scope: the **13 server-side** literals, the `*_dept_check` constraints on students/teams/tasks/releases, and the `CROSS JOIN (VALUES ('ECE'),('EEE'))` sites in `scripts/chase/` on branch `chase-lists` — those are on another branch and are the thing most likely forgotten at merge. **Leave `app.js` alone**: C1/C2 deletes it. **Not done until a third venue is a single INSERT and the conformance test passes for all three** | TODO |
| T3-B3 | `activities` table — one shape for task · project · quiz · assessment · survey. Submissions and scores point at `activity_id`. Not done until the conformance test passes for every type | TODO |
| T3-B4 | `releases.is_open_for(activity, venue)` is the **only** gate. Nothing checks day or department directly. Proved by `tests/gates.js` reaching budget 1, and by the conformance test | TODO |
| T3-B5 | One name per field. `is_lead` everywhere | TODO |
| T3-B6 | Roster import screen: CSV upload, **additive only**. Delete `load-eee.sql` and `load-ece.sql` in the same commit | TODO |

**Gate for the whole phase:** team totals, leaderboard order, attendance counts
and submission counts **identical** before and after, on a real dump. One point
of drift means stop.

`tests/gates.js` pins four budgets, each naming the phase that clears it:

| Budget | Today | Cleared by |
| --- | --- | --- |
| activity type gates, server-side | 17 | **B4**, by hand |
| venue literals, server-side | 13 | **B2**, by hand |
| venue literals, `app.js` | 24 | C1/C2, by deletion |
| page routes, `app.js` | 10 | C2, by deletion |

Only the first two are Phase B's work. The budget is lowered in the same commit
that removes the sites, so a win cannot quietly come back.

---

## Track 4 — scoring v3 · branch `v3-dev`

**Spec: `docs/scoring-v3-plan.md`.** Decided with Vishnu 19 Sep. Manual marking
is removed entirely; points are calculated; an admin can only add or subtract
through a logged adjustment; the leaderboard is live.

Items T4-01..T4-05 are done and tested. **Nothing is switched over** — every
screen still reads the old totals, and that is on purpose until the comparison
has been looked at.

| # | Item | Status |
| --- | --- | --- |
| T4-01 | Migration `2026-09-19-f-scoring-v3` + `down`. Settings row, `score_adjustments`, `scores_legacy`, `points_minutes` on releases/tasks/projects/surveys. Add-only, safe twice, rollback tested | DONE |
| T4-02 | ONE scoring function. `team_points_breakdown()` for a single team, `v_team_day_points_v3` set-based for the board. A test asserts the two agree for every team | DONE |
| T4-03 | `src/routes/scoring-v3.js` — leaderboard (with a version stamp for polling), a team's breakdown, adjustments add/undo/list, settings, release timer, old-vs-new comparison | DONE |
| T4-04 | `tests/scoring.js` — 39 assertions on the rules: the timer, late scoring zero but still accepted, one hand-in per team, the venue gate, the attendance window against a UTC clock | DONE |
| T4-05 | `tests/scoring-routes.js` — 50 assertions on the writes, every role, signed in. Includes the absence checks: nothing in the module closes a release, nothing schedules itself | DONE |
| T4-06 | **Admin adjustments screen.** Pick a team, see the earned total, add + or −, optional note, full history, undo each row, all-teams view with CSV | DONE — `web/src/pages/Adjust.jsx` |
| T4-07 | **Live leaderboard screen.** Polls on the version stamp, movement arrows, venue filter, pauses when the tab is hidden | DONE — `web/src/pages/BoardLive.jsx` |
| T4-08 | **Big-screen mode.** Projector view, top 10, no navigation, Esc to leave | DONE — in `BoardLive.jsx`. Auto-cycling between venues is NOT built; the venue is chosen before going full screen |
| T4-09 | **Timer control**, on the RELEASE BOARD rather than the create forms — the timer lives on the release because it starts at opening and the two venues open at different times. Presets 5/10/15/30/60/custom. Per venue, per item | DONE — `web/src/pages/Open.jsx` |
| T4-10 | **Attendance window enforced where a lead marks.** 09:00–10:00 IST. Shown before it matters, refused by the server either way. Absent and not-yet-marked are now different colours | DONE |
| T4-13 | **A default timer on the create forms**, copied onto the release when it is opened — and only ever into a blank, never over one someone set | DONE |
| T4-14 | **Student card shows the timer**, and says late work is still accepted | DONE |
| T4-11 | **Run the comparison.** Done on the fixture: 53 teams, 990 points before, 4197 after, **not one team went down**. The rise is attendance and the survey, which were always recorded and never scored. **Still to be run on a real dump before this is deployed** | DONE on fixture · TODO on real data |
| T4-12 | **CUTOVER.** `2026-09-19-g-scoring-cutover.sql`: archives `scores`, drops the four recalc triggers, rebuilds `v_leaderboard` over the automatic calculation with the SAME column names, zeroes the dead columns. `/api/mentor/score` answers **410** and names where to go. Marking screen and the second leaderboard deleted | DONE in the repo — **NOT DEPLOYED** |

T4-11 and T4-12 are blocked on a decision, not on work. Everything before them
can be built without one.


---

## Logged, not to be fixed by the agent

- **159 projects, all `is_open=false`** — never opened for either venue. The
  admin team is handling this. Do not touch it.
- **50 orphan Drive files from 27 teams** against Day 1's task 8 — real work
  linked to nothing. For Track 3.
- **36 CVs await their Drive copy.** A count, never names.

---

## Blocked / questions

Anything moved to `BLOCKED` is written up in `docs/questions-for-vishnu.md` with
the options and a recommendation.
