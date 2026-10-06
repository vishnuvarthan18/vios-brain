---
tags: project
status: done
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# STATE: araCreate Academy — VCET bootcamp dashboard

Full history: [[Projects/ac-training/SUMMARY]] · Log: [[Projects/ac-training/LOG]] · Dev: [[Projects/ac-training/DEV-LOG]]

## Where we are (as of 2026-10-06)
- Bootcamp at [[Companies/Velalar College of Engineering and Technology]] ran 18–26 Sep 2026. Project done; clean-up left. Last work 28 Sep.
- 206 students / 52 teams after test data removed (earlier 209 / 53 incl. test team) (to confirm).
- Dashboard still live at https://vcet.aracreate.academy (React v3, Node 22 / Postgres 16 / Caddy on Hetzner — to confirm).
- Final ranking set 26 Sep (to confirm which is official): top score ECE-T28 Link Force 220.0; hand-set top four ECE-T36-SWITCHSQUAD + EEE-T02-COREX joint 1st, ECE-T34-CODETEAM 3rd, EEE-T09-SPARKX 4th.
- Certificates open to students; 38 of 206 downloaded by 28 Sep.
- Client pack ready: final project photos (50 of 52 teams) + team details xlsx. Training report: delivered 28 Sep or still to do (to confirm).
- Secrets exposed in chats not yet confirmed rotated.

## Next steps
1. Merge the evaluation branch into `dev` before any new deploy.
2. Confirm official winner and client report status.
3. Rotate staff password and Google service-account key.
4. Remove project marks from admin exports if they go to staff.
5. Decide keep / archive / shut down the server; delete test team data.
6. Merge `dev` into `main`, update schema.sql, fix deploy.md rsync trap.

## Blockers
- None (bootcamp over). Only Vishnu's own clean-up jobs.

## Key places
- Live: https://vcet.aracreate.academy
- Repo: https://github.com/aracreate-group/aca-bootcamp-dashboard (dev)
- Mac: ~/araCreate/bootcamp-dashboard
- Server: Hetzner aca-htz-vcet (ssh `hetzner`), /opt/bootcamp-dashboard
- Drive: Shared Drive ac-vcet
- Client folder: "Bootcamp Final Projects - 2026"
