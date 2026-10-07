# WEB AND APPLICATION SOFTWARE

The `src/` layout for web apps, APIs and services. Referenced from [repo §2.2](../readme.md#22-src-by-project-type).

## Next.js (App Router)

```
src/
├── app/            # Routes, layouts, route handlers
├── components/     # UI components — ui/ for primitives, the rest by feature
├── lib/            # Business logic, data access, clients, utilities
├── hooks/          # React hooks
├── types/          # Shared TypeScript types
├── styles/         # Global CSS, theme tokens
└── data/           # Static JSON and content
public/             # Static assets — stays at repo root, Next.js requires it there
```

- **`app/` holds routing and nothing else.** A route file that grew logic has logic that belongs in `lib/`. The test for it: if you cannot call it without a request, it is in the wrong place.
- **`components/ui/` is primitives** — button, input, dialog — and knows nothing about the domain. Everything else in `components/` is grouped by feature, not by type. `components/checkout/` beats `components/forms/` the moment there are three features.
- **`lib/` is where the project actually lives.** Framework-free where it can be, so it is testable without rendering anything.
- **`public/` is at the repo root**, not under `src/`. Next.js requires that path; this is the one place the framework wins over the convention.

## API or service (no framework opinion)

```
src/
├── routes/         # HTTP surface — routing and validation only
├── services/       # Business logic, one module per domain concept
├── models/         # Data models and schema
├── db/             # Connection, queries, migrations
├── middleware/     # Auth, logging, error handling
├── config/         # Configuration loading and validation
└── utils/          # Genuinely generic helpers
```

- **`routes/` parses and delegates.** Validation of the request shape belongs here; deciding what to do with it does not.
- **`services/` never imports from `routes/`.** The dependency runs one way. When it stops doing so, the service has become a route handler with extra steps.
- **`config/` validates at startup and fails loudly.** A missing variable should stop the process on boot, not surface as `undefined` at 3am. Read every variable in one place; scattered `process.env` access is how `.env.example` falls out of date.
- **`utils/` is for things with no domain meaning.** Date formatting, yes. `calculateTax`, no — that is a service. A `utils/` folder that grows past a dozen files is a folder of misfiled services.

## Monorepo

Where one repo builds more than one deployable thing:

```
src/
├── apps/
│   ├── web/
│   └── api/
└── packages/
    ├── shared/     # Types and logic used by more than one app
    └── ui/         # Shared components
```

- **A package exists when two apps need it**, not in anticipation. Extracting later is cheap; unpicking a premature shared package is not.
- **Docs stay in `docs/<app>/`**, never inside the app folder. The copy nearer the code wins every time, so the one under `docs/` rots — keep only one, at the top.
- **Each app folder is its own build context.** A file moved to the repo root to be shared is absent from that app's container image. Things a human reads move out; things the build needs stay in.

## Conventions that apply to all of the above

- **One export per file for components**, named the same as the file.
- **Path aliases over relative chains.** `@/lib/auth`, not `../../../lib/auth`. Configure the alias once in `tsconfig.json`.
- **Tests live in `tests/`**, mirroring the `src/` structure. Co-located test files are fine for a component's own unit test; anything spanning modules goes in `tests/`.
- **Environment variables are read in one module** and exported typed. See [repo §4.3](../readme.md#43-environment).
