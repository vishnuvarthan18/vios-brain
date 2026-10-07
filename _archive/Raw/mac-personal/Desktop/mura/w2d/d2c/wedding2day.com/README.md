# wedding2day.com

Two product lines on one platform:

1. **Mandap marketplace** — browse third-party wedding venues, check availability, request a date. The venue manager confirms and money settles offline.
2. **Wedding packages** — wedding2day's own in-house team delivering end-to-end wedding services, sold as packages.

These are separate funnels. They don't bundle.

## Documentation

Read these before writing code.

| Doc | What's in it |
| --- | --- |
| [`AGENTS.md`](./AGENTS.md) | Rules for AI coding agents — constraints, conventions, definition of done |
| [`docs/PRD.md`](./docs/PRD.md) | Product requirements, scope, users, risks, success measures |
| [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) | Settled technical decisions and the edge-runtime constraints |
| [`docs/BACKLOG.md`](./docs/BACKLOG.md) | Sequenced tickets with acceptance criteria |
| [`src/db/schema.sql`](./src/db/schema.sql) | Canonical database schema |

## Stack

Next.js 16 (App Router), TypeScript, Tailwind CSS v4, deployed to Cloudflare Pages with
Cloudflare D1 for data and R2 for images.

**This runs on Workers, not Node.** `bcrypt`, `fs`, and native bindings will fail in
production even when the build passes. See `ARCHITECTURE.md` before adding any
dependency.

## Local development

```bash
npm install
npm run dev
```

Open http://localhost:3000.

The UI currently renders from placeholder data in `src/lib/mock-data.ts`. Ticket W-001
replaces this with real D1 queries.

## Database

Schema in `src/db/schema.sql`. To provision:

```bash
npx wrangler d1 create wedding2day-db
# copy the returned database_id into wrangler.toml
npx wrangler d1 execute wedding2day-db --file=src/db/schema.sql
```

## Deploying

```bash
npm install --save-dev @cloudflare/next-on-pages wrangler
npx @cloudflare/next-on-pages
npx wrangler pages deploy .vercel/output/static
```

## Current state

Scaffolding and page shells only — routes render from mock data, there's no auth and no
database connection yet. Start at ticket W-001 in the backlog.
