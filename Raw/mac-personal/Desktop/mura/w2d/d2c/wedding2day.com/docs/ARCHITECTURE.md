# Architecture Decisions

Companion to `PRD.md`. Every decision here is settled — build against it, don't
relitigate it mid-ticket. If something turns out wrong, change it here first, then in
code.

---

## Stack

| Layer | Choice | Version |
| --- | --- | --- |
| Framework | Next.js, App Router | 16.x |
| Language | TypeScript, strict | 5.x |
| Styling | Tailwind CSS | 4.x |
| Hosting | Cloudflare Pages | — |
| Database | Cloudflare D1 (SQLite) | — |
| ORM | Drizzle | latest |
| Object storage | Cloudflare R2 + Images | — |
| Validation | Zod | 3.x |

---

## The constraint that shapes everything: the edge runtime

Cloudflare Pages runs Workers, not Node. A large share of the npm ecosystem assumes
Node APIs and will fail — sometimes at build time, sometimes only in production.

**Not available:** `fs`, `net`, `child_process`, native `.node` binaries, most of
`crypto`'s Node-specific surface, and anything depending on them.

**Consequences to internalize:**

- **bcrypt does not work.** Neither does argon2. Use PBKDF2 via Web Crypto (`crypto.subtle`), which is available on Workers. This is why `users` has both `password_hash` and `password_salt`.
- **NextAuth / Auth.js is a poor fit.** Adapters assume Node. We roll our own sessions — it's ~150 lines and fully under our control.
- **No local filesystem.** Uploads go straight to R2. Never write to disk.
- **Check every new dependency for Workers compatibility before adding it.** If it isn't documented as edge-compatible, assume it isn't.

---

## Authentication

Custom, server-side, session-based. No JWTs.

- Password hashing: PBKDF2-SHA256 via `crypto.subtle`, minimum 100k iterations, unique random salt per user, both stored on the user row.
- On login: create a row in `sessions` with a cryptographically random id and an expiry (30 days), set it as an `httpOnly`, `Secure`, `SameSite=Lax` cookie.
- On each request: look up the session, join the user, expose `{ user, session }` to server components via a cached `getSession()` helper.
- On logout: delete the session row and clear the cookie.
- Expired sessions are cleaned up by a scheduled Worker, not on the request path.

**Why not JWTs:** we need to revoke sessions instantly when an admin suspends a
manager. Stateless tokens can't do that without a blocklist, which is a session table
with extra steps.

**Route protection** lives in `src/lib/auth/guards.ts` — `requireUser()`,
`requireRole('manager')`, `requireApprovedManager()`. Call these at the top of server
components and server actions. Middleware handles coarse redirects only; it must not be
the sole authorization check, because server actions bypass it.

---

## Data access

Drizzle ORM against D1. Schema lives in `src/db/schema.ts`, mirroring `schema.sql`.

**Rules:**

- All queries go through repository functions in `src/lib/repositories/`. Components never build queries inline.
- Every repository function that returns user-scoped data takes the acting user as an argument and filters by it. A manager must never be able to read another manager's bookings by changing a URL id.
- Migrations via `drizzle-kit` and `wrangler d1 migrations`. Never hand-edit a shipped migration.

**D1-specific limits worth knowing:** no `RETURNING *` in older bindings, limited
transaction support (use `batch()` for atomicity), and a query size ceiling. Keep
queries simple; do joins in SQL, not in JavaScript loops that fan out into N+1.

---

## Rendering

Server Components by default. `"use client"` only where genuinely needed: forms with
live validation, the availability calendar picker, image galleries, filter panels with
instant feedback.

- Public pages (venue browse, venue detail, packages) are statically generated where possible and revalidated on write.
- Dashboards are dynamic and never cached.
- Mutations use **Server Actions**, not API routes. Every action validates its input with Zod and re-checks authorization — never trust that the UI hid a button.

---

## Images

Cloudflare R2 for storage, Cloudflare Images for resizing and delivery.

- Uploads go through a server action that issues a presigned URL; the browser uploads directly to R2.
- Store the key in `mandap_images.url`, not a full URL, so the delivery domain can change.
- Validate MIME type and size server-side. Cap at 5MB per image, 15 images per venue.
- Every image needs an `alt`. It's an accessibility requirement and it's SEO.

---

## Search and filtering

D1 is SQLite. For the expected catalog size — hundreds to low thousands of venues per
city — indexed `WHERE` clauses are entirely sufficient. Do not reach for a search
service.

- Filters map to indexed columns: `city`, `capacity_max`, `base_price`, plus a join for amenities.
- Date availability filters via `NOT EXISTS` against the sparse `availability` table.
- Text search: SQLite `LIKE` on name and area is fine for v1. If it becomes inadequate, add an FTS5 virtual table before considering anything external.
- Paginate with cursor-based pagination, not `OFFSET`. Offset degrades and produces duplicates when rows shift.

---

## Availability model

`availability` is **sparse**: a row exists only for dates that are blocked or booked.
No row means the date is open.

This matters. The naive alternative — a row per venue per day — is 365 rows per venue
per year and makes calendar writes miserable. With the sparse model, opening a date is
a delete.

On confirming a booking, write the `availability` row and update the booking status in
a single `batch()` so the two can't diverge.

---

## Notifications

Email via Resend (works on Workers, simple API). Triggered from server actions after
the database write commits, never before.

Email will probably not be enough for venue managers — see the PRD risk section. WhatsApp
Business API is the likely follow-up. Structure the notification layer as
`src/lib/notifications/` with a channel-agnostic interface so adding WhatsApp doesn't
mean rewriting call sites.

Notification failures must never fail the user's request. Log and move on.

---

## Project structure

**This is the target structure. The repo does not match it yet** — the initial scaffold
was flat. Ticket W-000 does the reorganization and must run before anything else.

```
src/
  app/
    (public)/          # marketing, browse, detail — no auth required
      layout.tsx       # Nav + Footer, applied once
    (auth)/            # login, signup
    manager/           # manager portal, guarded
    admin/             # admin portal, guarded
  components/
    layout/            # Nav, Footer
    ui/                # primitives: Button, Input, Card
    venue/             # venue-specific composites
    package/
  lib/
    auth/              # session, guards, password hashing
    repositories/      # all database access
    notifications/
    validation/        # Zod schemas, shared client and server
    utils/
  db/
    schema.ts          # Drizzle schema
    schema.sql         # canonical DDL, source of truth
    migrations/
    seed.ts
```

Route groups — the parenthesised folders — don't affect URLs. `(public)/page.tsx` still
serves `/`. They exist so public pages share one layout carrying Nav and Footer, instead
of every page importing both itself.

---

## Environments

- **Local:** `wrangler dev` with a local D1 instance. Seed data via `npm run db:seed`.
- **Preview:** every branch deploys to a Pages preview with its own D1 binding pointed at a staging database.
- **Production:** `wedding2day.com`, separate D1 database. Never run untested migrations against it.

Secrets live in Cloudflare environment variables, never in the repo. `.dev.vars` is
gitignored and holds local secrets.

---

## Testing

Proportionate, not exhaustive.

- **Unit tests (Vitest)** for anything with real logic: password hashing, availability calculation, price formatting, Zod schemas.
- **Integration tests** for every server action, covering the authorization path explicitly. The test that matters most is "manager B cannot confirm manager A's booking."
- **No E2E suite in v1.** Not worth the maintenance at this stage.

Every ticket that touches authorization must include a test proving the negative case.

---

## Decisions deliberately deferred

Payments, multi-city tooling, a public API, and third-party vendor onboarding. Don't
build hooks for these speculatively — the schema is easy enough to extend later, and
speculative abstraction costs more than it saves.
