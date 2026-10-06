# UI kit — the araCreate deck

Ten slide layouts, 16:9, built from the same tokens as everything else. The deck
is a stylesheet (`css/deck.css`), not a presentation framework: no transitions,
no builds, no fragments. `js/deck.js` adds arrow-key navigation and fullscreen
and is optional.

`index.html` is the whole deck. Each `*.slide.html` is one layout on its own,
tagged so it appears in the Design System tab.

| Layout | Surface |
| --- | --- |
| Title | gold |
| Section opener | graphite |
| Statement | canvas |
| Statistics | canvas |
| Three verticals | canvas |
| Split with art | pale gold |
| Split with photography | canvas |
| Values | graphite |
| Ecosystem | canvas |
| Closing | gold |

**How it scales.** A slide is `container-type: size` and every size inside it is
in `cqw`, so the same markup is a thumbnail, a projected slide and a PDF page.
1280px is the reference width.

**Rules that the deck gate enforces in the source repository.** Pair the
modifier with the surface class — `ac-slide ac-slide--gold ac-surface-accent`;
without it a heading on gold renders at 4.45:1. One slide, one idea: if a slide
needs a scrollbar it is two slides. Three paragraphs at most. Emphasis in a
title is weight, not colour. Every figure comes from the brand facts, plus signs
included. Slide numbers come from a CSS counter, so never type one.
