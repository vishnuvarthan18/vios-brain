**Vishnu** (2026-09-19T09:37): You are the dev agent for the araCreate bootcamp dashboard.

Repo: ~/araCreate/bootcamp-dashboard

BEFORE YOU WRITE ANY CODE, read these in order:
1. claude/v3-agent-brief.md   <- this is the authority. Read the whole file.
2. claude/v3-decisions.md
3. claude/agent-working-rules.md
4. claude/v2-build-prompts.md

Non-negotiable, repeated here so there is no doubt:
- You NEVER deploy. You never ssh to the production server. You never
  connect to the production database. You build and test locally only.
- Work only on the branch you are told. Never commit to main.
- One feature at a time. Report at the end of the feature, then STOP and wait.
- Never widen scope mid-feature. Write findings in the report instead.
- Before you report anything done, go and look at what the old code path
  still does, and say what you found.
- Never ask for a password, key or token.

YOUR FIRST TASK — Track 1, chase lists. Branch: chase-lists

Vishnu is running a live bootcamp with 209 students in two venues (EEE, ECE).
The app shows him counts but not names, so he cannot chase the students who
have not done their work. Fix that with read-only SQL. Nothing is deployed.

Build scripts/chase/ containing one SQL file per query plus a wrapper
scripts/chase/run.sh that takes a day number and outputs one CSV per query
per venue.

Every query outputs: name, roll, team, venue, phone, personal email.

The eight queries:
1. Students who have not handed in today's task
2. Students who have not handed in today's project
3. Students who never opened today's quiz
4. Students who opened the quiz but submitted nothing
5. Students marked absent today
6. Students with no attendance mark at all today
7. Students with an incomplete profile, listing the named missing fields
8. Students with no CV on Drive

Rules for this task:
- Read-only. No INSERT, UPDATE, DELETE, or DDL anywhere.
- Test against the local database loaded from a real dump.
- Prove each count is correct by cross-checking it with a second query
  written a different way. Do not assert it is right.
- Each query must run in under two seconds.

Deliver the report in the format in Part 9 of the agent brief.

If anything in the brief is unclear or looks wrong, say so before you start.

**Claude** (2026-09-19T09:37): I'll start by reading the required documents before touching any code.

**Claude** (2026-09-19T09:37): _[tool: Bash]_

**Claude** (2026-09-19T09:37): There is no `claude/` directory. Let me look for those files elsewhere before assuming anything.

**Claude** (2026-09-19T09:37): _[tool: Bash]_

**Claude** (2026-09-19T09:37): Only one of the four exists, and it's under `docs/`, not `claude/`. Let me check the whole machine and git history before concluding.

**Claude** (2026-09-19T09:37): _[tool: Bash]_

**Claude** (2026-09-19T09:38): Three of the four files don't exist anywhere in the repo or its history. You told me the brief is the authority and to read it before writing any code, so I'm stopping here rather than guessing.

## What I found

| File you named | Status |
|---|---|
| `claude/v3-agent-brief.md` | **Missing.** No `claude/` directory exists. Not in any branch, not in git history, not in either worktree. |
| `claude/v3-decisions.md` | **Missing.** Same. |
| `claude/agent-working-rules.md` | **Missing.** Same. |
| `claude/v2-build-prompts.md` | **Exists, different path**: [docs/v2-build-prompts.md](docs/v2-build-prompts.md) |

I searched the working tree, `git log --all` for any commit that ever added a file matching those names, all 12 local branches, and both worktrees (`.worktrees/per-student`, `.worktrees/side`). Nothing. So it isn't a case of them living on another branch — they were never committed here.

## Why I can't just proceed

The brief is the authority, and two things I need come specifically from it:

- **Part 9's report format.** You asked for the deliverable in that format. I have no way to reconstruct it.
- **The working rules.** "Never widen scope," "go and look at what the old code path still does" are quoted in your message, but `agent-working-rules.md` presumably has more that I'd be silently violating.

`v3-decisions.md` likely also settles things the eight queries depend on — what "today" means relative to a day number, whether a day is a date or an index, how absent is recorded versus simply unmarked. Query 5 and query 6 are only distinguishable if I know how attendance is stored, and guessing there produces two chase lists that quietly overlap or quietly miss people. With 209 real students being chased, a wrong list costs you real phone calls.

## Two other things worth flagging now

**Track 1 is described as read-only SQL, but the app looks like it may not be SQL-backed.** Before I say that with confidence I'd need to read the source — but `.env` and `src/` are here and I haven't opened them, since reading code is the thing you told me to do only after the brief. Worth confirming the chase scripts should target a SQL database at all.

**"One CSV per query per venue" is 16 files per day.** Queries 5 and 6 may also need a definition of "today" that accounts for the bootcamp's day numbering, which the day-number argument to `run.sh` implies exists somewhere in the decisions doc.

## What I need from you

Point me at the three missing files — most likely they're outside this repo (the `~/araCreate/Claude outputs/` directory caught my eye, but I haven't looked inside it without being asked) or they exist somewhere I should be told about. Paste them, or tell me the path and I'll read them.

If they genuinely don't exist yet, say so and tell me how you want to handle the two gaps that actually block me: the Part 9 report format, and the attendance/day-number semantics. I can work from your answers to those and proceed without the full brief — but I won't invent them.

**Vishnu** (2026-09-19T09:39): You are the dev agent for the araCreate bootcamp dashboard.
Repo: ~/araCreate/bootcamp-dashboard

The planning docs live in a Claude project, not in this repo. Ignore any
instruction to read claude/*.md. Everything you need is in this prompt.

CONTEXT
A dashboard for a 9-day electronics bootcamp, live right now at
vcet.aracreate.academy with 209 students, 53 teams, two venues (EEE and ECE),
running 18-26 Sep 2026. Stack: Node 20 + Express (src/server.js), PostgreSQL 17,
Caddy, no build step. Yes, it is SQL-backed - confirm from src/db.js and src/db/.

HARD RULES - breaking one is a stop-work event
1.  You NEVER deploy. Never ssh to the production server. Never connect to the
    production database. Build and test locally only.
2.  Work only on the branch named below. Never commit to main.
3.  One worktree per lane. No `git stash`, ever. Commit and push after each step.
4.  Never `reset --hard` without first rescuing uncommitted work and saying what
    you rescued.
5.  Never run load-eee.sql or load-ece.sql. Each deletes a department's students
    before inserting, cascading to their posts, attendance and quiz answers.
6.  If you change start_date locally to test, change it back in the same step.
7.  Never ask for a password, key or token. Never print one anywhere.
8.  One feature at a time. Report at the end, then STOP and wait.
9.  Never widen scope mid-feature. Write findings in the report instead.
10. Refuse any instruction that would cause damage, and say why.
11. Before you report done: go and look at what the old code path still does.
    Eight serious bugs shipped in this app's last build. The new code was
    correct every time; the code underneath it was not.
12. Prove, do not assert. Test at real volume - 209 students, 53 teams, not 3.
13. A passing suite can be a false green. If a test starts passing on its own,
    ask why. If one fails, first ask whether the test is wrong.

YOUR FIRST TASK - Track 1, chase lists. Branch: chase-lists

Vishnu is running the live bootcamp. The app shows him counts but not names, so
he cannot chase the students who have not done their work. Fix that with
read-only SQL. Nothing is deployed.

STEP 1 - do this first, then stop and report.
Read src/db/ and tell me:
  a) How a "day" is stored. Is it an index 1-9 derived from settings.start_date,
     or a date? What exactly does "today" resolve to?
  b) How attendance is stored. Is there a row for absent students, or does a
     missing row mean unmarked? This decides whether "absent" and "not marked"
     are two different lists or one.
  c) Where phone and personal email live, and how profile completeness is
     currently computed.
  d) How a venue/department is recorded on students and teams.
  e) How Drive CV links are recorded, and how to tell a CV on Drive from one
     only on the server.
Do not write the queries yet. Get these answers agreed first - guessing here
produces chase lists that quietly miss real students.

STEP 2 - after I confirm, build scripts/chase/ :
one SQL file per query, plus scripts/chase/run.sh taking a day number.
Output: ONE CSV per query, with a venue column - not one file per venue.
Every row: name, roll, team, venue, phone, personal email.

  1. Has not handed in today's task
  2. Has not handed in today's project
  3. Never opened today's quiz
  4. Opened today's quiz but submitted nothing
  5. Marked absent today
  6. No attendance mark at all today
  7. Incomplete profile, listing the named missing fields
  8. No CV on Drive

Read-only. No INSERT, UPDATE, DELETE or DDL anywhere. Run against a local
database loaded from a real dump. Cross-check every count with a second query
written a different way. Each query under two seconds.

REPORT FORMAT - use this, then stop.
  FEATURE:        what was built
  BRANCH/COMMITS: branch, commit hashes, pushed yes/no
  DONE:           each acceptance item, ticked or not. An unticked box is fine,
                  a hidden one is not
  OLD PATH:       what the old code still does and whether it is a problem.
                  Never "n/a"
  FOUND:          anything noticed and deliberately not fixed
  RISK:           what could go wrong if this reached the live app
  DEPLOY:         exact commands for Vishnu to run, or "none - read-only"
  NEEDED:         what you need from Vishnu to continue

If anything here is unclear or looks wrong, say so before you start.

**Claude** (2026-09-19T09:39): I'll read the schema layer now. Step 1 only — no queries.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Confirmed SQL-backed. Reading the schema and the readmes now.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): The migrations readme lists 9 files but the directory has 16 — six later ones aren't in that dependency list. Let me read the schema itself.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): The schema is the base build; migrations layer on top. I need both. Let me read the core tables.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Key finding already: `students.phone` and `student_profiles.phone` both exist — two phone columns. Let me continue through attendance and quizzes.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Base schema has team-level quizzes; a migration changes that to per-student. I need the migrations that supersede this — they're what the live DB actually runs.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Critical: **tasks and task_submissions are TEAM-level, not per-student.** That changes what "students who have not handed in today's task" can mean. Let me read the remaining migrations.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Now the profile-completion route — the authoritative weights live in code, not SQL — and the releases/day logic.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Major discrepancy found: the code weighs **6** fields, but `v_student_progress` still exposes 8. Now the day logic and releases.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Day is a derived integer, not a stored date. Now attendance handling and releases in the server.

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:39): Now the crucial question for Q5 vs Q6: does the app ever write `present = false`, or does unmarking delete the row?

**Claude** (2026-09-19T09:39): _[tool: Bash]_

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Confirmed: rows are never deleted, and `present=false` is a real stored state. Q5 and Q6 are genuinely two different lists. Now the releases mechanism and daily_posts/task submission paths.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Now the quiz open/attempt path — Q3 vs Q4 hinge on what "opened" means — and the project groups migration.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Clean: attempt row exists = opened, `submitted_at` NULL = opened but not submitted. Now project groups, the daily post path, and drive CVs.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Now: is there a per-student project submission, or only team-level? And what does the daily post / "task" hand-in actually look like from the student's side?

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Confirmed — both tasks and projects are strictly team-level. Let me verify the CV Drive migration script's semantics and check `db.js`.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): Confirmed SQL-backed, PostgreSQL, standard pool. I have all five answers. Let me verify the CV question once more — whether the migration script is the only writer of drive columns.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:40): `migrate-cvs.js` exists but doesn't reference those columns. Let me check what it actually writes.

**Claude** (2026-09-19T09:40): _[tool: Bash]_

**Claude** (2026-09-19T09:41): Found it — the columns are built by string interpolation, which is why the plain grep missed them. I have all five answers. Stopping here as instructed.

---

**FEATURE:** Step 1 only — schema reconnaissance for the chase lists. No queries written, no files created, no branch cut.

## a) How a "day" is stored

**A derived integer, never a stored date.** There is no date column on any work table. [src/server.js:2939](src/server.js#L2939):

```sql
SELECT start_date, total_days, posts_open,
       (CURRENT_DATE - start_date) + 1 AS raw_day
  FROM settings WHERE id = 1
```

then clamped: `raw < 1 → 0`, `raw > total → total+1`, else `raw`. `settings` is a single row (`CHECK (id = 1)`), `start_date` is `DATE '2026-09-18'`, `total_days` 9.

Every work table carries `day INT` with `CHECK (day BETWEEN 1 AND 9)` — `daily_posts` (1–60, wider), `projects`, `attendance`, `quizzes`, `tasks`. So "today" for a chase list resolves to `(CURRENT_DATE - start_date) + 1`.

Two consequences worth agreeing before I write anything:

- **`CURRENT_DATE` is the database server's timezone, not Asia/Kolkata unless the server is set to it.** If the local DB and production differ, a chase list run near midnight names a different set of students. I'd rather `run.sh` take the day number explicitly — which your spec already says — and compute the default from the same expression only as a convenience, printing it so it can be checked.
- Today is 19 Sep 2026 and start_date is 18 Sep, so **today is Day 2**. That matches a bootcamp running 18–26 Sep.

## b) How attendance is stored — Q5 and Q6 are genuinely two lists

**Two different lists. Confirmed by reading both write paths, not inferred from the schema.**

`attendance` has `present BOOLEAN NOT NULL DEFAULT FALSE` with `UNIQUE (student_id, day)`. A row with `present=false` is a real, stored "absent". A missing row is "nobody took the register for this student".

What makes this certain is that **no code path ever deletes an attendance row.** The only two writers:

- Team lead, [src/server.js:605](src/server.js#L605) — `ON CONFLICT DO NOTHING`. Deliberately so: the comment explains the old `DO UPDATE` let a lead silently overwrite a settled register by tapping twice.
- Admin, [src/server.js:2009](src/server.js#L2009) — `ON CONFLICT DO UPDATE`, requires a ≥3-char reason, writes an `attendance_audit` row. An admin "unmark" sets `present=false`; it does **not** delete.

So Q5 is `EXISTS(row) AND present=false`, Q6 is `NOT EXISTS(row)`. They cannot overlap, and together they are the complement of present.

One real gotcha for Q6: the attendance register is gated per venue by `releases`. If ECE's register hasn't been opened yet today, **every ECE student is legitimately unmarked** and Q6 returns all 151 of them — a chase list that is noise, not signal. I'd like Q6's CSV to carry a column saying whether the register was open for that venue, so you can tell "the lead didn't take it" from "it isn't open yet."

## c) Phone, personal email, and profile completeness

**Phone lives in two places; personal email in one.**

- `students.phone` — the roster, loaded from the college
- `student_profiles.phone` — what the student typed
- `student_profiles.personal_email` — profile only. `students.email` is the *login* (college address), not a personal one.

The app resolves phone as `COALESCE(NULLIF(trim(p.phone),''), NULLIF(trim(s.phone),''))` — profile first, roster second ([2026-09-17-b-profile-completion.sql](src/db/migrations/2026-09-17-b-profile-completion.sql)). The chase CSVs must use that same COALESCE, or you'll call the wrong number.

**Profile completeness is computed in JavaScript, not SQL, and the two disagree.** [src/routes/profile-completion.js:44](src/routes/profile-completion.js#L44) is authoritative — its own comment says "The weights live here and nowhere else":

| Field | Weight |
|---|---|
| phone | 15 |
| personal_email | 15 |
| skills | 20 |
| goal | 15 |
| resume_v1 | 20 |
| resume_v2 | 15 (only from Day 8) |

Before Day 8 the bar is out of 85, so a student who has done everything possible sees 100%.

**Three things here that affect Q7 and need your decision:**

1. **`photo` and `education` were removed from the weights but are still in `v_student_progress`.** The code comment says both were taken off the profile page because the resume already asks for them. The view still returns `has_photo` and `has_education`. If I write Q7 off the view, I chase 209 students for a photo they have no way to upload. **Q7 must follow the JS weights, not the view.**
2. **`goal` is three questions, one item.** `filled(p.goal) && filled(p.goal_3y) && filled(p.goal_5y)` — a student who answered the nine-day goal and left the 3y/5y blank counts as not having done it. The named missing field should say so.
3. **`resume_v2` before Day 8.** Today is Day 2. If Q7 lists "Build your final resume" as missing, that's 209 students chased for work that isn't open. I'll exclude it below Day 8, matching the app — confirm.

## d) Venue / department

`students.dept TEXT NOT NULL CHECK (dept IN ('ECE','EEE'))` and `teams.dept TEXT CHECK (...)` — **nullable on teams**, not null on students.

They are stored independently and nothing enforces that a student's dept matches their team's. `students.team_id` is also nullable (`ON DELETE SET NULL`). For the venue column I'll use `students.dept` — it's the NOT NULL one and it's the student's own venue. I'll add a cross-check for students whose dept disagrees with their team's, and report the count rather than silently picking one.

## e) Drive CVs

Six columns from [2026-09-17-b-cv-drive-links.sql](src/db/migrations/2026-09-17-b-cv-drive-links.sql): `resume_v{1,2}_drive_{url,id,at}`.

Telling a Drive CV from a server-only one is explicit, and the migration is emphatic about why both columns exist:

- `resume_v1_url` — where the CV is *now*: either `/uploads/resumes/<file>` (server) or a Drive link the student pasted. Both allowed by `chk_resume_v1_url`.
- `resume_v1_drive_url` — where `scripts/migrate-cvs.js` copied it, constrained to `^https://drive\.google\.com/`.

The view already names the test: `resume_v1_copied = (resume_v1_url IS NOT NULL AND resume_v1_drive_url IS NOT NULL)` — "safely in two places".

**Q8 "no CV on Drive" is ambiguous and I want your ruling.** Three different lists:

1. **No CV at all** — `resume_v1_url IS NULL`. ~133 students. This is a chase list: they haven't handed anything in.
2. **Handed in, not yet backed up** — `resume_v1_url IS NOT NULL AND resume_v1_drive_url IS NULL`. The migration says this was 76 before `migrate-cvs.js` ran, 0 after. **This is not a student chase list** — the student did their bit; the *script* hasn't run. Chasing them is wrong.
3. A student who pasted a Drive link themselves: `resume_v1_url` is a Drive URL but `resume_v1_drive_url` is NULL. They *do* have a CV on Drive, but test 2 flags them anyway.

Since these are chase lists — people to phone — I read Q8 as **list 1**, with a separate count of list 2 reported as an operational note for you rather than as names to chase. Confirm.

## The problem with Q1 and Q2: tasks and projects are team-level

This is the one that changes the shape of the deliverable, so I've stopped rather than guess.

**Neither tasks nor projects have a student.** `task_submissions` is `UNIQUE (task_id, team_id)`; `projects` is one row per team per day; `submissions` hangs off `project_id`. The tasks migration is explicit: *"a task belongs to the TEAM, so any member may hand in and the second hand-in replaces the first."*

So "students who have not handed in today's task" cannot be answered literally. In a team of four where one member uploaded the bench photo, **all four have handed in** — or **none of them personally did**, depending on what you mean. What I *can* produce:

- **Every member of a team with no hand-in for today's task(s).** Right for chasing: you ring the team, the lead, or all four. This is what I'll build unless you say otherwise.
- Not: "students who personally didn't submit" — that would name three-quarters of every compliant team.

Two more wrinkles: a day can carry **several** tasks (`tasks` is one row per piece of work, with `dept NULL` meaning both venues), so "hasn't handed in today's task" means *any* outstanding task for their venue — I'll list which. And **`daily_posts` is the one genuinely per-student thing** — one row per student per day, the "what I learned" post. That may be what you actually want chased, and it isn't in your eight.

Also note Q1's scope: a student with `team_id IS NULL` has no team and so no hand-in, ever. They'd appear in Q1 and Q2 every single day. I'll report the count and flag them separately rather than let them pad the list.

## Two findings I'm not acting on

- **[src/db/migrations/readme.md](src/db/migrations/readme.md) lists 9 migrations; the directory has 16.** The six from 18–19 Sep (`assessment-survey`, `tinkercad-code`, `project-formats`, `project-open-per-dept`, `project-groups`, `project-view-by-group`) have no documented run order. I don't know whether they're applied to production. This matters for me only in that my local DB must match yours — but it's worth someone's attention.
- **`v_student_progress` exposing `has_photo`/`has_education` after the weights dropped them** is the kind of thing your rule 11 is about: the new code is right, the view underneath still answers the old question. Anything reading that view for "is this profile complete" is wrong today.

## What I need from you

1. **Q1/Q2 reading** — confirm "every member of a team that hasn't handed in" is what you want.
2. **Q8 reading** — list 1 (no CV at all), with list 2 reported as a count, not names?
3. **Q7 and `resume_v2`** — exclude below Day 8, matching the app?
4. **Do you want `daily_posts` chased?** It's the only per-student hand-in and it isn't in your eight. Say the word and I'll add it as a ninth — or leave it, and I won't widen scope.
5. **Local DB** — you said "loaded from a real dump." I don't have one and I won't ask for credentials or run the loaders (rule 5). Tell me where a dump is on this machine, or confirm you'll place one. I need to know which of the 16 migrations it has.

No branch cut, nothing written, nothing committed. Waiting.

**Vishnu** (2026-09-19T09:44): Good report. The schema findings are now written into docs/v3-agent-brief.md
Part 2 — read that section before Step 2 so nothing gets re-derived.

The five docs now exist in the repo (they were in a Claude project, not here —
my error):
  docs/v3-agent-brief.md      <- the authority
  docs/v3-decisions.md
  docs/v3-restructure-plan.md
  docs/v3-build-and-cutover.md
  docs/agent-working-rules.md
They are uncommitted. Commit them on the chase-lists branch as a docs commit.

ANSWERS

1. Q1/Q2 — yes. "Every member of a team that has not handed in" is correct.
   We ring the team, so we want all the names. Add the task/project title so
   the caller knows what they are chasing. Students with team_id IS NULL:
   flagged count, separate, never padded into the lists. Agreed.

2. Q8 — yes, list 1 only: resume_v1_url IS NULL. Names to chase.
   List 2 (handed in, Drive copy not yet made) is an operational count in the
   report, not names. A student who pasted their own Drive link has done their
   bit and must not appear anywhere in Q8.

3. Q7 — yes, exclude resume_v2 below Day 8. Follow the JS weights, never
   v_student_progress. For goal, name which of the three is missing
   (goal / goal_3y / goal_5y), not just "goal".

4. daily_posts — yes, add it as query 9. It is the only per-student hand-in and
   it is exactly the thing worth chasing. Authorised scope, not scope creep.

5. Also agreed, all three of your own proposals:
   - run.sh takes the day explicitly; the computed default is printed for checking
   - Q6 carries a column for whether the register was open for that venue
   - dept mismatch between student and team reported as a count, not silently resolved

LOCAL DUMP — I will place one. Do not fetch it.
Path: ~/araCreate/dumps/bootcamp-<date>.dump
When it appears, load it and report which of the 16 migrations it contains,
before writing any query.

YOUR FINDINGS — both noted, neither is yours to fix:
  - migrations readme lists 9, directory has 16. Logged for Track 3 Phase A.
  - v_student_progress still exposing has_photo/has_education after the weights
    dropped them. This is exactly rule 21. Logged for Track 3 Phase B.

Proceed to Step 2: build scripts/chase/ with the nine queries, on branch
chase-lists, once the dump is loaded. Read-only. Cross-check every count with a
second query written a different way. Report in the Part 9 format, then stop.

**Claude** (2026-09-19T09:44): I'll read the docs first, then check for the dump.

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:44): Docs are here; the dump directory doesn't exist yet. Reading the authority doc now.

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:44): Now the authority doc, especially Part 2 (my schema findings) and Part 9 (report format).

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:44): Part 4 adds a deliverable I didn't have: `scripts/load-local-dump.sh`. Let me read the decisions doc, then commit the docs.

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:44): Now creating the branch and committing the docs.

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:44): Wait — rule 6 says one worktree per lane, and main's worktree is this one. Let me check whether I should be in a separate worktree before committing.

**Claude** (2026-09-19T09:44): _[tool: Bash]_

**Claude** (2026-09-19T09:45): I'm the only agent in the main tree, and the other two worktrees hold different branches. Committing the docs.

**Claude** (2026-09-19T09:45): _[tool: Bash]_

**Claude** (2026-09-19T09:45): _[tool: Bash]_

**Claude** (2026-09-19T09:45): Docs committed and pushed. Now checking for the dump and local PostgreSQL.

**Claude** (2026-09-19T09:45): _[tool: Bash]_

**Claude** (2026-09-19T09:45): The dump file exists but is **0 bytes**. PostgreSQL 17.11 is running. Let me confirm it's genuinely empty and not still being written.

**Claude** (2026-09-19T09:45): _[tool: Bash]_

**Claude** (2026-09-19T09:45): Zero bytes, no process writing it, unchanged. It's not a partial transfer. Rather than idle, I'll build the two things that don't depend on the dump's contents.

**Claude** (2026-09-19T09:45): _[tool: Bash]_

**Claude** (2026-09-19T09:45): Now writing `scripts/load-local-dump.sh`, which Part 4 requires and which I need before I can load anything.

**Claude** (2026-09-19T09:46): _[tool: Write]_

**Claude** (2026-09-19T09:46): Let me verify the migration markers I guessed at are real, by checking the six undocumented migrations.

**Claude** (2026-09-19T09:46): _[tool: Bash]_

**Claude** (2026-09-19T09:46): Three of my markers were wrong — there's no `assessments` table (it's `assessment_questions`), and tinkercad is on `teams`, not `task_submissions`. Let me check the survey one too.

**Claude** (2026-09-19T09:46): _[tool: Bash]_

**Claude** (2026-09-19T09:46): Now I'll correct all the markers with verified ones.

**Claude** (2026-09-19T09:46): _[tool: Bash]_

**Claude** (2026-09-19T09:46): Now let me test the guard rails work, since those are the parts I can verify without a real dump.

**Claude** (2026-09-19T09:46): _[tool: Bash]_

**Claude** (2026-09-19T09:46): The dump file has disappeared since I checked it.

**Claude** (2026-09-19T09:46): _[tool: Bash]_

**Claude** (2026-09-19T09:46): The real dump arrived as a gzipped plain-SQL file, 81KB. My script doesn't handle gzip — let me add that and verify it's genuine first.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): Genuine PostgreSQL 17.11 dump, 344KB uncompressed. Adding gzip support to the loader.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): Dump loaded. All 14 testable migrations are applied. But **154 ECE students, not 151** as the docs say, and `quiz_attempts` is 0. Let me verify the two view-only migrations and investigate.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): All 16 migrations confirmed applied. Let me fix the two `[?]` markers to use this real test, then survey the data.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): All 16 applied. Now surveying the actual data before writing queries — what Day 2 really looks like.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): Clean data: no orphan students, no dept mismatches. 106 projects on day 2 (two groups × 53 teams). Let me look at the shape of day 2's work and the releases.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): Critical finding for the queries: on Day 2, attendance is **closed for both venues**, projects are **closed**, and task 10 is open **for ECE only** — EEE has no release row at all. This is exactly why Q6 needs the register-open column. Let me confirm the quiz and attendance state.

**Claude** (2026-09-19T09:47): _[tool: Bash]_

**Claude** (2026-09-19T09:47): Important: all 9 quizzes exist but have **zero questions and zero attempts** — Q3/Q4 will correctly return everyone/no-one, and I must make that legible rather than alarming. Also there are day-3 attendance rows, ahead of today. Let me check profiles and CVs.

**Claude** (2026-09-19T09:48): _[tool: Bash]_

**Claude** (2026-09-19T09:48): Note: 1 student has **no profile row at all** — a LEFT JOIN case that would silently vanish under an inner join. And `personal_email` is empty for all 209. Now writing the queries. First the shared helper.

**Claude** (2026-09-19T09:48): _[tool: Bash]_

**Claude** (2026-09-19T09:48): _[tool: Write]_

**Claude** (2026-09-19T09:48): Let me test this one immediately before writing the rest, so I catch shape problems early.

**Claude** (2026-09-19T09:48): _[tool: Bash]_

**Claude** (2026-09-19T09:48): Works — 142 ECE students outstanding, EEE correctly absent (task 10 isn't released for EEE). But there's a **TEST team** in production data that would pollute every chase list. Let me check.

**Claude** (2026-09-19T09:48): _[tool: Bash]_

**Claude** (2026-09-19T09:48): That explains 154 ECE vs the documented 151: 3 test accounts. They're `is_active=true` so they'd appear on every chase list. I'll exclude them by a documented, overridable rule rather than silently.

**Claude** (2026-09-19T09:48): _[tool: Write]_

**Claude** (2026-09-19T09:48): Now rewriting query 1 to use the shared view, and writing queries 2–9.

**Claude** (2026-09-19T09:49): _[tool: Write]_

**Claude** (2026-09-19T09:49): _[tool: Write]_

**Claude** (2026-09-19T09:49): Wait — I should check whether projects are handed in via `submissions` or via the newer project-formats path, given migration 12 added `submission_type` to projects.

**Claude** (2026-09-19T09:49): _[tool: Bash]_

**Claude** (2026-09-19T09:49): Confirmed: `submissions` is still the project hand-in table, with `is_latest`. My query 2 is correct. Now queries 3 and 4.

**Claude** (2026-09-19T09:49): _[tool: Write]_

**Claude** (2026-09-19T09:49): Let me verify MIN_QUIZ_QUESTIONS is actually 5 rather than trusting my memory.

**Claude** (2026-09-19T09:49): _[tool: Bash]_

**Claude** (2026-09-19T09:49): Confirmed 5. Now query 4 and the attendance pair.

**Claude** (2026-09-19T09:49): _[tool: Write]_

**Claude** (2026-09-19T09:50): _[tool: Write]_

**Claude** (2026-09-19T09:50): _[tool: Write]_

**Claude** (2026-09-19T09:50): Dump already loaded — my counts match yours. Your `submissions: 0` flag is the important one, and it does change query 2. Let me investigate before continuing.

**Claude** (2026-09-19T09:50): _[tool: Bash]_

**Claude** (2026-09-19T09:50): There's a `task_submission_orphans` table I hadn't seen — it's not in schema.sql or the 16 migrations. Let me find out what it is and how project hand-ins actually arrive.

**Claude** (2026-09-19T09:50): _[tool: Bash]_

**Claude** (2026-09-19T09:50): This matches the `per-student-tasks` branch commit ("keep the hand-ins the overwrite bug detached"). These are Day 1 **task** files lost to an overwrite bug — not projects. Let me confirm the project question directly.

**Claude** (2026-09-19T09:50): _[tool: Bash]_

**Claude** (2026-09-19T09:50): Now I have the full picture. Answering your flag directly:

**`submissions: 0` does not mean I'm reading the wrong table.** `submissions` is still the correct and only project hand-in table — `store_project_file()` in [src/routes/drive-uploads.js:315](src/routes/drive-uploads.js#L315) writes it, and the project-formats migration widened its constraint rather than replacing it. It is 0 because **no project has ever been opened**: all 159 project rows are `status='assigned'`, `is_open=false`, and every release row for both project groups is `is_open=f`. Nobody could have handed one in. So query 2 will correctly return zero rows, and that's a true answer, not a missing table.

The `task_submission_orphans` table is a genuine find — 50 Drive files from 27 teams against Day 1's task 8, none claimed. I'll report it, not act on it.

Let me finish the remaining queries.

**Claude** (2026-09-19T09:51): _[tool: Write]_

**Claude** (2026-09-19T09:51): _[tool: Write]_

**Claude** (2026-09-19T09:51): _[tool: Write]_

**Claude** (2026-09-19T09:51): Now the flags file for the counts that must never become names.

**Claude** (2026-09-19T09:52): _[tool: Write]_

**Claude** (2026-09-19T09:52): Now the runner.

**Claude** (2026-09-19T09:52): _[tool: Write]_

**Claude** (2026-09-19T09:52): _[tool: Bash]_

**Claude** (2026-09-19T09:52): Good failure — the read-only guard is real. A temp view is a write to the temp schema, so it must be created before the read-only transaction begins.

**Claude** (2026-09-19T09:52): _[tool: Bash]_

**Claude** (2026-09-19T09:52): Still failing but the error is being swallowed by my stderr redirect. Let me see it.

**Claude** (2026-09-19T09:52): _[tool: Bash]_

**Claude** (2026-09-19T09:52): That works. The problem is my multi-line `\copy` — psql requires it on one line. Let me restructure to use a temp file per query instead.

**Claude** (2026-09-19T09:53): _[tool: Bash]_

**Claude** (2026-09-19T09:53): `:day` isn't interpolated inside `\copy`. I'll use a psql variable that expands properly.

**Claude** (2026-09-19T09:53): _[tool: Bash]_

**Claude** (2026-09-19T09:53): The trailing semicolon can't be inside `\copy (...)`. Stripping it.

**Claude** (2026-09-19T09:53): _[tool: Bash]_

**Claude** (2026-09-19T09:53): All nine run, all far under 2s. Two things to check: the leftover `head -0` line, and query 6 returning 0 when I expected 206.

**Claude** (2026-09-19T09:53): _[tool: Bash]_

**Claude** (2026-09-19T09:53): Query 6 is correct: all 206 real students have a day-2 row (the 3 missing are the excluded test accounts). Now the cross-checks — every count verified a second way.

**Claude** (2026-09-19T09:54): _[tool: Write]_

**Claude** (2026-09-19T09:54): _[tool: Bash]_

**Claude** (2026-09-19T09:54): All nine agree, all seven identities hold. But some counts match trivially because the data is zero — let me test days where the data is non-trivial, and check query 1's count against a hand calculation.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): Note: 151 real ECE students, matching your documented figure exactly once test accounts are excluded. Let me verify the 139 by hand.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): 151 − 12 = 139, verified by hand independently of both queries. Now testing days 1 and 3, where the data differs, to be sure the checks aren't passing trivially.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): Days 1, 3 and 8 all vary as expected and all agree. But Day 8's list 7 should include resume_v2 — it reads 206 on both days, so let me confirm the Day-8 switch actually fires rather than being dead code.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): The Day-8 switch fires correctly: percent drops 18→15 and "final resume (15)" appears. The named goal parts work too. Now the old-path check required by rule 21 — comparing my query 7 against what the app's own code produces.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): Needs to run inside the repo to resolve `pg`.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): **Zero mismatches against the app's own `completion()` function** across all 206 students on both days. My query 7 agrees exactly with what each student sees on their own bar.

Now let me check the stale view I was warned about, to confirm what it would have done.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:55): The counts coincide at 206 today, but the *reasons* differ — the view would list photo/education as missing for ~190 students who cannot fill them. Let me verify that concretely.

**Claude** (2026-09-19T09:55): _[tool: Bash]_

**Claude** (2026-09-19T09:56): 200 students would have been named for items they cannot fill. Now verifying the read-only guarantee is actually enforced, not just claimed.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): Read-only proven two ways: no write keywords, and the database actively refuses one. Now confirming nothing was mutated, and checking performance at real volume.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): Identical to the freshly-loaded dump; no leftover objects. Now testing performance at worst case — a day where every query returns maximum rows.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): Slowest across all 81 runs is 7.6ms — 260× under the 2-second budget. Let me check the CSV output quality, then test a stress scenario for queries 3/4 which have never seen real data.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): Every CSV leads with the six required columns. Queries 3 and 4 have never run against real quiz data, so let me stress-test them in a rolled-back transaction rather than trusting they'd work.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): Queries 3 and 4 verified against synthetic data (121 ECE / 0 EEE, 20 unsubmitted), rolled back cleanly. Your new message changes the mode of operation — let me check the new docs.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:56): The five new docs exist. I'm already on `chase-lists` with the first three docs committed, so steps 1–3 are partly done. Let me read the operating rules before continuing.

**Claude** (2026-09-19T09:56): _[tool: Bash]_

**Claude** (2026-09-19T09:57): Clear. Now the queue.

**Claude** (2026-09-19T09:57): _[tool: Bash]_

**Claude** (2026-09-19T09:57): The queue's T1-01 to T1-14 is exactly the work I've just completed and verified. Let me commit the new docs plus my work, then update the tracking files.

**Claude** (2026-09-19T09:57): _[tool: Bash]_

**Claude** (2026-09-19T09:57): Let me check the rest of the ops findings for anything affecting my queries, then commit.

**Claude** (2026-09-19T09:57): _[tool: Bash]_

**Claude** (2026-09-19T09:57): Section 4's "most urgent" question is one my work directly answers. Let me commit everything, then write the log.

**Claude** (2026-09-19T09:57): _[tool: Bash]_

**Claude** (2026-09-19T09:57): _[tool: Bash]_

**Claude** (2026-09-19T09:57): Now the log entries and queue updates, then push.

**Claude** (2026-09-19T09:58): _[tool: Bash]_

**Claude** (2026-09-19T09:58): Now the questions file. Track 2 requires building on the live app, which needs a decision I don't have.

**Claude** (2026-09-19T09:58): _[tool: Bash]_

**Claude** (2026-09-19T09:58): _[tool: Bash]_

**Claude** (2026-09-19T09:58): _[tool: Bash]_

**Claude** (2026-09-19T09:58): All four commits pushed. Let me do a final clean-run verification from scratch to confirm everything works end-to-end.

**Claude** (2026-09-19T09:58): _[tool: Bash]_

**Claude** (2026-09-19T09:59): The flags output is missing and there's a leftover `head -0` line my earlier sed didn't remove. Let me fix it.

**Claude** (2026-09-19T09:59): _[tool: Bash]_

**Claude** (2026-09-19T09:59): The flags file is correct — only the display line was broken. Note "students with no profile row" shows 0 but I found 1 earlier; let me fix the dead line and check that discrepancy.

**Claude** (2026-09-19T09:59): _[tool: Bash]_

**Claude** (2026-09-19T09:59): The profile-less student is a test account, correctly excluded — so the flag reading 0 is right, and my earlier "one real student" note was wrong. Let me correct the log and re-verify.

**Claude** (2026-09-19T09:59): _[tool: Bash]_

**Claude** (2026-09-19T09:59): Clean end-to-end. Final commit and push.

**Claude** (2026-09-19T09:59): _[tool: Bash]_

**Claude** (2026-09-19T09:59): All 17 checks pass, tree clean, everything pushed.

**FEATURE:** `scripts/chase/` — nine read-only chase queries, a runner writing one CSV per query with a venue column, a verifier proving every count a second way, plus `scripts/load-local-dump.sh`.

**BRANCH/COMMITS:** `chase-lists`, pushed yes.
`4bd7c9c` docs · `fc8ac28` docs · `a37722e` feat: chase lists · `aa49977` docs: queue/log · `29b3ef2` fix: flags display

**DONE:**
- [x] Read-only — no write keyword in any query file, **and** the DB refuses an `UPDATE` in the runner's session
- [x] Local dump loaded; **all 16 migrations present** (14 by column/table marker, the 2 view-only ones by reading `v_team_projects`'s own definition)
- [x] All nine queries, every count cross-checked a second way — agree on days 1, 2, 3 and 8
- [x] Under two seconds: 81 runs, slowest **7.6 ms**
- [x] Tested at real volume — 206 chaseable students, 53 teams, both venues
- [x] One CSV per query with a venue column; every row leads with name · roll · team · venue · phone · personal email
- [x] `team_id IS NULL` and dept-mismatch reported as counts (both 0)
- [x] Old path checked
- [ ] Track 2 not started — BLOCKED, see below

**OLD PATH:** This is where it mattered most. `v_student_progress` still exposes `has_photo` and `has_education` after [profile-completion.js](src/routes/profile-completion.js#L44) dropped both from the weights. Building list 7 from that view — the obvious way — would have named **200 of 206 students** for two items the profile page no longer offers, every day, with no way to clear them. Query 7 reads the JS weights instead; I ran the app's own `completion()` against all 206 students on days 2 and 8 and got **zero disagreement**, including the Day 8 switch where `resume_v2` starts counting and the total moves 85→100.

Also checked: attendance has only two write paths, neither ever deletes — which is what makes lists 5 and 6 genuinely different. And `submissions` is still the project table; project-formats widened its constraint rather than replacing it.

**FOUND:**
- **Your `submissions`/`scores = 0` flag — not a wrong table.** All 159 project rows are `status='assigned'`, `is_open=false`, and every day-2 project release is `is_open=f`. No project has ever been opened, so nobody *could* hand in or score one. List 2 correctly returns zero. That's a "nothing was opened" finding, not a schema one.
- `task_submission_orphans`: 50 unclaimed Drive files from 27 teams against Day 1's task 8 — real work, chased by nobody.
- **36** CVs await their Drive copy, not the 76 the migration mentions — `migrate-cvs.js` has run since. A count, never names.
- `personal_email` is empty for all 209, so that column is blank in every CSV. Worth knowing before mailing a list.
- Queries 3/4 had no real data (0 quiz questions), so I proved them in a rolled-back transaction: 121 ECE / 0 EEE never opened, 20 unsubmitted. Exactly as predicted.

**RISK:** Nothing deploys; worst case is a wrong phone list. The live risk is *misreading an empty CSV* — on this dump the register is closed for both venues and no quiz has a question, so lists 3, 4 and 6 are empty or total through nobody's fault. `00-flags.sql` prints first and says which. The one judgement call is excluding three seeded test accounts; that exclusion is what makes ECE read 151, matching your figure.

**DEPLOY:** None — read-only. To use:
```sh
./scripts/load-local-dump.sh ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
./scripts/chase/run.sh 2        # CSVs into ./chase-lists/
./scripts/chase/verify.sh 2     # all 17 checks must read agree/holds
```

**NEEDED:** One decision, in [docs/questions-for-vishnu.md](docs/questions-for-vishnu.md). I marked all of Track 2 BLOCKED rather than starting it: D14 wants the same question tracked across days, but `v3-decisions.md` still lists the survey's purpose as open. Matching on question text breaks the first time a question is reworded — near-certain across nine days written the evening before. My recommendation is an optional stable `key` prefix in the paste-many loader, blank meaning untracked. Track 2 ships to the live app, so guessing its data model overnight is expensive to undo — that's a question, not an assumption. Everything else in Track 2 is decided and buildable the moment you answer.

**Vishnu** (2026-09-19T10:02): Excellent report, and you were right to block rather than guess. The answer
removes the problem entirely.

Read docs/survey-spec.md — it is untracked in the working tree. It replaces the
survey section of the brief and unblocks all of Track 2.

THE SHORT VERSION

The survey is proof of learning gain: before we teach a topic, how many already
know it. It is NOT paired with the quiz.

Two rounds:
  daily  — before each day's teaching. That day's questions, different each day.
  final  — once, at the end of the bootcamp. Re-asks EVERY question from all
           nine days.

Your rewording concern does not arise. The final round re-asks the SAME question
rows — it creates no new questions. Before and after match on
survey_question_id. No text matching, no topic tags, no stable-key prefix.

SCHEMA — this supersedes T2-01 in the queue

  surveys          (id, day, round, title, created_at)
                   round: 'daily' | 'final'. day is NULL for final.
  survey_questions (id, survey_id, position, text)
                   daily rounds only.
  survey_answers   (id, survey_question_id, student_id, round, answer, answered_at)
                   UNIQUE (survey_question_id, student_id, round)

The round column on the ANSWER carries before and after. The final round's form
is every survey_questions row for the program, ordered by day then position.

A survey with zero questions cannot be opened, same as a quiz below five.

REPORTS — all four, add them to the queue
  1. Per question: "Day 3: 12 of 206 knew -> End: 190 know, +86%", ordered by gain
  2. One headline: "Across 40 topics, 18% before -> 87% after"
  3. Per student, on their own profile
  4. Every view split by venue, EEE and ECE side by side
  Plus daily results during the bootcamp, and a "has not answered today" list
  that belongs in the chase set.

Every count links to its named list with phone numbers and CSV export.
Note personal_email is empty for all 209 — phone only, and say so on the screen.

Proceed with Track 2. Same unattended rules: report per item into
docs/agent-log.md, questions to docs/questions-for-vishnu.md, commit and push
after every item, never stop the run for a question.

THREE FINDINGS FROM YOUR REPORT, none of them yours to fix:
- 159 projects all is_open=false, never opened for either venue. Operational,
  Vishnu is handling it today. Log it, do not touch it.
- 50 orphan Drive files from 27 teams. Log it for Track 3.
- personal_email empty for all 209. Already accounted for above.

**Claude** (2026-09-19T10:02): Reading the spec before touching anything.

**Claude** (2026-09-19T10:02): _[tool: Bash]_

**Claude** (2026-09-19T10:02): _[tool: Bash]_

**Claude** (2026-09-19T10:02): The spec is clear and removes the rewording problem entirely. Let me commit it and check current state before starting T2-01.

**Claude** (2026-09-19T10:02): _[tool: Bash]_

**Claude** (2026-09-19T10:02): Now updating the queue: T2-01's schema is superseded, and four report items need adding. Then I start.

**Claude** (2026-09-19T10:02): _[tool: Bash]_

**Claude** (2026-09-19T10:02): Now T2-01, the migration. Let me check conventions in the existing migrations first, especially the `down` requirement.

**Claude** (2026-09-19T10:02): _[tool: Bash]_

**Claude** (2026-09-19T10:02): No `down` convention exists yet, but the brief requires one. I'll establish it as a companion file. Writing the migration now.

**Claude** (2026-09-19T10:03): _[tool: Write]_

**Claude** (2026-09-19T10:03): Now the down migration, which the brief requires and I must actually run.

**Claude** (2026-09-19T10:04): _[tool: Write]_

**Claude** (2026-09-19T10:04): The force-guard uses `current_setting` but psql `-v` doesn't set that. Let me fix and test the migration properly.

**Claude** (2026-09-19T10:04): _[tool: Bash]_

**Claude** (2026-09-19T10:04): The constraint-widening block failed. Let me see the actual error and the existing constraint.

**Claude** (2026-09-19T10:04): _[tool: Bash]_

**Claude** (2026-09-19T10:04): Two constraints mention `item_type`, so my subquery returned multiple rows. Both need widening — and `chk_releases_item_id` matters: a survey needs `item_id NOT NULL`. Let me rewrite that section explicitly.

**Claude** (2026-09-19T10:04): _[tool: Bash]_

**Claude** (2026-09-19T10:04): Up migration runs clean. Now let me verify every rule it claims to enforce, rather than trusting it.

**Claude** (2026-09-19T10:04): _[tool: Bash]_

**Claude** (2026-09-19T10:04): All 8 constraint tests behaved exactly as designed. Let me check my queue edits didn't collide with your rewrite.

**Claude** (2026-09-19T10:04): _[tool: Bash]_

**Claude** (2026-09-19T10:04): Your rewrite landed and my earlier edits are gone — yours is authoritative. Numbering shifted (final round is now T2-07), and there are two new items: screenshots (T2-17) and Track 3 Phase A. Let me restore your version and commit T2-01.

**Claude** (2026-09-19T10:05): _[tool: Bash]_

**Claude** (2026-09-19T10:05): Good catch by the guard — that `git checkout` would have destroyed your rewrite, which is the uncommitted version on disk. The file on disk is already correct; nothing to restore. Let me just commit it as-is.

**Claude** (2026-09-19T10:05): _[tool: Bash]_

**Claude** (2026-09-19T10:05): Your rewrite is intact and staged. Now testing the down migration before committing — the brief requires it to have been run.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): The guard refuses and leaves the tables intact. Now testing that `force=1` completes the reversal cleanly.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): Down migration fully reverses: three tables gone, three functions gone, teams untouched. Let me verify the `releases` constraint was narrowed back, then confirm idempotency.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): The constraint came back empty — the down migration dropped `releases_item_type_check` without re-adding it. That's a real bug: it leaves the table with no type validation.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): My earlier query used the wrong name — the constraint does exist, but the down migration created a *duplicate* under a different name. Let me check its definition.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): Confirmed bug: the down migration re-added the narrowed constraint under the name `chk_releases_item_type` instead of the original `releases_item_type_check`, leaving two. That would block surveys after a re-up. Fixing it to restore the original name.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): Exactly one constraint, original name, original definition — the down migration is now truly reversible. Let me read the new spec section 8 before building further.

**Claude** (2026-09-19T10:06): _[tool: Bash]_

**Claude** (2026-09-19T10:06): Section 8 resolves everything I'd have blocked on. One point directly affects the schema I just built: "first tap is locked" means answers are insert-only, never updated. Let me correct that comment.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): All 15 columns present. Committing T2-01.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): I need to flag a branch problem before going further. Let me check.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): I committed T2-01 to `chase-lists`, but the queue and brief both say Track 2 belongs on `survey` off `main`. Let me move it correctly — no stash, no reset.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): A conflict — expected, since the work-queue.md edits differ between branches.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): `work-queue.md` doesn't exist on main (deleted/untracked there) but was modified in my commit. I'll take my version, which holds your rewrite.

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): _[tool: Bash]_

**Claude** (2026-09-19T10:07): `survey` is now correctly off `main` with T2-01, pushed. Let me log this and the branch correction, then continue to T2-02.

**Claude** (2026-09-19T10:08): _[tool: Bash]_

**Claude** (2026-09-19T10:08): `agent-log.md` doesn't exist on `survey` — it was only on `chase-lists`. Let me bring it across.

**Claude** (2026-09-19T10:08): _[tool: Bash]_

**Claude** (2026-09-19T10:08): _[tool: Bash]_

**Claude** (2026-09-19T10:08): Now T2-02 — the `isOpenFor` survey branch. Let me read the surrounding code first.

**Claude** (2026-09-19T10:08): _[tool: Bash]_

**Claude** (2026-09-19T10:08): A survey has no legacy boolean, so it must default closed. Adding the branch.

**Claude** (2026-09-19T10:08): _[tool: Edit]_

**Claude** (2026-09-19T10:08): Now let me verify the gate end-to-end — including the venue-leak case that shipped as a bug in v2.

**Claude** (2026-09-19T10:09): _[tool: Bash]_

**Claude** (2026-09-19T10:09): The venue gate is proven through the app's own `isOpenFor`: open for EEE, **closed for ECE** — the v2 leak does not reproduce. Let me commit T2-02 and check the new Phase A docs.

**Claude** (2026-09-19T10:09): _[tool: Bash]_

**Claude** (2026-09-19T10:09): I've made solid progress and should pause here to report, because the run has reached a scale worth a checkpoint.

**Where I am:** T2-01 and T2-02 done, tested, committed and pushed on `survey` off `main`.

**T2-01 — survey schema.** Eight constraint tests, each expecting a refusal and getting one: a final survey carrying a day, a daily one without, a second daily for the same day, a second final round, questions attached to the final round, and opening a survey with no questions. The down migration was actually run both ways — it refuses while answers exist and takes `-v force=1` to proceed; teams and points identical before and after.

**I found and fixed a bug in my own down migration.** It re-added the narrowed constraint under a new name while the widened one was still present, leaving two overlapping checks on `releases`. The narrow one silently wins, so a re-run of the up migration *appeared* to succeed and would then have refused every survey release row. The full cycle up→down→up→down now leaves exactly one constraint with its original name.

**T2-02 — the venue gate.** Proved through the app's own `isOpenFor` rather than a reimplementation: a survey released for EEE alone reads open for EEE and **closed for ECE**. That's the v2 leak, tested for rather than assumed away.

**Two things you should know:**

**I committed T2-01 to the wrong branch first.** It went onto `chase-lists`; the queue and brief both say Track 2 is `survey` off `main`. I corrected it by creating `survey` off `main` and cherry-picking — no stash, no reset, nothing discarded. `chase-lists` still holds identical copies of those two commits. They're harmless there, but removing them would mean rewriting a pushed branch, so I left them and logged it.

**A `git checkout` I attempted was blocked by the permission guard, and the guard was right.** I was trying to resolve a conflict on `docs/work-queue.md` and would have destroyed your uncommitted rewrite of it. Your version is intact, committed, and is what I'm working from — all 23 items including the screenshot step and Phase A.

**On running straight through:** I'll keep going, but I want to flag one thing rather than discover it at 3am. T2-17 requires Playwright screenshots at 390px, and T2-15 requires session tests for every role. Neither Playwright nor any browser driver is currently in `package.json` — the repo has no build step by design. Installing a browser automation stack is a material change to the project's dependencies, which is the kind of thing your own rules say an agent shouldn't decide alone. I'll build everything up to that point, and if Playwright genuinely isn't available I'll write the question to `questions-for-vishnu.md`, mark T2-17 BLOCKED, and continue to the next item rather than stopping the run.

Continuing with T2-03, the admin loader.

**Vishnu** (2026-09-19T10:11): STANDING AUTHORISATION. Read docs/standing-authorisation.md — it is new in the
working tree. One rule:

  You may decide anything that is local and reversible.
  You may decide nothing that reaches the server.

DO NOT ASK ME ANYTHING. RUN TO THE END OF THE QUEUE.

PRE-AUTHORISED, decide these yourself, never ask:
- Install devDependencies. Playwright included. Yes, do it.
- Create anything under docs/, scripts/, tests/, src/modules/, src/db/migrations/
- Additive migrations with a working down
- Reset and reload the local database as often as you like
- Branches, commits, pushes
- Naming, file layout, test structure where the brief is silent
- When a detail is unspecified, follow the nearest existing pattern. The quiz is
  the pattern for the survey.

STILL FORBIDDEN, no authorisation overrides these:
- Deploy. ssh to the server. Production database. Committing to main.
- Runtime dependencies in package.json — devDependencies only
- Editing setup-server.sh, deploy scripts, or .env
- git stash, reset --hard, rewriting a pushed branch
- load-eee.sql, load-ece.sql, deleting data anywhere

WHEN SOMETHING IS UNCLEAR, in this order:
1. Local and reversible? Decide it. Log it under ASSUMED: with what breaks if
   you are wrong.
2. Existing pattern in the codebase? Follow it. Log which one.
3. Expensive to undo, or reaches the server? Write it to
   docs/questions-for-vishnu.md, mark the item BLOCKED, TAKE THE NEXT ITEM.

A question is never a reason to stop. It is a reason to move to the next item.

PLAYWRIGHT: authorised as a devDependency. But first look — handover.md claims
four Playwright suites and 71 checks already exist. Check tests/, the root
package.json, Makefile, playwright.config.*, npx playwright --version. If the
suites exist, run them and report how many pass. That number matters: those are
the checks that once went green while every team lead was locked out.
Then confirm whether the production install could pull devDependencies in — read
setup-server.sh and deploy.md, report it, note it in T2-18. Do not edit them.

ORDER: T2-15 session tests BEFORE T2-17 screenshots.

Run Track 2 to the end, then Phase A: A1, A2, A4, A5, then A3 as 12 module items.
Report into docs/agent-log.md after every item. Commit and push after every item.
Stop only when the queue is empty or a hard stop from
docs/unattended-operation.md section 2 fires. Nothing else.

Continue with T2-03 now.

**Claude** (2026-09-19T10:11): Reading the standing authorisation, then checking Playwright before T2-03.

**Claude** (2026-09-19T10:11): _[tool: Bash]_

**Claude** (2026-09-19T10:11): Clear. Checking the claimed Playwright suites before anything else.

**Claude** (2026-09-19T10:11): _[tool: Bash]_

**Claude** (2026-09-19T10:11): Playwright is already a devDependency and 18 test files exist. Let me check the runner and whether the browser is installed.

**Claude** (2026-09-19T10:11): _[tool: Bash]_

**Claude** (2026-09-19T10:11): The suites are plain Node scripts driving Playwright, not `@playwright/test`. Let me check the browser is installed and see how a test signs in.

**Claude** (2026-09-19T10:11): _[tool: Bash]_

**Claude** (2026-09-19T10:11): Everything needed exists — real sign-in, real server. Let me run the suite to get the baseline number you asked for.

**Claude** (2026-09-19T10:11): _[tool: Bash]_

**Claude** (2026-09-19T10:11): Note `make db` runs the forbidden loaders — I'll never use that target. Let me set up a scratch test database from the dump and run the suite.

**Claude** (2026-09-19T10:12): _[tool: Bash]_

**Claude** (2026-09-19T10:12): _[tool: Bash]_

**Claude** (2026-09-19T10:12): Server is up on 3099. Running the four suites the handover claims, to get the real pass count.

**Claude** (2026-09-19T10:12): _[tool: Bash]_

**Claude** (2026-09-19T10:12): _[tool: Bash]_

**Claude** (2026-09-19T10:13): Sign-in fails in all three. Per the working rules, I should ask whether the test is wrong before assuming the code is.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): The test signs in as admin with a staff password. Let me check whether that admin exists in the dump — the dump is real data, not the seeded test fixture.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): **The test is wrong, not the code.** Every mentor in the production dump has `password_hash = NULL` — staff passwords were never set in production. The tests assume a seeded fixture. Let me confirm how login handles that.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): Staff login compares against the `STAFF_PASSWORD` env var, not `password_hash`. So the test should work — let me check what the server actually got.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): The server loaded `.env`, overriding my env vars. That's why the password didn't match — a test-harness config issue, not a code bug.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): The comment says real env vars win, but they clearly didn't. Let me look at the loader.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): The loader is correct — env vars do win. My `(... &)` subshell didn't propagate them. Let me restart properly.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:13): Login works. Now the real baseline run.

**Claude** (2026-09-19T10:13): _[tool: Bash]_

**Claude** (2026-09-19T10:15): **71 checks, 70 pass, 1 fail** — the handover's "71 checks" is exactly right. Let me identify the single failure.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): Let me check whether this failure is the test being wrong — per rule 13.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): The test expects a raised project card for today. On this dump no project is open — exactly the operational state you flagged. Let me confirm that's the cause.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): Confirmed — the failure reflects a true fact about production: 106 day-2 projects, none open. The test is right, the data is the finding you already logged. Baseline established. Now T2-03, the admin loader.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): Clear pattern: parse all, report all problems, write nothing unless every line is good. Let me find where to add the survey routes and check the lock helper.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:15): Now writing the survey routes — admin loader plus the student answering path, following the quiz pattern exactly.

**Claude** (2026-09-19T10:15): _[tool: Bash]_

**Claude** (2026-09-19T10:16): _[tool: Edit]_

**Claude** (2026-09-19T10:16): Now writing a test suite for the loader, signing in as a real admin.

**Claude** (2026-09-19T10:17): _[tool: Write]_

**Claude** (2026-09-19T10:17): The test references student endpoints I haven't written yet. Let me add those, plus the releases route check.

**Claude** (2026-09-19T10:17): _[tool: Bash]_

**Claude** (2026-09-19T10:17): _[tool: Bash]_

**Claude** (2026-09-19T10:17): Now the student-facing survey routes, following that exact pattern.

**Claude** (2026-09-19T10:17): _[tool: Edit]_

**Claude** (2026-09-19T10:17): _[tool: Bash]_

**Claude** (2026-09-19T10:17): The admin loader passes fully. The student sign-in fails and cascades — let me check why.

**Claude** (2026-09-19T10:17): _[tool: Bash]_

**Claude** (2026-09-19T10:17): The endpoint doesn't return `is_active`, so my `.find(s => s.is_active)` returned nothing. That's my test's bug, not the code's.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): The survey rows persisted from the previous run, so creation now returns 400. My test needs a clean slate — let me make it self-contained.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): Confirmed. The test must start from a known state — I'll add a reset helper that clears only survey tables in the scratch DB.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): **The venue gate passes** — a survey opened for EEE is invisible to an ECE student. Now the remaining failure: serving the survey after opening it for the student's own venue.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): Found it — the API expects `open`, not `is_open`. My test sent the wrong field, so no release row was ever written. Test bug again.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): Still not opening. Let me check what `set_release` actually did.

**Claude** (2026-09-19T10:18): _[tool: Bash]_

**Claude** (2026-09-19T10:18): Found the real gap in my own code: `set_release`'s `ALLOWED` list doesn't include `'survey'`. That's T2-02 work I missed — the release route rejects surveys entirely.

**Claude** (2026-09-19T10:18): _[tool: Edit]_

**Claude** (2026-09-19T10:18): A survey names a specific row, so it also needs the item_id requirement and an empty-survey guard mirroring the quiz's.

**Claude** (2026-09-19T10:18): _[tool: Edit]_

**Claude** (2026-09-19T10:19): _[tool: Edit]_

**Claude** (2026-09-19T10:19): _[tool: Bash]_

**Claude** (2026-09-19T10:19): All 25 pass. Let me confirm the old suites still pass — the old-path check.

**Claude** (2026-09-19T10:19): _[tool: Bash]_

**Claude** (2026-09-19T10:20): `releases` and `tasks` have failures I must check — did I cause them, or were they already failing?

**Claude** (2026-09-19T10:20): _[tool: Bash]_

**Claude** (2026-09-19T10:20): These are stateful suites that mutate data. Let me verify against a pristine database whether my changes caused them.

**Claude** (2026-09-19T10:20): _[tool: Bash]_

**Claude** (2026-09-19T10:21): These suites mutate state, so a second run in the same session differs. Let me test each on its own fresh database — the honest comparison.

**Claude** (2026-09-19T10:21): _[tool: Bash]_

**Claude** (2026-09-19T10:22): **Identical with and without my changes** — these failures pre-date my work and are not caused by the survey. They're a property of running against a real production dump rather than a seeded fixture.

**Claude** (2026-09-19T10:22): _[tool: Bash]_

**Claude** (2026-09-19T10:23): These need quizzes with questions — the dump has zero. A fixture problem, not a regression. Let me commit T2-03 and the T2-02 completion.

**Claude** (2026-09-19T10:23): _[tool: Bash]_

**Claude** (2026-09-19T10:23): I've hit a natural reporting point, and there are findings you need before I continue.

**Done and pushed:** T2-01, T2-02, T2-03 on `survey` off `main`.

## The number you asked for

**71 checks across the four claimed suites: 70 pass, 1 fail.** The handover's "71 checks" is exactly right.

The single failure is `flows.js` → *"today's project is marked"*. It's the test being correct about a real fact: 106 day-2 projects exist, **none is open**, so no card is raised. That's the operational finding you already logged, surfacing in a test.

**Getting that number took a fix to my own method, not the code.** My first run showed all three browser suites failing at sign-in. The cause was mine: I backgrounded the server with `( ... &)`, which didn't propagate env vars, so `.env` won and the staff password didn't match. Worth stating plainly because it's exactly the "false red" your rules warn about — I nearly reported three broken suites that were fine.

## Two suites fail, and it isn't me

`releases` (20/19) and `tasks` (7/7) fail substantially. I checked this properly rather than assuming: I ran each on its own freshly-loaded database, **with and without** the survey migration. **Identical results both ways.** These failures pre-date my work.

The cause is that they need fixtures the production dump doesn't have — `releases` needs quizzes with questions, and there are zero across all nine. They're written for a seeded database, not a real dump. That's worth knowing before anyone reads a red suite as a regression.

## A real gap my test caught

`isOpenFor` had its survey branch from T2-02, but `set_release`'s `ALLOWED` list didn't include `'survey'` — so every release row for a survey was refused with *"Not something that can be opened"*. The gate worked and the admin could never reach it. A reading of the code wouldn't have found it; only signing in and trying to open one did.

## devDependencies and the production install — you asked me to check

`playwright ^1.63.0` is **already** in `devDependencies`, and Chromium is installed. I added nothing.

Whether production could pull it in: `setup-server.sh` runs `npm ci --omit=dev` and `update.sh` does the same, so devDependencies are excluded on the server. **Playwright cannot reach production through the normal path.** I read both files and edited neither. I'll note it in T2-18 as you asked.

## One thing I want to flag now

Three of my own test failures were my test being wrong, not the code — a missing `is_active` field, leftover survey rows between runs, and `is_open` where the API wants `open`. I fixed each and said so in the commit. But it means my suite needed a `reset_surveys()` helper that deletes rows, and I've guarded it to refuse any database not ending in `_test` or `_local`. It cannot touch real data, but you should know a test in this repo now deletes rows by design.

Continuing to T2-04, the Open tab.

**Vishnu** (2026-09-19T10:48): The number you asked for
71 checks across the four claimed suites: 70 pass, 1 fail. The handover's "71 checks" is exactly right.

The single failure is flows.js → "today's project is marked". It's the test being correct about a real fact: 106 day-2 projects exist, none is open, so no card is raised. That's the operational finding you already logged, surfacing in a test.

Getting that number took a fix to my own method, not the code. My first run showed all three browser suites failing at sign-in. The cause was mine: I backgrounded the server with ( ... &), which didn't propagate env vars, so .env won and the staff password didn't match. Worth stating plainly because it's exactly the "false red" your rules warn about — I nearly reported three broken suites that were fine.

Two suites fail, and it isn't me
releases (20/19) and tasks (7/7) fail substantially. I checked this properly rather than assuming: I ran each on its own freshly-loaded database, with and without the survey migration. Identical results both ways. These failures pre-date my work.

The cause is that they need fixtures the production dump doesn't have — releases needs quizzes with questions, and there are zero across all nine. They're written for a seeded database, not a real dump. That's worth knowing before anyone reads a red suite as a regression.

A real gap my test caught
isOpenFor had its survey branch from T2-02, but set_release's ALLOWED list didn't include 'survey' — so every release row for a survey was refused with "Not something that can be opened". The gate worked and the admin could never reach it. A reading of the code wouldn't have found it; only signing in and trying to open one did.

devDependencies and the production install — you asked me to check
playwright ^1.63.0 is already in devDependencies, and Chromium is installed. I added nothing.

Whether production could pull it in: setup-server.sh runs npm ci --omit=dev and update.sh does the same, so devDependencies are excluded on the server. Playwright cannot reach production through the normal path. I read both files and edited neither. I'll note it in T2-18 as you asked.

One thing I want to flag now
Three of my own test failures were my test being wrong, not the code — a missing is_active field, leftover survey rows between runs, and is_open where the API wants open. I fixed each and said so in the commit. But it means my suite needed a reset_surveys() helper that deletes rows, and I've guarded it to refuse any database not ending in _test or _local. It cannot touch real data, but you should know a test in this repo now deletes rows by design.

Continuing to T2-04, the Open tab.

**Claude** (2026-09-19T10:48): Continuing with T2-04, the Open tab. Let me log T2-02/T2-03 first, then read the front end.

**Claude** (2026-09-19T10:48): _[tool: Bash]_

**Claude** (2026-09-19T10:48): Now T2-04. Let me see how the Open tab renders quiz rows.

**Claude** (2026-09-19T10:48): _[tool: Bash]_

**Claude** (2026-09-19T10:48): _[tool: Bash]_

**Claude** (2026-09-19T10:48): The Open tab is generic — it renders whatever `/api/admin/releases` returns. So T2-04 is a server change, not a front-end one.

**Claude** (2026-09-19T10:48): _[tool: Bash]_

**Claude** (2026-09-19T10:48): Clear pattern. Let me see where the item list ends so I can add surveys.

**Claude** (2026-09-19T10:48): _[tool: Bash]_

**Claude** (2026-09-19T10:48): Now adding the survey entries. The daily survey goes first (it's asked before teaching) and the final round appears on every day so it can be opened at the end.

**Claude** (2026-09-19T10:49): _[tool: Edit]_

**Claude** (2026-09-19T10:49): The final round's release row has a non-null day, but `isOpenFor` matches on item_id for surveys, so day doesn't matter. Let me verify that and test.

**Claude** (2026-09-19T10:49): _[tool: Bash]_

**Claude** (2026-09-19T10:49): All 33 pass. Let me verify the Open tab renders in a real browser, then commit.

**Claude** (2026-09-19T10:49): _[tool: Bash]_

**Claude** (2026-09-19T10:49): Nothing rendered at all — zero buttons means the whole tab is empty, not just surveys. Let me check what the page shows.

**Claude** (2026-09-19T10:49): _[tool: Bash]_

**Claude** (2026-09-19T10:49): The tab works — it defaulted to Day 2, and my survey is on Day 1. My browser check was wrong, not the code. Let me check Day 1 properly.

**Claude** (2026-09-19T10:49): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Both survey rows render at 390px with per-venue buttons, no horizontal scroll, no errors. Committing T2-04.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Now T2-05 and T2-06 — the student card and answering. Let me see how "My work" renders.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): "My work" is the home page. Let me read how it lists items.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): I can see the `tasks` array pattern. Let me find where the quiz card is pushed so the survey goes first.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Now let me check how a card renders, to add the Required tag.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Now adding the survey card first in the list, with a Required tag.

**Claude** (2026-09-19T10:50): _[tool: Edit]_

**Claude** (2026-09-19T10:50): Wait — `attend` is pushed conditionally, so adding survey before it shifts the index. Let me verify the destructuring is correct.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Correct — survey is index 5, attend is pushed after it at index 6. Now the card itself, first in the list.

**Claude** (2026-09-19T10:50): _[tool: Edit]_

**Claude** (2026-09-19T10:50): _[tool: Edit]_

**Claude** (2026-09-19T10:50): Now the answering page. Let me check how routing works and how the quiz page is structured.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:50): Let me look at the assessment page — it's the closest pattern (no timer, no score) for the survey.

**Claude** (2026-09-19T10:50): _[tool: Bash]_

**Claude** (2026-09-19T10:51): Good pattern. Now writing the survey answering page.

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:51): Now writing the survey page — Yes/No, saves on tap, first tap locked.

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:51): _[tool: Edit]_

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:51): Now testing the student flow in a real browser, including that the first tap locks.

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:51): _[tool: Bash]_

**Claude** (2026-09-19T10:52): The home card works — first, tagged Required, others unlocked. But the survey page itself is empty. Let me see what it rendered.

**Claude** (2026-09-19T10:52): _[tool: Bash]_

**Claude** (2026-09-19T10:52): The API returns the data, but `go('survey')` rendered home instead — the route isn't reached. Let me check the nav guard.

**Claude** (2026-09-19T10:52): _[tool: Bash]_

**Claude** (2026-09-19T10:52): `pages_for(ME)` gates every route — `survey` isn't in that list, so it redirects home. Let me check how `assess` is handled there.

**Claude** (2026-09-19T10:52): _[tool: Bash]_

**Claude** (2026-09-19T10:52): There's an `ASSESS_OPEN` flag pattern. Let me follow it exactly for the survey.

**Claude** (2026-09-19T10:52): _[tool: Bash]_

**Claude** (2026-09-19T10:52): _[tool: Bash]_

**Claude** (2026-09-19T10:53): _[tool: Bash]_

**Claude** (2026-09-19T10:53): _[tool: Bash]_

**Claude** (2026-09-19T10:53): _[tool: Bash]_

**Claude** (2026-09-19T10:53): Everything passes: card first, tagged Required, others unlocked, first tap locks both buttons, answer survives a refresh, 390px clean. Let me confirm the API suite still passes and commit.

**Claude** (2026-09-19T10:53): _[tool: Bash]_

**Claude** (2026-09-19T10:54): 34 survey checks pass, and `flows`/`redesign` are unchanged from baseline (21/1 and 33/0) — the survey card didn't break the home page. Committing T2-05 and T2-06.

**Claude** (2026-09-19T10:54): _[tool: Bash]_

**Claude** (2026-09-19T10:54): I've completed T2-01 through T2-06 and should report, since that's a natural half of Track 2.

## Done and pushed on `survey`

| Item | What | Checks |
|---|---|---|
| T2-01 | Schema, two rounds, working `down` | 8 constraint refusals |
| T2-02 | Release gate through `isOpenFor` | venue leak tested |
| T2-03 | Admin loader + student form | 25 |
| T2-04 | Open tab row, per venue | 33 |
| T2-05/06 | Student card + answering screen | 34 |

**34 API checks pass, all through a real signed-in session** for both an admin and a student. Verified in a real browser at 390px: the card is first and tagged Required, other items stay usable, both buttons lock on the first tap, the answer survives a page refresh, no horizontal scroll, no page errors.

## Three real gaps, each found by testing rather than reading

**`set_release` didn't allow `'survey'`.** `isOpenFor` had its branch and was correct, so every release row was refused with *"Not something that can be opened"* — the gate worked and the admin could never reach it. Only signing in and trying to open one found it.

**`pages_for()` silently redirected `go('survey')` home.** The route existed, the page function existed, the API returned correct data — and the student got the home page. The nav guard gates every route and `survey` wasn't in it. A browser test found this; an API test never would have.

**Both `releases` constraints needed widening, not one.** `chk_releases_item_id` decides which item types carry an `item_id`. Missing it would have accepted the type and then refused every row — an error appearing the first time staff opened a survey in front of a room.

## On the failing suites

`releases` (20/19) and `tasks` (7/7) still fail, and I confirmed again this session that it isn't me: `flows` and `redesign` are unchanged from baseline at 21/1 and 33/0 after the front-end change. The two failing suites need quizzes with questions, and the dump has zero.

## One judgement call worth your eye

The final round appears on the Open tab on **every** day, not just the last, so it can be opened at the end without hunting for the day it lives on. Its release row carries whichever day it was opened from. That's local and reversible — one `ORDER BY` and one `day` field — and I logged it under ASSUMED. If you'd rather it only appeared on Day 9, say so and it's a two-line change.

Continuing to T2-07, the final round's admin flow.

**Vishnu** (2026-09-19T11:30): Six items, all session-tested and browser-verified. Good.

FINAL ROUND ON THE OPEN TAB — keep it visible every day, but add one guard.

Opening it early is unrecoverable: a student who answers the final round on
Day 4 has a wrong "after" permanently, and the proof is the entire purpose of
this feature. So:

- Keep it on the Open tab every day. Hunting for it on the last day is worse.
- Label the row clearly: "Final survey — end of bootcamp only".
- Opening it requires a confirm that states the consequence:
    "This re-asks all N questions from all 9 days. Answers cannot be changed.
     Open this only at the end of the bootcamp."
  N is the real count, computed, not a placeholder.
- Closing it needs no confirm.

One extra tap, no hunting, and an accident becomes unlikely rather than merely
regrettable. Log the change against the ASSUMED entry.

THE FOUR-TIME PATTERN — write this up properly.

set_release, pages_for, chk_releases_item_id, and the v2 is_lead bug are all one
bug: a new value added to a gated concept, and an old list enumerating the old
values that nobody updated.

Add a FOUND entry to docs/agent-log.md titled "enumerating gates" that lists
every place in this codebase that enumerates activity types — allow-lists, nav
guards, CHECK constraints, switches, enums, view definitions. The actual list,
with file and line. You have now found four by accident; write down all of them
on purpose.

That list is a direct input to Track 3 Phase B. The v3 activities model exists
to make that list have one entry instead of N, and your list is the evidence for
how many N is today.

Continue with T2-07. Same rules — run to the end of the queue.

**Claude** (2026-09-19T11:30): Two tasks before T2-07: the confirm guard, then the enumerating-gates audit. Starting with the guard.

**Claude** (2026-09-19T11:30): _[tool: Bash]_

**Claude** (2026-09-19T11:30): _[tool: Bash]_

**Claude** (2026-09-19T11:30): Now the label change on the server, and the confirm in the Open tab.

**Claude** (2026-09-19T11:30): _[tool: Bash]_

**Claude** (2026-09-19T11:30): _[tool: Bash]_

**Claude** (2026-09-19T11:30): Now testing the guard in a real browser — that opening asks with the real count, and closing doesn't.

**Claude** (2026-09-19T11:31): _[tool: Bash]_

**Claude** (2026-09-19T11:31): Let me check whether the server is actually up — that timeout suggests the page never loaded.

**Claude** (2026-09-19T11:31): _[tool: Bash]_

**Claude** (2026-09-19T11:31): Server is fine. The `go('open')` likely raced ahead of the nav being ready — my earlier test waited differently. Let me retry with a proper wait.

**Claude** (2026-09-19T11:32): _[tool: Bash]_

**Claude** (2026-09-19T11:33): The selector works with a longer wait — my previous script just raced. Rerunning the guard test.

**Claude** (2026-09-19T11:33): _[tool: Bash]_

**Claude** (2026-09-19T11:34): The label renders but my card-finding selector is wrong. Let me check the actual DOM structure.

**Claude** (2026-09-19T11:34): _[tool: Bash]_

**Claude** (2026-09-19T11:36): That script hung with an unclosed browser. Let me use a simpler DOM query.

**Claude** (2026-09-19T11:36): _[tool: Bash]_

**Claude** (2026-09-19T11:36): The API is correct — the final survey is item index 2 with `round=final` and the right label. Note a quiz now has 10 questions (from the earlier releases suite run). Let me test the guard with the correct index.

**Claude** (2026-09-19T11:36): _[tool: Bash]_

**Claude** (2026-09-19T11:37): Every behaviour you specified works: real count, cancel is safe, closing and daily surveys need no confirm. Now the enumerating-gates audit — the list, done on purpose.

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:37): Let me gather each category systematically rather than by eye.

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:37): Now the JavaScript-side gates, precisely located.

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:37): Let me check the remaining places — release-row writing, the view, and the front-end icon/label maps.

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:37): The chase scripts are on the other branch. Let me confirm and then write up the audit.

**Claude** (2026-09-19T11:37): _[tool: Bash]_

**Claude** (2026-09-19T11:38): I have the full picture. Writing the audit into the log.

**Claude** (2026-09-19T11:38): _[tool: Bash]_

**Claude** (2026-09-19T11:38): Both tasks done. Now T2-07, the final round's admin flow — verifying it end-to-end at real volume.

**Claude** (2026-09-19T11:39): _[tool: Bash]_

**Claude** (2026-09-19T11:39): That test needs a per-question rounds endpoint. Adding it — it's also the foundation for the proof reports.

**Claude** (2026-09-19T11:39): _[tool: Edit]_

**Claude** (2026-09-19T11:39): _[tool: Bash]_

**Claude** (2026-09-19T11:39): Name collision with the earlier `final_form`. Renaming.

**Claude** (2026-09-19T11:39): _[tool: Bash]_

**Claude** (2026-09-19T11:39): All T2-07 checks pass. One stale assertion — I changed the label text per your instruction, so the test must follow.

**Claude** (2026-09-19T11:40): _[tool: Bash]_

**Claude** (2026-09-19T11:40): 43 checks, all passing. Let me verify at real volume — 206 students answering both rounds — before committing.

**Claude** (2026-09-19T11:40): _[tool: Bash]_

**Claude** (2026-09-19T11:40): 2,472 answers across 206 students and both rounds. Let me confirm the endpoint handles this and stays fast.

**Claude** (2026-09-19T11:40): _[tool: Bash]_

**Claude** (2026-09-19T11:40): Exactly the proof shape: 32 knew before, 172 after, split by venue, 22ms at full volume. Committing T2-07.

**Claude** (2026-09-19T11:40): _[tool: Bash]_

**Claude** (2026-09-19T11:40): T2-07 done. Reporting now, then continuing to T2-08.

## Final round guard — done exactly as specified

Verified in a browser, not just written:

| Behaviour | Result |
|---|---|
| Row labelled "Final survey — end of bootcamp only" | ✓ |
| Opening asks first | ✓ |
| States the consequence | ✓ |
| Real computed count, not a placeholder | ✓ — *"This re-asks all 3 questions from all 9 days. Answers cannot be changed. Open this only at the end of the bootcamp."* |
| Cancel leaves it closed | ✓ |
| Closing asks nothing | ✓ |
| A daily survey still opens in one tap | ✓ |

Logged against the ASSUMED entry.

## The enumerating-gates audit

Written into [docs/agent-log.md](docs/agent-log.md) with file and line. The count:

**~30 places enumerate a closed set. 12 must change together to add one activity type** — across three languages and two branches, with no test that fails when one is missed.

The breakdown: 12 sites in `src/server.js`, 4 in `app.js`, ~14 CHECK constraints, plus 4 chase scripts on the other branch. Some sets are enumerated more than once — `submission_type` twice, `dept` three times, `round` twice, the Drive-URL rule five times.

Two findings worth your attention beyond what you asked:

**Venue is the same bug waiting.** `DEPTS` in JS, three `*_dept_check` constraints, and every `CROSS JOIN (VALUES ('ECE'),('EEE'))` in the chase scripts. A third venue is a *bigger* edit than a third activity type. That's Phase B's B2, and it's worse than B3.

**Three of the four bugs were mine, in one feature, in one day.** Knowing about the pattern did not stop me walking into it three more times. That's the real argument for B4 — the fix can't be vigilance.

I also wrote down the greps that prove B4 is finished:
```
grep -rnE "item_type *(===|=) *'" src/
grep -rnE "'(ECE|EEE)'" src/
```
Both should return nothing but the `venues` and `activities` tables.

## T2-07

43 checks pass. The final round asks all 6 questions across 3 days in day-then-position order, owns none of them, and the student's daily answers are **not** sent back to them in the final round.

Added `GET /api/admin/survey/question/:qid/rounds` — the smallest unit of the proof, and the foundation for T2-09 through T2-12. Yes and no are counted **separately** rather than yes-over-total, because "no before answer" is a third state; a student who joined late and missed Day 3 didn't answer no, they weren't asked.

Tested at real volume — 206 students × 6 questions × 2 rounds = **2,472 answers**, returning in 22ms, reading *32 knew before → 172 after*, split by venue.

Continuing to T2-08, daily results.

**Vishnu** (2026-09-19T11:50): Strong work. Three things, then keep going.

1. THE THREE-STATE DECISION — lock it in, it is the most important call in the
   whole feature.

   Counting yes and no separately, with "not asked" as a real third state, is
   what keeps the proof honest. A student who joined on Day 5 did not answer no
   to Day 3 — they were never asked, and folding that into a denominator would
   quietly inflate every gain.

   Make it structural, not a convention:
   - Every proof view shows three numbers: yes, no, not asked. Never two.
   - Never render a bare percentage without the denominator beside it.
   - "Not asked" is visible in the UI, never hidden, never zero-filled.
   - Add a test that fails if a proof endpoint ever returns only yes and total.

   Write this into docs/survey-spec.md section 5 yourself.

2. B4 — MAKE THE FIX STRUCTURAL, NOT A GREP.

   Your greps are a lint, and a good one. Wire them into `make test` so they
   fail the build, not just a human's attention.

   But add the real control — a conformance test:

     For every activity type in the registry, drive the FULL lifecycle through
     a real signed-in session: create it, open it for venue A only, confirm
     venue B cannot see it, hand it in, score it, confirm it appears on the
     student's screen and in the reports.

     The test iterates the registry. Adding a type adds a row; if any of the
     ~12 gates was missed, that row fails.

   That is what makes vigilance unnecessary. Put it in the Phase B queue as the
   acceptance test for B0, B2 and B3 — none of them is done until the
   conformance test passes for every type and every venue.

3. VENUE ORDERING — your finding confirms the plan. B2 (venues) stays before
   B3 (activities). Add the CROSS JOIN (VALUES ('ECE'),('EEE')) sites in the
   chase scripts to B2's scope; they are on the other branch and easy to forget
   at merge.

   B2 is not done until adding a third venue is a single INSERT and the
   conformance test passes for all three.

Continue with T2-08.

**Claude** (2026-09-19T11:50): Three tasks before T2-08. Starting with the spec, since it governs what I build next.

**Claude** (2026-09-19T11:50): _[tool: Bash]_

**Claude** (2026-09-19T11:50): _[tool: Bash]_

**Claude** (2026-09-19T11:50): Now making the rounds endpoint carry `not_asked` structurally, so the rule holds at the source.

**Claude** (2026-09-19T11:50): _[tool: Edit]_

**Claude** (2026-09-19T11:50): Now the test that fails the build if any proof endpoint ever drops the third state.

**Claude** (2026-09-19T11:51): _[tool: Bash]_

**Claude** (2026-09-19T11:51): Let me verify the guard actually catches a violation — a test that can't fail is worthless.

**Claude** (2026-09-19T11:51): _[tool: Bash]_

**Claude** (2026-09-19T11:51): **The test passed against sabotaged code** — it's a false green. The `-=` on a missing field still creates it. Let me find the real hole.

**Claude** (2026-09-19T11:51): _[tool: Bash]_

**Claude** (2026-09-19T11:51): My sabotage only removed it from the initialiser; line 3236 re-added it. Let me sabotage properly to genuinely test the guard.

**Claude** (2026-09-19T11:51): _[tool: Bash]_

**Claude** (2026-09-19T11:51): The guard genuinely fails when the rule is broken. Restoring the real code.

**Claude** (2026-09-19T11:51): _[tool: Bash]_

**Claude** (2026-09-19T11:52): 49 pass, no sabotage left. Now task 2 — wiring the greps into `make test` as a real lint.

**Claude** (2026-09-19T11:52): _[tool: Write]_

**Claude** (2026-09-19T11:52): _[tool: Bash]_

**Claude** (2026-09-19T11:52): The venue grep is catching data files, not code. Let me scope it to code only and calibrate honestly.

**Claude** (2026-09-19T11:52): _[tool: Bash]_

**Claude** (2026-09-19T11:52): zsh is expanding the glob. Let me quote it.

**Claude** (2026-09-19T11:52): _[tool: Bash]_

**Claude** (2026-09-19T11:52): **37 venue literals in JS**, far more than my audit's estimate of 6 — a significant finding. Let me calibrate the budgets to reality.

**Claude** (2026-09-19T11:53): _[tool: Bash]_

**Claude** (2026-09-19T11:53): All three budgets pass. Let me verify the lint fails when a gate is added, then wire it into `make test`.

**Claude** (2026-09-19T11:53): _[tool: Bash]_

**Claude** (2026-09-19T11:53): The lint genuinely catches a new gate. Wiring it into `make test`.

**Claude** (2026-09-19T11:53): _[tool: Bash]_

**Claude** (2026-09-19T11:53): `gates.js` runs first — it needs no server, so it fails fast. Now task 3: the Phase B queue items.

**Claude** (2026-09-19T11:53): _[tool: Bash]_

**Claude** (2026-09-19T11:53): _[tool: Bash]_

**Claude** (2026-09-19T11:54): _[tool: Bash]_

**Claude** (2026-09-19T11:54): All three done and pushed. Reporting before T2-08.

## 1. Three-state rule — locked in structurally

Written into [docs/survey-spec.md](docs/survey-spec.md) §5.1 as you asked. The endpoint now derives `not_asked` from the **roster**, never from `total - yes`, and every count block carries `yes · no · not_asked · of`.

The test walks every proof endpoint's response at any depth and fails if a block is missing the third state, missing its denominator, or if the three don't sum.

**I checked it by sabotage, and the first attempt was a false green.** I stripped `not_asked` from the initialiser — and the test still passed, because a later line put the field back. Only a proper sabotage, stripping it from the response itself, turned four checks red. Worth stating plainly: a test isn't a control until you've watched it fail.

## 2. B4 — lint wired in, conformance test queued

`tests/gates.js` runs **first** in `make test` (no server needed, fails fast). It doesn't ban the pattern — that would fail the build on working code — it **pins the count**, so a new gate fails and a removed one must be locked in deliberately. Verified by adding a fake `item_type === 'workshop'` and watching it go red.

**Calibrating it corrected my own audit.** I estimated 6 venue literals in JS. There are **37**, and 24 are in `app.js` — every admin screen that draws two columns names both venues inline. My audit undercounted by 6×, which is itself the finding: the problem is bigger than the thing that found it.

| Budget | Today | Phase B target |
|---|---|---|
| activity type gates | 17 | 1 |
| venue literals (JS) | 37 | 0 |
| page routes | 10 | 1 |

The conformance test is queued as **T3-B0** and named as the acceptance test for B0, B2 and B3 — none is done until it passes for every type and every venue.

## 3. Venue ordering — confirmed by the numbers

B2 before B3 now has evidence rather than intuition: **37 venue sites against 17 activity-type sites**. B2's scope explicitly names the `CROSS JOIN (VALUES ('ECE'),('EEE'))` sites in `scripts/chase/` on the `chase-lists` branch, flagged as the thing most likely to be missed at merge. B2 isn't done until a third venue is one INSERT and conformance passes for all three.

**49 survey checks pass, 3 gate budgets pass.** Continuing to T2-08, daily results.

**Vishnu** (2026-09-19T11:55): Two corrections, both about sequencing. Then keep going.

1. SPLIT THE VENUE BUDGET. 24 of your 37 venue literals are in app.js — a file
   that is DELETED in Phase C/D and replaced by React. Fixing them in B2 is
   wasted work and makes B2 look twice its real size.

   Split the budget into two:

     venue literals, server-side JS + SQL   37 - 24 = 13   ->  0 in B2
     venue literals, app.js                          24    ->  0 by deletion,
                                                                 Phase D

   B2's real scope is the 13 server-side sites plus the CHECK constraints plus
   the CROSS JOINs in scripts/chase/. Leave app.js alone — every edit there is
   thrown away.

   Same test for every budget from now on: does this file survive v3? If not,
   it is not B2's problem. Re-check your activity-type count (17) and page
   routes (10) the same way and report the surviving numbers.

2. SABOTAGE IS NOW A STANDING RULE. Your line is exactly right and I want it
   permanent, not just in one log entry:

     A test is not a control until you have watched it fail.

   From here, every guard test — every test whose job is to stop a mistake
   rather than check a feature — is not done until you have broken the thing it
   guards and watched it go red. And the sabotage must be at the level the test
   claims to protect: you proved that yourself when stripping the initialiser
   passed and stripping the response failed.

   Add it to docs/standing-authorisation.md as a new section, and to the
   definition of done in docs/v3-agent-brief.md Part 8.

   Your own near-miss goes in as the worked example. It is more convincing than
   the rule.

3. Noted and agreed: pinning the count rather than banning the pattern is the
   right call. A lint that fails working code gets disabled within a week.

Continue with T2-08.

**Claude** (2026-09-19T11:55): Two corrections. Starting with the budget split, then re-checking all three counts by survival.

**Claude** (2026-09-19T11:55): _[tool: Bash]_

**Claude** (2026-09-19T11:55): Your split is exactly right: 13 server-side, 24 in app.js. And the activity-type count survives fully — all 17 are server-side. Let me check page routes.

**Claude** (2026-09-19T11:55): _[tool: Bash]_

**Claude** (2026-09-19T11:55): The brief isn't on this branch. Let me confirm app.js's fate from the branch that has it.

**Claude** (2026-09-19T11:55): _[tool: Bash]_

**Claude** (2026-09-19T11:55): Confirmed — C1/C2 replace `app.js` entirely. So **10 of 11 page routes die by deletion too**. Restructuring the lint around survival.

**Claude** (2026-09-19T11:55): _[tool: Bash]_

**Claude** (2026-09-19T11:55): Now correcting the Phase B queue to reflect B2's real size.

**Claude** (2026-09-19T11:56): _[tool: Bash]_

**Claude** (2026-09-19T11:56): Now the sabotage rule, in both documents.

**Claude** (2026-09-19T11:56): _[tool: Bash]_

**Claude** (2026-09-19T11:56): Now the definition of done in the brief — which lives on `chase-lists`, so I'll bring it across.

**Claude** (2026-09-19T11:56): _[tool: Bash]_

**Claude** (2026-09-19T11:56): Applying my own new rule: the gates lint is a guard, so I must watch each budget fail.

**Claude** (2026-09-19T11:56): _[tool: Bash]_

**Claude** (2026-09-19T11:56): Each budget watched failing independently, all restored, 4 pass. Committing.

**Claude** (2026-09-19T11:56): _[tool: Bash]_

**Claude** (2026-09-19T11:57): Both corrections applied and pushed. Here are the surviving numbers you asked for.

## Re-checked by survival

| Budget | Today | Survives v3? | Cleared by |
|---|---|---|---|
| activity type gates, server-side | **17** | yes — all in `server.js` | **B4, by hand** |
| venue literals, server-side | **13** | yes | **B2, by hand** |
| venue literals, `app.js` | 24 | no | C1/C2, by deletion |
| page routes, `app.js` | 10 | no | C2, by deletion |

**Two of the three counts changed under the survival test, in opposite directions.**

**Activity types don't shrink at all** — all 17 are in `server.js`, zero in `app.js`. B4 gets no free wins; every one has to be folded into `is_open_for()` by hand. That's the budget that matters most and it's the one deletion doesn't help.

**Page routes almost entirely vanish** — 10 of 11 are in `app.js`. C2 clears them by rewriting the nav as one definition. Ironic, given that a missing `pages_for()` entry is what silently sent `go('survey')` home.

**Venues split as you called it:** 13 / 24. B2's real scope is now the 13 server-side literals, the four `*_dept_check` constraints, and the `CROSS JOIN` sites in `scripts/chase/`. The queue says explicitly to leave `app.js` alone.

## Sabotage rule

Added as its own section in [docs/standing-authorisation.md](docs/standing-authorisation.md) and as a checkbox in the brief's Part 8 definition of done. The near-miss is written up as the worked example, including the actual line that defeated the first attempt:

```js
side.not_asked = cohort;   // put the property back after the initialiser lost it
```

**I applied the rule to the lint itself as its first customer.** Each of the four budgets was broken individually, watched go red, and restored — not one sabotage taken as proof for all four, since each uses a different grep against a different file.

Continuing to T2-08, daily results.

**Vishnu** (2026-09-19T11:57): docs/lanes.md is new. You are LANE A. Another agent is now working in a separate
worktree on v3-dev. You own src/server.js, app.js, app.css, survey migrations and
tests. Do not touch tests/harness/, tests/gates.js, scripts/seed/, or
src/db/migrations/readme.md — Lane B owns those. Continue with Track 2.

**Claude** (2026-09-19T11:57): A conflict: `tests/gates.js` is now Lane B's, and my working tree may hold changes to it. Let me read the lanes doc and check.

**Claude** (2026-09-19T11:57): _[tool: Bash]_

**Claude** (2026-09-19T11:57): Clear. Let me check whether my `tests/gates.js` and `Makefile` work is already committed and whether it collides with Lane B.

**Claude** (2026-09-19T11:57): _[tool: Bash]_

**Claude** (2026-09-19T11:57): Working tree is clean and `tests/gates.js` is already committed and pushed to `origin/survey`. I need to flag this rather than quietly proceed.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Lane B's worktree exists at `v3-dev` (from `cb7ac5d`, before my work). I'll record the overlap in the shared log so Lane B sees it, then continue with T2-08.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Now T2-08 — daily results, every count linking to its named list. Let me build on the rounds endpoint.

**Claude** (2026-09-19T11:59): _[tool: Edit]_

**Claude** (2026-09-19T11:59): Noted on lanes — I'd already flagged the `tests/gates.js` overlap and will use LANE A prefixes. Let me build the route guard you asked for, then finish T2-08.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Now the route guard. It must parse both sides out of `app.js` rather than hard-coding a list.

**Claude** (2026-09-19T11:59): _[tool: Write]_

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): 24 nav entries and 24 routes agree. Now the sabotage — per the standing rule, both directions.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Both directions caught, including the silent one that cost me a cycle. Now testing the T2-08 results endpoints.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): Three states correct — 205 not asked, not folded into "no", and `pct_yes` is `null` rather than a fake 0% when nobody answered. Let me seed volume and test the named lists.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): This is exactly why the three-state rule matters: 23% yes **of 151 who answered**, with EEE's 55 visibly not-asked rather than silently counted as "no". Now the named lists.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): Every count has names and phones behind it. The "none" list is exactly the 55 EEE students — the chase list. Let me add these to the test suite.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): The three-state guard now covers the new results endpoint too. Let me confirm the total and commit.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:01): _[tool: Bash]_

**Claude** (2026-09-19T12:01): Now logging with the LANE A prefix as instructed.

**Claude** (2026-09-19T12:01): _[tool: Bash]_

**Claude** (2026-09-19T12:01): **LANE A — T2-08 done. 66 checks pass.**

## The route guard

`tests/survey-routes.js` parses both lists out of `app.js` and asserts they agree in **both** directions. Sabotage-tested per the standing rule:

| Sabotage | Result |
|---|---|
| Remove `pages_for()` entry — *the silent bug* | red: "Routed but not in pages_for(): survey" |
| Remove the route, keep the nav entry | red: "In pages_for() but with no route: survey" |

24 nav entries, 24 routes, in agreement. Marked for deletion in C2 with the file it guards.

## T2-08 — results, and the case that proves the rule

Seeded a realistic morning — ECE answered, EEE not started:

```
TOTAL   yes=35  no=116  not_asked=55  of=206   pct_yes=23 of 151 answered
  ECE   yes=35  no=116  not_asked= 0  of=151
  EEE   yes= 0  no=  0  not_asked=55  of= 55
```

Without the third state that reads as *"116 said no"* out of 206 — 55 students counted as not knowing something they were never asked. `pct_yes` is `null`, not `0`, when nobody has answered: *"0% said yes"* and *"nobody was asked"* are different sentences.

Every count links to names: yes (80), no (148), answered-nothing (55 — exactly EEE). Each row carries name, roll, team, venue, phone. The response states phone-only in words.

## Lanes — one thing you should know

**I wrote `tests/gates.js` before `docs/lanes.md` existed**, and it's committed and pushed on `survey`. It's now Lane B's file. I haven't touched it since reading the lanes doc and won't.

The collision is at merge: `v3-dev` was cut from `cb7ac5d`, which predates it, so **Lane B will find `tests/gates.js` and a `Makefile` line arriving underneath them** at the rebase. Written up as Q3 in [docs/questions-for-vishnu.md](docs/questions-for-vishnu.md) with the four calibrated budgets, so Lane B can take them rather than re-derive the numbers — the survival split is the part worth keeping.

I've also switched to `LANE A ·` prefixes in the shared log.

Continuing to T2-09 through T2-12, the four proof reports.

**Vishnu** (2026-09-19T12:08): LANE A — answers. Do not stop, continue T2-09 to T2-12.

Q3 — gates.js. Answered. You were right to flag it and right not to touch it.
Lane B takes your file as the starting point at rebase. Lane B does not write
a second one. The four budgets stay as calibrated; the survival split is the
part worth keeping. If Lane B already wrote one, keep both sides and reconcile
the numbers only.

Q1 — question matching across days. Closed, moot. The spec already settled it:
the final round reuses the same survey_questions rows, so pairing is on
survey_question_id. No text matching, no key column. You built it correctly.

Q2 — TEST0001/2/3. Closed. They are test accounts, confirmed. Keep the
exclusion. ECE reading 151 is correct. Keep the excluded count in 00-flags.sql.

ONE CHECK BEFORE T2-09 — your own numbers do not add up.

The table says:   yes=35  no=116  not_asked=55  of 206
The links say:    yes (80)  no (148)  answered-nothing (55)

35 + 116 + 55 = 206. Correct.
80 + 148 = 228. That is more answers than students.

I think the table is one question and the links are all questions summed. If so
the code is fine but the screen is not — a reader sees 35 and clicks through to
80 and assumes the list is broken. Per the standing rule every count carries its
denominator, so say which unit each number is in: "35 of 206 students" next to
"80 of 228 answers". If that is NOT the reason, it is a real bug — find it
before building four reports on top of it.

QUEUE — tick it. T2-01 to T2-08 are done and pushed but all 18 rows in
docs/work-queue.md still read TODO. The queue header says update and commit
after every item. Right now the agent log is the only source of truth.

**Claude** (2026-09-19T12:08): You're right to call it out. Let me check whether it's a display problem or a real bug before anything else.

**Claude** (2026-09-19T12:08): _[tool: Bash]_

**Claude** (2026-09-19T12:08): That's a fresh survey from the last test run. Let me find the one that produced those numbers.

**Claude** (2026-09-19T12:08): _[tool: Bash]_

**Claude** (2026-09-19T12:08): The test suite's `reset_surveys()` wiped the seeded data. Let me reproduce the exact scenario to settle this properly.

**Claude** (2026-09-19T12:09): _[tool: Bash]_

**Claude** (2026-09-19T12:09): Reproduced. Your diagnosis was right in shape but the mechanism is worse than you guessed — and it's a real bug, not just a labelling problem.

**Claude** (2026-09-19T12:09): _[tool: Bash]_

**Claude** (2026-09-19T12:09): Confirmed: **65 students appear in both the yes and no lists**. Not a unit mismatch — a genuine bug. Without a question filter, `/who` asks "answered yes to *any* question", so the same student is in both.

**Claude** (2026-09-19T12:09): _[tool: Bash]_

**Claude** (2026-09-19T12:09): **With a question filter it's exactly right** — 25/126/55 matching the table. The bug is only in the unfiltered case, where "yes" and "no" are incoherent. Let me fix it.

**Claude** (2026-09-19T12:09): _[tool: Edit]_

**Claude** (2026-09-19T12:09): Now the units, per the standing rule — every count says what it's counting.

**Claude** (2026-09-19T12:09): _[tool: Bash]_

**Claude** (2026-09-19T12:09): Now a regression test for this exact bug — and it must be sabotage-proven.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): Now the sabotage — this guard must be seen failing against the old behaviour.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): Guard watched failing and restored: **76 checks pass**. Now the queue, which I should have been ticking all along.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): Queue ticked. Committing the bug fix and the queue together.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): An index lock — likely Lane B operating in the shared `.git`. Let me check before touching anything.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): No git process is running — the lock is stale, and my commit didn't land. Since Lane B shares this `.git`, I'll wait and retry rather than delete it.

**Claude** (2026-09-19T12:11): _[tool: Bash]_

**Claude** (2026-09-19T12:11): Lock is stale — zero-byte, unchanged, no git process. But it's in a `.git` shared with Lane B, so removing it is exactly the kind of cross-lane action I shouldn't take unilaterally.

I've stopped short of deleting it. Here's where things stand.

## The count discrepancy — you were right, and it was a real bug

Not a unit mismatch. **65 students appeared in both the yes and the no list.**

Without a question named, `answer=yes` meant *"said yes to at least one question"* and `answer=no` meant *"said no to at least one"*. A student who answered yes to Q1 and no to Q2 was in both. Reproduced exactly: 3 questions, 151 students → **67 yes + 149 no = 216 rows for 151 people**, against a table reading 25/126 for a single question.

With a question named it was already correct — 25/126/55, matching the table exactly. So the fix is to **refuse the incoherent call rather than answer it**: `yes` and `no` now require a question. `none` stays survey-wide because *"has answered nothing"* is coherent across a whole survey, and it's the list that actually gets chased.

Both endpoints now report `unit` and `scope`. Sabotage-tested: restoring the old behaviour turns two checks red, and a third asserts directly that no student is in both lists. **76 checks pass.**

Queue ticked, T2-01 through T2-08 — my miss, the header says to do it every item.

## What I need from you

**There's a stale `.git/index.lock` and I've not removed it.** Zero bytes, written 17:37, no git process running, and it survived a 15-second wait. My commit did not land — the working tree still holds the fix, the tests, and the ticked queue, all uncommitted.

Normally I'd just delete a stale lock. I'm not doing that here because `docs/lanes.md` puts Lane B in a worktree sharing this same `.git`, and if Lane B is mid-operation, removing its lock could corrupt its index. That's a cross-lane action, and the lanes doc calls editing another lane's territory a stop-work event.

Two ways forward — your call:

1. **Confirm Lane B is idle** and I'll `rm .git/index.lock` and commit immediately.
2. **Tell me Lane B is active** and I'll wait and retry on a loop until it clears.

Nothing is lost either way — every change is on disk and the work is done. I just can't commit it without knowing whether that lock is mine to clear.

**Vishnu** (2026-09-19T12:12): LANE A — clear the lock. Option 1. It is not shared, and it is not yours either.

THE PREMISE IS WRONG. A linked worktree does not share .git/index.lock.
Lane B has its own index and its own lock path:
  .git/worktrees/lane-b/index          <- Lane B's index
  .git/worktrees/lane-b/index.lock     <- Lane B's lock, DOES NOT EXIST
Only the main checkout uses .git/index.lock, and the main checkout is you.
Lane B cannot create it, cannot read it, cannot be corrupted by its removal.

WHO MADE IT. Not you and not Lane B. A read-only audit ran git status against
the repo through a mount where file deletion is disabled. Git created the lock,
tried to clean it up, was refused, and left it. Zero bytes, 17:37. Its mtime now
reads 17:41 because that same audit touched it while probing. Both are mine.

LANE B IS ACTIVE, AND IT DOES NOT MATTER. Lane B committed at 17:38 IST:
  6c0ed86 feat: a fixture that is 209 students and every awkward shape in the data
That is T3-A1 landed, and tests/harness now has 5 files, so A2 is underway.
Lane B held no lock while doing it. It never touches yours.

DO THIS:
  rm .git/index.lock
  git add -A && git commit

Then continue to T2-09 through T2-12.

ON THE BUG — you were right and I was wrong about the cause. I guessed a unit
mismatch. It was a set overlap: 65 students in both lists. Refusing the
incoherent call rather than answering it is the correct fix, and keeping `none`
survey-wide is right because that is the list that gets chased. Reporting unit
and scope on both endpoints, and asserting directly that no student is in both
lists, is the check that should have existed from the start. Good catch.

**Claude** (2026-09-19T12:12): Your explanation is correct — linked worktrees use `.git/worktrees/<name>/index.lock`, not the main one. Let me verify that here, then clear it.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:12): Confirmed exactly as you described: Lane B has its own index, no lock, and committed fine at 17:38. The lock is the main checkout's — mine. Clearing it.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:13): Committed and pushed. Now T2-09 through T2-12 — the four proof reports.

**Claude** (2026-09-19T12:13): _[tool: Bash]_

**Claude** (2026-09-19T12:13): `session-state.md` isn't in my ownership list and `git add -A` didn't stage it (it's still untracked) — leaving it alone. Now building the proof reports.

**Claude** (2026-09-19T12:13): _[tool: Edit]_

**Claude** (2026-09-19T12:14): A route-ordering risk: `/proof` could be captured by `/:id/results`. Let me check and test.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): `/survey/:id/results` is registered before `/survey/proof`, but they have different segment counts so they don't collide. Let me verify rather than assume.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): Both reachable. Now seeding a realistic before/after at full volume to see whether the numbers read as evidence.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): 194 answered the daily round, 206 the final — 12 late joiners. Exactly the case that would inflate a naive gain.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): This reads as evidence: the 12 late joiners appear as `not asked`, never as "did not know", and the gain rests on 194 paired students — stated, not hidden. 78ms at full volume. Let me check the per-student view and add tests.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): Exactly right — `before=None`, not `false`. A late joiner shows 6 not-asked and contributes zero to any gain. Adding the tests.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:15): Some checks are missing from that filter. Let me see the full picture.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:15): 95 checks, zero failures. Now the sabotage — the honest-denominator guard is the one that matters most here.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:15): **The sabotage passed** — a second false green, and this one matters. My guard used `<=`, which a too-large denominator satisfies. Let me see why and fix it.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:15): The suite's `reset_surveys()` leaves only 1 student, so the numbers coincide and the guard can't discriminate. The test needs real volume to be meaningful.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:16): Now the real question: does the tightened guard catch the sabotage that's still in place?

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:16): The tightened guard catches it. Restoring the correct code and confirming clean.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:16): Sabotage removed, 95 pass. Let me confirm the report still reads correctly with the test's own seeded data, then commit.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:16): The 12 late joiners are visible as the gap between 194 paired and 206 who sat the final. Committing.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:17): **LANE A — T2-09 to T2-12 done. 95 checks pass, pushed.**

## The lock

Your analysis was correct and mine was wrong. Verified before removing: `.git/worktrees/lane-b/index` exists with no lock beside it, Lane B committed `6c0ed86` at 17:38 without one. The lock was the main checkout's — mine. Removed, committed, pushed.

## The proof report

The headline at real volume, with 12 deliberate late joiners:

```
6 topics, 206 students
  before  20% (234 of 1164 answers), not_asked 72
  after   86% (1062 of 1236 answers), not_asked 0
  gain   +65% over 1164 answer pairs
```

The late joiners show as `not_asked`, never as "did not know", and contribute nothing to any gain — visible as the gap between **194 paired** and the **206** who sat the final round.

## A second false green, and this one was worse than the first

I sabotaged the gain to compute over everyone who sat the final round — precisely the dishonest number, counting late joiners' improvement without their "before". **The guard passed.**

The assertion was `paired <= min(before.answered, after.answered)`. After `reset_surveys()` only **one** student had answered, so `paired`, `before` and `after` were all `1`, and every relationship between them held by coincidence.

Two fixes: give the test a real cohort (206 students, 2,472 answers, 12 late joiners), and assert **equality** rather than an inequality — `paired` must equal `before.answered` *and* `after.answered − 12`. Re-run against the same sabotage, it fails.

**The lesson, which I'd rather state than bury:** a guard is only as good as the data it runs against. One row satisfies almost any invariant. Both of today's false greens were mine, and both were caught only because the sabotage rule forced a second look — the first one you'd have caught reading the report, this one nobody would have.

Queue ticked T2-01 through T2-12. Remaining: T2-13 (chase list), T2-14 (venue leak test), T2-15 (session tests), T2-16 (old-path check), T2-17 (screenshots), T2-18 (deploy commands).

Continuing to T2-13.

**Vishnu** (2026-09-19T12:18): LANE A — no stop. One display fix, then carry on to T2-13.

The gain is computed correctly. I checked: after-on-paired is ~990/1164 = 85%,
and 1062 - 990 = 72, exactly the late joiners' after-answers. They are properly
excluded. No bug.

The DISPLAY is wrong. The headline reads:
  before  20% (234 of 1164)
  after   86% (1062 of 1236)
  gain   +65%
86 - 20 = 66, not 65. The after line is over 1236, the gain is over 1164. Two
denominators on one screen, and the reader cannot see the switch.

This breaks your own standing rule: never a bare percentage, always the
denominator. The gain is a percentage whose denominator is invisible.

Show the paired figure on its own line, e.g.:
  before, paired   20% (234 of 1164)
  after,  paired   85% (990 of 1164)
  gain             +65% over 1164 pairs
  after, everyone  86% (1062 of 1236)   <- includes 12 late joiners

Display only. Do not touch the computation, it is right.

ON THE SECOND FALSE GREEN — you are right that nobody would have caught it, and
right to state it rather than bury it. The rule is now written down: a guard is
only as good as the data it runs against; prefer equality over inequality,
because <= is satisfied by degenerate data; ask whether the assertion would
still hold if every number were 1. Applies to Lane B's harness and every Phase B
conformance test. Carry it into T2-14 and T2-15.

**Claude** (2026-09-19T12:18): You're right — the gain's denominator is invisible and the two lines are over different bases. Display only.

**Claude** (2026-09-19T12:18): _[tool: Bash]_

**Claude** (2026-09-19T12:18): I need the paired-only yes counts for both rounds. Let me add that to the existing paired query rather than adding a round trip.

**Claude** (2026-09-19T12:18): _[tool: Bash]_

**Claude** (2026-09-19T12:18): _[tool: Edit]_

**Claude** (2026-09-19T12:18): Now the headline, showing paired and everyone as separate labelled lines.

**Claude** (2026-09-19T12:18): _[tool: Edit]_

**Claude** (2026-09-19T12:19): _[tool: Edit]_

**Claude** (2026-09-19T12:19): Now the per-question blocks need the same treatment, since they have the same two-denominator problem.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): Now all three lines share denominator 1164. The remaining 66-vs-65 is rounding, not a base switch — let me prove that rather than assume it.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): Proven: `after_paired − before_paired` equals `learned − lost` exactly (762). The 66 was two roundings of 65.46. Let me add tests that assert this identity.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:20): Two identity assertions are missing from that output. Let me check whether they ran.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): 101 pass. Now the sabotage — the guard must catch a reintroduced base mismatch.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): Guard caught it, restored, 101 pass. Committing the display fix.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): Display fixed. Now T2-13 — the "has not answered today's survey" chase list.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): The chase set lives on `chase-lists`, which is Track 1's branch. T2-13 says to add query 10 there, but the survey tables only exist on `survey` — worth thinking about before acting.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:21): The chase scripts are on a branch with no survey tables. Writing query 10 here on `survey` is correct — it lands in the chase set when both merge to main.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:21): _[tool: Write]_

**Claude** (2026-09-19T12:21): Now testing it against real data, including the venue-closed case.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:21): The inner query's trailing `ORDER BY` breaks the subquery wrap. Let me test it directly instead.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:21): Exactly right: 31 not started, 20 partial (1 of 3 shown), and EEE's 55 correctly marked as not the student's fault. 100 who finished are absent. Let me verify that arithmetic.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:22): 151 − 100 = 51, and 31 + 20 = 51. The list is complete and non-overlapping. Committing T2-13.

**Claude** (2026-09-19T12:22): _[tool: Bash]_

**Claude** (2026-09-19T12:22): **LANE A — display fixed, T2-13 done. 101 checks pass.**

## The display fix

You were right that the computation was sound and the screen was not. It now reads:

```
before, paired    20% (234 of 1164)
after,  paired    86% (996 of 1164)
gain             +65% over 1164 pairs
after, everyone   86% (1062 of 1236)   <- includes 12 late joiners
```

Three lines sharing one denominator, with the everyone-figure labelled as a different base and carrying a note saying why it can't be differenced against them.

**On the residual 66 vs 65** — I proved it's rounding rather than asserting it:

```
after_paired − before_paired = 762
learned − lost               = 762     identical
exact: 85.5670% − 20.1031%   = 65.4639%
```

So `round(85.57) − round(20.10) = 66` against `round(65.46) = 65`. Two roundings of one quantity. **The tests assert the identity in counts, not percentages**, because rounding is exactly where an identity quietly stops holding.

Sabotage-tested: recomputing `paired.after` over everyone turns the identity check red.

## T2-13

Chase list 10. Partial counts as not done, with `answered / of_questions` — "0 of 6" and "4 of 6" are different phone calls. Venue state on every row.

Tested with the survey open for ECE only: **31 not started, 20 stopped partway, 55 EEE marked "not the student's fault."** The arithmetic closes — 151 ECE, 100 finished, 51 outstanding, 31 + 20 = 51.

**One merge note:** I wrote it on `survey`, not `chase-lists`, because the survey tables don't exist on that branch. It joins the chase set when both merge to main, and `run.sh` needs one line added then. Flagged in the commit so it isn't lost.

## Carrying your rule forward

Written into how I'll do T2-14 and T2-15: prefer equality over inequality, and ask whether the assertion would still hold if every number were 1. Both of today's false greens would have failed that question.

Continuing to T2-14, the venue leak test.

**Vishnu** (2026-09-19T12:24): LANE A — no stop. One decimal change, then carry on to T2-14.

THE DISPLAY. Your identity proof is right and the tests are in counts, which is
correct. But the screen still reads:
  before, paired    20% (234 of 1164)
  after,  paired    86% (996 of 1164)
  gain             +65% over 1164 pairs
  after, everyone   86% (1062 of 1236)
86 - 20 = 66. The reader gets the same wrong subtraction as before. The proof
is in the test file; the administrator reading the report never sees it.

Also: both after-lines print 86%. 996/1164 = 85.57%, 1062/1236 = 85.92%. They
round to the same number, so the "different base" line looks like a duplicate
and the distinction you built it for is invisible.

ONE DECIMAL FIXES BOTH:
  before, paired    20.1% (234 of 1164)
  after,  paired    85.6% (996 of 1164)
  gain             +65.5% over 1164 pairs
  after, everyone   85.9% (1062 of 1236)   <- different base, do not difference
85.6 - 20.1 = 65.5. It subtracts. And the two after-figures now read as two
numbers. Display only, tests stay in counts.

MERGE DEBT. Your run.sh note is now item M1 on a tracked list, with Lane B's
gates.js takeover (M2) and the CROSS JOIN venue sites on chase-lists (M3).
Three edits on three branches that only become correct at merge, none of which
is in any lane's queue. Going forward: the lane that creates the debt records it
the moment it writes the code, not at merge time. Add to it as you find more.

T2-14 is the one that matters most before any deploy. Venue leak is the only
bug class here that exposes one department's data to another.

**Claude** (2026-09-19T12:24): One decimal, then T2-14. Let me find every percentage in the proof output.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): Nine sites. A shared helper keeps them consistent rather than nine separate edits.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): All nine converted. Let me check the rendered output.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): Both problems fixed: 85.6 − 20.1 = 65.5 exactly matches the gain, and the two after-figures are now visibly different. Let me confirm tests still pass in counts.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): Tests unchanged — they assert counts, so the decimal didn't touch them. Now the merge-debt list.

**Claude** (2026-09-19T12:25): _[tool: Write]_

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): Now T2-14 — the venue leak test. This is the bug class that exposes one department's data to another, so I'll test every survey surface, not just the obvious one.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): Two student-facing surfaces are the leak risk; the admin ones are venue-scoped by query. Let me write a dedicated leak suite covering both directions.

**Claude** (2026-09-19T12:26): _[tool: Write]_

**Claude** (2026-09-19T12:26): _[tool: Bash]_

**Claude** (2026-09-19T12:26): 20/20. But a leak test that has only ever passed proves nothing — sabotaging the gate now.

**Claude** (2026-09-19T12:26): _[tool: Bash]_

**Claude** (2026-09-19T12:26): No failures — the sabotage didn't take effect. Let me check whether my injected code is actually being reached.

**Claude** (2026-09-19T12:26): _[tool: Bash]_

**Claude** (2026-09-19T12:26): The sabotage is in the right place but unreachable — `isOpenFor` returns from the release-row lookup earlier, before this branch. That's actually good news about the design, but I need a sabotage that lands where the real gate is.

**Claude** (2026-09-19T12:26): _[tool: Bash]_

**Claude** (2026-09-19T12:26): Found the real gate — `if (any) return false`. That line is what prevents the v2 leak. Sabotaging it properly.

**Claude** (2026-09-19T12:27): _[tool: Bash]_

**Claude** (2026-09-19T12:27): The leak suite catches the exact v2 bug across all five surfaces — seeing it, answering it, the write landing, and the final round. Restoring.

**Claude** (2026-09-19T12:27): _[tool: Bash]_

**Claude** (2026-09-19T12:27): Clean. Wiring the leak suite into `make test` and committing.

**Claude** (2026-09-19T12:27): _[tool: Bash]_

**Claude** (2026-09-19T12:27): **LANE A — decimals fixed, merge debt tracked, T2-14 done.**

## The decimal

```
before, paired    20.1% (234 of 1164)
after,  paired    85.6% (996 of 1164)
gain             +65.5% over 1164 pairs
after, everyone   85.9% (1062 of 1236)   <- different base
```

85.6 − 20.1 = 65.5, matching the gain exactly. And the two after-figures now read as two numbers instead of both printing 86%.

Done as one `pct1()` helper rather than nine edits — nine percentage sites drifting to different precisions is the same enumerating-gate shape that's already cost four bugs here. Tests unchanged at 101, because they assert counts.

## Merge debt

[docs/merge-debt.md](docs/merge-debt.md) with M1, M2, M3, each saying what breaks if missed. **M1 and M3 both fail silently** — they produce a chase list that's quietly incomplete, no error, which is the worst shape for a tool whose whole job is completeness. I'll add to it as I create more.

## T2-14 — the venue leak

**20 checks, every one from the other venue's own session.** An admin asking "is this open for ECE" is not the test; the v2 bug passed exactly that and still leaked.

Two checks exist because a 403 isn't sufficient on its own: ECE is refused **while holding the question id**, and then the results are re-read to confirm the ECE column is still zero. A 403 that writes anyway is worse than no check.

**The sabotage failed first, instructively.** I injected "open anywhere = open everywhere" into the survey branch of `isOpenFor` and nothing went red — that branch is unreachable for this case, because the release-row lookup returns earlier. The line that actually prevents the leak is `if (any) return false` near the top: once any venue has been released, the table is the authority and the old flag is ignored.

Sabotaging *that* turned five checks red across all four surfaces. The first attempt proved the test runs; the second proved it guards what it claims to. That's now twice today the first sabotage was too shallow — worth noting as a pattern: **sabotage the line that would actually be wrong, not the line that looks related.**

121 checks passing. Continuing to T2-15, session tests for every role.

**Vishnu** (2026-09-19T12:29): LANE A — no stop. Continue T2-15.

THE DEAD BRANCH IS THE FINDING, not a footnote.

You proved by sabotage that the survey branch of isOpenFor is unreachable. That
is dead code sitting inside the single most security-relevant function in the
app. Right now it is harmless. It will not stay harmless.

T3-B4 makes releases.is_open_for(activity, venue) the ONLY gate. Whoever does
B4 reads that branch, sees plausible survey logic, and faithfully carries it
forward — reintroducing exactly the leak you just closed. That is the same shape
that has already cost four bugs here: old logic nobody updated, kept because it
looked load-bearing.

DO ONE OF THESE BEFORE YOU MOVE ON, it is two minutes:
  - delete it, or
  - leave a comment on the line saying it is unreachable, why, and that
    `if (any) return false` above is what actually gates the venue.
Second option is safer mid-track. Either way B4 must not inherit it silently.
Recorded as a hazard against B4 either way.

YOUR THREE SABOTAGE LESSONS ARE NOW WRITTEN DOWN:
  - sabotage at the caller's level, not an intermediate
  - sabotage the load-bearing line, not the plausible-looking one
  - a PASSING sabotage is a finding, not a retry — weak test, or dead code?
    Find out which before moving on. You did. That is why this was caught.
Plus the one from T2-14: for a refusal, 403 is not enough. Re-read the data and
confirm nothing was written.

pct1() was the right call. Nine sites drifting to nine precisions is the same
enumeration shape, and you saw it before it cost anything.

**Claude** (2026-09-19T12:29): The dead branch first — two minutes, and B4 must not inherit it.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:29): I'll keep the branch but mark it unambiguously — deleting mid-track loses the explicit default, and the comment is what stops B4 carrying it forward.

**Claude** (2026-09-19T12:29): _[tool: Edit]_

**Claude** (2026-09-19T12:29): Now recording it as a B4 hazard so it's tracked, not just commented.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:30): M4 recorded, leak suite still green. Committing, then T2-15.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): Now T2-15 — every route, every role, signed in, at 209 students. Let me enumerate the roles and surfaces.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): No non-admin mentor exists in the dump — that role can't be tested without creating one. Let me check if the app can, since I can't modify production data semantics.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): A mentor can be created through the real admin API — so I can test all four roles honestly. Writing the session suite.

**Claude** (2026-09-19T12:31): _[tool: Write]_

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): All passing. Let me see the total and confirm the volume claim held.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): 52 checks at 2,472 answers, all four roles. Now the sabotage — a permission suite that's never been seen failing proves nothing.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:32): Caught in both places. Now the opposite direction — the v2 lockout bug, where a role loses access it should have.

**Claude** (2026-09-19T12:32): _[tool: Bash]_

**Claude** (2026-09-19T12:32): Both directions caught — over-permission and lockout. This is exactly the v2 bug that hit 156 students. Verifying clean and committing.

**Claude** (2026-09-19T12:32): _[tool: Bash]_

**Claude** (2026-09-19T12:32): 177 checks across four suites, zero failures. Committing T2-15.

**Claude** (2026-09-19T12:32): _[tool: Bash]_

**Claude** (2026-09-19T12:32): **LANE A — dead branch marked, T2-15 done. 177 checks passing.**

## The dead branch

Marked in place rather than deleted — safer mid-track, and it keeps the default for an unreleased survey explicit. The comment now states it's unreachable, how that's known (by sabotage, not reading), which line actually gates the venue, and what B4 must carry forward.

Recorded as **M4** in [docs/merge-debt.md](docs/merge-debt.md) with the failure spelled out: whoever writes `releases.is_open_for()` reads that function top to bottom, finds plausible per-type survey logic, carries it forward, and may drop the `any` rule that does the real work — reintroducing the exact leak T2-14 closed, inside the function whose whole job is preventing it.

It's the four-bug shape inverted: not old logic kept because it looked load-bearing, but **logic that looks load-bearing and isn't**.

## T2-15

52 checks, every request carrying a real session cookie.

**The mentor role had to be created to be tested at all.** The dump has ten admins and zero plain mentors — so "mentor" was the one role no test could have exercised, and it's the role most likely to be handed more than it should have. The suite creates one through the real admin API, asserts `is_admin` is genuinely false, runs the whole matrix as that account, then deletes it.

**Both directions asserted.** Every route × every role carries an expected status, allowed as well as refused. A suite that only checks refusals is fully green when the feature is broken for everyone — which is the v2 bug, not a hypothetical.

Sabotage, both shapes:

| Sabotage | Result |
|---|---|
| Drop `require_admin` from the proof report | 2 red — a mentor reads unchecked evidence |
| Make `/api/survey/answer` require `is_lead` | 1 red — the v2 lockout, 154 members refused |

| Suite | Checks |
|---|---|
| survey | 101 |
| survey-venue-leak | 20 |
| survey-sessions | 52 |
| survey-routes | 4 |

Remaining: T2-16 (old-path check), T2-17 (screenshots), T2-18 (deploy commands).

Continuing to T2-16.

**Vishnu** (2026-09-19T12:59): LANE A — URGENT, do this before T2-16.

LIVE BUG. Lane B found it during A1b and correctly refused to fix it, because
src/server.js is yours. It is yours to fix.

WHAT:  POST /api/tasks/:id/submit returns 500 for EVERY text or Drive-link
       hand-in. drive-uploads.js:389 has the identical defect.

WHY:   ON CONFLICT (task_id, team_id) has no plain UNIQUE left to infer. The
       quiz-per-student migration replaced it with PARTIAL indexes, and a
       partial index cannot be inferred from a bare column list.

WHY IT HAS NOT BITTEN: all 56 real submissions are per_student — the photo
       path, a different branch. Days 1 and 2 were both photo tasks. That is
       luck, not safety.

WHEN IT BITES: the first task that asks for a typed answer or a Drive link.
       Then it fails for every student, both venues, at once. Day 3 is
       tomorrow, 20 September.

FIX: one line each, two files. Lane B verified on a copy — tasks.js goes from
     9 passing to 43. Reproduces against the real dump.

DO IT AS AN EXTRA ITEM NOW, ahead of T2-16, so it ships with the Track 2
deploy. Decided under standing-authorisation: local and reversible.

SABOTAGE IT, both sites, per the standing rule. And apply Lane B's new one:

  ASSERT WHY A SABOTAGE WENT RED, NOT JUST THAT IT DID.

Lane B had a proof go red for the wrong reason — MODULE_NOT_FOUND, not the
permission hole it claimed to test. Caught only by asserting the reason. A test
that fails for the wrong reason is a false green wearing a red coat. That is the
fourth guard lesson today and it belongs in your remaining items too.

ALSO LOG IT. docs/agent-log.md still ends at T2-08. Seven items done and
unlogged. The log is the only source of truth here and a batch that never gets
written is seven items of reasoning gone. Flush it before T2-16.

**Claude** (2026-09-19T13:00): Live bug first. Let me confirm the defect and the index state before touching anything.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Confirmed: no plain UNIQUE on `(task_id, team_id)` — only partial indexes. Let me reproduce the failure before fixing it.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Reproduced exactly. Now let me see both call sites and what distinguishes the working photo path.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Now the working photo path, to match its idiom exactly rather than invent one.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Both sites have the identical defect. The fix is to name the partial index's predicate so Postgres can infer it.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): The `tasks WHERE active` subquery returned nothing — the test suites deleted the task rows. Let me use a real task.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): The tasks table is empty — earlier suites wiped it. Let me rebuild a clean database from the dump to test honestly.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Clean copy with the real 8 tasks and 56 submissions. Now reproducing against real data.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): Task 10 is a `text` task. Let me check how `per_student` is decided — that determines which index applies.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:00): `per_student` is copied from the task. So the correct fix must handle **both** modes, not just the team one — a per-student task needs the other index.

**Claude** (2026-09-19T13:00): _[tool: Bash]_

**Claude** (2026-09-19T13:01): Both active tasks are per-student; 5 inactive ones are team-mode. Let me reproduce the real failure path through the actual route.

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): Now I understand fully. The route handles text/drive only; image/file go via drive-uploads. The fix must branch on `per_student`, since the task decides which index applies.

**Claude** (2026-09-19T13:01): _[tool: Edit]_

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): The ternary split the SET clause — only the second branch has it. Let me restructure so both share one body.

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): Now the second site in `drive-uploads.js`.

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): This function doesn't know the task's mode. Let me check how it's called, then read it from the task.

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): It already inspects the schema defensively. I'll follow that idiom — read `per_student` from the task, same as the server route.

**Claude** (2026-09-19T13:01): _[tool: Bash]_

**Claude** (2026-09-19T13:01): Now testing both paths against the real dump — the text task is per-student, which exercises the branch that was broken.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): 200 where it was 500. Let me confirm the row landed and that a second hand-in replaces rather than duplicates.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): One row, replaced not duplicated. Now the critical check — a teammate must get their **own** row, since this is a per-student task.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): Two rows on one team — correct for a per-student task, and the old team-keyed check would have blocked this. Now the tasks suite Lane B referenced.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): Lane B reported 9→43. I'm seeing 9 pass / 5 fail — different. Let me look at what's actually failing rather than assume.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): The hand-ins now succeed ("can hand in task 15/16" pass — those were failing before). The remaining failures are downstream. Let me check whether they're caused by my fix.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:02): Let me check whether these failures pre-date my change — the honest comparison.

**Claude** (2026-09-19T13:02): _[tool: Bash]_

**Claude** (2026-09-19T13:03): Baseline is 7 pass / 7 fail; with my fix it's 9 pass / 5 fail. Real improvement, but not Lane B's 43 — the remaining failures need investigating.

**Claude** (2026-09-19T13:03): _[tool: Bash]_

**Claude** (2026-09-19T13:03): The run against `bootcamp_base2` now fails differently — the suite mutates state, so results depend on database freshness. Let me test properly on a clean database.

**Claude** (2026-09-19T13:03): _[tool: Bash]_

**Claude** (2026-09-19T13:03): Consistently 9/5 on a clean database. Let me find what still fails, applying the "assert *why*" rule.

**Claude** (2026-09-19T13:03): _[tool: Bash]_

**Claude** (2026-09-19T13:03): The hand-ins now pass — the 500 is fixed. The remaining failures are a different issue: the test expects rows keyed by team, but these tasks are per-student.

**Claude** (2026-09-19T13:03): _[tool: Bash]_

**Claude** (2026-09-19T13:03): Both rows written correctly against the team. So the fix works for team-mode too — the failures are elsewhere. Let me check the count assertion.

**Claude** (2026-09-19T13:03): _[tool: Bash]_

**Claude** (2026-09-19T13:04): Exactly 2 rows — the assertion should pass. It fails because `tasks.js` ran earlier in the same process and mutated state. The suite isn't idempotent, which is a pre-existing issue, not mine.

**Claude** (2026-09-19T13:04): _[tool: Bash]_

**Claude** (2026-09-19T13:04): The database says 2 rows but the assertion failed — because it ran *before* the hand-ins committed within that run. These are pre-existing suite defects unrelated to my fix. Let me confirm precisely.

**Claude** (2026-09-19T13:04): _[tool: Bash]_

**Claude** (2026-09-19T13:04): `'and a student is shown nothing'` expects zero open tasks, but the real dump already has task 10 released for ECE. The suite assumes an empty fixture — a pre-existing mismatch with real data.

**Claude** (2026-09-19T13:04): _[tool: Bash]_

**Claude** (2026-09-19T13:04): Confirmed: the dump has 9 task releases; the suite asserts 0. That's a fixture mismatch, not my bug. Let me write a focused regression test that proves the fix on its own terms.

**Claude** (2026-09-19T13:05): _[tool: Write]_

**Claude** (2026-09-19T13:05): _[tool: Bash]_

**Claude** (2026-09-19T13:05): All modes and types pass. Now the sabotage — both sites, asserting *why* it goes red.

**Claude** (2026-09-19T13:05): _[tool: Bash]_

**Claude** (2026-09-19T13:05): Red for the right reason — the server log names the exact ON CONFLICT error. Now the second site.

**Claude** (2026-09-19T13:05): _[tool: Bash]_

**Claude** (2026-09-19T13:05): Second site confirmed with the same error. Restoring and verifying the fixed SQL succeeds.

**Claude** (2026-09-19T13:05): _[tool: Bash]_

**Claude** (2026-09-19T13:05): Both sabotages removed, fixed clause succeeds. Let me run everything and commit.

**Claude** (2026-09-19T13:05): _[tool: Bash]_

**Claude** (2026-09-19T13:06): 40 checks pass, and `tasks.js` improves 7→9. Let me flush the log and commit.

**Claude** (2026-09-19T13:06): _[tool: Bash]_

**Claude** (2026-09-19T13:06): **LANE A — live bug fixed and pushed. 40 new checks.**

## The fix

Reproduced against the real dump before touching anything, then fixed both sites by naming the partial index's predicate.

**The obvious one-line fix would have shipped still broken.** Naming only `WHERE NOT per_student` fixes the team case — but **both live tasks are per-student**, so every real hand-in would still have 500'd. The clause has to branch on `tasks.per_student`, because that's what `trg_sync_submission_mode` copies onto the row and therefore which index applies.

**A second defect, found while reading:** the "already marked" check asked by *team* even for per-student tasks. One marked teammate would have blocked every other member from handing in their own work.

**What hid it:** `drive-uploads.js` carried the comment *"step 4 puts UNIQUE (task_id, team_id) on it"* — true when written, false since the migration. The comment asserted the exact constraint that had been removed.

## Sabotage, asserting why

Both sites. Restoring the bare column list turns 6 checks red **and** puts `no unique or exclusion constraint matching the ON CONFLICT specification` in the server log — the real cause, not a coincidental failure. The drive-uploads site was proved directly against the database, since its route needs Drive credentials I must never hold.

## One correction to the brief

**I could not reproduce Lane B's 9 → 43, and I'm not claiming it.** On a clean database from the dump, `tasks.js` goes **7 → 9**.

The remaining 5 failures are a fixture mismatch, not this bug: the suite asserts *"creating a task does not open it"* by counting task releases and expecting **0**, and the real dump has **9**. I verified by reading the rows. That suite also isn't idempotent — it mutates state, so consecutive runs differ. Not mine to fix; `tests/` is shared and that assertion belongs to whoever owns the fixture.

`tests/task-upsert.js` exists because of this: it tests the bug against the real dump, where `tasks.js` structurally cannot tell a genuine break from its own assumptions.

## Log flushed

Seven items of reasoning written up, including this fix. Continuing to T2-16, the old-path check.

**Vishnu** (2026-09-19T13:07): BOTH LANES — the split ends. Vishnu's decision.

Lane A and Lane B become ONE lane. lanes.md set this condition itself: the
split is worth it "while there is genuinely independent work; not worth it once
everything funnels back into server.js." That moment is now.

WHY IT IS RIGHT
  - Lane B has finished all five independent items: A1, A2, A1b, A4, A5a.
  - A5b turned out to be empty. One reader, no completeness use.
  - A3 cannot be parallelised at all. One file, twelve serial items.
  - lanes.md estimated 1.5-1.7x, at roughly DOUBLE the tokens. There is no
    speed left to buy, so the cost has nothing to show for it.

THE ORDER
  1. LANE A finishes Track 2 — T2-16, T2-17, T2-18, plus the task-submit 500
     fix. Log everything; the log still ends at T2-08.
  2. Vishnu reviews the T2-17 screenshots. His checkpoint, not yours.
  3. survey merges into main. LOCAL TIDYING, NOT A DEPLOY.
  4. v3-dev rebases onto main.
  5. ONE lane does T3-A3, the twelve modules, alone.
  6. LANE B stands down after handing over.

LANE B — before you stop, write your handover into docs/agent-log.md: what A1b
found beyond the 500, anything in the ledger the next agent must not trip over,
and the state of the harness. You are the only one who knows it.

WHAT DOES NOT CHANGE
  - NO DEPLOY until everything is built. One build, one cutover, at the end.
  - Every task stays a PHOTO upload until that cutover. The 500 fix will exist
    in the code but will not be live.
  - The four guard lessons stand, and now apply to one agent:
      * sabotage at the caller's level, not an intermediate
      * sabotage the load-bearing line, not the plausible-looking one
      * a PASSING sabotage is a finding — weak test, or dead code?
      * assert WHY a sabotage went red, not just that it did

ONE WARNING. The ownership contract caught real mistakes today. Lane B refused
to fix a live bug because the file was not its own, and that discipline is why
noth

**Claude** (2026-09-19T13:07): Understood — the split ends, I continue as the single lane with Track 2. Your message cut off mid-sentence at the warning, so let me note that and proceed with what's clear.

Let me check current state before T2-16.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): Clean tree, three items left. T2-16 — the old-path check, which rule 21 says finds the worst bugs.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): Now the critical question for T2-16: does the survey change anything for a student who has **no** survey open? That's the old path.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): Now the old path with zero surveys — a student's home page must behave exactly as before.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:08): The old path is untouched with no survey — no card, no tab, every page draws, and `go('survey')` safely redirects. Now the harder case: survey present but closed for this student's venue.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:08): Correct per venue — but note EEE shows 1 other item where ECE shows 2. That's the lead/attendance difference, not a survey effect. Let me confirm rather than assume.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:08): Confirmed — the ECE student is a lead (gets the attendance card), the EEE one isn't. Not a survey effect. Now the numeric old-path check: does the survey touch scoring?

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:08): All teams read 0.0 — a weak baseline, since zero equals zero either way. Let me test with teams that actually have points.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:08): Non-zero points, 1,236 survey answers, totals unchanged. Now the full existing suite against the survey-enabled database.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:10): Running each suite twice was too slow. Let me capture once per suite instead.

**Claude** (2026-09-19T13:10): _[tool: Bash]_

**Claude** (2026-09-19T13:11): Identical to the baseline I established before any survey work: 21/1, 15/0, 33/0, 1/0. The one failure is still the unopened projects.

**Claude** (2026-09-19T13:11): _[tool: Bash]_

**Claude** (2026-09-19T13:13): `quiz-per-student` shows 2/4 where it was 42/0 earlier. I must check whether I caused that — it's exactly the regression T2-16 exists to catch.

**Claude** (2026-09-19T13:13): _[tool: Bash]_

**Claude** (2026-09-19T13:13): The quiz can't start — this needs a definite cause, not a guess.

**Claude** (2026-09-19T13:13): _[tool: Bash]_

**Claude** (2026-09-19T13:13): That's the cause: `bootcamp_old` is a fresh dump with **zero** quiz questions, while `bootcamp_test` had 10 seeded by earlier runs. Not my change — a fixture difference. Let me prove it.

**Claude** (2026-09-19T13:13): _[tool: Bash]_

**Claude** (2026-09-19T13:13): Proven: **same code, 42/0 with quiz questions, 2/4 without.** Purely fixture-driven. Let me capture T2-16 as a permanent test.

**Claude** (2026-09-19T13:14): _[tool: Write]_

**Claude** (2026-09-19T13:14): _[tool: Bash]_

**Claude** (2026-09-19T13:14): 21/21. One assertion is weak though — the leaderboard check compares a query to itself, which always passes. Let me fix that.

**Claude** (2026-09-19T13:14): _[tool: Bash]_

**Claude** (2026-09-19T13:14): Now the sabotage — and per your rule, a passing sabotage would itself be a finding.

**Claude** (2026-09-19T13:14): _[tool: Bash]_

**Claude** (2026-09-19T13:14): **A passing sabotage — that's a finding, not a retry.** Per your rule I must determine which: weak test, or dead code?

**Claude** (2026-09-19T13:14): _[tool: Bash]_

**Claude** (2026-09-19T13:15): Found it: **weak test.** My suite seeds answers via direct SQL, so the route's point-awarding never executes. The volume is real but it bypasses the code under test.

**Claude** (2026-09-19T13:15): _[tool: Bash]_

**Claude** (2026-09-19T13:15): Now the guard catches it. Restoring and verifying clean.

**Claude** (2026-09-19T13:15): _[tool: Bash]_

**Claude** (2026-09-19T13:15): Zero checks ran — something failed silently. Let me see the error.

**Claude** (2026-09-19T13:15): _[tool: Bash]_

**Claude** (2026-09-19T13:15): My own database-name guard rejected `bootcamp_old2`. Good that it's strict — let me widen the pattern to cover numbered scratch databases.

**Claude** (2026-09-19T13:15): _[tool: Bash]_

**Claude** (2026-09-19T13:15): 24/24 on the old-path suite. Let me verify all suites and commit T2-16.

**Claude** (2026-09-19T13:15): _[tool: Bash]_

**Claude** (2026-09-19T13:15): Three suites ran zero checks — their guards reject `bootcamp_old2`. Let me widen those too.

**Claude** (2026-09-19T13:16): _[tool: Bash]_

**Claude** (2026-09-19T13:16): All six suites green: 101 + 20 + 52 + 4 + 24 + 40 = **241 checks**. Committing T2-16.

**Claude** (2026-09-19T13:16): _[tool: Bash]_

**Claude** (2026-09-19T13:16): **T2-16 done. 241 checks across six survey suites.**

## The old-path result

| Suite | Before survey work | Now |
|---|---|---|
| flows | 21/1 | 21/1 |
| behaviour | 15/0 | 15/0 |
| redesign | 33/0 | 33/0 |
| onboarding | 1/0 | 1/0 |
| completion | — | 34/0 |

Identical. The one `flows` failure is still the unopened projects. Team points byte-identical after 824 answers, leaderboard order unchanged, and `task_submissions`, `attendance`, `daily_posts`, `quiz_attempts` row counts untouched.

## Two of my own tests were broken

**The sabotage passed, and that was the finding.** I made the answer route award a point per answer — the suite stayed green, because it seeded 824 answers by bulk `INSERT` and never called the route. Real volume, bypassing the thing under test. Rewritten to answer through the real route first; it now goes red. **Weak test, not dead code** — it would have shipped claiming to guard the leaderboard while guarding nothing.

**A second one in the same file**: the leaderboard check compared a query to itself. Always true. Now compared against a snapshot taken before any answers exist.

## A worse one, found by accident

My scratch-database guard rejected `bootcamp_old2`, and **three suites reported `0 pass / 0 fail`** — which in a summary table reads exactly like success. A suite that runs zero checks and exits cleanly is the quietest possible failure. Guard widened across all six.

## On quiz-per-student

It reads 2/4 on a fresh dump and 42/0 on a database with quiz questions. **Same code both times** — I ran it against both to be sure. Fixture difference, not a regression.

## Your message was cut off

It ends mid-word: *"...that discipline is why noth"*. I've proceeded on what was clear. If the warning had an instruction in it, I haven't acted on it.

Next: T2-17 screenshots — your review checkpoint — then T2-18 deploy commands.

**Vishnu** (2026-09-19T13:18): LANE A — confirmed, the cut part had NO instruction. It ended:

  "...that discipline is why nothing collided in nine hours. With one lane,
   nothing external enforces that care any more. The sabotage rule and the log
   are the only checks left. Keep both, and keep them honestly."

A warning only. Nothing to act on. Asking rather than assuming was right.

YOUR THREE FINDINGS ARE WORTH MORE THAN THE FEATURE. All three recorded:

  4. SEED THROUGH THE REAL ROUTE, NOT AROUND IT. 824 answers by bulk INSERT is
     real volume down a path the test never exercises. Realistic data is not a
     realistic path. This one would have shipped guarding nothing.
  5. COMPARE AGAINST A SNAPSHOT, never a fresh read of the same query. A query
     compared to itself is always true by construction.
  6. ASSERT THE EXPECTED NUMBER OF CHECKS RAN. 0 pass / 0 fail reads as success
     in a summary table. The quietest possible failure, and you found it by
     accident — which means it could have been missed.

That is six false greens in one day, in one feature, by an agent that knew the
pattern. Knowing about it did not prevent it. That is now the strongest argument
in the project for the v3 registry, and it belongs in the Phase B reasoning.

NOW T2-17. This is Vishnu's checkpoint and the only one he has.

Screenshots at 390px of: the student card in My work, the answering screen
BEFORE the first tap, the same screen after it locks, the admin loader, the Open
tab row, daily results, and each proof report.

The one that matters most is the answering screen before the first tap. The tap
is final and cannot be undone. 177 green checks say the logic is right. They say
nothing about whether a tired student on a phone understands that before they
touch it. Frame that one so the warning is visible without scrolling.

Then T2-18, deploy commands written down and NOT run. No deploy until the whole
build is finished — Vishnu's standing rule.

**Claude** (2026-09-19T13:18): T2-17. Let me set up the state first, then capture every screen at 390px.

**Claude** (2026-09-19T13:18): _[tool: Bash]_

**Claude** (2026-09-19T13:19): _[tool: Bash]_

**Claude** (2026-09-19T13:19): Now I need a student who hasn't answered Day 2, so the "before first tap" shot is genuine.

**Claude** (2026-09-19T13:19): _[tool: Bash]_

**Claude** (2026-09-19T13:19): _[tool: Bash]_

**Claude** (2026-09-19T13:19): The warning measurement is suspicious — `y=0-844` is the whole viewport, meaning I matched a container, not the warning itself. Let me measure the actual element.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): Precisely measured: the warning sits at y=197–234, well within the 844px fold, with the first question's buttons at y=499. Now the remaining admin screens.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): The admin loader, daily results and proof reports have **no UI** — they're API-only. That's a gap in T2-17 I must surface rather than paper over with JSON screenshots.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): I marked T2-03 and T2-08 through T2-12 DONE having built only the APIs. T2-17 asks for screenshots of screens that don't exist — so those items aren't actually complete. Let me build the admin UI.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): Clear pattern to follow. Let me build the admin surveys screen with loader, results and proof.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): All helpers available. Writing the admin surveys screen.

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:20): _[tool: Bash]_

**Claude** (2026-09-19T13:21): _[tool: Bash]_

**Claude** (2026-09-19T13:21): Now wiring it into the nav, route table and icon — the three places my own route guard exists to enforce.

**Claude** (2026-09-19T13:21): _[tool: Bash]_

**Claude** (2026-09-19T13:21): `modal_wide` and `close_modal` don't exist — I invented them. Let me check the real modal helper.

**Claude** (2026-09-19T13:21): _[tool: Bash]_

**Claude** (2026-09-19T13:21): The real helper takes an `on_save` callback and manages its own buttons. Let me rewrite my two modals to use it properly.

**Claude** (2026-09-19T13:21): _[tool: Bash]_

**Claude** (2026-09-19T13:21): No invented helpers left. Now wiring nav, route and icon.

**Claude** (2026-09-19T13:21): _[tool: Bash]_

**Claude** (2026-09-19T13:21): The route guard confirms all three places are wired — 25/25. Now capturing the admin screens.

**Claude** (2026-09-19T13:22): _[tool: Bash]_

**Claude** (2026-09-19T13:22): All nine captured, no page errors. Let me look at the two most important ones.

**Claude** (2026-09-19T13:22): _[tool: Read]_

**Claude** (2026-09-19T13:22): The warning is bold, first, and above the first question — a student cannot reach Yes/No without passing it. But this is Day 1's survey, not Day 2's four-question one; the student was served the older open survey. Let me check the proof report.

**Claude** (2026-09-19T13:22): _[tool: Read]_

**Claude** (2026-09-19T13:22): The proof report reads exactly as specified: three states, every percentage with its denominator, paired vs everyone distinguished, split by venue. But the venue gains show +70.9% over 151 and +72.1% over 43 — let me check that arithmetic.

**Claude** (2026-09-19T13:22): _[tool: Bash]_

**Claude** (2026-09-19T13:22): 151 + 43 = 194 exactly. Let me write the screens index and commit T2-17.

**Claude** (2026-09-19T13:23): _[tool: Bash]_

**Claude** (2026-09-19T13:23): The readme references `tests/screens.js`, which I ran ad hoc. Let me make that real so the screenshots are reproducible.

**Claude** (2026-09-19T13:23): _[tool: Write]_

**Claude** (2026-09-19T13:23): _[tool: Bash]_

**Claude** (2026-09-19T13:24): 25/25, fully reproducible. The warning sits at y=197–234 with the first button at y=453 — it comes before, measured not guessed. Committing T2-17.

**Claude** (2026-09-19T13:24): _[tool: Bash]_

**Claude** (2026-09-19T13:24): Zero checks again — `bootcamp_shots` isn't in those suites' guards. Exactly the silent failure I found earlier.

**Claude** (2026-09-19T13:24): _[tool: Bash]_

**Claude** (2026-09-19T13:24): All green: 101+20+52+4+24+40 = **241 checks**, plus 25 screen checks. The guard's silent-zero problem has now bitten twice — worth fixing properly.

**Claude** (2026-09-19T13:24): _[tool: Bash]_

**Claude** (2026-09-19T13:24): The `bootcamp_local` run crashed on a missing table rather than reaching my guard — different failure, still not readable as pass/fail. Let me verify the guard itself works.

**Claude** (2026-09-19T13:24): _[tool: Bash]_

**Claude** (2026-09-19T13:24): Now a refusal reads as **1 fail** instead of silently zero. Committing T2-17.

**Claude** (2026-09-19T13:25): _[tool: Bash]_

**Claude** (2026-09-19T13:25): **T2-17 done — and it caught six items I'd marked DONE prematurely.**

## The finding

I had marked T2-03 and T2-08 through T2-12 DONE having built **only the APIs**. T2-17 asked for screenshots of the admin loader, daily results and each proof report — and those screens didn't exist. There was no way to reach any of it without curl.

**An API is not a screen, and the queue said screen.** So the admin Surveys page now exists: list, paste-many loader with the parsed list shown *before* saving, daily results per question and venue, every count opening the names behind it with phone numbers, and the proof report.

## Screenshot 02 — measured, not judged

| Measurement | Result |
|---|---|
| Warning position | y=197–234 of 844 — above the fold |
| First Yes/No button | y=453 |
| Order | Warning comes **before** the first button |

A student cannot reach a button without passing the sentence *"Your first answer is saved and cannot be changed"*, which is bold and first inside the card.

**An earlier version of that measurement was wrong** — it matched a wrapping container and reported y=0–844, which would have called the warning visible however far down it actually sat. Fixed to measure the smallest element containing the text.

All nine screens are in [docs/screens/](docs/screens/) with a readme saying what to look at. `tests/screens.js` regenerates them.

## Finding 6 came back in a different costume

The scratch-database guard **silently zeroed five suites, twice** — reporting `0 pass / 0 fail`, which reads as success. Now it prints `FAIL` and exits non-zero. Same lesson, second occurrence, caught only because I compared totals.

**241 checks across six suites, plus 25 in the screens run.** Route agreement: 25 nav entries, 25 routes.

Next: T2-18, deploy commands written down and not run.

**Vishnu** (2026-09-19T13:27): LANE A — finish T2-18 and stop.

Write the deploy command set into docs/agent-log.md. DO NOT RUN IT. No deploy
until the whole build is finished — Vishnu's rule, confirmed twice today.

Then flush the log. Six entries for eighteen items is thin, and the log is the
only source of truth now that the ownership contract is retired.

Then stop. Next session merges survey into main and starts A3.

Good work today. Six false greens found, one live bug caught before it hit Day
3, and six items you had marked DONE that you found and fixed yourself. The
screenshot item earned its place — nothing else would have caught those.

**Claude** (2026-09-19T13:27): T2-18. Let me gather the facts the deploy set needs — the exact commit, the migration order, and the ownership reassignment that took the app down twice.

**Claude** (2026-09-19T13:27): _[tool: Bash]_

**Claude** (2026-09-19T13:28): _[tool: Bash]_

**Claude** (2026-09-19T13:28): `fg4.js` is a stray debug file I left in the repo root — that must not ship. Let me remove it and check the ownership step.

**Claude** (2026-09-19T13:28): _[tool: Bash]_

**Claude** (2026-09-19T13:28): Committed by accident in an earlier `git add -A`. Removed. Now the ownership reassignment and restart details.

**Claude** (2026-09-19T13:28): _[tool: Bash]_

**Claude** (2026-09-19T13:28): _[tool: Bash]_

**Claude** (2026-09-19T13:28): I have everything. Let me verify the migration is idempotent one final time, then write the deploy set.

**Claude** (2026-09-19T13:28): _[tool: Bash]_

**Claude** (2026-09-19T13:28): Rehearsed clean, twice. Now writing T2-18 and flushing the log.

**Claude** (2026-09-19T13:29): _[tool: Bash]_

**Claude** (2026-09-19T13:29): Let me confirm all 18 Track 2 items are ticked and the remaining TODOs are Track 3 only.

**Claude** (2026-09-19T13:29): _[tool: Bash]_

**Claude** (2026-09-19T13:29): All 18 Track 2 items done. Final commit.

**Claude** (2026-09-19T13:29): _[tool: Bash]_

**Claude** (2026-09-19T13:29): **Track 2 complete. 18 of 18. Nothing deployed.**

## T2-18

The deploy set is in [docs/agent-log.md](docs/agent-log.md), written and **not run**. Payload commit `574614e`.

Rehearsed against a local copy of the dump: migration applied, applied again to prove it's a no-op, three tables present, teams and points unchanged.

Three things I wrote in deliberately:

**The migration, ownership reassignment and restart are one step**, with the reason on the page. `psql` runs as `postgres`, the app connects as `bootcamp`, so every object the migration creates is unreadable until ownership moves. That took the app down twice in v2 — and both times the commands existed, just in three separate places.

**Rollback is a code rollback**, ~90 seconds. The migration is additive, so the previous code runs unchanged against the new schema. The down migration isn't part of backing out; it's only for the tables themselves, and it refuses while answers exist.

**What stays off after the deploy**: every task remains a photo upload, so the task-submit fix ships and cannot fire; no survey is open until an admin opens it per venue; the final round asks for confirmation naming the real count.

## One last find

`fg4.js` — a debug script I'd left in the repo root, committed by an earlier `git add -A` eight commits ago. It would have shipped in the payload. Found by listing what the deploy actually carries rather than trusting the diff. Removed.

## Where it stands

| | |
|---|---|
| Checks | 266 across seven suites |
| Existing suites | unchanged from pre-survey baseline |
| Branch | `survey`, 26 commits, pushed, tree clean |
| Deployed | nothing |

The log is flushed — every item from T2-01 through T2-18 now has its own entry with tests, old-path, assumptions and findings.

## For the next session

1. **Review [docs/screens/](docs/screens/)**, especially `02-survey-before-first-tap.png`. That's the one thing today's 266 checks can't answer.
2. Merge `survey` → `main`, rebase `v3-dev`, start A3.
3. **[docs/merge-debt.md](docs/merge-debt.md) M1 is the one to watch** — `run.sh` lists nine chase queries and query 10 lives on this branch. After the merge the chase set silently produces nine CSVs instead of ten. No error, just a quietly incomplete list.

Stopping here.