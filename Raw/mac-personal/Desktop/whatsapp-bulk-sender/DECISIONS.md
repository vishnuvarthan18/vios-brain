# DECISIONS

Ambiguities resolved with the simpler/safer option during the autonomous build.

| Date | Decision | Why |
|------|----------|-----|
| 2026-08-07 | `BUILD_PLAN.md` was missing; authored at repo root from the run brief + consent-first guardrails, then treated as source of truth | Cannot follow a missing plan; writing it unblocks Phases 0–6 without mid-run questions |
| 2026-08-07 | Rebuild `whatsapp-sender-app/` in TypeScript (new `src/main|preload|renderer|shared` layout) rather than bolting TS onto legacy JS | Plan requires strict TS, safeStorage vault, health/suppression modules, and `npm run verify` |
| 2026-08-07 | Keep offline HMAC license secret compatible with v1 keys | Safer for existing buyers; no re-keying |
| 2026-08-07 | Use `sql.js` (WASM) instead of `better-sqlite3` | Prior native build failures on this machine; same local-only semantics |
| 2026-08-07 | Default transport in E2E/tests = `MockTransport`; production default = Baileys | Live QR needs user phone (BLOCKED); still prove Connect/send paths |
| 2026-08-07 | Warm-up always on; settings UI clamps to Section 7 floors | Guardrail: cannot disable or zero safety controls |
| 2026-08-07 | CSV import requires explicit consent attestation checkbox before commit | Consent-first product stance |
| 2026-08-07 | Code signing identity left `null` with env-hook placeholders | User certificates not available (known BLOCKED) |
| 2026-08-07 | `BLASTDESK_DELAY_SCALE` shortens waits in tests only; decideSend still applies Section 7 floors | Needed for CI campaign E2E without disabling safety math |
| 2026-08-07 | Dev/CI without `safeStorage` encryption writes `creds.dev.json` locally | Electron safeStorage unavailable in headless unit context; production uses encryptString |
| 2026-08-07 | Phase 5.5: hand-built shadcn-style primitives (no shadcn CLI / no Radix) | Offline-friendly; fewer deps; same visual API surface |
| 2026-08-07 | Phase 5.5: renderer-local `AppScreen` instead of extending shared `ScreenId` | Hard rule: do not modify `src/shared`; navigation is UI-only |
| 2026-08-07 | Phase 5.5: health-over-time via `localStorage` samples of `getHealth`/`onHealthUpdate` | No health history IPC; client derivation only |
| 2026-08-07 | Phase 5.5: Reports read/reply rates are labeled proxy estimates | API has no read receipts / replies; avoid inventing backend channels |
| 2026-08-07 | Phase 5.5: DM Sans + Fraunces via `@fontsource-variable` (bundled) | Expressive fonts without relaxing CSP for Google Fonts CDN |
