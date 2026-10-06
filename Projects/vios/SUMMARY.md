---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: viOS (Vishnu's personal operating system / "brain")

## 1. What this project is
- **Goal:** One "brain" that holds all of Vishnu's projects, AI agents and life. Any AI (any model, any device) connects to it, reads the context and continues the work.
- **Who it is for / client:** Vishnu himself (personal). It also holds his office ([[Companies/araCreate Group]]) projects.
- **Why it exists:**
  - Each new AI chat starts from zero; Vishnu wants all context saved in HIS system, not on an AI company's server.
  - Many projects and many AI agents; work should continue across tools without copy-paste.
  - Must be fully open source (no commercial tools inside).
  - Must have a part AI handles and a private part no AI ever touches.

## 2. Status now (as of 2026-10-06)
- **Current setup (v2, simple VPS build) is live:**
  - Brain = plain markdown notes folder with git history, on Vishnu's VPS at [[Companies/OVHcloud]].
  - Notes app = [[Tools/SilverBullet]] at https://notes.40-160-137-239.sslip.io (login protected).
  - Own MCP server "AI door" (list, read, search, write, append, move notes; cannot delete, cannot leave the brain folder). Link is secret (secret, not saved).
  - [[Tools/Claude]] desktop app connected to the AI door on both Macs (Mac 1 MacBook Air, Mac 2).
  - `RULES.md` in the brain tells every AI how to read/write.
  - Auto-save (git commit) every 5 minutes on the VPS.
  - Mesh / map: SilverBullet "Object Graph" works; project pages link to People / Companies / Tools pages.
  - Projects imported from Claude: 12 personal-account projects done on 2026-10-02/03; office chats imported 2026-10-04/05.
  - 2026-10-06: personal Claude export (chats + project docs), Claude Code sessions from both Macs (DEV-LOG pages), personal Mac files and Google export merged into project pages; brain clean-up run (pages merged and rewritten).
  - 2026-10-06: data exports requested — LinkedIn full archive, Google Takeout office account (62 products, some disabled by Workspace admin), Google Takeout personal account (64 products). Waiting for emails. About me updated.
- **Old Mac setup (v1) is stopped, to be deleted later (plan step 10):**
  - Code in `~/own/viOS-system`, old vault in `~/own/viOS`, packages `viOS-v1.1.tar.gz`, `viOS-v1.2.tar.gz`.
  - It was a 9-service [[Tools/Docker]] stack (via [[Tools/Colima]]): dashboard, [[Tools/LibreChat]], SilverBullet, Backlog.md task board, [[Tools/LiteLLM]] AI gateway, MCP server, vios-api, plus a LifeOS fork and 12 vi-* skills.
  - It worked on the Mac on 2026-10-01, but chat never answered (no AI API key), and Vishnu found it far too complex.
- **Not done yet:**
  - Password safe ([[Tools/Vaultwarden]] on VPS): script made, not confirmed run.
  - No backup: the brain lives only on the VPS.
  - Claude full-data export, GitHub cleanup, brain cleanup, home dashboard.
  - Other AIs (ChatGPT, Gemini, Cursor) not connected yet. App connectors not set up.
  - Old Mac memory (vios, Halle pages) not moved yet (not sure).

## 3. Next steps
1. Finish the brain clean-up (open questions in `Inbox/Questions for Vishnu.md`).
2. Bring in the LinkedIn and Google Takeout exports when the emails arrive.
3. Personal GitHub cleanup (not the office GitHub); link each repo to its project page.
4. Clean the brain: merge duplicates, archive empty "save chat" notes, answer `Inbox/Questions for Vishnu.md`, build a home dashboard.
5. Backup of the brain (daily backup to Mac and/or private GitHub repo `vios-brain`).
6. Password safe (Vaultwarden): create account, lock sign-up, store notes password + AI door link.
7. Connect other AIs (ChatGPT, Gemini, Cursor) and 2-3 app connectors.
8. Move old Mac memory (vios, Halle) into the new brain.
9. Delete the old Mac setup (`viOS-system`, tar.gz files, Logseq etc.) — plan step 10.
10. Link every project automatically (update global rule `~/.claude/CLAUDE.md` to save to the VPS brain); set up PM → dev agent flow through viOS.
- Later: more personal data (LinkedIn, ChatGPT, Google, social media), AI Worker, Telegram, team / family SilverBullet spaces.

## 4. Decisions
- 2026-10-06 — Office and client data may stay in the brain (Vishnu's decision). #decision
- 2026-08-21 — Obsidian ruled out (licence/cost); test: "can an agent read and write notes with no app running?"; SilverBullet as optional browser front end (now used). #decision
- 2026-09-30 — viOS = plain markdown + Git as the one source of truth; any AI connects via MCP + AGENTS.md — files outlast apps, no lock-in. #decision
- 2026-09-30 — Don't build from scratch; reuse proven open-source parts and only write glue. #decision
- 2026-09-30 — Fully open source, no commercial tools; Obsidian dropped (not open source, and Vishnu finds it too complex). #decision
- 2026-09-30 — Separate zones: AI zone, read-only core, and a Private zone no AI can touch, locked by the system not by a chat rule — safety. #decision
- 2026-09-30 — Claude builds overnight on the Mac (v0.1, then v1.0 full stack after Vishnu said v0.1 was "only 5%") — Vishnu wanted a full OS like ourlifeos.ai. #decision
- 2026-09-30 — Run v1 locally on the Mac (no server/domain) — Vishnu's choice. #decision
- 2026-10-01 — Keep code (`~/own/viOS-system`) and vault (`~/own/viOS`) in separate folders — Mac does not tell "vios" and "viOS" apart. #decision
- 2026-10-02 — Old Mac build too complex; make it simple — a second brain needs only notes + AI + private safety. #decision
- 2026-10-02 — Move the brain to Vishnu's existing VPS ([[Companies/OVHcloud]]) — reach it from any device and any AI. #decision
- 2026-10-02 — Notes app = SilverBullet (not Obsidian, not Flatnotes) — open source, browser-based, plain files, powerful enough; AI does the hard work. #decision
- 2026-10-02 — AI Worker (scheduled agent) and Telegram paused for now. #decision
- 2026-10-02 — Same viOS project, new simple build; build first, delete the old setup last — no data loss. #decision
- 2026-10-02 — VPS setup uses the existing [[Tools/nginx]] and new ports (3100, 8100); does not touch other apps or the firewall — server already runs other apps. #decision
- 2026-10-02 — Built own simple MCP "AI door" — the ready-made one used an old schema Claude could not read. #decision
- 2026-10-02 — Connect Claude through the local desktop config on each Mac — team plan (araCreate Group) lets only the owner add custom connectors. #decision
- 2026-10-02 — Password safe on the VPS = Vaultwarden (KeePassXC/Cryptomator are Mac-only) — one main password; AI never reaches it. #decision
- 2026-10-02 — All projects in one `Projects/` folder, no Personal/Office split — Vishnu's choice. #decision
- 2026-10-02 — Backup planned but later; focus first on bringing in Claude projects and personal GitHub cleanup (not office GitHub). #decision
- 2026-10-02 — Import Claude projects with one prompt per project (SUMMARY/STATE/LOG + links + tags), full export later — export loses the chat-to-project mapping. #decision
- 2026-10-02 — Every project links to viOS automatically, but only after the whole setup is done. #decision

## 5. Timeline
- 2026-08 — Earlier "V OS" / VOS-OSS design docs written on the personal Mac; research on 21 Aug.
- 2026-09-30 — Idea and deep research (second brain, MCP, AGENTS.md, ~90 sources); 3-zone design.
- 2026-09-30 — Overnight build: v0.1 (98 files, 3 zones, 12 skills), then v1.0 full stack (LifeOS fork, phone web app, LibreChat, SilverBullet, LiteLLM, Telegram bot, 250+ checks).
- 2026-09-30 — Switched to local Mac run (v1.1, then v1.2 with a file-name case bug fix).
- 2026-10-01 — Installed on Mac with Colima; fixed vios-api data-folder permission bug; all 9 services healthy; chat login made via sign-up; no AI key so chat could not answer.
- 2026-10-01 — State saved in old vault and claude.ai doc `viOS-status.md`; everything stopped; Mac cleaned (PostgreSQL, 3 open Python file servers, bootcamp-evaluation node server stopped).
- 2026-10-02 — Vishnu found the build too complex; new simple plan saved as `viOS-v2-plan.md`.
- 2026-10-02 — VPS setup on OVHcloud: brain folders, RULES, auto-save, SilverBullet, AI door, https. Script bug at auto-save step fixed and re-run.
- 2026-10-02 — Notes password and AI door link had been shown in chat, so both were changed.
- 2026-10-02 — New AI door built and deployed; Claude read/write tested OK.
- 2026-10-02 — Mac 2 connected to the brain.
- 2026-10-02 — First project imported (wedding2day-app) with 31 linked pages; mesh graph works.
- 2026-10-02/03 — 12 projects imported fully; 11 came out empty (prompt pasted into new chats); office account cannot search other chats.
- 2026-10-03 — Started reading office chats through Chrome; chat ends mid-work.
- 2026-10-04/05 — Office chats imported into the brain.
- 2026-10-06 — Personal Claude export, Claude Code sessions, Mac files and Google export merged; LinkedIn and Google Takeout exports requested; brain clean-up.

## 6. Key facts
- **People:** [[People/Vishnu]] — owner
- **Companies:** [[Companies/OVHcloud]] (VPS host, US branch), [[Companies/araCreate Group]] (office Claude team account)
- **Tools:** [[Tools/SilverBullet]], [[Tools/Claude]], [[Tools/Claude Code]], [[Tools/Claude in Chrome]], [[Tools/GitHub]], [[Tools/nginx]], [[Tools/Docker]], [[Tools/Vaultwarden]]; old setup: [[Tools/Colima]], [[Tools/LibreChat]], [[Tools/LiteLLM]], [[Tools/KeePassXC]], Cryptomator, Logseq, Backlog.md, Basic Memory, qmd, LifeOS, [[Tools/Telegram]]
- **Links / repos / servers / file paths:**
  - VPS: OVHcloud VPS-1, Oregon USA, about $6.31/month; IP 40.160.137.239 (vps-e8d92c83.vps.ovh.us); user `ubuntu`, root login off; also runs other apps behind nginx.
  - Notes app: https://notes.40-160-137-239.sslip.io (user `vishnu`; password secret, not saved)
  - AI door (MCP): on mcp.40-160-137-239.sslip.io (full link secret, not saved)
  - Password safe: https://vault.40-160-137-239.sslip.io (planned)
  - VPS paths: brain `~/vios/brain`, setup `~/vios` (secrets in `~/vios/.env`), scripts `~/setup-vps.sh`, `~/setup-vault.sh`.
  - Brain layout: About me, Goals, RULES, index, Projects/, Inbox/, Daily log/, Resources/, Archive/, People/, Companies/, Tools/.
  - Earlier design docs: "V OS" in `~/Downloads/VOS` (first blueprint) and `~/Downloads/VOS-OSS` (open-source version, Aug 2026): plain Markdown + git, `vos` CLI (`vos daily`, `vos jot`), folders 00-Inbox / 10-Journal / 20-Projects / 30-Areas / 40-Library / 90-System, Claude skills (inbox, meeting, weekly, decide); Android plan Markor + Syncthing.
  - Old Mac: code `~/own/viOS-system`, old vault `~/own/viOS`, `~/own/viOS-v2/`; global rule `~/.claude/CLAUDE.md` (vi-resume / vi-handoff).
  - Server 217.160.93.75 is Halle's live site — not for viOS.
- **Chats index:** [[Projects/vios/chats/INDEX]]
- **Related:** [[Projects/halle-web/SUMMARY]], [[Projects/wedding2day-app/SUMMARY]], [[Projects/system/SUMMARY]]

## 7. Files and documents
- `viOS-status.md` — status of the old Mac setup (claude.ai "viOS" project)
- `viOS-v2-plan.md` — the simple VPS plan and progress (claude.ai "viOS" project)
- `RULES.md` — rules every AI follows (brain on VPS)
- `Inbox/Questions for Vishnu.md` — open questions from project imports (brain)
- `Projects/vios/STATE.md` — viOS state written 2026-10-02 (brain)
- `core/RULES.md`, `ai/projects/vios/STATE.md`, `LOG.md`, `DECISIONS.md`, `ai/knowledge/wiki/vios-local-troubleshooting.md` — old vault on Mac (`~/own/viOS`)
- `setup-vps.sh`, `setup-vault.sh` — VPS setup scripts

## 8. Open questions and problems
- No backup yet: if the VPS dies, the brain is lost.
- Password safe not confirmed; where are the notes password and AI door link kept now?
- Possible duplicate projects: clockify / clockify-entry, nasa / nasa-space-apps-erode-2026, timer / arm-timer.
- Office Claude account blocks "search other chats" and custom connectors (owner only).
- Long saves through the AI door can time out (save in small parts).
- VPS needs a restart for a system update (also briefly stops other apps).
- An old LibreChat password was shown in the 2026-10-01 chat (secret, not saved) — matters only until the old setup is deleted.

## 9. All chats in this project
- [[Projects/vios/chats/2026-09-30 viOS personal operating system|viOS personal operating system]] — 2026-09-30
- [[Projects/vios/chats/2026-10-02 Status update|Status update]] — 2026-10-02
- [[Projects/vios/chats/2026-10-02 Moving project to viOS (4)|Moving project to viOS (4)]] — 2026-10-02
- [[Projects/vios/chats/2026-10-02 RULES.md from viOS brain|RULES.md from viOS brain]] — 2026-10-02
