---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_classifications.txt
---

feat(admin): classify queue reports with custom types

Bug keeps its flow (Processing, Fixed, Closed). Every other type is the
team's own, made on a new Classifications screen, with two steps: Open
and Done, listed on a new Classified page, one tab per type.

- Queue popup: one button per type beside Bug
- Any classified report can change type later; the history records it
- Types are hidden, never deleted; a hidden type stays on its reports
- Migration 0013: classifications, report_classifications, open/done
  statuses, type columns on report_events. reports stays append-only:
  status is still its only changing column
- CSV export gains a type column; Overview counts classified reports
- launch-reset clears report types with the reports

Spec: docs/classifications-spec.md
