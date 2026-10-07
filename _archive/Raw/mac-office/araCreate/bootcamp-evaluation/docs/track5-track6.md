# Track 5 and Track 6 — what was built, 20 Sep

Built on the Mac, **not committed, not built, not deployed.**
Nothing in here changes a single row: every route added is a GET.

## The four things asked for

| | |
| --- | --- |
| Team profile page | `web/src/pages/Team.jsx` · `#team/<id>` |
| Completion matrix | `web/src/pages/Matrix.jsx` · `#matrix`, in Reports |
| Every name and number a link | `web/src/components/ui/links.jsx` |
| New navigation | `web/src/lib/nav.js` · 5 admin groups, 5 student items |

**The student profile page (T5-3) was NOT built** — dropped on your say-so.
A small student card opens over the list instead, which is what a name
needs to be clickable.

## Server — one new file, four read-only routes

`src/routes/people.js`, mounted in `server.js` next to scoring v3.
All four are **staff**, not admin: a mentor chasing a missing hand-in
needs the phone number, and sending them to an admin for it is how the
chase does not happen.

```
GET /api/v3/matrix?dept=            students x days, one block each
GET /api/v3/teams/:id/profile       one team, members, hand-ins, adjustments
GET /api/v3/students/:id/card       one student, on a card
GET /api/v3/who?of=&id=&day=&state= THE LIST BEHIND A NUMBER
```

`/who` answers five questions in one shape — name, roll, team, venue,
phone, time — so one screen renders all five and *Copy all phone
numbers* works everywhere:

```
of=attendance&day=3&state=absent | present | unmarked
of=task&id=8&state=missing | done
of=quiz&id=2&state=missing | done
of=survey&day=3&state=missing | done
of=project&day=3&state=missing | done
```

All four are in `tests/harness/routes.js`, so each is asserted against
every role that may reach it and every role that may not.

## The matrix

Rows students, columns days. Five states, and **colour is never the only
carrier** — each block holds a character too:

```
#  everything done      o  some of it      ·  nothing done
–  absent               ?  nobody took the register
```

Only work that was **actually opened for a venue** counts. A task written
but never opened is not something a student failed to do, and colouring
it red would be a lie the whole screen is built on.

Hovering a block names the missing items — "still to do: Breadboard
basics" — because "2 of 3" sends you looking and a name does not.

Filters: venue, *Behind only*, *Missed a day*.

## The navigation

Sixteen flat admin items into five groups:

| Group | Items |
| --- | --- |
| Today | Home · Open · Register |
| Content | Tasks · Projects · Quizzes · Surveys · Assessment · Tinkercad |
| People | Students · Teams · Staff |
| Live | Quiz now · Leaderboard · Points |
| Reports | **Completion** · Profiles · Quiz results |

Student: **Today · My work · Board · Posts · You**. Five.

Quiz, Survey, Attendance and the assessment are no longer tabs — each is
open for part of one day, and a tab that reads "nothing here" on the
other eight is a tab worth not having. **Today is now the only way into
all four.** The assessment had no link from anywhere before this, so an
"Where you are" row was added to Today in the same change — without it,
cutting the tab would have made the assessment unreachable.

**The rule from here on: a new feature does not get a new nav item.** It
goes in one of the five groups, or it is not built.

## The hash now carries an id

`#team/12`. The part before the slash is the page, the part after is its
argument, and a refresh stays on the same team. `navFor()` is what is
DRAWN; `allowedPages()` is what may be ROUTED to, and it is bigger — it
holds the screens reached from inside another screen. Confusing those two
is what threw every refresh back to the first tab once already.

## Verified — overnight, 19/20 Sep

The Mac's bridge cannot reach Postgres and its `node_modules` are the wrong
architecture, so the whole app was rebuilt in a Linux container instead:
PostgreSQL 16, `schema.sql`, every migration in dependency order, and the
209-student / 53-team fixture. Then it was driven for real.

| | |
| --- | --- |
| Front end | builds clean |
| `harness/session-suite` | **405 passed, 0 failed** — every read route, every role, including 20 new assertions for the four Track 5 routes |
| `scoring` | **39 / 0** |
| `scoring-routes` | **50 / 0** |
| `releases` · `survey` · `survey-sessions` · `task-upsert` · `tinkercad` · `completion` · `gates` | **0 failures** |
| `tests/track5-browser.mjs` (new) | **77 passed, 0 failed** — four roles, every screen, both phone and laptop |

The browser suite is the one that matters. It signs in as student, team lead,
mentor and admin, visits every destination, and fails on any console error or
any failed request. It asserts, among other things:

- every one of the admin's destinations draws a real screen
- a refresh on `#team/12` stays on team 12
- Export CSV really downloads a file, and that file has the chase headings
- the matrix names the missing items, and its blocks carry a character, not
  only a colour
- **the points on a student's home are the points on the board** — they were
  not, see `known-issues.md`
- the quiz, survey, assessment and register are all still reachable after
  their tabs were cut
- no page scrolls sideways at 390px
- `/old/` can still sign a student in — the rollback, proved rather than hoped

Four suites still fail, in exactly the way they failed before any of this
work. Proved rather than assumed: the whole set was run twice on the same
database, once with the new route module mounted and once with it removed,
and the two runs are identical. They are written up in `known-issues.md`.

## Three defects found by looking at the screen

`known-issues.md` has them in full. In short: every student's home said "0
points"; the matrix made the page scroll sideways on a phone; a mentor's first
screen was a dead end they were not allowed to read. None threw an error.

## Going live

`docs/go-live.md`. Four commands, and `scripts/go-live.sh` does the server
side — backup, the missing migrations in dependency order, and a check that
the site actually answers. Rollback is `scripts/ui.sh old` for the screens, or
`scripts/rollback-live.sh` for the database. All three were rehearsed.
