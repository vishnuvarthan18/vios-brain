---
source: personal Mac ~/Downloads/tamil_harvest/design/overnight_progress.txt
---

Tamil harvest - Commons v3 run 5 (full run)
updated    2026-09-26 07:17:41   elapsed 9h 45m
state      finished
disk free  192.6 GB (stops below 20 GB)   added this run 1410 MB
requests   3559   downloads 2090   kept 2063   dupes 26   errors 0

surface        count/500  direct  region  techn  SHIP  STUDY   new  dupes errors
stone            500/500     492       8     50   130    432   435      8      0
palm_leaf        132/500     100      32     50    76    106   134      2      0
pottery          147/500     131      16     50    58    144   156      2      0
coins             78/500      69       9     50    40     89    87      1      0
copper_plate      65/500      47      18     50    69     47    86      1      0
rings              4/500       3       1     50    23     36    45      0      0
seals             14/500      13       1     50    30     34    59      1      0
temples          500/500     483      17     21   110    411   516      2      0
bronzes          500/500     450      50     50   351    199   545      9      0

count = direct + regional files in references/<surface>/ (automatic keyword relevance)
resume: cd ~/Downloads/tamil_harvest && nohup caffeinate -ims python3 design/fetch_commons_v3.py --supervise >> design/reference_engine/overnight_stdout.log 2>&1 &
