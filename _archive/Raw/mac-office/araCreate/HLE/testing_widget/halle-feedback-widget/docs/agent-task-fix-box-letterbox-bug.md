# Agent task — URGENT, box-only: fix the letterbox scaling bug

22 September 2026. Live testing after the last deploy showed the pointer
box landing in essentially unrelated blank space — worse than before.
Confirmed root cause by reading the current code directly (`app.ts`,
around the `review-target-box` and `review-marker-canvas` blocks, and the
`send_btn` stroke-scaling code).

**Scope: THIS BUG ONLY. Nothing else.** Do not touch capture speed, do not
touch the "images are also worse" report (separate, unconfirmed, needs
Vishnu's examples first — not this task). Do not redeploy at the end of
this task — build and test locally, then stop and report, same as always.
The live server is currently rolled back / being rolled back separately;
this task is preparing the real fix to go out afterward, once verified.

## The bug

The full-size review picture uses `object-fit: contain` on the `<img>`
(`styles.css` `.review-image-full`), so when the picture's own shape
(width÷height, fixed by `capture_viewport` at capture time) doesn't
exactly match the shape of the screen area it's displayed in, the browser
adds blank space on two opposite sides to avoid stretching it — the `img`
element's own box (`clientWidth`/`clientHeight`) still equals the FULL
area including that blank space, not just the real visible picture inside
it.

Three places in `app.ts` currently treat `img.clientWidth` /
`img.clientHeight` as if they were the real picture's own displayed size,
with no offset — all three need the same fix:

1. The `review-target-box` positioning, inside `img.addEventListener('load', ...)`:
   ```
   const scale_x = img.clientWidth / cv.w;
   const scale_y = img.clientHeight / cv.h;
   box.style.left = `${fp.x * scale_x}px`;
   box.style.top = `${fp.y * scale_y}px`;
   ```
2. The marker canvas sizing, in the second `img.addEventListener('load', ...)`:
   ```
   canvas.style.width = `${img.clientWidth}px`;
   canvas.style.height = `${img.clientHeight}px`;
   ```
3. The stroke-scaling at send time, in `send_btn`'s click handler:
   ```
   const scale = canvas ? { x: cv.w / canvas.width, y: cv.h / canvas.height } : { x: 1, y: 1 };
   ```
   (`canvas.width`/`canvas.height` here are the devicePixelRatio-scaled
   backing store from A1, but they still derive from the same
   frame-sized `canvas.style.width/height` set in #2, so they carry the
   same error.)

## The fix

Write one small shared helper — something like
`rendered_image_rect(img: HTMLImageElement): { x: number; y: number; w: number; h: number }`
— that works out the ACTUAL visible picture rectangle inside an
`object-fit: contain` image element: compare `img.naturalWidth /
img.naturalHeight` against `img.clientWidth / img.clientHeight`; whichever
dimension is the constraining one, the picture fills that dimension
exactly and is centered with equal blank space on the other dimension.
Standard, well-known formula — do not need to invent anything novel here.

Then:

1. **Canvas: fix at the root, not just patch the symptom.** Size AND
   position the `review-marker-canvas` element itself to exactly match
   `rendered_image_rect(img)` — not the full frame. This means the canvas
   only ever covers the real picture, never the blank letterbox space, so
   a tester literally cannot draw in the letterbox area, and
   `marker-pen.ts`'s existing `point_from()` (already based on
   `canvas.getBoundingClientRect()`) keeps working correctly with no
   further change needed there. Re-check A1's devicePixelRatio backing
   store math still holds once the canvas's CSS size comes from the
   rendered rect instead of the frame.
2. **Box:** use the rendered rect's own `x`/`y` as a base offset and its
   `w`/`h` (not `img.clientWidth`/`clientHeight`) as the scale reference:
   ```
   box.style.left = `${rect.x + fp.x * (rect.w / cv.w)}px`;
   box.style.top = `${rect.y + fp.y * (rect.h / cv.h)}px`;
   box.style.width = `${fp.w * (rect.w / cv.w)}px`;
   box.style.height = `${fp.h * (rect.h / cv.h)}px`;
   ```
3. **Stroke scaling at send:** once the canvas is sized to the rendered
   rect (item 1), `canvas.width`/`canvas.height` already reflect the real
   picture's own dimensions, not the frame's — so `scale = { x: cv.w /
   canvas.width, y: cv.h / canvas.height }` becomes correct automatically,
   no separate change needed here beyond item 1 actually landing.

## Verification — this needs to be real this time, not just a unit test

Trust is low right now because the last round's fix looked verified but
broke worse in real use. Do all of these, not just the automated test:

1. A new test that deliberately creates a letterbox mismatch — e.g. a
   very wide captured viewport shown in a narrow/tall container, or vice
   versa — and asserts the box lands at the exact expected pixel position
   accounting for the letterbox offset, not just at the un-letterboxed
   case the last round's test may have implicitly assumed.
2. **Manually reproduce the exact live scenario Vishnu hit**: a desktop
   browser window, pick an element on a real page
   (`halle-dev.webflow.io/products/glan-thompson-polarizing-prisms` is the
   exact page from his report), and confirm the box now lands on the
   actual picked element, with a screenshot proving it.
3. Also manually check a case with NO letterboxing (container shape
   already matches the picture) to confirm this fix doesn't regress the
   normal case.
4. Do this on at least two different browser window sizes/shapes, since
   the letterbox direction (bars on the sides vs bars on top/bottom)
   flips depending on which dimension is wider.

## Report back

Show the before/after screenshots from the manual test (not just describe
them), confirm the new letterbox-aware test passes and fails correctly
against the old code, and confirm the no-letterbox case still works.
Flag anything uncertain rather than asserting confidence — this is exactly
where the last round went wrong.

---

## Test it yourself, locally — do NOT deploy in this task

Do all of this yourself, in this same task, using your own local
environment (`make demo` per `docs/local-test-plan.md` §0 — the dev app,
the widget build, and `tests/widget/host-page.html` served locally). Not
production. Do not deploy anything as part of this task — stop after
reporting, and deploying is a separate step Vishnu will ask for once he
has looked at your proof.

This is exactly the "Verification" section above — do it there, locally,
yourself: the letterbox-mismatch case, the real page from Vishnu's report,
and the normal no-letterbox case, at at least two different window shapes.
Take real screenshots of each as you go (not a description of what you
expect to see) and include them in your report.

Given last round looked verified and still broke live, the standard this
time is: do not report success unless you can show it, and say plainly if
any case looks uncertain rather than rounding up to "this works.
