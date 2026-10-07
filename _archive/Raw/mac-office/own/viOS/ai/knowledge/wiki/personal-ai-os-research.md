---
title: personal-ai-os-research
type: wiki
zone: ai
date: 2026-09-30
status: solid
sources: [research session 2026-09-30, ~90 web sources]
tags: [wiki, vios, research, ai-memory, second-brain]
---
## For future agent
Compiled findings from the 2026-09-30 deep research behind [[vios]]: which personal AI OS / second-brain systems work, how to make memory portable across AIs, how to run many agents, and what failed. Facts are dated; re-verify numbers older than ~6 months.

# Personal AI OS — research summary

## Proven systems
- **Karpathy LLM Wiki** — raw/ + wiki/ + schema; ingest, query, lint. 5k+ stars on gist (as of 2026-09, gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- **Daniel Miessler PAI / LifeOS** — TELOS goals, 67 skills, hooks; 19.2k stars (as of 2026-09, github.com/danielmiessler/Personal_AI_Infrastructure)
- **obsidian-second-brain** — 45+ commands, 8 AI tools, CRITICAL_FACTS.md (~120 tokens), scheduled agents; 4.6k stars (as of 2026-09, github.com/eugeniughelbur/obsidian-second-brain)
- **COG second brain** — people CRM, verifier agents, AGENTS.md fallback (github.com/huytieu/COG-second-brain)
- **qmd** — local hybrid search (BM25 + vectors + rerank), MCP; 30.1k stars (as of 2026-09, github.com/tobi/qmd)
- **Basic Memory** — markdown + graph + MCP for Claude/ChatGPT/Cursor; 3.6k stars, AGPL (as of 2026-09, github.com/basicmachines-co/basic-memory)
- **Backlog.md** — markdown task board + MCP (github.com/MrLesk/Backlog.md)
- **Teresa Torres** — task-per-file, /today, nightly research digests (chatprd.ai/how-i-ai/teresa-torres-claude-code-obsdian-task-management)

## Portability
- MCP: 97M monthly SDK downloads, 10k+ servers; under Linux Foundation AAIF (as of 2026-03)
- AGENTS.md: 60k+ repos; Claude Code needs `@AGENTS.md` in CLAUDE.md (as of 2026-09, agents.md)
- Agent Skills (SKILL.md): 40+ tools support it (as of 2026-09, agentskills.io)
- Context files add 20%+ cost; overviews don't help; exact commands do (as of 2026-02, arxiv.org/abs/2602.11988)
- Anthropic harness: feature list JSON + progress file + git + smoke test (anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- OpenAI PLANS.md: Progress / Surprises / Decision log / Outcomes → 7+ hour runs (developers.openai.com/cookbook/articles/codex_exec_plans)
- Letta filesystem agent 74% vs Mem0 graph 68.5% on LoCoMo (letta.com/blog/benchmarking-ai-agent-memory)
- Memory benchmark numbers are mostly vendor-reported; treat with care (memnode.dev/articles/agent-memory-benchmarks-2026-real-numbers)

## Many agents
- Best: 3–5 parallel agents per human; verification is the bottleneck (addyosmani.com/blog/code-agent-orchestra)
- One task = one context window (Backlog.md, Beads)
- Agent-as-folder registry (github.com/denberek/obsidian-agent-fleet)

## Failures / safety
- ⚠ OpenClaw: ~20% malicious marketplace skills; tens of thousands of exposed instances (as of 2026-02)
- ⚠ Agent deleted 200+ emails after "confirm first" rule was lost in compaction (as of 2026-02-23)
- ⚠ Lethal trifecta: private data + untrusted content + exfiltration (simonwillison.net/2025/Jun/16/the-lethal-trifecta)
- ⚠ AI graded healthy heart "F" from wearable data (as of 2026-01, Washington Post)
- ⚠ Building the system can replace real work (joanwestenberg.com/claude-code-wont-fix-your-life)

## Related
- [[vios]] · `docs/BLUEPRINT.md` · `docs/SOURCES.md`
