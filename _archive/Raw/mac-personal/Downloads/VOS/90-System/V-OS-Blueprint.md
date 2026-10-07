# V OS — your second brain, designed properly

**For:** Vishnu, araCreate Group
**Date:** 20 August 2026
**Setup you chose:** full power rig, Mac + Android, everything in one place
**Reference point:** the gist you sent, which we are deliberately going to beat

---

## 1. What you are actually building

Three things, in this order. Most people get the order wrong and quit.

1. **A place where everything lands.** Plain text files on your own disk. One
   folder. No app can take it away, no company can shut it down, and in twenty
   years it still opens.
2. **A contract that an AI can obey.** Not "AI features". A written set of rules —
   where things go, what a note must contain, what the AI is forbidden to touch —
   so an agent can file, link, summarise and review *without* quietly wrecking
   your notes.
3. **Loops that run without you.** A nightly check, a weekly review, a monthly
   audit, a habit that deletes things. This is the part almost nobody builds, and
   it is the difference between a second brain and a landfill.

The files are the truth. Everything else — search indexes, embeddings, dashboards —
is disposable and rebuildable. If you remember one sentence from this document,
that's the one.

---

## 2. Why it failed last time, precisely

You said Obsidian "felt complex" a year back. That is the single most common
outcome, and the research is unusually clear about the mechanism. It is not that
you lacked discipline.

Three documented failure modes, and what we do about each:

**The mausoleum.** The best-known account is Joan Westenberg deleting ~10,000
notes after seven years: *"my second brain became a mausoleum… Instead of
accelerating my thinking, it began to replace it."* Capture became a substitute
for thinking. → **Fix:** V OS deletes on purpose. A script shows you three random
notes and asks you to bin them. Notes carry a `stale_after` date. The weekly
review is required to propose three things to drop. Nothing is added without
something leaving.

**The plugin tax.** One documented case: *"I spent my first three weeks in
Obsidian without actually writing a single meaningful note… I was so exhausted
from managing the tool that I had no brainpower left to actually use it."* → **Fix:**
zero plugins in week one. A hard cap of about twelve, ever. Install only when you
can name the specific thing you cannot do. Never browse the plugin directory for
fun.

**Organising instead of working.** People restructure their vault for months. The
long-term survivors — including one running a 26,000-file vault — use *one to six*
top-level folders, two levels deep at most, two to four tag axes, and let
**filenames** carry the organising load. One four-year vault owner scrapped his
whole system at 200 notes and switched to "non-perfectionism" as an explicit
principle. → **Fix:** the folder structure is decided once, in this document, and
declared closed. If you catch yourself renaming folders in week one, that is the
alarm.

One more finding worth internalising, because it will save you weeks: a
2.5-year survivor with journals going back to 2012 had linked **about twenty
notes in total** and still called his vault a success. Dense linking is not a
requirement. Neither is a perfect daily streak — his journal has month-long gaps.

---

## 3. The nine layers of V OS

Think of it as a building. You cannot furnish the fourth floor before pouring the
foundation, and that is exactly the mistake to avoid.

| # | Layer | What it is | Phase |
|---|---|---|---|
| 0 | **Foundation** | One Obsidian vault, plain Markdown, git underneath, sync to Android | 1 |
| 1 | **Structure** | 9 numbered folders, closed vocabulary, templates | 1 |
| 2 | **Capture** | Daily note, inbox, Android widget, voice, clipper, email | 1 & 4 |
| 3 | **Contract** | `CLAUDE.md` + `schema.md` + a validator script | 2 |
| 4 | **Hands** | Obsidian CLI + official Obsidian agent skills + plain shell | 2 |
| 5 | **Recall** | grep first, `qmd` hybrid semantic search second | 2 |
| 6 | **Loops** | Nightly tidy, weekly review, monthly audit, random-three deletion | 3 |
| 7 | **Truth** | Compiled Truth + append-only History + immutable `60-Raw/` | 3 |
| 8 | **Output** | Decision records with predictions, publishing, dashboards, recall | 5 |
| 9 | **Safety** | git snapshots, blast radius, `.agentignore`, layered backups | 1 & 3 |

---

## 4. The stack (exact tools, exact prices)

Obsidian is the right base in August 2026, and the reason is specific: it is the
only option where **the files are plain, the vendor ships an official command-line
tool, and the vendor's own CEO publishes agent skills** — while still having a
real Android app. Everything else fails at least one of those.

| Layer | Tool | Cost |
|---|---|---|
| Vault | **Obsidian 1.13.x** desktop + Android | Free (commercial licence is optional, $50/user/yr as patronage) |
| Sync Mac ↔ Android | **Obsidian Sync Standard** | $4/user/mo billed annually |
| Version history / undo | **git** + a private GitHub repo | Free |
| Backup | Obsidian **File Recovery** (5 min / 1 yr) + **Local Backup** plugin to an external disk + Apple Time Machine | Free |
| Agent hands | **Obsidian CLI** (built in, needs 1.12.7+) | Free |
| Agent knowledge | **`kepano/obsidian-skills`** — official skills for Obsidian Markdown, Bases, Canvas, CLI | Free |
| Semantic search | **`qmd`** — local hybrid search, ~2 GB of models, runs on your Mac, nothing leaves the machine | Free |
| Android capture | **MacroDroid** widget writing straight into the daily note | Free tier is enough |
| Android voice | **FUTO Voice Input** (offline Whisper dictation) | Free |
| Web clipping | **Obsidian Web Clipper** (Firefox on Android, any browser on Mac) | Free |
| Mac quick capture | **Raycast** Obsidian extension, or a one-line CLI script on a hotkey | Free |
| Meeting transcription | **whisper.cpp** or MacWhisper, local on Apple Silicon | Free / €59 once |
| Tasks | **Obsidian Tasks** in Markdown as the source of truth; phone reminders via TaskForge or TickTick | Free / small |

**Running cost: about $4/month.** Optional additions: Readwise Reader ($9.99/mo)
if you read a lot, a €5/mo VPS if you later want a Telegram capture bot that works
when your Mac is asleep.

**Sync warning, learned from other people's pain:** run exactly one sync
mechanism. Two syncers on one vault is the most common cause of duplicate-file
corruption. And iCloud is out — Apple ships no Android client, so it cannot work
for you at all. Obsidian Sync's Standard tier caps at 1 GB and 5 MB per file; if
you start putting client PSDs and video in the vault, go Plus ($8/mo) or keep
heavy media outside the vault.

**A deliberate architectural choice, against the gist.** The reference design
wraps the vault in a custom MCP server with ~17 operations. Skip that. `create
note`, `append`, `daily note` are one-line shell commands — the Obsidian CLI does
all of them, plus search, tasks and tags, with JSON output. Wrapping them in a
custom server costs you tokens on every session and, more importantly, costs you
`grep`, `find`, `git log` and pipes. There is real 2026 evidence that plain grep
beats vector search on personal notes for the questions people actually ask
(exact names, dates, "what did I say about X"). Keep MCP for the two things a
shell genuinely cannot do: the semantic index (`qmd` already ships its own MCP
server — don't rebuild it) and calendar/reminders.

---

## 5. Structure: nine folders, decided once

```
00-Inbox/      unfiled capture — always allowed to be messy
10-Journal/    YYYY/MM/YYYY-MM-DD.md dailies, weeklies, monthlies
20-Projects/   things with a finish line (client work AND your own projects)
30-Areas/      ongoing, no finish line (araCreate ops, finance, health, learning)
40-Library/    ideas and knowledge — flat, titles are claims
50-People/     one file per human
60-Raw/        transcripts, clippings, exports — IMMUTABLE, never edited
90-System/     templates, schema, scripts, logs, audits, dashboards
99-Archive/    finished and dead things — nothing is deleted, it moves here
```

Never more than two levels deep below any of those. Nine is at the upper edge of
what survivors run, and each one exists because it holds a genuinely different
*kind* of thing — which is what makes it possible for an agent to file correctly
without asking you every time.

**Why folders at all, in the age of semantic search?** Because a file path is free
context. Every tool — `grep`, an agent's file reader, your own eyes — sees the
folder name at zero cost. Frontmatter has to be parsed across the whole vault
before it means anything. So: **folders carry type and lifecycle, frontmatter
carries machine-checkable facts, tags carry topics only, links carry claims you
personally made.** That split is the whole design, and it resolves the
folders-vs-tags argument permanently.

**The vocabulary is closed.** Twelve `type` values, eight `status` values, listed
in `90-System/schema.md`. Adding one is a deliberate act. This is not
bureaucracy — it is the thing that lets a validator script catch an agent's
mistakes automatically, which is the only reason you can safely let an agent write
at all.

**Naming carries the load.** Dailies `2026-08-20.md`. Raw files date-first,
`2026-08-20-acme-call.md`. Library notes titled as the **claim**: "Retainers beat
project fees for small studios", not "Pricing". A note titled with a topic is a
folder pretending to be a note.

---

## 6. The truth rules — the part that makes this better than the gist

The reference design tells an agent *how* to write. It never says what happens
when a fact changes. That gap is where every one of these systems rots, and it
shows up at around a thousand notes: pages start citing other pages instead of the
original source, and errors become self-reinforcing.

Four rules, all in your `CLAUDE.md`:

**1. Compiled Truth on top, History below.** Every project, area and person note
opens with the current best understanding — short, accurate, the first thing an
agent reads. Underneath, `## History` is append-only: dated lines saying what
changed. Nothing above is ever edited away. You get the benefit of a temporal
database from one section heading.

**2. Supersede, never overwrite.** A replaced note gets `status: superseded` and
`superseded_by: "[[New note]]"`. It stays. Being wrong is data.

**3. `60-Raw/` is immutable and everything cites it.** Transcripts, clippings and
exports are captured once and never edited. Summaries live elsewhere and cite the
raw path. An agent may never copy a claim from another summary — it goes back to
source. This is the specific defence against the self-citation collapse.

**4. Contradictions get flagged, not resolved silently.** If new information
conflicts with an existing note, the agent adds a warning callout naming both
notes and both dates, and tells you. It does not pick a winner. This one rule is
worth more than any amount of clever search.

Plus one structural choice: **structured events go in append-only JSONL files**
under `90-System/logs/`, not in Markdown tables. An agent can silently rewrite a
table. It cannot silently rewrite an append. Decisions, meetings, contact
touchpoints and every batch of agent writes go there — the last one is your audit
trail.

---

## 7. Four commands. Exactly four.

The best documented negative result in this space is someone who succeeded on
their *sixth* attempt at a second brain, and only after cutting 25 commands down
to four. Your four are already written into the starter vault:

- **`/inbox`** — process everything in `00-Inbox/` and today's captures: file it,
  add frontmatter, link it, log it. Anything ambiguous gets left with a question,
  never guessed.
- **`/meeting <transcript>`** — raw transcript to a meeting note: decisions,
  actions with owners, open questions, updates to the people involved, and
  contradictions flagged. Calendar invites and emails are *drafted for approval*,
  never sent.
- **`/weekly`** — the review, built from evidence rather than memory: dailies, git
  activity, project notes, logs. It must report what stalled 14+ days, where your
  actions contradicted your stated plan, and three things to drop.
- **`/decide <question>`** — a decision record with options, the call, a
  **checkable prediction**, a confidence number, and a revisit date. `--revisit`
  comes back later and scores you.

That fourth one is the highest-leverage thing in the whole system and almost
nobody has it. A decision log that nobody scores is a diary. A decision log with
predictions and a scheduled revisit turns your vault into a calibration
instrument: after a year you will know, with numbers, which kinds of judgement
you are good at and which you should stop trusting.

---

## 8. The loops that keep it alive

| When | What runs | Where the output goes |
|---|---|---|
| Every agent session | git snapshot before the first write | git log |
| Nightly, 02:30 | mechanical health check: frontmatter validation, broken links, stale notes, counts, `qmd` reindex, auto-commit | `90-System/audits/YYYY-MM-DD-health.md` |
| Nightly | three random notes surfaced for deletion | in that same report |
| Friday | `/weekly` review generated | `10-Journal/` |
| Monthly | audit report and a goals check: rewrite them or formally mark them legacy | `90-System/audits/` |
| Monthly, 15th | `/decide --revisit` scores due predictions | decision notes + log |

The nightly job is a plain shell script — no AI, no writes to your notes, safe to
run unattended forever. It is already in the vault
(`90-System/scripts/nightly-tidy.sh`) with a launchd file next to it.

Because the health report is written to a **dated file**, vault health becomes a
time series rather than a one-off. In three months you will be able to see whether
your system is getting healthier or quietly rotting.

One thing to know before you get ambitious: cloud-scheduled AI runs sit behind a
network proxy and cannot reach your local vault or run `qmd`. Nightly maintenance
runs on your Mac, via launchd. That is not a limitation to work around; it is the
correct place for it.

---

## 9. Capture — the part that decides whether you use it at all

Capture friction is the binding constraint. If getting a thought into the vault
takes more than about five seconds, you will use Google Keep instead and the vault
dies. So:

**Android (your weak point — plan for it):**
- **Primary: MacroDroid** macro that appends a line straight to today's daily note
  file, on a home-screen shortcut and a quick-settings tile. Roughly two seconds,
  and it works whether or not Obsidian is running. This bypasses the `obsidian://`
  URL scheme deliberately — that scheme got confirmation dialogs in 1.13 and is
  now slower and less reliable.
- **Voice: FUTO Voice Input** — offline Whisper dictation keyboard. Dictate into
  the MacroDroid prompt. Voice is faster than typing and it is the only capture
  method that survives walking and driving.
- **Reading: Firefox for Android** + Obsidian Web Clipper. Chrome on Android
  cannot run extensions at all.
- **Email:** forward anything to a dedicated address and process it later. Zero
  install, works from any app.
- Set the Android vault to **Device Storage** with "all files access", and keep
  plugin count low — the Android lag and crash reports almost all trace back to
  desktop-grade plugins on a phone.

**Mac:**
- Raycast's "Append to Daily Note", or a hotkey running
  `obsidian daily:append content="..."`. No plugin, no dialog.
- Obsidian Web Clipper with a local model for private pages.
- Meetings: record, transcribe locally with whisper.cpp (a `small` model runs
  about 6–12× realtime on Apple Silicon), drop the transcript into
  `60-Raw/transcripts/`, then run `/meeting`.

**One thing to kill:** Google Keep. There is no maintained live bridge to a vault,
and there hasn't been for years. Do a one-time export import and stop using it.
Maintaining a bridge is more work than changing the habit.

---

## 10. Safety — because you are giving an AI write access to your business memory

Non-negotiable, all already set up in the starter vault:

1. **git snapshot before every agent session.** `pre-agent-commit.sh`. Then
   `git diff` is your review surface and `git checkout -- .` is your undo.
2. **Blast radius.** The agent writes freely only in `00-Inbox/`, `10-Journal/`
   and the logs. `60-Raw/`, `99-Archive/` and `.obsidian/` are off limits entirely.
3. **`.agentignore`.** Put anything private in there — contracts, health, family,
   credentials — and the agent treats it as invisible.
4. **The human gate.** The agent may draft a calendar invite, an email, an
   invoice. It may not send or create one. This matters more than it sounds:
   "read an untrusted transcript, then take an external action" is exactly the
   pattern that gets people burned by prompt injection hidden in a document.
5. **Sync is not backup.** Sync replicates a deletion everywhere instantly. Layer
   it: File Recovery at 5 minutes / 1 year, Local Backup zips to an external disk,
   git with real commits, Apple Time Machine. A text vault costs cents to back up
   off-site.
6. **Never let the agent have the whole vault by default,** and never install a
   plugin that reverse-engineers a private API.

---

## 11. The phase plan

Each phase has a "done" test. Do not start the next one until it passes. This is
the part where the last attempt went wrong.

### Phase 1 — Foundation (this weekend, about 3 hours)
Vault open, folders in place, templates wired, Obsidian Sync running to the phone,
git initialised, capture working on both devices. **Zero plugins.**
**Done test:** seven consecutive days with something in the daily note, and you
have not renamed a single folder.

### Phase 2 — The agent layer (one evening, week 2)
Enable the Obsidian CLI. Install `kepano/obsidian-skills`. Point Claude at the
vault so it reads `CLAUDE.md`. Install `qmd` and index everything — not just
"mature projects"; semantic search is *most* valuable on the old notes you have
forgotten. Run `/inbox` for the first time.
**Done test:** you ask a question about your own notes and get an answer with
correct citations, and `validate.py` comes back clean afterwards.

### Phase 3 — Loops (two hours, week 3)
Schedule `nightly-tidy.sh` via launchd. Run `/weekly` on Friday. Read the health
report.
**Done test:** three health reports exist and you did not create them by hand.

### Phase 4 — Real life (week 4)
Android capture properly wired, voice dictation, local meeting transcription, the
people layer populated with the twenty humans who actually matter, tasks in
Markdown with phone reminders.
**Done test:** a real client call goes from recording to filed meeting note with
actions in under ten minutes, and you didn't type the notes.

### Phase 5 — Compounding (month 2 onward)
Decision records with predictions. The monthly revisit. Bases dashboards over
projects and people. A publishing path out of the vault, so it produces and not
just absorbs. Optionally, spaced-repetition cards on the things you want to
actually remember.
**Done test:** at day 90, `/decide --revisit` scores your first predictions and
you learn something uncomfortable about your own judgement.

---

## 12. Things to deliberately not do

- Do not install plugins in week one. Not one.
- Do not build a custom MCP server for file operations. The CLI already does it.
- Do not add a fifth command until the four are habits.
- Do not tag by topic beyond about thirty tags total. Nothing depends on tags.
- Do not aim for a perfect daily streak. Gaps are fine; the survivors all have them.
- Do not link everything. Twenty deliberate links beat two thousand automatic ones.
- Do not let an agent create a wikilink to a note it has not verified exists.
- Do not run two sync tools.
- Do not put your entire business's client media inside a 1 GB sync plan.
- Do not restructure. If you feel the urge in the first month, write a note about
  the urge instead. That note will be more useful than the restructure.

---

## 13. How this beats the design you sent

The gist is a competent build. Here is where V OS is deliberately different:

| The gist | V OS |
|---|---|
| Custom MCP server, ~17 file operations | Official Obsidian CLI + shell. No custom server to maintain, keeps grep/git/pipes |
| No rules for what happens when a fact changes | Compiled Truth + append-only History + supersede + contradiction flagging |
| Summaries can cite summaries | `60-Raw/` immutable, every claim cites source — the fix for the ~1,000-note collapse |
| Human-triggered only | Nightly / weekly / monthly loops, health as a time series |
| Apple Reminders + Calendar | Markdown-native tasks; works on Android; nothing locked in one vendor |
| Apple-only capture | Android widget, offline voice, clipper, email — you are on Android |
| No backup or rollback story | git snapshot per agent run, layered backups, selective restore |
| No security boundary | Blast radius, `.agentignore`, human gate on all external actions |
| Semantic search only on "mature" projects | Index everything — old forgotten notes are where search pays |
| Absorbs only | Decision predictions with scoring, publishing path, recall |
| Nothing prevents rot | Random-three deletion, `stale_after` dates, forced "drop three" in every review |

---

## 14. The 90-day scoreboard

Not "does it feel organised". Three checks:

1. **Did I use it?** Count daily notes with content in the last 30 days. Under 15
   means the capture path is too slow — fix capture, not structure.
2. **Did it answer something I'd have otherwise lost?** Keep a running list in
   `40-Library/vos-wins.md`. If it's empty at day 90, the vault is a diary and
   that's fine — but stop building infrastructure for it.
3. **Is it getting healthier?** Compare the first and latest health reports:
   broken links, unvalidated notes, stale actives. Trend down, not up.

And one honest question at day 90: **what did you delete?** If the answer is
nothing, the mausoleum is already forming, and that is the only failure mode that
kills these systems for good.
