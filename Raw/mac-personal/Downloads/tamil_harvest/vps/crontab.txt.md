---
source: personal Mac ~/Downloads/tamil_harvest/vps/crontab.txt
---

# Semmozhi super-app schedule (UTC). Install:  crontab /srv/semmozhi/app/vps/crontab.txt
# One engine per slot so no site is hit by two engines at once. Wikimedia-heavy engines are spread over the week.
SHELL=/bin/bash
C=docker compose -f /srv/semmozhi/app/vps/docker-compose.yml run --rm -T crawler
L=>> /srv/semmozhi/data/logs/cron.log 2>&1
15 0 * * *   $C /app/vps/run_crawl.sh language      $L
15 1 * * *   $C /app/vps/run_crawl.sh literature    $L
15 2 * * *   $C /app/vps/run_crawl.sh history       $L
15 3 * * *   $C /app/vps/run_crawl.sh archaeology   $L
15 4 * * *   $C /app/vps/run_crawl.sh culture       $L
15 5 * * 1,3,5 $C /app/vps/run_crawl.sh places     $L
15 6 * * 2,4,6 $C /app/vps/run_crawl.sh people     $L
15 7 * * 0,3 $C /app/vps/run_crawl.sh world         $L
# MISSION engine: whole-web crawl for up to 6 h every night, Common Crawl monthly, dumps monthly, coverage report daily
0 10 * * *   MISSION_HOURS=6 $C /app/vps/run_crawl.sh mission --spiders mission_web    $L
0 18 1 * *   $C /app/vps/run_crawl.sh mission --spiders commoncrawl_tamil               $L
0 20 2 * *   $C /app/vps/fetch_dumps.sh                                                 $L
30 17 * * *  $C python3 -m engines.run coverage                                         $L
0 */2 * * *  $C python3 -m engines.run archive                                       $L
30 3 * * 0   $C python3 -m engines.run archive --backup-state                          $L
30 18 * * *  $C /app/vps/build_site.sh                                                  $L
#30 3 * * 0   tar czf /srv/semmozhi/backups/state_$(date +\%F).tgz -C /srv/semmozhi/data state logs/runs.jsonl
