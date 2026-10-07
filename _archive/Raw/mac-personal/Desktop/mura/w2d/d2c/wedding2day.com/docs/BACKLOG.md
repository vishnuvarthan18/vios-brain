# Build Backlog

Execute top to bottom. Tickets within an epic can sometimes run in parallel; epics
mostly cannot, because later ones depend on earlier schema and auth.

Hand Cursor **one ticket at a time**. Giving it a whole epic produces sprawling diffs
that are hard to review and easy to get subtly wrong.

Read `AGENTS.md` before starting. Every ticket inherits the definition of done there.

---

## Epic 0 — Foundation

Nothing else works until this is done. No user-facing output.

### W-000 · Reorganize into the target structure

**Do this first.** The initial scaffold was flat and does not match the structure in
`ARCHITECTURE.md`. Every later ticket assumes the target layout, so fixing it now avoids
compounding churn.

Move files (this is a `git mv`, preserve history):

| From | To |
| --- | --- |
| `src/app/page.tsx` | `src/app/(public)/page.tsx` |
| `src/app/mandaps/page.tsx` | `src/app/(public)/mandaps/page.tsx` |
| `src/app/mandaps/[id]/page.tsx` | `src/app/(public)/mandaps/[id]/page.tsx` |
| `src/app/login/page.tsx` | `src/app/(auth)/login/page.tsx` |
| `src/components/Nav.tsx` | `src/components/layout/Nav.tsx` |
| `src/components/Footer.tsx` | `src/components/layout/Footer.tsx` |

Leave `src/app/layout.tsx`, `globals.css`, and `favicon.ico` at the app root — Next.js
requires the root layout there. Leave `manager/` and `admin/` where they are.

Then:

- Create `src/app/(public)/layout.tsx` rendering `<Nav />`, `{children}`, `<Footer />` in the flex-column wrapper the pages currently each duplicate
- Remove the per-page `<Nav />` and `<Footer />` imports and JSX from all three public pages — the group layout handles it now
- Create `src/app/(auth)/layout.tsx` with a simple centered container; strip Nav/Footer from the login page
- Create empty-but-real directories with a `.gitkeep`: `src/lib/auth/`, `src/lib/repositories/`, `src/lib/notifications/`, `src/lib/validation/`, `src/lib/utils/`, `src/components/ui/`, `src/components/venue/`, `src/components/package/`
- Update all imports to the new paths

**Done when:** `npm run build` passes, every existing route still resolves at the same
URL (`/`, `/mandaps`, `/mandaps/[id]`, `/login`, `/manager/*`, `/admin/*`), and Nav and
Footer appear exactly once per page.

**Note:** route groups don't change URLs. `(public)/page.tsx` still serves `/`.

---

### W-001 · Drizzle + D1 wiring

Set up Drizzle against Cloudflare D1 with local development working end to end.

- Install `drizzle-orm`, `drizzle-kit`, `wrangler` (dev dependency)
- `src/db/schema.ts` — Drizzle schema mirroring `src/db/schema.sql` exactly
- `src/db/client.ts` — helper returning a Drizzle instance from the D1 binding
- `drizzle.config.ts` configured for D1
- `wrangler.toml` D1 binding named `DB`
- npm scripts: `db:generate`, `db:migrate`, `db:seed`

**Done when:** `wrangler dev` runs, a trivial query against local D1 returns a result,
and generated migrations match `schema.sql`.

**Blocked by:** W-000

---

### W-002 · Initial migration and seed data

- **Install `tsx` as a devDependency.** The `db:seed` script added in W-001 invokes it but it was never installed, so the script fails today.
- **Pulled forward from W-003:** build `src/lib/auth/password.ts` here. Seeded users need real `password_hash` and `password_salt` values or nobody can log in as them, so the hashing helper has to exist before the seed does.
- Generate the initial migration from the Drizzle schema
- `src/db/seed.ts` creating: one admin, two approved managers, three customers, six published mandaps across two cities with images, amenities, tiers, and some blocked dates; four service categories with services; three published packages
- Seed data must be realistic Indian venue and pricing data, not lorem ipsum — it's what every subsequent UI ticket is built against

**Done when:** `npm run db:seed` populates a fresh local D1 and every table has rows.

**Blocked by:** W-001

---

### W-003 · Session helpers

Pure logic, no UI. `src/lib/auth/password.ts` was pulled forward into W-002 — verify it
exists and is tested rather than rebuilding it.

**Security fix carried over from W-002:** the seed creates a hardcoded admin session
with the predictable id `session-admin-seed`. Harmless locally, an admin backdoor if the
seed ever runs anywhere else. Remove that row from `src/db/seed.ts`, and make the seed
refuse to run unless it is explicitly targeting a local database.

- `src/lib/auth/session.ts` — `createSession`, `getSession` (React `cache`-wrapped), `deleteSession`
- Session cookie: `httpOnly`, `Secure`, `SameSite=Lax`, 30-day expiry

**Done when:** unit tests cover hashing round-trip, wrong-password rejection, and
session create/read/delete.

**Blocked by:** W-001

---

### W-004 · Signup, login, logout

- `/signup` and `/login` pages, both mobile-first
- Server actions with Zod validation; specific field-level errors, not a generic failure message
- Signup takes a role choice: customer or manager. Managers are created with `manager_approved = 0`
- Logout clears the session row and cookie
- Redirect after login by role: customer → `/`, manager → `/manager`, admin → `/admin`

**Done when:** all three flows work, duplicate email is rejected cleanly, and passwords
are never logged or returned.

**Blocked by:** W-003

---

### W-005 · Route guards

- `src/lib/auth/guards.ts` — `requireUser()`, `requireRole(role)`, `requireApprovedManager()`
- Middleware for coarse redirects on `/manager/*` and `/admin/*`
- Guards called at the top of every protected server component and action

**Done when:** an unauthenticated visit to `/admin` redirects to login; a customer
hitting `/manager` is refused; an unapproved manager sees a "pending approval" screen
rather than the dashboard.

**Blocked by:** W-004

---

## Epic 1 — Public venue catalog

First real user-facing value. Read-only, no auth needed to view.

### W-006 · Venue browse page

Replace mock data in `/mandaps` with real queries.

- `src/lib/repositories/mandaps.ts` with `getPublishedMandaps(filters, cursor)`
- Only `status = 'published'` rows appear, ever
- Card: primary image, name, city and area, capacity range, starting price, average rating
- Cursor-based pagination, not offset
- Empty state and loading skeleton

**Done when:** seeded venues render, pagination works, and no unpublished venue is
reachable.

**Blocked by:** W-002

---

### W-007 · Search and filters

- Filter by city, guest capacity, price range, and amenities
- Text search across venue name and area
- Filters live in URL search params so results are shareable and back works
- Mobile: filters in a bottom sheet. Desktop: sidebar.
- Active filters shown as removable chips, plus a clear-all

**Done when:** filters combine correctly, the URL reflects state, and an over-filtered
search shows a helpful empty state rather than a blank page.

**Blocked by:** W-006

---

### W-008 · Venue detail page

- Image gallery with lightbox
- Description, capacity, address, amenities
- Venue-defined tiers with prices
- Published reviews with average rating
- Sticky "Request to book" CTA on mobile
- Per-venue SEO metadata and Open Graph tags
- `notFound()` for unpublished or missing venues

**Done when:** a seeded venue renders fully at 375px and desktop, and an unpublished
slug 404s.

**Blocked by:** W-006

---

### W-009 · Availability calendar (display)

- Month calendar on the venue detail page showing which dates are open
- Reads the sparse `availability` table: no row means open
- Past dates disabled; navigate forward up to 18 months
- Clearly distinguish open, booked, and blocked

**Done when:** seeded blocked dates render correctly and the component works on a
narrow screen.

**Blocked by:** W-008

---

### W-010 · Date-availability filter

Extend browse filters with "available on date X" using `NOT EXISTS` against
`availability`.

**Done when:** filtering by a date that a seeded venue has blocked excludes that venue.

**Blocked by:** W-007, W-009

---

## Epic 2 — Booking loop

The core of Line A. This is where the marketplace either works or doesn't.

### W-011 · Booking request submission

- Form on the venue detail page: date, event type, guest count, contact phone, notes, optional tier
- Requires login; unauthenticated users are sent to login and returned to the form afterward
- Server action with Zod validation
- Reject dates already booked or blocked, and dates in the past
- Reject duplicate requests for the same venue, customer, and date
- Success state making clear the venue still has to confirm

**Done when:** a valid request is persisted as `pending`, and every rejection case
returns a clear message.

**Blocked by:** W-005, W-009

---

### W-012 · Customer booking dashboard

- `/account/bookings` listing the customer's own requests with status
- Detail view with venue contact details revealed only once confirmed
- Cancel a pending request

**Done when:** a customer sees only their own requests. Include a test proving another
customer's booking id returns 404, not the record.

**Blocked by:** W-011

---

### W-013 · Manager booking inbox

- `/manager/bookings` with pending requests first
- Each shows date, guest count, event type, notes, and customer contact
- Confirm and reject actions; reject requires a reason
- Only the owning manager can act on a request

**Done when:** confirm and reject both work and a manager cannot act on another
manager's request. Ship with the negative-path test.

**Blocked by:** W-011

---

### W-014 · Availability updates on confirm

- Confirming a booking writes an `availability` row with status `booked` linked to the request
- Both writes happen in a single `batch()` so they cannot diverge
- Rejecting or cancelling a confirmed booking frees the date again
- Guard against two requests for the same date being confirmed

**Done when:** confirming marks the date booked on the public calendar, and a
concurrent double-confirm fails cleanly instead of double-booking.

**Blocked by:** W-013

---

### W-015 · Email notifications

- `src/lib/notifications/` with a channel-agnostic interface, Resend as the first channel
- Manager gets an email when a request arrives
- Customer gets an email on confirm or reject
- Sent after the DB write commits; a send failure logs but never fails the request

**Done when:** both emails send in local dev against a test key, and a forced send
failure doesn't break booking.

**Blocked by:** W-014

---

## Epic 3 — Manager portal

### W-016 · Manager approval flow

- Manager signup lands in an unapproved state with a clear pending screen
- `/admin/managers` lists pending managers with their details
- Admin approves or rejects; approval writes to `audit_log`
- Approved managers gain portal access

**Done when:** an unapproved manager cannot create a listing, and approval immediately
grants access.

**Blocked by:** W-005

---

### W-017 · Listing create and edit

- `/manager/listings` — the manager's own venues
- Create and edit: name, description, city, area, address, capacity range, base price, amenities, tiers
- New and edited listings go to `pending_review`, not straight to published
- Slug auto-generated from name, uniqueness enforced

**Done when:** a manager can create a listing that appears in admin review, and cannot
edit a listing they don't own.

**Blocked by:** W-016

---

### W-018 · Image upload

- Presigned R2 upload via server action; browser uploads directly
- Reorder and delete, set a primary image
- Validate MIME and size server-side. Max 5MB per image, 15 per venue
- `alt` text required

**Done when:** images upload, appear on the public detail page, and an oversized or
non-image file is rejected server-side.

**Blocked by:** W-017

---

### W-019 · Manager calendar management

- Calendar view of the manager's venue
- Block and unblock dates, including multi-date selection
- Dates tied to a confirmed booking cannot be unblocked without cancelling the booking
- This is the manager's most important screen — make it the portal landing page

**Done when:** blocking a date hides it from public availability immediately, and
booked dates are protected.

**Blocked by:** W-017

---

### W-020 · Admin listing review

- `/admin/listings` queue of `pending_review` venues
- Publish, request changes, or suspend
- Actions written to `audit_log`

**Done when:** publishing makes a venue publicly visible and suspending removes it.

**Blocked by:** W-017

---

## Epic 4 — Wedding packages (Line B)

Independent of Epics 1–3. Can run in parallel once Epic 0 is done, and has no supply
dependency — this is why it can launch before the marketplace.

### W-021 · Service and package admin

- Admin CRUD for `service_categories`, `services`, `packages`, `package_items`
- Package builder: add line items, either linked to a service or free text
- Draft and published states

**Done when:** an admin can build a package end to end and publish it.

**Blocked by:** W-005

---

### W-022 · Public packages browse

- `/packages` listing published packages
- Card: image, name, tagline, starting price, guest range
- Comparison of what each tier includes

**Done when:** seeded packages render and drafts never appear.

**Blocked by:** W-021

---

### W-023 · Package detail page

- Full inclusions grouped by service category
- Gallery
- Pricing shown as "starting from" when `price_from` is set
- Prominent inquiry CTA
- SEO metadata

**Done when:** a seeded package renders fully at 375px and desktop.

**Blocked by:** W-022

---

### W-024 · Package inquiry

- Form: name, phone, email, event date, guest count, city, message
- **No login required** — guest inquiries are allowed, and gating this would cost leads
- Links to the logged-in customer when there is one
- Zod validation and basic rate limiting by IP
- Confirmation state setting expectations on response time

**Done when:** a guest can submit an inquiry and it lands in the admin inbox.

**Blocked by:** W-023

---

### W-025 · Admin inquiry inbox

- `/admin/inquiries` with status pipeline: new → contacted → quoted → won/lost
- Assign to a staff member, add internal notes
- Filter by status and date

**Done when:** an admin can move an inquiry through the full pipeline.

**Blocked by:** W-024

---

## Epic 5 — Reviews

### W-026 · Mark bookings completed

- Scheduled Worker moving confirmed bookings past their event date to `completed`
- Triggers a review request email

**Done when:** a backdated confirmed booking flips to completed on the next run.

**Blocked by:** W-015

---

### W-027 · Review submission

- Only the customer on a `completed` booking can review, one review per booking
- Rating 1–5 plus optional title and body
- Submitted reviews start as `pending`

**Done when:** a valid review saves and a customer without a completed booking is
refused. Ship with the negative-path test.

**Blocked by:** W-026

---

### W-028 · Review moderation and display

- `/admin/reviews` queue to publish or reject
- Published reviews appear on the venue page with average rating
- Rating shown on browse cards, and filterable

**Done when:** publishing a review updates the venue's average immediately.

**Blocked by:** W-027

---

## Epic 6 — Admin operations

### W-029 · Venue prospect pipeline

- `/admin/prospects` — the private seeded lead list
- CSV import for scraped and sourced leads
- Outreach status tracking: new → contacted → interested → onboarded/rejected
- Convert a prospect into a draft `mandaps` record, linking the two
- **This data is never public.** Include a test asserting no public query can return prospect rows.

**Done when:** a CSV imports, a prospect converts to a draft listing, and the isolation
test passes.

**Blocked by:** W-005

---

### W-030 · Admin dashboard

Real metrics, not placeholders:

- Published venues, pending manager approvals, pending listing reviews
- Bookings this month and **manager response rate within 48 hours** — the number that predicts whether the marketplace works
- Open package inquiries

**Done when:** every figure is a real query and the response-rate calculation is
covered by a unit test.

**Blocked by:** W-014, W-025

---

## Epic 7 — Launch

### W-031 · Cloudflare deployment

**Undo the W-001 local-dev workaround first.** To make `wrangler dev` work before any
Pages build output existed, W-001 commented out `pages_build_output_dir` in
`wrangler.toml`, set `main = "src/db/dev-worker.ts"`, and added that dev-only worker.
Deploying in this state ships a bare Worker instead of the site.

- Restore `pages_build_output_dir = ".vercel/output/static"` in `wrangler.toml`
- Remove the `main = "src/db/dev-worker.ts"` line and delete `src/db/dev-worker.ts`
- Build via `@opennextjs/cloudflare` — NOT `@cloudflare/next-on-pages`, which has been superseded. W-037 already installs the adapter for local dev; this ticket wires the production build and deploy.
- Production and preview D1 databases with migrations applied
- R2 bucket and Cloudflare Images configured
- Secrets in Cloudflare environment variables
- `wedding2day.com` domain pointed at the Pages project

**Done when:** production builds and deploys serving the actual Next.js app (not the dev
worker), and a seeded staging environment is browsable.

**Blocked by:** Epics 1–4

---

### W-032 · SEO and analytics

- `sitemap.xml` covering venues and packages, `robots.txt`
- Structured data: `LocalBusiness` for venues, `Product` for packages
- Analytics with conversion events on booking request and package inquiry
- Open Graph images

**Done when:** the sitemap generates from live data and conversion events fire.

**Blocked by:** W-031

---

### W-033 · Rate limit package inquiries

Package inquiries (W-024) are intentionally open to guests with no login.
Without rate limiting, the form is an easy spam / abuse vector.

**Needs a schema change** (or equivalent durable store) — for example an
`inquiry_rate_limits` table keyed by IP (and optionally phone/email) with a
rolling window count, or columns on an existing audit table. Do not ship a
silent schema edit; add a migration and update `schema.sql` / `schema.ts`
together.

Suggested behaviour:

- Limit submissions per IP (e.g. 5 per hour) and optionally per phone
- Return a clear field- or form-level error when exceeded
- Must work on Cloudflare Workers (no Node-only rate-limit libraries); prefer
  D1-backed counters or Cloudflare rate-limiting API if available on Pages
- Never block legitimate guests who are comparing a few packages in one sitting

**Done when:** burst submissions from one IP are rejected after the limit, a
normal guest flow still succeeds, and the limit is covered by a test.

**Blocked by:** W-024 (done); needs schema migration approval

---

### W-034 · Close the login timing side channel

`performLogin` returns immediately when no user matches the email, but runs a full
100,000-iteration PBKDF2 verify when one does. The failure *message* is identical, but
the response *time* is not — so an attacker can still enumerate which emails have
accounts by measuring how long the request takes. This defeats the protection W-004 was
explicitly written to provide.

- On the user-not-found path, run a dummy `verifyPassword` against a constant hash and salt so both branches do equivalent work before returning
- Keep the shared `LOGIN_FAILURE_MESSAGE` exactly as it is
- Add a test asserting that the not-found and wrong-password paths both call `verifyPassword`

**Done when:** both failure paths perform a password verification and the test proves it.

**Blocked by:** W-004 (done)

---

### W-035 · Admin CRUD for package images

W-021 delivered admin management for categories, services, packages, and items, but not
`package_images` — it wasn't in the ticket. Galleries currently depend on seeded R2 keys,
so an admin can publish a package but cannot give it photos.

- Admin upload, reorder, delete, and set-primary for package images
- Required `alt` text on every image
- Validate MIME type and size server-side; cap at 5MB per image
- Depends on R2 upload, so schedule alongside or after W-018

**Done when:** an admin can add images to a package and they render on the public detail
page.

**Blocked by:** W-021 (done), W-018

---

### W-038 · Support duration-based venue pricing

**Found during Tamil Nadu market research — the current model doesn't match reality.**

`mandaps.base_price` is a single per-day figure. Chennai kalyana mandapams commonly quote
in blocks instead: a 12-hour rate, a 24-hour rate, and a 48-hour rate, at quite different
prices. A venue quoting ₹88,500 for 12 hours and ₹1,77,000 for 24 hours cannot express
that today.

This will surface the first time a real venue tries to list, so fix it before onboarding
starts rather than migrating live listings later.

- Schema change: either add `price_12hr`, `price_24hr`, `price_48hr` columns, or a `mandap_pricing` table keyed by duration
- `base_price` stays as the "from" figure used in browse cards and filters
- Detail page shows the full rate card
- Booking request captures which duration the customer wants
- Migration must preserve existing seeded data

**Needs schema approval before starting.** Discuss the column-vs-table choice first —
a separate table is more flexible if venues later want per-season or per-day-of-week
rates, which is plausible given muhurtham date demand.

**Blocked by:** needs approval

---

### W-037 · Make `npm run dev` work with local D1

**Blocking: the app cannot currently be viewed in a browser with real data.**

`getDb()` reads `globalThis.Cloudflare.env`, which only exists inside a Workers
runtime. Plain `next dev` has no binding, so every page that queries the database
throws or renders empty. Fourteen tickets have shipped and none of them have been seen
working.

- Install `@opennextjs/cloudflare`
- Call `initOpenNextCloudflareForDev()` in `next.config.ts` — this injects the Cloudflare bindings into the Next dev server, and is the step everyone misses
- Add `.dev.vars` with `NEXTJS_ENV=local`, and confirm it is gitignored
- Verify `npm run db:migrate && npm run db:seed && npm run dev` serves real seeded data at `/mandaps` and `/packages`

**Done when:** `npm run dev` renders the six seeded mandaps and three published
packages, and login works with a seeded account.

---

### W-036 · Move production deploys and migrations to CI

Right now the local machine is authenticated to Cloudflare via `wrangler login`, so any
process running locally — including an AI agent in auto-run mode — can reach production
D1 and Pages. Cursor's allowlist is explicitly "best-effort, not a security boundary,"
and its denylist was deprecated, so configuration alone does not close this.

The fix is to remove the capability rather than ask nicely:

- GitHub Actions workflow that runs `wrangler d1 migrations apply --remote` and `wrangler pages deploy` on merge to `main`
- Cloudflare API token stored as a CI secret, scoped to the minimum needed
- Developers run `wrangler logout` locally, or authenticate only to a non-production account
- Production migrations require a merged PR — never an ad-hoc local command

**Done when:** a merge to `main` deploys, and no local machine holds credentials that
can write to production.

**Blocked by:** W-031

---

## Suggested sequencing

**Milestone 1 — Line B live.** Epic 0, then Epic 4, then deploy. Packages are entirely
under your control, need no venue supply, and can start generating inquiries while the
marketplace is still being seeded.

**Milestone 2 — Catalog visible.** Epics 1 and 6, plus admin listing review from Epic 3.
Venues become browsable. Run venue onboarding hard during this phase.

**Milestone 3 — Marketplace live.** Epics 2 and 3. Do not open publicly until 15–20
verified responsive venues are in the launch city — see the launch gate in the PRD.

**Milestone 4 — Trust.** Epic 5 and the rest of Epic 6. Reviews only become meaningful
once real bookings have completed.
