# Nalaas — website

Marketing site and product catalogue for **Nalaas**, the in-house brand of
Christy Quality Foods (India) Pvt Ltd, Chithode, Erode.

Plain HTML, CSS and JavaScript. **No build step, no npm, no framework.**

---

## Run it

Open `index.html` in a browser. That's it.

For nicer development use the VS Code **Live Server** extension
(right-click `index.html` → *Open with Live Server*) so you get auto-reload
and clean URLs.

---

## Structure

```
nalaas-website/
├── index.html              home
├── pages/
│   ├── products.html       catalogue, 105 SKUs, search + filter
│   ├── product.html        product detail   → product.html?id=<slug>
│   ├── recipes.html        recipe index
│   ├── recipe.html         recipe detail    → recipe.html?id=<slug>
│   ├── about.html          our story
│   ├── quality.html        machinery, lab, standards
│   └── contact.html        contact + dealer enquiry
├── css/
│   ├── nalaas.css          design system — tokens, components, forms
│   └── home.css            home-page-only sections
├── js/
│   ├── site.js             header + footer, injected on every page
│   ├── cards.js            product + recipe card renderers
│   ├── catalogue.js        search and category filtering
│   ├── product.js          product detail renderer
│   └── recipe.js           recipe detail renderer
├── data/
│   ├── products.js         105 products    (generated)
│   └── recipes.js          12 recipes      (generated)
├── scripts/
│   └── build-data.mjs      regenerates the two data files
└── assets/
    ├── brand/              logo.png, favicon.png
    └── images/             product, recipe, infrastructure photography
```

---

## Design tokens

Everything lives in `:root` at the top of `css/nalaas.css`.
Colours were sampled from the official logo and cross-checked against the
Home Branding Kit:

| Token | Value | Use |
|---|---|---|
| `--red` | `#B80818` | primary actions, accent words — white text OK |
| `--amber` | `#F9AE19` | secondary surfaces — **black text only** |
| `--yellow` | `#F8E800` | highlight / marker sweep — **black text only** |
| `--orange` | `#F07800` | borders, gradients, icons |
| `--desc` | `#616161` | body copy |
| `--cream` | `#FFF6DE` | section bands |

**Contrast rule:** amber and yellow never carry white text — they fail
WCAG AA. Red surfaces take white, amber and yellow surfaces take black.

Type is **Rubik** (from the branding kit) at the size ramp taken from the
Figma template: 64 / 40 / 26 / 20 / 18 / 16 / 14.

Three brand devices repeat across the site, all derived from the logo:
the **oval** became the section eyebrow chip, the **yellow field** became
the marker sweep behind accent words, and the **wheat ear** became the
section divider.

---

## Editing content

**Products and recipes are data, not markup.** Never hand-edit
`data/products.js` — edit `scripts/build-data.mjs` and regenerate:

```bash
node scripts/build-data.mjs
```

Each product record looks like this:

```js
{
  id: "wheat-flour-atta",
  category: "flours",
  title: "Chakki Fresh Atta",
  tagline: "The bran and the germ, kept in.",
  body: "…",
  ingredients: [...],
  nutrition: { Energy: "341 kcal", … },
  packSizes: [],        // ← client to supply
  mrp: null,            // ← client to supply
  image: "assets/images/products/flours/wheat-flour-(atta).jpg",
  needsCopy: false
}
```

Products with `needsCopy: true` render a visible red flag on their detail
page so gaps can't ship silently.

---

## Known gaps

| Gap | Detail |
|---|---|
| **85 of 105 products have no description** | Only 20 SKUs had copy on either client site. All of it came from the *legacy* `nalaas.christygroup.in`, never the live one. |
| **No pack sizes, no prices** | Neither site publishes a single pack size or MRP. Client must supply. |
| **2 broken source images** | `karupu-kavuni-arisi` and `rice-flakes` popups 404 on the live site. Both fall back to a generated pack graphic. |
| **8 of 12 recipes have no method** | Carried over as titles only. On the current site just 4 of 22 recipes name a Nalaas product. |
| **Contact form posts nowhere** | Wire to Formspree, Netlify Forms or a PHP handler. |
| **Nutrition needs verification** | Health Mix selenium reads `133.3 mg` per 100 g on the source site — almost certainly a µg/mg error. Confirm before any label or print use. |
| **No FSSAI licence number** | Certification is claimed but the number is never shown. Legally expected on food e-commerce. |

---

## Assets

Photography was recovered from the client's live site and re-encoded to
max 900 px at JPEG q0.82: **138.3 MB → 28.5 MB, a 79% reduction**, with no
visible loss at rendered sizes. `_manifest.csv` in the asset drop maps every
original path to its optimised file with before/after byte counts.

For reference, four bottles of oil on the current site total roughly 50 MB.

---

## Next

- Fill the 85 missing product descriptions
- Get pack sizes and MRP from the client
- Rewrite the 8 stub recipes around the packs they should sell
- Add `Product`, `Recipe`, `Organization` and `LocalBusiness` structured data
- Wire the enquiry form
- Consider consolidating the duplicate `products/` and `products/popup/`
  image sets — 210 files for 105 products, mostly the same photo twice
