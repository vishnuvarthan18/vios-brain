# Report — server-side screenshot, compared against today's method

10 September 2026. Answers `docs/agent-task-server-side-capture-ab-test.md`.

> **DECIDED, 10 September 2026 — Vishnu: keep today's method.** This report's
> recommendation was accepted. The re-test under real conditions
> (`docs/report-server-capture-real-conditions-test.md`) was stopped without
> completing, because rendering on the production box needs 19 system
> packages that are not to be installed there. Route 2 is not adopted.

All numbers below are measured on the **real** Contact page
(`halle-dev.webflow.io/contact`), Chromium, 1440×900, `deviceScaleFactor: 1`.
Both methods run **from the same click at the same scroll position, with no
reload between them**, and today's method runs first on every pass so it is
never the one penalised by a warm cache. Medians of 5 passes per click,
3 clicks, so 15 captures per method.

**Side by side: open `docs/ab-capture/index.html`.**

**Headline: today's method is 2.4× faster. The server's picture is very nearly
the same picture — with one visible defect that today's method does not have,
and one 8px offset that would break the marker pen.**

| Click (scroll) | Today's method | Server method | Server's own drawing | Pixels differing |
| --- | --- | --- | --- | --- |
| top-of-page (0) | **393ms** | 999ms | 994ms | 7.71% |
| form-and-address (400) | **409ms** | 1,033ms | 1,027ms | 4.85% |
| page-bottom (1,131) | **527ms** | 1,064ms | 1,059ms | 0.61% |

Server failures: **0 of 15**. Client failures: **0 of 15**.

---

## Which one is faster

**Today's method, by 2.4×** — ~390–530ms against ~1,000–1,065ms.

The server method's cost is almost entirely the drawing itself: of ~1,030ms
total, **~1,027ms is the server rendering** and only **4ms is the browser
serialising**. Posting the payload and getting the image back is the
remaining handful of milliseconds, on loopback.

That 4ms is the genuinely interesting number, and it is the one thing the
research promised that did hold: describing the page costs the tester's
browser almost nothing. Route 1 spends ~400ms in the tester's browser;
Route 2 spends 4ms there and ~1,027ms somewhere else.

Two caveats on the timing, both of which make the server look *better* than
it would be in production:

- **The renderer was on the same machine as the browser.** A real tester
  posts ~410KB over the internet to the server in Germany. That is latency
  and upload time this measurement does not include.
- **The renderer was warm and completely idle** — one browser already
  launched, nothing else competing. On the real shared box, with the app
  and other projects running, it will not be idle.

So ~1,030ms is a floor for the server method, not an estimate.

## Which one looks more correct

**Today's method.** Not because the server's picture is bad — it is
remarkably close — but because the server's has two defects and today's has
neither.

The closeness is worth stating precisely, because the raw
"pixels differing" column overstates the difference. Correcting for a
uniform 8px offset (below), the mean absolute per-pixel difference between
the two renders falls from **19.03 to 0.27** on the top-of-page click. Once
aligned, they are essentially the same image. Fonts, text, colour, layout,
product photos, form fields, icons and buttons all render correctly on the
server, and the privacy masking is intact in both.

Reproduce that alignment measurement with
`node scripts/ab/measure-alignment.mjs`.

### Defect 1 — a stray "Search" label the tester never saw

On every server picture there is a "Search" label and a faint box under the
header's search field. It is not on the live page and not in today's
picture.

It is **not** a lost `display:none` — that is preserved correctly (checked
directly: `.search-overlay` and `.nav-dropdown-list` both compute
`display:none` in the renderer's document, exactly as on the live page).

The cause is a **layout width difference**. The header's `.search-container`
is **218px wide on the live page and 297px in the renderer** — same
`display:flex`, same `visibility`, same `opacity`, just wider. Its child
carries the text "Product Search Results", which the live page's narrower
box clips and the wider one reveals.

Why wider: **the live page's own JavaScript sizes that container after
load.** The renderer deliberately runs no page JavaScript — it renders a
static serialised snapshot — so any element whose final geometry is set by
script keeps its CSS-only width.

This is structural to the server-rendering approach, not a bug in the
renderer, and it is the most important finding in this report: **an element
positioned or sized by JavaScript can render at the wrong size in the
server's picture.** On this page it produces one stray label. On a page with
a JS-driven carousel, tab strip or sticky element, it could be worse.

### Defect 2 — the whole picture sits 8px out

The server's picture is offset by a uniform **dx=−8, dy=−8** against
today's. As established above, this is not a fidelity problem — corrected
for, the two images almost coincide.

But **it would break the marker pen.** Strokes are drawn in viewport
coordinates and burned into the picture, so an 8px whole-image shift puts
every stroke 8px off the thing the tester circled.

The cause: the client serialises the *inner* clone (a copy of `<body>`), not
the fixed-position wrapper `build_capture_clone()` parks it in, so on the
server the clone inherits whatever box that document's `<body>` gives it.

**I tried fixing it and reverted the fix**, because it made things worse:
wrapping the clone in a `position:fixed`, viewport-sized, `overflow:hidden`
div took divergence from 7.71%/4.85%/0.61% to **7.7%/37.24%/45.84%**, and
best-alignment mean difference from 0.27 to **101.23**. The reason is that
the clone carries the scroll offset as its own negative
`margin-left`/`margin-top`, and a `position:fixed` parent re-anchors that
against the viewport instead of the document flow — so the scrolled clicks
lost their offset entirely. The client wrapper gets away with it only
because it also sits at `left:-999999px` in an already-scrolled document.

Fixing it properly means serialising the wrapper and its geometry along with
the clone. That is a change to the client payload, which is more than a
comparison harness should quietly introduce, so it is left measured and
recorded for your decision.

## Memory, on the real server's terms

The ground rule was no server upgrade, and the app is already capped at
`MemoryMax=1G` (`deploy/halle-feedback.service`).

Measured while rendering: **renderer process ~95MB RSS, Chromium ~320MB RSS,
~415MB combined.**

**The renderer therefore runs as its own process on its own port, not as a
Next.js route** — deliberately. Inside the app's cgroup, 415MB of Chromium
would be charged against the app's 768M `MemoryHigh`/1G `MemoryMax`, and
systemd would OOM-kill **the app**. As a separate process, if the renderer
dies the app and every real report path are untouched. That is the whole
reason for the process split and it should not be "simplified" away.

Nothing was upgraded, and the current server was not modified.

## What was built

| Piece | Where |
| --- | --- |
| Client serialiser (`serialise_page`) | `src/widget/src/capture.ts` |
| Server renderer, one at a time | `src/render/renderer.mjs` |
| A/B harness | `scripts/ab/capture-ab.mjs` |
| Side-by-side viewer builder | `scripts/ab/build-viewer.mjs` |
| Alignment probe | `scripts/ab/measure-alignment.mjs` |
| Privacy check | `scripts/ab/verify-privacy.mjs` |
| Pictures, timings, viewer | `docs/ab-capture/` |
| `make ab-render`, `make ab-capture` | `Makefile` |

### The ground rules, each one honoured

- **No server upgrade.** Nothing installed, nothing changed on the box. The
  renderer uses the Playwright Chromium already present as a root
  devDependency — no new dependency, and none added to `src/widget/`
  (`agent-rules.md §1.4`).
- **One screenshot at a time.** No queue, no worker pool. A second
  concurrent request is **refused with 503** rather than buffered —
  verified: two concurrent requests returned exactly `503, 200`. Refusing is
  the honest simple behaviour and keeps the constraint visible.
- **Today's method is untouched.** `capture_screenshot()` is unchanged and
  remains the only source of a report's picture. `serialise_page` is
  referenced nowhere outside `capture.ts` — nothing on the tester's path
  calls it. **All 51 widget acceptance tests pass**, and `tsc --noEmit` is
  clean. `v1.js` is unchanged at 8,508 bytes gzipped (55.4% of budget); the
  new code lives in the lazy `capture.js` chunk (11,935 bytes gzipped, 38.9%
  of budget).
- **A server failure blocks nothing** (`§4`, `agent-rules.md §1.11`). The
  renderer returns 500 and logs it; nothing on the tester's path awaits it.
  Verified: a malformed payload and a garbage body both return a clean
  500/JSON error without crashing the renderer.
- **Privacy masking runs first** (`§5`). `serialise_page()` builds its
  payload from `build_capture_clone()` — the *same* masked clone Route 1
  rasterises — so `strip_clone()` has blanked every input, textarea,
  contenteditable and `[data-fb-block]` before a byte is read. Verified
  end-to-end with real typed secrets: `node scripts/ab/verify-privacy.mjs`
  types `(secret removed)` into every input and
  `secret-textarea-98765` into every textarea on the real Contact page, and
  neither string appears anywhere in the resulting 417KB payload — while the
  live page keeps its values (the real DOM is never mutated).
- **Marker pen untouched.** Not modified; it still draws on whichever
  picture the report actually uses.

## One measurement mistake worth recording

The first version of the pixel diff reported **"78.22% of pixels differ"**
at the page-bottom click, with over a million pixels differing by 81+ on a
0–255 scale. That number was wrong, and the way it was wrong is a trap worth
writing down.

Route 1's WebP **carries an alpha channel** and Route 2's does not
(`modern-screenshot` rasterises only what the captured subtree paints and
leaves the rest transparent; Chromium's own screenshot paints the page's
white base layer). At page-bottom, **70.2% of Route 1's pixels are fully
transparent**. A transparent pixel reads as RGBA `0,0,0,0` from
`getImageData` — so a naive RGB comparison sees **black** where the picture
actually displays **white**.

Composited over white — which is what the tester actually sees, on a white
review screen — the same two images have **identical** mean RGB
(251,251,252) and differ by **0.61%**. The diff now paints a white base layer
before comparing, and `capture-ab.mjs` carries the explanation so it is not
re-broken.

Two smaller corrections in the same spirit: the first click set used a
2,200px scroll on a page that is only 2,031px tall, so both methods rendered
the same near-empty band and the comparison said nothing; and `sips`
mis-decodes these alpha WebPs, so every pixel figure here comes from a
browser decode instead.

## Recommendation

**Keep today's method as the default. Do not adopt the server method on this
evidence.**

- It is **2.4× slower** (~1,030ms vs ~410ms), and that is its best case —
  measured on loopback against a warm, idle renderer, with no internet
  upload of the ~410KB payload and no competition from the app.
- Today's method **already meets the fidelity bar** the server method was
  meant to fix. The hoped-for win was font accuracy, and the server does
  render fonts correctly by fetching them from the public site — but so does
  today's method as committed, at 1,580ms with fonts embedded, and at
  ~410ms in this measurement.
- The server method has **two defects today's does not**: the stray "Search"
  label from JavaScript-set geometry, and the 8px offset that would
  misplace every marker-pen stroke.
- It costs **~415MB of Chromium** next to an app already capped at 1GB on a
  shared box, plus a second process to run and keep alive.

The one result that would justify revisiting this: **4ms of browser time.**
If the tester's device ever becomes the constraint — a low-end phone where
400ms of main-thread rasterising is really 4 seconds — then moving the work
off the device is the right shape of answer, and this harness is here to
measure it again. The blockers to fix first would be the 8px offset (fix the
payload, not the renderer) and the JavaScript-geometry defect (which may not
be fully fixable without running page JS).

**Not my call to make** (`§ Explicitly not in scope`): the numbers and both
sets of pictures are above and in `docs/ab-capture/index.html`.

## Reproducing

```
make ab-render     # terminal 1 — the renderer
make ab-capture    # terminal 2 — both methods, same clicks, builds the viewer
open docs/ab-capture/index.html
```

Also: `node scripts/ab/measure-alignment.mjs` (the 8px offset),
`node scripts/ab/verify-privacy.mjs` (masking, with real typed secrets).
