# Agent task — try to fix the remaining capture accuracy problem without a new server

## Decision

Vishnu reviewed the hybrid-server numbers (`docs/agent-task-hybrid-
server-capture-feasibility.md` and its results) and decided: **no new
server for now.** Keep everything running in the tester's own browser, no
new monthly cost, no new infrastructure to maintain. Try to fix the
remaining accuracy problem within that constraint first.

## The problem, recapped

Diagnosed in `docs/agent-task-capture-still-broken-23-of-26.md` and its
follow-up: the DOM/CSS clone going into the capture is provably correct
(pixel-identical to the live page's own computed style), but the final
picture still comes out wrong on many screens — an 8px shift on one
element, cascading into bigger misalignment further down the page. The
cause was traced to the capture library's own rendering step
(`modern-screenshot`'s SVG-`foreignObject`-based `domToBlob`), not to
anything in this project's own clone-building code in `capture.ts`.

## What to try, in this order

1. **Check `modern-screenshot`'s own GitHub issues/changelog** for any
   known problem or fix related to text positioning/sizing accuracy
   inside its SVG `foreignObject` rendering — the same way
   `claude/research-scroll-fix-restoreScrollPosition.md` found a working,
   documented fix for the earlier scroll bug by checking the library's own
   issue tracker and other projects using it, instead of hand-rolling a
   workaround. Look for an existing option/flag, or a documented
   workaround from another project hitting the same symptom (small,
   compounding text-position drift).
2. **If nothing in modern-screenshot itself solves it, test `html2canvas`
   narrowly, just for this specific bug.** It was rejected earlier in this
   project (`claude/PROJECT-INDEX.md`, "Screenshot library" decision) for
   being unmaintained and slower than modern-screenshot generally — that
   decision stands for the general case. But html2canvas does NOT use the
   SVG-`foreignObject` trick — it walks the DOM and paints text directly
   onto a canvas — so it may not have this specific text-position bug at
   all. Test it only against the exact screenfuls that fail today (the
   navy card, the growing text box, `/contact` at various depths), measure
   both accuracy (same pixel-diff method as the existing audit tool) and
   speed, and report honestly whether it's worth it even given its known
   downsides.
3. **If neither of the above works**, look for any other in-browser
   technique (e.g. a different way of asking the browser to rasterize the
   clone, or a manual text-measurement correction applied to the clone
   before capture) — but check with Vishnu before spending much time on a
   fully custom approach, since that's a bigger, riskier undertaking.

## What NOT to do

- Do not add a server-side step of any kind.
- Do not implement a swap to html2canvas as the new default without
  reporting the real numbers first — this is a test, not a decision.
- Do not touch the scroll-fix or hero-fix code that's already working.

## Report back

- What, if anything, modern-screenshot's own issue tracker says about this.
- If html2canvas was tested: real pixel-diff numbers on the known-failing
  screenfuls, and real speed numbers, side by side with today's method.
- A clear recommendation: is there a real, no-new-server fix here, or does
  this confirm the accuracy ceiling found yesterday is a genuine limit of
  staying in-browser.
