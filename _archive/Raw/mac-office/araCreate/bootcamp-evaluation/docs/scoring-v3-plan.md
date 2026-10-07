# Scoring v3 — automatic marks, live leaderboard, admin adjustments

Decided with Vishnu, 19 Sep 2026. **This replaces manual marking entirely.**
Read `scoring-is-dead.md` first for how scoring worked before this.

Built and tested 19 Sep; see `agent-log.md`. **Nothing is switched over yet.**

---

## 1. The decision, in one line

**Nobody marks by hand.** Points are calculated from four things the database
already records. The only human touch is an admin adding or removing points as
a separate, logged adjustment.

---

## 2. What earns points

| Source | When it scores |
| --- | --- |
| **Quiz answers** | Each correct answer, the moment it is saved |
| **Task / project handed in** | The moment a hand-in lands, before any review |
| **Attendance** | When the team lead marks present |
| **Survey answered** | When the day's survey is *finished* |

Nothing here needs a person to judge quality. That is the point.

Starting values, in `scoring_settings` — one row, no code:

| Thing | Points |
| --- | --- |
| Quiz, per correct answer | 1 |
| Handed in inside the timer | 5 |
| Handed in after the timer | 0 |
| Attendance, per day present | 2 |
| Survey completed, per day | 1 |

---

## 3. Timing — two rules

**Rule 1: nothing is ever refused.** Every hand-in is accepted, stored and
visible. Late work scores zero. There is no such thing as a rejected hand-in
and no "closes and refuses" anywhere.

**Rule 2: closing is always manual.** Nothing closes itself, for any type. An
activity stays open until an admin closes it on the release board. The timer
decides *points only*.

`tests/scoring-routes.js` asserts rule 2 as an **absence**: the module must
contain no `is_open = FALSE`, no `closed_at`, and no `setTimeout`/`setInterval`
/`cron`. An absence is exactly what nobody notices disappearing.

### The timer

One number per release: how many minutes it is worth full points for. Preset
buttons — **5 · 10 · 15 · 30 · 60 · custom**. It starts when the thing is
**opened**, not created, which is why it lives on `releases` and not on the
item: the same task opens for EEE and ECE at different times.

Task, project, assessment and survey carry a **default** set at creation. A
**quiz has no default** — its minutes are chosen at the moment it is opened,
which is how Vishnu asked for it. The per-question 30-second clock is separate
and unchanged; it lives inside the quiz, not around it.

**Blank minutes = no deadline = full points however late.** That is the
default and it is deliberately the safe one: an activity created in a hurry
must not quietly score everyone zero.

### Attendance — the one fixed window

**09:00–10:00 IST, automatic, every day.** The only place in the product where
a clock stops an action, because attendance recorded at four in the afternoon
does not mean anything. The lead is locked out after 10:00; an admin is not.

Stored against a named timezone, not by subtracting 5.5 hours in JavaScript.
The server runs UTC and 09:00 IST is 03:30 UTC — getting it wrong opens
attendance in the middle of the night.

---

## 4. Admin adjustments — a list, never an edit

`score_adjustments`: team, points (+ or −), **optional** note, who, when,
voided_at. Never updated, never deleted; undo voids.

One screen, admin only. **It is the only place in the product that can change
a number.** A mentor cannot — a mentor who can hand out points is a second
marking system wearing a different hat, and removing the first one was the
whole point.

**Team total = earned + adjustments**, always shown as two numbers.

---

## 5. Live leaderboard

Polls; every response carries a `version` fingerprint, so the client asks
"has anything moved since this?" and is told yes or no. That is what lets
teams slide up and down without reloading the page.

Rank, movement arrow, venue filter, projector mode. Every team name links to
the team profile. One calculation, shared with every other screen.

**It reads in ~25ms.** The obvious implementation — `SELECT team_auto_points(id)
FROM teams` — took **8.9 seconds**, because that is 53 function calls each
re-scanning the same tables. `v_team_day_points_v3` is the set-based version of
exactly the same rules, and `tests/scoring.js` asserts the two agree for every
team, because two ways of adding up points that disagree is the specific bug
this rewrite exists to kill.

---

## 6. What is built, and what is not

Built, tested, **not switched on**:

- `2026-09-19-f-scoring-v3.sql` and its `down`
- `scoring_settings`, `score_adjustments`, `scores_legacy`, the timer columns
- `team_points_breakdown()`, `team_auto_points()`, `team_total_points_v3()`
- `v_team_day_points_v3`, `v_team_points_v3`, `v_leaderboard_v3`, `…_by_dept`
- `attendance_window_open()`
- `src/routes/scoring-v3.js` — eight routes under `/api/v3/`
- `tests/scoring.js` (39) and `tests/scoring-routes.js` (50)

**Nothing existing changed.** `teams.total_points`, the old triggers,
`v_leaderboard` and `/api/mentor/score` are all untouched and still driving
every screen today. `GET /api/v3/scoring/compare` puts old and new side by side.

Screens, built 19 Sep and driven through a browser (`web/check-scoring.mjs`,
21 checks including a real write):

- `web/src/pages/Adjust.jsx` — **Points**. The only screen that can change a
  number. Earned and given always shown apart, optional reason, full history,
  undo, and an all-teams view with CSV.
- `web/src/pages/BoardLive.jsx` — **Live board**. Polls on the version stamp,
  movement arrows that fade, venue filter, projector mode, and it stops
  polling when the tab is not being looked at.
- `web/src/pages/Open.jsx` — the **points timer** on each venue's card.

Two things the screenshots caught, neither of which threw an error:

- The live board is **ranked by points per member** and was showing the
  **total**, so the column read 131, 94, 121 down the page and looked broken.
  The headline number is now whichever number the ranking used.
- A **closed** release printed "until 11:42 PM" — a deadline that is not
  running, next to a Closed badge. It now says "20 min from opening".

Not built yet:

- The attendance window enforced at the point a lead marks (T4-10)
- A default timer on the create forms, copied onto the release at open (T4-13)
- The timer shown on the student's own card (T4-14)
- Archiving `scores` into `scores_legacy` and deleting manual marking — the
  irreversible step, and it goes last

---

## 7. Still to confirm with Vishnu

1. **Ranking by total, or per member?** Default taken: **per member**, because
   the teams are not the same size. **This did not ship.** The column DEFAULT
   in `2026-09-19-f` was left at `'total'`, so every database built from it
   ranked by raw total and put all four three-person teams in the bottom
   seven. Corrected by `2026-09-23-a-rank-by-per-member`, which sets the row
   and the default. Applied locally 23 Sep; **not yet on production**.
2. **A daily cap?** Default taken: **no cap**. Implemented and tested, set to
   NULL.
3. **Can a team go below zero?** Default taken: **yes**.
4. **After 10:00, can an admin still mark someone present?** Default taken:
   **yes, admin only.**

All four are one row in `scoring_settings` or one line of code.
