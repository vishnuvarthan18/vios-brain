# Basic Electronics Workshop

The dashboard that runs araCreate's nine-day electronics bootcamp: 52 teams and
206 students across EEE and ECE, from 18 to 26 September 2026. Students hand in
a project a day and take a team quiz a day; mentors score; and every student
keeps a private log that turns the resume they arrived with into the one they
leave with.

**What it does**

- Email-only sign-in for students, a shared staff password for mentors and admin
- A project a day per team, scored 0–5 by that team's mentor
- A timed team quiz a day, auto-graded, one attempt, server-side clock
- Attendance marked by the team lead
- A leaderboard that cannot go stale — a database trigger recalculates on every score
- A private daily log per student, with a resume at each end of the nine days
- An admin panel for students, teams, staff, quizzes and progress

## Stack

| Layer | Tool |
| --- | --- |
| Runtime | Node 22 |
| Server | Express 4 |
| Database | PostgreSQL 16 |
| Front end | Plain JavaScript, no framework, no build step |
| Styling | The araCreate design system, vendored in `src/public/ds/` |
| Tests | Playwright, against a real server and database |
| Hosting | systemd behind Caddy for HTTPS |

## Layout

| Path | What it holds |
| --- | --- |
| `src/` | `server.js` (the whole API), `db.js`, `db/` (schema and loaders), `public/` (the whole front end) |
| `docs/` | The design spec, the handover, deployment, and the student rosters |
| `tests/` | Browser and API checks |
| `scripts/` | `motd`, server provisioning, update and UI rollback |
| `.archives/` | Front ends that were replaced, kept as rollback targets |

## Data model

Points belong to the team, never the individual. A team earns up to 5 a day for
its project and up to 5 for the quiz — 90 over the nine days. `teams.total_points`
is maintained by a trigger, so the leaderboard is never stale.

Team codes are one format everywhere: `DEPT-TNN-TEAMNAME`, for example
`EEE-T01-CIRCUITCREW` and `ECE-T35-CLOCKWORKS`. The number runs separately
inside each department.

A student's daily post is readable by that student and by an admin. Not by their
teammates, and not by their mentor. That is enforced on the server, not hidden in
the menu.

## Commands

```sh
make install    # install dependencies
make setup      # create .env, or top up the keys it is missing
make db         # build the database and load the rosters — empty database only
make dev        # run locally
make test       # run the browser and API checks
make backup     # dump the database
make deploy     # pull and restart on a server
```

`make db` refuses to run against a database that already has students. The roster
loaders delete their department's students first, which cascades to every daily
post, attendance mark and quiz answer those students have.

## Deployment

`scripts/setup-server.sh` provisions a fresh Ubuntu server end to end. See
[`docs/deploy.md`](docs/deploy.md).

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions).

Project-specific rules:

- The front end is built from the design system in `src/public/ds/`. Before
  writing any CSS, read [`docs/design.md`](docs/design.md) §6 — the system ships
  90+ components and the app is expected to use them rather than hand-roll them.
- `src/public/app.css` holds only what the design system has no component for.

## License

Proprietary. See [LICENSE](LICENSE). Copyright (C) 2026, araCreate Group.
