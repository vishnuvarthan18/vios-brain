---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: Tech to Me (personal tech learning and AI tools)

## 1. What this project is
- **Goal:** Learn tech properly and use AI tools well. Long plan: software, electronics, mechanical, tech/product strategy, plus sales and "engineering thinking".
- **Who it is for / client:** Vishnu himself (personal).
- **Why it exists:** Vishnu has no engineering degree and builds by "vibe coding". He wants to understand software, hardware and physical systems, because he thinks future AI products need people who know the full stack.

## 2. Status now (as of 2026-10-06)
- Learning plan made on 2026-09-01 (9 docs): diagnostic, roadmap and progress log for each domain.
- Software Phase 1 (variables, conditionals, loops, functions in JavaScript) was set to start 2026-09-02. No progress logged in the docs yet.
- GitHub groundwork done (2026-08-23): `conventions` repo (structure, headers, naming, commit format `<type>: <summary>`), `project-template` scaffold, profile README repo `vishnuvarthan18`. Audit of 15 repos (2026-08-24): most "large gap", `wedding2day-app` "worth fixing".
- Daily 7 AM IST morning briefing scheduled in Claude (calendar, urgent email, Slack mentions, AI/product/SaaS news, under 400 words).
- Claude setup work done: Claude Pro custom instructions, Claude Code install fixed (npm prefix), Webflow MCP connected in Claude Code, office and personal Claude accounts split.
- Started with a NodeMCU ESP8266 board on 2026-09-17. Setup not confirmed as done.
- Not done: Electronics, mechanical, strategy and sales tracks have not started.

## 3. Next steps
1. Do Software Phase 1 and log each session (hours, what was done, confidence 1-5, what was hard).
2. Finish NodeMCU ESP8266 setup on the Mac and run the blink test.
3. First speed checkpoint 4-6 weeks after start (about mid-October 2026).

## 4. Decisions
- 2026-06-03 — Use a layered set of custom instructions for Claude (core, reasoning mode, coding, project context) — to save usage credits. #decision
- 2026-06-09 — Connect Webflow MCP in Claude Code with stdio and an API token, not OAuth — OAuth kept failing on site scope. #decision
- 2026-06-21 — Fix npm global install by moving the npm prefix to a user folder, not sudo — avoids root-owned files. #decision
- 2026-07-14 — Split Claude into a personal account and a new office Claude Pro account; Wedding2day stays personal — keep office and personal work apart. #decision
- 2026-09-01 — Priority order: software, then electronics, then mechanical, then strategy; sales runs in parallel; 7-10 hrs/week — one focus at a time. #decision
- 2026-09-01 — Software goal is to read, debug and direct AI code, not to become a job-ready developer. #decision
- 2026-08-24 — GitHub: only fix projects still in use; focus on the commit format going forward. #decision

## 5. Timeline
- 2025-05-12 — Asked about free n8n hosting and which tech to learn for SaaS, embedded and mechanical.
- 2025-05-28 — Asked about font sizes for a 430px mobile screen.
- 2026-06-03 — New Claude Pro user; built custom instructions to save credits.
- 2026-06-09 — Connected a second Webflow site to Claude Code (Webflow MCP); many errors, switched to stdio.
- 2026-06-10 — Learned one Webflow token covers all sites in one workspace. Got an Arduino / embedded learning roadmap.
- 2026-06-11 — Learned to open a local VS Code Live Server site on the phone.
- 2026-06-12 — Asked which AI built torrix.ai (no way to tell).
- 2026-06-15 — Asked about Claude models and uses.
- 2026-06-21 — Fixed npm permission error when installing Claude Code on Mac.
- 2026-06-29 — Tried Headroom (token compression tool); learned it only works for terminal / VS Code Claude Code, not the app.
- 2026-07-14 — Moved office context to a new office Claude account.
- 2026-07-22 — Mac cleanup: ~140 GB of After Effects cache found and moved (not deleted).
- 2026-07-24 — Built an "Atelier" hero section in React + Tailwind v4 as a test; learned how to run it locally.
- 2026-08-23 — GitHub `conventions`, `project-template` and profile README repos made.
- 2026-08-24 — Read-only audit of 15 GitHub repos against the conventions.
- 2026-09-01 — Full multi-year learning plan and tracking system written (9 docs).
- 2026-09-17 — Started with NodeMCU ESP8266 V3 (CH340) board.
- 2026-10-06 — Mac scan listed all local git repos (india-platform engines, semmozhi `tamil-data-collector`, W2D, Vidivu, Kuzhali, Nalaas, BlastDesk).

## 6. Key facts
- **People:** [[People/Vishnu]] — learner
- **Tools:** [[Tools/Claude]], [[Tools/Claude Code]], [[Tools/Claude in Chrome]], [[Tools/VS Code]], [[Tools/n8n]], [[Tools/React]], [[Tools/Tailwind CSS]], [[Tools/Vite]], Webflow MCP, Headroom, Arduino IDE, NodeMCU ESP8266, KiCad and Fusion 360 (planned)
- **Links / repos / servers / file paths:** Headroom: github.com/headroomlabs-ai/headroom. Webflow MCP package: `webflow-mcp-server` (stdio). Webflow token (secret, not saved) — it was pasted in a chat, so it should be rotated. GitHub: `conventions`, `project-template`, `vishnuvarthan18` (profile README).
- **Dev history:** none here. The 2 Claude Code sessions once filed here (2026-07-29, folder `own/demo-projects/ai-siite-2`) were The Regen Room "Perimenopause Reset Programme" page and were moved to [[Projects/the-regen-room/DEV-LOG]]. Stub: [[Projects/tech-to-me/DEV-LOG]].
- **Chats index:** INDEX (archived: Projects/tech-to-me/chats/INDEX.md)
- **Related:** [[Projects/aracreate-academy/SUMMARY]] (NodeMCU ideas for the kids' academy), [[Projects/career/SUMMARY]], [[Projects/wedding2day-app/SUMMARY]]

## 7. Files and documents
- `claude-roadmap.md` — main roadmap, priority order, status (docs/)
- `claude-full-plan-timeline.md` — 3-5 year timeline with speed checkpoints (docs/)
- `claude-tracking-system.md` — how to log sessions, weekly rollups, checkpoints (docs/)
- `claude-software-engineering.md` — software diagnostic and phases (docs/)
- `claude-electronics-ee.md` — electronics diagnostic and phases up to PCB design (docs/)
- `claude-mechanical-engineering.md` — mechanical diagnostic and phases (docs/)
- `claude-tech-product-strategy.md` — strategy diagnostic and phases (docs/)
- `claude-sales.md` — sales diagnostic and phases (docs/)
- `claude-engineering-thinking.md` — core mental models used in every domain (docs/)

## 8. Open questions and problems
- Was the Webflow API token that was pasted in chat rotated? (not sure)
- Did the NodeMCU blink test work?
- Has Software Phase 1 really started? No sessions are logged.

## 9. All chats in this project
- Free Hosting Options for n8n Automation (archived: Projects/tech-to-me/chats/2025-05-12 Free Hosting Options for n8n Automation.md) — 2025-05-12
- Optimal Font Sizes for 430px Mobile Screens (archived: Projects/tech-to-me/chats/2025-05-28 Optimal Font Sizes for 430px Mobile Screens.md) — 2025-05-28
- Maximizing Claude Pro credits and usage (archived: Projects/tech-to-me/chats/2026-06-03 Maximizing Claude Pro credits and usage.md) — 2026-06-03
- Connecting Claude to multiple websites in Webflow (archived: Projects/tech-to-me/chats/2026-06-09 Connecting Claude to multiple websites in Webflow.md) — 2026-06-09
- Connecting multiple Webflow websites to Claude (archived: Projects/tech-to-me/chats/2026-06-10 Connecting multiple Webflow websites to Claude.md) — 2026-06-10
- Getting started with Arduino embedded systems (archived: Projects/tech-to-me/chats/2026-06-10 Getting started with Arduino embedded systems.md) — 2026-06-10
- Viewing local site in VS Code (archived: Projects/tech-to-me/chats/2026-06-11 Viewing local site in VS Code.md) — 2026-06-11
- AI website builder identification (archived: Projects/tech-to-me/chats/2026-06-12 AI website builder identification.md) — 2026-06-12
- Claude modules and their uses (archived: Projects/tech-to-me/chats/2026-06-15 Claude modules and their uses.md) — 2026-06-15
- Resolving npm permission denied errors (archived: Projects/tech-to-me/chats/2026-06-21 Resolving npm permission denied errors.md) — 2026-06-21
- Installing Headroom token reduction tool (archived: Projects/tech-to-me/chats/2026-06-29 Installing Headroom token reduction tool.md) — 2026-06-29
- Usage credits not working (archived: Projects/tech-to-me/chats/2026-06-29 Usage credits not working.md) — 2026-06-29
- Disconnecting office account from personal Claude (archived: Projects/tech-to-me/chats/2026-07-14 Disconnecting office account from personal Claude.md) — 2026-07-14
- Frontend design skill review (archived: Projects/tech-to-me/chats/2026-07-24 Frontend design skill review.md) — 2026-07-24
- NodeMcu ESP8266 V3 WiFi Dev Board setup (archived: Projects/tech-to-me/chats/2026-09-17 NodeMcu ESP8266 V3 WiFi Dev Board setup.md) — 2026-09-17
