# DOCS

The written record around the project. Read in this order before writing code.

**Before touching any real server — reading, deploying, or just telling
someone an address is live — read
[server-migration-plan.md](server-migration-plan.md)'s status table first.**
The production server moved on 30 September 2026; several older docs
(`session-handover.md`, `../deploy/runbook.md`) still name the retired
server and have not been corrected throughout, only flagged at the top.
Trust the migration plan's table over any address written anywhere else in
this repo, including this index.

| Doc | What it is |
| --- | --- |
| [server-migration-plan.md](server-migration-plan.md) | **Where the real server is, right now.** Check this before any deploy |
| [build-plan.md](build-plan.md) | Scope, stack, data model, milestones, risks. The plan of record |
| [agent-rules.md](agent-rules.md) | The hard rules for whoever writes the code, human or AI. Read every session |
| [build-spec.md](build-spec.md) | The full specification — schema, API, widget states, copy. **Not yet written** |
| [web/](web/) | Depth on the Next.js app |
| [widget/](widget/) | Depth on the embeddable script |

Reference material from third parties (Webflow limits, competitor
documentation) goes in `ref/<vendor>/`.
