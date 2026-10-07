---
tags: project
---
# DEV LOG: araMetrics platform (ARM) (Claude Code sessions)

## What was built
- araMetrics landing page in `arm-core-fe` (`src/aramerics/`): big hero SVG, section spacing and responsive work, `dataset.svg` section. Pushed to branch `feature/aramerics-landing-page` (337 files).
- `arm-website` (araMetrics marketing site, Vite + React): content rewrite in human words (no dashes), cookie banner, hero SVG layout, animated SVG screens for Projects and Vendors (moving cursor, progress bars), fluid text sizes for small screens, calendar image, `#555555` as the only dark color, logo links home, legal pages open at top. PR #1 on `arm-website` (`feat/landing-content-pass` → `main`).
- `arm-service-notification`: 4 email templates restyled to araMetrics brand (Poppins Light/Regular, inline SVG logo, tagline "Your schedule, synced across every module."), `make preview`, 2 test bugs fixed. PR #1 into `dev`.
- Cloned 10 ARM repos into `~/araCreate/ARM` (arm-make, arm-docs, arm-cli, arm-app-admin, arm-app-calendar, arm-session, arm-core-fe, arm-core-be, arm-service-notification, arm-util-clockify).
- New `timer` app (module-federation remote, `localhost:10001/timer/`): basic stopwatch with `useTimer` hook, using ARM UI (`UIProvider` from `@aracreate/test-arm-ui` + Radix Themes). See also [[Projects/timer/SUMMARY]] (not sure it exists).

## Timeline (newest first)
- 2026-10-05 — Started `timer` app in ARM dev workspace; basic timer built with ARM UI components; typecheck clean.
- 2026-09-08 — Cloned 10 ARM repos.
- 2026-09-03 — Notification emails redesigned; PR #1 into `dev` on arm-service-notification.
- 2026-09-03 — Website content pass, legal page fixes, logo link; PR #1 on arm-website (no `dev` branch existed). Section 05 dataset SVG tried and reverted; calendar image moves and reverts.
- 2026-09-02 — Website: cookie banner, hero SVG moves, client logos moved out of first view, animated Projects/Vendors SVGs, responsive pass.
- 2026-08-31 — arm-website run locally; audit workflow.
- 2026-08-29 to 08-30 — Landing hero SVG in arm-core-fe sized and moved in small steps; `dataset.svg` section; pushed `feature/aramerics-landing-page`.

## Decisions
- 2026-08-29 — Do only what Vishnu asks, in exact pixel steps; no "outsmarting". #decision
- 2026-08-30 — Exclude the 21 MB "Figma design build request" folder from git. #decision
- 2026-09-03 — No dashes in copy; humanised text; only `#555555` as dark grey. #decision
- 2026-09-03 — Emails: Poppins Light + Regular only; logo inline SVG; do not change API, topics or triggers. #decision
- 2026-09-03 — Push to `dev`, never `main`, for the notification service. #decision
- 2026-09-03 — arm-website: feature branch + PR into `main` (no `dev` branch); unused `02 Vendors.svg` (1.2 MB) kept out of git. #decision
- 2026-10-05 — Timer app uses ARM UI components only. #decision

## State at last session (2026-10-05)
- Timer app: basic stopwatch runs standalone; not tested inside the shell (backend not running).
- arm-website PR #1 and notification PR #1 open (as of 2026-09-03).

## Open items
- Website contact form saves locally only — needs a real endpoint.
- Legal pages describe an Indian entity (DPDP Act) but company is German — needs GDPR rewrite and Impressum. Terms conflict with marketing.
- Emails: "araMETRICS" spelling in artwork; "just hit reply" vs "No Reply" sender; old subjects.
- `@aracreate/test-arm-ui` is deprecated, renamed `@aracreate/arm-ui`; all apps still on the old name.
- `pnpm` not on PATH on the Mac.
- Timer: add task/project label next (asked, not answered).

## Session index
- araCreate-ARM-dev__2026-10-05_43367165 (archived: Projects/arm-ui/claude-code/araCreate-ARM-dev__2026-10-05_43367165.md) — 2026-10-05 — new timer app with ARM UI components
- araCreate-ARM-dev__2026-10-05_43c896f3 (archived: Projects/arm-ui/claude-code/araCreate-ARM-dev__2026-10-05_43c896f3.md) — 2026-10-05 — title-only side session
- home__2026-09-08_df42755a (archived: Projects/arm-ui/claude-code/home__2026-09-08_df42755a.md) — 2026-09-08 — clone 10 ARM repos
- araCreate-ARM-docker-webiste-arm-website__2026-09-03_c5ee5b57 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-03_c5ee5b57.md) — 2026-09-03 — run website locally (already on :3100)
- araCreate-ARM-docker-webiste-arm-website__2026-09-03_9a333fc6 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-03_9a333fc6.md) — 2026-09-03 — content pass, legal pages, PR #1
- araCreate-ARM-docker-webiste-arm-website__2026-09-03_7e2d7142 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-03_7e2d7142.md) — 2026-09-03 — section 05 dataset SVG, reverted
- araCreate-ARM-docker-webiste-arm-website__2026-09-03_6d78c6a6 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-03_6d78c6a6.md) — 2026-09-03 — asked for Gmail template; none in repo
- araCreate-ARM-docker-webiste-arm-website__2026-09-03_570f9c1f (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-03_570f9c1f.md) — 2026-09-03 — responsive, calendar image position
- araCreate-ARM-docker-arm-service-notification__2026-09-03_aed6b2b2 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-arm-service-notification__2026-09-03_aed6b2b2.md) — 2026-09-03 — email templates redesign, PR into dev
- araCreate-ARM-docker-webiste-arm-website__2026-09-02_81bed4d1 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-09-02_81bed4d1.md) — 2026-09-02 to 09-03 — animated Projects/Vendors SVGs
- araCreate-ARM-docker-webiste-arm-website__2026-08-31_ebf2e4c3 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker-webiste-arm-website__2026-08-31_ebf2e4c3.md) — 2026-08-31 to 09-03 — run website, cookie banner, hero, responsive text
- araCreate-ARM-docker__2026-08-29_aa78b203 (archived: Projects/arm-ui/claude-code/araCreate-ARM-docker__2026-08-29_aa78b203.md) — 2026-08-29 to 08-30 — arm-core-fe landing hero SVG, push feature branch
