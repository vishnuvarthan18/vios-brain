# Atlas session checkpoint — 14 August 2026 (updated: repo now live on GitHub)

Read this first when picking this back up. Chronological summary of this session's work.

## Canonical remote — use this from now on
**https://github.com/vishnuvarthan18/sathyamangalam-atlas** (private repo)

This is now the single source of truth for code, data, and project docs
(the repo has its own `docs/` folder mirroring this project's checkpoint
docs — read that too, especially on a session running under a different
Claude account, since Claude project memory doesn't cross accounts).

## Related docs from this session
- `sathyamangalam/data-bundle-integration-2026-08-14.md` — the initial data bundle review + first site update (hardcoded HTML edits)
- `sathyamangalam/git-pipeline-2026-08-14.md` — the git repo + reproducible export pipeline build

## What happened this session, in order
1. Reviewed the full `atlas.db` data bundle (89 places, 1,893 taxa, 74,900 occurrences, 7,190 documents / 928 tier A+B, 82 claims, 4 conflicts, 30.3% coverage).
2. Updated `sathyamangalam-atlas.html` in place with honest coverage numbers, the 4 disputed figures shown side by side, gazetteer/document gap disclosures.
3. Built a proper git repository: `data/atlas.db` as source of truth, `scripts/export_from_db.py` as the only way to regenerate `exports/*.json`, `site/index.html` rewritten to fetch data at runtime instead of hardcoding it. Found and fixed a real bug — the places table has 89 rows but every prior export only surfaced 19. Coverage recomputed to 31.5%.
4. Discussed Astro migration — recommended holding off; bottleneck is data coverage, not tooling. Parked, not rejected.
5. Discussed launch readiness — hard blocker is still domain registration (sathyamangalam.org not registered). Soft blockers (coverage, disputed figures, robots.txt exemption, hosting) are launchable-with-caveats since the site discloses its own gaps honestly.
6. Vishnu clarified his setup: two Macs (office + personal), two separate Claude accounts, office git policy of never pushing from a Claude session there. Resolution: GitHub is the shared source of truth for code; `docs/` inside the repo is the shared source of truth for context (since Claude project memory is per-account).
7. **This sandbox cannot push to arbitrary GitHub repos** — its network access to github.com is locked to a narrow pre-configured Claude Code Action integration ("sessions are bound to their configured repositories"), confirmed via `gh api user/repos` returning a 403 with that exact message. This is a hard environment limitation, not a credentials problem — getting a token doesn't fix it. Delivered the repo as a zip instead (`sathyamangalam-atlas-repo-with-git.zip`, one clean commit, `docs/` folder included).
8. Walked Vishnu through downloading the zip (had to re-send once — first file card wasn't visible to him), unzipping, `gh auth login` (device-code browser flow — worked cleanly with GitHub.com → HTTPS → Y → "Login with a web browser"), and `gh repo create sathyamangalam-atlas --private --source=. --remote=origin --push`.
9. **Push succeeded.** Repo is live at https://github.com/vishnuvarthan18/sathyamangalam-atlas (private).

## State of the repo right now
- Live on GitHub, private, pushed from Vishnu's personal Mac.
- One commit on `main`, tracking `origin/main`.
- Office Mac should `git clone https://github.com/vishnuvarthan18/sathyamangalam-atlas.git` — pull-only, per Vishnu's policy, never push from there.
- Fully functional locally: `python3 scripts/export_from_db.py && python3 -m http.server 8000`, open `http://localhost:8000/site/`.

## Immediate next actions (pick up here)
1. **Register sathyamangalam.org** (or decide on an alternative domain) — the actual next unblocking step, ~5 minutes, ~$10-15/year. Still the standing #1 blocker.
2. Clone the repo on the office Mac (pull-only) so both machines are in sync.
3. Set up hosting — Cloudflare Pages or Netlify, both free-tier, both git-integrated, both can deploy straight from the new GitHub repo once the domain exists.
4. Two long-standing open decisions: (a) authoritative source for the 4 disputed figures (core area, elephants, leopards, tigers), (b) written confirmation of the robots.txt exemption for 5 API hosts.
5. Astro migration is parked — revisit when the HTML becomes unwieldy to hand-edit or someone besides Vishnu starts contributing.
6. media/community domains in `atlas.db` have no backing tables yet — schema decision needed before next harvest cycle.

## Don't re-ask
- Whether to use Astro now — already answered (no, not yet).
- Whether to push from this session's machine — already answered (no; also technically impossible from this sandbox regardless).
- Git target — resolved: https://github.com/vishnuvarthan18/sathyamangalam-atlas, private, pushed from personal Mac.
- How to authenticate `gh` on a new machine — `gh auth login`, pick GitHub.com → HTTPS → Y → "Login with a web browser", it prints a one-time code and opens github.com/login/device.
