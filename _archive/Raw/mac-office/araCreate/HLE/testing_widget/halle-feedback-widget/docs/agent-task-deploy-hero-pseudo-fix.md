# Deploy task — the hero-blank capture fix

**For the VS Code dev agent. One task, nothing else. Do not start any other
work from this file.**

## What is already done — do not redo it

`src/widget/src/capture.ts` already contains the fix, uncommitted in the
working tree (+156 lines, one new function `inline_pseudo_backgrounds()` plus
one call site inside `capture_screenshot`). It type-checks clean with the
repo's own config (`./node_modules/.bin/tsc --noEmit -p src/widget/tsconfig.json`).

Root cause, so you understand what you are shipping: the hero's navy
background on desktop is painted by a CSS `::before` pseudo-element whose
background is an external Webflow CDN image. `modern-screenshot` inlines
images reachable from a real element (`<img src>`, a node's own inline
`background-image`) but never a pseudo-element's, so that URL stayed an
`https://` link inside the generated SVG — and a browser rasterising an SVG as
an image is forbidden from fetching anything external. The navy never painted,
the section fell back to white, and the hero's white heading text became
invisible on it. One cause, all three symptoms.

Measured on a complete offline copy of the real Home page, 1440x900, same
capture options: real browser 722,668 navy pixels; capture before the fix
17,507 (hero pixel `rgb(255,255,255)`); capture after the fix 725,435 (hero
pixel `rgb(40,48,139)`). Difference from the browser's own render fell from
58.4% to 7.2%, the remainder being the web fonts `EMBED_WEB_FONTS = false`
deliberately leaves out. Capture time unchanged: 934ms end-to-end, of which
the new step is ~74ms including the fetch.

## Do not change

- `EMBED_WEB_FONTS = false`
- `font: false` in the `domToBlob` options
- `CAPTURE_STYLE_PROPERTIES` (the trimmed allow-list)
- the `||` in `offscreen_nodes()`'s empty-rect test
- the forced `loading="eager"` / `decoding="sync"` on clone images
- anything in the admin app or the shadcn migrations

## Step 1 — check the change is the only change

```bash
cd <repo root>
git status --short
git diff --stat src/widget/src/capture.ts
```

Expect exactly one modified file: `src/widget/src/capture.ts`, 156 insertions,
0 deletions. Untracked `docs/*.md` files are fine and are not part of this task.

## Step 2 — type-check and build locally

```bash
./node_modules/.bin/tsc --noEmit -p src/widget/tsconfig.json
WIDGET_API_ORIGIN=https://feedback.arametrics.app \
  npm run build --workspace halle-feedback-widget-embed
```

Expect `tsc` silent, and the build to print `Built dist/v1.js and dist/capture.js`.

Then check the size budget still passes:

```bash
make size
```

The fix lives in `capture.ts`, which is its own separate bundle
(`dist/capture.js`) and is **not** counted against `v1.js`'s 15,360-byte
gzipped budget — so `v1.js` should be unchanged in size. If `v1.js` grew,
something was imported into the wrong bundle: stop and say so.

## Step 3 — run the widget tests

```bash
npm run test:widget
```

If a test fails, stop and report which one. Do not edit a test to make it pass.

## Step 4 — commit

One commit, this concern only, staged by path. Follow `agent-rules.md` §4 —
**no `Co-Authored-By` trailer.**

```bash
git add src/widget/src/capture.ts
git commit
```

Suggested message:

```
fix(widget): inline pseudo-element background images before capture

The hero captured blank because its navy background is painted by a
::before whose background is an external CDN image. modern-screenshot
inlines images reachable from a real element but not a pseudo-element's,
so that URL stayed external inside the generated SVG — and a browser
rasterising an SVG as an image cannot fetch external resources. The navy
never painted, the section fell back to white, and the hero's white
heading text became invisible on it.

inline_pseudo_backgrounds() now fetches those assets and re-declares the
background as a data: URL on the clone only, through a <style> scoped to
a per-capture random class so the tester's real page is never touched.
Fail-silent and capped at 2s, per agent-rules.md §1.11.

Real Home page, 1440x900: navy pixels 17,507 -> 725,435 against the
browser's own 722,668; difference from the real render 58.4% -> 7.2%,
the remainder being the fonts EMBED_WEB_FONTS deliberately omits.
Capture time unchanged (934ms; the new step ~74ms, cache hit).
```

Then ask Vishnu before pushing. On his yes:

```bash
git push origin dev
```

## Step 5 — deploy to the server

Per `deploy/runbook.md` "Deploying a code update later", but **only the widget
needs rebuilding** — no dependency changed, and `src/web/lib/widget-asset.ts`
reads `dist/` from disk on every request with `Cache-Control: no-store`, so
the web app does not need rebuilding and the service does not need restarting.

```bash
sudo -u halle-feedback -H bash
cd /opt/halle-feedback/app
git pull
WIDGET_API_ORIGIN=https://feedback.arametrics.app \
  npm run build --workspace halle-feedback-widget-embed
exit
```

**Note the origin.** `deploy/runbook.md` lines 266 and 593 still say
`http://212.227.213.174:3000`, which is stale — the live backend is
`https://feedback.arametrics.app`. Use the domain. Fix those two lines in the
RUNBOOK as a separate commit afterwards, not part of this one.

## Step 6 — confirm it is live

```bash
curl -s https://feedback.arametrics.app/capture.js | grep -c inline_pseudo_backgrounds
```

Expect a number greater than `0`. If it prints `0`, the build did not reach
the server — stop and report.

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://feedback.arametrics.app/v1.js
```

Expect `200`.

## Step 7 — hand back to Vishnu for the live test

Tell him it is deployed and that he should:

1. Open the real Home page with his tester link.
2. **Hard-refresh** (Cmd+Shift+R) so the browser drops the old `capture.js`.
3. Report a bug on the hero.
4. Check the picture in the admin queue.

**Expected:** the hero shows its navy background, and "Tradition Meets
Innovation" and its paragraph are readable.

If the hero is still blank, ask him to run `window.__halleCaptureLog` in the
browser console and send the output — there should be a
`pseudo backgrounds inlined` entry with `tagged: 1, inlined: 1`. If `tagged` is
1 but `inlined` is 0, the CDN refused the fetch on CORS grounds and we need a
different route to the asset; say so rather than guessing.

## Known, not part of this task

- `content: url(...)` on a pseudo-element is still not inlined — same class of
  bug, not used by this site today.
- `crossOrigin='anonymous'` is still set unconditionally on every image, which
  would break any image served without CORS headers. Latent, not triggering.
- No regression test yet for a pseudo-element background surviving a capture.
  Worth adding; not in this task.
