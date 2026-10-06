# Ownership model — who owns what, who sees what, how it scores

Decided 19 Sep 2026. **This is the authority on ownership.** Where any other
document disagrees, this one wins.

It is a **change** to how the app works today: tasks move from team-owned to
student-owned, and projects become lead-only.

---

## 1. The model in one table

| Thing | Owned by | Who hands it in | Who can see it | Scored | Mark goes to |
| --- | --- | --- | --- | --- | --- |
| **Project** | **Team** | **Team lead only** | **Team lead only** | Mentor, 0–5 | Team |
| **Task** | **Student** | Every student, their own | Each student their own; admin all | **Auto** — handed in 5, not 0 | Team, averaged |
| **Quiz** | **Student** | Every student | Each their own | Auto | Team, averaged |
| **Survey** | **Student** | Every student | Each their own | None | — |
| **Post** | **Student** | Every student | Student + admin only | None | — |
| **Attendance** | **Student** | **Team lead marks** | **Team lead dashboard only** | None | — |
| **Assessment** | **Student** | Every student | Each their own | None | — |
| **CV / profile** | **Student** | Every student | Student + admin; mentor sees profile, not posts | None | — |

**One team thing. Everything else per student.**

**Every mark ends up as a team mark.** Nothing is ever scored to an individual,
and there is no individual leaderboard.

---

## 2. Scoring rules

**Project** — mentor opens the team's hand-in and gives 0–5. Straight to the
team. Unchanged.

**Task** — no marking by anyone. Handed in = 5, not handed in = 0.
The team's task mark is the **average across every member of the team**, not
only those who did it.

> 4 of 6 members hand in → (4×5 + 2×0) ÷ 6 = **3.33**

The mark therefore means *how many of us did it*, which is the whole point of
making tasks per student.

**Quiz** — unchanged. Per student, auto-scored, team mark is the average of
those who **attempted**; never-opened is excluded.

**This is a deliberate inconsistency.** Task averages over everyone, quiz
averages over attempters. The reason: a quiz measures how well you did, so
someone who never sat it should not drag the team down. A task measures whether
you did it, so someone who did not must count. Do not "fix" one to match the
other.

**Day cap stays at 5 points**, and the 90-point maximum over nine days is
unchanged.

---

## 3. What the team lead sees that members do not

- **Projects.** Creating, handing in, and seeing the team's project work is the
  lead's job alone. An ordinary member has no project screen.
- **Attendance.** Marking the team's six, on the lead's dashboard only.

Everything else a lead sees, every member sees.

---

## 4. What this breaks today

**Tasks are team-owned in the live database.** `task_submissions` is
`UNIQUE (task_id, team_id)`, and the migration says so explicitly: *"a task
belongs to the TEAM, so any member may hand in and the second hand-in replaces
the first."*

Three consequences, none optional:

1. **Migration.** 56 task submissions and 50 orphans are keyed to teams. For a
   team hand-in there is no record of which student made it. Existing rows must
   be carried over in a way that does not invent a student who cannot be known —
   attribute to the team with a null student and mark them legacy, never guess.
2. **Chase list query 1 changes.** It was built as *every member of a team with
   no hand-in*. It becomes *each student with no hand-in of their own* — a
   different and much longer list.
3. **Scoring changes.** Any team whose task points were earned by one member
   handing in for six will score differently under the new rule.

**Do not apply this to the live app mid-bootcamp.** It changes scores that
students can already see. It belongs in the v3 cutover, between batches.

---

## 5. What this fixes

The four bugs found so far — `is_lead`, `set_release`'s allow-list,
`pages_for()`'s nav guard, `chk_releases_item_id` — are all the same bug: a new
value added to a gated concept, and an old list that enumerates values which
nobody updated.

A clear ownership model is what makes the v3 `activities` table possible: one
shape, one gate, one allow-list. Adding a type then touches one place instead of
four nobody remembers.

Every activity type declares two things and nothing else:

- **owner** — `team` or `student`
- **scoring** — `mentor` (0–5), `auto`, or `none`

Everything in section 1 falls out of those two fields. That is the product being
"in proper place with a proper flow" — not more screens, fewer rules.
