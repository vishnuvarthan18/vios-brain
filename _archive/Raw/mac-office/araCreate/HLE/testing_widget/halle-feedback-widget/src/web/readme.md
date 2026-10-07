# WEB

The Next.js application. Two jobs in one deployable:

- **The dashboard** — report grid, reports, issues, pages, testers,
  assignments, string editor. Staff, developer and client logins only.
- **The public API** — the two endpoints the widget calls, `GET /api/v1/config`
  and `POST /api/v1/reports`.

Run it with `make dev` from the repo root, on <http://localhost:3000>.

**First run**

```sh
make install     # dependencies
make setup       # writes src/web/.env from .env.example
                 # then set DATABASE_URL to your local Postgres
make db-migrate  # twelve tables, plus the append-only triggers
make db-seed     # araCreate, the B. Halle project, and the placeholder pages
make dev
```

Local Postgres is `postgresql@17`, matching the version CI runs.

Layout follows
[repo §2.1](https://github.com/aracreate-group/aracreate-conventions/blob/main/repo/readme.md#21-src-by-project-type)
— `app/`, `components/`, `lib/`, `types/`, `data/`, `public/`.

Depth is in [`docs/web/`](../../docs/web/).
