# BlastDesk v2 — BUILD_PLAN

Consent-first WhatsApp broadcast desktop app (Electron). Source of truth for Phases 0→6.

**Product:** BlastDesk — local-first, single-operator desktop app. Customer links their own WhatsApp, imports consenting contacts, and sends personalized broadcasts under hard pacing / warm-up / opt-out gates.

**Hard product stance:** Consent-first, not evasion. No detection-evasion, fingerprint spoofing, contact scraping/harvesting, or number generation.

---

## 1. Guardrails (non-negotiable)

1. Suppression / opt-out gate runs on **every** send candidate. Never bypassable.
2. Pacing floors, daily caps, and warm-up **cannot** be disabled or set to zero (UI + engine clamp).
3. TypeScript `strict`, no `any`.
4. WhatsApp session tokens stored **only** via Electron `safeStorage` (encrypted at rest).
5. Renderer talks to main **only** through `contextBridge` preload API.
6. Health engine + suppression modules: **≥90%** unit test coverage.
7. `npm run verify` = lint + typecheck + unit + relevant E2E. Green gate after each phase.

---

## 2. Repo layout

```
whatsapp-sender-app/          # shipped product (app root)
  package.json
  tsconfig.json               # shared strict baseline
  tsconfig.node.json
  tsconfig.web.json
  vitest.config.ts
  playwright.config.ts
  electron.vite.config.ts     # or vite + tsc equivalent
  .eslintrc.cjs
  src/
    main/                     # Electron main process
      index.ts
      ipc/
      db/
      whatsapp/               # Baileys + MockTransport
      health/                 # Sending-health engine (Section 7)
      suppression/
      sender/
      license/
      session/                # safeStorage-backed auth
    preload/
      index.ts                # contextBridge only
    renderer/                 # React UI
      App.tsx
      pages/
      components/
      styles/
    shared/                   # types + pure utils (no Node/Electron)
      types.ts
      constants.ts
      tags.ts
  tests/
    unit/
    e2e/
  scripts/
    keygen.ts                 # seller-only license generator
    verify.sh
```

Top-level tracking (repo root, committed):

- `BUILD_PLAN.md` (this file)
- `BUILD_REPORT.md`
- `DECISIONS.md`
- `SKIPPED.md`
- `.cursorrules`

---

## 3. Data model (SQLite via sql.js)

| Table | Purpose |
|-------|---------|
| `settings` | key/value; pacing floors, caps, warm-up day, default CC |
| `license` | cached offline license state |
| `contacts` | phone (unique), name, tags JSON, consent_attested_at, source, created_at |
| `suppression` | phone (unique), reason (`opt_out`\|`manual`\|`bounce`\|`complaint`), created_at, note |
| `campaigns` | id, name, template, media_path, status, created_at, started_at, finished_at |
| `messages` | id, campaign_id, contact_phone, body, status (`pending`\|`skipped_suppressed`\|`skipped_cap`\|`sending`\|`sent`\|`failed`), error, created_at, sent_at |
| `send_events` | id, at, phone_hash, campaign_id, outcome — for daily caps / warm-up counters |
| `health_snapshots` | id, at, score, state, reasons_json, sent_today, warm_up_day |

Indexes: `contacts(phone)`, `suppression(phone)`, `messages(campaign_id, status)`, `send_events(at)`.

---

## 4. Feature modules (map to Section 6)

| ID | Module | Phase |
|----|--------|-------|
| F1 | App shell + license gate (offline HMAC) | 0–1 |
| F2 | SQLite persistence + migrations | 1 |
| F3 | Preload IPC bridge (typed) | 1 |
| F4 | Session vault (`safeStorage`) | 1–2 |
| F5 | WhatsApp transport (Baileys) + MockTransport | 2 |
| F6 | Connect / QR / connection status UI | 2 |
| F7 | Suppression list + opt-out gate | 3 |
| F8 | Sending-health engine (Section 7) | 3 |
| F9 | Contacts CSV import + consent attestation | 4 |
| F10 | Message editor + `{Tag}` preview + media | 4 |
| F11 | Campaign create / list | 4–5 |
| F12 | Broadcast runner (pause/resume/stop) enforcing gates | 5 |
| F13 | Health dashboard + Settings (clamped) | 5 |
| F14 | Packaging + auto-update wiring | 6 |
| F15 | Code signing / notarization config | 6 (BLOCKED without certs) |
| F16 | Live QR pairing | 6 (BLOCKED without user phone) |

---

## 5. UI screens

1. **Activate** — paste license key
2. **Connect** — QR / status / disconnect
3. **Contacts** — CSV import, consent checkbox required, table, add-to-suppression
4. **Suppression** — list / add / remove (remove requires confirm)
5. **Compose** — template, tags, preview, media
6. **Campaigns** — list, create, open broadcast
7. **Broadcast** — start/pause/resume/stop, live status table
8. **Health** — score, state, warm-up day, sent today / cap, reasons
9. **Settings** — delays/caps/warm-up with hard floors; reset local data
10. **About** — BlastDesk branding

Sidebar navigation after license unlock.

---

## 6. Feature checklist (Section 6 — for BUILD_REPORT mapping)

- [ ] F1 License gate
- [ ] F2 SQLite + migrations
- [ ] F3 Typed preload IPC
- [ ] F4 safeStorage session vault
- [ ] F5 Baileys + MockTransport
- [ ] F6 Connect UI
- [ ] F7 Suppression / opt-out
- [ ] F8 Health engine
- [ ] F9 Contacts + consent CSV
- [ ] F10 Compose / tags / media
- [ ] F11 Campaigns
- [ ] F12 Broadcast runner
- [ ] F13 Health + Settings UI
- [ ] F14 Packaging + updater
- [ ] F15 Signing config (BLOCKED)
- [ ] F16 Live QR (BLOCKED)

---

## 7. Sending-health engine (spec)

### 7.1 Goals

Protect the operator’s WhatsApp account by **enforcing** safe send rates and consent. Not stealth. Not evasion.

### 7.2 Inputs

- `now: Date`
- `settings`: paced configuration (clamped)
- `sentTodayCount`
- `accountCreatedAt` / first-connect day → `warmUpDay` (1-based, saturates)
- `recentFailureRate` (last N attempts)
- `suppression.has(phone)`
- candidate message `{ phone, campaignId }`

### 7.3 Hard floors (cannot go below; UI cannot set 0)

| Setting | Floor | Default |
|---------|------:|--------:|
| `minDelaySec` | 20 | 20 |
| `maxDelaySec` | ≥ minDelaySec | 60 |
| `batchSize` | 10 | 50 |
| `batchPauseSec` | 60 | 300 |
| `dailyCap` | 20 | 100 |
| `warmUpEnabled` | always true | true |

Warm-up daily caps (override `dailyCap` when lower):

| Day | Cap |
|----:|----:|
| 1 | 20 |
| 2 | 30 |
| 3 | 40 |
| 4 | 50 |
| 5 | 60 |
| 6 | 80 |
| 7+ | `dailyCap` (still ≥ 20) |

### 7.4 Decision API

```ts
type HealthState = 'healthy' | 'throttled' | 'paused' | 'blocked';

interface HealthDecision {
  allow: boolean;
  delayMs: number;          // ≥ minDelaySec * 1000
  state: HealthState;
  score: number;            // 0–100
  reasons: string[];
  effectiveDailyCap: number;
  sentToday: number;
  warmUpDay: number;
}

decideSend(input): HealthDecision
```

Rules (evaluated in order):

1. If suppressed → `allow:false`, reason `suppressed`.
2. If `sentToday >= effectiveDailyCap` → `allow:false`, state `paused`, reason `daily_cap`.
3. If `score < 25` → `allow:false`, state `blocked`, reason `health_blocked`.
4. If `score < 50` → state `throttled`; delay = max(computed, minDelay*1.5).
5. Else state `healthy`; delay = random uniform `[minDelaySec, maxDelaySec]` seconds.
6. Batch pause: after every `batchSize` successful sends in campaign, next delay ≥ `batchPauseSec`.

Score starts at 100; subtract for recent failures, disconnects, and high velocity. Never used to hide automation — only to slow/stop sending.

### 7.5 Suppression gate

```ts
assertNotSuppressed(phone): void // throws / returns skip
```

Caller **must** invoke before every transport send. Runner marks `skipped_suppressed` if blocked.

### 7.6 Tests (≥90% coverage on `health/` + `suppression/`)

- Floors clamp zero/negative settings
- Warm-up caps by day
- Daily cap blocks
- Suppression always blocks
- Throttle increases delay
- Batch pause applied
- Score → blocked threshold

---

## 8. Phases

### Phase 0 — Bootstrap

- Author `.cursorrules`, tracking docs stubs
- TypeScript Electron + React + Vite scaffold
- ESLint, Vitest, Playwright smoke
- `npm run verify` script
- Empty shell window with BlastDesk title

**Done when:** `npm run verify` green on empty/smoke tests.

### Phase 1 — Data + IPC + license + session vault

- sql.js schema + migrations
- Typed IPC channels
- Offline HMAC license (port from v1)
- `safeStorage` session vault API (round-trip unit test with mock when safeStorage unavailable in CI)

**Done when:** verify green; license activate/reject unit tests pass.

### Phase 2 — WhatsApp transport

- `WhatsAppTransport` interface
- `BaileysTransport` + `MockTransport`
- Connect screen wired to mock in E2E / real Baileys in prod
- Session restore via vault

**Done when:** verify green; mock QR → connected E2E path passes. Live phone QR marked BLOCKED later.

### Phase 3 — Health + suppression

- Implement Section 7 fully
- ≥90% coverage on those modules
- Settings clamp helpers

**Done when:** coverage gate + verify green.

### Phase 4 — Contacts + compose + campaigns

- CSV/XLSX import; Name+Phone required; consent attestation required before import commits
- Tag detection + live preview
- Optional media path per campaign
- Suppression management UI/API

**Done when:** unit + E2E import/preview green.

### Phase 5 — Broadcast + remaining UI

- Sender loop: suppression → health.decide → delay → transport.send → persist
- Pause / resume / stop
- Health + Settings + About screens
- Live status IPC events

**Done when:** mock-transport campaign E2E green; verify green.

### Phase 6 — Package

- electron-builder Mac/Win config
- electron-updater → GitHub Releases
- Code-signing / notarization env hooks — **BLOCKED** (user certs)
- Live QR pairing checklist — **BLOCKED** (user phone)
- Mock transport + packaging dry-run as far as possible

**Done when:** unsigned build config works on this machine OR documented; BLOCKED items logged.

---

## 9. Verify gate

```bash
npm run verify
# lint && typecheck && test:unit && test:e2e
```

After each phase: green → Conventional Commit → next phase. Red → fix ≤5 attempts → BLOCKED in BUILD_REPORT → continue.

---

## 10. Out of scope / SKIPPED by guardrail

- Browser fingerprint spoofing
- Proxy rotation for “ban evasion”
- Scraping WhatsApp contacts / groups for cold outreach
- Random phone number generation
- Disabling warm-up / caps / suppression

If a subtask requires any of the above: skip that subtask only, log in `SKIPPED.md`, continue.
