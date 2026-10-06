# Agent task — fix pointer-box misplacement, cut capture time, finish the old marker-pen fixes

22 September 2026. Vishnu reported three problems after the hybrid
server-capture went live: some screenshots have elements out of place,
capture is slow again, and the pointer box is landing in the wrong spot.
Background: `docs/agent-task-deploy-server-capture-live.md` (the fix that
just shipped), `docs/agent-task-marker-and-capture-speed.md` (an older,
never-completed task in the same files). Read both first.

**Do not commit or push without asking Vishnu first, as always. One
concern per commit.**

This task has four parts. Do them in order — Part A is the highest
confidence fix and the smallest, Part D is the largest and lowest risk of
regressing something else if done last.

---

## Part A — fix the pointer-box / marker-canvas timing bug (do first)

Root cause, confirmed by reading the code: `src/widget/src/app.ts` computes
where to draw the pointer box (`review-target-box`) and scales the
marker-pen strokes at SEND time using the LIVE, CURRENT
`window.innerWidth` / `window.innerHeight` —

```
const scale_x = img.clientWidth / window.innerWidth;
const scale_y = img.clientHeight / window.innerHeight;
...
const scale = { x: window.innerWidth / canvas.width, y: window.innerHeight / canvas.height };
```

— instead of the screen size that was actually in effect at the moment the
tester clicked the element and the picture was taken. On a phone,
`window.innerHeight` changes when the address bar shows/hides, or the
instant the tester taps into the comment box and the on-screen keyboard
opens (the code already expects testers to type before the picture
arrives — see `carry_comment` in `render_review`). Any of that shifts the
box.

**Fix:** capture `{ width: window.innerWidth, height: window.innerHeight }`
once, at the exact moment `session.target_fingerprint = fingerprint(target)`
is set in `select()` (`app.ts`, inside `install_picker`) — the same instant
the box coordinates themselves are recorded. Store it on `session`
alongside `target_fingerprint`. Use that stored value everywhere the box
and stroke scaling math currently reads `window.innerWidth` /
`window.innerHeight` live: the `img.addEventListener('load', ...)` handler
that positions `review-target-box`, and the `send_btn` click handler that
scales `marker_pen.strokes()`. Do this for BOTH modes (pointer and plain
screenshot), since the marker-pen stroke scaling bug applies in screenshot
mode too even though the pointer box does not.

Verify by hand on a phone: pick an element, wait for the on-screen keyboard
to be up (type something) before the picture finishes loading, confirm the
box lands on the actual element, not shifted by roughly the keyboard's
height.

**Real examples from Vishnu, confirming this diagnosis** —
`docs/evidence/box-misplaced-fairs-card.png` and
`docs/evidence/box-misplaced-paragraph.png`, both from the live
production site (not the dev/staging one used for earlier testing). The
second one is the clearest: the red box is shifted down from the actual
paragraph (its top edge cuts through the middle of a line of text instead
of starting at the paragraph's top) AND is taller than the paragraph,
overshooting into blank space below it. That shifted-down-and-stretched
pattern is exactly what the code above would produce if
`window.innerHeight` was smaller at review time than it was at capture
time (the scale factor becomes larger than 1, so both the box's position
and its height get inflated) — consistent with the keyboard opening, or
the phone's address bar changing, in the gap between click and picture.
Use these two images to confirm the fix visually once Part A is done, not
just by the manual phone test.

---

## Part B — stop the worst-case capture time from growing (report numbers, then apply)

Two separate, additive causes — fix both, measure each on the real Contact
and Home pages (not the local fixture, per this project's standing rule).

### B1. The server-side render reloads the ENTIRE real page for nothing

`src/render/hybrid-renderer.mjs`'s `render()` calls
`page.goto(base_url, { waitUntil: 'load' })` before swapping in the masked
clone. This downloads every image, video, font, ad and tracking script the
real page has — none of which end up in the final picture, since the
clone fully replaces `document.body` right after. On the two pages
measured before going live this was cheap; it will not stay cheap on a
heavier page, and cost only grows as more pages get used.

Try: use `page.route()` to abort requests for resource types the clone
does not need loaded from the fresh page context (images, media, fonts —
the clone's own images/fonts are already inlined as data URLs by
`build_capture_clone()` on the client, per `capture.ts`'s own comments
about privacy/framing masking). Keep `document`, `stylesheet`, and `script`
requests flowing, since the clone relies on the real page's CSS classes
and any layout-affecting JS the header comment in `hybrid-renderer.mjs`
already explains is the whole reason this approach was chosen over a
static snapshot. Measure `waitUntil: 'load'` time before and after on both
pages. If blocking images/fonts breaks anything (a layout-affecting script
that reads image dimensions on load, for instance), say so and scale back
rather than force it.

### B2. The two capture methods run one after another, not together, on failure

`src/widget/src/capture.ts`: when the server attempt fails or times out
(`SERVER_CAPTURE_TIMEOUT_MS = 6_000`), the client-side fallback THEN starts
from zero and can itself take up to `CAPTURE_BUDGET_MS = 12_000`. Worst
case, a tester can now wait far longer than before this project started,
because the two are tried sequentially instead of in parallel.

Do not simply run both at once on every report — that would double the
work (network + CPU) for every SUCCESSFUL server capture too, most of
which are fine. Instead:

- Re-measure how long a successful server capture actually takes today on
  the real Contact and Home pages (the earlier number was ~2,012ms — get a
  fresh one, and one for a heavier third page if you can find one on the
  live site). Propose a `SERVER_CAPTURE_TIMEOUT_MS` tightened to comfortably
  cover a real success but fail fast otherwise, rather than the current
  6,000ms, which was not measured against real success times.
- Separately, look at whether `build_capture_clone()` and the font-ready
  wait (the setup work the CLIENT fallback needs before it can even start
  rendering) can begin at the same time as the server network request goes
  out, so that if the server does fail, the fallback does not start that
  prep work from zero. Only the render/upload step itself should wait on
  the server's outcome. If this is not a clean, low-risk change, report the
  numbers instead and leave it for a decision rather than forcing it.

**Report the before/after timing table for both B1 and B2 before moving on
— do not guess whether they helped.**

---

## Part C — record which method actually produced each report's picture

There is currently no way to tell, after the fact, whether a given report's
screenshot came from the new server method or the old client fallback. This
is needed to diagnose Vishnu's third complaint (some screenshots have
elements out of place) — it is not yet clear whether that is the old
method quietly covering for the new one under load, or a separate gap in
the new method itself, and there is no way to tell without this.

Add a field recording which path produced the final image — e.g.
`capture_method: 'server' | 'client' | 'none'` — somewhere in the report
payload (`ReportBody` in `types.ts`, or its `meta` — your call on the
cleanest spot) and thread it through from `capture_screenshot()`'s return
value (it already knows internally which path returned the blob) to
`send_report()`. State in your report where it ends up in the database /
admin view so it can actually be looked at later, even if surfacing it in
the admin UI itself is out of scope for this task.

---

## Part D — finish the marker-pen fixes from 10 September (never completed)

`docs/agent-task-marker-and-capture-speed.md` Part A (items A1 through A8)
was written and assigned on 10 September but never finished — there is no
report doc for it and no commit touching `marker-pen.ts` for it. Checked
today: the review screen is still a small docked panel
(`.review-image-wrap { width: 100% }` is 100% of the panel, not the
screen), so **A8 (full-size drawing surface) was not done**, and neither
was A1 (canvas backing-store at CSS size, not `devicePixelRatio`) — both
bugs are still live in `marker-pen.ts` / `app.ts` today.

Do Part A of that document now, items A1-A8, in the order given there.
Before starting, re-check each item against the CURRENT code (the widget
was rebuilt in React/shadcn since that doc was written, commit `dc96cb5`)
in case the rewrite incidentally already fixed something on the list —
skip anything already done and say so in your report, do not re-do it.

Part A's own doc has full detail per item (canvas resolution, stroke-width
mismatch, smoothing, coalesced pointer events, single-tap dots, the
always-on pen becoming a toggle icon, full-size drawing surface). Follow
it as written; nothing in this task changes any of its decisions.

Do NOT touch Part B of that older document (the 500ms speed work) — that
part is already done and reported (`docs/report-screenshot-speed-500ms.md`).

---

## Do not do

- Do not change `MAX_CONCURRENT_RENDERS` (still 1) without asking first —
  raising it trades against the server's very tight memory (see
  `docs/agent-task-install-browser-libs-on-prod-results.md`), and that
  trade-off is Vishnu's call, not yours, made on measured numbers if you
  think it is worth raising.
- Do not deploy any of this to the live server as part of this task.
  Everything here is build-and-test-locally, then stop and report — same
  as every previous task in this project. Deploying is always a separate,
  explicit step.
- Do not start the broader review-screen redesign (mentioned in the older
  doc as "getting its own dedicated design pass") — that is out of scope
  here.

## Report back

Per part: what was wrong, what changed, and how you know it works (a
before/after number for B, a description of the manual phone test for A,
which of Part D's items were already done vs newly fixed). For Part C,
where the new field ends up. Flag anything that needs a decision rather
than deciding it yourself, as usual.
