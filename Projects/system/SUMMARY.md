---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: System — Mac setup and cleanup

## 1. What this project is
- **Goal:** Keep Vishnu's Macs clean and fast: folder cleanup of `~/araCreate`, office/personal separation, stopping stray dev servers, fixing slowness.
- **Who it is for:** [[People/Vishnu]].
- **Note:** First closed as "done" after the 2026-08-20 folder cleanup; it is now ongoing upkeep (work on 09-03, 10-03, 10-05). Could become an Area later (not decided).

## 2. Status now (as of 2026-10-05)
- Office Mac: no local dev servers running (all killed 2026-10-05). Office Mac is for official work only (since 2026-07-14).
- Personal Mac (2026-10-03): slow — only 8 GB RAM and ~10 GB swap in use. Causes: too many processes (VS Code with 26 helpers incl. 6 idle Claude Code sessions, Chrome with 33 helpers, Claude desktop app), plus iCloud Drive sync and Spotlight reindexing. Advice given; not sure if Vishnu applied it.
- `~/araCreate` cleanup (2026-08-20): 886 MB moved into `araCreate/_to_delete/` (4 node_modules, 3 dist folders, 37 .DS_Store, 3 empty dirs). Folder now 2.2 GB. All projects keep package.json and src; git clean in 4 repos. `_to_delete/RESTORE-NOTES.md` has rebuild commands.

## 3. Next steps
1. Personal Mac: quit VS Code and old Claude Code sessions, close Chrome tabs, restart; disable login agents (Adobe ccxprocess / GC Invoker, Canva availability check, Amazon Music) — commands given, Vishnu to run.
2. Vishnu: drag `_to_delete/` to Trash.
3. Remove 463 MB duplicate `FST/the regen room/room images/OneDrive_1_8-9-2025 2` (byte-identical images).
4. Add `.claude/` to `.gitignore` in `SLK/www.sinolink.de`.
5. Move the `~/.claude` backup zip from the office Mac to the personal Mac ("later").
6. Install GitHub CLI (`gh`) if PRs from Claude are wanted.

## 4. Decisions
- 2026-07-14 — Office Mac for official work only; personal projects moved off via zip. #decision
- 2026-07-14 — Claude does not run permanent deletes; Vishnu runs them. #decision
- 2026-08-20 — Move, don't delete: files go to `_to_delete/` because the Mac bridge can't delete. #decision

## 5. Timeline
- 2026-07-14 — Office Mac set for official work only: `~/.claude` zipped to Desktop (176 MB); own-bala and MR Decors memory removed. "Ship first PR" starter: `gh` not installed.
- 2026-08-20 — Scan found ~1.4 GB reclaimable of 3.0 GB; 886 MB moved.
- 2026-09-03 — Stopped 6 local dev servers (arm-website and previews) and 8 more apps/services incl. Adobe and Google Drive.
- 2026-10-03 — Personal Mac slow: 8 GB RAM, ~10 GB swap; advice given.
- 2026-10-05 — Killed local dev runs on the office Mac (Next.js :3000, arm-app-timer, halle-feedback-widget). Postgres and apps left alone.

## 6. Key facts
- **Folders in ~/araCreate:** FST, NOA, TARA, SLK, AC, ISO, DSA, HLE, ARA-VCET, arm-ui, arm-website (later ARM reorganised — see [[Projects/timer/SUMMARY]]).
- **Backup:** `claude-full-backup-20260714.zip` on the office Mac Desktop.
- **Dev history:** [[Projects/system/DEV-LOG]] (Claude Code sessions). Chats index: [[Projects/system/chats/INDEX]].
- **Related:** [[Projects/the-regen-room/SUMMARY]] (room images), [[Projects/aracreate/SUMMARY]], [[Projects/feedback-widget/SUMMARY]].

## 7. Files and documents
- `~/araCreate/_to_delete/RESTORE-NOTES.md`

## 8. Open questions and problems
- Was `_to_delete/` emptied?
- Was the 463 MB OneDrive duplicate removed?
- Did the personal Mac speed-up steps work?
- Keep as a project or turn into an Area ("mac-setup")?

## 9. All chats in this project
- [[Projects/system/chats/2026-08-20 Folder cleanup|Folder cleanup]] — 2026-08-20
- Claude Code sessions: see [[Projects/system/DEV-LOG]].
