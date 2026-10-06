# v3 UI shape check — stubs against the real server

The v3 screens were built against stubbed API responses shaped by reading
`src/server.js`. This is the check of those assumptions against a server
running on a real database.

**How it was run.** `bootcamp_old` is the only local database matching the
current code (39 tables, 209 students, 824 survey answers, 427 attendance
rows). It was copied to a scratch database `shapecheck` — the original was
never written to — and the server run against the copy on port 3123. Signed in
as three real accounts and opened all 26 screens:

| Role | Account | What they are |
| --- | --- | --- |
| Student | `saravanasuthans@gmail.com` | ECE-T34-CODETEAM, not a lead |
| Team lead | `guruvishnu5123@gmail.com` | ECE-T03-OHMFORCE, lead |
| Admin | `vishnu@aracreate.group` | admin |

The data puts the bootcamp on **Day 2 of 9**. `quiz_questions` is empty, so no
quiz can be opened — the quiz screens were checked in their no-quiz state only,
and that is called out per screen below.

---

## The differences

Nine, in the order they matter. Each says what the code assumed, what the
server actually sends, and what it did on screen.

### Why the stub suites never caught 1–3

The fixture in `shot-reports.mjs` invented `id` and `team`. The screen read
`id` and `team`. **Both were wrong in the same way, so every assertion passed**
while the real screen showed an empty column on 209 rows and a dead button.

A stub written from the same guess as the code tests the guess, not the
server. The fixture now carries field names copied from a real response, and
there is a new assertion that the Team column is actually populated — the check
that was missing, not just the value that was wrong.

---

### 1. Progress: the whole Team column was empty · FIXED

**`GET /api/admin/progress`** → `rows[]`

| Assumed | Actually |
| --- | --- |
| `r.team` | `r.team_code` (and `r.team_name`) |

Every one of 209 rows showed `—` under Team. No error, no warning: a missing
field renders as nothing. This is the exact failure a shape check exists to
find, and the one a screenshot cannot.

### 2. Progress: the Open button did nothing · FIXED

**`GET /api/admin/progress`** → `rows[]`

| Assumed | Actually |
| --- | --- |
| `r.id` | `r.student_id` |

`setJourneyId(undefined)` left the list on screen. **The Journey screen was
unreachable from the UI** — an API with no screen to reach it, which is the
mistake section 8 of the migration doc says was made six times already.
It also meant every row shared `key={undefined}`.

### 3. Progress: search by team found nothing · FIXED

Same root cause as 1. Searching `OHMFORCE` returned **0 rows** where four
students are on that team.

### 4. Survey proof printed "+-0.5%" · FIXED

**`GET /api/admin/survey/proof`** → `gain.pct`

| Assumed | Actually |
| --- | --- |
| always ≥ 0, so a `+` could be hardcoded | **`-0.5`** on this data |

Five occurrences on screen. Learning went *down* between the two rounds here,
and the screen reported it with a doubled sign. The sign now comes from the
number.

### 5. Staff and Teams: "The 1 teams they mentor" · FIXED

**`GET /api/admin/staff`** → `teams`, **`GET /api/admin/teams`** → `members`

| Assumed | Actually |
| --- | --- |
| number | **string** (`"5"`) — Postgres `COUNT(*)` |

`deleting.teams === 1` is `false` for `"1"`, so the plural branch never fires:
a mentor with exactly one team reads "The 1 teams they mentor lose their
mentor." No such mentor exists in this data, so it is latent — but plural
agreement is a named rule in section 7, and the comparison is wrong.

### 6. Counts arrive as strings, widely

**`/api/admin/overview`** — every field: `teams`, `students`, `unassigned`,
`teams_without_lead`, `logged_in`, `waiting_to_score`, `quizzes_empty`
**`/api/admin/attendance`** — `present_count`, `total_marks`
**`/api/admin/quizzes`** — `questions`, `teams_done`, `students_done`
**`/api/leaderboard`** — `rank`, `projects_scored`, `quizzes_done`
**`/api/my-team`** — `team.rank`

| Assumed | Actually |
| --- | --- |
| numbers | strings |

**No change needed.** Every arithmetic use was already wrapped in `Number()`,
and every display use is a string either way. Recorded because the next person
to add a comparison here will get it wrong if it is not written down.

### 7. Staff `/api/me` has no `is_lead` · no change

| Assumed | Actually |
| --- | --- |
| `me.is_lead` present for everyone | absent for staff |

`roleLabel()` reads it and gets `undefined`, which is falsy, so the answer is
right by accident. Left alone — `pagesFor()` checks `kind === "staff"` first,
so nothing downstream depends on it — but it was an assumption, not a fact.

### 8. `/api/admin/assessments` rows carry no `team_code` on a survey-type set

`rows[].correct_count` and `score_percent` are `null` where the questions have
no right answer. The screen already keys its caption off the **count**, not the
average, which is the fix recorded in section 6.5. Confirmed correct on real
data: ECE reads "154 of 154 answered", not "Nobody has sat it yet".

### 9. `/api/admin/projects` has no `id`

Rows are keyed `(day, title)` and deleted by body, not by id — which is what
the screen does. Confirmed, no change.

---

## What could not be checked

- **Quiz, Quiz now, Quiz results, and the quiz builder's locked state.**
  `quiz_questions` is empty in this database, so no quiz can be opened. The
  screens were confirmed in their empty state (`{"quiz":null}` → "No quiz is
  open"), and Quiz now against a quiz with zero questions. The question flow,
  the 30-second clock, the resume-mid-quiz path and the answer-save retry are
  **still only checked against stubs.**
- **Assessment (student).** `/api/assessment/open` returns
  `{"assessment":null}` — none is open on Day 2. Empty state only.
- **Any write.** Nothing was POSTed, PUT or DELETEd against the copy. Hand-ins,
  marking, attendance saves, releases and the bulk loaders are read-path
  verified only.

---

## After the fixes

- **31/31 screens** render clean against the real server, signed in as all
  three roles. Progress grew from 13,095 to 16,455 characters of rendered text
  once the Team column and the Journey route started working.
- **272/272 stub assertions** pass, including the new Team-column check.
- The scratch database `shapecheck` can be dropped; `bootcamp_old` was never
  written to.

## Still worth doing

1. **Load real quiz questions** and repeat this for the quiz flow. Four of the
   26 screens are still stub-only on their main path, and the quiz is the one
   the migration doc calls the hardest screen.
2. **Check the write paths.** Nothing was saved during this pass.
3. `src/db/schema.sql` declares 15 tables; the live database has 26. The
   schema file is out of date by 11 tables — tasks, surveys, assessments,
   releases and their children.
