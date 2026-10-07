# Agent task — screenshot speed: a hard budget of 500ms

**Scope: the screenshot only.** The marker-pen work and the wider UI pass
are written up separately and are deliberately **not** part of this task
— see "Explicitly not in scope". Do this one thing, well.

Background and evidence, read both first:

- `docs/plan-speed-marker-ui.md` §1 — the live trace numbers.
- `docs/research-how-the-industry-solved-this.md` — the industry survey
  that settled the architecture.

**Do not commit or push without asking Vishnu at the time**, as always.

---

## The requirement

**Vishnu's requirement, 10 September: the picture must appear within
500ms of the click, on the real page.** The reasoning is a product one
and it is correct — a tester who waits six seconds abandons the report.
Treat 500ms as the budget this work is measured against, not an
aspiration.

Live baseline on the real Contact page (`halle-dev.webflow.io/contact`):
clone built in 11ms, first capture pass finished at 5,846ms, final result
at 11,576ms, 29,052 bytes, 71 of 82 images kept.

For context, this is normal for the technique rather than a defect in our
code — monday.com's engineers measured the same family of libraries on
real content at 21s (html2canvas) and ~7s (modern-screenshot). The
technique is the problem, so the work is to make it much cheaper.

## Settled — do not re-open

- **Freeze-then-draw stays.** The picture is taken and then annotated. No
  shipping tool in the category lets a user draw freehand on a live
  scrolling page; an earlier proposal in this project to do that is
  **withdrawn**.
- **The picture is taken when the tester clicks**, not before.
  Pre-capturing is declined.
- **Viewport only. No full-page capture, ever, in the widget.** iOS caps
  canvases at 4,096px and exceeding it yields a **blank image with no
  error**. No embedded JS widget in the survey ships reliable full-page
  capture.
- **No native screen capture.** The `getDisplayMedia` permission can
  never be persisted (W3C Working Draft, 27 August 2026), the picker
  cannot be removed, and there is zero mobile support on any browser.
- **Server-side rendering is held in reserve.** It is what Marker.io,
  Ybug, Usersnap and Userback all do by default, and it is the answer if
  this task misses its budget — but it is not this task. Do not start it.
- A failed or slow capture must still never block a report
  (`agent-rules.md §1.11`).

So the budget has to be met by making the capture itself cheaper. Work
the phases in order.

---

## Phase 0 — instrument first. No optimisation before this.

The current trace has four coarse events, which is why nobody can say
where the 5.8 seconds goes. Break the capture into named phases and time
each, on the **real** Contact page — the local fixture is what produced
the wrong "second capture is cheap" conclusion, so do not tune against
it:

1. clone + prune
2. computed-style copying onto the clone
3. image fetching / data-URL inlining
4. web-font embedding
5. SVG serialisation + `<img>` decode of the resulting data URL
6. canvas draw + WebP encode

modern-screenshot's `debug: true` option logs its own internal phase
timings ("wait until load", "clone node", "embed web font", "embed node",
"image to canvas", "canvas to blob") — use it to get this quickly, then
keep a permanent, cheaper version of the breakdown in
`window.__halleCaptureLog`.

**Report this table before changing anything.** Every lever below is
justified or discarded by these numbers.

Research expectation to test, not to assume: the dominant cost is likely
**asset inlining** — each image fetched, converted to base64 (+33%
size), concatenated into one giant SVG string that must then be parsed
and decoded serially on the main thread. Style copying is expected to be
real but secondary.

## Phase 1 — the levers that cost nothing in fidelity

Apply these, measuring after each:

### 1a. Remove the wasted second capture — fix the cause, don't sniff

`capture_screenshot()` in `capture.ts` calls `capture_once()` **twice**
and keeps only the second result, on every browser, to work around
Safari's blank first render. The code comment justifies it as "a
redundant second capture is cheap" — measured at 98ms on the test
fixture, but **5.8 seconds on the real page**. Roughly half the tester's
wait is a second capture that only Safari was ever meant to need.

Safari's blank first render is a **readiness** problem, not a quirk
needing a throwaway render. Await `document.fonts.ready`, and
`await img.decode()` on the generated image, before rasterising — then
the first pass is already correct and the second is unnecessary on every
browser, Safari included.

Verify on Safari specifically before removing the second pass. If it
still comes back blank there, fall back to conditioning the second pass
on WebKit-not-Chrome, and say so in your report.

Update the comment above `capture_screenshot()` at the same time — it
currently states the "second capture is cheap" reasoning as fact, and
that is exactly the assumption that proved wrong on a real page. Leaving
it invites the same mistake again.

### 1b. Reuse one context instead of building a fresh one per call

Each `domToBlob()` call currently creates its own context, so nothing is
ever cached between captures — image data URLs, font CSS and default
computed styles are all recomputed every time. modern-screenshot's own
README recommends `createContext()` reuse (plus `workerUrl`/
`workerNumber`) explicitly for "quick screenshots per second".

Create the context once when the capture chunk loads and reuse it, so
image and font fetching is already done by the time the tester clicks.

**This is not pre-capture and must not become it** — no picture is taken
early, nothing is snapshotted, the image is still rendered from the live
page at click time. Only the asset cache is warmed. If that cannot be
done without taking an early picture, stop and report rather than doing
it anyway.

### 1c. Cheap option changes

- **`features: { copyScrollbar: false }`.** With it on, every scrollable
  element costs seven extra `getComputedStyle` calls for the
  `::-webkit-scrollbar*` pseudo-elements, on top of `::before`/`::after`
  for every element. Scrollbars are worthless in a bug screenshot.
- **`font: { preferredFormat: 'woff2' }`.** Without it, every
  `@font-face` fetches each format it declares. Avoids most redundant
  font downloads while keeping the real fonts.
- **`filter`** out `<iframe>`, `<video>` and `<canvas>` subtrees.
  modern-screenshot clones an iframe's entire `contentDocument`; one
  embedded map or video player is enormous. A grey placeholder box is
  fine in a bug report.

## Phase 2 — real but acceptable costs, if Phase 1 misses 500ms

- **`includeStyleProperties` with a curated list.** The library's docs
  call this the option "for performance-critical scenarios". By default
  it copies every computed property (~340 strings) per node; a curated
  list of what actually affects a screenshot (box model, colour,
  background, border, font, flex/grid, transform, opacity, overflow,
  position) is a fraction of that. Potentially the largest remaining win
  on a node-heavy Webflow page. It also carries the most regression risk
  — compare before/after images.
- **`scale` below 1.** Fewer pixels to rasterise and encode; `scale: 0.5`
  is a quarter of the work. Cost: a softer picture. Confirm it is still
  clearly readable for a bug report before proposing it.
- **`workerUrl` / `workerNumber`.** Moves image fetching and encoding off
  the main thread. The widget already serves a separate `capture.js`
  chunk, so serving one more small file is precedented — weigh the added
  complexity against the measured gain.
- **`fetchFn`.** A custom image retrieval function, so images already
  warmed in Phase 1b are served from memory rather than re-requested.
- **`font: false`** — the last resort on fonts. Screenshot text renders
  in fallback fonts. **Report the saving and ask Vishnu; do not apply it
  unilaterally.**

## Phase 3 — only if 500ms is still out of reach: measure snapDOM

`@zumer/snapdom` is a newer library whose published benchmarks claim
roughly 4–5× faster than modern-screenshot. Two things to hold in mind:

- It is **the same technique**, not a new one — its own docs describe the
  same clone → inline → `foreignObject` → decode pipeline. The gains are
  engineering (deduplicated CSS classes rather than per-node inline
  style, WeakMap caches, MutationObserver cache invalidation).
- The benchmarks are the author's own, on synthetic elements. Research
  found **no independent real-page benchmark anywhere**. Treat the
  numbers as a reason to measure, not as fact.

If you get here: **spike it, timeboxed, on the real Contact page**, and
report capture time, a side-by-side fidelity comparison against current
output, and whether the privacy-stripping and viewport-scoping in
`capture.ts` port across cleanly. Note its cache default is `soft`
(clears every capture) — `cache: 'full'` plus `preCache()` is reportedly
its single biggest lever, so measure it configured properly.

**Do not swap the library on your own initiative.** `agent-rules.md`
allows exactly one dependency in `src/widget/`, and `modern-screenshot`
is currently it. Replacing that exception is Vishnu's decision, on your
numbers.

---

## One user-facing change that belongs with this work

`CAPTURE_FIRST_PAINT_MS = 3000` in `app.ts` opens the comment box after 3
seconds whether or not the picture is ready. **That behaviour is correct
and must stay** — do not change the timing, and do not turn the
first-paint deadline back into a give-up. The comments in that file
explaining why are right.

The problem is what is *shown*: the picture-less state renders the
literal text **"(no picture)"**, which reads as "this is broken", for up
to nine more seconds before the picture quietly appears.

Fix: while a capture is still in flight, show a "taking a picture of the
page…" state. Fall back to the existing "(no picture)" wording only once
the capture has actually failed or its budget is spent — the two states
must be distinguishable.

The new string must be **editable in admin like every other string**.
Note the trap found in the last round: the Wording screen has an explicit
field list, and a string missing from it reaches testers with no way to
change it. There is now a guard for this — make sure the new string
satisfies it.

---

## Explicitly not in scope

- **The marker pen.** Eight separate defects are written up and waiting
  (half-resolution canvas, preview-vs-sent thickness mismatch, no
  smoothing, redraw lag, lost samples on fast strokes, taps drawing
  nothing, arming the pen behind an icon, and moving to a full-size
  drawing surface). That is its own task, coming after this one. **Do not
  touch `marker-pen.ts` or the review screen's layout here.**
- **The wider UI pass.** Sixteen screens are being reviewed one at a time
  with Vishnu. Not now.
- **Server-side rendering.** Held in reserve, see above.
- **`WEBP_QUALITY` and `CAPTURE_SCALE`** — leave both alone except where
  Phase 2 explicitly proposes a `scale` change with a measurement behind
  it.
- Do not make the capture start earlier than the click.

## Report back

- The Phase 0 table first, before any change.
- Then per lever: what it saved, measured on the real page, not the
  fixture.
- The final number against the 500ms budget.
- If 500ms cannot be met, say so with the table and the best achieved
  number. A measured "we reached 900ms and here is where the rest goes"
  is a useful answer; silently missing the budget is not.
