# ACCESSIBILITY

What this system guarantees, what it leaves to you, and the numbers behind both.

The source repository runs seven gates, two of which are accessibility gates
walking every rendered text element on 26 pages. Those gates do not exist in this
project — there is no `make test` here. **What follows is the contract they
enforce, plus every measurement taken in this project by hand.** Where a figure
appears below, it was measured on a rendered page with translucent backgrounds
composited before measuring, not calculated from token values.

---

## The three things you may not undo

Carried verbatim from the source repository, and the reason each exists:

1. **The global `:focus-visible` ring.** 2px solid ink, 2px offset, white
   on an inverse surface. It is the only way a keyboard user knows where they
   are. Never remove it, never replace it with a colour change alone.
2. **Reduced-motion handling.** Every animation stops for a visitor who has asked
   their operating system for less. The marquee becomes a static scrollable row,
   scroll reveals resolve to visible, the skeleton sweep stops. Nothing is ever
   *hidden* from someone who cannot receive an animation.
3. **The inverse-surface context.** A dark band re-points every colour token
   inside it. This is why no component needs a dark variant — and forgetting it
   produces exactly one bug, which this project hit twice. See the ledger below.

---

## Contrast

**Golden Sun is never text.** `#f9bf3b` measures 1.67:1 on white *and* white
measures 1.67:1 on it — it fails in both directions at every size, including a
105px slide title. There is deliberately no token for gold text. Gold is
something you put *behind* type; ink on gold is 9.51:1. On the graphite surface
gold is legible at 4.58:1, which is the only place it becomes text.

### Light theme

| Pair | Ratio |
| --- | --- |
| Ink `#222222` on canvas | 14.72:1 |
| Graphite on canvas (body copy) | 6.90:1 |
| Muted `#6f6f6f` on canvas | 4.61:1 |
| White on the graphite band | 7.44:1 |
| Ink on Golden Sun | 9.51:1 |
| Error `#b3261e` on canvas | 6.05:1 |
| Success `#186a43` on canvas | 6.11:1 |

`#6f6f6f` is a **derived accessible value, not a brand colour.** The live site
used `#969696` for muted text, which measures 2.69:1. `#6f6f6f` is the lightest
grey that still passes.

### Dark theme

| Pair | Ratio |
| --- | --- |
| Pair | On the page | On raised `#4f4f4f` |
| --- | --- | --- |
| White (headings) | 7.44:1 | 8.36:1 |
| `#d8d8d8` (body) | 5.22:1 | 5.87:1 |
| `#cecece` (muted) | 4.68:1 | 5.26:1 |
| Error `#fdd8d4` | 5.65:1 | 4.77:1 on its own 8% wash |
| Success `#c3e9cf` | 5.62:1 | 4.76:1 on its own 8% wash |

**Every figure is measured on the page *and* on the lightest surface the text
can land on.** Skipping the second column is how the first pass at this theme
shipped a 3.53:1 card. A dark theme normally lifts a raised panel by going
lighter; graphite is mid-grey, so the headroom is all below it and both raised
and subtle step *down* (graphite tinted with ink — dark grey, not black). The
signature edge does the separating, which is this system's rule for a panel at
rest anyway.

Body copy is **not** pure white. 7.44:1 is more contrast than long-form reading
wants — it glares. Headings take the white.

**The dark page is graphite `#555555`, not black.** Black backgrounds left the
palette on 20 August 2026, and re-measuring against the lighter page moved every
figure in this table. Muted text went from `#a3a3a3` (2.99:1 on graphite —
failing) to `#cecece`, and the feedback pair had to go lighter twice: `#f28b82`
and `#81c995` measure 2.95:1 and 3.01:1 here, and the replacements passed on the
page but not on their own 13% wash. Neither the light pair nor either interim
dark pair survives; the dark theme still introduces two hues and only two.

---

## Target size

**Every interactive control is at least 44px in both directions**, applied as
padding and `min-height`, never by growing the type: a 12px label inside a 44px
target is fine, a 12px target is not.

- Checkboxes and radios stay 18px. The `<label>` carries the target, which is
  why the control is wrapped in one.
- Standalone links — navigation, footer, breadcrumb, pagination — carry the
  floor. Links *inside running text* are exempt; one cannot be 44px tall without
  wrecking the line.
- Adjacent targets get at least 8px between them, the smallest separation that
  measurably reduces mis-taps.
- Compact density lowers the floor to 36px **only on `pointer: fine`**. On touch
  it stays 44px, because a dense table on a phone is still operated by a finger.

Link size was raised from 12px to 15px on 20 August 2026. Type size never made a
link tappable; both problems were real and both are fixed.

---

## Keyboard

| Component | Keys |
| --- | --- |
| `Tabs` | ← → move between tabs, Home / End jump to the ends |
| `Dropdown` | Escape closes **and returns focus to the trigger** |
| `Modal`, `Drawer` | Escape closes, focus is trapped — both from `<dialog>`, not rebuilt |
| `Accordion` | Enter / Space toggles — native `<details>` |
| `Combobox` | ↓ opens and moves, ↑ moves, Enter picks, Escape closes |
| `DatePicker` | Tab through days; every day carries a full `aria-label` |
| `SegmentedControl` | arrow keys — real radios underneath |
| `Slider`, `NumberInput` | arrows, Home / End — real inputs |
| `Switch`, `Choice` | Space toggles |
| Mobile nav | Escape closes and returns focus to the button |

A menu that closes and dumps focus at the top of the page is worse than one that
does not close.

---

## Announcement

- **`aria-live="polite"`** on the toast region, deliberately. `role="alert"` is
  reserved for errors that block progress, because it interrupts.
- **`aria-busy`** on loading buttons (label stays in the DOM, hidden visually, so
  the width does not jump) and on skeleton containers, which also carry an
  off-screen "Loading". Individual skeleton bars are `aria-hidden` — the state is
  announced once, not as eighteen empty boxes.
- **`aria-current`** marks the current page or step. Never an invisible overlay:
  the template this system replaced covered the current-page link with a
  click-blocker, and because every downloaded page was `index.html`, 27 links
  matched at once and the whole menu died.
- **Every icon declares itself** — `aria-hidden="true"` for decoration, or
  `role="img"` with a label for content. The system outlines any that declares
  neither, so the omission is visible.
- **Every table has a caption or a label**, even a visually hidden one, and every
  `th` has `scope`.

---

## Colour is never the only signal

| Where | The other signal |
| --- | --- |
| Badges | a word, always |
| Status dots | sit beside text |
| Completed steps | a tick as well as the fill |
| Selected table rows | a gold left mark as well as the tint |
| KPI deltas | an arrow as well as the colour |
| Required fields | an asterisk |
| Absent pricing features | shown with a cross, not hidden |
| Combobox matches | weight, not colour |
| Slide title emphasis | weight, not colour |

Roughly one man in twelve cannot reliably distinguish the green from the red.

---

## The measured ledger — bugs found in this project

Recorded because each one passed everything automatic and was caught by looking.

| What | Measured | Fix |
| --- | --- | --- |
| Side-nav labels: graphite on the dark band | **1.00:1** | `.ac-app__nav` re-points the full token set |
| Side-nav current item: ink on the dark band | **1.55:1** | same |
| `.ac-banner` and `.ac-bulk-bar` children naming their own tokens | same class of fault | both re-point |
| Nav `--ac-text-heading` pinned to white, in the dark theme where the inverse surface flips to light | white on canvas | both text tokens follow `--ac-text-inverse` |
| Muted `#6f6f6f` on the *tinted* current nav row | **4.18:1** | graphite there — 6.20:1 |
| Table select-row checkbox rendered as the browser's blue default | not a contrast fault, a fidelity one | styled in `app.css` |

**Final sweep: 22 elements across both themes, 0 failures.**

The first two are bug 5 from [`decisions.md`](decisions.md) — *"the inverse
surface never re-pointed the surface tokens… the step markers rendered at
1.08:1"* — reproduced in a new file on the first attempt. Four gates passed for
an entire build with that fault in the foundations. Its note reads: **a system
nobody has built with is a guess.**

---

## What is NOT verified

Stated plainly, because an accessibility claim without its limits is worth less
than none:

- **No automated gate runs in this project.** The source repository's seven gates
  live there. Nothing here re-runs on every change.
- **No screen-reader pass.** Measurements catch missing names and broken
  references. They cannot tell you a label is confusing or an order is illogical.
  That needs a person and VoiceOver, and it is the source repo's own outstanding
  item too.
- **Contrast measured on the pages built here**, not exhaustively across every
  component in every theme, surface and state combination. The 22-element sweep
  covered the app surfaces in both themes; the marketing surfaces inherit values
  the source repository measured across 3,620 elements.
- **The compact density has not been contrast-swept.** It changes spacing, not
  colour, so it should be neutral — but "should be" is how the 4.18:1 got in.

**If you take one instruction from this file:** screenshot every new component
and read it. Both bugs above, and the card-body-bold bug the source repo records,
were found that way and by nothing else.
