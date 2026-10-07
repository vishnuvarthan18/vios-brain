# Lane D — people and reports

**Read `docs/lane-rules.md` first. It is short and every rule in it exists to
stop your branch conflicting with the other three.**

Your branch: `lane-d`. Your report: `docs/lanes/lane-d.md`.

| Order | Screen | Old function | Section |
| --- | --- | --- | --- |
| 1 | **Students** | `page_students` | 6.6 |
| 2 | **Teams** | `page_teams_admin` | 6.6 |
| 3 | **Staff** | `page_staff` | 6.6 |
| 4 | **Progress** | `page_progress` + three charts | 6.7 |
| 5 | **Quiz results** | `page_quiz_results` | 6.7 |
| 6 | **Journey** | `journey(student_id)` | 6.7 |

Six screens, but the first three are the same shape — a table, a filter, an edit
dialog, a delete. Build Students properly and the next two follow it.

---

## Delete — on all three people screens

The confirm box uses a **red danger button**, **names the thing**, and **says
what else goes with it**:

> Delete Ravi? Their daily posts go too. This cannot be undone.

Delete must never look like Edit. That was a real finding: they were the same
style, side by side, on a table of 209 students.

## Progress — three charts

`attendance_chart`, `points_chart` and `spread_html` are hand-drawn HTML and
CSS, using `.ac-chart__*` and `.ac-bars__*` from the design system — **not from
`app.css`**. Keep them hand-drawn. They are simple bars, the design system
already styles them, and a charting library here would be a second visual
language in the product.

## Journey

One student's nine days, reached from Progress. `/api/admin/journey/:id`.

It shows **daily post content, and that is admin-only.** Mentors and teammates
never see post content, anywhere, for any reason. If your screen can be reached
by a mentor, the posts are not on it. Say in your check list how you confirmed
that.

## Quiz results

Keep the search box and the marker on your own team.

---

When all six are done, report and stop. Do not start another lane's screens,
even if you finish first.
