# CONTRIBUTING

How to add to this system without pulling it apart. The rules are the source
repository's; the process is this project's.

---

## The five rules that outrank convenience

1. **Never write a raw value.** No hex colours, pixel sizes or timings in a
   component. A missing token is a signal, not an obstacle: use the nearest one,
   or add one deliberately. The source repo found itself breaking this eighteen
   times in its own files.
2. **Name the job, not the colour.** `--ac-action-bg`, never `--ac-yellow`.
3. **Golden Sun is never text.**
4. **Nothing invents a fact.** Every figure, company name and address is in
   [`brand-facts.md`](brand-facts.md). `300+ clients` is the fact — the plus sign
   is what makes it true.
5. **Do not undo the three things the system handles** — the focus ring,
   reduced-motion handling, and the surface contexts. See
   [`accessibility.md`](accessibility.md).

---

## Adding a component

1. **Check nothing existing covers it.** `Panel` and `Card` cover more than they
   look like. A KPI tile turned out to be `Card kind="stat"` with a different
   name; building it twice would have given the system two of everything.
2. **Build from semantic tokens only.** If you find yourself writing
   `#f9bf3b`, stop — the token is `--ac-surface-accent`.
3. **Give it every state**: rest, hover, active, focus-visible, disabled, and
   loading if it can wait for anything. A component with three states ships with
   three bugs.
4. **Name it `ac-thing`**, with `ac-thing__part` and `ac-thing--variant`. BEM is
   a declared deviation from strict param-case and it is enforced exactly:
   camelCase, single underscores and capitals are all rejected.
5. **Write the three files.** `<Name>.jsx` with a named PascalCase export,
   `<Name>.d.ts` with the props interface and a one-line "what and when" JSDoc,
   `<Name>.prompt.md` with a usage example and the variants.
6. **Add it to the directory's `@dsCard` HTML.** Dense and scannable — every
   state, not one default render.
7. **Add its "do not use it for" note** to
   [`components.md`](components.md). A component without one gets misused.
8. **Name it in `readme.md`.** That file is the component index.
9. **Log it in `changelog.md`** with the reasoning, not just the fact.
10. **Screenshot it and read it.** Not a formality — see below.

---

## Which group does it belong in?

| Group | Test |
| --- | --- |
| `core` | Would every surface use it? |
| `forms` | Does it collect input? |
| `feedback` | Does it tell the visitor something happened? |
| `navigation` | Does it move the visitor, or switch what they see? |
| `data` | Does it present a structured set? |
| `signature` | Is it one of araCreate's named devices? |
| `sections` | Is it a whole slab of a **marketing page**? |
| `app` | Is it only meaningful **signed in**? |

If it fits two, it belongs in the more general one. If it fits none, that is
worth a conversation before it becomes a ninth group.

---

## Screenshot every new component and look at it

This is the most important instruction in this file, and it is here because it
keeps paying.

The source repository records a whole build in which **card body copy rendered
bold** — a card that was a link inherited the link's font weight. Every gate
passed. It was found by taking a screenshot and reading it.

This project, in one turn, produced two more of the same kind:

- The side nav rendered graphite on brand black at **1.55:1**, and its current
  item at **1.00:1 — invisible.** Valid markup, working links, no console error.
- The table's select-row checkbox rendered as the browser's round blue default,
  because `components.css` styles checkboxes only inside `.ac-choice`.

**A check tells you what it was told to look for. It does not tell you the page
looks wrong.** Render the thing, read the numbers off it, and look at it.

---

## Changing a token

A token change is a system-wide visual decision. Two examples from the record,
both instructive:

- **Every radius set to `0`** cost five edits, because nothing hardcodes a
  radius. That is what the token layer is for.
- **Correcting brand black** from a measured `#2e2e2e` to the specified
  `#222222` *improved* every contrast figure touching it. The measured value was
  an average of three colours and matched none of them.

**Every `--ac-*` name is declared at `:root`.** A component may *re-point* one
inside its own scope — that is how a dark band works without a dark variant per
component — but it may not invent one. Verified 20 Aug 2026: nothing in the CSS
declares an `--ac-*` name that does not also exist at `:root`.

The design-system check reports ~80 custom properties "declared under
component-style selectors". Those are the re-pointings, and they stay: surface
and banner contexts re-pointing the text set, `[data-ac-theme="dark"]
.ac-surface-inverse` flipping the band back the other way, `.ac-btn--outline`
re-pointing the five button slots. They cannot move to `:root` — a value that
applies only inside one context is not a root value — and flattening them would
delete the mechanism the whole system rests on. What was *worth* fixing, and was
fixed the same day: modifiers that re-pointed a token nothing else read
(`.ac-edge--thick`, `--square`, `--round`, `--strong`, `--faint`, `--accent`,
the underline and marquee variants, the card grid and drawer widths) now set the
CSS property directly, and the button's five private locals became public
`--ac-btn-*` tokens.

So: change the token, do not override at the component. And if a value has a
recorded reason in [`decisions.md`](decisions.md), read that first — the heading
ladder being 3px apart and body copy being Light 300 are deliberate, locked, and
recorded so nobody "fixes" them without asking.

---

## The three locked deviations

| What | Status |
| --- | --- |
| Headings 36 / 33 / 31 / 28px, ~3px apart | **Locked.** Matches the live site exactly. |
| Body copy in Poppins Light (300) | **Locked.** Passes at 6.90:1, but thin weights read worse than that figure implies. |
| Links at 12px | **Unlocked 20 Aug 2026** → 15px, plus a 44px target. |

Each is one token. Do not change one without asking; do not assume any was an
oversight.

---

## What this project does NOT have

Said plainly so nobody assumes otherwise:

- **No test gates.** The source repository has seven, including contrast,
  accessibility, behaviour and an adherence linter that checks *other* projects.
  None of them run here. Verification in this project is by hand, recorded in
  [`accessibility.md`](accessibility.md).
- **No versioning or release process.** `changelog.md` is written by hand.
- **No screen-reader pass.** Outstanding here and in the source repo.
- **No production use.** Every `app` component is authored, rendered and
  measured, but nothing has shipped on it.

The single highest-value thing anyone could add is **the contrast and
accessibility gates, globbing rather than listing**, so a new card or kit is
covered the moment it exists rather than the moment someone remembers.

---

## Keeping this project and the source repository in step

They are two things, and the relationship is one-directional:

- `aracreate-design-system` (the repo) is **the system**. Its `src/*.css` is the
  source of truth for every value.
- This project is a **port**, with additions. `css/tokens.css`, `base.css`,
  `signature.css`, `components.css`, `sections.css` and `deck.css` are copies —
  edit them here and the change is lost on the next port.

**Additions live in files the repo does not have:** `css/app.css`,
`css/density.css`, `css/theme-dark.css`, `css/fonts.css`, and everything under
`components/`, `templates/` and `ui_kits/`.

[`changelog.md`](../changelog.md) is the reconciliation list. Anything in it
under "Changed" is a divergence somebody eventually has to decide about — the
dark theme, the density scale and the link size are all worth taking back to the
repo rather than leaving them only here.
