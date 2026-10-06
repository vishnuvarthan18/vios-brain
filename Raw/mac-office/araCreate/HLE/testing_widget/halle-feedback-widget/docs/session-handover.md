# SESSION HANDOVER — updated 10 September 2026

**Read this first in any new session on this project.**

> **⚠ STALE — this file was never updated after 10 September, and §2's**
> **server address is now wrong.** The server named below
> (`feedback.arametrics.app` / `212.227.213.174`) was retired 30 September
> 2026. **The real production server and its live address are in
> [`server-migration-plan.md`](server-migration-plan.md)** — read that
> file's status table before trusting anything about "where things stand"
> in this one. This file's §1 (roles, working style) still holds; its §2
> onward describes a state three weeks out of date. Before deploying
> anything, or telling anyone an address is live, confirm it against the
> migration plan, not against this file.

> **Editorial note, 10 Sept:** the older day-by-day narrative blocks that
> used to sit in this file have been compressed into §7 (History). Nothing
> was deleted — the full detail of each pass lives in its own document,
> linked from there. The decisions table (§5) and the bugs-found list (§6)
> are complete and carry everything from every previous pass.

---

## 1. Roles and working style — unchanged, and it matters

Vishnu = product owner and all decisions. Claude = **PM, tech lead and
architect — not the developer.** An AI agent in VS Code writes all the
code. Claude writes specs and task briefs, answers the agent's questions,
and reviews its work by inspecting the repo directly rather than trusting
its reports.

**Vishnu is not technical.** Explain in plain words, in points, no jargon.
Walk through every single command one at a time, wait for the actual
pasted result before giving the next step, and **say explicitly which
window each paste belongs in** ("your own Mac Terminal" / "the server
terminal" / "your browser" / "the browser console"). Expect small
mix-ups — correct them gently and keep going.

Two rules learned the hard way and proven again on 10 Sept:

- **Ask where a pasted block came from** before drawing conclusions from
  it. Misreading a Webflow editor's contents as published page HTML once
  cost an hour.
- **When Claude is wrong, say so plainly.** Twice on 10 Sept Claude gave
  a confident answer that research then overturned (see §4). Vishnu
  pushed back both times and was right to.

**Vishnu's own instruction on batching, 10 Sept:** he does not want work
handed to the dev agent piecemeal. Collect and finalise a complete list
first, then hand over **one focused task at a time**.

---

## 2. Where things stand right now

**Live and working.** Backend at **`https://feedback.arametrics.app`**
(Apache reverse proxy → `localhost:3000`, Let's Encrypt cert to
2026-12-08, auto-renewing). Admin at
**`https://feedback.arametrics.app/app`** — no SSH tunnel needed, that
workaround is retired. Widget script live on `halle-dev.webflow.io`.

**The full end-to-end test passed on 10 September.** Button → point at
element → screenshot → report lands in the queue → picture viewable in
admin, with the marker stroke and the element box burned in. This was the
long-standing open item and it is now closed.

**Deployed on 10 Sept:** all four consolidated fixes, the admin v3
rebuild (sidebar, Overview, design tokens), tester model changes, and the
capture hand-off fix. `dev` is pushed to GitHub; `main` is still
local-only until Vishnu says production.

**Known and accepted:** the screenshot takes ~11.6s on a real page. This
is the top priority and is now a task in the dev agent's hands (§3).

---

## 3. Task docs — what is done, what is live, what is queued

All task briefs now live in the **repo's `docs/` folder** (Vishnu's
instruction, 9 Sept: stop pasting long prompts into the agent; put the
file in the repo and tell the agent to read it). They are also mirrored
in this Claude project.

| Doc in `docs/` | State |
| --- | --- |
| `agent-task-consolidated-open-items.md` | **Done, committed, deployed.** Screenshot viewport-scoping, CSV token strip, 180-day retention, deploy-file sync, backups/log rotation/retention timers |
| `agent-task-admin-flow-simplify-and-ui-rebuild.md` | **Done, committed, deployed.** Tester name-only, assignments removed, tester revoke, Overview, sidebar, design tokens |
| `agent-task-capture-succeeds-but-ui-shows-no-picture.md` | **Done, committed, deployed.** The 3s caller timeout discarding good captures |
| `admin-v3-rebuild-plan.md` | The agent's own plan + build record for the admin rebuild |
| **`agent-task-screenshot-speed-500ms.md`** | **HANDED TO THE DEV AGENT 10 Sept — in flight.** The 500ms budget, phased |
| `plan-speed-marker-ui.md` | The analysis behind the speed and marker work, with Vishnu's decisions recorded |
| `research-screenshot-architecture-options.md` | First architecture options pass |
| `research-how-the-industry-solved-this.md` | **The important one.** Five-stream industry survey that settled the architecture. Read before proposing anything about capture |
| `halle-design-system-draft.md` | The real design system, now in the repo (the agent had been reconstructing it from a commit message) |

### Queued, written up, NOT yet handed over

1. **The marker pen — 8 fixes.** All eight are specified inside
   `agent-task-marker-and-capture-speed.md` (Part A). That file is the
   older combined brief; the speed half of it was split out into
   `agent-task-screenshot-speed-500ms.md` and handed over. **Part A is
   still pending and should be handed over as its own task next.** The
   eight: half-resolution canvas on HiDPI; preview-vs-sent thickness
   mismatch (~6× thinner in the sent picture); no stroke smoothing;
   redraw-everything lag plus newest segment drawn twice; fast strokes
   losing samples (needs `getCoalescedEvents`); a tap drawing nothing;
   the pen must be an **icon that arms drawing** (off by default,
   `pointer-events: none` when off); and the drawing surface becomes a
   **full-size frozen picture**, not a thumbnail.
2. **The 16 screens.** 6 widget (launcher, mode menu, pointer overlay,
   review screen, sent confirmation, expired-link notice) and 10 admin
   (login, Overview, Queue, Tracked items, All reports, Report detail,
   Pages, Page detail, Testers, Wording). Method agreed: **one screen per
   pass**, Vishnu screenshots it, we list what is wrong together, the
   agent changes only that screen, Vishnu confirms. Agreed starting
   screen: **the widget review screen.** Deliberately left unwritten for
   now on Vishnu's instruction.

---

## 4. 10 September — what happened, and two things Claude got wrong

### Deployed and verified live

Walked through by hand, one command at a time. Pushed 8 commits, pulled
to the server, migrated the production database, rebuilt, restarted,
tested live end to end. Then a second round: the capture hand-off fix,
pushed, pulled, widget rebuilt, verified.

### Operational facts learned on the server — these will bite again

- **The server needs Node 22 for migrations, and only has Node 20.**
  `make db-migrate` runs `node --experimental-strip-types`, which Node
  20 does not support. Fixed by installing a second Node **without
  touching the system one** (the VPS is shared with unrelated projects):
  Node 22.11.0 unpacked to **`/opt/node22`**. Any future migrate or build
  must prepend it:
  `sudo -u halle-feedback -H bash -lc 'export PATH=/opt/node22/bin:$PATH && cd /opt/halle-feedback/app && …'`
- **`git` on the server needed a safe-directory exception:**
  `git config --global --add safe.directory /opt/halle-feedback/app`.
- **`.next` had root-owned files** from an earlier root-run build, which
  broke the build as `halle-feedback`. Fixed with
  `chown -R halle-feedback:halle-feedback /opt/halle-feedback/app/src/web/.next`.
- **THE WIDGET IS A SEPARATE BUILD FROM THE WEB APP.** This cost real
  time on 10 Sept: the web app was rebuilt, the widget was not, and the
  screenshot bug appeared unfixed on the live site. A widget change needs
  `WIDGET_API_ORIGIN=https://feedback.arametrics.app npm run build --workspace halle-feedback-widget-embed`,
  and no app restart (`widget-asset.ts` reads `dist/v1.js` from disk per
  request).
- **Migrations 0006–0008 are now applied to production** (testers
  `revoked_at` added, `email` dropped, `tester_page_assignments` table
  dropped).
- **The `AF_NETLINK` guess was verified correct** against the live
  server: `RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX AF_NETLINK`.

### Claude was wrong twice, and the record matters

**(1) "Start the capture before the click" — proposed, rejected, and it
turned out to be the wrong shape of answer anyway.** Vishnu declined it
twice. Do not re-propose it.

**(2) "Let the tester draw on the live page" — proposed by Claude, then
disproved by research.** The five-stream survey found that **no shipping
tool in this category lets anyone freehand-draw on a live, scrolling
page.** Every tool with a pen freezes the picture first. Tools that work
on the live page support *pointing only*. **This proposal is formally
withdrawn.** Detail and sources: `docs/research-how-the-industry-solved-this.md`.

Claude also asserted "there is no third route" to faster capture, which
was wrong — making the capture itself cheaper is a third route, and it is
now the plan.

### What the research settled — do not re-litigate

- **Every mobile-capable competitor renders the screenshot on their own
  server.** Marker.io (`ssr.marker.io`), Ybug, Usersnap and Userback all
  default to server-side rendering; the browser only serialises DOM+CSS.
  Marker.io's renderer even **re-fetches assets by URL** (they publish
  four static renderer IPs), so the client payload is text, not inlined
  base64. **Rasterising in the tester's browser — what we do — is the one
  approach no serious vendor in this category ships.**
- **Native screen capture is dead as a default.** The `getDisplayMedia`
  permission **can never be persisted** (W3C Working Draft, 27 Aug 2026:
  the UA "MUST NOT store a 'granted' permission entry"), the picker
  cannot be removed (`preferCurrentTab` only makes the current tab
  prominent), and there is **zero mobile support on any browser**.
  `getViewportMedia` — permissionless self-capture — has been stalled
  since ~2021 on same-origin-policy grounds. Every vendor that offers
  native capture offers it as a desktop-only opt-in fallback.
- **Full-page capture must never be attempted in the widget.** iOS caps
  canvases at **4,096px** and exceeding it returns a **blank image with
  no error**. WebP maxes at 16,383px. Scroll-and-stitch duplicates sticky
  headers on every slice. No embedded JS widget in the survey ships
  reliable full-page capture.
- **Our 11.6s is normal for the technique, not a defect in our code.**
  monday.com measured the same library family at 21s (html2canvas) and
  ~7s (modern-screenshot) on real content.
- **snapDOM is the same technique, not a new one** (clone → inline →
  `foreignObject` → decode). Its 10–25× claims are the author's own,
  synthetic; **no independent real-page benchmark exists.** Measure
  before believing.
- **Vishnu's pen-as-icon instinct matches the industry exactly.** Miro,
  Figma and others make navigation the default gesture and require
  drawing to be deliberately armed. Usersnap and Sentry simply disable
  annotation on mobile.
- **The industry answer to "can I scroll while annotating" is a labelled
  mode**, not a silent scroll lock — Filestage, Atarim and Pastel all
  ship an explicit comment-mode / browse-mode toggle. Vishnu rejected a
  silent lock, and was right.

### Decisions taken 10 September

| Decision | Note |
| --- | --- |
| **Screenshot budget: 500ms** | Vishnu's number, on the product argument that a tester who waits abandons the report. Non-negotiable target; miss it only with measured numbers |
| **Route 1 first, Route 2 in reserve** | Fix the client pipeline and measure. Build server-side rendering only if 500ms is missed |
| **Freeze-then-draw stays** | Live-page drawing withdrawn on the evidence |
| **Draw on a full-size frozen picture** | Not a thumbnail. Fixes the pen thickness/resolution problems at the root |
| **Pen is an icon, drawing off by default** | Canvas takes no pointer events when off, so scrolling works normally |
| **Keep today's pen weight** | Fix the mismatch, do not make the line bolder |
| **No pre-capture before the click** | Declined twice |
| **Viewport only, never full-page** | iOS blank-image limit |
| **No native screen capture** | Permission can never persist; no mobile support |
| **Fonts: measure, then ask** | The agent reports the cost; Vishnu chooses. No unilateral font change |
| **Tester delete = revoke** | Token rotated + `revoked_at` stamped; reports and their attribution stay. The database made a true hard delete impossible (append-only trigger + FK), so this was the only option that did not weaken `agent-rules.md §1.1` |
| **Home screen = Overview** | Counts at a glance (new / bugs / fixed / total, per-template breakdown, latest reports), each linking into the existing filtered Queue. Queue and Tracked keep doing the triage work |
| **Expired tester link shows a message** | A docked navy notice — "Your testing link has expired / Please ask for a new link" — instead of the widget silently not appearing. **No quiet remembering of the token** (`agent-rules.md §1.3` holds) |
| **Deferred, on record** | Report-arrival confirmation, queue bulk actions, and a read-only role for B. Halle staff — all found by the agent, all explicitly deferred by Vishnu |
| **Task briefs go in `docs/`, not in chat** | Tell the agent to read the file. Vishnu's instruction |

---

## 5. Decisions on record — do not re-litigate

| Decision | Note |
| --- | --- |
| Not a SaaS product | Build for B. Halle only |
| Copyright is **B. Halle** | Client-owned. Vishnu's call, 7 Sept |
| **English only** | German dropped entirely. No locale mechanism |
| **No keyboard path** to element selection | Mouse and touch only. **No WCAG claim anywhere** |
| **No "this page was fine" button** | Problems only. The main screen is a **report grid, not a coverage grid**. Never use the word "coverage" |
| `reports.outcome` only ever holds `'problem'` | Column kept so the button can return without a migration |
| ~~Three roles~~ **one login, no roles** | Reversed 8 Sept. Roles, permissions, issue assignment, team list all stripped |
| **Consent screen removed** | 8 Sept. The picture is always sent. `screenshot_key` still nullable, because a failed capture must not block a report |
| Dropped from the widget | Multiple targets, "where did you look for it?", read aloud, bigger text |
| Invites table dropped | No email in scope. Users via `make user-create` |
| Hand-rolled auth | Replaced Better Auth. `build-plan.md §2`'s stack table is **stale** |
| **Real VPS hosting** | Plain VPS, systemd, Apache. No Vercel, Cloudflare, Docker |
| npm workspaces, not pnpm | One fewer tool for the agent |
| Postgres 17 locally, 18 in production | Confirmed compatible |
| Pictures are **WebP**, on local disk | One legal key shape, regex-enforced. **Retention 180 days** — done and verified 10 Sept |
| `issues.ref` is a plain integer | The `issues` entity is folded into a `status` column on `reports` |
| Launcher is token-gated | Invited link only. Fails closed and **silently** — no error, no console noise, no DOM node. Remember this when debugging "nothing appears". As of 10 Sept an **expired** token is the exception: it shows a notice |
| Widget API origin | esbuild build-time define plus optional `data-api` override. **Never** derived from the script's own src. **Changing the backend address always requires a widget rebuild** |
| Report list naming | **Page name + number**, e.g. "Contact page #14" |
| Screenshot-mode grouping | **Not grouped** — each is its own item |
| **v2 milestones are M7–M10** | M6a/M6b were already taken |
| **Where a doc and the repo disagree about what exists, the repo wins** | Standing rule, proven repeatedly |
| **Brand design system** | Navy `#29308A` / Text `#2A2924` / Border Gray `#EBEBEB` / Light Blue `#B5E0FA`, Helvetica Neue, 12px/8px radius, 8/16/24/32/40/48 spacing. Now at `docs/halle-design-system-draft.md` |
| **Success green darkened** | `#1E8E3E` failed 4.5:1 against white (4.21:1) → `#1B7A34` (5.41:1) |
| **GitHub: one private repo, `dev` pushed, `main` local** | `github.com/aracreate-group/halle-app-widget`. Push `main` only when actually going to production |
| **CSV token stripped** | Done and verified 10 Sept |
| **"How do we know a round is finished?" — deferred** | Do not build anything for it without a fresh decision |
| **No DPA needed** | Testers are internal B. Halle staff. Revisit only if testers ever include the public |
| **Commit history follows araCreate conventions** | Verified. `claude/agent-task-conventions-compliance.md` |
| **Bridge shell has no GitHub network access** | Any push/fetch needs Vishnu's own Terminal |
| **Webflow: Save changes ≠ Publish** | Custom code must be **saved** (button at the top of the Custom code page) before Publish does anything |
| **Tester creation: name only** | Email removed from the form and the schema, 10 Sept |
| **Page assignments removed** | 10 Sept. One tester link works on every page. There was never any server-side enforcement to remove — it was only ever a planning/counting feature |

Plus everything in §4's decisions table.

---

## 6. Bugs found — the full list

- **A false green.** M3's report claimed 148 tests passing over 10 clean runs. The suite raced on shared fixture literals.
- **A concurrency race in `group_report`** — 20 simultaneous identical reports crashed it. Fixed with `onConflictDoNothing`.
- **An unscoped `limit 1`** in a test's guard-seed, writing into another tenant's data under concurrency.
- **A malformed-id crash** leaking a raw Postgres error on a bad `/app/issues/<id>` URL.
- **A client component pulling the Postgres driver into the browser bundle** — happened twice, second time during the v3 rebuild (`lib/db/` import dragging in `node:fs`). `next build` catches it.
- **A trailing-slash bug in invitation links**, plus an N+1 in the same function.
- **A spec that was wrong about the repo** — caught by inspecting the repo.
- `make demo` broken on a fresh checkout; the picture-viewer zoom button silently doing nothing.
- A `Co-Authored-By` trailer and several wrongly-typed commits — caught by reading the conventions repo, not by any gate.
- **Two agent sessions doing the same rewrite task at once** — name exactly one session on any history-rewrite task.
- **Two whole folders of code existed only on Vishnu's Mac, never pushed** — the entire `deploy/` folder and the widget's serving routes. Found only because deployment failed. **`git status` before trusting any doc's claim about what's committed.**
- **Server's Postgres locale, old Node version, a leftover three-roles bug in two scripts, and a systemd setting one notch too strict** — all found by running the deploy.
- **Mixed Content blocked the widget** on the https Webflow site until the backend got HTTPS.
- **certbot silently copied the wrong `X-Forwarded-Proto`** into the SSL vhost and added no http→https redirect. Both hand-fixed, now synced into the repo template.
- **The Webflow script tag sat UNSAVED** in the Custom code box for an hour while every server layer was re-verified.
- **(9 Sept) The screenshot capture walked the whole page** — 1,170 fetch requests for one viewport picture, because `overflow:hidden` clips visually but the clone still contained the entire document. Fixed by pruning off-screen nodes from the clone.
- **(9 Sept) The capture ignored scroll offset** — a picture taken 4,000px down rendered the top of the document. Found by the agent, not in any report.
- **(10 Sept) A good capture was thrown away.** `app.ts` capped its wait at 3s while `capture_screenshot` is allowed 12s, and `Promise.race` kept whichever won — so every capture slower than 3s was discarded even though the library went on to return a perfectly good image. This is what "(no picture)" actually was.
- **(10 Sept) The fix for that introduced a second bug** — the first-paint timer fired unconditionally and tore a good picture back off the screen 3s after the tester was already looking at it. Caught only by sampling the DOM over time; all 48 existing tests were blind to it because each asserts the screen once, immediately. **Lesson: a test that asserts once cannot catch a timer bug.**
- **(10 Sept) Two admin strings were not editable.** The Wording screen has an explicit field list that nothing checked against `DEFAULT_STRINGS`, so a string could reach testers with no way to change it. Now guarded in both directions.
- **(10 Sept) Four visual defects invisible to a green test suite** — the Overview's four numbers not adding up (no tile for closed/deleted), "20 of 49" reading as a page count when it was a report total, Queue tables sizing to content so Bug/Delete overlapped and spilled outside the border, and bare `<h1>`/`<p>` margins collapsing "Testers" into "Add a tester". **Found by screenshotting each screen.**
- **(10 Sept) No heading on any admin screen used a design-system size** — `globals.css` never set heading font sizes, so all fell back to browser defaults. Only found once the real design system doc reached the agent.
- **(10 Sept) `color-scheme: light dark`** was set against a light-only palette — a latent bug.

---

## 7. History — the detail lives in these documents

| Pass | Document |
| --- | --- |
| v1 build, M0–M5 | `docs/build-plan.md`, `docs/overnight-run.md`, `docs/overnight-log.md`, `docs/blocked.md` |
| The v2 redesign (8 Sept) — Vishnu specified a different widget after using v1 | `docs/widget-v2-spec.md`, `docs/admin-v2-spec.md`, `docs/v2-build-plan-for-agent.md` (start at §0.1), `docs/v2-overnight-run.md` |
| v2 build + hand verification | `claude/v2-verification-report.md` |
| The market research — **the most valuable document in this project** | `claude/market-feature-ledger.md` (21 tools, verified 2 Sept). Read before proposing anything about the tester flow |
| Competitor teardowns | `claude/research-raw-markerio-teardown.md`, `-bugherd-`, `-ybug-`, `-uat-platforms`, `-feedback-widgets` |
| Brand design system + how it was applied | `claude/halle-design-system-draft.md`, also now `docs/` |
| Conventions compliance pass | `claude/agent-task-conventions-compliance.md` |
| Live server deployment, every bug found and fixed | `claude/server-deployment-plan.md` |
| Domain + HTTPS, the two certbot traps, the lost hour | `claude/https-domain-live-record.md` |
| Admin v3 rebuild plan + build record | `docs/admin-v3-rebuild-plan.md` |
| **The industry survey that settled the capture architecture** | `docs/research-how-the-industry-solved-this.md` |

**Four things given up in the v2 widget redesign — know them, don't
re-litigate them:** the five fixed answer sentences (research on elderly
and non-native readers found an empty box produces silence); randomised
answer order (the one thing no tool in a 21-product scan could do, and
the strongest build-vs-buy argument); the previous refusal of a freehand
marker pen; and the tester's ability to refuse the picture.

---

## 8. Where the work lives

**Repo on Vishnu's Mac:**
`/Users/vishnuvarthanvenkatapathy/araCreate/HLE/testing_widget/halle-feedback-widget`

**GitHub:** `https://github.com/aracreate-group/halle-app-widget.git`
(renamed from `halle-widget`; the old URL still redirects and still shows
a "repository moved" notice on push — harmless). `dev` pushed, `main`
local only.

**Server:** VPS at `212.227.213.174`, Debian 12, root login (password
known to Vishnu only, never stored). App at `/opt/halle-feedback/app`
under its own account `halle-feedback`. Shared box with unrelated
projects — never touch system-wide packages. See §4 for the Node 22,
safe.directory and `.next` ownership facts.

**Conventions:** `https://github.com/aracreate-group/aracreate-conventions`
— `README.md`, `repo/readme.md`, `git/git-conventions.md`. Two that
matter most: **no `Co-Authored-By` trailers ever**, and **committing or
pushing each needs Vishnu's own explicit instruction at the time**.

**Device bridge notes:** `git status` through the bridge can leave a
`.git/index.lock` it cannot delete — use `git --no-optional-locks status`.
**The bridge shell cannot reach GitHub at all.** Claude's browser pane is
blocked by policy from opening `halle-dev.webflow.io` — use `curl` from
the server, or ask Vishnu, for live-page checks.

**Repo docs that are the working source of truth:** `docs/widget-v2-spec.md`,
`docs/admin-v2-spec.md`, `docs/agent-rules.md` (12 nevers, 7 always — the
agent reads it every session), `docs/quality-gate.md`, `deploy/runbook.md`
(now including step 13, the backup/log/retention install).

---

## 9. Blocked on Vishnu, not on code

1. **The real 49 page URLs.** The database still has 3 placeholders.
   Vishnu has full Webflow access. (A bare `/` for Home is **correct by
   design** — not a placeholder.)
2. **The IP clause in the B. Halle agreement.** Raised three times, still
   unread.
3. **Review the storage/accessibility statement** —
   `claude/storage-accessibility-statement-draft.md`.
4. **Push `main`** — only when actually going to production, and only on
   Vishnu's explicit word.

---

## 10. What to do next

1. **Wait for the dev agent's report on `agent-task-screenshot-speed-500ms.md`.**
   The task requires the Phase 0 timing table **before** any code change
   — expect that as the first check-in. **If it reports numbers measured
   only on a local fixture rather than the real Contact page, reject
   them**; tuning against a simple fixture is exactly what produced the
   wrong "second capture is cheap" conclusion.
2. **Deploy and test it live** the same way as 10 Sept: push from
   Vishnu's own Terminal → `git pull` on the server → **rebuild the
   widget** (remember: separate from the web app) → hard-refresh and
   retest. Read `window.__halleCaptureLog` in the browser console to
   confirm the real timings.
3. **Then hand over the marker-pen task** (Part A of
   `docs/agent-task-marker-and-capture-speed.md`, 8 fixes).
4. **Then the 16 screens**, one at a time, starting with the widget
   review screen.
5. If the 500ms budget is missed with honest numbers, **build Route 2** —
   client serialises, server renders. It is what the whole industry does.
   `docs/research-how-the-industry-solved-this.md` has the evidence.
