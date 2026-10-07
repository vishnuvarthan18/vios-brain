---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: ACDS — araCreate Design System merge

## 1. What this project is
- **Goal:** Merge two Claude Design design systems into one. ACDS (`acds-aracreate-design-system`) is the base. The final result must follow ACDS structure, naming and values.
- **Who it is for / client:** [[Companies/araCreate Group]] — "the main one for the whole company". Others in the company use it to make designs across tools.
- **Why it exists:** There were two systems: ACDS (team system, imported 17 Aug 2026, owned by Ara) and the newer "araCreate Design System" that Vishnu built on 19–20 Aug (see [[Projects/aracreate/SUMMARY]]). The company needs one single source of truth.
- "ACDS" = araCreate Design System (confirmed in chat: project name `acds-aracreate-design-system`, namespace `AraCreateDesignSystem_4716e7`).

## 2. Status now (as of 2026-10-06)
- Reopened (phase 2) on 2026-10-06. New work in Claude Code: build the design system from the live araCreate site (audit → extract → tokens → components). See [[Projects/ac-ds/DEV-LOG]].
  - Principles doc written (`docs/DESIGN_SYSTEM_PRINCIPLES.md`, 20 sources, also as HTML page).
  - Folders: `01-input/` (Webflow exports "araCreate Template" + "araCreate Website"), `02-process/audit/`, `03-output/design-system/`. Both sites run locally (`serve.py`, :8001 / :8002).
  - CSS audits done. Design tokens v1 locked (7 W3C token files + README + visual preview page).
  - Button (`.cta-button`, 7 variants) and card docs done and checked. Nav, layout, footer, accordion, hero docs were still running at session end.
- Phase 1 (merge) was closed on 2026-08-24 (Vishnu: "all done, please close the project"). Result of phase 1:
- ACDS (Claude Design project `4716e773…`) holds the merged system and works: 84 component exports, no bundle errors, Forms / Core / Website cards render, `check_design_system` clean.
- ACDS is the master. GitHub repo `aracreate-group/aracreate-design-system` is backup only (Ara said so).
- Latest ACDS files were uploaded to the GitHub repo by Vishnu (web upload, in batches of under 100 files).
- Bugs fixed in ACDS: Card.jsx `Tag` name clash (5 components were missing), 7 duplicate form controls in `components/core/` deleted, `Hero.jsx` → `SiteHero.jsx`, muted text colour now passes contrast, 4 off-brand unused colour tokens removed, docs (`CLAUDE.md`, `github.md`, `readme.md`) corrected.
- Not done / left over:
  - Vishnu's own copy project "araCreate Design System — merged" (`22f6bdb1…`) is still broken (Card bug, duplicates, two Heroes). Low priority.
  - GitHub `README.md` and `src/readme.md` still say "repo is master". Vishnu said leave it for the next sync.
  - No "alpha scale" for see-through colours yet (25 of 33 hard-coded colours). Left as a separate job.
  - Old prompt files in `~/Downloads/ds/` are out of date and dangerous (`PROMPT-repair-acds.md`, `PLAN-clean-build-and-mirror.md`, `PROMPT-fix-acds-tokens.md`).

## 3. Next steps
Phase 2 (Claude Code):
1. Finish nav, layout, footer, accordion, hero component docs.
2. Visual check of type/spacing scale against real pages (Chrome blocks localhost — need another way).
3. Answer the drift questions (see Open questions).

Phase 1 leftovers:
1. Do not run any old prompt from `~/Downloads/ds/`. Maybe delete them or mark them "DO NOT USE".
2. At next sync: export ACDS from Claude Design → replace `src/claude-design-system/` in GitHub → also fix the GitHub README "source of truth" text.
3. Record the real commit sha in `github.md` at the next sync.
4. Decide what to do with project `22f6bdb1` (fix from an ACDS export, or delete) and the old `araCreate Design System` project (`a890ecee…`).
5. Optional: alpha-scale token job for white / graphite see-through colours.

## 4. Decisions
- 2026-08-21 — ACDS is the base; the newer system is poured into ACDS shape (folders, names, values). — ACDS is the original (imported 17 Aug). #decision
- 2026-08-21 — Keep muted grey `#8a8a8a` as-is at first; fix focus ring (`#222222`), success (`#186a43`) and button label on gold (`#222222`). — Vishnu's call on accessibility. #decision
- 2026-08-21 — Lock 6 items (resolved to 23 files): `acds-template-deck`, Isometric Iconography, Slide — Section Divider, Slide — Service Verticals + Stats, Slide — Service Vertical, araCreate Deck — Sample Slides (plus `slides.jsx`, helpers, logos, icons, `styles.css`). Landing Page card stays open. — "those are exact correct ones". #decision
- 2026-08-21 — "Don't do direct changes, get the plan." After the bad write, all work is plans/prompts for Vishnu to run. #decision
- 2026-08-22 — Do not push the merge straight into ACDS; upload the merged result to a new project first (`22f6bdb1`). — safety. #decision
- 2026-08-23 — Spacing: keep the newer 16-step values under ACDS names (`--ac-space-1…16`) with a conversion table, not ACDS's original 10 values. — reverting would break the new components. (Recorded as approved; not sure Vishnu said it in his own words.) #decision
- 2026-08-23 — `forms/` and `navigation/` components are canon; `core/` duplicates deleted. — done by the Claude Design in-app assistant. #decision
- 2026-08-23 — Muted text changed to `var(--ac-gray-450)` = `#6f6f6f` (4.65:1). 4 dead tokens deleted (`--ac-photo-overlay` navy, `--ac-true-black`, `--ac-gray-translucent`, `--ac-button-gray-light`). — colour audit. #decision
- 2026-08-23 — Merge Vishnu's ACDS into Ara's ACDS and keep it as single source of truth. — Ara's Slack request; already true. #decision
- 2026-08-24 — ACDS (Claude Design) is master; GitHub is backup only. Changes flow ACDS → GitHub, never back. — "ACDS is master, Ara said". #decision
- 2026-08-24 — Do not edit the GitHub README now; it is for the next sync. #decision
- 2026-10-06 — Phase 2: design first; code comes later from the design system. Reference doc lives in the repo as markdown. #decision
- 2026-10-06 — Gold `#F9BF3B` is primary; blue `#3347a0` also core. Palette: gold, blue, ink `#2e2e2e`, white, light gray `#f6f6f6`. #decision
- 2026-10-06 — Font Poppins; use the rem type and spacing scales (px values are noise). Card radius 20px. #decision
- 2026-10-06 — Unused red/green/gold status colours kept as reserved tokens for a planned feature. #decision
- 2026-10-06 — Extract from source code, pixel by pixel, not approximate. `.button` and `.cta-button` are separate variants (live site uses only `.cta-button`). #decision
- 2026-10-06 — Stop asking per component; follow the principles doc, build all core components, raise only real drift questions. #decision

## 5. Timeline
- 2026-08-17 — ACDS imported into Claude Design (per its `github.md`).
- 2026-08-18 — A pre-merge copy of ACDS saved locally (later used as backup).
- 2026-08-20 — Newer "araCreate Design System" pushed to Claude Design (from the aracreate project).
- 2026-08-21 — Merge plan written; Vishnu said "go"; Claude wrote 317 files straight into ACDS (Ara's published, org-default project) without checking ownership. Vishnu: "you spoiled the whole thing".
- 2026-08-21 — Incident analysis: 68 ACDS files overwritten, 249 added; generated bundle not rebuilt, so nothing went live.
- 2026-08-22 — Vishnu exported both systems to `~/Downloads/ds`. Bundle rebuilt locally by an agent (84 components); 65/65 cards render.
- 2026-08-22 — Merged system uploaded to new project "araCreate Design System — merged" (`22f6bdb1…`, 411 files).
- 2026-08-22 — Clean pre-merge copy found (`~/Downloads/acds-aracreate-design-system.zip`, 24/24 hashes). Restore wrote 165 files; delete of 249 files blocked (HTTP 403, owner only).
- 2026-08-22 — `gh api` gave 404 for the GitHub repo; Claude wrongly said the repo "never existed". Local `git init` commit `12c7b85`; `gh repo create` failed (no org permission).
- 2026-08-23 — Vishnu's own pre-merge backup verified (249 added / 68 overwritten / 0 deleted / 101 untouched). Half-restore left 34 undefined tokens; an agent wrote 411 files back into ACDS to fix it.
- 2026-08-23 — Claude Design in-app assistant fixed duplicates, Hero clash and Card/Tag bug. "Nothing is broken now."
- 2026-08-23 — Colour audit + colour fix (muted grey, 4 dead tokens). `CLAUDE.md` bullets fixed.
- 2026-08-23 — (about) Ara's Slack: ACDS tracked in git manually; merge into his ACDS as single source of truth. Repo exists (private). Docs corrected.
- 2026-08-24 — ACDS set as master, GitHub as backup. Vishnu uploaded ACDS files to GitHub. Phase 1 closed.
- 2026-10-06 — Phase 2 started in Claude Code: principles doc, folders, local sites, CSS audits, tokens v1, button/card docs (746 template components extracted; pretty-printer bug that dropped first letters of CSS names found and fixed).

## 6. Key facts
- **People:** [[People/Vishnu]] — did the merge work; not a tech person, wants simple step-by-step words. Ara — owner of ACDS, org default, decides policy (probably [[People/Aravinth Panch]], not sure).
- **Companies:** [[Companies/araCreate Group]]
- **Tools:** [[Tools/Claude Design]], [[Tools/Claude Code]], [[Tools/Claude]], [[Tools/GitHub]], [[Tools/Slack]], Playwright (headless checks)
- **Links / repos / servers / file paths:**
  - ACDS project `4716e773-3175-4bc2-a22e-f34c179aea34` (owner Ara, Org default, Published)
  - Old newer system `a890ecee-…` (untouched)
  - Vishnu's merged copy `22f6bdb1-5dfd-4e1c-9b53-c098478ea6e8`
  - Share links to the design projects: (secret, not saved)
  - GitHub (private): https://github.com/aracreate-group/aracreate-design-system — branch `main`, path `src/claude-design-system`
  - Local: `~/Downloads/ds/` (exports, `_clean-build` (418 files; once planned as source of truth, later stale), `_diff-vs-original/`, prompt files), `~/Downloads/ds/aracreate-design-system-main/` (pre-merge backup), `~/Downloads/acds-aracreate-design-system.zip` (verified pre-merge zip)
  - Bundle namespace `window.AraCreateDesignSystem_4716e7`
  - Brand colours: Golden Sun `#f9bf3b`, Graphite Gray `#555555`, Black `#222222`, Stroke `#cecece`, Canvas `#f6f6f6`. Fonts: Poppins (all text), Monument Extended (logo only).
- **Related:** [[Projects/aracreate/SUMMARY]]
- Dev history: [[Projects/ac-ds/DEV-LOG]]
- Lesson learned: "allowed to edit" is not "should edit" — plans first, hash-gated writes after review.

## 7. Files and documents
- `araCreate-DS-merge-plan.md`, `araCreate-DS-reshape-to-ACDS.md`, `araCreate-DS-merge-plan-v2.md`, `araCreate-DS-locked-files.md`, `araCreate-DS-RUNBOOK.md`, `RUN-REPORT.md` — merge planning (sent in chat)
- `ACDS-incident-analysis.md` — what the bad write did (sent in chat)
- `STATUS-what-happened.md` — mistakes and fixes (`~/Downloads/ds/`)
- `ACDS-half-restore-damage.md` — damage after the half restore (`~/Downloads/ds/`)
- `PROMPT-colour-audit.md`, `PROMPT-colour-fix.md` — finished (`~/Downloads/ds/`)
- `PROMPT-repair-acds.md`, `PLAN-clean-build-and-mirror.md`, `PROMPT-fix-acds-tokens.md` — out of date, DO NOT RUN (`~/Downloads/ds/`)
- `PROMPT-fix-repo-claim.md` — fixed docs about the repo (`~/Downloads/ds/`)
- `_merged-gallery.html` — local card gallery (in the ACDS export folder)
- `CLAUDE-for-araCreate-DS.md` — merged CLAUDE.md (API cannot write `CLAUDE.md`, must paste by hand)

## 8. Open questions and problems
- Project `22f6bdb1` still broken — fix or delete?
- Old project `a890ecee` — keep or retire?
- GitHub README / `src/readme.md` still say the repo is master.
- Original `github.md` sync history was deleted by mistake (should still be in git history).
- Was every file uploaded to GitHub in the batch upload? Only Vishnu checked.
- Alpha-scale for see-through colours not done.
- Phase 2 drift questions: Red Hat Mono — real part of the system (only 2 uses)? Template `.button` never reached the live site — keep or drop? `home-button`/`newsletter` margin drift and `dtf-menu` colour inversion in `.cta-button`.
- Lesson: check ownership ("should I edit", not only "can I edit") before writing to shared projects. Several agents (Claude Code, Claude Design assistant, Vishnu's agent) edited the same project without knowing about each other.
- Old open questions are answered: ACDS stays the company default (single source); the three merge tasks were done inside the big merge; Landing Page card is not locked; switch-over is done.

## 9. All chats in this project
- Index: INDEX (archived: Projects/ac-ds/chats/INDEX.md)
- Design systems merge (archived: Projects/ac-ds/chats/2026-08-21 Design systems merge.md) — 2026-08-21
- Claude Code sessions (2026-10-06): see [[Projects/ac-ds/DEV-LOG]] session index
