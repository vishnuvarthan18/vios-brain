# Agent task — test the "server photographs the already-built clone" idea, with real numbers, no new infrastructure yet

## Background

Today's diagnosis (`docs/agent-task-capture-still-broken-23-of-26.md` and
its follow-up) found that the capture library's own SVG-rasterization step
introduces small but real inaccuracies (an 8px shift on a navy card, text
boxes growing/moving) even when the DOM/CSS going into it is provably
pixel-identical to the live page. This is a limit of the SVG-foreignObject
rendering trick `modern-screenshot` uses, not something fixable by more
tuning of `capture.ts`'s clone-building logic (which has already been
proven correct at this point).

Vishnu's decision: before committing to a new server or any ongoing cost,
get real evidence on whether photographing that same, already-correct
clone with an actual real browser (instead of the SVG trick) actually
fixes this — and what it would cost/how fast it would be. **Do not
provision, pay for, or set up any new persistent server for this task.**
Test the core idea locally first, using this Mac's own working headless
browser (unlike the production server, this machine has the needed system
libraries).

## What to test

1. **Does it actually fix the bug?** Take the exact same clone-building
   code already in `capture.ts` (the part that produces a DOM clone with
   ~123 computed style properties baked in per node — already proven
   pixel-identical to the source page today). Instead of handing that
   clone to `modern-screenshot`'s SVG-based `domToBlob`, render it with a
   real local headless browser (Puppeteer or Playwright, already available
   or installable on this Mac) — load the clone's HTML in a real page and
   use its native `page.screenshot()`. Compare that output, pixel by
   pixel, against a real browser's screenshot of the live page, for the
   same screenfuls that failed in today's audit (the navy card, the
   heading whose box grew/moved, deeper scroll positions). Report whether
   the specific defects found today are actually gone.
2. **How fast is it, locally?** Measure real render time for this
   real-browser screenshot step, same pages/viewports as before. This
   won't include real network latency (see below) but establishes the
   rendering half honestly.
3. **What would the network cost be?** Don't guess — reuse the real
   number already measured in this project: `claude/report-server-
   capture-real-conditions-summary.md` found ~260-350ms round trip,
   client to production server, over the real internet. Note that as the
   expected added cost of sending the clone to any small external server,
   and flag if there's reason to think a new, purpose-built small server
   would differ meaningfully.
4. **What would a small dedicated server actually cost per month?** Look
   up real, current pricing (not a guess) for a small VPS suitable for
   running a headless browser reliably (2GB+ RAM recommended, given the
   production box's memory problems found in earlier testing) from a
   couple of real providers (e.g. Hetzner, DigitalOcean). Report actual
   current prices.

## What NOT to do

- Do not sign up for, provision, or configure any new server.
- Do not touch the production server.
- Do not commit or push any code — this is a feasibility test, run
  locally, thrown away or kept as scratch code, your call, but nothing
  goes into the real capture pipeline from this task.

## Report back

- Did the real-browser render actually fix the specific bugs found today?
  Show the pixel-difference numbers, same format as the existing audit
  tool.
- Local render speed, honestly labeled as "render only, no network".
- The real ~260-350ms network figure, cited from the existing report.
- Real current monthly pricing for a couple of small-VPS options.
- A plain summary: given these numbers, does this approach look worth
  pursuing for real, or not.
