---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, reference]
---

# V OS v2 — free, open source, yours

**For:** Vishnu, araCreate Group
**Date:** 21 August 2026
**Change from v1:** no Obsidian, no subscriptions, no closed-source dependency
**Recurring cost:** $0 (optionally €2-6/month if you later want a server)

---

## 1. What changed, and what didn't

You flagged two things: you don't want to depend on Obsidian, and you don't want
to pay. Worth separating them, because one was smaller than it looked.

**The money:** Obsidian itself was already free, including commercially. The only
paid part of v1 was **Sync at $4/month**. That's now replaced by Syncthing
(peer-to-peer, MPL-2.0, free) or plain git.

**The dependency:** this is the real point and you were right to raise it.
Obsidian is closed source, made by a ~15-person private company, and v1 leaned on
three Obsidian-specific things: its CLI, its Bases dashboards, and its app. All
three are now replaced by things you own.

**What did not change — about 85% of the design:**

- The nine-folder structure, unchanged
- The closed-vocabulary frontmatter schema, unchanged
- The agent contract (`AGENTS.md`), unchanged in substance
- The truth rules — immutable raw, Compiled Truth, append-only History,
  contradiction flagging — unchanged, and still the most valuable part
- The four commands (`/inbox`, `/meeting`, `/weekly`, `/decide`), unchanged
- The maintenance loops, unchanged
- The phase order, unchanged

That's the payoff of plain files: swapping the entire tool layer left the actual
system intact.

**The three replacements:**

| v1 (Obsidian) | v2 (yours) |
|---|---|
| Obsidian CLI | **`bin/vos`** — one dependency-free Python file, ~400 lines, in the kit. Does daily notes, capture, note creation from templates, search, frontmatter queries, validation, dashboards, git snapshots, and the nightly job. |
| Bases (database views) | **`vos dash`** — generates plain-Markdown dashboards into `90-System/dashboards/`. Readable on your phone, in any editor, by `grep`, and by your agent. Bases could never do that. |
| Obsidian the app | **Your choice of three** — see the front-end comparison document. |

One thing worth knowing: **Obsidian's CLI required the Obsidian app to be
running.** It was a remote control for a GUI, not a headless tool. `vos` runs
anywhere, including on a server at 2am with nothing open. You traded up.

---

## 2. The stack (every line free and open source)

| Layer | Tool | Licence | Cost |
|---|---|---|---|
| Notes | Markdown files in a folder | yours | $0 |
| History & undo | **git** | GPL-2.0 | $0 |
| Command line | **`vos`** (in the kit) | yours | $0 |
| Search | **ripgrep** | MIT / Unlicense | $0 |
| Frontmatter scripting | **jq**, **yq** | MIT | $0 |
| Mac editor | **VSCodium + Foam**, or **Zettlr** | MIT / GPL-3.0 | $0 |
| Android editor | **Markor** | Apache-2.0 | $0 |
| Sync Mac ↔ Android | **Syncthing** + **BasicSync** or Syncthing-Fork | MPL-2.0 / GPL-3.0 | $0 |
| Android capture | **Termux + Termux:API + Termux:Widget** | GPL-3.0 | $0 |
| Android voice | **Voxscribe** + **Sayboard** | MIT / GPL-3.0 | $0 |
| Semantic search | **qmd** (local models, nothing leaves the Mac) | MIT | $0 |
| Graph queries (optional) | **zk** | GPL-3.0 | $0 |
| Meeting transcription | **whisper.cpp** | MIT | $0 |
| Browser access (optional) | **SilverBullet** on Linux | MIT | €0-6/mo |
| Publishing (optional) | **Quartz** → Cloudflare Pages | MIT | $0 |
| Off-site backup | git remote (GitHub/Codeberg free private repo) | — | $0 |

Total: **$0/month.** The only optional cost is a server if you decide you want
notes in a browser from anywhere — and an old laptop at home does that for free.

---

## 3. Sync, honestly

This is the part that replaced a paid product, so it deserves the caveats.

**Syncthing** is peer-to-peer: your Mac and phone talk directly, encrypted, with
no cloud in the middle and no account. For a folder of text files it is
excellent, and it's what most people in this position use.

**Three things to get right:**

1. **Which Android app.** The official Syncthing Android app was archived in
   December 2024. Syncthing's own docs now point to two community apps:
   **BasicSync** (GPL-3.0, by a long-standing Android FOSS maintainer) and
   **Syncthing-Fork** (more features, but the project changed hands twice in
   2025-2026 and the handover was poorly announced). **I'd use BasicSync**, and
   install from F-Droid either way — F-Droid verifies that the app matches its
   published source.
2. **Conflicts are not merged.** Syncthing keeps both versions and renames the
   loser `...sync-conflict-<date>-....md`. The kit includes
   `90-System/scripts/deconflict.sh`, which union-merges journal files
   automatically (safe, because journals only ever get appended to) and flags the
   rest for you. `.gitattributes` does the same trick on the git side.
3. **Run exactly one sync tool.** Syncthing *or* git-based sync on a folder,
   never both. Two syncers on one folder is the classic way to corrupt it.

**Git is your backup, not your sync.** Commit locally, push to a free private
repo. Sync replicates deletions instantly; git lets you go back. They are
different jobs and you want both.

---

## 4. Capture — the part that decides everything

If getting a thought in takes more than ~5 seconds, you'll use something else and
the vault dies. So this is where the effort goes.

**Mac:** `vos jot "the thing"`. Wrap it in a macOS Shortcut with an "ask for text"
prompt and bind a hotkey — no third-party software, no plugin.

**Android — the primary path:** Termux + Termux:Widget, with the `termux-jot.sh`
script from the kit sitting on your home screen. Tap, type, done — appended to
today's note whether or not any app is running. Roughly two seconds.

All three Termux apps must come **from F-Droid**, all from the same source, or
their signatures won't match.

**Android — voice:** **Voxscribe** (MIT) is a dictation keyboard running Whisper
entirely on-device; it requests no internet permission at all. Install
**Sayboard** as well and set it as your system speech recogniser, which makes
offline voice work inside the Termux capture script too. Voice is faster than
typing and it's the only capture method that survives walking and driving.

I deliberately excluded FUTO Voice Input here — it's very good, but its licence
is "Source First", not open source.

**Reading:** Firefox for Android plus the MarkDownload extension. Chrome on
Android cannot run extensions at all. Markor's share-into handles quick links
with zero setup.

**Meetings:** record, transcribe locally with whisper.cpp on your Mac (a `small`
model runs roughly 6-12× realtime on Apple Silicon), drop the transcript into
`60-Raw/transcripts/`, run `/meeting`. Nothing leaves your machine.

---

## 5. Everything valuable from v1, still here

Restating the parts that actually make this better than the gist you sent,
because they survived the tool change entirely — which is the proof they were the
real design and not decoration.

**Truth discipline.** Compiled Truth at the top of every project, area and person
note; `## History` below it, append-only; `status: superseded` instead of
overwriting; `60-Raw/` immutable with every claim citing its source path;
contradictions flagged in a callout naming both sides rather than silently
resolved. This is what stops the rot that hits these systems around a thousand
notes, where summaries start citing summaries and errors become self-reinforcing.

**Append-only event logs.** Decisions, meetings, contact touchpoints and every
batch of agent writes go to JSONL files in `90-System/logs/`, not Markdown tables.
An agent can silently rewrite a table. It cannot silently rewrite an append.

**Four commands, deliberately.** `/inbox`, `/meeting`, `/weekly`, `/decide`. The
best-documented success in this space was someone's sixth attempt, which only
worked after cutting 25 commands down to four.

**Decisions with predictions.** `/decide` forces a checkable prediction, a
confidence number and a revisit date; `--revisit` scores you later. A decision log
nobody scores is a diary. This one turns the vault into a calibration instrument.

**Deletion as a feature.** `vos random 3` surfaces three notes to bin, `vos stale`
finds abandoned actives, and `/weekly` must propose three things to drop. This is
the answer to the mausoleum — the 10,000-note vault its owner deleted because
collecting had replaced thinking.

**Safety.** `vos snapshot` before every agent session, blast radius limits in
`AGENTS.md`, `.agentignore` for private folders, and a human gate on every
external action. An agent may draft a calendar invite or an email; it may not send
one. That matters because "read an untrusted transcript, then take an irreversible
external action" is precisely the pattern that gets people burned.

---

## 6. The phase plan

Unchanged from v1 in shape, because the order is the part that stops you quitting.

### Phase 1 — Foundation (this weekend, ~1 hour, then a week of use)
Vault in `~/VOS`, `vos` on your PATH, git initialised, an editor installed,
Syncthing to the phone, Markor pointing at the folder.
**Done test:** seven consecutive days with something captured, and you have not
renamed a single folder.

### Phase 2 — The agent layer (one evening, week 2)
Point Claude at the vault so it reads `AGENTS.md`. Install `qmd` and index
everything — not just "important" notes; semantic search pays most on the old ones
you've forgotten. Run `/inbox` for the first time.
**Done test:** you ask a question about your own notes, get an answer with correct
citations, and `vos check` is clean afterwards.

### Phase 3 — Loops (one hour, week 3)
`vos tidy` nightly via launchd. `/weekly` on Friday. Read the health report.
**Done test:** three dated health reports exist and you didn't create them.

### Phase 4 — Real life (week 4)
Termux widget capture, offline voice, local transcription, the twenty people who
actually matter in `50-People/`, tasks in Markdown.
**Done test:** a real client call goes from recording to filed meeting note with
owners and actions in under ten minutes, without you typing the notes.

### Phase 5 — Compounding (month 2+)
Decision records and the monthly revisit. Dashboards you actually read.
SilverBullet on a server if you want browser access. Quartz if you want to
publish. Spaced repetition if you want to remember rather than just store.
**Done test:** at day 90, `/decide --revisit` scores your first predictions and
tells you something uncomfortable about your own judgement.

---

## 7. Things to deliberately not do

- Don't compare editors for more than 20 minutes. They all read the same files.
- Don't run two sync tools on one folder.
- Don't set up a server in week one. Month two, if the habit held.
- Don't build a custom MCP server for file operations. `vos` plus `rg` is faster,
  cheaper and more debuggable.
- Don't add a fifth command until the four are habits.
- Don't tag by topic beyond ~30 tags total. Nothing depends on tags.
- Don't chase a perfect daily streak. One long-running vault owner has month-long
  gaps across twelve years and considers it a success.
- Don't link everything. Another survivor linked about twenty notes total.
- Don't let an agent create a wikilink to a note it hasn't verified exists.
- Don't rely on Oracle's free tier as your only copy of anything — the allowance
  was quietly halved in June 2026 and idle accounts can be reclaimed.
- Don't restructure. If you feel the urge in month one, write a note about the
  urge instead.

---

## 8. What you gained and what you gave up

**Gained:**
- $0 recurring cost, forever
- Every component inspectable, forkable and self-hostable
- A CLI that works headlessly — Obsidian's needed its GUI running
- Dashboards as plain Markdown, readable everywhere including your phone and by
  your agent, which Bases couldn't do
- No vendor able to change terms, add a paywall, or pivot its storage format
  (which is exactly what Logseq just did to its users)

**Gave up, honestly:**
- **Polish.** Obsidian is a better-looking, better-integrated app than anything
  here. VSCodium is a code editor wearing a notebook hat.
- **The graph view** and the plugin ecosystem. You will not miss the plugins —
  they were the top documented cause of people abandoning their vaults — but the
  graph view is genuinely nice to have.
- **One app everywhere.** You now use an editor on the Mac and Markor on the
  phone. SilverBullet solves this later if it bothers you.
- **Someone else's maintenance.** `vos` is ~400 lines of Python you own. That's
  the deal: nothing to pay, nothing to trust, and nobody to call.
- **A safe rename.** No tool here rewrites wikilinks when you rename a note from
  outside an editor. `vos check` catches broken links nightly, which contains the
  damage — but rename inside your editor where possible.

That last list is the honest cost. I think it's clearly worth it given what you
said you wanted, but you should know what you're accepting.

---

## 9. The 90-day scoreboard

Same three questions as before:

1. **Did I use it?** Count daily notes with content in the last 30 days. Under 15
   means fix capture, not structure.
2. **Did it answer something I'd otherwise have lost?** Keep the list in
   `40-Library/vos-wins.md`. Empty at day 90 means stop building infrastructure.
3. **Is it getting healthier?** Compare the first and latest health reports in
   `90-System/audits/`. Broken links and stale actives should trend down.

And the one that matters most: **what did you delete?** If the answer is nothing,
the mausoleum is forming, and that's the failure mode that kills these for good.
