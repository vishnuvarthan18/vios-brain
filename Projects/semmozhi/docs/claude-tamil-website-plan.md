# Tamil Heritage Website — Project Plan

Saved: 2026-09-23

## Mission
Spread the perumai (greatness/pride) of Tamil — the language, the culture, the whole "Tamil DNA" — to the world. Audience: global, including people with zero prior connection to Tamil, not just diaspora.

**Positioning decision (locked in, based on commissioned research):** Do NOT lead with "Tamil is the first/oldest language in the world." That claim is fact-checked false by independent fact-checkers, fails even with Tamil's own strongest academic advocate (George Hart, UC Berkeley — who never makes it), and is currently "won" online only by untrustworthy translation-agency blogs. Instead, build the airtight, undefeatable version of the case: Tamil is the only major Indian literary tradition independent of Sanskrit, with 2,000+ years of unbroken literary production, 60,000+ inscriptions (densest epigraphic record in South Asia), a verified Roman-era maritime trade network (Periplus, Muziris papyrus, Arikamedu), and UNESCO's own World Heritage citation using the phrase "the Tamil civilisation" (Great Living Chola Temples, Criterion iii). This version is more convincing to a skeptical global audience because it survives scrutiny — the maximalist version doesn't.

Full sourced research backing this is saved from session: "Is Tamil a Language or a Civilization?" report (Sept 6, 2026) — covers what's scholarly consensus vs. contested/nationalist claims, comparable cases (Persianate, Hellenistic, Sanskrit cosmopolis), India/TN/UNESCO official positions, and exact language for what's defensible vs. what gets you fact-checked.

## Competitive landscape (research done Sept 6, 2026)
Confirmed: **nothing like this project exists.** ~70 existing Tamil web properties checked. Pattern across all of them: massive archives (Noolaham 6.5M pages, TN state library 5.3M pages) with zero narrative layer; excellent scholarship locked in Tamil-only or in print; a few decaying English sites from the 2000s. English Wikipedia — the actual incumbent competitor — has its Chola dynasty and History of Tamil Nadu articles delisted from Featured status, Keeladi (the most newsworthy Tamil archaeology story) is a stub, no hub article exists.

Real gap confirmed: script evolution + named kings + named literary works + culture, combined, in English, done well, for outsiders — nobody owns this. Closest attempt (tamiltimeline.com) claims the scope but has been "coming soon" for years. Biggest live threat: itihaas.ai, an anonymous AI-generated pan-Indian history site with no citations, already ranking — beatable on credibility/provenance, not on raw volume.

Full report saved from session: "The Tamil Heritage Gap" (Sept 6, 2026) — full competitor table, SEO/demand data, benchmarks to study (Sefaria, hanziyuan.net, Heilbrunn Timeline, ORBIS).

## Data inventory — what we actually have (classified Sept 23, 2026)
Source: data_clean/ → data_classified/ (28,691 records, 776 files). Classifier: classify.py.

**Headline reality check:** only 11,299 of 28,691 records carry any running text at all; the rest are catalogue rows (OSM coordinates, OpenLibrary stubs, Wikidata items, dataset cards). Strip academic-abstract sources too and there are 6,692 records of real readable content, 1,173 of them substantial (≥800 words).

By category (records / prose / non-academic prose / rich ≥800w):
- uncategorized: 9,843 / 3,202 / 2,340 / 333
- literature_texts: 4,954 / 3,235 / 1,709 / 401
- script_language: 3,509 / 1,899 / 773 / 102
- places_heritage: 7,388 / 951 / 624 / 93
- kings_dynasties: 1,337 / 981 / 616 / 139
- culture_religion: 1,516 / 927 / 584 / 100
- trade_external_contact: 144 / 104 / 46 / 5

### What's ready to build NOW
**Chola dynasty — build this first.** Complete enough to look finished: Rajendra I (6,377w) and Rajaraja I (4,111w) full profiles in English + full parallel Tamil-language articles; Chola Empire overview (6,263w); History of Tamil Nadu frame (9,249w); supporting figures with real content (Kundavai 1,303w, Sembiyan Mahadevi 846w, Aditha Karikalan 977w, Rajaditya 777w); place pages (Gangaikonda Cholapuram, Pazhayarai, Uraiyur, Thanjavur); Uttaramerur inscriptions (2,294w); Chola art/architecture (3,196w); museum imagery (Art Institute Chicago, Cleveland).

**Classical Tamil literature reader — build second.** 12 major works with 24,000–66,000 clean, chapter-segmented words each, ready to paginate with near-zero writing needed: Kalittokai, Maṇimēkalai, Patiṟṟuppattu, Tirukkuṟaḷ, Tolkāppiyam, Cilappatikāram, Kamba Rāmāyaṇam, Akanāṉūṟu, Nālaṭiyār, Puṟanāṉūṟu, Paripāṭal, Tēvāram. Eṭṭuttokai complete 8/8. 44 distinct named works total, 555,394 words. Full list of all 46 named kings and all 44 named works with word counts saved in session transcript (Sept 23, 2026) — retrievable on request if not re-derivable from data_classified/.

### What's NOT ready — don't show yet
- **Script evolution / Tamil-Brahmi timeline.** Only passing mentions (127 records mention Tamil-Brahmi, 56 Vaṭṭeḻuttu, 137 Grantha) — no dedicated content. Not buildable without new targeted crawling. Fonts DO exist and are free/open if/when this gets built: Noto Sans Brahmi (Google Fonts, open license) and Adinatha Tamil Brahmi (Tamil-Brahmi-specific letterforms, license needs checking) — see fontinfo.opensuse.org and know-your-heritage.blogspot.com.
- **Chera, Pandya, Pallava king profiles.** Too thin — longest Pallava entry is 462 words, most famous Chera king (Senguttuvan) only 373 words.
- **Trade/external contact.** Only one usable page (Periplus of the Erythraean Sea commentary, 97,501w) — most valuable trade-related item in the whole corpus.
- **places_heritage as reading content.** 7,388 records look big but 5,746 are bare map coordinates — usable as a map layer only, not as prose.
- **Pattuppāṭṭu (10 idylls).** 0/10 exist as primary text, only as Wikipedia stubs — don't claim "the full Sangam corpus," claim Eṭṭuttokai specifically.

### Two crawler bugs to fix before next data run
1. `project_madurai` spider is indexing the site's navigation links instead of fetching the actual e-texts (2,686 junk nav records vs. only 10 real full texts via `project_madurai_texts`). This is costing the single richest source of full Tamil literary primary texts on the open web.
2. `english_wikisource_tamil` keyword-matches too loosely and pulls in unrelated content (Hobson-Jobson, Chaucer, 1911 Britannica/Arabia articles).
Both spiders are already in the uncommitted local working set (per earlier session notes).

## Build plan / phase order
1. **Chola dynasty section** (build first — richest, most complete, most immediately impressive to a first-time visitor).
2. **Classical Tamil literature reader** (build second — most unique asset in the corpus, minimal writing needed, high production value).
3. Fix the two crawler bugs above; consider a fresh, targeted crawl (not blind re-run) aimed specifically at: Pattuppāṭṭu full texts, Pallava/Chera/Pandya king depth, and dedicated Tamil-Brahmi/Vaṭṭeḻuttu/script-evolution sources — before attempting the script timeline feature.
4. Script evolution timeline feature — once real dedicated source content exists (not from current data).
5. Revisit civilizational-framing narrative page (the "airtight case" copy) once the above sections exist to link from.

## Other open decisions from earlier in this project (carried forward)
- Two repos (`tamil-data-collector` private, `tamil-data-dashboard` public) — user decided to merge into one private repo, no public dashboard for now. Scheduled crawls disabled since Sept 5, 2026 (schedule: block commented out in crawl.yml, workflow_dispatch still works). Repo `.git` was 367MB after 8 days of uncleaned history — needs a decision (history cleanup vs. keep data out of git going forward, e.g. LFS or external storage) before crawls resume at scale.
- Local viewer app (`viewer_app/`, stdlib + SQLite/FTS5, `python3 viewer_app/app.py`) exists for browsing/searching data_clean/ locally — private, dev-only, not public.
- User context: non-technical, uses a separate AI coding agent for all code (Claude writes prompts, verifies output). GitHub username `vishnuvarthan18`.
</content>
</invoke>
