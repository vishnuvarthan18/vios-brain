---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M0.txt
---

build: scaffold workspaces, schema, migrations and widget size gate

M0 foundation. No feature code: no widget UI, no dashboard screens, no API
routes. The skeleton later milestones fill in.

Two npm workspaces. src/web is Next.js with the App Router and the layout from
repo conventions 2.1. src/widget is TypeScript bundled by esbuild to
dist/v1.js and declares no dependencies at all — esbuild is a build tool in the
repo root, so the widget's own dependency list stays empty and verifiably so.

Twelve tables in Drizzle, migrations committed. Every row carries org_id, and
every table except organisations and users carries project_id. organisations
sets org_id to its own id rather than taking a self-referencing foreign key,
which keeps the "every table has org_id" rule mechanically checkable without a
cycle; projects does the same for project_id.

Tenant scoping is one helper. tenant_scope validates the org and project pair
and throws on a missing, empty or non-uuid value. scoped_where builds the
predicate every statement carries, and scoped_values stamps both columns onto
every insert, so an unscoped insert is as impossible as an unscoped read.
Nothing in lib/db exposes a raw table query.

reports is append-only, enforced three ways rather than one. The accessor
exports an insert and a read and nothing else, and it is the only module that
touches the table. A migration adds BEFORE UPDATE, BEFORE DELETE and BEFORE
TRUNCATE triggers, so a mutation fails from psql and from any future ORM call,
not only from reviewed code. A test audits the exported surface, greps every
source file, and asserts the database refuses all three operations. The guard
was checked by adding a deliberate delete path and confirming all four
assertions fire.

The size gate builds fresh before measuring, so it cannot pass on a stale
artefact, and exits non-zero above 15360 bytes gzipped. Verified in both
directions: 260 bytes passes, a padded bundle fails at 24439.

The database connection opens on first use rather than at import, so importing
the data layer never needs a live database.

Two decisions taken from the revised build plan, which was updated during the
work and outranks the task brief: users.role is staff, developer or client,
with a CHECK constraint; and the placeholder pages and root layout are English,
since German is scoped out on record.
