# WIDGET v2 — SPECIFICATION

**Decided by Vishnu, 8 September 2026. Flow finalised and mocked up 8 September
2026. Corrected against the repo, 8 September 2026 evening — see §11.** This
replaces the widget flow in `build-plan.md §5`. The team-side app, storage,
auth and exports are unchanged by this document, except where §5 below says
otherwise.

Audience: everyone — elderly testers, the client, designers and developers,
all using the same widget.

**A working, clickable mock of everything in this document exists:**
`widget-prototype-v2-flow-mock.html` in the Claude project. Click through it
before building — it is the flow, not a picture of the flow.

**This document is not a build order on its own.** §8 lists what was still
open when it was written. `v2-build-plan-for-agent.md §1` settles every one
of those that would block a build, and the ones it leaves open are marked
non-blocking there. Read the build plan, not this document, to decide what to
do when something here is unclear.

---

## 1 The flow

```
Report a Bug
   |
   +-- Point at the problem -> click an element -> picture taken at once,
   |                            element boxed in
   |
   +-- Screenshot -> picture taken at once, no clicking
   |
   +-- both land here --------------------------------------+
                                                              |
                              ONE SCREEN: picture + marker pen + comment box
                              (comment required) -> Send
                                                              |
                                                   Thank you, closes itself
```

### Launcher

- One button, bottom right. Label: **"Report a Bug"** (from config, editable).
- Clicking it opens a small panel **docked in the same corner**, not a box in
  the middle of the screen. See §4.
- **Visible only via the invited link.** No token check that shows/hides a
  button on a public page — the button only exists for someone who opened the
  page through their invited link. The client and designers get invited links
  too. Decided 8 Sept, replacing the earlier "token-gated launcher" open
  question.
- One link, every page. A tester's link is valid for reporting anywhere on
  the site; there is no set of assigned pages (tester-page assignment was
  removed 9 Sept — `admin-v2-spec.md §2`). The config's tester object
  therefore carries the tester's **name only**.

### Expired link

Added 9 Sept, from the flow review — the one genuinely bad tester-facing
problem found in it.

A token stops resolving more often than it sounds: the tester was removed, or
they reached the page from a bookmark, a search result, or a link a colleague
forwarded — none of which carry `?t=`. Previously the launcher simply did not
appear, which a tester cannot tell apart from the tool being broken.

- When a request carries a token that does **not** resolve, the config
  responds `tokenExpired: true` and no `tester` object.
- The widget then shows a small notice docked in the launcher's own corner —
  **"Your testing link has expired"** / **"Please ask for a new link, then
  open it again to carry on reporting."** Both strings are editable in admin
  like every other tester-facing string.
- The notice is dismissible. Dismissal lasts for that page load only —
  nothing is written to the device (`agent-rules.md §1.3`).
- No retry button: the token is wrong, not the network, and the next step is
  a human one.
- **A request with no token at all still renders nothing whatsoever.** This
  is the load-bearing half: an ordinary visitor to the client's public site
  must never see any part of this widget. Only someone who actually followed
  an invitation link is told anything.
- Deliberately does **not** distinguish "removed" from "never valid" — both
  mean the same thing to the person holding the link, and saying which would
  leak whether a token ever existed.
- A `tokenExpired` response is never cached (`private, no-store`), so a
  shared cache can never serve it on to a tester whose link is fine.

### Point at the problem

1. The user picks it.
2. Page enters pointing mode — hovering outlines the element under the
   cursor.
3. Click selects it. Click is intercepted in the capture phase so a link
   selects instead of navigating.
4. **The picture is taken immediately on click** — with the selected element
   drawn as a box on the image. There is no separate OK step between clicking
   and the picture being taken.
5. Goes straight to the one shared screen (§2).

### Screenshot

1. The user picks it.
2. A picture is taken immediately, no element selection.
3. Goes straight to the one shared screen (§2).

---

## 2 The one shared screen

Both paths land on the same screen. It shows, top to bottom:

1. The picture (with the element boxed in, if it came from Point at the
   problem).
2. The marker pen, live over the picture — **available on both paths**, not
   screenshot-only. Freehand strokes, Undo, Clear. See §6.
3. The comment box. **Required.** Send stays disabled until something is
   typed.
4. A small, closed-by-default section: **"What else we send with this"** —
   see §7.
5. Send and Cancel.

There is no separate confirmation step before this screen, and **no separate
consent step** — see §11. What the tester sees here is what gets sent.

---

## 3 What is reused from what already exists

| Piece | Status |
| --- | --- |
| Element pointing, hover outline, capture-phase click | Built (M2). Reuse |
| Element fingerprint — selector, text, tag, coordinates | Built (M2). Reuse |
| Screenshot capture, Safari double-capture, `crossorigin` | Built (M6b). Reuse for both modes |
| Privacy stripping — inputs, textareas, contenteditable, `data-fb-block` | Built (M6b). Keep, do not rewrite |
| Signed upload, storage, retention | Built (M6a). Reuse |
| Invited link, nothing written to the device | Built (M2), tightened per §1 above |
| Shadow DOM, size budget, silent failure | Built. Reuse |
| Dashboard, issues, roles, exports, admin, string editor | Built (M3–M5). Needs the payload changes in §5 |

## 4 The panel itself — structure the agent should follow

This is UI/UX behaviour, not just visual style. The mock demonstrates all of
it; build to match it, not to the description alone.

- **Docked, not floating.** The panel opens exactly where the launcher was
  and stays anchored to that corner for the whole flow. Never a box centred
  on the screen, never a dark overlay covering the page — the tester can see
  the site behind the panel the whole time.
- **Fixed header, fixed footer, scrolling middle.** The step title (and back
  button, where there is one) stay in a header that does not move. Send and
  Cancel stay in a footer that does not move. Only the middle — the picture,
  pen, comment box, details section — scrolls if a step is tall. This is what
  stops the panel jumping around as the tester moves through it.
- **The header carries a Back button** where going back makes sense (from
  "Point at the problem" pointing-mode back to the two choices). No back
  button once a picture has been taken — retaking is a separate open question
  (§8).
- **Icon plus words on every control**, including the mode-choice buttons,
  Undo, Clear, Back, and Close — never an icon alone. This audience includes
  elderly testers; an icon-only control fails the same recognition research
  that ruled out near-synonym checklists in v1.
- Steps cross-fade rather than hard-cutting, and the whole panel eases in
  rather than snapping into place. All of it respects
  `prefers-reduced-motion` — no motion forced on a tester whose system asks
  for none.
- Mobile: panel docks across the bottom of the screen rather than to a
  corner.

## 5 Report payload changes

```
comment       text          the user's typed comment (replaces answer_id + note). Required, min 1 char
mode          text          'pointer' | 'screenshot'
target_*      unchanged     populated for pointer mode, null for screenshot mode
markup        jsonb | null  the marker strokes — burned into the image as well as stored as coordinates
answer_id     null          retained column, no longer written
option_order  null          retained column, no longer written
meta          jsonb         see below — captured automatically, nothing extra asked of the tester
```

`meta` fields (this is the "What else we send with this" section, §7):

```
page_url        text
referrer        text | null
screen_size     text   e.g. "1512 × 982 px"
viewport        text   e.g. "1502 × 760 px"
device_type     text   'phone' | 'tablet' | 'desktop'
browser         text   name + version, e.g. "Chrome 129.0.6668"
os              text
input           text   'touch' | 'mouse / trackpad'
language        text   navigator.language
captured_at     timestamp
console_errors  jsonb  up to the last 5 JS errors/unhandled rejections on the page since it loaded,
                       each {at, text}. Cleared on page load, not persisted across page loads.
```

**Cut from `meta` before it reached this spec** — considered, rejected as not
worth a column: scroll position, dwell time on page before reporting, pixel
ratio, colour-scheme preference, connection speed, online/offline status,
time zone. None of them gave the team enough to act on.

`group_key` currently uses `answer_id`. With no answer, group by **page +
normalised element text** for pointer reports. **Screenshot-mode reports are
not grouped at all** — settled in `admin-v2-spec.md §5` and
`v2-build-plan-for-agent.md §1`, no longer open.

## 6 The marker pen

- Freehand strokes over the picture. One colour, one width.
- **Undo** and **Clear**, each icon plus word.
- **Available on both Point at the problem and Screenshot**, not
  screenshot-only. This reverses the original v2 spec (§10 below).
- Store the strokes as coordinates in `markup`, and also burn them into the
  uploaded image so the dashboard needs no special rendering.
- Pointer events, so it works with mouse and touch.
- Nothing is required — a report can be sent with no drawing at all.
- No keyboard equivalent exists for freehand drawing. Note that in the
  accessibility statement rather than claiming conformance.

## 7 "What else we send with this"

- Closed by default, on the one shared screen, directly under the comment
  box. The tester does not have to open it; nothing in it depends on the
  tester typing or clicking anything extra.
- Grouped: **This page, right now** (page, referrer, time) · **Your device**
  (screen, window size, device kind, browser, system, input) · **Other**
  (language) · **Errors on this page since it loaded**.
- The console-error capture is the one item here that can genuinely explain a
  bug outright rather than just describing the tester's device. Worth the
  build effort disproportionate to its size in the list.
- Named plainly here so it can also be named plainly in the accessibility /
  storage statement — see open item 6 in §8.

## 8 Open questions — status as of 8 September evening

1. **The picture is small inside a narrow, docked panel.** Drawing on it
   with the marker pen is fiddly, especially for an older tester. **Still
   open, not blocking** — parked for later in the build.
2. ~~Whole page or only what was on screen?~~ **SETTLED: the visible
   window only.** Already built that way in M6b (`capture.ts` captures
   `window.innerWidth × window.innerHeight`). Nothing to decide or change.
3. **Can the tester retake the picture?** **Still open, not blocking** —
   build without it; `v2-build-plan-for-agent.md §8`.
4. **Icon shapes.** **Still open, not blocking** — use the mock's shapes.
5. **Exact wording.** **Still open, not blocking** — use the mock's text; it
   is editable in the string editor afterwards without a rebuild.
6. **Is a browser/OS/device read too much for the accessibility statement?**
   Still open — a wording question for the statement, not a build question.
   Now sharper, because §11 removes the tester's ability to decline the
   picture: the statement must say plainly that a picture of the visible
   window is always sent.
7. **How long do we keep the last few console errors?** **SETTLED: 5,
   cleared on page load.**
8. ~~Grouping for screenshot-mode reports?~~ **SETTLED: never grouped.**
   `admin-v2-spec.md §5`.

## 9 Effort

| Work | Estimate |
| --- | --- |
| Mode chooser, docked panel structure (header/body/footer, transitions) | 1 day |
| One shared screen — picture, comment box, removing the five options | Half a day |
| Drawing the element box into the image | Half a day |
| Marker pen, both modes | 1 day |
| `meta` capture, including the console-error listener | Half a day |
| Dashboard, issue titles and CSV following the payload change | Half a day |
| Tests, and the quality gate on all of it | Half a day |

**About four days** — one more than the 8 September estimate, mainly the
docked-panel structure and the `meta` capture, neither of which existed in
the first pass of this spec. Capture, upload and storage are **not** in this
estimate because they are already built (§3, §11).

## 10 Recorded, without argument

These were raised and settled — some more than once — on 8 September.
Written down so the reasons are on the record if the round goes badly, not to
reopen them.

- The five fixed sentences existed because research on elderly and
  non-native readers found an empty box produces silence. Free text replaces
  them.
- Randomised option order was the one thing no competitor in a 21-tool scan
  could do, and the strongest argument for building rather than buying Ybug
  at €47/month. It is now gone.
- A freehand marker pen was previously declined on the grounds that it needs
  fine motor control and has no keyboard equivalent. It is now in — and,
  after a same-day reversal, in **both** modes rather than screenshot-only.
- Asking "Pointer or Screenshot?" up front puts a technical choice in front
  of the user before they can report anything. Kept anyway, worded in plain
  words with an icon each rather than the technical names.
- **The comment box was made required, same day, after first being specced
  as optional.** The original v1 decision — "free text, kept, optional,
  never depended on" — was made because optional free text loses roughly 41%
  of responses, and the project's own market research criticised Ybug
  specifically for making its comment field mandatory by default. Requiring
  it here reverses both of those. The counter-argument, also on the record:
  v2 has no fallback answer sentences left to fall back on, so an optional
  comment on an otherwise wordless report is a picture with nothing attached
  to it at all. Vishnu's call, 8 Sept, made knowingly.
- The picture was originally taken only after the tester pressed OK on a
  separate comment step. Reworked same day so the picture, the pen and the
  comment box are all one screen — no extra tap, and the tester sees exactly
  what gets sent before they send it.

## 11 The consent screen is removed — decided 8 September evening

**What was built in M6b:** after the picture was taken, the widget showed a
screen — *"Here is a picture of what you saw. This helps us understand the
problem. You can leave it out if you would rather not."* — with two buttons,
**Include this picture** and **Don't include it**. A tester could send a
report with no picture.

**Decision: that screen goes.** The picture is always sent. The tester sees
it on the one shared screen before pressing Send, and that is the agreement.
A tester who does not want it sent cancels the report.

Vishnu's call, asked and answered plainly, 8 Sept evening. What it costs, on
the record: a tester loses the ability to report a problem *without* a
picture of their screen, and the accessibility/storage statement can no
longer say the picture is optional (§8 item 6).

**What this does and does not change in the code:**

- Remove the consent step from the widget flow, and the four config strings
  behind it (`consentQ`, `consentSub`, `btnConsentInclude`,
  `btnConsentExclude`) along with their rows in the string editor.
- **Do not touch `strip_clone()` or anything else in `capture.ts`'s privacy
  path.** Removing the tester's choice makes that stripping more important,
  not less — it is now the only thing standing between a tester's typed
  personal data and a stored picture.
- **`reports.screenshot_key` stays nullable.** This is not a contradiction
  of the decision. A capture can still fail on its own (an old browser, a
  hostile CSS rule, a timeout) and `agent-rules.md §1.11` forbids a failed
  screenshot from blocking or failing a report. So: the tester is never asked
  and never chooses, and a report always tries to carry a picture — but a
  report whose capture failed is still sent, and shows in the admin queue as
  a report with no picture. See `v2-build-plan-for-agent.md §4`.
