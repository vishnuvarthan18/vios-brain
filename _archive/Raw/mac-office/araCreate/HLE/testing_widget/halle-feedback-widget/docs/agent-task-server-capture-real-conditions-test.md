# Agent task — server-side capture: re-test under real conditions

**Read first:**
- `docs/agent-task-server-side-capture-ab-test.md` — the original test brief.
- `docs/report-server-side-capture-ab-test.md` — what you found last time.
- `agent-rules.md`

## Why this task exists

Your own report flagged that the first result was a **best case for the
server method, not a fair one**: the renderer ran on the same machine as the
browser (no real internet trip for the ~410KB payload), and it was idle
(nothing else competing for it). Vishnu wants that gap closed before treating
"keep today's method" as fully settled. He has already seen the first test
himself, side by side, and agrees today's method wins there — this task is
about checking that the win still holds under real conditions, not
re-litigating it.

## Ground rules — same as last time, plus one new one

- **No server upgrade.** Still the current production VPS, as is.
- **Nothing permanent added to the production box.** Whatever the renderer
  needs to run there (Chromium included) must be installed **for the
  duration of this test only**, and cleanly removed afterward if it isn't
  something Vishnu decides to keep. Do not leave a Chromium install sitting
  on the production server after this task is done.
- **Watch memory like a hawk.** The app is capped at `MemoryMax=1G` and the
  renderer + Chromium alone measured ~415MB last time. Before running
  anything, check current free memory on the box. If running the renderer
  there risks the app itself, **stop and report that instead of pushing
  through** — do not let a benchmark script take down the real app, even
  though nobody is using it yet.
- **One screenshot at a time**, same as before. No load testing, no
  concurrency work.
- **Today's method stays untouched and stays the only one used for real
  reports.** Same as last time.
- **Do not commit or push without asking Vishnu, as always.**

## What "real conditions" means here

1. **The renderer runs on the actual production server**
   (`feedback.arametrics.app`'s box), not on a laptop next to the browser.
2. **The browser making the test click is somewhere else** — genuinely
   sending its ~410KB page description over the real internet to reach that
   server, the way a real tester's phone or laptop would. (Running the test
   from a developer's own machine, pointed at the real server's address
   instead of `localhost`, satisfies this.)
3. **The server is not artificially idle.** If practical, run the comparison
   while the real app is also running normally on the box (it already is,
   in production) — do not shut anything down to give the renderer a clear
   run.

## What to build / do

1. Reuse the existing harness (`scripts/ab/capture-ab.mjs`,
   `scripts/ab/build-viewer.mjs`) — the comparison logic doesn't need to
   change, only where the renderer lives and what URL the harness points at.
2. Temporarily get the renderer (`src/render/renderer.mjs`) running on the
   production box, reachable only for this test (not exposed permanently or
   publicly beyond what's needed to run it once).
3. Run the same 3 clicks × 5 passes as last time (top-of-page,
   form-and-address, page-bottom) against the real Contact page, with the
   renderer now on the real server.
4. Record the same numbers as before: time for each method, pixels
   differing, any failures — plus, this time, note how much of the server
   method's time is genuinely network travel versus rendering itself.
5. Tear down whatever was added to the production box for this test, unless
   Vishnu says to keep it.

## Explicitly not in scope

- No permanent deployment of the server method to production.
- No load testing, no concurrency handling.
- No change to today's method, the marker pen, or the review screen.
- Do not pick a winner — report the numbers, same as last time.

## Report back

- Same format as `docs/report-server-side-capture-ab-test.md`: a results
  table (today's method vs server method vs pixels differing) for the same
  3 clicks.
- How much of the server method's total time was the real network trip
  versus the rendering itself, this time.
- Whether the app's memory stayed safe throughout, with the actual numbers.
- Confirmation that the production box was left clean afterward (nothing
  left installed/running that wasn't there before), unless told to keep it.
