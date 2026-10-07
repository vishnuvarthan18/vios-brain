# Agent task — capture succeeds internally but UI still shows "(no picture)"

## What we found

Live-tested again on `halle-dev.webflow.io` (Contact page this time) after
the previous screenshot-capture fix was deployed. The widget still shows
"(no picture)" in the report box — but this time we checked the internal
debug trace (`window.__halleCaptureLog`, added by the previous fix) instead
of guessing, and it tells a different story than expected.

The log for this attempt:

```json
[
  { "at": 3903, "event": "start", "detail": { "viewport": "1864x999", "scroll": "0,0" } },
  { "at": 3912, "event": "clone built", "detail": { "images_in_clone": 71, "images_on_page": 82, "elapsed_ms": 11 } },
  { "at": 9748, "event": "first pass done", "detail": { "elapsed_ms": 5846 } },
  { "at": 15478, "event": "captured", "detail": { "bytes": 29052, "elapsed_ms": 11576 } }
]
```

Read literally: the capture pipeline (`capture_screenshot` in
`src/widget/src/capture.ts`, built into `dist/capture.js`) actually
**succeeded** — it produced a real 29,052-byte WebP image in 11.576
seconds, which is *inside* its own 12-second budget (`zt = 12e3` in the
built bundle). The viewport-scoping fix is also confirmed working: only
71 of 82 images on the page were pulled into the clone, not everything
on the page like before.

So: the capture library returned a real image successfully. The widget's
UI still rendered "(no picture)" anyway. **The bug is not in
`capture.ts`/`capture_screenshot` — it's in whatever code calls
`capture_screenshot()` and is supposed to hand the resolved image to the
review screen** (likely in `src/widget/src/app.ts`, based on the
capture.js bundle's exports: `capture_screenshot` and `burn_in_markup`
are both called from there).

## What to check

1. Find where `app.ts` (or wherever the widget's flow logic lives) calls
   `capture_screenshot()` and awaits/handles its result. Look for:
   - A separate timeout in the *caller*, shorter than the 12s one inside
     `capture_screenshot` itself, that gives up and shows "(no picture)"
     before the capture promise actually resolves.
   - A race condition: the UI might decide to render the review screen
     (with no image) based on some other signal (e.g. reaching the
     "point at the problem" step) before `capture_screenshot()`'s promise
     settles, and then ignores the late-arriving result even though it's
     a real, valid image.
   - Whether the resolved blob is actually being stored/passed into
     whatever state the review screen reads from — it's possible the
     value is captured correctly but written to the wrong place, or
     overwritten by something that runs afterward.
2. This makes the whole flow noticeably slow for testers (~11-12 seconds
   of waiting) even when it's "working." Once the hand-off bug above is
   fixed, it's worth separately considering whether the capture needs to
   be faster (fewer images, lower quality, something else) so testers on
   a normal page aren't waiting that long. Flag this as a secondary
   finding, not blocking — get the hand-off bug fixed first.
3. Reproduce using the same debug trace method: open a real page with
   many images (not just the test fixture), trigger a capture, and read
   `window.__halleCaptureLog` in the console afterward to confirm capture
   succeeded even when the UI shows no picture. Add a regression test
   that would have caught this — likely something that asserts the
   resolved image from `capture_screenshot` actually reaches whatever
   render/state layer the review screen reads.

## Do not do

- Do not touch `capture.ts`'s own internal timeout/budget logic — it's
  working correctly and finishing within its own limit. The bug is
  downstream of it.
- Do not commit or push without asking first, as usual.
