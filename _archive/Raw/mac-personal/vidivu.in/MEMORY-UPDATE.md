# Vidivu — Memory Update (Aug 2026 session)

> **Action required:** the project's `memory.md` is synced read-only into Cowork sessions, so it can't be written to directly. Paste the contents below into the project knowledge on claude.ai to persist it.

---

**Purpose & context**

Vishnu is building **Vidivu**, a services-based IT studio targeting startups and SMEs, positioned as an end-to-end tech partner rather than a narrowly vertical agency. Operates part-time and solo, one active launch build at a time with retainers stacking — **never reflected in public materials**.

Claude functions as strategist and execution partner across positioning, research, and the website build. Key buyers are founders and SME decision-makers, with Indian Tier-2 buyers a meaningful segment.

**Brand meaning:** Vidivu traces to the Tamil phrase "துன்பம் நீங்கி இன்பம் பிறத்தல்" (sorrow departs, joy is born). Decision made **not** to display the Tamil script on the site — the sentiment is carried in English hero copy instead: *"We solve your toughest tech problems and give your business a fresh start."*

---

## Current state — website

Next.js 16 + Tailwind v4, repo `vishnuvarthan18/vidivu.in-`. **Not yet deployed on Vercel.** Grown from one homepage to **15 pages**:

- Home, Services hub + 6 service pages (Product Engineering, UI/UX Design, MVP Sprints, Cloud & DevOps, API & Systems Integration, Retainers)
- Industries, Process, Technology, About, Contact
- Persona pages: `/startups`, `/smes`

**Shared components:** `Navbar` (client, with mobile menu), `Hero`, `PageHeader`, `Callout`, `Faq`, `MediaSlot`, `TrustBand`, `Comparison`, `PersonaSplit`, `ServiceDetail`, `Sections` (SpecBand/ModelGrid/Magazine/Motorsport/CtaBand/Footer).

**SEO built:** sitemap.ts, robots.ts, per-page metadata, FAQPage JSON-LD on Home/Services/Industries/Process/Contact, ProfessionalService JSON-LD in root layout.

**Design system:** unchanged and documented in `DESIGN.md` — near-black canvas, uppercase Inter 800 display + 300 body, tricolor stripe (#3ba0e0 → #1c69d4 → #e22718) used sparingly, 0px radius.

**Media:** programmatically generated dummy assets in `/public/media/` (PIL + ffmpeg) — dark dashboard/code-mockup compositions in brand colours. Hero uses an animated 5s looping mp4 (37KB) with jpg poster; service, persona, case-study, and founder-portrait images all wired. `MediaSlot` component handles image/video/placeholder states — swapping in real media is just changing the `src` prop. **Delete `build-loop.mp4`** — unused leftover, sandbox lacked permission to remove it.

---

## Locked decisions (do not violate)

- **No published pricing**, ever.
- **Never mention solo or part-time** in public materials.
- **No fabricated proof.** Claude twice refused requests to claim "10 years experience" and to scrape competitor stats/logos/testimonials onto the site. Positioning must rest on things true today: code ownership, NDA on every project, milestone billing, direct founder access.
- Milestone payment structure (advance, not 100% upfront).
- WhatsApp Business as primary conversion channel — now **data-validated**, not just preference.
- Entry offer is a named audit artifact, credited toward a build.
- No single vertical called out publicly.

---

## Growth strategy — two ICPs

Documented fully in `vidivu-strategy.md` (repo root).

- **Startups:** founder-led, fast decisions, MVP/speed-driven. Entry via MVP Sprints & Product Engineering. Indian MVP market rate ₹5–12L mid-range, ₹2–5L simple (internal context only — never publish).
- **SMEs:** owner/ops-led, risk-averse, modernization-driven (legacy systems, spreadsheets, security gaps). Entry via Cloud & DevOps, API Integration, Retainers. **This buyer was previously unserved** — site language was entirely startup-coded until `/smes` was built.

**Tier-2 India tailwind:** adoption growing ~40% faster than metros; 65%+ of Indian SMEs raising digital spend within 18 months; government schemes (CHAMPIONS, PM Vishwakarma, state MSME funds) subsidising 30–50% of cost. No reviewed competitor targets this segment.

**WhatsApp data:** 89% of Indian small business owners use it for business; 82–91% open rates vs 22–28% email; qualification time drops ~48hrs → <2hrs; most inquiries need 2–4 follow-ups before converting (Contact page now sets this expectation).

**Keywords:** long-tail over broad. Startup-intent ("MVP development company India") vs SME-intent ("replace legacy system small business India") vs Tier-2 city terms (Coimbatore, Madurai, Vadodara, Surat, Rajkot, Lucknow) — far less contested than "Chennai", which every competitor already owns.

---

## Competitive picture

Full table in `vidivu-market-table.md`. Six competitors fetched directly: Amrithaa (21+ yrs), K2B (18+ yrs), MacAppStudio (15 yrs, publishes pricing), Sieora (broad AI/IoT, aggressive SEO), Tentosoft (not a real competitor — surveillance product, but **best proof format**: named clients + hard metrics), **F22 Labs** (closest true peer — Chennai product studio, founder-led, lists founders publicly with email/LinkedIn).

**Core finding:** all compete on tenure or scale — unwinnable and not worth faking. Openings are (1) the underserved SME modernization buyer, (2) uncontested Tier-2 geography, (3) transparency as differentiation.

**Unfixable by on-page work:** competitors have years of backlinks and published case studies. Highest-leverage move is shipping real work and publishing metric-led case studies, Tentosoft-style.

---

## Blockers before launch

1. **Placeholder contact details** — WhatsApp number, email (`app/contact/page.tsx`) and founder name/email/LinkedIn (`app/about/page.tsx`), all marked `TODO`. **A lead literally cannot reach Vidivu today.** Highest priority.
2. Deploy to Vercel — also the only way to get real Lighthouse numbers (sandbox can't run `next dev` or `next build`; no npm registry access for the SWC binary, so verification has been typecheck + lint only, both consistently clean).
3. Replace dummy media with real screenshots/photography.
4. Case studies still say "coming soon" — weakest trust element on the site.

## Held deliberately

- **Tier-2 city landing pages** — not built until there's real capacity to serve those markets. An empty city page harms more than it helps.
- Top nav capped at 6 items; Startups/SMEs discovered via homepage PersonaSplit + footer + mobile menu rather than crowding it.

---

## Working patterns

- Vishnu communicates in direct, fragmented shorthand; prefers decisions over deliberation. Move fast, flag real problems, don't pad.
- Terminal guidance: plain text, one command at a time, no jargon.
- Research runs in parallel via external AI tools; Claude synthesises across reports.
- **Flag real bugs proactively** — the missing mobile nav (all links hidden below 768px with no hamburger, leaving mobile visitors with zero navigation) was caught during a UI pass, not reported by the user.
- Direct URL fetching beats web search for competitor page structure.
- Project is treated as a persistent brain across strategy, design, and build.
