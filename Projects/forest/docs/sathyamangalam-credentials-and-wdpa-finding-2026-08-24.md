# Credentials pass + the WDPA finding — 24 Aug 2026

Browser session run against Protected Planet, NASA FIRMS, eBird, IUCN, BHL, data.gov.in and Gmail. Five credentials resolved. One finding overturns the standing plan.

> **Secrets are deliberately not stored in this doc.** The five keys were delivered in session and belong in `harvest-engine/.dev.vars` and `wrangler secret put`, not in project knowledge.

## 🔴 `WDPA_API_KEY` will NOT deliver the Sathyamangalam polygon. Drop it from the critical path.

The handoff docs call the WDPA key "the single highest-leverage action" because it unblocks `reserve.boundary_geojson`. Checked directly on protectedplanet.net — it does not.

**India restricts its protected-area data.** The country page states it verbatim:

> "India chooses to restrict some data on its protected areas and/or OECMs. As a result, data on 900 protected area(s) are not publicly available and cannot be viewed or downloaded on this page."

India's public record is **90 protected areas**, dominated by international designations — Ramsar, UNESCO-MAB, World Heritage. Free-text search returns **no results** for `Sathyamangalam`, for the alternate spelling `Satyamangalam`, and — as a control — for `Mudumalai`. A famous, long-notified tiger reserve returning nothing is not a spelling problem; it confirms Indian national PAs sit in the restricted 900.

The restriction is at the data-publication level, not the access-tier level. A key authenticates you to the same public subset. **Getting the key does not change what is served.**

Confidence: high. Not independently verified: whether UNEP-WCMC grants restricted-data access under a separate research agreement — a correspondence process, not a registration form.

### Consequences

- `reserve.boundary_geojson` stays NULL longer than planned. Every spatial figure remains bbox-scoped over ~2,585 km² including farmland, Bhavanisagar reservoir and three towns.
- "Recompute NDVI and forest loss reserve-scoped" is blocked until a polygon arrives from elsewhere.

### Where the polygon should come from instead — unresolved

1. **OpenStreetMap via Overpass.** Untested. A mid-session claim that Nominatim proved an enclosing reserve polygon was **wrong and is retracted** — "Sathyamangalam Tiger Reserve" sits in a temple node's `addr:housenumber` tag, a data-entry quirk. The node's real hierarchy contains only administrative boundaries. OSM coverage is **unconfirmed**. Overpass is robots-disallowed to browser tooling and must be queried by the harvest engine — which reopens the long-standing, still-undecided **Overpass robots.txt exemption**.
2. **The 2013 TR notification and GOMS No.122 (2008)** — already inside the Stage 8 corpus.
3. **Archived WDPA releases** — pre-restriction editions carried Indian PAs. Adds provenance complexity.
4. **Bhuvan (ISRO)** — already a named source, delivering zero.

## Credentials — all five resolved

| Secret | State |
|---|---|
| `FIRMS_MAP_KEY` | ✅ Was **already issued 21 Aug** and sitting unread in Gmail. Re-request returned "email is already registered," which is how it was recovered. 5000 transactions / 10 min |
| `EBIRD_API_KEY` | ✅ Verified live — HTTP 200, real records inside the reserve (Indian Peafowl, Red Spurfowl nr Aracode) |
| `IUCN_API_KEY` | ✅ Verified live — v4 `taxa/scientific_name` returned `Panthera tigris`, sis_id 15955. **Note: the IUCN stream was never written.** Build item, not a credential item |
| `BHL_API_KEY` | ✅ Issued |
| `DATA_GOV_IN_API_KEY` | ✅ Issued via the **"Generate API Key"** button on the resource page (not My Account). Verified: cap lifted |
| `GEE_SERVICE_ACCOUNT_JSON` | Already a Cloudflare **production** secret. Write-only — cannot be read back. Regenerate a fresh key into `.dev.vars` for local runs |
| `WDPA_API_KEY` | ⛔ Deprioritised — see above |

## LGD — resolved, with a trap

`sources/lgd.json`'s `REPLACE_WITH_LGD_VILLAGES_RESOURCE_ID` is:

```
c967fe8f-69c4-42df-8afc-8a2c98057437
```

Endpoint:
```
https://api.data.gov.in/resource/c967fe8f-69c4-42df-8afc-8a2c98057437
  ?api-key=(secret, removed)
  &filters[stateNameEnglish]=Tamil Nadu
```

Dataset: Ministry of Panchayati Raj, updated 23 Aug 2026. **18,893 Tamil Nadu villages.**

### ⚠️ The silent-truncation trap

data.gov.in publishes a **sample key** on every resource page that caps at **10 records** regardless of the `limit` requested — and returns HTTP 200 with `status: "ok"`. Measured: requested `limit=100`, server forced `limit=10`, returned 10 of 18,893.

A stream using it would log a successful `job_run`, write 10 rows, and mark LGD as delivering at **0.05% coverage**. `lgd.js` must assert `count == total` or paginate to exhaustion, and must never silently accept a server-reduced `limit`.

### Schema worth knowing

Rows carry `villageNameLocal` and `districtNameLocal` in **Tamil script**, plus `villageCensus2011Code` — a direct join key to Census 2011. Both matter for the gazetteer and for the Stage 3 Census source that currently delivers zero. Tamil-script fields are only partially populated.

## Other live items

- **NASA retires Suomi NPP data products on 1 Nov 2026.** The FIRMS stream must move to NOAA-21 / NOAA-20 before then.
- **BHL API v2 retires 31 Dec 2026.** Use v3 (`/api3`). If `bhl.js` targets v2, update now.
- **BHL is not fully credential-gated.** OAI-PMH (`/oai`) and the TSV bulk exports (`title.txt`, `item.txt`, `part.txt`) need no key at all. Stage 2 BHL work could have started without waiting.
- **GitHub secret-scanning alert on `vishnuvarthan18/w2d-admin`**, opened 19 Aug, still unread: "Possible valid secrets detected." Different repo, but real and unattended.
