---
source: personal Mac ~/india-monorepo/engines/extinct/data/species_list.example.txt
---

# Extinct / possibly-extinct species recorded in India.
#
# THIS FILE IS DELIBERATELY A TEMPLATE, NOT DATA.
#
# PLAN.md §G.7: the Extinct Species engine "is mostly a manual compilation
# project, not a scraping project; budget researcher time, not engineering
# time." Populating this list from an automated source would mean asserting
# that a species is extinct on the authority of nothing — which is precisely
# the kind of unsourced claim the whole platform is built to avoid.
#
# A researcher fills this in from IUCN Red List EX/EW/CR(PE) assessments,
# ZSI records and the primary literature, one line per accepted scientific
# name. scripts/gbif_last_records.py then does the one automatable part:
# looking up each name's most recent GBIF occurrence in India.
#
# The three below are illustrative examples of the FORMAT ONLY. Verify each
# against a primary assessment before treating any of them as a finding.
#
# Acinonyx jubatus venaticus    # Asiatic cheetah — extirpated from India, 1952
# Rhinoceros sondaicus          # Javan rhinoceros — extirpated from India
# Ophrysia superciliosa         # Himalayan quail — last confirmed record 1876
