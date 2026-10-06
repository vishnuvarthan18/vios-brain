# Research — how the industry actually solved this

10 September 2026. Five parallel research streams, on Vishnu's
instruction to *"take all the records from the web timeline and see how
they solved it… deeper and wider."*

Three of the findings overturn things we believed, including one idea I
proposed earlier in this session. Sources are inline; every claim is
labelled where the evidence is weak.

---

## 1. Nobody rasterises in the browser. Everybody renders on their server.

This is the headline. Of every competitor that supports mobile:

| Product | Default capture method |
| --- | --- |
| **Marker.io** | DOM scanned client-side → posted → rendered at `ssr.marker.io` |
| **Ybug** | Server-side: "Ybug's rendering server needs access to your site's static assets" |
| **Usersnap** | Server-side: "has to reach out to the page you want to take a screenshot for" |
| **Userback** | Server-side is the standard engine |
| **BugHerd** | Undisclosed, but publicly rejected html2canvas, extensions **and** URL-fetch server rendering |
| **Jam.dev** | Browser extension only — not embeddable |
| **Bird Eats Bug** | Extension + `getDisplayMedia`; replays via rrweb |
| **Pastel** | Not a screenshotter at all — a proxy that serves the live site with comments layered on |

**Native browser screen capture is, in every single product that offers
it, the opt-in desktop-only fallback — never the default.** Marker.io,
Usersnap and Userback all state plainly that mobile falls back to
server-side rendering.

**So the approach we are using — rasterising the DOM in the tester's
browser — is the one approach no serious vendor in this category ships.**
That is the answer to "why are we using this": not because it was
chosen over the alternatives, but because it needs no server.

One detail that makes server rendering much cheaper than assumed:
Marker.io's renderer **fetches the page's assets by URL itself** — they
publish four static renderer IP addresses that must be able to reach the
customer's site. The browser therefore only has to send the HTML and CSS,
not megabytes of inlined base64 images. On a public Webflow site that
works without any allowlisting.

## 2. No shipping tool lets anyone freehand-draw on a live, scrolling page

I proposed exactly that earlier in this session. The survey says it does
not exist anywhere, and the split is clean:

- **Tools that work on the live page support pointing only** — a click
  or a pin on an element. BugHerd, Pastel, Filestage, Atarim.
- **Every tool that supports a freehand pen freezes the page first and
  draws on the frozen image.** Marker.io, Ybug, Usersnap, Jam,
  Instabug/Luciq.

Atarim states the reasoning outright: the screenshot is taken at the
moment of the comment "so markups remain accurate even after the site
updates." Filestage deliberately preserves even transient UI — "the
temporary dialog box or pop-up menu that the commenter saw."

**Conclusion: freeze-then-draw is correct and my live-page proposal was
wrong. Drop it.** What is wrong today is not that we freeze — it is that
we make the tester wait 11 seconds for the freeze, and then hand them a
thumbnail to draw on.

## 3. The answer to scrolling is a labelled mode, not a silent lock

Vishnu rejected locking the scroll, and he was right to — but the
industry did not solve it by allowing free scrolling either. It solved it
with **explicit, labelled modes**: Filestage ships *comment mode* vs
*navigation mode*; Atarim tells users to "switch to Browse to move
around, switch to Comment to leave feedback"; Pastel toggles the same
way.

The difference from what I proposed is honesty of state: the page is not
mysteriously frozen, the tester is visibly in a drawing mode and can see
how to leave it.

Jam does something additionally clever: a cropped shot **also** captures
the full page, and shows both side by side.

## 4. On touch, drawing must be explicitly armed — Vishnu's instinct was right

Vishnu asked for the pen to be an icon you click before you can draw.
That is exactly what the industry does, and for exactly this reason:

- Miro: one-finger drag pans, drawing needs a deliberate gesture.
- Figma: drag is a comment, so panning requires a modal tool switch —
  and there is a long-running user complaint about it.
- Usersnap on mobile: "screen capturing features and annotation toolbar
  are not supported" — they gave up entirely.
- Sentry: "screenshots aren't supported on mobile devices, so the
  screenshot button is hidden."

So: navigation is the default gesture, drawing is armed deliberately, and
a point-only fallback on mobile is a respectable, shipped answer.

## 5. Our 11.6 seconds is normal for this technique, not a bug in our code

monday.com's engineering team measured the same family of libraries on
real content: **html2canvas 21s → modern-screenshot ~7s**. Our 11.6s sits
squarely in that range.

The lineage matters: dom-to-image → html-to-image → modern-screenshot →
snapDOM are **all the same technique** (clone, inline styles, inline
assets as base64, wrap in an SVG `foreignObject`, decode as an image).
snapDOM is not a new idea — its own docs describe the same pipeline. Its
published 10–25× numbers are the author's own, synthetic, and **no
independent real-page benchmark exists**. Treat them as a reason to
measure, not as fact.

The dominant cost is almost certainly **asset inlining**: each image
fetched, converted to base64 (+33% size), concatenated into one giant SVG
string that must then be parsed and decoded serially on the main thread.

## 6. There is no permissionless native path, and none is coming

- `getDisplayMedia` permission **can never be remembered**. The W3C
  Working Draft of 27 August 2026 is normative: the browser *"MUST NOT
  store a 'granted' permission entry"*.
- `preferCurrentTab: true` does **not** remove the picker — the spec only
  says the current tab should be made prominent. Chrome/Edge only.
- **Zero mobile support**, on any browser.
- `getViewportMedia` — the one serious proposal for permissionless
  self-capture — has been stalled since about 2021. The blocker is
  structural: your viewport contains cross-origin iframes and rendered
  link-visited state, so reading your own pixels is a same-origin-policy
  bypass by another name.
- View Transitions and Firefox's `element()` are dead ends — snapshots
  are deliberately not readable by JavaScript.

**One genuinely useful finding:** because the grant cannot persist, the
right pattern *if* we ever used it is to acquire the stream **once** on
the tester's first report and keep it alive for the session — every
subsequent capture is then zero clicks and about 50ms. The price is a
permanent "sharing your screen" indicator. Desktop only, so it could only
ever be a fast path with a fallback underneath.

## 7. Do not attempt full-page capture in the widget

- **iOS canvas limit is 4,096 × 4,096**, and exceeding it fails
  **silently with a blank image** rather than throwing.
- Safari also enforces a total canvas memory cap (~384MB on iOS 15+).
- WebP maxes out at 16,383px regardless of platform.
- A 1440 × 20,000 page is ~115MB of raw bitmap — fine on desktop, fatal
  on iOS.
- Scroll-and-stitch duplicates every sticky header on every slice; the
  published fixes (CSS-transform stitching, reparenting sticky nodes) are
  what Applitools and Percy had to build.
- **No embedded JavaScript widget ships reliable full-page capture.**
  Marker.io's "capture entire page" is an *extension* feature and even
  then "might not work on every page".

If a taller-than-viewport region is ever needed, clip **before**
rasterising (offset the render so the region sits at the origin — which
is a small generalisation of the scroll-offset trick already in our
`capture.ts`), cap at roughly 4,000px tall and ~12 megapixels for iOS
safety, and emit several images rather than one monolith.

---

## What this means for us — two routes

### Route 1 — fix the client pipeline (no new infrastructure)

Everything already written in
`docs/agent-task-marker-and-capture-speed.md`, plus one better idea from
the research: **kill the double capture properly**. Rather than sniffing
for Safari, the blank-first-render is avoided by awaiting
`document.fonts.ready` and `img.decode()` before rasterising. That is a
correctness fix rather than a workaround, and it halves the time on every
browser.

With that, plus `scale`, subtree filtering, context reuse and workers,
the researcher's estimate is **low hundreds of milliseconds**. That would
meet the 500ms budget with no server work at all. Unproven until
measured, but credible.

### Route 2 — server-side rendering, like everybody else

Client serialises the DOM and CSS (milliseconds), posts it, our server
renders the image with headless Chrome. This is the industry default and
it fixes fidelity as well as speed.

Cost: headless Chrome on a low-memory shared VPS whose app is already
capped at 1GB. With a handful of internal testers filing occasional
reports, one render at a time behind a queue is feasible — but it is real
infrastructure to build, run and keep alive.

### Recommendation

**Route 1 first, Route 2 held in reserve.** Route 1 is already specified,
needs no new infrastructure, and the evidence says it can plausibly hit
the target. If it lands above 500ms after honest measurement, Route 2 is
the proven answer and we build it then — with numbers in hand rather than
on a guess.

**And regardless of route, the flow changes:**

1. **Keep freeze-then-draw.** My live-page drawing idea is dropped —
   nobody ships it, for good reasons.
2. **Draw on a full-size frozen image, never a thumbnail.** This is the
   real fix for the pen quality, on top of the six defects already
   listed.
3. **Arm the pen explicitly**, exactly as Vishnu asked, with navigation
   as the default gesture.
4. **Viewport only.** No full-page capture in the widget.
5. If the tester needs to move around, that is a **labelled mode switch**,
   not a silent scroll lock.

---

## Sources

Marker.io: [data masking](https://help.marker.io/en/articles/9657817-sensitive-data-masking),
[firewall/renderer IPs](https://help.marker.io/en/articles/6840044-configuring-marker-io-for-firewalls-and-secure-networks),
[native screenshot rendering](https://help.marker.io/en/articles/9615303-native-browser-screenshot-rendering),
[screenshot limitations](https://help.marker.io/en/articles/6282853-widget-screenshot-tips-limitations),
[extension FAQ](https://help.marker.io/en/articles/6501657-browser-extensions-faqs).
BugHerd: [screenshots without a browser extension](https://bugherd.com/blog/screenshots-without-a-browser-extension),
[CSP](https://support.bugherd.com/en/articles/11430711-content-security-policy-csp).
Ybug: [screenshot issues](https://ybug.io/docs/troubleshooting/screenshot-issues).
Usersnap: [feedback with a screenshot](https://help.usersnap.com/docs/feedback-with-a-screenshot),
[mobile beta](https://help.usersnap.com/docs/collecting-customer-feedback-on-mobile-apps-beta).
Userback: [native screenshot](https://support.userback.io/en/articles/9417749-native-screenshot).
Bird Eats Bug: [what Bird captures](https://birdeatsbug.com/help/which-data-does-bird-capture-and-log).
Pastel: [reviewing websites](https://help.usepastel.com/en/articles/1996011-reviewing-websites-in-pastel).
Atarim: [visual collaboration tools](https://atarim.io/help/core-features/visual-collaboration-tools-overview/).
Filestage: [review live websites](https://help.filestage.io/en/articles/5755744-review-live-websites).
Jam: [screenshot](https://jam.dev/docs/product-features/screenshot), [tldraw rebuild](https://tldraw.dev/blog/jam).
Miro: [touch input](https://help.miro.com/hc/en-us/articles/360017731053-Using-Miro-with-a-mouse-trackpad-or-touchscreen).
Sentry: [user feedback](https://docs.sentry.io/platforms/javascript/user-feedback/).
Performance: [monday.com engineering](https://engineering.monday.com/capturing-dom-as-image-is-harder-than-you-think-how-we-solved-it-at-monday-com/),
[snapDOM](https://github.com/zumerlab/snapdom), [modern-screenshot](https://github.com/qq15725/modern-screenshot),
[html-to-image](https://github.com/bubkoo/html-to-image), [rrweb overhead](https://launchdarkly.com/docs/tutorials/session-replay-performance),
[FullStory server rendering](https://www.fullstory.com/blog/creating-screenshots-puppeteer-and-session-recording/).
Native APIs: [W3C Screen Capture WD 2026-08-27](https://www.w3.org/TR/2026/WD-screen-capture-20260827/),
[MDN getDisplayMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia),
[prefer-current-tab](https://wicg.github.io/prefer-current-tab/), [caniuse](https://caniuse.com/mdn-api_mediadevices_getdisplaymedia),
[Element Capture](https://developer.chrome.com/docs/web-platform/element-capture),
[getViewportMedia proposal](https://github.com/w3c/mediacapture-viewport).
Canvas limits: [MDN canvas](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas),
[pqina on Safari canvas memory](https://pqina.nl/blog/total-canvas-memory-use-exceeds-the-maximum-limit/),
[Applitools sticky elements](https://help.applitools.com/hc/en-us/articles/360007188071-Using-CSS-transition-with-Applitools-as-an-alternative-to-standard-scrolling),
[Percy sticky elements](https://www.browserstack.com/docs/percy/stabilize-screenshots/sticky-elements),
[snapDOM huge page mosaic](https://snapdom.dev/blog/huge-page-mosaic/).
