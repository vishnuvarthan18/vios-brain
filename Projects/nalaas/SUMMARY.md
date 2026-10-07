---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Nalaas website

## 1. What this project is
- **Goal:** Marketing site and product catalogue for **Nalaas**, the in-house food brand of Christy Quality Foods (India) Pvt Ltd, Chithode, Erode.
- **Who it is for / client:** Christy Quality Foods (client). Visitors are shoppers and dealers.
- **Why it exists:** The client's current sites show little product info and very heavy images. The new site puts 105 products and 22 recipes in one clean catalogue.

## 2. Status now (as of 2026-10-06)
- Unasked demo site built and live at nalaas-website.pages.dev (Cloudflare Pages): 105 products, 22 recipes (earlier README count: 12). Pitched to the client on 17 Aug 2026. No reply or work noted since.
- Built as plain HTML, CSS and JavaScript. No build step, no framework.
- Pages: home, products (105 SKUs, search + filter), product detail, recipes, recipe detail, about, quality, contact + dealer enquiry.
- Products and recipes are data files made by `scripts/build-data.mjs`. Products missing copy show a red flag.
- Design tokens taken from the Nalaas logo and branding kit (red, amber, yellow, orange; Rubik font).
- Photos from the client's live site re-compressed: 138.3 MB to 28.5 MB (79% smaller).
- Not done: 85 of 105 products have no description; no pack sizes or prices; some recipes have no method (README: 8 of 12); contact form posts nowhere; no FSSAI licence number shown; some nutrition values need checking.

## 3. Next steps
1. Follow up on the 17 Aug pitch.
2. Get product descriptions, pack sizes and MRP from the client.
3. Write the stub recipes.
4. Wire the enquiry form (Formspree, Netlify Forms or PHP).
5. Add structured data (Product, Recipe, Organization, LocalBusiness).
6. Remove duplicate product image sets.

## 4. Decisions
- (date not known) — Plain HTML/CSS/JS, no build tools — easy to open and edit. #decision
- (date not known) — Products and recipes kept as generated data, not hand-written markup. #decision
- (date not known) — Amber and yellow never carry white text (fails WCAG AA). #decision

## 5. Timeline
- 2026-08-17 — Demo site pitched to the client; last commit in the `nalaas-website` repo ("Ignore wrangler local cache directory"); live on nalaas-website.pages.dev.

## 6. Key facts
- **Companies:** Christy Quality Foods (India) Pvt Ltd (client, Chithode, Erode)
- **Tools:** [[Tools/GitHub]], [[Tools/VS Code]], [[Tools/Figma]], [[Tools/Wrangler]], [[Tools/Cloudflare]]
- **Links / repos / servers / file paths:** Demo: nalaas-website.pages.dev; GitHub `vishnuvarthan18/nalaas-website` (private); Mac `~/Vishnu/nalaas-website`. Old client sites: live Nalaas site and legacy `nalaas.christygroup.in`.
- **Related:** [[Projects/vidivu/SUMMARY]] (client web work)

## 7. Files and documents
- `README.md` — structure, design tokens, known gaps, next steps (README (archived: Raw/mac-personal/Vishnu/nalaas-website/README.md))

## 8. Open questions and problems
- Did the client reply to the 17 Aug pitch? (demo was built without being asked, so not paid work yet)
- Will it move to a real domain?
- Health Mix selenium value (133.3 mg) is likely a unit error; client must confirm.

## 9. All chats in this project
- None imported yet.
