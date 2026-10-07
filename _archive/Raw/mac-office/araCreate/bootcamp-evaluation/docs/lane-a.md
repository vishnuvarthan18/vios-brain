# Lane A — the daily staff screens

**Read `docs/lane-rules.md` first. It is short and every rule in it exists to
stop your branch conflicting with the other three.**

Your branch: `lane-a`. Your report: `docs/lanes/lane-a.md`.

These are the screens the admin opens every morning and every evening. They
carry the heaviest behaviour of the fifteen left, which is why this lane has
only three screens.

| Order | Screen | Old function | Section |
| --- | --- | --- | --- |
| 1 | **Marking** | `page_marking` | 6.4 |
| 2 | **Admin home** | `page_admin` | 6.4 |
| 3 | **Register** | `page_register` | 6.4 |

Build them in that order. Marking first, because it is the one with the trap in
it, and you want your full attention on it.

---

## Marking — read this before you write anything

The comment box **saves on blur**, not only when a score button is clicked.

This was a real bug: a mentor typed a comment, did not touch a score, moved on,
and the comment was gone with no warning. If no score has been picked yet, the
box says **"Pick a score — the comment saves with it"** rather than throwing the
typing away. Each card shows saving / saved / the error, by itself.

The 0–5 score buttons carry a one-line rubric under them — mentors score
differently without one — and `aria-pressed`.

It covers both task hand-ins and project hand-ins: `/api/mentor/tasks`,
`/api/admin/submissions`, `/api/mentor/task-score`, `/api/mentor/score`.

## Admin home

Keep the **Today box with Open / Close**. Before it existed, opening the day's
quiz was three clicks deep inside a table, every morning.

## Register

Attendance taken by the admin rather than the lead, for a whole day at a time.
`/api/admin/attendance/day/:day` and `/api/admin/attendance/:day/:student_id`.

Read what the Attendance screen already does — `web/src/pages/Attendance.jsx`
is built and this is its admin-side twin. **Do not import from it and do not
edit it.** Copy the shape into your own file.

---

When all three are done, report and stop. Do not start another lane's screens,
even if you finish first.
