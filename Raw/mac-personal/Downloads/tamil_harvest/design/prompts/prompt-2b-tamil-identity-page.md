# PROMPT 2b — Tamil Identity page (REPLACES Part C of Prompt 2)

Do Parts A and B of Prompt 2 as written. Ignore Part C of Prompt 2 and do this instead. The owner said: the page must show the WHOLE Tamil identity (people, temples, books, land, culture), not only dynasty emblems.

## Page and files
- Page: `website_live/identity.html` (isolated; linked only from `styleguide.html` for now). Title: "Tamil Identity" / "தமிழ் அடையாளம்".
- One SVG per item: `website_live/identity/<group>/<slug>.svg` and a copy in `design/svg_kit/identity/<group>/<slug>.svg`.
- Data file: `website_live/identity/identity.json` with one row per item: id, group, slug, Tamil name, English name, period, one-line meaning, source, `verified` (true/false). The page reads this file to build the gallery. (Because file:// blocks fetch, also embed the same data in `identity/identity-data.js` as `window.IDENTITY_DATA`.)
- Human check list: `identity/CONTENT_TO_VERIFY.md` (every Tamil name, date, meaning, and its source).

## Drawing style (one style for all, so it looks like one family)
- 512 x 512 viewBox. Same stroke weight (16 units), same corner style, same margins (32 units).
- Three variants per item in one file using `<symbol>`: (1) solid glyph, (2) two-tone line, (3) "material" version: carved on stone, embossed on copper, or inked on palm leaf (use the Prompt 1 tokens and Prompt 2 materials).
- Under 8 KB each. No embedded raster. `<title>`, `<desc>`, `aria-label` on each.
- All ORIGINAL redraws from historical motifs and public descriptions. Do not trace any photo, statue, flag, or logo file. Do not copy Wikipedia or Commons SVGs.

## Groups and items (build in 3 batches; finish and report after each batch)

### Batch 1 — core identity (about 40 items)
1. Dynasties: Chera (bow and arrow), Chola (tiger), Pandya (twin fish, with canopy), Pallava (Nandi bull and lion pillar), Muvendar combined badge.
2. People (symbolic figures, see rule below): Thiruvalluvar (seated sage, palm leaf and stylus), Avvaiyar (elder woman with staff), Tolkappiyar, Ilango Adigal, Kambar, Nakkirar, Kapilar, Bharathiyar (deceased poet; symbolic silhouette with turban only), Rajaraja Chola I, Rajendra Chola I, Karikala Chola, Kulothunga Chola I (each as a crown + emblem cartouche, not a portrait), Tamil Thai (Mother Tamil as a concept: seated woman with crown and palm leaf; original design).
3. Books (each drawn as a leaf manuscript with the title in real Tamil text): Thirukkural, Tolkappiyam, Silappathikaram, Manimekalai, Purananuru, Akananuru, Aingurunuru, Natrinai, Kuruntokai, Pathitrupathu, Paripadal, Kalittokai, Thirumurugatrupadai, Pattinappalai, Thevaram, Naalayira Divya Prabandham, Kamba Ramayanam, Periya Puranam.
4. Script: the letter அ in modern Tamil, Tamil-Brahmi, Vatteluttu, and Grantha (reuse the project fonts); the aytham ஃ; the 12 vowels and 18 consonants as a grid glyph.

### Batch 2 — places, land, symbols (about 45 items)
5. Temples and architecture (original line drawings, not photo traces): Brihadisvara Temple, Thanjavur (vimana); Gangaikonda Cholapuram; Airavatesvara, Darasuram; Meenakshi Amman, Madurai (gopuram); Shore Temple, Mahabalipuram; Kailasanathar, Kanchipuram; Nataraja temple, Chidambaram; Srirangam Ranganathaswamy gopuram; Rameswaram corridor pillars; generic Dravidian gopuram, vimana, mandapa pillar, kalasam finial, Nandi, temple tank.
6. Sacred and cultural icons: Nataraja (Chola bronze silhouette, original), Murugan's vel (spear), kuthuvilakku (lamp), conch, kolam pattern, Pongal pot with sugarcane, Jallikattu bull (neutral, no people), Bharatanatyam pose, Silambam staff, Nadaswaram and thavil, Parai drum, Yazh (ancient harp), Villupattu bow, Tanjore painting frame, Kanchipuram silk border pattern, Chola bronze lost-wax mould, Kavadi.
7. Five lands (Aintinai): Kurinji (hill, kurinji flower), Mullai (forest, jasmine), Marutham (farmland), Neithal (coast, neithal flower), Paalai (dry land, palm). Each with its flower and its landscape in one badge.
8. Nature of Tamil Nadu: palmyra tree, glory lily (kantal), Nilgiri tahr, emerald dove, banyan, banana leaf.
9. Sites and heritage: Keeladi, Adichanallur, Arikamedu, Poompuhar (Kaveripattinam), Korkai (pearls), Muziris, Kallanai (Grand Anicut), Pamban bridge, Nilgiri mountain railway, Kodaikanal, Kanyakumari (sunrise over the sea).

### Batch 3 — trade, time, more (about 25 items)
10. Trade and sea: Chola sailing ship, Roman amphora, Roman coin, pepper, pearl, muslin cloth, ivory, teak.
11. Time: Tamil months wheel (Chithirai to Panguni), Tamil New Year (Puthandu), Thai Pongal sun, Karthigai lamps, Aadi Perukku river.
12. Daily life: banana-leaf meal, filter of plain rice and sambar bowl (no brands), palm-leaf fan, wet grinder, bullock cart.
13. Anything else you find with a reliable source that clearly belongs to Tamil identity. Add it and list it in the report.

## Rules for people, gods and sensitive items
- No portrait or likeness of any real person. There is no authentic portrait of Thiruvalluvar, Avvaiyar or the ancient poets: draw symbolic figures (pose, clothing, palm leaf, stylus) and say so in the caption: "symbolic, not a portrait".
- Do not copy or trace the Thiruvalluvar statue at Kanyakumari, the Tamil Thai statue, or any modern monument or temple photo (copyright unclear). Draw the idea from scratch.
- Do not draw the official Tamil Nadu government emblem, any state or national flag, any political party symbol or person, any company logo or brand.
- Religious items (Nataraja, vel, Vaishnava and Shaiva marks) are drawn respectfully and neutrally; no political or comparison text.
- Do not draw Indus seals (contested Indus-Tamil claim; owner decision).
- Every fact you write (Tamil name, dates, meaning) needs a real source. If you are not sure, write "unverified" in `identity.json`, hide it on the page, and list it in `CONTENT_TO_VERIFY.md`. Never invent a source.

## Page design
- Hero: three emblems (Chera, Chola, Pandya) rise and settle in sequence with GSAP (Prompt 2 motion rules).
- Filter bar: Dynasties, People, Books, Script, Temples, Culture, Land, Nature, Sites, Trade, Time, Daily life. Search box that works on Tamil and English.
- Gallery cards: emblem on its material, Tamil name (real text, at least 18 px), English name, period, one-line meaning, "symbolic" badge where needed.
- Click a card: it grows into a detail view (GSAP Flip). Detail shows all three variants, a Download SVG button, source line, and "verify" status.
- Keyboard: arrow keys move between cards, Enter opens, Escape closes.
- Mobile first: 2 columns at 360 px, 3 at 768, 5 at 1200.
- Performance: lazy-load SVGs, inline only the visible ones; page under 1.5 MB.

## Acceptance checks (run all; do not claim a pass you did not run)
1. Batch 1 first: build the 40 items and the page, then stop and report. Continue to batch 2 and 3 only if the report is accepted (the owner will say "continue").
2. Opens by double-click (file://) and via `python3 -m http.server`; zero console errors and failed requests.
3. Screenshots at 360, 768, 1200 px, a contact sheet of all icons at one size (`_checks/identity-sheet.png`), and a screen recording of the hero and the card open animation.
4. Every SVG under 8 KB, valid, has title/desc/aria-label; run an SVG validator and report.
5. Every item appears in `identity.json` and in `CONTENT_TO_VERIFY.md` with a source or "unverified".
6. Search test: search "வள்ளுவர்", "Chola", "கோவில்" and show results.
7. Reduced motion and keyboard checks as in Prompt 2.
8. `git diff --stat` shows only new files plus the files named in Prompt 2. Old pages unchanged. Commit per batch.

## Report back
Counts per group, contact sheet, list of unverified facts, list of items you skipped and why, and what is not tested (Safari, Firefox, real phone, screen reader).
