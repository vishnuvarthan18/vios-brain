---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M5.txt
---

feat: add M5 — admin

Gives staff a string editor, pages, testers, assignments and team screens so
a non-technical person can reword the widget, manage the page list and spin
up testers without a developer touching code or redeploying anything.

Every tester-facing string is now editable in one form, validated against a
zod schema, written as a config_revisions row before projects.config itself
updates, with rollback restoring a chosen revision as a new one rather than
erasing history. Bulk page import normalises through the same function
POST /reports already uses to resolve page_id, so the widget and the app can
never disagree about whether a page counts. The assignment generator is a
pure, seedable plan builder with a preview/commit split, committing through
onConflictDoNothing so running it twice — sequentially or at once — never
duplicates. Every mutation is staff-only, checked server-side through a new
admin-permissions module in the same shape as M4's issue permissions.

assignments and config_revisions already existed as tables since the initial
schema; this milestone is the admin logic and UI over them, no new
migration. Two real bugs surfaced while building this and are fixed: a
'use client' page-type select pulling the Postgres driver into the browser
bundle (moved the closed page_type set to its own dependency-free module),
and the testers invitation link missing its required slash before the query
string on a realistic site_url with no trailing slash, while also
re-querying the project once per row instead of once for the whole list.
