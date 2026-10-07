# Template — araCreate deck

**The single source for araCreate slide styling.** Consolidated 1 September
2026: the parallel `ui_kits/deck/` kit (a React slide library, a navigable
sample deck and fourteen specimen cards) was removed and every slide type it
held that this template lacked was merged in. There is no second place to look.

Ten slides, 1280×720, in the approved order:

| # | Slide | Background |
| --- | --- | --- |
| 01 | Title — 105px statement, one phrase in canvas | gold |
| 02 | Story — heading, subheading, three justified paragraphs | canvas |
| 03 | Statistics — eight figures on a four-column grid | canvas |
| 04 | Section divider — one word, one supporting line | gold |
| 05 | Contact — address and channels in two columns | gold |
| 06 | Service vertical — blurb one side, service domains the other | canvas |
| 07 | Projects — twelve dashed cards, six across | canvas |
| 08 | Case study — impact prose and eight figures | canvas |
| 09 | Testimonials — three dashed quote cards | canvas |
| 10 | Values — two-by-two, gold rule under each | canvas |

Delete the slides you do not need. Slides 06–10 repeat their structure per item,
so a shorter deck is a matter of removing `<div>`s, not rewriting styles.

**Two things to change when you copy this into a consuming project**

1. `ds-base.js` — point `base` at the bound design-system folder.
2. Asset paths — the `../../assets/…` references become `<base>/assets/…`.

**The rules the deck depends on**

- **Two backgrounds only:** the slide canvas `--ac-surface-slide` (the page
  canvas `#f6f6f6`; the printed deck's `#f3f3f1` was unified on it 1 Sep 2026) and Golden Sun `--ac-surface-accent`. No dark
  slides.
- **Every size is a token with a fallback.** `ds-base.js` loads `tokens.css` and
  `styles/deck.css`; the slides read `--ac-slide-title`, `--ac-slide-heading`,
  `--ac-slide-tagline`, `--ac-slide-prose`, `--ac-slide-figure`,
  `--ac-slide-label`, `--ac-slide-mark`, `--ac-slide-pad` and `--ac-slide-foot`
  through `var(--x, 105px)` so they paint before the stylesheet arrives and
  retune from one block once it has. No raw colour appears outside a fallback.
- **The Poppins weight scale** (see `tokens/typography.css`): 200 body
  paragraphs, 300 subheadings and labels, 400 the 88–105px display titles, 500
  emphasis and numbers, 700 section headings. Never 600.
- **Geometry is fixed and does not vary by surface:** 67px side gutters, 30px top
  padding, a 90px footer on every slide. The running head sits top-right at
  33/64 with a fixed `min-width` so it does not shift at slide 10. See
  `docs/deck-conventions.md`.
- **Footer on every slide:** wordmark bottom-left (46px chip mark on gold — the
  chip carries its own padding — and 23px bare mark on canvas), optional centred
  caption, triple chevron bottom-right at one size on every slide, graphite on
  gold and gold on canvas. It is one `<g id="ac-chev">` in a `<defs>` at the top
  of the template, referenced by `<use>` from all ten footers; do not paste the
  path again.
- **The chevrons point forward, except on the closing slide.** `>>>` reads as
  "carry on"; the last slide mirrors them to `<<<` for "finished"
  (`transform:scaleX(-1)`). Exactly one slide in a deck is mirrored — if you
  move or add a closing slide, move the mirror with it.
- One slide, one idea. If a slide needs a scrollbar it is two slides.
- Three paragraphs at most. Four is a document.
- Canvas on Golden Sun is 1.67:1 and fails at every size — it is a highlight for
  a phrase inside a 105px title, never body copy. Graphite on gold is 4.45:1,
  which holds at 19px/500 and larger. Gold is never text: the case-study
  figures are graphite at 500, like every other figure.
- Every figure comes from the brand facts, plus signs included.
- Photography is duotint only.
