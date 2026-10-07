# UI kit — aracreate.group

A click-through recreation of the araCreate Group marketing site, composed from
this system's own components. Nothing here is a new design: every layout is one
of the section families defined in `css/sections.css`, and every figure and
company name comes from the brand facts in `readme.md`.

**Screens** — the nav switches between them.

| Screen | File | Shows |
| --- | --- | --- |
| Home | `HomeScreen.jsx` | split hero, marquee band, service list, dark stats band, values grid, gold ecosystem table, duotint imagery, testimonial, journal listing, CTA band |
| About | `AboutScreen.jsx` | founder card, quote, values, gold stats band, ecosystem table, ventures grid |
| Services | `ServicesScreen.jsx` | vertical tabs with division lockups, capability cards, steps, engagement models, logo strip, FAQ accordion |
| Contact | `ContactScreen.jsx` | full form with live validation, alert, dropdown, tooltip, modal, toast, progress |
| 404 | `NotFoundScreen.jsx` | reached by the "Journal" nav link, on purpose |

**Deliberately blank.** The team grid, the client logo strip, the journal
articles and all pricing are visible placeholders — `Client name`, `00+`,
`Article title` — because none of that is in the sources. A visible
placeholder gets replaced; a plausible invention does not.
