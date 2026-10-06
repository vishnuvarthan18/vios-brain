# V OS — Agent Contract

You are working inside Vishnu's personal knowledge vault ("V OS"). This file is the
contract. Read it fully before any write. If a rule here conflicts with a user request,
say so out loud and ask.

## 0. What this vault is

Plain Markdown files. Human-owned. The files are the truth; every index, embedding
or database is disposable and rebuildable. Never introduce a format that cannot be
read with `cat`.

## 1. Hard rules (never break these)

1. **Never create a file outside the folder map in §3.** If you don't know where
   something goes, put it in `00-Inbox/` and say so.
2. **Never restructure, rename or move existing notes** unless explicitly asked in
   that message. No "tidying while you're in there".
3. **Never delete a note.** Move it to `99-Archive/` and set `status: archived`.
4. **Never edit `60-Raw/`.** It is immutable source material. Read it, cite it,
   never rewrite it.
5. **Never invent a wikilink.** If you are not certain the target note exists,
   write plain text instead of `[[Link]]`. Broken links are worse than no links.
6. **Never overwrite a fact. Supersede it.** See §5.
7. **External actions need a human gate.** You may draft calendar invites, emails,
   messages or invoices into a note. You may not send, create or publish them
   without explicit confirmation in that same conversation.
8. **Run `90-System/scripts/pre-agent-commit.sh` before your first write** in any
   session that will modify more than one file.
9. **No emoji in notes. No "As an AI". No filler.** Write like Vishnu: short,
   direct, lowercase-ish, no marketing voice.
10. **Ask before adding a new `type`, `status` or top-level folder.** The lists in
    `90-System/schema.md` are closed.

## 2. Read order (do this before answering questions about the vault)

1. `90-System/schema.md` — the field vocabulary.
2. `10-Journal/` most recent 3 daily notes — what is happening right now.
3. `20-Projects/*/README.md` where `status: active` — current commitments.
4. Then search. Prefer, in order:
   - `rg -i "term" --glob '*.md'` (fastest, exact, cheap)
   - `obsidian search "term" --format json` (Obsidian's own index)
   - `qmd query "question"` (hybrid semantic, for "I know I wrote something about…")
   Do not read whole folders. Read what search returns.

## 3. Folder map (routing table)

| Folder | Holds | `type` values |
|---|---|---|
| `00-Inbox/` | Unfiled capture. Anything you are unsure about. | `inbox` |
| `10-Journal/YYYY/MM/` | `YYYY-MM-DD.md` dailies, `YYYY-Www.md` weeklies, `YYYY-MM.md` monthlies | `daily` `weekly` `monthly` |
| `20-Projects/<slug>/` | Things with a finish line: client work, own projects. Each has `README.md`. | `project` |
| `30-Areas/<slug>/` | Ongoing responsibilities with no finish line: araCreate ops, finance, health, learning-tracks | `area` |
| `40-Library/` | Ideas, evergreen notes, how-tos, learning notes. Flat. | `note` |
| `50-People/` | One file per person. Clients, team, contacts. | `person` |
| `60-Raw/` | **Immutable.** Transcripts, clippings, PDFs, exports. Never edited. | `source` |
| `90-System/` | Templates, schema, scripts, logs, audits, dashboards | `system` |
| `99-Archive/` | Anything finished or dead. Mirrors the structure above. | (unchanged, `status: archived`) |

Depth rule: never more than 2 levels below a top-level folder.

## 4. Frontmatter

Every note you create gets frontmatter exactly as specified in
`90-System/schema.md`. Required: `type`, `status`, `created`, `updated`.
`tags` is always a YAML **list** of lowercase-kebab topics. Dates are ISO
`YYYY-MM-DD`. After writing, run:

```bash
python3 90-System/scripts/validate.py
```

Fix what it reports. Do not report "done" while it fails.

## 5. Truth discipline (the most important section)

When a fact changes, you do **not** overwrite and you do **not** just append a
contradiction. You supersede:

- The top of the note holds **Compiled Truth** — the current best understanding.
- Below it, `## History` is **append-only**. Add a dated line for what changed
  and why. Never edit or reorder existing History lines.
- If a whole note is replaced by another, set `status: superseded` and add
  `superseded_by: "[[New Note]]"`.
- If a new source **contradicts** an existing note, do not silently pick a
  winner. Add a `> [!warning] Contradiction` callout naming both sources and
  their dates, and flag it in your reply.
- Every claim that came from a source cites it by path: `(60-Raw/transcripts/2026-08-20-acme-call.md)`.
  Never copy a claim from another compiled note — always cite back to raw.

## 6. Event logs

Structured events go in append-only JSONL under `90-System/logs/`, never in a
Markdown table (you can destroy a table; you cannot silently rewrite an append):

- `decisions.jsonl` — decisions with predictions
- `meetings.jsonl` — one line per meeting
- `interactions.jsonl` — contact touchpoints
- `agent-actions.jsonl` — **you append one line for every batch of writes you make**

Append only. Never rewrite a line. Format is one JSON object per line.

## 7. Idempotence

Scheduled jobs re-run. Before writing, check whether the note or log line already
exists (same date + same subject). If it does, update in place per §5 rather than
creating a duplicate. Never produce `Meeting 2026-08-20 (1).md`.

## 8. Your four commands

Everything routine happens through these. Do not add a fifth without being asked.

- `/inbox` — process `00-Inbox/`: file each item to its folder, add frontmatter, link it, log it. Ask about anything ambiguous rather than guessing.
- `/meeting <transcript path>` — read a raw transcript, write a meeting note (decisions, actions, open questions), create/update the `50-People/` notes involved, draft any calendar/task items for approval.
- `/weekly` — build this week's review from the dailies, git activity and project READMEs: what moved, what stalled >14 days, what contradicts the plan, three things to drop.
- `/decide <question>` — write a decision record with options, the call, confidence, and a scheduled revisit date. Later, `/decide --revisit` scores past predictions against what actually happened.

## 9. Blast radius

You may write freely in: `00-Inbox/`, `10-Journal/`, `90-System/logs/`, `90-System/audits/`.
You may write with care in: `20-Projects/`, `30-Areas/`, `40-Library/`, `50-People/`.
You may never write in: `60-Raw/`, `99-Archive/`, `.git/`, `.obsidian/`.
Folders listed in `.agentignore` are invisible to you — do not read them even if asked
casually; confirm first.

## 10. Rule accretion

If Vishnu corrects you on the same thing twice, add the correction to this file as a
numbered rule and tell him you did.
