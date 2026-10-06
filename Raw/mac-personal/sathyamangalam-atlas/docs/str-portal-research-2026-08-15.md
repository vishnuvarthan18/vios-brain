# Full sathytiger.tn.gov.in portal research — 15 August 2026

## What this is

At Vishnu's request, every substantive page of the official Tamil Nadu
Forest Department STR portal (`sathytiger.tn.gov.in`) was fetched and its
content extracted, ahead of building the home-page range-hover-map feature.
This is real, current, official-source data — not an AI research pass on
secondary sources like `heritage-research-findings-2026-08-14.md` was.
Still, per this site's own standard, nothing here should be treated as
final until cross-checked against what's already in `data/atlas.db` —
which turned up one real, important conflict, corrected immediately
rather than held for later (see below).

## Correction to the previous session's work — the G.O. number

The previous session (14–15 Aug) OCR'd the primary 2013 tiger-reserve
notification PDF from `ntca.gov.in` and read the G.O. number as
**"G.O.(Ms.) No.4"** — verified twice, at two OCR resolutions (300dpi and
600dpi), both clean unambiguous reads. That number was then shipped as
settled fact across five site locations (`land.html`, `govern.html` ×2,
`history.html` ×2, `record.html`) and treated as resolving part of the
core-area discrepancy.

The Forest Department's own current portal, `sathytiger.tn.gov.in/about`,
states instead: *"Entire Sathyamangalam Sanctuary (1,40,840 ha) declared
as Tiger Reserve under Section 38(V) of Wildlife Protection Act, 1972
(G.O. Ms. No. 45)."* Fetched and confirmed with an exact-quote follow-up
request, not a paraphrase — this is a real conflict between two named
official sources, not a fetch/OCR artifact on either side.

**All five site locations have been corrected** (this session) to say
"the G.O." or similar rather than assert a specific number as settled,
and to state the discrepancy plainly — "No. 4" per the scanned primary
document vs. "No. 45" per the Department's own portal — flagged as not
yet resolved, matching the treatment already given to the core-area and
NH-948-date conflicts. The core-area figures themselves (793.49 /
614.91 / 1,408.40 km²) are unaffected — both the PDF and the portal agree
on those.

**Not yet done**: determining which G.O. number is actually correct.
Possible explanations, none confirmed: a transcription error on one side;
"No. 45" being a different, related order (e.g. renumbered on
re-publication); the scanned PDF being a draft or an earlier reference
number. Whoever resolves this should check the Tamil Nadu Government
Gazette's own official index of March 2013 G.O.s directly, not rely on
either of these two secondary renderings.

## Full portal data pulled (for reference / future use)

### `/about` — reserve-level facts
- Coordinates: N 11°26'46.21" to N 11°48'57.42"; E 76°49'55.66" to
  E 77°17'18.39" (more precise than the bounding box currently on
  `land.html`, which uses 11°29'15"–11°48'41"N — worth reconciling, not
  done this session)
- Core zone stated as 56.4% of total, buffer 43.6% — consistent with the
  793.49/614.91/1,408.40 figures already on the site
- **2011 enlargement figure**: "887.26 sq. km added via G.O. Ms. No. 93,
  11 August 2011" — the site's `land.html` currently just says "the area
  roughly tripled with the September 2011 enlargement" without a figure
  or exact date; this portal gives August (not September) and a precise
  hectare figure. **Discrepancy, not yet reconciled** — month conflict
  (Aug vs Sept) needs checking before use.
- Forest Department formation: "1856... under Dr. Cleghorn" — not
  currently on the site; Cleghorn is a real, checkable historical figure
  (Hugh Cleghorn, pioneering Madras Presidency forestry) worth verifying
  before use, not previously in this project's research.
- Leadership names (K Rajkumar IFS, Field Director; S. Gowtham IFS,
  Deputy Director Sathyamangalam; Yogesh Garg IFS, Deputy Director
  Hasanur) — current personnel, not stable facts suitable for permanent
  citation; if used, should be dated and expected to go stale.
- Hyena count: "25 stable" — not currently on the site's species/wildlife
  figures anywhere; new.

### `/organizational` — staff structure
Two-division hierarchy (Sathyamangalam: 6 Range Forest Officer posts;
Hasanur: 4) under a Field Director reporting to the PCCF (Wildlife) /
Chief Wildlife Warden in Chennai. "100–150 tribal community members" as
local guardians — broadly consistent with `people.html`'s framing of
tribal youth staffing anti-poaching camps, though the specific number
differs from the "100+ local watchers" already stated on `govern.html`
(the portal's `/anti_poaching` page gives a much more precise 219 total,
see below — worth using that over this vaguer 100–150 figure if adding
anything).

### `/anti_poaching` — camp and staffing detail
Significantly more precise than what's currently on the site
(`govern.html` says "54 anti-poaching camps, 100+ local watchers"):
- **54 camps**: Sathyamangalam 25, Hasanur 29
- **219 sanctioned APW (Anti-Poaching Watcher) personnel**: Sathyamangalam
  120, Hasanur 99
- M-STrIPES patrol tracking system in use; bi-annual sign surveys (pug
  marks, dung, carcasses, bark peelings, vegetation)
- "Sustainable income for 60+ families" — a new, more specific framing
  than the site's current generic "local youth employment" language
- **Usable as a direct upgrade** to the existing 54-camp/100+-watcher
  figure already on `govern.html`, if Vishnu wants it added — not done
  this session, flagged for a future pass.

### `/conservation` — management detail
- Invasive clearance rates: **Lantana ~100 ha/year, Prosopis
  ~200 ha/year** — the site currently has TN PIPER Lantana clearance
  figures (50→75 ha/month) on `life.html` for a different, more recent
  program; these portal figures look like a separate/older baseline
  figure, not obviously the same metric — needs reconciling before
  either replaces or supplements the other, not done this session.
- Buffer zone specifically named as spanning **five ranges**: Bhavanisagar,
  Sathyamangalam, T.N. Palayam, Germalam, Talavadi. **This is new,
  useful, real data not previously in this project** — it's the first
  source found naming which ranges make up the buffer zone specifically.
- Visitor carrying capacity: "~152 visits/day" — new, not currently
  stated anywhere on `visit.html`.
- Sathyamangalam Tiger Conservation Foundation: registered trust,
  established July 2015 — corroborated independently on the `/foundation`
  page (see below).

### `/foundation` — Sathyamangalam Tiger Conservation Foundation
- Established 15 July 2015, statutory foundation under WPA Section 38-X
- Employment claim: "more than 1,000 tribal families and 2,000 non-tribal
  families" — a much larger figure than anything currently cited on
  `people.html`; **flag before using** — this is the kind of large round
  employment-impact number that self-published foundation material
  sometimes overstates; would benefit from independent corroboration
  before being treated as citable fact, per this site's own standard.

### `/exg` — ex-gratia compensation (human-wildlife conflict)
Genuinely valuable, specific, dated data not found anywhere else in this
project's research so far:
- **₹3.99 crore distributed across 1,046 beneficiary families, 2022–2025**
  (31 deaths, 887 crop-damage cases, plus injury/livestock/property)
- Year-by-year breakdown table with deaths/injuries/crop/livestock/
  property counts and total ₹ amount for 2022-23 through 2025-26
  (partial year)
- **This is a strong candidate for `govern.html`'s "live pressures,
  quantified" section**, which currently mentions human-elephant conflict
  only qualitatively ("crop raiding and electrocution... 239 km of solar
  fencing") with no compensation/cost figures — not added this session,
  flagged for a future pass.

### `/our_activities` — tribal welfare and community programs
- **Tribal welfare**: "5 major settlements, 402 families, 1,807 total
  population, 28 Tribal Village Forest Committees" — more specific than
  `people.html`'s current "9 settlements" framing (note: 9 vs. 5 — these
  may be measuring different things, e.g. all tribal settlements vs.
  "major" ones, or core vs. core+buffer; **needs reconciling, not
  assumed to conflict**, before use)
- **43 Village Forest Committees, 25 JFM Committees** — new
- Aggregate impact metrics given without methodology: "85% reduction in
  forest incidents, 60% improvement in forest cover quality, 45% income
  growth" — **flag as unsourced-methodology round numbers, the kind of
  self-reported aggregate stat this site's own standard treats
  cautiously** (compare to how `land.html` already treats the "1,411 km²"
  figure from earlier research — a plausible-sounding number without a
  clear derivation is not the same as a verified one)

### Coming-soon / empty pages (nothing to cite)
`/safari`, `/accommodation`, and `/coracle_ride` are all literally
placeholder "Coming Soon" pages on the department's own live site as of
this fetch — worth noting because `visit.html` currently describes safari
booking as going "through the department's own STR portal," which is
accurate as a pointer but the portal itself has no content there yet.

### `/eco_park` — facilities list
Wildlife Interpretation Centre, Vulture Museum, fish aquarium, herbal/
butterfly gardens, watch towers, Bannari Tiger Awareness Centre. No
timings or pricing given. Community-run — "revenue... shared with EDCs."
Not currently mentioned anywhere on `visit.html`; a plausible future
addition if Vishnu wants `visit.html` to cover more than safari/permits.

### `/csr`, `/announcements`, `/policy`
No durable factual content — CSR page is an invitation-to-partner form
with no historical partnership data; announcements is a single dated bird
survey notice (14–15 Feb 2026, already in the past relative to this
project's "current date" of 15 Aug 2026 — not usable as a forward-looking
item); policy is a standard privacy policy, not reserve content.

## Next steps

- [x] Correct the G.O. number claim across the site — done this session
- [ ] Decide with Vishnu which of the new figures above (anti-poaching
      precision, ex-gratia compensation table, buffer-zone range names,
      eco park facilities) should actually ship to the site — none have
      been added yet beyond the G.O. correction; this file is the
      research/staging layer, matching how `heritage-research-findings-
      2026-08-14.md` was used before its own "Shipped" section
- [ ] Reconcile the 2011 enlargement month (Aug per portal vs. Sept per
      existing `land.html` copy) and hectare figure (887.26 sq. km, not
      currently stated)
- [ ] Reconcile "5 major settlements / 402 families" (portal) against
      "9 tribal settlements" (people.html) — check whether these are the
      same or different counts before treating as consistent or
      conflicting
- [ ] Verify the Dr. Cleghorn / 1856 Forest Department origin claim
      before use — plausible and checkable (Hugh Cleghorn is a real,
      well-documented figure in Madras Presidency forestry history) but
      not yet independently confirmed
- [x] Full per-range data for all 10 ranges (area, attractions, key
      features) found and shipped — see below.

## Shipped — 15 August 2026: the ten-range hover map (home page)

Vishnu's screenshot of an interactive hover-map turned out to be a real,
live feature already on `sathytiger.tn.gov.in/about` — not a mockup. Its
underlying JS data object (`rangeData`, embedded in the page's own
`<script>`) was located and fetched directly, giving genuine area,
attractions and key-features text for all 10 ranges — a first for this
project; previously only 4 ranges (the 2010 baseline) had any area figure
at all. The department's own SVG (`assets/img/STR.svg`, an Esri
ArcMap-exported GIS file, 79 paths) was also fetched but **not used
directly** — per Vishnu's decision, the site's own schematic/illustrative
shapes were drawn instead, in the same rough relative layout, rather than
copying the department's actual boundary geometry or JS code verbatim.

Added to `site/index.html` as a new home-page section (`.inst-rangemap`,
between the Gazetteer stripe and the Elephant Corridors teaser).

**Updated same day, second pass**: Vishnu asked for "the exact super good
map" — clarified as wanting the real GIS boundary geometry, not the
schematic placeholder shapes from the first pass. Decision this time:
use the actual department file directly (`media/str-range-map.svg`, a
straight copy of `sathytiger.tn.gov.in/assets/img/STR.svg`, 840KB, Esri
ArcMap export, 79 paths) rather than redrawing again — a reversal of the
first pass's "redraw, don't copy" call, made deliberately this time since
Vishnu explicitly asked for the accurate original.
The real SVG has no per-range element IDs, same as the department's own
site, so hover detection uses the identical technique their own script
uses: on hover, find the `<text>` label nearest the cursor position (read
live via `getBoundingClientRect()`, not hand-transcribed from the SVG's
transform matrices) and look up that range's data. Reimplemented as
original code for this site, not copy-pasted from their script.

**Known, disclosed discrepancy**: the 10 range areas as given by the
portal's own data sum to **145,530.92 ha**, about **4,690 ha (3.3%) more**
than the reserve's own official total of 140,840.541 ha (confirmed from
the primary 2013 G.O. last session). Per Vishnu's decision, shipped as-is
with an on-page caption disclosing the gap rather than silently adjusting
any figure — consistent with this site's standing policy of showing
conflicts rather than picking a winner. Not investigated further this
session; possible explanations (unconfirmed): range areas may include
some boundary overlap, or may be measured on a different basis than the
core+buffer total (e.g. gross range extent vs. reserve-notified area).
