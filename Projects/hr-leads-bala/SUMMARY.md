---
tags: project
status: paused
owner: "[[People/Vishnu]]"
---
# PROJECT: WFH HR and Lead Tracking App (hr-leads-bala)

## 1. What this project is
- **Goal:** A light PWA for a work-from-home team: attendance with location check (clock in/out, breaks), leave requests with approval, and manual lead entry for sales agents.
- **Who it is for / client:** A BPO client that sells Adobe and other software at low prices (client "Bala", not sure).
- **Why it exists:** The client asked Vishnu for an HR app. Agents get leads on WhatsApp and need one place to log them.

## 2. Status now (as of 2026-07-08)
- Planning done. Light scope agreed. Build not started (not sure).
- Plan: 10-14 working days (full scope was 22-34 days).
- WhatsApp message for the client drafted: running costs and list of info needed.

## 3. Next steps
1. Get info from the client: employee home addresses, shift and leave rules, product list and prices, written consent for location tracking.
2. Build v1 with AI tools (vibe coding).
3. Phase 2 (upsell): auto leads from Google and Meta ads.

## 4. Decisions
- 2026-07-08 — Stack: Supabase (Postgres, Auth, RLS, Edge Functions) + React Vite PWA on Vercel or Netlify. #decision
- 2026-07-08 — Build fully by vibe coding (Claude Code / Cursor / bolt.new) — Vishnu is a beginner. #decision
- 2026-07-08 — Employees and sales agents are the same people. #decision
- 2026-07-08 — Leads entered by hand from WhatsApp in v1; auto lead fetch from ads moved to phase 2 — ads are harder. #decision
- 2026-07-08 — Keep a duplicate phone number check on leads — stops agents fighting over leads. #decision
- 2026-07-08 — Light scope: cut leave balance tracking, offline queue, CSV/reports, price versions, kanban; keep a "no internet, retry" message. #decision

## 5. Timeline
- 2026-07-08 — Client asked for an HR app; full plan, cost and client message made; then cut to a light version.

## 6. Key facts
- **People:** [[People/Vishnu]] — developer
- **Tools:** [[Tools/React]], [[Tools/Vite]], [[Tools/Vercel]], [[Tools/Claude Code]], [[Tools/Cursor]], [[Tools/WhatsApp]], Supabase
- **Costs:** free in pilot; about ₹2,500/month later (estimate).
- **Related:** [[Projects/web-bala/SUMMARY]] (same client, not sure)

## 7. Files and documents
- Full project plan was saved as a markdown artifact in the chat (not in this folder).

## 8. Open questions and problems
- Did the client send the needed info and agree to the cost?
- Without offline queue, staff on bad home internet may lose clock-ins.
- Leave balance and price versions may be needed if the team is big or prices change often.

## 9. All chats in this project
- Work-from-home HR and lead tracking PWA (archived: Projects/hr-leads-bala/chats/2026-07-08 Work-from-home HR and lead tracking PWA.md) — 2026-07-08
