**Vishnu** (2026-09-19T11:57): You are LANE B. Read docs/lanes.md FIRST — it is the file-ownership contract and
breaking it is a stop-work event. Then docs/v3-agent-brief.md,
docs/standing-authorisation.md, docs/unattended-operation.md,
docs/phase-a-decisions.md.

You are in your own worktree on branch v3-dev. Another agent (Lane A) is working
on branch survey in the main checkout at the same time.

YOU MUST NOT TOUCH: src/server.js, src/public/app.js, src/public/app.css, any
survey migration, tests/survey*.js, docs/survey-spec.md. Lane A owns them.

YOU OWN: tests/harness/**, tests/gates.js, scripts/seed/**,
src/db/migrations/readme.md, docs/migration-ledger.md.

YOUR ORDER: A1 fake-data generator, A2 session test harness, A1b make the
releases and tasks suites run against the generator, A4 migration ledger,
A5a reader audit (READ-ONLY, report only, do not fix). Then STOP and wait —
A3 and A5b need Lane A's branch merged first.

Do not wait for me. Report into docs/agent-log.md after every item, prefixed
LANE B. Questions to docs/questions-for-vishnu.md — never stop the run. Commit
and push after every item.

A test is not a control until you have watched it fail. Every guard test must be
proven by sabotage at the level it claims to protect.

**Claude** (2026-09-19T11:57): I'll start by reading the contract documents before touching anything.

**Claude** (2026-09-19T11:57): _[tool: Bash]_

**Claude** (2026-09-19T11:57): I'm in the main checkout on branch `survey` — that's Lane A's. My worktree is `.worktrees/lane-b`. Let me move there and read the contract.

**Claude** (2026-09-19T11:57): _[tool: Bash]_

**Claude** (2026-09-19T11:57): `docs/lanes.md` doesn't exist on my branch. Let me check where it lives.

**Claude** (2026-09-19T11:57): _[tool: Bash]_

**Claude** (2026-09-19T11:58): All five briefing docs live only on Lane A's `survey` branch — my `v3-dev` branch is behind and has none of them. I must not guess at a file-ownership contract. Let me read them from the `survey` branch without checking it out or modifying it.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): The contract is clear and my orders match it exactly. Let me read the remaining four briefs.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Now the operational rules docs.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): `docs/unattended-operation.md` doesn't exist on `survey`. Let me search all branches and read the last two docs.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): `docs/unattended-operation.md` exists in commit `fc8ac28`. Let me read it there, plus the work queue and agent log.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): I have all five briefs. Let me check the work queue and the current state of my branch before starting A1.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Now let me survey the actual codebase state before writing the generator.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): My branch has no `tests/harness/`, no `tests/gates.js`, no `scripts/seed/` — all greenfield, all mine. Let me understand the schema and the existing test conventions.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Now the schema — the tables I need to populate.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Note: `schema.sql`'s `v_student_progress` has no `has_photo`/`has_education` — the migrations must add them. My memory says to diff schema.sql against live. Let me check the migrations and the live local DB.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Readme lists 9, directory holds 16 — exactly the A4 discrepancy. Let me check the local database state, since A1/A2 need one.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): Several local databases exist. Lane A is likely using `survey_test`. I need my own isolated database to avoid colliding with Lane A. Let me check what's in the main candidates.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:58): `bootcamp_local` has the real dump (209/53). Those contain real student contact details — my generator must never copy them. Let me confirm the env config and pick an isolated database name for Lane B.

**Claude** (2026-09-19T11:58): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Reading `.env` is blocked, and that's the right call — the brief says never print a secret. I only need the *key names*, not values. A2 will read the password at runtime from the file rather than me ever seeing it.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): `STAFF_PASSWORD` exists — A2 can read it at runtime without me ever seeing it. Let me return to my worktree and inspect the auth/login code the harness must drive.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Let me read the login route and role model precisely — the harness depends on getting this exactly right.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Now the login route itself — the exact contract the harness must satisfy.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): I have the full auth contract. Four roles: admin/mentor via `mentors` + `STAFF_PASSWORD`, student/lead via `students` + `BOOTCAMP_CODE`. Now let me check the remaining schema bits the generator must fill (tasks, releases, assessments, quiz-per-student).

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): 23 tables, 13 views — far beyond `schema.sql`. Let me capture the exact live column definitions for the tables the generator must populate.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Important drift: `teams.*_points` are `numeric` live but `INT` in schema.sql, and `task_submissions` has a `per_student` column. Let me capture the remaining tables plus all constraints, then build the generator.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Now let me capture the CHECK constraints so the generator produces only valid rows.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T11:59): Notably `task_submissions` has **no** `UNIQUE (task_id, team_id)` live, though the brief asserts it. That's a finding for the log. Let me verify before recording it.

**Claude** (2026-09-19T11:59): _[tool: Bash]_

**Claude** (2026-09-19T12:00): The uniqueness is a *partial* index (`WHERE NOT per_student`), not a plain constraint — the brief's shorthand is correct in spirit. Good. Now I have enough to build A1. Let me set up my isolated database and scratch space.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): Matches the brief exactly: ECE 154/39, EEE 55/14. Now let me write the A1 generator. First the directory and a schema-capture of what a fresh DB needs.

**Claude** (2026-09-19T12:00): _[tool: Bash]_

**Claude** (2026-09-19T12:00): Now I'll write the A1 generator. Key constraints: deterministic seed, invented names only, local-only, and a shape that produces the edge cases tests need.

**Claude** (2026-09-19T12:00): _[tool: Write]_

**Claude** (2026-09-19T12:00): Now the deterministic RNG and the main generator.

**Claude** (2026-09-19T12:00): _[tool: Write]_

**Claude** (2026-09-19T12:00): Now the main generator. It must refuse to run against a non-local database — that's a guard I'll later prove by sabotage.

**Claude** (2026-09-19T12:00): _[tool: Write]_

**Claude** (2026-09-19T12:00): Now the main generator itself.

**Claude** (2026-09-19T12:02): _[tool: Write]_

**Claude** (2026-09-19T12:02): Now I need a scratch database with the real schema. Let me create it from the structure of `bootcamp_local` (schema only, no data — no real student details copied).

**Claude** (2026-09-19T12:02): _[tool: Bash]_

**Claude** (2026-09-19T12:03): Schema loaded (36 relations, no data). Now let me test the guard refuses dangerous targets *before* running the generator for real.

**Claude** (2026-09-19T12:04): _[tool: Bash]_

**Claude** (2026-09-19T12:04): The guard refuses the real-dump database, the app database, and any remote host. Now let me run it for real.

**Claude** (2026-09-19T12:04): _[tool: Bash]_

**Claude** (2026-09-19T12:04): 209 students, 53 teams, 6 teamless, and all three attendance states present (1378 present / 283 stored-absent / 220 no-row). Now let me prove determinism — the same seed twice, and a different seed differing.

**Claude** (2026-09-19T12:04): _[tool: Bash]_

**Claude** (2026-09-19T12:04): Determinism fails. The fingerprint includes `team_id`, which comes from a fresh SERIAL sequence each run — so identity differs even when the logical roster is identical. Let me check whether that's the whole cause.

**Claude** (2026-09-19T12:04): _[tool: Bash]_

**Claude** (2026-09-19T12:04): The logical roster is perfectly deterministic; only surrogate IDs drift because sequences aren't reset. For fixtures that's a real usability problem — a test pinning `team_id=7` would break on reseed. Let me fix it by resetting sequences.

**Claude** (2026-09-19T12:04): _[tool: Edit]_

**Claude** (2026-09-19T12:04): That identifier-quoting line is convoluted and wrong. Let me write it properly with a real identifier quote.

**Claude** (2026-09-19T12:04): _[tool: Edit]_

**Claude** (2026-09-19T12:05): _[tool: Bash]_

**Claude** (2026-09-19T12:05): Fully deterministic including primary keys. Now let me verify the fixture actually contains the edge-case shapes the tests need, and that no real student data leaked in.

**Claude** (2026-09-19T12:05): _[tool: Bash]_

**Claude** (2026-09-19T12:05): Zero overlap on every identifying field — no real student data in the fixture. Now let me verify the shapes tests depend on.

**Claude** (2026-09-19T12:05): _[tool: Bash]_

**Claude** (2026-09-19T12:05): One shape is missing: "quiz opened but submitted nothing" is 0, yet that's chase list #4 in the brief. My `r.chance(0.9)` sets `submitted_at`, but `grade_quiz_attempt` then overwrites it with `COALESCE(submitted_at, now())`. Let me confirm and fix.

**Claude** (2026-09-19T12:05): _[tool: Bash]_

**Claude** (2026-09-19T12:05): Confirmed: `grade_quiz_attempt` stamps `submitted_at` unconditionally, so calling it on an unsubmitted attempt destroys that state. The fix is to only grade attempts that were actually submitted.

**Claude** (2026-09-19T12:06): _[tool: Edit]_

**Claude** (2026-09-19T12:06): _[tool: Edit]_

**Claude** (2026-09-19T12:06): _[tool: Bash]_

**Claude** (2026-09-19T12:06): All three quiz states now exist: 128 opened-but-not-submitted, 940 submitted, 23 never opened. Let me re-verify determinism after the change and check the venue-leak asymmetry.

**Claude** (2026-09-19T12:06): _[tool: Bash]_

**Claude** (2026-09-19T12:06): Determinism holds. Let me fix the query syntax.

**Claude** (2026-09-19T12:06): _[tool: Bash]_

**Claude** (2026-09-19T12:06): Strong asymmetry: quizzes open for ECE only (3/0), pre-assessment for EEE only (0/1). A venue leak cannot hide in this fixture. Let me add a README and commit A1.

**Claude** (2026-09-19T12:07): _[tool: Write]_

**Claude** (2026-09-19T12:07): Now let me create the shared log files (they don't exist on my branch) and write my first entry.

**Claude** (2026-09-19T12:07): _[tool: Bash]_

**Claude** (2026-09-19T12:07): Lane A is actively writing `docs/agent-log.md` on `survey`. These are append-only shared files. To minimise the merge conflict I'd otherwise create, let me start my copy from Lane A's current version rather than a fresh file.

**Claude** (2026-09-19T12:07): _[tool: Bash]_

**Claude** (2026-09-19T12:07): Now let me write my A1 log entry in the required format, prepending it as newest-at-top.

**Claude** (2026-09-19T12:07): _[tool: Bash]_

**Claude** (2026-09-19T12:07): The `---` separator after line 5 got absorbed. Let me fix the header formatting.

**Claude** (2026-09-19T12:07): _[tool: Bash]_

**Claude** (2026-09-19T12:07): Formatting is correct. Let me commit and push A1.

**Claude** (2026-09-19T12:08): _[tool: Bash]_

**Claude** (2026-09-19T12:08): A1 done and pushed. Now **A2 — the session test harness**, the safety net for everything after it. Let me check how the server boots and whether I can run it on an isolated port.

**Claude** (2026-09-19T12:08): _[tool: Bash]_

**Claude** (2026-09-19T12:08): Real env vars win over `.env`, so I can point the server at my scratch DB and port. Critically, the `.env` at the repo root is **outside** my worktree — let me verify what the server sees.

**Claude** (2026-09-19T12:08): _[tool: Bash]_

**Claude** (2026-09-19T12:08): My worktree has no `.env`, so the harness can set a local-only test password via environment variable — never reading or printing the real secret. That satisfies A2's credentials rule cleanly. Let me build the harness.

**Claude** (2026-09-19T12:08): _[tool: Write]_

**Claude** (2026-09-19T12:08): Now the core: a session client that signs in through the real login and carries the cookie.

**Claude** (2026-09-19T12:08): _[tool: Write]_

**Claude** (2026-09-19T12:09): Now the server lifecycle — start an isolated server against the scratch DB, and the role resolution.

**Claude** (2026-09-19T12:09): _[tool: Write]_

**Claude** (2026-09-19T12:09): Now the roles module — resolving four real roles from the fixture, plus the assertion helpers.

**Claude** (2026-09-19T12:09): _[tool: Write]_

**Claude** (2026-09-19T12:09): Now the assertion/reporting layer and the top-level harness entry point.

**Claude** (2026-09-19T12:09): _[tool: Write]_

**Claude** (2026-09-19T12:09): Now the orchestrator that ties it together — reseed, start server, sign in, run, tear down.

**Claude** (2026-09-19T12:10): _[tool: Write]_

**Claude** (2026-09-19T12:10): Now let me set up the harness database and do a first smoke test.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): The harness works end to end: reseeds, starts an isolated server, signs in all four roles through the real login, and refuses anonymous. Now let me enumerate every route so the suite can cover them all.

**Claude** (2026-09-19T12:10): _[tool: Bash]_

**Claude** (2026-09-19T12:10): 84 routes with clear guard tiers. Let me build this into a machine-readable route map so the suite can assert *every* route for *every* role, including the ones that must be refused.

**Claude** (2026-09-19T12:11): _[tool: Write]_

**Claude** (2026-09-19T12:11): Now the main session suite that drives this table for every role.

**Claude** (2026-09-19T12:11): _[tool: Write]_

**Claude** (2026-09-19T12:11): _[tool: Bash]_

**Claude** (2026-09-19T12:11): 217 passing, 12 failing — and the failures are **my table being wrong**, not the app. The brief warns: first ask whether the test is wrong. Two distinct causes. Let me check the parameterised-route regex first.

**Claude** (2026-09-19T12:11): _[tool: Bash]_

**Claude** (2026-09-19T12:11): The routes exist — my audit normalises the *concrete* URL, but `route.path` already holds the `:name` placeholder, so nothing matched. Let me fix the audit to normalise placeholders to `:id`.

**Claude** (2026-09-19T12:11): _[tool: Edit]_

**Claude** (2026-09-19T12:12): Now the second cause: `/api/my-team`, `/api/my-projects`, `/api/profile`, `/api/posts` are student-only in practice. Let me verify that's a real inner guard, not a bug.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:12): There's a real student-only check, applied *inside* route bodies rather than as middleware — so my `auth` tier was too coarse. That's a genuine fifth tier. Let me add it.

**Claude** (2026-09-19T12:12): _[tool: Edit]_

**Claude** (2026-09-19T12:12): Now let me check each `auth`-tier route for an inner student-only check and classify them correctly.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:12): That 25-line window is too wide — it's bleeding into the *next* route. `/api/me` passed for admin in the actual run, so this static guess is unreliable. Let me classify empirically instead, from real responses.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:12): Empirical truth: exactly four routes are student-only. Let me update the table.

**Claude** (2026-09-19T12:12): _[tool: Edit]_

**Claude** (2026-09-19T12:12): The `student` tier is enforced inside handlers, so the middleware-based audit can't see it. Let me make the audit aware of that rather than silently mismatching.

**Claude** (2026-09-19T12:12): _[tool: Edit]_

**Claude** (2026-09-19T12:12): The `refused` helper accepts 400, though — and staff get 400 on `/api/my-team`. Let me check that gap before rerunning.

**Claude** (2026-09-19T12:12): _[tool: Bash]_

**Claude** (2026-09-19T12:13): `refused` doesn't accept 400, but `/api/my-team` returns 400 for staff. Rather than widen `refused` (400 usually means malformed input, and accepting it everywhere would weaken every refusal check), I'll note the real behaviour precisely.

**Claude** (2026-09-19T12:13): _[tool: Edit]_

**Claude** (2026-09-19T12:13): _[tool: Bash]_

**Claude** (2026-09-19T12:13): Now let me mark which routes refuse with 400 and use the right helper in the suite.

**Claude** (2026-09-19T12:13): _[tool: Bash]_

**Claude** (2026-09-19T12:13): 229 passed, 0 failed. Now the critical step — **the sabotage proof**. Per the standing rule, these guards are not controls until I've watched them fail at the level they claim to protect.

**Claude** (2026-09-19T12:14): _[tool: Write]_

**Claude** (2026-09-19T12:14): Proofs 2 and 3 edit `src/server.js` — which Lane A owns. I must not touch it, even transiently. Let me verify the exact ownership wording before proceeding.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): The contract is unambiguous: `src/server.js` is Lane A's, and "an agent that edits a file it does not own is a stop-work event, **whatever the reason**." A temporary write-then-restore is still an edit, and in a shared worktree tree it could collide with Lane A's live edits.

Let me restructure the proofs to sabotage a **copy** of the server instead.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:14): `src/` is clean — I haven't touched Lane A's files. The server uses `__dirname` relative paths, so a sabotaged copy must live at a parallel `src/` depth. Let me rework the sabotage to use a disposable copy of the whole tree.

**Claude** (2026-09-19T12:14): _[tool: Edit]_

**Claude** (2026-09-19T12:14): Now the guard proof must also not edit the real file. Let me rewrite all three proofs to use the copy-based approach.

**Claude** (2026-09-19T12:14): _[tool: Bash]_

**Claude** (2026-09-19T12:15): Now I need `HARNESS_SERVER` support in the harness server module.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:15): Now let me run the sabotage proofs — the moment of truth.

**Claude** (2026-09-19T12:15): _[tool: Bash]_

**Claude** (2026-09-19T12:16): The suites go red under sabotage, but two proofs can't confirm they failed *for the right reason*. This is exactly the near-miss pattern — "went red" isn't proof if the cause is wrong. Let me see the actual output.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:16): Empty output — the suite is crashing before producing results. Let me see stderr.

**Claude** (2026-09-19T12:16): _[tool: Bash]_

**Claude** (2026-09-19T12:17): `MODULE_NOT_FOUND` — the copied server can't resolve `express`, because Node resolves `node_modules` from the *real* path of a symlinked directory's parent. So the suite went red for an unrelated reason: a vacuous proof.

Let me fix it by copying the real file into the real `src/` tree under a temp name.

**Claude** (2026-09-19T12:17): _[tool: Bash]_

**Claude** (2026-09-19T12:17): Placing an untracked file in `src/` risks Lane A seeing it. A cleaner fix: keep the temp tree but symlink `node_modules` directly into it, so resolution works.

**Claude** (2026-09-19T12:17): _[tool: Bash]_

**Claude** (2026-09-19T12:17): That symlinks `node_modules` at the temp root, but Node resolves from the file's *real* path. Let me verify the fix actually works before trusting it.

**Claude** (2026-09-19T12:17): _[tool: Bash]_

**Claude** (2026-09-19T12:18): Still failing the same way. My assumption about the fix was wrong — Node resolves from the real path, so the root symlink doesn't help. Let me confirm the actual error.

**Claude** (2026-09-19T12:18): _[tool: Bash]_

**Claude** (2026-09-19T12:18): Confirmed: the temp-tree approach can't resolve modules. `NODE_PATH` is the reliable fix — let me verify it works before wiring it in.

**Claude** (2026-09-19T12:18): _[tool: Bash]_

**Claude** (2026-09-19T12:18): `NODE_PATH` didn't help either (the temp dir is under a symlinked `/private/var` path). The simplest robust fix: make the sabotage tree a real `node_modules` symlink *inside* the temp dir at the right level.

**Claude** (2026-09-19T12:18): _[tool: Bash]_

**Claude** (2026-09-19T12:19): There's no `node_modules` in my worktree — it's in the main checkout. That's the actual root cause.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): My worktree has no `node_modules`; `express` resolves from the parent checkout by Node walking up. My temp tree is under `/tmp`, so that upward walk finds nothing. Now the fix is clear — symlink the real `node_modules` into the temp tree.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): Proof 2 now fails for the right reason — naming the exact route and role. Proof 3 still goes red for an unverified cause. Let me find out why.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): The pattern matches (otherwise it would throw). So the sabotage applies, but the run dies before reaching the HARNESS INVALID check. Let me see the actual output.

**Claude** (2026-09-19T12:19): _[tool: Bash]_

**Claude** (2026-09-19T12:19): That's my debug script failing (it lives in the scratchpad), not the proof. Let me instrument the real sabotage file to surface proof 3's output.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): The output is empty — meaning the suite wrote nothing to stdout and crashed. `execFileSync` puts a spawn-level error elsewhere. Let me capture stderr too.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:20): The guard **did** fire correctly — "HARNESS INVALID: /api/me answered 200 without a session" — it just goes to stderr, which my check wasn't reading. The test was wrong, not the code.

**Claude** (2026-09-19T12:20): _[tool: Bash]_

**Claude** (2026-09-19T12:21): All 10 sabotage proofs pass, each verifying **both** that the guard goes red and that it fails for the right reason. Let me confirm Lane A's files are untouched and no temp dirs leaked.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:21): Lane A's files untouched, no leaked temp dirs, my guard restored. Let me remove the debug block and add a harness README, then run everything once more.

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:21): _[tool: Write]_

**Claude** (2026-09-19T12:21): _[tool: Bash]_

**Claude** (2026-09-19T12:22): Let me log A2 and commit.

**Claude** (2026-09-19T12:22): _[tool: Bash]_

**Claude** (2026-09-19T12:23): _[tool: Bash]_

**Claude** (2026-09-19T12:23): A2 done. Now **A1b — make the `releases` and `tasks` suites run against the generator.**

**Claude** (2026-09-19T12:23): _[tool: Bash]_

**Claude** (2026-09-19T12:23): This suite already signs in properly and queries the DB for fixtures. Its blockers are a hardcoded admin email and `bootcamp_test` default. Let me see the rest and the tasks suite.

**Claude** (2026-09-19T12:23): _[tool: Bash]_

**Claude** (2026-09-19T12:23): Both suites need: an admin that exists in the fixture, the scratch DB, and a running server. Let me check whether my generator's admin email would work, and try running them as-is against the fixture.

**Claude** (2026-09-19T12:23): _[tool: Bash]_

**Claude** (2026-09-19T12:23): The suites default to a real admin email that isn't in the fixture, but both honour `ADMIN_EMAIL`. Let me start a harness-configured server and run them.

**Claude** (2026-09-19T12:23): _[tool: Bash]_

**Claude** (2026-09-19T12:24): Real progress: `releases.js` largely passes (fails only at a Playwright browser step), and `tasks.js` fails at line 124. Let me examine what `tasks.js` expects.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): `tasks.js` clears and builds its own data, so it barely needs my generator — the failure is that the student's hand-in returned non-200. Let me see the actual cause.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): The suite gets far: the student sees both tasks but the hand-in fails. Let me find out why the submit is rejected.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:24): The scratchpad can't resolve `pg`. Let me run the probe from the worktree instead.

**Claude** (2026-09-19T12:24): _[tool: Bash]_

**Claude** (2026-09-19T12:25): A 500 on submit — a server error, not a permission issue. Let me capture the server's stack trace.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): A real schema finding: `ON CONFLICT` at server.js:1371 has no matching constraint. This ties directly to the partial-index discovery from A1. Let me confirm.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): Confirmed. `ON CONFLICT (task_id, team_id)` can't use the *partial* index `WHERE NOT per_student` — Postgres requires the predicate be inferrable. Let me verify this hits real data too, not just my fixture.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): This is a **live-schema bug**, not a fixture artefact: `/api/tasks/:id/submit` returns 500 against the real database, and adding `WHERE NOT per_student` fixes it. This is why the fixture matters — a real-shaped database found it.

Let me check whether it's currently reachable in production.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): All 56 real submissions are `per_student` (file/image uploads via a different path). The **two text/drive hand-in routes are both broken** and unused so far — which is exactly why nobody has noticed.

Let me confirm the second site is equally affected.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): Both sites have the same defect, and the comment there even cites a plain `UNIQUE (task_id, team_id)` that the migration replaced with partial indexes. But `src/server.js` and `src/routes/` are **not mine to fix** — and rule 19 says don't widen scope. This is a finding, escalated as a question.

Let me finish A1b properly: make the suites runnable against the generator.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): `releases.js` needs a quiz with **zero** questions — my generator gives every quiz 5-8. Let me verify that's what it wants, then add the shape.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:25): These are guarded by `if (rows.length)` — so with no empty quiz they **silently skip**, which is a false green. The generator should supply one. Let me add it.

**Claude** (2026-09-19T12:25): _[tool: Bash]_

**Claude** (2026-09-19T12:26): Now let me create the A1b runner that wires both suites to the generator.

**Claude** (2026-09-19T12:26): _[tool: Write]_

**Claude** (2026-09-19T12:26): _[tool: Bash]_

**Claude** (2026-09-19T12:27): `releases.js`: **39 passing, 0 failing** against the generated fixture — including the empty-quiz checks that would otherwise have silently skipped. Now `tasks.js`.

**Claude** (2026-09-19T12:27): _[tool: Bash]_

**Claude** (2026-09-19T12:27): `tasks.js` runs correctly against the fixture and stops at the genuine `ON CONFLICT` bug — a **true red**, not a harness problem. The suite is doing its job; the server code is broken and isn't mine to fix.

Let me confirm the bug is the sole cause by testing the one-line fix on a sabotage copy.

**Claude** (2026-09-19T12:28): _[tool: Bash]_

**Claude** (2026-09-19T12:28): A stray `require` line in my throwaway script. Let me remove it and rerun.

**Claude** (2026-09-19T12:28): _[tool: Bash]_

**Claude** (2026-09-19T12:28): Still failing — so my one-line fix isn't sufficient. My diagnosis was incomplete. Let me get the actual error with the fix applied.

**Claude** (2026-09-19T12:28): _[tool: Bash]_

**Claude** (2026-09-19T12:28): The task creation failed this time — likely a title collision with the fixture's tasks. Let me use a unique title.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:29): **SUBMIT -> 200.** The one-line fix does work — my earlier verification was invalid because a leftover server held port 3131, so `tasks.js` hit the *unfixed* server. Let me clean up and confirm properly.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:29): That stale server is a real harness flaw — it silently invalidated a result. Let me fix the harness to fail loudly instead of attaching to a foreign server.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:29): Let me verify the new guard actually fires, and re-confirm the `tasks.js` fix cleanly.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:29): The output got swallowed. Let me check more carefully.

**Claude** (2026-09-19T12:29): _[tool: Bash]_

**Claude** (2026-09-19T12:30): Exit 1 (correctly refused to run), but I need to confirm it's the *right* message, not an incidental failure — the same "right reason" discipline.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): It fails at sign-in, not with my clear message — the child server's `EADDRINUSE` arrives asynchronously after `wait_for` already succeeded against the squatter. Let me fix the ordering.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): The probe races our own child. It must run *before* spawning. Let me move it.

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): _[tool: Bash]_

**Claude** (2026-09-19T12:30): The guard now fires with the correct message. Let me re-verify the `tasks.js` fix on a clean port.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): Decisive: the one-line fix takes `tasks.js` from **9 passing / 5 failing** to **43 passing**. The remaining 5 are separate pre-existing issues, not the ON CONFLICT bug. Let me confirm those are independent.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): Confirmed — that check looks for a `pg_constraint` unique, which the migration replaced with partial *indexes*. It's the same root discrepancy, and the test itself is now stale. This is exactly the A1 finding, independently reconfirmed.

Let me write the question for Vishnu and log A1b.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:31): Now let me update the harness readme and log A1b.

**Claude** (2026-09-19T12:31): _[tool: Bash]_

**Claude** (2026-09-19T12:32): Let me run everything once more, then log and commit A1b.

**Claude** (2026-09-19T12:32): _[tool: Bash]_

**Claude** (2026-09-19T12:33): All green. Let me log and commit A1b.

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): A1b done. Now **A4 — the migration ledger**: reconcile the readme's 9 against the directory's 16. Documentation only, nothing run.

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): 15 files on my branch (the brief says 16 — Lane A's survey migration is likely the 16th). 6 undocumented, matching the brief's "six from 18–19 Sep". Let me verify each against the live database.

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): No migration-tracking table, so applied-ness must be inferred from database objects. The 16th is Lane A's in-flight survey migration. Let me determine each migration's markers and verify.

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): Two migrations are view-only (`v_team_projects`) — the case the brief says Track 1 verified by reading the view's own definition. Let me verify all 15 against the live DB.

**Claude** (2026-09-19T12:34): _[tool: Bash]_

**Claude** (2026-09-19T12:34): All markers present — 13 of 15 confirmed by column/table/constraint. Now the two view-only ones, which need reading the view's own definition.

**Claude** (2026-09-19T12:35): _[tool: Bash]_

**Claude** (2026-09-19T12:35): The live view contains `group_id`, which only `19-b` introduces — so `19-b` is applied, and it superseded `18-c` (both rebuild the same view). Let me confirm the ordering dependency and run order.

**Claude** (2026-09-19T12:35): _[tool: Bash]_

**Claude** (2026-09-19T12:35): Only 3 of 15 have documented `down` sections — notable given rule 12. Let me verify dependency ordering, then write the ledger.

**Claude** (2026-09-19T12:35): _[tool: Bash]_

**Claude** (2026-09-19T12:35): I have everything I need. Let me write the ledger — documentation only, nothing run against any database.

**Claude** (2026-09-19T12:36): _[tool: Write]_

**Claude** (2026-09-19T12:36): Let me verify the ledger's factual claims — particularly item 4's marker and the transaction claim, which I asserted.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:36): Both claims verified. Note item 4's marker is weaker than I implied — the row could come from normal app use. Let me make the ledger honest about that.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:36): The ledger is honest about its evidence. Let me verify the one remaining unchecked claim — that `18-c` would actually regress the view.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:36): Confirmed: `18-c` has zero mentions of `group_id`, so running it would drop that column from the view. Let me check who reads it.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:36): Two live routes do `SELECT *` from it — so the regression risk is real and my ledger statement stands. Let me record the readers precisely, then log A4.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:36): Now let me update the migrations readme — which I own — to point at the ledger, and log A4.

**Claude** (2026-09-19T12:36): _[tool: Bash]_

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): A4 done. Now the final item before I stop: **A5a — the `v_student_progress` reader audit. READ-ONLY, report only, no fixes.**

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): The view has 33 columns including the two stale ones (`has_photo`, `has_education`). Let me find every reader across the whole repo.

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): Exactly **one** application reader: `src/server.js:3430`, a `SELECT *`. Let me examine what that route does with the columns.

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): The route does `SELECT *` and passes rows straight to the front end. So the real consumers are in `app.js` — Lane A's file, but I can read it.

**Claude** (2026-09-19T12:37): _[tool: Bash]_

**Claude** (2026-09-19T12:37): The front end reads `/api/admin/progress` but **never** uses `has_photo` or `has_education`. Let me confirm which completeness fields it actually uses, and check the authoritative JS weights.

**Claude** (2026-09-19T12:38): _[tool: Bash]_

**Claude** (2026-09-19T12:38): Confirms the brief: six weights, no photo or education. Let me verify this module is the only completeness authority and check which view columns the front end actually uses.

**Claude** (2026-09-19T12:38): _[tool: Bash]_

**Claude** (2026-09-19T12:38): This is the key finding: the progress screen uses 8 columns, **none** of them `has_photo`/`has_education`, and `profile-completion.js` never touches the view. Let me check the other consumers to be thorough.

**Claude** (2026-09-19T12:38): _[tool: Bash]_

**Claude** (2026-09-19T12:38): That's a test fixture creating the underlying *table*, not reading the view. The audit is complete and the answer is clean. Let me write the A5a report.

**Claude** (2026-09-19T12:38): _[tool: Bash]_

**Claude** (2026-09-19T12:38): Conclusive: nothing anywhere reads `has_photo` or `has_education`. Let me write the audit report.

**Claude** (2026-09-19T12:38): _[tool: Bash]_

**Claude** (2026-09-19T12:38): Let me add the A5b observation as a question for Vishnu, then commit.

**Claude** (2026-09-19T12:39): _[tool: Bash]_

**Claude** (2026-09-19T12:39): All five items are done. Let me do a final verification that everything is green and clean before stopping.

**Claude** (2026-09-19T12:39): _[tool: Bash]_

**Claude** (2026-09-19T12:40): All five items are complete, committed, and pushed. Stopping here as ordered — A3 and A5b need Lane A's `survey` branch merged first.

## LANE B — Phase A complete

**A1 · fake-data generator** — 209 students, 53 teams, two venues, nine days, from a seed. Deterministic including primary keys (the first version wasn't — sequences weren't reset). Zero of the generated names, rolls, phones or emails collide with the real dump, checked by hash. Shapes are deliberately uneven, and releases are asymmetric so a venue leak can't hide.

**A2 · session harness** — **229 checks, 0 failed** at full volume. Every route, every role that may reach it, every role that must not, all through a real signed-in login. **10 sabotage proofs, 0 failed.**

**A1b · existing suites on the generator** — `releases.js` 39 green. Neither suite edited; they lacked a database with the right shapes and a server pointed at it.

**A4 · migration ledger** — run order for all 15, with what each marker actually proves. Nothing run, no migration changed.

**A5a · reader audit** — read-only, no fixes.

### Three things that need you

**Q4 is the one that matters.** `POST /api/tasks/:id/submit` **returns 500 for every text or Drive-link hand-in**. `ON CONFLICT (task_id, team_id)` has no plain UNIQUE left to infer — the quiz-per-student migration replaced it with partial indexes. **It reproduces against the real dump.** Nobody has hit it because all 56 real submissions are `per_student`, the photo path. `drive-uploads.js:389` has the identical defect. One line fixes each; on a copy it takes `tasks.js` from 9 passing to 43. Not fixed — `server.js` is Lane A's.

**Q5:** A5b is nearly empty. `v_student_progress` has exactly one reader, and no reader uses it for completeness — so A5b's "fix the readers" step has no work in it, and the reason for deferring it past the merge has gone.

**Ledger:** `2026-09-18-c` must not be re-run on a live database — it would silently drop `group_id` from `v_team_projects`, which two live routes read with `SELECT *`. Also, 12 of 15 migrations have no `down`.

Two moments are worth flagging because they nearly produced false greens. A sabotage proof went red for the wrong reason (`MODULE_NOT_FOUND`, not the permission hole) — caught only by asserting *why* it failed. And a stale server on port 3131 made a working fix look ineffective; the harness now refuses to start if the port is occupied.

No Lane A file was edited at any point, including transiently — the sabotage proofs run against throwaway symlink trees. `git status src/` is clean.

**Vishnu** (2026-09-19T13:06): BOTH LANES — the split ends. Vishnu's decision.

Lane A and Lane B become ONE lane. lanes.md set this condition itself: the
split is worth it "while there is genuinely independent work; not worth it once
everything funnels back into server.js." That moment is now.

WHY IT IS RIGHT
  - Lane B has finished all five independent items: A1, A2, A1b, A4, A5a.
  - A5b turned out to be empty. One reader, no completeness use.
  - A3 cannot be parallelised at all. One file, twelve serial items.
  - lanes.md estimated 1.5-1.7x, at roughly DOUBLE the tokens. There is no
    speed left to buy, so the cost has nothing to show for it.

THE ORDER
  1. LANE A finishes Track 2 — T2-16, T2-17, T2-18, plus the task-submit 500
     fix. Log everything; the log still ends at T2-08.
  2. Vishnu reviews the T2-17 screenshots. His checkpoint, not yours.
  3. survey merges into main. LOCAL TIDYING, NOT A DEPLOY.
  4. v3-dev rebases onto main.
  5. ONE lane does T3-A3, the twelve modules, alone.
  6. LANE B stands down after handing over.

LANE B — before you stop, write your handover into docs/agent-log.md: what A1b
found beyond the 500, anything in the ledger the next agent must not trip over,
and the state of the harness. You are the only one who knows it.

WHAT DOES NOT CHANGE
  - NO DEPLOY until everything is built. One build, one cutover, at the end.
  - Every task stays a PHOTO upload until that cutover. The 500 fix will exist
    in the code but will not be live.
  - The four guard lessons stand, and now apply to one agent:
      * sabotage at the caller's level, not an intermediate
      * sabotage the load-bearing line, not the plausible-looking one
      * a PASSING sabotage is a finding — weak test, or dead code?
      * assert WHY a sabotage went red, not just that it did

ONE WARNING. The ownership contract caught real mistakes today. Lane B refused
to fix a live bug because the file was not its own, and that discipline is why
noth

**Claude** (2026-09-19T13:06): Understood — the split ends, and I write the handover before standing down. Your message cut off mid-sentence at the warning, but the instruction to me is clear.

Let me verify current state, then write the handover.

**Claude** (2026-09-19T13:06): _[tool: Bash]_

**Claude** (2026-09-19T13:06): Lane A has progressed to T2-14/T2-15. `survey` hasn't merged yet, as expected. Let me check the harness's current state and one thing that will matter to the next agent.

**Claude** (2026-09-19T13:06): _[tool: Bash]_

**Claude** (2026-09-19T13:06): Important gap found: my route table iterates only what's *in* it, so Lane A's 8 new survey GET routes will be **silently untested** after the rebase. That's exactly the failure mode the table exists to prevent. Let me verify.

**Claude** (2026-09-19T13:06): _[tool: Bash]_

**Claude** (2026-09-19T13:06): My table covers all 36 GET routes on `v3-dev` exactly. But after the rebase, Lane A's 8 survey routes arrive and nothing will notice they're absent. Let me confirm that gap concretely.

**Claude** (2026-09-19T13:06): _[tool: Bash]_

**Claude** (2026-09-19T13:06): Confirmed: 44 GET routes on `survey`, my table covers 36 — **8 survey routes will arrive silently untested**. This is worth fixing now rather than describing, since it's a one-line guard in a file I own.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): 265 passing (36 new coverage checks). Now let me prove this new guard actually fires — per the standing rule, and because this is precisely a guard test.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): **A passing sabotage — which your fourth guard lesson says is itself a finding.** The guard didn't catch the added route. Let me find out why.

**Claude** (2026-09-19T13:07): _[tool: Bash]_

**Claude** (2026-09-19T13:07): There it is: the audit reads `env.ROOT/src/server.js` — the **real** file — while the server under test is the sabotaged copy. So the static audit has always inspected a different file than the one being exercised.

This also means Proof 2's guard-tier audit was reading the wrong file. Let me fix it.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:08): 13 proofs, 0 failed. The passing sabotage was a real defect — the static audit inspected a file nobody was running. Let me confirm the whole suite is still green and re-verify Proof 2 was genuinely sound.

**Claude** (2026-09-19T13:08): _[tool: Bash]_

**Claude** (2026-09-19T13:09): Proof 2 was sound — it asserts on a live HTTP response, not the static audit. Let me update the readme and commit this before writing the handover.

**Claude** (2026-09-19T13:09): _[tool: Bash]_

**Claude** (2026-09-19T13:09): Now the handover. Let me gather the concrete facts the next agent will need.

**Claude** (2026-09-19T13:09): _[tool: Bash]_

**Claude** (2026-09-19T13:09): Let me check one more thing the next agent will hit immediately: whether my harness survives the rebase.

**Claude** (2026-09-19T13:09): _[tool: Bash]_

**Claude** (2026-09-19T13:09): Concrete rebase hazard: the generator's clear-list doesn't include the three survey tables. `survey_answers` references `students`, so after the rebase `DELETE FROM students` will fail on a foreign key. Let me verify that prediction rather than assume it.

**Claude** (2026-09-19T13:09): _[tool: Bash]_

**Claude** (2026-09-19T13:09): `ON DELETE CASCADE` — so it won't error; the rows will be **silently cascaded away**. Worse than a failure: the generator would appear to work while leaving survey data inconsistent. Let me make the generator future-proof instead of just documenting it.

**Claude** (2026-09-19T13:09): _[tool: Bash]_

**Claude** (2026-09-19T13:10): No warning on the current schema (correct — nothing unknown). Let me prove the warning actually fires by simulating the post-rebase state.

**Claude** (2026-09-19T13:10): _[tool: Bash]_

**Claude** (2026-09-19T13:10): The warning fires exactly on the post-rebase state, naming all three tables. Let me restore the scratch DB and commit.

**Claude** (2026-09-19T13:10): _[tool: Bash]_

**Claude** (2026-09-19T13:10): Now the handover document itself.

**Claude** (2026-09-19T13:11): _[tool: Bash]_

**Claude** (2026-09-19T13:11): Let me verify the handover's factual claims before committing — every line number and count.

**Claude** (2026-09-19T13:11): _[tool: Bash]_

**Claude** (2026-09-19T13:11): Every claim verified. Let me check the schema.sql counts and confirm the suites one final time.

**Claude** (2026-09-19T13:11): _[tool: Bash]_

**Claude** (2026-09-19T13:12): 15 tables / 5 views confirmed. Let me check the exact `INT` spelling so the handover is precise.

**Claude** (2026-09-19T13:12): _[tool: Bash]_

**Claude** (2026-09-19T13:12): Confirmed — `INT` in schema.sql, `numeric` live. All handover claims verified. Let me commit and push.

**Claude** (2026-09-19T13:12): _[tool: Bash]_

**Claude** (2026-09-19T13:13): Handover written and pushed. Lane B stands down.

## LANE B — stood down

Handover is in `docs/agent-log.md`, newest at top. Everything committed and pushed to `v3-dev`; working tree clean; `git status src/` clean — no Lane A file was ever edited.

**Final state:** session suite **265 passing / 0 failed** · sabotage proofs **13 / 0** · `releases.js` **39 passing** against the fixture.

### Two things I fixed on the way out, both found by asking the handover question

Writing "what will bite at the rebase" surfaced two live gaps, and both were cheap enough to close rather than merely document:

**The route table was silently incomplete-proof.** It's *iterated*, so a route not in it isn't tested and nothing says so. It was complete for the 36 routes on `v3-dev` — which is exactly why the gap was invisible. Lane A's 8 survey GET routes would have arrived untested at the rebase. The audit now runs both ways. **Expect 8 failures after the rebase naming the survey routes — that is the guard working**; the fix is 8 rows in the table.

**The generator doesn't fill the survey tables**, which cascade from `students`. Their rows would vanish with no error, leaving a fixture that looks healthy while carrying no survey data. It now warns and names them.

### The fourth guard lesson fired last, and hardest

Proof 4 **passed** when it should have gone red. Real cause: the static audit read `src/server.js` at its canonical path while the suite exercised a sabotaged *copy* — it was auditing a file nobody was running. Accepting that pass would have left a coverage guard inspecting the wrong file.

Worth noting for the single lane: **the ownership rule is what caught it.** Because `src/server.js` wasn't mine, the proofs run against throwaway symlink trees via `HARNESS_SERVER` — and that seam is the only reason the defect was visible. Editing the file in place would have hidden it. Proof 2 was unaffected; it asserts on a live HTTP response, not the audit.

### On the critical path

**Q4 — the task-submit 500.** Two sites, one line each: `src/server.js:1374` and `src/routes/drive-uploads.js:389` need `WHERE NOT per_student`. Verified on a copy — `tasks.js` goes 9 → 43 passing. It reproduces against the real dump. Consistent with your order, the fix exists in code but isn't live: tasks stay photo uploads until the cutover, and all 56 real submissions go through the photo path, which never reaches the broken statement.

One caution: **`tests/tasks.js:283` will still fail after the fix** — it asserts a unique *constraint* that is now a partial *index*. Stale test, not broken code. Don't change the schema to satisfy it.

Your message cut off mid-word at the warning ("...and that discipline is why noth"). I've acted on the ownership rule as written throughout, but if there was more to it, it didn't reach me.