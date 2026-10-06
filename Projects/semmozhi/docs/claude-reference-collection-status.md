# Reference collection status (2026-09-25)

## Folder
Single project root: Downloads/tamil_harvest (Mac). Created design/ with references/{palm_leaf,stone,copper_plate,pottery,coins,rings,seals}, svg_kit/, specs/, README.md, references/LICENSES.csv (header only). design/references/ added to .gitignore. Nothing else moved or deleted.

## What works / does not (tested)
- Cloud shell and Mac shell: no downloads (proxy 403 on nearly all hosts, even example.com). Not to be bypassed.
- WebFetch: Wikimedia Commons AND en.wikipedia File: pages are cache-only (no license/direct-URL lookup). en.wikipedia article text works.
- WebFetch on museum open-access APIs WORKS: Cleveland Museum of Art (openaccess-api.clevelandart.org, CC0, direct image URLs like openaccess-cdn.clevelandart.org/<accession>/<accession>_web.jpg), Art Institute of Chicago (api.artic.edu, is_public_domain + image_id, IIIF), Met (collectionapi.metmuseum.org). Output is summarised by a small model, so every URL must be re-verified by the download script (HTTP 200 + image content-type).
- User said: do NOT use the Mac's Chrome browser.

## Yield so far
- Cleveland "coin india": ~30 CC0 coin records, but all North Indian (Kushan, Gupta, Rajasthan punch-marked, Indo-Greek), not Tamil.
- Cleveland "chola": 16 CC0 Chola sculptures (not in the 7 surfaces).
- AIC "chola": 5 public-domain Chola/Tamil Nadu sculptures.
- Met: query "Pandya" = 5 object IDs (38133, 38134, 316105, 451314, 316106), "coin" + India = 29 IDs; contents not yet checked.
- No Tamil pottery sherds, Tamil-Brahmi stones, copper plates, rings or seals found in open-access museum APIs yet.

## Decision pending
Real Tamil coins/pottery/inscription photos live mainly on Wikimedia Commons (blocked to assistant). Options: user exports Commons file lists / saves files; assistant builds LICENSES.csv from museum APIs (CC0) for context items, and a download script the user runs on the Mac.
