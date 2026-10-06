# Bootcamp dashboard — build brief for the dev agent

**Read this whole file before writing any code.** It is the authority. Where it
disagrees with any other document, this file wins. Where it is silent, ask.

Companion documents, all in `docs/`:

- `v3-restructure-plan.md` — what is being built and why
- `v3-decisions.md` — every decision taken, numbered
- `agent-working-rules.md` — the rules learned from the v2 build
- `v2-build-prompts.md` — the current live state and every known issue

---

# Part 1 — Hard rules

These are not guidance. Breaking any one of them is a stop-work event.

### Deployment

1. **You never deploy. Ever.** Not to production, not in a window, not with
   approval. You build and test locally; Vishnu deploys.
2. **You never `ssh` to the production server.** Not to look, not to read logs,
   not to take a dump.
3. **You never connect to the production database.** Every query you write runs
   against your local copy.
4. When something is ready, you write the exact commands into a report. Vishnu
   runs them, at night, when nobody is working.

### Git

5. **Work only on your assigned branch.** Never commit to `main`.
6. **One worktree per lane.** Two agents in one working tree caused a production
   incident during v2.
7. **No `git stash`, ever.** Worktrees share one stash stack; it has already
   swallowed another lane's work once.
8. **Commit and push after every step.**
9. **Never `reset --hard`** without first rescuing uncommitted work and saying
   what you rescued.
10. **Never `rsync` from a working tree.** Deploy payloads are built with
    `git archive` of an explicit commit. This appears in the commands you write,
    never in something you run.

### Data

11. **Never run `load-eee.sql` or `load-ece.sql`.** Each deletes a department's
    students before inserting, cascading to every post, attendance mark and quiz
    answer.
12. **Every migration is additive and has a working `down`.** No dropped
    columns, no renamed tables, until the old ones have been dead for a month.
13. **If you change `start_date` locally to test, change it back** in the same
    step, and say so in your report.
14. Migrations run as `postgres` leave objects the app user cannot read.
    **Object ownership is reassigned in the same step as the restart**, in every
    deploy command set you write. This took the live app down twice.

### Secrets

15. **Never ask for a password, key or token.** If you need one, propose a way
    to proceed without it.
16. **Never print a secret** into a report, a commit, a log or a chat.
17. A password passed inline on a shell command lands in shell history. Read it
    from a file.

### Scope

18. **One feature at a time.** Report at the end of each feature, then stop and
    wait for the next instruction.
19. **Never widen scope mid-feature.** If you find something else that needs
    doing, write it in the report as a finding. Do not fix it.
20. **Refuse instructions that would cause damage**, and say why. Both v2 agents
    refused something at least once and were right every time.

### Before you report a feature done

21. **Go and look at what the old path still does.** Eight serious bugs in the
    v2 build. The new code was correct every time; the code underneath it was
    not. Old routes still open, old booleans still read, old screens never
    rebuilt, a view still answering a question the new weights no longer ask.

---

# Part 2 — What this is

A dashboard for a 9-day electronics bootcamp. Live at `vcet.aracreate.academy`
with **209 students, 53 teams, two venues (EEE, ECE)**, running 18–26 September
2026.

Stack: Node 20 + Express, PostgreSQL 17, Caddy, no build step. Front end is
`src/public/app.js` + `app.css`.

Roles: student · team lead · mentor · admin.

Core rules that must survive every change:

- Points belong to the **team**, never the individual. Max 5 a day.
- **Nothing opens by itself.** Every activity is opened by hand, per venue.
- A quiz below 5 questions cannot be opened by any route.
- Quiz: per student, 30 seconds per question, answer saved the instant it is
  picked. Team mark = average of those who attempted.
- Assessments carry zero points and no timer; compared as percent gain.
- A student's daily post is readable by that student and an admin. Nobody else.

## Schema facts established 19 Sep — do not re-derive

- **A day is a derived integer, never a stored date.**
  `(CURRENT_DATE - settings.start_date) + 1`, clamped. `settings` is one row,
  `start_date` 2026-09-18, `total_days` 9. Every work table carries `day INT`.
  Scripts take the day number **explicitly** and only compute the default as a
  convenience, printing it — `CURRENT_DATE` is the database server's timezone.
- **Attendance:** `present BOOLEAN NOT NULL DEFAULT FALSE`,
  `UNIQUE (student_id, day)`. No code path ever deletes a row. A row with
  `present=false` is a stored absent; a missing row means nobody took the
  register. These are two different lists.
- **Phone** resolves as
  `COALESCE(NULLIF(trim(profile.phone),''), NULLIF(trim(students.phone),''))` —
  profile first, roster second. Use that everywhere.
  `students.email` is the college login; `student_profiles.personal_email` is
  the personal one.
- **Profile completeness lives in JavaScript, not SQL.**
  `src/routes/profile-completion.js` is authoritative: phone 15, personal_email
  15, skills 20, goal 15, resume_v1 20, resume_v2 15 from Day 8 only.
  `v_student_progress` still exposes `has_photo` and `has_education` after the
  weights dropped them — **the view is stale, do not read it for completeness.**
  `goal` is three questions (`goal`, `goal_3y`, `goal_5y`) counting as one item.
- **Venue:** `students.dept` is NOT NULL; `teams.dept` is nullable and nothing
  enforces that they agree. Use `students.dept` and report mismatches as a count.
- **CVs:** `resume_v1_url` is where the CV is now (server path or a pasted Drive
  link). `resume_v1_drive_url` is where `scripts/migrate-cvs.js` copied it.
  "Safely in two places" is both being non-null.
- **Tasks and projects are team-owned**, not per student. `task_submissions` is
  `UNIQUE (task_id, team_id)`. A day can carry several tasks; `dept NULL` means
  both venues. `daily_posts` is the only genuinely per-student hand-in.
- **Migrations:** `src/db/migrations/readme.md` lists 9; the directory holds 16.
  The six from 18–19 Sep have no documented run order. Confirm against the dump
  before trusting either.

---

# Part 3 — The three tracks

They are separate. Do not mix them in one branch.

| Track | Branch | Ships to | Deploys |
| --- | --- | --- | --- |
| **1 — Chase lists** | `chase-lists` | Nothing. SQL files Vishnu runs | No deploy at all |
| **2 — Daily survey** | `survey` off `main` | The live app | One night deploy, by Vishnu, when fully tested |
| **3 — v3 rebuild** | `v3-dev` | Nothing until cutover | One cutover, between batches |

Track 2 ships before Track 3 exists. When it lands on `main`, `v3-dev` rebases
onto it.

---

# Part 4 — Environment

**Repo:** `~/araCreate/bootcamp-dashboard` on the Mac.

**Local database.** Vishnu places a `pg_dump` on the Mac. You load it into local
PostgreSQL. You never fetch it yourself.

Write `scripts/load-local-dump.sh`: takes a dump path, drops and recreates the
**local** database only, loads it, and prints row counts (students, teams,
submissions) plus the list of applied migrations, so a bad or stale dump is
obvious immediately.

**Every test runs at real volume** — 209 students, 53 teams. Everything in v2 was
tested with three students until someone ran 209.

---

# Part 5 — Track 1: chase lists

**Goal:** Vishnu needs to know *who* has not done today's work, not how many, so
he can chase a room. No deploy.

`scripts/chase/` — one SQL file per query, plus `scripts/chase/run.sh` taking a
day number. **One CSV per query, with a venue column** — not one file per venue.

Every row: name · roll · team · venue · phone · personal email.

1. Every member of a team with no hand-in for today's task(s), naming which task
2. Every member of a team with no hand-in for today's project
3. Students who never opened today's quiz
4. Students who opened today's quiz but submitted nothing
5. Students **marked** absent today — `EXISTS(row) AND present = false`
6. Students with **no attendance row** today, plus a column saying whether the
   register was open for that venue
7. Students with an incomplete profile, listing the **named** missing fields,
   following the JS weights and excluding `resume_v2` below Day 8
8. Students with **no CV at all** (`resume_v1_url IS NULL`). Students who handed
   one in but whose Drive copy has not been made are an operational count in the
   report, **not names to chase**
9. Students who have not written today's daily post — the only per-student
   hand-in

Students with `team_id IS NULL` are reported as a separate flagged count, never
padded into lists 1 and 2.

**Acceptance:** read-only — no INSERT, UPDATE, DELETE or DDL anywhere. Runs
against a local database loaded from a real dump. Every count cross-checked with
a second query written a different way. Each query under two seconds.

---

# Part 6 — Track 2: the daily survey

Built on the **current** stack — vanilla JS, no React. The UI is throwaway once
v3 lands; that is accepted.

### What it is

- **Yes / No** questions asked **before each day starts**, on every day.
- **Different questions each day**, written the evening before.
- **Every student answers individually. Zero points. No timer. No right answer.**
- Opened **by hand, per venue**, on the Open tab — like every other activity.
- For the student it is **just another item in "My work"** — first in the list,
  tagged **Required**. It does **not** lock the other items.

### Schema — additive only

```
surveys            (id, day, title, created_at)
survey_questions   (id, survey_id, position, text)
survey_answers     (id, survey_question_id, student_id, answer bool, answered_at)
```

Plus release rows in the existing `releases` table, one per survey per venue.
`UNIQUE (survey_question_id, student_id)` so a double tap cannot double-insert.

### Screens

**Admin — load:** paste-many, one question per line. Show the parsed list before
saving.

**Admin — open:** the survey appears on the Open tab with a per-venue control,
identical to a quiz.

**Student — answer:** first card in My work, tagged Required. **Each answer saves
the moment it is tapped.** Show saved state per question. No submit-everything
button that can be lost.

**Admin — results:** per question, Yes and No counts split by venue. Every count
links to the named list behind it, with phone numbers and CSV export. A question
appearing on more than one day shows a day-by-day line of Yes percentage. Each
student's answers appear on their own profile view.

### Gate before it is called done

- Opening the survey for EEE does **not** make it visible to ECE. Test this
  explicitly — the equivalent bug shipped in v2.
- A student with no survey open sees no card and no error.
- Every route tested **through a real signed-in session**, for every role.
- Tested at 209 students.
- Answers survive a page refresh mid-survey.
- The old My work path still behaves correctly with the new card present.

Then write the deploy command set into the report. Do not run it.

---

# Part 7 — Track 3: the v3 rebuild

Ordered. Do not start a phase until the one before it is reported and approved.

## Phase A — foundation, no visible change

**A1.** `v3-dev` off `main`. `scripts/load-local-dump.sh`. A fake-data generator
producing 209 students, 53 teams, two venues, nine days of plausible
submissions — so development never needs real data.

**A2.** Test harness that **signs in**. Every test signs in as a real role
through the real login and carries the session. A test that hits an endpoint
without a session does not count. Four roles.

**A3.** Module split. `src/server.js` becomes wiring only. Routes, service and
queries per module: `auth · programs · venues · students · teams · activities ·
releases · submissions · scoring · attendance · reports · storage`.
**Behaviour does not change.** Same assertions green before and after.

## Phase B — data model

**B1.** `programs` table, one row, backfilled everywhere.
**B2.** `venues` table replaces the dept enum, backfilled.
**B3.** `activities` table — one shape for task · project · quiz · assessment ·
survey. Submissions and scores point at `activity_id`.
**B4.** `releases.is_open_for(activity, venue)` is the **only** gate. Nothing
checks day or department directly. Prove it with a grep.
**B5.** One name per field. `is_lead` everywhere.
**B6.** Roster import screen: CSV upload, **additive only**. Delete
`load-eee.sql` and `load-ece.sql` in the same commit.

**Gate:** team totals, leaderboard order, attendance counts and submission counts
**identical** before and after, on a real dump. One point of drift means stop.

## Phase C — front end foundation

**C1.** Vite + React + Tailwind + shadcn/ui in `web/`, built to static files
served by Caddy. Backend untouched. araCreate colours as Tailwind tokens —
Golden Sun the single accent, one thing per screen. **Light mode only.**
Phone-first: 390px is the design width, no horizontal scroll anywhere.

**C2.** App shell — sign-in, routing, the five admin nav groups, the student's
five items, mobile nav.

**C3. STOP AND REPORT.** Build exactly three screens for approval: Student Home,
Admin Today, the completion matrix. Do not build screen four until the look is
approved. Nothing gets built twice.

## Phase D — screens

Student: Home · My work · My profile · My team · Leaderboard.
Team lead: attendance inside My team. Mentor: Home · Marking · My teams ·
Leaderboard. Admin: Today · Content · People · Live · Reports.

**One release board** under Live: rows are activities, columns are venues, a cell
is Closed / Open / opened-at, clicking toggles it.

## Phase E — visibility

**E1.** Every count endpoint gets a matching list endpoint.
**E2.** Activity detail page — Done / Not done / Scored / Not scored.
**E3.** Completion matrix — colour blocks, click a cell to open it.
**E4.** Full student profile.
**E5.** Full team profile.
**E6.** Every student and team name becomes a link.
**E7.** Every list exports CSV and copies all phone numbers.

## Phase F — reusable

Creating batch 2 through the UI with **no code change** is the acceptance test
for the whole of v3.

## Deferred — do not build

Certificates. Journey page. Transcript.

---

# Part 8 — Definition of done

- [ ] Works for **every role that can reach it**, tested through a real session
- [ ] Tested at **209 students and 53 teams**
- [ ] Opening something for one venue does **not** expose it to the other
- [ ] Loading, empty, error, offline and no-permission states all exist
- [ ] Works at **390px** with no horizontal page scroll
- [ ] **The old path was checked** and is reported on
- [ ] **Every guard test was watched failing.** A test whose job is to stop a
      mistake — a lint, an invariant, a permission check — is not done until
      the thing it guards has been broken, the test seen red, and the code
      restored. The sabotage must be at the level the test claims to protect.
      See `docs/standing-authorisation.md`: a near-miss on 19 Sep passed a
      three-state guard against code with the field deleted, because the
      sabotage was one level below what the test inspects
- [ ] Every new count has a list behind it
- [ ] Migration has a `down`, and it was run
- [ ] Suite green, and you can say why anything that changed, changed
- [ ] Committed and pushed

**A passing suite can be a false green.** If a test starts passing on its own,
"why did that change" is worth more than the green. If a test fails, first ask
whether the test is wrong — twice in v2 a failing assertion was the test's fault.

---

# Part 9 — The report

At the end of each feature, and not before. Then stop.

```
FEATURE:        what was built
BRANCH/COMMITS: branch, commit hashes, pushed yes/no
DONE:           each definition-of-done item, ticked or not.
                An unticked box is fine; a hidden one is not
OLD PATH:       what the old code still does, and whether it is a problem.
                Never "n/a"
FOUND:          anything noticed and deliberately not fixed
RISK:           what could go wrong when this reaches the live app
DEPLOY:         the exact commands, in order, for Vishnu to run.
                Payload built with `git archive` of a named commit.
                Migration + ownership reassignment + restart in ONE step.
                Excludes: uploads, .archives, logs, node_modules.
                A fresh dump first, verified by row count.
                How to roll back, and roughly how long it takes.
NEEDED:         what you need from Vishnu to continue
```

---

# Part 10 — Working with Vishnu

- He is running a live event. **Lead with the decision he needs to make**, not
  the reasoning.
- If he must do something himself, give the **exact commands or clicks**.
- Tell him plainly when something is his job and no agent can do it — writing
  survey and quiz questions, chasing a room that has not signed in.
- If he overrules a recommendation, state the risk once, then help him do it well.
- Keep it short and in plain English. Points, not paragraphs.
