# Reviewer findings — bug list (started 25 Aug 2026)

External reviewer looked at the technical blueprint export. Vishnu is logging the reviewer's concerns here as they come in, to be worked as bugs/fixes. Building incrementally through the conversation — add to this list, don't relitigate closed items.

## Confirmed issues (to fix)

1. **Coverage numbers are unreliable / not real measurements.**
   "31.6% coverage" is computed from `atlas.db`, not the D1 database the engine actually writes to — the two disagree by orders of magnitude in both directions. Any published completion percentage needs to be recomputed against a single canonical database once that's chosen, not trusted as-is.

2. **The `claim` table — the project's stated differentiator — is empty.**
   Plan says this is what makes the site "a reference work rather than another blog." Zero rows, nothing writes to it. Needs an actual implementation, not just schema.

3. **Status/completion reporting has been caught false at least once (`request_hash` bug).**
   A geometry change didn't invalidate a cached duplicate check, so `elevation` logged a "clean successful run" for a computation that never executed. Every historical `skipped_duplicate` needs auditing — can't trust "done" labels at face value until this is checked project-wide.

4. **Gazetteer stream is at ~4% of target and was reported as fine until checked.**
   58 places vs 800–2,000 expected. Plan calls this the easiest, highest-value stream. Likely an under-querying bug in the Overpass query, not a data limit. Needs root-cause fix, not just re-running.

5. **Project naming is inconsistent across the codebase/docs — collision, not just style.**
   - Whole project called "Sathyamangalam **Atlas**" in most docs (harvest-plan, position doc, handoff) but "Sathyamangalam **Record**" in the outreach doc, and the live site is "sathyamangalam.online."
   - Separately, one of the site's own page slugs is `/record` — distinct from the project-level "Record" name, but easy to confuse.
   - Fix: pick one canonical project name, apply everywhere (repo, docs, site copy, outreach), keep `/record` as just a page label.

## Explicitly ruled out (not issues — don't relitigate)

- Outreach having already gone out to Field Director / Chief Wildlife Warden / NGOs against a partly-stub site — Vishnu confirmed this is *not* a problem.

## Competitive research — 5 comparable projects (25 Aug 2026)

Reviewer's instruction: research similar projects worldwide before assuming our approach is right. Findings:

1. **Atlas of Living Australia (ALA)** — national biodiversity data portal, CSIRO-backed, government-funded. Aggregates species occurrence only, uses Darwin Core standard.
2. **NBN Atlas (UK)** — 385M+ occurrence records, 194 data partners, national biodiversity network trust.
3. **GBIF** — the global backbone all national atlases plug into. Government-funded international network, also occurrence-only.
4. **Minnesota Biodiversity Atlas** — closest in scale to a "landscape" project. Regional, museum-run (Bell Museum), built on 125 years of physical specimen collection, 400,000+ records.
5. **India Biodiversity Portal** — India's own version of this model. Already in our own source-atlas as an unbuilt source (403 on direct fetch, but known quantity).

### What this comparison reveals we're doing wrong

- **Scope mismatch.** Every one of these does ONE thing: species occurrence records. We're combining occurrence + gazetteer/toponyms + legal instruments + historical text mining + literature into a single system. Nobody else attempts all five at once. This is not a small ambition gap.
- **Institutional vs solo.** ALA = CSIRO. NBN = national trust with 194 partner orgs. Minnesota = a museum with 125 years of physical collection behind it. Nobody comparable is one person running automated scrapers and expecting parity in weeks. Argues for a longer, narrower timeline than currently assumed.
- **No precedent for the `claim` table approach.** None of the five actively juxtapose/adjudicate disputed figures as a feature — they publish records with source attached and leave interpretation to the reader. Our `claim` mechanism is genuinely unproven territory, not a known pattern we're failing to copy. Worth knowing there's no playbook to lean on here.
- **No shared data standard adopted.** All five use Darwin Core (or similar) for interoperability. We have a bespoke schema. Gap if we ever want to federate with GBIF / India Biodiversity Portal rather than stay standalone.
- **Launch timing.** None of the five launched or did outreach before their core data was substantially built out — they all reached critical mass first. Reinforces that a 4%-complete gazetteer is not launch-ready, independent of the officials-outreach question (which is separately ruled out as an issue).

## New bug-list items from research

6. **No shared/interoperable data standard.** Consider adopting Darwin Core (or documenting explicitly why not) so records could federate with GBIF/India Biodiversity Portal later instead of staying a closed bespoke schema.
7. **Scope is wider than any comparable project attempts, built by one person.** Either narrow scope to match capacity, or explicitly accept a much longer timeline than currently planned — this needs a decision, not just more building.

## Open / pending

- Waiting for more of the reviewer's feedback before starting fixes. Agent is still running in parallel on the request_hash audit / gazetteer per the 25 Aug handoff.
