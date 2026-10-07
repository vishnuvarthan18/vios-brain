# THE LIVE SITE — aracreate.group

> **Provenance.** This is a **snapshot**, not a live check. Everything below was
> read directly from the production Webflow project on **17 August 2026** and
> moved into this file unchanged on **21 August 2026**, when the 20 August
> design system was merged into ACDS. The site is under active CMS management,
> so instance counts, page lists and CMS collections drift. Treat every figure
> here as "true on 17 August 2026" and re-verify before relying on one.
>
> This file is the only place in the project that describes the live site. The
> readme cross-references it rather than repeating it.

---

## Verified site identity

The live CMS site is managed in **Webflow**. Verified against production on
2026-08-17:

- Site: **`aracreate.group`**
- Site id: **`63780fb6eec282197fc5547f`**
- Last published: **2026-06-09**
- Colours reconciled: **2026-08-17**

**Colours match.** Golden Sun `#f9bf3b`, Graphite Gray `#555`, canvas `#f6f6f6`
and the 0.6 / 0.8 / 0.9 tints are identical to the live Webflow "Base
collection".

**Four live variables were missing** and were added to `tokens/colors.css` on
2026-08-17: `--ac-photo-overlay` (`#2e419e`), `--ac-true-black` (`#000`),
`--ac-gray-translucent` and `--ac-button-gray-light`.

**All four were removed again on 2026-08-23.** Nothing in this system ever
referenced them — no stylesheet, component, card or template — and each is
outside the palette: `#2e419e` is a navy, `rgba(47,53,69,.6)` a blue-grey,
`rgba(46,46,46,.5)` is off the grey ramp, and `#000000` contradicts the brand
near-black `#222222`. Existing in Webflow was the only argument for them.
They stay recorded here as live-site facts; they are no longer tokens.

**Fonts.** Webflow hosts no custom fonts, so Poppins is served from a webfont
service. Monument Extended is licensed for the **logo wordmark only** and is
never used for headings or body — `tokens/typography.css` enforces this via
`--ac-font-logo`.

The live site's WebFont loader also requests **Inconsolata** alongside Poppins
and Red Hat Mono. This project's `tokens/fonts.css` loads Poppins and Red Hat
Mono only; Red Hat Mono is `--ac-font-mono`. If a page has to match the live
font stack exactly, Inconsolata has to be added there deliberately.

---

## Live page architecture

Every araCreate page follows the same skeleton, confirmed by reading the live
element trees:

```
Body
  └ menu / logo / right-line / right-section-seperator   (global component instances)
  └ .sections
      └ .section                       ← default canvas band
          └ .container (+ .hero)
          └ .banner-photo.<page>-page-banner
      └ .section.bg-brand-yellow-light  ← alternating pale-gold band
  └ footer
```

The rhythm is the design's backbone: full-width sections alternating between
canvas `#f6f6f6` and the pale gold wash, each holding one centred `.container`.
Page-specific banners are `.banner-photo` plus a page modifier
(`.clients-page-banner`, `.projects-page-banner`).

---

## Live component inventory (Webflow)

These are the reusable components defined in the production site — page-level
composites rather than primitives. The templates in `templates/` compose the
equivalents; the `components/` primitives in this project are the smaller pieces
those composites are built from.

| Component | Instances | Notes |
| --- | --- | --- |
| `menu` | 37 | Global nav with full-screen overlay |
| `logo` | 36 | Wordmark, image-prop driven |
| `footer` | 28 | Current footer |
| `right-line` | 15 | Fixed right-edge rule |
| `right-section-seperator` | 13 | Section divider on the right margin |
| `cta` | 8 | Call-to-action band |
| `projects-slider-section` | 7 | Project carousel |
| `trusted-by` | 6 | Client logo strip |
| `footer-old` | 6 | Legacy, archive pages only |
| `faq` | 3 | Accordion |
| `loader` | 2 | Page loader |
| `dtf-header` / `dtf-footer` | 2 each | DTF sub-brand chrome |
| `cms-testimonial`, `old-logo-slider` | 0-1 | Unused / legacy |

Live pages worth reading for further design context: Impact, Engineering,
Manufacturing, Media, Blogs, Projects, About, Contact, and the `/dtf/`
sub-site.

---

## CMS surfaces

The live CMS export (published June 2026) is the current production site. It
confirms the live palette, the WebFont stack and the animated isometric
"machine" hero, and it adds CMS-driven surfaces:

- Static pages: `index`, `about`, `engineering`, `manufacturing`, `media`,
  `projects`, `impact`, `contact`
- A **blog**: `blogs.html` plus `blogs/*`
- CMS lists: `lists/clients.html`, `lists/team.html`, `lists/testimonials.html`
- Legal pages under `etc/`

---

## The /archive/ and /template/ warning

**Pages under `/archive/` and `/template/` are legacy and are not valid design
reference.** They carry retired chrome — `footer-old` and `old-logo-slider`
appear only there — and copying from them reintroduces patterns the current site
has already dropped. When in doubt about whether a page is current, check
whether it uses `footer` or `footer-old`.

---

## DTF — the parallel `-dtf` namespace

**DTF** (Deep Tech Foundry — an araCreate venture) runs the same system under a
parallel `-dtf` namespace: `.section-dtf`, `.container-dtf`,
`.dtf-bg-brand-yellow-light`, with its own `dtf-header` and `dtf-footer`
components. Same alternating bands, same palette; treat it as a sub-brand of the
same system rather than a separate one.

---

## Notes carried with this material

**Substitution flag:** none required — Monument Extended and Poppins are both
available (Monument from the supplied OTFs, Poppins and Red Hat Mono from Google
Fonts).

**The live site's hero:** the production CMS site animates bespoke multi-layer
isometric "machine" SVG sprites (`machine-01…`, `machine-03…`). These are
intentionally **not** recreated in this project — the static isometric
illustrations in the brand asset library substitute for them.

**The 72-page audit.** The design system that merged into ACDS on 21 August 2026
was built partly from a measured audit of 72 pages of aracreate.group. Where a
value in `tokens/` is described as "as measured on the live site" — the heading
ladder, the 940px container, the 20px most-used space value — that audit is the
source, and this snapshot is the identity check on top of it.
