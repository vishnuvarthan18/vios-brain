# CHASE LISTS

Who has not done today's work, as CSV files you can open and ring from.
Read-only: no INSERT, UPDATE, DELETE or DDL anywhere, and the session is set
read-only so the database itself would refuse one.

```sh
./scripts/load-local-dump.sh ~/araCreate/dumps/bootcamp-2026-09-19.sql.gz
./scripts/chase/run.sh 2            # day 2, into ./chase-lists/
./scripts/chase/verify.sh 2         # prove the counts a second way
```

| File | What it lists |
| --- | --- |
| `_chased.sql` | Who counts as chaseable, and how name, phone and venue are read. Every query starts here |
| `00-flags.sql` | Counts that must never become names, and the state that says what the lists MEAN today |
| `01-no-task-handin.sql` | Every member of a team with no hand-in for an open task, naming the task |
| `02-no-project-handin.sql` | Every member of a team with no hand-in for an open project |
| `03-quiz-never-opened.sql` | No attempt row for today's quiz |
| `04-quiz-opened-not-submitted.sql` | Attempt row, `submitted_at` still NULL |
| `05-marked-absent.sql` | A row exists and says `present = false` |
| `06-no-attendance-mark.sql` | No row at all, plus whether the register was even open |
| `07-incomplete-profile.sql` | Incomplete profile, naming the missing fields |
| `08-no-cv.sql` | `resume_v1_url IS NULL` — nothing handed in |
| `09-no-daily-post.sql` | No daily post today. The only per-student hand-in |

Every CSV starts with the six columns that matter: name, roll, team, venue,
phone, personal email. Phone is the profile's, falling back to the roster's,
the same way the app resolves it.

## Read 00-flags first

An empty list has two completely different meanings, and only the flags tell
them apart:

- nobody is outstanding, or
- **nothing was opened for that venue today**, so nobody could have done it.

On the 19 Sep dump at Day 2, the attendance register is closed for both venues
and no quiz has a single question. Lists 3, 4 and 6 are therefore empty or
total, and none of it is any student's fault.

## What is deliberately NOT a name

Three things are counted in `00-flags.sql` and never appear as people to ring:

- **A CV handed in but not yet copied to Drive.** The student has done their
  bit; `scripts/migrate-cvs.js` has not run. Chasing them would be wrong.
- **A student who pasted their own Drive link.** Done. Appears nowhere.
- **A student with no team.** They cannot make a team hand-in, so they would
  sit in lists 1 and 2 every day of the bootcamp.

## Two things that decide a list's meaning

**A day is a number, not a date.** `(CURRENT_DATE - start_date) + 1`, worked out
in the database's timezone — which is UTC on the server and IST on this Mac.
`run.sh` therefore takes the day explicitly and prints what it would have
computed, so the two can be compared before anyone is rung.

**Tasks and projects belong to the team.** `task_submissions` is
`UNIQUE (task_id, team_id)`; any member may hand in for everyone. So lists 1 and
2 name every member of an outstanding team — which is the list worth having,
because chasing means ringing the team. Only `daily_posts` is per student.
