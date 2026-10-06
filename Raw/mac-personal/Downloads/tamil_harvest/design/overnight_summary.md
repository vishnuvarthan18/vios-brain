# Overnight Commons harvest - summary

- Status: **finished**
- Run 5 (full run), started 2026-09-25 21:32, ended 2026-09-26 07:17 (9h 45m)
- This run: 2090 downloads, 2063 kept, 26 moved to _dupes, 0 errors, 3559 requests

## Coverage against 500 per surface

| surface | counted | of 500 | direct | regional | technique (not counted) | off (kept, owner check) | SHIP | STUDY | new this run | status |
|---|---|---|---|---|---|---|---|---|---|---|
| stone | 500 | 100% | 492 | 8 | 50 | 12 | 130 | 432 | 435 | target reached |
| palm_leaf | 132 | 26% | 100 | 32 | 50 | 0 | 76 | 106 | 134 | sources exhausted |
| pottery | 147 | 29% | 131 | 16 | 50 | 5 | 58 | 144 | 156 | sources exhausted |
| coins | 78 | 15% | 69 | 9 | 50 | 1 | 40 | 89 | 87 | sources exhausted |
| copper_plate | 65 | 13% | 47 | 18 | 50 | 1 | 69 | 47 | 86 | sources exhausted |
| rings | 4 | 0% | 3 | 1 | 50 | 5 | 23 | 36 | 45 | sources exhausted |
| seals | 14 | 2% | 13 | 1 | 50 | 0 | 30 | 34 | 59 | sources exhausted |
| temples | 500 | 100% | 483 | 17 | 21 | 0 | 110 | 411 | 516 | target reached |
| bronzes | 500 | 100% | 450 | 50 | 50 | 0 | 351 | 199 | 545 | target reached |

Counted = direct + regional files in `references/<surface>/`. `tamil_relevance` is automatic (keywords in the Commons title, description and categories), not reviewed. SHIP = public domain, CC0 or CC BY per the Commons API license; STUDY = everything else.

## Disk

- `design/references/` now 2.24 GB (surface folders 1.83 GB, _dupes 0.03 GB, _unused 0.38 GB)
- added this run: 1410 MB
- free disk now: 192.6 GB (the run stops below 20 GB)

## Duplicates (nothing deleted)

- exact copies moved to `references/_dupes/`: 0; near duplicates (perceptual hash distance <= 6): 26; same Commons file seen again, not downloaded: 0. Details: `references/_dupes/_dupes.csv`.

## Skipped files

### Errors (download or network) - retried automatically on the next run, max 3 runs

None.

### Skipped or rejected, by reason (all runs)

| decision | reason code | files |
|---|---|---|
| rejected | not_surface | 7629 |
| skipped | too_small | 2207 |
| rejected | junk_word | 1688 |
| rejected | excluded_word | 459 |
| rejected | no_context | 280 |
| rejected | modern | 6 |
| rejected | blank_image | 1 |

Reason codes: not_surface / no_context / excluded_word / junk_word / modern = not the right object (never downloaded); too_small = long side under 1200 px; mime = not a photo format; http = a final HTTP answer such as 404. One line per file: `design/reference_engine/skipped.csv`.

## Import of the original library

| surface | expected rows | imported rows |
|---|---|---|
| stone | 100 | 100 |
| palm_leaf | 44 | 44 |
| pottery | 41 | 41 |
| coins | 37 | 37 |
| copper_plate | 25 | 25 |
| rings | 9 | 9 |
| seals | 0 | 0 |

- Counts match: **yes**; SHIP rows: 90; rows whose file is missing: none.
- Rows that share one file on disk (Commons names that differ only by case, macOS ignores case): [['STO-070', 'STO-071']].
- Duplicate ids already in LICENSES.csv (left as they are): PAL-028, PAL-029, PAL-030, PAL-031, POT-011, POT-012, POT-013, POT-014, POT-035, RIN-003, STO-086, STO-087, STO-088, STO-089, STO-090, STO-091, STO-092, STO-093, STO-094, STO-095, STO-096, STO-097, STO-098, STO-099, STO-100.
- Near-duplicate pairs among the imported files (left in place): [['STO-038', 'STO-044', 0], ['STO-064', 'STO-065', 0], ['STO-070', 'STO-071', 0], ['COP-001', 'COP-002', 6], ['COP-004', 'COP-005', 2], ['STO-087', 'STO-088', 0]].

### Needs the owner's decision

Imported rows the keyword check calls off-topic (kept in place, marked `off`):

| id | file | why |
|---|---|---|
| STO-003 | Front_copy.jpg | 'book covers' |
| STO-007 | Marumakan.jpg | 'inscri' without stone context |
| STO-009 | Parthivapuram_Grant__9th_century_AD__south_India___plates_I_and_V_.jpg | 'copper plate' |
| STO-018 | Tamil_Language.jpg | 'inscri' without stone context |
| STO-021 | THAMIZHI.jpg | 'signs' |
| STO-029 | ப_ரத__ந_ன_வ__இல_லம__ப_யர__பலக_.JPG | 'inscri' without stone context |
| STO-031 | 01History_Tamil_Script_by_Centuries_Dakshinchitra_Mamallapuram_2011.jpg | 'charts' |
| STO-034 | History_of_Tamil_script.jpg | 'charts' |
| STO-041 | Old_Tamil.png | 'font' |
| STO-043 | Tamil_Brahmi.png | 'chart' |
| STO-046 | История_тамильской_письменности.png | 'charts' |
| STO-049 | பழந_தம_ழ__எழ_த_த__வளர_ச_ச_.JPG | 'charts' |
| POT-011 | Keeladi-archeological-site-photos_01.jpg | no pottery word |
| POT-012 | Keeladi-archeological-site-photos_02.jpg | no pottery word |
| POT-013 | Keeladi-archeological-site-photos_03.jpg | no pottery word |
| POT-014 | Keeladi-archeological-site-photos_04.jpg | no pottery word |
| COP-022 | Arittapatti_inscriptions_in_Tamil_Brahmi_mentioning_Paravar.jpg | no copper_plate word |
| POT-035 | Archaeological_Excavation__Kodumanal.jpg | no pottery word |
| COI-020 | Coin_of_Western_Roman_Emperor_Valentinian_III_who_reigned_from_425-455_CE__Worthing_Museum_and_Art_Gallery.jpg | 'roman emperor' |
| RIN-004 | Toe_Ring_south_Indian_culture.jpg | 'ring' without rings context |
| RIN-005 | Toe_ring_in_a_traditional_hindu_wedding.jpg | 'wedding' |
| RIN-013 | Nose_ring__Himachal_Pradesh__India__19th_century__gold_and_enamel__Honolulu_Academy_of_Arts.JPG | 'nose ring' |
| RIN-014 | Nose_ring__northern_India__early_20th_century__gold__rubies__emeralds__diamonds__pearls_and_enamel__HAA.JPG | 'nose ring' |
| RIN-022 | Antique_Indian_Nose_Ring_Jewellery.jpg | 'nose ring' |

## Sources that added nothing this run

| source | files listed | ended |
|---|---|---|
| 060 pottery category: Category:Keezhadi archeological site | 348 | exhausted |
| 092 coins category: Category:Coins of the Chola dynasty | 30 | exhausted |
| 131 copper_plate category: Category:Copper plate Tamil language inscriptions | 8 | exhausted |
| 152 rings category: Category:Finger rings in India | 0 | exhausted |
| 034 palm_leaf category: Category:Palm-leaf manuscripts in Tamil | 24 | exhausted |
| 132 copper_plate category: Category:Copper plate inscriptions in Tamil script | 0 | exhausted |
| 153 rings category: Category:Jewellery of Tamil Nadu | 108 | exhausted |
| 002 stone category: Category:Inscriptions at the Brihadisvara Temple | 0 | exhausted |
| 035 palm_leaf category: Category:Tamilnadu Govt-Oriental Manuscripts Library And Research Center | 0 | exhausted |
| 062 pottery category: Category:Adichanallur earthenware burial urns | 0 | exhausted |
| 094 coins category: Category:Coins of the Vijayanagara Empire | 26 | exhausted |
| 133 copper_plate category: Category:Copper plate Tamil language inscriptions in Grantha script | 0 | exhausted |
| 174 seals category: Category:Seal impressions | 456 | exhausted |
| 003 stone category: Category:Tamil Brahmi script | 0 | exhausted |
| 036 palm_leaf category: Category:Sarasvati Mahal Library | 74 | exhausted |
| 175 seals category: Category:Stamp seals of archaeology | 650 | exhausted |
| 004 stone category: Category:Tamil language Vatteluttu inscriptions | 0 | exhausted |
| 156 rings search: signet ring ancient India | 4 | exhausted |
| 005 stone category: Category:Vatteluttu script | 48 | exhausted |
| 097 coins category: Category:Setu coins | 22 | exhausted |
| 157 rings search: Roman intaglio ring India | 4 | exhausted |
| 177 seals search: Leiden plates seal | 96 | exhausted |
| 098 coins category: Category:Jaffna kingdom | 122 | exhausted |
| 137 copper_plate category: Category:Vatteluttu script | 132 | exhausted |
| 158 rings search: inscribed ring Tamil | 0 | exhausted |
| 178 seals search: Pallava seal | 0 | exhausted |
| 007 stone category: Category:Tamil inscriptions in Sri Lanka | 0 | exhausted |
| 067 pottery search: Tamil Brahmi potsherd | 4 | exhausted |
| 138 copper_plate search: Velvikudi copper plate | 20 | exhausted |
| 159 rings search: Chola ring | 120 | exhausted |
| 179 seals search: Anaikoddai seal | 4 | exhausted |
| 041 palm_leaf search: ola chuvadi | 0 | exhausted |
| 068 pottery search: Keeladi potsherd | 0 | exhausted |
| 100 coins category: Category:Art of the Pandyan Dynasty | 116 | exhausted |
| 139 copper_plate search: Leiden copper plates Chola | 96 | exhausted |
| 160 rings search: gold ring Arikamedu | 0 | exhausted |
| 180 seals search: Tamil seal inscription | 118 | exhausted |
| 042 palm_leaf search: olai chuvadi | 0 | exhausted |
| 069 pottery search: Keeladi excavation | 352 | exhausted |
| 140 copper_plate search: Anaimangalam copper plates | 0 | exhausted |
| 161 rings search: South Indian gold ring museum | 32 | exhausted |
| 181 seals search: clay seal Tamil Brahmi | 0 | exhausted |
| 010 stone category: Category:Sittanavasal Cave inscriptions | 0 | exhausted |
| 043 palm_leaf search: olaichuvadi | 2 | exhausted |
| 070 pottery search: Keezhadi artefacts | 14 | exhausted |
| 141 copper_plate search: Sinnamanur copper plates Pandya | 0 | exhausted |
| 162 rings search: Sangam gold ring | 0 | exhausted |
| 182 seals search: Keeladi seal | 0 | exhausted |
| 044 palm_leaf search: palm leaf manuscript Tamil Nadu | 70 | exhausted |
| 071 pottery search: graffiti potsherd Tamil | 0 | exhausted |
| 103 coins search: Pandya coin | 14 | exhausted |
| 163 rings search: Kodumanal ring | 0 | exhausted |
| 183 seals search: Roman intaglio Arikamedu | 0 | exhausted |
| 045 palm_leaf search: Tamil manuscript stylus ezhuthani | 0 | exhausted |
| 072 pottery search: Arikamedu pottery | 4 | exhausted |
| 104 coins search: Chera coin | 28 | exhausted |
| 143 copper_plate search: Pallava copper plate | 10 | exhausted |
| 164 rings search: Keeladi ring | 0 | exhausted |
| 184 seals search: South India seal ancient | 20 | exhausted |
| 233 bronzes search: bronze Chola Cleveland Museum of Art | 24 | exhausted |
| 046 palm_leaf search: Thirukkural palm leaf | 2 | exhausted |
| 073 pottery search: black and red ware South India | 2 | exhausted |
| 105 coins search: Sangam coin | 0 | exhausted |
| 144 copper_plate search: Thiruvalangadu copper plates | 0 | exhausted |
| 165 rings search: ancient Indian seal ring | 100 | exhausted |
| 185 seals search: temple seal Tamil | 4 | exhausted |
| 234 bronzes search: Chola bronze Metropolitan Museum | 3 | exhausted |
| 047 palm_leaf search: palm leaf bundle Tamil | 1000 | exhausted |
| 074 pottery search: rouletted ware Arikamedu | 0 | exhausted |
| 145 copper_plate search: Chola copper plate seal | 96 | exhausted |
| 166 rings search: Sri Lanka Tamil ring | 24 | exhausted |
| 186 seals search: intaglio India ancient gem | 0 | exhausted |
| 048 palm_leaf search: Tamil medical palm leaf manuscript | 0 | exhausted |
| 075 pottery search: Kodumanal | 2 | exhausted |
| 107 coins search: Roman coin hoard India | 10 | exhausted |
| 146 copper_plate search: Karandai plates | 0 | exhausted |
| 167 rings search: ring India Cleveland Museum of Art | 40 | exhausted |
| 187 seals search: sealing terracotta Tamil Nadu | 0 | exhausted |
| 236 bronzes search: Nayaka period bronze | 1 | exhausted |
| 049 palm_leaf search: Tamil Heritage Foundation manuscript | 2 | exhausted |
| 076 pottery search: Korkai excavation | 0 | exhausted |
| 108 coins search: Roman coin Tamil Nadu | 6 | exhausted |
| 168 rings search: Indian gold ring 19th century | 24 | exhausted |
| 188 seals search: copper plate seal ring Chola Pandya | 0 | exhausted |
| 050 palm_leaf search: Sarasvati Mahal Library palm leaf | 1000 | exhausted |
| 077 pottery search: Sivagalai excavation | 0 | exhausted |
| 148 copper_plate search: Kerala copper plate Vatteluttu | 20 | exhausted |
| 189 seals search: Indian seal matrix medieval | 2 | exhausted |
| 078 pottery search: Porunthal | 0 | exhausted |
| 110 coins search: gold pagoda coin | 1994 | exhausted |
| 149 copper_plate search: செப்பேடு | 2 | exhausted |
| 170 rings search: மோதிரம் | 2 | exhausted |
| 052 palm_leaf search: Tamil manuscript leaf | 1886 | exhausted |
| 079 pottery search: Alagankulam | 0 | exhausted |
| 111 coins search: Vijayanagara coin | 30 | exhausted |
| 150 copper_plate search: செப்பேடுகள் | 2 | exhausted |
| 171 rings search: தங்க மோதிரம் | 0 | exhausted |
| 191 seals search: சோழர் முத்திரை | 0 | exhausted |
| 020 stone search: Jain cave Tamil Brahmi | 33 | exhausted |
| 053 palm_leaf search: palm leaf manuscript Sri Lanka | 86 | exhausted |
| 080 pottery search: Sangam age pottery | 2 | exhausted |
| 151 copper_plate search: செப்புப் பட்டயம் | 0 | exhausted |
| 081 pottery search: Iron Age urn burial Tamil Nadu | 0 | exhausted |
| 113 coins search: Jaffna coin Aryacakravarti | 0 | exhausted |
| 242 bronzes search: செப்புத் திருமேனி | 0 | exhausted |
| 082 pottery search: Adichanallur urn | 20 | exhausted |
| 243 bronzes search: ஐம்பொன் சிலை | 0 | exhausted |
| 056 palm_leaf search: ஓலைச் சுவடி | 0 | exhausted |
| 083 pottery search: megalithic burial urn South India | 2 | exhausted |
| 115 coins search: Rajaraja coin | 8 | exhausted |
| 244 bronzes search: நடராஜர் சிலை | 11 | exhausted |
| 116 coins search: Rajendra Chola coin | 6 | exhausted |
| 245 bronzes search: நடராசர் | 6 | exhausted |
| 085 pottery search: terracotta Government Museum Chennai | 0 | exhausted |
| 117 coins search: Kongu coin | 8 | exhausted |
| 059 palm_leaf search: எழுத்தாணி | 6 | exhausted |
| 119 coins search: Pandyan fish coin | 10 | exhausted |
| 088 pottery search: ஆதிச்சநல்லூர் | 4 | exhausted |
| 120 coins search: Cholas coin gold Kasu | 4 | exhausted |
| 089 pottery search: அகழாய்வு | 42 | exhausted |
| 121 coins search: Indian coin FindID Pandya OR Chola | 1990 | exhausted |
| 122 coins search: Ceylon punch-marked coin | 0 | exhausted |
| 123 coins search: Sri Lanka Roman coin hoard | 0 | exhausted |
| 124 coins search: Tranquebar coin | 2 | exhausted |
| 128 coins search: சோழர் நாணயம் | 0 | exhausted |
| 129 coins search: பாண்டியர் நாணயம் | 0 | exhausted |
| 130 coins search: சேரர் நாணயம் | 0 | exhausted |

## Resume

Same command; files already in `design/reference_engine/refs.db` are skipped:

```
cd ~/Downloads/tamil_harvest && nohup caffeinate -ims python3 design/fetch_commons_v3.py --supervise >> design/reference_engine/overnight_stdout.log 2>&1 &
```

Or in the foreground (Ctrl+C stops cleanly):

```
cd ~/Downloads/tamil_harvest && caffeinate -ims python3 design/fetch_commons_v3.py --supervise
```
