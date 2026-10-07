**Vishnu** (2026-09-22T01:51): Set up the full local environment for manual testing, on this machine, right now. Read docs/local-test-plan.md §0 and get make demo working — one command that starts the database check, migrations, seed data, the dev app, the widget build, and the static test host page (tests/widget/host-page.html), all together. Print the URLs at the end, including a working tester link with a real token already on it, ready to open in a browser. Do not touch production. Do not fix any bugs — just get the local environment running and give me the command + links.

**Claude** (2026-09-22T01:51): I'll start by reading the test plan and getting oriented.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:51): The project is in a subdirectory. Let me look there.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:51): Now let me read §0 of the test plan and the current Makefile.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:51): The plan asks for exactly the `make demo` target. Let me check the current Makefile and the stack.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:51): `make demo` already exists. Let me inspect the script and check the environment.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:51): The `make demo` target is already fully written and covers everything asked. Let me check prerequisites before running it.

**Claude** (2026-09-22T01:51): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Environment looks good — Postgres 17 is running and `.env` is populated. Let me verify Postgres responds and check the `db:demo` script exists.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Everything is in place, and there are already stale servers on both ports from a previous session. Let me stop those first so `make demo` starts clean and records its own PIDs.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Understood — that changes my plan. I was about to stop the servers on ports 3000 and 4319; I won't do that now. They likely belong to the other agent.

Let me identify what's running before touching anything.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Both servers are long-running and predate my session (test server from Sept 9, Next app from Sept 21 11:40). The Next app on 3000 is very likely what the other agent is using for its bug fix. Let me check for any active build/test processes too, then verify both servers are actually healthy.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Both servers are healthy and serving. There's a second next-server (84349) too. `demo.sh` is already written to detect running servers and leave them alone — so I can run it safely without conflicting.

Let me verify that skip-logic is genuinely safe, and confirm `demo.sh` won't write to any source files.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): `demo.sh` only writes inside `.demo/` and builds to `dist/` — both gitignored, no source files touched. 

One concern: the widget build rewrites `src/widget/dist/v1.js`, which the other agent might be mid-edit on if their bug fix is in widget code. Let me check what they're working on before I trigger a rebuild.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Confirmed the conflict: the other agent is actively editing `src/widget/src/app.ts` and `styles.css` — exactly the widget sources that `make demo` would rebuild into `dist/v1.js`. The dist was rebuilt at 07:19, minutes ago, by them.

Running the full `make demo` would rebuild the widget mid-edit, possibly from half-saved source, clobbering their build artifact. I'll run every step **except** the widget build, reusing their fresh dist.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Good — it's dev-guarded and safe. Now let me run the database steps directly (skipping the widget build).

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Migrations applied (notices are just idempotent skips). Now seed, fixture, and demo logins.

**Claude** (2026-09-22T01:52): _[tool: Bash]_

**Claude** (2026-09-22T01:52): Seed and fixture complete. Now the demo logins and tester token.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): Got a real token. Now let me write the demo host pages (into gitignored `.demo/` only), matching what demo.sh generates.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): That page already carries the correct current public key (`pk_live_3ea7db3f`) and API origin, so it's valid as-is — I don't need to rewrite it. Let me verify the bad-key page and confirm everything serves correctly end to end.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): All pages serve. Now let me verify the token actually authenticates against the running app — that's what makes the tester link genuinely "working" rather than just well-formed.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): `/api/config` is a 404 — I guessed that path. Let me find the actual endpoint the widget calls.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): It's `/api/v1/config`. Let me verify the real token and key work through the actual endpoint.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): The API works end to end: real key returns config, bad key 404s. Let me confirm the token itself is valid in the database and check how the widget gates the launcher on it.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): Note: `v1.js` now shows 07:23 — the other agent rebuilt it again while I worked, confirming they're actively building. Good thing I skipped the build step. Let me verify the token row.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:53): Token is live and not revoked. Everything is verified. Let me do a final end-to-end confirmation of all the links.

**Claude** (2026-09-22T01:53): _[tool: Bash]_

**Claude** (2026-09-22T01:54): Everything is up and verified. No code edited, no git commands, no processes killed.

## The environment is ready

**Your command:**
```sh
cd ~/araCreate/HLE/testing_widget/halle-feedback-widget
make demo
```

`make demo` already existed and does exactly what §0 asks — I didn't need to write it. **But I did not run it in full today**, for the reason below.

### Working tester link — open this
```
http://localhost:4319/.demo/host-page.html?t=-TdVUUR-bhHjc3pZnEaqUxEQ
```
Token is live in the DB and verified against the real API, not just well-formed.

### The rest of the URLs
| What | URL |
|---|---|
| App / admin | http://localhost:3000 |
| Login | http://localhost:3000/login |
| Test page **without** token (§1.2 — launcher must NOT appear) | http://localhost:4319/.demo/host-page.html |
| Invalid key (nothing must appear) | http://localhost:4319/.demo/host-page-bad-key.html |
| Second page (token-survives-navigation) | http://localhost:4319/tests/widget/host-page-2.html |

**Logins** — `staff@demo.test`, `developer@demo.test`, `client@demo.test`, all password `demo-pass-123`. (They're interchangeable; roles were removed in the v2 spec.)

### What I ran, and the one step I deliberately skipped
Ran: DB check → migrations → seed → dev fixture (21 testers, 21 reports) → demo logins/tester.

**Skipped: the widget build.** The other agent is mid-fix on the letterbox bug in `src/widget/src/app.ts` and `styles.css` — the exact sources `make demo` compiles into `dist/v1.js`. They rebuilt that file at 07:19 and again at 07:23 while I was working. Rebuilding would have raced their build, possibly from half-saved source. You're testing **their current build**, which is fresh.

Also left alone: the Next app (pid 85414, running since Sept 21) and the static server (pid 72190) — both already healthy, so `make demo` would have skipped them anyway by design.

### Two things worth knowing
- **Don't run `make demo-stop`** while the other agent is working — the test-server pidfile points at the pre-existing server (72190), so it would kill a server it didn't start.
- **Once the other agent finishes**, run the full `make demo` to pick up their fixed widget build. It's safe to re-run and will skip the running servers.

One caveat on the letterbox bug itself: since you're on their in-progress build, if the pointer box misaligns during §1.4, that's the bug they're fixing — not your environment.