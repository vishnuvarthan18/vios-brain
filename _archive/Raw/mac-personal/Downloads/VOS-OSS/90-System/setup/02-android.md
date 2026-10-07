---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, setup]
---

# 2. Android setup (all free, all open source)

Two apps do everything: one to sync the folder, one to edit it. Plus Termux if
you want two-second capture from the home screen.

## Install F-Droid first

Get F-Droid from [f-droid.org](https://f-droid.org). It matters: F-Droid builds
from source and verifies that the app you install matches the published code.
Prefer it over the Play Store for everything below.

## The editor: Markor

**Markor** — Apache-2.0, v2.16.1 (March 2026), on F-Droid.

It is a plain-text editor that browses a real folder. Set its notebook to your
synced vault folder and you are done. It has what actually matters: YAML
frontmatter awareness, `[[wikilinks]]`, recursive content search, checkboxes,
a QuickNote home-screen widget, and share-into (send a link or text from any app
straight into a note).

Set **notebook folder** to `/storage/emulated/0/Documents/VOS`.

## The sync: Syncthing

Peer-to-peer, no server, no account, no cost, end-to-end encrypted, works over
your local Wi-Fi or the internet. Nothing of yours touches anyone's cloud.

**Which Android app:** the official Syncthing Android app was archived in
December 2024. Syncthing's own documentation now points to two community apps:

- **BasicSync** (GPL-3.0, by chenxiaolong) — minimal, long-standing maintainer.
  **Recommended.**
- **Syncthing-Fork** — more features, but the project changed maintainers twice
  in 2025-2026 and the handover was not well announced. Install from F-Droid only.

Either is fine. BasicSync is the more conservative choice for an app that will
hold your business notes and needs all-files access.

**Setup:**

1. Mac: `brew install syncthing && brew services start syncthing`, then open
   <http://127.0.0.1:8384>.
2. Add folder `~/VOS`, type **Send & Receive**.
3. Turn on **File Versioning → Simple, keep 10** on the Mac. Second undo button.
4. Android: install the app, grant **All files access**, exclude it from battery
   optimisation, and enable auto-start if your phone has that setting.
5. Pair the devices (scan the QR code), accept the folder, set its path to
   `/storage/emulated/0/Documents/VOS`.
6. Point Markor at that same path.

**Non-negotiables:** the folder must live in normal shared storage, never inside
`Android/data`. And run exactly **one** sync tool — never Syncthing *and* git
sync on the same folder at the same time.

**Conflicts:** Syncthing does not merge. If two devices edit the same file
offline it keeps both and renames the loser
`filename.sync-conflict-<date>-<time>-<device>.md`. Run
`90-System/scripts/deconflict.sh` on the Mac to union-merge those automatically
for journal-style files.

## Two-second capture: Termux

Install **Termux**, **Termux:API** and **Termux:Widget** — all GPL-3.0, all from
**F-Droid** (they must all come from the same source or the signatures won't match).

```bash
termux-setup-storage
pkg install termux-api jq
mkdir -p ~/.shortcuts/tasks && chmod 700 ~/.shortcuts ~/.shortcuts/tasks
cp /storage/emulated/0/Documents/VOS/90-System/scripts/termux-jot.sh ~/.shortcuts/tasks/jot.sh
chmod 700 ~/.shortcuts/tasks/jot.sh
```

Long-press your home screen → Widgets → Termux:Widget → pick `jot.sh`. Tap it,
type a line, done — appended to today's daily note whether or not any notes app
is running.

## Voice capture, offline and open source

- **Voxscribe** (MIT, F-Droid) — a push-to-talk dictation keyboard running
  Whisper on-device. It requests no internet permission at all.
- **Sayboard** (GPL-3.0, F-Droid) — install it too and set it as your system
  speech recogniser; that makes offline voice work inside Termux, so you can
  dictate into the capture script.

Note: FUTO Voice Input is excellent but its licence is "Source First", not open
source. Excluded on purpose.

## Web clipping

Use **Firefox for Android** (Chrome on Android cannot run extensions at all) and
either the **MarkDownload** extension, or share the URL to Termux and let
`~/bin/termux-url-opener` fetch and convert it. Markor's share-into is the
zero-setup option for saving a link with a note.
