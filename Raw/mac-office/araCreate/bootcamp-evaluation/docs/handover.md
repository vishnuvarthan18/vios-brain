# Handover

Everything someone taking this over needs, in one file.

**Live:** https://vcet.aracreate.academy
**Runs:** Friday 18 September → Saturday 26 September 2026, nine days
**Scale:** 52 teams, 206 students — EEE 14/55, ECE 38/151

---

## 1 What it does

| Who | What they get |
| --- | --- |
| Student | Home, their own profile and log, team page, projects, daily quiz, leaderboard |
| Team lead | Also attendance and handing in the team's work |
| Mentor | Home, and scoring the projects of their own teams |
| Admin | Everything, plus students, teams, staff, quizzes and progress |

**Points belong to the team, never the individual.** Up to 5 a day for the
project and 5 for the quiz — 90 over the nine days. `teams.total_points` is
maintained by a database trigger, so the leaderboard cannot go stale between a
score being saved and a page being opened.

**The nine-day arc.** Day 1 the student hands in the resume they already have.
Every day they write one short private post. Day 9 they build a proper resume.
On the last day you open their journey page and show them the two side by side,
with nine days of their own notes as the evidence.

---

## 2 Sign-in

- **Students:** their email from the roster plus one shared bootcamp code. No
  passwords — 206 people signing in on one morning, and a forgotten password is
  a queue at the desk.
- **Mentors and admin:** their email plus one shared staff password from `.env`.
- A signed cookie, 15 days, marked `secure` and `httpOnly`. No session table.

Codes and passwords are in the go-live checklist and in
`/opt/bootcamp-dashboard/.env` on the server.

---

## 3 Team codes

One format everywhere: `DEPT-TNN-TEAMNAME`.

```
EEE-T01-CIRCUITCREW  …  EEE-T14-WATTMINDS
ECE-T01-VOLTSQUAD    …  ECE-T38-SILICONCREW
```

The number runs separately inside each department. Leave the code box empty in
Add team and the right one is built from the department and the team name.

---

## 4 The two departments

They run the same nine days in two different places. **Nothing students see is
split** — one leaderboard, one set of quizzes, one start date.

The admin panel has a **Both / EEE / ECE** switch on Students, Teams, Progress
and Home. It remembers the choice. Quizzes has no switch, because opening a quiz
opens it for everyone.

---

## 5 Privacy

A student's daily post is readable by that student and by an admin. That is all.

- Teammates cannot see it — the team page returns name, register number,
  department, year and role, nothing else.
- **Mentors cannot see it either.** `/api/posts` and `/api/profile` are
  students-only; the journey and progress endpoints are admin-only.
- There is no endpoint anywhere that hands one student another student's post.

Enforced on the server, not hidden in the menu.

---

## 6 Layout

| Path | What |
| --- | --- |
| `src/server.js` | The whole API, plus a small `.env` loader |
| `src/db.js` | The connection pool and the startup check |
| `src/db/` | Schema, migrations, roster loaders |
| `src/public/` | The whole front end, and the design system it is built on |
| `docs/` | This file, the design spec, deployment, the rosters |
| `tests/` | Four Playwright suites, run against a real server and database |
| `scripts/` | Provisioning, deploy, UI rollback, the motd |
| `.archives/` | Front ends that were replaced, kept as rollback targets |

No build step. `make dev` runs it; `make test` checks it.

The repo follows
[aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions):
structure, SPDX headers, param-case file names, snake_case identifiers,
conventional commits.

---

## 7 The server

`ssh hetzner` · Debian 13 · `89.167.82.144` · Node 20, PostgreSQL 17, Caddy.

```sh
ssh hetzner 'sudo journalctl -u bootcamp -f'      # logs
ssh hetzner 'sudo systemctl restart bootcamp'      # restart
ssh hetzner 'sudo -u postgres psql -d bootcamp'    # database
```

Postgres listens on loopback. The app listens on loopback. Caddy is the only
thing facing the internet. ufw allows 22, 80 and 443. `pg_dump` runs nightly at
01:00 into `/var/backups/bootcamp`, 14 days kept.

Deploying is in [`deploy.md`](deploy.md). The setup script is safe to re-run:
it keeps `.env`, leaves the data alone, clears a stuck service, and repairs a
box whose database password and `.env` have drifted apart.

---

## 8 Decisions, and why

- **Email-only sign-in for students.** Passwords would have broken the first
  morning. The shared code stops classmates opening each other's pages. Staff
  get a real password because they control scores.
- **No file uploads.** Google Drive links only. A small VPS would fill with 52
  teams × 9 days of video and Proteus files, and upload bugs on Day 1 are the
  worst kind.
- **Team quiz, one attempt, server-side clock.** Closing the browser does not
  give time back, and a sweeper grades any attempt whose time ran out — so a
  team that walks away still scores what it answered.
- **Quiz questions lock the moment a team starts.** A quiz must not change under
  a team halfway through it.
- **Daily posts are private.** Readable by the student and the admin. Not by
  teammates, not by mentors.
- **Nothing lands anyone on a form.** Every role's first screen answers what to
  do right now. A student who lands on a page of empty fields does nothing.
- **The front end uses the design system rather than reimplementing it.** It
  ships more than ninety components; the app was using about twelve, and the
  difference was why it read as a tool rather than a product.

---

## 9 Do not do these

- **Never run `load-eee.sql` or `load-ece.sql` again.** Each deletes its own
  department's students before inserting, and that cascades to every daily post,
  attendance mark and quiz answer those students have. They are setup scripts,
  not start commands. `make db` and the server setup both refuse to run them
  against a database that already has students; running them by hand has no
  such guard.
- **If you move `start_date` to test something, move it back.** Students would
  post to the wrong day.

---

## 10 If something looks wrong on the day

**The UI:**

```sh
ssh hetzner 'cd /opt/bootcamp-dashboard && ./scripts/rollback-ui.sh && sudo systemctl restart bootcamp'
```

Front end only. No database change, no restart of anything else.

**Anything else:** the logs first, then `systemctl restart bootcamp`. If the
unit refuses to start after a crash loop, `systemctl reset-failed bootcamp`
clears the rate limit.

---

## 11 What it stands up to

Measured on 18 Sep against a copy of production, not estimated. The numbers
are per wave: every student in the bootcamp doing the thing at the same
moment, which is what the end of a session actually looks like.

| Load | Result |
| --- | --- |
| 209 students sitting a quiz, full paper | 209 finished, 0 errors, p95 19ms |
| 209 uploading a 400 KB resume | 209 ok, 0 errors, p95 ~30ms |
| 209 uploading a 2 MB scan | 209 ok, 0 errors, p95 184ms |
| 5 waves of 209 uploads back to back | 1,045 uploads, 0 errors, RSS flat at 196 MB |

Nothing was lost in any of them: 209 rows, 209 files, no duplicate attempts,
no orphaned uploads, no team over 90 points.

**Why it holds.** Writes are bounded by an admission queue —
`MAX_IN_FLIGHT=12` run at once, `MAX_WAITING=120` queue behind them, and past
that the server answers 503 with `Retry-After`. The front end waits that out
silently, three times, with jitter (`send()` in `app.js`). Under a full wave
of 209 that sheds 70-120 requests, and every one of them comes back. No
student sees an error; the slowest sees about a second.

The database pool is 20 and peaks at 13-14, so it is never the limit. The
admission queue is reached first, by design.

**The dial, if a morning is worse than this.** `MAX_WAITING` in `.env`. Raise
it and more requests queue rather than being shed; lower it and the server
sheds sooner. `MAX_IN_FLIGHT` controls how many run at once — raising it
above ~16 starts competing with the pool.

**Two things that are NOT bounded**, and would matter if they ever grew:
uploads buffer the whole file in memory before touching disk (10 MB cap, so
12 concurrent is 120 MB worst case — measured peak RSS was 469 MB under 209
maximum-size uploads), and there is no per-student rate limit, so one script
could fill the queue. Neither is a problem at this size.

