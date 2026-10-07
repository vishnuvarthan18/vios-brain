---
title: vios/LOG
type: log
zone: ai
project: vios
date: 2026-09-30
---
## For future agent
Append-only session log for [[vios]]. Newest at the bottom.

# LOG — vios

## 2026-09-30 · claude-opus (research session)
- Did: deep research (~90 sources) on personal AI OS, second brains, portable memory, multi-agent hubs, safety.
- Decided with Vishnu: name viOS; fully open source; Option A (open OS, any AI can plug in); Logseq as viewer; 3 zones.

## 2026-09-30 · claude-opus (overnight build)
- Did: built viOS v0.1 in ~/own/viOS from proven open-source patterns (see docs/SOURCES.md).
- Next: Vishnu runs setup/install-mac.sh and fills core/ files.

## 2026-10-01 · claude-opus (v1.0 build)
- Did: Vishnu said v0.1 was ~5% of the goal; wants a full OS reachable from anywhere, all AIs/APIs, like ourlifeos.ai (= LifeOS).
- Built v1.0: fork LifeOS as engine + cloud server (Caddy, Authelia 2FA/OIDC, LiteLLM, LibreChat, SilverBullet, Backlog board, vios-api REST+MCP+PWA, Hermes Telegram/WhatsApp, restic backups) + Mac installer.
- Verified by: component test suites + Hermes→vios-api end-to-end (daily note created + committed). Containers not run (registries blocked in build sandbox).
- Next: deploy on a real VPS.
