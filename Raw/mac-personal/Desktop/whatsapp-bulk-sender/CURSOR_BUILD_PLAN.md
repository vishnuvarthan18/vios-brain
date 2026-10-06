# WhatsApp Bulk Sender — Sellable Desktop App — Build Plan for Cursor

Product: a Windows/Mac desktop app, sold as either a one-time purchase or a monthly subscription, that a customer installs on their own computer, links their own WhatsApp, and uses to send personalized bulk messages safely. No server, no hosting cost for us — each customer runs their own copy.

Hand these tasks to Cursor **in order**, back to back — see `START_HERE_FOR_CURSOR.md` for the single autonomous prompt covering all of them plus the "no questions asked" rules.

Status as of 2026-08-05: all 7 tasks below are complete and shipped (v1.0.0). Kept here as a reference for future versions/fixes.

---

## Before starting: reuse what already works

Two folders in this project are reference material, not obsolete — read them before writing new code:
- `frontend/src/` — our own tested React dashboard (Connect/Message/Contacts/Broadcast tabs, CSV upload, live status table), built against the old Docker server. Port its component structure and UX into the new Electron renderer.
- `reference/bulkpro/` — the original open-source BulkPro project this was based on (client/ + server/). Its CSV parsing, `{Tag}` templating, and anti-ban delay logic (20-60s random delay, 50-message/5-minute batch cooldown) were already proven working end to end in real testing.

Don't redesign any of this from theory — port the logic and UI patterns from both folders, and rewire only the parts that talk to WhatsApp/the database (swap Evolution API's REST calls for direct Baileys calls, swap Postgres for SQLite). Neither folder should ever be tracked in the shipped product's git repo — reference only.

## Task 1 — Project scaffolding + core WhatsApp engine

**Goal:** an Electron app skeleton that can show a QR code and connect to WhatsApp, with all data stored locally — no Docker, no external database.

```
Create a new project called whatsapp-sender-app with:
- An Electron main process (Node.js) that embeds Baileys (@whiskeysockets/baileys, MIT licensed) directly — no whatsapp-web.js, no Puppeteer, no Docker.
- SQLite (via better-sqlite3 or similar) as the only data store, bundled inside the app — tables for: contacts, campaigns, messages (status log), license (cached license state).
- A React-based renderer UI (Electron + React + Vite), single window, with a sidebar for: Connect, Message, Contacts, Broadcast — same shape as the earlier prototype's UI.
- On the Connect screen: generate and display the WhatsApp QR via Baileys' pairing flow, show live status (Disconnected / Scan QR / Connected), and persist the session locally (Baileys' built-in auth state saved to disk) so it survives app restarts without rescanning.
- package.json scripts for `npm run dev` (local development) and a placeholder `npm run build` (wired up fully in Task 6).
```

**Done when:** running the app locally shows a real scannable WhatsApp QR code, and scanning it shows "Connected" and survives an app restart without re-scanning.

---

## Task 2 — CSV upload, message editor, contacts

**Goal:** same contact/message experience as the earlier prototype, now single-tenant (one WhatsApp account per install, no company/slug concept needed).

```
Add to the app:
- A CSV/Excel upload zone (Contacts screen) using a library like PapaParse/SheetJS, requiring at minimum Name and Phone columns (case-insensitive matching), sanitizing phone numbers to digits-only with country code, storing rows in the local SQLite contacts table, showing validation errors per row.
- A message editor (Message screen) with a textarea, {Tag} placeholders auto-detected from the uploaded file's columns, a live preview rendered against the first contact, and support for one optional media attachment (image/video/document) per campaign.
- Store each drafted message as a "campaign" in SQLite (message_template, optional media path, created_at).
```

**Done when:** a CSV can be dragged in, parsed, and previewed; a message can be written with tags pulled from the actual uploaded columns and shows an accurate live preview.

---

## Task 3 — Sending engine with anti-ban delays and batching

**Goal:** the actual safe-sending logic, unchanged in spirit from the earlier prototype, now calling Baileys directly instead of a hosted API.

```
Add a Broadcast screen and matching main-process logic:
- On "Send", create one row per contact in the messages table (status=pending), then process them one at a time via Baileys' send-message function.
- Random delay between 20 and 60 seconds between messages (configurable via a settings screen, defaults as stated).
- After every 50 sent messages, pause 5 minutes before continuing.
- Update each message's status (sent/failed + error) in SQLite as it's processed, and push live updates to the renderer UI (Electron IPC) so the status table updates without polling.
- Pause/Resume/Stop controls that actually stop the background loop, not just hide it in the UI.
```

**Done when:** sending a small test campaign (3-5 real contacts) shows visible delays, correct pacing, and the live status table updates accurately as each message resolves.

---

## Task 4 — Licensing: offline, self-generated keys (no Lemon Squeezy, no internet)

**Goal:** the app only runs with a valid license key, but the whole flow is fully offline. Vishnu collects payment manually (UPI/bank transfer, outside the app) and generates the key himself with a small separate tool — no payment processor, no online account, no server-side validation.

```
Build two things:

1. A standalone key-generator (a simple local CLI script, e.g. `keygen.js` run via `node keygen.js`, NOT bundled inside the customer-facing app). It takes:
   - license type: onetime | subscription
   - for subscription: validity length in days (e.g. 30)
   - optional buyer name/email, for Vishnu's own record-keeping only
   It outputs a license key string encoding: license type, expiry date (subscription only — onetime has none), and an HMAC-SHA256 signature over that data using a secret embedded as a constant in both this script and the shipped app. Encode the whole thing (type + expiry + signature) as a compact base32/base64 string so it's easy to paste/email/WhatsApp to a buyer.

2. In the shipped app, a license-activation screen shown before the main app if no valid license is cached locally:
   - User pastes their key.
   - App decodes it and recomputes the HMAC locally using the same embedded secret — reject if the signature doesn't match (tampered/invalid key).
   - Onetime keys: no expiry field — once the signature checks out, cache "valid forever" locally, never check again.
   - Subscription keys: expiry date is embedded in the key itself — on every app launch, compare it to the system clock (no internet call, ever). If expired, lock the app back to the license screen with a clear "your subscription has expired, contact Vishnu to renew" message. Renewal = Vishnu generates and sends a fresh key with a new expiry; user re-enters it.
   - No activation-limit enforcement — this is an accepted tradeoff of going fully offline (a key could technically be shared; mitigated socially, not technically, since keys are hand-delivered to known buyers). Note this explicitly in README.md.
   - Never hardcode a bypass, backdoor, or dev key that skips validation in the shipped build.
```

**Done when:** a key generated by `keygen.js` (onetime) unlocks the app and survives a restart with no internet connection at all; a subscription key generated with a 1-minute test expiry (temporarily, for testing) correctly locks the app back out once expired, with the real default set back to days/months before finishing; an invalid/tampered key is rejected with a clear message.

---

## Task 5 — Package as installers + auto-update

**Goal:** a real Windows `.exe` and Mac `.dmg` a customer can download and double-click, plus a way to push fixes later without customers manually reinstalling.

```
Set up electron-builder to produce:
- A Windows NSIS installer (.exe)
- A Mac .dmg
Configure electron-updater so the app checks a hosted "latest version" manifest on launch and offers to download/install updates — point it at GitHub Releases on a repo for this project (electron-builder + electron-updater support this natively via the "publish" config), so shipping an update later is just publishing a new GitHub Release.

If a code-signing certificate is available locally, sign the installers; otherwise ship unsigned and note in RESULT.md that customers will see an "unknown publisher" warning until a certificate is added. No account/API credentials needed for this task.
```

**Done when:** `npm run build` (or equivalent) produces a working installer for at least the platform Cursor is running on, and the app has functioning auto-update-check wiring even if not fully exercised end-to-end.

---

## Task 6 — End-to-end test pass (self-run, don't ask the user to do this)

**Goal:** verify the whole product actually works before calling it done, the way a QA pass would before shipping v1.

```
Run through, and log results in TEST_RESULTS.md:
1. Fresh install (delete local SQLite/session data) → app launches → shows license screen (no license cached).
2. Generate a one-time key with keygen.js, activate with it → app unlocks → restart app, with no internet connection → still unlocked without re-entering the key.
3. Generate a subscription key with keygen.js using a short test expiry (e.g. 1 minute) → app unlocks → wait for expiry → relaunch → app correctly locks back to the license screen. Then confirm the real default (30 days or whatever was chosen) is restored before finishing.
4. Connect screen → real WhatsApp QR → scan with a phone → Connected → restart app → still connected without re-scanning.
5. Upload a small test CSV → write a message with a {Tag} → preview matches expectations.
6. Send to 2-3 real test contacts → confirm delays are visibly enforced, messages actually arrive on WhatsApp, and the status table updates correctly to Sent/Failed.
7. Paste a deliberately tampered/invalid key → confirm it's rejected with a clear error, not a crash.
8. Any failure at any step: fix it and re-run that step, don't skip forward with a known-broken piece.
```

**Done when:** all 7 steps pass and are logged with pass/fail + notes in TEST_RESULTS.md. Note: steps involving a real phone (QR scan, real send) must actually be run against a real device — an agent with no phone available should say so plainly rather than marking PASS on unverified automation.

---

## Task 7 — Ship it

**Goal:** Vishnu has everything he needs to sell this manually — the installer, the key-generator tool, and clear instructions for the sale-to-delivery flow.

```
1. Confirm the built Windows/Mac installer(s) run standalone (no dev environment needed) — this is the one file Vishnu sends to each buyer.
2. Confirm keygen.js runs standalone on Vishnu's machine (node keygen.js, no other setup) and produces valid onetime/subscription keys.
3. Write RESULT.md with: confirmation both pieces work end to end, a summary of the license flow (onetime vs subscription behavior, no activation-limit enforcement noted as a known tradeoff), and a short step-by-step "what I do for each sale" paragraph for Vishnu: collect payment manually → run keygen.js with buyer's chosen plan → send installer + key via email/WhatsApp → buyer pastes key into license screen.
4. Print a short final summary as your last message.
```

**Done when:** RESULT.md exists, the installer and keygen.js both run standalone, and the manual sale-to-delivery steps are written out clearly.
