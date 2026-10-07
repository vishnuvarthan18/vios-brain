# araCreate — Deck UI kit

Sample pitch deck recreated from `uploads/aracreate-deck.pdf` (araCreate Deck, 04.02.2026). Slides are 1280×720 and reuse the brand's foundations — Poppins throughout (Monument Extended is reserved for the logo wordmark), Golden Sun accent, warm light-gray and golden backgrounds.

## Files
- `index.html` — navigable deck (arrow keys / on-screen controls, position persisted). The main deliverable.
- `slides.jsx` — slide component library, all exported to `window`:
  - `SlideFrame` — shell (variant `canvas|gold`, running head, footer logo + caption + chevron)
  - `Header` — Poppins bold title + Poppins-light tagline; `Chevrons` — triple-chevron motif
  - `VerticalSlide` (`name="Engineering|Manufacturing|Media"`)
  - `ProjectGridSlide`, `CaseStudySlide`, `TestimonialsSlide`, `ValuesSlide`, `ContactSlide`
- `card-*.html` — individual slide specimens for the Design System tab.

**System**
- **Two backgrounds only:** warm light-gray canvas (`#f3f3f1`) + Golden Sun. No dark slides.
- **Page headers** are Poppins bold, top-left, with a Poppins-light tagline beneath (the signature ellipsis line, e.g. "Where … …"). No numbered eyebrows. (Monument Extended is reserved for the logo wordmark and never appears as a heading.)
- **Section dividers, title and closing** are golden, with one huge Poppins-light word/phrase; the title highlights "mind to market" in white, the closing highlights "ideas".
- **Stat / impact numbers** are Poppins-semibold in Golden Sun.
- **Content slides** use two columns with a golden 2px underline under each column label; project/testimonial cards use a 1.5px dashed border.
- **Footer** on every slide: brand logo bottom-left (default lockup on canvas, white-on-graphite on gold), centered caption, triple-chevron (›››, reversed ‹‹‹ on the closing) bottom-right.

All content (stats, verticals, projects, Jodel case study, testimonials, values, contact) is transcribed from the deck — nothing invented.
