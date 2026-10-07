# LIB

Business logic and data access for the web app.

| Path | What it is |
| --- | --- |
| [db/schema.ts](db/schema.ts) | The Drizzle schema — twelve tables, every row carrying `org_id` |
| [db/tenant.ts](db/tenant.ts) | **The tenant scope every query goes through.** No query bypasses it |
| [db/reports.ts](db/reports.ts) | The append-only accessor for `reports` — an insert and a read, and nothing else |
| [db/client.ts](db/client.ts) | The Postgres connection, opened on first use |
| [db/migrations/](db/migrations/) | Committed migrations. Never hand-edit the database |
| [public-key.ts](public-key.ts) | `pk_live_<8 hex>` generation and validation |

Two rules govern everything here, both from
[`docs/agent-rules.md`](../../../docs/agent-rules.md):

- **§1.7** Every query carries an org and project scope, via `tenant.ts`.
- **§1.1** `reports` is append-only. There is no update path and no delete
  path, in code or in the database, and none may be added.
