---
tags: chat
project: "[[Projects/feedback-widget/SUMMARY]]"
date: 2026-09-02
source: Claude office account (Cowork)
session: cse_01NX31fEoMzdKqg2qmSTrrtZ
---
# Planning and readiness check

- Project: [[Projects/feedback-widget/SUMMARY]]
- Date: 2026-09-02
- Full chat, word for word (tool steps shortened).

---

**Vishnu:** do we have all the thins needed can we plan for buiding.. jsut asnwer

**Claude:**   _[tool: Projects — claude/PROJECT-INDEX.md]_

**Claude:**   _[tool: Projects — claude/BUILD-SPEC.md]_

**Claude:** Yes — you can start building. Nothing missing is a design question; what's left is small facts and one piece of content.

**Ready:** schema, API contract, all five widget states, full English copy, visual spec, assignment rules, dashboard scope, phase split with acceptance criteria. A build agent could start Phase 1 today on the English path.

**Three things to clear first (about a day, mostly not yours):**

1. **How the live site switches language** — 10 minutes on halle-dev.webflow.io. This genuinely blocks the D2 language work, since the widget has to read whatever the site's switcher sets.
2. **The German five answers** — don't exist yet, can't be machine-translated. ~half a day of content work by someone with the research context.
3. **BUILD-SPEC rewrite** — it still says `page_ok` and reminder emails throughout. `scope-decisions.md` overrides it in four places, and handing a build agent two documents that contradict each other is how you get the wrong widget. This is the one thing I'd insist on before code starts.

**Non-blocking, can run in parallel:** the 404-page check, confirming Jakob's Webflow plan allows custom code, the "page opened" logging decision, the tester list.

**Not a build problem but will decide whether testing succeeds:** with no confirmations and no reminder emails, there's no completion signal and no nudge. Recruitment plus the ~20-minute onboarding now carries all of it, and neither is planned yet (open items 8 and 10).

Want me to do the BUILD-SPEC rewrite so the build agent reads one clean document?
