# v3 — restructure plan

Turning the bootcamp app from a live-event build into a proper product.

**Decided 19 Sep 2026:**

- Organise all four — product flow, code/repo structure, docs, way of working —
  plus the **UI/UX structure**.
- **Dev and deploy only when students are not working.**
- **Goal: a reusable product** for future batches and other colleges.
- The app shows numbers but not **who**. Student and team profile pages are
  missing. Sections 4 and 5 below.

The live run (VCET Basic Electronics, 18–26 Sep) must not be disturbed.

---

## 1. Why it feels messy

Nothing here is a code-quality complaint. It is what happens when ten features
are added in one night onto a live app.

**Navigation.** The admin has one flat list of 16 items:

> Home · Marking · Projects · Leaderboard · Quiz results · Open · Register ·
> Quiz now · Tasks · Tinkercad · Assessment · Students · Teams · Progress ·
> Staff · Quizzes

No grouping. Daily actions, setup screens and reports sit side by side. Every
new feature became a new nav item because there was no structure to fit it into.

**One concept, many names.**

- Task, Project, Quiz, Assessment, Tinkercad — five nav items, one idea:
  *work given to students*.
- Quiz, Quiz now, Quiz results, Quizzes — four items, one idea.
- Marking, Progress, Leaderboard, Quiz results — four report pages, no report
  section.

**Three ways to do one thing.** Opening something can happen on the Open tab, on
the Home "Today" box, or from the thing's own screen. Three paths, one action —
this is where the EEE-quiz-leaked-to-ECE bug came from.

**Numbers with no names.** Every screen reports a count and stops there. See
section 4 — this is the biggest day-to-day problem, not the nav.

**No profile pages.** There is no one page that answers "how is this student
doing" or "how is this team doing". See section 5.

**Code.** The whole API is `src/server.js`. The whole front end is
`src/public/app.js` + `app.css`. Nothing is a module, so nothing can be tested or
replaced on its own.

**Hardcoded to this one batch.** Start date, the two departments, nine days,
the join code, the college — all baked in.

**Docs in two places**, overlapping and partly stale — `handover.md` still says
52 teams / 206 students and "no file uploads", both untrue since v2.

---

## 2. Name things once

The nav almost writes itself once the words are fixed. One vocabulary, used in
the database, the API, the URL and the screen heading.

| Word | Means |
| --- | --- |
| **Program** | One bootcamp run — college, dates, number of days, join code |
| **Venue** | A room running the program (today: EEE, ECE). Replaces "department" |
| **Team** → **Student** | Unchanged |
| **Activity** | The one parent word for anything given to students |
| **Release** | An activity opened for one venue. Already a table — make the UI match it |
| **Submission** → **Score** | Unchanged |

Activity has five types, one shape: **Task · Project · Quiz · Assessment ·
Survey**. Same create form, same open path, same hand-in path, same scoring
path — only the type differs. Attendance stays its own thing. Tinkercad becomes
a field on a team, not a page.

---

## 3. New navigation

### Admin — 5 groups instead of 16 items

| Group | Holds | Its one job |
| --- | --- | --- |
| **Today** | Run-the-day board | What must I do right now |
| **Content** | Tasks, Projects, Quizzes, Assessments, Surveys | Make the work |
| **People** | Students, Teams, Staff, Register | Who is here |
| **Live** | Release board, Attendance, Marking queue | Run the day |
| **Reports** | Progress, Leaderboard, Quiz results, Attendance, Exports | How is it going |

Sub-tabs inside each group. Nothing is deleted — it is put somewhere.

### One release board

Every openable thing in **one table**. Rows = activity. Columns = one per venue.
Cell = Closed / Open / opened-at time, and clicking the cell toggles it.

That removes the three-paths problem permanently, and it is the same screen
whether a program has two venues or six.

### Student — 5 items, never more

Home · My work · My profile · My team · Leaderboard

"My work" holds today's survey, task, project, quiz and assessment in one list.
A student should never have to know which type their work is.

- **Team lead** adds Attendance, inside My team.
- **Mentor:** Home · Marking · My teams · Leaderboard.

### The rule going forward

**A new feature does not get a new nav item.** It goes inside one of the five
groups, or it is not built.

---

## 4. See who, not just how many

Today the app answers *how many*. Running a room needs *who*. "43 handed in"
cannot be acted on. "These 10 have not, here are their phone numbers" can.

The data is already there — every submission carries a student id, a team id and
a timestamp. This is a screen gap, not a database gap.

### The rule

**No number is a dead end.** Every count anywhere in the product is a link, and
clicking it opens the named list behind it. Including the negative side.

| Where you see | Clicking gives |
| --- | --- |
| "43 of 53 handed in" | the 43, **and** the 10 who did not |
| "Attendance 86%" | the students marked absent today |
| "Quiz average 3.2" | every attempt, and who never opened it |
| "Profile 70% complete" | the exact fields still blank |
| "12 waiting to be scored" | the 12 submissions, each one click from scoring |

Every list carries name, roll number, team, venue, phone, email and time, and
every list has **Export CSV** and **Copy all phone numbers** — because chasing a
room is the actual job.

### Activity detail page — the new core screen

One page per activity, filtered by venue.

- **Tabs:** Done · Not done · Scored · Not scored
- Rows are students or teams, depending on who owns that activity
- Columns: name · team · venue · handed in at · what they handed in (link opens
  it) · score · who scored it
- Sort by anything, filter by venue and team, export the view

### The completion matrix

- Rows = students (or teams). Columns = each day, or each activity.
- Cell = done · not done · scored · absent, as a colour block.
- A row that is mostly empty is a student falling behind. You see it in one
  second instead of opening nine reports.
- Filter by venue, team, mentor. Click any cell to open that submission.

This one screen replaces most of what Progress, Marking and Quiz results do
separately today.

---

## 5. The pages that are missing

### Student profile — full page

- **Header** — photo, name, roll, department, year, team + role, venue, phone,
  personal email, and a **Copy contact** button.
- **Completion bar** with the missing items named, not just a percentage.
- **CVs** — old CV and new CV, each opening the Drive file, with dates.
- **Goal** — what they wrote on Day 0.
- **Attendance strip** — nine day boxes, present / absent / not yet.
- **Work table** — every activity: type, day, done or not, when, score.
- **Quiz detail** — per quiz: score, questions right, questions never answered.
- **Survey answers** — every daily survey they answered.
- **Assessment** — pre score, post score, **percent gain** (never raw marks;
  the question sets differ).
- **Daily posts** — the timeline. **Admin only.** Mentors and teammates do not
  see post content. That decision stands.
- **Actions** — copy email, copy phone, export this student.

The student's own profile shows the same about themselves, minus admin actions.

### Team profile — full page

- **Header** — team code, name, venue, track, lead, mentor, Tinkercad code,
  Drive folder link.
- **Rank and points** — current rank, total, and the day-by-day breakdown.
- **Members** — each row showing that student's own completion, each name
  opening their profile.
- **Member × day matrix** — attendance and work in one grid.
- **Submissions** — everything handed in, with links and scores.
- **Actions** — export the team, copy all phone numbers.

### Where they hang off

Every student name and every team name anywhere in the app links to these pages.
A name that is plain text and not a link is a bug, from now on.

---

## 6. Product flow

**Setup (before the program)** — create program → venues → days → import roster
(CSV, additive) → teams → staff → Drive root → join code.

**Day 0 — onboarding** — student signs in → old CV → goal → profile →
pre-assessment.

**Each day** — morning: admin opens the day's activities **per venue** →
students do the work → lead marks attendance → evening: mentors score →
admin closes → admin loads tomorrow's quiz and survey questions.

**Last day** — post-assessment → new CV → journey page → exports.

Every one of those verbs should be a card on **Today**, in order, ticking off as
it is done. Today is the only screen anyone has to learn.

---

## 7. Code structure

```
src/
  config/          program config loader (no hardcoded batch data)
  db/              schema, numbered migrations, seeds
  modules/
    auth/          routes + service + queries
    programs/
    venues/
    students/
    teams/
    activities/    tasks, projects, quizzes, assessments, surveys — one shape
    releases/      the single gate: is_open_for()
    submissions/
    scoring/
    attendance/
    reports/       every list behind every number lives here
    storage/       Drive adapter behind one interface
  web/
    pages/         one file per screen
    components/
    design-system/
  server.js        wiring only
tests/
  unit/ · api/ · e2e/
docs/
scripts/
```

**Rules that stop the old bugs coming back**

- One module owns its tables. No cross-module SQL.
- `releases.is_open_for()` is the **only** gate. Nothing checks day or
  department directly, anywhere.
- One name per field. `is_lead` everywhere — the lead bug was a field read by
  two different names.
- Every route declares its role, and every role has a test that **signs in**.
- No batch data in code. It comes from the program row.
- Storage behind one interface, so server disk vs Drive is a config choice.
- **Every count endpoint has a matching list endpoint.**

---

## 8. Making it reusable

Everything keys off `program_id`.

- `programs` table — college, name, start date, number of days, join code, staff
  password hash, Drive root.
- `venues` table replaces the EEE/ECE enum. A program can have 1 venue or 6.
- **Roster import becomes a screen**, CSV upload, **additive only**.
  `load-eee.sql` and `load-ece.sql` are deleted from the repo.
- Points rules move to program settings.
- Branding — college name, logo — a program field.

A second batch is then created in the UI in ten minutes, with no code change.

---

## 9. Docs

```
docs/
  readme.md      the index — start here
  product/       flow, information architecture, screen inventory
  design/        design system rules
  dev/           architecture, conventions, testing
  ops/           deploy, runbook, known issues, backups
  history/       v1 and v2 build notes — archive, not current
```

First job: fix the stale facts. 53 teams / 209 students, uploads and Drive now
exist, and every deploy section must show `git archive`, not `rsync`.

---

## 10. Way of working

See `docs/agent-working-rules.md`, plus:

- **Deploy window: students not working.** The one mid-morning deploy cost 19
  seconds of downtime and a student's photo.
- Two lanes, one worktree each, report before deploy.
- Fresh dump before every deploy, and check it holds the rows you expect.
- Watch the log across the restart, not just that the service came back.
- Before reporting a feature done: **go and look at what the old path still
  does.** Eight serious bugs, and the new code was right every time.
