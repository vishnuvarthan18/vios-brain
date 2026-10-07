---
type: agent
zone: ai
date: 2026-09-30
name: vi-weekly
status: ready
runner: hermes (server cron) — Mac launchd copy stays OFF to avoid double runs
model: any
schedule: "0 18 * * 5"
projects: [vios]
tools_allowed: [read, write:ai/daily, write:ai/inbox]
tools_denied: [delete, network-send, core-write, private]
tags: [agent]
---
## For future agent
Scheduled weekly review agent. Drafts the weekly review for Vishnu to read. Runs on the viOS server via Hermes cron; summary sent to Telegram.

# Agent — vi-weekly

## Job
- Run vi-review: read last 7 daily notes, all STATE.md, inbox. Write `ai/daily/YYYY-Www-review.md` with wins, stuck items, suggested focus.
