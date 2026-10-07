---
tags: chat
project: "[[Projects/ac-training/SUMMARY]]"
date: 2026-09-19
source: Claude office account (Cowork)
session: cse_012xqeN8Rb1bwjdvry5U7Jfz
---
# Rework status report

- Project: [[Projects/ac-training/SUMMARY]]
- Date: 2026-09-19
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** hi i am back after lot of rework.... 


i need a proper report where we stared where we are now and what are the chages we have done

**Claude:**   _[tool: Projects]_

**Claude:**   _[tool: Projects — claude/index.md]_

**Claude:**   _[tool: Projects — claude/v2-plan.md]_

**Claude:**   _[tool: Projects — claude/bootcamp-plan.md]_

**Claude:**   _[tool: Projects — claude/v2-build-prompts.md]_

**Claude:**   _[tool: Projects — claude/agent-working-rules.md]_

**Claude:**   _[tool: Projects — claude/features-and-risk.md]_

**Claude:**   _[tool: Projects — claude/handover.md]_

**Claude:**   _[tool: Projects — claude/ux-fixes-applied.md]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Here is the full picture, in points.

## Where we started (16 Sep, 2 days before Day 1)

- A plan only: Discord + a simple dashboard, 7 tables, 25 teams, 150 students, 5 days.
- Points idea: 5 per project + 5 per quiz = 10 a day.
- Quiz was **one per team**, one attempt, 10-minute clock.
- No file uploads at all — only Google Drive links pasted by the team lead.
- Login: student email + one shared code `ARA2026`; staff get a password.
- A working app existed but was rough — a UX audit found **5 blockers, 10 high, 11 medium** problems.
- Roster loaded was EEE only: 14 teams, 55 students.

## What we changed (16 Sep — v1 go-live)

- Fixed all 26 UX problems (attendance opens on today, quiz resume after refresh, plain-English errors, own-team highlight, mobile menu, skeleton loading, confirm before delete).
- Loaded the ECE roster on top of EEE → 52 teams, 206 students.
- Standardised every team code to `DEPT-TNN-TEAMNAME`.
- Rebuilt the front end on the araCreate design system — Home page per role, charts, toasts, breadcrumbs, empty states.
- Added an **onboarding day** (17 Sep) so the day before Day 1 was not blank.
- Brought the repo to araCreate conventions (structure, headers, snake_case, conventional commits).
- 71 Playwright checks across 4 suites, against a real server + real PostgreSQL.
- Deployed live at https://vcet.aracreate.academy behind HTTPS.

## The v2 rework (built overnight 17–18 Sep, deployed 01:20)

Decision: build all ten changes in one night instead of spreading them across the bootcamp — so the cohort never lives under two different rules mid-bootcamp.

Ten changes shipped:

1. Full student profile page.
2. Progress bar = **profile completion only** (photo, phone, email, education, skills, goal, old CV, new CV).
3. Pre/post assessment — separate, **zero points, no timer**, compared as % gain.
4. Quiz is now **per student**, not per team. Team mark = average of those who attempted.
5. Quiz timer is **30 seconds per question**; answer saves the instant it is picked.
6. Projects → **Tasks**, several a day, day still capped at 5 points.
7. Admin task creator with 5 hand-in types (image / Drive link / text / file / none).
8. Attendance once a day, plus an audit trail.
9. **Full department split** — nothing opens for everyone. Quiz, tasks, attendance, assessments and projects all opened **by hand, per venue**. No cron, no schedule.
10. Google Drive storage + CV migration (service account + Shared Drive).

Everything was add-only — no table dropped, no column renamed, because it landed on a live app.

## Where we are now (19 Sep, Day 2)

| | |
| --- | --- |
| Students | 209 (plan said 206) |
| Teams | 53 — EEE 14, ECE 39 |
| Projects | 159 in 3 groups |
| Drive | 142 CVs copied, 53 team folders |
| Tinkercad codes | 52 of 53 teams |
| Deploys since v2 | 6 more across Day 1 and Day 2 morning |
| Current state | Everything **closed** — nothing opens itself |

## Bugs found and closed (8 serious ones)

1. `rsync --delete` from a working tree would have wiped `uploads/` and the pre-deploy dump → now `git archive` of one named commit.
2. Migrations run as `postgres` left objects the app could not read — took the app down twice.
3. Lead check read the wrong field (`is_team_lead` vs `is_lead`) — every team lead was locked out of their own project.
4. Every quiz route required lead → 156 of 209 students locked out.
5. Opening a quiz for EEE **leaked it to ECE** through a fallback.
6. No locking on team folder creation → twin folders on Drive.
7. NUMERIC came back as a string — leaderboard would have sorted 9 above 45.
8. Two screens were never rebuilt after their backend changed.

**The pattern:** in all 8 cases the new code was right. The bug was in the old code underneath it.

## Still pending

- **Quiz questions — all nine days have 0.** Nothing opens below 5 questions. This is your job each evening.
- Open the **pre-assessment for EEE** — that room still has not had it.
- Re-run the CV migration each evening for stragglers.
- Delete the server CVs — **next week only**, after copies verify.
- Caddy access logging + `jq` (today no incident can be traced from logs).
- One session-level test per identity-gated route.

## Known issues (small, live)

- 19-second outage on 18 Sep during a mid-morning deploy — one photo lost (student 81), not contacted.
- `PUT /api/profile/details` returns ok for keys it silently drops.
- `submission_type: 'drive'` is the one hand-in path never tested.
- Profile photos are on the server only — not on Drive, not in any backup.
- 4 orphan CV files on disk with no database row.

## The real risk is not code

- **The EEE venue.** 53 of 55 students had never signed in on Day 1 morning. Chasing worked — CV hand-ins went 83 → 143 in one day. Keep chasing that room.

**Rollback if needed:** tag `pre-merge-v2`, branch `backup-v2-premerge`, PREDEPLOY and PREMIGRATE dumps in `.archives/` on both machines.

Sources: `claude/index.md`, `claude/bootcamp-plan.md`, `claude/features-and-risk.md`, `claude/ux-fixes-applied.md`, `claude/handover.md`, `claude/v2-plan.md`, `claude/v2-build-prompts.md`, `claude/agent-working-rules.md`

**Vishnu:** so for more it is not in a proper product seructore we need to oragains this is a proer flpw

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: Projects — claude/design.md]_

**Claude:**   _[tool: Projects — claude/go-live-checklist.md]_

**Claude:**   _[tool: Projects — claude/v3-restructure-plan.md]_

**Claude:** Plan saved to the project as `claude/v3-restructure-plan.md`. Here it is in points.

## Why it feels messy

- Admin nav is **16 flat items** — Home, Marking, Projects, Leaderboard, Quiz results, Open, Register, Quiz now, Tasks, Tinkercad, Assessment, Students, Teams, Progress, Staff, Quizzes.
- No grouping. Daily actions, setup and reports all mixed together.
- Every new v2 feature became a **new nav item**, because there was no structure to put it in.
- One idea, many names: Task / Project / Quiz / Assessment / Tinkercad are all just "work given to students".
- Quiz, Quiz now, Quiz results, Quizzes — four items, one thing.
- **Three ways to open something** (Open tab, Home Today box, the thing's own screen). That is exactly where the EEE→ECE leak bug came from.
- Code: the whole API is one `server.js`; the whole UI is one `app.js`.
- Batch data is hardcoded — dates, EEE/ECE, 9 days, `ARA2026`, college name.
- Docs in 2 places, some stale (handover still says 206 students and "no file uploads").

## Fix 1 — name things once

- **Program** (one bootcamp run) → **Venue** (was "department") → **Team** → **Student**
- **Activity** = one word for everything given to students. Five types: Task, Project, Quiz, Assessment, Attendance. Same create form, same open, same hand-in, same scoring.
- **Release** = an activity opened for one venue. Table already exists — make the UI match it.
- Tinkercad becomes a field on a team, not a page.

## Fix 2 — new navigation

Admin: **5 groups instead of 16 items.**

| Group | Holds |
|---|---|
| Today | run-the-day board |
| Content | Tasks, Projects, Quizzes, Assessments |
| People | Students, Teams, Staff, Register |
| Live | Release board, Attendance, Marking |
| Reports | Progress, Leaderboard, Quiz results, Exports |

- **One release board.** One table — rows = activity, columns = EEE / ECE, cell = Closed / Open. Click the cell to toggle. Kills the three-paths problem for good.
- Student gets **5 items only**: Home, My work, My profile, My team, Leaderboard. "My work" holds today's task + project + quiz + assessment together — the student never needs to know which type it is.
- Lead adds Attendance (inside My team). Mentor: Home, Marking, My teams, Leaderboard.
- **New rule: a new feature never gets a new nav item.** It goes in one of the five groups or it is not built.

## Fix 3 — code structure

- `src/modules/` — auth, programs, venues, students, teams, activities, releases, submissions, scoring, attendance, reports, storage. Each = routes + service + queries.
- `server.js` becomes wiring only.
- `src/web/pages/` — one file per screen, instead of one giant `app.js`.
- **`releases.is_open_for()` is the only gate.** Nothing checks day or department directly, anywhere.
- One name per field (`is_lead` everywhere) — the lead bug was one field read by two names.
- Every route declares its role, and every role has a test that **actually signs in**.

## Fix 4 — make it reusable

- Everything keys off `program_id`.
- `programs` table: college, dates, days, join code, staff password, Drive root.
- `venues` table replaces the EEE/ECE enum — a program can have 1 venue or 6.
- Roster import becomes a **screen with CSV upload, additive only**.
- **Delete `load-eee.sql` and `load-ece.sql`** — they delete students before inserting, the most dangerous thing in the repo.
- Result: batch 2 is created in the UI in 10 minutes, no code change.

## Fix 5 — docs

- Repo `docs/` is the truth; project docs only link to it.
- `product/ · design/ · dev/ · ops/ · history/` + one README index.
- First job: fix stale facts (53 teams / 209 students, uploads exist, deploy uses `git archive` not the old `rsync` still printed in the checklist).

## Fix 6 — way of working

- **Deploy only when students are not working** — night, after the day ends. The one mid-morning deploy cost 19s downtime and a photo.
- Two lanes, one worktree each, one step per prompt, report before deploy.
- Fresh dump before deploy, check row counts, watch logs across restart.
- Before saying a feature is done: **go look at what the old path still does.**

## Rollout

| Phase | When | What |
|---|---|---|
| 0 Freeze | now → 26 Sep | Bug fixes only, night only. Quiz questions. EEE pre-assessment. CV migration nightly |
| 1 Docs + tests | 27 Sep → ~3 Oct | One doc set, session-based tests. No behaviour change |
| 2 Code split | ~1 week | server.js → modules. Pure move |
| 3 Nav + release board | ~1 week | The 5 groups, one release board, student "My work" |
| 4 Activity model | ~1 week | All four types become one shape |
| 5 Multi-program | ~1–2 weeks | programs + venues, roster import screen |
| 6 Pilot | next batch | Create batch 2 in the UI, zero code change |

Phases 1 and 2 are safe even with live data. Phase 3 onward waits until after 26 Sep.

Want me to start on Phase 1 now (docs cleanup — zero risk to the live app), or first draw the new nav as a screen-by-screen map you can approve?

**Vishnu:** and ther is lot. of thins unknow thingw like we can see who are are completr the task and the proejct as now we can see only the numbers 

and proper student profile team profile all are misosng

**Claude:**   _[tool: Projects — claude/v3-restructure-plan.md]_

**Claude:** Added to the plan. In points:

## The real problem: numbers with no names

- The app tells you **how many**. Running a room needs **who**.
- "43 handed in" cannot be acted on. "These 10 did not, here are their numbers" can.
- **Good news: the data is already there.** Every submission has a student id, team id and timestamp. This is a screen gap, not a database gap.

## New rule — no number is a dead end

Every count becomes a link, and the **negative side** is also a link.

| You see | Clicking gives |
|---|---|
| 43 of 53 handed in | the 43 **and** the 10 who did not |
| Attendance 86% | the students absent today |
| Quiz average 3.2 | every attempt + who never opened it |
| Profile 70% | the exact fields still blank |
| 12 waiting to score | the 12, one click from scoring |

Every list carries name, roll, team, venue, **phone, email**, time — plus **Export CSV** and **Copy all phone numbers**. Chasing the room is the actual job.

## New screen 1 — Activity detail

One page per task / project / quiz / assessment, per venue.

- Tabs: **Done · Not done · Scored · Not scored**
- Columns: name · team · venue · handed in at · what they handed in (opens it) · score · who scored
- Sort, filter, export

## New screen 2 — Completion matrix

- Rows = students or teams. Columns = days or activities.
- Cell = done / not done / scored / absent, as colour blocks.
- An empty row = a student falling behind, visible in one second.
- Click any cell to open that submission.
- This replaces most of what Progress, Marking and Quiz results do separately.

## New screen 3 — Student profile (full)

- Header: photo, name, roll, dept, year, team + role, venue, phone, email, **Copy contact**
- Completion bar **with the missing items named**, not just a %
- CVs — old and new, opening the Drive file
- Goal from Day 0
- Attendance strip — 9 boxes
- Work table — every activity: done / not / when / score
- Quiz detail — right, wrong, never answered
- Assessment — pre, post, **% gain**
- Daily posts timeline — **admin only**, that privacy rule stands

## New screen 4 — Team profile (full)

- Header: team code, venue, track, lead, mentor, Tinkercad code, Drive folder link
- Rank + points, broken down by day
- Members — 6 rows, each showing their own completion, each name clickable
- Member × day grid
- All submissions with links and scores
- Copy all six phone numbers

**Linking rule:** every student name and team name anywhere in the app becomes a link to these pages. A plain-text name is a bug from now on.

## Open questions I need answers on

I wrote these into the plan so they stop being unknowns:

1. Can a **team lead see which of his 6 members** finished today? — I recommend **yes for status, no for content**.
2. Can a **mentor** open a student profile? — recommend yes, minus the daily posts.
3. Can a student see their **own full record**? — recommend yes, it makes Day 9 stronger.
4. What counts as **"done"** for a task with no hand-in type?
5. **Absent vs not-yet-marked** — today both look the same. Must be two colours.
6. Profile photos are **server only, no backup**. Move to Drive or accept the risk?
7. **Certificates / Day 9 export** — not built at all.
8. Who writes quiz questions if you are not in the room next time?

## My recommendation for this week

Build **"who, not how many"** as read-only pages in one night window.

- New pages only — reads existing tables, changes nothing.
- Cannot break a student's day.
- You can use it the next morning to chase EEE.

Everything else (profiles, matrix, nav regroup) waits until after 26 Sep.

Shall I start on that one — the "who has not done it" lists — for tonight's window?

**Vishnu:** ok can we fixx all and plna for total new deplot at once

**Claude:**   _[tool: Projects — claude/v3-build-and-cutover.md]_

**Claude:** Yes — one build, one deploy is the right call. Plan saved as `claude/v3-build-and-cutover.md`.

## The rule that makes big-bang safe

- **Nothing touches the live server until v3 is finished and proven on a copy of the real data.**
- v2 stays frozen until the cutover night.
- No half-migrated state, no feature flags, no two systems running different rules on the same students. That was exactly what went wrong with v2.

## Timing — the important part

| | |
|---|---|
| Build window | 20 Sep → ~24 Oct, on a `v3` branch + staging box. Nothing deployed |
| v2 meanwhile | Frozen. Bug fixes only, night only |
| Cutover | One night, **between batches** |
| Realistic date | End of October, before batch 2 |

- **The cutover is not urgent.** This batch finishes on v2 and finishes fine. v3 exists for **batch 2**.
- Rushing it into the bootcamp gains nothing and risks the run.

## But you still need "who hasn't done it" for 7 more days

**Answer: no deploy at all.**

- I write SQL queries you run over `ssh hetzner`.
- Output: CSV per venue — who has not handed in, who is absent, who never opened the quiz, **with phone numbers**.
- Run morning and evening, under a minute.
- Read-only. Zero risk. No restart, no deploy.

That covers the bootcamp completely. Everything else waits.

## v3 is NOT a rewrite from scratch

Keeps: the database and all data · the araCreate design system · every rule that works (per-venue opening, release gate, 30s per question, team average, 5-point cap) · Drive integration · the deploy hardening already learned.

Restructures: code, navigation, the model above the tables, and the screens.

## Scope — 8 blocks

- **A. Model** — programs + venues + one activity shape, one gate, one field name, load scripts deleted
- **B. Visibility** — every number links to its list, activity detail page, completion matrix, CSV + copy phones
- **C. Missing pages** — full student profile, full team profile, every name a link
- **D. Navigation** — 5 admin groups, one release board, student "My work"
- **E. Code** — modules, one file per screen, server.js = wiring only
- **F. Tests** — every route, every role, **through a real signed-in session**, on 209 students not 3
- **G. Docs** — one tree, one index, stale facts fixed
- **H. Day 9** — certificates, journey export, transcript — build or drop, decide now

## How it gets built

- `v3` branch. New `src/` **beside** the old one. Old tree deleted in the **last** commit, not the first.
- **Staging on the same Hetzner box**: second DB `bootcamp_v3` loaded from a **real production dump**, second systemd unit, staging hostname behind basic auth, refreshed every few days.
- Two lanes, one worktree each, one step per prompt — rules unchanged.
- Every migration has a **down**, and runs **5 times** against real dumps before the real one.

## Data migration — nothing re-entered

1. Create `programs` → insert VCET Basic Electronics
2. Create `venues` → EEE, ECE
3. Backfill `program_id` / `venue_id` everywhere
4. Create `activities`, backfill from tasks/projects/quizzes/assessments
5. Point submissions + scores at `activity_id`
6. **Verify: row counts, team totals, leaderboard order identical**
7. Old tables kept read-only one month

**Step 6 is the gate.** One team's total moves by one point → cutover does not happen that night.

## Gate list — v3 does not ship until all true

- Every route has a session-based test, per role
- Full suite green on staging, on a fresh production dump
- Migration run 5× with verified down each time
- Totals, leaderboard, attendance identical before and after
- Every count opens its list — checked by hand, screen by screen
- Full cutover rehearsal, **timed**, rollback rehearsed too
- **Batch 2 created on staging through the UI with zero code change** ← the real proof it's a product
- Verified dump, row counts checked

## Cutover night — 11 steps

Dump + verify → copy off box → stop app, maintenance page → migrate + reassign ownership in the same step → deploy `git archive` of a tagged commit → start, **watch the log across the restart** → verification queries → hand-walk smoke list (4 roles, open per venue, hand in, score, click a number) → lift page → watch 30 min.

Rollback at any step: restore dump, deploy `v2-final` tag, restart. Rehearsed, so it's a known number of minutes.

## Timeline — 5 weeks, two lanes

| Week | Lane A | Lane B |
|---|---|---|
| Now→26 Sep | Bootcamp only + SQL chase-lists | Staging box, dumps, `v3` branch, session test harness |
| W1 | programs/venues/activities migrations | Module split: auth, students, teams, releases |
| W2 | Activity engine — one shape | Module split: activities, submissions, scoring, reports |
| W3 | Visibility + matrix | Student profile, team profile, name linking |
| W4 | Nav regroup + release board | CSV import, multi-program, Day 9 exports |
| W5 | Gate list, 5 migration runs | Docs, cutover runbook, rehearsal |

## Biggest risks, each with an answer

- No partial rollback → full rehearsal, rollback rehearsed too
- Migration wrong → 5 runs on real dumps, totals verified not assumed
- A screen nobody rebuilt → gate list walks every screen by hand (this bit v2 twice)
- **Scope creep, cutover never comes** → the 8 blocks are the scope; anything new goes to a v3.1 list
- Old paths still open → old `src/` deleted in the last commit

**The one thing to hold to: the cutover happens between batches, never during one.**

Two things I need from you to start: (1) yes/no on the SQL chase-lists for this week, and (2) a decision on block H — are certificates and the Day 9 export in scope, or dropped?

**Vishnu:** ask me question one by one

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Noted — certificate stays simple (name, roll, program, dates, one line). Exact wording later.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Understood — a survey before each day starts, all 9 days. That's a new feature. Let me pin it down.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Compulsory — noted. Go ahead, tell me the rest about the survey. I'll hold my next question until you're done.

**Vishnu:** now now the app loosk veyr worst we need to make it like a top tire app 

look feel respoince newss use componest only from any opne source to make it very easy

**Claude:** Got it — top-tier look, and use an open-source component library instead of hand-rolling. One question at a time.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Fair question — I was assuming. Here is where that came from:

- `design.md` says the app "sits on the araCreate design system, which ships **90+ components**" and the app was only using about twelve.
- The repo follows `aracreate-conventions`, a separate shared repo in your GitHub org.

So I read it as a **shared araCreate library** meant for more than this one app. If the bootcamp dashboard is the only thing using it, then it is not really shared and the question is much simpler.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Perfect — simpler. shadcn + Tailwind with araCreate colours, this app only.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: Projects — claude/v3-decisions.md]_

**Claude:** All 23 decisions saved in `claude/v3-decisions.md`. Quick recap of the UI ones:

- **React + Tailwind + shadcn/ui**, araCreate colours, **light mode only**, **phone-first**
- **Vite + React as static files** — backend untouched, easy rollback
- Old araCreate CSS replaced in this app only — nothing else uses it
- **3 screens first for your approval** before the other 20+ get built

One thing I have to flag before the next question: **certificates (D6) and the daily survey (D12) are needed for this batch**, which means we *do* have to deploy to v2 before 26 Sep. That breaks the freeze. It should be **one night deploy, not several**.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Monday = Day 4. So it must be built tomorrow and deployed Sunday night after Day 3 ends. Tight but doable.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Right — the survey becomes a normal activity on the Open tab, opened per venue by you, like everything else. That fits the release model exactly.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Understood — for students there is **one "My work" list**. Survey, task, project, quiz, assessment are all just *their work*. The five types are our language, not theirs.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Clear — **no calendar deadlines for deploys. Build it fully, test it fully, then deploy in the next window when nobody is working.** I'll stop attaching dates to deploys.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Certificates out of our scope — another dev. Dropping them.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Two devs on one live app is exactly how v2 got hurt. Need to sort this.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Ah — misunderstood. "Another dev" = **developed later**, not another person. Good, no coordination problem.

**Claude:**   _[tool: Projects — claude/v3-decisions.md]_

**Claude:** Decisions log updated. Three tracks now:

- **Track 1 — chase-lists.** Read-only SQL, no deploy, usable tomorrow morning.
- **Track 2 — daily survey.** Built on the current app, deployed when fully done and tested.
- **Track 3 — v3 rebuild.** Restructure + visibility + profiles + nav + the full React/shadcn front end. One cutover, between batches.
- **Deferred:** certificates, journey page, transcript.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Right — one complete plan with hard guardrails, handed to a dev agent. Two guardrail questions before I write it.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** No deploy by the agent. Local only, dev branch, full local test first, you deploy at night. Locked in.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Writing the master plan now.

**Claude:**   _[tool: Projects — claude/v3-agent-brief.md]_

**Claude:** The full brief is written — `claude/v3-agent-brief.md`. Hand this one file to the dev agent.

## What is in it

**Part 1 — 21 hard rules.** Breaking any one is a stop-work event.

- Never deploy. Never ssh to production. Never touch the production database.
- Own branch only, one worktree, no `git stash`, push after every step.
- Never run `load-eee.sql` / `load-ece.sql`.
- Every migration additive, with a working `down`.
- Ownership reassignment in the same step as the restart.
- Never ask for or print a secret.
- One feature at a time, never widen scope mid-feature.
- **Before reporting done: go look at what the old path still does.**

**Part 3 — three separate tracks, three branches**

| Track | Branch | Ships to |
|---|---|---|
| Chase lists | `chase-lists` | Nothing — SQL you run |
| Daily survey | `survey` off `main` | Live app, one night deploy |
| v3 rebuild | `v3-dev` | Cutover, between batches |

**Part 4 — local setup.** You pull the nightly dump to the Mac, agent loads it locally. Agent never fetches it. Every test at 209 students / 53 teams.

**Part 5 — chase lists.** 8 queries, CSV with name, roll, team, venue, phone, email. Cross-checked against a second query written a different way — prove, don't assert.

**Part 6 — the survey.** Full spec: 3 new tables, release rows per venue, paste-many loader, first card in My work tagged Required, answers save on tap, results with Yes/No counts linking to named lists. Explicit gate: **opening for EEE must not leak to ECE.**

**Part 7 — v3 in phases A→F.** Foundation → data model → front end → screens → visibility → reusable. **Phase C3 is a hard stop** — three screens, you approve the look, nothing gets built twice.

**Part 8 — definition of done.** Ten boxes. An unticked box is fine; a hidden one is not.

**Part 9 — the report format** he sends you after each feature. The DEPLOY section hands you the exact commands; he never runs them.

Two things I still need from you when you get a moment:

- What the survey questions are actually for — you said you had more to add.
- The open items in the restructure plan: can a **team lead see which of his 6 members finished**, and what counts as **"done"** for a task with no hand-in.

**Vishnu:** save all in project folder

**Claude:**   _[tool: Projects]_

**Claude:** All four new documents are saved. Let me also fix the index so they're findable.

**Claude:**   _[tool: Projects — claude/index.md]_

**Claude:** Everything is in the project folder.

## New documents saved

- **`v3-agent-brief.md`** — the one to hand the dev agent. 21 hard rules, 3 tracks, full survey spec, v3 phases A–F, definition of done, report format.
- **`v3-decisions.md`** — all 24 decisions you made, numbered.
- **`v3-restructure-plan.md`** — what is being rebuilt and why.
- **`v3-build-and-cutover.md`** — staging, data migration, gate list, cutover runbook.

## Also updated

- **`index.md`** — now points to the new docs first, corrects the numbers to **209 students / 53 teams**, and flags which old docs are stale (`handover.md`, `features-and-risk.md`, `bootcamp-plan.md`, and the old `rsync` deploy command).

Reading order for the agent: `v3-agent-brief.md` → `v3-decisions.md` → `agent-working-rules.md` → `v2-build-prompts.md`.

Still open when you're ready: what the survey questions are for, whether a team lead can see which of his 6 members finished, and what counts as "done" for a task with no hand-in.

**Vishnu:** jsut give me the start pormt

**Claude:** ```
You are the dev agent for the araCreate bootcamp dashboard.

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
```

**Vishnu:** What I found
File you named	Status
claude/v3-agent-brief.md	Missing. No claude/ directory exists. Not in any branch, not in git history, not in either worktree.
claude/v3-decisions.md	Missing. Same.
claude/agent-working-rules.md	Missing. Same.
claude/v2-build-prompts.md	Exists, different path: docs/v2-build-prompts.md
I searched the working tree, git log --all for any commit that ever added a file matching those names, all 12 local branches, and both worktrees (.worktrees/per-student, .worktrees/side). Nothing. So it isn't a case of them living on another branch — they were never committed here.

Why I can't just proceed
The brief is the authority, and two things I need come specifically from it:

Part 9's report format. You asked for the deliverable in that format. I have no way to reconstruct it.
The working rules. "Never widen scope," "go and look at what the old code path still does" are quoted in your message, but agent-working-rules.md presumably has more that I'd be silently violating.
v3-decisions.md likely also settles things the eight queries depend on — what "today" means relative to a day number, whether a day is a date or an index, how absent is recorded versus simply unmarked. Query 5 and query 6 are only distinguishable if I know how attendance is stored, and guessing there produces two chase lists that quietly overlap or quietly miss people. With 209 real students being chased, a wrong list costs you real phone calls.

Two other things worth flagging now
Track 1 is described as read-only SQL, but the app looks like it may not be SQL-backed. Before I say that with confidence I'd need to read the source — but .env and src/ are here and I haven't opened them, since reading code is the thing you told me to do only after the brief. Worth confirming the chase scripts should target a SQL database at all.

"One CSV per query per venue" is 16 files per day. Queries 5 and 6 may also need a definition of "today" that accounts for the bootcamp's day numbering, which the day-number argument to run.sh implies exists somewhere in the decisions doc.

What I need from you
Point me at the three missing files — most likely they're outside this repo (the ~/araCreate/Claude outputs/ directory caught my eye, but I haven't looked inside it without being asked) or they exist somewhere I should be told about. Paste them, or tell me the path and I'll read them.

If they genuinely don't exist yet, say so and tell me how you want to handle the two gaps that actually block me: the Part 9 report format, and the attendance/day-number semantics. I can work from your answers to those and proceed without the full brief — but I won't invent them.

**Claude:** The agent is right and the mistake is mine — those docs live in the **Claude project**, not in the repo. Nothing was ever committed there.

Two fixes. First, the corrected prompt — self-contained, nothing to look up:

```
You are the dev agent for the araCreate bootcamp dashboard.
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
```

Three things the agent flagged, answered:

- **It is PostgreSQL** — confirmed, Postgres 17, database `bootcamp`.
- **16 CSVs a day** — good catch, changed to one CSV per query with a venue column.
- **Attendance and day semantics** — I don't have the schema, and neither did the old brief. So I turned it into Step 1: the agent reads the schema and reports back before writing a single query.

Second fix: those docs should be **in the repo**, not only in Claude. Want me to write the four of them into `docs/` on your Mac? I'd need you to approve folder access to `~/araCreate/bootcamp-dashboard`.

**Vishnu:** yes keep it in the exact proejct repo itslef

**Claude:**   _[tool: mcp__remote-devices__device_request_folder_access]_

**Claude:**   _[tool: mcp__remote-devices__device_list_dir — /Users/vishnuvarthanvenkatapathy/araCreate/bootcamp-dashboard/docs]_

**Claude:**   _[tool: Bash — mkdir -p /mnt/user-data/outputs/docs && cat > /mnt/user-data/outputs/docs/agent-working-rules.md <<']_

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/v3-decisions.md <<'ACDEC' # v3 — decisions log  Running list. Ever]_

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/v3-restructure-plan.md <<'ACPLAN' # v3 — restructure plan  Turning]_

**Vishnu:** FEATURE: Step 1 only — schema reconnaissance for the chase lists. No queries written, no files created, no branch cut.

a) How a "day" is stored
A derived integer, never a stored date. There is no date column on any work table. src/server.js:2939:


SELECT start_date, total_days, posts_open,
       (CURRENT_DATE - start_date) + 1 AS raw_day
  FROM settings WHERE id = 1
then clamped: raw < 1 → 0, raw > total → total+1, else raw. settings is a single row (CHECK (id = 1)), start_date is DATE '2026-09-18', total_days 9.

Every work table carries day INT with CHECK (day BETWEEN 1 AND 9) — daily_posts (1–60, wider), projects, attendance, quizzes, tasks. So "today" for a chase list resolves to (CURRENT_DATE - start_date) + 1.

Two consequences worth agreeing before I write anything:

CURRENT_DATE is the database server's timezone, not Asia/Kolkata unless the server is set to it. If the local DB and production differ, a chase list run near midnight names a different set of students. I'd rather run.sh take the day number explicitly — which your spec already says — and compute the default from the same expression only as a convenience, printing it so it can be checked.
Today is 19 Sep 2026 and start_date is 18 Sep, so today is Day 2. That matches a bootcamp running 18–26 Sep.
b) How attendance is stored — Q5 and Q6 are genuinely two lists
Two different lists. Confirmed by reading both write paths, not inferred from the schema.

attendance has present BOOLEAN NOT NULL DEFAULT FALSE with UNIQUE (student_id, day). A row with present=false is a real, stored "absent". A missing row is "nobody took the register for this student".

What makes this certain is that no code path ever deletes an attendance row. The only two writers:

Team lead, src/server.js:605 — ON CONFLICT DO NOTHING. Deliberately so: the comment explains the old DO UPDATE let a lead silently overwrite a settled register by tapping twice.
Admin, src/server.js:2009 — ON CONFLICT DO UPDATE, requires a ≥3-char reason, writes an attendance_audit row. An admin "unmark" sets present=false; it does not delete.
So Q5 is EXISTS(row) AND present=false, Q6 is NOT EXISTS(row). They cannot overlap, and together they are the complement of present.

One real gotcha for Q6: the attendance register is gated per venue by releases. If ECE's register hasn't been opened yet today, every ECE student is legitimately unmarked and Q6 returns all 151 of them — a chase list that is noise, not signal. I'd like Q6's CSV to carry a column saying whether the register was open for that venue, so you can tell "the lead didn't take it" from "it isn't open yet."

c) Phone, personal email, and profile completeness
Phone lives in two places; personal email in one.

students.phone — the roster, loaded from the college
student_profiles.phone — what the student typed
student_profiles.personal_email — profile only. students.email is the login (college address), not a personal one.
The app resolves phone as COALESCE(NULLIF(trim(p.phone),''), NULLIF(trim(s.phone),'')) — profile first, roster second (2026-09-17-b-profile-completion.sql). The chase CSVs must use that same COALESCE, or you'll call the wrong number.

Profile completeness is computed in JavaScript, not SQL, and the two disagree. src/routes/profile-completion.js:44 is authoritative — its own comment says "The weights live here and nowhere else":

Field	Weight
phone	15
personal_email	15
skills	20
goal	15
resume_v1	20
resume_v2	15 (only from Day 8)
Before Day 8 the bar is out of 85, so a student who has done everything possible sees 100%.

Three things here that affect Q7 and need your decision:

photo and education were removed from the weights but are still in v_student_progress. The code comment says both were taken off the profile page because the resume already asks for them. The view still returns has_photo and has_education. If I write Q7 off the view, I chase 209 students for a photo they have no way to upload. Q7 must follow the JS weights, not the view.
goal is three questions, one item. filled(p.goal) && filled(p.goal_3y) && filled(p.goal_5y) — a student who answered the nine-day goal and left the 3y/5y blank counts as not having done it. The named missing field should say so.
resume_v2 before Day 8. Today is Day 2. If Q7 lists "Build your final resume" as missing, that's 209 students chased for work that isn't open. I'll exclude it below Day 8, matching the app — confirm.
d) Venue / department
students.dept TEXT NOT NULL CHECK (dept IN ('ECE','EEE')) and teams.dept TEXT CHECK (...) — nullable on teams, not null on students.

They are stored independently and nothing enforces that a student's dept matches their team's. students.team_id is also nullable (ON DELETE SET NULL). For the venue column I'll use students.dept — it's the NOT NULL one and it's the student's own venue. I'll add a cross-check for students whose dept disagrees with their team's, and report the count rather than silently picking one.

e) Drive CVs
Six columns from 2026-09-17-b-cv-drive-links.sql: resume_v{1,2}_drive_{url,id,at}.

Telling a Drive CV from a server-only one is explicit, and the migration is emphatic about why both columns exist:

resume_v1_url — where the CV is now: either /uploads/resumes/<file> (server) or a Drive link the student pasted. Both allowed by chk_resume_v1_url.
resume_v1_drive_url — where scripts/migrate-cvs.js copied it, constrained to ^https://drive\.google\.com/.
The view already names the test: resume_v1_copied = (resume_v1_url IS NOT NULL AND resume_v1_drive_url IS NOT NULL) — "safely in two places".

Q8 "no CV on Drive" is ambiguous and I want your ruling. Three different lists:

No CV at all — resume_v1_url IS NULL. ~133 students. This is a chase list: they haven't handed anything in.
Handed in, not yet backed up — resume_v1_url IS NOT NULL AND resume_v1_drive_url IS NULL. The migration says this was 76 before migrate-cvs.js ran, 0 after. This is not a student chase list — the student did their bit; the script hasn't run. Chasing them is wrong.
A student who pasted a Drive link themselves: resume_v1_url is a Drive URL but resume_v1_drive_url is NULL. They do have a CV on Drive, but test 2 flags them anyway.
Since these are chase lists — people to phone — I read Q8 as list 1, with a separate count of list 2 reported as an operational note for you rather than as names to chase. Confirm.

The problem with Q1 and Q2: tasks and projects are team-level
This is the one that changes the shape of the deliverable, so I've stopped rather than guess.

Neither tasks nor projects have a student. task_submissions is UNIQUE (task_id, team_id); projects is one row per team per day; submissions hangs off project_id. The tasks migration is explicit: "a task belongs to the TEAM, so any member may hand in and the second hand-in replaces the first."

So "students who have not handed in today's task" cannot be answered literally. In a team of four where one member uploaded the bench photo, all four have handed in — or none of them personally did, depending on what you mean. What I can produce:

Every member of a team with no hand-in for today's task(s). Right for chasing: you ring the team, the lead, or all four. This is what I'll build unless you say otherwise.
Not: "students who personally didn't submit" — that would name three-quarters of every compliant team.
Two more wrinkles: a day can carry several tasks (tasks is one row per piece of work, with dept NULL meaning both venues), so "hasn't handed in today's task" means any outstanding task for their venue — I'll list which. And daily_posts is the one genuinely per-student thing — one row per student per day, the "what I learned" post. That may be what you actually want chased, and it isn't in your eight.

Also note Q1's scope: a student with team_id IS NULL has no team and so no hand-in, ever. They'd appear in Q1 and Q2 every single day. I'll report the count and flag them separately rather than let them pad the list.

Two findings I'm not acting on
src/db/migrations/readme.md lists 9 migrations; the directory has 16. The six from 18–19 Sep (assessment-survey, tinkercad-code, project-formats, project-open-per-dept, project-groups, project-view-by-group) have no documented run order. I don't know whether they're applied to production. This matters for me only in that my local DB must match yours — but it's worth someone's attention.
v_student_progress exposing has_photo/has_education after the weights dropped them is the kind of thing your rule 11 is about: the new code is right, the view underneath still answers the old question. Anything reading that view for "is this profile complete" is wrong today.
What I need from you
Q1/Q2 reading — confirm "every member of a team that hasn't handed in" is what you want.
Q8 reading — list 1 (no CV at all), with list 2 reported as a count, not names?
Q7 and resume_v2 — exclude below Day 8, matching the app?
Do you want daily_posts chased? It's the only per-student hand-in and it isn't in your eight. Say the word and I'll add it as a ninth — or leave it, and I won't widen scope.
Local DB — you said "loaded from a real dump." I don't have one and I won't ask for credentials or run the loaders (rule 5). Tell me where a dump is on this machine, or confirm you'll place one. I need to know which of the 16 migrations it has.
No branch cut, nothing written, nothing committed. Waiting.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/v3-build-and-cutover.md <<'ACCUT' # v3 — one build, one deploy  **]_

**Claude:** Agent did excellent work — that's exactly the reconnaissance the old brief couldn't have given it. Let me finish putting the docs in the repo, then answer it.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/v3-agent-brief.md <<'ACBRIEF' # Bootcamp dashboard — build brief f]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** All five docs are now in `docs/` in the repo — and I folded the agent's schema findings into the brief, so nothing has to be re-derived.

Now the reply to paste back to it:

```
Good report. The schema findings are now written into docs/v3-agent-brief.md
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
```

Two things for you, not the agent:

- **Place the dump.** The nightly one already exists on the server, so this costs production nothing:

```
mkdir -p ~/araCreate/dumps
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
ssh hetzner 'sudo cat /var/backups/bootcamp/<newest-file>' > ~/araCreate/dumps/bootcamp-2026-09-19.dump
ls -lh ~/araCreate/dumps/
```

- The agent found something real: **`v_student_progress` still reports `has_photo` and `has_education`** after those were dropped from the scoring. Anything reading that view for "is this profile complete" is wrong right now — including, possibly, a screen you are looking at today. Worth a check before the next chase round.

**Vishnu:** Last login: Thu Sep 17 10:10:10 on ttys000
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % mkdir -p ~/araCreate/dumps
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
ssh hetzner 'sudo cat /var/backups/bootcamp/<newest-file>' > ~/araCreate/dumps/bootcamp-2026-09-19.dump
ls -lh ~/araCreate/dumps/
total 40K
-rw-r--r-- 1 root root 19K Sep 17 03:05 pre-deploy-2026-09-17-0305.sql.gz
-rw-r--r-- 1 root root 19K Sep 17 03:25 pre-goals-2026-09-17-0325.sql.gz
bash: -c: line 1: syntax error near unexpected token `newline'
bash: -c: line 1: `sudo cat /var/backups/bootcamp/<newest-file>'
total 0
-rw-r--r--  1 vishnuvarthanvenkatapathy  staff     0B 19 Sep 15:14 bootcamp-2026-09-19.dump
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** `<newest-file>` was a placeholder — my fault, I should have said so. But the output shows something more important.

## Your backups are not running

- Only **two files**, both from **17 Sep**. Today is the 19th.
- Both are named `pre-deploy` and `pre-goals` — those are **manual dumps taken before deploys**, not nightly ones.
- `handover.md` says a `pg_dump` runs nightly at 01:00, 14 days kept. There is no evidence of it ever having run.

So right now, with 209 students two days into a live bootcamp, **your newest backup is from before v2 even shipped.** Everything since — Day 1, Day 2, every CV hand-in, every quiz answer — exists in one place only.

This is more urgent than the chase lists. Fix it first.

**1. Take a dump now** (read-only, tiny, safe to run while students work):

```
rm ~/araCreate/dumps/bootcamp-2026-09-19.dump
ssh hetzner 'sudo -u postgres pg_dump -d bootcamp | gzip' > ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
ls -lh ~/araCreate/dumps/
gunzip -t ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz && echo "gzip ok"
```

Expect it to be noticeably bigger than 19K. If it comes back at 19K, tell me — that would mean it is not what we think it is.

**2. Check it has the rows** — not just that it is valid gzip:

```
zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep -c "^COPY public"
zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep "^COPY public.students\|^COPY public.teams\|^COPY public.attendance"
```

**3. Find out why the nightly never ran:**

```
ssh hetzner 'sudo ls -la /etc/cron.d/'
ssh hetzner 'sudo crontab -l; echo ---; sudo -u postgres crontab -l'
ssh hetzner 'systemctl list-timers --all | grep -i -e backup -e dump -e bootcamp'
```

Paste what comes back and I will tell you what is wrong with it.

Once the dump lands, the agent can carry on with Step 2 — but get the backup fixed tonight, in the window, before anything else.

**Vishnu:** Last login: Thu Sep 17 10:10:10 on ttys000
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % mkdir -p ~/araCreate/dumps
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
ssh hetzner 'sudo cat /var/backups/bootcamp/<newest-file>' > ~/araCreate/dumps/bootcamp-2026-09-19.dump
ls -lh ~/araCreate/dumps/
total 40K
-rw-r--r-- 1 root root 19K Sep 17 03:05 pre-deploy-2026-09-17-0305.sql.gz
-rw-r--r-- 1 root root 19K Sep 17 03:25 pre-goals-2026-09-17-0325.sql.gz
bash: -c: line 1: syntax error near unexpected token `newline'
bash: -c: line 1: `sudo cat /var/backups/bootcamp/<newest-file>'
total 0
-rw-r--r--  1 vishnuvarthanvenkatapathy  staff     0B 19 Sep 15:14 bootcamp-2026-09-19.dump
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % rm ~/araCreate/dumps/bootcamp-2026-09-19.dump
ssh hetzner 'sudo -u postgres pg_dump -d bootcamp | gzip' > ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
ls -lh ~/araCreate/dumps/
gunzip -t ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz && echo "gzip ok"
total 256
-rw-r--r--  1 vishnuvarthanvenkatapathy  staff    79K 19 Sep 15:15 bootcamp-2026-09-19.sql.gz
gzip ok
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep -c "^COPY public"
zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep "^COPY public.students\|^COPY public.teams\|^COPY public.attendance"
zcat: can't stat: /Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz (/Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz.Z): No such file or directory
0
zcat: can't stat: /Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz (/Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz.Z): No such file or directory
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** Dump is good — **79K, gzip ok**. The `zcat` error is just a macOS quirk: BSD `zcat` looks for `.Z` files. Use `gzcat`.

```
gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep -c "^COPY public"
gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep "^COPY public\.\(students\|teams\|attendance\) "
```

And to actually count the rows, not just see the headers:

```
gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | awk '
  /^COPY public\./ {t=$2; c=0; f=1; next}
  f && /^\\\.$/   {print c, t; f=0; next}
  f               {c++}
' | sort -rn | head -20
```

You want to see roughly **209 students, 53 teams**, and attendance rows for Days 1–2.

Then the backup diagnosis:

```
ssh hetzner 'sudo ls -la /etc/cron.d/'
ssh hetzner 'sudo crontab -l; echo ---; sudo -u postgres crontab -l'
ssh hetzner 'systemctl list-timers --all | grep -i -e backup -e dump -e bootcamp'
```

The dump is on your Mac now, so the agent is unblocked either way — tell it the path is `~/araCreate/dumps/bootcamp-2026-09-19.sql.gz`.

**Vishnu:** Last login: Thu Sep 17 10:10:10 on ttys000
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % mkdir -p ~/araCreate/dumps
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
ssh hetzner 'sudo cat /var/backups/bootcamp/<newest-file>' > ~/araCreate/dumps/bootcamp-2026-09-19.dump
ls -lh ~/araCreate/dumps/
total 40K
-rw-r--r-- 1 root root 19K Sep 17 03:05 pre-deploy-2026-09-17-0305.sql.gz
-rw-r--r-- 1 root root 19K Sep 17 03:25 pre-goals-2026-09-17-0325.sql.gz
bash: -c: line 1: syntax error near unexpected token `newline'
bash: -c: line 1: `sudo cat /var/backups/bootcamp/<newest-file>'
total 0
-rw-r--r--  1 vishnuvarthanvenkatapathy  staff     0B 19 Sep 15:14 bootcamp-2026-09-19.dump
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % rm ~/araCreate/dumps/bootcamp-2026-09-19.dump
ssh hetzner 'sudo -u postgres pg_dump -d bootcamp | gzip' > ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
ls -lh ~/araCreate/dumps/
gunzip -t ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz && echo "gzip ok"
total 256
-rw-r--r--  1 vishnuvarthanvenkatapathy  staff    79K 19 Sep 15:15 bootcamp-2026-09-19.sql.gz
gzip ok
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep -c "^COPY public"
zcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep "^COPY public.students\|^COPY public.teams\|^COPY public.attendance"
zcat: can't stat: /Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz (/Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz.Z): No such file or directory
0
zcat: can't stat: /Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz (/Users/vishnuvarthanvenkatapathy/araCreate/dumps/bootcamp-2026-09-19.sql.gz.Z): No such file or directory
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep -c "^COPY public"
gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | grep "^COPY public\.\(students\|teams\|attendance\) "
23
COPY public.attendance (id, student_id, team_id, day, present, marked_by, marked_at, source, note) FROM stdin;
COPY public.students (id, roll_no, name, email, dept, year, phone, team_id, is_team_lead, discord_id, is_active, last_login, created_at) FROM stdin;
COPY public.teams (id, code, name, track_id, mentor_id, lead_student_id, discord_channel_id, drive_folder_url, project_points, quiz_points, total_points, created_at, dept, tinkercad_code, tinkercad_set_at, tinkercad_set_by) FROM stdin;
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % gzcat ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz | awk '
  /^COPY public\./ {t=$2; c=0; f=1; next}
  f && /^\\\.$/   {print c, t; f=0; next}
  f               {c++}
' | sort -rn | head -20
508 public.assessment_answers
419 public.attendance
209 public.students
208 public.student_profiles
159 public.projects
154 public.assessment_attempts
56 public.task_submissions
53 public.teams
50 public.task_submission_orphans
16 public.releases
10 public.mentors
9 public.quizzes
8 public.tasks
7 public.daily_posts
4 public.assessment_questions
3 public.tracks
1 public.settings
0 public.submissions
0 public.scores
0 public.quiz_questions
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** Dump is valid — **209 students, 53 teams**. The agent is unblocked: path is `~/araCreate/dumps/bootcamp-2026-09-19.sql.gz`.

But those row counts are telling you four things you should look at today.

## 1. Daily posts are basically not happening

- **7 posts.** Two days in, 209 students.
- The whole nine-day arc — showing each student their own notes on Day 9 — rests on this.
- Either it is not open, or nobody has been told. Check whether posting is released for both venues.

## 2. `task_submission_orphans` has 50 rows

- 56 real task submissions, **50 orphans**. Nearly one in two.
- That table exists because something was breaking the link between a submission and its task or team.
- Worth knowing whether those 50 are real student work that is not being counted.

## 3. `submissions` = 0 and `scores` = 0

- 159 projects exist. **Zero submissions, zero scores.**
- Either project hand-ins now write to a different table, or no project has been handed in or scored at all since v2 landed.
- If the leaderboard reads `scores`, it is ranking on nothing.

## 4. The assessment confirms the EEE gap

- 154 attempts. ECE is 154 students. **EEE's 55 have not done it** — exactly as `v2-build-prompts.md` said.
- Also only **4 questions** in the whole assessment.

## Smaller

- 208 profiles for 209 students — one student has no profile row.
- `quiz_questions` = 0, as expected. Still blocking every quiz.
- Attendance 419 over two days ≈ full marking. That part is working well.

## Still pending

The backup diagnosis. Your last real backup before today was 17 Sep:

```
ssh hetzner 'sudo ls -la /etc/cron.d/'
ssh hetzner 'sudo crontab -l; echo ---; sudo -u postgres crontab -l'
ssh hetzner 'systemctl list-timers --all | grep -i -e backup -e dump -e bootcamp'
```

Of the four, **#3 is the one I would check first** — if projects genuinely have no submissions or scores two days in, that is the core of the bootcamp not working, and no chase list will tell you because it is a schema question, not a student one.

**Vishnu:** vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % ssh hetzner 'sudo ls -la /etc/cron.d/'
ssh hetzner 'sudo crontab -l; echo ---; sudo -u postgres crontab -l'
ssh hetzner 'systemctl list-timers --all | grep -i -e backup -e dump -e bootcamp'
total 20
drwxr-xr-x  2 root root 4096 Sep 17 06:40 .
drwxr-xr-x 81 root root 4096 Sep 19 06:04 ..
-rw-r--r--  1 root root  166 Sep 16 17:59 bootcamp-backup
-rw-r--r--  1 root root  188 May  9 11:36 e2scrub_all
-rw-r--r--  1 root root  102 Jun 13  2025 .placeholder
no crontab for root
---
no crontab for postgres
Sun 2026-09-20 00:00:00 UTC       14h Sat 2026-09-19 00:00:04 UTC       9h ago dpkg-db-backup.timer         dpkg-db-backup.service
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** The cron file **exists** (`/etc/cron.d/bootcamp-backup`, created 16 Sep) but has never produced a file — the two dumps from the 17th are both manual pre-deploy ones. So the job is defined and failing silently.

```
ssh hetzner 'sudo cat /etc/cron.d/bootcamp-backup; echo "---END---"'
ssh hetzner 'sudo tail -c 20 /etc/cron.d/bootcamp-backup | xxd | tail -2'
ssh hetzner 'systemctl is-active cron; systemctl is-enabled cron'
ssh hetzner 'sudo journalctl -u cron --since "3 days ago" --no-pager | tail -40'
```

The four usual causes, in the order I would bet on:

1. **No trailing newline** on the cron.d file — cron silently ignores the last line. The `xxd` above checks this; you want the last byte to be `0a`.
2. **Missing user field.** Files in `/etc/cron.d` need a user column that `crontab -e` files do not: `0 1 * * * root /path/script.sh`. Without `root` there, it never runs.
3. **The script path is wrong or not executable.**
4. **`cron` is not running at all** — though `e2scrub_all` sits in the same directory, so probably it is.

Paste the output of the first command and I will tell you which one it is and give you the fixed file.

**Vishnu:** vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % ssh hetzner 'sudo cat /etc/cron.d/bootcamp-backup; echo "---END---"'
ssh hetzner 'sudo tail -c 20 /etc/cron.d/bootcamp-backup | xxd | tail -2'
ssh hetzner 'systemctl is-active cron; systemctl is-enabled cron'
ssh hetzner 'sudo journalctl -u cron --since "3 days ago" --no-pager | tail -40' 
0 1 * * * postgres pg_dump bootcamp | gzip > /var/backups/bootcamp/$(date +\%F).sql.gz
30 1 * * * root find /var/backups/bootcamp -name '*.sql.gz' -mtime +14 -delete
---END---
bash: line 1: xxd: command not found
active
enabled
Sep 19 01:00:01 aca-htz-vcet CRON[153616]: pam_unix(cron:session): session opened for user postgres(uid=101) by postgres(uid=0)
Sep 19 01:00:01 aca-htz-vcet CRON[153618]: (postgres) CMD (pg_dump bootcamp | gzip > /var/backups/bootcamp/$(date +%F).sql.gz)
Sep 19 01:00:01 aca-htz-vcet CRON[153616]: (CRON) info (No MTA installed, discarding output)
Sep 19 01:00:01 aca-htz-vcet CRON[153616]: pam_unix(cron:session): session closed for user postgres
Sep 19 01:17:01 aca-htz-vcet CRON[154060]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 01:17:01 aca-htz-vcet CRON[154062]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 01:17:01 aca-htz-vcet CRON[154060]: pam_unix(cron:session): session closed for user root
Sep 19 01:30:01 aca-htz-vcet CRON[154387]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 01:30:01 aca-htz-vcet CRON[154389]: (root) CMD (find /var/backups/bootcamp -name '*.sql.gz' -mtime +14 -delete)
Sep 19 01:30:01 aca-htz-vcet CRON[154387]: pam_unix(cron:session): session closed for user root
Sep 19 02:17:01 aca-htz-vcet CRON[155214]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 02:17:01 aca-htz-vcet CRON[155216]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 02:17:01 aca-htz-vcet CRON[155214]: pam_unix(cron:session): session closed for user root
Sep 19 03:10:01 aca-htz-vcet CRON[155771]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 03:10:01 aca-htz-vcet CRON[155773]: (root) CMD (test -e /run/systemd/system || SERVICE_MODE=1 /sbin/e2scrub_all -A -r)
Sep 19 03:10:01 aca-htz-vcet CRON[155771]: pam_unix(cron:session): session closed for user root
Sep 19 03:17:01 aca-htz-vcet CRON[155858]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 03:17:01 aca-htz-vcet CRON[155860]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 03:17:01 aca-htz-vcet CRON[155858]: pam_unix(cron:session): session closed for user root
Sep 19 04:17:01 aca-htz-vcet CRON[156599]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 04:17:01 aca-htz-vcet CRON[156601]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 04:17:01 aca-htz-vcet CRON[156599]: pam_unix(cron:session): session closed for user root
Sep 19 05:17:01 aca-htz-vcet CRON[158010]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 05:17:01 aca-htz-vcet CRON[158012]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 05:17:01 aca-htz-vcet CRON[158010]: pam_unix(cron:session): session closed for user root
Sep 19 06:17:01 aca-htz-vcet CRON[158891]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 06:17:01 aca-htz-vcet CRON[158893]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 06:17:01 aca-htz-vcet CRON[158891]: pam_unix(cron:session): session closed for user root
Sep 19 06:25:01 aca-htz-vcet CRON[158960]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 06:25:01 aca-htz-vcet CRON[158962]: (root) CMD (test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; })
Sep 19 06:25:01 aca-htz-vcet CRON[158960]: pam_unix(cron:session): session closed for user root
Sep 19 07:17:01 aca-htz-vcet CRON[160345]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 07:17:01 aca-htz-vcet CRON[160347]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 07:17:01 aca-htz-vcet CRON[160345]: pam_unix(cron:session): session closed for user root
Sep 19 08:17:01 aca-htz-vcet CRON[161048]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 08:17:01 aca-htz-vcet CRON[161050]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 08:17:01 aca-htz-vcet CRON[161048]: pam_unix(cron:session): session closed for user root
Sep 19 09:17:01 aca-htz-vcet CRON[163759]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Sep 19 09:17:01 aca-htz-vcet CRON[163761]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Sep 19 09:17:01 aca-htz-vcet CRON[163759]: pam_unix(cron:session): session closed for user root
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** Found it. The cron is fine — it **ran** at 01:00:01 today as `postgres`:

```
(postgres) CMD (pg_dump bootcamp | gzip > /var/backups/bootcamp/$(date +%F).sql.gz)
(CRON) info (No MTA installed, discarding output)
```

It ran, it failed, and **the error was thrown away** because there is no mail agent. Nothing ever told you.

The cause is almost certainly that **`postgres` cannot write into `/var/backups/bootcamp`** — the two existing files are owned by `root`, so the directory is root-owned too. The redirect fails, no file appears, silence.

## Confirm it in two commands

```
ssh hetzner 'sudo ls -ld /var/backups/bootcamp'
ssh hetzner 'sudo -u postgres sh -c "pg_dump bootcamp | gzip > /var/backups/bootcamp/permtest.sql.gz"; echo "exit=$?"'
```

You should see `Permission denied`.

## The fix

Safe to run now — backups are read-only and nothing restarts.

**1. Let postgres write there:**

```
ssh hetzner 'sudo chown postgres:postgres /var/backups/bootcamp && sudo chmod 750 /var/backups/bootcamp && sudo ls -ld /var/backups/bootcamp'
```

**2. Replace the one-liner with a script that cannot lie to you:**

```
ssh hetzner 'sudo tee /usr/local/bin/bootcamp-backup.sh >/dev/null <<'"'"'EOF'"'"'
#!/bin/sh
set -eu
DIR=/var/backups/bootcamp
OUT="$DIR/$(date +%F-%H%M).sql.gz"
TMP="$OUT.tmp"
pg_dump bootcamp | gzip > "$TMP"
gzip -t "$TMP"
mv "$TMP" "$OUT"
echo "$(date -Is) ok $(wc -c < "$OUT") bytes $OUT"
EOF
sudo chmod 755 /usr/local/bin/bootcamp-backup.sh'
```

Why it matters: it writes to `.tmp`, **verifies the gzip**, and only then renames. A half-finished dump can never look like a good backup.

**3. Point cron at it and keep the output:**

```
ssh hetzner 'sudo tee /etc/cron.d/bootcamp-backup >/dev/null <<EOF
0 1 * * * postgres /usr/local/bin/bootcamp-backup.sh >> /var/log/bootcamp-backup.log 2>&1
30 1 * * * root find /var/backups/bootcamp -name "*.sql.gz" -mtime +14 -delete
EOF
sudo touch /var/log/bootcamp-backup.log && sudo chown postgres /var/log/bootcamp-backup.log'
```

**4. Prove it works — do not wait until 01:00:**

```
ssh hetzner 'sudo -u postgres /usr/local/bin/bootcamp-backup.sh'
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

You want a new file around **79K**, matching the dump on your Mac. If it comes back near 19K, stop and tell me.

**5. Tomorrow morning, check it actually fired:**

```
ssh hetzner 'sudo cat /var/log/bootcamp-backup.log'
```

## Two notes

- The 14-day cleanup line **has** been running correctly as root. Nothing was deleted wrongly — there was simply nothing to delete.
- This is the same pattern as the eight v2 bugs: the new thing was written correctly, and the layer underneath it silently refused. Worth adding "a backup job that has never produced a file" to the v3 gate list.

**Vishnu:** vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % (postgres) CMD (pg_dump bootcamp | gzip > /var/backups/bootcamp/$(date +%F).sql.gz)
(CRON) info (No MTA installed, discarding output)
zsh: parse error near `CMD'
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** That was log output I was quoting back at you, not a command — nothing to run there. No harm done, zsh just refused it.

Run these, in order.

**1. Confirm the cause:**

```
ssh hetzner 'sudo ls -ld /var/backups/bootcamp'
```

```
ssh hetzner 'sudo -u postgres sh -c "pg_dump bootcamp | gzip > /var/backups/bootcamp/permtest.sql.gz"; echo exit=$?'
```

Expect `Permission denied` and a non-zero exit.

**2. Fix the permission:**

```
ssh hetzner 'sudo chown postgres:postgres /var/backups/bootcamp && sudo chmod 750 /var/backups/bootcamp && sudo ls -ld /var/backups/bootcamp'
```

**3. Test it immediately:**

```
ssh hetzner 'sudo -u postgres sh -c "pg_dump bootcamp | gzip > /var/backups/bootcamp/2026-09-19.sql.gz"; echo exit=$?'
```

```
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

You want a new file around **79K** and `exit=0`.

Stop there and paste the output. If that works, the nightly cron will start working on its own tonight with no further change — the script and logging in my last message are an improvement, not a requirement, and we can do those in the night window.

**Vishnu:** vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % ssh hetzner 'sudo ls -ld /var/backups/bootcamp'
drwxr-xr-x 2 root root 4096 Sep 17 03:25 /var/backups/bootcamp
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** Confirmed. `drwxr-xr-x root root` — postgres can read the directory but **cannot write to it**. That is the whole bug. The cron has been running every night and failing at the redirect, silently, since 16 September.

Fix it:

```
ssh hetzner 'sudo chown postgres:postgres /var/backups/bootcamp && sudo chmod 750 /var/backups/bootcamp && sudo ls -ld /var/backups/bootcamp'
```

Then prove it:

```
ssh hetzner 'sudo -u postgres sh -c "pg_dump bootcamp | gzip > /var/backups/bootcamp/2026-09-19.sql.gz"; echo exit=$?'
```

```
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

Want: `exit=0` and a new file around **79K**.

**Vishnu:** drwxr-x--- 2 postgres postgres 4096 Sep 17 03:25 /var/backups/bootcamp
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % ssh hetzner 'sudo -u postgres sh -c "pg_dump bootcamp | gzip > /var/backups/bootcamp/2026-09-19.sql.gz"; echo exit=$?'
exit=0
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ % ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
total 120K
-rw-rw-r-- 1 postgres postgres 80K Sep 19 09:49 2026-09-19.sql.gz
-rw-r--r-- 1 root     root     19K Sep 17 03:05 pre-deploy-2026-09-17-0305.sql.gz
-rw-r--r-- 1 root     root     19K Sep 17 03:25 pre-goals-2026-09-17-0325.sql.gz
vishnuvarthanvenkatapathy@Vishnuvarthans-MacBook-Air ~ %

**Claude:** Fixed. **80K, exit=0**, owned by postgres. Tonight's 01:00 cron will now work on its own — no further change needed.

**Claude:**   _[tool: Projects — claude/ops-findings-19-sep.md]_

**Claude:** All of it is written up in `claude/ops-findings-19-sep.md`.

## Done

- Backups fixed. First real nightly will land at 01:00 tonight.
- Between 17 and 19 Sep your only backup predated v2 entirely. Two days of live bootcamp existed in one place. That is closed now.

## Check tomorrow morning

```
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

You want a `2026-09-20` file around 80K. If it is not there, tell me.

## Two small improvements for the night window

Not urgent, but worth doing before v3: write to `.tmp`, run `gzip -t`, then rename — so a half-finished dump can never look like a good one. And log the output somewhere, because tonight it still goes nowhere. The commands are in the findings doc.

## Back to the agent

It is unblocked. Send it:

```
Dump is on the Mac: ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz

It is a plain gzipped SQL dump (not -Fc), 80K, verified. Taken 19 Sep 15:15 IST,
Day 2. Row counts: 209 students, 53 teams, 419 attendance, 159 projects,
56 task_submissions, 50 task_submission_orphans, 7 daily_posts,
154 assessment_attempts, 0 submissions, 0 scores, 0 quiz_questions.

Load it and report which of the 16 migrations it contains, before writing any
query. Then proceed to Step 2 with the nine queries as agreed.

Flag if the zero in `submissions` and `scores` means the queries need to read a
different table than you expected.
```

The thing I would actually want to know tonight: **159 projects, zero submissions, zero scores.** If that is real rather than a table that moved, no project has been handed in or marked since v2 shipped, and the leaderboard is ranking on nothing.

**Vishnu:** ok o have a question what do need to chcek to orrwo morning and what chrone is runnung

**Claude:** Two answers.

## What cron is running

**Your backup job** — `/etc/cron.d/bootcamp-backup`, two lines:

| When (UTC) | When (IST) | As | What |
|---|---|---|---|
| 01:00 | **06:30** | postgres | `pg_dump bootcamp \| gzip > /var/backups/bootcamp/<date>.sql.gz` |
| 01:30 | **07:00** | root | delete `*.sql.gz` older than 14 days |

**Important: the server runs on UTC, not IST.** So your "1 AM nightly backup" actually fires at **6:30 in the morning your time**. Don't look for it before then.

**Debian's own jobs** — nothing to do with your app, leave them alone:

- `/etc/cron.hourly` at :17 past every hour
- `/etc/cron.daily` at 06:25 UTC
- `e2scrub_all` at 03:10 UTC — filesystem check
- `dpkg-db-backup.timer` at 00:00 UTC — systemd, backs up the package database

**No root crontab, no postgres crontab.** Everything is in `/etc/cron.d`.

## What to check tomorrow morning

**After 7 AM IST — the backup (30 seconds):**

```
ssh hetzner 'sudo ls -lh /var/backups/bootcamp'
```

- A file named `2026-09-20.sql.gz` must be there.
- Size around **80K or bigger**. Bigger is fine and expected — more data each day.
- **If it is missing, or it is 19K, tell me.** Missing means the fix did not hold. 19K means it dumped the wrong thing.

**Then your normal Day 3 routine:**

- Open today's activities **per venue** — nothing opens itself.
- **Quiz questions.** All nine day-quizzes still have 0. Nothing opens below five. Only you can write these.
- **Open the pre-assessment for EEE.** Yesterday's dump confirmed it: 154 attempts, which is exactly the ECE cohort. EEE's 55 have never had it.
- Re-run the CV migration for stragglers.

**And the one real question from the dump:**

- 159 projects, **0 submissions, 0 scores**. Ask the agent about this first thing. If it is real, no project has been handed in or scored since v2 shipped and the leaderboard is ranking on nothing.

**Vishnu:** ok save all will the agnet run non stop or do i need to be in fromt of the mac

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/ops-findings-19-sep.md <<'ACOPS' # Ops findings — 19 Sep 2026 (Day]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:**   _[tool: Projects — claude/ops-findings-19-sep.md]_

**Claude:** Saved in both places — the Claude project and `docs/ops-findings-19-sep.md` in the repo.

## Will the agent run non-stop?

**No — not as you have set it up, and that was your own decision.** You chose "report at the end of each feature." So it builds one feature, writes the report, and **stops and waits for you**.

Three separate things can stop it:

1. **The report gate.** By design. End of feature, it stops.
2. **Tool approvals.** Unless auto-accept is on, it pauses on every file write and shell command.
3. **The Mac sleeping.** If the laptop sleeps or the terminal closes, it dies mid-step.

So right now: yes, you need to be there.

## If you want it to run while you sleep

The safety net is already strong — **it cannot deploy, cannot ssh to the server, cannot touch the production database.** Worst case is a messy local branch you delete. So a long unattended run is less risky than it sounds.

Three things to change:

- **Stop the Mac sleeping.** Run the agent under `caffeinate -i` — then the lid staying open is the only requirement.
- **Turn on auto-accept for edits** so it does not pause on every file write. It still cannot run anything the rules forbid.
- **Give it a whole phase, not one feature.** E.g. "Do all nine chase queries, report once at the end" instead of stopping after each.

## My honest recommendation

- **Tonight: stay with report-per-feature.** The agent has never worked on this repo before, and its first report was very good precisely because it stopped and asked instead of guessing. Let it earn the longer leash.
- **Once Track 1 is done and you have read one full report**, switch to phase-at-a-time with `caffeinate` and auto-accept for the v3 work. That is where the long unattended runs actually pay off — Phase A3, the module split, is hours of mechanical work.

One correction to make now, though: the brief says "one feature at a time." If you later want longer runs, tell me and I will update `docs/v3-agent-brief.md` so the rules and how you actually work do not drift apart.

**Vishnu:** it need to run withou tme plan for tht

**Claude:** You've decided — so let me make unattended running actually safe rather than just possible.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/unattended-operation.md <<'ACUNATT' # Unattended operation — how t]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** Four new files in `docs/`. Here is how it runs without you.

## The key idea

**Questions never stop the run.** That was the thing that would otherwise waste your night. The agent writes the question down, marks the item BLOCKED, and moves to the next one. A night that clears eight of ten items is a good night; a night stuck on item three is a wasted one.

## It works a written queue

`docs/work-queue.md` — **27 items already written out**, in order. Track 1's nine queries plus the wrapper and verification, then the whole survey feature. That is more than one night of work, so it never runs dry.

It takes an item, builds it, tests it, commits, pushes, logs it, ticks it, takes the next. No permission asked.

## It stops completely for only five things

1. It would need to touch production
2. A data-loss risk it cannot avoid
3. A verification gate fails in a way suggesting the data is wrong
4. The same item fails three times
5. The queue is empty

Everything else is skip-and-continue.

## Why this is safe

The agent **cannot deploy, cannot ssh to the server, cannot touch the production database.** The worst possible outcome of a bad overnight run is a messy local branch you delete in the morning. Nothing it does can reach the 209 students.

## Your morning, three files

- `docs/agent-log.md` — what happened, newest first. Any STOP is the first line.
- `docs/questions-for-vishnu.md` — what is blocked and needs you.
- `docs/work-queue.md` — done, blocked, left.

Every log entry has an **OLD PATH** line that cannot be skipped. That is the line that found the two worst bugs in the v2 build.

## Launching it tonight

```
cd ~/araCreate/bootcamp-dashboard
git status                 # must be clean
git branch --show-current  # must NOT be main
caffeinate -i &
```

Lid open, power connected. Turn on auto-accept for edits, then:

```
Work docs/work-queue.md from the top, following docs/unattended-operation.md.
Do not wait for me. Report into docs/agent-log.md after every item.
Questions go to docs/questions-for-vishnu.md — never guess, never stop the run
for a question. Commit and push after every item.
```

## The one rule that makes this work

**Only put work in the queue whose decisions are already made.** Everything in there now is decided. The moment you queue something still being argued about, the agent either guesses or blocks — and both waste the night.

Two honest costs: a wrong assumption now runs for hours instead of minutes, and reviewing a night's work takes about twenty minutes in the morning, not two.

**Vishnu:** no not night now itself

**Claude:** Nothing stops it starting now. The agent never deploys, never touches the server — students are unaffected by local work. The night rule was only ever about **deploys**, and the agent has none.

Run this. **Stop and tell me if the first command does not say `main`.**

```
cd ~/araCreate/bootcamp-dashboard
git branch --show-current
git status --short
```

If it says `main`:

```
git add docs/
git commit -m "docs: v3 plans, agent brief, unattended operation, work queue"
git push
git checkout -b chase-lists
git push -u origin chase-lists
```

Keep the Mac awake if you are walking away:

```
caffeinate -i &
```

Then turn on auto-accept for edits and give the agent this:

```
Start now. Work docs/work-queue.md from the top, following
docs/unattended-operation.md. Read docs/v3-agent-brief.md first — Part 2 has the
schema facts you established, do not re-derive them.

You are on branch chase-lists. The dump is at
~/araCreate/dumps/bootcamp-2026-09-19.sql.gz — plain gzipped SQL, 80K, taken
19 Sep 15:15 IST (Day 2).

Do not wait for me. Report into docs/agent-log.md after every item. Questions go
to docs/questions-for-vishnu.md — never guess, never stop the run for a question,
mark the item BLOCKED and move to the next one. Commit and push after every item.

Answers already given, do not re-ask:
- Q1/Q2: every member of a team that has not handed in. team_id IS NULL is a
  separate flagged count.
- Q8: no CV at all only. Not-yet-copied-to-Drive is a count, never names.
- Q7: exclude resume_v2 below Day 8. JS weights only, never v_student_progress.
  Name which of goal/goal_3y/goal_5y is missing.
- Q9 daily_posts: yes, added as the ninth query. Authorised.
- run.sh takes the day explicitly and prints the computed default.
- Q6 carries a column for whether the register was open for that venue.
- dept mismatch between student and team: report as a count.
```

One thing worth watching for in the first hour: **T1-02 tells you which of the 16 migrations the dump actually has.** If that comes back short, the local database does not match production and every query after it is built against the wrong schema. Check that one before you stop looking.

**Vishnu:** tell me step by step do i  need ro stop the dev agent

**Claude:** **No. Do not stop it.** It is idle and waiting, which is exactly the state you want. Stopping it loses everything it learned about your schema.

Also: **do not run the git commands yourself.** Two people in one repo is what broke v2. Let the agent do its own git.

## Step by step

**Step 1.** Open a **separate** terminal tab — not the agent's — and keep the Mac awake:

```
caffeinate -i
```

Leave that tab running. Lid open, power connected.

**Step 2.** Go back to the agent's window. Turn on **auto-accept for edits** (in Claude Code, press `Shift+Tab` until it says auto-accept edits on).

**Step 3.** Paste this to the agent as-is:

```
Good report. Everything you found is now written into docs/v3-agent-brief.md
Part 2, so it never has to be re-derived.

The docs you could not find now exist in the working tree, untracked:
  docs/v3-agent-brief.md          <- the authority, read this first
  docs/unattended-operation.md    <- how you run without me, read this second
  docs/work-queue.md              <- your queue, 27 items
  docs/agent-log.md               <- you write here after every item
  docs/questions-for-vishnu.md    <- you write here instead of asking me
  docs/v3-decisions.md, docs/v3-restructure-plan.md,
  docs/v3-build-and-cutover.md, docs/agent-working-rules.md,
  docs/ops-findings-19-sep.md

YOUR GIT STEPS, do these yourself:
  1. git checkout -b chase-lists
  2. git add docs/ && git commit -m "docs: v3 plans, brief, queue"
  3. git push -u origin chase-lists

THEN START, and do not wait for me.
Work docs/work-queue.md from the top. Report into docs/agent-log.md after every
item. Questions go to docs/questions-for-vishnu.md — never guess, never stop the
run for a question: mark the item BLOCKED and move to the next one. Commit and
push after every item.

The dump is at ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz — plain gzipped SQL,
80K, taken 19 Sep 15:15 IST, Day 2. Not -Fc.

ANSWERS TO YOUR FIVE QUESTIONS, do not re-ask:
1. Q1/Q2 — yes, every member of a team that has not handed in. Include the
   task/project title. team_id IS NULL students: separate flagged count, never
   padded into the lists.
2. Q8 — list 1 only, resume_v1_url IS NULL. Not-yet-copied-to-Drive is an
   operational count in the log, never names. A student who pasted their own
   Drive link must not appear at all.
3. Q7 — yes, exclude resume_v2 below Day 8. JS weights only, never
   v_student_progress. For goal, name which of goal/goal_3y/goal_5y is missing.
4. daily_posts — yes, added as query 9. Authorised, not scope creep.
5. Dump — placed, path above. Report which of the 16 migrations it contains
   before writing any query.

Your three own proposals: all agreed. run.sh takes the day explicitly and prints
the computed default. Q6 carries a column for whether the register was open for
that venue. dept mismatch reported as a count.

Your two findings are logged for Track 3 and are not yours to fix now.
```

**Step 4.** Walk away. Check back in an hour or two.

## When you come back, read three files

```
cd ~/araCreate/bootcamp-dashboard
git log --oneline chase-lists | head -20
cat docs/questions-for-vishnu.md
head -60 docs/agent-log.md
```

## The one thing to look at first

**T1-02** — which migrations the dump contains. If it comes back short of 16, your local database does not match production, and every query built after it is wrong. That is the only early failure that silently poisons the whole night.

**Vishnu:** will the end to end dev will happen

**Claude:** Honest answer: **partly.** Here is exactly what will and will not happen.

## What WILL happen without you

- **Track 1 — chase lists.** Fully. Nine queries, the wrapper, the cross-check verification. Usable the moment it finishes.
- **Track 2 — the survey.** Built end to end: migration, release rows, admin loader, Open tab control, student card, results screen, venue-leak test, session tests at 209 students.

That is genuinely complete development for both.

## What will NOT happen

**1. Deploy.** By design — your own rule. The survey will sit finished on a branch until you deploy it at night. The agent writes the exact commands; you run them.

**2. The v3 rebuild.** Not in the queue. It is roughly five weeks, and Phase C3 is a hard stop where you approve the look of three screens before 20+ more get built. Queuing it now means the agent builds everything twice.

**3. Content.** Quiz questions and survey questions. No agent can write those — they come from what was taught that day.

**4. Anything still undecided.** Four things will hit `questions-for-vishnu.md` and block their items:

- What the survey questions are actually for — you said you had more to tell me
- Can a team lead see which of his six members finished
- What counts as "done" for a task with no hand-in type
- Absent vs not-yet-marked as two colours

**5. Looking at it.** The agent cannot judge whether the survey card looks right on a phone. It will write tests that pass while the screen looks wrong.

## The honest risk

The `submissions = 0, scores = 0` finding is unresolved. If project hand-ins moved to a different table, **query 2 may be built against a dead one** — and it will pass its own cross-check, because both queries would read the same wrong place. The agent may catch it. It may not.

## Three things that would close most of the gap

1. **Answer the four open questions now**, before you walk away. Then nothing blocks.
2. **Add a screenshot step** to Track 2 — Playwright saves phone-width PNGs of every new screen into `docs/screens/`, so you judge the look in the morning instead of reading code.
3. **Queue Track 3 Phase A after Track 2** — the module split is hours of mechanical work with no decisions in it. Perfect unattended work, and it cannot be built twice.

Want me to do those three now? It is about ten minutes and it turns one good night into a genuinely end-to-end one.

**Vishnu:** ask me questiosn

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** That changes the design — and makes it much more valuable. It is a **before/after knowledge check per topic**, i.e. proof of learning gain.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Before only — and the "after" comes from the quiz. So the proof is: *before teaching, 12 said they knew this; after teaching, 48 got it right.*

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Let me show it with a real example.

**Morning — survey (before teaching):**

> Do you know how to use a multimeter? — **Yes / No**
> 12 students say Yes.

**Evening — quiz (after teaching):**

> What does a multimeter measure? A/B/C/D
> 48 students get it right.

You want the report to say: **"Multimeter: 12 before → 48 after."**

The problem: the app has no way to know that the survey question and the quiz question are about the **same thing**. To a computer they are just two unrelated rows of text. Something has to tell it they belong together.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Now it is clear — and it solves the pairing problem by itself.

- **Daily survey** — before each day's topic. "Do you know X?" Yes/No.
- **Final survey** — at the end of the whole bootcamp, opened by you. The same questions asked again.

Because it is the *same questions*, the app matches them automatically. No topic tagging needed.

**Claude:**   _[tool: AskUserQuestion]_

**Vishnu:** All 17 checks pass, tree clean, everything pushed.
FEATURE: `scripts/chase/` — nine read-only chase queries, a runner writing one CSV per query with a venue column, a verifier proving every count a second way, plus `scripts/load-local-dump.sh`.
BRANCH/COMMITS: `chase-lists`, pushed yes.
`4bd7c9c` docs · `fc8ac28` docs · `a37722e` feat: chase lists · `aa49977` docs: queue/log · `29b3ef2` fix: flags display
DONE:

* Read-only — no write keyword in any query file, and the DB refuses an `UPDATE` in the runner's session
* Local dump loaded; all 16 migrations present (14 by column/table marker, the 2 view-only ones by reading `v_team_projects`'s own definition)
* All nine queries, every count cross-checked a second way — agree on days 1, 2, 3 and 8
* Under two seconds: 81 runs, slowest 7.6 ms
* Tested at real volume — 206 chaseable students, 53 teams, both venues
* One CSV per query with a venue column; every row leads with name · roll · team · venue · phone · personal email
* `team_id IS NULL` and dept-mismatch reported as counts (both 0)
* Old path checked
* Track 2 not started — BLOCKED, see below

OLD PATH: This is where it mattered most. `v_student_progress` still exposes `has_photo` and `has_education` after [profile-completion.js](vscode-webview://0iuirl8fi5rpmba1pv0mps2qijtpb1tbiupphaujdvevhh0gr77n/src/routes/profile-completion.js#L44) dropped both from the weights. Building list 7 from that view — the obvious way — would have named 200 of 206 students for two items the profile page no longer offers, every day, with no way to clear them. Query 7 reads the JS weights instead; I ran the app's own `completion()` against all 206 students on days 2 and 8 and got zero disagreement, including the Day 8 switch where `resume_v2` starts counting and the total moves 85→100.
Also checked: attendance has only two write paths, neither ever deletes — which is what makes lists 5 and 6 genuinely different. And `submissions` is still the project table; project-formats widened its constraint rather than replacing it.
FOUND:

* Your `submissions`/`scores = 0` flag — not a wrong table. All 159 project rows are `status='assigned'`, `is_open=false`, and every day-2 project release is `is_open=f`. No project has ever been opened, so nobody could hand in or score one. List 2 correctly returns zero. That's a "nothing was opened" finding, not a schema one.
* `task_submission_orphans`: 50 unclaimed Drive files from 27 teams against Day 1's task 8 — real work, chased by nobody.
* 36 CVs await their Drive copy, not the 76 the migration mentions — `migrate-cvs.js` has run since. A count, never names.
* `personal_email` is empty for all 209, so that column is blank in every CSV. Worth knowing before mailing a list.
* Queries 3/4 had no real data (0 quiz questions), so I proved them in a rolled-back transaction: 121 ECE / 0 EEE never opened, 20 unsubmitted. Exactly as predicted.

RISK: Nothing deploys; worst case is a wrong phone list. The live risk is misreading an empty CSV — on this dump the register is closed for both venues and no quiz has a question, so lists 3, 4 and 6 are empty or total through nobody's fault. `00-flags.sql` prints first and says which. The one judgement call is excluding three seeded test accounts; that exclusion is what makes ECE read 151, matching your figure.
DEPLOY: None — read-only. To use:

```sh
./scripts/load-local-dump.sh ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
./scripts/chase/run.sh 2        # CSVs into ./chase-lists/
./scripts/chase/verify.sh 2     # all 17 checks must read agree/holds

```

NEEDED: One decision, in [docs/questions-for-vishnu.md](vscode-webview://0iuirl8fi5rpmba1pv0mps2qijtpb1tbiupphaujdvevhh0gr77n/docs/questions-for-vishnu.md). I marked all of Track 2 BLOCKED rather than starting it: D14 wants the same question tracked across days, but `v3-decisions.md` still lists the survey's purpose as open. Matching on question text breaks the first time a question is reworded — near-certain across nine days written the evening before. My recommendation is an optional stable `key` prefix in the paste-many loader, blank meaning untracked. Track 2 ships to the live app, so guessing its data model overnight is expensive to undo — that's a question, not an assumption. Everything else in Track 2 is decided and buildable the moment you answer.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** That report is exactly right, and its one blocker is now answerable — your last few answers solved it. But three findings need you **before** the survey.

## 1. No project has ever been opened

All 159 projects are `is_open=false`. Not a schema bug — **nobody has opened a single project for either venue since the bootcamp started.** Two days in, no team can hand in project work, and no mentor can score anything.

That is the core of the bootcamp not running. Fix it on the Open tab today.

## 2. 50 orphan Drive files from 27 teams

Real student work, handed in against Day 1's task, **linked to nothing and counted by nobody.** More than half of all task submissions.

## 3. `personal_email` is empty for all 209 students

Every chase CSV will have a blank email column. **Phone only.** Do not plan a mailing list.

Also worth noting: its OLD PATH check caught the `v_student_progress` trap. Building query 7 the obvious way would have named **200 of 206 students** every day for a photo and education fields the profile page no longer offers. That one check paid for the whole rule.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/survey-spec.md <<'ACSURV' # Daily survey — full specification  Ans]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** `docs/survey-spec.md` is in the repo. Paste this to the agent:

```
Excellent report, and you were right to block rather than guess. The answer
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
```

One thing for you, not the agent: **open the projects today.** Two days in, nothing has been opened, so no team can hand in and no mentor can score. That is worth more than anything in the queue.

**Vishnu:** leave that to admin they will take care fo thse

**Claude:** Understood — admin's job, not mine to chase further.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/work-queue.md <<'ACQ2' # Work queue  The agent works this file fro]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** Queue rewritten. Track 1 marked done; the survey now has 18 items, and Track 3 Phase A is queued behind it so the agent never runs dry.

Two things I added beyond what you asked:

- **T2-17 — screenshots.** Playwright saves every new screen at 390px into `docs/screens/`. You judge the look in the morning by opening PNGs, not by reading code. This was the biggest gap in unattended running.
- **Track 3 Phase A** — fake-data generator, session test harness, the module split, the migration ledger, and the `v_student_progress` fix. All mechanical, all decided, no user-visible change. Ideal unattended work, and none of it can be built twice.

Projects, orphan files and the CV backlog are logged in the queue under "not to be fixed by the agent" so it does not wander into them.

Add one line to the message you paste:

```
docs/work-queue.md has been rewritten. Track 1 is DONE. Re-read it — Track 2 now
has 18 items including the four proof reports and a screenshot step, and Track 3
Phase A is queued behind it. Keep going into Phase A without asking.
```

The one thing still not covered for a fully unattended night: the agent cannot tell whether a screen *looks* right, only that it renders. The screenshots narrow that to a five-minute morning review rather than removing it.

**Vishnu:** now will it run fully wihtou me ior now alos it will ask questions

**Claude:** Honest answer: **it will run much further, but not question-free yet.** I can count about five questions it is likely to hit in Track 2. Three of them I can decide for you with a sensible default. Two are genuinely yours, and one of them affects whether your proof is trustworthy.

Two minutes now and the night is clean.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: Bash — cat >> /mnt/user-data/outputs/docs/survey-spec.md <<'ACSURV2'  ---  ## 8. Decisions that were likely]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** Eight likely questions are now pre-answered in `docs/survey-spec.md` section 8 — the three you decided, plus five I defaulted (where the loader lives, survey titles, late joiners, unanswered questions, and whether anything closes itself).

## So: will it run without you now?

**Track 2 — yes, almost certainly.** Every decision it needs is written down.

**Track 3 Phase A — mostly.** One item can still block: **T3-A5**, fixing `v_student_progress`. The brief tells it to find every reader and report before fixing, so if a reader turns out to depend on those columns it will write a question and move on rather than guess. That is correct behaviour, not a failure.

**What it still cannot do alone:**

- Judge whether a screen *looks* right. The screenshots narrow that to a five-minute morning review.
- Deploy. By design.
- Write your survey questions.

## One flag on your "first tap is locked" choice

It is the stricter option and it is defensible — but a mis-tap on a phone is permanent, and you will get students at the desk. I have written into the spec that the screen must say **"your first answer is saved, you cannot change it"** *before* the first question, not after. That turns a complaint into a warning they were given.

Paste this to the agent:

```
docs/survey-spec.md section 8 is new — eight decisions pre-answered so you never
have to stop for them. docs/work-queue.md has also been rewritten: Track 1 DONE,
Track 2 now 18 items, Track 3 Phase A queued behind it. Keep going into Phase A
without asking.
```

**Vishnu:** is phase a big

**Claude:** Yes — and one item is most of it.

| Item | Size | Notes |
|---|---|---|
| A1 fake-data generator | **Small–medium** | A few hours. Mechanical, no decisions |
| A2 session test harness | **Medium** | Half a day. High value — this is the safety net for everything after |
| **A3 module split** | **Large** | **A day or more on its own.** `server.js` is 3,000+ lines and is the entire API |
| A4 migration ledger | **Small** | An hour |
| A5 `v_student_progress` fix | **Small**, may block | The fix is easy; finding every reader is the work |

**A3 is roughly 70% of Phase A.**

## Two things about A3 that matter

**1. It has no safety net until A2 is genuinely good.** The existing 71 Playwright checks are the ones that passed while every team lead was locked out — because none of them signed in. Splitting a 3,000-line API with that suite as your only check is not safe. A2 must land properly first. The queue order already does this, but it is the thing to verify in the morning before letting A3 run.

**2. It should not be one queue item.** A single overnight commit that moves 3,000 lines is unreviewable — you cannot tell a clean move from a subtle behaviour change. It should be **12 items, one module at a time**, each committed separately with the suite green before and after. Then a bad move is one small revert, not a lost night.

## My recommendation

- Let it run **A1, A2, A4, A5 tonight**. All safe, all mechanical, and A2 is the one that makes everything after it trustworthy.
- **Hold A3** until you have read A2's report and seen the tests actually sign in. Then let it run as 12 small items.

Want me to rewrite the queue that way — A3 split into one item per module, and marked HOLD until you release it?

**Vishnu:** so it need to run tll a5 now qithoud askung questions

**Claude:** Then let me remove every reason it could stop — including reordering so the big one runs last.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/phase-a-decisions.md <<'ACPA' # Phase A — decisions pre-answered  ]_

**Claude:**   _[tool: Bash — python3 - <<'PY' import re p='/mnt/user-data/outputs/docs/work-queue.md' s=open(p).read() new_a = ""]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** Done. Two changes.

## Order flipped so nothing blocks

**A1 → A2 → A4 → A5 → A3.** The module split now runs **last**, not third. The small safe items get banked first, and A3 only starts once the session test harness exists to catch it.

## A3 is now 12 items, one per module

`auth → students → teams → venues → releases → activities → submissions → scoring → attendance → reports → storage → programs`

Each one committed and pushed on its own, suite green before and after. A bad move is one small revert, not a lost night. And if a module will not move cleanly, it logs it and takes the next one instead of stalling.

## Every Phase A decision pre-answered

`docs/phase-a-decisions.md` covers the four things it would otherwise have asked:

- **A1** — invented names and phone numbers only, never real students' details in a committed fixture. Seeded so it is repeatable.
- **A2** — read the staff password from the local `.env`, never print it, never ask. If absent, set a local-only one and say so. Join code is `ARA2026`.
- **A4** — documentation only. Change no migration, run nothing.
- **A5** — **do not drop the view columns.** Find the readers, fix the completeness ones, mark the columns deprecated. Dropping them overnight breaks whatever silently depends on them.

Paste to the agent:

```
docs/phase-a-decisions.md is new — every Phase A decision is pre-answered, read
it before starting Phase A and do not stop to ask.

docs/work-queue.md is updated. Phase A order is now A1, A2, A4, A5, then A3.
A3 is 12 items, one module each, and runs last.

Run straight through Track 2 and Phase A without stopping.
```

The one thing I would still check in the morning before A3 gets far: **open A2's report and confirm the tests genuinely sign in.** If that harness is weak, the module split has no safety net and every item after it is guesswork.

**Vishnu:** now will it run till a5 and tell me how muhc time it will take

**Claude:** Honest estimate. These assume it runs continuously with auto-accept on, and that things mostly work first time.

## Track 2 — the survey

| Items | Hours |
|---|---|
| Migration, release rows, loader, open tab (T2-01→04) | 2 |
| Student card, answering, final round (T2-05→07) | 2 |
| Daily results + four proof reports (T2-08→12) | 3–4 |
| Not-answered list, leak test, session tests, old path (T2-13→16) | 2.5–3 |
| Screenshots, deploy commands (T2-17→18) | 1 |
| **Track 2 total** | **10–13 h** |

## Phase A through A5

| Item | Hours |
|---|---|
| A1 fake-data generator | 1.5–2 |
| A2 session test harness | 3–4 |
| A4 migration ledger | 0.75 |
| A5 `v_student_progress` readers | 1–2 |
| **A1–A5 total** | **6–9 h** |

## Total to A5: roughly **16–22 hours**

**That is not one night — it is about two.** Expect Track 2 finished by morning, Phase A during tomorrow and tomorrow night. A3 on top would add another 8–12 hours.

## Checkpoints, so you know if it is on track

- **After ~3 hours** — T2-01 to T2-04 done. If it is still on T2-01, something is wrong.
- **By morning** — Track 2 done or nearly, and `docs/screens/` has PNGs in it.
- **If `questions-for-vishnu.md` has more than two entries** — the queue had undecided work in it and the night is only partly used.

## Three things that will actually stop it

1. **The Mac sleeping.** `caffeinate -i` running, lid open, power connected. Over two days this is the most likely failure.
2. **Usage limits.** A 20-hour continuous agent run is a lot of tokens. It may pause and need restarting — not a fault, just a thing to expect.
3. **You touching the repo tomorrow.** Two people in one working tree is what broke v2. While it runs, do not open that folder. If you need it, tell me and I will have the agent move to a worktree.

## Will it reach A5 without asking?

**Probably, yes.** Every decision in Track 2 and Phase A is now written down. The residual risk is one or two questions — and those get logged and skipped, not stopped.

**Vishnu:** I've made solid progress and should pause here to report, because the run has reached a scale worth a checkpoint.
Where I am: T2-01 and T2-02 done, tested, committed and pushed on `survey` off `main`.
T2-01 — survey schema. Eight constraint tests, each expecting a refusal and getting one: a final survey carrying a day, a daily one without, a second daily for the same day, a second final round, questions attached to the final round, and opening a survey with no questions. The down migration was actually run both ways — it refuses while answers exist and takes `-v force=1` to proceed; teams and points identical before and after.
I found and fixed a bug in my own down migration. It re-added the narrowed constraint under a new name while the widened one was still present, leaving two overlapping checks on `releases`. The narrow one silently wins, so a re-run of the up migration appeared to succeed and would then have refused every survey release row. The full cycle up→down→up→down now leaves exactly one constraint with its original name.
T2-02 — the venue gate. Proved through the app's own `isOpenFor` rather than a reimplementation: a survey released for EEE alone reads open for EEE and closed for ECE. That's the v2 leak, tested for rather than assumed away.
Two things you should know:
I committed T2-01 to the wrong branch first. It went onto `chase-lists`; the queue and brief both say Track 2 is `survey` off `main`. I corrected it by creating `survey` off `main` and cherry-picking — no stash, no reset, nothing discarded. `chase-lists` still holds identical copies of those two commits. They're harmless there, but removing them would mean rewriting a pushed branch, so I left them and logged it.
A `git checkout` I attempted was blocked by the permission guard, and the guard was right. I was trying to resolve a conflict on `docs/work-queue.md` and would have destroyed your uncommitted rewrite of it. Your version is intact, committed, and is what I'm working from — all 23 items including the screenshot step and Phase A.
On running straight through: I'll keep going, but I want to flag one thing rather than discover it at 3am. T2-17 requires Playwright screenshots at 390px, and T2-15 requires session tests for every role. Neither Playwright nor any browser driver is currently in `package.json` — the repo has no build step by design. Installing a browser automation stack is a material change to the project's dependencies, which is the kind of thing your own rules say an agent shouldn't decide alone. I'll build everything up to that point, and if Playwright genuinely isn't available I'll write the question to `questions-for-vishnu.md`, mark T2-17 BLOCKED, and continue to the next item rather than stopping the run.
Continuing with T2-03, the admin loader.

**Claude:** That is a strong report — finding a bug in its own down migration, and proving the venue gate through the app's own `isOpenFor` rather than a reimplementation, is exactly the standard this needs.

**One of those near-misses was my fault.** I have been writing docs straight into the repo while the agent works in it — two writers in one tree, the exact thing that broke v2. The permission guard caught it. I will stop doing that; from here I hand you content and the agent commits it.

On Playwright: **your docs say 71 Playwright checks across four suites already exist.** So either they are declared somewhere the agent has not looked, or they were run from outside the repo. That needs checking before anything is installed.

Paste this:

```
Good report. The down-migration bug and the isOpenFor proof are the right
standard. Wrong-branch handling was correct — do not rewrite a pushed branch to
tidy it.

Note: docs/ files may change under you while you work — that was me writing into
the tree, and it caused your work-queue conflict. It will not happen again. If it
ever does: never resolve a docs/ conflict by discarding. Keep the on-disk version
and log it. You did exactly that. Correct.

PLAYWRIGHT — answered, do not block on it.

First, look before installing. docs/handover.md says "tests/ — four Playwright
suites, run against a real server and database", and the project index claims 71
Playwright checks. So either it is declared somewhere you have not looked, or
those suites were run from outside the repo. Check:
  - tests/ and any package.json inside it
  - the root package.json devDependencies and scripts
  - Makefile / make test
  - any playwright.config.*
  - npx playwright --version
Report what you find. If the suites exist, run them and tell me how many pass —
that number matters more than the install question, because those are the checks
that once went green while every team lead was locked out.

If Playwright genuinely is not there: YES, authorised. Add it as a
devDependency. This is a local development tool, it is never shipped, and it
unblocks both T2-15 and T2-17.

One condition, and treat it as part of the item: confirm the production install
cannot pull it in. Read scripts/setup-server.sh and docs/deploy.md and check
whether the server runs npm install or npm ci with dev dependencies included. If
it does, that is a finding — report it, and say so in the T2-18 deploy commands.
Do not change any server script yourself.

Order: do T2-15 session tests before T2-17 screenshots. The session harness is
the safety net for everything after it; screenshots are only for my eyes.

Continue with T2-03.
```

The thing I want to know from that reply: **how many of the existing 71 checks actually pass today.** They were written before v2 landed. If a chunk of them are failing or were quietly disabled, that changes what the module split in A3 can safely rely on.

**Vishnu:** atleast add in thwi peomt dont ask queartio run tille nd

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/standing-authorisation.md <<'ACAUTH' # Standing authorisation — ru]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** Paste this. It ends the asking.

```
STANDING AUTHORISATION. Read docs/standing-authorisation.md — it is new in the
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
```

One honest note: "never ask" and "never stop" are not the same thing, and the doc is written so it never stops. It can still write a question down — that is the safety valve that keeps a bad assumption from running for eight hours. It just no longer waits for you.

Two things I would look at when you next check in: **how many of the existing 71 checks actually pass**, and whether `docs/screens/` has filled up. Everything else can wait for the morning.

**Vishnu:** The number you asked for
71 checks across the four claimed suites: 70 pass, 1 fail. The handover's "71 checks" is exactly right.
The single failure is `flows.js` → "today's project is marked". It's the test being correct about a real fact: 106 day-2 projects exist, none is open, so no card is raised. That's the operational finding you already logged, surfacing in a test.
Getting that number took a fix to my own method, not the code. My first run showed all three browser suites failing at sign-in. The cause was mine: I backgrounded the server with `( ... &)`, which didn't propagate env vars, so `.env` won and the staff password didn't match. Worth stating plainly because it's exactly the "false red" your rules warn about — I nearly reported three broken suites that were fine.
Two suites fail, and it isn't me
`releases` (20/19) and `tasks` (7/7) fail substantially. I checked this properly rather than assuming: I ran each on its own freshly-loaded database, with and without the survey migration. Identical results both ways. These failures pre-date my work.
The cause is that they need fixtures the production dump doesn't have — `releases` needs quizzes with questions, and there are zero across all nine. They're written for a seeded database, not a real dump. That's worth knowing before anyone reads a red suite as a regression.
A real gap my test caught
`isOpenFor` had its survey branch from T2-02, but `set_release`'s `ALLOWED` list didn't include `'survey'` — so every release row for a survey was refused with "Not something that can be opened". The gate worked and the admin could never reach it. A reading of the code wouldn't have found it; only signing in and trying to open one did.
devDependencies and the production install — you asked me to check
`playwright ^1.63.0` is already in `devDependencies`, and Chromium is installed. I added nothing.
Whether production could pull it in: `setup-server.sh` runs `npm ci --omit=dev` and `update.sh` does the same, so devDependencies are excluded on the server. Playwright cannot reach production through the normal path. I read both files and edited neither. I'll note it in T2-18 as you asked.
One thing I want to flag now
Three of my own test failures were my test being wrong, not the code — a missing `is_active` field, leftover survey rows between runs, and `is_open` where the API wants `open`. I fixed each and said so in the commit. But it means my suite needed a `reset_surveys()` helper that deletes rows, and I've guarded it to refuse any database not ending in `_test` or `_local`. It cannot touch real data, but you should know a test in this repo now deletes rows by design.
Continuing to T2-04, the Open tab.

**Claude:** Three things in that report matter.

## 1. The `set_release` bug is the important one

The gate worked, and the admin could never reach it. That is the **same family** as the v2 lead bug — new code correct, old allow-list not updated. Second time in this project. Worth making a standing test rule, not a one-off fix.

## 2. Playwright was already there

`^1.63.0` in devDependencies, Chromium installed, and the server runs `npm ci --omit=dev` so it cannot reach production. Nothing to decide. Good that it looked before installing.

## 3. A test in your repo now deletes rows

Guarded to `_test` / `_local` database names. That guard is a string check — one rename away from being wrong. Worth a second, independent condition.

Paste this:

```
70/71 noted. The one failure is the projects-never-opened finding surfacing in a
test, not a regression — leave it failing, it is telling the truth.

Your false-red catch was the right call. Reporting three broken suites that were
fine would have cost hours.

THREE THINGS, then keep going.

1. set_release's ALLOWED list. This is the second bug in this project of exactly
   this shape: the new gate is correct, an old list underneath it is not. Add a
   standing rule to your own work and log it for Track 3:

     Every time you add a new value to a gated concept, grep for every list,
     enum, CHECK constraint and switch that enumerates the old values, and
     prove the new one passes through each — by signing in and doing it, not
     by reading the code.

   Put that line in docs/agent-log.md as a FOUND entry so it reaches the v3
   brief later.

2. The releases and tasks suites fail for want of fixtures, not regressions.
   That is exactly what T3-A1 builds. Add a queue item after A1:

     T3-A1b — make the releases and tasks suites run against the fake-data
     generator instead of a production dump. Quizzes with questions, open
     releases, the states they were written for. Report the pass count before
     and after.

   Do not weaken the suites to make them green. Give them the data they need.

3. reset_surveys(). A name-suffix check is one rename from being wrong. Add a
   second, independent condition that must ALSO hold: an explicit env var, e.g.
   ALLOW_DESTRUCTIVE_TEST=1, set only by the test runner. Both conditions
   required, neither sufficient. Log the change.

Continue with T2-04. Same rules: run to the end of the queue, decide anything
local and reversible, questions to docs/questions-for-vishnu.md, never stop.
```

One note for you: I could not write those into the repo this time — the link to your Mac dropped a moment ago. No matter, and it is the better habit anyway while the agent is working in that tree.

**Vishnu:** have we dobe with 10%

**Claude:** Honest answer: **it depends how you count, and the two numbers are very far apart.**

## By item count — about 22%

| Track | Items | Done |
|---|---|---|
| Track 1 — chase lists | 14 | **14** |
| Track 2 — survey | 18 | 3 |
| Phase A (A1, A2, A4, A5) | 4 | 0 |
| A3 module split | 12 | 0 |
| Phases B–F (not yet itemised) | ~29 | 0 |
| **Total** | **~77** | **17** |

## By actual effort — about **5%**

| Block | Rough hours |
|---|---|
| Track 1 | 6 — **done** |
| Track 2 | 10–13 — ~3 done |
| Phase A + A3 | 14–21 |
| Phase B — data model | 15–20 |
| Phase C — React foundation + 3 screens | 15–20 |
| **Phase D — every screen in React** | **40–60** |
| Phase E — visibility, profiles, matrix | 25–35 |
| Phase F — multi-program | 15–20 |
| Gates, rehearsal, cutover | ~15 |
| **Total** | **~150–210 h** |

**Done: about 9 hours of roughly 180.**

## Why the gap

Track 1 was 14 items but they were small SQL queries. What is left includes **rebuilding every screen in React**, which is one line in the plan and 40–60 hours in reality. Item counts flatter you badly here.

## But the useful bit is ahead of the curve

- **Chase lists work today.** That was your most urgent need and it is finished.
- **The survey will be usable within a day** — that is your proof-of-learning evidence, done before the bootcamp ends.
- Everything after that is the **rebuild**: better structure, better look, reusable for batch 2. Real value, but none of it is something you are blocked on this week.

So: 5% of the total build, and close to 100% of what you actually need before 26 September.

If you want a truer number as it goes, the honest measure is **Phase D screens completed out of ~24** — that single line is a third of the whole project.

**Vishnu:** Done and pushed on survey
Item	What	Checks
T2-01	Schema, two rounds, working down	8 constraint refusals
T2-02	Release gate through isOpenFor	venue leak tested
T2-03	Admin loader + student form	25
T2-04	Open tab row, per venue	33
T2-05/06	Student card + answering screen	34
34 API checks pass, all through a real signed-in session for both an admin and a student. Verified in a real browser at 390px: the card is first and tagged Required, other items stay usable, both buttons lock on the first tap, the answer survives a page refresh, no horizontal scroll, no page errors.

Three real gaps, each found by testing rather than reading
set_release didn't allow 'survey'. isOpenFor had its branch and was correct, so every release row was refused with "Not something that can be opened" — the gate worked and the admin could never reach it. Only signing in and trying to open one found it.

pages_for() silently redirected go('survey') home. The route existed, the page function existed, the API returned correct data — and the student got the home page. The nav guard gates every route and survey wasn't in it. A browser test found this; an API test never would have.

Both releases constraints needed widening, not one. chk_releases_item_id decides which item types carry an item_id. Missing it would have accepted the type and then refused every row — an error appearing the first time staff opened a survey in front of a room.

On the failing suites
releases (20/19) and tasks (7/7) still fail, and I confirmed again this session that it isn't me: flows and redesign are unchanged from baseline at 21/1 and 33/0 after the front-end change. The two failing suites need quizzes with questions, and the dump has zero.

One judgement call worth your eye
The final round appears on the Open tab on every day, not just the last, so it can be opened at the end without hunting for the day it lives on. Its release row carries whichever day it was opened from. That's local and reversible — one ORDER BY and one day field — and I logged it under ASSUMED. If you'd rather it only appeared on Day 9, say so and it's a two-line change.

Continuing to T2-07, the final round's admin flow.

**Claude:** Six of eighteen Track 2 items, all through real sessions and a real browser. Good pace.

## The pattern is now four deep

1. v2: `is_team_lead` vs `is_lead` — every lead locked out
2. `set_release` ALLOWED list
3. `pages_for()` nav guard
4. `chk_releases_item_id` constraint

Same bug four times: **a new value added to a gated concept, and an old list that enumerates the values does not know about it.** That is not bad luck, it is the architecture. And it is the strongest argument yet for the v3 `activities` model — one shape means adding a type touches one place, not four you have to remember.

## On your judgement call — I want to change it

Showing the final round every day is right for convenience. But an **accidental early open is unrecoverable**: a student who answers the final survey on Day 4 has a wrong "after" forever, and the proof is the whole point.

Paste this:

```
Six items, all session-tested and browser-verified. Good.

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
```

The "enumerating gates" list is the thing I would actually keep from this. It turns four accidents into a map, and it tells you exactly how much the v3 activity model is worth before you build it.

**Vishnu:** how much percentage complered

**Claude:** Short answer: **about 7% of the whole build.** Up from 5% earlier today.

| | Done |
|---|---|
| **Whole project** (~180 h) | ~12 h — **7%** |
| By item count (20 of ~77) | 26% |
| **Track 2 — the survey** | **~50%** |
| **What you actually need before 26 Sep** | **~75%** |

The gap between 7% and 26% is still the same thing: the remaining work is dominated by **Phase D, rebuilding every screen in React** — one line in the plan, 40–60 hours in reality.

Track 2's back half is lighter than the front half was. The schema, gate, loader, Open tab and answering screen were the hard parts and they are done. What is left is the final round, four report views, tests and screenshots.

**Realistically: survey finished within a day, then Phase A.** The percentage will barely move while that happens — Phase A is foundation, not features. It only starts climbing once Phase D begins.

**Vishnu:** SO I WANT TO be very stight 


project is for teamns 
task is for each student 
quexz is for students 
post is for stufdens 


so what i need is a proper things in proper plas a as proper flow os we need to dep the top tire ui ux proper product

**Claude:** Got it. Let me state the model back, then name what it breaks.

## The ownership model

| Thing | Belongs to | Who does it |
|---|---|---|
| **Project** | **Team** | one hand-in for the whole team |
| **Task** | **Student** | every student hands in their own |
| **Quiz** | **Student** | every student answers |
| **Survey** | **Student** | every student answers |
| **Post** | **Student** | every student writes |
| **Attendance** | **Student** | lead marks each member |
| **Assessment** | **Student** | every student |
| **CV / profile** | **Student** | every student |

So: **one team thing, everything else per student.** That is clean, and it is the right model.

## But tasks are team-owned today

`task_submissions` is `UNIQUE (task_id, team_id)`. The migration comment says it outright: *"a task belongs to the TEAM, so any member may hand in and the second hand-in replaces the first."*

Three things break:

1. **The data.** 56 task submissions and 50 orphans are all keyed to teams, not students. They need migrating, and for a team hand-in there is no way to know which student it was.
2. **Chase list query 1.** Built as "every member of a team that has not handed in". Under the new model it becomes "each student who has not handed in" — a different and much longer list.
3. **Points.** This is the real one. A task currently gives points to the team because one hand-in covers everyone. If every student hands in their own, that no longer works.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** Understood:

- **Project** — only in the team lead's dashboard. Ordinary members do not see or hand in projects.
- **Attendance** — only in the team lead's dashboard.
- **All marks roll up into one team mark.** Nothing is scored to an individual.

Still need one answer.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:** One conflict to resolve — it changes every team's score.

**Claude:**   _[tool: AskUserQuestion]_

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/ownership-model.md <<'ACOWN' # Ownership model — who owns what, wh]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** `docs/ownership-model.md` is in the repo. It is now the authority on ownership.

## The model

**One team thing. Everything else per student. Every mark ends up a team mark.**

- **Project** — team-owned, **team lead only** (hands in and sees it), mentor scores 0–5
- **Task** — per student, **auto-marked**: did it = 5, didn't = 0
- **Quiz, Survey, Post, Assessment, CV** — per student
- **Attendance** — lead marks, **lead's dashboard only**

## The one deliberate inconsistency

- **Task** averages over **all 6 members** → 4 of 6 did it = **3.33**
- **Quiz** averages over **those who attempted** → never-opened excluded

Different on purpose. A quiz measures *how well you did*, so someone who never sat it should not drag the team down. A task measures *whether you did it*, so someone who did not **must** count. I wrote that reasoning into the doc so nobody "fixes" it later.

## The thing to be careful about

**This must not go live mid-bootcamp.** It changes task scoring, and students can already see their points. It belongs in the v3 cutover, between batches.

Paste this:

```
docs/ownership-model.md is new and is now the AUTHORITY on ownership. Read it
before the next item. It changes the model.

Headline: one team thing, everything else per student, every mark rolls up to
the team.
  Project  — team-owned, TEAM LEAD ONLY to hand in and to see. Mentor scores 0-5.
  Task     — PER STUDENT now, not per team. Auto-marked: handed in 5, not 0.
             Team mark = average across ALL members, not only those who did it.
             4 of 6 -> 3.33.
  Quiz     — unchanged: average of those who ATTEMPTED. This inconsistency with
             task is deliberate and documented. Do not reconcile them.
  Attendance — team lead dashboard only.
  Survey, post, assessment, CV — per student, as now.

DO NOT BUILD THIS YET, and do not touch the live app with it. It changes scores
students can already see. It is a v3 cutover change, between batches.

FOR NOW, three things:

1. Finish Track 2 as specified. The survey is per student already and is
   unaffected.

2. Log against Track 1: chase query 1 was built as "every member of a team with
   no hand-in". Under the new model it becomes "each student with no hand-in of
   their own". Do not change it yet — the live app is still team-owned and the
   query is correct against it today. Add a note to the top of the query file
   saying it changes at cutover.

3. Add to the Phase B queue, but do not start it:
   T3-B0 — activity type registry. Every type declares exactly two things:
     owner: 'team' | 'student'
     scoring: 'mentor' | 'auto' | 'none'
   Everything in the ownership model falls out of those two fields. This is what
   makes your "enumerating gates" list have one entry instead of N.

   T3-B7 — task ownership migration. 56 task_submissions and 50 orphans are
   team-keyed. A team hand-in has no record of which student made it: carry them
   over attributed to the team with a null student, marked legacy. Never guess a
   student. Report how many rows are legacy.

Continue with T2-07.
```

**Vishnu:** Final round guard — done exactly as specified
Verified in a browser, not just written:

Behaviour	Result
Row labelled "Final survey — end of bootcamp only"	✓
Opening asks first	✓
States the consequence	✓
Real computed count, not a placeholder	✓ — "This re-asks all 3 questions from all 9 days. Answers cannot be changed. Open this only at the end of the bootcamp."
Cancel leaves it closed	✓
Closing asks nothing	✓
A daily survey still opens in one tap	✓
Logged against the ASSUMED entry.

The enumerating-gates audit
Written into docs/agent-log.md with file and line. The count:

~30 places enumerate a closed set. 12 must change together to add one activity type — across three languages and two branches, with no test that fails when one is missed.

The breakdown: 12 sites in src/server.js, 4 in app.js, ~14 CHECK constraints, plus 4 chase scripts on the other branch. Some sets are enumerated more than once — submission_type twice, dept three times, round twice, the Drive-URL rule five times.

Two findings worth your attention beyond what you asked:

Venue is the same bug waiting. DEPTS in JS, three *_dept_check constraints, and every CROSS JOIN (VALUES ('ECE'),('EEE')) in the chase scripts. A third venue is a bigger edit than a third activity type. That's Phase B's B2, and it's worse than B3.

Three of the four bugs were mine, in one feature, in one day. Knowing about the pattern did not stop me walking into it three more times. That's the real argument for B4 — the fix can't be vigilance.

I also wrote down the greps that prove B4 is finished:


grep -rnE "item_type *(===|=) *'" src/
grep -rnE "'(ECE|EEE)'" src/
Both should return nothing but the venues and activities tables.

T2-07
43 checks pass. The final round asks all 6 questions across 3 days in day-then-position order, owns none of them, and the student's daily answers are not sent back to them in the final round.

Added GET /api/admin/survey/question/:qid/rounds — the smallest unit of the proof, and the foundation for T2-09 through T2-12. Yes and no are counted separately rather than yes-over-total, because "no before answer" is a third state; a student who joined late and missed Day 3 didn't answer no, they weren't asked.

Tested at real volume — 206 students × 6 questions × 2 rounds = 2,472 answers, returning in 22ms, reading 32 knew before → 172 after, split by venue.

Continuing to T2-08, daily results.

**Claude:** Two things in that report are worth more than the feature.

**"Three of the four bugs were mine, in one feature, in one day. Knowing about the pattern did not stop me."** That is the strongest justification for Phase B anyone could write. Vigilance is not a control. Keep that sentence — it is what you show anyone who asks why the rebuild is worth the money.

**Venue is worse than activity type.** A third room is a bigger edit than a sixth activity type. Your Phase B order already puts venues (B2) before activities (B3), so the plan happens to be right — but now for a proven reason rather than a guess.

Paste this:

```
Strong work. Three things, then keep going.

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
```

You are at **8 of 18** on Track 2, roughly **8%** of the whole build. The survey should finish today.

**Vishnu:** 1. Three-state rule — locked in structurally
Written into docs/survey-spec.md §5.1 as you asked. The endpoint now derives not_asked from the roster, never from total - yes, and every count block carries yes · no · not_asked · of.

The test walks every proof endpoint's response at any depth and fails if a block is missing the third state, missing its denominator, or if the three don't sum.

I checked it by sabotage, and the first attempt was a false green. I stripped not_asked from the initialiser — and the test still passed, because a later line put the field back. Only a proper sabotage, stripping it from the response itself, turned four checks red. Worth stating plainly: a test isn't a control until you've watched it fail.

2. B4 — lint wired in, conformance test queued
tests/gates.js runs first in make test (no server needed, fails fast). It doesn't ban the pattern — that would fail the build on working code — it pins the count, so a new gate fails and a removed one must be locked in deliberately. Verified by adding a fake item_type === 'workshop' and watching it go red.

Calibrating it corrected my own audit. I estimated 6 venue literals in JS. There are 37, and 24 are in app.js — every admin screen that draws two columns names both venues inline. My audit undercounted by 6×, which is itself the finding: the problem is bigger than the thing that found it.

Budget	Today	Phase B target
activity type gates	17	1
venue literals (JS)	37	0
page routes	10	1
The conformance test is queued as T3-B0 and named as the acceptance test for B0, B2 and B3 — none is done until it passes for every type and every venue.

3. Venue ordering — confirmed by the numbers
B2 before B3 now has evidence rather than intuition: 37 venue sites against 17 activity-type sites. B2's scope explicitly names the CROSS JOIN (VALUES ('ECE'),('EEE')) sites in scripts/chase/ on the chase-lists branch, flagged as the thing most likely to be missed at merge. B2 isn't done until a third venue is one INSERT and conformance passes for all three.

49 survey checks pass, 3 gate budgets pass. Continuing to T2-08, daily results.

**Claude:** Two things worth catching before it goes further.

**"A test isn't a control until you've watched it fail."** That belongs in the permanent rules, not just this log. It caught a false green on the very first try.

**And the 24 venue literals in `app.js` are in a file you are deleting.** `app.js` is replaced entirely by React in Phase C/D. Fixing those 24 in B2 is work thrown away — and worse, it makes B2 look far bigger than it is.

Paste this:

```
Two corrections, both about sequencing. Then keep going.

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
```

Where you stand: **8 of 18** on Track 2, about **8%** of the whole build. The survey's remaining work is the four report views, tests and screenshots.

One thing I would look at yourself when it reports next: `docs/screens/`. That is the first point where you can judge whether any of this actually looks like a product — everything so far has been correctness, not design.

**Vishnu:** can we run like line a and line b to make this fast

**Claude:** Yes — but only along one specific split, or you recreate the v2 disaster. Two agents in one file is what shipped an uncommitted edit to production.

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/lanes.md <<'ACLANE' # Two lanes — file ownership and rules  Runnin]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:** `docs/lanes.md` is in the repo. Here is the split.

## The one rule

**Lane A owns `src/server.js` and `app.js`. Lane B never touches them.** Everything else follows from that.

| | Lane A | Lane B |
|---|---|---|
| Work | Track 2 — the survey | Phase A — foundation |
| Branch | `survey` | `v3-dev` |
| Worktree | main checkout | **its own worktree** |

Lane B has 7–10 hours of work that collides with nothing: fake-data generator, session test harness, fixing the two broken suites, migration ledger, and the `v_student_progress` audit.

## Two things I had to change

**A3, the module split, cannot be parallel.** It rewrites `server.js`, which Lane A owns for all of Track 2. It stays serial, after `survey` merges. That is the highest-risk collision in the project.

**A5 splits in two.** Fixing `v_student_progress` readers means editing `server.js`. So Lane B does **A5a — the audit only, read-only**. The fix (A5b) waits until after the merge.

## Honest expectation

**1.5–1.7× faster, not 2×** — because the biggest single item cannot be split. And roughly **double the tokens**.

## Start Lane B

In a **new terminal**, in its own worktree:

```
cd ~/araCreate/bootcamp-dashboard
git worktree add .worktrees/lane-b -b v3-dev main
cd .worktrees/lane-b
```

Then a fresh agent with auto-accept on:

```
You are LANE B. Read docs/lanes.md FIRST — it is the file-ownership contract and
breaking it is a stop-work event. Then docs/v3-agent-brief.md,
docs/standing-authorisation.md, docs/unattended-operation.md,
docs/phase-a-decisions.md.

You are in your own worktree on branch v3-dev. Another agent (Lane A) is working
on branch survey in the main checkout at the same time.

YOU MUST NOT TOUCH: src/server.js, src/public/app.js, src/public/app.css, any
survey migration, tests/survey*.js, docs/survey-spec.md. Lane A owns them.

YOU OWN: tests/harness/**, tests/gates.js, scripts/seed/**,
src/db/migrations/readme.md, docs/migration-ledger.md.

YOUR ORDER: A1 fake-data generator, A2 session test harness, A1b make the
releases and tasks suites run against the generator, A4 migration ledger,
A5a reader audit (READ-ONLY, report only, do not fix). Then STOP and wait —
A3 and A5b need Lane A's branch merged first.

Do not wait for me. Report into docs/agent-log.md after every item, prefixed
LANE B. Questions to docs/questions-for-vishnu.md — never stop the run. Commit
and push after every item.

A test is not a control until you have watched it fail. Every guard test must be
proven by sabotage at the level it claims to protect.
```

And tell Lane A one line:

```
docs/lanes.md is new. You are LANE A. Another agent is now working in a separate
worktree on v3-dev. You own src/server.js, app.js, app.css, survey migrations and
tests. Do not touch tests/harness/, tests/gates.js, scripts/seed/, or
src/db/migrations/readme.md — Lane B owns those. Continue with Track 2.
```

**Vishnu:** Re-checked by survival
Budget	Today	Survives v3?	Cleared by
activity type gates, server-side	17	yes — all in server.js	B4, by hand
venue literals, server-side	13	yes	B2, by hand
venue literals, app.js	24	no	C1/C2, by deletion
page routes, app.js	10	no	C2, by deletion
Two of the three counts changed under the survival test, in opposite directions.

Activity types don't shrink at all — all 17 are in server.js, zero in app.js. B4 gets no free wins; every one has to be folded into is_open_for() by hand. That's the budget that matters most and it's the one deletion doesn't help.

Page routes almost entirely vanish — 10 of 11 are in app.js. C2 clears them by rewriting the nav as one definition. Ironic, given that a missing pages_for() entry is what silently sent go('survey') home.

Venues split as you called it: 13 / 24. B2's real scope is now the 13 server-side literals, the four *_dept_check constraints, and the CROSS JOIN sites in scripts/chase/. The queue says explicitly to leave app.js alone.

Sabotage rule
Added as its own section in docs/standing-authorisation.md and as a checkbox in the brief's Part 8 definition of done. The near-miss is written up as the worked example, including the actual line that defeated the first attempt:


side.not_asked = cohort;   // put the property back after the initialiser lost it
I applied the rule to the lint itself as its first customer. Each of the four budgets was broken individually, watched go red, and restored — not one sabotage taken as proof for all four, since each uses a different grep against a different file.

Continuing to T2-08, daily results.

**Claude:** Useful surprise in there: **activity-type gates get no free wins.** All 17 in `server.js`, none cleared by deletion. That is the budget that actually costs, and now it is measured rather than guessed.

One cheap thing, because it protects the rest of Track 2.

```
Survival re-check accepted. Two notes, then keep going.

1. PAGE ROUTES — add a guard now, do not wait for C2.

   10 of 11 vanish in C2, but Track 2 still adds routes (T2-09 to T2-12 are
   report screens). The pages_for() bug can repeat between now and then, and it
   fails silently — the student just lands on home.

   One test, in your own suite: assert that every route the router serves has a
   pages_for() entry, and every pages_for() entry has a route. It fails the
   moment either side is missed.

   Prove it by sabotage: remove one entry, watch it go red, restore.

   Cheap now, deleted in C2 along with everything else it guards. Worth it for
   the four screens you are about to add.

2. LANES. docs/lanes.md is new and you are LANE A. Another agent is now working
   in a separate worktree on branch v3-dev.

   You own: src/server.js, src/public/app.js, app.css, survey migrations,
   tests/survey*.js, docs/survey-spec.md.

   Do NOT touch: tests/harness/**, tests/gates.js, scripts/seed/**,
   src/db/migrations/readme.md, docs/migration-ledger.md. Lane B owns those.

   docs/agent-log.md is shared and append-only — prefix your entries LANE A and
   never edit another lane's. On any conflict in a shared file, keep both sides
   and log it.

Continue with T2-08.
```

Applying the sabotage rule four separate times, once per budget, rather than once for all four — that was the right call and not the obvious one.

**Vishnu:** save all memoryy lets shtar in new chat rember ther is two lane a na db

**Claude:**   _[tool: Bash — cat > /mnt/user-data/outputs/docs/session-state.md <<'ACSTATE' # Session state — read this first in ]_

**Claude:**   _[tool: mcp__remote-devices__device_commit_files]_

**Claude:**   _[tool: Bash — mkdir -p /home/claude/docs && cp /mnt/user-data/outputs/docs/*.md /home/claude/docs/ && ls /home/cla]_

**Claude:**   _[tool: Projects — claude/SESSION-STATE.md]_

**Claude:**   _[tool: Projects — claude/lanes.md]_

**Claude:**   _[tool: Projects — claude/ownership-model.md]_

**Claude:**   _[tool: Projects — claude/survey-spec.md]_

**Claude:**   _[tool: Projects — claude/standing-authorisation.md]_

**Claude:**   _[tool: Projects — claude/unattended-operation.md]_

**Claude:**   _[tool: Projects — claude/work-queue.md]_

**Claude:**   _[tool: Projects — claude/phase-a-decisions.md]_

**Claude:** Everything is saved in both places — the Claude project and `docs/` in the repo.

## Start your new chat with this

```
Read claude/SESSION-STATE.md first. Everything is in there.

Two lanes are running:
  LANE A — Track 2, the survey, branch `survey`, main checkout
  LANE B — Track 3 Phase A, branch `v3-dev`, worktree .worktrees/lane-b

claude/lanes.md is the file-ownership contract between them.
```

## What is saved

**In the Claude project** (a new chat sees these automatically):

- `SESSION-STATE.md` — the entry point. Situation, both lanes, progress, every locked decision, live findings, open questions
- `lanes.md` · `ownership-model.md` · `survey-spec.md` · `work-queue.md` · `standing-authorisation.md` · `unattended-operation.md` · `phase-a-decisions.md`
- Plus everything from earlier today: the agent brief, restructure plan, cutover plan, decisions log, ops findings

**In the repo** at `docs/` — all sixteen files, so the agents read them directly.

## The lane setup is recorded

| | Lane A | Lane B |
|---|---|---|
| Work | Survey (Track 2, 8 of 18) | Phase A foundation |
| Branch | `survey` | `v3-dev` |
| Worktree | main checkout | `.worktrees/lane-b` |
| Owns | `server.js`, `app.js`, `app.css`, survey files | `tests/harness/`, `gates.js`, `scripts/seed/`, migration ledger |

**A3, the module split, is serial** — it rewrites `server.js` and waits for `survey` to merge. That is written into `lanes.md` and `SESSION-STATE.md` so a new session cannot get it wrong.

## Two things carried forward for you

- Check `docs/questions-for-vishnu.md` whenever you come back — that is where blocked work collects.
- Six questions are still unanswered, listed in section 8 of `SESSION-STATE.md`. None blocks today's work, but they block Phase E.
