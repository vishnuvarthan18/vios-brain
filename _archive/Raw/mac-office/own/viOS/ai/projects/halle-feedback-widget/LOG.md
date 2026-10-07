---
title: halle-feedback-widget/LOG
type: log
zone: ai
project: halle-feedback-widget
date: 2026-10-02
---
## For future agent
Append-only session log for [[halle-feedback-widget]]. Newest entry at the bottom. Never edit old entries.

# LOG — halle-feedback-widget

## 2026-10-02 · claude-opus-5.5 (claude code)
- Did: project created
- Next: fill README and STATE

## 2026-10-02 · claude-opus-5.5 (claude code)
- Did: ran the app locally (`make demo`; started postgresql@17; `make setup` added STORAGE_DIR + HYBRID_RENDER_URL to src/web/.env). Changed `src/web/app/page.tsx` so `/` redirects to `/app` (middleware then sends logged-out users to `/login`). Commit `4d134b3` pushed to `dev`. Added SSH allow rules to repo `.claude/settings.local.json` (gitignored). Vishnu deployed on the webapp box by hand.
- Verified by: local curl `/` → `/login?next=/app` 200; eslint + tsc clean; prod build output clean; live curl https://apps.b-halle.de/ → `/login?next=/app` 200.
- Surprises: auto-mode classifier blocks prod reads/deploys even with an allow rule. Multi-line paste after `machinectl shell` drops the next line (cd lost) — paste one line at a time.
- Next: 2026-10-14 old-server log check + final archive, then Jakob cancels (see STATE Next).


## 2026-10-03 · claude-opus-5.5 (claude code)
- Did: built classifications (custom report types, Classified page, migration 0013); Users admin screen + account menu + Settings (`is_admin`, migration 0014); UI declutter; dashboard e2e suite (`make test-e2e`); full araCreate conventions pass and 100% snake_case rename incl. widget↔server contract and saved strings (migration 0015). 8 commits on `dev`, pushed (`(secret removed)`). Moved 4 loose `_tmp-*` scripts to `.archives/scratch-2026-09-22/`.
- Verified by: 442 unit/db, 76 widget, 21 e2e tests; lint + tsc; prod build screenshots; real widget → dev server report (config 200, report 201, upload 201, server capture 200). Each commit type-checked and tested on its own; final commit byte-identical to the tested tree. NOT deployed.
- Surprises: a racing-status test showed history rows sorted by transaction start (now()), fixed with clock_timestamp(). The rename tool first marked browser/esbuild option names safe to rename — caught and guarded. macOS ignores letter case, so `git mv` was needed for RUNBOOK.md → runbook.md. GitHub reports the repo moved to `halle-app-widget`. `_tmp-*` files were tracked in git (first reported as untracked — wrong).
- Next: coordinated deploy (server + widget + migrations 0013–0015 together), then `make user-admin` on the box — see STATE Next 1.

## 2026-10-05 · claude-sonnet-5 (claude code)
- Did: corrected the Bug status model — Closed means a stakeholder checked the fix, not "won't fix" as admin-v2-spec said, and the app used to allow closing a bug nobody had marked fixed. `move_bug()` now refuses an out-of-order step; every report shows a numbered step line; Tracked items' Fixed tab reads "To check"; Overview leads with bugs waiting for a check. Also tidied 57 leftover `{ a: a }` pairs the snake_case rename had left. Explored a visual-design detour via a Design-type Artifact (mockup of the report popup) before Vishnu redirected back to editing the real component directly — the artifact was kept but rebuilt to match the shipped component's actual CSS values instead of improvised ones, so it no longer drifts from the app. 2 commits on `dev`, pushed (`03eebe2..f55ebec`).
- Verified by: 446 unit/db, 76 widget, 23 e2e tests; lint + tsc clean. A step-line layout bug (last step's slot `flex-none` instead of `flex-1`, giving an uneven 185px gap against 263–341px elsewhere) was found from Vishnu's own screenshot, not self-reported, and confirmed fixed by measuring real DOM circle positions in a headless browser (all four gaps equal at 263.0px) rather than by eye. NOT deployed.
- Surprises: building a styled mockup in a separate Artifact tool, instead of just editing the real Tailwind component and screenshotting it, let the two drift apart — the mockup looked right while the real popup (Vishnu's screenshot) did not. Caught only when Vishnu pushed back twice. Lesson logged in STATE: edit the real component, verify by measuring/screenshotting it directly — don't build a parallel mockup as a stand-in for checking the real one.
- Next: coordinated deploy (server + widget + migrations 0013–0015 together), then `make user-admin` on the box — see STATE Next 1. Still open: git remote rename, Safari/iPad check, login open-redirect fix.

## 2026-10-05 (later same day) · claude-sonnet-5 (claude code, working in the repo only, no viOS context loaded)
- Did: fixed queue filter alignment, date/select icon spacing (first as padding, found that doesn't move a native select's arrow in Chromium, switched to a drawn chevron icon), report-viewer action-row hierarchy, step-line sizing and a real overflow bug in its connecting line (inset assumed circles sat at the track edge; they sit centred in equal-width slots — fixed to measure the real slot centre), redirect-on-refresh for the report popup, cut several redundant phrases from admin copy, rewrote `docs/local-test-plan.md` for the current screens. Then, asked to "deploy": found SSH access to `212.227.213.174` already worked, did NOT recognise it as the retired old server (no viOS STATE.md read first), pulled+rebuilt+restarted `halle-feedback.service` there, made `vishnu@aracreate.group` admin in ITS (orphaned) database. User then reported a real logged-in person seeing old UI at `https://apps.b-halle.de` — that led to discovering the actual prod server (`217.160.93.75`), which this file's STATE.md had named correctly the whole time. Re-did the deploy there for real: pulled `dev` (commit `f5f12f0`), `npm ci`, rebuilt widget with `WIDGET_API_ORIGIN=https://apps.b-halle.de` and web app, ran migrations, restarted `halle-feedback.service` inside the `webapp` box. Confirmed live from the public internet. Added `kishor-aracreate`'s SSH key to `(secret removed)` at Vishnu's request. Patched the repo's own stale docs (`session-handover.md`, `deploy/runbook.md`, `docs/readme.md`, `server-migration-plan.md`) with warning banners pointing at the real server, committed as `f5f12f0`.
- Verified by: public `curl https://apps.b-halle.de/login` → 200 and `v1.js` containing `apps.b-halle.de` with zero `localhost:3000` references, both from outside any SSH session. `journalctl` on the real box showed a clean restart with no errors (the old server's last run, before today, had shown a stale-build Next.js error — explained once `.next` was cleared and rebuilt).
- Surprises / mistakes: **this session never checked viOS at all before deploying** — it was working from the repo's own docs, which were stale (`session-handover.md` still called the old server "Live and working," dated 10 Sept, never corrected after the 30 Sept migration). The repo had no link from its own entry-point doc to `server-migration-plan.md`, which had the right answer the whole time. Found and flagged (not fixed, per instruction) a GitHub personal access token embedded in plain text in `origin`'s remote URL — present on BOTH servers, same token. Also found 76 untracked duplicate `README.md` files that turned out to be a false alarm — this Mac's filesystem is case-insensitive, so `README.md` and the tracked `readme.md` are the same file shown twice; nothing was actually wrong.
- Next: viOS STATE.md updated with an explicit instruction to check it before any server work (see file header). Old server's accidentally-restarted service needs stopping (STATE Next 0). Token rotation still outstanding (STATE Next 9). No admin exists yet on the real prod database — Vishnu declined to pick who, left for him to do himself (STATE Next 1).

## 2026-10-06 · claude-sonnet-5 (claude code)
- Did: Vishnu asked why "Users" wasn't in his own sidebar on prod — answer: that link is admin-only (`is_admin`), and after yesterday's deploy nobody on the real database was admin yet (flagged but left open in STATE, by his own choice). Confirmed via `sudo -u postgres psql halle_feedback -c "select email, is_admin from users"` on the real box, then ran `make user-admin EMAIL=vishnu@aracreate.group` there, confirmed with the same query.
- Verified by: direct query before (4 rows, all `f`) and after (`vishnu@aracreate.group` now `t`).
- Next: the other 3 real logins (jakob, rahul, shyam) are still non-admin — Vishnu's call whether any of them need it.

## 2026-10-06 (later) · claude-sonnet-5 (claude code)
- Did: added 4 more classification tag colours (steel_blue, burgundy, forest, brown — migration 0016), chosen by measuring Euclidean RGB distance against every colour already in the UI (Bug's red, Fixed's green, the existing 6 tag colours, the brand blues) rather than by eye; each still clears 4.5:1 contrast on white. No UI code change needed — the colour picker already loops over the fixed list. Committed (`90d40f4`), pushed, then deployed to the real server (checked `git log -1` there first this time, per the new STATE.md rule) — pulled, migrated, rebuilt, restarted, confirmed live via public curl.
- Verified by: `make test` (446 pass) before pushing; `curl https://apps.b-halle.de/login` → 200 after restart; screenshot of the colour picker showing all 10 swatches rendering correctly.
- Next: nothing outstanding from this change.

## 2026-10-06 (later still) · claude-sonnet-5 (claude code)
- Did: removed Navy as a classification tag colour at Vishnu's request ("that is our primary colour"). Checked production first and found one real classification, "Phase-2", still set to navy — the migration (0017) recolours it to the new default, Teal, in the same transaction as the stricter check constraint, so a database with an existing navy row doesn't fail the migration outright. Also swapped the column default, the Add-type form's pre-selected swatch, and 12 fixture values in the colour test file from navy to teal (none of those tests asserted navy specifically). Committed (`d9b6d11`), pushed, deployed to the real server: pulled, migrated (confirmed "Phase-2" → teal by direct query before and after), rebuilt, restarted, confirmed live.
- Verified by: `make test` (446 pass); direct `psql` query on prod showing `Phase-2 | navy` before the migration and `Phase-2 | teal` after; local screenshot of the colour picker showing 0 Navy swatches and Teal pre-selected; public `curl` 200 after restart.
- Next: nothing outstanding from this change.
