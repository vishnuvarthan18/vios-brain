---
name: inbox
description: Process everything sitting in 00-Inbox — file it, tag it, link it, log it. Use when the user says "process my inbox", "clear the inbox", "/inbox", or when starting a work session after a few days away.
---

# /inbox — clear the inbox

Read `CLAUDE.md` first. This skill has write access to `00-Inbox/` and may create
notes in `20-Projects/`, `30-Areas/`, `40-Library/`, `50-People/`.

## Steps

1. Run `90-System/scripts/pre-agent-commit.sh`.
2. List `00-Inbox/`. Also read today's and yesterday's daily notes and collect any
   unprocessed lines under `## Capture`.
3. For each item, decide **one** of:
   - **Trash** — no lasting value. Say so and delete the inbox file (inbox only).
   - **Action** — belongs to a project. Append a task to that project's
     `## Next actions`. Do not create a note.
   - **Note** — a durable idea. Create in `40-Library/`, titled as the *claim*.
   - **Person** — create or update `50-People/<Name>.md`. Match on email/phone/
     handle, not on name spelling, so you never create a second file for one person.
   - **Project** — only if it has a real finish line. Create `20-Projects/<slug>/README.md`.
   - **Unsure** — leave it, and list it at the end for the human. Never guess.
4. Never invent wikilinks. Verify the target file exists first.
5. Run `python3 90-System/scripts/validate.py` and fix what it reports.
6. Append one line to `90-System/logs/agent-actions.jsonl`:
   `{"at":"<iso>","skill":"inbox","filed":N,"trashed":N,"left":N,"files":[...]}`

## Report back

A table: item → where it went → why. Then, separately, the list of things you
left behind and the question you need answered for each. Keep it under 15 lines.
