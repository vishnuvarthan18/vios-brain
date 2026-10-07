# wedding2day.com — agent instructions

Read this before writing any code. `docs/PRD.md` has product requirements,
`docs/ARCHITECTURE.md` has settled technical decisions, `docs/BACKLOG.md` has the
ticket queue.

---

## What this product is

Two **separate** product lines on one site. They do not bundle.

1. **Mandap marketplace** — third-party wedding venues. Customer requests a date, venue manager confirms or rejects, money settles offline.
2. **Wedding packages** — wedding2day's own in-house services team (photography, decor, catering, makeup, and more), sold as packages. Customer submits an inquiry, our staff follow up.

A booking request and a package inquiry are different objects with different tables and
different funnels. Never merge them.

Three roles: `customer`, `manager`, `admin`.

---

## Hard constraints — violating these breaks production

**This runs on Cloudflare Workers, not Node.** The build may pass and still fail live.

- **Never use `bcrypt`, `argon2`, `fs`, `net`, `child_process`, or any package with native bindings.** Password hashing is PBKDF2 via `crypto.subtle`. This is not negotiable.
- **Never add a dependency without confirming Workers/edge compatibility.** If the docs don't say it works on edge runtimes, don't add it. Flag it instead of guessing.
- **Never write to the filesystem.** Uploads go to R2.
- **No `localStorage` or `sessionStorage` for anything security-related.** Sessions are server-side rows, always.

---

## Non-negotiable rules

**Authorization is checked server-side, in every server action, every time.** Hiding a
button in the UI is not authorization. Every action that touches user-scoped data
re-validates that the acting user owns or may access that record. The specific bug to
never ship: manager B confirming manager A's booking by changing an id.

**All database access goes through `src/lib/repositories/`.** No inline queries in
components or actions. Repository functions that return user-scoped data take the
acting user as a parameter and filter by it.

**All mutations use Server Actions with Zod validation.** Not API routes. Validate
input at the top of the action, before anything else.

**Scraped venue data is never published.** `venue_prospects` is admin-only and must
never be rendered on a public page or returned by a public query. Published venues live
in `mandaps` and only exist after a real venue onboards. If a ticket seems to ask you to
show prospect data publicly, stop and flag it.

**Availability is sparse.** A row in `availability` exists only for blocked or booked
dates. No row means open. Never generate a row per day.

---

## Conventions

**Components.** Server Components by default. Add `"use client"` only for interactive
forms, the date picker, galleries, and filter panels. Keep client components small and
push them to the leaves of the tree.

**Naming.** Components `PascalCase.tsx`. Everything else `kebab-case.ts`. Repository
functions read as verbs: `getPublishedMandaps`, `createBookingRequest`,
`confirmBookingRequest`.

**Types.** No `any`. No `@ts-ignore`. If types fight you, fix the type. Zod schemas in
`src/lib/validation/` are the single source of truth for shapes crossing the
client/server boundary — infer TypeScript types from them rather than declaring twice.

**Styling.** Tailwind utility classes only. No CSS modules, no styled-components, no
inline `style` except for genuinely dynamic values. Mobile-first: base styles target
mobile, `sm:` and up layer on desktop. Most users are on phones.

**Money.** Store as `REAL` in paise-free rupees. Format for display with
`toLocaleString("en-IN")` and a `₹` prefix. Never format inside a repository function —
format at the render layer.

**Dates.** Store as ISO `yyyy-mm-dd` strings, not timestamps, for event and availability
dates. Timestamps (`created_at`, etc.) are Unix integers. Never rely on the server's
local timezone.

---

## Definition of done

A ticket is done when all of the following hold:

- `npm run build` passes with no TypeScript errors
- `npm run lint` passes
- Every acceptance criterion in the ticket is met
- Authorization tickets include a test proving the negative case
- The page works at 375px width, not just desktop
- No `console.log` left behind
- Loading and error states exist for anything that fetches data
- Empty states exist for any list that can be empty

---

## Never run these

These reach production infrastructure. Never run them, even if a ticket seems to call
for one — tell me and I will run it myself:

- `wrangler d1 execute --remote` — writes to the production database
- `wrangler d1 migrations apply --remote` — runs DDL on production
- `wrangler pages deploy` — ships to the live site
- `git push --force` — destroys history
- `rm -rf` on anything outside `node_modules` or `.next`

Local equivalents are fine: `npm run db:migrate` and `npm run db:seed` are guarded to
local-only and are safe to run freely.

## When to stop and ask

Don't guess on these — flag them instead:

- A ticket requires a dependency you can't confirm is edge-compatible
- A ticket seems to contradict `PRD.md` or `ARCHITECTURE.md`
- A ticket would require publishing prospect data, adding a payment flow, or bundling the two product lines
- The schema doesn't support what the ticket asks for and you'd need to add a table

Schema changes are fine, but they need a migration and a note in the ticket — not a
silent edit to `schema.sql`.
