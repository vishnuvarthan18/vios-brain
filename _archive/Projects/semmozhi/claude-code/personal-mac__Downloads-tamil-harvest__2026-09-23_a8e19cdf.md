**Vishnu** (2026-09-23T15:56): Analyze the data in data_clean/ (28,691 cleaned records) to figure out what we can realistically build and show people right now, versus what's too thin to use yet.

1. Classify every record into one of these categories based on title, source spider, and content: script_language, kings_dynasties, literature_texts, culture_religion, trade_external_contact, places_heritage, uncategorized. Add this as a "_category" field, write to data_classified/ (don't touch data_clean/).

2. For kings_dynasties: list every DISTINCT named king/ruler we actually have real content on (not just mentioned in passing — has a real paragraph or more about them). Group by dynasty (Chola, Pandya, Chera, Pallava, Sangam-era chieftains). Give me actual names and counts, not estimates.

3. For literature_texts: list every DISTINCT named literary work we have real content on (Sangam works, epics, devotional texts, etc.), with how much text we have for each (word count is fine).

4. For script_language: what do we actually have on Tamil script evolution (Tamil-Brahmi, Vaṭṭeḻuttu, etc.)? Is there enough to build a visual "script through time" feature, or is this category too thin right now?

5. Pick the 10 single best, richest, most complete records across all categories — ones with real, substantial, well-written content — and show me their titles, sources, and a short excerpt of each. These are candidates for "first thing we could actually publish."

6. Give me an honest verdict: based on what's really in this data (not what we hoped to collect), what's the ONE thing we could build first that would look genuinely good and complete — a king's profile, a literature showcase, a script timeline, something else? What's clearly still too thin to show anyone?

Report exact numbers and real names/titles, not estimates or summaries.

**Claude** (2026-09-23T15:56): I'll start by understanding the data structure.

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:56): Let me look at the record shapes and source distribution.

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:56): _[tool: Bash]_

**Claude** (2026-09-23T15:57): _[tool: Bash]_

**Claude** (2026-09-23T15:57): _[tool: Bash]_

**Claude** (2026-09-23T15:57): _[tool: Bash]_

**Claude** (2026-09-23T15:57): Now I'll build the classifier.

**Claude** (2026-09-23T15:59): _[tool: Bash]_

**Claude** (2026-09-23T15:59): _[tool: Bash]_

**Claude** (2026-09-23T15:59): _[tool: Bash]_

**Claude** (2026-09-23T16:00): _[tool: Bash]_

**Claude** (2026-09-23T16:00): _[tool: Bash]_

**Claude** (2026-09-23T16:00): _[tool: Bash]_

**Claude** (2026-09-23T16:00): Now let me build a named-ruler roster and match it against article titles.

**Claude** (2026-09-23T16:01): _[tool: Bash]_

**Claude** (2026-09-23T16:02): Now the literary works.

**Claude** (2026-09-23T16:03): _[tool: Bash]_

**Claude** (2026-09-23T16:03): _[tool: Bash]_

**Claude** (2026-09-23T16:03): _[tool: Bash]_

**Claude** (2026-09-23T16:03): Now the script_language category.

**Claude** (2026-09-23T16:03): _[tool: Bash]_

**Claude** (2026-09-23T16:03): _[tool: Bash]_

**Claude** (2026-09-23T16:04): _[tool: Bash]_

**Claude** (2026-09-23T16:04): _[tool: Bash]_

**Claude** (2026-09-23T16:04): Now let me find the richest individual records.

**Claude** (2026-09-23T16:04): _[tool: Bash]_

**Claude** (2026-09-23T16:04): _[tool: Bash]_

**Claude** (2026-09-23T16:05): _[tool: Bash]_

**Claude** (2026-09-23T16:06): Done. All 28,691 records classified and written to `data_classified/` (776 files, 119 MB); `data_clean/` untouched.

Each record gained four fields: `_category`, plus `_text_chars`, `_word_count`, and `_prose` (≥400 chars of real text and not from a catalogue source) — the last three are what make the honest numbers below possible. Classifier: [classify.py](../../private/tmp/claude-501/-Users-vishnuvarthanv-Downloads-tamil-harvest/a8e19cdf-0c28-4c5e-8d79-a079a584d8d5/scratchpad/classify.py) in the scratchpad.

## 1. Classification

| category | records | of which prose | prose, non-academic | rich (≥800w) |
|---|---:|---:|---:|---:|
| uncategorized | 9,843 | 3,202 | 2,340 | 333 |
| literature_texts | 4,954 | 3,235 | 1,709 | **401** |
| script_language | 3,509 | 1,899 | 773 | 102 |
| places_heritage | 7,388 | 951 | 624 | 93 |
| kings_dynasties | 1,337 | 981 | 616 | 139 |
| culture_religion | 1,516 | 927 | 584 | 100 |
| trade_external_contact | 144 | 104 | 46 | 5 |
| **total** | **28,691** | **11,299** | **6,692** | **1,173** |

The headline number is misleading and you should know it up front. Of 28,691 records, only 11,299 carry running text at all. The rest are catalogue rows: 5,746 OpenStreetMap heritage features (coordinates + name), 2,686 Project Madurai records that are *just the site's nav links*, 2,491 OpenLibrary stubs, 1,127 Wikidata items, 533 HuggingFace dataset cards. Strip the academic-abstract sources too (OpenAlex, Crossref, Zenodo, DOAJ, arXiv — 5,024 records of NLP paper metadata) and you have **6,692 records of actual readable content, 1,173 of them substantial.**

## 2. kings_dynasties — 46 distinct named rulers with real prose

Matched a curated roster against article titles of prose records from encyclopedic sources only (academic abstracts that merely say "Chola" excluded).

**Chola (15).** Rajendra I (6,377w best, 6 pages) · Rajaraja I / Arulmozhivarman (4,111w, 10 pages) · Kundavai Pirāttiyār (1,303w) · Pugazh Chola Nayanar (1,096w) · Aditha Karikalan (977w) · Sembiyan Mahadevi (846w) · Kulothunga III (802w) · Rajaditya Chola (777w) · Madhurantakan Kandaradityan (736w) · Kulothunga II (641w) · Vikrama Chola (340w) · Boothi Vikramakesari (314w) · Sundara Chola (216w) · Vijayalaya Chola (100w) · Parantaka I (64w).

**Pandya (7).** Marudhu Pandiyar (857w) · Raghunatha Kilavan Sethupathi (685w) · Vikramaditya Varaguna (650w) · Velu Nachiyar (543w) · Sadayavarman Adhiveerarama Pandyan (528w) · Nedunjeliyan "Ariyappadai Kadanda" (345w) · the Maravarman/Jatavarman lines (165w, generic).

**Chera (4).** Chera Perumals / Kulasekhara (2,062w) · Cheran Senguttuvan (373w) · Perum Cheral Irumporai (340w) · Kolathiri (176w).

**Pallava (9).** Kopperunjinga I (462w) · Bhavavarman I (320w) · Narasimhavarman I Mamalla (290w) · Aiyadigal Kadavarkon Nayanar (247w) · Nandivarman II (238w) · Simhavishnu (236w) · Parameshvaravarman II (193w) · Mahendravarman I (177w) · Narasimhavarman II Rajasimha (109w).

**Sangam-era chieftains (11).** Elara/Ellalan (1,244w) · Malaiyamān Thirumudi Kāri (1,237w) · Vēl Pāri (683w) · Athiyamān Nedumān Añci (671w) · Ay Andiran (604w) · Ilandiraiyan (599w) · Ko Kizhan Adikal (571w) · Karunakara Tondaiman (396w) · Irunkōvēl (359w) · Nalli (339w) · Perumbidugu Mutharaiyar II (219w).

Three caveats I'd rather state than hide: the Parantaka I "record" is a Commons image caption, not an article; the Sundara Chola hit is an article about a *fictional character*; and Rajaraja I's 10 pages are largely the same article in 10 languages — the multilingual spider crawled Rajaraja/Rajendra/Chola dynasty in ~30 languages, which inflates counts without adding facts.

**Apply a ≥500-word bar and only 17 rulers survive.** Apply a 1,000-word bar and you have four: Rajendra I, Rajaraja I, Kundavai, Pugazh Chola Nayanar, plus Elara and Malaiyamān Kāri.

## 3. literature_texts — 44 distinct named works, 555,394 words

The strongest asset in the corpus, and it is **primary text, not description**. `tamil_wikisource_classics` yielded 18 works in clean verse form across ~30 chapter-pages each. Verified by reading: Kambaramayanam Sundara Kandam comes through as properly segmented verse with section headings intact.

| work | total words | pages | longest page |
|---|---:|---:|---:|
| Kalittokai | 66,653 | 31 | 49,857 |
| Maṇimēkalai | 64,246 | 37 | 24,754 |
| Patiṟṟuppattu | 57,635 | 28 | 8,064 |
| Tirukkuṟaḷ | 48,802 | 62 | 6,918 |
| Tolkāppiyam | 38,239 | 50 | 6,977 |
| Cilappatikāram | 34,126 | 34 | 9,702 |
| Kamba Rāmāyaṇam | 33,486 | 34 | 4,641 |
| Akanāṉūṟu | 31,178 | 43 | 1,062 |
| Nālaṭiyār | 30,228 | 32 | 1,515 |
| Puṟanāṉūṟu | 30,135 | 35 | 2,326 |
| Paripāṭal | 29,792 | 26 | 6,349 |
| Tēvāram / Tirumuṟai | 29,568 | 32 | 10,544 |
| Aiṅkuṟunūṟu | 10,276 | 34 | 587 |
| Naṟṟiṇai | 10,250 | 34 | 681 |
| Tiruvācakam | 9,195 | 31 | 1,021 |
| Nālāyira Divya Prabandham | 5,506 | 16 | 1,727 |
| Tiruppukaḻ | 4,418 | 5 | 2,225 |
| Kuṟuntokai | 4,011 | 7 | 1,547 |
| Ācārakkōvai | 3,266 | 3 | 1,899 |
| Tirumantiram | 2,366 | 1 | 2,366 |

Plus 24 more at encyclopedia-stub depth only (100–950 words each): Tirumurukāṟṟuppaṭai 944 · Mullaippāṭṭu 819 · Elāti 777 · Neṭunalvāṭai 753 · Ciṟupāṇāṟṟuppaṭai 736 · Paṭṭiṉappālai 700 · Kuṟiñcippāṭṭu 685 · Porunarāṟṟuppaṭai 629 · Iṉiyavai Nāṟpatu 620 · Nāṉmaṇikkaṭikai 618 · Maturaikkāñci 602 · Iṉṉā Nāṟpatu 446 · Tiruvāymoḻi 441 · Ciṟupañcamūlam 440 · Aintiṇai Eḻupatu 432 · Naṉṉūl 423 · Palamoḻi Nāṉūṟu 398 · Aintiṇai Aimpatu 379 · Kār Nāṟpatu 284 · Tirikaṭukam 228 · Iṟaiyaṉār Akapporuḷ 204 · Tiruppāvai 197 · Perumpāṇāṟṟuppaṭai 162 · Malaipaṭukaṭām 101.

Key gap: **Eṭṭuttokai is 8/8 covered, but Pattuppāṭṭu is 0/10 as primary text** — all ten idylls exist only as short Wikipedia stubs. Of the five great epics we have Cilappatikāram and Maṇimēkalai in full; Cīvaka Cintāmaṇi, Valayāpati and Kuṇṭalakēci returned **nothing**, and Periya Purāṇam is absent entirely.

Separately: `project_madurai_texts` holds only **10 real e-texts** (pmuni0001–0007, 2,312–17,114 words each). The 2,686-record `project_madurai` haul next to it is the site's link index — FAQ, volunteers page, contact page. The richest single source of full Tamil literary texts on the web was crawled as a sitemap.

## 4. script_language — too thin. Don't build the timeline.

3,509 records, but the composition kills it: 1,257 are academic abstracts (838 OpenAlex + 198 Crossref + 89 Zenodo + 83 arXiv + 49 DOAJ), mostly Tamil NLP/OCR papers. 1,130 are single-word Wiktionary entries. 634 are OpenLibrary stubs.

I grepped the full corpus for the terms a script timeline needs:

| term | records mentioning |
|---|---:|
| Tamil-Brahmi / Tamil Brahmi | 127 |
| Vaṭṭeḻuttu (Latin + வட்டெழுத்து) | 56 |
| Grantha | 137 |
| puḷḷi | 11 |
| Damili | 4 |

**Every one of these is a passing mention inside an article about something else.** There is no dedicated page on Tamil-Brahmi, none on Vaṭṭeḻuttu, none on Grantha, none on Tamil palaeography. The densest hits are "Chera dynasty" (10 mentions) and "Tamil language" (8). The only long-form script documents in the corpus are the 1911 *Britannica* "Inscriptions" article (34,734w, about Indian inscriptions generally) and two chapters of *The Origin of the Bengali Script* (10,638w) — wrong script, wrong region, swept in by keyword.

There are ~12 individual inscription pages (Uttaramerur 2,294w, Kollam Pillar 447w, Manur 293w, Poonjeri 274w, Mangulam 225w, Thirumittacode 244w) and a handful of Commons inscription photographs. That is material for *one* page about inscriptions as a historical source. It is not a script-evolution feature — you'd be writing the content yourself and using the corpus for nothing but illustrations.

## 5. The ten best single records

| # | title | source | words | excerpt |
|---|---|---|---:|---|
| 1 | Rajendra I | en.wikipedia (via `wikipedia_tamil_articles`) | 6,377 | "Rajendra I (26 July 971 – 1044), often referred to as Rajendra the Great, was a Chola Emperor who reigned from 1014 to 1044… During his reign, the Chola Empire reached its zenith in the Indian subcontinent; it extended its reach via trade and conquest across the Indian Ocean…" |
| 2 | Chola Empire | en.wikipedia | 6,263 | "…a medieval thalassocratic empire based in southern India… evident in their expeditions to the Ganges, naval raids on cities of the Srivijaya Empire on the island of Sumatra, and their repeated embassies to China." |
| 3 | History of Tamil Nadu | en.wikipedia | 9,249 | "The region of Tamil Nadu… shows evidence of having had continuous human habitation from 15,000 BCE to 10,000 BCE… The three ancient Tamil dynasties, namely the Chera, the Chola, and the Pandya, were of ancient origins." |
| 4 | Rajaraja I | en.wikipedia | 4,111 | "Rājarāja I (Middle Tamil: Rājarāja Colan; 3 November 947 – January/February 1014), also known as Rajaraja the Great… Rajaraja's birth name is Arulmozhi Varman." |
| 5 | Chera dynasty | en.wikipedia | 5,249 | "…known as one of the mu-ventar (the Three Crowned Kings) of Tamilakam alongside the Cholas and Pandyas, have been documented as early as the third century BCE. The Chera country was geographically well placed… to profit from maritime trade via the extensive Indian Ocean networks." |
| 6 | சோழர் | ta.wikipedia | 4,867 | "சோழர் (Chola dynasty) என்பவர் பழந்தமிழ்நாட்டை ஆண்ட மூவேந்தர்களுள் ஒரு குலத்தவராவர்… 'சோழ நாடு சோறுடைத்து' என்பது பழமொழி." |
| 7 | பாண்டியர் | ta.wikipedia | 3,234 | "பாண்டியர்கள் என்பவர்கள் பழந்தமிழ் நாட்டை ஆண்ட வேந்தர்களுள் ஒரு குலத்தவராவர்… இந்தியாவில் எந்த ஒரு மன்னர் குலத்துக்கும் இல்லாத நெடிய வரலாறு பாண்டியர்களுக்கு உண்டு." |
| 8 | திருக்குறள் | ta.wikipedia | 6,918 | "…குறள் வெண்பா என்னும் பாவகையினாலான 1,330 ஈரடிச் செய்யுள்களைக் கொண்டது. இந்நூல் முறையே அறம், பொருள், இன்பம் ஆகிய மூன்று தொகுப்புகளைக் கொண்டது." |
| 9 | The Kural or the Maxims of Tiruvalluvar (V. V. S. Aiyar, 1916) | en.wikisource | 14,415 | "THE KURAL OR The Maxims of Tiruvalluvar, TRANSLATED BY V. V. S. AIYAR — 'One of the highest and purest expressions of human thought.' — M. Ariel." |
| 10 | கம்பராமாயணம் / சுந்தர காண்டம் / காட்சிப் படலம் | ta.wikisource | 4,641 | "அசோகவனத்துள் அனுமன் புகுதல் — மாடு நின்ற அம் மணி மலர்ச் சோலையை மருவி, 'தேடி, இவ் வழிக் காண்பெனேல், தீரும் என் சிறுமை…'" |

Honourable mention outside the ten: *Periplus of the Erythraean Sea/Notes* (97,501w, en.wikisource) — genuinely relevant to Tamil trade contact, and single-handedly most of the trade category. Note also that `english_wikisource_tamil`'s biggest records are traps: Hobson-Jobson volumes, Chaucer, *1911 Britannica/Arabia* — keyword matches with no Tamil content.

## 6. Verdict

**Build the Chola dynasty first — specifically a Rajaraja I / Rajendra I pair of profiles anchoring a Chola section.**

It is the only subject where every layer you need is already present and substantial: two emperors at 4,111 and 6,377 words in English *and* full parallel Tamil articles; a 6,263-word empire overview; a 9,249-word historical frame; supporting figures with real articles (Kundavai 1,303w, Sembiyan Mahadevi 846w, Aditha Karikalan 977w, Rajaditya 777w); place pages (Gangaikonda Cholapuram, Pazhayarai, Uraiyur, Thanjavur); the Uttaramerur inscriptions at 2,294 words; Chola art and architecture at 3,196; and museum records with public-domain imagery from the Art Institute of Chicago and Cleveland. A Chola section would look finished because it *is* finished.

**The strong second, and arguably the better product,** is a Sangam/classical literature reader. Twelve works with 24,000–66,000 words of clean primary verse each, chapter-segmented, ready to paginate. Eṭṭuttokai complete at 8/8. This is the most *unusual* thing in the corpus — encyclopedia articles are everywhere, well-structured Tamil primary text is not. It needs no writing from you at all, only presentation. If your goal is "genuinely good and complete," this is it; if your goal is "immediately legible to a general visitor," lead with the Cholas.

**Too thin to show anyone:**
- **Script timeline.** Zero dedicated pages on Tamil-Brahmi or Vaṭṭeḻuttu. Not buildable without new crawling.
- **Trade and external contact.** 144 records, 46 non-academic prose, 5 substantial — and one 97k-word Periplus commentary carries the category. One page at most.
- **Pallava kings.** Nine rulers, the longest at 462 words. Mahendravarman I has 177 words. A Pallava section would be nine stubs.
- **Chera and Pandya profiles.** Four and seven rulers; Senguttuvan, the most famous Chera, has 373 words.
- **Pattuppāṭṭu.** 0/10 as primary text — don't promise "the Sangam corpus," promise Eṭṭuttokai.
- **places_heritage.** Looks like the second-largest category at 7,388 records, but 5,746 are bare OSM coordinate rows. Real prose: 624 records. Usable as a *map layer*, useless as reading.

Two crawler bugs worth fixing before the next run, both of which are costing you the most valuable content: `project_madurai` is indexing links instead of fetching the e-texts behind them (2,686 nav records vs. 10 actual texts), and `english_wikisource_tamil` is keyword-matching so loosely it pulled in Chaucer and *Britannica/Arabia*. Both spiders are in your uncommitted working set.