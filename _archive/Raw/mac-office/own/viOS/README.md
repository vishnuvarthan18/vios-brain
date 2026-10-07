# viOS

**Vishnu's personal operating system.** One place for all projects, AI agents, knowledge and life.
Plug any AI into it — Claude, ChatGPT, Cursor, OpenCode, goose, a local model — and it continues the work.

- 100% open source parts. Plain Markdown + Git. You own every file.
- Built from proven open-source systems (see `docs/SOURCES.md`).
- New here? Read `START-HERE.md` first.

## The 3 zones

```
~/own/viOS/core/     You write · AI reads only      (who you are, goals, values, rules)
~/own/viOS/ai/       AI reads + writes              (projects, tasks, agents, knowledge, daily)
~/viOS-Private/      Encrypted · no AI ever          (journal, family, health, money, secrets)
```

## The layers

| Layer | What | Tool (open source) |
|---|---|---|
| 1. Brain | Markdown files + Git | git |
| 2. Viewer | Graph, backlinks, journals | Logseq (AGPL) |
| 3. Rules | `AGENTS.md` + `core/CORE.md` | open standard |
| 4. Skills | `vi-*` commands | Agent Skills (SKILL.md) standard |
| 5. Plug | Any AI connects over MCP | Basic Memory (AGPL), Backlog.md (MIT), qmd (MIT) |
| 6. Agents | Run tasks, schedules | OpenCode (MIT), goose (Apache-2.0), Hermes (MIT), launchd |
| 7. Models | Local or cloud | Ollama (MIT) + open models, or Claude/ChatGPT |
| 8. Dashboard | One page overview | `tools/dashboard.py` → `ai/DASHBOARD.md` |

## Daily use

- Say to any AI: **"resume <project>"** → it reads STATE and continues.
- At the end: **"handoff"** → it saves state so the next AI can continue.
- **"capture: <thought>"** → goes to inbox. **"today"** → your plan for the day.
- `python3 tools/dashboard.py` → refresh the dashboard.

## Folders

See `AGENTS.md` §4 for the full map.
