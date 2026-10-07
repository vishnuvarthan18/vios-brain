# Research — is there a better way to take the picture?

Written 10 September 2026. Vishnu's challenge, in his words: *"why are we
using this… we need to find a more easy and more reliable way"*, with a
target of **500ms** and the product argument that **a tester who waits
will abandon the report**.

Both points are correct. This document answers the architecture question
rather than defending the current choice.

---

## 1. What we do today, and what it actually is

The widget uses `modern-screenshot`. It clones the visible DOM, copies
every computed style onto the clone, downloads and inlines every image
and font as base64, wraps the result in an SVG `foreignObject`, hands
that to the browser as an image, and rasterises it to WebP.

It is not a photograph of the screen. **It is a re-render** — the browser
is asked to draw a reconstruction and hope it matches. That is why it is
slow (5.8s per pass on the real Contact page) and why it has a permanent
tail of fidelity bugs.

## 2. What the competitors actually do — from our own teardowns

This project already researched three of them. The relevant findings were
sitting in `claude/research-raw-*-teardown.md`:

**Marker.io — the closest product to ours.** Its **default** method is
**server-side rendering**: the DOM and assets are serialised in the
browser, sent to Marker.io's own infrastructure (`ssr.marker.io`, which
their published CSP list requires), and the image is rendered *there*.
They call it the "screenshot renderer" and rewrote it, CTO-led, to be
"faster and more stable". Their two alternatives are a native browser
screenshot API — which "requires you to manually choose the screen or
browser tab you want to capture **each time**", desktop-only — and a
browser extension.

**BugHerd** keeps its method a trade secret, but published what it
**rejected**, which is more useful:

- **html2canvas rejected outright** — it "fakes a screenshot by manually
  rebuilding your webpage in Canvas", has "partial CSS support", and is
  "an interpretation", unsuitable when "a customer is reporting a layout
  issue".
- Extensions rejected — "browser support is variable, and mobile devices
  are not supported".
- Server-side rendering rejected — "cannot support mobile browsers".

**The uncomfortable conclusion: our current approach is the same family
of thing BugHerd publicly rejected as "an interpretation".** We are not
using it because it is best; we are using it because it needs no
permission, no install and no server. Those are real advantages, but they
were never weighed against the cost we are now paying.

Marker.io's own documented capture failures, which apply to any method in
this family and therefore largely to ours: external iframes, Shadow DOM,
canvas and WebGL, embedded video, CSP-restricted sites, Mapbox and
mapping libraries.

---

## 3. The four architectures actually available to us

### A. Keep browser rendering, optimise it hard

The 500ms programme in `docs/agent-task-marker-and-capture-speed.md`:
stop the double capture, reuse the asset cache, cut the copied style
properties, skip scrollbars and iframes, possibly swap to snapDOM.

- **For:** no server work, no permission, no install, nothing changes for
  the tester, and it is already written up.
- **Against:** may not reach 500ms — unknown until measured. And it does
  nothing about fidelity; it stays "an interpretation" with a permanent
  tail of edge cases.

### B. Serialise in the browser, render on our server — what Marker.io does

The browser collects the DOM and stylesheets (fast — no image fetching,
no rasterising, tens of milliseconds) and posts that. The server renders
the picture with headless Chrome.

- **For:** the tester waits for almost nothing. Fidelity is high, because
  the render is of the tester's actual DOM at that moment. It is the
  market leader's default choice.
- **Against:** it needs headless Chrome on **our** server, and that
  server is a low-memory shared VPS that already strains on a Next.js
  build (the app is capped at 1GB by its own systemd unit). Chrome wants
  several hundred MB per render. With a handful of internal testers and
  reports arriving one at a time this is probably survivable — one render
  at a time, queued — but it is real new infrastructure to build, run and
  keep alive, on a box that also hosts unrelated projects.
- Also: the tester cannot see or draw on the picture before sending,
  unless the flow changes — see §4.

### C. Native browser screenshot (`getDisplayMedia`)

- **For:** genuinely instant and pixel-perfect. It is a real photograph.
- **Against:** a permission prompt every time, and a "you are sharing
  your screen" indicator. **Our own Marker.io teardown already reached a
  verdict on this: "Disqualifying for elderly non-technical testers."**
  That verdict still stands.

### D. Browser extension

- **For:** pixel-perfect, the method every vendor agrees is most
  accurate.
- **Against:** every tester must install it, and there is no mobile
  support at all. B. Halle's testers are internal staff, so an install is
  not unthinkable — but it is a significant ask of non-technical people
  and it kills phone testing outright.

---

## 4. The flow insight — the tester should never wait for a picture at all

The reason the picture currently blocks the tester is that they **draw on
it**. That is the only reason it must exist before Send.

But they do not have to draw on a picture. **They can draw directly on
the live page.** The strokes are recorded as coordinates either way.

That changes the shape of the problem completely:

- Press "Report a Bug" → choose Pointer or Screenshot.
- Pointer: click the element. Instant — a box is drawn on the live page.
- Screenshot: draw straight onto the live page with the pen. Instant, and
  on a far bigger, more natural surface than a thumbnail preview.
- Type the comment, press Send. The report posts **immediately**.
- The picture is produced in the background and attached to the report
  afterwards.

**The architecture already supports this.** `screenshot_key` is nullable
by design, and pictures are uploaded through a separate signed slot — so
a report can be stored first and its image attached seconds later. That
was built for a different reason, but it is exactly what is needed here.

This also **removes three of the marker-pen defects outright**, rather
than fixing them: there is no small preview canvas, so there is no
half-resolution drawing surface, no preview-versus-sent scaling
mismatch, and no cramped drawing area. The tester draws at full size on
the real page, and those exact coordinates are burned in later.

**The cost:** the tester no longer sees the final picture before sending.
Against that — they are drawing on the real page, which is what gets
captured, so "what you see is what you get" is arguably *more* true than
it is today, not less.

This is a flow change and therefore Vishnu's call, not the agent's. It
can be combined with A or B; it is not an alternative to them.

---

## 5. Recommendation

**Do §4 (never make the tester wait) and A (optimise the capture) — in
that order.**

Reasoning:

- §4 solves the actual product problem Vishnu named — *"the user will go
  away"*. It makes the wait **zero**, not 500ms, and it does so without a
  permission prompt, without an install, and without putting headless
  Chrome on a struggling VPS.
- Once the capture is off the tester's critical path, A stops being
  urgent and becomes ordinary housekeeping — worth doing, but no longer
  the thing standing between a tester and a filed report.
- B stays on the table as the answer if fidelity — not speed — turns out
  to be the real complaint later. It is the market leader's choice and it
  is the right *eventual* architecture, but it is a much bigger build and
  the server it would run on is already the weakest part of this setup.
- C and D are already ruled out by this project's own research and
  audience.

**One risk to hold honestly:** with §4 the tester is gone before anyone
knows whether the capture worked. Today a failed capture is visible as
"(no picture)" while they are still there. So the admin side must show
clearly when a report arrived without its picture, and the capture must
be given a fair chance to finish after Send — started at Send, uploaded
when ready, with the report already safely stored either way.

---

## 6. What Vishnu needs to decide

1. **Adopt the §4 flow** — draw on the live page, send instantly, picture
   attaches in the background? This is the recommendation, and it is the
   only option that makes the wait genuinely zero.
2. If not §4, then **A or B** — optimise what we have and accept
   whatever number that reaches, or build server-side rendering on the
   VPS.

Sources for §2 are the vendor pages cited in
`claude/research-raw-markerio-teardown.md` and
`claude/research-raw-bugherd-teardown.md`.
