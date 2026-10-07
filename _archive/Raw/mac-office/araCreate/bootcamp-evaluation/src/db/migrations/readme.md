# MIGRATIONS

> **This file lists 9. The directory holds 15.**
> The six from 18–19 Sep are missing here, and two of them fight over the same
> view. The full run order, how each one was verified as applied, and the one
> that must **not** be re-run on a live database, are in
> **`docs/migration-ledger.md`**. Read that before running anything.

Run these **in the order below**, one at a time, checking each before starting
the next. Alphabetical order is *not* dependency order — two of them fail if
run out of turn, and the failure is a clean rollback rather than a mess, but
it is still a stop.

| # | File | Depends on |
| --- | --- | --- |
| 1 | `2026-09-17-progress-view.sql` | — |
| 2 | `2026-09-17-a-attendance-audit.sql` | — |
| 3 | `2026-09-17-a-releases.sql` | — |
| 4 | `2026-09-17-a-closed-by-default.sql` | **3** — it seeds a `releases` row |
| 5 | `2026-09-17-a-quiz-per-student.sql` | — |
| 6 | `2026-09-17-a-tasks.sql` | **5** — `recalc_team_points_for()` calls `recalc_team_quiz_points()` |
| 7 | `2026-09-17-a-assessments.sql` | — |
| 8 | `2026-09-17-b-profile-completion.sql` | — |
| 9 | `2026-09-17-b-cv-drive-links.sql` | **8** — reads `student_profiles.photo_url` |
| … | *(the 2026-09-18 migrations, in date order)* | — |
| n | `2026-09-19-c-per-student-tasks.sql` | **6** — redefines `team_task_points_for_day()` from `a-tasks` |
| n+1 | `2026-09-19-d-task-submission-orphans.sql` | **6** — references `tasks` and `teams` |
| n+2 | `2026-09-19-e-load-task8-orphans.sql` | **n+1** — loads 50 rows into that table |

`2026-09-19-c-per-student-tasks` depends on `a-tasks` and on nothing else. In
particular it does **not** depend on either 2026-09-19 project migration: it
touches no project object at all, so it can be run on a database where those
have not been applied. It was deployed that way on 19 Sep.

As one line, from the repo root:

```sh
for m in 2026-09-17-progress-view \
         2026-09-17-a-attendance-audit \
         2026-09-17-a-releases \
         2026-09-17-a-closed-by-default \
         2026-09-17-a-quiz-per-student \
         2026-09-17-a-tasks \
         2026-09-17-a-assessments \
         2026-09-17-b-profile-completion \
         2026-09-17-b-cv-drive-links; do
  echo "== $m"
  psql -v ON_ERROR_STOP=1 -d "$DB" -f "src/db/migrations/$m.sql" || break
done
```

## What they are

| File | What it does |
| --- | --- |
| `progress-view` | The admin progress list, as a view |
| `a-attendance-audit` | Where a mark came from, and every admin change to one |
| `a-releases` | Opens each thing per department instead of for everyone at once |
| `a-closed-by-default` | New work starts closed; Day 1 attendance starts closed |
| `a-quiz-per-student` | Every student sits the quiz, on a per-question clock |
| `a-tasks` | A day carries several tasks and is still worth five points |
| `a-assessments` | The pre and post assessments, measured but never scored |
| `b-profile-completion` | The photo and education line, and the completion weights |
| `b-cv-drive-links` | Drive links for the CVs, beside the server paths |
| `c-per-student-tasks` | A task can be handed in by each student; no student can overwrite another |
| `d-task-submission-orphans` | Somewhere to hold hand-ins the overwrite bug detached from their row |
| `e-load-task8-orphans` | The 50 task-8 files recovered from Drive, loaded into it |
| `f-scoring-v3` | Automatic marking: what each thing is worth, the points timer, admin adjustments, the live leaderboard views. Add-only; switches nothing over. Its `down` destroys the timers — read the file |

## Rules these were written to

- **Add-only.** No table is dropped, no column is dropped, nothing is renamed.
  Two constraints go, and each is the whole point of its change:
  `quiz_attempts`' UNIQUE (quiz_id, team_id), and `task_submissions`'
  UNIQUE (task_id, team_id) — the second is what let one student's hand-in
  overwrite another's.
- **Safe to run twice.** Every one of them. Re-running is a no-op, and in
  particular cannot reopen something staff have since closed by hand.
- **Each ends with a CHECKS section** of commented-out verification queries.
  Run them after the migration rather than trusting the absence of an error.
