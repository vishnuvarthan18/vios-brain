# Migration ledger

T3-A4. The real run order for every migration in `src/db/migrations/`, and how
each one was verified as already applied.

**This is documentation. No migration was changed and nothing was run against
any database.** Every check below is a `SELECT` against the local
`bootcamp_local`, which is loaded from the dump.

`src/db/migrations/readme.md` lists **9**. The directory holds **15** on
`v3-dev`, and **17** on `survey` once Lane A's two survey files land. The six
from 18–19 Sep had no documented run order; they do now.

---

## There is no migration table

Nothing records what has been applied — no `schema_migrations`, no Flyway, no
Liquibase. So "is this applied?" can only be answered by looking for something
the migration creates. That is what the **Marker** column is: a specific object
whose presence means that file has run.

Three kinds of marker, in order of how much they prove:

| Kind | What it is | Weakness |
| --- | --- | --- |
| **column** | a column the migration adds | strong; a column is not created by accident |
| **table** | a table or sequence it creates | strong |
| **definition** | text inside a view's own body | the only option for a view-only migration, because rebuilding a view leaves no new object |

The two view-only migrations both rebuild `v_team_projects`, so neither creates
anything. They are told apart by reading the live view's definition and looking
for a phrase only the later one introduces. This is the method Track 1 used.

---

## Run order

Alphabetical order is **not** dependency order. This is.

| # | File | Marker (kind) | Applied | Depends on |
| --- | --- | --- | --- | --- |
| 1 | `2026-09-17-progress-view` | `v_student_progress` exists (definition) | ✅ | — |
| 2 | `2026-09-17-a-attendance-audit` | `attendance_audit` table | ✅ | — |
| 3 | `2026-09-17-a-releases` | `releases` table | ✅ | — |
| 4 | `2026-09-17-a-closed-by-default` | a Day 1 `attendance` row in `releases` (weak — see below) | ⚠️ | **3** — it seeds a `releases` row |
| 5 | `2026-09-17-a-quiz-per-student` | `quiz_attempts.student_id` (column) | ✅ | — |
| 6 | `2026-09-17-a-tasks` | `tasks` table | ✅ | **5** — `recalc_team_points_for()` calls `recalc_team_quiz_points()` |
| 7 | `2026-09-17-a-assessments` | `assessment_questions` table | ✅ | — |
| 8 | `2026-09-17-b-profile-completion` | `student_profiles.photo_url` (column) | ✅ | — |
| 9 | `2026-09-17-b-cv-drive-links` | `student_profiles.resume_v1_drive_url` (column) | ✅ | **8** — reads `student_profiles.photo_url` |
| 10 | `2026-09-18-a-assessment-survey` | `assessment_questions.correct` is nullable (constraint) | ✅ | **7** — alters `assessment_questions` |
| 11 | `2026-09-18-a-tinkercad-code` | `teams.tinkercad_code` (column) | ✅ | — |
| 12 | `2026-09-18-b-project-formats` | `projects.submission_type` (column) | ✅ | **3, 6** — rewrites `releases`' item-type check, which must list `task` |
| 13 | `2026-09-18-c-project-open-per-dept` | superseded — see below (definition) | ⚠️ | **3** — the view reads `releases` |
| 14 | `2026-09-19-a-project-groups` | `projects.group_id` (column), `project_group_seq` | ✅ | **3, 12** — rewrites `chk_releases_item_id` again |
| 15 | `2026-09-19-b-project-view-by-group` | `v_team_projects` body contains `group_id` (definition) | ✅ | **14** — the view selects `group_id` |
| 16 | `2026-09-19-c-surveys` | `surveys` table | — | — |
| 17 | `2026-09-19-c-per-student-tasks` | `task_submissions.per_student` (column) | ✅ | **6** — redefines `team_task_points_for_day()` |
| 18 | `2026-09-19-d-task-submission-orphans` | `task_submission_orphans` table | ✅ | **6** |
| 19 | `2026-09-19-e-load-task8-orphans` | 50 rows in `task_submission_orphans` | ✅ | **18** |
| 20 | `2026-09-19-f-scoring-v3` | `scoring_settings` table | **NOT APPLIED ANYWHERE** | **3, 6, 16** — releases, tasks, surveys |
| 21 | `2026-09-23-a-rank-by-per-member` | `scoring_settings.rank_by` default is `per_member` (definition) | local only | **20** — updates the `scoring_settings` row |
| 22 | `2026-09-23-b-certificates` | `certificates` table | local only | — (base schema only) |
| 23 | `2026-09-23-c-certificates-on-disk` | `certificates.file_path` (column) | local only | **22** — alters `certificates` |
| 24 | `2026-09-23-d-remove-test-data` | no `TEST%` roll numbers remain | local only | — (data only; NO down, restore from a dump) |

### Number 20 — scoring v3, 19 Sep

Add-only: three new tables, four new columns, six functions, four views.
Nothing existing is dropped, renamed or rewritten, so it can be applied to a
live database without changing a single number anyone currently sees.

Built and tested on a fresh database in a cloud session: applied, applied
twice, rolled back, rolled back twice, re-applied. Clean each time.

**Its `down` is not a no-op.** It drops the `points_minutes` columns, so a
down-then-up cycle leaves them empty — no deadlines, every late hand-in becomes
full points, and totals go UP. The down file says so and carries the backup
commands. Do not treat a rollback here as free.

Lane A's `2026-09-19-c-surveys` and `-surveys-down` are on `survey` and are not
part of this ledger until that branch merges.

### The verification queries

Column and table markers:

```sql
SELECT EXISTS (SELECT 1 FROM information_schema.columns
                WHERE table_name = 'quiz_attempts' AND column_name = 'student_id');
SELECT to_regclass('tasks') IS NOT NULL;
```

The two view-only ones, by reading the view's own body:

```sql
SELECT pg_get_viewdef('v_team_projects'::regclass) LIKE '%group_id%';
```

---

## Number 4's marker is weak, and says so

`2026-09-17-a-closed-by-default` seeds one `releases` row per venue for Day 1
attendance, with `ON CONFLICT DO NOTHING`. It creates no column and no table,
so that row is the only evidence it ran.

The trouble is that **an admin opening Day 1 attendance by hand creates the
same row**. Four such rows exist locally, which is consistent with the
migration having run — and equally consistent with it never having run and
staff having opened attendance themselves.

So: ⚠️, not ✅. It is additive and safe to re-run, so if there is ever doubt,
re-running it is cheap and settles nothing either way. What would settle it is
a migration table, which is a Phase B argument.

## Number 13 is the one to understand

`2026-09-18-c-project-open-per-dept` and `2026-09-19-b-project-view-by-group`
**both drop and recreate `v_team_projects`**. Neither creates any other object,
so neither leaves a marker of its own.

The live view's body contains `group_id`, which only `19-b` introduces. So:

- `19-b` has certainly run, and is what the database has now.
- `18-c` **cannot be confirmed either way**, because `19-b` overwrote the only
  evidence it would have left. Running `18-c` today would be a **regression** —
  it would replace the current view with the older one and drop `group_id` from
  it, breaking whatever reads that column.

**Therefore: on a database that already has the current view, skip 13 and run
14 → 15.** On a database being built from nothing, run 13 in its place in the
order; it is harmless there and 15 replaces it a moment later.

`18-c`'s text contains no `group_id` at all, so this is not a guess: running it
would produce a `v_team_projects` without that column. Two live routes read the
view with `SELECT *` —

```
src/server.js:444    a student's own team's projects
src/server.js:1542   a mentor reading one team's projects
```

— so the column would simply stop arriving at the front end, with no error
anywhere to say why.

This is the practical hazard in the whole set: the run order is not a sequence
you can replay blindly on a live database, because two files fight over one
view. That is worth fixing in Phase B, not tonight.

---

## Re-running

The `readme.md` says all nine of its migrations are safe to run twice. The six
undocumented ones carry idempotence guards too — `IF NOT EXISTS`,
`OR REPLACE`, `DROP CONSTRAINT IF EXISTS` before `ADD CONSTRAINT`:

| File | Guards |
| --- | --- |
| `2026-09-18-a-assessment-survey` | 6 |
| `2026-09-18-a-tinkercad-code` | 4 |
| `2026-09-18-b-project-formats` | 6 |
| `2026-09-18-c-project-open-per-dept` | 2 |
| `2026-09-19-a-project-groups` | 4 |
| `2026-09-19-b-project-view-by-group` | 1 |

The counts are a smell test, not a proof: a file can be guarded and still not be
safe to re-run, which is exactly the case with **13**. Guarded means "will not
error"; it does not mean "will not change anything".

`2026-09-19-a-project-groups` also carries an `INSERT INTO releases`. It is
written to be a no-op on a second run, but it is the only one of the six that
writes a row rather than only changing shape, so it is the one to read before
re-running.

---

## Most of them have no `down`

Rule 12 of the brief: *every migration is additive and has a working `down`.*

Of the 15, **3** carry a documented down section — `a-closed-by-default`,
`a-quiz-per-student`, `a-tasks`. The other **12 have none**, including all six
of the undocumented ones.

They are all additive, so the risk is not data loss; it is that there is no
written way back if one of them turns out to be wrong. Reversing
`2026-09-19-a-project-groups` by hand, for instance, means knowing to drop a
column, an index, a sequence **and** to put back the previous
`chk_releases_item_id` — which the file itself replaced, and whose earlier text
now exists only in `2026-09-18-b-project-formats`.

**Not a tonight problem, and not this lane's to fix**: writing downs for twelve
already-applied migrations is Phase B work, and doing it now would mean editing
files that have already run in production.

---

## What this ledger does not claim

- It does not prove a migration ran **completely**. A marker proves the
  statement that creates it ran. A file that failed halfway would still show its
  marker if the marker came first. None of these are written that way — each is
  a single `BEGIN … COMMIT` — but the ledger's evidence is the marker, not the
  transaction.
- It says nothing about the **production** database. Every check here ran
  against the local copy loaded from the dump. That is the whole of what this
  lane may touch.
