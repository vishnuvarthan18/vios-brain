# Atlas Harvester — Status Report (13 Aug 2026, updated)

Live read of `data/atlas.db`. Supersedes earlier same-day version — two items resolved.

## Complete
- Schema, SQLite store, provenance enforcement
- GBIF occurrence harvest (verified)
- Sensitive-taxon location coarsening (verified)
- OpenAlex literature harvest — 5,514 docs
- Crossref literature harvest — 1,268 docs
- **Relevance scoring — now applied with `--write`, tiers persisted**
- 5 critical bugs found and fixed (documented)
- DB rebuilt after corruption (verified)
- Test suite: 86 passing

## Live DB counts
| table | rows |
|---|---|
| occurrence | 74,180 |
| taxon | 1,839 |
| document | 6,914 |
| source | 47 |
| place | 19 |
| claim | 82 |
| historical_passage / legal_instrument / news_event / observation_layer | 0 |

Documents: OpenAlex 5,514, Crossref 1,268, Semantic Scholar 93, other 39.
Quality: DOI 92%, abstract 74%, full text 42%.

## Relevance tiers — now stored (`document.relevance_tier`)
| tier | count | % | meaning |
|---|---|---|---|
| A | 401 | 5.8 | names reserve or a place inside it |
| B | 396 | 5.7 | regional context, needs review |
| C | 1,167 | 16.9 | ambiguous term only |
| D | 4,950 | 71.6 | no landscape term |

Plausibly relevant (A+B): **797 (11.5%)**. Top matched terms are "tamil nadu" (537) and "western ghats" (326) — both regional, not local. Only 139 docs name Sathyamangalam directly. Full detail: `out/relevance_report.csv`.

**Quote 797, not 6,914, in any external material.**

## Verification — all passing
- Coordinates bounded correctly (11.48–11.82N, 76.83–77.46E)
- Sensitive taxa coarsened to 0.05 (elephant 17, tiger 2, leopard 4); non-sensitive left precise (chital 14, peafowl 1,368)
- All 74,180 occurrence rows single-sourced (GBIF DOI)

## Phase status
1-foundation not run · 2-gazetteer blocked (Overpass robots.txt) · 3-biodiversity complete · 4-literature harvested + tiered · 5-government not started (needs ATLAS_CONTACT) · 6-history code fixed, needs 6 manual downloads · 7-media blocked (Google News robots.txt)

## Resolved since previous report
- **Relevance tiers stored** (see table above).
- **DB file-handle scare was a false alarm.** `.fuse_hidden*` files are a FUSE-layer artifact of SQLite unlinking `-wal`/`-journal` under an open handle — not proof of a stray process. Confirmed clean by acquiring an exclusive write lock on `data/atlas.db`; no other writer exists. **Standing method note: use a lock test, not file timestamps, to answer "is something else holding this DB open."**

## Still open
- Semantic Scholar citation expansion died mid-run (429 backoff, 93 records only). Re-run.
- `out/*.json` stale (written 15:50, pre-literature-harvest, pre-tiering). Re-export.
- `data/atlas.db.corrupt` (26MB) still present — safe to delete once rebuild confidence is confirmed.
- Coverage weighting in §below still charges documents at the raw 6,914 (capped 100%), not the real 797 — worth fixing the coverage formula itself, not just flagging it in prose each time.

## Coverage (recomputed, stale JSON ignored)
Weighted overall ≈ **36%**, up from 15.2% at 15:50. Flatters the work — see caveats.

| domain | have | target | % | weight |
|---|---|---|---|---|
| places | 19 | 1,200 | 1.6 | 12 |
| taxa | 1,839 | 2,000 | 92.0 | 16 |
| occurrences | 74,180 | 30,000 | 100 (capped) | 8 |
| documents | 6,914 | 400 | 100 (capped) | 12 |
| historical | 0 | 600 | 0 | 10 |
| legal | 0 | 120 | 0 | 8 |
| layers | 0 | 25 | 0 | 10 |
| news | 0 | 800 | 0 | 6 |
| media | 195 | 1,500 | 13.0 | 8 |
| community | 0 | 100 | 0 | 10 |

Caveats: document coverage figure is inflated (real tier-A+B count 797, not 6,914). Occurrences all single-sourced, no corroboration.

## Unresolved factual conflicts (in `claims` table)
| field | values seen |
|---|---|
| core area km² | 793.49 / 917.27 |
| elephants | 350–450 / 651 |
| leopards | 111 / ~20 |
| tigers | 112 / 8–10 |

Needs a source-authority ruling and a recorded reason — not more harvesting.

## Two decisions that are not technical (block more value than any remaining code work)
1. **Overpass robots.txt exemption** — 1,200 places at weight 12, single largest coverage gap. robots.txt governs crawlers, not authenticated API clients; proposed fix is a documented per-host exemption list in `config.py`. Escalate first.
2. **Authoritative source ruling** for the four conflicting figures above.

## Next actions (priority order — corrected from earlier draft, which still listed resolved items)
1. Robots.txt decision with management (Overpass unblock)
2. Re-run Semantic Scholar `citations` harvester (429 backoff killed it mid-run)
3. Re-export: `python3 -m atlas export` (stale JSON predates tiering)
4. Confirm ATLAS_CONTACT resolves to real org page, start 5-government
5. Manually download 6 colonial `_djvu.txt` files into `data/archive/`
6. Obtain free EBIRD_API_KEY and IUCN_API_KEY
7. Delete `data/atlas.db.corrupt` once rebuild confidence confirmed
8. Fix coverage formula to use tier-A+B document count instead of raw count

## Standing rule
5 defects on 13 Aug reported success in logs while producing wrong data (CSS as history, papayas as pythons, 6,000 irrelevant papers, arctic wildlife under valid DOI, two processes destroying the DB — the last of which turned out to be a false alarm from mistaking FUSE artifacts for evidence). Check the rows, and check with a real test (lock, not timestamp) — the summary line is not evidence.
