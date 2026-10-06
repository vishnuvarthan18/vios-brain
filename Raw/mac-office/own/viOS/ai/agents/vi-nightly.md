---
type: agent
zone: ai
date: 2026-09-30
name: vi-nightly
status: ready
runner: hermes (server cron) — Mac launchd copy stays OFF to avoid double runs
model: any
schedule: "30 23 * * *"
projects: [vios]
tools_allowed: [read, write:ai/, qmd, python-tools]
tools_denied: [delete, network-send, core-write, private]
tags: [agent]
---
## For future agent
Scheduled nightly maintenance agent. Runs doctor + lint, refreshes dashboard and search index. Runs on the viOS server via Hermes cron; summary sent to Telegram.

# Agent — vi-nightly

## Job
- `python3 tools/doctor.py` → fix simple issues (missing frontmatter, broken links → stubs).
- vi-lint skill on `ai/knowledge/wiki/`.
- `python3 tools/dashboard.py`, `qmd update && qmd embed`.
- Log to `ai/logs/runs/YYYY-MM-DD-nightly.md`. Commit.

## Limits (Rule of Two)
- Untrusted input: no · Private data: no · Can send out: no
