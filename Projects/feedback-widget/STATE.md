---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: Feedback widget for B. Halle

Full history: [[Projects/feedback-widget/SUMMARY]] · Log: [[Projects/feedback-widget/LOG]] · Dev: [[Projects/feedback-widget/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Live at https://apps.b-halle.de on the new IONOS server (`webapp` box) since 30 Sep; latest deploy `d9b6d11` (6 Oct).
- (earlier: feedback.arametrics.app on old server 212.227.213.174 — retired.)
- Webflow site halle-dev loads the widget from apps.b-halle.de/v1.js.
- Screenshots taken on our server, ~0.5 s; red box correct on phone and long pages.
- Admin: popup with Activity timeline, Tracked tabs, Classifications, Users (admin only), Settings; strict bug flow Processing → Fixed → Closed.
- 99 Halle pages (EN + DE). Logins: Vishnu (admin), Jakob, Rahul, Shyam.
- Code on GitHub `dev` (repo `halle-app-widget`).
- Old server's feedback service was restarted by mistake on 5 Oct — needs stop + disable.

## Next steps
1. Revoke GitHub token in the git remotes on both servers; use a deploy key.
2. Stop + disable the feedback service on the old server.
3. ~14 Oct: old-server log check + final archive; then Jakob cancels old contract.
4. Decide admin for others; make Classifications admin-only.
5. Safari/iPad test; check login open redirect; update local git remote.

## Blockers
- None hard. Old-server cancel waits for the ~14 Oct clean-days check.

## Key places
- Live: https://apps.b-halle.de · Test site: https://halle-dev.webflow.io
- Repo: github.com/aracreate-group/halle-app-widget (branch `dev`); Mac `~/araCreate/HLE/testing_widget/halle-feedback-widget`
- Server: new IONOS server 217.160.93.75 (boxes `webapp`, `jupyter`)
- Rule: check viOS STATE before touching any server.
