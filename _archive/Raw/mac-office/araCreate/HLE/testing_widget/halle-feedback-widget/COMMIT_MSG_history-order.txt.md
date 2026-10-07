---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_history-order.txt
---

fix(db): stamp report history when written, not when transaction began

now(), the column default, is the start of the transaction. A change
that waited on the report's row lock began before the change it waited
for, so its history row sorted first though it happened second, and
the timeline could end on a status the report no longer had. Found by
a racing-classifications test; the same gap was in the status history.
