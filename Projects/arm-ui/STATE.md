---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: arm-ui (araMetrics platform UI/UX)
Summary: [[Projects/arm-ui/SUMMARY]] · Log: [[Projects/arm-ui/LOG]] · Dev: [[Projects/arm-ui/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Active (earlier: "paused since 2026-07-14" — wrong; repo work ran 08-29 → 10-05).
- 2026-10-05: new `timer` app started in ARM (stopwatch with ARM UI components); runs standalone, not tested in the shell.
- arm-website PR #1 (`feat/landing-content-pass` → `main`) and arm-service-notification PR #1 (into `dev`) open since 2026-09-03.
- arm-core-fe landing page on `feature/aramerics-landing-page` (2026-08-30).
- v1 UI/UX pipeline: Stages 1–4 locked; Stage 5/6 in progress (Radix theme + Core shell sidebar built); login/sign-up on hold.

## Next steps
1. Timer: add task/project label; test inside the shell.
2. Get the two open PRs reviewed and merged.
3. Website: real contact-form endpoint; GDPR legal pages + Impressum.
4. Move apps from `@aracreate/test-arm-ui` to `@aracreate/arm-ui`.
5. v1 UI: decide self-serve sign-up vs admin-only provisioning; then top bar, Home payloads, Admin Users slice, Calendar Merger polish.

## Blockers
- Backend not running locally (timer can't be tested in the shell).
- Self-serve sign-up not reconciled with the spec.
- `admin-portal-proposal.md` missing from project files.

## Key places
- ARM repos: `~/araCreate/ARM` (10 repos + arm-website, timer at `localhost:10001/timer/`)
- v1 UI code: Next.js repo `arametrics` (path not known); spec `support/spec.md`
- Dev domain: dev.arametrics.app
