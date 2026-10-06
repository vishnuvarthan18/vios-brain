repo: aracreate-group/aracreate-design-system
branch: main
path: src/claude-design-system

# GitHub Repository

Private. https://github.com/aracreate-group/aracreate-design-system — branch
`main`, subtree `src/claude-design-system/`. Verified by reading the remote on
1 September 2026.

## Source of truth

**ACDS (this project) is the master.** The repository is the backup and audit
trail. Changes flow from here to the repository by export; nothing is imported
back. Settled by Vishnu, 24 August 2026.

**Still outstanding on the repo side:** its `README.md` and `src/readme.md` say the
opposite. Corrected copies are in `docs/export/`; the export procedure copies them
over. Until that lands, anyone reading the repo first draws the wrong conclusion.

## Rules

- "The repository" means `origin/main`. This project cannot see it. Every field
  describing it stays `UNVERIFIED` until Claude Code sends a sha and says whether
  it was pushed. A local commit is not a push; a bug report is not a ship report.
- Before every export, run the gate in `docs/export/EXPORT.md`.
- The zip carries `docs/export/state.json`; Claude Code asserts against it.

## How to export

`docs/export/EXPORT.md` — download this project, replace the subtree, copy the two
repo READMEs, set `VERSION`, commit, push, paste the commit sha below. A 404 from
an unauthorised client means no access, not no repo.

## Last sync

date: 2026-09-01T18:54:13Z
version: v2.0.1 here · UNVERIFIED on the remote
         (v2.0.0 committed locally by Claude Code as 9ef7587e1e4ab8d6de95e95c52d8974b2b89d2b3, unpushed;
          last remote read 1 Sep 2026 showed v1.1.0 at tree d697544)
commit: UNVERIFIED (none pushed yet)
files: 359 here (the export) · 393 on the pre-v2.0.0 remote at tree d697544

**Nothing imported.** The remote was last read 1 Sep 2026 at tree d697544, the
same hash as 24 August. **Remote state is UNVERIFIED** until Claude Code sends a
sha with a push confirmation; only that message fills `commit:` and the
changelog's "In the repo" column.

### The diff, this project vs repository

- Repo has, this project has removed: `styles.css`, `components/content/` (7),
  `templates/marketing-page/` (4), `ui_kits/` (44 files in four kits).
- This project has, repo lacks: `tokens.css`, `docs/history.md`,
  `docs/deck-conventions.md`, `docs/export/`, `templates/web/`, `templates/app/`.
- Modified: all six token files, `system.css`, `styles/deck.css`, `styles/base.css`,
  `templates/deck/*`, six foundation cards, `tests/checks.html`, every root doc,
  four `docs/*.md`, the three generated files.

### Updated in this project

- Wrote `docs/export/EXPORT.md` (procedure, delete list, breaking changes).
- Wrote corrected repo `README.md` and `src/readme.md` into `docs/export/`.
- Retired `docs/back-port.md`; its checklist is complete here.
- Confirmed the repo tree hash is unchanged since 24 August.
- 1 Sep, later: applied Claude Code's export rules — UNVERIFIED markers, generic
  token gate, `state.json`, HANDOFF.md dropped from the tree.

## Sync history

### 2026-08-24T10:10:03Z

v1.2.0 here · v1.1.0 in the repository. Remote compared; `tokens/colors.css`,
`CLAUDE.md`, `styles.css`, `system.css` byte-identical. Nothing imported. File
count corrected to 393 (an earlier 166 was a capped listing).

### 2026-08-24T05:15:13Z

Read the remote to verify existence; captured root `README.md` and `src/readme.md`.
Corrected `github.md`, `readme.md` and `CLAUDE.md`, which had claimed the repo did
not exist on the strength of a 404. Flagged the repo README's master claim.

## Screen map

| Area in this project | Repo files |
| --- | --- |
| Whole system | `src/claude-design-system/**` — same tree, replaced wholesale on export |
| Repo-level docs | `README.md`, `src/readme.md` — sourced from `docs/export/*.repo.md` |
| Release tooling | `Makefile`, `VERSION` — repo only; `VERSION` is set at export |

## Local copies (23–24 August 2026)

- `~/Downloads/ds/acds-aracreate-design-system` — local git, commit `12c7b85`, no
  remote configured. Superseded by the export procedure; point it at the repo or
  discard it.
- `~/Downloads/ds/aracreate-design-system-main/src/claude-design-system` — the
  pre-merge ACDS of 20 August 2026 (169 files), with sha256 manifests in
  `~/Downloads/ds/_diff-vs-original/`. Byte-exact rollback of the 21 August merge
  remains possible from there.

## Provenance of the 21 August 2026 merge

Moved to [`docs/history.md`](docs/history.md).
