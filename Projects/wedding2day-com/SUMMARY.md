---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: wedding2day.com — mandap marketplace + wedding packages

> Source note: built from the Cowork project's saved memory (last updated 2026-07-28), the Mac repo `~/Desktop/mura/w2d/d2c/wedding2day.com` and the Google export (2026-10-06).
> Not the same as [[Projects/wedding2day-app/SUMMARY]] (the B2B Android trade app). Same brand and domain.

## 1. What this project is
- A customer-facing (B2C) website for weddings in Tamil Nadu. Started 2026-07-27.
- **Three parts (kept separate, never bundled):**
  1. **Mandap marketplace** — third-party kalyana mandapams are listed. Customer asks for a date, venue manager confirms, money is paid offline. No payment gateway in v1. Vishnu owns no venues.
  2. **Wedding packages** — Vishnu's own in-house team (already working today) sells A-to-Z services: photography, decor, catering, makeup, mehendi. Customer sends an inquiry, staff follow up.
  3. **Vendor tie-ups** (added 2026-07-28) — local decorators/caterers/photographers in cities the in-house team cannot reach. They deliver under the wedding2day brand. **Not listed publicly.** No database table yet.
- **Plan line:** "the hall is the hook, the wedding is the sale". Launch order: packages first; marketplace fills with venues in parallel.
- **How it makes money:** venue listings are free forever. Money comes only from wedding packages. The marketplace exists to bring in leads for the services business.
- **Area:** all of Tamil Nadu, but launched **city by city**. A city opens when it has 15–20 signed venues.
- **Roles:** [[People/Vishnu]] = business owner. [[Tools/Claude]] = project manager (plans, specs, research, writes prompts). [[Tools/Cursor]] = writes all the code. Vishnu does no manual tech steps.

## 2. Status now (as of 2026-10-06)
- Status: **paused** (not sure). Last recorded work: **2026-07-28** (last commit: plan, go-to-market kit, ops call-sheet data). Investor deck made Aug 2026.
- **Code found:** repo on the Mac at `~/Desktop/mura/w2d/d2c/wedding2day.com` (the old `~/Desktop/wedding2day.com` folder is empty).
- **Code (as of 2026-07-28):** both product lines built and tested on `main`. About 31 of 39 backlog tickets done; 132 tests pass.
  - Mandap marketplace: browse, booking request, venue manager portal, admin review, images on [[Tools/Cloudflare]] R2, calendar.
  - Wedding packages: admin create/edit, browse, detail page, guest inquiry, admin inbox, package images (W-035), inquiry rate limit 5 per IP per hour (W-033).
- **New build not live.** Left before go-live: remote D1 migration `0001_inquiry_rate_limits` and `R2_PUBLIC_BASE_URL`; W-018 image upload live test; W-031 Cloudflare deploy + domain; W-036 CI deploys; W-029 venue import.
- **Old site:** a wedding2day.com site was live in 2025 (Hostinger; contact-form message Sep 2025). Hostinger hosting expired 13 Jun 2026; domain moved to Cloudflare Jul 2026.
- **Domain risk:** Cloudflare zone deleted 12 Aug 2026; Hostinger domain renewal failed 20 Aug 2026 — the domain may be lapsing (check now).
- **Not done in code:** Reviews (W-026–029), SEO (W-032), CI (W-036), duration pricing (W-038, waiting on a schema choice).
- **Real bottleneck is not code:** venue sign-ups were at **zero**. Who answers inquiries (delivery) was not decided.

## 3. Next steps
1. Check the wedding2day.com domain registration now (renewal failed 20 Aug; Cloudflare zone deleted 12 Aug).
2. **Supply:** call and sign 15–20 mandapams in the first city. Use the call sheet `data/wedding2day-ops-call-sheets.xlsx`. Also sign vendors (same visit-and-check bar as venues).
3. **Demand:** [[Tools/Instagram]] of real past weddings, muhurtham-date content, referrals. Do not fight Mandap.com on venue-page SEO.
4. **Delivery:** decide who answers each inquiry and how fast.
5. Go-live tasks: remote D1 migration, `R2_PUBLIC_BASE_URL`, W-018 live upload test, **W-031 deployment** (via a [[Tools/Cursor]] queue prompt), W-036 CI, W-029 venue import.
6. Decide W-038 schema: new column or new table.
7. Later: Reviews epic (W-026–029), SEO (W-032). Design a vendor table.
8. Open the site in a browser and test on a phone (375px) before launch.

## 4. Decisions
- 2026-07-27 — Two separate product lines: mandap booking and wedding packages. A booking request and a package inquiry are different things — never merge — Vishnu: "mandap booking separate and package separate". #decision
- 2026-07 (not sure of day) — Inquiry-first booking, no online payment — whole Indian market ([[Companies/Mandap.com]], [[Companies/VenueLook]]) works by inquiry + offline payment; instant pay would triple the build. #decision
- 2026-07 (not sure) — Scraped venue data stays private in a `venue_prospects` table (sales list only, never shown) — a venue that never agreed to be listed will not reply, and that customer is lost. #decision
- 2026-07 (not sure) — Run on [[Tools/Cloudflare]] Workers, not Node. Passwords hashed with PBKDF2 (`crypto.subtle`). Own session-based login in D1, no NextAuth — bcrypt/argon2/node:crypto fail on Workers. #decision
- 2026-07 (not sure) — Sparse availability table: a row = booked/blocked, no row = open — avoids 365 rows per venue per year. #decision
- 2026-07 (not sure) — Do not use [[Tools/Google Places API]] to fill the catalog — its terms ban storing names, phones, ratings, photos. Use manual collection or directory scraping. #decision
- 2026-07 (not sure) — [[Tools/Claude]] is PM, [[Tools/Cursor]] writes all code. Vishnu gets copy-paste prompts, no manual steps — "all need to done by cursor using prompt, no manual work". #decision
- 2026-07 (not sure) — Cursor work runs as a ticket queue with stop rules + `HANDOFF.md`; commit each ticket as `W-0XX: ...`; no edits to `schema.sql` / `schema.ts` / `wrangler.toml` — lets Cursor run alone safely. #decision
- 2026-07 (not sure) — Always re-check Cursor's "all tests pass" — this caught a login timing leak (W-034) and a 14-ticket gap where the app could not open locally (W-037). #decision
- 2026-07-28 — Cover all of Tamil Nadu, launched city by city; each city opens at 15–20 signed venues — do not wait for the whole state. #decision
- 2026-07-28 — Revenue = services-attach: venue listings free forever, money only from packages — charging venues makes the venue the customer and listing quality rots (seen at [[Companies/Mandap.com]], [[Companies/WedMeGood]]). #decision
- 2026-07-28 — Vendor tie-ups as a third part: local vendors deliver under the wedding2day brand, not listed publicly; must pass visit-and-verify like venues — makes "all of Tamil Nadu" possible without an in-house team everywhere. #decision
- 2026-07-28 — Do not fight [[Companies/Mandap.com]] on venue-page SEO — they own Chennai neighbourhood pages with [[Companies/Matrimony.com]] behind them. Get demand from [[Tools/Instagram]], referrals and muhurtham-date content. #decision
- 2026-07-28 — Launch order: packages first, marketplace fills in parallel. #decision
- 2026-07-28 — No public "available" calendar at launch. Show known booked dates only; everything else = "availability on request". Ops keeps calendars current via weekly [[Tools/WhatsApp]] check-in — a wrong "available" date loses a customer for good. #decision

## 5. Timeline
- 2025 — Old wedding2day.com site live on Hostinger (contact-form message Sep 2025).
- 2026-06-13 — Hostinger hosting expired.
- 2026-07-27 — Project started in Cowork. [[Tools/Claude]] set as PM, [[Tools/Cursor]] as developer.
- 2026-07 (not sure) — Competitor research: [[Companies/Mandap.com]], [[Companies/VenueLook]] → inquiry-first booking chosen.
- 2026-07 (not sure) — Checked [[Tools/Google Places API]] terms → cannot seed catalog.
- 2026-07 (not sure) — Early tickets built (login W-004 etc.). Review found login timing leak → W-034. Found app could not open in `next dev` without D1 → W-037.
- 2026-07 (not sure) — Mistake: sent Cursor a queue for W-021–025 that were already done. New rule: check `git log --oneline --all | grep "W-0"` first.
- 2026-07-28 — Scope pushed: all of Tamil Nadu, city by city. Revenue model settled (services-attach). Vendor tie-ups added. No-public-calendar rule. SEO fight with Mandap.com ruled out.
- 2026-07-28 — `docs/go-to-market/COMPETITORS.md` added ([[Companies/Mandap.com]], [[Companies/WedMeGood]], [[Companies/Weddingz.in]] flows).
- 2026-07-28 — Ops call sheet `data/wedding2day-ops-call-sheets.xlsx` built (Mandapams + Vendors tabs).
- 2026-07-28 — Snapshot: ~31 of 39 tickets done, both product lines complete, not deployed. Venue sign-ups = 0.
- 2026-07-28 — Handoff: W-035 package images and W-033 inquiry rate limit done; 132 tests pass. Last commit.
- 2026-07 — Domain moved to Cloudflare.
- 2026-08 — Investor deck made.
- 2026-08-05 — Chat: Zoho mail DNS setup on Cloudflare.
- 2026-08-12 — Cloudflare zone deleted.
- 2026-08-20 — Hostinger domain renewal failed.
- 2026-10-03 — Old connected folder found empty. Project moved into viOS.
- 2026-10-06 — Repo found at `~/Desktop/mura/w2d/d2c/wedding2day.com`.

## 6. Key facts
- **People:** [[People/Vishnu]] — business owner; runs the existing in-house wedding services team. Does not write code.
- **Companies (competitors):** [[Companies/Mandap.com]] (owned by [[Companies/Matrimony.com]]), [[Companies/WedMeGood]], [[Companies/Weddingz.in]], [[Companies/VenueLook]].
- **Tools:** [[Tools/Cloudflare]] (Pages on Workers runtime, D1 database, R2 images), [[Tools/Next.js]] 16, [[Tools/Wrangler]], [[Tools/Drizzle ORM]], [[Tools/Vitest]], [[Tools/SQLite]] (D1 is SQLite), [[Tools/Cursor]], [[Tools/Claude]] (Cowork), [[Tools/GitHub]] (remote not recorded), Zoho Mail, [[Tools/Instagram]], [[Tools/WhatsApp]], [[Tools/Google Places API]] (ruled out).
- **Where work was done:** Claude Cowork project "wedding2day.com" (planning/PM) + [[Tools/Cursor]] (all code).
- **Local folder:** `~/Desktop/mura/w2d/d2c/wedding2day.com` (old path `~/Desktop/wedding2day.com` is empty).
- **Domain:** wedding2day.com (may be lapsing — see status). Also hosted the landing page of [[Projects/wedding2day-app/SUMMARY]] on [[Tools/Cloudflare]] Pages from 2026-07-16 — how the two share the domain is not clear.
- **Repo remote:** not recorded.
- **Chats index:** INDEX (archived: Projects/wedding2day-com/chats/INDEX.md)
- **Ticket IDs:** W-001 to W-039. Done out of order often. Always check git log, not BACKLOG.md order.
- **Tables:** `venue_prospects` (private), `availability` (sparse), session rows in D1. No vendor table.
- **Testing note:** `npm test` crashes (SIGBUS) in the cloud sandbox — not a code bug. Local tests need `.wrangler/` folder.
- **Cursor prompt rules:** read `AGENTS.md`, `ARCHITECTURE.md`, `BACKLOG.md` first; one ticket at a time; build/lint/test after each; stop on schema changes or doubt; write `HANDOFF.md`.

## 7. Files and documents
(In the repo at `~/Desktop/mura/w2d/d2c/wedding2day.com`.)
- `docs/PLAN.md` — current strategy (replaces order in LAUNCH-SEQUENCE.md).
- `docs/PRD.md` — product requirements.
- `docs/ARCHITECTURE.md` — tech design.
- `docs/BACKLOG.md` — tickets W-001 to W-039.
- `AGENTS.md` — rules for Cursor.
- `docs/go-to-market/LAUNCH-SEQUENCE.md` — older launch order.
- `docs/go-to-market/COMPETITORS.md` — Mandap.com / WedMeGood / Weddingz.in flows (2026-07-28).
- `data/wedding2day-ops-call-sheets.xlsx` — live call list. Tabs: Mandapams, Vendors, Read Me. Mandapams columns A–H match `venue_prospects`; drop columns after H before import.
- `schema.sql`, `src/db/schema.ts`, `wrangler.toml` — do not let Cursor change these without a decision.
- `HANDOFF.md` — Cursor's report after each queue run.
- Cowork project memory: 5 notes (project, key decisions, working mode, Cursor prompt pattern, verify Cursor claims).

## 8. Open questions and problems
- Is the wedding2day.com domain still registered? (renewal failed 20 Aug 2026)
- Is the project paused, active, or dropped?
- W-031 deployment not done as of 07-28 — anything since?
- How many venues have been signed? Is the call sheet being filled?
- Who answers inquiries, and how fast? (delivery — undecided)
- W-038 duration pricing: new column or new table?
- Vendor tie-ups: no database table yet. How to track vendors?
- How does this site share wedding2day.com with the [[Projects/wedding2day-app/SUMMARY]] landing page?
- Which city launches first? (not recorded — maybe Chennai or home city)
- What is the home city of the in-house team?
- GitHub remote for the repo?
- Is this related to the B2B app, or a separate business line?

## 9. All chats in this project
- Zoho mail DNS configuration on Cloudflare (archived: Projects/wedding2day-com/chats/2026-08-05 Zoho mail DNS configuration on Cloudflare.md) — 2026-08-05
- Cowork session `bb49ff4c` (title not known) — 2026-07-27 to 2026-07-28
- Save project to viOS — 2026-10-03
- (2026-07-16 "QR code for customer data collection" moved to [[Projects/wedding2day-app/SUMMARY]] — it is the app's pre-launch landing page.)
