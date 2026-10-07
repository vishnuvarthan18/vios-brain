# Lane B — opening work, and the quiz side

**Read `docs/lane-rules.md` first. It is short and every rule in it exists to
stop your branch conflicting with the other three.**

Your branch: `lane-b`. Your report: `docs/lanes/lane-b.md`.

| Order | Screen | Old function | Section |
| --- | --- | --- | --- |
| 1 | **Open** | `page_open` | 6.4 |
| 2 | **Quizzes** | `page_quiz_admin` + `quiz_builder` | 6.5 |
| 3 | **Quiz now** | `page_quizlive` | 6.4 |
| 4 | **Tasks** | `page_tasks_admin` + `task_day_block` + `task_form` | 6.5 |

---

## Open — the most dangerous screen in the app

This is the release board: rows are things, columns are venues, and a cell says
Closed / Open / the time it was opened.

**Opening something for one venue must never expose it to the other.** That
exact bug happened — an EEE quiz became visible to ECE — and it happened because
there were three different paths to open something and they did not agree. This
screen exists to be the only path.

Everything goes through `/api/admin/releases`. Nothing on this screen checks a
day or a department by itself.

In your check list, say plainly: *open something for EEE only, then sign in as
an ECE student and confirm it is not there.* That is the check that matters.

## Quizzes and Tasks — the bulk loaders

Both take pasted lines. The rules are the same for both, and they are not
optional:

- One item per line
- **The parsed list is shown before saving**, so a mistake is caught before it
  is stored
- A bad line is **rejected with a clear message**, never silently skipped
- The count added is shown afterwards

Quiz question format: `question | A | B | C | D | correct letter`. Trim spaces.
The correct letter is case-insensitive.

## Quiz now

The live view while a quiz is running: who has started, who has finished.
`/api/admin/quiz/:id/live`.

---

When all four are done, report and stop. Do not start another lane's screens,
even if you finish first.
