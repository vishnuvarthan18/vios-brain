# Moving the Tamil crawler to your VPS (plain-English guide)

**What changes**
- The crawler runs on your VPS every day by itself (no GitHub Actions limits, no 6-hour jobs).
- Data lives in `/srv/semmozhi/data` on the VPS — **not in git** — so the repo stops growing (it hit 357 MB).
- Every run skips pages it already has and only saves what is new or changed.
- The website is built from the data and served from the same VPS (free HTTPS if you have a domain).
- **Super-app layout:** 9 separate engines (language, literature, history, archaeology, culture, places, people, world, photo library). Each has its own topic list (`engines/<name>/config.py`), its own data folder (`data/raw/<name>/`), its own dedup database and its own page on the site (Explore). Change what an engine collects by editing its config file only.
- Crawlers used by the engines (old notes below): `wikipedia_topics` (≈700 topics: language, literature, history, archaeology, culture, places, people, English + Tamil), `commons_deep` (photos **with author + licence**), `wikisource_pattuppattu` (the Ten Idylls and other Sangam texts, with verse line breaks), `project_madurai_index` (fixes the navigation-link bug), `wikidata_tamil_sites` (archaeological sites, temples, writers for maps/timelines).

**What you need**: a VPS with Ubuntu 22.04 or 24.04, 2 GB RAM, 40 GB disk (more is better: your old data is 1.3 GB), and the IP address + root login. A domain name is optional.

> Honest note: the new crawlers were written and unit-tested offline (parsing, dedup, file output) but **have not touched the live internet yet** — my workspace has no access to Wikipedia. Step 6 is a small test run; if a crawler misbehaves, send me the log file from `/srv/semmozhi/data/logs/` and I will fix it.

## Steps

**1. Log in to the VPS** (Mac Terminal): `ssh root@YOUR_VPS_IP`

**2. First-time setup** (installs Docker, firewall, folders):
```
apt-get update && apt-get install -y git curl
```
Then on YOUR computer, from the `tamil_harvest` folder, copy the setup script up and run it:
```
scp vps/setup_vps.sh root@YOUR_VPS_IP:/root/ && ssh root@YOUR_VPS_IP "bash /root/setup_vps.sh"
```

**3. Copy the project code** (code only — no data, no .git):
```
rsync -avz --exclude data --exclude data_clean --exclude data_classified --exclude .git --exclude website --exclude site_data --exclude '*.tgz' --exclude __pycache__ ./ root@YOUR_VPS_IP:/srv/semmozhi/app/
```

**4. Move your existing data** (1.3 GB, one time):
```
rsync -avz --progress data/ root@YOUR_VPS_IP:/srv/semmozhi/data/legacy/
```

**5. Build and start**  (on the VPS):
```
cd /srv/semmozhi/app
docker compose -f vps/docker-compose.yml build
docker compose -f vps/docker-compose.yml up -d web
```
Now `http://YOUR_VPS_IP` shows the website (after step 7). For a domain: point its DNS "A record" at the VPS IP, then edit `vps/Caddyfile` — change `:80` to `yourdomain.org` — and run `docker compose -f vps/docker-compose.yml restart web`.

**6. Test one spider** (about 2–10 minutes):
```
docker compose -f vps/docker-compose.yml run --rm crawler /app/vps/run_crawl.sh archaeology --spiders wikipedia_topics --limit 5
ls /srv/semmozhi/data/raw/archaeology/wikipedia_topics/
```
You should see a `.jsonl` file with about 5-10 lines. If yes, all good.

**7. Publish the website** (copy the site files once, then build the data):
```
# on YOUR computer:
rsync -avz website/ root@YOUR_VPS_IP:/srv/semmozhi/site/
# on the VPS:
docker compose -f vps/docker-compose.yml run --rm crawler /app/vps/build_site.sh
```

**8. Turn on the daily schedule** (on the VPS):
```
crontab /srv/semmozhi/app/vps/crontab.txt
```
Then check progress any time at `http://YOUR_VPS_IP/status.html`.

## Then also
- **Retire GitHub Actions**: crawls are already disabled; you can leave `crawl.yml` as-is.
- **Clean the git repo**: keep `.gitignore`d `data*/` and consider making a fresh repo from the code only (`git init` in a copy without `.git`) — the 357 MB history is only old data.
- **Backups**: `crontab.txt` backs up the dedup database weekly. Also snapshot the VPS disk monthly from your provider's panel.
- **Photos**: only images with a free licence (public domain, CC) come from `commons_deep`, and every record has the author and licence to show as a caption. "Non-profit" does not remove copyright — free-licence + credit is what makes photos safe to publish.

## Commands you will use
```
run_crawl.sh list                     # (use: python3 -m engines.run list)
run_crawl.sh archaeology              # crawl one engine
run_crawl.sh all                      # crawl everything
run_crawl.sh culture --spiders commons_deep
python3 -m engines.run status         # records + last run per engine
build_site.sh [engine]                # rebuild the website data
```

## Mission engine (whole-web)
`run_crawl.sh mission --spiders mission_web` crawls every page of the seed sites (list in `engines/mission/config.py`, add yours), reads sitemaps, and discovers new Tamil sites through links. It keeps every URL it has seen in `state/mission/frontier.db`, so it resumes where it stopped. `commoncrawl_tamil` pulls Tamil pages from the Common Crawl archive (reaches pages no link leads to). `fetch_dumps.sh` downloads the complete Tamil Wikipedia/Wikisource/Wiktionary/Wikibooks/Wikiquote/Wikinews. Everything found is filed under the right topic engine by keywords and shows up on that engine's page (Web pages / Documents tabs). Check progress: `python3 -m engines.run coverage`. Disk: expect tens of GB; use a 200 GB+ disk.
