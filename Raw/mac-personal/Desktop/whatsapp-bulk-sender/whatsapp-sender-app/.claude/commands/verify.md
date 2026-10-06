---
description: Run the full verification gate and report results
---

Run `npm run verify` (lint + typecheck + unit + mock E2E) in the project root.

- If green: confirm in one line and stop.
- If red: show me the failing output, diagnose the root cause, and propose a fix — but do NOT change any file under `src/main/`, `src/preload/`, or `src/shared/` without telling me why first, and never weaken a test or a guardrail to make it pass.
