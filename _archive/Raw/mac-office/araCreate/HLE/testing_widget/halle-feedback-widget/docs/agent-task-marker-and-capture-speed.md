# Agent task — marker pen quality, and two capture-speed fixes

Background and evidence: `docs/plan-speed-marker-ui.md` (§1 and §2) for
the live trace numbers, and
**`docs/research-how-the-industry-solved-this.md`** for the industry
survey that settled the architecture. Read both first. This file is the
actionable subset, with Vishnu's decisions already applied.

**Settled by that research and by Vishnu on 10 September — do not
re-open:**

- **Freeze-then-draw stays.** No shipping tool in the category lets a
  user draw freehand on a live scrolling page. An earlier proposal in
  this project to do that is **withdrawn**.
- **Route 1: fix the client pipeline, measure honestly against 500ms.**
  Server-side rendering (what Marker.io, Ybug, Usersnap and Userback all
  do by default) is held **in reserve** — to be built only if Route 1
  misses the budget, and then with measured numbers in hand.
- **Viewport only. No full-page capture, ever, in the widget.** iOS caps
  canvases at 4,096px and exceeding it yields a **blank image with no
  error**. No embedded JS widget in the survey ships reliable full-page
  capture.
- **Native screen capture is not an option.** The permission can never be
  persisted (W3C Working Draft, 27 August 2026), the picker cannot be
  removed, and there is zero mobile support on any browser.
- **The drawing surface becomes full-size** — see A8.

All of this is bug-fixing. No product decisions are open except where
marked "report back, do not decide".

**Do not commit or push without asking Vishnu at the time**, as always.

---

## Part A — the marker pen (highest priority)

Vishnu's words: the line drawing is the worst part of the widget. Six
separate defects, all confirmed by reading
`src/widget/src/marker-pen.ts` and the review-screen section of
`src/widget/src/app.ts`. They are independent — fix all six.

### A1. The canvas is at half resolution on any Retina/HiDPI screen

`app.ts` (the `img.addEventListener('load', ...)` handler that sizes the
marker canvas) sets `canvas.width = img.clientWidth` and
`canvas.height = img.clientHeight` — the backing store is sized in **CSS
pixels**. On a device with `devicePixelRatio` 2 or 3 every line is drawn
at a fraction of the screen's real resolution and stretched up by the
browser. This is the single biggest reason the line looks bad.

Fix: size the backing store as `clientWidth * devicePixelRatio` (same for
height), keep the CSS size as-is, and scale the 2D context by
`devicePixelRatio` so drawing coordinates stay in CSS space. Make sure the
stroke-scaling maths at send time (see A2) still lands correctly after
this change — the two interact.

### A2. What the tester draws is not what gets sent

The preview canvas is the size of the **displayed** image (a few hundred
CSS pixels wide inside the panel). The captured image is
**viewport-sized** (~1,800px wide). At send time `app.ts` correctly scales
the stroke *coordinates* up into image space — but `burn_in_markup()` in
`capture.ts` then draws them with a fixed `MARKER_WIDTH = 4`, the same
number the small preview used.

Net effect: a line drawn as a bold mark on a ~300px preview is burned into
the sent picture roughly six times thinner in proportion. The dashboard
picture does not match what the tester drew.

Fix: scale the burned-in line width by the same factor the coordinates are
scaled by, so the mark is proportionally identical to the preview.
`BOX_WIDTH` (the element box) has the same class of problem — check
whether the box outline is drawn in image space with a preview-sized width
and correct it the same way if so.

**Vishnu's decision: keep today's visual weight.** The line should look
the same thickness it looks now in the preview — do not make it bolder or
thinner, just make the sent picture agree with the preview.

### A3. No smoothing — the line is a raw polyline

`redraw()` walks recorded points with `moveTo`/`lineTo`, so any hand
movement that is not slow and steady shows visible corners.

Fix: render each stroke as a smooth curve through its sample points.
Quadratic curves through segment midpoints is the standard cheap approach
and is a few lines of maths. **No new dependency** — `agent-rules.md`
forbids adding one to `src/widget/`, and `modern-screenshot` is the single
existing exception, for `capture.ts` only.

The same smoothing must be used in **both** the live preview
(`marker-pen.ts`) and the burned-in composite (`burn_in_markup()` in
`capture.ts`), or the sent picture will not match the preview again by a
different route. Consider whether the stroke-rendering maths can be shared
between the two rather than written twice.

### A4. Every pointer move redraws everything, then draws the last bit twice

`on_pointer_move()` calls `redraw()` — which clears the canvas and redraws
**all** strokes — and then separately strokes the newest segment on top of
what it just drew. Two consequences: the newest segment is drawn twice so
it renders denser than the rest of the line, and the work per pointer-move
grows with everything drawn so far, so the line lags behind the finger on
a busy drawing and cuts corners.

Fix: draw the in-progress stroke incrementally without redrawing finished
strokes on every move (a second canvas for committed strokes is one clean
way; there are others). The double-draw of the newest segment must go.

### A5. Fast strokes lose their samples

The pen listens to `pointermove` only, and browsers batch pointer movement
to the frame rate, so a quick flick loses the points in between.

Fix: use `PointerEvent.getCoalescedEvents()` where available for the full
high-frequency sample set, falling back to the single event where it is
not.

### A6. A tap draws nothing

`end_stroke()` discards any stroke with fewer than two points, so tapping
a spot — a natural way to say "here" — leaves no mark, with no explanation.

Fix: render a single-point stroke as a dot, in both the preview and the
burned-in composite.

### A7. The pen must be a tool you turn on — not always-on

**Vishnu's instruction, 10 September.** Today the marker canvas sits over
the picture permanently and captures every pointer press, so drawing is
always live: any accidental drag across the picture leaves a red line, and
on a touch device a tester trying to *scroll* with a finger on the picture
draws instead of scrolling.

Required behaviour:

- The pen is shown as an **icon** in the review screen's marker controls.
- **Drawing is off by default.** With the pen off, the marker canvas must
  not intercept pointer input at all (`pointer-events: none` on the canvas,
  not merely ignoring the events) so the picture behaves like a plain
  picture and normal scrolling/touch works.
- **Clicking the pen icon turns drawing on**, and it stays on until the
  tester turns it off again — do not switch it off after each stroke.
  Clicking it again turns it off.
- The on/off state must be **obvious at a glance** — a clearly selected
  state on the icon, using the design system's own colours
  (`docs/halle-design-system-draft.md`), not a subtle tint. Change the
  cursor to a crosshair over the picture while the pen is on.

Constraints that apply:

- **No new dependency and no external icon font** — inline SVG only
  (`agent-rules.md`). The design system specifies outline-style icons with
  a consistent stroke weight, and sizes of 18/24/32/36px; the pen is a
  button icon, so 36px in a tap target of **at least 44px** (the widget's
  existing minimum tap target, enforced by `computed-styles.spec.ts`).
- An icon-only button still needs an **accessible name**, and that text
  must be **editable in admin** like every other tester-facing string —
  mind the Wording-screen field-list trap again (a string missing from
  that list reaches testers with no way to change it; there is now a guard,
  make sure the new string satisfies it).
- Undo and Clear stay as they are. Consider disabling them while nothing
  has been drawn yet, since they do nothing in that state — do this only
  if it is genuinely trivial; do not redesign that row, the review screen
  is getting its own dedicated design pass with Vishnu straight after this
  task.

### A8. The tester draws on a FULL-SIZE frozen picture, not a thumbnail

**Vishnu's decision, 10 September**, taken on the industry research in
`docs/research-how-the-industry-solved-this.md`.

The frozen picture currently renders as a small preview inside the widget
panel, and the tester draws on that. This is the root cause of A1 and A2
— a tiny canvas at the wrong resolution, whose coordinates then have to
be scaled up by roughly 6× into the real image.

Required: the frozen picture is presented **at full size** — filling the
screen (or as near as the viewport allows) — and the tester draws on it
at 1:1 with the captured image. The comment box and Send/Cancel move to
sit over or beside it rather than the picture sitting inside a small
panel.

Why this matters beyond looks: at 1:1 there is **no scaling factor left
to get wrong**, which fixes A2 at the root rather than by arithmetic, and
it removes the cramped drawing area that made careful marking impossible.
This is what every tool in the survey does — Marker.io, Ybug, Usersnap
and Jam all hand the annotator a full-size frozen image, never a
thumbnail.

Keep the design-system values (`docs/halle-design-system-draft.md`) and
the existing minimum tap targets. This is a layout change to the review
screen; the screen gets its own dedicated design pass with Vishnu
straight after this task, so build it cleanly but do not gold-plate it.

### Testing note for Part A

The existing widget tests did not catch any of these, because they assert
structure rather than rendered output. At minimum add coverage that would
fail if A2 regressed (preview stroke width vs burned-in stroke width in
image space), if A1 regressed (backing-store size vs `devicePixelRatio`),
and if A7 regressed (a pointer drag across the picture with the pen OFF
must produce no stroke, and must not swallow the event). A visual check on
a real Retina screen is worth more than any of them — say so in your report
if you cannot do one.

---

## Part B — capture speed: a hard budget of 500ms

**Vishnu's requirement, 10 September: the picture must appear within
500ms of the click, on the real page.** His reasoning is a product one and
it is correct — a tester who waits six seconds abandons the report. Treat
500ms as the budget this work is measured against, not an aspiration.

Live baseline on the real Contact page: clone built in 11ms, first capture
pass finished at 5,846ms, final result at 11,576ms.

**Constraints that do NOT move** (Vishnu has ruled on each):

- The picture is still taken **when the tester clicks**, not before.
  Pre-capturing the image is declined — see "Explicitly NOT in scope".
- **No screen-share permission prompt** (`getDisplayMedia`). The audience
  includes elderly, non-technical testers.
- A failed or slow capture must still never block a report
  (`agent-rules.md §1.11`).

So the budget has to be met by making the capture itself cheaper. Work in
the phases below **in order**, and do not skip Phase 0 — the current trace
is too coarse to tell which lever matters, and this project has already
been burned twice by optimising on an assumption instead of a measurement.

### B1. Stop paying Safari's workaround on every browser

`capture_screenshot()` in `capture.ts` calls `capture_once()` **twice** and
keeps only the second result, on every browser. The code comment justifies
this as "a redundant second capture is cheap" — measured at 98ms on the
test fixture, but **5.8 seconds on the real page**. Roughly half the
tester's wait is a second capture that only Safari needs.

Do both of these:

- **B1a. Fix the cause, do not sniff the browser.** Safari's blank first
  render is a *readiness* problem, not a Safari quirk that needs a second
  throwaway render. Await `document.fonts.ready`, and `await img.decode()`
  on the generated image, before rasterising — then the first pass is
  already correct and the second is unnecessary on every browser,
  including Safari. This is a correctness fix rather than a workaround,
  and it removes the whole second pass instead of hiding it behind a
  user-agent check. Verify on Safari specifically before removing the
  second pass; if it still comes back blank there, fall back to
  conditioning the second pass on WebKit-not-Chrome and say so in the
  report.
- **B1b.** Where the second pass still runs, stop it re-doing all the
  work. Each `domToBlob()` call currently creates its own fresh context,
  so pass 2 re-fetches and re-embeds every image and font from scratch
  with no cache sharing. modern-screenshot supports creating a context
  once and reusing it across calls — use that so the second pass is
  near-free.

Update the code comment above `capture_screenshot()` at the same time. It
currently states the "second capture is cheap" reasoning as fact, and that
is precisely the assumption that turned out to be wrong on a real page —
leaving it there invites the same mistake again.

### B2. Stop showing testers "(no picture)" while it is still working

`CAPTURE_FIRST_PAINT_MS = 3000` in `app.ts` opens the comment box after 3
seconds whether or not the picture is ready. That behaviour is correct and
must stay — do not change the timing, and do not turn the first-paint
deadline back into a give-up (there are comments in the file explaining
why; they are right).

The problem is what is *shown*. The picture-less state renders the literal
text **"(no picture)"**, which reads as "this is broken", for up to nine
more seconds before the picture quietly appears.

Fix: while a capture is still in flight, show a "taking a picture of the
page…" state instead. Fall back to the existing "(no picture)" wording
only once the capture has actually failed or its 12s budget is spent —
i.e. the two states must be distinguishable.

The new string must be **editable in admin like every other string**. Note
the trap found last round: the Wording screen has an explicit field list,
and a string missing from it reaches testers with no way to change it —
there is now a guard for this, make sure the new string satisfies it.

### Phase 0 — instrument properly first. No optimisation before this.

The current trace has four coarse events, which is why nobody can say
where the 5.8 seconds goes. Break the capture into named phases and time
each one, on the **real** Contact page (the local fixture is what produced
the wrong "second capture is cheap" conclusion — do not tune against it):

1. clone + prune
2. computed-style copying onto the clone
3. image fetching / data-URL inlining
4. web-font embedding
5. SVG serialisation + `<img>` decode of the resulting data URL
6. canvas draw + WebP encode

modern-screenshot has a `debug: true` option that logs its own internal
phase timings (`log.time`/`timeEnd` around "wait until load", "clone
node", "embed web font", "embed node", "image to canvas", "canvas to
blob") — use it to get this quickly, then keep a permanent, cheaper
version of the breakdown in `window.__halleCaptureLog`.

**Report this table before changing anything.** Every lever below is
justified or discarded by these numbers.

### Phase 1 — the levers that cost nothing in fidelity

Apply these, measuring after each:

- **B1 above** (drop the double capture; reuse one context). Expected to
  be the single biggest win — roughly half.
- **Warm the caches when the widget opens, not on click.** A
  modern-screenshot context caches fetched image data URLs, font CSS and
  default computed styles. Today a brand-new context is built for every
  `domToBlob` call, so nothing is ever reused. Create the context once
  when the capture chunk loads and reuse it, so image and font fetching is
  already done by the time the tester clicks.
  **This is not pre-capture and must not become it** — no picture is
  taken early, nothing is snapshotted, the image is still rendered from
  the live page at click time. Only the asset cache is warmed. If this
  cannot be done without taking an early picture, stop and report rather
  than doing it anyway.
- **`features: { copyScrollbar: false }`.** With it on, every scrollable
  element costs seven extra `getComputedStyle` calls for the
  `::-webkit-scrollbar*` pseudo-elements, on top of `::before`/`::after`
  for every element. Scrollbars are worthless in a bug screenshot.
- **`font: { preferredFormat: 'woff2' }`.** Without it, every `@font-face`
  fetches each format it declares. This alone avoids most redundant font
  downloads while keeping the real fonts.
- **`filter`** out `<iframe>`, `<video>` and `<canvas>` subtrees.
  modern-screenshot clones an iframe's entire `contentDocument`; one
  embedded map or video player on a page is enormous. A grey placeholder
  box is fine in a bug report.

### Phase 2 — levers with a real but acceptable cost, if Phase 1 misses 500ms

- **`includeStyleProperties` with a curated list.** The library's own docs
  call this the option "for performance-critical scenarios". By default it
  copies every computed property (~340 strings) for every node; a curated
  list of what actually affects a screenshot (box model, colour,
  background, border, font, flex/grid, transform, opacity, overflow,
  position) is a fraction of that. On a node-heavy Webflow page this is
  potentially the largest remaining win. Verify the picture still looks
  right — this is the one with real regression risk, so compare before/
  after images.
- **`scale` below 1.** Fewer pixels to rasterise and encode; `scale: 0.5`
  is a quarter of the work. Cost: a softer picture. Check it is still
  clearly readable for a bug report before proposing it.
- **`workerUrl` / `workerNumber`.** Moves image fetching and encoding off
  the main thread. The widget already serves a separate `capture.js`
  chunk, so serving one more small file is precedented — but weigh the
  added complexity against the measured gain.
- **`fetchFn`.** A custom image retrieval function, so images already
  warmed in Phase 1 are served from memory rather than re-requested.
- **`font: false`** — the last resort on fonts. Screenshot text renders in
  fallback fonts. **Report the number this would save and ask Vishnu; do
  not apply it unilaterally.**

### Phase 3 — only if 500ms is still out of reach: evaluate snapDOM

`@zumer/snapdom` is a newer library in the same space, with published
benchmarks claiming roughly 4–5× faster than modern-screenshot on the
hardest synthetic case (364ms vs 1,686ms) and dramatically faster on
simpler ones. Those numbers are the author's own, on synthetic elements,
with no fidelity comparison — treat them as a reason to measure, not as
fact.

If you get here: **spike it, timeboxed, on the real Contact page**, and
report three things — capture time, a side-by-side fidelity comparison
against the current output, and whether the privacy-stripping and
viewport-scoping work in `capture.ts` port across cleanly.

**Do not swap the library on your own initiative.** `agent-rules.md`
allows exactly one dependency in `src/widget/`, and `modern-screenshot` is
currently it. Replacing that exception is Vishnu's decision, made on your
numbers.

### If 500ms still cannot be met

Come back with the phase table and the best achieved number rather than
quietly shipping something slower. A measured "we got to 900ms and here is
where the rest goes" is a useful answer; silently missing the budget is
not.

---

## Explicitly NOT in scope

- **Do not** make the capture start earlier (i.e. when the tester picks
  "Point at the problem" rather than after they click the element).
  Vishnu considered this and declined it on 10 September — the picture
  keeps being taken after the click, as `widget-v2-spec.md` says.
- **Do not** touch `WEBP_QUALITY` or `CAPTURE_SCALE`. Both are already
  conservative and lowering them degrades the bug report for very little.
- **Do not** start the wider UI redesign work. That is being done screen
  by screen with Vishnu, starting with the widget review screen, after
  this task lands.

---

## Report back

Per item: what was wrong, what changed, and how you know it works. For B1,
give the before/after timing on a real image-heavy page, not on the test
fixture — the fixture is what produced the wrong conclusion last time. For
B3, give the numbers and stop.
