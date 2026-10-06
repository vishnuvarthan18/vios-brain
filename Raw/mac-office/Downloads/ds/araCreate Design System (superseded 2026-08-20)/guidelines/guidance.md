> **Ported verbatim from the araCreate Design System repository, `docs/guidance.md`.**
> This is araCreate's own record and has not been rewritten. Path references
> (`src/`, `docs/`, `tests/`, `make` targets) point at that repository, not at
> this project.
>
> The component notes here describe the CSS classes. This project also wraps each one as a React component with the same behaviour — see `components/` and each `<Name>.prompt.md`.

---

# GUIDANCE

When to use each component, and — more usefully — when not to. The gallery in
[`components.html`](components.html) shows what each one looks like; this says
what each one is *for*.

Read the three rules first. Most mistakes are one of them.

## The three rules

1. **Never write a raw value.** No hex colours, pixel sizes or timings in a
   component. A missing token is a signal, not an obstacle: use the nearest
   one, or add a token deliberately. `make adhere PATH_=<your project>` checks
   this on any page, anywhere, and `src/manifest.json` lists every token and
   class there is.
2. **Name the job, not the colour.** `--ac-action-bg`, never `--ac-yellow`.
3. **Yellow is never text.** It is 1.67:1 on white. There is deliberately no
   token for yellow text.

---

## Voice

The system decides how a page looks. This decides how it sounds. A page built
correctly out of these components and then written in the wrong voice is still
off-brand, and it is the failure nobody runs a gate against.

araCreate sounds **professional, clear and friendly** — simple enough for
someone in the middle of a real project to read once and understand. Four
pillars, all four at once rather than one at a time: **innovative** (energetic),
**trustworthy** (has authority), **professional** (dependable), **friendly**
(hospitable).

The shorthand: confident, industrial, human. Engineering precision, warmed by a
gold accent and by talking to people like people.

### Person

**"We" for araCreate, "you" for the client.** Not "clients can" — "you can".
Not "araCreate delivers" in a sentence araCreate is writing; "we deliver".

The master tagline is *Empowering ideas from mind to market*. It is a fixed
string. So are the three vertical couplets — *from Prototype to Production*,
*from Drawing to Delivery*, *from Sketch to Screen*. Those are in
[`brand-facts.md`](brand-facts.md), which is the list of things you may not
reword.

### Casing

**Sentence case for headlines and body.** Not Title Case. A headline reads
"Where complex ideas need broader services", not "Where Complex Ideas Need
Broader Services".

**Wide-tracked UPPERCASE only for buttons, eyebrows and small labels.** That is
what `ac-eyebrow` and `--ac-tracking-wide` are for, and it is why a heading set
in caps looks wrong in this system — caps are already spoken for.

Section eyebrows are numbered: `01 — araCreate Group`, `02 — About`,
`03 — Services`. The number is part of the rhythm of a page, not decoration.

### Sentences

- **Short. One idea each.** Two ideas want two sentences.
- **Say the thing, then stop.** No wind-up, no "in today’s fast-moving world".
- **Concrete over abstract.** "Rapid prototyping through series production"
  beats "end-to-end manufacturing solutions".
- **No jargon the client has not used first.** If they said "PCB", say PCB. If
  they did not, say circuit board.
- Section headlines are the one place the brand is allowed to be aspirational,
  and they trail off with an ellipsis: *Where boundless passion meets performing
  results …*

### Never

- **No emoji.** Not in copy, not as an icon, not in a heading, not as a bullet.
  The brand has an isometric line-illustration language instead — see
  [`assets.md`](assets.md). This is absolute.
- **No exclamation marks** in body copy. Friendly comes from directness, not
  punctuation.
- **No invented facts.** Every number, company name and figure a page may state
  is in [`brand-facts.md`](brand-facts.md). If it is not there, ask; do not
  estimate. `300+ clients` is the figure — never `300 clients`, and never a
  number you worked out yourself.
- **No "click here".** The link text says where it goes. This is also an
  accessibility requirement, since a screen reader can list links out of
  context.
- **No shouting a whole sentence** in caps. Caps are for labels.

### Microcopy

The small strings are where a voice actually lives, because they are the ones a
person reads every time.

- **Buttons say what happens**, in the visitor’s terms: "Send message", not
  "Submit". "Let’s talk about your idea" is the brand’s own CTA and is worth
  reusing.
- **Errors say what to do**, not that something went wrong. "That email address
  is missing an @", not "Invalid input". Never blame the visitor.
- **Empty states say what to do next**, not "No data".
- **Hints go under the field**, phrased as help rather than warning.

---

## Button

**Use** for something that happens — submit, open, save, apply.

**Do not use** a button for navigation. If it goes somewhere, it is a link:
`<a class="ac-btn">`. Middle-click, right-click, "open in new tab" and the
status bar preview all depend on it being a real `<a href>`, and a screen reader
announces "link" rather than "button" so people know what to expect.

- **One primary button per view.** Two things competing for "the main action"
  means neither is. Everything else is `--outline` or `--ghost`.
- **`--danger` is for destructive and irreversible.** Not for "cancel". Cancel is
  `--ghost`.
- **Icon-only needs `aria-label`.** The system draws a red outline round any that
  lacks one, so the omission is visible rather than silent.
- **Loading uses `aria-busy="true"`.** Do not replace the label with a spinner in
  the markup — the label stays and is hidden visually, so the button keeps its
  width and the state is announced.
- Never remove the focus ring.

## Field, input, choice, switch

**Use `ac-field`** for every form control. Label, control, hint and error as one
unit.

**Never label a field with a placeholder alone.** It disappears the moment
someone types, so anyone interrupted mid-form has lost the question. The
accessibility gate fails on this.

- **Hints go under the field, not in a tooltip.** A tooltip is invisible on a
  touchscreen and gone as soon as the pointer moves.
- **Errors say what to do**, not that something is wrong: "That number is too
  short to be valid", not "Invalid input".
- **Required is an asterisk**, not a colour. Colour alone is invisible to a
  colour-blind visitor and silent to a screen reader.
- **Switch or checkbox?** A switch takes effect immediately. A checkbox takes
  effect when the form is submitted. Getting this backwards is why people press
  Save and wonder why nothing happened.
- Do not put more than one idea in one field. Two questions need two fields.

## Badge and tag

**Use a badge** for state — Live, Closed, Starts September.

**Never let colour carry the meaning alone.** Roughly one man in twelve cannot
reliably distinguish the green from the red. Every badge in this system pairs
its colour with a word, and the status dot always sits beside text.

**Do not use a badge as a button.** It is not interactive and does not look
interactive. If it should be clickable it is a tag with a remove control, or a
button.

## Avatar

**Use initials when there is no photograph.** Do not use a generic silhouette for
a real person — initials are more informative and less bleak.

Group avatars overlap deliberately, and the ring is the page colour so they read
as separate on any surface. Cap the group and show a "+13" rather than thirty
faces.

## Icon

**Every icon is one of two things** and must declare which: decoration
(`aria-hidden="true"`) or content (`role="img"` with a label). The system outlines
any icon that declares neither.

**An icon is rarely enough on its own.** A label beside it costs a few pixels and
removes the guesswork. Icon-only is for controls people meet constantly —
close, search, menu.

Icons inherit `currentColor`, so never set their colour directly; set the text
colour of what contains them.

## Card

**Use** for a repeating thing in a list — a course, a project, a person.

**Do not use a card for a single item.** A lone card on a page is a box round
nothing. Use a section with a heading.

- **Make the whole card the link**, not a "read more" inside it. One large
  target, one tab stop. A card containing three separate links is three tab
  stops and a confusing one.
- **`alt=""` on the card image** when the title already says what it is.
  Describing it twice is noise, not accessibility.
- **Six kinds exist** — course, service, article, person, stat, project. If none
  fits, that is worth a conversation before inventing a seventh.
- Cards in a grid should hold comparable things. A grid of unlike items is a
  list wearing a costume.

## Accordion

**Use** for questions with long answers, where most people want one of them.

**Do not hide anything important in an accordion.** Collapsed content is not
read, is weaker for search, and is missed entirely by people skimming. Prices,
delivery times and warnings belong on the page.

Default to letting several open at once — people compare answers.
`data-ac-accordion-single` forces one at a time, which suits a stepper more than
an FAQ.

## Tabs

**Use** for alternative views of the same kind of thing, where nobody needs two
at once.

**Do not use tabs for sequential steps** — that is `ac-steps`. And do not use
them to hide bulk on a long page; people do not find tabbed content they were not
looking for.

Arrow keys move between tabs, Home and End jump to the ends. Panels are only
hidden once the script runs, so a failed script shows everything rather than
nothing.

## Table

**Use** for data with two dimensions — rows and columns that both mean something.

**Do not use a table for layout.** Use `ac-grid`.

- **Every `th` needs `scope`.** Without it a screen reader cannot tell a row
  header from a column header, and the table becomes a wall of cells.
- **Every table needs a caption or a label**, even a visually hidden one.
- **Right-align numbers** with `ac-table__num` so the digits line up.
- **Sorting compares numbers as numbers.** If you build your own sort, do the
  same: a text sort puts 100 before 20, which is the most common table bug there
  is.
- On a phone, wrap it in `ac-table-wrap` and let it scroll. Do not shrink the
  type until it is unreadable.

## Alert and toast

**Alert** for something on the page that stays. **Toast** for something that just
happened and does not need acting on.

**Never put anything necessary in a toast.** It disappears. If the visitor must
read it, it is an alert or a modal.

- `role="alert"` interrupts a screen reader — use it only for errors that block
  progress. `role="status"` waits its turn; that is right for almost everything.
- The toast region is `aria-live="polite"` deliberately.
- Four kinds, and only two new colours: warning reuses the brand yellow,
  information is charcoal. Resist adding a fifth.

## Modal

**Use** when the page must not continue until something is decided.

**Do not use a modal for anything that could be a page.** Modals cannot be
linked to, bookmarked, or found again. They are hostile on a small screen.

Built on `<dialog>`, so focus trapping, Escape and the backdrop come from the
browser. Never rebuild those. Always give it `aria-labelledby` pointing at its
title, and always leave a visible close control — Escape alone is not
discoverable.

## Dropdown

**Use** for actions. **Do not use it for choosing a value in a form** — that is a
`select`, which works with the phone’s native picker and with autofill.

Escape closes it and returns focus to the trigger. A menu that closes and dumps
focus at the top of the page is worse than one that does not close.

## Tooltip

**Use** for something genuinely optional.

**Never for anything required.** Invisible on touch, easy to miss, gone when the
pointer moves. If it matters, it is a hint under the field or text on the page.
This system’s tooltip appears on keyboard focus as well as hover, which most do
not — that makes it better, not sufficient.

## Progress

**Use a determinate bar** when you know the length, and an indeterminate one when
you do not. The percentage lives in `aria-valuenow`; the script mirrors it into
the CSS, so the number cannot disagree with the drawing.

**Do not fake progress.** A bar that reaches 90% and stops is worse than a
spinner.

## Steps

**Use** for a real sequence where order matters. Mark the current one with
`aria-current="step"` and completed ones with a tick as well as the fill — colour
is never the only signal.

## Pricing

Three plans is the practical maximum on a laptop; more than four and nobody
compares them. Mark absent features with `data-ac-absent` and show them rather
than hiding them — an omission the visitor discovers later costs more than one
they saw up front.

## Header and navigation

**Never cover the current-page link.** The template this replaced put an
invisible click-blocker over it; because every downloaded page was
`index.html`, 27 links matched "current" at once and the whole menu died. Mark
the current page with `aria-current="page"` and style it. A link to the page you
are on is harmless.

- The mobile panel uses the `hidden` attribute, not `opacity: 0`, so when closed
  it is genuinely out of the tab order.
- Escape closes it and returns focus to the button.
- Five or six top-level links is the limit. More than that is a menu people scan
  rather than read.

## Surfaces

Four, reachable the same way: `ac-surface-page`, `ac-surface-raised`,
`ac-surface-inverse`, `ac-surface-accent`, plus `ac-surface-subtle`.

**Put the class on the band, not on each thing inside it.** A surface re-points
every colour token within it — text, borders, code, focus ring and the surfaces
themselves. That is why no component needs a dark variant, and why forgetting it
produces exactly one bug: light components with white text.

## Corners

**There are none.** Every radius token is `0`: cards, panels, inputs, buttons,
badges, tags, avatars, switches, step markers, dots, the loading spinner. A
square dotted panel sitting beside a pill-shaped switch and a circular avatar
reads as three different systems on one page.

**Do not add a `border-radius` to a component.** If something needs to be round,
that is a decision about the whole system and it belongs in `tokens.css` — the
five radius values there, and nowhere else.

One consequence worth knowing: **radio buttons are now square, like checkboxes.**
They stay distinguishable by what appears inside — a checkbox shows a tick, a
radio shows a filled square — but the round-versus-square shorthand for "pick
one" versus "pick many" is gone. Restoring it is one token:
(secret removed) 50%`.

## Deck

Slides live in `deck.css`, which is **not in the bundle** — link it on a deck
page only. `docs/deck.html` is the working example and the ten layouts are listed
on it.

**Pair the modifier with the surface class.** A gold slide is
`ac-slide ac-slide--gold ac-surface-accent`; a dark one is
`ac-slide ac-slide--dark ac-surface-inverse`. The modifier paints the
background; the surface class re-points every colour token inside it, including
headings, which `base.css` colours explicitly and which therefore ignore plain
inheritance. Forget it and a heading on gold renders graphite-on-gold at
4.45:1 — the same 0.05-below-threshold miss this system already repaired once
for buttons. `make test` fails on the omission.

- **One slide, one idea.** If a slide needs a scrollbar, it is two slides. The
  deck gate measures this, because a slide silently clips anything past its
  bottom edge and nothing else will tell you.
- **Three paragraphs at most.** Four is a document.
- **Every figure comes from [`brand-facts.md`](brand-facts.md)**, plus signs
  included. `300+ clients` is the fact; `300 clients` is a different and untrue
  claim.
- **Emphasis in a title is weight, not colour.** `<em>` inside
  `ac-slide__title` sets weight 700 and stays the ink colour. White on Golden
  Sun is 1.67:1 and fails at every size, including 105px.
- **Photography is duotint only.** A raw full-colour photograph is not an
  araCreate image.
- Numbers on slides come from a CSS counter, so inserting a slide renumbers the
  deck. Do not type slide numbers.

## Depth

**Three shadows, and a component picks one.** `--ac-shadow-lift` for hover,
`--ac-shadow-float` for a dropdown or a toast, `--ac-shadow-overlay` for a
modal. Nothing composes its own `box-shadow` and nothing is ever coloured —
araCreate reads industrial and flat, not glassy.

**There is deliberately no resting shadow.** A card at rest is flat; the thing
separating it from the page is the signature edge. A resting shadow as well
would say the same thing twice.

## Layout

`ac-container` for the standard width — 1200px, which is the outer bound the
system was built and reviewed at. `ac-container--narrow` is 940px, which is what
every `.container` on aracreate.group actually measures. **Use the narrow one on
a page that should feel exactly like the live site**; the default is wider
because changing it would restyle everything already approved. `ac-section` for vertical rhythm.
`ac-bleed` to run edge to edge — it uses a measured viewport width, not `100vw`,
because `100vw` includes the scrollbar and is the usual cause of a few pixels of
horizontal scroll.

`ac-stack`, `ac-row` and `ac-grid` cover most arrangements. The utility list is
deliberately short: a system needing fifty utilities has components that are not
doing their job.

---

## Adding a component

1. Check nothing existing covers it. `ac-panel` and `ac-card` cover more than
   they look like.
2. Build from semantic tokens only.
3. Give it every state: rest, hover, active, focus-visible, disabled, and
   loading if it can wait for anything.
4. Name it `ac-thing`, with `ac-thing__part` and `ac-thing--variant`.
5. Add a live specimen to `components.html`, not a screenshot.
6. Add the "when not to use it" note here. A component without one gets misused.
7. Run `make test`. All six gates.

## What the gates do not catch

`make test` checks conventions, adherence, contrast, accessibility, behaviour and
that a real page needs no custom CSS. It does not tell you the page reads badly,
that a label is confusing, or that a layout is ugly.

Card body copy rendered **bold** for a whole build of this system — a card that
was a link inherited the link’s font weight. Every gate passed. It was found by
taking a screenshot and reading it.

**Screenshot every new component and look at it.** That is not a formality.
