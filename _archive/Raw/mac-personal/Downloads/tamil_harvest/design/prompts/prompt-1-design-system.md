# PROMPT 1 — Design system for the Tamil "writing surfaces" site

Copy everything below the line into the dev agent.

---

## Your job
Build the design system (colors, fonts, textures, spacing, components, motion) for the Tamil website in `~/Downloads/tamil_harvest/website_live/`. Do NOT redesign any page yet. Only create the shared style layer and one test page that shows every piece. Pages come in later prompts.

## Before you touch anything (safety)
1. Run `git status`. Commit everything that is not saved yet (design/, engines/, scripts/, new spiders, PROJECT_MAP.md). Do not commit `design/references/` (it is git-ignored on purpose).
2. Create a git branch `redesign-design-system`.
3. Copy `website_live/` to `website_live_backup_2026-09-25/` before editing. Never delete the backup.
4. Do not rename or move the two inner `tamil_harvest` folders.

## Theme
The site is themed on the materials Tamil was written on. Each section of the site has one material:
- Palm leaf (ஓலைச்சுவடி) = literature (Thirukkural, Sangam texts)
- Stone + pottery sherds = origins, earliest script (Tamil-Brahmi)
- Copper plate (செப்பேடு) + seals = kings and royal grants
- Temple-wall stone = Chola section
- Coins + rings = trade (Roman contact)

Give each material its own color set, so a visitor feels the change when moving between sections.

## Files to create
- `website_live/css/tokens.css` — all colors, fonts, spacing, radii, shadows, motion timings as CSS variables. No page may use a raw color; only variables.
- `website_live/css/components.css` — the components below.
- `website_live/css/textures.css` — texture layers (CSS gradients and inline SVG noise only; no big image files).
- `website_live/styleguide.html` — one page that shows every token and component in every state, in light view. Add a toggle "Tamil sample text" that swaps sample text to Tamil.
- `website_live/DESIGN_SYSTEM.md` — short notes: what each token is for and when to use it.

Do not change index.html, tamil.html, scripts.html, grantha.html, vatteluttu.html, font.html, fonts.html in this prompt.

## Color tokens (start from these; adjust only if contrast fails)
Palm leaf (measured from real photos, eyeballed):
- `--leaf-centre: #C98A3F`
- `--leaf-edge: #A86A2C`
- `--leaf-bundle: #4A3426` (stacked leaves seen edge-on)
- `--ink-soot: #1E1712` (dark brown-black script)
- `--thread: #E9A23B` (orange-yellow cotton thread)
- `--catalogue-red: #A33A2A` (small catalogue numbers)
Stone: `--stone-light #B9B4A8`, `--stone-mid #8A857A`, `--stone-dark #4B4842`, plus a warm shadow `#2B2925`.
Copper plate: `--copper #8C5A3C`, `--copper-patina #5E7A66` (green-brown), `--copper-dark #3E2A1E`.
Terracotta (pottery, page background accents): `--terracotta #B5553A`, `--terracotta-soft #E8C9B0`.
Page ground: cream `--paper #F6EEDD`, text `--ink #241B14`.
Coins: `--coin-gold #C9A24A`, `--coin-silver #B7B9BC`.
Rule: body text on any background must reach contrast 4.5:1 or better; large text 3:1. Check each pair and print a contrast table in DESIGN_SYSTEM.md. If a pair fails, change the lighter or darker token, not the rule.

## Typography (free fonts only, self-hosted, `font-display: swap`)
- Tamil body and headings: choose one free Tamil-capable serif with good weights. Recommended: Noto Serif Tamil. Fallback: Noto Sans Tamil.
- Latin display: one serif with italic accents (cream/terracotta museum feel). Recommended: a free serif such as Cormorant Garamond or Fraunces.
- Historical scripts (already noted in the plan): Noto Sans Brahmi for Tamil-Brahmi, and the Adinatha font for Vatteluttu/Grantha if the project already uses it. Check `font.html` and `fonts.html` to see what is in use and reuse it.
- Tamil text must never be smaller than 18px on mobile. Line height for Tamil at least 1.7 (Tamil marks need room).
- Set `lang="ta"` on Tamil blocks so the right font and line breaking apply.

## Textures (must not hurt readability)
- Palm leaf: warm amber gradient from centre to darker edges, very faint horizontal grain lines, uneven slightly rounded ends, tiny nicks on the edge.
- Stone: grey gradient plus SVG noise; carved letters shown as an inset shadow (light top-left edge, dark bottom-right).
- Copper: brown-to-green patina gradient, thin raised rim.
- Terracotta sherd: rough edge, scratched marks.
- Keep every texture under 10 KB. Text always sits on a plain flat area inside the texture, never on the noisy part.

## Components (each with default, hover, focus, active, disabled where it applies)
1. `leaf-strip` — 9:1 ratio strip. Two round holes set in from the ends at about 30% and 70%. Text area avoids the holes. Holds one Kural (2 short lines of Tamil, transliteration, meaning). Must also work at 3:4 tall on phones (rotate to vertical reading, holes at top and bottom).
2. `leaf-bundle` — stack of leaves seen edge-on: dark brown block with fine horizontal lines, wrapped with orange thread (vertical wraps and an X crossing), a small paper label, plain wooden boards top and bottom. Used as a navigation card.
3. `stone-slab` — grey slab with carved-letter heading, used for Chola and origins content.
4. `copper-plate` — plate with a ring through one edge and a small round seal, used for kings and grants.
5. `coin-card` — round coin (gold or silver) with a caption band.
6. `ring-seal` — signet ring and seal impression.
7. `sherd-card` — irregular terracotta shard with a scratched mark and a caption.
8. Plain UI pieces: button (primary in terracotta, secondary outlined), link, tag/chip, source-citation line, breadcrumb, top navigation, footer, figure with credit line, quote block.
9. `credit-line` — small text under any image: object name, license, credit. Data comes from `design/references/LICENSES.csv`.

Each component must be plain HTML + CSS and reusable by adding a class. No framework.

## Motion (all optional and short)
- Leaf flip: next/previous leaf turns like a page, 350 ms, ease-out.
- Thread untie: on bundle click, the X thread slides away, 500 ms, then the bundle opens.
- Hover: a small lift of 2 px with a soft shadow, 150 ms.
- `prefers-reduced-motion: reduce` must turn ALL motion off (instant state change).
- No autoplay motion, no motion longer than 600 ms.

## Layout and mobile
- Mobile first. Design at 360 px wide first, then 768 px, then 1200 px. No horizontal scroll at any width.
- Spacing scale: 4, 8, 12, 16, 24, 32, 48, 64 px, as variables.
- Touch targets at least 44 x 44 px.
- Max reading width 68 characters for Tamil and Latin text.

## Accessibility
- Every interactive part works with keyboard only; visible focus ring (3 px, high contrast).
- Every decorative texture or SVG has `aria-hidden="true"`; every meaningful image has alt text.
- Tamil text is real text, never an image.
- Do not rely on color alone to show state.

## Images and licenses
- Do not add photos in this prompt. Textures and components are drawn in CSS and inline SVG.
- If you need a reference, look in `design/references/<surface>/` and `design/specs/palm_leaf_visual_notes.md` for look only. Only files marked SHIP_OK in `LICENSES.csv` (public domain, CC0, CC BY) may ever be shipped, and they need a credit. CC BY-SA, CC BY-NC and unclear files are study only.

## Acceptance checks (do all, then report)
1. `styleguide.html` opens by double-click with no server and shows every token and component with no console errors.
2. It looks right at 360, 768 and 1200 px wide (take a screenshot of each and save them in `website_live/_checks/`).
3. Contrast table is in `DESIGN_SYSTEM.md` and every body-text pair passes 4.5:1.
4. Reduced-motion test: with the setting on, nothing animates.
5. Keyboard test: Tab reaches every button and link in order, focus ring visible.
6. Old pages (`index.html` etc.) are unchanged: run `git diff --stat` and show that only the new files were added.
7. Total CSS added is under 60 KB, and no external requests except self-hosted fonts.

## Report back
Reply with: list of files created, the 3 screenshots, the contrast table, and anything you could not do. Say clearly what is verified and what is not. Do not claim a check passed unless you ran it.
