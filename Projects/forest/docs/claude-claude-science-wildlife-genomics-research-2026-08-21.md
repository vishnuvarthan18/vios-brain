# Claude Science Fit — Wildlife/Animal Genomics Ideas for the Nilgiris Landscape
**Date:** 21 August 2026
**Status:** Research/exploration only — nothing built yet, no decision locked

---

## Context

Vishnu asked how "Claude Science" (Anthropic's beta desktop app for scientific research —
genomics, proteomics, cheminformatics, single-cell/RNA-seq, 60+ database connectors like
PubMed/ChEMBL) could apply to the conservation/forest work, on top of the existing
Sathyamangalam/Senna satellite-mapping track (which stays locked and unaffected).

First pass explored plant genetics (Senna spectabilis, Lantana camara) — user corrected this:
**scope must stay strictly animal / wildlife / forest / mountain**, not invasive plants.

## What Claude Science actually is

Beta macOS/Linux desktop app. Native rendering for proteins, molecular structures, genomic
tracks; pre-built modules for genomics, single-cell RNA-seq, proteomics, structural biology,
cheminformatics; 60+ scientific database connectors; runs analysis locally/HPC/cloud;
reproducible pipelines saved as reusable skills. Built for molecular/wet-lab-adjacent science,
not remote sensing — so it's a genuine complement to the GEE/Sentinel-2 satellite track, not a
replacement.

## Top animal/wildlife/mountain ideas researched, ranked by evidence strength

### 1. Elephant TB (tuberculosis) surveillance — strongest, most concrete
Three wild bull elephants confirmed *Mycobacterium tuberculosis*-positive in Wayanad Wildlife
Sanctuary (Muthanga/Kurichiyat ranges) 2007–2013, confirmed via PCR + sequencing of three gene
regions (16S–23S ITS, hsp65, rpoB). Wayanad is inside the same Nilgiri Biosphere Reserve as
Sathyamangalam/Mudumalai. Authors explicitly call the epidemiology "yet to be elucidated" and
ask for continued surveillance — an open, admitted gap in the exact landscape.
Source: https://wwwnc.cdc.gov/eid/article/23/3/16-1741_article

### 2. Elephant corridor gene-flow genomics — directly complements the locked GEE work
2024 Conservation Genetics paper: 379 elephants genotyped (10 microsatellites) + 33 individuals
mtDNA D-loop sequenced across Peninsular India (two Western Ghats populations split by the
Palghat Gap, plus an Eastern-Central India population). Finding: ongoing gene flow, 39% mixed
ancestry, recommends managing as one conservation unit. Gap: does NOT drill into the
Sathyamangalam–Nilgiris–Mudumalai corridor specifically. Pairs naturally with the satellite
project — GEE shows whether forest canopy is connected; elephant DNA would show whether
elephants are genetically using that connection.
Source: https://link.springer.com/article/10.1007/s10592-024-01630-w

### 3. Nilgiri tahr population divergence — flagship endemic mountain species
Published study found the Palghat Gap produced two genetically diverged Nilgiri tahr
populations in the Western Ghats. Real, peer-reviewed, confirms active genetics research on
this species — but full methods/sample sizes were NOT verified (page blocked by CAPTCHA on
fetch attempt). Needs a second, more careful pull before relying on details.
Source: https://pmc.ncbi.nlm.nih.gov/articles/PMC7800121/ (unverified detail)

### 4. Tiger landscape-genetics corridor functionality — methodology exists, not applied here
Terai Arc Landscape studies established a landscape-genetics + modelling method to test
whether tiger corridors are functionally connecting populations (not just present on a map).
No hit found applying this to the Sathyamangalam–BRT–Nilgiris–Mudumalai tiger corridor
specifically — but this is absence-of-a-hit, not a confirmed gap like #1/#2.
Source: https://link.springer.com/article/10.1007/s10592-022-01460-8

### 5. eDNA stream biodiversity monitoring — real method, weakest local evidence
Metabarcoding/eDNA methodology is well established and growing (including a forest-carbon-market
angle — eDNA proposed for biodiversity co-benefit verification). Western Ghats stream fish
diversity in protected areas has been studied, but no eDNA-specific study found for
Sathyamangalam or immediate Nilgiris streams specifically.
Sources: https://www.nature.com/articles/s43247-024-01970-y ,
https://www.cambridge.org/core/journals/oryx/article/do-terrestrial-protected-areas-conserve-freshwater-fish-diversity-results-from-the-western-ghats-of-india/34ED61F055BCFF475691A823852D488A

## Ruled out / deprioritized
- **Lantana camara biotype mapping** — real dataset exists (NCBI BioProject PRJNA1471594, 359
  individuals, 36 Indian sites incl. Ooty/Nilgiris, ddRAD-seq, 19,008 SNPs) but explicitly
  out of scope per user's correction: plant genetics, not animal/wildlife.
- **Senna spectabilis phylogeography** — same reason, out of scope.
- **Wildlife forensics / COI barcoding** — WII already institutionally owns this niche
  (dedicated Wildlife Forensic & Conservation Genetics Cell); weak wedge for an outsider.

## Status / next step
Nothing built. #1 (elephant TB) and #2 (elephant corridor gene-flow) are the strongest
candidates — both real, sourced, landscape-specific, and #2 directly reinforces the existing
GEE/Sentinel-2 regrowth-mapping track rather than competing with it. Not yet decided which (if
either) becomes an actual project doc / build.

## Unrelated note from this session
A status update on the **separate Sathyamangalam harvest-engine coding session** was relayed
here (Stage 0-3 committed and pushed; Stage 4 government-document crawlers built but blocked on
a wrangler@3→4 upgrade for PDF extraction, approved but execution status unconfirmed — that
session is not visible from this one, so its overnight run outcome is unknown and needs to be
checked directly in that session).
