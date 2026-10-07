---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_snake-case.txt
---

refactor: rename code and widget contract to snake_case

The araCreate conventions want snake_case for every variable, property
and JSON key; this repo used camelCase for props, row fields and the
widget's API. Renamed with the TypeScript language service, so every
use moved with its declaration, in the dashboard, the widget and tests.

The widget and server now exchange tester_token, page_title,
capture_method, upload_url, token_expired and the capture fields in
snake_case, and the saved widget strings use snake_case keys. Migration
0015 renames those keys in projects.config and every config revision,
leaving the wording itself untouched.

Names a library or the browser reads keep their own spelling; the
exceptions are listed in agent-rules §5. The server, the widget bundle
and migration 0015 must be deployed together.
