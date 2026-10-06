---
title: halle-feedback-widget/STATE
type: state
zone: ai
project: halle-feedback-widget
as_of: 2026-10-05
updated_by: claude-sonnet-5 (claude code)
status: active
---
## For future agent
Current state of [[halle-feedback-widget]]. READ THIS FIRST when resuming. Always true "as of" the date above. Update at every handoff.

**Before touching any server for this project, read this file's "Now"
section below — not the repo's own docs.** On 2026-10-05 a separate Claude
Code session (no viOS, working from the repo alone) deployed to the OLD,
retired server (212.227.213.174 / feedback.arametrics.app) because it
never checked here first and the repo's own `docs/session-handover.md`
still called that server "Live and working." This file already had the
correct answer (see line below, unchanged since 2026-09-30) the whole
time. The repo's docs have since been patched with warning banners
(commit `f5f12f0`), but **this file, not the repo, is the fast path to
the right answer** — check it before any deploy, every time.

# STATE — halle-feedback-widget

## Now
- **Live:** web app on the new server, box `webapp` (10.10.0.20) on host 217.160.93.75, at https://apps.b-halle.de (since 2026-09-30). **Deployed 2026-10-06: commit `d9b6d11`** — removes Navy as a classification tag colour (it's the brand's own primary colour; migration 0017 auto-recoloured the one prod row that used it, "Phase-2", to the new default Teal), on top of the same day's earlier deploy (`90d40f4`: 4 more tag colours — steel_blue, burgundy, forest, brown) and 2026-10-05's (`f5f12f0`: classifications, users admin, bug-step ordering, conventions pass, filter/UI fixes, popup-refresh redirect). Migrations 0013–0017 applied. `vishnu@aracreate.group` is admin on prod (made so 2026-10-06). Other 3 real logins (jakob.silbermann@b-halle.de, rahul@aracreate.group, shyam@aracreate.group) still non-admin.
- **On `dev`, pushed, NOT deployed (2026-10-05, head `f55ebec`, 10 commits):**
  - Classifications: custom report types beside Bug; Queue popup "Sort as"; Classified page (Open/Done); Admin → Classifications. Migration 0013.
  - Users admin screen (admins only, `users.is_admin`), account menu at sidebar foot, Settings page (own name + password). Reset signs the person out everywhere. Migration 0014.
  - UI declutter (Manage popups, Testers, Pages, Wording sections, sidebar divider).
  - Dashboard end-to-end suite: `make test-e2e` (23 tests, own server :3201 + test DB).
  - araCreate conventions: every file header, param-case names, JSON `_meta`, and 100% snake_case — including the widget↔server contract (`tester_token`, `upload_url`, …) and saved string keys (migration 0015). Exceptions (library names) listed in repo `docs/agent-rules.md` §5.
  - **2026-10-05:** bug steps now enforced in order — Processing → Fixed → Closed. Closed always meant "a stakeholder checked the fix", not "won't fix" as the old spec said; the screen used to let anyone close a bug nobody had marked fixed. `move_bug()` refuses a step the current status does not allow. Every report shows a numbered step line (ringed current step, filled line up to it); Tracked items' Fixed tab reads "To check"; Overview leads with bugs waiting for a check. `admin-v2-spec.md` corrected. No schema change.
  - **2026-10-05:** leftover `{ a: a }` pairs from the snake_case rename (57 of them, 25 files) tidied to `{ a }` — pure formatting, no behaviour change.
- **Verified locally (2026-10-05):** 446 unit/db tests, 76 widget tests, 23 e2e tests, lint + tsc clean; a real report from the rebuilt widget to the dev server stored with tester, page title, capture method and picture, also via server capture. The step-line layout bug (uneven gap before the last step — `last:flex-none` instead of `flex-1`) was found from Vishnu's screenshot and fixed; verified by measuring real DOM positions (all four gaps 263.0px), not by eye.
- **Local dev DB** has migrations 0013–0015; admins there: vishnu@aracreate.group, staff@demo.test (local demo login; its password is in repo `src/web/scripts/db-demo.mts`).
- **Old server (212.227.213.174): ⚠ re-started 2026-10-05, needs re-stopping.** A session without viOS context found it reachable, didn't know it was retired, and ran `git pull` + rebuild + `systemctl start halle-feedback` on it (also made `vishnu@aracreate.group` admin in its now-orphaned database — harmless, nobody uses that database). No real traffic reaches it (Apache no longer forwards there per the migration), but the service is live on that box right now where this file says "stopped + disabled." **Next 0 below fixes this.** NOT cancelled yet regardless (see Next 6).
- A GitHub personal access token was found embedded in plain text in `origin`'s remote URL on BOTH servers (`git remote -v`). Flagged to Vishnu 2026-10-05, not yet confirmed rotated — check before relying on git push/pull credentials on either box.
- `kishor-aracreate`'s SSH public key was added to `(secret removed)`'s `authorized_keys` 2026-10-05, at Vishnu's request — full root access to prod, same as the agent's own key.
- Claude Code auto mode blocks production reads/deploys by default → normally Vishnu deploys by hand. **2026-10-05 exception:** Vishnu explicitly walked the agent through the real deploy step by step in chat (confirmed via AskUserQuestion at each stage: SSH access, which server, proceed y/n) — not a default/unprompted deploy. Still the right default going forward: don't deploy without an explicit, in-the-moment instruction, viOS or not.

## Next (in order)
0. **Re-stop the old server's accidentally-restarted service**, so STATE matches reality: `ssh -i ~/.ssh/halle_agent (secret removed) 'systemctl stop halle-feedback && systemctl disable halle-feedback'`. Confirm with `systemctl status halle-feedback` → inactive, disabled. Low urgency (no real traffic reaches it) but leaving it running contradicts this file and the migration plan.
1. ~~Coordinated deploy of `dev` to apps.b-halle.de~~ **Done 2026-10-05**, commit `f5f12f0`. ~~No admin on prod~~ **Fixed 2026-10-06**: vishnu@aracreate.group made admin (he noticed "Users" missing from his own sidebar, asked why — that's the admin gate). The other 3 logins are still non-admin; Vishnu's call whether to add more.
2. Update the repo's git remote: GitHub says it moved to `https://github.com/aracreate-group/halle-app-widget.git` (pushes still work via redirect).
3. Check the dashboard in Safari and on iPad (tests ran in Chrome only).
4. Fix the login redirect: `next` accepts `//other-site.com` (open redirect) in `src/web/app/login/actions.ts`.
5. **On 2026-10-14:** read the old server's Apache access log; confirm no real visitors; final archive per repo `docs/server-migration-plan.md` § Part 4 into `~/araCreate/HLE/server/old-server-final.tgz`.
6. Ask Jakob for the new IONOS contract's 30-day money-back end date; after it and after step 5, Jakob cancels old M+ contract 111321277.
7. After cancellation: update repo `deploy/runbook.md` (renamed from RUNBOOK.md) for the box layout + new IP.
8. Delete local server backups in `~/araCreate/HLE/server/` 30 days after cancellation (ask Vishnu first).
9. **Rotate the exposed GitHub token** on both servers' git remotes (see "Now" above) — revoke on GitHub, reconfigure `origin` to use a deploy key or fresh token.

## Blockers / questions for Vishnu
- ~~Who should be admin on prod?~~ Resolved 2026-10-06: vishnu@aracreate.group. Should the other 3 real logins get it too?
- The money-back window end date (Jakob knows).
- Should the agent get production deploy rights, or keep manual deploys? **2026-10-05 note:** today's deploy happened with the agent driving, Vishnu approving each step live — a middle ground between "fully manual" and "agent decides alone." Worth deciding explicitly which mode is the standing default.
- Has the exposed GitHub token been rotated yet?

## How to run / test
- Local: `cd ~/araCreate/HLE/testing_widget/halle-feedback-widget && make demo` → app http://localhost:3000, tester page http://localhost:4319/.demo/host-page.html?t=<token printed by make demo>.
- Deploy (prod, one line at a time): `ssh -i ~/.ssh/halle_agent (secret removed)` → `machinectl shell webapp` → `cd /opt/halle-feedback/app` → `sudo -u halle-feedback -H git pull --ff-only` → `sudo -u halle-feedback -H env (secret removed) npm run build --workspace halle-feedback-web` → `systemctl restart halle-feedback`.
- Widget rebuild only with `WIDGET_API_ORIGIN=https://apps.b-halle.de`, or reports silently go nowhere.

## Key files
- Repo `docs/server-migration-plan.md` — server layout, migration status, Part 4 steps.
- Repo `deploy/runbook.md` — from-scratch setup steps for the OLD, retired server; now carries a warning banner pointing here instead, but the body itself still needs rewriting for the new box layout (Next 7).
- Repo `docs/classifications-spec.md`, `docs/users-admin-spec.md` — the two new features.
- Repo `tests/e2e/` — dashboard end-to-end tests; `tests/readme.md` lists every suite.
- Repo `src/web/middleware.ts` — session gate for `/app/*`.
