# Export — ACDS → aracreate-design-system

Repository: `aracreate-group/aracreate-design-system`, branch `main`, subtree
`src/claude-design-system/`. Current version here: see `state.json`. Remote state
is UNVERIFIED from this side; Claude Code reports it after each commit.

**Direction:** this project → repo. The subtree is replaced wholesale; nothing is
merged back. `../../github.md` is the record; this file is the procedure.

## Gate — run before every export

Mechanical, whole tree, no hand-written name list.

```sh
# every var(--ac-*) reference resolves to a definition
grep -rhoE 'var\(--ac-[a-zA-Z0-9-]+' . | sed 's/var(//' | sort -u > /tmp/refs
grep -rhoE '\-\-ac-[a-zA-Z0-9-]+[[:space:]]*:' . | sed 's/[[:space:]]*:$//' | sort -u > /tmp/defs
comm -23 /tmp/refs /tmp/defs
# only acceptable output: --ac-bar-pct --ac-progress --ac-slider-pct --ac-value --ac-vw
# (set from JS at runtime; every CSS use carries a fallback)

# no retired name in a live file (prose in changelog/history may mention them)
grep -rnE -- '--ac-(black|gray-900|gray-700|gray-600|white|weight-semibold|slide-foot-lg|canvas-warm)\b' \
  --include='*.css' --include='*.js' --include='*.jsx' --include='*.html' .   # nothing

# counts match _ds_manifest.json and state.json; version line agrees everywhere
jq '{components:(.components|length),cards:(.cards|length),templates:(.templates|length),tokens:(.tokens|length)}' _ds_manifest.json
grep -m1 '^\*\*Version' readme.md; grep -m1 '^\*\*Current version' changelog.md
find . -type f | wc -l     # equals state.json fileCount
```

The v2.0.0 export shipped with `styles/app.css` reading a removed token because
the check then was a hand-typed file list. This gate is the replacement.

## Procedure

```
# 1. download this project (zip) and unpack next to a clone of the repo
git -C aracreate-design-system checkout main && git pull
rm -rf aracreate-design-system/src/claude-design-system
mkdir  aracreate-design-system/src/claude-design-system
cp -R  acds-aracreate-design-system/. aracreate-design-system/src/claude-design-system/
# the zip carries no uploads/ and no HANDOFF.md; docs/export/state.json is the handshake

# 2. repo-level files (they still say the repo is the master)
cp acds-aracreate-design-system/docs/export/README.repo.md      aracreate-design-system/README.md
cp acds-aracreate-design-system/docs/export/src-readme.repo.md  aracreate-design-system/src/readme.md
jq -r .version acds-aracreate-design-system/docs/export/state.json > aracreate-design-system/VERSION

# 3. commit
cd aracreate-design-system
git add -A
git commit -m "ACDS v<version> - <one line from changelog.md>"
git push origin main
git rev-parse HEAD     # report back: sha, pushed yes/no, gate results
```

`rm -rf` then copy is deliberate: `git add -A` records the deletions below and
the renames as renames. Do not hand-merge.

## What comes back

Claude Code sends the sha, whether it was pushed, and the gate results. Only
then are `github.md` → `commit:`, the changelog's "In the repo" column and
`state.json` → `remote` filled. Until then they read `UNVERIFIED`.

## What the v2.0.0 diff showed

**Deleted (56 files)**
- `styles.css`
- `components/content/ (7 files)`
- `templates/marketing-page/ (4 files) — replaced by templates/web/`
- `ui_kits/academy/ (5 files)`
- `ui_kits/deck/ (16 files) — folded into templates/deck/`
- `ui_kits/web_app/ (6 files) — folded into templates/app/`
- `ui_kits/website/ (17 files) — folded into templates/web/`

**Added**
- `tokens.css`
- `docs/history.md`
- `docs/deck-conventions.md`
- `docs/export/`
- `templates/web/ (Web.dc.html, README.md, ds-base.js, support.js, .thumbnail)`
- `templates/app/ (App.dc.html, Dashboard.jsx, Jobs.jsx, Settings.jsx, SignIn.jsx, README.md, ds-base.js, support.js, .thumbnail)`
- `templates/deck/.thumbnail`
- `.thumbnail`

**Modified** — every token file, `system.css`, `readme.md`, `changelog.md`,
`CLAUDE.md`, `SKILL.md`, `github.md`, `thumbnail.html`, the six foundation
cards that linked `styles.css`, `styles/deck.css`, `styles/base.css`,
`templates/deck/*`, `tests/checks.html`, `docs/decisions.md`,
`docs/contributing.md`, `docs/back-port.md`, and the regenerated
`_ds_bundle.js` / `_ds_manifest.json` / `_adherence.oxlintrc.json`.

## Breaking for anyone reading the repo

- `styles.css` → `tokens.css`.
- `--ac-black`, `--ac-gray-900/700/600` → `--ac-graphite-gray`; `--ac-white` → `--ac-canvas`;
  `--ac-weight-semibold` and `--ac-slide-foot-lg` gone.
- `ServiceCard` → `Card kind="service"`; `StatBlock` → `Stat`.
- Bundle namespace is `AraCreateDesignSystem_4716e7` (the repo README still says `_9b36ca`).

