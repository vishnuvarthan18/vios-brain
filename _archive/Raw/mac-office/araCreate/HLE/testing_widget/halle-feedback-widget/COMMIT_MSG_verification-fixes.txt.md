---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_verification-fixes.txt
---

fix: repair db-fixture after M7

`scripts/db-fixture.mts` still imported `group_report` from
`lib/db/issues.ts`, a file M7 deleted along with the whole issues/roles
working layer. `make demo` (and `make db-fixture` on its own) failed
outright on a fresh checkout. Rewrote the reports-generation step to
insert v2-shaped rows directly (comment/mode/markup/meta/status),
matching the schema and submit_report's own shape — no grouping step
exists any more, so none is called. Reports now get a fixed status
rotation (queue/bug/fixed/closed) instead of all landing in the queue,
so the demo queue and tracked-items screens both have something to show.

Verified with a full dropdb/createdb + `make demo` run on a fresh dev
database, clean end to end, run twice.

Files touched: src/web/scripts/db-fixture.mts.

Note: the picture-viewer zoom-toggle CSS fix, found in the same
verification pass, is committed separately as part of the design-system
rebrand (next commit) rather than here — by the time it was written it
had already picked up the rebrand's rounded spacing values, so splitting
it back out would have meant re-deriving the pre-rebrand pixel values
from scratch for no real benefit.
