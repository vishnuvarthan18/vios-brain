---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_conventions-files.txt
---

chore: bring file headers, names and json meta in line with conventions

Every source file now opens with the licence header, a "This file
contains" description and one blank line; the shadcn components, the
global stylesheet, three migrations and one test had none. File names
are param-case (runbook.md, session-handover.md,
apache-halle-feedback-optional.conf), the deploy readme heading is
uppercase, and JSON data files carry _meta, with the A/B scripts writing
and reading the new { _meta, data } shape.
