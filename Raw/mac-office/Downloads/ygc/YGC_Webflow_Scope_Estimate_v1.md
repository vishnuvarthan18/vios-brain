# YGC — WordPress to Webflow Migration
## Internal Scope & Estimate — v1

- **Client:** Your German Company (YGC) — yourgermancompany.com
- **Project:** Move website from WordPress (Divi) to Webflow
- **Prepared by:** araCreate Group
- **Date:** 11 September 2026
- **Status:** Internal draft. Not for client. Add rates before sending out.

---

## 1. What we know today

### Current site (audited from live site + sitemap)

- Platform: WordPress with Divi theme v4.27.8
- SEO plugin: Yoast
- Pages: **11** (Home, Services, About Us, Contact, Blog, Disclaimer, Imprint, Privacy, plus 3 old/sample pages)
- Blog posts: **3** (2020–2021, dated URL structure)
- Language: English only. No language switcher.
- Nav: Home, Services, About Us, Contact, "Get in touch" CTA
- Footer: Company links, 4 external useful links, 3 legal links, 5 social icons
- Contact form is complex: name, contact method (8 options), email, phone + country code, 4 availability slots, help category dropdown, message
- Services page is one long page. ~1,800–2,000 words. 9 service blocks. No service detail pages.
- Dead weight to kill or redirect: `/old-services/`, `/services/sample-service/`, `/services/sample-service-category/`

### Mockups supplied (3 PNGs in the ygc folder)

| File | What it covers |
|---|---|
| `1_ygc.png` | Homepage wireframe, desktop, sections 01–06 with goals |
| `2_ygc.png` | Homepage full, 10 sections, split 3 parts + Webflow build guide strip |
| `3_ygc.png` | Services page wireframe with full build spec (classes, px, hex) |

### Design direction already fixed in the mockups

- Canvas: 1440px container, 12-column grid, 120–160px section spacing
- Type: Playfair Display (headlines, serif) + Inter (body, sans)
- Colours: `#111111` dark, `#FAF9F7` off-white, `#E10600` red, `#F7941D` orange (gradient accent), `#D6D6D6`, `#E9E9E9`, `#6B6B6B`, `#4A4A4A`
- Animation brief: subtle scroll reveals only, no heavy effects
- Named classes already given: `section-hero`, `section-intro`, `section-services`, `section-cta`, `footer`

**This is good news.** The design tokens and class naming are decided, so the Webflow style system can be built without guesswork.

---

## 2. Page inventory for the new site

| # | Page | Designed? | Type |
|---|---|---|---|
| 1 | Home (10 sections) | Yes | Static |
| 2 | Services overview (5 sections) | Yes | Static |
| 3 | Service detail — Market Entry & Business Setup | **No** | CMS template |
| 4 | Service detail — Operating Model & Org Design | **No** | CMS item |
| 5 | Service detail — Governance & Op Compliance | **No** | CMS item |
| 6 | Service detail — Cross-Border & Group Integration | **No** | CMS item |
| 7 | About | **No** | Static |
| 8 | Insights listing | **No** | CMS list |
| 9 | Insight detail | **No** | CMS template |
| 10 | Contact + form | **No** | Static |
| 11 | Imprint | **No** | Static |
| 12 | Privacy | **No** | Static |
| 13 | Disclaimer | **No** | Static |
| 14 | 404 + password page | **No** | Utility |

- **2 of 14 pages are designed.** Mockups are desktop only — no tablet or mobile.
- Homepage sections 06 and 07 both show "Learn more →" links, so service detail pages are implied by the design but not drawn.

### CMS collections needed

- **Insights** — title, slug, date, category, hero image, thumbnail, body, SEO fields, author
- **Services** — number, category label, title, short description, hero image, body, order
- Optional: Categories collection if insights need filtering

---

## 3. Effort estimate (hours)

### Phase 0 — Discovery & setup — 8h

- Kickoff call, confirm scope and open questions — 2
- Collect copy, images, brand assets, logo files — 3
- Webflow workspace, site creation, plan selection, access setup — 3

### Phase 1 — Design completion — 34h

*Only if araCreate does the missing design. Drops to 0 if client's designer delivers.*

- Mobile + tablet adaptation of Home and Services — 10
- Service detail page template design — 8
- About, Contact, Insights list, Insight detail, legal template — 16

### Phase 2 — Webflow foundation — 20h

- Design tokens: variables for colour, typography scale, spacing — 6
- Layout system: container, 12-col grid, section padding classes — 4
- Global components: navbar, footer, buttons, links, cards, eyebrow labels — 6
- CMS collections and field setup — 4

### Phase 3 — Page build — 64h

- Home — 10 sections, 4 breakpoints — 20
- Services overview — 5 sections — 8
- Service detail template + 4 CMS entries — 8
- About — 6
- Insights listing + detail template — 8
- Contact page + full form build — 8
- Legal pages ×3 — 4
- 404 and utility pages — 2

### Phase 4 — Interactions & responsive — 18h

- Scroll reveals, nav scroll behaviour, hover and arrow states — 8
- Responsive QA and fixes across all breakpoints — 10

### Phase 5 — Migration & SEO — 19h

- Migrate 3 blog posts with images into CMS — 3
- Port SEO titles and meta descriptions from Yoast — 4
- 301 redirect map (~15 URLs incl. dated post permalinks) — 3
- Sitemap, robots.txt, OG images, favicon — 3
- Analytics / GTM + GDPR cookie consent banner — 6

### Phase 6 — QA, launch, handover — 36h

- Cross-browser and device QA — 8
- Accessibility pass: contrast, alt text, heading order, focus states — 6
- Client review rounds (2 included) and revisions — 12
- DNS cutover, SSL, form delivery testing, post-launch checks — 5
- Handover doc + training video for CMS editing — 5

### Totals

| Scenario | Hours | Days (8h) |
|---|---|---|
| **Full scope (we do the missing design)** | **199h** | ~25 days |
| Build only (client supplies all design) | 165h | ~21 days |

---

## 4. Timeline

Assumes 1 designer + 1 Webflow developer, partly parallel.

| Week | Work |
|---|---|
| 1 | Discovery, asset collection, design of missing pages starts |
| 2 | Design completion + Webflow foundation (tokens, components, CMS) |
| 3 | Home + Services build |
| 4 | Remaining pages, CMS templates, forms |
| 5 | Interactions, responsive, migration, SEO |
| 6 | QA, client review round 1 + 2, fixes |
| 7 | Launch, DNS cutover, handover |

**~6–7 weeks calendar.** Add 1 week buffer if copy or images arrive late — this is the most common delay.

---

## 5. Assumptions

- English only. No German version. (Webflow Localization is extra cost + effort if needed.)
- Final copy supplied by client. We do not write copy.
- Architecture photography supplied and licensed by client. The mockup images are placeholders.
- Client pays Webflow hosting and workspace fees directly.
- 2 rounds of revisions included.
- Standard Webflow CMS is enough — no e-commerce, memberships, or logged-in areas.
- Contact form submits to email / Webflow form handler. No CRM integration.
- Blog stays small (3 posts). No bulk import tooling needed.

---

## 6. Out of scope

- Copywriting and content strategy
- Photography, licensing, illustration
- German or any other language version
- CRM, marketing automation, or ERP integration
- WordPress decommissioning and hosting cancellation
- Ongoing maintenance and support retainer
- Custom code beyond minor embeds
- Email/newsletter system setup

---

## 7. Risks and open questions

### Risks

| Risk | Impact | Handling |
|---|---|---|
| 12 of 14 pages have no design | High — biggest unknown in the estimate | Confirm who designs before quoting |
| Desktop-only mockups | Medium — mobile decisions get made during build | Price mobile design in, or get client sign-off on dev-led responsive |
| Complex contact form (8 methods, time slots, conditional feel) | Medium | Webflow native forms are limited for conditional logic. May need a form tool or custom JS. |
| German client, GDPR | Medium | Cookie consent + privacy compliance is mandatory, not optional |
| Old services URLs indexed | Low–medium | Redirect map must be done before cutover or rankings drop |
| Image weight | Low | Mockups are photo-heavy. Need compressed/WebP assets or Lighthouse suffers. |

### Questions to ask the client

1. Who designs the remaining 12 pages — you or us?
2. Is final copy ready, or does it still need writing? Services page has ~2,000 words today.
3. Who supplies and licenses the architecture photography?
4. German language version now, later, or never?
5. Do the 4 services become separate pages, or stay as one long page like today?
6. Keep the existing contact form exactly, or simplify it?
7. Keep the 3 old blog posts, or start Insights fresh?
8. Who owns the Webflow account and pays hosting?
9. Any launch date tied to a campaign or event?
10. Do they need training to edit CMS content themselves?

---

## 8. Recommendation for the pitch

- Lead with **the design system is already decided** — tokens, type, colours and class names are in their spec. That de-risks the build and we should say so.
- Push for **services as 4 separate CMS pages**. Better SEO than one 2,000-word page, and the homepage design already links to them.
- Flag the **missing 12 page designs early**. If we do not, it becomes a scope fight in week 3.
- Offer a **Phase 1 design sprint** as a separate small engagement if they are unsure — gets us in the door and locks the build.

---

*Internal document. Effort figures are working estimates from the 3 supplied mockups and a live audit of yourgermancompany.com on 11 Sep 2026. Confirm the open questions before turning this into a client quote.*
