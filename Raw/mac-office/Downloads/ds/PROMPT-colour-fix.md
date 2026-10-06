# PROMPT — apply the two colour decisions

Paste into the Claude Design assistant with ACDS open. Two changes only.

```
Two decisions from the colour audit. Make these changes and nothing else.
Report before you finish. Do not touch the locked files.

=====================================================================
CHANGE 1 — muted text must pass contrast
=====================================================================
--ac-text-muted is #8a8a8a. It measures 3.19:1 on canvas, which fails the
4.5:1 standard, and the file's own comment records that. The system already
has an accessible grey at the same hue: --ac-gray-450 (#6f6f6f, 4.65:1).

Point muted text at it:
    --ac-text-muted: var(--ac-gray-450);

Then find and fix everything that still states the old value or the old
measurement. At minimum check:
  - the comment on that line in tokens/colors.css
  - docs/accessibility.md
  - changelog.md, where this is recorded as a "knowingly-kept failure" — it is
    no longer kept, so that entry needs correcting, not deleting
  - readme.md
  - foundations/colors-text.card.html and any swatch card showing #8a8a8a
  - docs/decisions.md

Do NOT change the dark-theme override of --ac-text-muted in tokens/theme-dark.css.
That is a different value on a different background and it already passes.

Then re-measure. Report the new ratio for muted text on:
  --ac-surface-page, --ac-surface-card, and --ac-surface-accent (gold).
If any of those now fails, stop and tell me before going further.

=====================================================================
CHANGE 2 — delete four dead tokens
=====================================================================
These four are off-brand and, per the audit, used by nothing:
    --ac-photo-overlay      #2e419e                (navy)
    --ac-true-black         #000000                (brand black is #222222)
    --ac-gray-translucent   rgba(46, 46, 46, 0.5)
    --ac-button-gray-light  rgba(47, 53, 69, 0.6)

FIRST re-verify that yourself. Grep the whole project for each token name.
If ANY of the four is referenced by a stylesheet, component, card or template,
do not delete that one — report it and leave it in place.

For the ones confirmed unused, remove the declaration and clean up every place
that mentions them:
  - tokens/colors.css, including the header comment that maps
    "--white / --black -> --ac-pure-white / --ac-true-black"
  - docs/live-site.md
  - docs/assets.md
  - _adherence.oxlintrc.json (the allow-list)

foundations/colors-photo-wash.card.html exists only to display the navy.
Before removing it, confirm it documents nothing else. If it is purely the
photo-wash swatch, delete the card. If it covers anything still in the system,
keep the card and edit it instead — and say which you did.

=====================================================================
DO NOT
=====================================================================
- Do not touch: templates/deck/*, ui_kits/deck/*, foundations/brand-icons.card.html,
  styles.css.
- Do not touch templates/*/support.js — both are generated runtime files marked
  "do not edit", and the orange in them is editor chrome, not a rendered surface.
- Do not start on the alpha-scale problem. That is a separate job.
- Do not change any other token.

=====================================================================
REPORT
=====================================================================
1. The old and new value of --ac-text-muted, and the three contrast ratios.
2. For each of the four tokens: confirmed unused, or found in use where.
3. Every file you edited, and what you changed in it.
4. What you did with colors-photo-wash.card.html and why.
5. Whether check_design_system is still clean, and whether the Forms, Core and
   Website cards still render.
```
