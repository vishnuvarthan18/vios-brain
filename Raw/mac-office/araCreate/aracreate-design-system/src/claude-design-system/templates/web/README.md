# Template — acds-template-web

The whole araCreate website in ONE file: `Web.dc.html`. It is the only
description of the site in this project: it replaced the `marketing-page/`
template, the eight loose section files that sat unused in `ui_kits/website/`,
and then the website kit itself.

**Three pages, switched by the `page` tweak or by the nav:**

- **home** — every band below, one scroll, anchors for services, projects,
  journal and contact.
- **about** — breadcrumb, split hero, four-stat graphite band, the founder
  split with the values list, the eight group companies as a table, the ventures
  grid.
- **404** — oversized code and two ways back.

The kit's Services, Projects and Contact screens are not pages here: their
content is bands on the home page, which is how a site this size is actually
built.

**Every element of the site, in page order.**

| # | Band | Classes it demonstrates |
| --- | --- | --- |
| 1 | Sticky header, working mobile toggle | `.ac-header`, `.ac-nav`, `.ac-nav__panel` |
| 2 | Split hero | `.ac-hero--split`, `.ac-dash--accent` |
| 3 | Marquee | `.ac-marquee--bordered`, `.ac-marquee--slow` |
| 4 | Trusted-by logos | `.ac-logos`, `.ac-logo-tile` |
| 5 | Service list | `.ac-service-list` |
| 6 | Stats band, graphite | `.ac-surface-inverse`, `.ac-stat-row` |
| 7 | Values grid | `.ac-card--service`, `.ac-card-grid` |
| 8 | Ecosystem table, gold band | `.ac-surface-accent`, `.ac-table--hover` |
| 9 | Projects | `.ac-project-grid`, `.ac-card--project` |
| 10 | Testimonial | `.ac-quote`, `.ac-panel` |
| 11 | Journal listing | `.ac-post-list`, `.ac-post` |
| 12 | Contact details + form | `.ac-contact__inner`, `.ac-form`, `.ac-field`, `.ac-input` |
| 13 | CTA band | `.ac-cta`, `.ac-btn--inverse` |
| 14 | Footer | `.ac-footer` |

**Tweaks.** `showTrustedBy` and `showJournal` drop bands 4 and 11 — the two a
page usually does not have copy for yet. Everything else is edited in place.

**One line to point at a design system.** `ds-base.js` links `system.css`, the
WHOLE system. `tokens.css` is variables only and would leave every `.ac-*`
class unstyled. If you copy this folder into a consuming project, edit the `base` line.

**Placeholders are visible on purpose** — `Client name`, `Company name`,
`Project title`, `Article title`, `00+`. A visible placeholder gets replaced; a
plausible invention does not.
