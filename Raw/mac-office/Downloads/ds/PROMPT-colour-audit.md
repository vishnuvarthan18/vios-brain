# PROMPT — colour audit (report only, change nothing)

Paste into the Claude Design assistant with ACDS open.

```
Audit every colour in this design system. REPORT ONLY — do not change,
delete or rename anything. I will decide what to fix after I read your report.

THE BRAND RULE
araCreate is a deliberately tight palette: Golden Sun #f9bf3b and Graphite Gray
#555555 do the work, over canvas #f6f6f6, with near-black #222222 for headings
and #cecece for hairlines. Status colours (one green, one red) are the only
other hues allowed. There is no blue, no orange, no purple in this brand.
Note that near-black is #222222 — pure #000000 is NOT the brand black.

PART 1 — the token layer
Read tokens/colors.css, tokens/theme-dark.css and tokens/density.css.
List every token whose value is a colour that is NOT one of:
  - Golden Sun #f9bf3b or a tint/alpha of it
  - Graphite Gray #555555 or a step on the grey ramp
  - #222222, #f6f6f6, #ffffff, #cecece
  - the two status colours and their tints
For each, give the token name, its value, its line, and where it is used.
Say plainly whether you think it belongs in this brand or leaked in from
somewhere else.

I already believe four are wrong. Confirm or contradict each — if you think
I am wrong, say so and why:
  --ac-photo-overlay      #2e419e                  (this looks like a blue)
  --ac-button-gray-light  rgba(47, 53, 69, 0.6)    (this looks like a blue-grey)
  --ac-gray-translucent   rgba(46, 46, 46, 0.5)    (not on the grey ramp)
  --ac-true-black         #000000                  (brand black is #222222)

PART 2 — raw colours written outside the token layer
Find every hard-coded colour — #hex, rgb(), rgba(), hsl(), and the CSS keywords
black / white — anywhere outside tokens/.

EXCLUDE these, they are legitimate:
  - foundations/colors-*.card.html and the other foundations/*.card.html swatch
    cards. Their job is to display hex values, so raw colour there is correct.
  - anything inside a comment.
  - assets/ (SVG artwork).

DO NOT PROPOSE CHANGES to these locked files, only report what they contain:
  templates/deck/Deck.dc.html, templates/deck/ds-base.js, templates/deck/support.js
  ui_kits/deck/slides.jsx, ui_kits/deck/index.html, ui_kits/deck/card-section.html
  ui_kits/deck/card-stats.html, ui_kits/deck/card-vertical.html
  foundations/brand-icons.card.html, styles.css

For everything else give me a table: file, line, the colour, and which existing
token it should have been. If no existing token matches, say so — that is the
interesting case.

I count roughly 49 raw colour uses across 7 files, the worst being
styles/app.css, styles/base.css and styles/signature.css. Check whether that
matches what you find, and tell me if your number is different.
Also look at templates/marketing-page/support.js — I think there is an orange
in there, rgba(217,119,87,0), which would be off-brand.

PART 3 — the answer I actually want
Three lists:
  A. Colours that are off-brand and should go.
  B. Colours that are hard-coded but correct — they just need to become a token
     reference instead of a literal.
  C. Colours you are not sure about, and what you would need to decide.

Then stop. Do not fix anything. Do not delete anything. Do not touch the locked
files listed above.
```
