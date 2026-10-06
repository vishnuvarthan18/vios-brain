---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, reference]
---

# Which front end? Three honest options

**Date:** 21 August 2026
**Constraints:** free forever, open source, macOS + Android, agent-writable plain files

You asked to see the options before picking. Here they are, with the trade-offs
stated plainly rather than sold.

---

## First, the thing that makes this decision low-stakes

Your notes are Markdown files in a git repo. **The front end is just a window
onto them.** All three options below read and write the same files. Switching
later costs you an afternoon of installing something, not a migration.

So: pick the one that sounds least annoying, start, and change your mind in a
month if you want. Deliberating longer than 20 minutes on this is the actual risk.

There is one hard test every option here passes, and it is the reason Obsidian
was replaceable at all: **can an AI agent read and write your notes with no app
running?** Yes for all three, because the notes are just files. Options that fail
this test — Joplin, Logseq's new version, Anytype, Notion — are excluded no
matter how nice they look.

---

## Option A — VSCodium + Foam *(recommended for you)*

**What it is:** VSCodium is the open-source build of VS Code with Microsoft's
branding and telemetry removed (MIT). Foam is an extension (MIT) that adds
wikilinks, autocomplete, a backlinks panel, and link-updating when you rename a
note. Together they turn a folder of Markdown into a proper linked notebook.

| | |
|---|---|
| Cost | $0 |
| Licence | MIT + MIT |
| Server needed | **None** |
| Android | Markor (separate app, same files) |
| Setup time | 15 minutes |
| Offline | Always |

**Why it fits you.** You're running a full-power rig with an AI agent in the
loop. That agent already lives in a terminal and a file tree — the same place
VSCodium does. Nothing to host, nothing to expose to the internet, nothing that
can break at 2am. It is also the most future-proof option here: there is no
app-side database to corrupt, ever, because there is no app-side state at all.

**What you give up.** It looks like a code editor, because it is one. No graph
view out of the box. Writing long prose in it is fine but not delightful. And on
your phone you're using a separate app (Markor), so it's two tools rather than one.

**Verdict:** the boring correct answer. Start here.

---

## Option B — Zettlr

**What it is:** a purpose-built Markdown writing app (GPL-3.0). Opens your folder,
gives you wikilinks, a file tree, citations, exports, and a pleasant writing
surface. Closer in feel to what you expected Obsidian to be.

| | |
|---|---|
| Cost | $0 |
| Licence | GPL-3.0 |
| Server needed | None |
| Android | Markor (separate app) |
| Setup time | 10 minutes |
| Offline | Always |

**Why you might prefer it.** If most of what you do is *write* — proposals,
articles, client documents — this is more comfortable than a code editor. Smaller
and simpler than VSCodium, with nothing to configure.

**What you give up.** A much smaller ecosystem. Fewer power features. No
extension marketplace. Its release cadence is slower than the others here, which
is worth watching but not disqualifying for a plain-text editor.

**Verdict:** a good choice if the code-editor look genuinely puts you off. You
can install it alongside Option A and use both on the same files.

---

## Option C — SilverBullet on a server

**What it is:** a self-hosted web app (MIT) over a folder of plain Markdown on
the server's disk. Open it in any browser, install it on Android as an app from
the browser menu, and it works fully offline with automatic sync when you
reconnect. It has wiki links, a query language, and Lua scripting written inside
your notes.

| | |
|---|---|
| Cost | €0-6/month (free on an old laptop at home; ~€2-6 on a rented box) |
| Licence | MIT |
| Server needed | **Yes** — Linux, plus HTTPS |
| Android | Real app experience, installed from the browser, offline |
| Setup time | An evening the first time |
| Offline | Yes, full local copy in the browser |

**Why it's tempting.** It is the only option that gives you one interface
everywhere, a genuinely good phone experience, and access from any machine you
happen to be sitting at. The maintainer explicitly confirms that editing the
files directly from outside the app — git, cron, your agent — is fine, which is
what makes it compatible with everything else in your setup.

**What you must accept — three real constraints:**

1. **It must run on Linux, not your Mac.** The docs discourage case-insensitive
   filesystems and macOS's default is case-insensitive. So this means a server:
   an old laptop at home, or a cheap VPS.
2. **HTTPS is mandatory**, not optional. Browsers only allow the offline features
   on a secure origin. The clean free path is a Cloudflare Tunnel (outbound only,
   no open ports, certificates handled for you) plus a domain, ~$10/year.
3. **Don't leave a page open on your phone while the agent writes to it.** It
   doesn't merge concurrent edits — it keeps both and makes a conflict copy. Also
   keep it under roughly 1,000-2,000 notes, because the search index is built on
   the device, and the first build on a phone takes minutes at that size.

**Verdict:** the best experience, at the cost of being the only option with
something to maintain. Worth doing — in **month two**, not week one.

---

## Side by side

| | A. VSCodium + Foam | B. Zettlr | C. SilverBullet |
|---|---|---|---|
| Licence | MIT | GPL-3.0 | MIT |
| Cost | $0 | $0 | €0-6/mo |
| Server | none | none | Linux + HTTPS |
| Setup | 15 min | 10 min | an evening |
| Feels like | a code editor | a writing app | a personal wiki |
| Phone | Markor | Markor | itself, as an app |
| Access from anywhere | no | no | **yes** |
| Graph / queries | backlinks panel | wikilinks | query language + Lua |
| Agent-safe with no app running | **yes** | **yes** | **yes** |
| Things that can break | nothing | nothing | server, TLS, sync conflicts |
| Long-form writing | fine | **best** | good |

---

## What I'd actually do

**Week 1:** Option A. Install VSCodium + Foam, Markor on the phone, Syncthing
between them. Zero servers. Build the habit first — the whole reason last attempt
failed was time spent on the tool instead of in it.

**Month 2, if the habit stuck:** add Option C. Put SilverBullet on an old laptop
or a €2-6 box, pointed at a git clone of the same vault. Keep Option A on the
Mac. You lose nothing by having both, because they're both just windows onto the
same files — and by then you'll know from use whether you actually want browser
access or were just attracted to the idea.

**If the habit didn't stick:** the front end was never the problem. Look at
capture friction instead — `vos jot` on the Mac and the Termux widget on Android
are the two things that decide whether this works.

---

## What is NOT in the running, and why

| Excluded | Reason |
|---|---|
| **Obsidian** | Closed source; sync is paid. Your call, and defensible. |
| **Logseq** | Split in April 2026. The Markdown version is now maintenance-only; the new version stores notes in a database and its Markdown export is **read-only**, so an agent can read but not write. |
| **Joplin** | Nice app, but notes live in SQLite. The "file sync" target is a sync mirror of item IDs, not your notes. Not agent-writable. |
| **Trilium, SiYuan, AppFlowy, Memos, Notesnook, Standard Notes** | All store notes in a database or encrypted blobs. Several explicitly warn that external file sync corrupts data. |
| **Anytype** | Licence is "source available", not open source. Encrypted object store, not files. |
| **Outline** | BSL licence — not open source. |
| **Dendron** | Development formally ceased in 2023. |
| **FUTO Voice Input** | Excellent, but "Source First" licence is not open source. Voxscribe (MIT) does the same job. |
| **MacroDroid / Tasker / Automate** | All proprietary. Termux + Termux:Widget (GPL-3.0) replaces them for our purposes. |
| **Nextcloud / Seafile** | Nextcloud's Android sync still cannot see files changed by other apps — a bug open since October 2024 — which breaks the entire point. Seafile's app is a library browser, not a folder syncer. |
