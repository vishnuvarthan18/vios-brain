# Moving the Tamil crawler to your VPS (plain-English guide)

**What changes**
- The crawler runs on your VPS every day by itself (no GitHub Actions limits, no 6-hour jobs).
- Data lives in `/srv/semmozhi/data` on the VPS — **not in git** — so the repo stops growing (it hit 357 MB).
- Every run skips pages it already has and only saves what is new or changed.
- The website is built from the data and served from the same VPS (free HTTPS if you have a domain).
- 5 new crawlers fill the gaps: `wikipedia_topics` (≈700 topics: language, literature, history, archaeology, culture, places, people, English + Tamil), `commons_deep` (photos **with author + licence**), `wikisource_pattuppattu` (the Ten Idylls and other Sangam texts, with verse line breaks), `project_madurai_index` (fixes the navigation-link bug), `wikidata_tamil_sites` (archaeological sites, temples, writers for maps/timelines).

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
docker compose -f vps/docker-compose.yml run --rm crawler scrapy crawl wikipedia_topics -s CLOSESPIDER_ITEMCOUNT=20
ls /srv/semmozhi/data/raw/wikipedia_topics/
```
You should see a `.jsonl` file with about 20 lines. If yes, all good.

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
