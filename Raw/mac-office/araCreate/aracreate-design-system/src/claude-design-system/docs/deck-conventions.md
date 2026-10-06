# Deck conventions

Rules for building a deck on this system: chrome geometry, type sizing, layout and copy
readability. Written down because they are the decisions that otherwise get re-litigated on
every slide.

Adopted 1 September 2026 from Vishnu's working notes. Two things about scope:

- **Sizes here are at the system's 1280 reference**, not the 1920 the notes were written at.
  `styles/deck.css` expresses them in `cqw` — one percent of the slide's own width — so the
  deck is resolution-independent and 1cqw = 12.8px at the reference.
- **`styles/deck.css` enforces what CSS can.** The rest is authoring guidance and cannot be
  automated: copy length, reading level, what a caption says. Where a rule is enforced, the
  class is named.

`acds-template-deck` was **reconciled with the chrome rules on 1 September 2026** — one
footer height, one motif size, one shared motif reference, a non-shifting page marker. Its
ten slides still carry inline styles rather than the classes named below, because a Design
Component styles inline by format, so the class-based rules here are enforcement for pages
that use `deck.css` and *documentation* for the template. Where a value disagrees, this
document is the rule.

---

## Chrome is identical on every surface

A deck alternates surfaces — accent for the title, dividers and the close; canvas for
content. The chrome must not change with the surface. A reader tracking the page marker
across slides should never see it move.

- **One padding value on every slide**, whatever its surface: `2.4cqw` top, `5.25cqw` sides
  (31px / 67px at the reference). Enforced by `.ac-slide__stage`, which does not vary by
  surface. Do not give the title or divider slides their own geometry.
- **One footer height on every slide**: `--ac-slide-foot`, 90px at the reference.
  *Changed 1 September 2026* — gold and graphite slides took a 126px band, which moved the
  standing line and the motif every time the deck crossed a surface. `--ac-slide-foot-lg`
  was removed in v2.0.0.
- **Top left — nothing.** The notes called for a numbered section eyebrow; Vishnu's call on
  1 September was to skip it. The heading carries the section name and the marker sits top
  right alone. `.ac-slide__eyebrow` still exists for a deck that wants one.
- **Top right — page marker.** `.ac-slide__mark`, weight 200, with a fixed `min-width` and
  `text-align: right` so the block does not shift when the number goes from one digit to
  two. The number is a CSS counter, so inserting a slide renumbers the deck.
- **Bottom left — one standing line**, identical on all slides. `.ac-slide__standing`, with
  divider punctuation lighter than the text.
- **Bottom right — the deck motif**, `.ac-chevrons`, one size everywhere.
  *Changed 1 September 2026* — the motif used to grow on gold and graphite to match the
  taller footer. Mirror it on the final slide only: `.ac-slide--end`. `>>>` carries on,
  `<<<` is finished. Exactly one slide per deck.
- **Never set the logo wordmark typeface in chrome.** Monument Extended is the logo and
  nothing else; set in a footer it reads as a watermark. The logo *asset* bottom-left is
  fine and is what the template uses — the objection is to the typeface, not the mark.

**Notes are not footer furniture.** A caveat or footnote goes on its own line above the
footer — `.ac-slide__note` — never in the footer's left slot, where it displaces the
standing line.

## Type

The Poppins weight scale (`tokens/typography.css`) governs a slide as it governs everything
else: 200 body, 300 subheads and captions, 400 display, 500 figures and emphasis, 700
section headings. Never 600.

- **Display and divider titles** are the largest thing on a slide, at **400**, never bold.
  `.ac-slide__title`, 105px. Emphasis inside a title is weight, not colour — white on
  Golden Sun is 1.67:1 and fails at every size.
- **Section headings** are **700**. `.ac-slide__heading`, 71px.
  *This resolves a conflict in the source notes*, which said titles and headings alike are
  never bold. Ara's instruction of 1 September was that section headings are bold, and the
  two rules are about different classes: a 105px display line does not need weight to
  dominate, a 71px heading over a subhead does.
- One heading size for the whole deck. If a heading wraps to two lines, add
  `.ac-slide__heading--step` to **that heading** rather than retuning the deck's ladder.
- Subheads sit clearly below the heading and use **300**. `.ac-slide__tagline`, 36px.
- Body and card prose one step below that, at **200**. `.ac-slide__prose`, 29px.
- Figures, tier names and small labels take **500** — weight carries emphasis on a slide,
  not colour.
- **Floor: 16px at the reference** (24px at 1920). Nothing below it except the page marker,
  which sits at 14px.
  *Decided 1 September 2026, since the notes left it open:* the floor binds **prose** —
  anything a reader reads as a sentence. **Card metadata is exempt**: a category, a country,
  a date or a role in a dense grid may go to 13px, because those are scanned, not read, and
  the alternative is fewer items on the slide than the content needs. Slide 07 of the
  template (twelve project cards, 15px titles and 13px meta) is the case this exemption
  exists for. Never solve an overflow in *prose* by going under the floor — shorten the copy.

## Colour

- Two background colours across the whole deck: canvas and one accent. Accent for the title,
  section dividers and the close; canvas for content. No dark slides.
- One text colour. Do not tint prose to create hierarchy — use size and weight.
- Muted grey is for secondary labels only, and must be the token that passes contrast.
- **Accent colour never carries small text.** Golden Sun carries display text or no text;
  anything below 19px goes on Golden Soft `#fdf3d8`, where graphite measures 6.63:1. See
  `docs/accessibility.md`. Or use gold as a 2px rule above a column, which is what the
  template does.

## Layout

- **Left-align every grid on the slide's own column.** Enforced: `.ac-slide__grid` sets
  `justify-items: start`.
- Bottom content blocks clear the footer by roughly half the footer's padding. Do not pin a
  block with `align-items: flex-end` — it collides with the chrome at the first copy change.
- When content does not fill the slide, centre the block in the space that is left rather
  than leaving a void between the subhead and the content. `.ac-slide__stage--centred`.
- **Sibling items occupy the same number of lines.** Write the descriptions so all of them
  wrap the same way — if one needs two lines, give them all two. The rule is *consistency*,
  not brevity: four items where three are one line and one is two lines looks like a bug,
  and the repair is matching the others up, not cutting the long one down.
- **The rule binds copy you own, not copy you are quoting.** Two kinds of text are off
  limits: an attributed quote, where padding a line would put invented words in a named
  person's mouth, and a stated fact — a country, a category, a date, a client name — where
  matching line counts would mean inventing one. For those, anchor the block instead:
  `justify-content: space-between` on a card body, or `margin-top: auto` on the element that
  should sit to the bottom, so uneven copy still lines up across siblings. Give the anchored
  block a FIXED height sized for the worst case, not a `min-height` — a min-height still
  grows with its own copy, and in a card the media block above it absorbs the difference, so
  the placeholders come out at different heights even though the cards match. Slides 07 and 09
  of `acds-template-deck` are both this case; slide 10, which is araCreate’s own copy, is
  matched by rewriting.
- Cap card and sketch grids at an explicit column width instead of `1fr` when the row must
  not grow past the slide.

## Copy and readability

Target an average reader, around grade 7–8. Long sentences on a slide are read aloud badly
and skimmed worse.

- Short sentences, one idea each. Cut the wind-up.
- Replace insider vocabulary with the plain equivalent. Anything a customer would not say in
  a shop does not belong on the slide: prefer "ask for a price" to "enquiry flow", "rules for
  customer artwork" to "artwork intake standard".
- Avoid hyphenated trade compounds that are hard to say aloud. Say what the thing is instead.
- Use examples the audience would recognise from real work, not invented illustrations.
- Keep prices out of slide copy. Ranges age badly and invite negotiation mid-presentation.
- Keep approval lines, dates and legal entity names off the title slide.
- Where a name has a fixed casing, write it that way every time, and rewrite any sentence
  that would start with a lowercase-initial name. "araCreate" never opens a sentence.
- Every figure comes from `docs/brand-facts.md`, plus signs included.

## Images

- Reference and placeholder frames use a drop-in image slot with a placeholder saying what
  belongs there, and a distinct id so a dropped file survives reload.
  *Not applied to the template on 1 September 2026* — its grey blocks stay flat pending a
  rebuild.
- Reference frames fit **contain**, not cover. `.ac-slide__art`. A cover crop will eventually
  cut through the part of the subject the slide is about; `--cover` exists for art that fills
  half a split, where there is no subject to lose.
- Caption a reference with what it demonstrates, not what it is.
- Photography is duotint.

## Section rhythm

*Not adopted on 1 September 2026 — Vishnu kept the template's ten-slide shape.* Recorded
because the notes call for it and a longer deck will want it:

- Open with a title slide, then a contents slide naming the sections.
- Give each section an accent divider carrying its number, its name, and one trailing
  half-sentence in the brand's ellipsis style.
- A divider is not a content slide: a title and one line, nothing more.
- Close on the accent surface with the ask, in three short items at most. The template
  closes on contact details instead.
