---
type: agent
zone: ai
date: 2026-09-30
name: vi-morning
status: ready
runner: hermes (server cron) — Mac launchd copy stays OFF to avoid double runs
model: any
schedule: "45 7 * * *"
projects: [vios]
tools_allowed: [read, write:ai/daily, write:ai/DASHBOARD.md, backlog-read]
tools_denied: [delete, network-send, core-write, private]
tags: [agent]
---
## For future agent
Scheduled morning agent. Creates today's daily note with top tasks. Runs on the viOS server via Hermes cron; result is sent to Telegram.

# Agent — vi-morning

## Job
- Run the vi-today skill: create `ai/daily/YYYY-MM-DD.md`, list top 3 tasks, overdue items, and blockers from all STATE.md files.

## Limits (Rule of Two)
- Untrusted input: no · Private data: no · Can send out: no
