# Project context — read this first in any new chat

I don't have a way to auto-carry memory into a new chat, so this file is the substitute — read it fully before doing anything else on this project.

## What this is

A WhatsApp bulk-messaging tool, aimed to be **sold as a desktop app** (not hosted), to small businesses. Vishnu is non-technical and builds everything through Cursor — I (Claude) act as the planning "brain," Cursor is the one that writes/runs code.

## How we got here (so you don't repeat settled decisions)

1. Started as a hosted web app (React + Node backend) using `whatsapp-web.js`.
2. Switched engine to reused free GitHub projects instead of building from scratch: **Evolution API** (multi-tenant WhatsApp connector) + **BulkPro** (`kunaldevelopers/whatsapp-bulk-sender-dashboard`, MIT-licensed dashboard UI) as the reference for CSV upload/messaging/sending UX.
3. Tried hosting on Oracle Cloud (free tier) — blocked for hours by "out of host capacity" on the free ARM shape. Abandoned.
4. Tried DigitalOcean (~₹510/mo, Bangalore datacenter) as the paid fallback — decided on this, then scope changed again before building it.
5. **Direction changed**: Vishnu's actual goal is to *sell* this as installable software, not run it as a hosted service. This makes the whole server/multi-tenant/hosting conversation (Evolution API, Oracle, DigitalOcean, Docker) obsolete for the product itself.
6. Researched "free" GitHub bulk-sender apps as a shortcut — found most either have **no license** (not legally reusable) or are **paid CodeCanyon products mirrored on GitHub with anti-resale terms** (e.g. WaBoApp). Lesson: the underlying engine libraries (Baileys, whatsapp-web.js) are genuinely MIT/Apache and safe to build on; finished "bulk sender" wrapper apps found on GitHub mostly are not safe to resell as-is.
7. Landed on current direction (below).

## Current direction (locked in, don't re-litigate)

- **Product**: Windows/Mac desktop app (Electron), each customer installs their own copy, links their own WhatsApp. No server, no hosting cost for Vishnu.
- **Engine**: Baileys (`@whiskeysockets/baileys`, MIT) directly — no whatsapp-web.js/Puppeteer, no Docker.
- **Storage**: SQLite, bundled locally in the app (actually shipped as `sql.js` — native `better-sqlite3` failed to build, see build history — same local-only behavior).
- **Licensing**: dual model — **one-time purchase** and **monthly subscription**, both offered (not just one).
- **Selling method (changed 2026-08-05, supersedes Lemon Squeezy plan)**: fully offline, no payment processor, no online account. Vishnu collects payment manually (UPI/bank transfer) outside the app, then generates a license key himself using a small local key-generator tool (`keygen.js`, runs only on his own computer, never shipped). Key = license type + expiry (subscription only) + HMAC signature, checked locally against a secret embedded in the app — no internet call, ever. One-time keys never expire. Subscription keys carry an expiry date Vishnu sets at generation time; renewal = pay again, he sends a new key. No activation-limit enforcement possible offline (accepted tradeoff — mitigated by small buyer base + personal delivery, not a technical control). Lemon Squeezy is dropped entirely — `PRODUCT_SETUP.md` and `product-details.txt` are obsolete/deleted.
- **Anti-ban behavior** (carried over from all earlier testing, still the standard): 20-60s random delay between messages, 5-minute cooldown every 50 messages, per earlier research this is table stakes, not the actual determinant of ban risk — cold outreach to non-consenting numbers is the real risk driver regardless of delay tuning.
- Vishnu's real budget ceiling discussed earlier was ₹300/month for *hosting* — no longer relevant now that there's no server.
- **"SaaS" clarified (2026-08-05)**: when Vishnu says "make this a proper SaaS," he means production-grade/professional quality, NOT a literal pivot to hosted/multi-tenant. Confirmed explicitly — do not reintroduce servers/hosting.

## What's already been proven to work (don't redo this validation, build on it)

- A local Docker-based prototype (Evolution API + Postgres + our Express/React app) **did successfully connect a real WhatsApp account via QR** and show "Connected."
- A real broadcast test send failed with "Internal Server Error" from Evolution API — root cause was never fully confirmed (was mid-diagnosis when direction changed). Not necessarily relevant to the new Baileys-direct architecture, but worth knowing the old server-based path had an unresolved bug.
- The dashboard UI (Connect/Message/Contacts/Broadcast tabs, CSV upload with Phone/Name columns, live status table) was built and looked correct in screenshots.
- **v1.0.0 shipped and personally verified by Vishnu (2026-08-05)**: real WhatsApp QR scan on his own phone, real broadcast send to 3 real contacts (Venkat/Vishnu/Kumar), delays enforced (20-60s), messages confirmed received on WhatsApp. Mac DMG and Windows .exe both built. Windows .exe built via GitHub Actions CI but never actually run on a real Windows machine — still an open risk, see TEST_RESULTS.md/AUDIT_RESULTS.md in whatsapp-sender-app/.
- A pre-launch hardening pass was run (AI-trace scrub from git history + production edge-case audit: CSV edge cases, disconnect-mid-campaign handling, sql.js persistence/corruption recovery, license cache corruption, phone-number masking in logs). Results in `whatsapp-sender-app/AUDIT_RESULTS.md`.
- **Caution (2026-08-05)**: that same hardening pass rewrote git history in a way that moved the repo root up a folder level and deleted this file plus `CURSOR_BUILD_PLAN.md`/`START_HERE_FOR_CURSOR.md` as unwanted "AI trace" — they were never part of the GitHub repo and shouldn't have been touched. Restored from chat context. It also pulled the old obsolete `frontend/`, `scripts/`, `sample-contacts.csv`, and `test-contacts.csv` (containing Vishnu/Venkat/Kumar's real phone numbers) into the tracked GitHub repo — needs cleanup, only `whatsapp-sender-app/` should be tracked/pushed.

## Reusable assets still in this project folder

- `frontend/src/` — the tested React dashboard UI from the old build. Already ported into the Electron renderer; kept locally for reference only. Should NOT be tracked in the shipped product's git repo.
- `reference/bulkpro/` — the actual cloned BulkPro source (client/ + server/), the original reference for CSV parsing, `{Tag}` templating, and delay/batching logic. Correctly gitignored — do not commit.
- `scripts/`, `sample-contacts.csv`, `test-contacts.csv` — minor leftovers, low value; `test-contacts.csv` has real personal phone numbers and should never be committed/pushed.
- Deleted as obsolete: old Oracle/DigitalOcean hosting docs, `docker-compose.yml`, `.env`, old `backend/` (raw whatsapp-web.js Express server), old README/DEPLOY/RESULT files, `PRODUCT_SETUP.md`, `product-details.txt` (Lemon Squeezy plan, dropped).

## Files that define the current plan

- **`CURSOR_BUILD_PLAN.md`** — the full 7-task build plan (engine → CSV/messaging → sending → offline self-generated licensing → packaging/auto-update → mandatory end-to-end test → ship). All 7 tasks complete as of 2026-08-05.
- **`START_HERE_FOR_CURSOR.md`** — the single paste-into-Cursor prompt that ran all 7 tasks autonomously. Already used; kept for reference/future re-runs (e.g. a v2).

## Vishnu's stated working style (important, keep following this)

- Non-technical. Wants plain language, no unexplained jargon, short concrete steps.
- Wants responses **in bullet points**, not paragraphs (explicitly requested).
- Wants Cursor to run long autonomous stretches (hours) without stopping to ask questions — all ambiguous decisions should be resolved by Claude/Cursor proactively, not punted back to him mid-build.
- Cost-conscious, but priorities can shift (went from "must be free" to accepting paid hosting to abandoning hosting entirely) — always confirm current constraints rather than assuming old ones still hold.
- Wants AI involvement invisible in anything customer- or public-facing (code comments, git history, GitHub repo) — verify this stays true after any future Cursor pass.

## Immediate next step (as of 2026-08-05)

1. Fix repo scope: only `whatsapp-sender-app/` should be tracked/pushed to GitHub — remove `frontend/`, `scripts/`, `sample-contacts.csv`, `test-contacts.csv` from git tracking and history (the last one has real phone numbers).
2. Smoke-test the Windows `.exe` on a real Windows machine — never actually run yet.
3. Code signing still pending for both platforms (optional, can launch without it).
4. Otherwise: ready to sell on Mac today.
