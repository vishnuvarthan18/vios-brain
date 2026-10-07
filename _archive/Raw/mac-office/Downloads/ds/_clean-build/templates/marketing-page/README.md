# Template — araCreate marketing page

A full araCreate section page: sticky header, split hero, marquee band, service
list, dark stats band, values cards, gold CTA band, footer. Every layout is one
of the section families in `css/sections.css`; every figure is a brand fact.

**Two things to change when you copy this into a consuming project**

1. `ds-base.js` — point `base` at the bound design-system folder (e.g.
   `_ds/<folder>` at the project root, `../_ds/<folder>` one level down).
2. Asset paths — the `../../assets/…` references in the page resolve against
   this design system's own tree. In a consuming project they become
   `<base>/assets/…` with the same `base`.

**Then replace, in order:** the hero title and lead, the three service rows
(keep the couplets verbatim — they are part of each vertical's name), the four
value cards, and the CTA. The stat row is already correct and should not be
edited: the `+` is what makes each figure true.
