# Clean build, then mirror — plan and agent prompts

**23 August 2026** · source of truth: `~/Downloads/ds/_clean-build/` (418 files)

---

## What this actually changes

**Nine files, all corrections of statements that are demonstrably false.** Plus
`CLAUDE.md`, which the API will not accept and has to be pasted by hand.

Nothing is added. Nothing is deleted. No token value moves. No component
changes. Verified below.

| File | What was wrong |
|---|---|
| `tokens/spacing.css` | Header claimed "Sizes are in the name, so `--ac-space-7` is unambiguous". `--ac-space-7` is 20px. Replaced with the truth plus a full ACDS→current conversion table. |
| `docs/decisions.md` | Same false claim — "with the pixel size in the token name". |
| `readme.md` | Called the ladder "ACDS's indexed ladder". It is not — it replaced ACDS's on 21 Aug. Also named a GitHub repo that has never existed as the "Primary repository". |
| `github.md` | Rewritten. Recorded an import timestamp, three dated sync entries and commit `main@83-file-diff-from-3dfa6b3212da` against a repository that does not exist. Also claimed assets were "not present in this tree" — there are 69 of them. |
| `changelog.md` | An "Open after this merge" block listing three defects, **none of which are real**: `styles.css`, `brand-icons.card.html` and `assets/` are all present, and `tests/checks.html` resolves 65 of 65 paths. Replaced with what is genuinely open. |
| `docs/components.md` | "core — 15 components". There are 16. |
| `styles/app.css` | Comment pointed at `guidelines/decisions.md`; that folder is `docs/` since the merge. |
| `components/core/Icon.jsx` | Comment pointed at `guidelines/api-audit.md`. Same. |
| `ui_kits/web_app/README.md` | Same dead `guidelines/` pointer. |
| `CLAUDE.md` | Named the nonexistent repo as the home of the source PDFs. **Manual paste — reserved path.** |

### Why the spacing values were not reverted

`--ac-space-5` means 24px in ACDS and 15px now. Reverting it would fix the
documentation and simultaneously break every one of the 249 components authored
against the current ladder. The names were made honest instead, and a conversion
table added for anyone porting pre-21-August code.

---

## Verification

| Check | Result |
|---|---|
| Files differing from the verified export | **10** — exactly the ones above |
| `tokens/spacing.css`, `styles/app.css`, `Icon.jsx` with comments stripped | **byte-identical** to the originals — comment-only edits |
| Headless render, all cards | **66 / 66**, 0 JS errors, `__ds_errors` empty, 0 blank |
| Remaining references to the nonexistent repo | 3 — all of them the corrections saying it doesn't exist |
| Remaining dead `guidelines/` or `group_website/` pointers in code | **0** |

### Pre- and post-image hashes (sha256, first 12)

```
changelog.md                 91eb30257153 -> 08acaf0289b6
components/core/Icon.jsx     e1227538df0c -> ecb108542586
docs/components.md           2c6cdb93922e -> eccd0923b28f
docs/decisions.md            6dd28ba6828f -> b291d06ea134
github.md                    f0d4c4ca3474 -> 5d84cf65cd8f
readme.md                    ad52da3916f1 -> 693e67926edd
styles/app.css               1b9a6f6b2569 -> 190a2e1fd830
tokens/spacing.css           905bf2c8907f -> 3ceee16f9abf
ui_kits/web_app/README.md    f1c3620b82be -> 3967da18dc0e
CLAUDE.md                    aa00ac4a5c1f -> 2da2e1e1d2c8   (manual paste)
```

---

## Order of operations

1. **Plan A** — your own project `22f6bdb1`. Nine writes.
2. Open it in Claude Design so the app rebuilds the bundle and manifest. Confirm the cards still render.
3. Paste `CLAUDE.md` by hand.
4. **Plan B** — mirror the same nine files into ACDS `4716e773`, only once step 2 has confirmed step 1 was clean.

Do not run B before A. If something is wrong with the corrections, it should be
wrong in your project, not in the published org default.

---

# PROMPT A — your own project

```
Use the DesignSync MCP tool. Target project: 22f6bdb1-5dfd-4e1c-9b53-c098478ea6e8
("araCreate Design System — merged"). Local source directory:
/Users/vishnuvarthanvenkatapathy/Downloads/ds/_clean-build

Write exactly these nine files, and nothing else:

  changelog.md
  components/core/Icon.jsx
  docs/components.md
  docs/decisions.md
  github.md
  readme.md
  styles/app.css
  tokens/spacing.css
  ui_kits/web_app/README.md

PRECONDITIONS — check all of these before writing. If any fails, stop and
report; do not proceed and do not improvise a fix.

1. get_project on 22f6bdb1-…  Confirm canEdit is true and that the owner is
   Vishnu, not Ara. If the owner is not Vishnu you have the wrong project — stop.
2. list_files. Confirm all nine paths already exist in the project. Do not
   create any of them.
3. For each of the nine, get_file and compute sha256. The first 12 hex
   characters must match the BEFORE column:
       changelog.md              91eb30257153
       components/core/Icon.jsx  e1227538df0c
       docs/components.md        2c6cdb93922e
       docs/decisions.md         6dd28ba6828f
       github.md                 f0d4c4ca3474
       readme.md                 ad52da3916f1
       styles/app.css            1b9a6f6b2569
       tokens/spacing.css        905bf2c8907f
       ui_kits/web_app/README.md f1c3620b82be
   A mismatch means the project has drifted from the snapshot these
   corrections were built against. Stop and report which file and its actual
   hash. Do not overwrite it.

THEN:

4. finalize_plan with localDir set to the clean-build directory above,
   writes = exactly those nine paths, deletes = [] (empty — this plan deletes
   nothing).
5. write_files using localPath for each, reading from the clean-build
   directory. Do not use inline data.
6. Re-read all nine with get_file and confirm sha256[:12] now matches:
       changelog.md              08acaf0289b6
       components/core/Icon.jsx  ecb108542586
       docs/components.md        eccd0923b28f
       docs/decisions.md         b291d06ea134
       github.md                 5d84cf65cd8f
       readme.md                 693e67926edd
       styles/app.css            190a2e1fd830
       tokens/spacing.css        3ceee16f9abf
       ui_kits/web_app/README.md 3967da18dc0e

Report: the nine before-hashes you observed, the nine after-hashes, and any
precondition that failed. Do not touch any other project. Do not call
delete_files. Do not write CLAUDE.md — the API rejects that path.
```

---

# PROMPT B — mirror into ACDS

**Run only after Prompt A has completed and you have opened the merged project
and confirmed the cards render.**

```
Use the DesignSync MCP tool. Target project: 4716e773-3175-4bc2-a22e-f34c179aea34
("ACDS"). This project is owned by Ara, is the organisation default, and is
published. Treat it accordingly: this task writes nine documentation and
comment corrections and nothing else.

Local source directory:
/Users/vishnuvarthanvenkatapathy/Downloads/ds/_clean-build

Same nine files as before:

  changelog.md
  components/core/Icon.jsx
  docs/components.md
  docs/decisions.md
  github.md
  readme.md
  styles/app.css
  tokens/spacing.css
  ui_kits/web_app/README.md

HARD CONSTRAINTS:

- deletes MUST be an empty list. Never call delete_files against this project.
- Do not write any path outside the nine listed.
- Do not write CLAUDE.md.
- If any precondition fails, stop and report. Do not improvise.

PRECONDITIONS:

1. get_project. Confirm the projectId is 4716e773-3175-4bc2-a22e-f34c179aea34.
2. list_files. Confirm all nine paths exist.
3. get_file each of the nine; sha256[:12] must equal the BEFORE hashes:
       changelog.md              91eb30257153
       components/core/Icon.jsx  e1227538df0c
       docs/components.md        2c6cdb93922e
       docs/decisions.md         6dd28ba6828f
       github.md                 f0d4c4ca3474
       readme.md                 ad52da3916f1
       styles/app.css            1b9a6f6b2569
       tokens/spacing.css        905bf2c8907f
       ui_kits/web_app/README.md f1c3620b82be
   If ACDS has been edited since 22 August these will not match. That is a
   stop condition, not something to work around.

THEN finalize_plan (writes = the nine paths, deletes = []), write_files with
localPath, and verify the AFTER hashes:
       changelog.md              08acaf0289b6
       components/core/Icon.jsx  ecb108542586
       docs/components.md        eccd0923b28f
       docs/decisions.md         b291d06ea134
       github.md                 5d84cf65cd8f
       readme.md                 693e67926edd
       styles/app.css            190a2e1fd830
       tokens/spacing.css        3ceee16f9abf
       ui_kits/web_app/README.md 3967da18dc0e

Report before-hashes, after-hashes, and anything that failed.
```

---

## After both runs

- **Paste `CLAUDE.md`** into each project by hand from
  `~/Downloads/ds/_clean-build/CLAUDE.md`. The API refuses that path.
- **Open each project once** so the app recompiles `_ds_bundle.js`,
  `_ds_manifest.json` and `_adherence.oxlintrc.json`. Until you do, the card
  index and component count stay stale — this is what made ACDS look unchanged
  for two days.
- **The 249 merge-added files remain in ACDS.** Nothing here removes them;
  bulk delete needs the owner. That is unchanged and not addressed by this plan.
