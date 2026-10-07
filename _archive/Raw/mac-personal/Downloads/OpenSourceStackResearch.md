# Open-source, zero-cost PKM stack — research report

**Date:** 21 August 2026
**Question:** with Obsidian ruled out on licence and cost grounds, what is the best
free, open-source way to run a plain-Markdown second brain on macOS + Android that
an AI agent can read and write?
**Method:** three parallel research sweeps (app/editor layer; sync and Android
clients; self-hosting, CLI and dashboard replacements), ~350 searches and page
fetches. Key claims spot-verified directly against primary sources — noted inline.
Unconfirmed items flagged **UNVERIFIED**.

---

## 1. The one test that decides everything

For an AI agent to be useful, the question is not "does this app have AI
features". It is: **can an agent read and write the notes with no app running?**

That is a property of *storage architecture*, not features, and it splits the
field cleanly in two.

**Passes — the canonical store is plain files on disk:**
SilverBullet · Zettlr · VSCodium + Foam/Marksman/Memo · Markor (Android) ·
Tolaria · OpenKnowledge · Files.md · zk · nb · obsidian.nvim · Quartz (publishing)
· Logseq "OG" (frozen)

**Fails — the canonical store is a database or encrypted blobs; any files are a
mirror or export:**
Joplin · Logseq 2.x · SiYuan · Trilium/TriliumNext · AppFlowy · Memos · Outline ·
Docmost · BookStack · Karakeep · Anytype · Standard Notes · Notesnook

The second group is not worse software. Several are excellent. They are
*categorically* excluded by the requirement.

---

## 2. Editor / app layer

| App | Licence | Store | macOS | Android | Server | Agent w/o app | Latest |
|---|---|---|---|---|---|---|---|
| **SilverBullet** | **MIT** ✅ | plain .md in a "Space" | browser/PWA | **PWA, installable** | required (Linux) | **yes, maintainer-blessed** | 2.8.1 (20 May 2026) confirmed on repo; 2.9.0 (11 Jun 2026) reported — **UNVERIFIED** |
| **VSCodium + Foam** | MIT + MIT ✅ | plain .md folder | native | — | none | **yes** | Foam 0.44.5 (26 Jul 2026) |
| **Zettlr** | GPL-3.0 ✅ | your .md folder | native | — | none | **yes** | 4.7.0, exact date **UNVERIFIED** |
| **Markor** (Android) | **Apache-2.0** ✅ | plain files, any folder | — | **native, excellent** | none | **yes** | **2.16.1 (20 Mar 2026)** — verified on F-Droid |
| **Tolaria** | AGPL-3.0 | .md + YAML, vault = git repo | native (Tauri) | **none** | none | yes | v2026-07-14 |
| **OpenKnowledge** (Inkeep) | GPL-3.0 | .md + git, MCP-native | native | none | none | yes | 0.14.0 (17 Jun 2026) |
| **Files.md** | MIT | plain .md | PWA / Go binary | server mode only† | optional | yes | **UNVERIFIED** |
| **Logseq OG** | AGPL-3.0 | plain .md | native | native | none | yes | 1.0.0-beta (15 Apr 2026) — **maintenance-only** |
| **Logseq 2.x** | AGPL-3.0 | **SQLite** | native | native | none | **no** — Markdown Mirror is read-only | ongoing |
| **Joplin** | AGPL-3.0 apps; **server licence non-OSI** | **SQLite** | native | native, good | optional | **no** | 3.5.13 (25 Feb 2026) |
| **TriliumNext** | AGPL-3.0 | SQLite | native | 3rd-party only | Docker | **no** | 0.104.1 (25 Jul 2026) |
| **SiYuan** | AGPL-3.0 | `.sy` JSON blocks | native | native | Docker | **no** — docs warn external sync **corrupts data** | **UNVERIFIED** |
| **Anytype** | **"Any Source Available License 1.0" — not OSI** ❌ | encrypted objects | native | native | optional | no | **UNVERIFIED** |
| **Outline** | **BSL 1.1 — not open source** ❌ | Postgres | web | iOS only | heavy | no | 1.9.1 (13 Jul 2026) |
| **Dendron** | Apache-2.0 | plain .md | VS Code | none | none | yes | **development ceased Feb 2023** |
| **DokuWiki** | GPL-2.0 | plain `.txt`, no DB | web | web | PHP | yes, but **not Markdown** | 2025-05-14 |

† The File System Access API that Files.md's local mode relies on is unsupported
on **every** mobile browser, so Android requires its server mode.

### 2.1 SilverBullet — verified details

Confirmed directly on the repo and docs: **MIT licence**; notes are "a collection
of Markdown Pages organized within a Space"; install via **single binary, Docker,
or local dev**; latest release shown on the repo **2.8.1, 20 May 2026**.

Two operational facts verified on the install page, both load-bearing:

- *"It is **highly discouraged** to run SilverBullet (in real use) on a **case
  insensitive** file system."* macOS's default APFS is case-insensitive →
  **run the server on Linux.**
- *"browsers require `https://` (or `localhost`) for SilverBullet's service
  worker, crypto, and clipboard APIs to work, so **you cannot** reach a remote
  SilverBullet server over plain `http://`"* → **TLS is mandatory**, not optional.

The decisive quote for our purposes, from maintainer Zef Hemel on editing the
Space directly from outside the app: *"It should be perfectly fine to do all the
things you mention on the FS directly."*
([community thread](https://community.silverbullet.md/t/managing-editing-files-outside-of-sb/3217))

Caveats: renaming a file outside the app **does not rewrite inbound wikilinks**;
concurrent edits are **not merged** — it creates a conflict copy and notifies you
([Sync docs](https://silverbullet.md/Sync)); and because the index is built
client-side, a ~1,300-note space takes ~40 s to index on desktop and ~3 minutes
on a phone. Resource use on the server is trivial: ~120 MB RAM, near-zero CPU.

Worth knowing: **SilverBullet+** is the maintainer's *proprietary* desktop app,
free for personal use, with a paid Pro tier for workplaces. The self-hosted MIT
server is unaffected and remains free — this is not the Obsidian pattern (no
paywall on the open product), but it is a monetisation vector to be aware of.

### 2.2 Logseq — the reason not to pick it

**24 April 2026: Logseq formally split into two products.** "Logseq OG" (the
Markdown/file version) moved to `github.com/logseq/og`, AGPL-3.0, latest 1.0.0
beta (15 Apr 2026), with an official commitment of *"security fixes and patches"*
plus dependency upgrades — explicitly *"maintenance and reliability rather than
new feature development"*. The product called Logseq is now the **SQLite** version.

Its "Markdown Mirror" writes a plain-Markdown projection to disk, but per the
[16 May 2026 update](https://discuss.logseq.com/t/whats-new-with-logseq-db-may-16th-2026/35020),
the two-way version *"is in active development on a feature branch and not yet
shipped"*. So an agent can read that mirror but **cannot write it**.

Note that most "best open-source Obsidian alternative" listicles still claim
Logseq stores plain Markdown. That is now only true of the frozen branch.

### 2.3 Joplin — why the good app still loses

Joplin's own architecture spec is unambiguous: *"All applications use a local
SQLite database to store notes, settings, cache, etc."* Markdown is the authoring
syntax, not the storage. The "file system" sync target writes a **sync mirror** —
one file per *item* named by item ID, with metadata appended — not a readable note
tree. Writing into it out-of-band means forging item metadata and racing lock
files. Also flagged: **Joplin Server ships under a non-OSI "Personal Use"
licence**, a partial source-available downgrade.

### 2.4 Newcomers worth watching

Three genuinely new plain-file, agent-native projects that the listicles missed:

- **Tolaria** (AGPL-3.0, by Luca Rossi of *Refactoring*) — desktop app where each
  vault *is* a git repo, plain .md + YAML, and an AGENTS file gives Claude Code
  persistent context across sessions. Purpose-built for this exact workflow.
  **No mobile.**
- **OpenKnowledge** (GPL-3.0, Inkeep) — MCP + git native, `AGENTS.md`, embedded
  agent terminal, WYSIWYG over plain Markdown on local disk. macOS-first, v0.14.0,
  VC-backed (so relicensing is a tail risk).
- **Files.md** (MIT) — plain .md, PWA plus a self-hostable single Go binary,
  opinionated default tree. Held back by the mobile-browser filesystem API gap.

None are mature enough to build on today, but Tolaria in particular is worth
re-checking in six months.

---

## 3. Sync — Mac ↔ Android, free and open source

| Option | Licence | Status | Conflicts | Verdict |
|---|---|---|---|---|
| **Syncthing** + **BasicSync** (Android) | MPL-2.0 / GPL-3.0 | active | conflict copies, no merge | **Recommended** |
| Syncthing + **Syncthing-Fork** | MPL-2.0 | active monthly, **custody churn** | same | OK from F-Droid; see below |
| **git** (Termux on Android) | GPL-3.0 | Termux stable 0.118.3 | true 3-way line merge | Strong fallback, better semantics |
| **Gitling** (Android git GUI) | GPL-3.0+ | 1.0.59 (10 Aug 2026) | JGit merge | Useful companion to Markor |
| Nextcloud | GPL-2.0 | active | server-side copies | **Broken for this** — see below |
| Seafile / Seadroid | AGPL-3.0+ | active | server-side copies | Library browser, not folder sync |
| rclone bisync | MIT | active | rich `--conflict-*` flags | Scriptable, sharp edges, needs an endpoint |
| FolderSync / Autosync | **proprietary** | — | — | Rejected |

### 3.1 The Syncthing Android situation, verified

Verified on Syncthing's own [community contributions
page](https://docs.syncthing.net/users/contrib.html): the official
`syncthing/syncthing-android` is listed under *"Older, Possibly Unmaintained"* and
marked **"Archived on 2024-12-03"**. The two apps Syncthing now points to are
**Syncthing-Fork** and **BasicSync**.

The custody history matters for an app that holds all-files access:

- Oct/Dec 2024 — official app discontinued and archived.
- ~Dec 2025 — `Catfriend1/syncthing-android` was **redirected to a `researchxxl`
  account** and the signing key transferred. Per
  [HN discussion](https://news.ycombinator.com/item?id=46184730), the receiving
  account was roughly three weeks old, with no prior announcement and no key
  rotation. Comparisons to the xz incident were made.
- Dec 2025 — an independent `nel0x` fork appeared and was **archived within three
  weeks**.
- 2026 — researchxxl ships monthly; the repo issue where the takeover was
  discussed has since been deleted.

F-Droid states its builds for the current package are *"built and signed by the
original developer, and guaranteed to correspond to this source tarball"* — i.e.
reproducible-build verification is intact. But reproducibility proves the binary
matches the source, not that the source is benign. **Hence the recommendation:
BasicSync** (GPL-3.0, chenxiaolong — a long-standing Android FOSS maintainer),
which runs the same Syncthing engine and delegates configuration to Syncthing's
own web UI.

Two more practical notes: the package ID changed
(`...syncthingandroid` → `...syncthingfork`), so **in-place upgrade is
impossible** — export settings, uninstall, reinstall. And a v2 battery-drain
regression caused by Local Discovery was fixed in 2.0.14.2; anything ≥2.1.x is
past it.

### 3.2 Requirements and conflict behaviour

Android 11+ requires **"All files access"** (`MANAGE_EXTERNAL_STORAGE`) — SAF is
not viable for Syncthing's access pattern. The folder must live in normal shared
storage (e.g. `/storage/emulated/0/Documents/VOS`), **never** under
`Android/data`. Also: exclude the app from battery optimisation and enable vendor
auto-start, or sync stops when the screen locks.

Per [Syncthing's docs](https://docs.syncthing.net/users/syncing.html), the loser
of a conflict is renamed `<file>.sync-conflict-<date>-<time>-<modifiedBy>.<ext>`;
the **older modification time loses**; conflict copies are normal files and
propagate everywhere; and **nothing is ever merged**.

The mitigation used in the starter kit is `git merge-file --union` on journal-type
files, which is always correct for append-only notes, plus
`10-Journal/**/*.md merge=union` in `.gitattributes` for the git path.

### 3.3 Why not Nextcloud

Disqualified by an open bug, not by philosophy. Nextcloud Android issue
[#13752](https://github.com/nextcloud/android/issues/13752), open since 9 Oct
2024: *"When creating, modifying, or deleting a file or folder outside the
Nextcloud app, the app does not recognize the changes."* Since the entire
workflow is "another app edits the file", that's fatal. Issue #16352 (Jan 2026)
adds that subfolders are ignored.

---

## 4. Android editing and capture

**Editor: Markor** — verified on F-Droid: **Apache-2.0, v2.16.1, 20 March 2026**,
and per its own description *"compatible with any other plaintext software on any
platform — edit with notepad or vim, filter with grep"*, with the notebook folder
settable to any location. It is the only FOSS Android candidate combining
frontmatter awareness, `[[wikilinks]]`, recursive content search, checkboxes,
home-screen widgets and share-into.

**Automation: there is no credible FOSS general-purpose automation app in 2026.**
MacroDroid, Tasker and Automate are all proprietary; Easer (GPL-3.0) has been
dormant since September 2022. The replacement is **Termux + Termux:Widget**
(GPL-3.0 both): scripts in `~/.shortcuts/tasks/` with `0700` permissions become
home-screen buttons or launcher shortcuts, and run as background tasks. All Termux
apps must come from the same install channel — mixing F-Droid and GitHub builds
breaks signature compatibility.

**Voice, offline and open source:**

| App | Licence | Latest | Notes |
|---|---|---|---|
| **Voxscribe** | **MIT** | 1.0.1 (31 Jul 2026) | Whisper keyboard, **no INTERNET permission at all** |
| **WhisperType** | MIT | 1.5.1 (19 Aug 2026) | Full keyboard + dictation; fetches models from HF |
| **Sayboard** | GPL-3.0 | 4.2.1 (Sep 2024) | Vosk; **implements the system RecognitionService**, which is what makes offline STT work inside Termux |
| **Transcribro** | ISC | v7 (Aug 2025) | Whisper + VAD, English only, not on F-Droid |
| **FUTO Voice Input** | **"Source First" — not OSI** ❌ | — | Excellent, excluded on licence |

---

## 5. Replacing the Obsidian CLI

**The reframing that matters:** Obsidian's own docs state *"Obsidian CLI requires
the Obsidian app to be running"* — verified. It is a remote control for a GUI, not
a headless tool. It could never have run on a server or in a cron job. So this is
not a downgrade.

| Tool | Licence | Latest | Covers |
|---|---|---|---|
| **`zk`** | GPL-3.0 | v0.15.6 (**26 Jul 2024** — no release in 2 years) | Closest single match: daily notes, FTS, tags, **backlinks, orphans, broken links** |
| **`nb`** | AGPL-3.0 | 7.25.4 (28 Apr 2026) | Notes + bookmarks + git sync; weaker graph queries |
| **`ripgrep`** | MIT / Unlicense | 15.2.0 (15 Jul 2026) | Search — the agent's primary tool |
| **`yq`** | MIT | v4.53.3 (6 Jun 2026) | Frontmatter reads *and writes* |
| **`jq`** | MIT | 1.8.2 (20 Jun 2026) | JSON shaping |
| **`marksman`** | MIT | release 8 Feb 2026 | LSP: broken-link diagnostics, rename refactoring — editor-side only |
| **`markdown-oxide`** | Apache-2.0 | v0.25.10 (2 Nov 2025) | PKM LSP; no standalone CLI |

Coverage against the Obsidian CLI: daily notes, create/read/move, search,
backlinks, links, orphans, tags, broken links — **all covered, several better**
(Obsidian has no broken-link command). Two genuine gaps: **no tool filters on
arbitrary frontmatter** (workaround: `zk list --format '{{json .}}' | jq
'select(...)'`, or the `vos ls` command in the kit), and **no safe scriptable
rename-with-link-rewrite** — which is the one worth a small script and a nightly
`vos check`.

Risk to note: `zk` has had no release since July 2024. Stable and triaged under
the `zk-org` organisation, but budget for it going quiet. The starter kit
therefore does not depend on it.

---

## 6. Replacing Bases (dashboards over frontmatter)

| Tool | Licence | Role |
|---|---|---|
| **`markdowndb` / `mddb`** | MIT | Purpose-built: indexes frontmatter, tags, **wikilinks (for backlinks)** and tasks from a Markdown folder into SQLite |
| **`sqlite-utils`** | Apache-2.0 | 3.39 (24 Nov 2025) — the glue: JSON/CSV in, SQL and FTS out |
| **`datasette`** | Apache-2.0 | 0.65.2 stable — browsable UI + JSON API over that SQLite, with canned queries |
| SilverBullet's own query language | MIT | Live queries embedded in a page — most maintainable *if* you run SilverBullet |
| Quartz / Eleventy / Astro / Hugo | MIT etc. | Only if you're already building a site |

**Most maintainable answer, and what the kit does:** generate dashboards as
**plain Markdown** into the vault on a nightly schedule. They then sync to your
phone, open in any editor, are readable by `grep` and by the agent, and are
versioned in git — none of which Bases could do. Add Datasette later only if you
want interactive exploration.

---

## 7. Local search and recall

| Tool | Licence | Latest | Apple Silicon |
|---|---|---|---|
| **`qmd`** | **MIT** | v2.5.3 (29 May 2026) | ✅ Metal GPU |
| **`ripgrep`** | MIT / Unlicense | 15.2.0 | ✅ |
| **`sqlite-vec`** | Apache-2.0 + MIT | v0.1.9 (31 Mar 2026) | ✅ pre-v1, breaking changes expected |
| **`llama.cpp`** / **Ollama** | MIT / MIT | Ollama v0.32.14 (15 Aug 2026) | ✅ |
| **`basic-memory`** | **AGPL-3.0** | v0.20 | ✅ MCP over plain Markdown + SQLite |
| **`khoj`** | **AGPL-3.0** | 2.0.0-beta.28 | ✅ still beta, heaviest |

All OSI-licensed and free. `qmd` is the pick: BM25 + vector + reranking in one
local tool, ~2 GB of models, MIT, ships its own MCP server, no service to run.

Worth keeping in mind: a May 2026 preprint found **plain grep beat vector search
on personal notes for every model tested**, because personal questions hinge on
verbatim spans — exact names, dates, preferences. Small sample, vendor-affiliated,
**UNVERIFIED** — but it justifies "grep first, semantic second".

---

## 8. Hosting, if you later want a server

| Option | Cost | Notes |
|---|---|---|
| **Old laptop / Mac mini + Cloudflare Tunnel** | **$0** + ~$10/yr domain | Outbound-only, no open ports, TLS at the edge, no reverse proxy needed. **Best $0 answer.** |
| **netcup VPS piko** | ~€1.54/mo net | Cheapest credible box, 1 GB RAM — enough for SilverBullet's ~120 MB |
| **Contabo Cloud VPS 10** | ~€4.50/mo | 8 GB RAM, best RAM per euro |
| **Hetzner CAX11, IPv6-only** | ~€5.99/mo net | 2 ARM vCPU / 4 GB; skip the €0.50 IPv4 since the tunnel is outbound-only |
| **Oracle Cloud Always Free** | $0 | 2 OCPU / 12 GB ARM — but **halved from 4/24 in June 2026 without announcement**, capacity is hard to obtain, and idle accounts are *"eligible for suspension or termination"* after 30 days. **Fine as a replica, not as your only copy.** |

Also: **Hetzner raised prices sharply on 15 June 2026** — CPX/CCX lines more than
doubled (CPX22 €7.99 → €19.49). CX/CAX are the only sane Hetzner lines now.

---

## 9. Publishing, if you want it

**Quartz 5** (MIT, v5.0.0, 24 May 2026) is the clear winner: native wikilinks and
transclusions, native backlinks, and an `ExplicitPublish` mode that publishes
**only** notes marked `publish: true`. Config moved from TypeScript to YAML, so no
coding needed. Host free on **Cloudflare Pages** (500 builds/month, 20,000
files) from a **private** repo.

⚠️ One security caveat, quoted from its docs: *"all non-markdown files will be
emitted and available publically in the final build."* The publish flag filters
notes, not assets. Keep publishable images in a separate folder.

Hugo is the wrong fit — wikilink support was closed as *"not planned"*
([issue #13533](https://github.com/gohugoio/hugo/issues/13533)).

---

## 10. Verdict

**VSCodium + Foam (Mac) + Markor (Android) + Syncthing/BasicSync + git + `vos` +
ripgrep + qmd.** Every component MIT/Apache/GPL, total recurring cost $0, nothing
to host, and the notes remain plain files an agent can operate on at any time.

**Add SilverBullet on a Linux box in month two** if you decide you want one
interface everywhere and browser access from any machine. It is the only
self-hosted option that keeps notes as plain files *and* whose maintainer
explicitly supports outside-the-app editing — which is why it's the upgrade path
rather than a competitor.

---

## 11. Flagged as unverified

- SilverBullet 2.9.0 (11 Jun 2026) — repo showed 2.8.1 (20 May 2026) as latest on
  fetch; both are recent, the discrepancy doesn't change anything.
- Zettlr 4.7.0's exact release date (download page and GitHub releases disagree).
- Exact filename scheme of Joplin's file-system sync target (SQLite-canonical is
  confirmed; per-file layout is not documented in primary docs).
- Latest versions for SiYuan, AppFlowy, Memos, Karakeep, Notesnook, Anytype.
- Star counts for Tolaria and OpenKnowledge (reported values implausible).
- Files.md release history; GitJournal's activity status (appears stalled).
- Whether `termux-speech-to-text` honours a non-Google RecognitionService on
  Android 14+ — test Sayboard before relying on the voice-into-Termux path.
- Whether Markor requests all-files access on Android 13+ or falls back to SAF
  (use its file-browser "mount" option if it can't see the synced folder).
- "50 free Cloudflare Zero Trust seats" — could not confirm on cloudflare.com.
- The "grep beats vector search" preprint — small n, vendor-affiliated, not replicated.

---

## Primary sources verified directly during this pass

[SilverBullet repo](https://github.com/silverbulletmd/silverbullet) ·
[SilverBullet Install](https://silverbullet.md/Install) ·
[SilverBullet Sync](https://silverbullet.md/Sync) ·
[Editing files outside SilverBullet](https://community.silverbullet.md/t/managing-editing-files-outside-of-sb/3217) ·
[Syncthing community contributions](https://docs.syncthing.net/users/contrib.html) ·
[Syncthing conflict handling](https://docs.syncthing.net/users/syncing.html) ·
[Markor on F-Droid](https://f-droid.org/packages/net.gsantner.markor/) ·
[Obsidian CLI docs](https://obsidian.md/help/cli) ·
[Obsidian pricing](https://obsidian.md/pricing) ·
[Joplin architecture spec](https://joplinapp.org/help/dev/spec/architecture/) ·
[Logseq DB update, May 2026](https://discuss.logseq.com/t/whats-new-with-logseq-db-may-16th-2026/35020) ·
[Nextcloud Android #13752](https://github.com/nextcloud/android/issues/13752) ·
[Syncthing-Android custody, HN](https://news.ycombinator.com/item?id=46184730) ·
[Foam](https://github.com/foambubble/foam) ·
[qmd](https://github.com/tobi/qmd) ·
[zk](https://github.com/zk-org/zk) ·
[Quartz](https://github.com/jackyzha0/quartz) ·
[Oracle Always Free resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm) ·
[Hugo wikilinks #13533](https://github.com/gohugoio/hugo/issues/13533)
