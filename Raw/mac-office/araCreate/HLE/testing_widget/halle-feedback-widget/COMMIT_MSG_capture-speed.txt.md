---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_capture-speed.txt
---

fix: stop waiting on lazy images that never load

A capture cost 11.6s on the real Contact page. 81% of that was the library
waiting for images the browser had been told not to fetch.

Every image on the page carries loading="lazy". The capture clone is parked
at left:-999999px so it never flashes in front of the tester, which puts
every cloned image a million pixels outside the viewport — so the browser
correctly never requests them, and they never fire load or error.
waitUntilLoad() can then only resolve them when the per-asset timeout
expires, which is why a pass cost 5,001ms against a 5,000ms
ASSET_TIMEOUT_MS. Of 71 images in the clone, 61 never settled.

Forcing loading="eager" and decoding="sync" on the clone takes that phase
from 5,001ms to ~7ms.

Also, of those 71 images only 13 have a real rendered box. 55 of the rest
sit inside a display:none Webflow collection list, reporting a 0x0 rect —
fetched, base64'd and inlined to draw nothing. Non-rendering images are now
pruned with the off-screen ones.

With the cause fixed, the double capture is no longer needed. It existed for
Safari's blank first render and was justified as "a redundant second capture
is cheap", measured at 98ms on the local fixture; on the real page it was
5.8 seconds. Fonts are now awaited before rendering (bounded, so a stalled
host font cannot hang a capture) and the library already awaits img.decode()
internally, which is the readiness the throwaway pass stood in for. NOT
verified on Safari — no WebKit build available here.

includeStyleProperties with a curated list of what affects a screenshot cuts
the per-node style copy from ~192ms to ~48ms; copyScrollbar:false cuts the
canvas draw from ~357ms to ~160ms; iframe/video/canvas subtrees are filtered
out.

font: { preferredFormat: 'woff2' } is deliberately NOT used. On this page it
strips every @font-face src rather than picking a format — all 15 faces are
.otf with no woff2, and filterPreferredFormat returns an empty string when
nothing matches. It looked like a free 4x win; it was really the fidelity
trade-off behind EMBED_WEB_FONTS, taken without anyone deciding to take it.

Net: 11,576ms to 1,580ms with fonts embedded. EMBED_WEB_FONTS=false reaches
152ms and is left as Vishnu's decision, documented with both numbers.

Two regression tests on a new fixture reproducing both real-page conditions;
both fail against the old code.

Measurements and the full phase table: docs/report-screenshot-speed-500ms.md
