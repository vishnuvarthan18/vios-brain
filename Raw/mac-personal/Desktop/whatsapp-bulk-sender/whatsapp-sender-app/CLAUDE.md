# CLAUDE.md — BlastDesk (working context for Claude Code)

You are continuing an in-progress product. Read this first, then `BUILD_PLAN.md`, `BUILD_REPORT.md`, `SKIPPED.md`, and `DECISIONS.md` at the repo root before making changes.

## What this is
BlastDesk — a **consent-first desktop bulk-WhatsApp campaign tool** (Electron + React + TypeScript). Positioning: SaaS-grade desktop app that beats competitors on UI and a genuinely intelligent, consent-first **sending-health engine**. Official WhatsApp Business API is out of scope for v1 (future bridge only).

## Current state (as of last build)
- Phases 0–5 built; UI overhaul (Phase 5.5) done. `npm run verify` and `electron-vite build` are green.
- **Backend is solid and should not be casually changed:** health engine (`src/main/health/*`), suppression gate (`src/main/suppression/*`), broadcast runner (`src/main/sender/runner.ts`), SQLite-via-sql.js DB with atomic writes (`src/main/db/*`), license (`src/main/license/*`), session vault (`src/main/session/vault.ts`), typed IPC (`src/main/ipc/*`, preload).
- **UI:** design system + shell in `src/renderer/components/**`; pages in `src/renderer/pages/**` including Dashboard, Reports (recharts), Campaign Wizard (5 steps), Numbers, Templates.
- Transport: `src/main/whatsapp/` has `baileysTransport.ts` (real) + `mockTransport.ts`. Tests/CI use mock via `BLASTDESK_TRANSPORT=mock`.

## Stack (as actually built — note the deviations)
React 18 + TypeScript (strict) · electron-vite · Tailwind + hand-built shadcn-style UI (no Radix installed) · recharts · lucide-react · Baileys · **sql.js** (WASM, not better-sqlite3) · Vitest (unit + mock E2E; no Playwright). Don't "fix" these to match an older plan without a reason logged in `DECISIONS.md`.

## HARD GUARDRAILS — do not violate
- **Consent-first, not evasion.** Never build: detection-evasion, fingerprint spoofing, proxy rotation for ban-dodging, contact scraping/harvesting, or phone-number generation. If a task needs one, refuse it, note it in `SKIPPED.md`, continue.
- **Safety floors are never disableable.** Pacing, daily caps, and warm-up are clamped (`src/shared/settingsClamp.ts`) — no UI or setting may set them to zero or off.
- **Suppression/opt-out is a hard gate** on every send — never add a bypass.
- **Consent attestation** stays required on contact import.
- TypeScript strict, no `any`. Session tokens only via `safeStorage`. Renderer talks to backend only through the preload/IPC (`src/renderer/lib/api.ts`), never directly.

## How to run / verify
```bash
npm install
BLASTDESK_TRANSPORT=mock npm run dev   # run app with mock WhatsApp (no phone needed)
npm run verify                          # lint + typecheck + unit + e2e — must stay green
npm run build                           # electron-vite build
```
Keep `npm run verify` green. Health + suppression modules must stay ≥90% covered.

## Open items / priorities (in order)
1. **Live-QR smoke test** — connect a real number (drop the mock env), send to a tiny opt-in list, watch the health engine. This is the biggest unknown; the whole product premise is unproven live. (Needs the user's phone.)
2. **Windows packaging** — produce a distributable build; runs unsigned with a dismissible SmartScreen warning. macOS signing is parked for now.
3. **Backend QA** — add error-path tests around the runner and Baileys reconnect/session-expiry handling.
4. **UI polish** — edge cases, empty/error states, real-data QA on Dashboard/Reports.

## Working style
Work task by task. After each change run `npm run verify`; only commit when green (Conventional Commits). If a guardrail blocks a task, skip + log, don't work around it. Ask the user before anything that needs their machine (live QR, certificates, distribution).
