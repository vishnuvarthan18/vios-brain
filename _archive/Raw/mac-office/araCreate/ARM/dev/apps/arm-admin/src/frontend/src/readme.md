# SOURCE

The admin remote's own source — a React + Vite app loaded as a Module Federation remote at `/admin/*`, no defined layout in the araCreate template's Next.js-oriented breakdown, so this follows a feature-folder convention instead:

```
src/
├── AdminApp.tsx           # MF entry point — exposed as ./AdminApp
├── main.tsx                # Standalone dev only — wraps AdminApp in BrowserRouter
├── api/                     # adminClient (axios → BFF proxy), usersApi, auditApi
├── components/               # admin-auth-provider (403 fallback), access-denied
├── layouts/                   # admin-layout — header + tabs nav
├── pages/
│   ├── users/                  # Paginated list, search, disable/enable/delete, edit form
│   └── audit/                   # Paginated audit log + filter
├── routes/                       # <Routes> only — no Router wrapper, lives in host's BrowserRouter
├── types/                         # TypeScript types mirroring backend DTOs
└── lib/                             # constants, toast (lazily imports host's MF toast)
```

No third-party code is vendored here. See this app's own [CLAUDE.md](../CLAUDE.md) for the full Module Federation wiring and route map.
