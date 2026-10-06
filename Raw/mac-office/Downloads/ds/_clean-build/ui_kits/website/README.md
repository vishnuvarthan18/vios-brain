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
| Projects | `ProjectsScreen.jsx` | breadcrumb, hero, `.ac-project-grid` of `.ac-card--project` cards, one placeholder card, CTA band |
| Contact | `ContactScreen.jsx` | full form with live validation, alert, dropdown, tooltip, modal, toast, progress |
| 404 | `NotFoundScreen.jsx` | reached by the "Journal" nav link, on purpose |

**Shared band** — not a screen of its own.

| Band | File | Shows |
| --- | --- | --- |
| Trusted by | `TrustedByStrip.jsx` | `LogoTile` row in `.ac-logos`, rendered on the home screen under the marquee |

Both `ProjectsScreen.jsx` and `TrustedByStrip.jsx` carry their copy over from
the older ACDS website kit unchanged: three project titles with their vertical
tags, and the six trusted-by names.

**Deliberately blank.** The team grid, the ecosystem logo strip, the journal
articles, the fourth project card and all pricing are visible placeholders —
`Client name`, `00+`, `Article title`, `Project title` — because none of that is
in the sources. A visible placeholder gets replaced; a plausible invention does
not.
