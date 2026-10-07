---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, setup]
---

# 1. Mac setup (30 minutes, all free and open source)

## Put the vault somewhere sane

```bash
mkdir -p ~/VOS
# copy the contents of this starter kit into ~/VOS, then:
cd ~/VOS
git init && git add -A && git commit -m "V OS day zero"
```

Git is not optional. It is your undo button for everything an agent does.

## Install the tools

```bash
# Homebrew, if you don't have it — https://brew.sh
brew install ripgrep fd jq yq git
```

| Tool | Licence | What it's for |
|---|---|---|
| ripgrep | MIT / Unlicense | Fast text search. Your agent's primary tool. |
| fd | MIT / Apache-2.0 | Finding files |
| jq | MIT | Reading JSON output |
| yq | MIT | Reading and writing frontmatter from scripts |
| git | GPL-2.0 | Version history, undo, sync to a remote |

## Put `vos` on your PATH

```bash
echo 'export PATH="$HOME/VOS/bin:$PATH"' >> ~/.zshrc
echo 'export VOS_VAULT="$HOME/VOS"'      >> ~/.zshrc
source ~/.zshrc
vos help
```

`vos` is pure Python 3 standard library — macOS already has Python 3. No pip
install, no dependencies, nothing to break.

## Pick an editor (see the front-end comparison doc)

**Option A — VSCodium** (MIT, the de-branded build of VS Code):

```bash
brew install --cask vscodium
```

Then in VSCodium: Extensions → search **Foam** (MIT, from the Open VSX registry,
which VSCodium uses by default) → install. You now have wikilinks,
autocompletion, backlinks, and link-updating when you rename a note.

**Option B — Zettlr** (GPL-3.0, a purpose-built Markdown writing app):

```bash
brew install --cask zettlr
```

Point it at `~/VOS`. Nicer for long-form writing, fewer developer features.

Both edit the same plain files. You can install both and switch freely — that is
the entire point of owning plain text.

## Optional but excellent: semantic search

```bash
brew install node        # if you don't have it
npm install -g @tobilu/qmd
qmd collection add ~/VOS --name vos
qmd embed                # downloads ~2 GB of models, once, runs entirely local
qmd query "what did we decide about retainer pricing"
```

`qmd` (MIT) does keyword + vector search + reranking, all on your machine,
nothing sent anywhere. Index everything, not just "important" notes — semantic
search pays most on old notes you have forgotten writing.

## Quick capture on a hotkey

Add to `~/.zshrc`:

```bash
jot() { vos jot "$*"; }
```

Then `jot had an idea about the pricing page` from any terminal. For a global
hotkey without installing anything closed-source, use macOS Shortcuts:
*Run Shell Script* → `$HOME/VOS/bin/vos jot "$1"` with an *Ask for Text* input,
then assign a keyboard shortcut.
