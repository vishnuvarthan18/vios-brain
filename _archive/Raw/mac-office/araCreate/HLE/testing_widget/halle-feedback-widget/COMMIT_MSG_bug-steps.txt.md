---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_bug-steps.txt
---

feat(admin): put bug steps in order with step line and next actor

Closed means a stakeholder checked the fix, so a bug now moves in order:
Processing, Fixed, then Closed. The server refuses a step out of order,
so nobody can close a bug that was never marked fixed, and a failed
check sends it back to Processing.

Each report shows its steps, whose move it is next, and one main button
per step: Mark fixed, then Confirm fix or Not fixed, reopen. The Fixed
tab is called To check, and Overview leads with the fixed bugs waiting
for a check. admin-v2-spec §4 corrects what Closed means.
