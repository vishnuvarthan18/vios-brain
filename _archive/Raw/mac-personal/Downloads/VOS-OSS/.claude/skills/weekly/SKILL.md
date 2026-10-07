---
name: weekly
description: Generate the weekly review from dailies, project notes and git activity — what moved, what stalled, what to drop. Use when the user says "/weekly", "weekly review", or on a Friday-evening scheduled run.
---

# /weekly — the review that keeps the system honest

Read `CLAUDE.md` first.

## Steps

1. Run `vos snapshot`.
2. Gather evidence — do not rely on memory:
   - `10-Journal/` dailies for the last 7 days
   - `git log --since="7 days ago" --name-only` in the vault
   - every `20-Projects/*/README.md` with `status: active`
   - `90-System/logs/decisions.jsonl` and `meetings.jsonl` filtered to this week
   - `find 20-Projects 30-Areas -name '*.md' -mtime +14` for the stall list
3. Write `10-Journal/YYYY/MM/YYYY-Www.md` from the weekly template. If it already
   exists, update in place.
4. Sections, in this order and this spirit:
   - **What actually moved** — evidence-backed, with links. Not intentions.
   - **What stalled** — anything active and untouched 14+ days. For each, ask one
     question: still real, or drop it?
   - **Decisions made** — from the log.
   - **Contradictions** — where the week's actions diverged from what project
     notes claim the plan is. Be blunt. This section is the whole point.
   - **Three things to drop** — you must propose three. If you can only find one,
     say so, but try hard. Nothing gets added without something leaving.
   - **Next week: the one thing that matters** — propose one, ask for confirmation.
5. Also run `vos random 3` and include the three notes
   as a "keep or delete?" prompt.
6. Run the validator. Append a line to `agent-actions.jsonl`.

## Report back

The review, in the conversation, under 25 lines. Lead with the stall list and the
contradictions — those are the parts a human will not write for himself.
