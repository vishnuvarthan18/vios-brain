# YGC — Webflow Build
## Detailed Internal Estimate — v2

- **Client:** Your German Company (YGC) — yourgermancompany.com
- **Scope:** Direct Webflow development. No separate design phase.
- **Prepared by:** araCreate Group
- **Date:** 15 September 2026
- **Basis:** Client requirement email + 3 supplied mockups + audit of live WP site
- **Status:** INTERNAL. Hours only, no pricing. Apply rate before sending.

---

## 1. What changed from v1

- Client confirmed: **go straight to Webflow development**. No design phase quoted.
- Page list is now fixed by the client — 10 page types (Disclaimer dropped, Cookie Policy added).
- Client explicitly asks for **tablet + mobile**, cookie consent, and post-launch support.
- Multi-language must appear as an **optional** line.

**Important:** "no design phase" does not mean no layout work. Only Home and Services overview have mockups. The other 8 page types still need layouts built from scratch in Webflow, following the same system. That effort sits inside the page-build phases below, not hidden.

---

## 2. Page and template inventory

| # | Page / Template | Mockup exists? | Build type |
|---|---|---|---|
| 1 | Home — 10 sections | ✅ Yes | Static |
| 2 | Services overview — 5 sections | ✅ Yes | Static + CMS list |
| 3 | Service detail template | ❌ No | CMS template |
| 4–7 | 4 service pages | ❌ No | CMS items |
| 8 | About | ❌ No | Static |
| 9 | Insights overview | ❌ No | Static + CMS list |
| 10 | Insight article template | ❌ No | CMS template |
| 11 | Contact | ❌ No | Static + form |
| 12 | Imprint | ❌ No | Static |
| 13 | Privacy Policy | ❌ No | Static |
| 14 | Cookie Policy / Cookie Settings | ❌ No | Static + modal |
| 15 | 404 page | ❌ No | Utility |

**2 of 15 designed. 13 built in-browser from the design system.**

### CMS collections

- **Insights** — title, slug, date, category (ref), excerpt, thumbnail, hero image, rich body, author, SEO title, SEO description, OG image, featured toggle
- **Categories** — name, slug
- **Services** — number, eyebrow label, title, short description, thumbnail, hero image, rich body, sort order

---

## 3. Detailed hours breakdown

### Phase 0 — Setup & planning — 10h

| Task | h |
|---|---|
| Requirement review, internal kickoff, scope lock | 2 |
| Webflow workspace, site creation, staging URL, access | 1 |
| Asset audit: logo files, images, font licences, copy check | 3 |
| Site architecture + URL structure document | 2 |
| Redirect mapping prep from existing WordPress URLs | 2 |

### Phase 1 — Design system in Webflow — 24h

| Task | h |
|---|---|
| Colour variables (8 values from client spec) | 2 |
| Typography scale — Playfair Display + Inter, H1–H6, body, eyebrow, caption, with responsive sizing | 5 |
| Spacing system — section padding 120–160px scaling down to mobile | 3 |
| Container + 12-column grid system (1440px canvas) | 3 |
| Button set — primary dark, secondary outline, red, text+arrow link, all states | 4 |
| Navbar component — desktop, tablet, mobile menu | 4 |
| Footer component — 4 columns, social icons, legal row | 3 |

### Phase 2 — Reusable components — 18h

| Task | h |
|---|---|
| Section wrapper + eyebrow label pattern | 2 |
| Icon + text card (used in 3 different sections) | 3 |
| Numbered service card | 2 |
| Journey / stage card | 2 |
| Insight card, CMS-bound, reused on Home and Insights | 3 |
| Image + text split block (hero, about, CTA variants) | 3 |
| CTA band component | 2 |
| Inner page hero template | 1 |

### Phase 3 — CMS setup — 10h

| Task | h |
|---|---|
| Insights collection + all fields | 3 |
| Categories collection + reference setup | 2 |
| Services collection + all fields | 3 |
| Collection list sorting, filtering, empty states | 2 |

### Phase 4 — Page build, desktop — 66h

| Task | h |
|---|---|
| Home — 10 sections | 18 |
| Services overview — 5 sections | 7 |
| Service detail template | 7 |
| 4 service CMS entries populated | 3 |
| About page | 7 |
| Insights overview + CMS list + pagination | 6 |
| Insight article template — rich text styles, share, related posts | 7 |
| Contact page layout | 4 |
| Legal template + Imprint, Privacy, Cookie Policy | 6 |
| 404 page | 1 |

### Phase 5 — Responsive: tablet + mobile — 32h

*No mobile designs supplied. Every breakpoint decision is made during build.*

| Task | h |
|---|---|
| Home — tablet + mobile (10 sections) | 10 |
| Services overview — tablet + mobile | 4 |
| Service detail + About | 5 |
| Insights list + article template | 5 |
| Contact + form | 3 |
| Legal pages + 404 | 2 |
| Mobile navigation menu build and test | 3 |

### Phase 6 — Contact form — 12h

*Existing form is complex: name, 8 contact-method options, email, phone with country code, 4 availability slots, help category, message.*

| Task | h |
|---|---|
| Form build — all fields, layout, validation states | 6 |
| Success and error states, redirect handling | 2 |
| Spam protection + email notification routing | 2 |
| Cross-device testing + deliverability check | 2 |

### Phase 7 — Interactions & hover states — 12h

| Task | h |
|---|---|
| Scroll reveal system (subtle, per client spec) | 4 |
| Navbar scroll behaviour | 2 |
| Button and arrow-link hover states | 2 |
| Card hover states | 2 |
| Light image reveal effects | 2 |

### Phase 8 — SEO & technical — 16h

| Task | h |
|---|---|
| Heading structure audit — H1/H2/H3 across all pages | 3 |
| SEO titles + meta descriptions — 10 main pages + 2 CMS templates | 4 |
| Open Graph images — default + per page + CMS dynamic | 3 |
| Favicon + webclip icon set | 1 |
| Sitemap.xml, robots.txt, canonical tags | 2 |
| 301 redirect map from old WordPress URLs (~15) | 3 |

### Phase 9 — Cookie consent & GDPR — 10h

*German client. This must be done properly, not a fake banner.*

| Task | h |
|---|---|
| Cookie consent tool setup and configuration | 4 |
| Script blocking by category, consent gating | 3 |
| Cookie Settings page / modal wiring | 2 |
| Testing all consent states | 1 |

### Phase 10 — Content migration — 8h

| Task | h |
|---|---|
| Migrate 3 existing blog posts into Insights CMS | 3 |
| Image optimisation + WebP conversion, all assets | 3 |
| Alt text pass across site | 2 |

### Phase 11 — QA, accessibility, performance — 18h

| Task | h |
|---|---|
| Cross-browser — Chrome, Safari, Firefox, Edge | 4 |
| Device testing — iOS, Android, iPad | 4 |
| Accessibility — contrast, focus states, alt text, heading order, keyboard nav | 6 |
| Performance — Lighthouse, image loading, font loading | 4 |

### Phase 12 — Client review & revisions — 16h

| Task | h |
|---|---|
| Review round 1 + fixes | 8 |
| Review round 2 + fixes | 6 |
| Final sign-off checks | 2 |

### Phase 13 — Launch & handover — 14h

| Task | h |
|---|---|
| Webflow hosting plan + site settings | 2 |
| DNS cutover, SSL, www redirect | 2 |
| Post-launch checks — forms, redirects live, indexing | 3 |
| Google Analytics / GTM + Search Console + sitemap submit | 3 |
| CMS training session, recorded | 2 |
| Handover documentation | 2 |

---

## 4. Totals

| Phase | Hours |
|---|---|
| 0 — Setup & planning | 10 |
| 1 — Design system | 24 |
| 2 — Reusable components | 18 |
| 3 — CMS setup | 10 |
| 4 — Page build, desktop | 66 |
| 5 — Responsive tablet + mobile | 32 |
| 6 — Contact form | 12 |
| 7 — Interactions | 12 |
| 8 — SEO & technical | 16 |
| 9 — Cookie consent | 10 |
| 10 — Content migration | 8 |
| 11 — QA & accessibility | 18 |
| 12 — Review & revisions | 16 |
| 13 — Launch & handover | 14 |
| **TOTAL** | **266h** |

- **266 hours ≈ 33 working days**
- 1 senior Webflow developer: **6–7 weeks**
- 2 developers partly parallel: **5 weeks**

### If we need a leaner number

| Cut | Saves |
|---|---|
| 1 review round instead of 2 | −6h |
| Simplify contact form to 6 fields | −5h |
| Skip deep performance pass | −4h |
| Client writes their own alt text | −2h |
| **Lean total** | **249h** |

Do not cut below this. Cookie consent, accessibility and the redirect map are not optional for a German B2B advisory site.

---

## 5. Optional items — quote separately

### German language version (client asked for this as optional)

| Task | h |
|---|---|
| Webflow Localization setup + locale config | 4 |
| Translate/duplicate all static pages (translation supplied by client) | 16 |
| CMS localization — Insights + Services | 6 |
| Language switcher component | 3 |
| hreflang tags + localized SEO fields | 3 |
| QA across both locales | 4 |
| **Subtotal** | **36h** |

Plus: Webflow Localization is a paid add-on on top of their hosting plan. Client pays that directly. Translation copy is not included — client supplies it.

### Other optional add-ons

| Item | h |
|---|---|
| Each extra service page beyond 4 | 3 |
| Each extra blog post migrated beyond 3 | 1 |
| Conditional-logic form via third-party tool | 8 |
| Newsletter signup + email tool integration | 6 |
| Custom icon set design | 8 |
| CRM integration (HubSpot / Pipedrive) | 10 |

---

## 6. Support — she asked about this directly

She asked two things: support **during** the build, and support **after** launch.

### During the build — include at no extra cost

- Named point of contact
- Response within 1 business day
- Weekly progress update + staging link
- Questions and clarifications unlimited

Say yes to this clearly. It costs us little and it is the thing she is worried about.

### After launch

- **Warranty:** 30 days of free bug fixes after go-live. Included.
- **Then one of:**

| Option | Hours/month | Good for |
|---|---|---|
| Ad-hoc | Pay as used, 1h minimum | Occasional fixes |
| Light retainer | 5h | Small content edits, minor tweaks |
| Standard retainer | 10h | Regular updates + new Insights posts |
| Growth retainer | 20h | Ongoing pages, SEO, A/B changes |

Recommend pitching **Light or Standard**. She publishes articles, so she will need us.

---

## 7. Timeline

| Week | Work |
|---|---|
| 1 | Setup, design system, reusable components |
| 2 | CMS setup, Home build, Services overview |
| 3 | Service detail, About, Insights, Contact, legal pages |
| 4 | Tablet + mobile responsive, contact form |
| 5 | Interactions, SEO, cookie consent, content migration |
| 6 | QA, accessibility, performance, review round 1 |
| 7 | Review round 2, launch, DNS cutover, handover |

**6–7 weeks from receipt of all copy and images.**

Clock starts when assets land, not at signature. Put this in writing.

---

## 8. Assumptions

- No design phase. The 13 undesigned pages follow the Home/Services system. Client approves via staging link, not Figma.
- Final copy for all 10 page types supplied in one batch before build starts.
- All photography supplied by client, licensed, high resolution.
- English only in main scope. German quoted separately as optional.
- Client owns the Webflow account and pays hosting + CMS plan directly.
- 2 rounds of revisions included.
- Service pages built as CMS items, not hand-built static pages.
- Contact form delivers to email. No CRM.
- No e-commerce, memberships, or logged-in areas.

---

## 9. Out of scope

- Copywriting and content strategy
- Photography, illustration, licensing
- German translation copy (localization build is quoted; translation is not)
- WordPress decommissioning and hosting cancellation
- Webflow subscription fees
- Cookie consent tool subscription, if a paid one is chosen
- Ongoing maintenance beyond the 30-day warranty
- Paid advertising, SEO campaigns, link building

---

## 10. Risks

| Risk | Why it matters | How we handle it |
|---|---|---|
| 13 of 15 pages have no design | Client may not like a layout only after it is built | Build 2 approval gates: after Home, and after the first inner page. Get written sign-off at each. |
| No mobile designs | Every breakpoint is our judgement call | State in the proposal that responsive behaviour is dev-led and follows the desktop system |
| Copy not ready | This is the #1 cause of delay on these projects | Start clock only when copy arrives. Say it plainly. |
| Complex contact form | Webflow native forms have limited conditional logic | Confirm whether she wants the existing 8-option form or a simpler one |
| Photography weight | Mockups are photo-heavy. Poor assets will hurt Lighthouse. | Ask for originals at 2500px+, we compress |
| Revision creep | "No design phase" projects invite endless tweaks | 2 rounds included, written. Extra rounds billed hourly. |
| Disclaimer page | Exists on the current site, missing from her list | Ask. Do not silently drop a legal page. |

---

## 11. Questions to send her

1. The current site has a **Disclaimer** page. Your list does not include it. Keep, merge into Imprint, or drop?
2. Should the 4 service pages be CMS-based (easy to add a 5th later) or fixed pages? We recommend CMS.
3. Who supplies the photography? The mockups use placeholder architecture images.
4. Is the copy final for all 10 pages, or only Home and Services?
5. Cookie consent tool preference — Cookiebot, Usercentrics, or a free option? Costs differ.
6. Keep the current contact form exactly (8 contact methods, availability slots), or simplify it?
7. Move the 3 existing blog posts into Insights, or start fresh?
8. Do Insights need category filtering and search, or a simple date-ordered list?
9. Is there a target launch date?
10. Newsletter signup on the Insights page — needed now or later?

---

## 12. Notes for the proposal

- Say **yes, clearly** to both support questions. That is what she is really testing.
- Put **multi-language as a separate optional block** with its own price, exactly as she asked.
- Lead with the fact that her spec already defines the design system — tokens, type, grid, class names. It shows we read her brief and it de-risks the build.
- Recommend **CMS for services**. It is a small upsell now and saves her money later.
- Be explicit that responsive layouts are dev-led, since no mobile designs exist. Protects us from "this is not what I imagined" in week 5.

---

*Internal working document. Hours are estimates based on the client requirement email of Sept 2026, the 3 supplied mockups, and an audit of yourgermancompany.com. Confirm section 11 before converting to a client proposal.*
