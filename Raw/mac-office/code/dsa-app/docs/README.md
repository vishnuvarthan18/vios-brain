# DreamSpace Academy

Youth makerspace platform for DreamSpace Sri Lanka.

## Prerequisites

- Node.js 20+
- pnpm 9+ (`npm install -g pnpm`)
- Docker (for local PostgreSQL + MongoDB)
- Supabase CLI (`npm install -g supabase`)

## Project structure

```
dreamspace/
├── CLAUDE.md          # Claude Code context — read first
├── api/               # Express + Prisma backend
├── web/               # React + Vite (coordinators + admins)
├── mobile/            # Capacitor (trainers + parents)
└── public/            # Next.js (portfolio showcase + cert verify)
```

## Initial setup

### 1. Clone and install

```bash
git clone https://github.com/dreamspace/platform.git
cd dreamspace
pnpm install        # installs all workspaces
```

### 2. Environment variables

```bash
cp api/.env.example api/.env
cp web/.env.example web/.env
cp mobile/.env.example mobile/.env
# Fill in all values — see each .env.example for details
```

### 3. Database setup

```bash
# Start local Supabase (PostgreSQL + Realtime + Storage)
supabase start

# Run migrations
cd api
npx prisma migrate dev --name init

# Create global sequences (run once)
npx prisma db execute --file prisma/sequences.sql

# Seed initial data (platform sections, terms version)
npx prisma db seed
```

### 4. Run locally

```bash
# Terminal 1 — API
cd api && pnpm dev          # http://localhost:4000

# Terminal 2 — Web
cd web && pnpm dev          # http://localhost:5173

# Terminal 3 — Mobile (web preview)
cd mobile && pnpm dev       # http://localhost:5174
```

### 5. Native mobile (when needed)

```bash
cd mobile
pnpm build
npx cap sync ios            # or android
npx cap open ios            # opens Xcode
```

## Key commands

```bash
# Generate Prisma client after schema changes
cd api && npx prisma generate

# Create a new migration
cd api && npx prisma migrate dev --name describe_change

# Open Prisma Studio (DB browser)
cd api && npx prisma studio

# Type check all packages
pnpm typecheck

# Lint all packages
pnpm lint
```

## Architecture decisions

See `CLAUDE.md` for the full context including all architectural decisions, business rules, and constraints.

## Contributing

- **[git-workflow.md](./git-workflow.md)** — source of truth for branching, Conventional Commits, PR flow, releases, and the rules Claude Code follows. (`main` / `staging` / `develop` + `feature|bugfix|hotfix|release|chore|docs/*`.)
- **[KNOWN-ISSUES-CHECKLIST.md](./KNOWN-ISSUES-CHECKLIST.md)** — verification checklist distilled from real bugs on KathiraGreens and Viyanix. Run the relevant sections before opening any PR.

## Deployment

- **[DEPLOYMENT.md](./DEPLOYMENT.md)** — Dokploy (self-hosted VPS) setup for all three apps (API, web PWA, public Next.js). Covers Docker files, environment variables, one-time DB setup, auto-deploy webhooks, rollback, and troubleshooting.

Never commit directly to `main`, `staging`, or `develop` — always branch first.
