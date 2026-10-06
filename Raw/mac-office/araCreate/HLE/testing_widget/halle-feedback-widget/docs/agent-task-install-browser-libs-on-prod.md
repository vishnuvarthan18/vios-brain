# Agent task — install the missing browser libraries on the production server, carefully

## Decision, overriding an earlier standing rule

This project's earlier decisions (`claude/session-handover.md` §4/§9,
`claude/PROJECT-INDEX.md`) explicitly closed this question: "no server
upgrade, ever, for this test" — because the production server
(`212.227.213.174`) is shared with an unrelated project (JupyterHub), is
missing 9 system libraries a real browser needs (`libnspr4.so`,
`libnss3.so`, `libnssutil3.so`, `libatk-1.0.so.0`,
`libatk-bridge-2.0.so.0`, `libXdamage.so.1`, `libxkbcommon.so.0`,
`libasound.so.2`, `libatspi.so.0` — confirmed twice, most recently in
`docs/test-chrome-headless-shell-21-sept.md`), and has no swap space at
all with RAM often tight (dipped to 109MB free during a prior test).

**Vishnu has now explicitly overridden that standing rule**, after being
told the risks plainly (shared box, no swap, ~19 packages including
desktop/session software like `dbus-user-session`/`at-spi2-core`), because
he'd rather use the existing server than pay for a new one. This is a
deliberate, informed decision — proceed, but carefully, per the steps
below. **Record this override in `docs/agent-rules.md`** the same way the
21 Sept widget-migration override was recorded (see that file's "Amended
21 September 2026" section for the pattern to follow) before or as part of
this work.

## Why this is needed

Two free, no-new-server attempts were tried today and both fell short:
1. Tuning the existing capture library — the bug is upstream/unfixed
   (`docs/agent-task-tune-in-browser-capture.md`'s results).
2. Swapping to html2canvas — introduces its own unfixable bugs
   (`docs/agent-task-html2canvas-full-check.md`'s results, 64% pass rate,
   two new defect classes, 4x size increase).

The earlier "hybrid" test (`docs/agent-task-hybrid-server-capture-
feasibility.md`) proved that photographing the already-correct page clone
with a REAL browser — instead of either the buggy SVG trick or the buggy
html2canvas — fixes the accuracy problem essentially completely. That test
ran on this Mac, which already has a working browser. Production doesn't,
which is what this task fixes.

## Do this carefully, in order — stop and report after step 4, before wiring anything

1. **Take stock first, before touching anything.** Check and record: free
   disk space, free RAM, what's currently running (confirm JupyterHub and
   the halle-feedback app are both healthy right now, as a "before"
   baseline). If the hosting provider offers a snapshot/backup feature for
   this VPS, check whether one exists or can be taken first — ask Vishnu
   if you're not sure how, don't guess at billing-affecting actions.
2. **Install only the needed libraries**, via `apt`, as narrowly as
   possible. Expect this to pull in a dependency chain (~19 packages
   total, based on earlier investigation) — that's expected, not a sign
   something's wrong. Do not install anything beyond what's needed to make
   a headless browser (Chromium/Playwright/Puppeteer) actually launch.
3. **Verify nothing else broke.** After installing: confirm JupyterHub is
   still running and reachable, confirm the halle-feedback app is still
   running (don't restart it unless something requires it — if a restart
   is needed, treat that as noteworthy and report it clearly), check free
   RAM and disk again and compare to the "before" numbers from step 1.
4. **Confirm the actual goal**: launch a real headless browser on the
   server now and take one test screenshot, to prove the missing-library
   problem is actually solved (the same check that failed in
   `docs/test-chrome-headless-shell-21-sept.md`).

**Stop here and report back** — do not wire the capture pipeline to
actually use the server yet. That's a separate follow-up task once this
is confirmed safe and working.

## Report back

- The "before" and "after" numbers (disk, RAM, both other services' health).
- Exact list of packages actually installed.
- Confirmation a real headless browser now launches and can screenshot
  something.
- Anything unexpected, even if it seemed to resolve itself.
