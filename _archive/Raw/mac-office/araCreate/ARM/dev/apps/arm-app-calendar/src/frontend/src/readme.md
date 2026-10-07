# SOURCE

The frontend's own source — a React + Vite app loaded as a Module Federation remote, no defined layout in the araCreate template's Next.js-oriented breakdown, so this follows a feature-folder convention instead:

```
src/
├── main.tsx                    # Entry point — mounts the UI provider and root App component
├── App.tsx                      # Router + top-level routes
├── vite-env.d.ts
├── config/                      # API base URL and env vars
├── assets/fonts/                 # Poppins font files
├── styles/                       # Global + calendar-scoped CSS
└── features/calendar/
    ├── components/                # Page-level components (account page, sync view/detail)
    ├── components/ui/              # Modals and dialogs
    ├── hooks/                       # use-calendar-sync, use-active-syncs, use-calendar-sse, use-theme, etc.
    ├── pages/                        # auth-callback.tsx (Google/Microsoft OAuth popup return page)
    ├── services/                     # api.ts — all fetch calls to the BFF proxy
    ├── types/                        # TypeScript types for the calendar domain
    └── constants/, utils/            # Shared constants and helpers
```

No third-party code is vendored here. See this app's own [CLAUDE.md](../../../CLAUDE.md) for the full architecture writeup.
