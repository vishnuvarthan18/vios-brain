# Known issues

Things that have gone wrong and are worth knowing about. Facts only. Fixes and
procedure live in the file that owns them, not here.

## 19 Sep 2026 — tests/tasks.js: four failures on `dev`, not caused by scoring v3

Found while checking whether the scoring v3 work had broken anything. It had
not: `tests/tasks.js` and `tests/releases.js` were run twice on the same
freshly built database and the same fixture, once with the scoring v3 commit
and once with it removed, and the two runs are **byte-identical**.

| Suite | Before scoring v3 | After |
| --- | --- | --- |
| `tests/releases.js` | 39 pass, 0 fail | 39 pass, 0 fail |
| `tests/tasks.js` | 45 pass, **4 fail** | 45 pass, **4 fail** |

Those four were already failing. They are two separate things.

> **Update, after the cutover (2026-09-19-g):** item 1 below is now obsolete
> rather than broken. `teams.total_points` is deliberately always 0 and the
> column carries a comment in the database saying so. The assertion is left
> failing on purpose, with a note in `tests/tasks.js` explaining why; it goes
> when someone removes that whole section deliberately. Item 2 is unchanged
> and still needs a decision.

### 1. `teams.total_points` does not follow a mark

> FAIL  and the team total follows without a page being opened

`team_task_points_for_day()` returns the right number — the assertion above it
passes. What does not happen is `teams.project_points` / `teams.total_points`
being updated by the trigger when a hand-in is marked. The stored total only
catches up when something else opens a page that recalculates it.

This is the old scoring path, and it is exactly what scoring v3 replaces: the
v3 views compute from the source rows every time and store no total at all, so
there is nothing to fall out of date. Worth knowing the old one is already
drifting while it is still the one every screen reads.

### 2. `POST /api/admin/projects/open` — the test and the code disagree

> FAIL  the old global project switch is retired
> FAIL  and points at where opening happens now
> FAIL  and it opened nothing

The test expects **410 Gone**. The route was not retired — it was rewritten to
be venue-aware and now takes `dept` / `depts` and opens through `releases`.
Called with no venue it still opens **both**, which is the thing the test was
written to prevent, but it does it through the release rows rather than the old
blanket `UPDATE projects SET is_open`.

**Someone has to decide which is right**, and this is not a decision to take
while tidying a test run:

- If the route should be gone, delete it and the test passes.
- If the venue-aware version is wanted, the test is stale and should assert the
  new behaviour — including whether "no venue given" should open both or be
  refused as ambiguous.

Left failing on purpose. A test quietly edited to match the code it is meant to
check is how this project got six false greens in a day.

### How to run these properly

They default to `http://localhost:3099` and the database `bootcamp_test`.
Port 3099 currently holds a **stale server on the old `bootcamp` database**, so
running them with no environment set points the test at one database while it
writes rows into another. That produces a long list of failures that mean
nothing. Set all of it explicitly:

```sh
PGDATABASE=bootcamp_dev \
BASE_URL=http://127.0.0.1:3099 \
STAFF_PASSWORD=test-staff-pw \
ADMIN_EMAIL=<an admin in that database> \
  node tests/tasks.js
```

with a server started against that same database, on that port, with that
password.


## 18 Sep 2026 — 19-second outage during the 03:23 UTC deploy

**54 requests failed** with `EACCES`, between 03:23:01 and 03:23:20 UTC.
Affected `/api/profile`, `/api/profile/completion`, `/api/my-team`,
`/api/my-projects`, `/api/quiz/open`, `/api/assessment/open`,
`/api/tasks/mine`, `/api/quiz/mine`, `/api/attendance/1` and `index.html`.

The service recovered on the restart at 03:26:00 UTC and has logged no errors
since. No data in the database was affected.

**Cause and fix are recorded in [`deploy.md`](deploy.md)** — step 4 under
*Afterwards*, and step 4 under *Running migrations*. Not repeated here.

### One photo upload was lost

Student id **81**. `uploads/photos/s81.jpg` was never written, and
`student_profiles.photo_url` for that student is empty. The upload failed at
03:23:18 UTC at `src/routes/profile-completion.js:212`.

Nothing else of that student's is affected — profile row, team, attendance and
points are all intact. It is the only lost work from the incident.

**No student was contacted, by decision.** Nothing is owed and nothing is
pending. If that student uploads a photo at any later point it will simply
work; the route is healthy and writes to `uploads/photos/` succeed.

If they never do, nothing surfaces it. The photo carries **no weight in the
profile completion bar** — it and the education line were taken off the profile
page because the resume already asks for both, so a missing photo does not show
up as an outstanding item or hold the percentage below 100. The `photo_url`
column stays empty and nothing prompts for it.

## 18 Sep 2026 — the Projects create form shipped without an end-to-end proof

The Projects admin screen regained a create form and was deployed the same
afternoon. **The planned end-to-end proof was not run.** It is the only thing
that shipped that day without one.

### What was proven

- The form renders and submits, driven by a browser against a test database:
  53 rows created (one per team), the day, title, description and
  submission type all stored as the form sent them, and the work **not** open
  on creation.
- The screen and the form at 390px and 768px with touch emulation: no sideways
  scroll, nothing overflowing an unscrollable parent, no tap target under 44px.
- The served `app.js` on production hashes byte-identical to the commit that
  was deployed.
- The five hand-in formats and the per-venue gate were proven live earlier the
  same day, against the staff test team, on the commit this one builds on.
  Those are unaffected by this change, which touches `src/public/app.js` only.

### What was not proven

**Nobody has created a project through the form on production and handed work
in against it.** Specifically unverified end to end:

- an admin creating a project through the live form
- opening that project for one venue through the live Open screen
- a student in the other venue being refused it
- a team lead handing a photo in against a project and it reaching Drive

### Why it was not run

Two reasons, both structural rather than anyone's decision to skip a step.

**The test team is ECE-only.** `ECE-T99-TESTTEAM` is the only team safe to
write to, and all three of its members are ECE. The plan was to open the
project for EEE and hand in as an EEE lead, which that team cannot do. The
alternative — using a real EEE student's account — would have put a fake
project row and a Drive file against a real team's record, so it was refused.
Inverting the venues (open for ECE, refuse an EEE student, hand in as the ECE
lead) proves the same logic and was the agreed plan.

**The admin half needs a staff session.** Creating a project and opening it per
venue are both admin-only, and the agent doing the work does not hold staff
credentials and does not ask for them — a private key was pasted into a chat on
17 Sep and had to be rotated, and the rule since is that staff passwords are not
requested. So those two steps had to be driven by a person.

The run was then stood down deliberately: the first real project serves as the
test instead. A bad project is visible immediately and deletable, and the parts
carrying real risk — the hand-in path and the venue gate — were already proven
live that day.

### What to watch

The first real project created through this form. If it appears on the Open
screen, opens for one venue, and a team hands work in against it, the gap
closes on its own. If it does not, the failure is in the create form or in the
per-venue badge, both of which are in `page_projects_admin` in
`src/public/app.js` and in no other screen.

One bug was found and fixed while building it, which is worth knowing if the
badge ever reads oddly: the status column read `r.is_open`, a field the route
stopped returning when a project became openable for one venue and not the
other. Every row said "closed" whatever was actually open. It now reads
`open_eee` and `open_ece` and names the venue, because "open" alone reads as
both.

## 18 Sep 2026 — production carried a copy of another lane's worktree

From the morning of 18 Sep until the evening deploy of that day,
`/opt/bootcamp-dashboard/.worktrees/side` held a complete copy of a second
agent's checkout: 102 files, including its own `src/routes/` and `src/public/`.

**Nothing served it.** The app runs `src/server.js` from the application root,
and no route, static mount or require path reaches under `.worktrees/`. No
student request was ever answered from those files and no data passed through
them.

It arrived the same way the morning's `EACCES` outage did: the deploy copied
the working **tree** with rsync rather than the **commit**. rsync sends whatever
is sitting on disk, so a second lane's checkout — untracked, uncommitted, in no
commit anywhere — was shipped along with the release.

It was removed by the first deploy to build its payload with `git archive` of an
explicit SHA. That step is what closed the root cause: `git archive` can only
produce what is in the commit, so nothing uncommitted can reach the server, and
`--delete` then removes what earlier deploys left behind. The procedure is in
[`deploy.md`](deploy.md), step 2.

Same root cause as the outage recorded above. Both are fixed by the same step.

## 20 Sep 2026 — three defects found by LOOKING, during the Track 5/6 build

All three were found by driving a real browser and reading the screen. None
threw an error. None would have been caught by a check that only reads a
response, and two of them had been true for days.

### 1. Every student's home said their team had 0 points  — FIXED

`GET /api/my-team` selected `teams.total_points`. The scoring cutover
(`2026-09-19-g`) pins that column at 0 deliberately and the database carries a
comment saying so. So a student's own screen said **0 points** while the board
two taps away showed the real total and the real rank.

Two ways of telling a team what it has scored, disagreeing — which is the
exact bug the whole scoring rewrite exists to kill, sitting on the screen
students look at most.

Fixed by reading `v_leaderboard` (which wraps the v3 calculation) in that
route. `tests/track5-browser.mjs` now asserts that the number on a student's
home equals the number on the board, and that it is not zero.

### 2. The completion matrix made the whole page scroll sideways on a phone — FIXED

193px of it, at 390px wide. Section 7 of the migration doc forbids a page that
scrolls sideways outright.

The cause is worth remembering: each coloured block carries a `sr-only` span
saying in words what it means. `sr-only` is `position: absolute`, and the
block had no `position` of its own, so those spans resolved against the PAGE
rather than against their cell — landing outside the table's `overflow-x`
box, which is the thing that is supposed to contain the sideways scroll.
Adding `relative` to the block fixed it.

Accessibility markup can break layout in a way that is invisible on a laptop
and invisible to every automated check that is not measuring `scrollWidth`.
The browser suite measures it now, on three wide screens.

### 3. A mentor's first screen was "Admin only / Try again" — FIXED

Every panel on the admin Home reads an admin-only endpoint, but Home was
offered to all staff, and it is the first item in the list — so a mentor
signing in landed on a screen they are not allowed to read, with a Try again
button that could never work.

True since before the v3 migration. Invisible because **every test signs in as
an admin.** There are no mentor accounts on the live site, so it has never
affected a real person; it would have, the first day one was made.

Home is now admin-only and a mentor starts on the Leaderboard.

### Also, while in there

Table headings had no horizontal padding while the cells below them did, so
two short headings read as one word — "DayWhat", "PointsNote" — and every
heading sat a few pixels off the column it named. Fixed in
`web/src/components/ui/admin.jsx`, which every admin table uses.

---

## 20 Sep 2026 — the other suites, and what their failures actually mean

Run against a fresh database built from `schema.sql` plus every migration,
seeded with the 209/53 fixture. **`tests/tasks.js` is 45/4, the same four as
19 Sep.** The rest:

| Suite | Result | What the failures are |
| --- | --- | --- |
| `harness/session-suite` | **405 / 0** | every read route, every role |
| `scoring` | **39 / 0** | |
| `scoring-routes` | **50 / 0** | every write, every role |
| `releases` · `survey` · `survey-sessions` · `task-upsert` · `tinkercad` · `completion` · `gates` | **all 0 fail** | |
| `tasks` | 45 / 4 | the four above, unchanged |
| `quiz-per-student` | 38 / 4 | team quiz averages — the old scoring path, which v3 replaced |
| `project-formats` | 50 / 2 | `isOpenFor` and the project group id — the open decision in SESSION-STATE §6 item 19 |
| `assessment` | 9 / 3 | the fixture has no open EEE assessment |
| `attendance` | 1 / 2 | fixture-specific: it looks for a day this team has not marked, and the fixture has marked them all |

**These were proved unrelated to the Track 5/6 work**, not assumed to be: the
whole set was run twice on the same database, once with the new route module
mounted and once with it removed, and the two runs are identical.

They also need their environment stated or they produce a long list of
failures that mean nothing — in particular `ADMIN_EMAIL`, which defaults to a
person who does not exist in the fixture and makes every admin sign-in fail:

```sh
PGDATABASE=bootcamp_test BASE_URL=http://127.0.0.1:3099 \
STAFF_PASSWORD=test-staff-pw ADMIN_EMAIL=<an admin in that database> \
  node tests/tasks.js
```

---

## 20 Sep 2026 — THE DEPLOY BROKE FOUR SCREENS: the migrations ran as the wrong user

The most expensive mistake of this project so far, and the one most worth
reading twice.

### What happened

`scripts/go-live.sh` applies migrations with `sudo -u postgres psql`. So every
table, view, sequence and function those migrations created was **owned by
`postgres`**. The app connects as the non-superuser role **`bootcamp`**, which
was never granted anything on them.

The migrations applied perfectly. Every check the script made passed. The site
broke anyway:

```
Sep 20 00:53:01  error: permission denied for table surveys
Sep 20 00:54:21  error: permission denied for view v_leaderboard_v3
Sep 20 00:58:20  error: permission denied for view v_leaderboard
```

Four screens died, and they were exactly the four that read new objects:
**Completion**, **Points**, **the live board** and **Surveys**.

**The rollback did not save it.** `2026-09-19-g` drops and recreates
`v_leaderboard`, so after the cutover that view was postgres-owned too — which
means the OLD front end's Leaderboard was broken as well, and
`GET /api/my-team` with it. Rolling the screens back moved the failure, it did
not remove it. That is the 00:58 line above, logged after the rollback.

### Why nothing caught it

Every rehearsal ran in a container **as a superuser**, where object ownership
cannot matter. Three rehearsals passed: a bare schema, a re-run, and a full
209-student database. None of them could have failed this way.

> **A check that cannot fail the way production fails is not a check.**

The same shape as every other entry in this file: the check confirmed itself.
It proved the migrations *apply*. It never proved the app could still *read*
what they created.

### The fix, applied 01:05

```sql
GRANT USAGE ON SCHEMA public TO "bootcamp";
GRANT ALL ON ALL TABLES    IN SCHEMA public TO "bootcamp";
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO "bootcamp";
GRANT ALL ON ALL FUNCTIONS IN SCHEMA public TO "bootcamp";
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES    TO "bootcamp";
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO "bootcamp";
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON FUNCTIONS TO "bootcamp";
```

Verified by reading as the app's own role: `v_leaderboard_v3` 53,
`v_leaderboard` 53, `surveys` 0, `score_adjustments` 0, `scoring_settings` 1.
No data was changed — only permissions.

The `ALTER DEFAULT PRIVILEGES` half carries no `FOR ROLE`, so it applies to
whichever role runs it. That is deliberate: naming `postgres` explicitly would
be wrong the day someone runs the migrations as anyone else.

### The permanent fix — `go-live.sh` section 2b

The script now, after applying migrations and before declaring anything:

1. reads the app's role out of `$APP/.env` (`PGUSER`), and **stops** if it
   cannot tell what that role is
2. applies the grants above
3. **reads fourteen objects as that role** — the four v3 views, scoring
   settings, adjustments, the three survey tables, releases, task submissions,
   attendance, students, teams — and dies naming the object if any of them
   comes back anything other than a number

Both halves were tested: on a database whose app role is not the migrator, all
fourteen come back readable; and with the grant revoked, the check sees
`ERROR: permission denied for view v_leaderboard_v3` and stops, as it should.

### What to remember

- **Ownership is invisible until it is fatal.** It does not show up in a
  migration's output, in a row count, or in any test run as a superuser.
- **Verify as the role that will actually be used.** Applying SQL successfully
  and being able to read the result are two different claims.
- **A rollback across a migration is not a rollback** when the migration
  rebuilt something the old code also reads.
