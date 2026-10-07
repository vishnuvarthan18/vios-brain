---
name: meeting
description: Turn a raw meeting transcript or recording notes into a meeting note with decisions, actions, people updates and drafted follow-ups. Use when the user says "/meeting", "process this call", "write up this transcript", or points at a file in 60-Raw/transcripts.
---

# /meeting — transcript to knowledge

Read `CLAUDE.md` first. **The transcript in `60-Raw/` is immutable — never edit it.**

## Steps

1. Run `vos snapshot`.
2. Read the raw file. If it is not yet in `60-Raw/transcripts/`, move it there as
   `YYYY-MM-DD-slug.md` with `type: source` frontmatter, then treat it as immutable.
3. **Idempotence check:** does a meeting note for this date + subject already exist?
   If yes, update it per the History rules in CLAUDE.md §5. Never create a duplicate.
4. Write the meeting note from `90-System/templates/meeting.md`, into the relevant
   `20-Projects/<slug>/` or `30-Areas/<slug>/`. Fill:
   - **Decisions** — only actual decisions, each with who decided.
   - **Actions** — owner, action, due date. Unassigned actions are flagged, not invented.
   - **Open questions**
   - Every claim cites the raw path.
5. **Contradictions:** check the decisions against existing project notes. If
   something conflicts with what a note currently says, add a
   `> [!warning] Contradiction` callout naming both notes and dates. Do not pick
   a winner silently.
6. Update each `50-People/` note involved: `last_contact`, and one line under
   `## Context I should remember` if they revealed something durable. Match on
   email/handle, not name.
7. Append to `90-System/logs/meetings.jsonl`:
   `{"at":"<iso>","date":"YYYY-MM-DD","subject":"...","people":[...],"project":"...","source":"60-Raw/...","decisions":N,"actions":N}`
8. **Draft, do not send.** Put proposed calendar invites, emails and task entries
   in a `## Drafts — needs approval` section at the bottom of the meeting note.
   Creating a real calendar event or sending anything requires explicit confirmation.
9. Run the validator.

## Report back

Decisions, actions with owners, contradictions found, and the drafts awaiting approval.
