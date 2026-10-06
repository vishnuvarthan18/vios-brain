# Halle Feedback Widget

A feedback tool for website testing, built for people who are not comfortable
with computers. A tester on the live site points at whatever looked wrong,
answers one question in plain words, and optionally types a sentence. The team
sees what came in, and works it to a resolution.

Built for B. Halle (halle-dev.webflow.io) via araCreate. Testers are 10–30
elderly, German-speaking, non-technical people across 49 pages.

**What it does**

- Sits on the client's live Webflow site as one script tag.
- Asks one question with five plain options, in plain English.
- Records which pages have been checked, and which nobody has looked at.
- Turns incoming reports into issues with a status, an owner and a history.
- Exports the whole record as CSV — the client's sign-off evidence.
- Writes nothing to the tester's device, and asks for no browser permissions.
- Lets staff manage pages, testers, assignments and every tester-facing
  string from the app — no rebuild, no deploy, no developer required to
  reword the widget.

## Stack

| Layer | Tool |
| --- | --- |
| Widget | Vanilla JS, zero dependencies, esbuild, Shadow DOM, <15 KB gzipped |
| Web app | Next.js (App Router) |
| API | Next.js route handlers — two public endpoints, the rest server actions |
| Database | Postgres — `postgresql@17` locally, EU cloud region later |
| Data access | Drizzle, migrations in the repo |
| Auth | Staff, developer and client logins — testers never log in |
| Screenshot storage | Local disk behind an interface (`src/web/lib/storage/`), self-signed upload urls — no S3 |
| Workspaces | npm |

## Layout

```
src/web/      Next.js app — dashboard + public widget API
src/widget/   the embeddable script
docs/         build plan, agent rules, specification
tests/        test suites and prototypes
releases/     the versioned widget bundles as shipped
```

## Data model

Seven core tables — `organisations`, `users`, `projects`, `pages`, `testers`,
`assignments`, `reports`. Every row carries `org_id`, and every table except
`organisations`, `users` and `login_attempts` also carries `project_id`.
`login_attempts` (login lockout) carries no tenant scope at all, by design —
see [`docs/web/readme.md`](docs/web/readme.md).

Then the working layer: `issues`, `issue_reports`, `comments`, `issue_events`,
`config_revisions`.

The one rule to understand before reading the schema: **`reports` are
append-only evidence and never change.** An `issue` is the mutable thing the
team works on, and it holds one or more reports. No report is ever hidden or
discarded by grouping.

## Commands

```sh
make install      # install dependencies
make setup        # top up the local environment file
make dev          # run the web app
make dev-widget   # rebuild the widget on change
make build        # build both
make size         # check the widget's 15 KB gzipped budget
make db-migrate   # apply migrations
make db-seed      # seed the database
make user-create  # create a login (EMAIL=... NAME=... ROLE=...)
make user-disable # disable a login (EMAIL=...)
make user-enable  # re-enable a login (EMAIL=...)
make user-admin   # make a login an admin (EMAIL=..., REVOKE=1 to undo)
make lint         # lint and type-check
make test         # run tests
make test-e2e     # run the dashboard end to end in a browser (test database)
make release      # cut a semantic release
make clean        # remove build artefacts
```

`make help` lists them.

## Conventions

See [aracreate-conventions](https://github.com/aracreate-group/aracreate-conventions).

Project-specific rules, and the reasons behind them, are in
[`docs/agent-rules.md`](docs/agent-rules.md). Read it before writing code —
several of the constraints look arbitrary and are not.

## License

Proprietary. Copyright (C) 2026, B. Halle. See [LICENSE](LICENSE).
