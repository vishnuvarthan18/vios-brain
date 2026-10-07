# Data audit (2026-10-03)

What the data engine has actually collected, measured from `data/` and `data_clean/`. Read-only; nothing was changed.

## Headline
| | Number |
|---|---|
| Raw records in `data/` (776 files, 1.2 GB) | 513,541 |
| Of those, unique (source + URL/title) | **22,707** (95.6% are repeats from re-running the same crawler) |
| Cleaned records in `data_clean/` | 28,691 (19,198 unique) |
| Records with 1,000+ characters of text | 6,915 |
| Records with real Tamil text (50+ Tamil letters) | ~5,300 |
| Tamil characters in total | **~15.9 million** |
| Last crawl | 2026-09-05 (schedule is off) |
| Topic engines (`engines/`) | 10 defined, **0 records, never run** |

Caveat: Tamil text was counted in each record's main text field (`text`, `content`, `extract`, `abstract`, `description`). A record that keeps its text in another field is undercounted.

## Where the value is
About 10 Wikimedia and archive sources hold nearly all the Tamil text:

| Source | Records | Tamil characters |
|---|---|---|
| tamil_wikisource_texts | 491 | 3.35 M |
| tamil_wikipedia_categories | 1,082 | 2.74 M |
| tamil_wikisource_classics | 510 | 2.70 M |
| tamil_wikiquote | 427 | 1.65 M |
| tamil_wikipedia_texts | 416 | 1.42 M |
| internet_archive_tamil_fulltext | 94 | 1.22 M |
| project_madurai_texts | 17 | 0.95 M |
| tamil_wikibooks | 420 | 0.66 M |
| tamil_wikinews | 408 | 0.50 M |
| internet_archive_tamil_language | 1,597 | 0.18 M |

## Mostly metadata (catalogues, no Tamil text)
openlibrary_tamil (2,491), overpass_tamil_heritage (5,746 map places), wikidata_tamil_works (1,127), project_madurai (index, 2,686), huggingface_tamil_datasets, met_museum, art_institute_chicago, cleveland_museum, arxiv, doaj, openalex, crossref, zenodo. Useful for links, maps, bibliographies and the Lab, not as reading text.

## Problems
1. **Repeats.** Every run kept a new copy. Keep only the latest file per source.
2. **Licence is not recorded** on the Wikimedia text records (only crossref, zenodo, commons and huggingface carry it). Before any of this text is published, each record needs its licence and source URL (see `docs/CONTENT_RULES.md`). Wikipedia/Wikisource text is usually CC BY-SA, which has conditions.
3. **Topic engines never ran**, so Explore and Engine pages have nothing to read (the pages were deleted; see ARCHITECTURE.md).
4. **The website uses almost none of this**: 28 articles and 17 works.

## Recommendation
1. Enable only the ~10 text sources above, plus `wikidata_*` and `commons_deep` for links and images.
2. Keep one latest file per source; archive older runs instead of deleting.
3. Add licence + URL to every record at build time.
4. Run each topic engine once, small, on a laptop; check output before it goes on the server.
