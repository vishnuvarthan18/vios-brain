---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: ACDS — araCreate Design System

Full history: [[Projects/ac-ds/SUMMARY]] · Log: [[Projects/ac-ds/LOG]] · Dev: [[Projects/ac-ds/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Reopened as phase 2: build the design system from the live araCreate site (Claude Code).
- Done: principles doc, input/process/output folders, both sites running locally, CSS audits, tokens v1 (7 W3C files), button + card docs.
- Running at last session: nav, layout, footer, accordion, hero docs.
- Phase 1 (merge, closed 2026-08-24): ACDS in Claude Design is master and working (84 exports); GitHub repo is backup only (Ara's rule).
- 249 unwanted files from the August bad write remain in ACDS; only Ara (owner) can delete them.
- Vishnu's copy project `22f6bdb1…` is still broken (low priority). Old prompts in `~/Downloads/ds/` — do not run.

## Next steps
1. Finish the remaining core component docs.
2. Visual check of type/spacing scale against real pages.
3. Decide: Red Hat Mono, template `.button`, `.cta-button` drift.
4. Next ACDS sync: export → replace `src/claude-design-system/` in GitHub; fix README "source of truth"; record commit sha.
5. Decide fate of `22f6bdb1…` and `a890ecee…`. Optional: alpha-scale token job.

## Blockers
- Chrome blocks localhost (org policy), Safari can't be driven — visual checks need another way.
- GitHub README change waits for the next sync (Vishnu's choice).

## Key places
- Phase 2 folders: `01-input/`, `02-process/audit/`, `03-output/design-system/tokens/`
- ACDS project `4716e773…` (Claude Design, owner Ara)
- https://github.com/aracreate-group/aracreate-design-system (private, `src/claude-design-system`)
- `~/Downloads/ds/` (phase 1 exports, plans, prompts)
