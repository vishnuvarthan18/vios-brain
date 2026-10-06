# Semmozhi status (2026-09-24, end of session 3) — READ THIS FIRST in a new chat

## Who / goal
vishnu, not a technical person: explain in plain, simple English, step by step. Non-profit Tamil heritage website (Semmozhi / செம்மொழி) + a big data engine. Positioning: NOT "first language / everyone spoke Tamil"; strongest provable case + a "Myths vs Evidence" page. Photos: free licence only (Commons), with author + licence caption.

## Where things are
Mac folder `~/Downloads/tamil_harvest` (a git repo tied to github.com/vishnuvarthan18/tamil-data-collector, .git is 357 MB of old data; plan: make a fresh small code-only repo, data never in git; user has not decided yet, nothing committed).
Latest code is already extracted into that folder (from semmozhi_v3c.tgz): `engines/`, `tamil_harvest/`, `scripts/`, `vps/`, `website/`. Leftover files (tgz, v2/, website_tmp, m_*.png) could not be deleted (needs delete approval).

## What is built (all tested only offline / on a local fake site; NOTHING run against the live internet yet)
- Website v1: Home, Cholas, Literature reader (Tirukkural), Brahmi Lab, Font page, About. Plus `explore.html` (hub + search) and `engine.html?e=<engine>` (tabs Articles, Texts, Photos, Places, Research, Books, Web pages, Documents).
- 9 topic engines (`engines/<name>/config.py`): language, literature, history, archaeology, culture, places, people, world, media. Runner: `python3 -m engines.run list|crawl|build|status|coverage|archive`.
- MISSION engine (whole web): `mission_web` (seeds, sitemaps, link discovery, resumable frontier.db), `commoncrawl_tamil` (CDX filter syntax UNVERIFIED), Wikimedia dumps (`scripts/dump_import.py`, `vps/fetch_dumps.sh`), router to topics, `coverage` report. "Zero missed data" is impossible; goal is max coverage + measured gaps. Web pages shown on site as excerpt + link only.
- Architecture diagram: semmozhi_architecture.html (in outputs).

## Decision this session: data goes to GOOGLE DRIVE, not the server
User has 50-500 GB on Drive. `engines/archive.py`: compacts raw files (small local copy for the site builder) then `rclone move` raw to `gdrive:semmozhi/raw` every 2 h; state DBs backed up weekly to `state_backup/`; crawler pauses if server free space < 6 GB. Setup guide: `vps/DRIVE-SETUP.md` (rclone authorize on Mac -> paste token into /srv/semmozhi/rclone/rclone.conf on server -> test `lsd gdrive:`). Compaction tested; the rclone upload itself NOT tested.

## The user's server
ssh ubuntu@40.160.137.239 (Ubuntu 26.04, 38 GB disk, ~29 GB free, 2+ docker containers of ANOTHER project `core-infra`: core-postgres, core-minio; do not touch). Docker works there. Our Caddy web container wants ports 80/443: must check `sudo ss -ltnp | grep -E ':80|:443'` first; change port if taken. Claude cannot ssh; the user runs commands in Terminal, so give copy-paste steps. (The user's Mac has 199 GB free; Docker is not running on the Mac.)

## NEXT STEPS (in order)
1. User does Drive login (DRIVE-SETUP.md steps A+B) and reports the `lsd gdrive:` result.
2. Ask for output of the ports check.
3. Walk user through: rsync/scp code to /srv/semmozhi/app (excluding data, .git, website/data), `docker compose -f vps/docker-compose.yml build`, small test `run_crawl.sh archaeology --spiders wikipedia_topics --limit 5`, then check `python3 -m engines.run status`. Fix bugs from logs.
4. Upload the old 1.3 GB `data/` from the Mac straight to Drive with rclone (not via server).
5. Start nightly cron (`vps/crontab.txt`), then review the seed list in `engines/mission/config.py` with the user.
6. When real data exists: build Language, Archaeology, History timeline, Culture pages + Myths vs Evidence.
7. Set up a fresh code-only git repo (needs user OK).

## Open items
About-page footnotes (4 claims); Tevaram sample thin; Adinatha font licence; seed domains partly unverified guesses; `classify.py` not found; old files to delete on the Mac.
