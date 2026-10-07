# Phase 5.5 — UI Overhaul (paste this whole thing into Cursor)

Overhaul the BlastDesk renderer UI to a modern, SaaS-grade product. Run **autonomously, end to end, no questions**. Backend is done and correct — **do not change it**.

## HARD SCOPE RULES
- **Only touch `whatsapp-sender-app/src/renderer/**` and renderer build/config (Tailwind, PostCSS, `index.html`).**
- **Do NOT modify** anything in `src/main/**`, `src/preload/**`, `src/shared/**`, the health engine, suppression gate, runner, DB, license, or IPC channel definitions.
- **Reuse the existing data layer as-is:** all data comes through `src/renderer/lib/api.ts` (the `api()` object) and types from `@shared/types`. If a screen needs data the API doesn't expose, derive it on the client from existing calls — do NOT add new IPC channels.
- Keep every guardrail: consent attestation on import stays, safety-floor settings stay clamped (read-only display of floors is fine), suppression/opt-out UI stays. Never add UI to disable pacing, caps, or warm-up.
- After work, `npm run verify` must be green. Self-correct up to 5 times, then mark BLOCKED in `BUILD_REPORT.md` and continue. Commit when green.

## DESIGN SYSTEM (build first)
- Add **Tailwind CSS** + **PostCSS** to the renderer. Add **shadcn/ui**-style primitives (or hand-built equivalents using Radix if shadcn CLI can't run offline): Button, Card, Table, Dialog, Tabs, Badge, Input, Select, Toast, Skeleton, Switch, Tooltip.
- Add **lucide-react** for icons and **recharts** for charts.
- Tokens: neutral gray surfaces, restrained WhatsApp teal/green accent, `rounded-lg`, soft shadows, 8pt spacing, clear type scale.
- **Light + dark mode** with a theme toggle (persist choice; no browser localStorage restriction here — this is Electron renderer, use it).
- Global app shell: left sidebar nav + top bar, consistent page header pattern, empty states, loading skeletons, and toasts on every async action.
- Replace the old `styles/app.css` look; keep only what's still needed.

## SCREENS
Enhance the existing pages (`src/renderer/pages/*.tsx`) and add the missing ones. Keep their existing `api()` calls; just rebuild the presentation.

**Rebuild (already exist):** Connect, Contacts, Compose, Campaigns, Broadcast, Health, Suppression, Settings, About, Activate — into polished, consistent components with the new design system.

**Add (missing — the priority):**
1. **Dashboard (new home screen):** number-health cards (score + trend from health data via `api()`), recent campaigns, quick "New campaign" CTA, opt-out/suppression counter, connection status. This becomes the default route.
2. **Reports / Analytics (new):** charts via recharts — delivery/sent/failed, read rate, reply rate, opt-out rate, and **number-health-over-time** (the key differentiator). Derive series from existing campaign/delivery/health data exposed by `api()`.
3. **Campaign Wizard (rework Compose+Campaigns into a 5-step flow):** Audience → Message → Sending settings (account + pacing profile + schedule) → **Review & consent check** (show audience size, how many are suppressed/excluded, estimated duration, consent-source warnings) → Launch. Wire Launch to the existing broadcast/runner API used by BroadcastPage.
4. **Numbers / Accounts (new or expand Health):** per-number health score + trend, warm-up stage, daily cap remaining, connect/disconnect.
5. **Templates (new or split from Compose):** editor with merge-field `{{name}}` + spintax `{a|b}` helpers and a **live preview** pane against a sample contact.

## UX GUARDRAILS
- The campaign flow must always pass through the suppression/consent **Review** step before Launch — no direct-launch shortcut.
- Health and "why a number/campaign paused" must be visible at a glance (dashboard + monitor).
- First-run onboarding: from Connect to first campaign in under 5 minutes.
- Every list has an empty state; every async action has loading + error + toast.

## ACCEPTANCE (auto-verify, then finish)
- `npm run verify` green (lint + typecheck + unit + e2e). Do not weaken existing tests.
- `electron-vite build` green.
- App runs with `BLASTDESK_TRANSPORT=mock npm run dev`; every screen renders with real data or a proper empty state; dark mode works.
- Append a **"Phase 5.5 UI" section to `BUILD_REPORT.md`**: screens added/reworked, components created, any BLOCKED item, and confirmation that no `src/main|preload|shared` files changed.
- Commit: `feat(ui): SaaS-grade renderer overhaul — dashboard, reports, campaign wizard, design system`.

Begin now with the design system + app shell, then Dashboard, then Reports, then the Campaign Wizard, then the rest. Keep going until done.
