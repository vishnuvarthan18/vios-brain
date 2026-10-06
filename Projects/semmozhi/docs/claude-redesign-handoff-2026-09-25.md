# Redesign handoff: Tamil "writing surfaces" website (saved 2026-09-25)

READ THIS FIRST in the new chat. It is the full memory of the previous session.

## 1. How we work (rules)
- User (Vishnu) is non-technical. Use simple English. Answer first, no preamble, no filler, no "let me know". Give a recommendation, not a menu. Short.
- Claude does NOT build the website. Claude writes PROMPTS. A separate dev agent (AI coding agent) builds. Claude checks the dev agent's output when the user shows it.
- User gets frustrated by round trips and unverified claims. Say plainly what is real and what is not. Never invent links or file names.
- Do not use the user's Mac Chrome browser (user said no).
- Do not work around blocked domains.

## 2. Project and folder
- Mission and positioning: see `claude/tamil-website-plan.md` (do not lead with "Tamil is the oldest language"; lead with the airtight, sourced case: Sangam literature, 60,000+ inscriptions, Roman trade, UNESCO Chola temples).
- SINGLE project folder on the Mac: `~/Downloads/tamil_harvest` (the only folder to connect). The two other "tamil_harvest" folders are inside it (`tamil_harvest/tamil_harvest` = crawler package, `v2/tamil_harvest` = VPS copy). Do not rename or move them.
- Map of the folder: `PROJECT_MAP.md` in the folder root.
- Sites: `website_live/` = current live site (7 pages: index, tamil, scripts, grantha, vatteluttu, font, fonts). `website/` = older site with pages the live one lacks (about, brahmi, chola, engine, explore, literature). Keep both.
- Not yet saved in git: design/, engines/, scripts/, new spiders, PROJECT_MAP.md. Commit before big changes. `design/references/` is git-ignored on purpose.
- Deleted already (moved to Trash by user): harvest_v2.tgz, site_data.tgz, website_tmp, website.tgz, semmozhi_v3.tgz, semmozhi_v3b.tgz. Remaining: semmozhi_v3c.tgz.

## 3. Design decision (locked in with the user)
Theme = the materials Tamil was written on over time, not only palm leaf:
- palm leaf (ஓலைச்சுவடி) -> literature (Thirukkural, Sangam texts)
- stone -> earliest script (Tamil-Brahmi cave beds), hero stones, temple walls
- copper plate (செப்பேடு) -> royal grants, kings, administration
- pottery sherds -> earliest writing, Sangam-era towns (Keeladi, Arikamedu, Adichanallur)
- coins, rings, seals -> trade, kings, daily life (Roman contact)
Proposed mapping (needs user confirmation): stone+pottery = origins; palm leaf = literature reader (each Kural on its own leaf recommended); copper+seals = kings/grants; temple-wall stone = Chola section (build first); coins/rings = trade.
Build order from the plan: 1) Chola section, 2) literature reader. Script-evolution timeline is not ready (no source content).
Open question: all surfaces on the site, or palm leaf as main theme with others supporting? Recommendation given: use all, one per section.

## 4. Real look of a palm leaf (from photos viewed 2026-09-25; eyeballed, see `design/specs/palm_leaf_visual_notes.md` on the Mac)
- Single leaf: very long thin strip, about 9:1 (Tirukkural scan 2081x233). Warm orange-amber, ~#C98A3F centre, darker brown edges ~#A86A2C. Uneven, slightly rounded ends, small nicks.
- Two round holes set in from the ends (about 30% and 70% along). Text stops around the holes.
- Script: dark brown-black, 5 to 7 tight lines, round Tamil letters. Small notes in the left margin, a red catalogue number at one end.
- Bundle: leaves stacked edge-on = dark brown block (~#4A3426) with fine horizontal lines; tied with orange/yellow cotton thread (vertical wraps + X crossings), small paper number labels, plain wooden boards.
- Stylus (ezhuthani): thin iron point in a wooden handle, plus a small knife.
- Other surfaces in the same museum case: inscribed pot sherds (terracotta red-brown with scratched marks), inscribed conch shell, fan of pale bone strips. That photo is blurry: concept only.
- Round Tamil letter shapes exist because straight strokes split the leaf.
- UI reference (not images): CICT digital library site (digitalarchives.cict.in): cream background, terracotta orange, serif display font with italic accents, cards. CICT images are CC BY-NC (non-commercial only).

## 5. Reference image library (on the Mac)
Location: `~/Downloads/tamil_harvest/design/references/<surface>/`, 256 images, ~742 MB. Sheet: `design/references/LICENSES.csv` (id, surface, object name, license, credit, source URL, notes SHIP_OK or REFERENCE_ONLY). Backup: `LICENSES.before_curation.csv`. Junk moved (not deleted) to `references/_unused/` with `_unused/LICENSES_unused.csv`.
Counts in place: stone 100, palm_leaf 44, pottery 41, coins 37, copper_plate 25, rings 9, seals 0. SHIP_OK (public domain / CC0 / CC BY, credit needed) = about 90. Everything else is mostly CC BY-SA = study only until license checked per file.
Rule: only public domain, CC0, CC BY may ship; CC BY-NC and unclear = reference only. Redrawn original SVG is the safest to ship.
Verdict: Commons is exhausted for coins, rings, seals, copper plates (few Tamil items exist as free images). 100 per surface is NOT achievable; rings and seals must be drawn as original SVG. Museum open-access APIs (Cleveland, Art Institute of Chicago, Met) give CC0 items with direct image URLs but are mostly non-Tamil (Chola sculptures, North Indian coins, Mughal rings, Indus seals). Keep Indus seals out of the Tamil site (contested Indus-Tamil claim).
Strongest real files found: Tirukkural manuscript, Purananuru manuscript, Tolkappiyam leaf, Cilappatikaram-era sherd case, Velvikudi Pandya grant (public domain), Leiden Rajendra Chola charter (CC BY), Chennai Museum copper plates (CC BY), Adichanallur urns and catalogue plates (public domain / CC BY), Chola coins (Kulothunga, Virarajendra public domain), Pudukkottai Roman hoard coins, Tamil-Brahmi cave inscriptions (Arittapatti, Mangulam: CC BY-SA).

## 6. Tools written (all in `~/Downloads/tamil_harvest/design/`)
- `fetch_commons_v2.py` + `commons_sources.csv` (105 queries): downloads from Wikimedia Commons, writes license rows immediately. Run by the USER in Terminal (the sandbox and the Claude shell on the Mac have no internet). `python3 design/fetch_commons_v2.py 2>&1 | tee design/last_run.txt`. `--dry-run`, `--surface X`, `--discover "words"` available.
- `curate_refs.py`: moves off-topic and surplus-series files to `references/_unused/`, re-files mis-sorted items, rewrites LICENSES.csv. `--dry-run` available. Nothing is ever deleted.
- `fetch_commons.py` (v1) and `commons_categories.csv` are old.
- Note: a commit of a changed file to the Mac once arrived stale; patch on the Mac directly with a short python replace if that happens.

## 7. Environment limits learned
- Claude cloud shell: almost no internet (Commons, archive.org, museums blocked). WebFetch cannot open Commons or Wikipedia File: pages (cache-only). WebFetch CAN call museum open-access APIs (Cleveland, AIC, Met) but its output is summarised by a small model: verify every URL.
- Claude shell on the user's Mac (device_bash): also no internet (proxy 403). It can read/write/move files in the connected folder, and run local python.
- Projects tool stores text docs only (an image upload failed).
- Stage files from the Mac into the cloud (device_stage_files) to look at images with Read.
- No delete on the Mac without user permission; moves are fine.

## 8. Next step (what the new chat should do)
Claude writes UI/UX PROMPTS for the dev agent. Suggested sequence:
1. Confirm the surface-to-section mapping and the Kural reader choice (one verse per leaf vs long scroll) with the user (one question each).
2. Prompt 1: design system spec: palette (amber, brown, soot, stone grey, copper green-brown, terracotta), typography (Tamil-capable serif + Noto Sans Brahmi/Adinatha fonts noted in the plan; free fonts only), textures, spacing, components (leaf strip 9:1 with two holes, bundle with thread, stone slab with carved letters, copper plate with ring and seal, coin, sherd), motion (leaf flip, thread untie), accessibility (contrast, Tamil font size), mobile first.
2. Prompt 2: original SVG kit to be drawn by the dev agent from the photo notes (leaf, bundle, stylus, slab, plate, coin, ring, seal, sherd) saved in `design/svg_kit/`.
3. Prompt 3: home page mockup (bundle as navigation).
4. Then Chola section, then literature reader (data: see plan doc; 12 works ready).
Each prompt must name the folder (`website_live` or a new `site/redesign`), the reference files to use (only SHIP_OK in production; add credits page from LICENSES.csv), and acceptance checks.
Decision pending from user: keep `website_live` as base or start a fresh `site/redesign` folder (recommend fresh folder, keep old ones untouched).
