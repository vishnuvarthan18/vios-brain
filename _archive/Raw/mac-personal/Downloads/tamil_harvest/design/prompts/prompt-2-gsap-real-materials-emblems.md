# PROMPT 2 — Real materials, GSAP animation, and the Emblems page

Copy the start prompt from the chat, or paste everything below the line.

---

## Context (read first)
Prompt 1 is done and accepted: `website_live/css/tokens.css`, `textures.css`, `components.css`, `js/components.js`, `styleguide.html`, `DESIGN_SYSTEM.md`. Keep working on branch `redesign-design-system` (commit the Prompt 1 files first as their own commit). Keep `website_live_backup_2026-09-25/` untouched. Do not change the old pages (index, tamil, scripts, grantha, vatteluttu, font, fonts) in this prompt.

The owner tested the style guide and said: the animation and textures do not feel real yet. Goal now: make them feel like the real objects. Also build a separate page of Tamil dynasty and temple emblems.

## Part A — Real textures (replace the flat CSS ones)
"Real" means it must look like a photo of the material, not a gradient. Use these two methods, in this order:

1. Procedural realism (default, no license risk): SVG filters `feTurbulence` + `feDiffuseLighting` / `feSpecularLighting` + `feDisplacementMap`, layered with CSS blend modes (`multiply`, `overlay`). Build these per material:
   - Palm leaf: warm amber base, fine parallel fibre lines running along the length, small dark blotches (age spots), uneven fibre brightness, a soft translucent glow at thin edges, torn or nicked edge made with displacement. Two round holes with a darker inner rim and slight bevel. Letters on the leaf look scratched then inked: a dark ink line inside a thin lighter scratch groove.
   - Stone: granular grain, pits, weathering stains, chisel marks; carved letters with real depth (inner shadow, lit edge, tiny dust in the groove).
   - Copper plate: hammered surface (many small dents), green patina creeping from the edges and grooves, dull metal highlight that shifts with mouse or device tilt.
   - Terracotta sherd: rough clay grain, chipped edge, slip-colour bands, scratched marks.
   - Coin: raised relief edge, worn high points, soft metal shine, rim.
   - Cotton thread (bundle): twisted fibre look, slight shadow on the leaves under it.
2. Real photo tiles (only if a file is SHIP_OK): from `design/references/<surface>/` pick at most 1 to 2 close-up crops per material where `LICENSES.csv` says SHIP_OK (public domain, CC0, CC BY). Crop a plain texture area (no text, no faces), save as WebP under 60 KB each in `website_live/img/tex/`, and use it as a low-opacity overlay under the procedural layer. Add each used file to `website_live/credits-data.json` with id, object, license, credit, source URL. Never use CC BY-SA, CC BY-NC, or unclear files in production.

Rules: text stays on a flat readable area (contrast rules from Prompt 1 still apply). Each texture file under 60 KB. Total added images under 400 KB. Provide a `data-quality="low"` fallback (plain colour) for slow phones and for `prefers-reduced-data` / low-power devices. Filters are heavy: render each filter once on a static layer, do not animate filter parameters.

Update `styleguide.html` with a "Real materials" section that shows each material large, next to the old flat version, so the owner can compare.

## Part B — Animation with GSAP
Use GSAP (https://gsap.com/) for all new animation. Steps:
1. Self-host: download `gsap.min.js` plus the plugins you use (`ScrollTrigger`, `Flip`, `Draggable`, `MotionPathPlugin`, `SplitText`, `DrawSVGPlugin`, `MorphSVGPlugin`, `CustomEase`) from the official GSAP release (npm package `gsap`) into `website_live/js/vendor/gsap/`. Do not load from a CDN; the site must open by double-click with no server and no internet.
2. Check the current license at https://gsap.com/standard-license and copy the license text into `website_live/js/vendor/gsap/LICENSE.txt`. Report in one line what the license says about using it on this site. If any plugin you want is not covered, do not use it.
3. Replace the CSS-only motion from Prompt 1 with GSAP timelines. Keep the old CSS as the reduced-motion and no-JS fallback.

Animations to build (physical, with weight, not just fades):
- Leaf flip: the leaf lifts, swings on one hole like a real stacked leaf (3D transform, perspective, slight bend using a two-part skew), the next leaf slides underneath, a soft shadow follows. 600 to 800 ms, custom ease. Draggable: the visitor can also drag the leaf left or right and it settles.
- Bundle untie: thread unwinds along an SVG path (DrawSVG or stroke-dashoffset with MotionPath), knot loosens, boards swing open on their hinge, leaves fan out one by one with a stagger, then the reader opens. Reverse plays when closing.
- Ink writing: Tamil letters on a leaf appear as if written by the stylus: an SVG mask reveals each line left to right with an iron-point nib following it, leaving a scratch groove first and ink second.
- Stone carving reveal: heading letters appear as chisel strikes (tiny dust particles, letter depth animates from flat to carved).
- Copper plate: light sheen sweeps across as the pointer moves; the ring swings with a small physical spring when the plate loads.
- Scroll storytelling (ScrollTrigger): as the visitor scrolls through a section, the material changes: stone slab to copper plate to palm leaf, with a slow parallax of layers and gentle depth-of-field. Pin one scene at a time, at most 1 pinned scene per screen.
- Page load: a short intro (under 1.8 s, skippable) where one leaf slides in, the two holes catch the light, and the site title is inked.
- Micro-interactions: buttons press in with a small spring; cards lift with a soft shadow; cursor over a leaf shows a small stylus cursor (desktop only, disabled on touch).

Rules:
- `prefers-reduced-motion: reduce` turns all GSAP motion off: use `gsap.matchMedia()` and show final states instantly.
- Nothing longer than 1.2 s except scroll-linked motion. No autoplay loops except a very slow ambient dust or light drift (opacity under 0.15), and it must pause when the tab is hidden.
- Only animate `transform` and `opacity` where possible. Use `will-change` sparingly. Keep 60 fps on a mid phone: test with CPU throttle 4x in Chrome and report the frame rate.
- All animations must keep keyboard access: a Next/Prev button always exists next to any drag.
- Keep Tamil text real text (not an image, not canvas).
- Add a small `js/motion.js` that holds all timelines with clear names, so later pages just call `Motion.leafFlip(el)`, `Motion.untieBundle(el)`, etc.

## Part C — Emblems page: `website_live/emblems.html`
An isolated page (its own file, linked only from `styleguide.html` for now) showing original SVG emblems of Tamil dynasties and temple architecture. Also save each emblem as its own file in `website_live/emblems/<name>.svg` and `design/svg_kit/emblems/<name>.svg`.

Emblems to draw (all ORIGINAL redraws from the historical motifs, not copies of any specific artwork or flag file):
- Chera: bow and arrow (two-line motif, sometimes with palm).
- Chola: tiger.
- Pandya: twin fish (often shown with a parasol/canopy above and a lamp-stand).
- Pallava: seated bull (Nandi) and lion-pillar motif.
- Later Cholas / Rajaraja era: tiger with bow-and-fish crossed (the three crowned kings together, showing Chera, Chola, Pandya together).
- Muvendar (three crowned kings): combined badge with the three emblems.
- Kalabhra and other small dynasties: only if you find a reliable source; otherwise skip and say so.
- Sangam-era chieftains (Ay, Ori, Pari, Adiyaman): skip unless a source is found.
- Vijayanagara boar: skip (not Tamil dynasty; can be added later).
- Temple: Vimana (Thanjavur Brihadisvara style stepped tower), Gopuram (Dravidian gateway tower), Mandapa pillar, Kalasam (finial), Nandi.
- Sangam-symbols: palm leaf and stylus ("ezhuthani"), the Tamil letter "அ" in Tamil-Brahmi, lamp (kuthuvilakku), conch, spear (vel).
- Coins: Chola tiger coin, Pandya fish coin, Chera bow coin (simple line versions).

Design of each emblem: one SVG, 512 x 512 viewBox, three variants in one file using `<symbol>`: (1) solid single-colour glyph, (2) two-tone line, (3) "material" version shown as carved stone, embossed copper, or ink on leaf. Use the Prompt 1 tokens as CSS variables in the SVG (`currentColor` and `var(--...)`). Keep each SVG under 8 KB. No embedded raster. Give each a `<title>` and `<desc>` and an `aria-label`.

Page layout: a gallery with filters (Dynasty, Temple, Symbols, Coins). Each card shows the emblem on its material, the name in Tamil and English (real text), the period, one line of meaning, and a "Download SVG" button. A big detail view on click, animated with GSAP (Flip plugin: card grows into the detail view). Add a "Muvendar" hero at the top where the three emblems rise and settle in sequence. Add a "How this was drawn" note: original redraws from historical motifs, not official logos.

Honesty rule for content: for every caption (Tamil name, period, meaning), mark it `data-verify="true"` in the HTML and list all of them in `emblems/CONTENT_TO_VERIFY.md` with the source you used. Do not invent sources. If you are not sure of a fact, write "unverified" and leave it out of the visible text.

## Files to create or change
- New: `js/vendor/gsap/*`, `js/motion.js`, `css/materials.css`, `img/tex/*` (only if allowed), `credits-data.json` (if used), `emblems.html`, `emblems/*.svg`, `emblems/CONTENT_TO_VERIFY.md`, `design/svg_kit/emblems/*.svg`.
- Changed: `styleguide.html` (Real materials section, link to emblems page), `js/components.js` (call Motion), `css/components.css`, `DESIGN_SYSTEM.md` (add materials + motion docs).
- Not changed: the 7 old pages.

## Acceptance checks (run all; do not claim a pass you did not run)
1. Opens by double-click (file://) and via `python3 -m http.server`, zero console errors, zero failed requests, no CDN.
2. Screenshots at 360, 768, 1200 px for `styleguide.html` and `emblems.html` saved in `_checks/`, plus a side-by-side "old flat vs new real" for each material.
3. A short screen recording (GIF or MP4) of the leaf flip, bundle untie, ink writing and the emblem card opening, saved in `_checks/`.
4. Reduced motion: with the setting on, no GSAP tween runs and final states show.
5. Performance: report FPS with 4x CPU throttle for leaf flip and scroll story; page weight total; largest image.
6. Contrast: rerun the Prompt 1 table for any changed text-on-texture pair; all body pairs at least 4.5:1.
7. Keyboard: every animated control works with Tab, Enter, Space, arrow keys, Escape; focus ring visible.
8. License: GSAP license file present and summarised in one line; every photo tile listed with SHIP_OK proof; if none used, say so.
9. `git diff --stat` shows only the files listed above; old pages unchanged.
10. Commit in small steps on the branch: "Prompt 1 files", "Real materials", "GSAP motion", "Emblems page".

## Report back
List files, screenshots, recording, FPS numbers, the GSAP license line, what is verified and what is not (browsers, real phone, screen reader), and every fact in `CONTENT_TO_VERIFY.md` that needs a human check.
