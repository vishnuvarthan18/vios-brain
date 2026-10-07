# Agent task — push, then deploy this round's fixes to production

22 September 2026. Vishnu's decision: push the 4 commits already made
(`8c6b185`, `e247408`, `38aa6ba`, plus `0f01492` already pushed earlier)
to GitHub, then deploy them to the live production server so he can test
the box-position fix himself on his own phone against the real site.

Background: `docs/agent-task-fix-box-speed-marker.md` and its followup (the
work being deployed), `docs/agent-task-deploy-server-capture-live.md` (the
last successful deploy of this kind — same server, same process, follow
that pattern). `deploy/runbook.md` is the actual deploy procedure; it was
already fixed for the `npm ci` / stale-IP issues during the last round, no
reason to expect either again, but check anyway.

**Do not deploy anything beyond what is described here. Stop and report
rather than fixing anything unexpected under pressure — same standing rule
as every previous deploy in this project. This is the SAME shared,
memory-tight server as before (no swap) — treat it with the same care.**

## Steps

1. **Push first.** `git push origin dev` from your own environment (the
   device bridge cannot authenticate to GitHub — confirmed last round).
2. **Health check the server BEFORE touching anything** — same checks as
   the last deploy report: memory free, both services currently running
   (`halle-feedback`, `halle-feedback-hybrid-render`), current git SHA on
   the server.
3. **Deploy via `deploy/runbook.md`.** This includes a DB migration this
   time — the new `reports.capture_method` column
   (`38aa6ba`). `make db-migrate` needs Node 22.6+; the server's system
   Node is 20 — use the already-installed `/opt/node22` the same way as
   the last two deploys (`export PATH=/opt/node22/bin:$PATH` before that
   one command, nothing else touches the system Node).
4. **Restart BOTH services, not just the app.** This round's B1 fix
   changed `src/render/hybrid-renderer.mjs` itself (the resource-blocking
   change), so `halle-feedback-hybrid-render.service` needs a restart too,
   not only `halle-feedback`. Confirm both are back up and healthy
   (`/health` on the renderer, same as last time) before moving on.
5. **Health check AFTER.** Same shape as before: memory free, both
   services up, git SHA matches what was just pushed.
6. **Submit one real test report** on the live site yourself (pick an
   element, add a comment, send) — confirm it completes normally and, if
   you can check the database directly, confirm the new report's
   `capture_method` column is populated (not null) — that is the signal
   the whole Part C change actually reached production correctly.
7. **Report back**, then stop. Vishnu will do his own phone test against
   the live site after this — do not consider the box-position fix
   confirmed until he has.

## Do not do

- Do not raise `MAX_CONCURRENT_RENDERS` or touch any of its safety limits.
- Do not install anything new — nothing in this round's changes needs a
  new dependency or system library.
- Do not "fix" anything you notice that is unrelated to this deploy while
  you're on the server. Report it separately instead.

## Report back

Before/after health numbers, confirmation both services restarted cleanly,
the test report's result (including whether `capture_method` was
populated), and anything that needed a judgment call.
