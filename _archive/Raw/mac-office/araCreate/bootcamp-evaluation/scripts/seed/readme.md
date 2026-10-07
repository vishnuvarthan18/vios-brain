# SEED — the fake-data generator

T3-A1. Builds a full-volume local fixture so development and tests never need
real student data: **209 students, 53 teams, two venues, nine days.**

```sh
createdb bootcamp_seed
pg_dump --schema-only --no-owner --no-privileges -d bootcamp_local | psql -d bootcamp_seed
node scripts/seed/generate.js --db=bootcamp_seed --seed=1
```

| Option | Meaning |
| --- | --- |
| `--db=<name>` | Target database. Must be local and marked scratch |
| `--seed=<int>` | Default `1`. The same seed gives the same data, every time |
| `--quiet` | Print nothing but errors |

## It will not run against anything but a local scratch database

`guard.js` refuses unless **all three** hold: the host is loopback, the database
name matches `(^|_)(seed|fake|fixture|dev|scratch|harness)(_|$)`, and the name
the server actually connected to passes the same test. The generator deletes
every row before inserting, so pointed at the wrong database it is
`load-eee.sql` with a different name.

`bootcamp_local` and `bootcamp` are deliberately **not** scratch names —
`bootcamp_local` holds the real dump.

The guard is proved by sabotage in `tests/harness/`, not by having been read.

## Nothing here is a real student

Every name, roll number, phone number and email is invented —
`scripts/seed/names.js` holds the parts, and the generator combines them.
Phone numbers sit in a documentation range that is never allocated.
Fixtures get committed; real contact details must not be. This is checked:
zero of the 209 generated names, rolls, phones or emails appear in the real
dump.

## Determinism goes all the way down

The same seed gives the same rows **and the same primary keys** — every
sequence is reset before inserting. Without that, a test pinning `team_id = 7`
would break on reseed for no visible reason.

## The shapes, and why they are not uniform

A uniform fixture finds nothing. Each team draws a temperament — complete,
partial, struggling, silent — which it keeps for all nine days, because real
cohorts clump and the clumping is what the chase lists exist to find.

What the fixture deliberately contains:

| Shape | Why it is there |
| --- | --- |
| Students with **no team** | They must never be padded into the chase lists |
| Attendance `present=true` | Marked present |
| Attendance `present=false` | A **stored** absent — someone took the register |
| Attendance **row missing** | Nobody took the register. A different list |
| Quiz **opened, not submitted** | Chase list 4, distinct from "never opened" |
| Quiz **never opened** | Chase list 3 |
| Tasks with `dept NULL` | Both venues — the case most likely to leak |
| CV handed in, **no Drive copy** | An operational count, never names to chase |
| **No CV at all** | Names to chase |
| Post-assessment thinner than pre | Makes a naive percent-gain wrong, so it is testable |

**Releases are deliberately asymmetric.** Quizzes open for ECE only,
the pre-assessment for EEE only. A fixture where both venues are always open
cannot catch a venue leak — and the equivalent bug shipped in v2.

## It tells you what it does not fill

Any base table not in the generator's list is named in a warning at the end of
a run. Most such tables reference `students` or `teams` with `ON DELETE
CASCADE`, so they do not error — their rows simply vanish when `students` is
emptied, and the fixture looks healthy while carrying none of that feature's
data.

Lane A's `surveys`, `survey_questions` and `survey_answers` are the first case,
arriving at the `v3-dev` rebase. The warning fires on them today if the
migration is applied to a scratch database.

It is a warning, not an error: an unknown table is not a reason to refuse to
build a fixture, but it is a reason to know the fixture has a hole in it.

## One trap worth knowing

`grade_quiz_attempt()` ends with `submitted_at = COALESCE(submitted_at, now())`.
Calling it on an unsubmitted attempt marks it submitted, and the
"opened it but handed in nothing" list silently empties itself. The generator
grades only attempts it actually submitted.
