# BACK-PORT

What this project added that does not exist in the araCreate Design System
repository, and what should go back into `src/`.

**Why this file exists.** This project is a port. Six of its CSS files are
copies, and editing them here loses the change on the next port. Everything
below lives *only* here, so it either goes back to the repo or it drifts —
and drift is the exact failure `decisions.md` spent two days untangling.

Written 20 August 2026. [`../changelog.md`](../changelog.md) has the full
reasoning for each item; this is the action list.

---

## Copies — never edit here

| File here | Source |
| --- | --- |
| `css/tokens.css` | `src/tokens.css` |
| `css/base.css` | `src/base.css` |
| `css/signature.css` | `src/signature.css` |
| `css/components.css` | `src/components.css` |
| `css/sections.css` | `src/sections.css` |
| `css/deck.css` | `src/deck.css` |
| `js/signature.js`, `js/components.js`, `js/deck.js` | `src/` |
| `assets/**` | `src/assets/` |
| `guidelines/{brand-facts,guidance,decisions,licence-policy,assets}.md` | `docs/` |

Two exceptions were edited, both minimal and both worth taking back:

1. **`css/tokens.css`** — `--ac-size-link` 12px → 15px, the dark-theme comment
   block rewritten to point at the new file, and the link-size deviation note
   rewritten to record it as unlocked. Nine `/* @kind other */` annotations were
   added to tokens whose kind cannot be inferred; those are cosmetic and specific
   to this project's compiler, so **do not take them back**.
2. **`css/deck.css`** — `--ac-chevron-size` hoisted from `.ac-chevrons__mark` to
   `.ac-deck`. It was the one property in the system with no declaration in a
   token scope. **Take this back.**

---

## Take these back, in this order

### 1 · `css/theme-dark.css` → `src/theme-dark.css` — HIGH VALUE

The dark theme `tokens.css` left a door open for. Its own prediction held
exactly: **no component was touched**, because none contains a raw colour.

Three things needed more than a token swap and are documented in the file:
Golden Sun does not move; the inverse surface flips to light; error and success
need dark-specific values because the light pair fails on the graphite page, as
does the earlier dark pair. The dark page is graphite `#555555` — there is no
black in this system.

**Not bundled** — link it separately, like `deck.css`. Most sites will not want
it, and it is 200 lines.

**Gate consequences.** `check-contrast.js` should walk every page twice, once
with `data-ac-theme="dark"` on `<html>`. That doubles the sweep and is the only
way the theme stays honest.

### 2 · `css/density.css` → `src/density.css` — HIGH VALUE

The 44px target floor and the `[data-ac-density="compact"]` scope. Two things in
one file because they are the same argument: control size.

**This one changes the look of existing pages.** Footer and navigation links get
larger, and footer columns get taller. That is the link-size fix, not a
regression, but it needs saying before someone reports it.

**Gate consequence.** A new rule worth writing: every interactive control's
rendered box is at least 44 × 44, exempting links inside running text. It would
have caught `.ac-btn--sm` at 32px.

### 3 · `--ac-size-link`: 12px → 15px — DECISION, NOT CODE

One token. The client unlocked it on 20 August 2026, having kept the other two
deviations. Navigation, footer and breadcrumb links also moved off
`--ac-size-caption` onto `--ac-size-link`, because they are links, not captions
— that change is in `density.css` rather than `sections.css` so the copy stayed
diffable, but **in the repo it belongs in `sections.css`.**

### 4 · Three bug fixes — TAKE THESE BACK REGARDLESS

Found in this project, but two of them are faults in the *pattern*, not in the
new code:

| Fault | Where it belongs |
| --- | --- |
| Bare checkboxes (no `.ac-choice` wrapper) fall through every selector and render as the browser's blue default | `src/components.css` §4 — style `input[type="checkbox"]` generally, not only inside `.ac-choice` |
| `--ac-grey-450` (#6f6f6f) passes on plain canvas at 4.61:1 but measures **4.18:1** on a *tinted* row | worth a note in `tokens.css`: the muted grey has no headroom for a tint |
| `--ac-chevron-size` had no base declaration | `src/deck.css` |

### 5 · `tests/checks.html` — MAYBE

A browser-runnable port of the contrast and accessibility gates, globbing rather
than listing. The repo already has better versions in Playwright, so this is only
worth taking if a no-install check is useful to someone without Node.

---

## Do NOT take these back

- **The React components** (`components/**`). `decisions.md` records the decision
  not to have React, and the reasoning is sound: this system's components *are*
  CSS classes, and a wrapper would either duplicate every value or add a build
  step to write `class="ac-btn"`. They exist here because a Claude design system
  consumes components as React. **That decision should not be revisited on the
  strength of this project.**
- **`css/app.css`** — *probably not*, and this is the one genuine judgement call.
  It is 1,000 lines of application surfaces (shell, selectable table, empty
  state, skeleton, date picker, drawer, chart frame) with **no counterpart in the
  repo and no production use anywhere.** Taking it back means the repo starts
  carrying a product UI layer for a product that does not exist yet. Take it when
  araCreate actually builds a signed-in tool, and take it *then* against real
  screens rather than my demonstration.
- **`css/fonts.css`** as written. It self-hosts Monument Extended, which the repo
  deliberately excludes — see the licence note in `../readme.md`. The Google
  Fonts import is worth taking; the two `@font-face` rules are not.
- **The `@kind` token annotations.** Specific to this project's compiler.
- **`ui_kits/**`** — three of the four are recreations of things the repo already
  documents, and the fourth is a demonstration.

---

## The reconciliation, as a checklist

```
[ ] copy css/theme-dark.css      → src/theme-dark.css   (not bundled)
[ ] copy css/density.css         → src/density.css      (bundled)
[ ] set  --ac-size-link: 15px    in src/tokens.css
[ ] move nav/footer/breadcrumb link sizing to src/sections.css
[ ] widen the checkbox selector  in src/components.css
[ ] hoist --ac-chevron-size      in src/deck.css
[ ] note the tint headroom       in src/tokens.css
[ ] extend check-contrast.js     to sweep the dark theme too
[ ] add a 44px target-size rule  to check-a11y.js
[ ] update docs/decisions.md     — the dark theme is built; the link is unlocked
[ ] update docs/guidance.md      — density, target size, dark theme
[ ] bump VERSION / let semantic-release cut it
```

Eleven lines. The dark theme and the density scale are the two that matter.
