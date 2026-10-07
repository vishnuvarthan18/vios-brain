# Agent task — deploy the server-side capture live, carefully

## Decision

Vishnu reviewed the real numbers from `docs/agent-task-wire-server-
capture.md`'s results (0.233% accuracy difference — essentially fixed;
~2 seconds total, slower than first estimated; server memory gets tight
during a render) and decided to go live and test it for real.

## What to do

1. **Push the three local commits** (`9feb57c`, `338a94c`, `9671728`) to
   `origin/dev`. Check `git log --oneline dev ^origin/dev` first to
   confirm exactly what's going out.
2. **Deploy to the production server**, following the existing deploy
   process in `deploy/runbook.md` (already corrected today — use the real
   commands from it, not the old broken ones).
3. **Start the new renderer process properly** — not just as a manual test
   run like before, but as a real, persistent service (systemd, matching
   how `halle-feedback` itself runs, so it survives a server reboot and
   restarts if it crashes). Follow whatever pattern the existing
   `renderer.mjs` service (from the 10 Sept A/B test, if it was ever set
   up as a service) or the main app's own systemd unit uses.
4. **Turn the feature on**: set `HYBRID_RENDER_URL` (or whatever the real
   env var name is — check `capture.ts`/the new endpoint code) in the
   production environment, then restart the app so it picks it up.
5. **Verify carefully, in this order, before telling Vishnu it's ready**:
   - Confirm both `halle-feedback` and JupyterHub are healthy after the
     restart (same before/after check pattern used in the library-install
     task).
   - Submit one real test report through the actual widget on the live
     site and confirm the screenshot comes back correct.
   - Watch server memory during that real report — confirm it behaves the
     same way as the earlier isolated test (dips but recovers, doesn't
     crash anything).
   - Confirm the one-at-a-time limit is actually in effect on the real
     deployed service, not just in the test code.
6. **Do a final live memory/health check** a minute or two after, to
   confirm nothing degraded after the restart settled.

## If anything looks wrong at any point

Stop and report rather than trying to fix forward under pressure — this is
now affecting production. The fallback path (client-side capture) means
worst case is "no faster/more accurate than before," not "broken" — so
there's no need to rush a fix live if something looks off; report it and
wait for direction.

## Report back

- Confirmation of what's now live (commit hash, service status).
- The real report you submitted and its result.
- Real memory numbers from the live test, compared to the earlier isolated
  test numbers.
- Anything that needs Vishnu's attention before this is considered fully
  settled.
