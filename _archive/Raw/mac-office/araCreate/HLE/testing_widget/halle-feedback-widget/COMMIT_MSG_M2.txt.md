---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M2.txt
---

feat: build the widget — all five states, Shadow DOM, local test page

M2. idle -> pointing -> question -> detail -> sent, both escape hatches,
on a local test page. Screenshots are out of scope here (M6), and so is
anything past this milestone's own test page (M2b is the live Webflow
site, tomorrow).

- state machine: launcher gated by config.launcherVisibility ('all' |
  'token', new field, defaults to 'token'), mousemove -> elementFromPoint
  outline in the Shadow DOM, click captured in the CAPTURE phase with
  preventDefault/stopPropagation/stopImmediatePropagation, re-read at
  click/touch-confirm time rather than trusting the hover target
- touch: first tap highlights and shows a confirm bar, second tap
  confirms — re-reads the element again at confirm time, not the first
  tap's element, in case the page moved between the two taps
- question: five options shuffled per session (Fisher-Yates) plus
  "something else" always last, shown order sent as optionOrder; radio
  list, arrow keys + Enter/Space, one tap/click advances, no submit
  button
- detail: optional textarea, Send or Skip and send, both submit
- sent: thank-you, then "anything else on this page?" loops back to
  pointing in the same session, or closes to idle
- fingerprint: selector built walking up to <body>, Webflow's w-/w--
  classes stripped before building it, element text captured separately
  since a renamed style changes the class and not the words
- token: read once from ?t=, held in a module closure only — never
  localStorage/sessionStorage/a cookie; same-origin <a href> values
  rewritten to carry it forward, on load and on DOM mutations that add
  links; history.replaceState guards the visible URL
- API origin: baked in at build time via an esbuild define
  (WIDGET_API_ORIGIN, defaulting to localhost:3000 for dev), with a
  data-api attribute on the script tag as a per-embed override — the
  widget's own script src can't be used for this, since the bundle is
  served from a CDN origin different from the API's; a missing/invalid
  origin renders nothing, same as an unknown key or a config fetch
  failure
- failure handling: config fetch failure or 404 renders no DOM node at
  all; a broken report POST still shows the thank-you screen after two
  retries with backoff, queued in memory only
- accessibility: everything from the question step onward is fully
  keyboard operable (focus trap, Escape closes, arrow keys move through
  options, visible focus ring) — element selection itself stays mouse
  and touch only, by decision (build-plan.md §9), and there is no WCAG
  conformance claim anywhere
- tests/widget/host-page.html + host-page-2.html: a deliberately hostile
  global stylesheet (aggressive div/button/p/a/table selectors), a
  sticky header, and an element that relocates on mousedown, to prove
  Shadow DOM isolation and the re-read-at-click-time rule under
  conditions designed to break them
- tests/widget/acceptance.spec.ts: 22 Playwright checks against the host
  pages — every state transition and both escape hatches, option order
  differing between sessions with the shown order captured, tap-confirm,
  nav-link interception without navigating, the sticky-header/mover
  re-read case from both the wrong-click and right-click sides, unknown
  key and broken-API silence, hostile-CSS non-leakage in both
  directions, token survival across a real navigation with zero cookies/
  localStorage/sessionStorage afterwards, full keyboard-only completion
  from the question step on, and config strings changing the widget with
  no rebuild
- @playwright/test added as a root devDependency, for tests/ only —
  never a dependency of src/widget or src/web

Found while wiring the picker: the capture-phase click handler was
calling stopImmediatePropagation() before checking whether the click
landed on the widget's own controls, which made the pointing bar's own
"It was the whole page" and "Stop" buttons unclickable. Reordered so the
own-host check runs first — only a click that lands on the HOST PAGE
gets captured and stopped.

5,638 bytes gzipped, comfortably inside the 15 KB budget.

See docs/build-plan.md §5 and §7 (M2), and agent-rules.md §1.2, §1.3,
§1.4, §1.9, §1.10, §1.11, §2.2, §2.3, §2.4, §2.5.
