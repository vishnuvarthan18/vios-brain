# SOURCE

The shell's own source — a React + Vite app that hosts Module Federation remotes, no defined layout in the araCreate template's Next.js-oriented breakdown, so this follows a feature-folder convention instead:

```
src/
├── main.tsx                    # Entry point
├── App.tsx                     # Root component
├── routes/                     # BrowserRouter (web) / HashRouter (Electron), route table, lazy MF remote imports
├── api/                        # Axios clients (NoAuthClient / AuthClient) + per-domain services
├── components/                 # auth/, common/, home/, layout/
├── hooks/                      # use-auth, use-auth-sync, use-health, use-otp-verification, etc.
├── landing/                    # Marketing landing page + sections
├── layouts/                    # auth/ (unauthenticated) and home/ (authenticated shell) layouts
├── pages/                      # auth/ and home/ page components
├── store/                      # Zustand — auth-store, theme-store (theme-store exposed to remotes via MF)
├── shared/                     # toast.ts — exposed to remotes via MF
├── lib/                        # Runtime config (VITE_* env vars) and constants
├── types/                      # TypeScript type definitions
├── utils/                      # api-error, load-remote-module
├── assets/                     # Fonts, icons, images
└── test/                       # Vitest setup (setup.ts, msw/server.ts) — not a test suite, see ../tests/readme.md
```

Unit tests are colocated (`*.test.ts`/`.tsx` next to source, via Vitest) rather than in a separate directory — see `../tests/readme.md`. No third-party code is vendored here. See this app's own [CLAUDE.md](../CLAUDE.md) for the full architecture writeup, route map, and Module Federation wiring.
