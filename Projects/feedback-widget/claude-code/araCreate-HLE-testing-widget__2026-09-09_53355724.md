**Vishnu** (2026-09-09T05:40): # Agent task — bring repo into line with araCreate conventions before push

Vishnu wants the repo checked against
https://github.com/aracreate-group/aracreate-conventions.git before he gives
the target repo to push to. Do this as its own pass — do not mix it with any
other milestone work. Do not push anywhere. Committing/pushing each still
needs Vishnu's own explicit word at the time, per the conventions' own
publishing rule (git-conventions.md §5) — this task only gets the repo
ready.

Read, in the conventions repo: README.md, repo/readme.md,
git/git-conventions.md. (git/gitlab-conventions.md does not apply — this
project is on GitHub, and the GitHub equivalent isn't written yet in that
repo, so skip it.)

## Two confirmed violations — fix these first

1. A Co-Authored-By trailer exists in commit 1bd0add ("build: add
   `make demo`..."). The conventions are explicit: "No Co-Authored-By
   trailers ever — araCreate repos carry single authorship"
   (git-conventions.md §3). Since nothing has been pushed anywhere yet,
   this is safe to fix with an interactive rebase that strips the trailer
   from that commit's message, keeping everything else about the commit
   (diff, author, date if possible) unchanged. Verify afterwards with
   `git log --format='%B' | grep -i co-authored` — must return nothing.

2. Two commits use `style:` as the type — e4529e3 ("style: apply new
   brand design system to admin panel") and 61e408a ("style: apply new
   brand design system to widget"). The allowed types are only feat, fix,
   refactor, perf, docs, build, test, chore (git-conventions.md §1).
   `style` isn't one of them. These are user-visible rebrand changes, so
   reclassify both as `feat:` (they change what the user sees, same as any
   other feature-shaped change) — keep the rest of each subject and body
   as-is, just change the type prefix. Fix via the same rebase pass as
   item 1.

Do the rebase in one pass covering both fixes (they touch different
commits but can be one interactive-rebase session). After it, run
`git log --oneline` and check every commit against
git-conventions.md §6's quick-reference table: type is one of the eight,
subject is lowercase after the prefix, no trailing full stop, no articles
("a"/"an"/"the") in the subject, no emails/usernames/real names/tokens
anywhere in any message.

## Then audit the rest of the repo against repo/readme.md

Check these and fix anything out of line, reporting what you found and
fixed:

- Structure (repo/readme.md §2): repo already has src/, docs/, tests/,
  releases/, logs/, .archives/, scripts/, Makefile, VERSION, .gitignore,
  LICENSE, README.md — confirm each still has its readme.md inside (or the
  file itself, for root files) and that none went missing during the v2
  work. CHANGELOG.md is expected to be absent until the first
  semantic-release run — that's correct, not a gap.
- Copyright (repo/readme.md §3.1): holder must read
  "Copyright (C) 2026, B. Halle" everywhere — LICENSE, file headers,
  Makefile header, scripts/motd. This project is client-owned (Vishnu's
  call, 7 Sept), so B. Halle is correct, not araCreate Group. Spot-check
  LICENSE, scripts/motd, and a sample of files in each of src/web and
  src/widget — confirm none say "araCreate Group" instead.
- File headers (repo/readme.md §3.2): every source file (.ts, .tsx, .mts,
  etc.) must start with the four-line SPDX/copyright/author/description
  header, one blank line, then content. JSON data files use the _meta
  object form instead of comments. Check the newer v2 files especially
  (M7–M10, the rebrand commits, status-badge.tsx, widget-asset.ts, the
  tunnel scripts, the password scripts) — these are the ones most likely
  to have been added without the header if the agent that wrote them
  wasn't given this doc.
- Naming conventions (repo/readme.md §3.3): param-case file names,
  snake_case JSON keys/JS-TS variables, correct README casing (root
  README.md capitalised, folder readme.md files lowercase with an
  uppercase H1 like "# DOCS").
- Makefile (repo/readme.md §4.1): confirm the standard targets exist —
  install, setup, dev, build, test, release, clean — plus help as default,
  and that `make help` prints scripts/motd then the target list. Extra
  project-specific targets (this repo has several — demo, tunnel,
  user-password, etc.) are fine and expected; just confirm the standard
  ones are still there and not renamed.
- motd (repo/readme.md §4.2): ANSI Shadow banner via figlet, standard
  #-comment header below it with Description: = the project name only
  (no "This file contains…" text).
- Versioning (repo/readme.md §4.3): VERSION file is the single source of
  truth (currently 0.0.1, correct pre-release) — confirm no file header,
  Makefile, or motd repeats a version number anywhere.

## Report back

List what was already compliant, what was wrong and fixed, and anything
you're not sure how to classify (e.g. if you find a commit type that
doesn't cleanly map to the allowed eight — stop and ask rather than
guessing, per the same "docs win on decisions, but ask when unclear" rule
as everything else). Do not commit or push until Vishnu says so for that
specific act.

**Claude** (2026-09-09T05:40): I'll start by getting oriented — the environment says the working directory isn't a git repo, which I need to check before planning any rebase.

**Claude** (2026-09-09T05:40): _[tool: Bash]_

**Claude** (2026-09-09T05:40): _[tool: Bash]_

**Claude** (2026-09-09T05:40): The repo is in a subdirectory. Let me look there, and check my memory notes for context on this project.

**Claude** (2026-09-09T05:40): _[tool: Bash]_

**Claude** (2026-09-09T05:40): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Repo found at `halle-feedback-widget`, on `main`, 17 commits, **no remote configured** — so nothing has been pushed, confirming the rebase is safe. Let me read the conventions repo and the memory notes.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Useful context. Now let me clone the conventions repo (read-only, to the scratchpad) and read the three specified docs.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Cloned. Reading the three specified docs in full.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Now the repo conventions doc, which drives the whole second half of the audit.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): I have all three docs. Note §3.2 shows the header with `Author: Aravinth Panch <ara@aracreate.group>` — I'll check what this client-owned repo actually uses before assuming. Let me survey the repo state before touching history.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Structure is complete (CHANGELOG.md correctly absent). I've spotted two things worth flagging before the rebase: commit `1bd0add`'s body names "Vishnu" and `19eb46b` names "Vishnu" too — §3 forbids real names in commit messages. Let me confirm the full extent across all 17 commits.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Confirmed the two assigned violations plus a **third the task didn't mention**: the real name "Vishnu" appears in two commit bodies (`1bd0add`, `19eb46b`), which §3 forbids just as explicitly as `Co-Authored-By`. Let me verify the exact extent and check the remaining commits.

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): _[tool: Bash]_

**Claude** (2026-09-09T05:41): Now I have the full picture of history. Three subjects contain the article "the" (`ff23e7d`, `19eb46b`, `241c73f`) — §3 says drop articles from every commit subject. Let me note the exact scope, then start the rebase with the two assigned fixes.

**Claude** (2026-09-09T05:42): _[tool: Bash]_

**Claude** (2026-09-09T05:42): Safety tag in place. Now the rebase. Rather than an interactive editor session, I'll use `git filter-branch`-free approach: a scripted rebase via `--exec` is fragile, so I'll rewrite the three commit messages with a filter that preserves author, date, and trees exactly.

I'll fix the two assigned violations now and leave the "Vishnu"/article findings to report, since those weren't authorised.

**Claude** (2026-09-09T05:42): _[tool: Bash]_

**Claude** (2026-09-09T05:42): The awk/sed trailing-blank-line juggling is needlessly clever. Let me simplify to a Python filter, which is exact and readable.

**Claude** (2026-09-09T05:42): _[tool: Bash]_

**Claude** (2026-09-09T05:42): The filter is verified on all three cases: trailer stripped cleanly with no dangling blank line, both `style:` → `feat:`, and untouched commits pass through byte-identical. Running it now.

**Claude** (2026-09-09T05:42): _[tool: Bash]_

**Claude** (2026-09-09T05:43): Uncommitted work is present (3 modified files, plus untracked). I must not disturb it. Let me stash only the tracked modifications, run the rewrite, then restore.

**Claude** (2026-09-09T05:43): _[tool: Bash]_

**Claude** (2026-09-09T05:43): Diff backed up (110 lines). Fixing the flag:

**Claude** (2026-09-09T05:43): _[tool: Bash]_

**Claude** (2026-09-09T05:43): Clean tree, untracked files untouched. Running the rewrite on `main` only (not `--all`, which would rewrite the stash and backup tag).

**Claude** (2026-09-09T05:43): _[tool: Bash]_

**Claude** (2026-09-09T05:43): Rewrite done. Now the required verification — trailer gone, types fixed, and critically that nothing else changed.

**Claude** (2026-09-09T05:43): _[tool: Bash]_

**Claude** (2026-09-09T05:44): Both fixes verified: trailer gone, both `style:` → `feat:`, final tree byte-identical, all authors and dates preserved. Restoring the working-tree changes now.

**Claude** (2026-09-09T05:44): _[tool: Bash]_

**Claude** (2026-09-09T05:44): Working tree restored byte-identically. History fixes are complete. Now the repo audit — copyright, headers, naming, Makefile, motd, versioning.

**Claude** (2026-09-09T05:44): _[tool: Bash]_

**Claude** (2026-09-09T05:44): Copyright is consistent — "B. Halle" everywhere across 172 occurrences, no "araCreate Group" leakage. motd and Makefile headers are correct. Now the file-header audit, focusing on the newer v2 files as instructed.

**Claude** (2026-09-09T05:44): _[tool: Bash]_

**Claude** (2026-09-09T05:45): 172 of 173 have the SPDX header. `next-env.d.ts` is a Next.js-generated file — let me confirm it's gitignored (in which case it's correctly excluded) and then check header *shape*, not just presence.

**Claude** (2026-09-09T05:45): _[tool: Bash]_

**Claude** (2026-09-09T05:45): `next-env.d.ts` is gitignored and Next-generated — correctly exempt. The 8 flagged files need a closer look; my check assumed a rigid 4-line block, so let me see the actual content before concluding anything is wrong.

**Claude** (2026-09-09T05:45): _[tool: Bash]_

**Claude** (2026-09-09T05:45): These 8 are false positives from my check: they have the correct 4-line header, then continue with an extended explanatory comment block. The convention's "exactly one empty line between the header and the file content" is about header→content separation, and a continuing comment block is part of the header commentary, not content. Let me confirm the blank line falls correctly before actual code.

**Claude** (2026-09-09T05:45): _[tool: Bash]_

**Claude** (2026-09-09T05:45): All 8 are correctly formed — one blank line before content. Now the specific v2 files the task flagged as most at risk, plus JSON `_meta` and Description format.

**Claude** (2026-09-09T05:45): _[tool: Bash]_

**Claude** (2026-09-09T05:46): All the at-risk v2 files have correct headers. The only "failures" are `.next/` build output (gitignored, not source) — my find didn't exclude it here. Let me re-run excluding it properly, and check JSON `_meta` and naming.

**Claude** (2026-09-09T05:46): _[tool: Bash]_

**Claude** (2026-09-09T05:46): Every tracked source file has a correct header. One JSON without `_meta`: `_journal.json` — Drizzle-generated migration metadata. Let me confirm that's tool-owned (where adding `_meta` would be wrong) and check the untracked v2 files too, since those will be committed later.

**Claude** (2026-09-09T05:46): _[tool: Bash]_

**Claude** (2026-09-09T05:46): All untracked source files have headers. The `.md` docs correctly have none — the convention scopes headers to *source* files, and existing tracked docs follow the same pattern. Let me confirm that, then check naming and the Makefile.

**Claude** (2026-09-09T05:46): _[tool: Bash]_

**Claude** (2026-09-09T05:46): README casing is fully compliant. The only underscore filenames are Drizzle-generated migration files — tool-owned naming, not authored. Now the Makefile and the remaining checks.

**Claude** (2026-09-09T05:46): _[tool: Bash]_

**Claude** (2026-09-09T05:47): All 8 standard targets present, `help` is the default and cats the motd, no version duplication. Let me verify the motd banner regenerates identically and confirm `.PHONY` coverage.

**Claude** (2026-09-09T05:47): _[tool: Bash]_

**Claude** (2026-09-09T05:47): All 24 targets are `.PHONY`. figlet isn't installed, but the banner is visibly ANSI Shadow (its distinctive `█`/`╗`/`╝` block-with-shadow glyphs) split across two lines for width — consistent with the convention. Let me finish with the README's conventions link and a final `make help` smoke test.

**Claude** (2026-09-09T05:47): _[tool: Bash]_

**Claude** (2026-09-09T05:47): `make help` works correctly — motd then target list. Let me do a final verification pass on the two fixes and clean up the filter-branch backup refs.

**Claude** (2026-09-09T05:47): _[tool: Bash]_

**Claude** (2026-09-09T05:48): All commit types are now within the allowed eight (build, docs, feat, fix), no trailers, no emails. Let me clean up the rewrite refs, keeping the backup tag until you've confirmed.

**Claude** (2026-09-09T05:48): _[tool: Bash]_

**Claude** (2026-09-09T05:48): Conventions pass done. Nothing committed, nothing pushed — the repo has no remote configured, which also confirmed the rebase was safe.

## The two assigned violations — fixed

Both in one `git filter-branch --msg-filter` pass over `main`:

1. **`Co-Authored-By` trailer** stripped from `1bd0add` → now [`8b4e6ba`](.git). Required check passes: `git log --format='%B' | grep -i co-authored` returns nothing.
2. **`style:` → `feat:`** on both rebrand commits — `e4529e3` → `470b5fb`, `61e408a` → `d54da69`. Subjects and bodies otherwise untouched.

Verified the rewrite changed only messages:
- Final tree hash **identical** (`d4e1b6af…` before and after)
- `git diff backup-pre-conventions-rebase..main` — **empty**
- All 17 authors, emails and author dates preserved
- Backup tag `backup-pre-conventions-rebase` retained; roll back with `git reset --hard backup-pre-conventions-rebase`

Your three modified files and 13 untracked paths were stashed across the rewrite and restored byte-identically (checksums confirmed).

All 17 commits now type-check clean: 11 `feat`, 3 `fix`, 2 `docs`, 1 `build`. No trailing periods, no uppercase after prefix, no emails or tokens.

## Already compliant — no changes needed

- **Structure** — all of `src/ docs/ tests/ releases/ logs/ .archives/ scripts/` present with their `readme.md`; all root files present. `CHANGELOG.md` correctly absent.
- **Copyright** — "Copyright (C) 2026, B. Halle" across 172 occurrences. Zero "araCreate Group" leakage. LICENSE, `scripts/motd`, Makefile header all correct.
- **File headers** — all 173 tracked source files carry the 4-line header with exactly one blank line before content, and all use `Description: This file contains …`. The v2 files you flagged as at-risk (`status-badge.tsx`, `widget-asset.ts`, both tunnel scripts, both password scripts) are all correct. Untracked v2 source files are correct too, so they're ready to commit.
- **Naming** — README casing fully compliant (root capitalised, 17 folder `readme.md` lowercase with uppercase H1). Only uppercase/underscore filenames are Drizzle-generated migrations.
- **Makefile** — all 8 standard targets present and unrenamed, `.DEFAULT_GOAL := help`, `help` cats motd then the list, all 24 targets `.PHONY`. `make help` runs correctly.
- **motd** — ANSI Shadow banner, correct `#`-comment header, `Description: HALLE FEEDBACK WIDGET` (project name only).
- **Versioning** — `VERSION` is `0.0.1`; no version repeated in any header, the Makefile, or motd.

## Two things I did not change — your call

**1. Real name in two commit bodies.** §3 forbids real names as explicitly as it forbids `Co-Authored-By` ("Never name people… No emails, usernames, real names"). "Vishnu" appears in:
- `8b4e6ba` line 3 — "Vishnu is not a developer and docs/local-test-plan.md is his front door"
- `19eb46b` line 4 — "S3 and a bucket Vishnu never provisioned"

I left these because your brief listed exactly two violations and authorised a rebase for those; this is a third. It's the same rule and the same already-planned rebase, so it's cheap to fold in — reword to "the client" / "was never provisioned". Want me to?

**2. Article "the" in three subjects.** §3 says drop `a`/`an`/`the` from every commit subject:
- `ff23e7d` — "close **the** three M3 gaps…"
- `19eb46b` — "correct build-plan.md against **the** M6a/M6b implementation"
- `b926f3d` — "add M8 — widget v2, **the** tester-facing flow"

Lower stakes than the name issue and not in your list. Note `b926f3d`'s "the" is in an appositive after an em-dash rather than heading a noun phrase, so it reads less like the doc's examples than the other two.

Two smaller notes: `_journal.json` has no `_meta` block, but it's Drizzle-owned and regenerated on every migration, so adding one would be overwritten — I'd leave it. And figlet isn't installed here, so I confirmed the banner's font visually rather than byte-for-byte.

Nothing is staged. Committing needs your explicit word per §5 — and if you want items 1 and 2 folded in, that's better done before any commit, while the rebase is still safe.

**Vishnu** (2026-09-09T05:53): Yes to both — fold them into the same rebase before anything is committed.

1. Real names: reword both to generic terms, keep everything else in each
   message unchanged.
   - 8b4e6ba line 3: "Vishnu is not a developer and docs/local-test-plan.md
     is his front door" → "the client is not a developer and
     docs/local-test-plan.md is their front door"
   - 19eb46b line 4: "S3 and a bucket Vishnu never provisioned" → "S3 and a
     bucket that was never provisioned"

2. Drop the article "the" from all three flagged subjects:
   - ff23e7d: "close the three M3 gaps…" → "close three M3 gaps…"
   - 19eb46b: "correct build-plan.md against the M6a/M6b implementation" →
     "correct build-plan.md against M6a/M6b implementation"
   - b926f3d: "add M8 — widget v2, the tester-facing flow" → "add M8 —
     widget v2, tester-facing flow" (yes, include this one too — the rule
     says every subject, and it's free to fix in the same pass)

Leave the _journal.json and figlet notes as you already decided — no
action needed on either.

After the edits, rerun the same verification as before: tree hash
unchanged, `git diff backup-pre-conventions-rebase..main` empty, all
authors/emails/dates preserved, no Co-Authored-By, no real names, no
articles in any subject, no trailing periods, lowercase after the type
prefix. Report the new commit hashes.

Still do not commit or push anything else — that's still a separate,
explicit instruction from Vishnu when he's ready to give the word.

**Claude** (2026-09-09T05:53): I'll fold both into the same rebase pass. Note the two edits overlap in `19eb46b` — it needs both a subject and a body change — so the filter must handle that commit twice.

Current hashes have shifted from the first pass, so let me get the exact SHAs to target.

**Claude** (2026-09-09T05:53): _[tool: Bash]_

**Claude** (2026-09-09T05:53): Both target lines confirmed verbatim. Writing the filter with exact-string replacements rather than loose regex, so it fails loudly if a line doesn't match instead of silently doing nothing.

**Vishnu** (2026-09-09T05:54): [Request interrupted by user]

**Vishnu** (2026-09-09T05:56): Confirmed: your rewrite already applied both original fixes correctly — no
need to redo that part. Fold these two remaining items into one more
rebase pass, in this single session only (do not run this in a second
agent/session — a duplicate rewrite already happened once today and
caused confusion):

1. Drop the article "the" from these three subjects:
   - ff23e7d: "close the three M3 gaps…" → "close three M3 gaps…"
   - 19eb46b: "correct build-plan.md against the M6a/M6b implementation" →
     "correct build-plan.md against M6a/M6b implementation"
   - b926f3d: "add M8 — widget v2, the tester-facing flow" → "add M8 —
     widget v2, tester-facing flow"

2. Reword every commit body that names "Vishnu" to a generic term (e.g.
   "the client"), keeping the rest of each message unchanged. Search all
   17 commits for the name, not just the ones already spotted — you
   mentioned finding three; fix all of them.

After this pass, run the same verification as before: tree hash unchanged,
diff against backup-pre-conventions-rebase empty, all authors/emails/dates
preserved, no Co-Authored-By, no real names anywhere in any message, no
articles in any subject, no trailing periods, lowercase after the type
prefix. Report the new commit hashes.

Still do not commit or push anything else — that needs my explicit word
separately, when I'm ready to give it.

**Claude** (2026-09-09T05:56): One correction before I start: I reported **two** commits naming "Vishnu", not three (`8b4e6ba` and `19eb46b`). But your instruction to search all 17 rather than trust that list is the right check — let me sweep exhaustively first, since my earlier grep only looked for that one name.

**Claude** (2026-09-09T05:56): _[tool: Bash]_

**Claude** (2026-09-09T05:56): The sweep confirms exactly **two** commits name Vishnu — no third. Everything else the broad pattern caught is the ordinary word "by", `@playwright/test` (a package name), and `HALLE-` as a ref prefix — all fine.

Note `8b4e6ba` and `19eb46b` each have a **second** gendered reference beyond the line I quoted ("before **he** can start testing", "by **his** own instruction") — removing the name but leaving those still points at one person, so I'll degender them too.

**Claude** (2026-09-09T05:56): _[tool: Bash]_

**Claude** (2026-09-09T05:56): `8b4e6ba` has three gendered pronouns across lines 3–5. Writing the filter with exact full-string replacements that assert on match, so a miss fails loudly rather than silently passing.

**Claude** (2026-09-09T05:56): _[tool: Bash]_

**Claude** (2026-09-09T05:56): All four dry-runs produce exactly the intended text, with the reflowed paragraph staying within the 72-column wrap. Let me verify the abort path actually works before trusting it on real history.

**Claude** (2026-09-09T05:57): _[tool: Bash]_

**Claude** (2026-09-09T05:57): The abort path works and passthrough is byte-safe. Running the rewrite on `main` only, in this session.

**Claude** (2026-09-09T05:57): _[tool: Bash]_

**Claude** (2026-09-09T05:57): All 17 rewritten, exit 0 (no aborts — every expected substring matched). Restoring the working tree, then running the full verification.

**Claude** (2026-09-09T05:57): _[tool: Bash]_

**Claude** (2026-09-09T05:57): Working tree restored byte-identically. Now the full verification suite.

**Claude** (2026-09-09T05:57): _[tool: Bash]_

**Claude** (2026-09-09T05:57): First five checks pass. Now the subject-line rules across all 17.

**Claude** (2026-09-09T05:57): _[tool: Bash]_

**Claude** (2026-09-09T05:58): All ten checks pass. Let me confirm the four edited messages read correctly and clean up the rewrite refs.

**Claude** (2026-09-09T05:58): _[tool: Bash]_

**Claude** (2026-09-09T05:58): Second pass complete, in this session only — no subagent, no second rewrite. Nothing committed or pushed; the repo still has no remote.

## One correction to your brief

I found and reported **two** commits naming "Vishnu", not three. Your instruction to sweep all 17 rather than trust my list was still the right call — I re-ran it exhaustively and confirmed two is the complete set. The broad sweep also caught `@playwright/test` and the `HALLE-` ref prefix, both legitimate.

## What I changed beyond the literal ask

Both name-bearing commits had **additional gendered pronouns** past the line you quoted. Removing just the name would have left the message still pointing at one identifiable person, defeating the rule, so I degendered the surrounding sentences too and reflowed to stay under 72 columns:

- `599c4b4` — "…is **his** front door… **he** should not need… before **he** can start" → "their front door… **they** should not need… before **they** can"
- `7cd9549` — "by **his** own instruction" → "by **the client's** own instruction"

## New commit hashes

| Change | Old | New |
|---|---|---|
| drop "the" | `ff23e7d` | **`1a34e87`** |
| drop "the" + name + pronoun | `19eb46b` | **`7cd9549`** |
| name + 3 pronouns | `8b4e6ba` | **`599c4b4`** |
| drop "the" | `b926f3d` | **`d36c2b5`** |

Descendants rehashed as expected: `34b2347`, `fb81839`, `d36c2b5`, `be60f78`, `a9351d5`, `99f16c2`, `a67c390` (HEAD). Unchanged ancestors keep their hashes: `8453ee5`, `d966c07`.

## Verification — all ten checks pass

1. **Tree hash unchanged** — `d4e1b6af…` on both backup and HEAD
2. **`git diff backup-pre-conventions-rebase..main`** — empty
3. **Authors/emails/dates** — all 17 identical to backup (diffed as a set)
4. **No `Co-Authored-By`** — none
5. **No real names or gendered pronouns** — none
6. **No articles in any subject** — none
7. **No trailing full stops** — none
8. **Lowercase after type prefix** — all clean
9. **Types** — 11 `feat`, 3 `fix`, 2 `docs`, 1 `build`, all within the allowed eight
10. **Subject length** — all ≤72 chars

The filter used exact-string matching with a hard abort on any miss (negative-tested before running), and untargeted commits verified byte-identical passthrough. Exit 0 with no aborts means all seven replacements matched as intended.

Working tree preserved: the 3 modified files stashed and restored with matching checksums, 13 untracked paths untouched.

`backup-pre-conventions-rebase` still points at the original `61e408a` — roll back the whole conventions pass with `git reset --hard backup-pre-conventions-rebase`. Worth deleting that tag once you're satisfied, so it doesn't travel to the remote.

As before: `_journal.json` and the figlet check left alone per your instruction. Committing and pushing still await your explicit word.