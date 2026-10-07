<!--
SPDX-License-Identifier: LicenseRef-Proprietary
Copyright (C) 2026, araCreate Group
Author: Vishnu Varthan <vishnu@aracreate.group>
Description: This file holds the prompts that build v2, one step at a time
-->

# v2 build prompts

Written 17 Sep 2026. Live counts from the production dump: 53 teams, 209 students (EEE 14 teams / 55 students, ECE 39 teams / 154 students). Day 1 is 18 Sep. The app must work at 9 AM for 209 students.

Paste each block into Claude Code, from the repo root. **One at a time.**
Run the check at the end of each. If a check fails, stop. Do not paste the next one.

## What the repo already has

Read before starting. Three things people assume are missing are already built.

| Already there | Where |
| --- | --- |
| `attendance` UNIQUE (student_id, day) | `src/db/schema.sql` — marking twice is already impossible |
| `students.dept` and `teams.dept`, both 'ECE'/'EEE' | `src/db/schema.sql` — no new column needed for the split |
| `projects.is_open`, opened and closed by staff | `src/db/migration-project-open.sql` |
| `quizzes.is_open` | `src/db/schema.sql` |
| A quiz scores 1 point per correct answer | `src/db/migration-quiz-per-question.sql` |
| `grade_quiz_attempt()` is the function the app calls | same file, addendum 2 |
| `recalc_team_points_for()` keeps `teams.total_points` | `src/db/schema.sql` |

So the real gaps are: open/close is global not per department, a quiz is one shot
per team not per student, the timer is one clock for the whole quiz, and a day
carries one project not many tasks.

## Decisions already made

| Question | Answer |
| --- | --- |
| Leaderboard | Split per department. Combined view as a second tab. |
| Quiz length | 20 questions, 30 seconds each |
| Quiz scoring | Unchanged: 1 point per correct answer |
| Team quiz points | The average of the members who attempted, rounded to a whole number |
| Opened the quiz, answered nothing | An attempt. Scores 0. Pulls the average down. |
| Never opened the quiz | Not an attempt. Left out of the average. |
| Task points | A day still gives a maximum of 5, however many tasks it carries |
| Attendance window | Admin opens and closes it by hand, per department. No timer. |

## Order and time

| # | Step | Time | Blocks |
| --- | --- | --- | --- |
| 1 | Backup + staging DB | 30 min | everything |
| 2 | Attendance finish | 30 min | Day 1 |
| 3 | Department split + `releases` | 90 min | 4, 5, 6, 7 |
| 4 | Tasks engine | 60 min | Day 1 |
| 5 | Quiz per student | 90 min | Day 1 |
| 6 | Profile + completion bar | 45 min | — |
| 7 | Pre-assessment | 30 min | Day 1 or 2 |
| 8 | Drive + uploads | 45 min | — |
| 9 | Copy CVs to Drive, verify only | 30 min | — |
| 10 | Full dry run on staging | 45 min | deploy |
| 11 | Deploy | 20 min | — |

**Not tonight:** the post-assessment screen (used Day 9) and deleting the CVs off
the server (permanent — leave it a week after the copies are verified).

**Hard cutoff:** past 4 AM with step 10 not passed, deploy nothing. The current app
runs Day 1 perfectly well.

---

## 0 — Context. Paste first in every new session.

```
Read docs/handover.md, docs/design.md, docs/readme.md and src/db/schema.sql before doing anything.

This is the live bootcamp app at vcet.aracreate.academy. 53 teams, 209 students, two venues: EEE 14 teams and 55 students, ECE 39 teams and 154 students. Onboarding was 17 Sep. Day 1 is 18 Sep. We are building v2 tonight, on a live app, hours before students use it.

Hard rules, no exceptions:
- Never run src/db/load-eee.sql or src/db/load-ece.sql.
- Never change start_date in settings on the server.
- Add-only. Do not drop a table, drop a column, or rename anything. Old columns and old tables stay even when nothing reads them.
- Every schema change is one file in src/db/migrations/ named YYYY-MM-DD-short-name.sql. Follow the house style exactly: SPDX header, copyright, Author line, Description line, then a comment block explaining WHY in plain English, then BEGIN/COMMIT, then a CHECKS section of commented-out verification queries at the end. Read src/db/migration-quiz-per-question.sql first and copy its shape.
- Every migration must be safe to run twice.
- Use the existing design system components. Do not write new CSS.
- Do not deploy. I deploy by hand at the end.

Work on branch v2. Tell me what you changed and why before you move on.
```

---

## 1 — Backup and staging. No code.

```
Before any code. Do these in order and give me a pass/fail line for each.

1. pg_dump the production database, gzip it, write it to .archives/ with a timestamp in the name.
2. Copy that dump off the server onto this machine.
3. Create a local database bootcamp_staging and restore the dump into it. Report row counts for teams, students, attendance, quizzes, quiz_attempts, projects, scores.
4. Run the app locally against bootcamp_staging and sign in as one real student. Confirm the home page loads.
5. Create branch v2 off main and push it.

Write no feature code in this step.
```

**Check:** a dump file sits on your own machine, and the app runs on the restored copy.

---

## 2 — Finish attendance

```
attendance already has UNIQUE (student_id, day), so double marking is already impossible in the database. Do not add that constraint again. What is missing is the handling around it.

1. Find the mark-attendance route in src/server.js. Confirm that a second mark for the same student and day returns a clear message, not a 500 and not a silent success. Fix it if it does not.
2. Migration: add to attendance
   source TEXT NOT NULL DEFAULT 'self' CHECK (source IN ('self','admin'))
   note   TEXT
   Both nullable-safe on existing rows.
3. Migration: create attendance_audit (id, attendance_id, student_id, day, action TEXT, old_present BOOLEAN, new_present BOOLEAN, actor_id INT, reason TEXT, at TIMESTAMPTZ DEFAULT now()). Every admin change to attendance writes a row. A student marking themselves does not.
4. UI: once marked, the button becomes a flat "Marked at HH:MM" state and cannot be pressed again.
5. Admin screen: today's attendance, one column per department, with a present count out of the department total, and the ability to mark or unmark one student with a required reason.

Add a Playwright test in tests/ that posts attendance twice for the same student and day and expects the second to be refused with a readable message.
Run make test.
```

**Check:** marking twice gives a readable message. An admin unmark writes an audit row.

---

## 3 — Department split. The big one.

```
Today quizzes.is_open and projects.is_open are single booleans, so opening anything opens it for all 206 students across both venues at once. The two venues do not run to the same clock and this breaks the first morning one of them starts late. Everything must open per department.

1. Migration: create table releases
   id          SERIAL PRIMARY KEY
   item_type   TEXT NOT NULL CHECK (item_type IN ('quiz','task','attendance','pre_assessment','post_assessment'))
   item_id     INT                                  -- quiz id or task id; NULL for attendance and assessments
   dept        TEXT NOT NULL CHECK (dept IN ('ECE','EEE'))
   day         INT CHECK (day BETWEEN 1 AND 9)
   is_open     BOOLEAN NOT NULL DEFAULT FALSE
   opened_at   TIMESTAMPTZ
   opened_by   INT
   closed_at   TIMESTAMPTZ
   closed_by   INT
   UNIQUE (item_type, item_id, dept, day)

2. Keep quizzes.is_open and projects.is_open. Do not drop them. They become the fallback: if no releases row exists for an item and a department, fall back to the old boolean, so nothing goes dark mid-bootcamp.

3. One helper, used by every student-facing route: isOpenFor(itemType, itemId, dept, day). Routes for quizzes, tasks, attendance and assessments must all call it. A student must not see, and must not be able to POST to, anything that is not open for their own students.dept. Wrong department is a 403, not an empty page.

4. Admin home: two columns side by side, EEE and ECE. Each column lists today's quiz, each task, the attendance window and the pre-assessment, each with its own Open and Close button. Under each, show who opened it and at what time.

5. An "Open for both" button next to each item, for days the venues run together. One click. It is not the default.

6. A red dot next to anything not yet opened today for either department.

7. No cron. No time-based auto-open. An admin opens everything by hand.

8. Backfill in the same migration: for every item currently is_open = true, write releases rows for BOTH departments, so nothing a student is working on right now closes under them.

Add a Playwright test: open a quiz for EEE only, then as an ECE student confirm the quiz is absent from the home page and the direct route returns 403.
Run make test.
```

**Check:** sign in as an ECE student with only EEE open. Nothing visible, and the direct URL 403s.

---

## 4 — Tasks engine

```
A day currently carries one project per team. It needs to carry several tasks.

1. Keep the projects table, the submissions table and the scores table. Do not drop or rename any of them. They stay as history and the old screens keep working.
2. Note projects.max_points has CHECK (max_points = 5). Do not touch that constraint. Tasks are a new table.
3. Migration: create table tasks
   id, day INT CHECK (day BETWEEN 1 AND 9), title TEXT NOT NULL, description TEXT,
   dept TEXT CHECK (dept IN ('ECE','EEE')),          -- NULL means both
   submission_type TEXT NOT NULL CHECK (submission_type IN ('image','drive','text','file','none')),
   max_points INT NOT NULL DEFAULT 5,
   created_by INT, created_at TIMESTAMPTZ DEFAULT now(), active BOOLEAN DEFAULT TRUE
4. Migration: create table task_submissions
   id, task_id, team_id, submitted_by INT, content_text TEXT, drive_url TEXT,
   file_path TEXT, submitted_at TIMESTAMPTZ,
   points INT, scored_by INT, scored_at TIMESTAMPTZ,
   UNIQUE (task_id, team_id)
5. A task belongs to the TEAM. Any member can submit for the team. Points go to the team, never the individual.
6. Day scaling: a day is still worth a maximum of 5 points however many tasks it carries. The day score is (sum of points scored / sum of max_points for that day's tasks) * 5. Round to one decimal. The 90-point maximum over nine days must not change.
7. Extend recalc_team_points_for() so project_points counts BOTH the old scores table and the new task_submissions, without ever counting a day twice. If a day has tasks, tasks win and the old project score for that day is ignored.
8. Admin task creator: title, description, day, department or both, submission type, max points. Creating a task does NOT open it. It writes a tasks row, and the admin opens it from the releases screen built in step 3.
9. Mentor screen: submissions for their own teams, score 0 to max, one click.
10. Update v_leaderboard and add a v_team_tasks view alongside v_team_projects.

Add a test: one day, two tasks, both scored full marks. The team must get 5 for that day, not 10.
Run make test.
```

**Check:** two tasks in one day still cap at 5 points, and `teams.total_points` is right.

---

## 5 — Quiz taken per student

```
quiz_attempts today is UNIQUE (quiz_id, team_id) with an answered_by column: one member answers for the whole team. Change it so every member answers for themselves.

1. Do not drop quiz_attempts or quiz_answers. Migrate them.
2. Migration: add student_id INT REFERENCES students(id) to quiz_attempts. Backfill student_id from answered_by for every existing row. Then drop the UNIQUE (quiz_id, team_id) constraint and add UNIQUE (quiz_id, student_id). Keep team_id and keep answered_by. Keep every existing attempt row.
3. Per-question timer, 30 seconds, replacing the single expires_at clock for the whole quiz. Add to quiz_answers: shown_at TIMESTAMPTZ, answered_at TIMESTAMPTZ, time_taken_ms INT, timed_out BOOLEAN DEFAULT FALSE. When 30 seconds run out the question locks, is marked wrong, and the next one loads. There is no going back. Keep quizzes.time_limit_min in the table but stop using it as the gate.
4. Save the answer to the server the moment it is picked, not at submit. If the phone dies mid-quiz, everything already answered survives. Re-entering the quiz resumes at the first unanswered question, never from the start.
5. Scoring stays as it is: one point per correct answer, by grade_quiz_attempt(). Do not change that rule.
6. Team quiz points for one quiz = the average of the points of the members who attempted, rounded to a whole number.
   - Opened the quiz but answered nothing: an attempt, scoring 0, included in the average.
   - Never opened it: not an attempt, left out of the average.
   - Nobody attempted: the team scores 0 for that quiz.
   Put this in one function, recalc_team_quiz_points(team_id), and have recalc_team_points_for() call it.
7. Open per department, through the releases table from step 3.
8. Leaderboard: one table per department by default, combined as a second tab. teams.dept already exists, use it.
9. Update v_quiz_results to be per student, and add a per-team rollup view.
10. Admin live screen: per department, how many students have attempted, how many are still in progress, how many have not started.

Add a test: a team of 3 on a 10-question quiz. Two attempt and score 8 and 6, the third never opens it. The team must get 7, not 4.67.
Run make test.
```

**Check:** on a real phone — answer a question, lock the screen, come back. The answer is still there and the quiz resumes where it stopped. This is the one that matters most tomorrow.

---

## 6 — Profile and completion bar

```
1. Full student profile page: photo, phone, personal email, education, skills, goal, old CV, new CV. student_profiles already exists, extend it rather than making a new table.
2. The home page progress bar shows PROFILE COMPLETION ONLY. Not attendance, not marks, not points. Weights: photo 10, phone 10, personal email 10, education 15, skills 15, goal 10, old CV 15, new CV 15.
3. The new CV only counts from Day 8. Before Day 8 the bar is out of 85 and is shown as a percent of 85, so a student who has done everything possible sees 100 percent, not 85.
4. Under the bar, list what is still missing. Each item is a link that jumps to that field.
5. Compute it in ONE place on the server and send the result to the page. Do not repeat the weights in the front end.
6. v_student_progress exists. Update it rather than writing a second one.

Add a test for the Day 8 boundary: same profile, Day 7 and Day 9, different denominators.
Run make test.
```

---

## 7 — Pre-assessment

```
1. Migration: assessment_questions (id, kind TEXT CHECK (kind IN ('pre','post')), position, question, option_a..option_d, correct CHAR(1)) and assessment_attempts (id, kind, student_id, correct_count, total_count, score_percent NUMERIC, submitted_at, UNIQUE (kind, student_id)).
2. Individual, not team. Worth ZERO points. It must never touch teams.total_points or the leaderboard.
3. NO TIMER. Not per question, not overall.
4. The pre and post question sets are allowed to be different. Never compare raw marks. Compare score_percent only.
5. Results are hidden from the student. Admin only.
6. Opens per department through releases, from step 3.
7. Build the post side now but leave its screen switched off behind a flag. It is used on Day 9.

Run make test.
```

---

## 8 — Google Drive and uploads

```
1. Google Drive through a service account writing into a Shared Drive. Read credentials from environment variables, add them to .env.example with empty values.
2. Never commit the key file. Add it to .gitignore and confirm with git check-ignore that it is ignored.
3. One folder per team inside the Shared Drive, created on demand. Write the link to teams.drive_folder_url, which already exists.
4. Uploads for tasks with submission_type 'image' or 'file' go to Drive. Store the Drive file id and link, not the bytes.
5. Cap uploads at 10 MB, images and PDF only. Anything else is refused with a readable message.
6. If Drive is unreachable the upload fails with a clear message and the rest of the submission is not lost.

Add a test with Drive mocked out.
Run make test.
```

---

## 9 — Copy the CVs to Drive. Copy only.

```
1. Find every CV file on the server. scripts/export-resumes.sh exists, read it first. Report the file count and total size.
2. Copy each to that team's Drive folder. Delete nothing.
3. Write the Drive link onto the student row. Leave the existing server path column exactly as it is.
4. Verify every single file: the Drive copy exists and the byte size matches the server file. Report any mismatch by student name and roll number.
5. Make a tar.gz of the whole CV directory and leave it in .archives/.

DO NOT DELETE ANY SERVER FILE. Deleting is a separate job for next week, once these copies have been checked.
```

**Check:** zero mismatches. If not zero, stop and tell me.

---

## 10 — Full Day 1 dry run, on staging only

```
Against bootcamp_staging only. Never the server. Set start_date locally so today is Day 1, and put it back afterwards.

Before you start:
- Read src/db/migrations/readme.md and apply the migrations IN DEPENDENCY ORDER, not alphabetical. Two of them fail out of turn: closed-by-default needs releases, and cv-drive-links needs profile-completion. Both fail as clean rollbacks, but they are a stop mid-run.
- tests/drive.js has one wall-clock assertion ("gives up within the timeout"). It can flake when the machine is busy. If it fails, rerun it once before investigating.
- Attendance is marked by TEAM LEADS for their own members, never by students individually. Anything below that says "students mark attendance" is wrong.

Run the whole day as real users:
1. Admin opens attendance for EEE only. An ECE student must see nothing.
2. EEE team leads mark attendance for their own members. Marking twice must be refused with a readable message.
3. Admin opens attendance for ECE an hour later. It works, and EEE is unaffected.
4. Admin creates two tasks for Day 1. Opens one for both departments, one for EEE only.
5. A team submits both. A mentor scores both at full marks. The day must give 5 points, not 10.
6. Admin opens the quiz for EEE. Three members of one team take it in three separate browser sessions. One opens it and answers nothing. Confirm the team mark is the average of the two who answered plus the zero, and that a member who never opened it is left out.
7. Confirm a question locks at 30 seconds, is marked wrong, and the next one loads.
8. Kill a browser mid-quiz and reopen. The answers so far must still be there and the quiz resumes at the right question.
9. Leaderboard per department, and the combined tab.
10. Sign in as a student from each department. Neither can see the other's items.
11. Confirm start_date is back to its original value.

Give me a pass/fail list. Then run make test, all suites.
```

**Check:** every line passes. Any failure means fix it and run step 10 again from the top.

---

## 11 — Deploy

```
Only after step 10 passed in full.

1. Confirm the dump from step 1 still exists and gunzip -t passes on it.
2. Take a second pg_dump right now, immediately before deploying.
3. Commit in conventional commit format. Merge v2 into main.
4. Deploy with the exact steps in docs/deploy.md.
5. Run the migrations on the server one at a time, in order, reporting each before starting the next.
6. Tail the logs. Confirm the service is healthy and the site loads.
7. Sign in on the live site as one student from each department and confirm the right home page.
8. Confirm start_date on the server is unchanged.

If anything fails after step 5, restore from the dump taken in step 2 and tell me at once.
```

---

## If it breaks at 9 AM

- The dumps from steps 1 and 11 are on this machine and on the server
- Every change was add-only, so the old tables still hold the old data
- Fastest rollback is redeploying the previous commit — the old code still reads the old tables
- Do not debug live with 206 students waiting. Roll back first, fix after.

---

# Running two agents at once

Set up 17 Sep, after step 1 passed. Two worktrees, two branches, one repo.

| | Lane A — critical path | Lane B — side path |
| --- | --- | --- |
| Folder | the repo root | `.worktrees/side` |
| Branch | `v2` | `v2-side` |
| Steps | 2, 3, 5, 4, 7 | 6, 8, 9 |
| Database | `bootcamp_staging` | `bootcamp_staging_b` |
| Migration prefix | `2026-09-17-a-*.sql` | `2026-09-17-b-*.sql` |
| App port | 3111 | 3112 |

## The rules that keep them apart

- **`src/server.js` belongs to Lane A.** Lane B does not edit it beyond adding a
  single `require` line per new route file. Every Lane B route lives in a new file
  under `src/routes/`.
- Migration filenames carry the lane letter, so two agents never pick the same name.
- Each lane runs migrations against its own database only. Never the other one's,
  never the server.
- Lane B merges into `v2` when it finishes. Lane A never merges into `v2-side`.
- Neither lane deploys. Step 10 runs once, on `v2`, after the merge.

## Why this split and not another

Steps 4, 5 and 7 all call `isOpenFor()`, which step 3 creates, so they cannot start
before it. That makes 3 the neck of the whole night and it has to sit on the lane
that is not waiting for anything. Steps 6, 8 and 9 touch the profile page, a new
Drive module and a copy script — none of which step 3 goes near.

## Merging at the end

```
git checkout v2 && git merge v2-side
```

Expect conflicts only in `src/server.js` and `.env.example`. If Lane B kept to the
rules there will be at most a few lines in each.

# Quiz content: loaded each evening

Questions are written after the day's class, from what was actually taught, and
loaded that evening for the next quiz. That is the intended workflow, not a gap.
The bulk paste flow already exists at Admin > Quizzes > Paste many:

```
question | A | B | C | D | correct letter
```

**What this means for the build.** Nothing currently stops an admin opening a quiz
that has no questions in it yet. With questions arriving the evening before, an
empty quiz WILL get opened by mistake at some point, and every student who takes it
scores zero out of zero. `trg_quiz_max_points` sets `max_points` from the question
count, so an empty quiz is worth nothing and the mark is unrecoverable.

Guard for step 3: opening a quiz for a department must fail with a clear message when
that quiz has fewer than 5 questions. Admin sees the question count next to every
quiz on the release screen, so it is obvious before the click, not after.

---

# File ownership, settled 17 Sep

Both lanes edited `tests/flows.js`. Deciding this once, so it does not happen again.

| File or folder | Owner | Note |
| --- | --- | --- |
| `src/server.js` | Lane A | Lane B adds one `require` line per route file, nothing else |
| `src/routes/*` | Lane B | new files only |
| `src/public/app.js` | shared | each lane appends its own helper; never reformat the file |
| `tests/flows.js`, `tests/behaviour.js`, `tests/onboarding.js` | **Lane A** | already modified there |
| new test files | whoever writes them | name them after the step |
| `src/db/migrations/2026-09-17-a-*` | Lane A | |
| `src/db/migrations/2026-09-17-b-*` | Lane B | |
| `.gitignore`, `Makefile` | shared | append only, never rewrite |

## Git rules for both lanes

- **Commit after every step.** Uncommitted work in two worktrees is one bad command
  from gone. Neither lane had committed anything as of step 6.
- **No `git stash`.** Worktrees share one stash stack, so a stash in one lane can
  swallow the other lane's work.
- No `git checkout` of the other lane's branch. No `git reset --hard`. No `git clean`.
- To test a baseline, use `git worktree add` a third throwaway tree, or `git show`.

# Merge checklist, for when Lane B folds into v2

- [ ] `GET /api/profile` in `src/server.js` selects a fixed column list that predates
      `photo_url` and `education`. Lane B serves those two off its own completion
      endpoint instead, correctly, to avoid touching Lane A's file. **At merge, fold
      them into the main profile SELECT and drop the duplicate**, so two endpoints do
      not drift apart.
- [ ] `tests/flows.js` hardcodes 52 teams / 38 ECE teams / 151 ECE students. Real
      figures are 53 / 39 / 154. Lane A fixes this — it already owns the file.
- [ ] `tests/onboarding.js` expects a seeded open project. Production has zero
      projects. **Do not fix this by deleting the assertion.** Step 4 replaces projects
      with tasks, so rewrite this test against tasks once step 4 lands.
- [ ] Student photos are stored locally under `uploads/photos/`, not Drive. They are
      not in `pg_dump`, so rebuilding the server loses them. Acceptable for nine days.
      Step 8 covers Drive for CVs and task submissions only.
- [ ] Run `make test` on the merged `v2` before step 10, not before.

---

# Step 9 corrections, found 17 Sep from the production dump

The step 9 written earlier assumed the CVs sit on this machine and are all PDFs.
Neither is true. Checked against `bootcamp-prod-2026-09-17-164842.sql.gz`:

| Finding | Number |
| --- | --- |
| Students with a CV | 76 of 209 |
| Stored as | `/uploads/resumes/v1-NN.ext` — real files **on the server**, not Drive links |
| PDF | 60 |
| **DOCX** | **16** |
| v2 (new CV) | 0 — correct, those arrive Day 8 |
| CV files in `uploads/` on this Mac | **0** |

Two things follow.

**The files are on the server.** `uploads/` locally is empty, so step 9 cannot run
from this machine against local files. Either pull the 76 files down first, or run
the copy on the server against a Drive token. Decide before starting.

**16 CVs are .docx and step 8's validator refuses them.** The upload path built in
step 8 sniffs magic bytes and allows JPEG, PNG, WebP and PDF only. A .docx is a ZIP
container — it will be rejected. Roughly one CV in five fails silently unless the
migration path gets its own allowlist of PDF plus DOCX. A DOCX is `PK\x03\x04` with
`word/` inside; do not widen the general upload path to accept all ZIPs.

# Step 8 fixes, before Lane B continues

- **Guard the Google URL overrides.** `GOOGLE_TOKEN_URL`, `GOOGLE_UPLOAD_URL` and
  `GOOGLE_FILES_URL` are read straight from the environment with no guard. A stray
  value in a server `.env` would send 209 students' CVs to someone else's host, and
  nothing would look wrong. Ignore all three unless `NODE_ENV === 'test'`, and log
  loudly when an override is in force.
- **`store_task_file()` is an unfinished handoff.** The upload records nothing until
  step 4 creates `tasks` and `task_submissions` and calls it. This belongs in Lane A's
  step 4 prompt, or files land on Drive attached to nothing.

## DOCX: the repo already had this right

`src/server.js` around line 1914 already accepts a resume as PDF **or .docx**, with
a magic-byte sniff (`%PDF`, or `PK` for the docx zip) and a separate, friendly
message for a legacy `.doc`:

> That is an old .doc file. Open it and use Save as → PDF or .docx.

Lane B's `src/routes/drive.js` allows `.jpg .png .webp .pdf` only. Its comment says
"same rule as sniff() in server.js", but `.docx` was left out, so the new path is
**stricter than what production already accepts**. That is a regression, not just a
migration problem: 16 of the 76 existing CVs are .docx, and the new CV students
upload on Day 8 will be too.

The fix is to match what already works, not to invent a new rule:

- Add `.docx` to `ALLOWED` and to the sniff, exactly as `server.js` does it.
- Keep refusing legacy `.doc`, and reuse the same wording, so a student gets one
  consistent message wherever they upload.
- Do not widen the rule to all ZIP files.
- Images stay images: a task with `submission_type = 'image'` still takes JPEG, PNG
  and WebP only. A document is allowed where a document is expected — CVs, and tasks
  with `submission_type = 'file'`.

Also confirmed: `chk_resume_v1_url` and `chk_resume_v2_url` already allow a
`drive.google.com` or `docs.google.com` URL, so step 9 can write Drive links
straight into `resume_v1_url` with no schema change.

---

# Step 3 follow-up: close the second door

Step 3 passed its own check, but the old global control survived it.

`POST /api/admin/quiz/:id/open` in `src/server.js` still exists, and `src/public/app.js`
still calls it in two places (around lines 1942 and 2581). It does three things that
now conflict with `releases`:

1. Sets `quizzes.is_open` with no department, so it opens for **both venues at once**.
2. Runs `UPDATE quizzes SET is_open = FALSE WHERE id <> $1`, closing every other quiz
   without touching `releases` — so the two sources of truth disagree on screen.
3. Refuses only an empty quiz, while the new screen refuses fewer than 5 questions.
   Two different rules for the same action.

`isOpenFor()` falls back to the old boolean when a quiz has no `releases` row, and only
currently-open items were backfilled. So for Day 2 onwards the old button silently wins.

**This is live tomorrow.** The old button is the one staff already know, and it is still
on screen. One click opens a quiz for all 209 students in both venues.

Fix: one code path, one screen.

- The old route delegates to the same function the Open tab uses, writing `releases`
  rows for **both** departments, and applying the same 5-question minimum.
- Drop the "close every other quiz" line, or express it through `releases`.
- Remove the old buttons from `app.js` so there is only the Open tab.
- A test that the old route can no longer open a quiz for one venue only, and cannot
  open a quiz with 4 questions.

---

# BLOCKED until morning: Drive access for the CV copy

Parked at 00:00 on 18 Sep. **Nothing on Day 1 depends on this.**

## Where it got to

| Check | Result |
| --- | --- |
| Service account key and JWT | PASS — token returned, `drive/v3/about` identifies the caller |
| Google Drive API enabled | PASS — a disabled API would 403, this returns 200 |
| Service account is a member of `ac-vcet` | **FAIL** |

`GET /drive/v3/drives` returns an empty list and `drives/0AEKdlFvN8BfeUk9PVA` returns
404. An empty list plus a 404 on the id means "not a member", not "lacks permission"
— a permissions problem on a visible drive would be 403.

## Why it could not be fixed

Vishnu's role on `ac-vcet` is **Content manager**, not **Manager**. Only a Manager can
add or remove Shared Drive members, which is why the Manage members dialog has no
add field at all. A Content manager *can* share a folder, which is how the service
account ended up on the `2-backend` folder — that share grants nothing here.

## Morning actions, in order of preference

1. Create a new Shared Drive (`ac-vcet-student-files` or similar). The creator is
   automatically Manager. Add the service account as Content manager, take the id
   from the URL, update `GOOGLE_SHARED_DRIVE_ID` in `.env`. Needs nobody else, and
   keeps 209 students' CVs out of a drive built for trainer material.
2. Ask whoever holds Manager on `ac-vcet` to add the account. None of the six current
   members is a Manager, so it is likely a Workspace admin or the original creator.
3. Have a Workspace admin promote Vishnu to Manager: Admin console → Apps → Google
   Workspace → Drive and Docs → Manage shared drives → ac-vcet → Manage members.

## Also in the morning

- **Rotate the service account key.** Key `ad8162d7…` was pasted into a chat. Delete
  it, create a new JSON, and put the new `private_key` into `.env` by hand.
- Remove the stray share on the `2-backend` folder once drive membership works, so
  nobody later assumes that share is what makes it run.
- `scripts/migrate-cvs.js` is written, tested (94 checks) and waiting. It is re-run
  safe, has no delete path, and copies v1 and v2. Run the dry run first.
- The live database has **83** CV rows, not the 76 in the dump, and the count is
  still rising as students hand in.
