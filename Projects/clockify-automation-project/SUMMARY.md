---
tags: project
status: active
owner: "[[People/Vishnu]]"
---
# PROJECT: Clockify Automation (Calendar → Clockify)

## 1. What this project is
- **Goal:** Turn [[Tools/Google Calendar]] events into [[Tools/Clockify]] time entries automatically.
- **Who it is for:** [[People/Vishnu]] / araCreate time tracking.
- **How:** Google Calendar → [[Tools/Cloudflare]] Worker (`calendar-clockify.vishnu-317.workers.dev`) → Clockify workspace. Cron polling, not webhooks.

## 2. Status now (as of 2026-08-18)
- Works. End-to-end test passed 2026-08-02.
- All 18 projects + 8 tags mapped.
- Project had 0 saved docs — everything was only in the project description.

## 3. Next steps
1. Change cron day Sunday → Monday (1-line change).
2. Confirm the Clockify API key was rotated (it was shown in a chat on 2026-08-02).
3. Optional: Workers Analytics for cron monitoring.
4. Write a proper project doc.

## 4. Decisions
- (before 2026-08-18) — Event title convention `#project #tag description`: first hashtag = project, second = tag, exact match only; falls back to event description, else blank. Clockify description = clean event title. Phase/task never auto-set. #decision
- (before 2026-08-18) — Cron polling instead of webhooks. #decision

## 5. Timeline
- 2026-08-02 — End-to-end test passed.
- 2026-08-18 — Context check chat: 3 items still open.

## 6. Key facts
- **Worker:** `calendar-clockify.vishnu-317.workers.dev` ([[Tools/Cloudflare]]).
- **Clockify workspace ID:** `5db9c8d5bb56233f9550fbb0`.
- **Related:** [[Projects/clockify/SUMMARY]], [[Projects/clockify/SUMMARY]]. Also araMetrics has `arm-util-clockify` (Python script doing calendar → Clockify) — see [[Projects/timer/SUMMARY]].

## 7. Files and documents
- None saved. Worker source code not seen in chat.

## 8. Open questions and problems
- API key rotation not confirmed.
- Where is the Worker source code (repo)?

## 9. All chats in this project
- Context verification (archived: Projects/clockify-automation-project/chats/2026-08-18 Context verification.md) — 2026-08-18
