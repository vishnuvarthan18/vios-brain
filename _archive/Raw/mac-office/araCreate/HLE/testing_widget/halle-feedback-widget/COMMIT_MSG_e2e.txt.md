---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_e2e.txt
---

test(e2e): drive dashboard end to end in browser

A Playwright suite for the dashboard, beside the widget's: every screen
on desktop and phone, classifications, Users and Settings. 21 tests.

- Runs on its own server (:3201, build folder .next/e2e) against the
  test database, which tests/e2e/prepare-db.mjs resets and fills with
  the project's own seed, fixture and demo scripts on every run. The
  dev server and dev database are never touched
- make test-e2e; tests/readme.md and README list it
- tsconfig includes .next/e2e/types, so a run no longer rewrites it
