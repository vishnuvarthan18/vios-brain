# Report — screenshot speed against the 500ms budget

10 September 2026. Answers `docs/agent-task-screenshot-speed-500ms.md`.

All numbers below are measured on the **real** Contact page
(`halle-dev.webflow.io/contact`), Chromium, 1440×900, `deviceScaleFactor: 1`,
a fresh browser context per run, medians as stated. The local fixture was
not used for any timing claim — it is what produced the wrong "second
capture is cheap" conclusion in the first place.

**Headline: the budget is met, but only with one setting that is Vishnu's to
approve, not mine.**

| Configuration | Median capture | Budget |
| --- | --- | --- |
| Before this work (live trace) | 11,576ms | ✗ |
| Before this work (re-measured) | 12,398ms | ✗ |
| After this work, web fonts embedded (**as committed**) | **1,580ms** | ✗ |
| After this work, web fonts not embedded (**one flag away**) | **152ms** | ✓ 3.3× inside |

---

## Phase 0 — the table, before any change

`domToBlob` with `debug: true`, both passes of today's shipped double
capture, median of 3.

| Phase | Pass 1 | Pass 2 | Both |
| --- | --- | --- | --- |
| clone + prune (our own code) | 6ms | — | **6ms** |
| `wait until load` | 5,001ms | 5,003ms | **10,004ms** |
| `clone node` (computed-style copy) | 233ms | 233ms | 466ms |
| `embed web font` | 5ms | 2ms | 7ms |
| `embed node` (asset inlining) | 1,074ms | 91ms | 1,165ms |
| `image to canvas` | 344ms | 366ms | 710ms |
| `canvas to blob` (WebP encode) | 49ms | 47ms | 96ms |
| **Total** | **5,846ms** | **5,730ms** | **12,398ms** |

Output: 31,048 bytes, 71 of 82 images kept in the clone, 497 nodes.

**The research expectation was wrong, and the trace says so plainly.** The
dominant cost was not asset inlining. It was `wait until load`, at 5,001ms
per pass — pinned to within 4ms of `ASSET_TIMEOUT_MS`, which is the
signature of a timeout expiring rather than work being done. 81% of the
capture was the library waiting for images that were never going to arrive.

Two of the task's own assumptions also turned out to be wrong, and both are
recorded below rather than quietly dropped.

### Why it was waiting — the actual root cause

Every one of the page's 82 images carries `loading="lazy"` (Webflow's
default). Our capture clone is deliberately parked at `left:-999999px` so it
never flashes in front of the tester. That places every cloned image about a
million pixels outside the viewport, so the browser does exactly what lazy
loading asks and **never requests them at all** — they therefore never fire
`load` *or* `error`. `waitUntilLoad()` awaits every `<img>`, and the only
thing that can resolve an image which will never load is the per-asset
timeout.

Measured directly: of the 71 images in the clone, **61 never settled**; 10
did. Forcing `loading="eager"` took the wait from 6,003ms to **72ms**, and
adding `decoding="sync"` took it to **2ms**.

This was our bug, in our own clone-building code. The library was never slow
here.

### The second finding — 58 of 71 images could never paint

Of the 71 images surviving the off-screen prune, only **13 have a real
rendered box**. The other 58 report a 0×0 rect, and **55 of those sit inside
a `display:none` Webflow CMS collection list** (`.w-dyn-list`), with
`offsetParent: null` and `naturalWidth: 0`. Each was being fetched,
base64-encoded (+33%), concatenated into the SVG string and parsed back out
— to draw nothing. This is `plan-speed-marker-ui.md` §1.5, and it was worth
more than expected.

`offscreen_nodes()` keeps zero-sized elements on purpose, and that is right
for a layout-less *wrapper* whose children may be visible. It is wrong for
an `<img>`, which renders only itself.

---

## Per lever, measured

Each measured on the real page, cumulative in the order applied.

| Change | Effect | Note |
| --- | --- | --- |
| **1a** force `loading="eager"`/`decoding="sync"` on the clone | 12,398 → 3,521ms | the root cause; `wait until load` 5,001ms → ~7ms |
| **1a** drop the second capture pass | 3,521 → 2,389ms | see below — no Safari workaround needed |
| **prune non-rendering `<img>`** | 2,389 → ~1,316ms | 71 images → 13 |
| **1c** `features: { copyScrollbar: false }` | `image to canvas` 357 → ~160ms | free |
| **1c** `filter` out iframe/video/canvas | no change on this page | it has none; kept as insurance |
| **2** `includeStyleProperties` (123 curated) | `clone node` 192 → ~48ms | fidelity verified identical |
| **Committed total (fonts on)** | **1,580ms** | 7.3× faster than the 11,576ms live baseline |
| **`font: false`** | **152ms** | +10× again — **needs your decision** |

### 1a — the second pass is gone, and did not need replacing

Removed. Instead of throwing away a whole render to paper over Safari's
documented blank first capture, the two things whose absence causes it are
now awaited directly: `document.fonts.ready` before rendering (raced against
a 400ms ceiling, so a host page with a permanently pending font cannot stall
every capture), and `img.decode()`, which modern-screenshot already awaits
internally in its image-to-canvas path.

**Caveat, stated plainly: I could not verify this on Safari.** Only Chromium
is installed in this environment, and no WebKit build was available. The
fallback the task specifies — conditioning a second pass on
WebKit-not-Chrome — is *not* in place; there is one pass on every browser. If
Safari does come back blank, that fallback is the fix, and the readiness
work above stays either way. Worth ten minutes on a real Mac before this
ships to testers.

The comment above `capture_screenshot()` that asserted "a redundant second
capture is cheap" is rewritten, and now records why that sentence was wrong
so the same mistake is not repeated from the same reasoning.

### 1b — context reuse: measured, and NOT adopted

`createContext()` reuse was built and measured. On this page it does not pay,
because the thing it caches is not what costs: image fetches are already
served from the browser's HTTP cache on a second capture (612ms cold → 0ms
warm, with no context reuse at all), and the fonts — the real cost — are
re-fetched regardless. Measured click-time capture: cold 1,621ms, warmed HTTP
cache 1,436ms, warmed data-URL cache via `fetchFn` 1,478ms. That is inside
run-to-run noise on a cost that is dominated by fonts.

It also carries a real risk for no gain: a reused context holds `node`,
`width` and `height` from the clone it was built against, so every capture
must re-point them, and a missed field is a silently wrong picture. Not
adopted. If fonts are ever turned off, the remaining 152ms has nothing left
in it worth caching.

### 1c — `font: { preferredFormat: 'woff2' }` was a trap, and is not shipped

The task lists this as a free win. **It is not free on this page — it is
`font: false` in disguise.** The library's `filterPreferredFormat()` rewrites
each `@font-face` `src` and returns an **empty string** when none of the
declared formats matches. The page declares **15 `@font-face` rules, every
one `.otf`, and no woff2 anywhere**. So the option silently stripped the
`src` from all 15 faces.

It measured as a 4× speed-up that looked free. It was really the fidelity
trade-off below, taken without anyone deciding to take it. Removed, with the
reasoning recorded in the code so it is not "optimised" back in.

### 2 — `scale` below 1 is not proposed

`scale: 0.5` saved little once the real problems were fixed (the encode is
already only ~48ms) and costs a softer picture. Not proposed. `WEBP_QUALITY`
and `CAPTURE_SCALE` are untouched, as instructed.

### 2 — `workerUrl` / `workerNumber` is not worth it

Measured at roughly −137ms, and it cannot help structurally: `embedNode`
pushes **already-started** promises onto its task queue, so the "4 concurrent
runners" only await work whose fetches all launched synchronously anyway.
Concurrency is already unbounded; workers do not throttle or parallelise what
is already in flight. Not adopted, for a saving that does not justify serving
another chunk.

### 3 — snapDOM was not needed

The budget is reachable without changing library, so the timeboxed spike was
not spent. `modern-screenshot` stays, and `agent-rules.md` §1.4's single
dependency exception is untouched.

---

## The one decision I did not take: web fonts

**This is the whole remaining distance to the budget, and it is a fidelity
call, so it is yours.** `EMBED_WEB_FONTS` in `capture.ts` is the flag, set to
`true` (faithful) with the full reasoning beside it.

| | Median capture | Picture |
| --- | --- | --- |
| `EMBED_WEB_FONTS = true` (committed) | 1,580ms | always matches the page |
| `EMBED_WEB_FONTS = false` | **152ms** | identical on any machine that has the font locally |

Why it costs so much: the page declares ~1.97MB of `.otf` faces, and the cost
is billed to the library's `embed node` phase rather than `embed web font` —
`embedWebFont` only *pushes* its fetch promises onto the shared task queue
that `embed node` later awaits, which is why the original trace showed fonts
at 5ms and hid 1.7 seconds inside another line.

Why turning it off may be free: the picture is rasterised by **the tester's
own browser**, which has the host page open, so any font that browser can
already resolve renders correctly whether or not it was inlined. Verified
through the real shipped module: identical 31,838-byte output and an
identical pixel checksum with fonts on and off.

Why it is nevertheless your call: that verification was on a Mac, which has
Helvetica Neue installed locally. **A Windows tester without that family
would get the page's fallback (Arial) in the picture** instead of the face
they were looking at. For a layout or content bug that changes nothing; for a
bug about type it would mislead.

Worth knowing when you decide: the captured viewport uses **exactly one**
font family, out of the 15 faces being downloaded. Almost the entire 1.4s is
fonts that never appear in the picture.

If you want the budget met, say the word and it is a one-line change.

---

## The user-facing change

`CAPTURE_FIRST_PAINT_MS = 3000` is untouched, as instructed, and is still a
first-paint deadline rather than a give-up.

What changed is what the picture-less screen *says*. There are now two
distinguishable states, and neither is hardcoded:

- **`capturePending`** — "Taking a picture of the page…", shown while the
  capture is still in flight.
- **`captureNone`** — "We could not take a picture, but you can still send
  this.", shown only once the capture has actually settled with no picture.

The literal `(no picture)` is gone; it was both misleading and the last
hardcoded tester-facing string in the review screen (`agent-rules.md` §1.8).

Both strings are editable in admin and satisfy the guard: they are in
`DEFAULT_STRINGS`, the API schema, the widget's `Strings` type, and the
Wording screen's `STRING_FIELDS` list. `tests/web/string-editor-coverage.test.ts`
passes, in both directions.

The pending state pulses gently, and the animation is suppressed entirely
under `prefers-reduced-motion` (`agent-rules.md` §2.5) — the wording carries
the meaning, so nothing is lost when motion is off.

---

## Tests

- **Two new regression tests** for the five-second bug, on a new fixture
  (`host-page-lazy-images.html`) that reproduces both real-page conditions —
  `loading="lazy"` images and images inside a `display:none` collection list.
  They assert behaviour, not wall-clock timing, so they will not flake on CI.
  **Both were confirmed to fail against the old `capture.ts` and pass against
  the fix.**
- The existing capture specs now assert the correct one of the two
  picture-less states per scenario, rather than a single shared string.
- Full widget acceptance suite: **51 passed**. Lint and type-check clean in
  both workspaces. Size budget: v1.js 8,508/15,360 gzipped, capture.js
  11,726/30,720 gzipped.
- The 9 failing vitest files are pre-existing `DATABASE_URL`-less DB tests
  plus the Playwright specs vitest cannot run; verified identical on a clean
  checkout (which actually fails one test that this branch fixes).

## Two things worth flagging beyond the budget

1. **A scrolled capture used to produce nothing at all.** At scroll offsets
   on the real page the old code exhausted its whole 12s budget and returned
   `null` — the tester got no picture, every time, not merely a slow one.
   It now returns a correct picture in ~800ms–2.1s. This was not in the task;
   it fell out of the same fixes.
2. **Not verified on Safari or on a machine without Helvetica Neue.** The two
   caveats above, repeated here because they are the only places this work is
   unproven.
