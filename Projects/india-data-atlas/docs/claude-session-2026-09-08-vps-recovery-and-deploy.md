# Session summary — 2026-09-08: VPS access recovery + full deploy

## What happened this session

Started from a stuck state: the previous overnight build session (D-53–D-65) had produced 7 new DB migrations, 9 new harvester/normalizer scripts, and new systemd units across 5 repos — all committed locally, but **nothing was deployed** because that session had no SSH path to the VPS at all.

### 1. Root-caused the "SSH doesn't work" problem
It was never a real block. In order, ruled out:
- OVH Edge Network Firewall — confirmed disabled, zero rules, not the cause.
- OVH account/login confusion — account is on the **US subsidiary** (`auth.us.ovhcloud.com`), billed in USD via HDFC card as "OVH US LLC," not Canada/UK/India as first suspected.
- Rescue mode (`root` + one-time password) confirmed the VPS and SSH itself work fine.
- **Actual cause**: the correct login is `ssh ubuntu@40.160.137.239`, not `root@...` — Ubuntu cloud images disable direct root password login by default. Once using the right username, SSH worked immediately, always had.
- Side finding: Claude's own cloud sandbox and the `device_bash` bridge to the user's Mac both have restricted/no general network egress (no raw TCP to arbitrary hosts/ports) — this made the problem look worse than it was. Real fix required using the user's own native Mac Terminal.

### 2. Fixed "work only exists locally" risk (flagged in prior handoff as a real risk)
Discovered the project is actually **9 repos**, not 3:
- `ecotourism` (main app, since **renamed to `india-data-platform`** on GitHub to match current scope)
- `india-data-core` (shared core API)
- 7 engine repos: `india-species-engine`, `india-forest-engine`, `india-water-engine`, `india-culture-engine`, `india-geo-engine`, `india-extinct-engine`, `india-laws-engine`

All 9 were local-only or partially local-only on the developer's Mac. All are now pushed to GitHub under `github.com/vishnuvarthan18/`. (A GitHub Organization move was discussed and explicitly deferred — not done, low priority, purely cosmetic.)

### 3. Deployed via Claude Code running directly on the VPS
Installed Claude Code on the VPS itself (`ubuntu@vps-e8d92c83.vps.ovh.us`) and had it do the actual deploy work, with explicit safety gates (cloned to scratch dirs for diff-review before touching any live directory, paused on ambiguity like the `pa-engine` naming mismatch).

**Results (also logged as D-66–D-71 in the repo's own `DECISIONS.md`):**
| engine | script | result |
|---|---|---|
| forests_land | wetlands, desertification | 112+31 entities, all cross-checks pass |
| mountains_geography | Wikidata passes/ranges | 1,329 entities |
| mountains_geography | Overpass grid walk | 615 entities (612 peaks, 3 passes) / 829 facts from 14/64 grid cells before hitting an `overpass-api.de` outage; idempotent, will finish automatically via the existing Saturday timer (next run 2026-09-12 04:23 UTC) |
| tribal_culture | Census ST (30 states) | 615 entities, 30/30 cross-checks pass |
| tribal_culture | FRA J&K | 20 entities, 2 known cross-check failures (real source-data bug, already documented, loaded as-is) |
| protected_areas | Elephant Reserves | 33 entities via full pa-harvest pipeline |

Real bugs found and fixed properly during the run (not papered over):
- `culture-engine` was missing `harvest-secrets.env` in its compose file (new FRA script needs `DATA_GOV_IN_API_KEY`)
- Census harvester failed TLS against `censusindia.gov.in` in a fresh container — fixed by installing the missing intermediate CA cert, not by disabling verification
- 3 new scripts had no systemd units at all — installed and enabled 6 new timers total

Verification pass: queried the heartbeat table directly (not just `/v1/ops/alerts`) — 19 jobs, all healthy except the one expected partial. Zero stale sources, zero empty-successful runs, confirmed genuine.

## New open item found this session
**Heartbeat monitoring has a real gap**: `/v1/ops/alerts` flags jobs as stale using a flat 36-hour `silence_after` regardless of actual job cadence — so weekly/monthly/quarterly jobs will always look "stale" between real runs. Not fixed this session (correctly deferred), logged in `DECISIONS.md` as D-70 for prioritization.

## Current full open-items list (carried forward + new)

**Needs the user directly:**
1. Register 5 pending API keys — WDPA, IUCN v4, GeoNames, OpenTopography, GFW (blocks the most future work — species, protected areas, forests engines)
2. Rotate `DATA_GOV_IN_API_KEY`
3. OSM vs WDPA geometry-provider decision (currently parked on OSM)
4. India Code / Indian Kanoon build-or-not call for `laws-engine` (still fully unstarted, 0 entities)
5. Capacity call on multi-GB bulk datasets + retest India-WRIS/MoTA from an Indian egress point (VPS is in Oregon, USA — may be why those sources look unreachable)

**Lower priority / housekeeping:**
6. `india-extinct-engine` — deliberately undeployed, real repo exists on GitHub, never cloned to VPS. Eventually needs a build-or-drop decision.
7. Heartbeat `silence_after` fix (D-70, above)
8. Saturday's automatic Overpass timer run (Sept 12) will finish the remaining 50/64 grid cells on its own — no action needed

## Infra reference (for next session)
- VPS: `vps-e8d92c83.vps.ovh.us`, IPv4 `40.160.137.239`, Ubuntu 26.04, OVH US subsidiary, VPS-1 2027 plan (~$6.31/mo)
- SSH: `ssh ubuntu@40.160.137.239` (NOT root — root password login is disabled by design)
- Claude Code is installed on both the developer's Mac and the VPS itself; running it directly on the VPS (via SSH) is the working pattern for future deploy sessions — avoids all the sandbox network restrictions that caused this session's early confusion
- All 9 repos live under `github.com/vishnuvarthan18/`: `india-data-platform` (renamed from `ecotourism`), `india-data-core`, `india-species-engine`, `india-forest-engine`, `india-water-engine`, `india-culture-engine`, `india-geo-engine`, `india-extinct-engine`, `india-laws-engine`
