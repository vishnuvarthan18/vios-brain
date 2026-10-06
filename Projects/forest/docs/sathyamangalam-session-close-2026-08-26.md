# Session close — 26 Aug 2026 — what we learned, what to change next

Read this first in the next chat. This is the handoff for the "future of the field" research thread and what it means for the Sathyamangalam Atlas and the career plan.

## STANDING NOTE — execution-plan brief deferred (added 26 Aug, later same day)

`sathyamangalam/deep-research-brief-execution-plan-2026-08-26.md` has been drafted but **will NOT be run now — deliberately deferred, not forgotten.** Do not chase this, do not ask "should we run it," do not treat it as an open loop blocking other work. It gets run later, when Vishnu decides to. Everything else in this doc (gazetteer bug fix, sending the 5 outreach documents, the 6 Atlas engineering decisions) proceeds independently of this brief — none of it depends on the execution-plan report coming back.

## What we did this session

1. Cleaned and deeply analyzed AI #1's full three-round deep-research report on worldwide comparable conservation-atlas projects (Round 1: Tasks 1–4, 250+ projects; Round 2: Angles A–H, non-English/registry search; Round 3: closing flagged unknowns + 5 drafted outreach documents).
2. Built and published two HTML artifacts from that research: a searchable "library" of all 602 parsed project records with facets/search (https://claude.ai/code/artifact/a6dc271c-a59b-486f-9e31-d72dac1f2595), and an earlier simpler table version (https://claude.ai/code/artifact/53124df3-54be-403b-967e-338ad8a1c594).
3. Wrote and delivered a "top 10 most similar projects" ranked list.
4. Mirrored all 44 Claude Project docs (plus the PDF playbook) onto the Mac at `~/sathyamangalam-atlas/docs/claude-project/`, organized by the same folders as the Project (`sathyamangalam/`, `claude/`, `senna-regrowth/`, `misc/`). Existing `docs/` files were left untouched.
5. Wrote and ran (via a separate Cowork session) a corrected life-direction research brief — NOT about jobs/salaries, but about whether "technology + forest/conservation" is a real field worth a life's work, as builder and/or researcher, India + global. Report came back and is now saved: `sathyamangalam/deep-research-report-tech-forest-future-2026-08-26.md`.

## THE VERDICT FROM THE LIFE-DIRECTION RESEARCH (the single most important finding)

**Yes, the field is real — high confidence.** GBF Target 21 (UN treaty, 196 countries, 2030 deadline) makes biodiversity data infrastructure a binding obligation, not a nice-to-have. India's ISFR has run biennially for 37 years. NISAR (free open radar data) went live July 2025. GBIF tracks its own growth as a formal global indicator.

**Yes, one person can build a lasting body of work — high confidence.** 16 named solo/near-solo builders verified at 15–24 years each in the research corpus. Solo projects outlive grant-funded ones 3–4x, because a solo project's only failure mode is the person stopping; grant money has a scheduled death built in.

**No — the field will NOT reliably fund an independent life, at least not yet.** This is the finding to sit with. Zero of the 15–24-year solo survivors in the corpus were sustained by conservation-tech grants. All had another livelihood running in parallel. ~$344 of available WILDLABS funding existed per applicant in 2025. Most major international funders (JRS, GBIF BID, CEPF, NSF) explicitly exclude India. Rufford covers India but its grant ladder is designed to terminate.

**The one shape that solves both problems: Krushnamegh Kunte's model.** A scientist inside an institution (NCBS–TIFR) running a peer-reviewed, editorially-governed volunteer platform (Biodiversity Atlas – India, 1,000+ participants). Institutional salary + independent body of work + editorial authority, simultaneously. The brief states explicitly: **the existing career plan (WII/FSI/ICFRE/NRSC contract roles + IGNOU M.Sc. Geoinformatics) already points at exactly this shape, and this research strengthens rather than replaces that plan.**

**The single most credible next move — not a job application:** publish one small, complete, citable thing with a DOI within 12 months. Specifically recommended: a Sathyamangalam place-name gazetteer dataset in Linked Places Format, deposited with a DOI, with a short data paper. Reasoning: bounded, achievable solo, no existing competitor, directly usable by others, converts "person with a plan" into "person with a citable artefact," and is the correct opening move for every downstream path (Rufford application, WII/FSI conversation, an email to the French Institute of Pondicherry that's worth answering).

**Do the gazetteer BEFORE the People's Biodiversity Register (PBR) digitization idea**, even though the PBR gap is huge (272,648 statutory registers nationwide, only ~4.4% verified, none digitised). The PBR work has a real consent/benefit-sharing problem — communities' knowledge was extracted without compensation in at least one documented case (Maduranthakam). That needs a governance decision and standing (ideally via the gazetteer credibility first), not a data pipeline.

### Honest weak points in the research, flagged by the brief itself — do not paper over these
- BISS/TDWG conference abstracts (the field's technical-conference indicator) peaked in 2023 at 159 and fell to 100 by 2025 — a 35% drop, not a growth curve. Cause unknown.
- No credible 10+ year forecast exists from any qualifying government/UN/research body on this field's trajectory. Any "$X billion by 2035" claim should be treated as marketing.
- India-specific government spending on remote sensing for forestry specifically is not published anywhere (Dept. of Space's "Space Applications" line is too broad a proxy).
- Whether the UN's GBF Target 19 finance milestone ($20bn/year by 2025) was actually met is unconfirmed — if badly missed, the finance case weakens.
- No funding instrument publishes its success rate.
- How the long-survivor solo builders actually paid their bills in real life is undocumented in every case but one (direct quote from one).

## Standing decision from this research: the career plan is CONFIRMED, not reopened

Do not re-litigate the six-track/no-government/current-plan-locked history in the next chat. The current locked plan (`claude/current-plan-locked-aug-2026.md`: no permanent government job, IGNOU M.Sc. Geoinformatics distance mode starting Jan 2027, contract/project work with WII/IFGTB/FSI/NRSC/SACON/DGRE staying in play, build a public portfolio Aug–Dec 2026) is now validated by independent research, not just personal judgment. The Kunte-model finding is the argument FOR staying on this exact path, not a reason to change it.

**What actually needs to change/decide in the next chat, given this research:**

1. **Add a concrete near-term deliverable to the Aug–Dec 2026 portfolio plan: the Sathyamangalam gazetteer dataset in Linked Places Format, with a DOI.** This wasn't in the original current-plan-locked doc as a specific artifact — it should be folded in as the flagship first-project output, ahead of or alongside the Senna/GEE mapping notebook. Decide: does the gazetteer or the Senna notebook ship first? The research says the gazetteer is the more strategically valuable one to be first (bounded, no competitor, zero consent issues, cites into everything downstream).

2. **Reconcile this new research against the still-open Sathyamangalam Atlas engineering decisions** (unchanged from before this session, still blocking):
   - Which database is canonical — D1 or atlas.db
   - When/whether to launch given the 7-of-7-categories-at-50%-data gate (People category structurally cannot pass by harvesting alone)
   - Production migration 0005 not yet applied — needs dashboard password
   - NDVI 2026 anomaly / "drier forest" framing — editorial call still pending (original hypothesis wrong 6 of 7 years; 2026 is an unexplained anomaly, agent recommends its own dedicated investigation)
   - `claim` table — still unbuilt, still the project's stated differentiator, still zero rows
   - Project naming — Atlas vs Record — still unresolved
   - Gazetteer under-querying bug — root cause not yet fixed (58 places vs 800–2,000 target; the "under-querying bug, not a data limit" diagnosis from 23 Aug is unresolved). **This now matters even more given the new research says the gazetteer dataset is the single most valuable next artifact** — fixing this bug is now on the critical path for the career-direction plan, not just the Atlas.

3. ~~Reconcile the execution-plan brief against the new life-direction verdict once it comes back.~~ **DEFERRED — see standing note at top of this doc. Do not run this brief yet; do not treat it as blocking.**

4. **Decide: send the 5 already-drafted outreach documents** (IFP/Muthusankar Gowrappan, Tamil Nadu Biodiversity Board RTI, GBIF hosted-portal eligibility email, Keystone Foundation, FES, Tamil Nadu Forest Department RTI for the authoritative reserve area figure). These cost nothing, several have statutory response deadlines (RTI = 30 days), and the new research explicitly recommends sending them now, independent of any build decision. This has been sitting un-actioned for a full research round already.

5. **Decide whether/when to run the PBR consent-and-governance investigation** (Task 3 from the not-yet-run execution-plan brief covers legal/consent groundwork for tribal-knowledge data — this is part of the deferred brief, so also on hold) before touching the 272,648-register opportunity at all.

## Files to re-read first in the next chat, in this order

1. This file (session close)
2. `sathyamangalam/deep-research-report-tech-forest-future-2026-08-26.md` — the verdict, full detail
3. `claude/current-plan-locked-aug-2026.md` — the standing career plan this research validates
4. `sathyamangalam/master-project-log-index-2026-08-25.md` — Atlas engineering status index
5. `sathyamangalam/reviewer-findings-and-bug-list-2026-08-25.md` — the 7 open Atlas bugs
6. `sathyamangalam/deep-research-brief-execution-plan-2026-08-26.md` — DEFERRED, do not run yet (see standing note)

## Do NOT re-ask in the next chat
- Whether to pursue this field at all — decided, yes, with the funding caveat understood.
- Whether to reopen the government-job question — no, locked.
- Whether to keep researching "the past" (comparable projects) — done, sufficient, three full rounds.
- Framing this around jobs/salaries — explicitly rejected earlier in this thread ("no you are looing his like a empolyee") — the researcher/builder framing stands.
- Whether/when to run the execution-plan brief — deferred on purpose, not an open question. Wait for Vishnu to raise it.
