# Plan — capture speed, marker pen quality, UI pass

Written 10 September 2026, after the first successful live end-to-end test
on `halle-dev.webflow.io`. Three things Vishnu raised, in his order:

1. The picture takes far too long to appear after clicking.
2. The marker pen line looks bad.
3. Both UIs (widget and admin) need a proper page-by-page pass, done
   together with Vishnu rather than handed over wholesale.

**This is a plan, not a task brief.** §1 and §2 are ready to become agent
tasks once Vishnu signs off the decisions in §4. §3 is deliberately NOT an
agent task — it is a way of working.

---

## 0. Where the evidence comes from

Not guesswork. Three sources:

- **The live trace** from the real Contact page
  (`window.__halleCaptureLog`): clone built in 11ms, first pass done at
  5,846ms, captured at 11,576ms, 29,052 bytes, 71 of 82 images kept.
- **`src/widget/src/capture.ts`** read directly (the repo wins over any
  doc — standing project rule).
- **`src/widget/src/marker-pen.ts`** and the review-screen section of
  **`src/widget/src/app.ts`** read directly.

The viewport-scoping fix from the previous round is confirmed working: 71
images instead of 1,170 requests. What follows is what is left.

---

## 1. Why the picture is slow, and what to do about it

### 1.1 Half the wait is a Safari workaround being paid by everyone

`capture.ts` `capture_screenshot()` calls `capture_once()` **twice**, keeps
only the second result, and does this on every browser. The code comment
says why: Safari's first DOM-to-image capture is documented as blank across
this whole family of libraries, and the author chose "capture twice
everywhere" over sniffing the browser, reasoning that "a redundant second
capture is cheap".

That reasoning was measured on a test fixture where a capture took 98ms.
On the real page a capture takes ~5.8 seconds. The trace shows it exactly:
first pass done at 5,846ms, final result at 11,576ms. **Roughly half of the
tester's wait is a second capture that only Safari needs.**

Options, best first:

- **(a) Only double-capture on WebKit-not-Chrome.** modern-screenshot
  already detects this internally; the widget can do the same two-line
  check. Expected: ~11.6s → ~5.8s on Chrome/Firefox/Edge, unchanged on
  Safari. Risk: a browser-detection check is one more thing that can be
  wrong — but the cost of getting it wrong is a blank first picture on an
  unusual browser, which the existing "never block a report" rule already
  handles gracefully.
- **(b) Reuse one context across both passes.** Each `domToBlob()` call
  currently builds its own fresh context, so pass 2 re-fetches and
  re-embeds every image and font from scratch with no cache sharing.
  modern-screenshot supports creating a context once and reusing it.
  Expected: pass 2 becomes near-free even where it is still needed.
  Risk: lower than (a); no browser detection involved.
- **(a) and (b) together** is the strongest outcome and they do not
  conflict.

### 1.2 Fonts are probably the next biggest cost — but measure first

modern-screenshot walks every stylesheet on the page and inlines each
`@font-face` as a data URL, fetching the font files to do it. A Webflow
marketing page typically carries several large font files. The widget
currently passes no font option at all, so this runs in full.

Three settings exist, in descending fidelity: leave as-is; `font: { minify:
true }` (subset to only the glyphs actually used); `font: false` (skip web
fonts entirely — the picture renders in fallback fonts, so text in the
screenshot may not look exactly like the real page).

**Do not pick one blind.** Instrument first: add timing to the trace around
the font-embedding step and the asset step separately, run it on the real
Contact page, and see the split. Then choose with a number in hand. This is
the same mistake the "second capture is cheap" comment made, and it should
not be repeated.

### 1.3 The tester is shown the word "(no picture)" while it is working

Separate from raw speed, and arguably worse. `app.ts` opens the comment box
after `CAPTURE_FIRST_PAINT_MS = 3000` whether or not the picture is ready —
which is correct behaviour, it stops the tester staring at nothing. But the
picture-less state renders the literal text **"(no picture)"**, which reads
as *"this is broken"*, for up to nine more seconds, right before the
picture silently appears.

Fix: while the capture is still in flight, show a "Taking a picture of the
page…" state (wording editable in admin, like every other string), and fall
back to today's "(no picture)" wording only once the capture has actually
failed or the 12s budget is spent. This costs nothing in speed and removes
most of the *felt* problem.

### 1.4 The capture could start earlier — CONSIDERED AND DECLINED

**Vishnu's decision, 10 September: no. Keep taking the picture after the
click, as the spec says. Do not build this.** Recorded here with the
reasoning so it is not proposed again from scratch.

Today the picture is taken **after** the tester clicks the element, so the
entire wait sits in front of them doing nothing.

The picture is of the visible window, and the element box is burned in
afterwards from coordinates — so the capture does not actually need to wait
for the click. It could start the moment the tester picks "Point at the
problem", while they are still moving the mouse to find what is wrong. On a
real page that is typically several seconds of free time.

**This changes what `widget-v2-spec.md` describes** ("the picture is taken
immediately on click"), so it is Vishnu's decision, not the agent's. Two
things must be handled if we do it:

- If the page **scrolls** between choosing the mode and clicking, the
  early picture no longer matches what the tester is pointing at. Detect
  scroll and re-take. That re-take costs the tester the same wait they have
  today, so this is strictly better, never worse.
- The widget's own overlay (the grey wash and the green hover highlight)
  must not appear in the picture. The capture already works from a pruned
  clone, so this is checkable, not assumed.

### 1.5 Smaller wins, worth doing only if the above is not enough

- Images that are inside the viewport but invisible (`visibility: hidden`,
  `opacity: 0`, a carousel slide parked behind a transform) are still
  fetched and inlined. Pruning those extends the existing off-screen prune.
- `WEBP_QUALITY` is 0.8 and `CAPTURE_SCALE` is 1 — both already
  conservative. Leave them alone; dropping quality further degrades the
  bug report for very little speed.

### 1.6 What "good" looks like

Target: **picture visible within about 3 seconds of the click on a real
Webflow page**, with a truthful "taking a picture" state in the meantime.
(a)+(b) alone should get roughly halfway there from 11.6s; §1.4 hides most
of what remains.

---

## 2. Why the marker pen looks bad — six separate defects

All six are in `marker-pen.ts` and the review-screen part of `app.ts`.
They are independent; the first two are the ones you actually see.

### 2.1 The drawing canvas is at half resolution on a Retina screen

`app.ts` sets `canvas.width = img.clientWidth` — the canvas's backing store
is sized in **CSS pixels**, while a Mac (or any modern phone) has two or
three device pixels per CSS pixel. So every line is drawn at half
resolution and then stretched up by the browser. That is exactly what makes
a line look soft, fat-edged and blocky.

Fix: size the backing store by `devicePixelRatio` and scale the drawing
context to match, keeping the CSS size unchanged. Standard, well-understood,
low risk.

### 2.2 What you draw is not what gets sent

The preview canvas is the size of the **displayed** image (a few hundred
pixels wide, inside the panel). The real captured image is
**viewport-sized** (~1,800px wide). At send time `app.ts` correctly scales
the stroke *coordinates* up to image space — but `burn_in_markup()` then
draws them with a **fixed `MARKER_WIDTH = 4`**, the same number the preview
used at its much smaller size.

Result: a line the tester drew as a bold 4px mark on a 300px-wide preview
gets burned into the sent picture about six times thinner in proportion.
The picture in the dashboard does not match what the tester drew.

Fix: scale the burned-in line width by the same factor as the coordinates.

### 2.3 The line is a raw polyline with no smoothing

`redraw()` walks the recorded points with `moveTo`/`lineTo` — straight
segments between raw pointer samples. Any hand movement that is not slow
and steady produces visible corners.

Fix: draw the stroke as a smooth curve through the sample points (quadratic
curves through segment midpoints is the standard, cheap approach — a few
lines, no dependency, which matters because the widget has a hard "zero new
dependencies" rule).

### 2.4 Every mouse move redraws every stroke, then draws the last bit twice

`on_pointer_move()` calls `redraw()` — which clears and redraws **all**
strokes — and then separately strokes the newest segment on top of what it
just drew. Two consequences: the newest segment is drawn twice so it looks
denser than the rest, and the work per mouse-move grows with everything
drawn so far, so the line starts lagging behind the finger on a busy
drawing, which cuts corners.

Fix: draw the in-progress stroke incrementally without redrawing the
finished ones (or keep finished strokes on a second canvas). This also
removes the double-draw.

### 2.5 Fast strokes lose their samples

The pen listens to `pointermove` only. Browsers batch pointer movement to
the frame rate, so a quick flick loses the points in between.
`getCoalescedEvents()` exists precisely for this and returns the full
high-frequency sample set for each event.

Fix: use it where available, fall back to the single event where not.

### 2.6 A tap draws nothing

`end_stroke()` discards any stroke with fewer than two points, so tapping a
spot — a natural way to say "here" — leaves no mark at all, with no
explanation.

Fix: render a single-point stroke as a dot.

---

## 3. The UI pass — how we work through it together

Vishnu's instruction: *"the UI work in both the widget and the panel, we
will work page by page together."* So this section is a method and an
inventory, not a brief to hand over.

### 3.1 Every screen there is

**Widget (what a tester sees)** — 6 screens:

1. The launcher button ("Report a Bug")
2. The two-choice menu (Point at the problem / Screenshot)
3. The pointer overlay (grey wash, hover highlight, "Click on the part
   that did not look right")
4. The review screen (picture, marker pen, Undo/Clear, comment box,
   Send/Cancel) — **the most important screen in the product**
5. The "sent" confirmation
6. The expired-link notice (new, built this round, never reviewed on screen)

**Admin panel** — 10 screens:

1. Login
2. Overview (new)
3. Queue
4. Tracked items
5. All reports
6. Report detail (picture viewer, Bug/Delete)
7. Pages
8. Page detail
9. Testers
10. Wording

### 3.2 The method, per screen

For each screen, in this order:

1. Vishnu opens it and screenshots it as it really is.
2. We look at it together and write down what is wrong — specifics, not
   "make it nicer".
3. Claude turns that into a small, precise brief for that one screen,
   including the design-system values it must use.
4. The agent changes only that screen.
5. Vishnu re-opens it and confirms, or we go again.

**One screen per pass. Nothing else touched.** The reason is on record:
this project's own history shows large multi-screen rebuilds shipping
visual defects that only appeared when someone actually looked at the
rendered page (four were caught that way last round, by screenshotting).
Small passes with a real pair of eyes at the end catch those before they
land, and keep any single change easy to undo.

### 3.3 Order suggested

Widget review screen first (it is what every tester spends their time on,
and §2's marker work lands there anyway), then the rest of the widget in
flow order, then the admin screens starting with Queue and Report detail
(the two the team will actually live in day to day).

Vishnu picks — this is only a suggestion.

---

## 4. Decisions — all four settled, 10 September

1. **Start the picture earlier?** (§1.4) — **NO.** The picture keeps being
   taken after the click, as `widget-v2-spec.md` says. Do not build the
   early-capture behaviour.
2. **Font fidelity vs speed** (§1.2) — **MEASURE FIRST, THEN ASK.** The
   agent instruments and reports the numbers; it does not change any font
   setting on its own. Vishnu decides once the split is known.
3. **Marker pen thickness** (§2.2) — **Fix the mismatch, keep today's
   visual weight.** The sent picture must match what the tester drew; the
   line should not get bolder or thinner than it looks in the preview now.
4. **First screen of the UI pass** (§3.3) — **the widget review screen**
   (picture + marker + comment + Send).

---

## 5. Sequencing

- **First**, §2 (marker pen) plus §1.1 and §1.3 — six clear marker bugs,
  the wasted second capture, and the misleading "(no picture)" wording.
  All bugs, no decisions attached. Written up as
  `docs/agent-task-marker-and-capture-speed.md`.
- **Alongside**, the §1.2 font measurement — numbers reported back, no
  change made.
- **Then**, §3, one screen at a time, starting with the widget review
  screen, for as long as it takes.
- **Not doing**: §1.4 (declined), §1.5 (only if §1.1 proves insufficient).

Nothing in §1 or §2 should be committed without the usual rule: no commit,
no push, without Vishnu's word at the time.
