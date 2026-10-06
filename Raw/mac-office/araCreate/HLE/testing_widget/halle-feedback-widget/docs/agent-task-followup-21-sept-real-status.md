# Agent task — real remaining items, 21 September (late evening check)

Written after checking the actual repo state directly (git log, file
timestamps) rather than relying on session notes, which had fallen behind
what an earlier session today already finished.

**Already done — do not redo these:**
- Admin dashboard migrated to React + shadcn/ui (commits `8bfcc25`..`43fbe0a`).
- Widget migrated to React + shadcn/ui, all six screens (commits `aea41a3`
  through `f36c146`, including the shadow-dom re-verification in `f425d19`).
- `docs/agent-rules.md` section 1.4/1.9 already carries the override note
  (21 Sept section, already committed).
- The scroll-misframing capture bug and the hero-blank capture bug are both
  fixed (`0965f27`, `6d73f2c`, `f36c146`).
- **All of the above is already committed AND pushed** — `dev` and
  `origin/dev` are in sync right now, nothing ahead.

Do the three items below, each its own commit, **commit locally only — do
not push** until told to.

---

## 1. Fix the broken capture audit, then actually run it

`audit-report.md` at the repo root (dated today) shows all 6 checked
screens failing with the same error: `Failed to fetch dynamically imported
module: https://feedback.arametrics.app/capture-te...`. This is not a real
result — the audit tool itself is broken, so nobody actually knows yet
whether the 16-screen capture problem found earlier today is fixed.

- Find out why `scripts/audit-capture.mjs`'s dynamic import of
  `capture-test.js` from the live server is failing (check: does that file
  actually exist at that path on the production server right now? Wrong
  MIME type? CORS? Was it removed or renamed by the React rebuild?).
- Fix it, then run the audit for real across all 16 screens (desktop and
  mobile), not just the 6 in the current broken report.
- Report the real pass/fail numbers.

## 2. Fix the two known bugs in `deploy/runbook.md`

Confirmed still present, file untouched since 9 Sept:

1. The documented deploy command `npm ci --omit=dev --ignore-scripts`
   strips `esbuild`, which the widget build needs. Replace with
   `npm install`.
2. Every example command in the file still uses the old placeholder IP
   `212.227.213.174` (lines 21, 36, 51, 235, 266, 316, 330, 413, 593
   at least) instead of the real live address
   `https://feedback.arametrics.app`. Replace all of them.

## 3. Check whether production is actually running today's build yet

This is the important one. The widget went from about 25KB to about 260KB
today (`v1.js`) and the admin dashboard's whole UI changed. Before treating
any of this as finished:

- Check the production server directly (commit hash currently deployed,
  restart time) the same way past deploys in this project were verified —
  do not just assume today's local build is live.
- If it is NOT deployed yet, say so clearly and stop — do not deploy it
  without Vishnu's explicit word, this is too large a change to push live
  silently.
- If it IS already deployed, say so and report what was checked to confirm
  it.

---

## Explicitly OUT OF SCOPE — still needs a decision from Vishnu first

- The marker-pen fixes.
- The screen-by-screen "16 screens" UX redesign pass (different from the
  audit in item 1 — this is a content/design review, not a pixel check).
- Switching screenshot capture from client-side to server-side rendering.
- Rotating the GitHub personal access token on the production server.
- "Notify tester when fixed" / "notify team on new report" features.
- Console-error redaction.
- The 12px "text-micro" accessibility question.

## Report back

For items 1-3: what you found, what you fixed, and the real numbers/status
— not just "done". Do not push anything without being told to.
