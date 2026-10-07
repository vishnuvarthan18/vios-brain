---
title: vios/DECISIONS
type: decisions
zone: ai
project: vios
date: 2026-09-30
---
## For future agent
Decision record for [[vios]]. Newest at the bottom.

# DECISIONS — vios

## D1 · 2026-09-30 · Plain Markdown + Git is the brain
- Why: works with every AI; Anthropic/OpenAI/Letta results show files+git beat databases for handoff; apps die (Reor archived 2026).

## D2 · 2026-09-30 · Fully open source, Option A
- Decision: every viOS part is open source. Commercial AIs (Claude, ChatGPT) may plug in, but are never required.
- Why: Vishnu's choice. Standards (MCP, AGENTS.md, SKILL.md) are open.

## D3 · 2026-09-30 · Logseq instead of Obsidian
- Why: Obsidian is free but not open source. Logseq (AGPL) gives graph + journals on the same files. Use file-based graph mode.

## D4 · 2026-09-30 · Three zones
- Core = AI read-only. AI = read/write. Private = outside folder, encrypted, no AI.
- Why: locks must live in the system (folders, permissions, encryption), not in chat rules. (Lesson: OpenClaw email deletion incident, Feb 2026.)

## D5 · 2026-09-30 · Project state lives in viOS
- STATE/LOG/DECISIONS in `ai/projects/<name>/`. Code repos get a small AGENTS.md block pointing here.
- Why: one place for every AI to resume from (pattern from the nrjn "How I Work" gist).

## D6 · 2026-09-30 · Tools chosen
- Backlog.md (tasks, MIT), Basic Memory (MCP for chat apps, AGPL), qmd (semantic search, MIT), OpenCode/goose (agent runners), Ollama (local models), launchd (schedules). Task Master and n8n rejected (not OSI open source).

## D7 · 2026-10-01 · Fork LifeOS as the engine
- Decision: LifeOS (MIT, ourlifeos.ai) runs on the Mac as the AI harness; viOS overlay wires it to the vault (core/ stays the source of truth).
- Why: proven, 19k stars; Vishnu's reference product.

## D8 · 2026-10-01 · Small cloud server + one AI gateway
- Decision: Ubuntu VPS (4 GB) with Docker; LiteLLM as the only door to AI models; access via web/PWA, Telegram/WhatsApp (Hermes), remote MCP (OAuth via Authelia), terminal agents.
- Why: reachable from anywhere; one key store + cost tracking; all parts open source.

## D9 · 2026-10-01 · Private zone stays Mac-only
- The server never holds private data. Hermes gets vault access only via vios-api MCP (zones enforced), not the filesystem.
