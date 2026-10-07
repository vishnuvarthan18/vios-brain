---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Halle website (B. Halle Webflow UI fix)

## 1. What this project is
- **Goal:** Fix the UI of the B. Halle website in [[Tools/Webflow]] (spacing, colours, text sizes, buttons, responsiveness) by first making one clean standard (style guide) and then fixing pages against it.
- **Who it is for / client:** [[Companies/HALLE]] (B. Halle Nachfl. GmbH, b-halle.de). Built by [[Companies/araCreate Group]].
- **Why it exists:** Vishnu said "the problem is fully with the UI, on every page". The [[Tools/Figma]] design itself is also not consistent, so the Webflow site copied the mess.

## 2. Status now (as of 2026-10-06)
- Last design work: 2026-09-09. No new work on the site since then.
- Earlier (Jul 13–17, Claude Code): home hero curve (exact Figma SVG), hero/logos split, commemorative card made clickable, text hierarchy plan written (not yet done).
- Done: UI audit of the Webflow site (18 pages) and the Figma file.
- Done: "Standard values" sheet (colours, type scale, line height, icons, spacing, corners, letter spacing, grid, button/input specs).
- Done: Figma "Design System" page with tokens (variables), 8 text styles and 8 components (Button, Input Field, Product Card, Navigation Bar, Text Link, Tag, Alert, Dropdown).
- Done: 3 "Standardized (Draft)" frames in Figma (Home, Contact, Polarizers). Originals not touched.
- Done: Webflow Style Guide page (`/style-guide`) with "Design Tokens" variables and `sg-` classes. Only this page was edited.
- Done (2026-09-09): line-height cleanup (all 18 overrides removed, one `normal` on page wrapper), buttons matched to Figma, responsive rules table added.
- Done: whole site published to the staging domain halle-dev.webflow.io on 2026-09-09; style guide at halle-dev.webflow.io/style-guide for dev team handoff.
- Not done: Vishnu's own visual check of the published page.
- Not done (left to dev team): hover/focus/disabled states, mobile menu, hero on mobile.
- Not done: real fixes on the other pages (hidden nav links, 3 H1s on Home, hidden duplicate sections, unfinished "new-home" page).
- Not done: font is still a stand-in (Inter / system font), not real Helvetica Neue. Some icons and product photos still placeholders.

## 3. Next steps
1. Vishnu checks halle-dev.webflow.io/style-guide visually.
2. Hand style guide to the dev team; team builds hover/focus/disabled states, mobile menu and mobile hero.
3. Someone with Helvetica Neue installed swaps the font back in Figma (one select-all change), and the font file is added to Webflow.
4. Team signs off the new states Claude guessed (hover, focus, error, disabled).
5. Use the style guide to fix the other pages (nav, H1s, hidden sections) and decide: finish "new-home" or fix current Home.
6. Get real icons and product photos.

## 4. Decisions
- 2026-09-08 — Fix Figma values first, then build the style guide, then fix pages, then test all screens. — A style guide built on a messy design copies the mess. #decision
- 2026-09-08 — Use the nearest real Figma value, do not invent new ones ("the nearest value from the figma, not much ahead"). #decision
- 2026-09-08 — Build real Figma Variables + Components (option 1), not page-by-page value patching. — Patching does not stop new drift. #decision
- 2026-09-08 — Official brand palette from the "branding-guidance" Figma page: Navy #2A318A (#29308A in use), Dark Gray #2A2924, White, Light Blue #B5E0FA, Pale Blue #D3EDFC. #787747 removed as an accident. #decision
- 2026-09-08 — Add standard status colours (red/green). Motion = small, minimal only. Tablet/mobile not designed yet, adapt from desktop later. #decision
- 2026-09-08 — Webflow work only on the new Style Guide page; never change any other page. Never change the design, only the values. #decision
- 2026-09-08 — Form Fields section removed from the style guide ("no need forms, delete that"). #decision
- 2026-09-08 — Button sizes follow the rule book, not raw Figma numbers: small 40px, large 50px. #decision
- 2026-09-08 — No em dashes (—) on the style guide page; text humanised. #decision
- 2026-09-09 — Remove ALL line-height overrides (18 styles); only one `normal` on the page wrapper. — Button text looked off-centre with line-height 1. #decision
- 2026-09-09 — Rule: "Figma is a starting point, use the nearest correct value." Corners 8px (40px buttons) / 12px (50px buttons); spacing snapped to 8-scale; button text sizes unified. #decision
- 2026-09-09 — Success green kept at #1B8038 (passes contrast). #decision
- 2026-09-09 — Match Figma for buttons: Contact Us white bg / navy text, all buttons Regular weight, exact Ionicons arrow, soft shadow on Go To Products, Contact Us icon 36px to 24px. #decision
- 2026-09-09 — Hover/focus/disabled states, mobile menu and mobile hero left to the dev team. #decision

## 5. Timeline
- 2026-09-09 — Line-height fix, buttons matched to Figma, responsive rules table added; whole site published to halle-dev.webflow.io for dev handoff.
- 2026-09-08 — Progress notes saved to project (`halle-webflow-style-guide-progress.md`).
- 2026-09-08 — Many bugs found by Vishnu and fixed on the style guide: empty text boxes, wrong form design, uneven text spacing, link underlines, uneven button heights, wrong arrow icon.
- 2026-09-08 — Webflow Style Guide page built (Draft, not in nav), then reworked to two-column layout from Vishnu's reference templates (Mitchy, Duotint).
- 2026-09-08 — Figma Design System page built (tokens, text styles, 8 components); contrast fixes (placeholder grey, success green).
- 2026-09-08 — 3 standardized draft frames built in Figma; colour mistake (#B5E0FA) found and fixed.
- 2026-09-08 — Figma data pulled from Home, Contact, Polarizers frames; standard values sheet written.
- 2026-09-08 — First Webflow audit: 2 hidden nav links, 3 H1s on Home, hidden duplicate logo sliders, half-hidden sections, unfinished "new-home" page.
- 2026-07-17 — Commemorative publication card made fully clickable with hover; text hierarchy audit and safe fix plan written.
- 2026-07-14 — Home v2 dropped; hero and trust-logos split into two sections; logo gap ~21px.
- 2026-07-13 — Hero curve now uses the exact Figma SVG; published to halle-dev staging.

## 6. Key facts
- **People:** [[People/Vishnu]] — owner, works with the client and the dev team.
- **Companies:** [[Companies/HALLE]] — client; [[Companies/araCreate Group]] — builder.
- **Tools:** [[Tools/Webflow]], [[Tools/Figma]], [[Tools/Claude]].
- **Links / repos / servers / file paths:**
  - Live domain: b-halle.de. Webflow staging: halle-dev.webflow.io. Style guide: halle-dev.webflow.io/style-guide.
  - Figma file: figma.com/design/A7xoUgwhqye2UmZ7R2Obfa/www.b-halle.de
  - Figma frames: Home 3148-5833, Contact 3158-4303, Polarizers 3166-9896, Achromatic Retarders 4379-5571 (only partly read).
  - Figma "Design System" page: node 374-362. Home draft: node 5626-1495.
  - Webflow site id 6672e259ffca23748c51b4cd; Style Guide page id 6aa053018c44eb05710cb998.
- **Standard values (short):** Font Helvetica Neue (Light/Regular/Medium/Bold). Type scale 42 / 26 / 24 / 22 / 20 / 18 / 16 / 12 px. Spacing 8, 16, 20, 24, 32, 40, 48, 64, 80. Corners 4 / 8 / 12 / pill. Icons 18 / 24 / 32 / 36. Grid 1440 page, 80px side margin, 1280 content. Buttons 40px and 50px tall. Input 56px. Placeholder grey #737373. Error #D93025. Success #1B8038.
- **Responsive rules (2026-09-09):** breakpoints 992 / 768 / 480; side margins 80 / 40 / 24 / 16; H1 42 / 36 / 32 / 28 (more rows in the Style Guide table).
- **Brand reference:** B. Halle Optik (Berlin) logo design guide V01, May 2024 (B-Halle-Optik-Logodesign-Guide-V01.pdf (archived: Raw/mac-personal/Vishnu/projects /creative work/Branding/B-Halle-Optik-Logodesign-Guide-V01.pdf.md)).
- **Dev history:** [[Projects/halle-web/DEV-LOG]] (Claude Code sessions, office Mac). Server work from 2026-10-05 (Kishor's SSH key, tunnel to :3000) was moved to [[Projects/feedback-widget/SUMMARY]].
- **Chats index:** INDEX (archived: Projects/halle-web/chats/INDEX.md)
- **Related:** [[Projects/ac-ds/SUMMARY]] (design system work, link not sure)

## 7. Files and documents
- `halle-standard-design-values.md` — the standard values sheet (Claude project doc).
- `halle-webflow-style-guide-progress.md` — progress notes, button numbers, mistakes, tool quirks (Claude project doc).
- `halle-design-system-next-steps.md` — priority list: done vs needs the team (Claude project doc).
- `halle-style-guide-plan.md` — first 5-step plan (sent as file in chat).
- Figma "Design System" page and 3 "Standardized (Draft)" frames (in the Figma file).

## 8. Open questions and problems
- Font: Helvetica Neue not available to Claude; Figma and Webflow use stand-in fonts until someone swaps it.
- Hover/focus/error/disabled states were guessed by Claude; team must confirm.
- Contact page draft in Figma: footer address text overlaps (side effect of Inter font).
- Only 3 Figma pages checked; the file has 50+ pages. Others may have the same drift.
- Many repeated bugs from Claude during the build (empty text, wrong nesting, browser defaults left unset). Vishnu was frustrated ("lot of bug"). Needs a careful visual check.
- On 2026-09-08 the exact arrow icon could not be downloaded from Figma (blocked); the 2026-09-09 chat says the exact Ionicons arrow is now used.
- Does the site use Checkbox, Radio or Modal? Not confirmed.
- Fixes on the other pages not started.
- Text hierarchy plan from 2026-07-17 (Phase 1–3) not carried out yet.

## 9. All chats in this project
- Webflow vs custom code decision (archived: Projects/halle-web/chats/2026-06-18 Webflow vs custom code decision.md) — 2026-06-18
- Webflow connection setup (archived: Projects/halle-web/chats/2026-07-13 Webfloe connection setup.md) — 2026-07-13
- Halle website UI analysis (archived: Projects/halle-web/chats/2026-09-08 Halle website UI analysis.md) — 2026-09-08
- Button line height issue (archived: Projects/halle-web/chats/2026-09-09 Button line height issue.md) — 2026-09-09
