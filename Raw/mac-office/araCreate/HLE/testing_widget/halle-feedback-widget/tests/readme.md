# TESTS

Test suites and isolated prototypes, separate from `src/`.

Run the unit and database suites with `make test`, the widget in a browser
with `make test-widget`, and the dashboard end to end with `make test-e2e`.

| Suite | Covers |
| --- | --- |
| [db/schema-tenancy.test.ts](db/schema-tenancy.test.ts) | Every table carries `org_id`, and every table but `organisations` and `users` carries `project_id` |
| [db/tenant-scope.test.ts](db/tenant-scope.test.ts) | The scope helper rejects a missing or empty `org_id` / `project_id` |
| [db/reports-append-only.test.ts](db/reports-append-only.test.ts) | **No update or delete path for `reports` is exported from the data layer** |
| [db/public-key.test.ts](db/public-key.test.ts) | The generator matches `^pk_live_[0-9a-f]{8}$` |
| [db/seed-idempotent.test.ts](db/seed-idempotent.test.ts) | Running the seed twice adds no second organisation and no duplicate pages |
| [db/classifications.test.ts](db/classifications.test.ts) | Types: add, rename, hide; classifying a report keeps status and type in step, with history, under a race |
| [db/users-admin.test.ts](db/users-admin.test.ts) | Users: admin only, the last admin cannot go, a reset signs the person out |
| [db/bug-steps.test.ts](db/bug-steps.test.ts) | A bug's steps in order: Processing, Fixed, Closed only after a check |
| [e2e/](e2e/) | The dashboard in a real browser: every screen, classifications, a bug's steps, Users, Settings |

`e2e/` runs on its own server (:3201) and the **test** database, reset and
filled by [e2e/prepare-db.mjs](e2e/prepare-db.mjs) at the start of every run.
The dev server and dev database are never touched. It cannot run at the same
time as `make test`, which resets the same test database.

`reports-append-only` is the one that matters most. It is the rule the whole
audit trail depends on, and it fails loudly the day someone adds a mutation.

What has to be covered before a later milestone is called done:

- The widget leaves the host page untouched when the API is broken or
  JavaScript is disabled.
- A keyboard-only run completes a full report.
- Two matching reports produce one issue, and both reports stay readable.
- No action anywhere mutates or deletes a row in `reports`.
- The widget stays under 15 KB gzipped.
- Every screen renders in German, including every failure state.

Browser matrix includes Safari and iPad — the testers are not on Chrome.
