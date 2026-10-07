---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, setup]
---

# 3. The agent layer

Your vault is a folder of files in git. That is the whole integration. There is
no plugin to install, no server to run, and nothing breaks when a vendor changes
their API.

## Point the agent at the vault

Open a session with the vault as the working directory. The agent reads
`AGENTS.md` (and `CLAUDE.md`, which points at it) and follows the contract:
where things go, what frontmatter is required, what it may never touch, and what
needs your approval.

## Give it hands

Everything the agent needs is already here:

| Job | Command |
|---|---|
| Snapshot before writing | `vos snapshot` |
| Today's note | `vos daily` |
| Capture a line | `vos jot "text"` |
| Create a note correctly | `vos new note "Title as a claim"` |
| Text search | `vos find "term"` or `rg -i "term"` |
| Query by frontmatter | `vos ls --type project --status active --json` |
| Validate its own work | `vos check` |
| Rebuild dashboards | `vos dash` |
| Nightly maintenance | `vos tidy` |

Two habits matter more than any tooling: **`vos snapshot` before writing**, and
**`vos check` after**. The first makes every mistake reversible with
`git checkout -- .`. The second catches the mistakes automatically.

## Optional: graph queries

`zk` (GPL-3.0) adds real link-graph queries over a Markdown folder — backlinks,
orphans, broken links, tag filters, full-text search — headlessly.

```bash
brew install zk
```

`vos check` already reports broken links and `vos ls` handles frontmatter
queries, so treat `zk` as an upgrade rather than a requirement. Note its last
release was July 2024: stable, but not actively developed.

## Optional: an MCP server

If you want a chat client (rather than a coding agent) to reach your notes, the
open-source option is **basic-memory** (AGPL-3.0), which speaks MCP over a plain
Markdown folder. You do not need it for an agent that can already run shell
commands — files plus `rg` is faster and cheaper than any MCP wrapper.

## What you are NOT doing

Not building a custom MCP server with a dozen file operations. `create note`,
`append` and `daily note` are one line of shell each — that is what `vos` is.
Wrapping them in a server costs tokens on every session and takes away `grep`,
`git` and pipes, which is where the real power is.
