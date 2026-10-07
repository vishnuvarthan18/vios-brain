# Content rules

The site's promise is that a skeptical reader can check everything. These rules keep that true.

## Claims
- Do **not** say Tamil is "the first" or "the oldest" language. Independent fact-checkers reject it and the scholars who know Tamil best do not say it.
- State what the evidence supports, with the source: Sangam poetry and the Tolkāppiyam grammar, the epigraphic record, the Periplus and Muziris papyrus, the UNESCO citation for the Great Living Chola Temples.
- Every claim on the site is tracked on the About page (`#claims`). A claim that cannot be sourced comes off the site.
- Numbers (counts of inscriptions, years, words) need a source and a date, or an honest "about".

## "Sample · unverified"
Placeholder or unchecked text must carry the label "sample · unverified" (and its Tamil form). Never remove the label to make a page look finished. Remove it only when a Tamil speaker and a source check have both confirmed the text. Tamil text should be read by a Tamil speaker before it is called checked.

## Sources and licences
- Wikipedia-derived text: CC BY-SA 4.0, credit its contributors. Wikisource texts: CC BY-SA. Project Madurai e-texts: free for non-commercial distribution. Keep these lines in the site footer and About page.
- Every record in `website/data` keeps its source and licence. Do not strip them.
- Semmozhi fonts: SIL Open Font License 1.1 (licence files in `website/fonts/`). Third-party fonts keep their own OFL notices.

## Photos
- Only **public domain, CC0 and CC-BY** images may ship, each with a visible credit and a row in `design/references/LICENSES.csv`.
- **CC-BY-SA, CC-BY-NC and unclear** images are reference only: measure from them, never copy them or derive textures from them into `website/`.
- `design/references/` is git-ignored. Do not commit it.

## Tone and language
- Plain, specific sentences. No claims of "greatest" or "oldest".
- Tamil text uses Unicode, `lang="ta"`, and the Tamil font stack from the design tokens. Check that conjuncts render at phone size.
- Transliteration style follows the existing pages (for example Puṟanāṉūṟu, Paṭṭiṉappālai).

## Accessibility
Text contrast, 44px touch targets, visible focus, and reduced-motion support are part of the design system. Keep them when you add components.
