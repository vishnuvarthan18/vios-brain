# Atlas data bundle integration — 14 August 2026

## What happened
Vishnu uploaded the `atlas.db` data bundle (built 14 Aug 2026 from the harvest pipeline) plus the existing `sathyamangalam-atlas.html` site file. The site was updated in place to reflect real backend data instead of placeholder/aspirational numbers, and delivered back as a file (not saved to this project — see note at bottom).

## Bundle contents received
- `atlas.db` — SQLite, 26MB, 10 tables: source, place, taxon, occurrence, document, historical_passage, legal_instrument, news_event, observation_layer, claim
- `documents.json` — 928 tier A+B assessed-relevant docs (raw harvest is 7,190; ~87% excluded as noise from over-broad queries)
- `species.json` — 1,839 taxa
- `places.json` / `places.geojson` — 19 places (15 settlements with description text from claims.json, 4 forest-range entries without point coords)
- `claims.json` — 82 sourced claims + 4 unresolved conflicts (policy: no silent winner-picking, both values shown with sources)
- `coverage.json` — per-domain completeness, overall 30.3%
- `legal.json`, `news.json`, `passages.json` — all empty (0% domains)
- `manifest.json` — build metadata, matches README

## Key honest numbers (safe to quote)
- Occurrences: 74,180 (single citable GBIF download, 13 Aug 2026, doi.org/10.15468/dl.w56s6d)
- Taxa: 1,839 (92% of 2,000 target)
- Documents: **928** (not 7,190 — that's raw noise)
- Places: 19 (only **1.6%** of 1,200 target — the single largest gap)
- Overall coverage: **30.3%** — dropped from a previously reported 36% not from regression but from tightening the documents target/methodology; the lower number is the truthful one
- Automated harvest ceiling: 62–68%. Remainder needs management-plan extraction, physical archives, Forest Dept relationship, original photography.

## The 4 disputed figures (both values published, no winner picked)
| Field | Value A | Value B |
|---|---|---|
| Core area | 793.49 km² (TNFD/STR portal, 2013) | 917.27 km² (Wikipedia) |
| Elephants | 350–450 (Mgmt Plan 2010) | 651 (TNFD, 2024–25) |
| Leopards | 111 (TNFD, 2024–25) | ~20 (Mgmt Plan 2010) |
| Tigers | 112 (TNFD, 2024–25) | 8–10 (Mgmt Plan 2010, scat DNA) |

Two decisions still needed from Vishnu (unchanged from README): (1) authoritative source for the four disputed figures, (2) written confirmation of the robots.txt exemption for 5 API hosts (Overpass, iNaturalist, eBird, Shodhganga, Google News).

## Site changes made to sathyamangalam-atlas.html
1. **Places section** — added callout: 19/1,200 places, 1.6% complete, largest gap in the atlas.
2. **Record section** — corrected bibliography note: 928 assessed-relevant documents (61.9% of 1,500 target), explicit statement that the 7,190 raw harvest is excluded as mostly noise.
3. **Data section** — added a full per-domain coverage table (occurrences 100%, taxa 92%, documents 61.9%, places 1.6%, historical/legal/layers/news/media/community all 0%), the overall 30.3% figure with the 36%→30.3% correction explained, and a table of the four disputed figures with sources side by side.
4. **Species section** — updated note to state the 1,839-taxon backend size, cite the GBIF DOI, and disclose the 0.05° sensitive-species coordinate rounding (385 records verified coarsened).
5. Left the hero stats band (1,408 km², 112 tigers, 651 elephants, etc.) untouched — those are reserve facts, not atlas-completeness metrics; mixing them with coverage numbers would have confused the message.

## Still not done / open follow-ups
- Species list shown on-site is still a curated ~74-entry subset of the 1,839-taxon backend, not a full sync — full species.json → site pipeline not built.
- Bibliography shown on-site (~40 curated sources) not synced to the full 928-document set — still hand-curated.
- Legal, news, historical-passage, media, community, and environmental-layer domains are all empty in the backend (0%) and have no site sections reflecting them yet, beyond the coverage table disclosure.
- Domain registration for sathyamangalam.org is still the biggest open blocker per the site dossier (unchanged from prior session).

*(Note: this doc's numbers reflect the state before the git pipeline was built — see atlas-git-pipeline-2026-08-14.md for the corrected 89-place / 31.5%-coverage figures.)*
