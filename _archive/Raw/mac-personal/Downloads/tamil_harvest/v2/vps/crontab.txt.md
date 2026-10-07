---
source: personal Mac ~/Downloads/tamil_harvest/v2/vps/crontab.txt
---

# Semmozhi harvest schedule (UTC). Install with:  crontab /srv/semmozhi/app/vps/crontab.txt
# The three groups run on different days/hours so Wikimedia sites are not hit all at once.
SHELL=/bin/bash
C=docker compose -f /srv/semmozhi/app/vps/docker-compose.yml run --rm -T crawler
15 1 * * *   $C /app/vps/run_crawl.sh new        >> /srv/semmozhi/data/logs/cron.log 2>&1
15 4 * * 1,4 $C /app/vps/run_crawl.sh wikimedia  >> /srv/semmozhi/data/logs/cron.log 2>&1
15 9 * * 2,5 $C /app/vps/run_crawl.sh other      >> /srv/semmozhi/data/logs/cron.log 2>&1
45 12 * * *  $C /app/vps/build_site.sh            >> /srv/semmozhi/data/logs/cron.log 2>&1
# weekly backup of the state DB and run logs
30 3 * * 0   tar czf /srv/semmozhi/backups/state_$(date +\%F).tgz -C /srv/semmozhi/data state/seen.db logs/runs.jsonl
