# viOS Blueprint (v0.1 · 2026-09-30)

## Goal
- One personal OS for all projects, AI agents, knowledge and life.
- Any AI can plug in and continue work. Fully open source. Vishnu owns every file.

## Big ideas (proven in research)
1. **Files + Git are the brain.** Anthropic's long-running agents use progress files + git; OpenAI PLANS.md gave 7+ hour runs; Letta's file agent (74%) beat a graph memory (68.5%) on LoCoMo. Apps die (Reor archived 2026) — files don't.
2. **Open standards connect everything:** MCP (tools/data), AGENTS.md (rules), SKILL.md (procedures).
3. **Small always-loaded core.** Big instruction files cost 20%+ more with no gain (ETH study, 2026). Load details on demand.
4. **Handoff is the heart.** Start: read STATE + git log. End: update STATE + LOG + commit.
5. **Locks in the system, not in chat.** A chat rule got lost and an agent deleted 200+ emails (Feb 2026). viOS uses folders, permissions, hooks and encryption.
6. **Compile, don't pile.** Sources go to `raw/`; the AI keeps a clean wiki (ingest → query → lint).

## Architecture

```
                 ┌──────────── any AI ────────────┐
   OpenCode · goose · Claude Code · Cursor · Kiro · Claude Desktop · local Ollama
                 │ AGENTS.md + skills        │ MCP
                 ▼                           ▼
   ~/own/viOS ───────────────────────────────────────────────
   core/   (AI read-only)   CORE · USER · GOALS · VALUES · RULES
   ai/     (AI read+write)  INDEX · inbox · daily · projects/<p>/{README,STATE,LOG,DECISIONS}
                            tasks (Backlog.md) · agents (registry) · people · life · knowledge/{raw,wiki} · logs
   .agents/skills/          vi-resume · vi-handoff · vi-capture · vi-today · vi-new-project · vi-ingest
                            vi-query · vi-lint · vi-review · vi-agent · vi-decide · vi-life
   tools/                   dashboard.py · doctor.py · new_project.py
   setup/                   install · private zone · schedules (launchd) · MCP configs
   ──────────────────────────────────────────────────────────
   ~/viOS-Private   (encrypted, no AI)   Journal · Family · Health · Money · People · Ideas
```

## Zone enforcement (layers)

| Layer | Core (read-only) | Private (no AI) |
|---|---|---|
| Location | inside viOS | **outside** viOS (`~/viOS-Private`) |
| AGENTS.md / RULES.md | "never edit" | "never access" |
| Claude Code `.claude/settings.json` | deny Edit/Write `core/**` | deny Read/Edit `~/viOS-Private/**` |
| OpenCode `opencode.json` | edit deny `core/**` | `external_directory: deny` |
| MCP servers | Basic Memory + qmd point at `ai/` only | never pointed there |
| Git pre-commit hook | blocks core/ commits unless `(secret removed)` | blocks private paths + secrets |
| Encryption | — | Cryptomator vault |
| OS user (optional) | `viosagent` read-only ACL | no permission at all |

## Roadmap
- **v0.1 (tonight):** structure, rules, skills, tools, configs, install script.
- **v0.2 (week 1):** Vishnu fills core/, adds top projects + agents, uses resume/handoff daily by hand.
- **v0.3 (week 2):** qmd index, Claude Desktop/Cursor MCP connected, first repo linked.
- **v0.4 (week 3–4):** scheduled agents on (morning / nightly / weekly), optional agent user.
- **Later:** own small viOS MCP server (vi_resume / vi_handoff as tools), phone capture (Logseq mobile / Syncthing), remote MCP for ChatGPT with auth, Forgejo for self-hosted git.

## Known limits
- Logseq's newer DB version is not file-based — use a **file graph** so files stay plain Markdown.
- Scheduled agents need the Mac awake. MCP is pull-only (nothing wakes an AI on new events yet).
- Basic Memory has MCP tools that can add projects — only the OS agent-user lock fully stops misuse.
- Basic Memory may add a `permalink:` line to note frontmatter when it indexes. Harmless; check the first run with `git diff`.
- Logseq: if daily notes don't show as journals, check `logseq/config.edn` → `:journals-directory`. Files stay in `ai/daily/` either way.
- Health/money: AI output is a suggestion only.
