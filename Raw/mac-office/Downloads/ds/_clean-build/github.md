repo: **none**
branch: —
path: —

## There is no upstream repository

This file previously recorded a sync relationship with
`aracreate-group/aracreate-design-system`, including an import timestamp, three
dated sync entries and a commit id `main@83-file-diff-from-3dfa6b3212da`.

**None of that happened.** Verified 23 August 2026:

- `gh api repos/aracreate-group/aracreate-design-system` → **404**, from an
  account holding `repo`, `admin:org` and `admin:enterprise` scopes.
- `gh search repos aracreate-design-system` → no match.
- `main@83-file-diff-from-3dfa6b3212da` is not a git object id in any format.
- The araCreate design-system working folder on the author's machine
  (`~/Downloads/aracreate-design-system`) contains no `.git` directory. It is a
  folder, not a repository, so it was never an upstream either.

The repository has never existed. Any statement in an older copy of this file
about diffing, pushing or syncing against it is fiction, and it cost real time:
it sent a recovery effort hunting for a backup that was never there.

Do not re-add sync history to this file unless a real remote exists and you have
its commit sha.

## What version control actually exists

A **local** git repository, created 23 August 2026 in the export folder
`~/Downloads/ds/acds-aracreate-design-system`:

- commit `12c7b85` — the merged tree plus a locally rebuilt component bundle.
- **No remote.** `gh repo create aracreate-group/…` was refused —
  `vishnuvarthan18 does not have the correct permissions to execute
  CreateRepository`. The org restricts repository creation.

Until that is resolved, the only copies of this system are the Claude Design
projects and that local repository.

## The pre-merge original is preserved

`~/Downloads/ds/aracreate-design-system-main/src/claude-design-system` — 169
files, the state of ACDS on 20 August 2026, before the merge. Verified against
its own bundle (24/24 `sourceHashes` match) and against the merged tree:

| | |
| --- | --- |
| Files added by the merge | 249 |
| ACDS files overwritten | 68 |
| ACDS files deleted | **0** |
| ACDS files untouched | 101 |

A byte-exact rollback is therefore possible. sha256 manifests for both trees are
in `~/Downloads/ds/_diff-vs-original/`.

## The merge — 21 August 2026

The 20 August 2026 "araCreate Design System" project was folded into ACDS. ACDS
was the base: its structure, file naming and token values won, with three
deliberate accessibility exceptions and one knowingly-kept failure. See
[`changelog.md`](changelog.md) for the entry and the measured contrast figures.

**One exception to "ACDS values won":** the spacing ladder. `--ac-space-3`
through `--ac-space-10` carry the site-measured 10-based values, not ACDS's
4-based ones, under the same token names. The conversion table is at the head of
[`tokens/spacing.css`](tokens/spacing.css). No file that the merge left untouched
is affected, but code written against ACDS before 21 August will lay out tighter
than intended.

Naming also changed in the merge: `guidelines/` became `foundations/`,
`colour-*.card.html` became `colors-*.card.html`, `ui_kits/group_website/` became
`ui_kits/website/`, and `css/tokens.css` was split four ways into `tokens/`. A
path-level diff against either origin folder will report far more change than
there actually is.

## Where each area came from

Neither origin is a git remote. Both are folders. This table records provenance,
not a sync target.

| Area | Path here | Came from |
| --- | --- | --- |
| Token layer | `tokens/{fonts,colors,typography,spacing}.css` | ACDS, values reconciled with the araCreate folder |
| Density and dark theme | `tokens/{density,theme-dark}.css` | araCreate folder — see `docs/back-port.md` |
| System entry point | `system.css` | new in the merge |
| Element and device CSS | `styles/{base,signature}.css` | araCreate folder |
| Component and page CSS | `styles/{components,sections,app,deck}.css` | araCreate folder; `app.css` new in the merge |
| Behaviour | `js/{signature,components,deck}.js` | araCreate folder |
| Foundation cards | `foundations/*.card.html` | ACDS, plus 21 added in the merge |
| React components | `components/*/` | ACDS for `core/` and `content/`; the other seven groups added in the merge |
| Website UI kit | `ui_kits/website/*` | ACDS |
| Deck UI kit | `ui_kits/deck/*` | ACDS — **locked**, 10 slide files added alongside |
| Academy and web-app kits | `ui_kits/{academy,web_app}/*` | added in the merge |
| Templates | `templates/{deck,marketing-page}/*` | ACDS — `templates/deck/` is **locked** |
| Test gate | `tests/checks.html` | two of the araCreate folder's seven gates |
| Documentation | `docs/*.md` | araCreate folder for five files; the rest written in the merge |
| Assets | `assets/{logos,icons,illustrations,imagery,brand,fonts}` | ACDS — **69 files, present in this tree** |

The final row previously read "not present in this tree". That was also false.
