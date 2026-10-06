# Deep analysis — AI #1's Round 3 (closing the flagged unknowns) + consolidated master doc

**Source:** `Sathyamangalam_Atlas_Round3.md` and `Sathyamangalam_Atlas_Research_Report_2.md`. **Confirmed:** Report_2 is the fully merged master document (Task 1–4 + Round 2 + Round 3, "twelve findings" version) — it contains no material beyond what's in the standalone Round 3 file. **This is still all AI #1.** The second, independently-initiated AI's report has still not been supplied.

**Status of Round 3:** self-reported complete. Of 12 flagged items from Rounds 1–2: 7 resolved, 2 partially resolved, 3 closed as tested negatives or confirmed environment blocks. Plus 5 outreach documents drafted and ready to send.

---

## The single most useful output of Round 3: five ready-to-send documents

Round 3 stopped researching and started producing action items. These are drafted, addressed, and ready to go:

1. **Email to the French Institute of Pondicherry** (`ramesh.br@ifpindia.org`) — asking whether the Historical Atlas of South India's place data is reusable, whether the Western Ghats Portal arrangement still stands, and whether IFP would collaborate. Ask for Muthusankar Gowrappan by name.
2. **RTI (not email) to the Tamil Nadu Biodiversity Board** — ₹10 statutory request for BMC/PBR status of panchayats bordering Sathyamangalam. Deliberately asks about *access terms* (items 5–6) before asking for the registers themselves (avoids a blanket refusal on traditional-knowledge grounds), and puts the benefit-sharing question (item 7) on record given the documented Maduranthakam case of uncompensated knowledge extraction.
3. **Email to GBIF** (`hostedportals@gbif.org`) — the four questions that would resolve India's genuinely contradictory Observer/Participant status.
4. **Three short enquiries** — Keystone Foundation (platform confirmation), FES (bibliography access), Tamil Nadu Forest Department (RTI for the current notified area with G.O. number — flagged as **the cheapest possible win in the whole three-round report**: ₹10 for an authoritative area figure to anchor the disputed-figures feature against).

**Recommendation: send these five now, independent of anything else.** They cost nothing, several have statutory response deadlines (RTI = 30 days), and they don't require any build decision first.

## What Round 3 actually resolved (vs. what it left open)

**Resolved, changes the plan:**
- **GeoNature is now the default stack recommendation, not a candidate.** Seven releases in ten months (v2.17.4, 8 July 2026), live documentation, a November 2026 national user meeting, a commercial support market. This is the strongest maintenance signal found for any stack across all three rounds.
- **The WII National Wildlife Database Cell exists and holds two of the atlas's five domains already** — protected-area gazette notifications and a national wildlife bibliography — with a named contact (Dr J. S. Kathayat). This is a potential data-partner, not just a source to scrape.
- **TIGERNET's data is accessible through a back door**: NTCA publishes the same mortality data openly (1,581 events, 2012–2025) even though TIGERNET itself blocks automated access. NTCA also publishes its own inclusion rule verbatim ("no tiger death is entered unless an authentic source from the State Government reports it") — a real-world model for the atlas's own disputed-figures inclusion rules.
- **Keystone Archives confirmed running Omeka S**, ~1,055 items, with a Kumu.io "browse by people" network view worth copying as a UI idea.
- **India Observatory/IBIS is active, not dormant** (running a "Code4Nature Challenge 2026" despite a 2019 copyright footer) — reverses a Round 1 call.
- **Two more disputed-figures cases found, both bigger in scope than anything before them:** WII's own site shows 1,014 vs. 682 protected areas (49% discrepancy) and 175,169 vs. 164,074 km² — a national-scale contradiction inside one government institute's own website. Tamil Nadu's tank count is disputed threefold (39,000 peer-reviewed vs. 119,797 commercial, likely counting different things, neither government-sourced). **The demonstration set for the disputed-figures feature is now seven cases** — five originally, plus these two — two of them national in scope, so the pitch is no longer "just this one reserve."

**Explicitly unresolved, and correctly reported as such rather than glossed over:**
- **India's GBIF Hosted Portal eligibility cannot be determined from published sources at all** — three published GBIF statements conflict and nothing reconciles them. Treat as a genuine open question pending the email reply, not something to assume either way.
- Assam Biodiversity Portal (still 403 after three attempts — now correctly downgraded to a "completeness item," since the more important sub-portal, Western Ghats, was already resolved in Round 2).
- Post-2020 Sathyamangalam management plan — not public; the RTI is the only route.
- FLAME's Districts Project — still just aims, no outputs, though now has a contact email.
- Bhuvan's actual layer specifications (resolutions, scales, coverage years) — confirmed to exist but not extractable by automated means; flagged as a 30-minute manual browser task, not a research gap.
- **The Wayback Machine is confirmed permanently unreachable in this AI's environment**, across three rounds and two distinct refusal mechanisms. Every "dead project" claim across all three rounds rests on DNS/HTTP/TLS/staleness evidence at the moment of checking — none on a dated archived snapshot. The report itself flags this as the single highest-value manual task left (~1 hour in a normal browser) to convert the whole dead-projects list from "dead as of Aug 2026" into an actually-dated mortality record.

## The methodological lesson worth internalizing generally

**A stale copyright footer has now misled this same research process three separate times** — Costa Rica's CRBio, WII's National Wildlife Database, and India Observatory/IBIS were each initially called dormant/dead from a dated footer, and all three turned out to be live or partly live. Worth remembering when evaluating any of the corpus's dead/dormant calls, or when triaging your own future research: a stale-looking copyright date is a prompt to look harder, not a verdict.

## Two negative findings that are actually useful (searched, not assumed)

- **No Indian equivalent of the UK's ecological-desk-study business model exists.** Confirmed by search: Indian EIA consultancies do their own fieldwork and cite IUCN/ZSI/BSI as reference material rather than buying from a records centre, because no records-centre layer exists to buy from. This is Round 1's proposed India-transferable revenue model — it's a real opportunity with no incumbent, but also no proven willingness to pay. Don't assume a market exists just because the UK analogue does.
- **Myanmar, Mongolia, and the Gulf states genuinely have no national biodiversity portal**, tested this round in Burmese, Mongolian, and Arabic respectively (one query each — enough to say "tested and nothing surfaced," not enough to assert confident absence). Their biodiversity record exists only as printed/PDF literature (CBD national reports, national environment volumes), consistent with Round 1's Central Asia finding.

---

## Still pending

This completes the analysis of all of AI #1's output — Round 1 (Tasks 1–4), Round 2 (Angles A–H), and Round 3 (closing the flagged unknowns). **The second, independently-initiated AI's report has not yet been supplied.** Once it arrives, run the full three-way comparison (AI #1's complete corpus vs. AI #2) — agreement, contradiction, and unique findings each side has that the other missed — before touching `reviewer-findings-and-bug-list-2026-08-25.md`.
