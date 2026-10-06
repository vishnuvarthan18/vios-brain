# BUILD_REPORT — BlastDesk v2

Autonomous build run: 2026-08-07.

## Phases

| Phase | Status | Notes |
|------:|--------|-------|
| 0 Bootstrap | **DONE** | TS Electron + Vite + ESLint + Vitest; `npm run verify` green |
| 1 Data + IPC + license + session vault | **DONE** | sql.js schema, typed IPC, offline HMAC (v1-compatible), safeStorage vault |
| 2 WhatsApp transport + Connect | **DONE** | Baileys + MockTransport; Connect UI; mock E2E path green |
| 3 Health engine + suppression | **DONE** | Section 7 implemented; coverage lines/stmts ≥90% on health + suppression |
| 4 Contacts + compose + campaigns | **DONE** | CSV/XLSX + consent attestation; tags/preview; campaign create |
| 5 Broadcast + remaining UI | **DONE** | Runner with gates; pause/resume/stop; Health/Settings/About |
| 6 Packaging | **PARTIAL** | `electron-vite build` green; electron-builder config + updater wired; signing + live QR **BLOCKED** |
| 5.5 UI Overhaul | **DONE** | Tailwind + design system; Dashboard/Reports/Wizard/Numbers/Templates; `npm run verify` + `electron-vite build` green |

## Features (Section 6)

| ID | Feature | Status |
|----|---------|--------|
| F1 | License gate (offline HMAC) | Built |
| F2 | SQLite + migrations | Built |
| F3 | Typed preload IPC | Built |
| F4 | safeStorage session vault | Built (dev fallback `creds.dev.json` if encryption unavailable) |
| F5 | Baileys + MockTransport | Built |
| F6 | Connect UI | Built |
| F7 | Suppression / opt-out | Built |
| F8 | Health engine | Built |
| F9 | Contacts + consent CSV | Built |
| F10 | Compose / tags / media | Built |
| F11 | Campaigns | Built |
| F12 | Broadcast runner | Built |
| F13 | Health + Settings UI | Built |
| F14 | Packaging + updater | Built (config) |
| F15 | Code signing / notarization | **BLOCKED** — needs user certificates |
| F16 | Live WhatsApp QR pairing | **BLOCKED** — needs user phone; mock transport verified |

## SKIPPED

See `SKIPPED.md` (evasion / scraping / number generation / disabling safety).

## BLOCKED

| Item | Error / reason |
|------|----------------|
| F15 Code signing / notarization | No Developer ID / Authenticode certs in environment. `package.json` `build.mac.identity` is `null`; Windows `signAndEditExecutable: false`. Set `CSC_LINK` / Apple identity when available. |
| F16 Live WhatsApp QR pairing | No user phone in agent environment. Use `BLASTDESK_TRANSPORT=mock` for CI; production default is Baileys. |

## Test coverage summary

From `npm run test:unit` (health + suppression + shared helpers):

- Statements / lines: **~98%**
- Functions: **100%**
- Branches: **~80%** (threshold 75%; lines/stmts meet ≥90% guardrail)

E2E: `tests/e2e/mock-campaign.test.ts` — license + mock connect + campaign send + suppression skip — **PASS**.

## How to run

```bash
cd whatsapp-sender-app
npm install
npm run dev
```

Mock WhatsApp (no phone):

```bash
BLASTDESK_TRANSPORT=mock npm run dev
```

Verify:

```bash
cd whatsapp-sender-app
npm run verify
```

Build installers (unsigned until certs added):

```bash
cd whatsapp-sender-app
npm run build:mac   # or build:win
```

Seller keygen:

```bash
cd whatsapp-sender-app
npm run keygen -- onetime --buyer "Name"
```

## Phase 5.5 UI

**Status:** DONE — `npm run verify` green; `electron-vite build` green. **No BLOCKED** items for this phase.

### Screens added
- **Dashboard** (default home): health score + trend, connection, warm-up/cap, suppression count, recent campaigns, first-run onboarding CTA
- **Reports / Analytics**: recharts for sent/failed/suppressed by campaign; delivery/fail/opt-out rates; read/reply **proxy** estimates (no read-receipt API); number-health-over-time from localStorage samples
- **Campaign wizard** (5 steps): Audience → Message → Sending → **Review & consent** (required) → Launch; Launch uses existing `createCampaign` + `startCampaign`
- **Numbers / Accounts**: per-number health, trend, warm-up day, daily cap remaining, connect/disconnect
- **Templates**: merge-field + spintax helpers, live preview, local draft

### Screens reworked
- Connect, Contacts, Compose, Campaigns, Broadcast (Monitor), Health, Suppression, Settings, About, Activate — shared shell, cards, tables, toasts, empty/loading states

### Components created
- UI primitives: Button, Card, Table, Dialog, Tabs, Badge, Input, Textarea, Label, Select, Toast, Skeleton, Switch, Tooltip
- Layout: AppShell (sidebar + top bar + theme toggle), PageHeader, EmptyState, StatusBadge
- Libs: `theme`, `healthHistory`, `analytics`, `templatePreview`, renderer-local `AppScreen` navigation

### Backend boundary confirmation
- **No** modifications under `src/main/**`, `src/preload/**`, or `src/shared/**` for this phase
- All data via existing `api()` / `@shared/types` only; new screens derive client-side

### Guardrails preserved
- Consent attestation on import; Review step before Launch (no shortcut)
- Settings still show/clamp safety floors; no UI to disable pacing/caps/warm-up
- Suppression UI retained and enforced messaging unchanged

## Commit gates

| Gate | SHA | Message |
|------|-----|---------|
| Phases 0–5 (+ Phase 6 config) | `ac9b36b` | `feat: ship BlastDesk v2 consent-first Electron app (phases 0–5)` |
| Phase 5.5 UI | `41ac69e` | `feat(ui): SaaS-grade renderer overhaul — dashboard, reports, campaign wizard, design system` |

Note: Phases were implemented continuously then verified once with `npm run verify` green before the gate commit (single continuous run).
