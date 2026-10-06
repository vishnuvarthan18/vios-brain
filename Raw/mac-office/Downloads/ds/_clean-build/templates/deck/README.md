# Template — araCreate deck

Six of the ten slide layouts, in one 16:9 deck. The full set is in
`ui_kits/deck/`, one file per layout.

**Two things to change when you copy this into a consuming project**

1. `ds-base.js` — point `base` at the bound design-system folder.
2. Asset paths — the `../../assets/…` references become `<base>/assets/…`.

**The rules the deck depends on**

- Pair the surface class with the modifier: `ac-slide ac-slide--gold
  ac-surface-accent`, `ac-slide ac-slide--dark ac-surface-inverse`. The
  modifier paints the background; the surface class re-points every colour
  token inside, headings included. Forget it and a heading on gold renders at
  4.45:1.
- One slide, one idea. If a slide needs a scrollbar it is two slides.
- Three paragraphs at most. Four is a document.
- Emphasis in a title is `<em>`, which sets weight 700 and keeps the ink
  colour. White on Golden Sun is 1.67:1 and fails at every size.
- Slide numbers come from a CSS counter. Never type one.
- Every figure comes from the brand facts, plus signs included.
- Photography is duotint only.
