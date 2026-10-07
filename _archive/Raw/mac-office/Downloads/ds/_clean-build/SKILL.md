---
name: aracreate-design
description: Use this skill to generate well-branded interfaces and assets for araCreate Group, either for production or throwaway prototypes/mocks/etc. Contains essential design guidelines, colours, type, fonts, assets, and UI kit components for prototyping.
user-invocable: true
---

Read `readme.md` within this skill first, then `CLAUDE.md` for the standing rules, and explore the other available files.
If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out and create static HTML files for the user to view. If working on production code, you can copy assets and read the rules here to become an expert in designing with this brand.
If the user invokes this skill without any other guidance, ask them what they want to build or design, ask some questions, and act as an expert designer who outputs HTML artifacts _or_ production code, depending on the need.

Quick reference:
- **Colours:** Golden Sun `#f9bf3b` (accent, never text), Graphite Gray `#555555` (body text), brand black `#222222`, stroke `#cecece`, canvas `#f6f6f6` (the page), white `#ffffff` (the raised surface), pale gold `#fdf3d8` (the alternating band). Stay in palette. Danger `#c0492f`, success `#186a43`; warning reuses gold and information is graphite.
- **Type:** Monument Extended (logo wordmark ONLY) + Poppins (all headings, titles, body and UI — lean on Light 300, Bold 700 for metrics) + Red Hat Mono (specs). Headings 36 / 33 / 31 / 28 / 21 / 16px, with a separate display tier at 64 / 48 / 36 / 28px. Eyebrows and buttons are wide-tracked UPPERCASE; everything else is sentence case.
- **Shape:** buttons square, inputs 4px, panels 9px, cards 20px, pills full — but anything carrying the signature edge stays square. The signature edge is a 1px border, dotted top and right, solid bottom and left. Hairline dividers, four low neutral shadows, hover lifts ~5px. Every interactive control is at least 44px. No emoji. Imagery is isometric illustration and duotint photos.
- **Voice:** professional, clear, friendly. "We" / "you". Sentence case. Tagline: *"Empowering ideas from mind to market."* 360° services. Never invent a fact — `docs/brand-facts.md` is the list, and `300+ clients` keeps its plus sign.
- **Stylesheets:** link `system.css` for the whole system. `styles.css` is the older, smaller entry point — four token imports and nothing else — and is pinned for the locked deck files; do not widen it.
- **Tokens:** `tokens/` (fonts, colors, typography, spacing, density, theme-dark). **CSS:** `styles/` (base, signature, components, sections, app, deck). **Behaviour:** `js/`. **Components:** `components/` — 83 exported across nine groups (core, forms, feedback, navigation, data, signature, sections, app, content), React, also exposed as `(secret removed)`; `docs/components.md` is the reference. **Specimens:** `foundations/`. **Kits:** `ui_kits/` (website, academy, web_app, deck). **Starting folders:** `templates/`.
- **Assets:** `assets/` carries the real files — `logos/` (21 SVG plus 11 PNG rasters and three Affinity source sheets), `icons/` (17 isometric and UI SVGs), `illustrations/` (8), `imagery/` (5 duotint photographs), `brand/` (the seal and the letterhead) and `fonts/` (both Monument Extended weights). `docs/assets.md` says which logo belongs on which background — they are not interchangeable.
- **The live site:** `docs/live-site.md` is a snapshot of aracreate.group verified 17 August 2026. Pages under `/archive/` and `/template/` are not valid reference.
