---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_server-capture-ab.txt
---

feat: compare server-side screenshot against today's method

Answers docs/agent-task-server-side-capture-ab-test.md: a quality-and-speed
comparison of server-side rendering against today's client-drawn capture, on
the current server, without changing today's method.

Today's method is 2.4x faster (~410ms vs ~1,030ms) and has neither of the
server method's two defects. Full numbers and both sets of pictures in
docs/report-server-side-capture-ab-test.md and docs/ab-capture/index.html.

What is added, all of it alongside today's method rather than instead of it:

- serialise_page() in the widget's capture chunk, which reuses
  build_capture_clone() so the privacy masking, the off-screen prune and the
  scroll-offset shift are inherited rather than reimplemented. Referenced
  nowhere outside capture.ts, so nothing on the tester's path calls it.
- A renderer as its own process, one screenshot at a time, refusing a
  concurrent request with 503 rather than queueing it. Separate from the app
  on purpose: Chromium measures ~415MB RSS and the app unit is capped at
  MemoryMax=1G, so inside that cgroup a render would OOM-kill the app.
- An A/B harness capturing both ways from the same click at the same scroll
  position, medians of 5 passes, plus a side-by-side viewer.
- make ab-render and make ab-capture.

Two findings worth carrying forward:

- The server's picture sits a uniform 8px out, which would misplace every
  marker-pen stroke. Correcting for it drops the mean per-pixel difference
  from 19.03 to 0.27, so the two renders otherwise almost coincide. A fix
  via a position:fixed wrapper was tried and reverted, having made
  divergence worse (0.61% to 45.84%).
- A stray "Search" label appears because .search-container renders 297px
  wide instead of 218px: the live page sizes it with JavaScript, which the
  renderer does not run. Structural to server rendering, not a renderer bug.

Today's method is untouched: capture_screenshot() unchanged, all 51 widget
acceptance tests pass, tsc clean, v1.js unchanged at 8,508 bytes gzipped.
No dependency added, nothing installed on the server, no upgrade.
