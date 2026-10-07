# PROMPT — correct the "repo does not exist" claim (3 files)

```
I got something wrong and it is written into this project as verified fact.
Correct it. Three files. Do not change anything else.

WHAT IS TRUE
The repository EXISTS and is private:
    https://github.com/aracreate-group/aracreate-design-system
    branch: main   path: src/claude-design-system

An earlier check ran `gh api` from an account without access and got 404.
GitHub returns 404 — not 403 — for private repositories you cannot see. That
404 was read as proof of absence. It was not. The conclusion drawn from it was
wrong, and the sync history that was deleted on the strength of it was most
likely genuine.

=====================================================================
FILE 1 — github.md
=====================================================================
Replace everything from the first line ("repo: **none**") down to and including
the line ending "...the Claude Design projects and that local repository."
with exactly this:

repo: **aracreate-group/aracreate-design-system**
branch: main
path: src/claude-design-system

## Upstream

https://github.com/aracreate-group/aracreate-design-system

The repository is private. `gh api` returns **404** to accounts without access —
that is how GitHub hides private repositories. A 404 here means "not visible to
you", not "does not exist".

### Correction, 23 August 2026

An earlier version of this file stated that this repository "has never existed"
and called its recorded sync history fiction. **That was wrong.** The claim came
from reading a 404 as proof of absence. The import timestamp and the three dated
sync entries that used to be here were most likely genuine, and were deleted in
error. They are not recoverable from this file — see the repository's own commit
history instead.

### How ACDS is kept in git — manual, as of 23 August 2026

1. Export ACDS from Claude Design as a zip.
2. Delete `acds/src/claude-design-system/` in the local clone.
3. Unpack the export in its place.
4. Review the diff, then commit and push to `main`.
5. In the ACDS chat in Claude Design, use **+ → sync from upstream**; it verifies
   the changes and updates the docs.

To be automated later.

**Direction of truth:** ACDS the Claude Design project is where edits are made;
git is the record. Anything wrong in ACDS reaches the repository on the next push.

## Other copies

- A local git repository in the export folder
  `~/Downloads/ds/acds-aracreate-design-system`, commit `12c7b85`, **no remote**.
  `gh repo create` was refused for lack of org permission. It is a snapshot, not
  a clone of the upstream.

--- end of replacement ---

Then, further down that same file, find the two lines that read:
    "Neither origin is a git remote. Both are folders. This table records
    provenance, not a sync target."
Replace them with:
    "ACDS's upstream is the repository above, under `src/claude-design-system/`.
    The araCreate working folder had no repository of its own. This table records
    provenance."

Leave the rest of github.md alone — the merge notes, the spacing-ladder note,
the pre-merge rollback figures and the provenance table are all still correct.

=====================================================================
FILE 2 — readme.md
=====================================================================
Find the line under "### Sources" that begins "**There is no source
repository.**" Replace that whole paragraph with:

Primary repository: **https://github.com/aracreate-group/aracreate-design-system** (`src/claude-design-system/`, branch `main`) — private; ACDS is kept in sync with it by the manual procedure recorded in [`github.md`](github.md). An earlier version of this line claimed the repository did not exist; that was a misread of a 404 from an account without access, and it was wrong. The source PDFs and deck page renders live in that repo's `uploads/` folder and are not copied into this project.

=====================================================================
FILE 3 — CLAUDE.md
=====================================================================
Find the bullet beginning "Source PDFs and deck renders are intentionally not
stored in this project". Replace that whole bullet with:

- Source PDFs and deck renders are intentionally not stored in this project; their content is absorbed into tokens, foundations, components and UI kits. Originals live in `aracreate-group/aracreate-design-system` under `src/claude-design-system/uploads/`. The repository is private — a 404 from `gh api` means you lack access, not that it is missing. A note claiming it never existed was added on 23 Aug 2026 in error and has been removed.

=====================================================================
CHECK
=====================================================================
Grep the whole project for these phrases and report every remaining hit:
    "has never existed"
    "There is no source repository"
    "repo: **none**"
    "no upstream repository"
There should be none outside the correction notes themselves.

Report each file you changed and quote the new text back.
```
