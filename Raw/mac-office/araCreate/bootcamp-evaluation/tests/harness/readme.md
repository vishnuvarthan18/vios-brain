# HARNESS — tests that sign in for real

T3-A2. The safety net everything after it is built on.

> Every test signs in through the **real** login and carries the session.
> A test that hits an endpoint without a session does not count and must not
> be written.

```sh
# once
createdb bootcamp_harness
pg_dump --schema-only --no-owner --no-privileges -d bootcamp_local \
  | psql -d bootcamp_harness

node tests/harness/session-suite.js   # every read route, every role
node tests/harness/sabotage.js        # the proofs that those checks work
```

## Writing a suite

```js
const { main } = require('./tests/harness');

main('what this proves', async (h) => {
  h.allowed('admin can read the board', await h.admin.get('/api/admin/releases'));
  h.refused('a mentor cannot',          await h.mentor.get('/api/admin/releases'));
});
```

`main()` reseeds the scratch database, starts a server against it, signs in
every role, runs the body, and always stops the server again — including when
the body throws, which is when a stray server process is hardest to find.

### What `h` carries

| | |
| --- | --- |
| `h.student` `h.lead` `h.mentor` `h.admin` | the four roles, signed in |
| `h.other_venue` | a student in EEE, for venue-leak tests |
| `h.teamless` | a student with no team |
| `h.anonymous` | deliberately not signed in |
| `h.rows` | the fixture rows behind them, for ids |
| `h.ok` `h.allowed` `h.refused` `h.status` | checks |

Every client has `get`/`post`/`put`/`patch`/`del`, carries its cookie like a
browser, and reports the role in any failure message.

There is no way to make a request without signing in first: `sign_in()` is the
only thing that returns a client, and it throws rather than returning one that
is not authenticated. An anonymous client comes from `anon()`, named so it
cannot pass for a session in review.

## It is isolated on three axes at once

The harness starts **its own server**, rather than testing whatever is on
3000 — which points at a real database.

| | |
| --- | --- |
| database | `bootcamp_harness`, a scratch database. Never `bootcamp_local` |
| port | 3131, loopback only |
| password | a local-only fixture password, set for the server it starts |

Nothing here reads the real `.env`, so nothing here can leak the real staff
password. This worktree has no `.env` at all — it lives at the repo root and
worktrees do not share it. A real environment variable beats the file, and the
harness sets one.

Google credentials are blanked for the started server, so a run cannot reach
Drive.

## Running the suites that already existed

```sh
node tests/harness/run-suite.js releases        # 39 checks, green
node tests/harness/run-suite.js tasks           # stops on a real server bug
node tests/harness/run-suite.js releases tasks
```

`tests/releases.js` and `tests/tasks.js` are **not edited**. They already sign
in through the real login and already read their fixtures out of the database;
what they lacked was a database with the right shapes and a server pointed at
it. The runner supplies both, and reseeds between suites, because each one
writes rows.

The fixture carries an **empty quiz** (Day 9) for exactly this reason. Four of
`releases.js`'s checks look for a quiz with no questions and skip themselves —
`if (rows.length)` — when there is none. A fixture where every quiz has
questions turns those four into a silent pass.

`tasks.js` currently stops at a genuine server bug, written up as **Q4** in
`docs/questions-for-vishnu.md`: hand-in by text or Drive link 500s, because
`ON CONFLICT (task_id, team_id)` cannot infer a partial unique index. It is a
true red, not a harness problem.

## The route table

`routes.js` names each route and who may reach it. Everyone else in the four
roles is asserted to be **refused**, from the same line — which is what stops
the refused half being quietly forgotten. The worst v2 bug was a team lead
locked out of their own work while 41 checks passed.

`student` is a real tier, not a variety of `auth`: four routes carry `auth` as
middleware and then turn staff away *inside* the handler. That was found by
calling them as each role, not by grepping — `/api/me` contains the same
`kind !== 'student'` string in a nearby branch and is open to everyone.

The audit runs **both ways** on every run:

- every table entry must exist in `server.js`, with the guard the table claims;
- every `app.get('/api/…')` in `server.js` must have a row in the table.

The second direction matters more than it looks. The table is *iterated*, so a
route that is not in it is simply not tested and nothing says so. It was
complete for the 36 routes on `v3-dev` — and Lane A's survey work adds eight
more that would have arrived silently uncovered at the rebase. A new route now
fails the run until someone records who may reach it.

## The sabotage proofs

`docs/standing-authorisation.md`: a guard test is not done until you have
watched it fail, **at the level it claims to protect**.

| Proof | Level attacked |
| --- | --- |
| the seed guard | the refusal itself — `bootcamp_local` is accepted once the check is removed |
| the session suite | a real admin route loses `require_admin`, and a mentor gets in |
| the harness itself | `auth` stops requiring a cookie, and the harness must refuse to run at all |
| table coverage | a real route is added to the server with no row in the table |

Each asserts **both** halves: red when broken, green when restored. Each also
asserts it went red **for the right reason** — that second assertion is not
decoration. The first version of proof 2 was vacuous: the sabotaged server died
with `MODULE_NOT_FOUND` because this worktree has no `node_modules` of its own,
so the suite went red without ever testing a permission. A red run proves
nothing if the cause is not the hole that was cut.

Proof 4 then failed the other way round — it **passed** when it should have
gone red, which is a finding in its own right. The static audit was reading
`src/server.js` at its canonical path while the suite exercised a sabotaged
*copy*: it was auditing a file nobody was running. It now reads
`HARNESS_SERVER` when that is set. Proof 2 was unaffected, because it asserts
on a live HTTP response rather than on the audit.

### They never edit a file this lane does not own

`docs/lanes.md` gives `src/server.js` to Lane A, and editing a file this lane
does not own is a stop-work event *whatever the reason* — writing it and
putting it back is still editing it, and with two agents working in parallel it
could land on top of Lane A's own change.

So a proof that needs a broken server builds a throwaway tree: every entry
symlinked back to the real one, except the single file under attack, which is
written there as a modified copy. `HARNESS_SERVER` points the harness at it.
The real file is never opened for writing.
