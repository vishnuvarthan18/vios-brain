# PROMPT — repair ACDS

Replaces Prompt B. Do not run the old nine-file Prompt B: its preconditions no
longer match, and it would leave ACDS broken.

```
Use the DesignSync MCP tool. Target project:
4716e773-3175-4bc2-a22e-f34c179aea34 (ACDS — owned by Ara, org default, published).

CONTEXT YOU NEED
A previous run restored 165 files in this project to pre-merge content, but its
companion deletion of 249 files was refused (owner-only). The project is now half
of one system and half of another: 34 tokens are undefined, 9 files reference
them, and 25 more files render at the wrong size. The restore cannot be
completed, so it must be reversed. This task makes the project one coherent
system again.

SOURCE
/Users/vishnuvarthanvenkatapathy/Downloads/ds/_clean-build

STEP 0 — prove the source is good before touching anything
  cd ~/Downloads/ds/_clean-build && python3 _check.py
It must print ALL CHECKS PASSED and exit 0. If it does not, stop and report.

STEP 1 — build the upload list
Walk the source folder and take every file EXCEPT:
  - CLAUDE.md                    (the API rejects this path)
  - _ds_bundle.js, _ds_manifest.json, _adherence.oxlintrc.json   (app-generated)
  - any file or folder whose name starts with "." (.thumbnail, .DS_Store, .git)
  - _check.py, _verify.html, _changes.html                       (my harness)
Expect exactly 411 files. If the count is not 411, stop and report it.

STEP 2 — confirm the target
  get_project on 4716e773-…  Confirm canEdit is true.
  list_files. Confirm tokens/spacing.css exists.
  get_file tokens/spacing.css. It MUST currently contain "--ac-space-10: 128px"
  — that is the broken pre-merge version this task repairs. If it instead
  contains "--ac-space-16", the project has already been repaired: STOP, change
  nothing, and report that.

STEP 3 — plan
  finalize_plan with:
    localDir = /Users/vishnuvarthanvenkatapathy/Downloads/ds/_clean-build
    deletes  = []            <-- MUST be empty. Never call delete_files here.
    writes   = these 17 patterns:
        SKILL.md
        changelog.md
        github.md
        readme.md
        styles.css
        system.css
        thumbnail.html
        assets/**
        components/**
        docs/**
        foundations/**
        js/**
        styles/**
        templates/**
        tests/**
        tokens/**
        ui_kits/**

STEP 4 — write
  write_files using localPath for every file (never inline data).
  Max 256 files per call, so split into two calls under the same planId.
  If ANY single write is rejected: stop, report which path and the exact error.
  Do not retry, do not widen the plan, do not delete anything.

STEP 5 — verify
  Re-read these four and confirm the merged token layer is back:
    tokens/spacing.css     must contain  --ac-space-16: 140px
    tokens/colors.css      must contain  --ac-success: #186a43
    tokens/typography.css  must contain  --ac-leading-h1
    tokens/fonts.css       must contain  .ac-wordmark
  And confirm these merge-only files are still present and unchanged:
    styles/base.css, styles/sections.css, tokens/density.css, tokens/theme-dark.css

  Then confirm the locked deck items are untouched — sha256 first 12 chars:
    ui_kits/deck/slides.jsx          must still match what it was before you started
    templates/deck/Deck.dc.html      must still match what it was before you started
  (read them with get_file BEFORE step 4 and compare after.)

REPORT
  a. the _check.py result
  b. the file count from step 1
  c. what tokens/spacing.css contained before you wrote
  d. how many files written, and any rejection with its exact error text
  e. the step 5 confirmations
  f. the locked-file hashes, before and after

Do not call delete_files. Do not write CLAUDE.md. Do not touch any other project.
```

## After it runs

- Open ACDS once in Claude Design so it rebuilds the bundle and card index.
- Paste `CLAUDE.md` by hand from `~/Downloads/ds/_clean-build/CLAUDE.md`.
- The 249 extra files stay. Nothing here removes them and nothing can, without Ara.
