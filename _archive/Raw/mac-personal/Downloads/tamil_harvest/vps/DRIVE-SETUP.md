# Save all data to Google Drive (not on the server)

How it works: the server only crawls. Every 2 hours the big data files are copied to your Google Drive folder `semmozhi/raw/` and then deleted from the server. The server keeps only small files (the website data and the "already seen" lists). Your server has ~29 GB free, so this is needed.

## One-time setup (about 10 minutes)

**A. On your Mac** (Terminal):
1. `brew install rclone`   (if you do not have brew: download rclone from rclone.org/downloads)
2. `rclone authorize "drive"`
   A browser opens. Log in to the Google account whose Drive has the space, click Allow.
   Terminal then prints a long text that starts with `{"access_token"...`. Copy that whole line.

**B. On the server** (`ssh ubuntu@40.160.137.239`):
1. `sudo mkdir -p /srv/semmozhi/rclone /srv/semmozhi/data /srv/semmozhi/site && sudo chown -R ubuntu:ubuntu /srv/semmozhi`
2. Create the config file: `nano /srv/semmozhi/rclone/rclone.conf` and paste this (put your copied line where it says PASTE):
```
[gdrive]
type = drive
scope = drive
token = (secret removed)
```
   Save with Ctrl+O, Enter, Ctrl+X.
3. Test (it must print a folder list, no error): `docker run --rm -v /srv/semmozhi/rclone:/root/.config/rclone rclone/rclone lsd gdrive:`

**C. Check** after the first crawl: in Google Drive you will see a folder `semmozhi` with `raw/` inside.

## Good to know
- Google Drive allows about 750 GB of uploads per day and is slow with millions of tiny files. Our files are big (one per run), so this is fine.
- If the Drive gets full, uploads stop and the crawler pauses itself when the server has less than 6 GB free. Nothing is lost.
- The URL lists are backed up to Drive every Sunday (`semmozhi/state_backup/`).
- Your server already runs another project (core-infra: postgres, minio). This project does not touch it. Before starting our website container, check that ports 80/443 are free: `sudo ss -ltnp | grep -E ':80|:443'`. If they are used, tell me and I will change the port.
