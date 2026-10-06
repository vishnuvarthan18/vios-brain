---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_server-capture.txt
---

Three commits, staged by path. Not committed — waiting for Vishnu's go.

--- 1 ---
git add src/widget/src/capture.ts src/web/app/api/internal/capture/route.ts src/render/hybrid-renderer.mjs

fix(capture): render server pictures from the full page with scripts off

The widget now sends a light description of the page (the whole masked
body, its html/head styling and the scroll position) instead of building
an off-screen copy in the live page. The renderer keeps one warm browser
context per device kind, runs none of the page's scripts, waits for the
stylesheets and images the picture depends on, scrolls natively, and
queues up to six requests behind three renders at once. The old request
shape is still accepted during a deploy.

Measured on the real site, phone/tablet/computer, 29 screens: worst
difference 25.1% -> 0.76%, median render ~1.3s -> ~0.4s, phone-side work
~90-360ms -> ~10-30ms. Server timeout in the widget back to 6s.

--- 2 ---
git add deploy/halle-feedback-hybrid-render.service

build(deploy): size the renderer for three renders on its own box

--- 3 ---
git add scripts/audit-server-capture.mjs

test: add server capture audit for phone, tablet and computer
