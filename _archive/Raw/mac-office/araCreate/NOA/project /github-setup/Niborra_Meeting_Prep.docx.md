---
source: office Mac ~/araCreate/NOA/project /github-setup/Niborra_Meeting_Prep.docx
---

Niborra — Client Meeting Prep
Questions to ask · project status · what happens next
20 July 2026 · araCreate ↔ Niborra (Ahmad Taleb)
Where the project stands
11 epics · 15 user stories · 100 technical tasks · 306 story points · 10 sprints over 20 weeks (13 Jul – 27 Nov 2026), landing ahead of the February 2027 pilot shipment. Design and development run in parallel rather than back to back, which reclaimed roughly 6 weeks against the original 24-week plan.
Everything is tracked in GitHub: each story and task is an issue, labelled by epic, sprint, owner, and priority, on a board that moves Backlog → Ready → In Progress → In Review → Done. Sprint milestones have real due dates, so progress is visible without anyone writing a status report.
Two review passes since the PRD landed
The backlog was checked against the formal PRD twice, and 13 tasks were added as a result (G01–G13): Linux support, firmware OTA push plus failure/rollback handling, consumables prompts, data encryption, ISO 27001 access review, trial expiry handling, the Pro analytics dashboard, pilot account provisioning, a dedicated humanization engine, and earlier hardware + cross-platform testing. Scope grew 253 → 306 points. That's real work found before kickoff rather than discovered mid-build.
Part 1 — Ask in this meeting
1. Capacity — the big one
Kishor is our only developer and he's scheduled at 23–28 story points per sprint for six sprints straight (Sprints 3–8). His demonstrated solo pace is 12–14. This plan works only if nothing goes wrong.
ASK: Which of these do you want: (a) extend the timeline — we have ~8 weeks of real buffer before the Feb 2027 pilot shipment, (b) cut scope to Phase 2 — the Pro analytics dashboard is the obvious first candidate, or (c) add a second developer for the Sprint 4–8 window?
2. E5 Automations — confirm the deferral
The Shopify auto-trigger feature depends on the REST API/webhooks, which your PRD states are post-launch, not v1. So it cannot ship at launch.
ASK: Confirming you're happy with that. Also: the Pro tier pricing table says "REST API / Webhooks: Yes" — that needs a "coming post-launch" caveat until it actually ships, or it's a promise we can't keep on day one.
3. Card format spec file
The PRD references niborra_format_matrix.xlsx for exact card dimensions, but the file was never attached.
ASK: Can you send it? We need the precise measurements to finish the card designer specs (E2).
4. Handwriting: capture vs. style library
Our story E1.2 has the user writing out their own alphabet once so the machine copies their real handwriting. But your pricing table says "5 handwriting styles / Full library" by tier, which reads more like picking from presets.
ASK: Are these the same feature or two different ones? We're proceeding with live capture for v1 — just confirming that's what you meant.
Part 2 — Ask later, when we reach that work
These do not need answers today. Raise each one shortly before the sprint that depends on it — flagged here so nothing gets forgotten.

Part 3 — Already decided (no discussion needed)
Stated here so everyone is working from the same assumptions.

Part 4 — After the meeting
Once answers come back, these get updated so GitHub stays the single source of truth:
Capacity decision → if scope is cut, move those issues to the "Phase 2 (Post-Launch)" milestone. If the timeline extends, shift sprint milestone due dates.
E5 sign-off → close the question; the issue already sits in Phase 2.
Format matrix file → finalise E2 acceptance criteria and update those issues.
Handwriting confirmation → either keep E1.2 as-is, or rewrite the story and its acceptance criteria to match a preset style library.
Any answered "ask later" item → remove the flag:needs-decision label from that issue and fill in the spec.
Rule of thumb: decisions get recorded on the GitHub issue they affect, not in chat or email — that way the reason behind a change is still there in six weeks.