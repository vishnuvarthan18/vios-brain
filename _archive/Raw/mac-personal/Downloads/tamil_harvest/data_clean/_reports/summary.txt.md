---
source: personal Mac ~/Downloads/tamil_harvest/data_clean/_reports/summary.txt
---

========================================================================
CLEANING REPORT
========================================================================
source dir              : data/  (776 .jsonl files)
output dir              : data_clean/  (originals not modified)

total records before    : 513541
total records after     : 28691
removed - empty/missing : 56223
removed - duplicate     : 428627
removed - total         : 484850
unparseable lines       : 0
retained                : 5.59%

flagged but KEPT        : 2879
    html               2522
    short              358
    error_text         27

------------------------------------------------------------------------
REMAINING RECORDS BY SOURCE
------------------------------------------------------------------------
source                                 kind          kept    empty      dup    flag
ai4bharat_catalog                      text             1        0       26       1
art_institute_chicago_tamil            text           278    10969    12553     278
arxiv_tamil_nlp                        text           200        0     3469       0
cleveland_museum_tamil                 text            92        0     2594      22
crossref_tamil_studies                 text          1106    26037    20390    1106
doaj_tamil                             text           341      102     5769       8
english_wikisource_tamil               text           178        0     1693       3
huggingface_tamil_datasets             metadata       533        0     8152       0
internet_archive_tamil                 text           676      371     6059      25
internet_archive_tamil_collections     text          1091     6339     7870       6
internet_archive_tamil_fulltext        text            94      113     1138       0
internet_archive_tamil_language        text          1597     1037    25603     173
met_museum_tamil                       metadata       275        0     4349       0
mozilla_data_collective                text            27        0        0      27
openalex_tamil_studies                 text          2359     5692    40819      22
openlibrary_tamil                      metadata      2491        0    45194       0
overpass_tamil_heritage                metadata      5746        0    66668       0
project_madurai                        metadata      2686        0    76973       0
project_madurai_texts                  text            17      238      369       2
tamil_nlp_catalog                      text             1        0       26       1
tamil_wikibooks                        text           420     1496     6584       7
tamil_wikinews                         text           408     1564     6528       0
tamil_wikipedia_categories             text          1082      107     4338       7
tamil_wikipedia_stats                  metadata        13        0        1       0
tamil_wikipedia_texts                  text           416        0     4957       9
tamil_wikiquote                        text           427     1241     6732       0
tamil_wikisource                       metadata        50        0     1250       0
tamil_wikisource_classics              text           510        0     5377       6
tamil_wikisource_texts                 text           491      215      234       0
tamil_wiktionary                       text           384        1        0       0
thevaaram_thirumurai                   text            91        0     1351       0
wikidata_tamil_entities                metadata        12        2      110       1
wikidata_tamil_works                   metadata      1127        0    11073       0
wikimedia_commons_tamil                text           915      221    14451     237
wikipedia_tamil_articles               text            19      144      293       0
wikipedia_tamil_categories             text           414       20     7844       0
wikipedia_tamil_multilingual           text           210        9     2910       0
wikivoyage_tamil_nadu                  text           150       34     2305       0
wiktionary_tamil_etymology             text           746        0     4677       0
zenodo_tamil                           text          1017      271    17898     938
TOTAL                                               28691    56223   428627    2879

------------------------------------------------------------------------
FLAGGED EXAMPLES (source | title | text preview)
------------------------------------------------------------------------
[html]
    ai4bharat_catalog |  | # :bookmark: The Indic NLP Catalog
 _A Collaborative Catalog of Resour
    art_institute_chicago_tamil | Shiva as Lord of the Dance (Nataraja) | <p>Shiva, one of the most important Hindu divinities, is here depicted
    art_institute_chicago_tamil | Buddha Shakyamuni Seated in Meditation (Dhyanamudra) | <p>This meditating Buddha comes from the coastal town of Nagapattinam 
    art_institute_chicago_tamil | Shiva Nataraja Enshrined at Chidambaram Temple with Attendants | <p>Executed in vibrant pigments and gold leaf on cloth stretched over 
    art_institute_chicago_tamil | A Sunday on La Grande Jatte — 1884 | <p>In <em>Ferris Bueller’s Day Off</em>, Ferris’s best friend Cameron 
    art_institute_chicago_tamil | Nighthawks | <p>About <em>Nighthawks</em> Edward Hopper recollected, “unconsciously
    art_institute_chicago_tamil | Lion (One of a Pair, South Pedestal) | <p>Iconic guardians of the Art Institute of Chicago, the <em>Lions</em
    art_institute_chicago_tamil | American Gothic | <p>In <em>American Gothic</em>, Grant Wood directly evoked images of a
[short]
    internet_archive_tamil | Srimath Ramayana Saramrutham Tamil By Dr.Peru. Harikesava Raamaanuja D | ebooks.tirumala.org
    internet_archive_tamil | Alwargal Arucheyalil Avatarangal By Dr. T. Aranganathan In Tamil | ebooks.tirumala.org
    internet_archive_tamil | Gita Makarandam Eranam Pakuti 2 3 4 5 Attiyangal By Dr. K. Sarvothama  | ebooks.tirumala.org
    internet_archive_tamil | Deivatirupukazh By Sri. M.V. Kumar In Tamil | ebooks.tirumala.org
    internet_archive_tamil | FSProd - Thiruvizha (2019) | Tracklist
    internet_archive_tamil | Iraavanan (Singles) | Tracklist
    internet_archive_tamil | IFT PROD - Khalaas (2021) | Tracklist
    internet_archive_tamil | Kathiravan - AAYUTHA EZHUTHU (2024) | Tracklist

full flagged list: data_clean/_reports/flagged.csv (2879 rows)
