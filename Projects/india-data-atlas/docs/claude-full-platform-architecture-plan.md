# Full Platform Architecture Plan — India Data Platform

_Written 2026-09-08. This is the top-level architecture doc tying together every layer of the platform: today's backend, the admin control panel (`ops-dashboard-plan.md`), and everything still needed for a public product. Read this before `ops-dashboard-plan.md` or `project-status-and-plan.md` — those are now sub-plans under this one._

## The six layers

```
1. Database        Postgres + PostGIS — single source of truth, all engines write through Core, never direct
2. Engines          8 independent data-collection services (harvest -> normalize -> load via Core API)
3. Core API         Validation, dedup, licensing, publish_precision gating, the only writer to the DB
4. Public API       Read-only, versioned, rate-limited layer other people/apps consume (does not exist yet)
5. Website/Web App  Public-facing product for end users (does not exist yet — open item #6)
6. Admin Control Panel   Private, for you only (planned in ops-dashboard-plan.md)
```

Today: layers 1-3 exist and are live for 6/8 engines. Layers 4, 5, 6 are all unbuilt. This doc plans 4-6 properly instead of treating "the website" as one vague future task.

## Layer 4: Public API (new — not previously planned as its own layer)

This sits between Core and everything public-facing (your own website, and eventually third parties). Without it, the website would query Core directly, which mixes internal write-validation logic with public read traffic and makes future third-party access impossible to bolt on safely.

- **Versioning**: `/v1/...` from day one — data platforms live a long time, breaking changes will happen.
- **Read-only, rate-limited**: protect the VPS from scraping/abuse once this is public.
- **Respects `publish_precision`**: this is where "withhold" and "district-aggregate" actually get enforced for public consumption — Core stores the precision, Public API is what refuses to leak full precision to the outside world.
- **Caching layer**: most queries (browse by state/engine/entity type) are cacheable — a thin cache (even just in-process or Redis) avoids hammering Postgres on every public page load.
- **API keys for third parties**: if you ever want researchers/other apps to use this data (likely, given the citation-backed design), you need key issuance + per-key rate limits + usage tracking. Plan for it now even if you gate it off at launch.
- **Docs**: OpenAPI spec + a docs page — needed the moment anyone outside you touches this.

## Layer 5: Public Website / Web App (open item #6, now detailed)

This is the actual product for end users — separate from your admin panel.

- **What it's for**: browsing India's protected areas, species, water systems, etc. — a discovery/reference site, likely with maps as the centerpiece given how geo-heavy the data is.
- **Core features**: search/browse by engine, entity detail pages with full citations (this is the whole point of "citation-backed"), interactive maps (reuse the same map component thinking as the admin data browser, but public-safe precision only), state/district drill-down given how much data is administratively organized that way.
- **Content for gaps**: where data doesn't exist yet (grasslands, coral reefs, laws, extinct species) — decide whether the site shows "coming soon" placeholders or simply omits those sections. Worth deciding early since it affects site structure.
- **SEO**: this is reference content people will search for — India's protected areas, tribal population by district, etc. Static-generation or server-rendering (not a pure client SPA) matters here so pages are indexable.
- **Multilingual**: real open question for an India-wide platform — Hindi at minimum, possibly regional languages given the tribal/culture engine's content is inherently regional. Decide scope before building, not after.
- **Accessibility**: WCAG basics (the `design:accessibility-review` skill can audit once there's a UI to review).
- **Hosting**: almost certainly should NOT live on the same 2-vCore/4GB VPS as the database and engines — a public site with real traffic needs to scale independently. Plan for it on separate infrastructure (even a simple static host + CDN for the frontend, hitting the Public API layer above).

## Layer 6: Admin Control Panel

Already planned in detail in `ops-dashboard-plan.md` — engine health, run control, data browser, alerts, server metrics, API key management, decision log, open items tracker. That doc stands as-is; this section just places it in the overall architecture: it talks to Core/DB directly (privileged), not through the Public API.

## What's missing from the plan so far (the actual answer to "what am I missing")

**Infrastructure & scaling**
- Everything currently runs on one VPS with 4GB RAM. The moment the public website and Public API go live, database load, engine harvests, and public traffic are all competing for the same box. Needs a scaling plan: bigger VPS, or split DB/engines from the public-facing layer onto separate machines, before real traffic hits.
- No staging environment exists — every change so far has gone straight to production. Worth a cheap staging setup (even a second small VPS or local Docker Compose) before the public site makes mistakes visible to real users.
- No CI/CD pipeline — deploys are currently manual, session-by-session. Fine for now, but the public site/API will need actual tested deploys.
- No automated tests mentioned anywhere in the repos to date — worth confirming what test coverage (if any) exists before public traffic depends on uptime.

**Backups & disaster recovery**
- Flagged already in the dashboard plan but worth restating: it's unconfirmed whether Postgres/MinIO are backed up at all. This becomes non-optional once a public product depends on this data.

**Security**
- Public API needs rate limiting/abuse protection (mentioned above) plus basic security hardening (the VPS currently has no public interface for DB/MinIO — keep it that way, only the API layers should ever be exposed).
- Secrets management: currently `.env`/`harvest-secrets.env` files per engine. Fine at this scale, but the admin panel's planned key-rotation feature should tighten this rather than loosen it.

**Legal & compliance**
- **Data licensing**: you're aggregating from many sources (data.gov.in, Wikidata, Census, GBIF, HydroLAKES, JRC, MoEFCC, etc.) each with its own license terms. Before public launch, someone needs to verify attribution requirements are actually met on every public page, not just tracked internally in the license field.
- **Terms of Service / Privacy Policy** for the public site — not started, needed before any public launch, especially if user accounts or India-specific data-protection law (DPDP Act) applies.
- If the Public API ever supports user accounts (logins, saved searches, contributions), that's a whole new privacy/data-handling surface.

**Product/business questions** (not urgent, but worth having an opinion on before building layer 5)
- Is this free/public-good, or is there a monetization angle (API tiers, sponsorships)? Changes how the Public API's key/rate-limit system should be designed from the start.
- Any user-generated content planned (corrections, photo submissions, "report an issue")? If yes, that needs a moderation workflow — a new, currently unplanned piece.
- Domain name / branding for the public site — not decided yet as far as these docs show.

**Observability beyond the admin panel**
- Uptime monitoring from *outside* the VPS (e.g. a third-party pinger) — the admin panel checks health from the inside; if the VPS itself goes down, you want to be paged from somewhere else.
- Error tracking for the eventual website/API (e.g. Sentry-style) — nothing planned yet for catching frontend/API errors in production.

## Suggested build order across everything

1. Finish current backend deepening (already mostly done) + resolve the 5 "needs the user" items — unblocks remaining engines.
2. Build the Admin Control Panel (`ops-dashboard-plan.md`) — you need visibility before you build public-facing things on top of this data.
3. Build the Public API layer — thin, versioned, rate-limited, precision-aware. This is a prerequisite for the website, not optional scaffolding.
4. Decide the product/legal questions above (licensing attribution, ToS, multilingual scope, monetization) — cheap to decide now, expensive to retrofit.
5. Build the public website/web app against the Public API.
6. Add staging + CI/CD + backups + external uptime monitoring — ideally before step 5 goes live with real traffic, not after.

## Status
Planning only — nothing in this doc is built. This supersedes the informal "build the website" line item in open-items-next-session.md; that item should now point here.
