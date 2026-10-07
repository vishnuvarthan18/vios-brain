# Harvester Research — Full Working Notes

_Compiled 6 September 2026. These are the raw, unedited research passes behind the summary report, including every URL gathered. Sections 09-A and 09-B are the adversarial verification passes — where they conflict with sections 01–08, **the verification pass wins**._



---

# How real-world large-scale scraping/crawling systems organize and schedule work

Research compiled 2026-09-06. Every substantive claim below has a URL in the Sources section.
Dates are given for each source; staleness flags are called out inline and collected in §10.

**Framing note:** the literature and practice split into two nearly disjoint worlds that get conflated:

- **Broad-web crawling** (Common Crawl, Internet Archive, search engines) — unbounded URL discovery,
  a *frontier* is the central data structure, scheduling is per-URL and statistical.
- **Targeted multi-source harvesting** (what the questioner is building) — a bounded, known set of
  sources, each with its own access protocol; the central data structure is a *source registry*, and
  scheduling is per-source, not per-URL.

Most of the frontier/queue machinery (§2.1, §5) is designed for the first and is frequently
over-applied to the second. Conversely most orchestrator/DAG practice (§2.2) is designed for the
second and breaks when URL discovery is unbounded. This report keeps them separate.

---

## 1. Architectural patterns: who uses which, and why

### 1.1 Frontier/queue crawlers, batch flavour — Apache Nutch

Nutch runs discrete MapReduce phases in a loop: `generate` (select a fetchlist from CrawlDb) →
`fetch` → `parse` → `updatedb` → `invertlinks` → `index`. New links found in cycle *N* are only
fetchable in cycle *N+1*.

Common Crawl migrated **from a bespoke crawler to Nutch in 2014** (blog dated 2014-02-20). The stated
reason was not throughput but *heterogeneity*: their custom crawler was "highly tuned to our data
center environment" — identical machines, lots of RAM, fast local networking — and did not survive
the move to cloud instances with varying resources and unreliable infrastructure. Their pre-Nutch
crawler hit **~40,000 pages/second aggregate** in the Spring 2013 crawl. The blog explicitly names the
batch-loop cost: fetching, parsing and link-extraction happen as sequential MapReduce jobs feeding the
*next* batch, "rather than discovering and immediately crawling new pages within a single job cycle."

Common Crawl is still Nutch-based per their FAQ (undated, current as of 2026): "Nutch-based web
crawler that makes use of the Apache Hadoop project."

Concrete current scale — **August 2025 crawl (CC-MAIN-2025-34)**: 2.44 billion pages, 424 TiB
uncompressed, 47.5M hosts, 38.5M registered domains, crawled **2–16 August** (a ~2-week window, then
the archive is published; crawls are monthly). **675 million URLs were new**, i.e. roughly 72% of the
crawl was revisits of URLs seen in earlier crawls. That new/revisit ratio is itself a scheduling
decision.

### 1.2 Frontier/queue crawlers, streaming flavour — StormCrawler, Heritrix

**StormCrawler** (now Apache StormCrawler) runs the same logical stages as continuously-running Storm
bolts, with a status/index store (Elasticsearch, OpenSearch, SQL) replacing the CrawlDb. Julien Nioche
— who is *both* a Nutch committer and StormCrawler's creator, and discloses this — benchmarked the two
on one server over 1,000 websites (2017-01-03):

| | URLs/min | duration | pages fetched |
|---|---|---|---|
| Nutch | 6,038 | 32 h | 10.6 M |
| StormCrawler | 9,792 | 66 h | 32.6 M |

~60% higher throughput for StormCrawler. His characterisation of the trade-off: "Nutch achieves
greater spikes in the fetching step but does not fetch continuously as StormCrawler does" — batch
crawlers produce bursty bandwidth, streaming crawlers a flat line. Caveats he states himself: the
benchmark **omitted Nutch's indexing step** because of reliability problems (which would have made
Nutch look worse), and Nutch at the time had features StormCrawler lacked, notably document
deduplication. **Flag: 2017 benchmark, both projects have moved several major versions since.**

**Heritrix** (Internet Archive) is the archival-quality end: WARC output, one connection per server at
a time, per-hostname queues. Its politeness model is `delay-factor` × the last request's completion
time, clamped by `min-delay-ms`/`max-delay-ms`/`min-interval-ms` — i.e. adaptive to the *server's*
observed latency rather than a fixed sleep. A worked example in the wiki: `min-interval-ms=2000`,
`max-delay-ms=30000`, `delay-factor=1`.

### 1.3 Distributed frontier as a separate service — Frontera, Scrapy Cluster

**Frontera** (Scrapinghub/Zyte) separates the frontier from the fetchers entirely: spiders ↔ Kafka
"spider log" ↔ strategy workers (scoring) ↔ DB workers (metadata + batch generation) ↔ HBase.
Key design choice: **Kafka topics partitioned by domain name**, so "each domain was downloaded at most
by one spider" — politeness is enforced by *partitioning*, not by a lock. Numbers from the 2015-08-06
post: one spider ≈ **1,200 pages/min across 100 hosts in parallel**; **15,000 pages/min needs 12
spiders + 3 strategy/DB worker pairs ≈ 18 cores**; strategy workers need **1 GB each** for state
cache; original design target **~1 billion pages/week**. Docs cite testing at **45M documents /
100K websites**. Stated advantage: "the crawling strategy can be changed without having to stop the
crawl."

**Scrapy Cluster** (IST Research) is the Kafka+Redis equivalent, notable for one feature the others
lack: "coordinated throttling of crawls from independent spiders on separate machines, but behind the
same IP address" — i.e. politeness keyed on the *egress IP*, not just the target host.

**Staleness flag — both are effectively dormant.** Frontera's last release is **v0.8.1, 2019-04-05**;
Scrapy Cluster's last stable release is **1.2.1, 2018-01-23**. No formal deprecation notice on either,
but neither should be adopted as a live dependency in 2026 without an audit.

### 1.4 DAG orchestrators — Airflow, Dagster, Prefect

This is what most non-search-engine data teams actually run. The pattern is one DAG (or one
Dagster *asset*) per source, scheduled on cron, with the frontier reduced to "list pages 1..N".

Scale evidence and failure modes, from **Shopify Engineering, 2022-05-23**: **>10,000 DAGs**, **>400
tasks running at any moment**, **>150,000 task runs/day**. Their enumerated problems are all
scheduler-side, not scraper-side:
- GCSFuse for DAG files did not scale → replaced with an in-cluster NFS server synced from GCS.
- Metadata DB growth → **28-day** retention DAG pruning DagRun/TaskInstance/Log tables.
- No ownership tracking → a **YAML manifest** mapping DAG namespace → owner + repo.
- Resource contention → Celery queues, priority weights, pools (pool config synced from a K8s ConfigMap).
- **Thundering herd**: fixed schedules made everything fire at once → **deterministic randomized
  scheduling seeded on the DAG ID** to smear start times. (This is directly relevant: N sources all
  on `0 * * * *` is a self-inflicted DDoS on your own workers.)
- Cluster policies to enforce which queue/pool/namespace a DAG may use.

The HN thread on that post (2022-05, id=31480320) is the counterweight, and is worth reading for the
dissent: one operator at ~5,000 DAGs / 100k+ daily executions says "We've had SO many headaches
operating airflow over the years, and each time we invest in fixing the issue I feel more and more
entrenched"; another reports **5–15 minutes of scheduling overhead per task** before tuning; several
report the UI being "completely useless" beyond ~1,000 tasks in a DAG; and the crystallising comment:
"we have a pretty simplified DAG structure, I wish we had gone with a simpler, more robust/scalable
solution (even if just rolling our own scheduler) for our specific needs." Alternatives named in that
thread: Prefect 2.x, Dagster, Temporal, Argo Workflows, Luigi, AWS Step Functions.

### 1.5 Event-driven task queues — Celery/RabbitMQ, Kafka

The lowest-ceremony option that still gives retries and observability. ScrapeOps' playbook (undated)
describes the canonical shape: Celery Beat for schedules (rather than system cron, "lacks built-in
retries, logging, and monitoring… fails silently"), named queues (`-Q scraping`) for routing, Flower
for per-task visibility and manual revoke/replay, **tasks written to be idempotent**, and explicit
dispatch throttling — "throttling task dispatch over time rather than launching all jobs
simultaneously." Their worked example: **50,000+ scrapes/day, 10+ workers on separate servers**.

### 1.6 Durable workflow engines — Temporal

**Grepsr** (web-data vendor) published a migration case study with Temporal on **2025-07-24**. Their
prior architecture was exactly the thing that grows organically here: "a combination of cron jobs,
bespoke scripts, and message queues." The three failures they name are the ones that bite unattended
multi-month systems:
1. failures untraceable across logs,
2. **workflow state lost on crash → full restart**,
3. concurrent job execution without orchestration → inconsistency.

Scale: **>600 million records/day from >10,000 sources**. Claimed results: 99% delivery reliability,
60% reduction in incident resolution time. The article does *not* document how workflows map onto
crawl phases or what retry policies they use — treat the numbers as vendor-published.

Temporal's own docs also cover the long-running-workflow caveat (event history growth →
`continue-as-new`), which matters for a "runs for months" requirement.

### 1.7 Summary of the architectural trade-off

| | Frontier crawler | DAG orchestrator | Task queue | Durable workflow |
|---|---|---|---|---|
| Best when | URL set unbounded, discovery-driven | source set known, per-source batch | high fan-out, heterogeneous work | long-lived stateful multi-step jobs |
| Politeness | first-class (per-host queues) | must be bolted on | must be bolted on | must be bolted on |
| Backfill/replay | re-generate from CrawlDb | first-class (DAG runs / asset partitions) | manual | first-class (event history) |
| Cost at N sources | high fixed cost, low marginal | linear-ish, scheduler pressure at 10^4 | low | medium |
| Failure mode | frontier corruption / starvation | scheduler & metadata DB | silent queue backlog | history size, worker/version skew |

---

## 2. Per-source configuration: "scraper as data", and where it breaks

### 2.1 What the config-driven end looks like

- **Nutch / StormCrawler**: the crawl *is* a config file. Every knob in §5 (`fetcher.server.delay`,
  `partition.url.mode`, `db.fetch.interval.default`, `generate.max.count`) is XML/YAML. Nutch even
  allows **per-host overrides via JEXL expressions against a HostDB** —
  `generate.max.count.expr` and `generate.fetch.delay.expr` (NUTCH-2368) — which is the cleanest
  real-world example of "politeness policy as data, keyed per source."
- **Archive-It**: an entire production web-archiving service where a source is a *seed* plus a
  frequency plus limits, chosen from a fixed enum (§3.1). No code at all.
- **data.gov harvester**: each source is a registered endpoint + format + schedule; records are
  validated against **DCAT-US** and failures logged and skipped, not crashed on.
- **Dagster Components** (docs, current): explicitly motivated by "multiple similar workflows in data
  pipelines." A source becomes a YAML block:
  ```yaml
  etl_job:
    - bucket: my_bucket
      source_object: raw_transactions.csv
      target_object: cleaned_transactions.csv
      sql: SELECT * FROM source WHERE amount IS NOT NULL;
  ```
  Dagster's own docs state the boundary honestly: YAML suffices for homogeneous jobs sharing an
  identical structure; "custom business logic, complex dependencies, or non-standard processing steps
  that violate the factory's assumptions" require going back to Python.
- **pupa / Open States** (the closest existing analogue to a many-government-sources harvester):
  the framework fixes the *shape* — a `Scraper` base class, a `save_object()` that runs
  `pre_save()` → `as_dict()` → `validate()` against a schema, UUID identity, reusable mixins
  (`SourceMixin`, `LinkMixin`, `ContactDetailMixin`) — while the per-jurisdiction `scrape()` stays
  hand-written Python. Bill scrapers are further split into `get_bill_ids()` then `get_bill()`, with a
  `ContinueScraping` exception to skip — a two-phase discovery/extraction split baked into the base
  class. Note what pupa did *not* do: it did not try to make the extraction itself declarative.

### 2.2 Where declarative breaks down — the evidence

**Zyte, 2018-07-02**, on a real price-intelligence programme, is the single most useful data point:

- Zyte overall: **>8 billion pages/month**, 3 billion of them product pages.
- One customer project: **~4,000 spiders targeting ~1,000 e-commerce sites**.
- **20–30 spiders fail per day** on that project.
- Staffed by **18 full-time crawl engineers + 3 dedicated QA engineers**.
- Their recommended pattern is *one* extraction spider handling all layout rules rather than a spider
  per layout — and they concede such spiders end up **"thousands of lines long"** to cover edge cases.
- Explicit two-phase design: "separate your product discovery spiders from your product extraction
  spiders," with **one extraction spider per ~100,000-page bucket**.
- Serial (non-parallel) scrapers top out around **<40,000 requests/day** before you must shard.
- Breakage detection is *statistical, not exception-based*: schema/type validation, cross-region
  variance checks, **volume-based anomaly detection on record counts**, plus active structural checks
  against the target site. (Zyte open-sourced this as Spidermon.)

That is the empirical answer to "can each source be a config?": at ~1,000 sites, a 4:1 spider-to-site
ratio and 21 engineers, and the thing that keeps it alive is not the config format but the monitoring.

**The cautionary tale**: Portia, Scrapinghub's visual point-and-click scraper — the maximal
"scraper as data" product, announced ~2014 — **was archived read-only on 2026-06-24**. Its README
still carries no deprecation notice, which is itself a lesson about maintaining generated-scraper
tooling.

**Where the split usually lands in practice**, synthesising the above: *acquisition* is highly
amenable to config (URL patterns, auth, rate limits, pagination style, schedule, expected volume);
*extraction* resists it and drifts back to code; *validation* has to be declarative or you have no way
to detect breakage at all.

---

## 3. Scheduling

### 3.1 Fixed schedules — what people actually set

Real defaults and menus, not opinions:

- **Archive-It** offers exactly nine frequencies: twice daily (12 h crawl time limit), daily (24 h),
  weekly (3 d), monthly (3 d), bimonthly (3 d), quarterly (3 d), semiannual (5 d), annual (5 d),
  one-time (3 d). Guidance: run a test crawl before scheduling twice-daily/daily because they
  "consume substantial data budgets quickly"; a crawl stopped by its time limit can be **resumed
  within seven days**.
- **data.gov harvester**: "Most harvest sources run automatically on a set schedule – typically daily
  or weekly, depending on how the source was configured," with manual runs on admin request.
- **StormCrawler `DefaultScheduler` defaults**: `fetchInterval.default` **1440 min (24 h)**;
  `fetchInterval.error` **-1 (never refetch)**; `fetchInterval.fetch.error` **120 min**;
  `max.fetch.errors` **3** consecutive failures before a URL is marked ERROR.
- **Nutch defaults**: `db.fetch.interval.default` **2,592,000 s (30 days)**; `db.fetch.interval.max`
  **7,776,000 s (90 days)** — "after this period every page in the db will be re-tried, no matter what
  is its status." That max-interval floor is a good pattern in itself: an absolute staleness ceiling
  independent of the adaptive logic.
- **Common Crawl**: monthly crawl archives, each crawled over roughly a 2-week window.

Note the *shape* of these: a small enum of frequencies, not per-source arbitrary cron. Archive-It and
data.gov both chose an enum. Shopify then had to add randomised jitter *within* the schedule (§1.4) —
so the pattern that survives at scale is "coarse enum + per-source deterministic jitter."

### 3.2 Adaptive / change-rate-learned scheduling — the classic result

**Cho & Garcia-Molina, "Effective page refresh policies for Web crawlers," ACM TODS 28(4), 2003**
(experiments run 1999). This is the canonical reference and its headline result is *counterintuitive
and still widely mis-stated*:

- Metrics: **freshness** (fraction of the local copy that is up to date) and **age** (average staleness).
- Three allocation policies compared: **uniform** (same rate for every page), **proportional** (rate ∝
  observed change frequency), **optimal** (derived).
- **Finding: the uniform policy beats the proportional policy under any change-rate distribution.**
  Their explanation: under a fixed resource budget, "it is better to focus on what we can track" — a
  page that changes every visit can never be kept fresh, so spending budget on it is wasted.
- The true optimal is **non-monotonic**: revisit rate rises with change rate up to a point, then falls
  to *zero* for the fastest-changing pages.
- Reported gain: the optimal policy achieved **~500% freshness improvement over proportional** in
  their real-world scenario.

Their empirical measurements — **720,000 pages across 270 sites over 4 months** — are directly
relevant to a government/scientific corpus:

- **>20% of pages changed on every visit**
- **~40% of .com pages changed daily**
- **>50% of .edu and .gov pages did not change at all in the entire 4 months**

That last number is the single most actionable finding for this project: for `.gov`/`.edu` sources,
naive fixed-interval polling is dominated by wasted requests, and conditional requests (§6.3) or
change-feeds (§3.4) pay for themselves immediately.

**Caveat/staleness:** the measurement is from 1999. The .gov web in 2026 is far more
dynamic-application-driven; the 50%-static figure should be re-measured, not assumed. The *policy*
result (uniform > proportional) is a mathematical result under a Poisson change model and does not
decay, but its assumptions (fixed budget, uniform page importance, Poisson independence) do not hold
when sources have hard rate limits and wildly different importance.

### 3.3 Modern equivalent

**Azar, Horvitz, Lubetzky, Peres & Shahaf, "Tractable near-optimal policies for crawling," PNAS,
2018.** Directly extends Cho & Garcia-Molina from uniform page importance to arbitrary per-page
utility µᵢ and change rate Δᵢ. Cho & Garcia-Molina had concluded explicit formulas were unattainable
in general and solved numerically for small n; this paper gives **O(n log n)** algorithms for the
optimal randomized policy, then derandomizes it into a deterministic "carousel" schedule via
earliest-deadline-first. Empirically, on 1,000-page synthetic sets the derandomized policy attained
**99% of the numerically-computed optimum**, well above utility-proportional and random baselines.
This is the algorithm ("LambdaCrawl") to cite if you want per-source importance weights in the
scheduler.

### 3.4 Adaptive scheduling as actually implemented

**Nutch `AdaptiveFetchSchedule`** — real parameter names and shipped defaults from
`conf/nutch-default.xml` (master branch, verified 2026-09-06):

| parameter | default | meaning |
|---|---|---|
| `db.fetch.schedule.class` | `DefaultFetchSchedule` | adaptive is **opt-in**, not the default |
| `db.fetch.schedule.adaptive.inc_rate` | **0.4** | interval × (1+0.4) when page unmodified |
| `db.fetch.schedule.adaptive.dec_rate` | **0.2** | interval × (1−0.2) when page modified |
| `db.fetch.schedule.adaptive.min_interval` | **60.0 s** | floor |
| `db.fetch.schedule.adaptive.max_interval` | **31,536,000 s (365 d)** | ceiling, but capped by `db.fetch.interval.max` (90 d) |
| `db.fetch.schedule.adaptive.sync_delta` | **true** | shift next fetch toward the observed change time |
| `db.fetch.schedule.adaptive.sync_delta_rate` | **0.3** | fraction of (lastModified − lastFetch) to shift by |

The config comments explicitly warn that inc_rate, dec_rate and sync_delta_rate "should not exceed 0.5,
otherwise the algorithm becomes unstable." Note the asymmetry: it backs off slower than it speeds up
(0.4 up vs 0.2 down), i.e. biased toward freshness.

**And it has a known, long-standing bug.** **NUTCH-1564** (reported by Sebastian Nagel, a Common Crawl
/ Nutch committer; still unresolved as of the ticket): `sync_delta` computes a reference time by
subtracting `delta × SYNC_DELTA_RATE` from the fetch time; when a document has been unchanged for
longer than the max interval, that reference lands far in the past and adding the capped interval
produces a **next-fetch time already in the past** — forcing immediate refetch every cycle. Reported
real-world consequence: in a daily continuous crawl, documents unchanged for 30 days were refetched
**every cycle** despite a configured 7-day maximum interval. Proposed fix: clamp next-fetch into
`[currentFetchTime + MIN_INTERVAL, currentFetchTime + MAX_INTERVAL]`.

That is the honest state of adaptive scheduling in the most widely deployed open-source crawler: it
exists, it is off by default, and its cleverest sub-feature has an open correctness bug. **If you
implement adaptive intervals, clamp the output and add an absolute staleness ceiling.**

**StormCrawler** ships an `AdaptiveScheduler` as an alternative to `DefaultScheduler`, using content
signatures to detect change; the default remains the fixed 1440-minute interval.

**Google** (developer docs, current) describes the same idea in prose without numbers: crawl budget =
**crawl capacity limit** (server health / response time driven, adjusted automatically) × **crawl
demand** (perceived inventory, popularity, **staleness** — "our systems want to recrawl documents
frequently enough to pick up any changes"). Google names `<lastmod>` in sitemaps, **HTTP 304**, and
**If-Modified-Since** as the supported change signals, and pointedly publishes **no concrete recrawl
frequencies**.

**Common Crawl's CCBot** implements the *reactive* half rather than the predictive half: default wait
"a few seconds" between requests, honours robots.txt `Crawl-delay`, and runs "an adaptive back-off
algorithm that slows down requests to your website if your web server is responding with HTTP 429 or
5xx." Adaptivity applied to *politeness*, not to *refresh interval*.

### 3.5 Push instead of poll

The genuinely different scheduling model: let the source tell you.

- **IndexNow** (Bing/Yandex/Naver/Seznam; Google does **not** participate) — key file on the site,
  POST the changed URLs, engines "prioritize crawl while limiting the need for costly exploratory
  crawls." Not usable for government sources unless they adopt it, but the *shape* — a change
  notification endpoint — is what RSS/Atom and ResourceSync Change Lists provide for you.
- **ResourceSync Change Notification** (OAI/NISO) defines a push channel over the same vocabulary as
  the pull-based Change Lists (§6.2).
- For gov/sci, the practical push-equivalents are: RSS/Atom feeds, OAI-PMH `from=` polling, SEC EDGAR
  daily-index files, PubMed daily update files, and S3 inventory manifests (§6).

---

## 4. Priority, politeness, partitioning, dedup

### 4.1 Politeness: what the shipped defaults actually are

| system | setting | default |
|---|---|---|
| Nutch | `fetcher.server.delay` | **5.0 s** between successive requests to the same server |
| Nutch | `fetcher.threads.per.queue` | **1** (>1 disables host blocking *and* makes Nutch ignore robots.txt Crawl-delay) |
| Nutch | `fetcher.min.crawl.delay` | = `fetcher.server.delay` — "guarantees that a value set in the robots.txt cannot make the crawler more aggressive than the default configuration" |
| StormCrawler | `fetcher.server.delay` | **1 s** |
| StormCrawler | `fetcher.threads.per.queue` | **1**; `fetcher.threads.number` **10** total |
| Scrapy AutoThrottle | `AUTOTHROTTLE_ENABLED` | **False** (opt-in) |
| Scrapy AutoThrottle | START_DELAY / MAX_DELAY / TARGET_CONCURRENCY | **5.0 s / 60.0 s / 1.0** |
| Heritrix | model | one connection per server at a time; `delay = delay-factor × last-request-duration`, clamped |
| CCBot | model | "a few seconds", honours `Crawl-delay`, backs off on 429/5xx |

**Scrapy's AutoThrottle algorithm** is the most transferable one for a many-sources system because it
is *latency-derived*, not fixed: target delay = observed latency ÷ `AUTOTHROTTLE_TARGET_CONCURRENCY`;
the applied delay is the average of previous and target; **non-200 responses may only increase the
delay, never decrease it**; the result is clamped to `[DOWNLOAD_DELAY, AUTOTHROTTLE_MAX_DELAY]`. That
one-way ratchet on errors is the property you want for unattended multi-month operation.

Note the two structural choices embedded here: Nutch's `fetcher.min.crawl.delay` refuses to let a site
make you *faster*; Scrapy's ratchet refuses to let recovery be fast. Both are conservatism-by-default
for unattended running.

### 4.2 Partitioning / work assignment

- **StormCrawler**: `fetcher.queue.mode` and `partition.url.mode` both default to **`byHost`**;
  alternatives **`byDomain`, `byIP`**. `byIP` matters when many gov subdomains share one host —
  per-host limits then under-count your real load on the server.
- **Nutch**: `partition.url.mode` default **`byHost`**, also `byDomain`/`byIP`;
  `generate.max.count` (default **-1**, unlimited) caps URLs per host/domain in one fetchlist, with
  `generate.count.mode` default `host`.
- **Frontera**: Kafka topics partitioned by domain → **at most one spider per domain**, which makes
  politeness a consequence of routing rather than of shared state.
- **Scrapy Cluster**: throttling coordinated across machines **behind the same egress IP** — the case
  the others miss.

The generalisable rule from all four: *make the politeness key the partition key.* If two workers can
ever be assigned the same politeness key, you need distributed locking; if they can't, you don't.

### 4.3 Deduplication of the frontier

- **Crawlee `RequestQueue`**: dedup by `uniqueKey`, auto-derived from the URL and overridable (so you
  can deliberately enqueue the same URL twice under different keys). Documented consistency caveat:
  because the queue is distributed storage, `isFinished()` "may occasionally return a false negative,
  but it shall never return a false positive" — i.e. it errs toward *not* declaring completion. Also
  documented: it "is not optimized for operations that add or remove a large number of URLs in a
  batch."
- **Nutch**: dedup is implicit in the CrawlDb (URL is the key), plus a separate `dedup` job for
  *content* duplicates by signature.
- **Bloom filters / DRUM**: the classic disk-backed "URL-seen" structure from **IRLbot** (Lee, Leonard,
  Wang, Loguinov — "IRLbot: Scaling to 6 Billion Pages and Beyond"), still the reference design for
  frontier dedup that exceeds RAM; there is a maintained Go implementation.

For a bounded-source harvester, note that URL-level frontier dedup is largely a non-problem — the
interesting dedup is *record-level* (same document reached via two sources) and *content-level* (has
this document actually changed since last time), which is signature/hash-based, not set-membership.

### 4.4 Priority

Least-standardised area. What exists: Frontera scores documents in strategy workers and DB workers
generate batches in score order; Airflow has `priority_weight` and pools (Shopify used both);
Celery/RabbitMQ support priority queues but practitioners mostly use *separate named queues* per
class of work instead. Nobody publishes a principled priority function; in practice priority is
"tier the sources by hand" plus "errors go to a slower queue."

---

## 5. Decoupling fetch → parse → load; raw-archive-first; idempotency

### 5.1 The pattern

**Common Crawl is the reference implementation**: each crawl publishes **WARC** (raw HTTP request +
response bytes), **WAT** (extracted metadata) and **WET** (extracted plain text) as three separate
derived datasets from one fetch. The parse is a *derivative* of an immutable archive, so a parser bug
is fixed by re-deriving, never by re-fetching. Heritrix writes WARC for the same reason.

**Nutch** does the same internally: `segments` hold the fetched content, and `parse` is a separate job
over a segment — you can re-parse a segment without re-fetching it.

**pupa/Open States** applies it to structured government data: `scrape` writes validated JSON objects
to disk, `import` loads them into Postgres. Two commands, two failure domains; you can re-import
without re-scraping.

The practitioner-level argument is made directly in "Save First, Parse Later" (Fabrício Barbacena,
2021-12-16). Benefits he lists: fewer requests during parser development/debugging, offline work,
historical snapshots, lower ban risk, and — the underrated one — **the "save" phase generalises across
projects while parsing logic does not**. Costs he concedes: useless for JS-rendered pages unless you
capture post-render DOM; a two-pass loop; storage overhead ("storage is cheaper nowadays and HTML
files are usually small").

### 5.2 Idempotency and replay

- **Celery/task-queue guidance**: tasks "must be structured as idempotent operations — rerunnable
  without side effects," because at-least-once delivery means every task *will* run twice eventually.
- **Temporal**: replay is the mechanism, not a feature — workflow state is reconstructed from event
  history, which is precisely what Grepsr cited as the fix for "workflow state was lost during crashes
  requiring full restarts."
- **Airflow/Dagster**: idempotency comes from partitioning — a DAG run or asset partition is keyed by
  a logical date/interval, so re-running produces the same output. This is the reason to make each
  harvest window an explicit partition key rather than "whatever was new when the job ran."
- **OAI-PMH** builds idempotency into the protocol: reissuing a `resumptionToken` after a network
  error is specified to give a consistent result — "idempotency of the most recent incomplete list
  request."

### 5.3 The natural layering for a gov/sci harvester

Synthesising §5.1–5.2 (this is a pattern description, not a recommendation):

1. **Acquire** — write the raw bytes plus full response metadata (status, headers, ETag,
   Last-Modified, fetch timestamp, source config version) to immutable object storage, keyed by
   (source, logical partition, content hash).
2. **Parse** — pure function from raw blob → records, versioned, re-runnable over the archive.
3. **Load** — upsert by stable identifier.

Every stage boundary is a replay point. The cost is storage and one extra hop; the benefit is that a
parser regression discovered in month 5 does not require re-hitting a rate-limited government API.

---

## 6. Government & scientific sources: bulk-first, and the incremental protocols

### 6.1 The bulk-over-HTML rule is stated explicitly by the sources themselves

- **NCBI**, unambiguously: **"NCBI does not allow scripting against our web pages. If you script, we
  may restrict your access."** Sanctioned paths instead: E-utilities (**3 req/s default; 3–10 req/s
  with an API key; above 10 only by Help Desk request with project details**), specialised APIs
  (PubChem PUG, NCBI Datasets), **"Most NCBI data are available from the FTP Site"**, and cloud
  (SRA, PMC on AWS). Also: third-party apps should let users supply their **own** API key rather than
  share the developer's quota.
- **SEC EDGAR**: **"no more than 10 requests per second, regardless of the number of machines used to
  submit requests"** — note the explicit anti-distribution clause — plus a declared User-Agent
  requirement, and "The SEC does not allow 'unclassified' bots or automated tools to crawl the site."
  Sanctioned bulk paths: `/edgar/daily-index/` (HTML/XML/JSON), `/edgar/full-index/` (current quarter
  through previous business day), and full daily archives as tar/gzip at `/edgar/Feed/` and
  `/edgar/Oldloads/`.
- **api.data.gov** (the shared gateway in front of many federal APIs): **1,000 requests/hour** per key,
  **rolling** hourly reset, HTTP **429** on exceed, `X-RateLimit-Limit` / `X-RateLimit-Remaining`
  headers; `DEMO_KEY` is **30/IP/hour, 50/IP/day**. Higher limits are per-agency, by request.
- **Regulations.gov v4**: commenting (POST) capped at **50 req/min, 500 req/hour**; GET limits inherit
  from api.data.gov.

**Practical consequence:** for a multi-source gov/sci harvester the binding constraint is almost never
your own throughput — it is a published per-source quota that *cannot be worked around by adding
workers* (SEC says so in the rule text). This inverts the usual scaling design: you need a
per-source token bucket enforced globally, and horizontal scaling buys you nothing on the hot sources.

### 6.2 Incremental harvesting protocols

**OAI-PMH** (v2.0, spec 2002; harvester guidelines page current). The mechanics that matter:
- **Resumption tokens** carry list state; an empty token ends the sequence; **`badResumptionToken`
  means the token expired and you must restart the entire list request** — so a long harvest is not
  arbitrarily resumable and you need checkpointing at a coarser granularity.
- **Incremental harvesting** uses `from`/`until` datestamps, and the guidelines say to **overlap**
  successive harvests by one datestamp increment, basing the next `from` on the **`responseDate` of the
  first partial-list response**, with extra overlap for second-granularity repositories. (I.e. the
  protocol assumes you will double-fetch a boundary window; your loader must be idempotent.)
- **Flow control**: repositories return **503 with `Retry-After`**; the guidelines say if
  `Retry-After` is absent, harvesters "should not automatically retry without considerable delay
  (minutes) or, preferably, manual intervention," and **"must not be written to retry indefinitely."**

**ResourceSync** (OpenArchives/NISO, v1.1; XML schema updated 2023-07-19) is the Sitemaps-based
successor: Capability List → Resource List (baseline inventory) → Change List (incremental) →
Resource Dump / Change Dump (packaged bulk), with `rs:md` carrying change type (created/updated/
deleted), modification time, hash, and length.

**Empirical comparison** (Edinburgh/repofringe presentation, 2018) — the numbers are dramatic and
worth flagging as the strongest argument for preferring ResourceSync where offered:

- OAI-PMH throughput ranged from **2 records/s (Fedora) to 225 records/s (EPrints)** — i.e. an order
  of magnitude of variance driven purely by repository software.
- Oxford: OAI-PMH **572.58 s** vs ResourceSync **1.64 s** — a **~349×** difference.
- Requests per resource: ResourceSync **~2.4–2.8**; OAI-PMH **20 to 572+**.
- Their conclusion: the "15 year old" OAI-PMH "is not scalable for large quantities of resources" and
  "suffers from inconsistent implementations."

**Counterpoint, for balance:** OAI-PMH is *vastly* more widely deployed in practice than ResourceSync,
and it harvests **metadata**, whereas ResourceSync synchronises **resources** — the comparison is not
entirely like-for-like. CORE's engineering blog (2018-03-17) describes a middle path: getting
repositories to expose **on-demand Resource Dumps** to speed up harvesting that otherwise runs over
OAI-PMH. The disagreement is real and unresolved: the standards community prefers ResourceSync, the
installed base is OAI-PMH.

### 6.3 Bulk-file patterns with baseline + increments

- **PubMed**: an **annual baseline** (complete XML snapshot; NLM's guidance is to *overwrite* your
  local copy each year) plus **daily update files** containing new, revised and deleted citations.
  The stated procedure is "load the baseline files first, then load the daily update files in
  numerical order"; multiple update files may be released on the same day, and revised/deleted
  citations must replace local records. This is the cleanest example in existence of the
  baseline+CDC-log pattern for a government source — and the "annual full reload" step is a deliberate
  drift-correction mechanism, not laziness.
- **PMC**: **major 2026 change — flag as breaking.** PMC eliminated "bulk baseline and incremental
  files that contained XML or TXT for millions of articles"; **all legacy PMC Article Dataset files on
  FTP and Cloud Services were removed the week of 2026-08-24**. The replacement is the
  **`pmc-oa-opendata`** S3 bucket (us-east-1, world-readable, `--no-sign-request`), where individual
  files carry **timestamps and ETags** that change with content, and change detection is done against
  a **daily S3 inventory snapshot** that **lags the bucket by about a day**. `oa_comm`/`oa_noncomm`
  directory separation is gone; license and article type now live in per-article JSON metadata.
  Any code or tutorial written before mid-2026 for PMC bulk access is broken.
- **SEC EDGAR**: full-index/daily-index files are the "what changed" log; the tar/gz daily feeds are
  the bulk archive. Same shape.
- **data.gov**: the harvester "reads each dataset record and checks whether it is new, updated, or
  unchanged since the last harvest," validates against DCAT-US, and **logs and skips** invalid records
  rather than failing the harvest. (The public docs do not specify the identifier/dedup key.)

### 6.4 Cursor/keyset pagination as the incremental mechanism

**Regulations.gov v4** documents the pattern explicitly and it generalises to many gov APIs with
result caps: results are capped (their example: **>5,000 items**), so you **sort by `lastModifiedDate`**,
paginate (`page[size]=250`), and when you hit the pagination ceiling you re-query with
`filter[lastModifiedDate][ge]` = the last document's timestamp and paginate again, repeating until
exhausted. GSA flags `lastModifiedDate` as **in beta** and says a permanent bulk-download solution is
planned. This is keyset pagination doubling as a resumable incremental cursor — and note the same
boundary-overlap/idempotency requirement as OAI-PMH `from=`.

### 6.5 Conditional requests

Google's crawling docs confirm Googlebot supports **HTTP 304** and **If-Modified-Since**, and
recommends sitemap **`<lastmod>`** for sites with frequently updated content. Given Cho &
Garcia-Molina's finding that **>50% of .edu/.gov pages were unchanged over four months** (§3.2), the
combination of `If-Modified-Since`/`If-None-Match` + `lastmod` is the cheapest available freshness
mechanism for exactly this corpus — you get change detection at the cost of a 304. I did not find a
rigorous published study quantifying the bandwidth savings; treat the magnitude as unmeasured.

---

## 7. Where practitioners disagree (both sides, unresolved)

**7.1 Uniform vs proportional refresh.**
*Cho & Garcia-Molina (TODS 2003)*: uniform beats proportional under **any** change-rate distribution;
optimal is non-monotonic and gives *zero* budget to the fastest-changing pages.
*Practice*: essentially every shipped adaptive scheduler (Nutch `AdaptiveFetchSchedule`, StormCrawler
`AdaptiveScheduler`) is **monotonically proportional-ish** — faster refresh for pages that change.
Nobody I found implements the non-monotonic optimum. Whether that is pragmatism (importance is not
uniform, so the theorem's premise fails) or two decades of ignoring a result is not settled anywhere I
could find.

**7.2 Batch vs streaming crawl loop.**
*Nioche (2017, with disclosed dual interest)*: streaming wins on throughput (~60%) and smoothness.
*Common Crawl (2014, still Nutch in 2026)*: batch MapReduce, chosen for robustness on heterogeneous
unreliable cloud hardware, and never reverted. Both positions are held by people who ship at scale.

**7.3 Orchestrator vs "just cron + a queue".**
*Pro-framework* (HN 43439939): "Airflow is python with cron, and the option for very sophisticated…
orchestration tools, like retries, dependencies, etc. All the stuff you'll end up rolling yourself."
*Pro-minimal* (same thread): "Pure python with cron or periodic tasks… works great. Celery task for
parallelization… you can actually get really far without needing a proper orchestration layer"; one
commenter's hand-rolled scheduler "worked 24/7 for 3 years" after "half a day" of development; another
inherited an Airflow install that was "just overly complex for what's really a pretty simple data
flow."
*Pro-framework, from the scarred* (HN 31480320, Shopify thread): even people running 5,000–10,000 DAGs
say they wish they had rolled their own for their specific, uniform workload.
The disagreement is genuinely about workload *uniformity*, not about tool quality: everyone agrees
frameworks pay off for heterogeneous DAGs and cost more than they return for N copies of one shape.

**7.4 Cron vs Celery Beat vs orchestrator schedules.**
*ScrapeOps*: system cron is disqualifying at scale — "lacks built-in retries, logging, and
monitoring… fails silently."
*HN 21216993 / 43439939 practitioners*: cron plus a supervisor plus alerting is fine and has fewer
moving parts to fail unattended.

**7.5 Declarative spider specs vs code.**
*Dagster docs / Archive-It / data.gov*: config-as-source works and is how production harvesting
services are built.
*Zyte (2018)*: at ~1,000 sites you end up with ~4,000 spiders, "thousands of lines long," 20–30
failures/day, and 21 people. Portia — the maximal declarative product — was **archived 2026-06-24**.
Both are true simultaneously; they differ on whether *extraction* can be data.

**7.6 OAI-PMH vs ResourceSync.**
*Edinburgh 2018 benchmark*: ResourceSync is up to ~349× faster with ~2.4–2.8 requests/resource vs
OAI-PMH's 20–572+; OAI-PMH "is not scalable."
*Installed base*: OAI-PMH is what almost every repository actually exposes, and it harvests metadata
rather than resources. CORE's answer was to add on-demand Resource Dumps *alongside* OAI-PMH rather
than replace it.

**7.7 Save-raw-first vs parse-inline.**
*Barbacena (2021), Common Crawl (WARC/WAT/WET), Heritrix, pupa*: separate the phases; storage is
cheap; re-parse beats re-fetch.
*Counterpoint acknowledged in the same article*: two passes, and it does not solve JS-rendered pages
unless you capture post-render DOM. Also unaddressed anywhere I found: legal/retention exposure of
keeping a full raw archive of third-party content.

---

## 8. Concrete numbers, collected

| Metric | Value | Source |
|---|---|---|
| Common Crawl, Aug 2025 crawl | 2.44B pages, 424 TiB, 47.5M hosts, 38.5M domains, 675M new URLs, crawled Aug 2–16 | commoncrawl.org |
| Common Crawl pre-Nutch peak | ~40,000 pages/sec aggregate (Spring 2013) | commoncrawl.org, 2014 |
| Nutch vs StormCrawler (1 server, 1,000 sites) | 6,038 vs 9,792 URLs/min | digitalpebble, 2017 |
| Frontera spider throughput | ~1,200 pages/min per spider, 100 hosts parallel | Zyte, 2015 |
| Frontera to reach 15,000 pages/min | 12 spiders + 3 SW/DBW pairs ≈ 18 cores; 1 GB per strategy worker | Zyte, 2015 |
| Frontera design target / tested scale | ~1B pages/week; tested at 45M docs / 100K sites | Zyte 2015 / docs |
| Zyte total | >8B pages/month (3B product pages) | Zyte, 2018 |
| Zyte one large project | ~4,000 spiders / ~1,000 sites; 20–30 spider failures/day; 18 crawl engineers + 3 QA | Zyte, 2018 |
| Serial scraper ceiling | <40,000 requests/day | Zyte, 2018 |
| Extraction spider sharding | one per ~100,000-page bucket | Zyte, 2018 |
| Shopify Airflow | >10,000 DAGs, >400 concurrent tasks, >150,000 runs/day, 28-day metadata retention | Shopify, 2022 |
| Airflow pain point reported | 5–15 min scheduling overhead per task (pre-tuning) | HN 31480320, 2022 |
| Grepsr / Temporal | >600M records/day from >10,000 sources | Temporal, 2025 |
| Celery/RabbitMQ worked example | 50,000+ scrapes/day, 10+ workers | ScrapeOps |
| Nutch default refetch | 30 days; hard max 90 days | nutch-default.xml |
| Nutch adaptive | inc 0.4 / dec 0.2 / min 60 s / max 365 d / sync_delta_rate 0.3 | nutch-default.xml |
| StormCrawler default refetch | 1440 min; fetch-error retry 120 min; 3 errors → ERROR | stormcrawler docs |
| Nutch / StormCrawler per-server delay | 5.0 s / 1.0 s; 1 thread per queue both | configs |
| Scrapy AutoThrottle | start 5.0 s, max 60.0 s, target concurrency 1.0, disabled by default | Scrapy docs |
| SEC EDGAR | 10 req/s total across all machines | sec.gov |
| NCBI E-utilities | 3 req/s; 3–10 with key; >10 by request | NLM |
| api.data.gov | 1,000 req/hour (DEMO_KEY 30/IP/hr, 50/IP/day) | api.data.gov |
| Regulations.gov POST | 50 req/min, 500 req/hr; ~5,000-result cap → keyset by lastModifiedDate | GSA |
| Archive-It frequencies | 9 presets, twice-daily → annual; 12 h–5 d crawl time limits; resume within 7 days | Archive-It |
| Cho & Garcia-Molina corpus | 720,000 pages / 270 sites / 4 months (1999) | TODS 2003 |
| .gov/.edu change rate | >50% unchanged over 4 months; >20% of all pages changed every visit; ~40% of .com daily | TODS 2003 |
| Optimal vs proportional freshness | ~500% improvement | TODS 2003 |
| LambdaCrawl (PNAS 2018) | O(n log n); derandomized policy hits 99% of optimum on n=1,000 | PNAS 2018 |
| OAI-PMH vs ResourceSync | 572.58 s vs 1.64 s (Oxford); 20–572+ vs 2.4–2.8 requests/resource; 2–225 rec/s by platform | Edinburgh, 2018 |

---

## 9. Things I looked for and did **not** find

- Any published, rigorous measurement of bandwidth/quota savings from `If-Modified-Since`/ETag on a
  real crawl (only SEO-blog assertions).
- Any production system implementing Cho & Garcia-Molina's **non-monotonic** optimal policy, or
  LambdaCrawl, outside the papers.
- A published, principled priority function for multi-source harvesting (everything found is
  hand-tiered).
- Common Crawl's current seed-selection algorithm in detail (the FAQ says sitemaps are used and that
  they crawl "a randomly selected subset" of a site, but the selection logic is not documented).
- Current-decade measurements of `.gov` page change rates.
- A first-party engineering blog from Bright Data on their internal scheduling architecture.

---

## 10. Staleness flags

- **Frontera** — last release **v0.8.1, 2019-04-05**; 78 open issues; dormant. No deprecation notice.
- **Scrapy Cluster** — last stable **1.2.1, 2018-01-23**; dormant.
- **Portia** — **archived read-only 2026-06-24**; README still carries no notice.
- **Nutch vs StormCrawler benchmark** — **2017**; both projects several majors on; StormCrawler has
  since become an Apache project (`stormcrawler.apache.org`).
- **Common Crawl → Nutch post** — **2014-02-20**; still directionally accurate (FAQ confirms Nutch)
  but the described tooling is 12 years old.
- **Cho & Garcia-Molina empirical figures** — measured **1999**; policy result durable, web
  measurements are not.
- **PMC bulk access** — **changed August 2026**; legacy FTP/cloud dataset files removed week of
  **2026-08-24**; anything written before mid-2026 about PMC OA bulk packages is wrong.
- **Regulations.gov `lastModifiedDate`** — GSA labels it **beta**; a "permanent bulk download
  solution" is announced but not shipped.
- **Zyte 100-billion-pages post** — **2018-07-02**; pre-dates the current anti-bot landscape and
  Zyte's own product shifts, so the org/staffing ratios may not transfer.
- **HN threads** — 2019/2022/2025; Airflow 3.x has changed the scheduler substantially since the
  Shopify-era complaints.

---

## Sources

**Broad-web crawl operations**
- Common Crawl, "Common Crawl's Move to Nutch" (2014-02-20) — https://commoncrawl.org/blog/common-crawl-move-to-nutch
- Common Crawl, "August 2025 Crawl Archive Now Available" — https://commoncrawl.org/blog/august-2025-crawl-archive-now-available
- Common Crawl FAQ (CCBot, Nutch, sitemaps, 429/5xx back-off) — https://commoncrawl.org/faq
- Common Crawl monthly statistics — https://commoncrawl.github.io/cc-crawl-statistics/
- Common Crawl data root — https://data.commoncrawl.org/
- DigitalPebble (Julien Nioche), "The Battle of the Crawlers: Apache Nutch vs StormCrawler" (2017-01-03) — http://digitalpebble.blogspot.com/2017/01/the-battle-of-crawlers-apache-nutch-vs.html
- InfoQ, "Julien Nioche on StormCrawler" (2016-12) — https://www.infoq.com/news/2016/12/nioche-stormcrawler-web-crawler/
- Apache StormCrawler docs — https://stormcrawler.apache.org/docs/ ; configuration reference — https://stormcrawler.apache.org/docs/latest/configuration.html
- Apache Nutch `conf/nutch-default.xml` (master) — https://github.com/apache/nutch/blob/master/conf/nutch-default.xml
- NUTCH-1564, "AdaptiveFetchSchedule: sync_delta forces immediate refetch for documents not modified" — https://issues.apache.org/jira/browse/NUTCH-1564
- NUTCH-2368 (per-host JEXL expressions for maxCount/fetch delay), referenced in nutch-default.xml
- Nutch AdaptiveFetchSchedule javadoc (2.0) — https://nutch.apache.org/apidocs/apidocs-2.0/org/apache/nutch/crawl/AdaptiveFetchSchedule.html
- Heritrix3 politeness parameters wiki — https://github.com/internetarchive/heritrix3/wiki/Politeness-parameters
- Heritrix3 repo — https://github.com/internetarchive/heritrix3
- Heritrix docs, configuring crawl jobs — https://heritrix.readthedocs.io/en/latest/configuring-jobs.html
- Archive-It, "Scheduled crawls" — https://support.archive-it.org/hc/en-us/articles/208333013-Scheduled-crawls
- End of Term Web Archive (gov-specific crawl campaign) — https://github.com/end-of-term/eot2020

**Frontiers, queues, dedup, politeness**
- Zyte, "Distributed Frontera: Web crawling at scale" (2015-08-06) — https://www.zyte.com/blog/distributed-frontera-web-crawling-at-large-scale/
- Zyte, "Frontera: The brain behind the crawls" — https://www.zyte.com/blog/frontera-the-brain-behind-the-crawls/
- Zyte, "Improved Frontera: Python 3 support" — https://www.zyte.com/blog/improved-frontera-web-crawling-at-scale-with-python-3-support/
- Frontera docs overview — https://frontera.readthedocs.io/en/latest/topics/overview.html
- Frontera repo (last release 2019-04-05) — https://github.com/scrapinghub/frontera
- Scrapy Cluster repo — https://github.com/istresearch/scrapy-cluster
- Scrapy AutoThrottle docs — https://docs.scrapy.org/en/latest/topics/autothrottle.html
- Crawlee (Python) architecture overview — https://crawlee.dev/python/docs/guides/architecture-overview
- Crawlee `RequestQueue` API (uniqueKey, consistency caveats) — https://crawlee.dev/js/api/core/class/RequestQueue
- Crawlee `AutoscaledPool` — https://crawlee.dev/js/api/core/class/AutoscaledPool
- DRUM (IRLbot disk-based URL-seen structure), Go implementation + paper reference — https://github.com/moredure/drum

**Refresh-rate / scheduling theory**
- Cho & Garcia-Molina, "Effective Page Refresh Policies for Web Crawlers," ACM TODS 28(4), 2003 — https://dl.acm.org/doi/10.1145/958942.958945 (open PDF: https://www.csd.uoc.gr/~hy561/papers/integration/crawling/Effective%20Page%20Refresh%20Policies%20For%20Web%20Crawlers.pdf)
- Azar, Horvitz, Lubetzky, Peres, Shahaf, "Tractable near-optimal policies for crawling," PNAS 2018 — https://erichorvitz.com/Crawl_1801519115.full.pdf
- Google, "Risk and optimality in estimating refresh rates for web pages" (Google Research) — https://research.google.com/pubs/archive/34570.pdf
- Google, "Crawl Budget Management" (crawl capacity limit, crawl demand, 304 / If-Modified-Since / lastmod) — https://developers.google.com/crawling/docs/crawl-budget
- IndexNow (push-based change notification) — https://www.bing.com/indexnow

**Orchestration & durable execution**
- Shopify Engineering, "Lessons Learned From Running Apache Airflow at Scale" (2022-05-23) — https://shopify.engineering/lessons-learned-apache-airflow-scale
- HN discussion of the above (2022-05) — https://news.ycombinator.com/item?id=31480320
- HN, "Ask HN: What is the simplest data orchestration tool you've worked with?" (2025) — https://news.ycombinator.com/item?id=43439939
- HN, cron-in-production thread (2019) — https://news.ycombinator.com/item?id=21216993
- Temporal, "How Grepsr uses Temporal to deliver scalable and reliable web data" (2025-07-24) — https://temporal.io/blog/how-grepsr-uses-temporal-to-deliver-scalable-and-reliable-web-data
- Temporal, "Managing very long-running Workflows" — https://temporal.io/blog/very-long-running-workflows
- Temporal, use cases and design patterns — https://docs.temporal.io/evaluate/use-cases-design-patterns
- Dagster, "Componentizing asset factories" — https://docs.dagster.io/guides/build/components/asset-factories-to-components
- Dagster, "Creating asset factories" — https://docs.dagster.io/guides/build/assets/creating-asset-factories
- Dagster discussion, "Asset factory — build a set of assets from YAML specs" — https://github.com/dagster-io/dagster/discussions/11045
- ScrapeOps, "Web Scraping With Celery & RabbitMQ" — https://scrapeops.io/web-scraping-playbook/celery-rabbitmq-scraper-scheduling/

**Per-source config / spider maintenance**
- Zyte, "Lessons learned scraping 100 billion product pages" (2018-07-02) — https://www.zyte.com/blog/price-intelligence-web-scraping-at-scale-100-billion-products/
- Zyte, Spidermon (spider monitoring) — https://www.linkedin.com/pulse/monitor-your-web-data-like-pro-open-source-tool-spidermon-zytedata
- Portia (archived 2026-06-24) — https://github.com/scrapinghub/portia
- Zyte, "Announcing Portia, the open-source visual web scraper" — https://www.zyte.com/blog/announcing-portia/
- pupa ARCHITECTURE.md — https://github.com/opencivicdata/pupa/blob/master/ARCHITECTURE.md
- pupa repo — https://github.com/opencivicdata/pupa
- Open States scrapers — https://github.com/openstates/openstates-scrapers
- Open States, "Contributing to Scrapers" — https://docs.openstates.org/contributing/scrapers/
- Open Civic Data, "Getting Started Writing Scrapers" — https://open-civic-data.readthedocs.io/en/latest/scrape/basics.html

**Raw-archive-first / decoupling**
- Barbacena, "Save First, Parse Later: In Defense of a Different Approach to Web Scraping" (2021-12-16) — https://medium.com/better-programming/save-first-parse-later-in-defense-of-a-different-approach-to-web-scraping-9edfe65adf04

**Government & scientific bulk / incremental harvesting**
- SEC, Developer Resources (10 req/s, User-Agent, full-index/daily-index/Feed) — https://www.sec.gov/about/developer-resources
- SEC, Accessing EDGAR Data — https://www.sec.gov/edgar/searchedgar/accessing-edgar-data.htm
- SEC, new rate control limits announcement — https://www.sec.gov/filergroup/announcements-old/new-rate-control-limits
- NLM/NCBI, "How to Access NCBI Data in Bulk" (no scripting against web pages; E-utilities limits; FTP; cloud) — https://support.nlm.nih.gov/kbArticle/?pn=KA-05510
- NCBI, E-utilities usage guidelines and API key — https://eutilities.github.io/site/API_Key/usageandkey/
- NCBI Insights, "Release Plan for E-utility API Keys" (2018-08-14) — https://ncbiinsights.ncbi.nlm.nih.gov/2018/08/14/release-plan-for-e-utility-api-keys/
- PubMed, "Download PubMed Data" (annual baseline + daily updates) — https://pubmed.ncbi.nlm.nih.gov/download/
- PubMed baseline README — https://ftp.ncbi.nlm.nih.gov/pubmed/baseline/README.txt
- NLM Technical Bulletin, "PubMed Update: FTP Improvements Available for Testing" (2025 May–Jun) — https://www.nlm.nih.gov/pubs/techbull/mj25/mj25_pubmed_ftp_data.html
- PMC FTP service (2026 removal of legacy dataset files) — https://pmc.ncbi.nlm.nih.gov/tools/ftp/
- PMC on AWS (`pmc-oa-opendata`, ETags, daily S3 inventory) — https://pmc.ncbi.nlm.nih.gov/tools/pmcaws/
- api.data.gov Developer Manual (1,000/hr, DEMO_KEY, 429, X-RateLimit headers) — https://api.data.gov/docs/developer-manual/
- api.data.gov Agency Manual — https://api.data.gov/docs/agency-manual/
- GSA, Regulations.gov API (rate limits, 5,000-result cap, lastModifiedDate keyset pagination) — https://open.gsa.gov/api/regulationsgov/
- resources.data.gov, "What is Harvesting?" (daily/weekly schedules, DCAT-US validation) — https://resources.data.gov/resources/harvester-what-is-harvesting/
- resources.data.gov, Harvester API — https://resources.data.gov/harvester-api/
- Data.gov CKAN catalog announcement — https://data.gov/announcements/datagov-ckan-catalog/
- GovInfo API (usgpo) — https://github.com/usgpo/api
- OAI-PMH v2.0 specification — https://www.openarchives.org/OAI/openarchivesprotocol.html
- OAI-PMH Guidelines for Harvester Implementers (resumption tokens, from/until overlap, 503 Retry-After, idempotency) — https://www.openarchives.org/OAI/2.0/guidelines-harvester.htm
- OAI-PMH Guidelines for Repository Implementers — https://www.openarchives.org/OAI/2.0/guidelines-repository.htm
- ResourceSync Framework Specification v1.1 — https://www.openarchives.org/rs/1.1/resourcesync
- ResourceSync Change Notification — https://www.openarchives.org/rs/notification
- ResourceSync table of contents — https://www.openarchives.org/rs/toc
- NISO ResourceSync standards page — https://www.niso.org/standards-committees/resourcesync
- "Evaluating the Performance of OAI-PMH and ResourceSync" (Repository Fringe, 2018) — https://libraryblogs.is.ed.ac.uk/repofringe18/files/2018/06/2.-Evaluating-the-performance-of-OAI-PMH-and-ResourceSync.pdf
- CORE blog, "Increasing the Speed of Harvesting with On Demand Resource Dumps" (2018-03-17) — https://blog.core.ac.uk/2018/03/17/increasing-the-speed-of-harvesting-with-on-demand-resource-dumps/
- py-resourcesync — https://github.com/resourcesync/py-resourcesync



---

# Failure Handling in Long-Running, Multi-Source Harvesting Systems

Research notes for a continuously-running, unattended, many-engine harvesting system hitting
government and scientific sites/APIs. Findings only — no single recommendation. Where
practitioners disagree, both sides are given with attribution.

Research date: 2026-09-06. Publication dates noted per source; staleness flagged inline.

---

## 1. Retry strategy in practice

### 1.1 Backoff and jitter — the canonical source

Marc Brooker, **"Exponential Backoff And Jitter"**, AWS Architecture Blog, **published 2015-03-04
(page notes an update in May 2023)** — https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/

Simulation setup: optimistic-concurrency-control writes against a remote database over a
network with **mean delay 10ms, variance 4ms**, with contending clients (100 clients in the
headline runs).

The four algorithms, as published:

| Name | Formula |
|---|---|
| No Jitter (capped exponential) | `sleep = min(cap, base * 2**attempt)` |
| Full Jitter | `sleep = random(0, min(cap, base * 2**attempt))` |
| Equal Jitter | `sleep = min(cap, base*2**attempt)/2 + random(0, min(cap, base*2**attempt)/2)` |
| Decorrelated Jitter | `sleep = min(cap, random(base, sleep_last * 3))` |

Results: no-jitter capped exponential backoff produced the **most total work and the longest
completion time**. Jittered variants cut total work **by more than half**. Full Jitter and Equal
Jitter both substantially reduced call counts; Decorrelated Jitter was slightly higher in call
count than Full Jitter but comparable overall.

The mechanism matters more than the ranking: jitter converts *clustered* retry attempts into an
approximately *constant request rate*. Without it, a single blip synchronises every client and
the retry wave re-collides on each cycle.

Companion post by the same author: **"Jitter: Making Things Better With Randomness"** (2015-03-21)
— https://brooker.co.za/blog/2015/03/21/backoff.html

### 1.2 The dissent within the same camp: backoff is overrated

Marc Brooker, **"What is Backoff For?"** (2022-08-11) — https://brooker.co.za/blog/2022/08/11/backoff.html

This is the important nuance and it is *by the author of the canonical backoff post*. His thesis:

> "Backoff helps in the short term. It is only valuable in the long term if it reduces total work."

His distinction is between **bounded** and **unbounded** client populations:

- With a **small, sequential** client pool, backoff genuinely reduces offered load: each client's
  retry timing depends on its own previous failure, so delaying it removes work from the window.
- With **many independent clients** (the unbounded case), backoff between retries provides
  minimal benefit during *sustained* overload. Each newly arriving client has no idea others
  have backed off. Deferred retries **postpone** work rather than eliminate it.

He separates "first tries" from "second tries": backoff only ever governs second tries. If the
system is drowning in first tries, backoff cannot save it — you must reduce work.

His conclusion: retry *policy* (adaptive token buckets) matters more than the backoff curve alone.

**Relevance to a harvester:** a harvesting fleet is closer to the *bounded* case than a consumer
API is — you control every client. That makes backoff more effective for you than Brooker's
pessimistic case, but it also means you own the total-work budget outright.

### 1.3 Retry budgets and token buckets

Marc Brooker, **"Fixing retries with token buckets and circuit breakers"** (2022-02-28) —
https://brooker.co.za/blog/2022/02/28/retries.html

Concrete scheme he proposes: **each success deposits 0.1 tokens; each retry consumes 1 token.**
Effect: behaves like aggressive retrying when the failure rate is low, and self-throttles
automatically as failures rise — without anyone choosing an explicit failure threshold.

He notes the token bucket is preferable to both fixed per-request retry counts and to circuit
breakers alone, because circuit breakers are **modal** (hard switch between retrying and not),
and because individual clients with **small sample sizes** trip prematurely — a specific problem
in serverless / many-short-lived-worker architectures.

### 1.4 Google SRE: the actual numbers

**"Handling Overload"**, Site Reliability Engineering (O'Reilly, 2016; free online) —
https://sre.google/sre-book/handling-overload/

- **Per-request retry budget: 3 attempts maximum.** Rationale quoted: if a request has already
  landed on overloaded tasks three times, "it's relatively unlikely that attempting it again will
  help because the whole datacenter is likely overloaded."
- **Client-side retry budget: 10%.** Each client tracks the ratio of requests that are retries;
  a request is only retried while that ratio is **below 10%**.
- **Anti-amplification:** when a backend knows a request cannot be served, it returns an
  explicit **"overloaded; don't retry"** error rather than a generic task-overloaded error. This
  confines retrying to the layer immediately above the rejecting layer.
- **Adaptive throttling** (client-side "client rejects its own requests"):
  `max(0, (requests - K*accepts) / (requests + 1))`, with **K = 2** as the general default.
  Reducing K toward 1.1 makes throttling more aggressive.
- **Criticality levels** propagated with requests: `CRITICAL_PLUS`, `CRITICAL`, `SHEDDABLE_PLUS`
  (default for **batch jobs**), `SHEDDABLE`. Note that batch/harvest work is explicitly the
  class expected to absorb partial unavailability.

**"Addressing Cascading Failures"**, same book — https://sre.google/sre-book/addressing-cascading-failures/

- "Always use randomized exponential backoff when scheduling retries."
- **Retry amplification arithmetic:** 3 retries (4 attempts) at each of 3 layers = **4³ = 64**
  attempts at the database for one user action.
- **Service-wide retry budget example: 60 retries per minute per process.**
- **Queue sizing: keep queue length at 50% or less of thread-pool size** for steady traffic.
- **Bimodal latency worked example:** if 5% of requests never complete and the deadline is 100s,
  those requests occupy **5,000 threads**; with only 1,000 threads available the service serves
  **19.6%** of requests — an **80.4% error rate** grown out of an initial 5% problem.
- **Deadline sanity:** avoid deadlines "orders of magnitude larger than mean latency" — their
  example is 100ms mean latency against a 100s deadline (a 1000x multiplier).
- **Recovery is not symmetric:** if a service handles 10,000 QPS but cascades at 11,000 QPS,
  dropping back to 9,000 QPS will **not** stabilise it; load may need to fall to **~1,000 QPS
  (10%)** while only 10% of servers are healthy.

### 1.5 Why naive retries produce metastable failure

**"Metastable Failures in Distributed Systems"** — Bronson (Rockset), Aghayev, Charapko, Zhu.
**HotOS '21**, May 31–June 2 2021 —
https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-bronson.pdf
(ACM DL: https://dl.acm.org/doi/10.1145/3458336.3465286)

Three states: **stable**, **vulnerable** (efficient but trippable), and **metastable failure** —
a bad state that **persists after the trigger is removed**, with very low goodput.

Central claim, and the reason this paper matters for an unattended system:

> the root cause of a metastable failure is the **sustaining feedback loop**, rather than the trigger.

Named sustaining effects:

- **Retries as work amplification.** Outage → retransmits → overload → high latency → more client
  timeouts → more retries. Self-sustaining; removing the original outage does not clear it.
- **Look-aside cache collapse.** Cache loss → low hit rate → slow DB → cannot refill cache.
  A 90% hit rate lost implies **~10× query amplification** on the database.
- **Slow error paths** consuming more resources than success paths during failure.
- **Link imbalance** via connection-pool MRU policies — this one went **unexplained for more than
  two years** at the reporting organisation.

Recommended mitigations from the paper: adaptive policies that **disable retries/failover during
overload**; **retry budgets**; **prioritise original requests over retries**; **fast error paths**
with bounded queues and throttled logging; production stress testing to find scale-dependent
loops; tracking characteristic metrics (queueing delay, latency, cache hit rate) that indicate
*vulnerability* before the trip; and measuring **hidden capacity** — the maximum load at which
the system still self-heals.

Commentary and follow-up work:
- Marc Brooker, "Metastability and Distributed Systems" (2021-05-24) — https://brooker.co.za/blog/2021/05/24/metastable.html
- Aleksey Charapko, "Metastable Failures in the Wild" — https://charap.co/metastable-failures-in-the-wild/
- USENIX ;login: — https://www.usenix.org/publications/loginonline/metastable-failures-wild
- Murat Demirbas, "Analyzing Metastable Failures in Distributed Systems" (2025-06) — https://muratbuffalo.blogspot.com/2025/06/analyzing-metastable-failures-in.html

**Harvester-specific reading:** the classic metastable trap for a harvester is a retry queue that
grows faster than it drains after a source recovers. The source is fine; the harvester is still
dead. Backlog draining must itself be rate-limited, or recovery re-triggers the outage.

---

## 2. Circuit breakers — an unresolved disagreement

### 2.1 The case for

Martin Fowler, **"CircuitBreaker"** (**2014-03-06**) — https://martinfowler.com/bliki/CircuitBreaker.html

State machine: **closed** (calls pass) → **open** (calls fail immediately without executing) →
**half-open** (a trial call probes recovery). His illustrative defaults in the sample code:
**failure threshold 5**, invocation timeout 0.01s, reset timeout 0.1s (these are toy values for
the example, not production guidance).

His operational advice is the durable part: **"any change in breaker state should be logged and
breakers should reveal details of their state for deeper monitoring."** Operators should be able
to **manually trip or reset** a breaker, and breaker behaviour is "a good source of warnings
about deeper troubles in the environment."

Note the age: 2014. The pattern is not stale but the ecosystem around it has moved (see 2.3).

Implementations: Netflix Hystrix (now maintenance-only), **resilience4j**
(https://resilience4j.readme.io/), **Polly** for .NET (https://www.pollydocs.org/).

### 2.2 The case against

Marc Brooker, **"Will circuit breakers solve my problems?"** (2022-02-16) —
https://brooker.co.za/blog/2022/02/16/circuit-breakers.html

He concedes breakers do two things well: **avoid wasting resources by failing fast**, and enable
**graceful degradation** when an optional dependency is unavailable.

His core objection is **partial failure**. His worked example is a sharded database where one
shard (keys A–H) is overloaded while the others are healthy. Then:

> "Is this database down? Have failures reached a threshold?"

If the breaker trips, requests that *would have succeeded* against healthy shards now fail. If it
doesn't trip, the breaker is useless. Circuit breakers **convert partial failures into complete
failures**, which runs against the grain of modern distributed system design, where partial
availability is the goal. His summary: when you try to balance partial-failure tolerance against
breaker protection, "one mechanism will likely defeat the other."

His suggested alternatives: tight coupling between client and service internals; **servers
telling clients specific overload information**; or statistical/ML inference of per-shard health.

### 2.3 Netflix moved away from Hystrix

The Hystrix README itself (https://github.com/Netflix/Hystrix/blob/master/README.md) states:

> "Hystrix is no longer in active development, and is currently in maintenance mode."

Final release **1.5.18** (matching the last internally-used stable version, 1.5.11). Netflix will
not review issues, merge PRs, or cut releases. The stated reason is a shift toward

> "more adaptive implementations that react to an application's real time performance rather than
> pre-configured settings (for example, through adaptive concurrency limits)."

For new projects Netflix points to **resilience4j**.

The replacement approach: **"Performance Under Load"** — Eran Landau, William Thurston, Tim
Bozarth, Netflix Tech Blog, **2018-03-23** —
https://netflixtechblog.medium.com/performance-under-load-3e6fa9a60581

- The problem with static limits: they were set by "an arduous process of performance testing and
  profiling," and went stale whenever topology changed (outages, autoscaling, deploys).
- Theory: **Little's Law** (concurrency = average service time × arrival rate) plus **TCP
  congestion control**, specifically latency-based algorithms (TCP Vegas).
- Algorithm: `gradient = RTTnoload / RTTactual`; `newLimit = currentLimit × gradient + queueSize`,
  with the queue allowance set to **sqrt(currentLimit)** so limits grow fast at low concurrency
  and stay stable at high concurrency. Gradient near 1 → room to grow; lower → excess queueing.
- Outcome: server-side limits reject excess traffic, latencies stay low, manual tuning eliminated.
- Open-sourced: https://github.com/Netflix/concurrency-limits

**The disagreement, stated plainly and left unresolved:**

- *Fowler / Hystrix / resilience4j / Polly camp:* an explicit, observable, manually-overridable
  breaker per dependency is a well-understood operational control, and its state changes are
  themselves a valuable alarm signal.
- *Brooker / Netflix camp:* thresholds are hard to choose and harder to test, breakers are modal,
  small clients trip on noise, and — most importantly — they turn partial failures into total
  ones. Prefer continuous, adaptive mechanisms: token-bucket retry budgets and adaptive
  concurrency limits that degrade smoothly rather than switching.

Note that Brooker's own token-bucket post (§1.3) allows a *narrow* role for breakers: **breaking
retries** while letting first attempts through. That is a materially weaker claim than the
general "circuit-break the dependency" pattern, and it is the one place the two camps overlap.

---

## 3. Timeouts, deadlines, hedging, bulkheads

### 3.1 Deadlines over timeouts

From Google SRE "Addressing Cascading Failures" (URL above):

- **Deadline propagation**: an *absolute* deadline travels with the request tree. Server A sets
  30s, spends 7s, and issues its RPC to B with **23s** remaining — not a fresh 30s. Without this,
  servers burn capacity on requests the caller already abandoned. Their counterexample: B starts
  8s in and hardcodes a 20s deadline to C when only 2s remain.
- Both extremes are named as failure modes: no deadline (or a huge one) lets a transient problem
  consume resources indefinitely; too-short deadlines make expensive requests fail permanently.
- Mitigations: monitor the **latency distribution**, not the mean; return errors early rather
  than waiting out the full deadline; per-keyspace request limits so one client cannot exhaust
  capacity.

### 3.2 Hedged requests

**"The Tail at Scale"** — Jeffrey Dean and Luiz André Barroso, **Communications of the ACM, Vol.
56 No. 2, February 2013** — https://cacm.acm.org/research/the-tail-at-scale/

- **Hedged requests:** send the same request to a second replica *only after* the first has been
  outstanding longer than the **95th-percentile expected latency**. Extra load is therefore
  bounded at roughly **5%**.
  Measured: reading 1,000 keys across 100 servers, **99.9th-percentile latency fell from 1,800ms
  to 74ms** for **2% additional requests**.
- **Tied requests:** enqueue on two servers, each of which cancels the twin when it starts
  executing; insert a delay of **2× average network message delay (≤1ms in a datacenter)** before
  the second send. Measured: **16% median** and **~40% at 99.9th percentile** improvement, at
  **<1% extra disk utilisation**.
- **Micro-partitioning:** ~**20 partitions per machine**, so load can be shed in ~**5% increments**.
- **Selective replication** of hot items; **latency-induced probation** — temporarily remove slow
  machines while continuing to send shadow requests to decide when to reinstate them.

**Caveat for this use case:** hedging is designed for *replicated* backends you own. Against a
single government endpoint, a hedge is just a duplicate request — it doubles your load on a
source you are trying to be polite to and may look like abuse. Hedging is applicable to your
*internal* fan-out (parsers, storage, proxy pools), and to genuinely mirrored sources
(e.g. multiple mirrors of the same dataset), not to the source fetch itself.
**Latency-induced probation, by contrast, maps directly onto "park a slow source."**

### 3.3 Bulkheads / per-source isolation

resilience4j Bulkhead — https://resilience4j.readme.io/docs/bulkhead

Two implementations with published defaults:

- **SemaphoreBulkhead** — `maxConcurrentCalls = 25`, `maxWaitDuration = 0ms` (non-blocking).
  Documented as working "well across a variety of threading and I/O models."
- **FixedThreadPoolBulkhead** — max pool size = available processors; core pool = processors − 1;
  **queue capacity 100**; keep-alive 20ms.

Design note: unlike Hystrix, resilience4j's semaphore bulkhead deliberately provides **no
"shadow" thread pool**; the caller must size its own pools coherently.

The harvesting translation of a bulkhead is a **per-source concurrency cap plus a per-source
queue**, so that one slow or hanging source cannot occupy the shared worker pool. Google SRE's
"keep queue length ≤ 50% of thread pool size" (§1.4) is the sizing heuristic; the bimodal-latency
worked example (5% hanging requests → 80.4% error rate) is the concrete failure this prevents.

### 3.4 Per-source adaptive pacing, as actually shipped

Scrapy **AutoThrottle** — https://docs.scrapy.org/en/latest/topics/autothrottle.html

- Target delay `= latency / AUTOTHROTTLE_TARGET_CONCURRENCY`; the applied delay is the **average
  of the previous delay and the target delay** (smoothing).
- Defaults: `AUTOTHROTTLE_START_DELAY = 5.0s`, `AUTOTHROTTLE_MAX_DELAY = 60.0s`,
  `AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0`.
- Critical asymmetry, quoted: **"latencies of non-200 responses are not allowed to decrease the
  delay."** Errors usually return *faster* than successes, so feeding error latencies into the
  controller would speed the crawler up exactly when it should slow down.

That asymmetry is a real, shipped instance of the metastable-failure mitigation in §1.5, and it
is the single most transferable detail in this section: **any adaptive rate controller keyed on
latency must ignore or invert fast failures.**

---

## 4. Classifying failures — the part most systems get wrong

### 4.1 Retry classification as shipped by Scrapy

Scrapy `RetryMiddleware` — https://docs.scrapy.org/en/latest/topics/downloader-middleware.html

- `RETRY_ENABLED = True`
- **`RETRY_TIMES = 2`** — "Maximum number of times to retry, in addition to the first download."
  (So 3 total attempts — the same number Google SRE arrives at independently.)
- **`RETRY_HTTP_CODES = [500, 502, 503, 504, 522, 524, 408, 429]`**
- `RETRY_EXCEPTIONS` — connection refused, DNS resolution failure, download timeout,
  `ResponseDataLossError`, assorted Twisted connection errors, `OSError`, `TunnelError`.
- **`RETRY_PRIORITY_ADJUST = -1`** — retries are scheduled at *lower* priority than fresh
  requests. This is exactly the Metastable-Failures recommendation to "favor original requests
  over retries" (§1.5), implemented as a one-line default.

Note what is **absent** from `RETRY_HTTP_CODES`: **403** and **404**. A 403 block and a 404 are
treated as terminal, not transient — a deliberate classification choice, not an oversight.

### 4.2 The failure taxonomy a harvester actually needs

Consolidating the sources, the distinct classes and their conventional handling:

| Class | Signal | Conventional handling |
|---|---|---|
| DNS / connection refused / TLS error | exception before response | Retry with backoff; if persistent, the *source* is down, not the request |
| Timeout | no response within deadline | Retry a bounded number of times; count toward source health |
| 5xx (500/502/503/504) | server-side | Retry with jittered backoff. 503 specifically may carry `Retry-After` |
| 429 Too Many Requests | explicit rate limit | **Obey `Retry-After`**; reduce concurrency for that source; do *not* treat as generic 5xx |
| 403 / 401 / block page | anti-bot or auth | **Not retryable by backoff.** Retrying makes it worse. Escalate/park |
| 404 | resource genuinely gone | Terminal for that URL; but a *spike* in 404s across a source = structural change |
| **Soft 404** | HTTP 200 + "not found"/empty body | Invisible to HTTP-level retry logic. Needs content inspection |
| **Selector missing** | HTTP 200, page parsed, field absent | Site redesign or A/B test. The dangerous one: pipeline reports success |
| **Silently empty data** | HTTP 200, valid structure, zero rows | Often an upstream outage rendered as an empty result set |
| Poison record | one record kills the parser repeatedly | DLQ / quarantine, do not block the batch |

The last four are the ones that distinguish harvesting from generic RPC resilience: **the
transport succeeded and the data is wrong.** No amount of retry/circuit-breaker machinery
detects them, because every layer below reports 200 OK.

Practitioner write-ups on this specific gap (note: mostly vendor blogs and Medium posts, i.e.
secondary/marketing-adjacent sources — treat as corroboration of a widely-felt problem, not as
rigorous evidence):

- "200 OK but no data: Diagnosing incorrect page responses", Web Scraper —
  https://webscraper.io/blog/200-ok-but-no-data-diagnosing-incorrect-page-responses
- "Debugging Web Scraper Failures: 403s, 429s, Timeouts, and Empty Results", Context.dev —
  https://www.context.dev/blog/debugging-web-scraper-failures
- "Scraper Monitoring in Production: A Practical Guide", Context.dev —
  https://www.context.dev/blog/scraper-monitoring-in-production
- "Why Web Scraping Fails Silently (And Why That's Worse Than a Crash)", Ficstar (2026-07) —
  https://ficstar.medium.com/why-web-scraping-fails-silently-and-why-thats-worse-than-a-crash-2b4a8f615ccf
- "Why you need to monitor long-running, large-scale scraping projects", Apify —
  https://dev.to/apify/why-you-need-to-monitor-long-running-large-scale-scraping-projects-5flj
- "Scraping observability: success metrics, block-rate dashboards, and silent failures" —
  https://blog.crawlex.net/blog/scraping-observability/

The recurring recommendation across all of them is the same and is **statistical, not
per-request**: track per-source **item yield per run**, **field fill rates**, **response size
distribution**, and **block rate**, and alert on *deviation from the source's own baseline*
rather than on absolute thresholds. A run that returns 40% of yesterday's row count with 200 OK
throughout is the canonical silent failure.

### 4.3 Soft-404 and near-duplicate detection

Soft-404s are a recognised problem in the search-engine world, predating scraping. Google Search
Central's HTTP/network errors page
(https://developers.google.com/search/docs/crawling-indexing/http-network-errors) treats them as a
named error class: Search Console reports a **`soft 404`** when page content suggests an error
**despite a 2xx status code**. The same page shows how a very large crawler classifies rate
signals — **"Google's crawlers treat the `429` status code as a signal that the server is
overloaded, and it's considered a server error"**, grouped with 5xx as prompting crawlers to
"temporarily slow down," after which "Google gradually increases the crawl rate for the site"
once 2xx responses resume. Note the **asymmetric ramp**: fast to slow down, slow to speed back
up — the same shape as Scrapy AutoThrottle (§3.4) and CCBot (§7.2).

Detection techniques that appear across the literature:
- **Fetch a known-bad URL** on the same host (a random path) and fingerprint the response;
  compare live responses against that "known 404 shape."
- **Near-duplicate detection** across responses within a source — if many distinct URLs return
  byte-similar or shingle-similar bodies, they are probably all the same error page.
  (Classic technique: **SimHash** — Charikar, "Similarity Estimation Techniques from Rounding
  Algorithms", STOC 2002; applied at web scale in Manku, Jain, Das Sarma, "Detecting Near-
  Duplicates for Web Crawling", WWW 2007 — https://dl.acm.org/doi/10.1145/1242572.1242592)
- **Content-length / entropy thresholds** per source.
- **Sentinel/canary records**: a small set of URLs whose expected extracted values are known and
  stable; if the canary's fields stop extracting, the parser broke regardless of HTTP status.

---

## 5. Queues, DLQs, quarantine, and delivery semantics

### 5.1 Dead-letter queues and poison pills

AWS SQS documentation — "Capturing problematic messages"
(https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/capturing-problematic-messages.html)
and "Amazon SQS dead-letter queues"
(https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

The mechanism: a **redrive policy** with **`maxReceiveCount`** — the number of times a consumer
may receive a message before SQS moves it to the DLQ. AWS's stated purpose is precisely poison-pill
containment: DLQs "help reduce poison pill messages (messages that are received but can't be
processed)" and stop the `ApproximateAgeOfOldestMessage` metric from being distorted by messages
that will never succeed. AWS documents **redrive** back to the source queue once the underlying
bug is fixed.

Two documented gotchas that matter for a months-long system:

- **DLQ retention is measured from the ORIGINAL enqueue timestamp** (standard queues). Moving a
  message to the DLQ does **not** reset its clock. AWS's own worked example: a message that sat
  1 day in the source queue, then moved to a DLQ with a 4-day retention, is deleted **3 days
  later** — 4 days from original enqueue, not from the move. Best practice per AWS: set the DLQ
  retention period **longer than the source queue's**. (FIFO queues *do* reset the timestamp on
  move — the behaviour differs between queue types.)
- For standard queues with `maxReceiveCount > 3`, a message received 3+ times without deletion is
  moved to the **back of the queue** before eventually reaching the DLQ.
- AWS explicitly cautions **against** DLQs with FIFO queues when message order is critical, since
  pulling a message out breaks the sequence.

The **parking lot** variant (a second queue beyond the DLQ, for messages that failed even after
DLQ reprocessing) is widely described in the RabbitMQ/Kafka community, though mostly in
secondary sources:
- https://dev.to/dakshim/retries-dead-letter-queues-parking-lot-api-integration-essentials-2g8d
- https://dev.to/thiago_souza_1510/exploring-rabbitmq-queue-types-routing-key-dead-letter-and-parking-lot-i8d

A dissenting operational view worth recording — Oskar Dudycz, "On rebuilding read models,
Dead-Letter Queues and Why Letting Go is Sometimes the Answer" —
https://www.architecture-weekly.com/p/on-rebuilding-read-models-dead-letter
Argues DLQs frequently become write-only graveyards nobody drains, and that for some workloads
the honest answer is to **drop and re-derive** rather than to preserve every failed message.
For a harvester this is a live question: a failed fetch is usually **re-derivable by re-fetching**,
which makes the DLQ a *diagnostic* record rather than a *recovery* mechanism — a materially
different design point from financial messaging.

### 5.2 Quarantining a failing source

The pattern that appears repeatedly (Netflix's latency-induced probation in §3.2; the
"park the source" language in scraping blogs; CKAN's harvest-source disabling in §7) is:

- Track health **per source**, not per request.
- On sustained failure, move the source to a **quarantine/parked** state with a long, jittered
  re-probe interval, rather than continuing to burn the shared worker pool.
- Continue **low-rate probing** (shadow requests) so recovery is detected automatically.
- Alert on *entry into* quarantine, and on sources that have been quarantined longer than N days
  (the "silently stopped harvesting this agency six weeks ago" failure).

This is functionally a circuit breaker at source granularity — and notably, Brooker's main
objection (§2.2, partial failure within a sharded dependency) is *weaker* here, because a single
government site is much closer to an all-or-nothing dependency than a sharded database is. The
partial-failure objection returns if a source is itself sharded/paginated and only some
partitions fail.

### 5.3 At-least-once, exactly-once, idempotent writes

The practical consensus across the sources: **exactly-once delivery is not available; exactly-once
*effect* is, via idempotent writes.**

- **Temporal** recommends Activities be idempotent and states the reason plainly
  (https://docs.temporal.io/activity-definition): "Activities may be retried, these functions may
  be executed more than once. A non-idempotent Activity could adversely affect the state of the
  system." The precise hazard is named: "Activities won't record to the Event History until they
  return or produce an error. **If an Activity fails to report to the server at all, it will be
  retried**" — i.e. an Activity that *succeeded* but died before reporting runs again.
  Their suggested key is the **Workflow Run ID + Activity ID**, "guaranteed to be consistent
  across retry attempts but unique among Workflow Executions." Crucially they add that the key
  "should be enforced by the service you are calling from your Activity, **not by the Activity
  itself**" — which, for a harvester, means the *storage layer* must enforce uniqueness, since
  the remote government API certainly will not.
- AWS SQS standard queues are at-least-once with possible duplicates; FIFO queues offer
  deduplication within a **5-minute** dedup window —
  https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html

For a harvester the natural idempotency key is `(source_id, resource_id, content_hash)` or
`(source_id, url, fetch_run_id)`, enforced as a uniqueness constraint in the store with
upsert-by-key — following Temporal's rule that the *callee* enforces the key. Content-hashing
pays twice: it makes "the site returned the same bytes as yesterday" (skip the write, but record
the check) and "the site returned a different error page every time" (a §4.2 silent failure)
both cheaply detectable, and it makes re-running a harvest safe by construction, which is what
makes the "drop and re-fetch" stance in §5.1 viable at all.

Note the interaction with §6.1: OAI-PMH's recommended datestamp **overlap** deliberately
re-fetches records, so incremental harvesting *depends* on idempotent writes to be correct
rather than merely efficient.

---

## 6. Checkpointing, resumability, durable workflow engines

### 6.1 The protocol-level version: OAI-PMH

For scientific/government metadata specifically, the resumability problem is **standardised**.

OAI-PMH **Implementation Guidelines for Harvester Implementers**, version dated
**2005-01-19** (protocol v2.0, 2002-06-14) —
https://www.openarchives.org/OAI/2.0/guidelines-harvester.htm
Protocol spec: https://www.openarchives.org/OAI/openarchivesprotocol.html

**Flag: this document is over 20 years old.** It remains the normative guidance and OAI-PMH is
still in wide use across repositories, but expect its transport assumptions to be dated.

Concrete guidance it gives:

- **Flow control via 503 + `Retry-After`.** Quoted: *"Harvesters that encounter a 503 reply
  without a Retry-After header should not automatically retry without considerable delay
  (minutes) or, preferably, manual intervention."* Note: **no numeric backoff is recommended**;
  manual intervention is explicitly preferred. This is a notably more conservative stance than
  anything in the SRE literature, and reflects the norms of harvesting *someone else's* server.
- **`resumptionToken` semantics.** On `badResumptionToken`, the harvester "must assume that the
  resumptionToken has either expired or is invalid" and **restart the entire list sequence**.
  Tokens are **idempotent on re-issue**, so a network error mid-sequence can be recovered by
  simply re-issuing the last token — but an *expired* token means losing all progress in that
  sequence.
- **Incremental harvesting:** overlap successive incremental harvests **by one datestamp
  increment** (one day at `YYYY-MM-DD` granularity) to avoid missing items that changed during
  the harvest; base the next `from` on the **`responseDate` of the first response**, not on the
  harvester's own clock.

That last point is a general lesson worth extracting: **checkpoint against the source's clock,
not yours.** Clock skew between harvester and source silently drops records at the boundary.

### 6.2 Durable execution engines

**Temporal** retry policy defaults — https://docs.temporal.io/encyclopedia/retry-policies

| Parameter | Default |
|---|---|
| Initial Interval | **1 second** |
| Backoff Coefficient | **2.0** |
| Maximum Interval | **100 × Initial Interval** (100s) |
| Maximum Attempts | **∞ (unlimited)** |
| Non-Retryable Error Types | `[]` (none) |

Activities retry automatically by default; **Workflows do not**, because workflow code must stay
deterministic for replay. The documented guidance is to retry *Activities within* the workflow.

**The unlimited-default is a genuine hazard for this use case**: an unattended harvester with
`maximumAttempts = ∞` against a permanently-removed dataset retries forever at a 100s cap —
roughly 864 requests/day, indefinitely, against a government endpoint. Non-retryable error types
(e.g. 403, 404) must be configured explicitly; the default configures none.

Temporal **Activity Heartbeats** — https://docs.temporal.io/encyclopedia/detecting-activity-failures

A heartbeat is "a ping from the Worker that is executing the Activity to the Temporal Service."
If the **Heartbeat Timeout** elapses with no ping, the Activity Task fails and retries per policy.

The load-bearing feature for long harvests: a heartbeat "can include an application layer payload
that can be used to *save* Activity Execution progress," and a retrying attempt can **access and
continue from that payload**. This is the built-in answer to "we were 80,000 records into a
100,000-record paginated harvest when the worker died."

Implementation detail that matters: workers **throttle** heartbeats to the smaller of 80% of the
heartbeat timeout or a configurable max (**default 60s**) — so not every heartbeat reaches the
service — but **the final heartbeat before failure always transmits**, so progress data is not
lost at the moment it matters.

Two distinct timeouts, easily confused:
- **Start-To-Close** — one Activity Task attempt, applied to **each retry independently**. Docs
  recommend always setting this explicitly, because the server "relies on the Start-To-Close
  Timeout to force Activity retries" when a worker goes silent.
- **Schedule-To-Close** — the **whole** Activity Execution including all retries. This is the
  parameter that actually bounds "how long do we keep trying this source."

**AWS Step Functions** error handling —
https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html

Verified `Retry` field defaults:

| Parameter | Default | Notes |
|---|---|---|
| `IntervalSeconds` | **1** | seconds before first retry; max 99,999,999 |
| `MaxAttempts` | **3** | 0 = never retry; max 99,999,999 |
| `BackoffRate` | **2.0** | multiplier per attempt |
| `MaxDelaySeconds` | none (unbounded) | must be >0 and <31,622,401 |
| `JitterStrategy` | **`NONE`** | allowed values `FULL` \| `NONE` |

**Worth flagging:** AWS shipped Marc Brooker's Full Jitter (§1.1) as a named, first-class option —
and then **defaulted it to `NONE`**. Uncapped, unjittered exponential backoff at `BackoffRate 2.0`
is what you get unless you opt in. This is the single most common configuration gap in the
sources reviewed.

Structured error classification is built in via predefined error names — `States.ALL` (wildcard;
must appear alone, and cannot catch `States.DataLimitExceeded` or `States.Runtime`),
`States.TaskFailed` (wildcard excluding timeouts), `States.Timeout`, `States.HeartbeatTimeout`,
`States.Runtime` (**explicitly not retriable**), `States.Permissions`, `States.Http.Socket`
(HTTP task timeout after 60s), plus Map-state errors including
`States.ExceedToleratedFailureThreshold`. `Catch` routes to a fallback state; when both are
present **Retry runs first, then Catch**. That `Retry`-then-`Catch` split is a clean expression
of the transient-vs-terminal distinction of §4.2.

**Restate** — https://restate.dev/ ; comparison page (vendor-authored, treat accordingly):
https://restate.dev/vs/temporal

**The trade-off, both sides:**

- *For durable engines:* checkpointing, retries, timers, and resumption are solved, tested, and
  observable. A months-long harvest with mid-run worker restarts is precisely the workload they
  exist for. Heartbeat-based partial progress is hard to hand-roll correctly.
- *Against:* they impose determinism constraints and a versioning discipline on workflow code
  (Restate's own writing on the "immutability problem" —
  https://www.restate.dev/blog/solving-durable-executions-immutability-problem — is candid that
  changing a long-running workflow's code mid-flight is genuinely hard). For a harvester whose
  parsers change *constantly* as sites change, that is a direct and recurring cost. They also add
  an operational dependency (a cluster, or a vendor). A hand-rolled loop with a cursor in
  Postgres is comprehensible by anyone and versions freely — at the cost of re-implementing
  retry, backoff, timers, idempotency, and observability yourself, usually incorrectly at first.

The distinguishing question in the sources is not scale but **how often the code changes relative
to how long a single execution lives.** Long executions + stable code favour durable engines;
short executions + churning parser code favour cursors in a database.

### 6.3 Checkpointing granularity

Patterns visible across the sources:
- **Cursor per source** (last successful datestamp/page/token), committed only after the batch it
  covers is durably written — commit the data and the cursor in the same transaction, or the
  cursor after the data, never before.
- **Partial-progress semantics must be explicit.** A run that fetched 80% of pages is either
  (a) published as partial with a coverage figure, or (b) held back entirely. Both are defensible;
  what breaks consumers is doing one while they assume the other.
- OAI-PMH's overlap-by-one-datestamp (§6.1) is the standard defence against boundary loss, and
  it depends on downstream idempotency (§5.3) to absorb the resulting duplicates.

---

## 7. Real-world war stories

### 7.1 data.gov: stuck harvest jobs (government, directly on point)

**GSA/data.gov wiki, "Stuck Harvest Fix"** — https://github.com/GSA/data.gov/wiki/Stuck-Harvest-Fix

This is an operational runbook from the US government's own open-data portal, harvesting from
federal agencies via CKAN. It is the closest published analogue to the system in question.

Documented reality:
- **"Mostly, harvest jobs get stuck in the fetch stage."**
- Causes listed: fetch **workers go idle** while the job still shows status `Running`; the
  **supervisor fails to restart** jobs, orphaning them; **large jobs exhaust Redis memory** and
  crash the harvester while the job remains queued; the **gather process never starts** because
  the system is backed up.
- Diagnosis: query for jobs in `Running` status with no recent harvest-object activity; inspect
  `/var/log/harvest-fetch.log` to see whether workers are idle.
- Remediation is **manual**: `sudo supervisorctl restart all`; hand-written SQL to insert error
  records and force stuck `harvest_object` rows to `state = 'ERROR'`, and to reset never-started
  jobs to `status = 'New'`; and for severe cases, SSHing to the Redis host and running
  `redis-cli -a <pw> KEYS "*harvest-job-id*" | xargs redis-cli -a <pw> DEL`.
- The wiki states the intent: **"The goal is to automate this process so that the system will
  automatically fix a stuck harvest source after 24 hours."** The continued existence of the
  manual runbook indicates that automation was not completed.

**Lessons this concretely supports:** (1) the dominant long-run failure mode is not a crash but a
**job stuck in a running-but-doing-nothing state** — which means *liveness* monitoring (progress
since last checkpoint) matters more than *error-rate* monitoring; (2) an unbounded job that
exhausts a shared resource (Redis) takes down the harvester for *every* source — the bulkhead
argument of §3.3 in the wild; (3) any state machine with a `Running` state needs an automatic
timeout out of it.

Related open issues in the same ecosystem, showing these are long-lived structural problems:
- ckanext-harvest #112, "Catch more errors to avoid limbo states" —
  https://github.com/ckan/ckanext-harvest/issues/112
- ckanext-dcat #147, "RDF job never ends if some dataset raises exception in gather stage" —
  https://github.com/ckan/ckanext-dcat/issues/147 (a **poison-pill** taking down a whole job)
- GSA/data.gov #3944, duplicate datasets from the same harvest object —
  https://github.com/GSA/data.gov/issues/3944 (an **idempotency** failure, §5.3)

### 7.2 Common Crawl — a crawler that has run continuously for over a decade

Common Crawl FAQ — https://commoncrawl.org/faq

Verified operational facts, quoted:

- **Architecture:** "a Nutch-based web crawler that makes use of the Apache Hadoop project."
- **Scope is deliberately partial:** "Common Crawl's dataset is a sample of the web, and we do not
  generally archive any entire website but a randomly selected subset of it." A long-running
  crawler that explicitly does **not** promise completeness has a far easier failure model — it
  has no "the harvest is incomplete" alarm to raise. A government harvester usually cannot make
  that trade, which is precisely why §8's coverage metrics become mandatory.
- **Adaptive backoff keyed on error class:** "The crawler uses an adaptive back-off algorithm that
  slows down requests to your website if your web server is responding with a **HTTP 429 or 5xx**
  status. By default our crawler waits few seconds before sending the next request to the same
  site."
- **Politeness:** checks robots.txt before fetching, honours **`Crawl-delay`** and `nofollow`.

That 429/5xx-triggered adaptive backoff is independent confirmation of the same design as Scrapy's
AutoThrottle asymmetry (§3.4) and the SRE overload literature (§1.4): **error class drives the rate
controller, and the controller only ever slows down in response to errors.**

### 7.3 Vendor/practitioner reports on silent breakage

Listed in §4.2. The consistent claim across independent vendors (Zyte-adjacent, Apify, Ficstar,
Context.dev, PromptCloud) is that **selector/layout drift, not blocking, is the top cause of
long-run data loss**, and that it is usually detected by *downstream consumers* rather than by
the pipeline. These are commercial blogs with an interest in selling monitoring; the convergence
across competitors is suggestive but is not independent evidence.

---

## 8. Graceful degradation and staleness

Google SRE ("Addressing Cascading Failures", URL above) defines graceful degradation as
**reducing the quality of work rather than dropping requests** — their example is a search
service answering from cached data only, or using a faster/less accurate ranker. Their
implementation cautions:

- Choose the metric that triggers degradation (CPU, latency, queue length).
- Decide automatic vs manual activation.
- **Exercise the degraded path regularly in production** — otherwise it is broken when needed.
- **Monitor and alert when degradation activates frequently** — silent permanent degradation is
  the failure mode.

Applied to harvesting, the sources support a distinction the requester already named:

- **"The harvest is late"** — last-known-good data is still served, flagged with its `as_of`
  timestamp and age. Degradation is *visible* and *bounded* by a staleness budget per source
  (a slow-moving statistical release tolerates weeks; a daily filing feed does not).
- **"The harvest is wrong"** — new data was written but is silently incomplete or misparsed.
  This is strictly worse: it is undetectable downstream and it *overwrites* the last-known-good.

The defence pattern implied throughout: **validate before publishing, and prefer serving stale
data over publishing suspect data.** Concretely — write to a staging location, run the
per-source statistical checks of §4.2 (yield, fill rates, size distribution) against the
source's own baseline, and promote to the served dataset only on pass; on fail, keep serving the
previous version and raise an alert. Every record carries `fetched_at` and a coverage/quality
flag so consumers can make their own staleness decisions.

The counter-consideration, stated for balance: strict promotion gates create their own failure
mode — a source that legitimately shrinks (an agency really did publish fewer records this
quarter) is blocked by an anomaly detector, and repeated false positives train operators to
click through. The staleness/quality trade-off is not resolvable in general; it is a per-source
policy decision about which error is more expensive.

---

## 9. Points of genuine disagreement (unresolved, by design)

1. **Circuit breakers: useful or harmful.** Fowler (2014) and the Hystrix/resilience4j/Polly
   lineage vs Brooker (2022) and Netflix (2018). Turns on whether your dependency fails
   partially or wholly, and whether you can tune and *test* a threshold. Narrow overlap: both
   camps accept breakers for *suppressing retries* specifically.
2. **Backoff: essential or overrated.** Brooker 2015 vs Brooker 2022 — the same author.
   Reconciliation: backoff is necessary but insufficient; with unbounded clients it defers work
   rather than removing it. A harvester's bounded, self-owned client population sits on the
   favourable side of this argument.
3. **Durable workflow engine vs hand-rolled cursor loop.** Correctness and free resumability vs
   determinism/versioning constraints on code that changes constantly. Hinges on execution
   lifetime relative to code churn.
4. **DLQs: recover or discard.** Standard queueing practice (AWS redrive) vs the view that DLQs
   become unread graveyards and re-derivable work should simply be dropped and re-fetched.
   Harvesting leans toward re-derivable, which weakens the classic DLQ case.
5. **Retry aggressiveness against third-party public infrastructure.** SRE-literature numbers
   (3 attempts, 10% budget, 60/min) were written for services you *own*. OAI-PMH's guidance —
   minutes of delay, or *prefer manual intervention* — reflects a different ethic for someone
   else's server. Government/scientific endpoints are frequently under-resourced; the polite
   number is often far below the technically optimal one, and blocking is a real cost.

---

## 10. Methodology and gaps

**What was verified:** every numeric claim above was read from the cited page during this
research pass rather than recalled. Where a page did not actually support a claim, the claim was
narrowed or removed — e.g. an initial assertion that Temporal's docs state "at-least-once
execution" was corrected to what the Idempotency section literally says.

**Known gaps:**

- **No first-person postmortem from a multi-year scraping operation was located in a primary
  source.** The war-story material that *is* primary and verifiable is the data.gov runbook
  (§7.1) plus its CKAN issue tracker — genuinely on point for government harvesting, but a
  single organisation. The vendor blogs in §4.2/§7.3 converge on the same claims but are
  commercially interested and are not independent evidence. Zyte's and Diffbot's engineering
  blogs, Bright Data, and HN threads were not retrieved (this session's web-search budget was
  exhausted mid-pass); those remain the likeliest sources of the missing first-person accounts
  and are worth a follow-up.
- **GDELT** publishes little about its own failure handling; nothing citable was found.
- **Restate** material is largely vendor-authored; treat its Temporal comparisons as marketing.
  The exception is its "immutability problem" post, which is candid about a real limitation of
  the entire durable-execution category.
- **Dates:** the OAI-PMH harvester guidelines (2005) and Fowler's CircuitBreaker (2014) are the
  oldest load-bearing sources here. Both remain normative in their communities, but neither
  anticipates adaptive concurrency control. The Google SRE chapters (2016) describe pre-2016
  internal practice; the numbers are still widely cited but are not fresh measurements.

## Sources

**Retry, backoff, jitter**
- Marc Brooker, "Exponential Backoff And Jitter", AWS Architecture Blog, 2015-03-04 (updated 2023-05) — https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- Marc Brooker, "Jitter: Making Things Better With Randomness", 2015-03-21 — https://brooker.co.za/blog/2015/03/21/backoff.html
- Marc Brooker, "What is Backoff For?", 2022-08-11 — https://brooker.co.za/blog/2022/08/11/backoff.html
- Marc Brooker, "Fixing retries with token buckets and circuit breakers", 2022-02-28 — https://brooker.co.za/blog/2022/02/28/retries.html

**Overload, cascading and metastable failure**
- Google SRE Book, "Handling Overload" (2016) — https://sre.google/sre-book/handling-overload/
- Google SRE Book, "Addressing Cascading Failures" (2016) — https://sre.google/sre-book/addressing-cascading-failures/
- Bronson, Aghayev, Charapko, Zhu, "Metastable Failures in Distributed Systems", HotOS '21 — https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-bronson.pdf | https://dl.acm.org/doi/10.1145/3458336.3465286
- Marc Brooker, "Metastability and Distributed Systems", 2021-05-24 — https://brooker.co.za/blog/2021/05/24/metastable.html
- Aleksey Charapko, "Metastable Failures in the Wild" — https://charap.co/metastable-failures-in-the-wild/
- USENIX ;login:, "Metastable Failures in the Wild" — https://www.usenix.org/publications/loginonline/metastable-failures-wild
- Murat Demirbas, "Analyzing Metastable Failures in Distributed Systems", 2025-06 — https://muratbuffalo.blogspot.com/2025/06/analyzing-metastable-failures-in.html

**Circuit breakers and adaptive limits**
- Martin Fowler, "CircuitBreaker", 2014-03-06 — https://martinfowler.com/bliki/CircuitBreaker.html
- Marc Brooker, "Will circuit breakers solve my problems?", 2022-02-16 — https://brooker.co.za/blog/2022/02/16/circuit-breakers.html
- Netflix/Hystrix README (maintenance mode) — https://github.com/Netflix/Hystrix/blob/master/README.md
- Landau, Thurston, Bozarth, "Performance Under Load", Netflix Tech Blog, 2018-03-23 — https://netflixtechblog.medium.com/performance-under-load-3e6fa9a60581
- Netflix/concurrency-limits — https://github.com/Netflix/concurrency-limits
- resilience4j docs — https://resilience4j.readme.io/
- Polly docs — https://www.pollydocs.org/

**Tail latency, hedging, bulkheads**
- Dean & Barroso, "The Tail at Scale", CACM 56(2), Feb 2013 — https://cacm.acm.org/research/the-tail-at-scale/
- resilience4j Bulkhead — https://resilience4j.readme.io/docs/bulkhead

**Crawler/harvester frameworks**
- Scrapy Downloader Middleware (RetryMiddleware) — https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
- Scrapy AutoThrottle — https://docs.scrapy.org/en/latest/topics/autothrottle.html
- Common Crawl FAQ — https://commoncrawl.org/faq

**Queues, DLQs, idempotency**
- AWS SQS, "Capturing problematic messages" (DLQ/redrive) — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/capturing-problematic-messages.html
- AWS SQS FIFO queues (deduplication) — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html
- Oskar Dudycz, "On rebuilding read models, Dead-Letter Queues..." — https://www.architecture-weekly.com/p/on-rebuilding-read-models-dead-letter

**Durable execution**
- Temporal Retry Policies — https://docs.temporal.io/encyclopedia/retry-policies
- Temporal Activity Definition — Idempotency — https://docs.temporal.io/activity-definition
- Temporal Activities overview — https://docs.temporal.io/activities
- Temporal, detecting Activity failures / heartbeats — https://docs.temporal.io/encyclopedia/detecting-activity-failures
- AWS Step Functions error handling (Retry/Catch, JitterStrategy) — https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html
- Restate — https://restate.dev/ ; "Solving durable execution's immutability problem" — https://www.restate.dev/blog/solving-durable-executions-immutability-problem

**Harvesting standards (government/scientific)**
- OAI-PMH Implementation Guidelines for Harvester Implementers, 2005-01-19 — https://www.openarchives.org/OAI/2.0/guidelines-harvester.htm
- OAI-PMH protocol v2.0 — https://www.openarchives.org/OAI/openarchivesprotocol.html

**War stories**
- GSA/data.gov wiki, "Stuck Harvest Fix" — https://github.com/GSA/data.gov/wiki/Stuck-Harvest-Fix
- ckanext-harvest #112, "Catch more errors to avoid limbo states" — https://github.com/ckan/ckanext-harvest/issues/112
- ckanext-dcat #147, RDF job never ends on gather-stage exception — https://github.com/ckan/ckanext-dcat/issues/147
- GSA/data.gov #3944, duplicate dataset from same harvest object — https://github.com/GSA/data.gov/issues/3944

**Silent-failure / soft-404 (secondary, vendor-authored — corroborative only)**
- Web Scraper, "200 OK but no data" — https://webscraper.io/blog/200-ok-but-no-data-diagnosing-incorrect-page-responses
- Context.dev, "Debugging Web Scraper Failures: 403s, 429s, Timeouts, and Empty Results" — https://www.context.dev/blog/debugging-web-scraper-failures
- Context.dev, "Scraper Monitoring in Production" — https://www.context.dev/blog/scraper-monitoring-in-production
- Ficstar, "Why Web Scraping Fails Silently", 2026-07 — https://ficstar.medium.com/why-web-scraping-fails-silently-and-why-thats-worse-than-a-crash-2b4a8f615ccf
- Apify, "Why you need to monitor long-running, large-scale scraping projects" — https://dev.to/apify/why-you-need-to-monitor-long-running-large-scale-scraping-projects-5flj
- Crawlex, "Scraping observability" — https://blog.crawlex.net/blog/scraping-observability/
- Google Search Central, HTTP and network errors (soft 404) — https://developers.google.com/search/docs/crawling-indexing/http-network-errors
- Manku, Jain, Das Sarma, "Detecting Near-Duplicates for Web Crawling", WWW 2007 — https://dl.acm.org/doi/10.1145/1242572.1242592



---

# Change Detection: Did the Source's Data Actually Change, or Is It Just Noise?

Research memo for a long-running, unattended, multi-source harvesting system (government + scientific sites and APIs).
Date of research: 2026-09-06. No single recommendation is made; trade-offs and disagreements are shown with attribution.

---

## 0. The layered model everyone converges on

Practitioner and academic sources independently describe the same escalation ladder, cheapest first:

1. **Ask the server if anything changed** (conditional HTTP, `updated_since` cursors, OAI-PMH `from`, feed datestamps, changelogs). Cost: one request, often no body.
2. **Cheap whole-body fingerprint** (hash of raw bytes). Cost: full transfer, but trivial CPU. *Extremely noisy on HTML.*
3. **Canonicalize, then fingerprint** (strip boilerplate/dynamic markup, normalize, hash). Cost: parse + rules.
4. **Extract the fields you actually want, hash those** (field-level fingerprints / row hashes). Cost: full extraction, but highest signal.
5. **Near-duplicate similarity** (SimHash / MinHash) when you need "how much did it change" rather than "did it change".
6. **Semantic/structured diff and versioning** (JSON Patch, SCD Type 2, snapshots) to record *what* changed.
7. **Separately**: detect that the *site structure* changed (scraper breakage) as distinct from the data changing.

Layers 1–2 have known false *negatives*; layers 2–3 have known false *positives*. This tension is the core of the topic.

---

## 1. Cheap server-side signals

### 1.1 HTTP conditional requests (RFC 9110)

RFC 9110 (HTTP Semantics, June 2022, Internet Standard, STD 97) is the current normative source; it obsoletes RFC 7230–7235 and the older RFC 2616.

**Validators — §8.8.1.** RFC 9110 defines:
- *Strong validator*: "representation metadata that changes value whenever the representation data changes."
- *Weak validator*: "representation metadata that might not change for every modification to the representation data."

**`Last-Modified` — §8.8.2.** "represents the date and time at which the origin server believes the representation was last modified." The RFC's own worked example of why this is weak: *"if a document changed twice in one second, both changes would result in the same Last-Modified value, but the representation data might be different."* HTTP date values have **one-second resolution**. Consequence for a harvester: a source that regenerates a page multiple times per second, or that touches mtime without changing content (very common with static-site regeneration and rsync deploys), makes `Last-Modified` both false-positive and false-negative prone.

**`ETag` — §8.8.3.** "an opaque validator for distinguishing between multiple representations of the same resource over time." May be strong (`"abc"`) or weak (`W/"abc"`).

**`If-None-Match` — §13.1.2.**
- "A recipient MUST use the weak comparison function when comparing entity-tags for If-None-Match."
- For GET/HEAD, when the condition is false, the origin server "MUST generate a 304 (Not Modified) response"; for other methods, 412 (Precondition Failed).

**`If-Modified-Since` — §13.1.3.** A server "MUST ignore the If-Modified-Since header field if the request contains an If-None-Match header field." I.e. **If-None-Match takes precedence**; sending both is safe and is the standard belt-and-braces pattern, but only the ETag will actually be honored by a conformant server.

**Precedence of preconditions — §13.2.2** defines the full evaluation order when multiple precondition fields are present.

**Practical gotcha, documented by MDN:** *"Because a change of content encoding requires a change to an ETag, some servers modify ETags when compressing responses from an origin server (reverse proxies, for example). Apache Server appends the name of the compression method (`-gzip`) to ETags by default, but this is configurable using the `DeflateAlterETag` directive."* Effect: the same resource fetched with different `Accept-Encoding`, or through different CDN/origin nodes, can return **different ETags for identical bytes** → spurious "changed" verdicts. Inode-based ETags (Apache's historical `FileETag INode MTime Size`) break the same way across a load-balanced fleet.

**How much can you actually rely on validators being present?** HTTP Archive Web Almanac 2021 (Caching chapter, crawl of Dec 15 2021):
- `Last-Modified`: **70.5%** of responses (desktop and mobile)
- `ETag`: **46.5%**
- **both**: 41.6%
- **neither**: **24.5%**
- ~0.9% of mobile / 0.7% of desktop responses had an **invalid `Last-Modified` date format**.
- Trend noted: ETag adoption rising slightly year over year, `Last-Modified` down ~1.5%.

So roughly a quarter of the web gives you no validator at all, and that share is likely *worse* for the dynamically-generated government application pages (search result pages, ASP.NET/ColdFusion portals, ArcGIS endpoints) that a gov-focused harvester hits. Treat "no validator" as the default assumption per-source and *measure* it during onboarding rather than assuming.

**Correctness caution:** a 304 is a claim by the origin, not a proof. Reverse proxies, WAFs and "smart" CDNs in front of government sites sometimes serve 304 from a stale edge, or return 200 with identical content forever. Cheap validation: periodically ignore your cached validator and do a full fetch + content hash to confirm the 304s were truthful (a "trust-but-verify" full refresh on some cadence, e.g. weekly/monthly).

### 1.2 HEAD requests and `Content-Length`

`HEAD` is the cheapest possible probe and RFC 9110 requires that the header fields be identical to what a GET would return. In practice:

- `Content-Length` is **absent whenever the response is chunked** (`Transfer-Encoding: chunked`), which is the norm for dynamically generated pages — exactly the pages you care about. MDN's `Transfer-Encoding` page documents that chunked responses omit `Content-Length`.
- Servers are inconsistent about `Content-Length` on HEAD for chunked responses (see e.g. line/armeria issue #4509, where a `0` `Content-Length` was being emitted on HEAD for a chunked response) — so a naive harvester can read "0 bytes" and conclude "empty/unchanged".
- `Content-Length` reflects the **encoded** (possibly gzip/br) length, so recompression at a CDN changes it without content changing.
- Length is a terrible change signal anyway: a page whose only change is a swapped date string, a flipped status word, or a reordered table has the *same* length surprisingly often. It is a usable *negative* filter (length changed by >X% ⇒ definitely investigate) but never a reliable *positive* one.
- Many origins are slower on HEAD than GET, or don't support it at all, or return 405. Some sites treat HEAD as bot behavior.

Verdict from the sources: HEAD/`Content-Length` is a supplementary heuristic, not a primary detector.

### 1.3 Sitemap `<lastmod>` — and why it is untrustworthy

Google's own documentation (developers.google.com, *Build and submit a sitemap*) is explicit:

- **"Google ignores `<priority>` and `<changefreq>` values."**
- **"Google uses the `<lastmod>` value if it's consistently and verifiably (for example by comparing to the last modification of the page) accurate."**

That is a conditional trust statement, and the industry reporting around it (Search Engine Roundtable, *Google Either Trusts Or Doesn't Trust Your Sitemap's Lastmod Date*, and follow-ups citing Google's Gary Illyes) frames it as effectively **binary and site-wide**: Google evaluates whether *your* lastmod values are honest and then either believes them or discards them wholesale. Google's guidance further says lastmod should reflect *significant* updates (main content, structured data, links), not a copyright-year bump.

**Implication for a harvester:** `<lastmod>` is a *hint with an unknown, per-site error rate in both directions*. Two independent failure modes seen in the wild:
- CMSs that stamp `lastmod = build time` for every URL on every deploy ⇒ 100% false positives, whole sitemap "changes" nightly.
- CMSs that never update lastmod after initial publication ⇒ silent false negatives.

The defensible engineering pattern is: use `<lastmod>` to *prioritize* the crawl queue, never to *skip* a fetch permanently; and score each source's lastmod reliability empirically (fraction of lastmod-changed URLs that turn out to have a changed content hash, and vice versa) so you know which sitemaps to believe.

### 1.4 RSS/Atom

RFC 4287 (Atom Syndication Format, Dec 2005) §4.2.15 defines `atom:updated` as "a Date construct indicating the most recent instant in time when an entry or feed was modified **in a way the publisher considers significant**," and adds the killer caveat: **"Therefore, not all modifications necessarily result in a changed atom:updated value."** §4.2.9 `atom:published` is "an instant in time associated with an event early in the life cycle of the entry."

So Atom's own spec tells you `updated` is a *publisher-discretion* signal, not a change detector. RSS 2.0's `pubDate`/`lastBuildDate` are even weaker (no normative semantics for silent edits).

Feeds do compose well with conditional GET, though: fetching a feed with `If-None-Match`/`If-Modified-Since` is the single cheapest "did anything happen in this section of the site" probe available, and feed servers are unusually good about supporting 304 because feed readers hammer them. Feeds are also usually **truncated** (last N items) — if your polling interval exceeds the window, you silently lose items. Compute the feed's item-turnover rate and set polling to a safe fraction of it.

### 1.5 OAI-PMH datestamps (the scientific/repository case)

The OAI-PMH v2 spec (openarchives.org) is the best-specified incremental-harvest protocol in this space:

- A record's **datestamp** reflects "the most recent date and time of the creation, modification, or deletion" of that record.
- **Granularity** is either `YYYY-MM-DD` (mandatory for all repositories) or `YYYY-MM-DDThh:mm:ssZ` (optional). The `Identify` response declares which. **Day-granularity repositories force you to re-harvest whole days** and dedupe locally.
- `from` / `until` on `ListRecords`/`ListIdentifiers` are **inclusive**: "`from` specifies a bound that must be interpreted as 'greater than or equal to', `until` … 'less than or equal to'." So consecutive windows overlap by one tick — you must dedupe.
- **Deleted records**: repositories declare `deletedRecord` = `no`, `transient`, or `persistent`. The spec's own warning: **"If a repository does not keep track of deletions then such records will simply vanish from responses and there will be no way for a harvester to discover deletions through continued incremental harvesting."** With `deletedRecord=no`, incremental harvesting *cannot* detect retractions — you need periodic full re-harvests to find disappearances.
- `resumptionToken` is **opaque** (do not parse), may carry `expirationDate`, `completeListSize`, `cursor`. Tokens expiring mid-harvest is a common cause of partial harvests that look like "no changes".

### 1.6 API `updated_since` cursors — the ties/skew problem

The generic pattern (`?updated_since=T`, sort by `updated_at`, save max seen as new cursor) has well-documented failure modes.

Airbyte's incremental-sync docs describe the mechanics and the mitigations:
- Semantics: "records whose `updated_at` value is less than or equal than that cursor value have been synced already, and … the next sync should only export records whose `updated_at` value is greater than the cursor value."
- **Lookback window**: exists precisely because APIs update records but may only let you filter by a coarse or wrong timestamp; you subtract a duration from the last cursor and re-scan. Airbyte frames it as an explicit trade-off between "data consistency and the amount of synced records."
- **Cursor granularity / step slicing**: intervals are sized so "the start of an interval doesn't overlap with the end of the last one," with ISO-8601 step durations (e.g. `P10D`) to bound page counts.
- Documented caveat: "If the last record read has a datetime earlier than the end time of the stream interval, the end time of the interval will be stored in the state" — i.e. state can advance past records you never saw.

The generic hazards to design against (widely reported; Knit's *API Pagination Stability* writeup is a decent practitioner summary):
- **Timestamp ties** at the boundary: if 5,000 rows share `updated_at`, a strict `>` cursor skips some and a `>=` cursor loops forever. Fix: composite cursor `(updated_at, id)` with a keyset comparison.
- **Clock skew / write-then-timestamp races**: a row written at T but committed at T+ε is invisible to a cursor already at T. Fix: lag the high-water mark behind wall-clock by a safety margin.
- **Mutation during pagination**: offset pagination shuffles rows under you; cursor/keyset pagination is stable. Prefer cursor.
- **Server sets `updated_at` on no-op writes** ⇒ false positives, whole tables "change" nightly.

**Concrete government example — Regulations.gov API v4** (open.gsa.gov/api/regulationsgov): there is a hard **5,000-record pagination ceiling** (20 pages × 250). The *documented* workaround is exactly the cursor pattern: sort by `lastModifiedDate`, take the last record's `lastModifiedDate` off the final page, and re-query with `filter[lastModifiedDate][ge]=…`. The docs also flag a timezone trap — the API's `lastModifiedDate` filter is interpreted in **Eastern time** while the returned values are UTC (their example: `2020-08-10T15:58:52Z` "translates to `2020-08-10 11:58:52` in Eastern time"). Rate limits per api.data.gov; the commenting endpoint is 50 req/min, 500 req/hr.

**Concrete government example — data.gov harvesting.** GSA's `datagov-harvesting-logic` package computes a per-dataset comparison against the existing catalog and classifies each source record as **create / update / delete** by comparing DCAT-US records field-by-field (their test fixture `dcatus_compare.json` shows exactly this: added identifier ⇒ create, missing identifier ⇒ delete, changed `modified` field ⇒ update). Pipeline is extract → jsonschema (draft 2020-12) validate → load into CKAN. Note this is *record diffing*, not cryptographic hashing — worth knowing if you plan to mirror data.gov, because their update signal is only as good as upstream agencies' DCAT `modified` fields.

### 1.7 Database changelogs / CDC concepts

Gunnar Morling's *Five Advantages of Log-Based Change Data Capture* (Debezium blog, 2018-07-19) is the canonical statement of why polling loses information, and every point transfers directly to scraped sources:

1. **All data changes are captured** — "By reading the database's log, you get the complete list of all data changes in their exact order." Polling misses **intermediate states** between polls.
2. **Low delays without CPU overhead** — no repeated polling queries.
3. **No impact on the data model** — "Polling requires some indicator to identify those records that have been changed since the last poll," i.e. an `updated_at` column you may not control.
4. **Can capture deletes** — "Polling will not allow you to identify any records that have been deleted since the last poll."
5. **Can capture old record state and metadata** — "Log-based CDC can provide the old record state for update and delete events. Whereas with polling, you'll only get the current row state."

For scraped sources you almost never get a log, so **you are structurally stuck with polling and must accept (1) and (4) as permanent limitations**: you will miss intermediate revisions between polls, and deletions/retractions require an explicit reconciliation pass (full re-enumeration, or a "not seen in N runs ⇒ tombstone" rule). This is the same problem OAI-PMH `deletedRecord=no` creates. Design the tombstone rule up front; it is the single most commonly omitted piece.

---

## 2. Content hashing and its failure mode

### 2.1 Why hashing raw HTML produces constant false positives

Hashing the raw response body (MD5/SHA-1/SHA-256/xxhash) is the obvious approach and it is *correct* — it just answers the wrong question. The bytes of a modern page change on essentially every request for reasons unrelated to the data:

- **CSRF / anti-forgery tokens** (`__RequestVerificationToken`, `authenticity_token`) — rotate per request. Endemic on government forms and search portals.
- **Session IDs** embedded in URLs (`;jsessionid=…`, `?sid=…`) — classic on older Java/ColdFusion gov systems.
- **Timestamps rendered into the page** ("Page generated 2026-09-06 14:22:31", "Data as of…", "Last accessed").
- **View / hit counters, "N users online".**
- **Ad slots, ad-call cache-busters, tracking pixels with random `cb=` params.**
- **Rotating banners / carousels / "featured item" widgets** that pick randomly per render.
- **A/B test markup** — experiment IDs, variant class names, feature-flag payloads.
- **CSRF-free but volatile framework state** — `__VIEWSTATE` (ASP.NET WebForms, ubiquitous on state government sites) is a base64 blob that changes constantly; Next.js `__NEXT_DATA__` build IDs; Webpack chunk hashes in `<script src>`.
- **Whitespace / attribute-order / minification differences** across load-balanced backends or after a deploy.
- **Server-injected node identifiers** (`X-Served-By` echoed into comments, edge node names).
- **CDN/WAF injected scripts** and challenge tokens.

Empirically, Fetterly et al. quantified how much of "change" is pure markup noise: in their 151M-page, 11-week study, **9.2%** of week-over-week page pairs "differed only in markup" — the MD5 checksum changed but *every* 5-word shingle matched. Combined with 65.2% with no change at all, **74.4% of pairs had zero or markup-only change**. So on their corpus a raw checksum detector generated false positives on ~9 pages in 100 per week *even in 2003*, before A/B testing, CSRF-everywhere, and per-request build hashes. The modern rate on JS-heavy government portals is far higher; several practitioners report raw-HTML hashing being effectively useless (100% change rate) on such pages.

Adar et al. (WSDM 2009) supply the complementary structural number: **DOM elements survive 99.3% at 2 hours and 84.3% at 5 weeks**, and they explicitly note navigation/persistent elements are highly stable while **ads change rapidly** — i.e. the volatility is concentrated in identifiable regions, which is exactly what makes region-based stripping work.

### 2.2 What people do instead

Four distinct strategies, not mutually exclusive:

**(a) Regex/rule-based scrubbing before hashing.** Cheap, brittle, per-site. urlwatch's documented pipeline is the canonical shape:
```yaml
url: https://example.com/page.html
filter:
  - re.sub: '\s*href="[^"]*"'
  - re.sub:
      pattern: 'csrf_token=[^&]*'
      repl: ''
  - html2text
  - strip
```
urlwatch also ships `sha1sum` ("Calculate the SHA-1 checksum of the content") as a terminal filter, plus `css`, `xpath`, `grep`/`grepi`, `re.findall`, `sort`, `reverse`, `remove-duplicate-lines`, `jq`, `pdf2text`, `ocr`, `shellpipe`. Cost: a hand-written rule set per source, which does not scale to "many separate scrapers" without discipline, and silently rots when the site changes.

**(b) Extract-then-hash (field-level fingerprints).** Parse to a typed record, hash only the fields you care about. This is what data.gov's harvester does (field-by-field DCAT comparison) and what dbt's `check` strategy does (`check_cols`). It gives you the highest signal-to-noise ratio and, crucially, **change attribution** — you know *which field* moved. Cost: you must have a working extractor, so it can't run before extraction, and it inherits extractor breakage (see §6).

**(c) Boilerplate removal / main-content extraction, then hash** — §3.

**(d) Similarity rather than equality** — §4.

---

## 3. Canonicalization and noise stripping

### 3.1 Boilerplate removal tools

The main-content extraction lineage: **boilerpipe** (Kohlschütter et al., WSDM 2010, shallow text features; `boilerpy3` is the maintained Python port), **jusText** (Pomikálek, block classification by link density/stopword density), **readability** (Arc90 heuristics; `readability-lxml` in Python, `@mozilla/readability` in JS), **dragnet** (Peters & Lecocq, ML on block features — *note: effectively unmaintained, last meaningful release years ago; treat as stale*), **trafilatura** (Barbaresi, rule-based + fallbacks, actively maintained), **resiliparse** (ChatNoir/Webis, speed-focused), **magic-html**, **news-please**, **goose3**, **newspaper3k/4k**.

**Trafilatura's own published benchmark** (docs, evaluation run dated 2026-08-04, 990 documents, 2,951 text and 2,966 boilerplate segments, Python 3.13):

| Tool | Precision | Recall | Accuracy | F1 | Rel. time |
|---|---|---|---|---|---|
| trafilatura 2.2.0 (standard) | 0.906 | 0.943 | 0.923 | **0.924** | 3.2x |
| magic-html 0.1.8 | 0.887 | 0.891 | 0.889 | 0.889 | 3.5x |
| jusText 3.0.2 (custom) | 0.864 | 0.859 | 0.862 | 0.862 | 2.3x |
| news-please 1.6.16 | 0.932 | 0.758 | 0.852 | 0.836 | 20.5x |
| readability-lxml 0.8.4.1 | 0.898 | 0.764 | 0.839 | 0.826 | 2.6x |
| resiliparse 1.0.9 | 0.705 | **0.955** | 0.778 | 0.811 | **0.3x** |
| goose3 3.1.22 | 0.936 | 0.714 | 0.833 | 0.810 | 10.2x |
| boilerpy3 1.0.7 | 0.818 | 0.796 | 0.810 | 0.807 | 1.6x |
| newspaper4k 0.9.6 | 0.878 | 0.736 | 0.817 | 0.801 | 6.6x |
| inscriptis 2.7.4 | 0.534 | 0.991 | 0.564 | 0.694 | 1.1x |
| beautifulsoup4 4.15.0 | 0.532 | 0.980 | 0.561 | 0.690 | 2.1x |
| html2text 2025.4.15 | 0.525 | 0.900 | 0.544 | 0.663 | 2.8x |

Author's caveat: "boilerpy3 and newspaper4k modules do not work without errors on every HTML file," attributed to "malformed HTML, encoding or parsing bugs."

> **DISAGREEMENT / bias flag.** This is the *tool author's own* benchmark on a dataset he curated — trafilatura winning it is weak evidence. Independent evaluations exist and rank differently: Bevendorff et al., *An Empirical Comparison of Web Content Extraction Algorithms* (SIGIR 2023, ACM DOI 10.1145/3539618.3591920 — the PDF 403s for automated fetch, cite from the DOI), and a multilingual evaluation (ACM 10.1145/3726302.3730234, 2025), plus a Sandia National Laboratories evaluation (SAND2024-10208, Aug 2024, osti.gov/servlets/purl/2429881 — robots-blocked for automated fetch). **Do not treat any single leaderboard as settled; run your own eval on a sample of *your* government/scientific pages**, which look nothing like the news-article corpora these benchmarks are built from. Government pages are frequently tables, forms, PDF-wrappers, and dataset landing pages — a class where "main content extraction" trained on news articles performs notably worse.

**Critical caveat for change detection specifically:** extractors are *heuristic and non-deterministic across versions*. If trafilatura 2.2.0 and 2.3.0 pick a different `<div>`, every page in your corpus "changes" on upgrade. Pin extractor versions, and treat an extractor upgrade as a deliberate full-corpus re-baseline event, not a routine dependency bump.

### 3.2 DOM normalization

Independent of boilerplate removal, canonicalize the parse tree before fingerprinting:
- Serialize through a single parser (lxml / html5ever) so attribute order and quoting are normalized.
- Sort attributes; drop `style`, `class`, `id` when they carry build hashes; drop `<script>`, `<style>`, `<noscript>`, comments.
- Collapse whitespace runs; normalize Unicode (NFC) and entity encoding; normalize `&nbsp;` → space (a real reported source of false positives — changedetection.io discussion #577 is specifically about ignoring `&nbsp;` changes).
- Strip or canonicalize URL query params that are cache-busters.
- Normalize numeric/date formatting *only* if you're hashing text, not if you're hashing fields.

### 3.3 Extract only the fields you care about and hash those

The strongest signal-to-noise option. Shape:
```
record_hash   = H(canonical_json(sorted(fields_of_interest)))
field_hashes  = { field: H(normalize(value)) for field in fields }
```
Storing per-field hashes (not just a record hash) buys you: change attribution ("the effective date moved, the title didn't"), targeted alerting, and cheap detection of *suspicious* changes (e.g. every field on every record changed at once ⇒ almost certainly a layout change, not a data change — see §6).

Pierluigi Vinciguerra (The Web Scraping Club, 2023-10-15) argues the adjacent point: prefer the site's *own* structured payloads where they exist — embedded `__NEXT_DATA__`, `__PRELOADED_STATE__`, or an underlying JSON API — because those are far more stable than rendered HTML, and write "generic selectors … so that everything happening in the higher-level nodes doesn't impact the success."

### 3.4 Content-defined chunking (CDC)

For large documents (multi-MB XML dumps, PDF corpora, bulk data files) where you want *both* storage dedup and localized change detection, CDC beats fixed-size chunking. Xia et al., **FastCDC** (USENIX ATC '16, Denver, Jun 22–24 2016):
- CDC solves the **boundary-shift problem**: with fixed-size chunking, inserting bytes at the start of a file shifts every boundary and destroys all dedup. Content-defined boundaries survive insertions. CDC detects "10–20% more redundancy than FSC."
- Rabin CDC computes a polynomial fingerprint over a rolling **48-byte** window; expensive byte-by-byte.
- FastCDC is ~**10× faster than Rabin CDC** and ~**3× faster** than Gear/AE, with "nearly the same deduplication ratio" (within ±0.1–1.4%; e.g. RDB dataset: Rabin 95.53% vs FastCDC 95.50%).
- Typical parameters: expected chunk 4/8/16 KB, min/max 2 KB/64 KB; cut-point skipping raises the min to 4–8 KB. "Normalized chunking" tightens the size distribution.

Practical use here: chunk each large artifact, store chunk hashes, and a re-fetch tells you *which regions* changed without diffing the whole file. This is also how you make "store every version" affordable (§5.3).

---

## 4. Near-duplicate detection at scale

When you need *degree* of change ("is this materially different?") rather than binary equality.

### 4.1 SimHash (Charikar 2002; Manku et al. 2007)

Charikar, *Similarity Estimation Techniques from Rounding Algorithms* (STOC 2002) introduced the random-hyperplane LSH whose fingerprint is now called simhash.

**Manku, Jain, Das Sarma, "Detecting Near-Duplicates for Web Crawling", WWW 2007 (Banff, May 8–12 2007), Google.** The load-bearing operational paper. Concrete parameters:
- **64-bit** fingerprints.
- **Hamming distance threshold k = 3.**
- Corpus: **8 billion (2^34) web pages.**
- Justification for k=3 (their Figure 1, precision/recall swept over k = 1..10): *"Choosing k = 3 is reasonable because both precision and recall are near 0.75."* — i.e. **~75% precision and ~75% recall.** That number is worth internalizing: even Google's tuned near-dup detector is only ~75/75 at its chosen operating point. Anyone quoting "3-bit Hamming on 64-bit simhash" as a precise rule should be shown this figure.
- **Hamming Distance Problem algorithm**: build *t* sorted tables T₁…T_t, each with a bit permutation π_i and a prefix length p_i; probe candidates matching the top p_i bits of π_i(F), then verify ≤ k differing bits. Their Example 3.1 gives concrete designs: 20 tables with p ∈ [31,33]; 16 tables with p = 28; 10 tables with p ∈ [25,26]; 4 tables with p = 16 — the classic time/space trade-off dial.
- **vs Broder shingling**: Broder's fingerprints need "24 bytes per fingerprint" vs simhash's 8 bytes — a **3× storage reduction**, which is why Google moved to simhash for web-scale.

Caveat the paper itself implies and practitioners repeat: simhash is tuned for *long* documents. On short texts (a table row, a headline, a status field) 64-bit simhash is unstable — small edits blow past 3 bits. For short records, use exact field hashes or edit distance instead.

Implementations: `scrapinghub/python-simhash` (C extension), `leonsim/simhash`, `seomoz/simhash-py`.

### 4.2 MinHash / shingling / LSH (Broder)

Broder, *On the Resemblance and Containment of Documents* (SEQUENCES '97, Compression and Complexity of Sequences, Positano, Jun 1997):
- Documents → sets of **w-shingles** (contiguous w-word subsequences).
- **Resemblance** r(A,B) = |A∩B| / |A∪B| (Jaccard); **containment** c(A,B) = |A∩B| / |A|.
- Estimated by min-wise independent permutations: keep the *s* smallest hashed shingle values as a **sketch**; the fraction of matching minima estimates resemblance unbiasedly.
- Deployed at AltaVista over a ~30M-document crawl; typical thresholds used in that line of work put "near-duplicate" at resemblance ≳ 0.9.

Broder, Glassman, Manasse, Zweig, *Syntactic Clustering of the Web* (WWW6, 1997) is the companion systems paper. Later refinement: Henzinger, *Finding Near-Duplicate Web Pages: A Large-Scale Evaluation of Algorithms* (SIGIR 2006) — compares Broder's shingling against Charikar's simhash on 1.6B pages and reports **neither dominates**: shingling has higher precision on pages from the *same* site (where boilerplate dominates), simhash better across sites; Henzinger proposed a combined scheme. This is a genuine, well-documented disagreement in the literature about which fingerprint is better, and it is *site-structure dependent* — directly relevant if your corpus is many pages from a handful of government domains, which is the regime where shingling reportedly does better.

**Practical parameters seen in the wild:** 5-word shingles (Fetterly et al. used exactly this); sketches of 64–200 minima; MinHash-LSH banding with b bands × r rows tuned so the S-curve threshold sits at your target Jaccard. `datasketch` (Python) is the common library.

### 4.3 Choosing thresholds

There is no universal number. What the sources give you:
- Manku et al.: k=3 on 64-bit simhash ⇒ ~0.75/0.75 precision/recall at 8B-page scale.
- Broder-lineage work: resemblance ≥ 0.9 as "near-duplicate".
- Adar et al. give the distribution you'd be thresholding against: mean inter-version **Dice = 0.794** for pages that changed at all; **median 0.950**; at 2-minute intervals Dice 0.937 mean / 0.985 median. So "most changes are small" is the empirical baseline — a similarity threshold set at 0.95 will fire on roughly half of all real changes and miss the other half. Set it from *your* measured distribution per source, not from a paper.

---

## 5. Semantic / structured diffing and versioning

### 5.1 JSON diff / patch

**RFC 6902, JSON Patch** (April 2013, Proposed Standard, media type `application/json-patch+json`): operations `add`, `remove`, `replace`, `move`, `copy`, `test`, with targets addressed by **JSON Pointer (RFC 6901)**. "Operations are applied sequentially in the order they appear in the array"; over HTTP PATCH the application is atomic — if any op fails, no changes are made. **RFC 7386, JSON Merge Patch** is the simpler alternative (null means delete; cannot express array element ops).

For change detection this gives you a *canonical, storable, machine-readable representation of what changed* — much better than a text diff for downstream alerting ("alert me when `/status` changes but not when `/view_count` changes"). Libraries: `jsonpatch`/`jsondiffpatch`, `deepdiff` (Python), `json-diff` (Node).

Prerequisite: **canonical JSON serialization** (sorted keys, stable number formatting, stable array ordering where order is not semantic) — otherwise you diff serialization noise. Array reordering is the classic false positive; if the array is a *set*, sort it by a stable key before diffing.

### 5.2 Row-level hashing in a warehouse / SCD Type 2 / dbt snapshots

dbt's `snapshot` implements **Type-2 slowly changing dimensions** and is the most-documented off-the-shelf version of "detect and record row changes":

- **`timestamp` strategy** (dbt's recommended default): uses an `updated_at` column; if newer than the snapshot, invalidate old row and insert new. dbt's stated reasons for preferring it: only one column to track, and it "automatically handles new or removed columns in the source table" — "less prone to errors when the table schema evolves."
- **`check` strategy**: compares an explicit `check_cols` list. dbt: "The `check` strategy is useful for tables which **do not have a reliable `updated_at` column**."
- dbt's explicit warning on `check_cols='all'`: *"It is better to explicitly enumerate the columns that you want to check. Consider using a surrogate key to condense many columns into a single column."* — i.e. the vendor itself tells you a hash-everything strategy is both slow and noisy. (The "surrogate key" advice *is* row-level hashing: `md5(concat_ws('|', col1, col2, …))`.)
- Meta-columns added: `dbt_scd_id`, `dbt_updated_at`, `dbt_valid_from`, `dbt_valid_to` (NULL for current), plus `dbt_is_deleted` when `hard_deletes='new_record'`.
- **`hard_deletes`** (dbt ≥1.9): `ignore` (default), `invalidate` (set `dbt_valid_to`), `new_record` (insert a tombstone row with `dbt_is_deleted=True`). This is your deletion-detection mechanism, and the default is to *not* detect deletions.
- Cadence: *"Snapshots are a batch-based approach to change data capture. The `dbt snapshot` command must be run on a schedule … snapshots are intended to be run between hourly and daily."*
- Snapshots **ignore `--full-refresh`** by design so history is preserved.

**Fundamental limitation dbt shares with all polling**: batch snapshots capture *state at snapshot time*, not the change sequence. Anything that changed and changed back between runs is invisible — the same point Morling makes about polling vs log-based CDC (§1.7).

### 5.3 Store every version vs store only diffs

Three positions in the sources, all defensible:

- **Store every version** (WARC/wget, git-scraping, Wayback). Simple, replayable, survives extractor changes because you can re-extract from raw. Expensive — but far less so with CDC/dedup (§3.4) or content-addressed storage, since unchanged bytes are stored once. Simon Willison's **git scraping** (simonwillison.net, 2020-10-09) is the minimal-effort version: a GitHub Actions cron (his example: minutes 6, 26, 46) fetches, normalizes via `jq`, and commits — **git itself is the change detector, because identical content produces an identical blob hash and therefore no commit**. Examples given: California wildfire incidents, SF's 190k-tree dataset, PG&E outages, FARA registrations. Follow-up tool `git-history` (2021-12-07) turns the commit log into a queryable change history. This is genuinely the cheapest credible "store every version + detect change" system for small-to-medium JSON/CSV endpoints, and it composes with the fact that government open-data endpoints are often exactly that.
- **Store only diffs / SCD-2 rows.** Compact, queryable ("what did this record look like on 2025-03-01?"), but you cannot re-derive fields you didn't extract, and a diff chain is fragile if any link is lost.
- **Hybrid (most common in production):** raw WARC/blob archive at low resolution (or content-addressed with dedup) + SCD-2 field-level history at full resolution. Costs more but is the only shape that survives both extractor bugs and schema evolution.

**Wayback/CDX as a free change-history source.** The CDX Server API (github.com/internetarchive/wayback, wayback-cdx-server README; docs at archive.org/services/docs/api/wayback-cdx-server.html) returns per-capture `["urlkey","timestamp","original","mimetype","statuscode","digest","length"]`. The **`digest`** is a content hash, and **`collapse=digest`** gives you "only unique captures by digest" — with the documented caveat that *"only adjacent digest are collapsed, duplicates elsewhere in the cdx are not affected."* So a single CDX query can hand you an approximate historical change timeline for a URL for free — useful for **estimating a source's change rate before you build a scraper for it**, and for backfilling history you missed. Filters: `filter=[!]field:regex`; pagination via `page`/`showNumPages`/`pageSize`; default server cap **150,000 results**, `limit=N` / `limit=-N`. Caveat: Wayback's capture cadence is itself irregular, so absence of a digest change is weak evidence of no change. (There is an open Internet Archive forum thread — archive.org/post/1009990 — questioning whether CDX digests accurately capture duplicates.)

---

## 6. Detecting that the SITE'S STRUCTURE changed (selector drift), not the data

This is a *different problem* from content change and requires different signals. A layout change looks, to a naive content hasher, like a giant data change; and to a naive extractor, like every record suddenly having null fields.

**The key insight all practitioner sources converge on: don't diff the page, monitor the extraction's statistical shape.**

Signals, roughly in order of usefulness (synthesizing Zyte's **Spidermon** docs and Yahia Bakour's *Scraper Monitoring in Production*, context.dev, 2026-08-16):

1. **Field coverage / null rate per field.** Bakour's concrete example: *"a jump in null prices from 2 percent to 70 percent usually indicates extraction failure."* Spidermon ships "required fields coverage" monitors for exactly this. This is the single highest-value breakage signal.
2. **Item count deltas vs a baseline or vs the previous job.** "Compare each run with an expected minimum row count or a recent baseline, and alert when the count falls outside a reasonable range." Spidermon supports comparison between spider executions.
3. **Schema validation on every item.** Spidermon "can check the output data produced by Scrapy … and verify it against a schema or model that defines the expected structure, data types and value restrictions," via `jsonschema` (or schematics). Fail-fast at the item level, count failures, alert on rate.
4. **Selector presence / match-count monitoring.** "Watch whether required selectors disappear" or whether "match count changes beyond a threshold." Monitoring *DOM shape and targeted selectors* rather than full-page diffs is explicitly recommended to reduce false positives.
5. **Canary / golden pages.** *"Choose a stable page with predictable content, run it on a schedule through the same browser, proxy, and parsing path as production jobs, and verify a small set of expected values."* This isolates parser breakage from source-data change: if the golden page's known-fixed values stop parsing, it's you, not them.
6. **HTTP 200 is not success.** Bakour: scrapers may "return an empty array, a login page, or records with required fields set to null" despite HTTP 200 — soft 404s, WAF interstitials, and "your session expired" pages all return 200. Validate content, never status alone.
7. **"Everything changed at once" heuristic.** If >X% of records in a source change in a single run, the prior should be *layout change / template change*, not a mass data update. Route to human review rather than to the change feed. (This is the field-level-hash payoff from §3.3.)
8. **Page-structure fingerprint.** Hash the *tag skeleton* (element names + nesting, attributes and text stripped) or the set of CSS paths you depend on. Structure hash changed + content hash changed ⇒ suspect breakage; content hash changed + structure hash stable ⇒ likely a real data change. Adar et al.'s DOM-survival numbers (99.3% at 2h, 84.3% at 5 weeks) say a structure fingerprint is a *stable* baseline over weeks, which is what makes this work.
9. **Tech-stack change detection.** Vinciguerra recommends Wappalyzer-style stack fingerprinting to catch a site swapping in a new anti-bot vendor or framework (he cites ~$450/mo for API access) — a leading indicator of impending breakage.
10. **Warehouse-side schema-drift assertions.** Great Expectations (`expect_table_columns_to_match_set`, `expect_column_values_to_not_be_null`, `expect_column_values_to_be_between`), dbt tests, or Databricks pipeline `expectations` — a second net downstream of the scraper.

**Structural alternatives that dodge selector drift entirely:**
- Use the site's own JSON (`__NEXT_DATA__`, `__PRELOADED_STATE__`, XHR endpoints) rather than rendered HTML (Vinciguerra).
- Use ML/visual extraction: Diffbot's Article/Analyze APIs use "image-recognition algorithms to categorize the page as one of 20 different types" and operate on "the raw pixels of a web page" rather than CSS selectors, which is their claimed robustness to redesigns. Trade-off: cost, a vendor dependency, and non-determinism — a model update can silently change your extraction, which is the *same* re-baseline hazard as a trafilatura upgrade.
- "Self-healing selectors" (LLM- or heuristic-repaired locators) are an active practitioner area but are, as of now, blog-grade rather than research-grade; they also convert a loud failure into a silent wrong-answer, which is arguably worse for an unattended multi-month system.

---

## 7. Tools that exist

| Tool | What it is | Change-detection approach | Notes |
|---|---|---|---|
| **changedetection.io** (github.com/dgtlmoon/changedetection.io) | Self-hosted web page monitor | Fetch → filter (CSS/XPath 1.0+2.0 with `re:test`/`re:match`/`re:replace`, JSONPath, **jq**, visual selector) → "remove text by selector" / "ignore text" rules → diff. "Trigger on text" and conditional actions. PDF monitoring by text/filesize/checksum. Restock/price extraction. | Apache-2.0, ~33.4k stars. Fast HTTP fetcher *or* Chrome/Playwright/WebDriver for JS. Apprise notifications. Has an AI mode for plain-English rules. Community threads show the classic noise problems (e.g. discussion #577 on ignoring `&nbsp;`). |
| **urlwatch** (urlwatch.readthedocs.io) | CLI/cron page + command monitor | Filter pipeline then diff; `sha1sum` filter for pure change-flag use. Filters: `css`, `xpath`, `html2text`, `strip`/`striplines`, `grep`/`grepi`, `re.sub`/`re.findall`, `sort`, `reverse`, `remove-duplicate-lines`, `jq`, `pdf2text`, `ocr`, `shellpipe`, `hexdump`. | Config-as-YAML, scriptable, no browser by default (successor project `webchanges` adds more). Best fit for "many sources, declarative rules, version-controlled config". |
| **Visualping** (visualping.io) | SaaS monitor | Four modes: **Visual** (pixel/CV screenshot diff), **Text** (structured text diff), **Element** (user-drawn region), **All** (default). Noise handled by region selection ("ignore headers, footers, rotating ads") plus an AI "Important Alerts" classifier. | Blog post dated 2026-03-12 (upd. 2026-04-12) claims the AI classifies **~87% of detected changes as non-critical** — an unusually explicit admission of the base-rate noise problem, though it's vendor marketing and the 87% is unaudited. No pixel-percentage threshold published. |
| **Diffbot** (diffbot.com/docs/extract/article) | Commercial extraction API | Not a change detector — an *extractor*. Visual ML page classification into ~20 page types; returns title, text, normalized HTML, author, date, images, sentiment, tags. | Use as the "field extraction" layer feeding field-level hashes. Removes boilerplate automatically. Selector-free ⇒ redesign-resistant, but model updates are an uncontrolled re-baseline risk. |
| **Wayback CDX API** (github.com/internetarchive/wayback/blob/master/wayback-cdx-server/README.md) | Archive index query | `digest` field + `collapse=digest` for unique-capture timelines | Free historical change-rate estimation and backfill. 150k result cap. Adjacent-only collapse. |
| **wget --warc / wpull / Heritrix / warcio** | Archival crawling | WARC = full request/response record; enables re-extraction later and byte-exact provenance | The "store every version" substrate. Pairs with CDC dedup. |
| **Spidermon** (spidermon.readthedocs.io, Zyte) | Scrapy monitoring framework | Item schema validation (jsonschema/schematics), stat-based alert expressions, field coverage, run-to-run comparison, periodic monitors | The purpose-built answer to §6. Notifications: email, Slack, Telegram, Discord; actions for S3, Sentry, SNS. |
| **git scraping** (simonwillison.net/2020/Oct/9/git-scraping/) | Pattern, not a tool | Git blob hashing *is* the change detector; no commit when content is identical. GH Actions cron. | Free versioning + free dedup + free diff UI. Requires you to normalize (e.g. `jq`) first or you commit noise every run. `git-history` (2021-12-07) queries the resulting history. |
| **dbt snapshots** (docs.getdbt.com/docs/build/snapshots) | Warehouse SCD-2 | `timestamp` or `check`/`check_cols` strategies; `hard_deletes` | The downstream half of the system. |
| **Debezium** (debezium.io) | Log-based CDC | N/A to scraping directly, but the reference semantics for "what polling loses" | Useful as the mental model and for your *own* store's change stream. |
| **Airbyte** (docs.airbyte.com) | Connector framework | Incremental sync with cursor field, lookback window, step slicing, cursor granularity | Good reference implementation of the `updated_since` pattern's mitigations. |

---

## 8. Empirical change rates: government/reference vs news

Four studies, spanning 1999–2009, all agreeing on the direction and disagreeing somewhat on magnitude (measurement definitions differ, which is the main reason).

### Cho & Garcia-Molina, *The Evolution of the Web and Implications for an Incremental Crawler*, VLDB 2000 (Cairo)
- **720,000 pages** across **270 sites**, monitored **Feb 17 – Jun 24, 1999** (~4 months), daily.
- Change by TLD (their headline result):
  - **`.com`: >40% changed daily.**
  - **`.edu` and `.gov`: <10% changed daily**, and **>50% of `.edu`/`.gov` pages did not change at all over the whole 4 months.**
  - `.net`/`.org`: intermediate.
- "More than 20% of pages had changed whenever we visited them."
- Average change interval across all pages ≈ **4 months**.
- **50% turnover time: 11 days for `.com`; ~4 months for `.gov`.**
- **>70% of pages remained accessible for more than one month.**
- Models page changes as a **Poisson process**; reports the model "predicts the observed data very well" for pages with detectable change patterns, with poorer fit at the extremes (very frequent and completely static pages).
- Implications drawn: per-page adaptive revisit frequency, in-place update over shadowing, continuous over batch crawling.

### Fetterly, Manasse, Najork, Wiener, *A Large-Scale Study of the Evolution of Web Pages*, WWW 2003 / *Softw. Pract. Exper.* 2004; 34:213–237
- **151 million HTML pages**, **11 weekly crawls**, Nov 2002 – Feb 2003.
- Method: MD5 checksums **plus** feature vectors of **5-word shingles** — the shingle layer is precisely what let them separate markup noise from content change.
- **65.2%** of successive-week page pairs showed **no change**; **9.2%** differed **only in markup**; ⇒ **74.4%** zero-or-markup-only.
- **`.com` changed more frequently than `.gov` and `.edu`**; after excluding spam, `.gov`/`.edu` were substantially lower than commercial.
- `.de` anomaly: **27%** underwent large/complete change weekly vs a **3%** overall average.
- **Larger documents change more often and more severely** (32 KB+ vs ≤4 KB) — counterintuitive but replicated.
- **"Past changes to a document are a good predictor of future changes"** (demonstrated across three successive weeks). This is the empirical license for adaptive per-URL scheduling.
- **88%** of URLs still reachable at the 11th crawl.

### Adar, Teevan, Dumais, Elsas, *The Web Changes Everything: Understanding the Dynamics of Web Content*, WSDM '09 (Barcelona, Feb 9–12 2009)
The richest source for *how much* changes, not just *whether*.
- **55,000 pages** (54,788 final), **hourly crawls for 5 weeks** from May 24, 2007; sub-hourly for 22,818; pages selected from Live Search toolbar logs of **612,000 users**.
- **34%** of pages showed **no change** over the whole study.
- **65.7%** of pages changed within 60-minute intervals (for the changing subset).
- Median inter-change time for changers: **123 hours.**
- **Amount** of change: mean inter-version **Dice = 0.794**, **median 0.950**. At 2-minute intervals: Dice **0.937 mean / 0.985 median**. → **Most changes are small.**
- **Knot point** (where content stops diverging / stabilizes): mean **145 hours**, median **92 hours**.
- By category: **News/Sports have the earliest knot points (~33–66 h)**, news pages inter-version Dice **0.8700**; **industry/trade** decays gradually (~218 h mean interval), Dice **0.6649**.
- **`.gov`/`.edu` pages: less frequent change, higher stability.**
- **Term "staying power" is bimodal** — high-staying-power terms are navigation and ongoing-topic terms; low-staying-power terms are transient. This is effectively an automatic, data-driven boilerplate detector.
- **DOM element survival: 99.3% at 2 hours, 84.3% at 5 weeks**; nav/persistent elements highly stable, **ads change rapidly**.

The companion paper **Adar, Teevan, Dumais, Elsas, *Resonance on the Web: Web Dynamics and Revisitation Patterns*, CHI 2009** (April 2009) analyzes an **hourly crawl of 40K+ pages** against logs from **2.3M users**, correlating change with revisitation.

> **Note on staleness.** These are the canonical numbers and they are 17–26 years old. Directionally the `.gov` < `.com` finding is robust and repeatedly replicated, but the *absolute* rates are from a pre-SPA, pre-CDN, pre-A/B-testing web. Today's government sites: the *data* still changes slowly, but the *bytes* change constantly (analytics tags, CMS build IDs, cookie banners, USWDS asset hashes). So the modern reading is: **the gap between "raw bytes changed" and "data changed" is much wider now than these studies measured**, which strengthens rather than weakens the case for canonicalization. There is no equally-authoritative modern replication; the closest recent methodological work is longitudinal analysis over **Common Crawl** (arXiv:2404.09770, *Improved methodology for longitudinal Web analytics using Common Crawl*, 2024), which discusses the sampling biases that make such replication hard.

### Practical translation to schedules

| Source class | Expected real-change cadence | Implied polling posture |
|---|---|---|
| Statute/regulation reference pages, archived datasets, historical tables | months–years; >50% never change (Cho) | weekly/monthly conditional GET; rely heavily on ETag/304; full re-baseline quarterly |
| Agency "news"/press release listings, Federal Register | daily | daily; feeds + conditional GET; feed truncation is the risk |
| Dataset catalogs (data.gov, CKAN, DCAT) | irregular, bursty | `metadata_modified` cursor + periodic full enumeration for deletions |
| Scientific repositories (OAI-PMH) | daily–weekly | OAI `from`/`until`; full re-harvest periodically to catch `deletedRecord=no` vanishings |
| Live indicators, dashboards, status pages, real-time monitors | minutes–hours | field-level polling only; never page hashing |
| News | hours; knot at ~33–66h (Adar) | not your primary use case, but the noisiest class |

---

## 9. Open disagreements, stated rather than resolved

1. **Shingling vs SimHash.** Henzinger (SIGIR 2006) found neither dominates: shingling wins on same-site pairs, simhash across sites; she proposed combining them. Manku et al. (WWW 2007) chose simhash for Google largely on **storage** grounds (8 bytes vs 24) and reported ~0.75/0.75 precision/recall at k=3. If your corpus is many pages from few government domains, that is the same-site regime where Henzinger favors shingling.
2. **Which boilerplate remover is best.** Trafilatura's self-published benchmark puts trafilatura at F1 0.924, well ahead of readability (0.826) and boilerpy3 (0.807). Independent evaluations (Bevendorff et al. SIGIR 2023; Sandia SAND2024-10208; ACM 10.1145/3726302.3730234, 2025) rank differently and use different corpora. Every published benchmark uses news-article corpora that are unrepresentative of government tables/forms/PDF-wrappers. Unresolved; run your own eval.
3. **Trust `<lastmod>` or not.** The sitemaps.org spec treats it as informational metadata. Google's docs say it is used only "if it's consistently and verifiably … accurate" — an all-or-nothing per-site trust decision. Practitioners split between "use it to skip fetches" (fast, risks silent misses) and "use it only to prioritize" (safe, costs bandwidth). No authority resolves this; it's a per-source empirical question.
4. **Store every version vs store diffs.** Git-scraping/WARC camp: raw is cheap with dedup and survives extractor changes. SCD-2/warehouse camp: only structured history is queryable and diffs are compact. Both are defended by serious practitioners; the hybrid costs more than either.
5. **`check_cols` scope.** dbt explicitly warns against `check_cols='all'` ("It is better to explicitly enumerate the columns"), while a lot of production code uses a hash of all columns as a surrogate key — which dbt *also* recommends as the alternative. These are in tension in practice: enumeration is precise but rots as schemas evolve; hash-everything is robust to schema evolution but noisy and slow.
6. **`timestamp` vs `check` strategy.** dbt recommends `timestamp` because it "automatically handles new or removed columns" — but that recommendation depends entirely on the source's `updated_at` being honest, which for scraped government sources it usually is not. dbt's own fallback advice ("useful for tables which do not have a reliable `updated_at` column") is the more relevant branch here.
7. **AI/LLM change classification.** Visualping claims its AI classifies ~87% of detected changes as non-critical and that Q1-2026 model work improved handling of "banner rotations, footer updates, cookie popups." That is vendor-reported and unaudited. Counter-position (implicit in Spidermon/Bakour): deterministic schema + coverage checks are auditable and reproducible; an LLM filter is a non-deterministic component in a system meant to run unattended for months, and it converts loud failures into silent suppressions.
8. **Self-healing selectors.** Practitioner blogs advocate LLM-repaired locators; there is no peer-reviewed evidence of reliability, and the failure mode (silently extracting the wrong element) is worse for unattended operation than a loud break. No authority endorses this yet.

---

## 10. Failure modes worth designing against explicitly

- **304s that lie** (stale edge cache, misconfigured CDN). Mitigation: periodic validator-ignoring full fetch.
- **ETags that vary by node/encoding** (Apache `-gzip` suffix, inode-based ETags behind a load balancer). Mitigation: pin `Accept-Encoding`, or fall back to content hash when ETag churn exceeds content churn.
- **`Last-Modified` bumped by deploys** without content change. Mitigation: confirm with a canonicalized content hash before emitting a change event.
- **Sitemap lastmod = build time.** Mitigation: measure lastmod↔hash agreement per source; downgrade untrustworthy sitemaps to "prioritize only".
- **Feed truncation** — items scroll out of the window between polls. Mitigation: measure turnover, poll at a fraction of it, plus periodic full enumeration.
- **OAI-PMH `deletedRecord=no`** — retractions are invisible to incremental harvesting (spec's own warning). Mitigation: periodic full re-harvest + tombstone by absence.
- **Cursor ties and clock skew** — records skipped at window boundaries. Mitigation: composite `(updated_at, id)` keyset cursor, lookback window, lagged high-water mark.
- **Extractor version upgrade** re-baselines the whole corpus. Mitigation: pin versions; treat upgrades as planned re-baselines; keep raw so you can re-extract both ways and diff the *diffs*.
- **Layout change masquerading as mass data change.** Mitigation: the >X%-of-records-changed circuit breaker (§6.7) + structure fingerprint (§6.8).
- **Soft 200s** (login walls, WAF interstitials, "no results" pages). Mitigation: content assertions, not status codes; golden/canary pages.
- **Deletions generally.** Polling cannot see them (Morling's point 4). Mitigation: full enumeration on a slower cadence + explicit tombstone rule, decided up front.

---

## Sources

**Standards / specifications**
- RFC 9110, *HTTP Semantics* (June 2022, STD 97) — §8.8.1 weak vs strong validators, §8.8.2 Last-Modified, §8.8.3 ETag, §13.1.2 If-None-Match, §13.1.3 If-Modified-Since, §13.2.2 precedence: https://www.rfc-editor.org/rfc/rfc9110.html (browsable: https://httpwg.org/specs/rfc9110.html)
- RFC 4287, *The Atom Syndication Format* (Dec 2005) — §4.2.9 `atom:published`, §4.2.15 `atom:updated`: https://www.rfc-editor.org/rfc/rfc4287
- RFC 6902, *JavaScript Object Notation (JSON) Patch* (Apr 2013): https://www.rfc-editor.org/rfc/rfc6902
- RFC 6901, *JSON Pointer*: https://www.rfc-editor.org/rfc/rfc6901
- RFC 7386, *JSON Merge Patch*: https://www.rfc-editor.org/rfc/rfc7386
- OAI-PMH v2.0 protocol specification — datestamps, granularity, `from`/`until`, `resumptionToken`, `deletedRecord`: https://www.openarchives.org/OAI/openarchivesprotocol.html
- Sitemaps protocol: https://www.sitemaps.org/protocol.html
- MDN, *HTTP conditional requests* (incl. the Apache `-gzip` ETag / `DeflateAlterETag` warning): https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests
- MDN, *304 Not Modified*: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/304
- MDN, *Transfer-Encoding* (chunked ⇒ no Content-Length): https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Transfer-Encoding

**Measurement of the deployed web**
- HTTP Archive Web Almanac 2021, *Caching* chapter (crawl 2021-12-15): Last-Modified 70.5%, ETag 46.5%, both 41.6%, neither 24.5%, ~0.9%/0.7% invalid Last-Modified dates: https://almanac.httparchive.org/en/2021/caching

**Academic — near-duplicate detection**
- Manku, Jain, Das Sarma, *Detecting Near-Duplicates for Web Crawling*, WWW 2007, Banff (Google): 64-bit simhash, k=3, 8B pages, ~0.75/0.75 precision/recall, permutation-table algorithm, 8 vs 24 bytes vs Broder: https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/33026.pdf
- Charikar, *Similarity Estimation Techniques from Rounding Algorithms*, STOC 2002 (origin of simhash).
- Broder, *On the Resemblance and Containment of Documents*, SEQUENCES '97: resemblance/containment, w-shingles, min-wise sketches, AltaVista: https://www.semanticscholar.org/paper/On-the-resemblance-and-containment-of-documents-Broder/8addb1718c2bc6bbb0d82cd1a57b41198bf65965 · dblp: https://dblp.org/rec/conf/sequences/Broder97.html
- Broder, Glassman, Manasse, Zweig, *Syntactic Clustering of the Web*, WWW6 1997.
- Henzinger, *Finding Near-Duplicate Web Pages: A Large-Scale Evaluation of Algorithms*, SIGIR 2006 (shingling vs simhash; neither dominates): https://dl.acm.org/doi/10.1145/1148170.1148222
- `scrapinghub/python-simhash`: https://github.com/scrapinghub/python-simhash
- Xia et al., *FastCDC: a Fast and Efficient Content-Defined Chunking Approach for Data Deduplication*, USENIX ATC '16: https://www.usenix.org/system/files/conference/atc16/atc16-paper-xia.pdf
- *A Thorough Investigation of Content-Defined Chunking Algorithms for Data Deduplication* (2024): https://arxiv.org/html/2409.06066v3

**Academic — web change rates**
- Cho & Garcia-Molina, *The Evolution of the Web and Implications for an Incremental Crawler*, VLDB 2000: 720k pages / 270 sites / Feb–Jun 1999; .com >40% daily vs .edu/.gov <10%; 11-day vs ~4-month 50% turnover; Poisson model: https://www.vldb.org/conf/2000/P200.pdf
- Cho & Garcia-Molina, *Estimating Frequency of Change*, ACM TOIT 2003: https://dl.acm.org/doi/10.1145/857166.857170
- Fetterly, Manasse, Najork, Wiener, *A Large-Scale Study of the Evolution of Web Pages*, WWW 2003 / SP&E 2004;34:213–237: 151M pages, 11 weeks, 65.2% no change, 9.2% markup-only, .gov/.edu lower, larger pages change more, past predicts future: https://marc.najork.org/papers/spe2004.pdf · https://dl.acm.org/doi/10.1145/775152.775246
- Adar, Teevan, Dumais, Elsas, *The Web Changes Everything: Understanding the Dynamics of Web Content*, WSDM 2009: 55k pages hourly × 5 weeks, 34% no change, Dice 0.794 mean / 0.950 median, knot 145h mean / 92h median, DOM survival 99.3%@2h / 84.3%@5wk: http://www.cond.org/wsdm09-change-camready.pdf · https://dl.acm.org/doi/10.1145/1498759.1498837
- Adar, Teevan, Dumais, Elsas, *Resonance on the Web: Web Dynamics and Revisitation Patterns*, CHI 2009: hourly crawl of 40K+ pages, 2.3M users: https://www.microsoft.com/en-us/research/publication/resonance-on-the-web-web-dynamics-and-revisitation-patterns/
- Mallawaarachchi, Meegahapola, Madhushanka et al., *Change Detection and Notification of Web Pages: A Survey*, ACM Computing Surveys 53(1), 2020: https://dl.acm.org/doi/10.1145/3369876 (paywalled; open copy via https://www.researchgate.net/publication/339084778)
- *Improved methodology for longitudinal Web analytics using Common Crawl* (2024): https://arxiv.org/pdf/2404.09770

**Content extraction / boilerplate removal**
- Trafilatura benchmark (evaluation dated 2026-08-04, 990 docs): https://trafilatura.readthedocs.io/en/latest/evaluation.html
- Bevendorff et al., *An Empirical Comparison of Web Content Extraction Algorithms*, SIGIR 2023: https://dl.acm.org/doi/10.1145/3539618.3591920
- *Multilingual Evaluation of Main Content Extractors for Web Pages* (2025): https://dl.acm.org/doi/10.1145/3726302.3730234
- Sandia National Laboratories, *An Evaluation of Main Content Extraction*, SAND2024-10208, Aug 2024: https://www.osti.gov/servlets/purl/2429881
- scrapinghub article-extraction benchmark: https://github.com/scrapinghub/article-extraction-benchmark

**Tools**
- changedetection.io: https://github.com/dgtlmoon/changedetection.io · `&nbsp;` noise discussion #577: https://github.com/dgtlmoon/changedetection.io/discussions/577
- urlwatch filters reference: https://urlwatch.readthedocs.io/en/latest/filters.html · advanced: https://urlwatch.readthedocs.io/en/latest/advanced.html
- Visualping, *How Visualping Uses AI to Monitor Web Changes* (2026-03-12, upd. 2026-04-12): https://visualping.io/blog/visualping-ai
- Diffbot Article API: https://www.diffbot.com/docs/extract/article · Analyze API: https://www.diffbot.com/docs/extract/analyze
- Wayback CDX Server API README: https://github.com/internetarchive/wayback/blob/master/wayback-cdx-server/README.md · docs: https://archive.org/services/docs/api/wayback-cdx-server.html · digest-accuracy thread: https://archive.org/post/1009990
- Spidermon (Zyte): https://spidermon.readthedocs.io/en/latest/
- Simon Willison, *Git scraping* (2020-10-09): https://simonwillison.net/2020/Oct/9/git-scraping/ · git-history (2021-12-07): https://simonwillison.net/2021/Dec/7/git-history/
- dbt snapshots: https://docs.getdbt.com/docs/build/snapshots · snapshot configs: https://docs.getdbt.com/reference/snapshot-configs
- Debezium features: https://debezium.io/documentation/reference/stable/features.html
- Gunnar Morling, *Five Advantages of Log-Based Change Data Capture* (2018-07-19): https://debezium.io/blog/2018/07/19/advantages-of-log-based-change-data-capture/
- Airbyte incremental sync (cursor, lookback window, step, granularity): https://docs.airbyte.com/platform/connector-development/connector-builder-ui/incremental-sync
- Great Expectations: https://greatexpectations.io/ · Databricks pipeline expectations: https://docs.databricks.com/aws/en/ldp/expectations

**Government / scientific source specifics**
- Google Search Central, *Build and submit a sitemap* ("Google ignores `<priority>` and `<changefreq>` values"; lastmod used "if it's consistently and verifiably … accurate"): https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap
- Google Search Central blog, *Sitemaps ping endpoint is going away* (June 2023): https://developers.google.com/search/blog/2023/06/sitemaps-lastmod-ping
- Search Engine Roundtable, *Google Either Trusts Or Doesn't Trust Your Sitemap's Lastmod Date* (secondary, reporting Gary Illyes): https://www.seroundtable.com/google-sitemap-lastmod-binary-trust-37554.html
- Regulations.gov API v4 (5,000-record cap, lastModifiedDate pagination workaround, Eastern-time filter caveat, rate limits): https://open.gsa.gov/api/regulationsgov/
- Federal Register API v1: https://www.federalregister.gov/developers/documentation/api/v1
- GovInfo search service API: https://www.govinfo.gov/features/search-service-overview
- data.gov harvester API: https://resources.data.gov/harvester-api/ · `datagov-harvesting-logic` (create/update/delete DCAT-US record comparison): https://pypi.org/project/datagov-harvesting-logic/ · GitHub: https://github.com/GSA/datagov-harvesting-logic

**Practitioner**
- Yahia Bakour, *Scraper Monitoring in Production: A Practical Guide*, context.dev, 2026-08-16 (null-rate 2%→70% example, canary pages, "HTTP 200 is not success"): https://www.context.dev/blog/scraper-monitoring-in-production
- Pierluigi Vinciguerra, *Change detection for web scraping*, The Web Scraping Club, 2023-10-15 (prefer embedded JSON / `__NEXT_DATA__`, generic selectors, Wappalyzer stack monitoring): https://www.scraping.club/p/change-detection-for-web-scraping
- Knit, *API Pagination Stability: How to Avoid Duplicates, Gaps, and Cursor Drift* (2026): https://www.getknit.dev/blog/how-to-preserve-api-pagination-stability



---

# Avoiding Blocks & Bans in Large-Scale Government/Scientific Data Harvesting

**Research date:** 2026-09-06. All rate-limit numbers verified against official documentation on this date unless flagged otherwise.

**This is not legal advice.** The legal section reports cases and statutes as information. Scraping law is unsettled, jurisdiction-specific, and moving fast.

---

## 0. Executive framing: two different problems

There are two distinct problems that get conflated:

1. **Being a well-behaved client of sources that *want* to serve you.** Government and scientific sources (NCBI, SEC, data.gov, Crossref, arXiv, EBI) publish rate limits and issue API keys. They are not trying to stop you. Getting blocked here is an *own goal* — it means you exceeded a documented limit or failed to identify yourself. The fix is engineering discipline, not evasion.
2. **Defeating commercial bot-management systems** (Cloudflare, Akamai, DataDome, PerimeterX) on sources that *don't* want automated access — typically journal publishers, some university sites, and commercial aggregators.

For a corpus dominated by government/scientific sources, **problem 1 is the overwhelming majority of the work** and problem 2 is a narrow, expensive, legally exposed tail. The single highest-leverage design decision is maximizing the fraction of the corpus reached via category 1.

Critically: for category-1 sources, the adversarial toolkit is not merely unnecessary — **it is actively counterproductive.** Rotating IPs and forging User-Agents against NCBI destroys the identity that gets you a 10 req/s key, and NCBI's remedy for abuse is IP-blocking followed by a requirement that you register `tool` and `email`. You would be spending money to make yourself un-whitelistable.

---

## 1. The "don't get into that position" path — verified published limits

### 1.1 NCBI E-utilities (PubMed, GenBank, etc.)

Source: [NCBI E-utilities: General Introduction / Usage Guidelines and Requirements](https://www.ncbi.nlm.nih.gov/books/NBK25497/) (verified 2026-09-06)

- **Without API key:** "post no more than three URL requests per second"
- **With API key:** "By including an API key, a site can post up to 10 requests per second by default."
- Key is passed as the `api_key` parameter. **One API key per NCBI account**; requesting a new key invalidates the old one. This is an important architectural constraint — you cannot trivially shard a single institutional account across many workers to multiply throughput. Higher limits require contacting NCBI directly.
- **Off-peak guidance (explicit and unusual — most sources don't state this):** NCBI recommends limiting large jobs to "either weekends or between 9:00 PM and 5:00 AM Eastern time during weekdays."
- **`tool` and `email` parameters:** required for identification. Developers whose IPs are blocked must register `tool` (string uniquely identifying the software) and `email` (developer's contact, *not* an end user's) by emailing NCBI. "Once **tool** and **email** values are registered, all subsequent E-utility requests from that software package should contain both values."

**Note the enforcement model:** NCBI blocks by IP, and the un-blocking path is *identifying yourself more*, not less. This is the general pattern across scientific infrastructure.

### 1.2 SEC EDGAR

Sources: [SEC Webmaster FAQ](https://www.sec.gov/os/webmaster-faq), [SEC Internet Security Policy / Privacy & Security](https://www.sec.gov/about/privacy-information#security) (verified 2026-09-06)

- **Rate limit:** "Our current maximum access rate is 10 requests per second." The security policy is more explicit about aggregation: "Current guidelines limit users to a total of no more than **10 requests per second, regardless of the number of machines used to submit requests**."
  - That clause is important for distributed architectures: **the limit is per-organization, not per-IP.** Horizontally scaling workers across machines does not raise your ceiling and is arguably a violation even if no single machine exceeds 10/s.
- **Mandatory declarative User-Agent.** Required headers:
  ```
  User-Agent: Sample Company Name AdminContact@<sample company domain>.com
  Accept-Encoding: gzip, deflate
  Host: www.sec.gov
  ```
- **Enforcement:** "If a user or application submits more than 10 requests per second, further requests from the IP address(es) may be limited for a brief period." Recovery: "Once the rate of requests has dropped below the threshold for 10 minutes, the user may resume accessing content on SEC.gov."
- **Explicit bot policy:** "The SEC does not allow 'unclassified' bots or automated tools to crawl the site." A declarative UA is what "classifies" your bot.
- SEC states it does **not** offer technical support for programmatic downloading, and points heavy/real-time users to the EDGAR Public Dissemination Service (a paid subscription feed).
- **SEC's robots.txt** ([sec.gov/robots.txt](https://www.sec.gov/robots.txt)) explicitly `Allow: /Archives/edgar/data`, disallows admin/search/user paths and `/Archives/edgar/vprr/`. **It contains no Crawl-delay directive** — the 10 req/s number lives only in prose documentation, not in machine-readable form. This is a recurring trap: *robots.txt compliance alone does not make you compliant with a site's stated limits.*

### 1.3 api.data.gov (fronts many US federal agency APIs)

Source: [api.data.gov Developer Manual](https://api.data.gov/docs/developer-manual/) (verified 2026-09-06)

- **Default limit: "Hourly Limit: 1,000 requests per hour"** per API key. That is ~0.28 req/s sustained — far more restrictive than the per-second headline figures elsewhere, and the binding constraint for many federal APIs.
- `X-RateLimit-Limit` and `X-RateLimit-Remaining` headers returned on every response — machine-readable budget tracking is available and should be consumed rather than guessed at.
- Over limit returns **HTTP 429 (Too Many Requests)**.
- Key passed three ways: `X-Api-Key` header, `api_key` query parameter, or HTTP basic auth (key as username, empty password). **Prefer the header** — query parameters leak keys into server logs, proxy logs, and Referer headers.
- Higher limits: "please reach out to the agency responsible for the API you would like higher rate limits for." Note the limit is *per-agency-negotiated*, not centrally raised.

### 1.4 Crossref REST API — **CHANGED December 2025, verify anything older**

Source: [Announcing changes to REST API rate limits](https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/) (posted 2025-11-05, effective 2025-12-01)

Crossref imposed explicit numeric limits for the first time: "We haven't changed the rate limits since the REST API was launched in 2013."

| Pool | Single-record requests | List queries |
|---|---|---|
| **Public (anonymous)** | 5 req/s, 1 concurrent | 1 req/s, 1 concurrent |
| **Polite (`mailto` supplied)** | 10 req/s, 3 concurrent | 3 req/s, 3 concurrent |

- Polite pool entry: include an email via the `mailto` parameter (or User-Agent). "The second case here will be directed to the polite pool because an email is included using the 'mailto' parameter."
- Driver: "In the past five years, the number of requests to the REST API has tripled and the number of metadata records has increased by a third, from 120 million to around 180 million."
- **Note the concurrency limits, not just rates.** List queries in the public pool at 1 req/s / 1 concurrent are punishingly slow — the polite pool gives a 3× rate and 3× concurrency improvement for the cost of putting an email in a URL. This is the single cheapest compliance win available anywhere in this research.
- Crossref's [Tips for using the REST API](https://www.crossref.org/documentation/retrieve-metadata/rest-api/tips-for-using-the-crossref-rest-api/) adds: "Very occasionally we have had to block users who misuse our APIs, usually through carelessness rather than malice." Recommends cursors for large result sets, caching, splitting large queries, and monitoring for HTTP 429.
- A paid "Crossref Plus" tier exists for higher guaranteed throughput.

### 1.5 OpenAlex — **BREAKING CHANGE Feb 2026; the polite pool is GONE. Its own docs are stale.**

This is the most important stale-material finding in this research, and a live self-contradiction in OpenAlex's own materials.

- **Announcement** ([openalex-users Google Group, "API keys required starting Feb 13"](https://groups.google.com/g/openalex-users/c/rI1GIAySpVQ), posted 2026-01-14, effective 2026-02-13):
  - "**No more polite pool! No more email parameter in your calls—it was never secure and couldn't scale.**"
  - With API key: "You get 100,000 credits per day with your API key"
  - Without: "Without a key you get 100 credits per day, which is fine for testing and demos but not for real work" — then **409 errors**.
- **Current help center** ([help.openalex.org/api/authentication/](https://help.openalex.org/api/authentication/)): "Two things return `429 Too Many Requests`: exceeding your daily budget, or making more than **100 requests per second**." Key passed as `api_key=` parameter or `Authorization: Bearer YOUR_KEY` header. [help.openalex.org/api/](https://help.openalex.org/api/) describes a free tier "no key required," with a free API key raising the daily budget "10×," and heavier use "pay-as-you-go."
- **Contradiction / stale docs:** the GitHub docs repo [`ourresearch/openalex-docs` rate-limits-and-authentication.md](https://github.com/ourresearch/openalex-docs/blob/main/how-to-use-the-api/rate-limits-and-authentication.md) *still* says "You don't need an API key to use OpenAlex" and still documents the mailto polite pool as current. `docs.openalex.org` now 302-redirects to `help.openalex.org`, and a legacy `docsold.openalex.org` is still live and indexed.
- **Also note the error-code discrepancy:** the announcement says exhausted quota yields **409**; the current help center says **429**. Any client must handle both, and must not treat 409 as a permanent/non-retryable failure.
- **Practical implication:** OpenAlex moved from free-and-polite to metered/credit-based within this year. Its pricing page is at [help.openalex.org/access/pricing](https://help.openalex.org/access/pricing/). Any architecture, tutorial, or library predating Feb 2026 that relies on `mailto` for OpenAlex is broken. Client libraries lag: see [pyalex issue #100, "Rate-limit headers discarded, content download bugs, no 429 handling"](https://github.com/J535D165/pyalex/issues/100).

**Generalizable lesson:** free scholarly APIs are drifting toward metered, authenticated, paid tiers. Budget for the possibility that other sources do the same over a multi-month/multi-year run, and design so that adding auth to a source is a config change, not a rewrite.

### 1.6 arXiv

Source: [arXiv API Terms of Use](https://info.arxiv.org/help/api/tou.html) (verified 2026-09-06)

- "make no more than **one request every three seconds**, and limit requests to a **single connection at a time**."
- That is ~0.33 req/s with concurrency 1 — the most restrictive limit in this set. Bulk needs are explicitly redirected to other channels: OAI-PMH interface, RSS feeds, bulk data downloads (arXiv offers requester-pays S3 bulk access), and the SWORD deposit API.
- Higher rates: contact arXiv support.
- The ToU does not specify a User-Agent requirement, but identifying yourself is still advisable.

### 1.7 Europe PMC / EMBL-EBI — **weakly documented, flag as uncertain**

- The [Europe PMC developers page](https://europepmc.org/developers) lists FTP, OAI-PMH, SOAP and RESTful API access but **publishes no numeric rate limit**. It does state a hard restriction: "**It is not permissible to use any kind of automated process to bulk download other content from Europe PMC**" — i.e. bulk acquisition must go through the designated FTP/OAI-PMH channels, not by crawling the REST API or the website.
- The only concrete numbers found are from a staff reply on the EBI-hosted support group ([epmc-webservices, "REST API / Limited number of requests"](https://groups.google.com/a/ebi.ac.uk/g/epmc-webservices/c/MfxQ8nvIT5Q)), Mohamed Selim, **2020-03-24**: "Currently we apply some throttling of **10 requests per second and 500 requests per minute**."
  - ⚠️ **Six years old, forum post, not official documentation, word "currently" implies subject to change.** Treat as indicative only. Note 500/min is *not* 10/s × 60 (=600) — the per-minute cap binds first, so sustained throughput is ~8.3 req/s.
- **Recommendation for sources like this:** where no limit is published, the defensible engineering position is to pick a conservative self-imposed limit, identify yourself, honor 429/503 + Retry-After, and email the operator to ask. Asking is cheap and creates a record of good faith that matters enormously if you are later accused of abuse.

### 1.8 Summary table of verified limits

| Source | Anonymous | Authenticated / polite | Concurrency | Notes |
|---|---|---|---|---|
| NCBI E-utilities | 3 req/s | 10 req/s (`api_key`) | — | 1 key/account; off-peak guidance; `tool`+`email` |
| SEC EDGAR | 10 req/s | n/a | — | **Org-wide, not per-IP**; UA w/ email mandatory |
| api.data.gov | — | 1,000 req/**hour** | — | 429 + `X-RateLimit-*` headers |
| Crossref (single) | 5 req/s | 10 req/s | 1 → 3 | Changed 2025-12-01 |
| Crossref (lists) | 1 req/s | 3 req/s | 1 → 3 | Changed 2025-12-01 |
| OpenAlex | 100 credits/day | 100,000 credits/day | 100 req/s cap | Key required since 2026-02-13 |
| arXiv | 1 req / 3 s | n/a | **1** | Bulk → OAI-PMH / S3 |
| Europe PMC | ~10 req/s, 500/min *(2020, unofficial)* | — | — | Bulk download via API prohibited |

---

## 2. robots.txt: what it actually is, and the binding debate

### 2.1 RFC 9309 — the standard

Source: [RFC 9309, Robots Exclusion Protocol](https://www.rfc-editor.org/rfc/rfc9309.html), **September 2022, Standards Track**

After ~28 years as a de facto convention (the 1994 Martijn Koster convention), the REP was finally standardized.

What RFC 9309 defines: `allow`/`disallow` rules, user-agent matching (case-insensitive, `*` wildcard), longest-match path precedence, special characters (`#`, `$`, `*`), and access methods.

What it **does not** define:
- **Crawl-delay is not in the standard.** It is not mentioned as a defined record.
- Sitemaps are only permitted as an extension: "Crawlers MAY interpret other records that are not part of the robots.txt protocol -- for example, 'Sitemaps'".

Concrete requirements worth implementing:
- **Parsing limit: "The parsing limit MUST be at least 500 kibibytes [KiB]."**
- **Caching: "Crawlers SHOULD NOT use the cached version for more than 24 hours, unless the robots.txt file is unreachable."** — i.e. re-fetch robots.txt at least daily; this matters for a months-long unattended run where a site may add restrictions mid-flight.

### 2.2 Google's implementation

Source: [Google robots.txt specification](https://developers.google.com/search/docs/crawling-indexing/robots/robots_txt)

- Supports exactly four fields: `user-agent`, `allow`, `disallow`, `sitemap`.
- **Explicitly does not support crawl-delay:** "Google supports the following fields (other fields such as `crawl-delay` aren't supported)".
- File size limit: "Google enforces a robots.txt file size limit of 500 kibibytes (KiB). Content which is after the maximum file size is ignored."
- Caching: "Google generally caches the contents of robots.txt file for up to 24 hours, but may cache it longer in situations where refreshing the cached version isn't possible".
- Lenient parsing: "Google ignores invalid lines in robots.txt files."

### 2.3 Crawl-delay: who honors it

Crawl-delay is non-standard and **support is genuinely inconsistent** — this is a real disagreement between major operators, not a settled question:

- **Google: explicitly ignores it** (documented above; Google directs site owners to Search Console crawl-rate settings instead).
- **Bing: historically honors it** and documents crawl-delay values in its webmaster guidance (Bing's crawl-control help page has moved/404'd during this research — treat the specific URL as unstable, and verify current Bing behavior directly if it matters).
- **Yandex: historically honored it**, later also moved toward its own webmaster-tools control.
- **Cloudflare treats it as an obligation for verified bots** — failure to "respect the crawl-delay directive in robots.txt" is listed as grounds for revoking Verified Bot status ([Cloudflare verified bots](https://developers.cloudflare.com/bots/concepts/bot/verified-bots/)). So even though it isn't in RFC 9309, a major infrastructure gatekeeper enforces it as policy.

**Practical stance for a harvesting system:** parse and honor crawl-delay even though it's non-standard and Google ignores it. You are not Google; you have no reciprocal value to offer site operators, and the cost of honoring it is low relative to the reputational and access cost of being seen to ignore it.

### 2.4 Is robots.txt binding? The debate, with attribution

**Legally:** robots.txt is not a contract and there is no US case holding that violating robots.txt is *per se* unlawful. It is a machine-readable request. However:
- Under **EU DSM Art. 4(3)**, a machine-readable reservation of rights — which robots.txt can constitute — *does* have legal effect for TDM purposes (see §5.4). A Dutch court has held a TDM opt-out must be machine-readable ([IPKat, Feb 2025](https://ipkitten.blogspot.com/2025/02/dutch-court-holds-that-tdm-opt-out-must.html)).
- Ignoring robots.txt is powerful **evidence of bad faith** in any dispute, and factors into trespass-to-chattels and unjust-enrichment style arguments.

**Empirically, compliance is poor.** [*Scrapers Selectively Respect robots.txt Directives: Evidence From a Large-Scale Empirical Study*](https://arxiv.org/html/2505.21733) (arXiv 2505.21733; data collected 2025-02-12 to 2025-03-29; ~3.9M external requests, 36 university-controlled sites, 130 self-declared bots, 19,250 unique user agents) deployed four progressively restrictive robots.txt versions and measured compliance:

- Compliance falls as restrictions tighten: **60.9% for crawl delays, 31.0% for endpoint restrictions, 30.7% for full disallow.**
- By category: SEO crawlers highest (**69.5%**), AI assistants and AI search crawlers ~**63%**, **headless browsers lowest at 15.5%**, "other" 25.5%.
- Individual variance is large: Amazonbot 97.3% crawl-delay / 100% endpoint compliance; Applebot only 44.4% endpoint compliance "despite claiming to respect robots.txt"; Bytespider (ByteDance) 39.8% / 0% / 0%.
- Between 9–15 bots per experiment **never fetched robots.txt at all**, yet some still partially complied. AI assistants/search crawlers checked robots.txt in under 40% of 24-hour windows.
- Authors' conclusion: "**relying on robots.txt to prevent unwanted scraping is risky**," and they call for alternative deterrence mechanisms.

**The disagreement to report plainly:**
- *Normative camp* (crawler-ethics tradition, Cloudflare's verified-bot policy, most academic guidance): robots.txt expresses operator consent and should be treated as binding regardless of legal force.
- *Descriptive/skeptical camp* (the arXiv study's own framing; commercial scrapers): it is unenforceable, widely ignored, and site operators should not rely on it.
- *Fiesler's position* (see §6) cuts across both: legality and ethics are distinct, and neither ToS nor robots.txt compliance is a sufficient ethical checklist — nor is violation automatically unethical.

**For government/scientific sources specifically**, this debate is largely moot: these operators generally *want* their data harvested, publish access channels, and their robots.txt files are permissive toward the data (SEC explicitly `Allow: /Archives/edgar/data`). The debate matters at the margins — publisher sites, university pages, commercial aggregators.

---

## 3. Identification: making yourself known

The counterintuitive core principle for this class of system: **identity is an asset, not a liability.** Every rate-limit upgrade documented in §1 is purchased with identity. Anonymity buys you the worst tier everywhere.

### 3.1 What a good User-Agent looks like

Consensus pattern across SEC, Crossref, NCBI, and crawler norms:

```
ProjectName/1.0 (+https://example.org/crawler; contact@example.org)
```

Components and why:
- **Stable product token + version** — lets operators allowlist you and correlate behavior changes with your releases.
- **`+https://` info URL** — a crawler info page (the convention popularized by Googlebot/Bingbot). It should state: who runs the crawler, what it collects, why, what rate limits you self-impose, how to request exclusion or rate reduction, and a monitored contact address.
- **Contact email** — SEC *mandates* this; Crossref/OpenAlex historically rewarded it with better pools; NCBI requires `email` as a parameter.

**The crawler info page is the highest-value, lowest-cost artifact in the whole compliance stack.** When an overloaded sysadmin sees unfamiliar traffic, the difference between "block the IP range" and "email them and ask them to slow down" is usually whether there's a URL in the User-Agent. It also gives you a place to publish your IP ranges.

### 3.2 The `From:` header

`From:` (an HTTP header carrying an email address, defined in RFC 9110 §10.1.2) is the standards-blessed place for a contact address and is explicitly intended for robotic agents. In practice it is rarely logged or checked; almost all operators look at User-Agent. **Send both** — it costs nothing.

### 3.3 Reverse-DNS verifiability

Source: [Google, Verifying Googlebot and other Google crawlers](https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot)

The established pattern for proving a crawler is who it claims to be, since User-Agent is trivially forged:

1. "Run a reverse DNS lookup on the accessing IP address from your logs, using the `host` command."
2. "Verify that the domain name is either `googlebot.com`, `google.com`, or `googleusercontent.com`."
3. "Run a forward DNS lookup on the domain name retrieved in step 1 using the `host` command."
4. "Verify that it's the same as the original accessing IP address from your logs."

Google also publishes machine-readable IP ranges in JSON (CIDR) for automated allowlisting.

**Implication for architecture:** if you want to be verifiable, you need **stable IPs with controllable reverse DNS** — i.e. your own address space or a hosting provider that grants PTR control. This is *directly and fundamentally incompatible with rotating residential proxies.* You must choose one posture or the other per source; you cannot be both verifiable and evasive from the same egress.

### 3.4 Emerging: cryptographic bot identity (Web Bot Auth)

[draft-meunier-web-bot-auth-architecture](https://datatracker.ietf.org/doc/draft-meunier-web-bot-auth-architecture/) (Thibault Meunier & Sandor Major; -05 last updated 2026-03-02, now **expired and replaced by** `draft-meunier-webbotauth-httpsig-protocol`) proposes "an architecture for identifying automated traffic using HTTP-MESSAGE-SIGNATURES" — bots sign requests with a private key rather than asserting identity via forgeable headers or IP.

Status caveat: individual Internet-Draft, "no formal standing in the IETF standards process," and the architecture draft has already expired/been superseded once. **Do not build on it as a dependency**, but note that Cloudflare already accepts "cryptographic Web Bot Auth signature" as one of three verification methods for Verified Bots — so it is real in at least one major deployment and the direction of travel is toward cryptographic crawler identity.

### 3.5 Cloudflare Verified Bots — the formal legitimacy channel

Source: [Cloudflare, Verified bots](https://developers.cloudflare.com/bots/concepts/bot/verified-bots/)

Two bars to clear:

1. **Honest self-identification** via one of: "cryptographic Web Bot Auth signature", "published IP list with a stable user-agent", or "reverse DNS".
2. **Non-abusive behavior:** bots must "obey robots.txt and crawl directives, maintain reasonable request rates, and have not been observed evading website owner preferences or attacking sites."

Application is via an online form in the Cloudflare dashboard. Benefits: inclusion in BotBase and Cloudflare Radar's bots-and-agents directory, exemption from default bot-blocking configurations, and access to customers who allowlist verified bots.

Verification can be **revoked** for adding unauthorized IPs, security breaches, unpatched vulnerabilities, or failing to "respect the crawl-delay directive in robots.txt."

**This is the single most underrated option for a long-running research crawler.** A large fraction of the non-government web sits behind Cloudflare. Getting verified converts a permanent adversarial cost into a one-time application — but it requires committing to stable IPs, a stable UA, and genuine robots.txt compliance, i.e. the exact opposite of the evasion stack.

---

## 4. Politeness engineering

### 4.1 Rate limiting and per-domain concurrency

Design points specific to a many-engines/many-sources system:

- **The scheduling unit must be the *source authority*, not the worker.** SEC's "regardless of the number of machines used to submit requests" and NCBI's one-key-per-account rule both mean per-worker limits are wrong by construction. A distributed system needs a **shared, centralized token bucket per source** (Redis or equivalent), not per-process rate limiters. This is the most common architectural mistake in this domain and the most likely cause of an unattended multi-month run getting an organization IP-banned.
- **Concurrency is a separate limit from rate** and is often the binding one (Crossref: 1 vs 3 concurrent; arXiv: "a single connection at a time"). Enforce both.
- Configure limits **below** the published ceiling. Clock skew, retries, and health checks all consume budget; running at exactly 10 req/s against SEC guarantees occasional overage.
- Consume `X-RateLimit-Remaining` where offered (api.data.gov) rather than modeling the budget blindly.

### 4.2 Adaptive throttling — Scrapy AutoThrottle

Source: [Scrapy AutoThrottle documentation](https://docs.scrapy.org/en/latest/topics/autothrottle.html)

Stated design goals:
1. "be nicer to sites instead of using default download delay of zero"
2. "automatically adjust Scrapy to the optimum crawling speed, so the user doesn't have to tune the download delays"

Algorithm: start at `AUTOTHROTTLE_START_DELAY`; on each response compute target delay as **latency / N** where N = `AUTOTHROTTLE_TARGET_CONCURRENCY`; set the new delay to the **average of previous and target**; clamp between `DOWNLOAD_DELAY` and `AUTOTHROTTLE_MAX_DELAY`. Crucially, **"Non-200 responses can only increase delays, never decrease them"** — errors ratchet politeness up and never down, which is the correct asymmetry.

| Setting | Default |
|---|---|
| `AUTOTHROTTLE_ENABLED` | `False` |
| `AUTOTHROTTLE_START_DELAY` | 5.0 s |
| `AUTOTHROTTLE_MAX_DELAY` | 60.0 s |
| `AUTOTHROTTLE_TARGET_CONCURRENCY` | 1.0 |

**Note it is off by default** — an important gotcha, and the reason many Scrapy deployments are accidentally impolite.

**Important limitation:** AutoThrottle targets *latency*, using rising response times as a congestion signal. It does **not** know about published limits. A site that returns fast 429s will not slow AutoThrottle down much on latency grounds alone. **Adaptive throttling complements, and does not replace, a hard configured limit derived from published documentation.**

### 4.3 Retry-After, 429 and 503

Source: [RFC 9110 §10.2.3](https://www.rfc-editor.org/rfc/rfc9110.html#name-retry-after) (June 2022; obsoletes RFC 7230–7235 et al.)

`Retry-After` accepts either an **HTTP-date** or **delay-seconds**, and is used with **503 (Service Unavailable)**, **429 (Too Many Requests)**, and 3xx redirects. Both formats must be parsed. It "enabl[es] them to respect server-indicated retry windows rather than implementing arbitrary backoff strategies."

Correct behavior:
- **Always honor `Retry-After` when present** — it is the server telling you exactly what it wants, and ignoring it while claiming to be polite is indefensible.
- Absent `Retry-After`, use exponential backoff **with jitter** (uncorrelated retries across many workers; without jitter a fleet retries in lockstep and creates a thundering herd that looks exactly like an attack).
- Set a backoff ceiling and a circuit breaker: after N consecutive failures, stop hitting the source entirely and alert. For an unattended months-long run, **a circuit breaker is essential** — the failure mode you must design against is hammering a source for three weeks while nobody is watching.
- Distinguish 429 (you're too fast — slow down, retry) from 403 (you're blocked — stop and escalate to a human). Retrying a 403 aggressively converts a soft block into a permanent one.
- Handle **409** for OpenAlex quota exhaustion (per its announcement) as well as 429.

### 4.4 Caching and conditional requests

The cheapest request is the one you don't make. For a system re-running for months over largely-static corpora this is the dominant efficiency lever.

- **Conditional requests** (RFC 9110): store `ETag` and `Last-Modified`; send `If-None-Match` / `If-Modified-Since`. A **304 Not Modified** costs the server almost nothing and costs you no bandwidth. On a re-crawl of a stable government corpus, this can eliminate the large majority of payload transfer.
- **Honor cache directives** and persist a real HTTP cache (Scrapy's HttpCacheMiddleware, `requests-cache`, or a shared cache in front of all engines).
- **`Accept-Encoding: gzip, deflate`** — SEC explicitly requires it in its declared header set; it reduces load for both parties.
- **Prefer bulk channels over crawling** wherever they exist. This is the biggest single reduction in block risk available:
  - NCBI: FTP bulk downloads, datasets
  - SEC: EDGAR full-index files, financial statement data sets, daily/quarterly archives, and the PDS subscription for real-time
  - arXiv: OAI-PMH and requester-pays S3 bulk access
  - Europe PMC: FTP and OAI-PMH (and note bulk downloading *via other means is prohibited*)
  - Crossref: public data file torrents
  - OpenAlex: full data snapshot on S3
  
  Using a bulk channel converts thousands of rate-limited requests into one file transfer and removes the source from your block-risk surface entirely.
- **Incremental/delta harvesting**: OAI-PMH's `from`/`until` selective harvesting and API `updated_since` filters mean steady-state runs touch only changed records.

### 4.5 Off-peak scheduling

NCBI is the only source in this set that publishes explicit off-peak guidance ("weekends or between 9:00 PM and 5:00 AM Eastern time during weekdays") — and it should be treated as a hard scheduling constraint, not advice. For sources without published guidance, scheduling heavy jobs against the **operator's local night** is a low-cost goodwill measure. This requires the scheduler to be timezone-aware per source, which is a real design requirement for a global source list and is easy to overlook.

### 4.6 Operational monitoring for unattended running

For a months-long unattended system, the block-avoidance requirements are as much observability as they are politeness:
- Per-source dashboards of status-code mix; alert on any sustained rise in 403/429/503.
- Alert on response-body content changes (a source silently serving a CAPTCHA or block page with HTTP 200 is common and invisible to status-code monitoring alone).
- A monitored inbox for the address in your User-Agent — publishing a contact address you don't read is worse than publishing none.
- Kill switches per source, so a human can stop one engine without stopping the fleet.

---

## 5. The adversarial side — reported factually

**Scope note:** this section describes what exists and what it costs. For the government/scientific corpus at issue, essentially none of it should be needed, and several elements (IP rotation, UA forgery) are *actively harmful* because they destroy the identity that earns higher rate limits and forfeit Verified Bot eligibility.

### 5.1 How commercial bot management detects automation

**Cloudflare** ([bot score docs](https://developers.cloudflare.com/bots/concepts/bot-score/)):
- Score 1–99: "A bot score from 1 to 99 that indicates how likely that request came from a bot." **1 = Automated; 2–29 = Likely automated; 30–99 = Likely human; 0 = not computed;** plus a separate Verified Bot classification.
- Four engines: **Heuristics** ("Catches automated traffic through pattern matching against a database of known malicious fingerprints"); **Machine Learning** ("Catches sophisticated bots by analyzing request features across billions of daily requests" — produces most scores 2–99); **Anomaly Detection** ("Detects outlier requests by comparing traffic against a learned baseline for your specific site" — *being deprecated*); **JavaScript Detections** ("Catches headless browsers and other automation tools").

**DataDome** (per [Scrapfly's analysis, dated Aug 2026](https://scrapfly.io/blog/posts/how-to-bypass-datadome-anti-scraping) — note this is a scraping-vendor source with a commercial interest, but the technical description is consistent with other accounts):
- **TLS/JA3**: "Different operating systems, web browsers, or programming libraries perform the TLS encryption handshake uniquely, which results in different JA3 fingerprints."
- **HTTP/2 fingerprinting**: protocol version and header ordering; HTTP/1.1 flagged as suspicious.
- **JavaScript engine fingerprinting**: runtime, hardware, OS, browser capabilities.
- **IP classification**: residential (positive trust), mobile (positive), **datacenter (negative)**.
- **Behavioral ML** on navigation patterns.
- **"over 85,000 customer-specific and use-case-specific models"** — meaning "no universal bypass exists" and a technique that works on one protected site may fail on another.

Common browser-level signals across vendors: `navigator.webdriver`, Canvas and WebGL rendering fingerprints, font/plugin enumeration, screen and timezone consistency, CDP (Chrome DevTools Protocol) traces, and mouse/scroll/timing behavior.

### 5.2 TLS and HTTP/2 fingerprinting: JA3 → JA4

Source: [FoxIO-LLC/ja4](https://github.com/FoxIO-LLC/ja4) (the authors' own repo)

JA4+ is a suite: **JA4** (TLS client), **JA4S** (TLS server), **JA4H** (HTTP client), **JA4L/JA4LS** (latency), **JA4X** (X.509 certs), **JA4SSH**, **JA4T/JA4TS** (TCP), **JA4D/JA4D6** (DHCP), **JA4N** (NTP).

Why JA4 replaced JA3 — directly relevant to why evasion decays:
- "applications and libraries choose a unique cipher list more than unique ordering" — so JA3's sensitivity to ordering was noise.
- **"Google updated Chromium browsers to randomize their extension ordering"** (a.k.a. "cipher stunting"), which broke JA3 stability. JA4 sorts extensions and incorporates signature algorithms to stay stable despite randomization.

**Licensing (a real compliance trap if you build detection or resell):** JA4 (TLS client fingerprinting) is **BSD 3-Clause**. **All other JA4+ methods are under the FoxIO License 1.1**, which is "permissive for most use cases, including for academic and internal business purposes, but is not permissive for monetization" — vendors embedding it in commercial products need an OEM license.

The key structural point: **the fingerprinting side got *better* specifically in response to evasion.** This is the arms race made concrete, and it runs against the evader.

### 5.3 Evasion tooling: current state and churn

Tools that exist: `undetected-chromedriver`, `nodriver` (its successor), `playwright-stealth`, `patchright`, `rebrowser-playwright`, `camoufox`, `curl-impersonate` / `curl_cffi` (TLS impersonation), `tls-client`, plus commercial "anti-detect browsers."

**Benchmark evidence** — [Anti-Detect Browser Benchmark, ianlpaterson.com](https://ianlpaterson.com/blog/anti-detect-browser-benchmark-patchright-nodriver-curl-cffi/), published 2026-05-13, updated 2026-08-15. Methodology: three independent sweeps from a single residential macOS IP against 31 targets across four categories (JS detection panels, TLS fingerprint endpoints, live Cloudflare/anti-bot production sites, high-traffic content sites); **651 verdicts, zero drift across five hours.**

Results: **nodriver** led with 28 passing targets and zero blocks. Patchright, CloakBrowser and Camoufox clustered at 25–26. Vanilla Playwright and rebrowser-playwright tied at 24. Notably, **`curl_cffi` — a minimal HTTP wrapper — matched CloakBrowser's 130MB patched Chromium fork exactly**, which strongly suggests that for many targets the browser layer buys little over correct TLS impersonation.

Key finding: **the decisive signal was automation-protocol fingerprinting, not static fingerprints.** Cloudflare targets blocked all patched Chromium variants but passed nodriver, which "drives Chrome over a direct WebSocket connection" without Playwright's control-plane signature.

**Maintenance/churn evidence — the central practical point:**
- `rebrowser-playwright`: "no real code commits since September 2024, effectively unmaintained."
- `undetected-chromedriver` ([repo](https://github.com/ultrafunkamsterdam/undetected-chromedriver)): 12.8k stars, no deprecation notice in the README, but the README's currency references are dated (v3.5.0 / "selenium 4.9 or above") and the community has substantially migrated to the same author's `nodriver`. **Status ambiguous — verify commit history directly before depending on it.**
- Commentary titles alone track the churn: ["Playwright Stealth Not Working in 2026"](https://humanbrowser.cloud/blog/playwright-stealth-not-working-2026), ["The Plugin Era Is Over"](https://botcloud.dev/blog/playwright-anti-fingerprinting-alternatives-2026/).

**Source-quality caveat:** almost all writing in this space is published by proxy/scraping vendors with a direct commercial interest in claiming detection is both serious (buy our product) and beatable (buy our product). The ianlpaterson benchmark is the most methodologically transparent source found, but it is a single author, a single IP, and a single point in time. **Treat all bypass success rates as perishable.**

### 5.4 Proxies: types and prices

Source: [Proxidize Proxy Pricing Index 2026](https://proxidize.com/research/proxy-pricing-index-2026/), published 2026-07-22, updated 2026-08-24, surveying **14 market leaders**, comparing advertised vs. independently-reported pricing.

| Type | Price range | Notes |
|---|---|---|
| **Residential** | **advertised $0.49–$8.00/GB; actual entry $1.00–$8.00/GB** | "only a small number present entry pricing that closely reflects what a new buyer actually pays" |
| **Datacenter** | $0.18–$1.50/IP (per-IP, not per-GB) | Webshare aggressive at $0.018–$0.05/IP |
| **ISP / static residential** | $0.27–$4.60/IP/month | Decodo from $0.27/IP |
| **Mobile** | $0.50–$20/GB | fell "as much as 98%" 2023–2025; sometimes now below residential |

⚠️ **Proxidize is itself a proxy vendor** (and lists itself at $1/GB in its own index). Corroborating sources ([DataImpulse](https://dataimpulse.com/blog/residential-proxy-pricing-comparison/), [Proxidize blog](https://proxidize.com/blog/residential-proxy-pricing/)) are also vendors. Treat the **order of magnitude** (residential ≈ $1–8/GB, datacenter ≈ cents per IP) as reliable and specific figures as marketing.

**Cost modeling:** residential proxies are billed by **bandwidth**, so cost scales with page weight, not request count. A corpus of heavy HTML/PDF at $3/GB becomes expensive fast; the same corpus fetched directly costs nothing. Datacenter proxies are cheap but carry DataDome's explicit "negative score."

**Ethical/legal note on residential proxies:** residential proxy networks source IPs from real consumer devices, often via SDKs bundled into free apps, with contested disclosure quality. Routing government/scientific research traffic through third-party consumer devices raises consent questions independent of the target site, and can breach institutional network policy. This is a materially different risk category from a datacenter IP.

### 5.5 The arms race, stated plainly

The recurring structural facts:
- Detection improves *specifically in response to* evasion (JA3 → JA4 exists because browsers randomized to defeat fingerprinting).
- Per-customer ML models (DataDome's "85,000+") mean no portable, durable bypass.
- Tools go stale on a timescale of months (rebrowser-playwright dead since Sept 2024; "the plugin era is over").
- **Maintenance cost is recurring and unbounded**, and it is *engineering* cost — someone must diagnose novel blocks, which is unschedulable interrupt work, exactly the wrong shape for a system meant to run unattended for months.
- Evasion is **incompatible** with the legitimacy path: stable IPs + reverse DNS + stable UA (required for verification and for higher API tiers) versus rotating residential IPs + spoofed fingerprints. You cannot do both from one egress.

---

## 6. Legal and policy landscape

> **This is information, not legal advice.** Retain counsel for any specific system. Scraping law is unsettled, differs by jurisdiction, and several key holdings below are district-court decisions that are not binding precedent and may be appealed or distinguished.

### 6.1 CFAA: Van Buren narrowed it

**Van Buren v. United States**, 593 U.S. 374, decided **June 3, 2021**, **6–3**, Barrett J. (Thomas, Roberts, Alito dissenting). [Opinion PDF](https://www.supremecourt.gov/opinions/20pdf/19-783_k53l.pdf)

Holding on "exceeds authorized access": the Court adopted a **"gates-up-or-down"** inquiry. "An individual 'exceeds authorized access' when he accesses a computer with authorization but then obtains information located in particular areas of the computer—such as files, folders, or databases—that are off limits to him."

The Court **rejected** the government's purpose-based reading under which violating a use policy is a CFAA violation. Van Buren's conduct violated department policy but not the statute, because he accessed information within his authorized scope.

**Effect on scraping:** substantially undercuts the theory that violating a website's ToS is a federal crime/tort under the CFAA. It does **not** immunize accessing password-protected or technically-gated areas, or continuing after a technical block.

### 6.2 hiQ Labs v. LinkedIn — the full arc, including the part usually omitted

The case is very frequently cited for the 2019/2022 pro-scraper CFAA rulings while **omitting that hiQ ultimately lost and paid.** Both halves matter.

| Date | Event |
|---|---|
| 2017 | N.D. Cal. preliminary injunction for hiQ, 273 F. Supp. 3d 1099 |
| **2019-09-09** | 9th Cir. affirms injunction, **938 F.3d 985** — scraping public data likely not CFAA "without authorization" |
| **2021-06-14** | SCOTUS **GVR**s in light of *Van Buren*, 141 S. Ct. 2752 |
| **2022-04-18** | 9th Cir. on remand **reaffirms**, **31 F.4th 1180** — CFAA "without authorization" does not apply to public websites; LinkedIn "took no steps to demarcate their data as private or require the use of an authorization system" |
| **Nov 2022** | N.D. Cal. (**Judge Chen**) grants summary judgment **for LinkedIn on breach of contract** — hiQ breached the User Agreement via automated scraping *and* by hiring crowdsourced workers to create fake profiles; hiQ expressly agreed to the User Agreement when it created a corporate account |
| **~2022-12-06** | Stipulation and **consent judgment**: permanent injunction against scraping; deletion of all source code, data and algorithms; **$500,000 to LinkedIn**; hiQ stipulated to CFAA and California-equivalent liability (non-precedential) |

Sources: [Zwillgen, "hiQ v. LinkedIn Wrapped Up"](https://www.zwillgen.com/alternative-data/hiq-v-linkedin-wrapped-up-web-scraping-lessons-learned/); [Proskauer](https://newmedialaw.proskauer.com/2022/12/08/hiq-and-linkedin-reach-proposed-settlement-in-landmark-scraping-case/); [Morgan Lewis](https://www.morganlewis.com/blogs/sourcingatmorganlewis/2022/12/linkedin-v-hiq-landmark-data-scraping-suit-provides-guidance-to-data-scrapers-and-web-operators); [Wikipedia](https://en.wikipedia.org/wiki/HiQ_Labs_v._LinkedIn).

**The lesson:** the CFAA route closed; **breach of contract became the surviving theory.** hiQ won the famous constitutional-ish fight and lost the case. Note also the aggravating fact — fake accounts — which is doing real work in the outcome and is often dropped from summaries.

### 6.3 Meta Platforms v. Bright Data — contract theory narrowed for logged-out access

**Meta Platforms, Inc. v. Bright Data Ltd.**, N.D. Cal., **Judge Edward Chen** (same judge as hiQ), **2024-01-23**, 2024 WL 251406.

Meta sued for breach of contract over post-termination scraping. Chen granted summary judgment **for Bright Data**:
- Meta's terms apply to "users" (account holders); non-logged-in access falls outside that definition.
- "there is a strong and compelling argument that Bright Data was not 'using' Facebook as contemplated by the Terms when it scraped public data while not logged-in."
- Meta's survival clause was held unenforceable as perpetual, lacking reasonable time or geographic limits.

Sources: [Eric Goldman's Technology & Marketing Law Blog (guest post)](https://blog.ericgoldman.org/archives/2024/01/game-on-bright-data-scores-major-victory-in-web-scraping-dispute-with-meta-guest-blog-post.htm); [Farella Braun + Martel](https://www.fbm.com/publications/major-decision-affects-law-of-scraping-and-online-data-collection-meta-platforms-v-bright-data/); [Quinn Emanuel](https://www.quinnemanuel.com/the-firm/news-events/client-alert-meta-v-bright-data-significant-decision-for-web-scraping-industry/); [Lowenstein Sandler](https://www.lowenstein.com/news-insights/publications/client-alerts/meta-v-bright-data-ruling-has-important-implications-for-webscraping-activities-by-investment-advisers-im).

**Caveats flagged by commentators:** district court decision, not binding precedent; other doctrines (CFAA, trespass to chattels) were not resolved; open question whether password protection is now effectively the only reliable access restriction. Meta subsequently dismissed its remaining claims ([Proskauer release](https://www.proskauer.com/release/proskauer-secures-dismissal-of-scraping-claims-against-bright-data); exact terms/prejudice status not confirmed in accessible sources).

**The logged-in / logged-out line is the single most practically useful distinction in current US scraping law.** Creating an account and agreeing to ToS materially changes your exposure (hiQ), while logged-out access to public pages does not (Meta v. Bright Data).

### 6.4 X Corp. v. Bright Data — copyright preemption, and the survival of trespass

**X Corp. v. Bright Data Ltd.**, N.D. Cal., No. 3:23-cv-03698.

- **2024-05-09 ruling:** X's breach-of-contract and state-law claims **preempted by federal copyright law** under a conflict-preemption theory — enforcing the ToS would let X "entrench its own private copyright system that rivals, even conflicts with, the actual copyright system." The court's three conflicts: X holds only a non-exclusive license so cannot claim "greater rights than it is entitled under the Copyright Act"; enforcement would defeat fair use; and it would protect non-copyrightable content (e.g. short comments), shrinking the public domain. ([Skadden analysis](https://www.skadden.com/insights/publications/2024/05/district-court-adopts-broad-view); [Columbia Blue Sky Blog](https://clsbluesky.law.columbia.edu/?p=60360))
- **2024-11-26 ruling on the amended complaint:** **claims tied to server impairment survived** — trespass to chattels and breach of contract proceeded, as did anti-hacking claims, on allegations that automated scraping "overwhelmed servers, causing system glitches" and forced purchase of additional capacity. Data scraping/resale claims remained preempted; unfair competition rejected. ([report](https://tagteam.harvard.edu/hub_feeds/3624/feed_items/12701108/content))

**This is the most operationally important legal holding in this entire document.** It ties legal exposure directly to §4's engineering: **the surviving theory is server burden.** A crawler that demonstrably stays under published limits, honors `Retry-After`, backs off on 429/503, caches aggressively, and uses bulk channels is not just being polite — it is directly negating the elements of the claim most likely to succeed against it. Politeness engineering *is* legal risk mitigation.

### 6.5 EU: DSM Directive TDM exceptions

**Directive (EU) 2019/790** on copyright in the Digital Single Market, **17 April 2019**, in force **6 June 2019**, transposition deadline **7 June 2021**. ([EUR-Lex](https://eur-lex.europa.eu/eli/dir/2019/790/oj/eng); [Kluwer Copyright Blog analysis](https://legalblogs.wolterskluwer.com/copyright-blog/the-new-copyright-directive-text-and-data-mining-articles-3-and-4/))

**Article 3 — research/cultural heritage TDM:**
- Beneficiaries: **research organisations** (non-profit or public-service) and **cultural heritage institutions**.
- Permits "acts of reproduction and extraction" for TDM, plus secure storage and retention of copies for verification of scientific research.
- Requires **"lawful access"** — subscriptions, open licences, or freely available online content.
- **Cannot be overridden by contract** (Art. 7(1) renders contrary contractual provisions unenforceable). This is the crucial feature: for a qualifying research organisation, a publisher's ToS "no TDM" clause is void as to Art. 3 mining.
- Rightsholders may apply measures to protect "security and integrity of the networks and databases" but only as "necessary to achieve that objective" — **not** for commercial reasons. (Arguably this is where rate limiting is legitimate and blanket anti-TDM blocking is not.)
- No compensation required.

**Article 4 — general TDM exception:**
- Beneficiaries: **everyone**, including commercial actors.
- Permits reproduction and extraction for TDM, including retention.
- **Subject to opt-out:** rightsholders may reserve rights "in an appropriate manner, such as **machine-readable means** in the case of content made publicly available online" — metadata, robots.txt, or contractual terms.

**Consequence:** the legal position of an identical crawler can differ sharply depending on whether the operator qualifies as a "research organisation." If it does, Art. 3 is far stronger than Art. 4 and is contract-proof.

**Live disagreement on the opt-out:** whether Art. 4(3) is workable is actively contested — see [JIPLP, "text and data mining opt-out in Article 4(3) CDSMD: Adequate veto right for rightholders or a suffocating blanket for European artificial intelligence innovations?"](https://academic.oup.com/jiplp/article/19/5/453/7614898), [Creative Commons' critical statement](https://creativecommons.org/wp-content/uploads/2021/12/CC-Statement-on-the-TDM-Exception-Art-4-DSM-Final.pdf), and [Kluwer, "The TDM Opt-Out in the EU – Five Problems, One Solution"](https://legalblogs.wolterskluwer.com/copyright-blog/the-tdm-opt-out-in-the-eu-five-problems-one-solution/). A Dutch court held in Feb 2025 that a TDM opt-out **must** be by machine-readable means ([IPKat](https://ipkitten.blogspot.com/2025/02/dutch-court-holds-that-tdm-opt-out-must.html)) — which elevates robots.txt from etiquette to legal signal in the EU.

### 6.6 GDPR / personal data

[EDPB guidance on web scraping for generative AI, adopted **2026-07-08**](https://www.edpb.europa.eu/news/edpb-sheds-light-on-anonymisation-and-web-scraping-for-generative-ai-and-adopts-final-version_en) provides "further clarifications and examples on the use of the legitimate interest legal basis in the specific context of web scraping for AI training."

Safeguards indicated:
- Scrape only from "reliable sources, recording the timestamp, and validating the data before using them in AI training."
- Comply with **accuracy** and **data minimisation**; pay "particular attention" to **purpose limitation** and **transparency**.
- Individual notification may be excused where "impossible or require excessive effort" (Art. 14(5)(b)).
- **Special categories:** "processing special categories of personal data is in principle prohibited" — requires both an Art. 6 basis and an Art. 9(2) exception. For incidental/residual collection, controllers may rely on appropriate technical and organisational measures to prevent collection and dissemination, but "**there is no general exemption**" and "each case must be assessed individually."

Note commentary disagreement on feasibility: [Reed Smith, "EDPB web scraping guidelines for AI: Making the impossible possible?"](https://www.reedsmith.com/our-insights/blogs/technology-law-dispatch/102nbqu/edpb-web-scraping-guidelines-for-ai-making-the-impossible-possible/).

**Relevance here:** most government/scientific data is non-personal (statistics, filings, sequences, metadata). But **author names, affiliations and emails in PubMed/Crossref/OpenAlex/arXiv metadata are personal data**, as are named individuals in SEC filings. GDPR can attach to a "public data only" corpus, and this is routinely underestimated.

### 6.7 CCPA / CPRA

The CCPA excludes "publicly available information" from "personal information." Originally limited to information "lawfully made available from federal, state, or local government records." **CPRA (effective 2023-01-01)** expanded it to information a business "has a reasonable basis to believe is lawfully made available to the general public by the consumer or from widely distributed media, or by the consumer; or information made available by a person to whom the consumer has disclosed the information if the consumer has not restricted the information to a specific audience." Biometric information collected without the consumer's knowledge is expressly excluded from the exemption. ([TrueVault summary](https://www.truevault.com/blog/what-is-publicly-available-information-under-the-ccpa); Cal. Civ. Code §1798.140)

Under this, much scraped public web data plausibly falls **inside** the exemption — a notably more permissive posture than GDPR. Privacy settings matter ("not restricted to a specific audience"). Reform pressure exists: [Michigan Law Review, "Unfair Collection: Reclaiming Control of Publicly Available Personal Information from Data Scrapers"](https://michiganlawreview.org/journal/unfair-collection-reclaiming-control-of-publicly-available-personal-information-from-data-scrapers/); [Oakland Privacy brief on amending the PAI exemption (May 2026)](https://oaklandprivacy.org/wp-content/uploads/2026/05/pai-brief-1.pdf).

**US/EU divergence is a genuine, unresolved conflict**, not a detail: the same scraped author-metadata corpus is largely exempt in California and squarely regulated in the EU.

### 6.8 US federal works are public domain — but that's narrower than it sounds

**17 U.S.C. § 105**: "Copyright protection under this title is not available for any work of the United States Government," though the government may hold copyrights transferred by assignment or bequest. "Work of the United States Government" means "a work prepared by an officer or employee of the United States Government as part of that person's official duties" (§101). ([govinfo](https://www.govinfo.gov/content/pkg/USCODE-2020-title17/html/USCODE-2020-title17-chap1-sec105.htm))

**Important limits:**
- **Federal only.** State and local government works are **not** covered and are frequently copyrighted.
- Does not cover **contractor-produced** works, or privately-authored works merely published by the government.
- A 2019 amendment carves out civilian faculty at certain defense institutions.
- ⚠️ **Copyright status is not access permission.** SEC filings are public domain, but SEC still rate-limits at 10 req/s and blocks unclassified bots. Public-domain content can sit behind ToS, rate limits, and trespass-to-chattels exposure. **§6.4's server-burden theory applies to public-domain data too.**
- See also [resources.data.gov Open Licenses](https://resources.data.gov/open-licenses/) and [ARL issue brief on the copyright status of federal government works](https://www.arl.org/wp-content/uploads/2015/06/copyright-status-of-government-works.pdf).

### 6.9 Net legal picture

- **CFAA:** weak against logged-out public scraping post-*Van Buren*/*hiQ*. Still live for gated/authenticated content and for continuing after a technical block.
- **Breach of contract:** the surviving theory — **but** only where a contract was actually formed (hiQ: account created; Meta: no account, so no contract) and only where not preempted (X Corp.).
- **Copyright preemption:** an increasingly effective *defense* against ToS claims (X Corp., May 2024).
- **Trespass to chattels:** **rising**, and keyed to demonstrable server harm (X Corp., Nov 2024). Directly mitigated by politeness engineering.
- **Privacy law:** independent of all the above and often the binding constraint; GDPR materially stricter than CCPA.
- **EU TDM:** Art. 3 is a strong, contract-proof exception for qualifying research organisations; Art. 4 is opt-out-defeasible.

---

## 7. Ethics and norms for research scraping

### 7.1 Fiesler et al.

**"No Robots, Spiders, or Scrapers: Legal and Ethical Regulation of Data Collection Methods in Social Media Terms of Service"**, Casey Fiesler, Nathan Beard, Brian C. Keegan, **ICWSM 2020**. ([AAAI](https://ojs.aaai.org/index.php/ICWSM/article/view/7290); [author's accessible summary on Medium](https://cfiesler.medium.com/spiders-and-crawlers-and-scrapers-oh-my-law-and-ethics-of-researchers-violating-terms-of-service-27496894f6de))

Empirical basis: analysis of data-collection provisions in **116 social media sites' ToS**. Findings: provisions are **vague and inconsistent** across platforms, making it hard for researchers to know what is prohibited; and they **lack context** — ignoring what data is collected, by whom, for what purpose, and with what potential harms.

Fiesler's normative position: **"law and ethics are not the same thing."** She rejects *both* the claim that ToS violation is inherently unethical *and* the claim that ToS compliance is the only ethical concern (jaywalking analogy: breaking a rule for ethically justified reasons can be defensible). Recommendations: contextual analysis of each study; consider community norms and user expectations; **document ethical reasoning transparently in methods sections**; consult gatekeepers or community members. Ethics "requires more work" than following clear rules.

### 7.2 Brown et al. 2025 — the current comprehensive institutional guidance

**"Web scraping for research: Legal, ethical, institutional, and scientific considerations"**, Megan A. Brown, Andrew Gruen, Gabe Maldoff, Solomon Messing, Zeve Sanderson, Michael Zimmer, ***Big Data & Society*, 2025-11-20**. ([SAGE, DOI 10.1177/20539517251381686](https://journals.sagepub.com/doi/10.1177/20539517251381686))

The most current comprehensive framework found. Four dimensions:

- **Legal:** prefer publicly accessible sites without authentication — "websites and services that are accessible to the general public without requiring authentication entail significantly lower legal risk." Collect the minimum necessary; secure it properly. Notes courts increasingly limit browsewrap enforceability, but cautions that contract and CFAA claims remain possible. *(This aligns exactly with the Meta v. Bright Data logged-out line.)*
- **Ethical:** reflexivity — **"remember the human"**; assess contextual appropriateness; consider power dynamics; public availability does not make collection ethically appropriate, especially for sensitive topics or vulnerable populations.
- **Institutional:** **"engage their IRB early and often"**; also work with general counsel, IT services, and external bodies (e.g. the Social Media Archive). IRBs often deem public online data exempt from full review, **but practice varies significantly across institutions** — so exemption cannot be assumed.
- **Scientific:** "researchers should develop deliberate sampling strategies rather than relying on convenience samples." Threats: unknown sampling frames, missing data, platform algorithm changes, construct validity. Caveat findings accordingly.
- **Server burden:** be "mindful of the impact on the platform's resources and using techniques such as rate limiting and respectful crawling behavior to minimize disruption."
- **Data sharing:** document clearly; consider limiting scope where misuse risk is high; specify licensing restrictions on shared datasets.

### 7.3 The scientific-validity angle is under-appreciated

Brown et al.'s "scientific" dimension is worth separating out because it is a *technical* requirement on the harvester, not just a caveat for the paper. If block-avoidance measures (rotating proxies, geo-varied exit nodes, partial retries, silent drops) cause **non-random missingness**, the resulting dataset is biased in ways that are invisible downstream. A harvester that silently drops 3% of records when a source rate-limits produces a corpus with unknown selection effects.

**Requirement:** log every request outcome, retain per-source completeness metrics, and record the crawl configuration alongside the data as provenance. For a months-long unattended run this is the difference between a reproducible dataset and an unusable one — and it is a reason to prefer bulk/OAI-PMH channels, which have well-defined completeness semantics, over opportunistic crawling, which does not.

---

## 8. Trade-offs, stated without a recommendation

**Path A — full legitimacy (API keys, declared UA, published limits, stable IPs, bulk channels, Verified Bot registration).**
- Cost: registration/key management per source; centralized shared rate limiting; timezone-aware scheduling; a monitored contact inbox; slower peak throughput (arXiv at 0.33 req/s, api.data.gov at 1,000/hr are hard floors).
- Benefit: higher documented limits than anonymous access; near-zero block risk on category-1 sources; negates the trespass-to-chattels theory (§6.4); eligible for verified-bot allowlisting; defensible to an IRB, to counsel, and to a source operator who emails you.
- Ceiling: cannot reach sources that genuinely refuse automated access.

**Path B — evasion (proxies, fingerprint spoofing, stealth browsers).**
- Cost: residential bandwidth ≈ $1–8/GB scaling with page weight; continuous engineering to track detection changes; tools going unmaintained on a months timescale; interrupt-driven debugging incompatible with unattended operation; per-site variance (85,000+ DataDome models) means no reusable solution.
- Risk: forfeits verified-bot eligibility and API-tier identity; ignoring robots.txt is evidence of bad faith and has legal effect in the EU under Art. 4(3); server-burden claims survive preemption; residential proxy sourcing raises independent consent problems; likely conflicts with institutional policy and IRB expectations.
- Benefit: reaches sources otherwise unreachable.

**Path C — mixed, with a hard boundary.** Legitimacy everywhere it works (which for this corpus is most of it); explicit per-source decisions, documented, for the remainder; separate egress so the two postures never contaminate each other. The main cost is governance overhead and the discipline to keep the boundary from drifting.

**The cross-cutting facts:**
- For a government/scientific corpus, Path B's addressable surface is small and Path A's is nearly the whole corpus.
- Bulk/OAI-PMH channels (§4.4) shrink the problem more than any other single measure, and simultaneously improve scientific completeness semantics.
- Published limits change — Crossref Dec 2025, OpenAlex Feb 2026, both within a year. **Any system running unattended for months must re-verify limits periodically and treat documentation as a live dependency**, and must not trust its own libraries' cached assumptions (pyalex #100).
- Politeness engineering and legal risk mitigation are the same work (§6.4).

---

## Sources

### Official API documentation (rate limits verified 2026-09-06)
- NCBI E-utilities usage guidelines — https://www.ncbi.nlm.nih.gov/books/NBK25497/
- SEC Webmaster FAQ — https://www.sec.gov/os/webmaster-faq
- SEC Internet Security Policy / Privacy & Security — https://www.sec.gov/about/privacy-information#security
- SEC robots.txt — https://www.sec.gov/robots.txt
- api.data.gov Developer Manual — https://api.data.gov/docs/developer-manual/
- api.data.gov rate limits — https://api.data.gov/docs/rate-limits/
- Crossref, Announcing changes to REST API rate limits (2025-11-05, effective 2025-12-01) — https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/
- Crossref, Tips for using the REST API — https://www.crossref.org/documentation/retrieve-metadata/rest-api/tips-for-using-the-crossref-rest-api/
- OpenAlex, "API keys required starting Feb 13" (2026-01-14) — https://groups.google.com/g/openalex-users/c/rI1GIAySpVQ
- OpenAlex Help Center, API reference — https://help.openalex.org/api/
- OpenAlex Help Center, Authentication — https://help.openalex.org/api/authentication/
- OpenAlex pricing — https://help.openalex.org/access/pricing/
- OpenAlex docs repo (STALE — still documents mailto polite pool) — https://github.com/ourresearch/openalex-docs/blob/main/how-to-use-the-api/rate-limits-and-authentication.md
- pyalex issue #100 (client library lag on rate limits) — https://github.com/J535D165/pyalex/issues/100
- arXiv API Terms of Use — https://info.arxiv.org/help/api/tou.html
- Europe PMC developers — https://europepmc.org/developers
- Europe PMC rate limits, staff reply 2020-03-24 (⚠️ dated, unofficial) — https://groups.google.com/a/ebi.ac.uk/g/epmc-webservices/c/MfxQ8nvIT5Q

### Standards
- RFC 9309, Robots Exclusion Protocol (Sept 2022, Standards Track) — https://www.rfc-editor.org/rfc/rfc9309.html
- RFC 9110, HTTP Semantics — Retry-After (June 2022) — https://www.rfc-editor.org/rfc/rfc9110.html#name-retry-after
- Google robots.txt specification — https://developers.google.com/search/docs/crawling-indexing/robots/robots_txt
- Google, Verifying Googlebot and other Google crawlers — https://developers.google.com/search/docs/crawling-indexing/verifying-googlebot
- draft-meunier-web-bot-auth-architecture (expired; superseded) — https://datatracker.ietf.org/doc/draft-meunier-web-bot-auth-architecture/

### Politeness engineering
- Scrapy AutoThrottle — https://docs.scrapy.org/en/latest/topics/autothrottle.html

### robots.txt compliance evidence
- *Scrapers Selectively Respect robots.txt Directives: Evidence From a Large-Scale Empirical Study* (arXiv 2505.21733, data Feb–Mar 2025) — https://arxiv.org/html/2505.21733

### Bot management / fingerprinting
- Cloudflare bot score — https://developers.cloudflare.com/bots/concepts/bot-score/
- Cloudflare Verified Bots — https://developers.cloudflare.com/bots/concepts/bot/verified-bots/
- FoxIO JA4+ fingerprinting suite (authors' repo; licensing terms) — https://github.com/FoxIO-LLC/ja4
- Scrapfly, DataDome detection analysis (Aug 2026; ⚠️ vendor source) — https://scrapfly.io/blog/posts/how-to-bypass-datadome-anti-scraping
- Scrapfly, PerimeterX analysis (⚠️ vendor source) — https://scrapfly.io/blog/posts/how-to-bypass-perimeterx-human-anti-scraping
- Anti-Detect Browser Benchmark, 651 verdicts (2026-05-13, upd. 2026-08-15) — https://ianlpaterson.com/blog/anti-detect-browser-benchmark-patchright-nodriver-curl-cffi/
- undetected-chromedriver (status ambiguous) — https://github.com/ultrafunkamsterdam/undetected-chromedriver
- Camoufox stealth overview — https://camoufox.com/stealth/
- "Playwright Anti-Fingerprinting Alternatives 2026: The Plugin Era Is Over" (⚠️ vendor) — https://botcloud.dev/blog/playwright-anti-fingerprinting-alternatives-2026/
- "Playwright Stealth Not Working in 2026" (⚠️ vendor) — https://humanbrowser.cloud/blog/playwright-stealth-not-working-2026

### Proxy pricing (⚠️ all vendor-published)
- Proxidize Proxy Pricing Index 2026 (2026-07-22, upd. 2026-08-24, 14 providers) — https://proxidize.com/research/proxy-pricing-index-2026/
- Proxidize residential proxy pricing — https://proxidize.com/blog/residential-proxy-pricing/
- DataImpulse residential proxy pricing comparison — https://dataimpulse.com/blog/residential-proxy-pricing-comparison/

### Legal — primary
- Van Buren v. United States, 593 U.S. 374 (2021-06-03) — https://www.supremecourt.gov/opinions/20pdf/19-783_k53l.pdf
- Directive (EU) 2019/790 (DSM Directive) — https://eur-lex.europa.eu/eli/dir/2019/790/oj/eng
- 17 U.S.C. § 105 — https://www.govinfo.gov/content/pkg/USCODE-2020-title17/html/USCODE-2020-title17-chap1-sec105.htm
- EDPB, web scraping & anonymisation guidance (adopted 2026-07-08) — https://www.edpb.europa.eu/news/edpb-sheds-light-on-anonymisation-and-web-scraping-for-generative-ai-and-adopts-final-version_en

### Legal — analysis
- Zwillgen, hiQ v. LinkedIn Wrapped Up — https://www.zwillgen.com/alternative-data/hiq-v-linkedin-wrapped-up-web-scraping-lessons-learned/
- Proskauer, hiQ/LinkedIn settlement — https://newmedialaw.proskauer.com/2022/12/08/hiq-and-linkedin-reach-proposed-settlement-in-landmark-scraping-case/
- Morgan Lewis, LinkedIn v. hiQ guidance — https://www.morganlewis.com/blogs/sourcingatmorganlewis/2022/12/linkedin-v-hiq-landmark-data-scraping-suit-provides-guidance-to-data-scrapers-and-web-operators
- Wikipedia, HiQ Labs v. LinkedIn (procedural dates/citations) — https://en.wikipedia.org/wiki/HiQ_Labs_v._LinkedIn
- Eric Goldman blog, Bright Data v. Meta (guest post, Jan 2024) — https://blog.ericgoldman.org/archives/2024/01/game-on-bright-data-scores-major-victory-in-web-scraping-dispute-with-meta-guest-blog-post.htm
- Farella Braun + Martel, Meta v. Bright Data — https://www.fbm.com/publications/major-decision-affects-law-of-scraping-and-online-data-collection-meta-platforms-v-bright-data/
- Quinn Emanuel, Meta v. Bright Data client alert — https://www.quinnemanuel.com/the-firm/news-events/client-alert-meta-v-bright-data-significant-decision-for-web-scraping-industry/
- Lowenstein Sandler, Meta v. Bright Data implications — https://www.lowenstein.com/news-insights/publications/client-alerts/meta-v-bright-data-ruling-has-important-implications-for-webscraping-activities-by-investment-advisers-im
- Proskauer, dismissal of scraping claims against Bright Data — https://www.proskauer.com/release/proskauer-secures-dismissal-of-scraping-claims-against-bright-data
- Skadden, copyright preemption in X Corp. v. Bright Data (May 2024) — https://www.skadden.com/insights/publications/2024/05/district-court-adopts-broad-view
- Columbia Blue Sky Blog on same — https://clsbluesky.law.columbia.edu/?p=60360
- X Corp. v. Bright Data, trespass to chattels survives (2024-11-26) — https://tagteam.harvard.edu/hub_feeds/3624/feed_items/12701108/content
- Kluwer Copyright Blog, DSM Arts. 3 & 4 — https://legalblogs.wolterskluwer.com/copyright-blog/the-new-copyright-directive-text-and-data-mining-articles-3-and-4/
- JIPLP, Art. 4(3) TDM opt-out critique — https://academic.oup.com/jiplp/article/19/5/453/7614898
- Creative Commons statement on the Art. 4 TDM exception — https://creativecommons.org/wp-content/uploads/2021/12/CC-Statement-on-the-TDM-Exception-Art-4-DSM-Final.pdf
- Kluwer, The TDM Opt-Out in the EU – Five Problems, One Solution — https://legalblogs.wolterskluwer.com/copyright-blog/the-tdm-opt-out-in-the-eu-five-problems-one-solution/
- IPKat, Dutch court on machine-readable TDM opt-out (Feb 2025) — https://ipkitten.blogspot.com/2025/02/dutch-court-holds-that-tdm-opt-out-must.html
- Reed Smith, EDPB web scraping guidelines critique — https://www.reedsmith.com/our-insights/blogs/technology-law-dispatch/102nbqu/edpb-web-scraping-guidelines-for-ai-making-the-impossible-possible/
- TrueVault, "publicly available information" under CCPA — https://www.truevault.com/blog/what-is-publicly-available-information-under-the-ccpa
- Michigan Law Review, Unfair Collection — https://michiganlawreview.org/journal/unfair-collection-reclaiming-control-of-publicly-available-personal-information-from-data-scrapers/
- Oakland Privacy, PAI exemption brief (May 2026) — https://oaklandprivacy.org/wp-content/uploads/2026/05/pai-brief-1.pdf
- resources.data.gov, Open Licenses — https://resources.data.gov/open-licenses/
- ARL, Copyright Status of U.S. Federal Government Works — https://www.arl.org/wp-content/uploads/2015/06/copyright-status-of-government-works.pdf

### Ethics
- Fiesler, Beard & Keegan, "No Robots, Spiders, or Scrapers", ICWSM 2020 — https://ojs.aaai.org/index.php/ICWSM/article/view/7290
- Fiesler, accessible summary — https://cfiesler.medium.com/spiders-and-crawlers-and-scrapers-oh-my-law-and-ethics-of-researchers-violating-terms-of-service-27496894f6de
- Brown, Gruen, Maldoff, Messing, Sanderson & Zimmer, "Web scraping for research", *Big Data & Society* (2025-11-20) — https://journals.sagepub.com/doi/10.1177/20539517251381686



---

# 05 — Catching Bad Data Before It Enters the Database

Research date: **2026-09-06**. Scope: validation, silent-failure detection, statistical checks, tooling, architecture, data contracts, parser regression testing, and provenance — for a fleet of long-running scrapers/API clients hitting government and scientific sources.

Standing caveat on staleness: several key sources below are 2018–2023 and are flagged inline. The tooling landscape moved a lot in 2024–2026 (GX 1.x rewrite, Soda Core v4, dbt-expectations unmaintained, Polars-native validators appearing). Treat any tool comparison older than ~18 months as directional only.

---

## 1. Validation layers, and where each belongs

### 1.1 The layering that practitioners converge on

Nobody serious puts all validation in one place. The recurring shape is roughly:

| Layer | Where | Catches | Typical tech |
|---|---|---|---|
| Transport/response validation | In the fetcher, before parsing | non-200, redirect walls, tiny "challenge stub" bodies, vendor block markers | plain code + HTTP client |
| Structural/parse validation | Immediately after parse, per record | missing fields, wrong types, out-of-range, regex/enum violations | Pydantic, JSON Schema, Pandera, Spidermon item validation |
| Batch/job-level validation | End of a scrape job, before write | item count floors, field coverage %, validation-error rate | Spidermon monitors, custom |
| Dataset/statistical validation | On the landed batch, before publish | row-count deltas, null-rate spikes, distribution drift, freshness | Deequ, Soda, GX, Elementary, dbt tests |
| Cross-dataset/referential | After load, in warehouse | FK violations, orphan rows, cross-source consistency | SQL, SodaCL `reference`, dbt `relationships` |

The most useful framing of "where checks belong" is Ananth Packkildurai's WAP-based argument in *An Engineering Guide to Data Quality — A Data Contract Perspective (Part 2)*: validation is a distinct **audit phase**, and "the system can fail or apply a circuit breaker in the pipeline if the data contract fails." He also lands a critique worth keeping: most tools "lack data quality as a first-class semantics" — dbt and Airflow treat validation as bolted-on. https://www.dataengineeringweekly.com/p/an-engineering-guide-to-data-quality

### 1.2 Record-level schema validation options

**Pydantic** — best fit for per-record validation right at the parser boundary. The DEV article *Stop Silent Scraper Failures: Using Pydantic for Instant Layout Change Detection* shows the practical pattern: `@field_validator(mode='before')` to normalise known format variants (commas, "1.2k" suffixes, currency symbols) and let everything else raise `ValidationError` with field-level diagnostics rather than silently coercing. It proposes an **error budget**: if validation failures exceed ~15% of items in a run, treat it as a systematic layout change and alert, rather than dropping items one by one. https://dev.to/withatte/stop-silent-scraper-failures-using-pydantic-for-instant-layout-change-detection-4p1k

**JSON Schema** — what Spidermon uses for Scrapy item validation. Settings: `SPIDERMON_VALIDATION_SCHEMAS`, `SPIDERMON_VALIDATION_ADD_ERRORS_TO_ITEMS`, `SPIDERMON_VALIDATION_DROP_ITEMS_WITH_ERRORS`, `SPIDERMON_VALIDATION_ERRORS_FIELD`. Crucially, errors land in job stats under `spidermon/validation/fields/errors/*`, so a *monitor* can assert on the error **rate**, not just individual items. Default behaviour is keep-with-error-metadata; dropping is opt-in. https://spidermon.readthedocs.io/en/latest/item-validation.html

**Pandera** — dataframe-level, statistical checks in addition to column checks; supports Pandas, Polars, Arrow. Trade-off flagged by both Arthur Turrell and the Pointblank team: it **raises on failure rather than producing a report**, and it stops at the first failure rather than reporting everything at once. https://aeturrell.com/blog/posts/the-data-validation-landscape-in-2025/ · https://posit-dev.github.io/pointblank/blog/validation-libs-2025/

**Avro / Protobuf + a schema registry** — relevant only if the harvest goes through a queue. The concrete lever is Confluent's compatibility modes: `BACKWARD` (default, non-transitive — only checked against the latest version), `BACKWARD_TRANSITIVE`, `FORWARD`, `FULL`, `FULL_TRANSITIVE`, `NONE`. BACKWARD allows adding optional fields and removing fields; FORWARD allows adding fields and removing optional ones; FULL allows only optional add/remove. https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html

**The enforcement caveat.** Robert Allen (2026-04-29) argues most "contract" tooling never actually enforces anything — Confluent Schema Registry Community is **client-side only** ("anything that bypasses the client library... sidesteps the entire enforcement chain"; broker-side validation is a paid Enterprise feature), and DataHub stores contract objects but needs external runners: "You have only declared an intention to enforce." He names Redpanda Data Transforms (WASM at the broker), OpenMetadata 1.9+, `datacontract-cli`, and Buf as things that do enforce. https://zircote.com/blog/2026/04/most-data-contract-tools-dont-enforce-contracts/

### 1.3 Referential checks

SodaCL has a first-class `reference` check: "validate that the values in a column in a table are present in a column in a different table" (https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/reference.md). dbt has `relationships`. Deequ's paper classifies these as **inter-relation consistency constraints** (see §3.2). For a scraping fleet, the analogue is cross-engine consistency: does every detail-page record trace back to a listing-page ID you actually saw?

---

## 2. What schema validation does NOT catch

This is the crux for scrapers. A well-formed empty string passes `str`. A "no results" page parses fine. Practitioners handle this with a **cheap-first cascade** of non-schema checks.

### 2.1 The canonical failure catalogue

Ficstar's *Why Scraping Fails Silently and Why That's Worse Than Crashing* enumerates:
- **Selector drift** after a redesign — selectors still match, but the wrong elements. Their example: "a product extractor starts mapping category names into the product title field." Schema validation passes; every field is a valid string.
- **Bait pages / soft blocks** — anti-bot systems return truncated or placeholder content with **HTTP 200**.
- **JavaScript rendering gaps** — headless timing issues return empty or placeholder values.
- **Proxy/routing problems** — rate-limited or cached duplicate content returned as 200.

They claim a 2% silent error rate creates "massive business loss" at their scale (they cite >1 billion product prices/month, 50+ quality checks per delivery). https://www.ficstar.com/why-scraping-fails-silently

Context.dev's production monitoring guide adds: empty results with 200; type mismatches (price arriving as free text); widespread nulls; **login page returned** instead of content; malformed-but-schema-valid records. https://www.context.dev/blog/scraper-monitoring-in-production (2026-08-16)

### 2.2 The cheap-first detection cascade (best single source)

Crawlex, *Scraping observability: success metrics, block-rate dashboards, and silent failures* (2026-03-14) is the most concrete thing I found. It opens with a case study of **nine days of null prices marked "success"** because HTTP 200 masked a JavaScript challenge. Its detection ladder:

| Check | Cost | Catches |
|---|---|---|
| Status code ≠ 200 | free | honest blocks (403, 429, 503) |
| **Final URL ≠ requested URL** | free | redirect walls |
| **Response size below a floor** | free | challenge stubs — "typically 1–2KB vs tens of KB for real content" |
| Vendor block markers (DataDome cookies/headers) | free | named soft blocks |
| Canary validation + schema checks | medium | decoy data, selector drift |
| Crawl-to-crawl diff | medium | distributional anomalies |

Its metrics framing is **RED + 2**: request rate, transport error rate, **block rate split out from transport errors**, duration distribution (p50/p99, not means), and **yield** — "useful extracted records to attempted requests." The line that matters: *"A request can succeed at every layer RED inspects and yield nothing."* It argues against static thresholds in favour of burn-rate alerts on yield using EWMA/CUSUM to absorb seasonality, and to page on yield collapse rather than on individual block types. https://blog.crawlex.net/blog/scraping-observability/

### 2.3 Specific failure modes → specific detectors

- **Layout change → empty-but-valid strings.** Field coverage monitoring. Spidermon's `FieldCoverageMonitor` with `SPIDERMON_FIELD_COVERAGE_RULES` (plus `SPIDERMON_FIELD_COVERAGE_TOLERANCE`, `SPIDERMON_LIST_FIELDS_COVERAGE_LEVELS`) asserts *what fraction of items had each field populated*. This is the single highest-value scraper-specific check and it is not a schema check. https://spidermon.readthedocs.io/en/latest/monitors.html
- **"No results" page that parses fine.** Item-count floors (`SPIDERMON_MIN_ITEMS`) plus response-size floors plus canaries. Context.dev: "row counts compared against expected minimums or recent baselines."
- **Pagination silently truncates.** Compare items-per-run against the historical baseline, and cross-check against the site's own reported total where available. Deequ-style `Size` metric with a relative-rate-of-change strategy (§3.2) is the generic version.
- **Numeric field starts arriving as "N/A".** Null-rate monitoring on the *post-coercion* column. Context.dev gives the concrete number: "a jump in null prices from 2 percent to 70 percent usually indicates extraction failure," and gives 20% as an example alert threshold on price null rate.
- **Units / currency change.** Distribution checks (min/max/mean vs history) catch a 100× or 1.1× shift that range checks won't, because the value stays "plausible." Deequ's `Minimum`/`Maximum`/`Mean`/`ApproxQuantile` metrics over a metrics repository are exactly this.
- **Encoding mojibake.** `ftfy` has a purpose-built heuristic: ~400 Unicode characters that occur in UTF-8 mojibake, a `badness(text)` score counting "unlikely character sequences" (e.g. "lowercase accented letters followed immediately by currency symbols"), and a fast `is_bad(text)` regex early-exit. Also `fix_and_explain()` to see what it changed. https://ftfy.readthedocs.io/en/latest/heuristic.html · https://alexwlchan.net/notes/2025/ftfy-fix-and-explain/ — run `badness()` as a *metric* per batch, not just as a fixer.
- **Date format flips (DD/MM ↔ MM/DD).** No library reliably detects this from a single value; the practical detection is statistical — the fraction of parsed dates with day-of-month > 12 should be roughly stable (~60% of days in a uniform month); if a source flips format, values ≤12 stay parseable and the >12 ones fail or shift. Monitor (a) parse-failure rate on the date column and (b) the day-of-month histogram. Also monitor max/min date vs "now" (freshness bound).
- **A whole section quietly drops.** Field coverage + schema-diff of the parsed record's key set against the historical key set. Soda's `schema` check validates "column presence, absence, or position in a table, or the type of data" — https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/schema.md

### 2.4 The semantic gap is real and acknowledged

Zyte's 2018 enterprise QA post is blunt that structural automation only goes so far: "Verifying the semantics of textual information... is still a challenge for automated QA as of today," requiring manual QA. Their framework splits **quality/correctness** (fields taken from the *correct* page elements) from **coverage** (item coverage and field coverage). Four layers: pipelines → Spidermon → manually-executed automated tests → manual/visual QA. https://www.zyte.com/blog/data-quality-assurance-for-enterprise-web-scraping/ (2018-09-27, **stale but structurally still the standard model**) · https://www.zyte.com/blog/automated-data-qa/ (2022-11-01)

Note the honest implication: **no purely automated system catches "the right value from the wrong element."** Golden records (§7) are the only cheap mechanical defence.

---

## 3. Statistical and volume checks

### 3.1 Spidermon's job-level thresholds (concrete defaults)

From https://spidermon.readthedocs.io/en/latest/monitors.html:

| Monitor | Setting | Default |
|---|---|---|
| ItemCountMonitor | `SPIDERMON_MIN_ITEMS` | — (you set it) |
| ItemValidationMonitor | `SPIDERMON_MAX_ITEM_VALIDATION_ERRORS` | — |
| FieldCoverageMonitor | `SPIDERMON_FIELD_COVERAGE_RULES`, `SPIDERMON_FIELD_COVERAGE_TOLERANCE` | — |
| ErrorCountMonitor | `SPIDERMON_MAX_ERRORS` | — |
| FinishReasonMonitor | `SPIDERMON_EXPECTED_FINISH_REASONS` | `['finished']` |
| UnwantedHTTPCodesMonitor | `SPIDERMON_UNWANTED_HTTP_CODES_MAX_COUNT` | **10** |
| " | `SPIDERMON_UNWANTED_HTTP_CODES` | **[400, 407, 429, 500, 502, 503, 504, 523, 540, 541]** |
| DownloaderExceptionMonitor | `SPIDERMON_MAX_DOWNLOADER_EXCEPTIONS` | — |
| RetryCountMonitor | `SPIDERMON_MAX_RETRIES` | **-1** (disabled) |
| PeriodicExecutionTimeMonitor | `SPIDERMON_MAX_EXECUTION_TIME` | — |

`FinishReasonMonitor` is underrated: a Scrapy job that ends with `closespider_timeout` or `shutdown` instead of `finished` produced a *truncated* dataset that will otherwise look merely "smaller than usual."

### 3.2 Deequ: metrics repository + anomaly detection over history

Schelter, Lange, Schmidt, Celikel, Biessmann, Grafberger, *Automating Large-Scale Data Quality Verification*, **PVLDB 2018** — https://www.vldb.org/pvldb/vol11/p1781-schelter.pdf

- Dimensions: **completeness**, **consistency** (intra-relation: types, ranges, column rules; inter-relation: cross-table constraints), **accuracy** (syntactic vs semantic).
- 24+ computable metrics: `Size`, `Completeness`, `Compliance`, `Uniqueness`, `Distinctness`, `ValueRange`, `DataType`, `Predictability`, `Minimum`, `Maximum`, `Mean`, `StandardDeviation`, `CountDistinct`, `ApproxCountDistinct`, `ApproxQuantile`, `Correlation`, `Entropy`, `Histogram`, `MutualInformation`.
- **Incremental computation**: maintains state `S` plus update function `f` and metric function `g`, so metrics are computed over deltas rather than recomputing on the full growing dataset. This matters a lot for months-long continuous harvesting.
- Anomaly detection runs over "historic time series of data quality metrics"; the default `OnlineNormal` detector maintains running mean/variance and compares against a user-defined bound in standard deviations.

Concrete threshold from the Deequ anomaly example: `RelativeRateOfChangeStrategy(maxRateIncrease = Some(2.0))` — "the number of rows on a given day should not be more than double of what we have seen on the day before." https://github.com/awslabs/deequ/blob/master/src/main/scala/com/amazon/deequ/examples/anomaly_detection_example.md

### 3.3 Elementary's defaults (useful as a starting calibration)

From Paradime's mirror of Elementary docs — https://docs.paradime.io/app-help/documentation/integrations/observability/elementary-data/anomaly-detection-tests/anomaly-tests-parameters :

| Parameter | Default |
|---|---|
| `time_bucket` | `{period: day, count: 1}` |
| `anomaly_sensitivity` | **3** (standard deviations) |
| `anomaly_direction` | `both` |
| `training_period` | **14 days** |
| `detection_period` | **2 days** |
| `detection_delay` | 0 |
| `seasonality` | none |
| `timestamp_column` | none |

Important gotcha: `training_period`, `detection_period`, `time_bucket`, `seasonality` and `detection_delay` **only work if `timestamp_column` is configured**; without it, metrics are computed per-run rather than time-bucketed. Freshness anomalies default to `anomaly_direction: spike` (alert only on delays), 14-day training / 2-day detection. https://docs.paradime.io/app-help/documentation/integrations/observability/elementary-data/anomaly-detection-tests/freshness-anomalies

14 days of training is short for government/scientific sources with weekly, monthly, or quarterly release cadences — you will need seasonality configured or much longer windows.

### 3.4 Soda's anomaly detection

Uses **Facebook Prophet**. Needs **at least four measurements** at a *stable frequency* before it evaluates (returns `[NOT EVALUATED]` until then; running several scans minutes apart does not count). Defaults: window length **1000 measurements**, aggregation `last`, frequency `auto`, warning ratio **0.1**, confidence interval ratio **0.001**, directionality `upper_and_lower_bounds`. Syntax: `- anomaly detection for row_count`. https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/anomaly-detection.md — note this page is labelled **deprecated** in Soda v3 docs; verify current status before adopting.

### 3.5 Freshness

SodaCL freshness is refreshingly simple: `- freshness(start_date) < 3d`, with thresholds in `#d` / `#h` / `#m` / `#d#h` / `#h#m` form, and a `NOW` variable for deterministic testing (`soda scan ... -v NOW="2022-05-31 21:00:00"`). With alert configs only `>` is allowed: `warn: when > 3256d` / `fail: when > 3258d`. https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/freshness.md

For government sources the more meaningful freshness signal is often **source-declared** (a "last updated" field on the page or in the API), not your own ingest time — track both, and alert when your ingest advances while the source's declared date does not (you're re-ingesting a frozen page) *and* when the source's date advances but your row count doesn't move.

### 3.6 Distribution drift

Population Stability Index is the workhorse. Formula: `PSI = Σ (Actual(b) − Expected(b)) × ln(Actual(b)/Expected(b))`. Thresholds per Fiddler: **< 0.1** distributions similar; **0.1–0.2** moderately different; **> 0.2** act. Caveat: empty bins make PSI undefined/unbounded — add a smoothing value (0.01 to bin proportions, or +1 count per bin). https://www.fiddler.ai/blog/measuring-data-drift-population-stability-index

Note the common disagreement in the literature: some sources use 0.25 rather than 0.2 as the "significant shift" line. Both are rules of thumb from credit scoring, not derived thresholds — calibrate on your own history.

---

## 4. Tooling landscape, with the actual complaints

### 4.1 Great Expectations

**What it is now:** GX Core 1.x requires seven components — Data Context, Data Source, Data Asset, Batch Definition, Expectation Suite, Validation Definition, Checkpoint (documented at version **1.22.0**). https://docs.greatexpectations.io/docs/core/introduction/gx_overview

**Critiques, attributed:**
- Arthur Turrell (2025-03-05): "Perhaps because it has become production-grade, it now seems a little more difficult to configure and get going with than some of the other options." He also notes the **website heavily promotes the hosted product** and the open-source package "requires searching to find." Counterweight: he still recommends GX for production *because* of its action triggers (Slack notifications etc. on failed validation) — a feature Pointblank lacks. https://aeturrell.com/blog/posts/the-data-validation-landscape-in-2025/
- The Data Letter (2025-10-20): steep learning curve; requires understanding Data Contexts, Batch Requests, Expectation Suites; early versions relied on "complex YAML configurations," improved post-v1.0. https://www.thedataletter.com/p/tool-review-soda-core-vs-great-expectations
- Dataroots (Nov 2023, **stale**): 8,900+ GitHub stars then; "understanding and defining expectations can be a bit complex for newcomers"; **performance issues with large datasets** (Pandas limitation); **"the lack of a stable version" causes upgrade disruptions**; limited Data Docs customisation. https://dataroots.io/blog/state-of-data-quality-october-2023
- Bruno Gonzalez (2023-09-11, **stale**): OSS version "incredibly complete," biggest community; but requires substantial Python, "steep learning curve to extend" with custom assertions, and the CLI was **retired in April 2023**. https://medium.com/@brunouy/a-guide-to-open-source-data-quality-tools-in-late-2023-f9dbadbc7948
- Posit/Pointblank (June 2025): **GX "does not yet offer native Polars support."** https://posit-dev.github.io/pointblank/blog/validation-libs-2025/

**Fairest summary of the "heavyweight" complaint:** it's about the *object model and configuration ceremony*, not expressiveness. Pandera's maintainer kvnkho put it well (2021-08-31): "Great Expectations has a larger surface area when it comes to your project, but you have to opt-in to get those benefits." https://github.com/unionai-oss/pandera/discussions/598

### 4.2 Soda Core

**What it is now:** v4 rebranded as a "data quality and **data contract** verification engine" — YAML contracts validating schema and data. Package naming changed from `soda-core-{source}` to `soda-{source}`, which breaks v3 installs. https://github.com/sodadata/soda-core/blob/main/README.md

Example syntax:
```yaml
columns:
  - name: size
    checks:
      - invalid:
          valid_values: ['S', 'M', 'L']
```

**Critiques:**
- Sifflet (2025-10-24, and note **Sifflet is a competitor** — discount accordingly): "There's no field-level lineage, impact tracing, or root cause analysis"; "no profiling, no schema diffing, and no automated suggestions." Crucially: **"Key features, including alerts, dashboards, and data contracts, are gated in Soda Cloud."** Soda Core OSS is CLI-only with no centralised tracking. Also cites documentation gaps for connector support and pricing complexity at scale. https://www.siffletdata.com/blog/soda-review
- The Data Letter: constrained within a declarative framework for highly specialised logic; ease-of-use costs programmatic flexibility — but the gap is "narrower than it initially seems" thanks to custom SQL checks and Python UDFs.
- Dataroots: SodaCL is a **new syntax to learn**; SodaCore GitHub documentation "not convenient to use."

**Disagreement worth noting:** Dataroots (Nov 2023) picked **Soda as the most versatile ready-to-use solution**; Bruno Gonzalez (Sep 2023) noted Soda has a **smaller community and less extensive data source coverage** than GX. Same quarter, opposite emphasis.

### 4.3 dbt tests

- Native dbt tests are `unique`, `not_null`, `accepted_values`, `relationships` plus custom SQL. Dataroots: "not a full data quality solution... requires additional tools (dbt-expectations, Elementary, Soda) for comprehensive coverage."
- Data Settler (2023-11-26) gives the operational critique, which is the one that actually bites at scale: **alert fatigue** ("many tests in their warehouse, and not all pass" → real issues lost "in the sea of existing failures"), **threshold suppression** masking real problems (their example: a "Garbage" status value in orders went undetected because thresholds had been suppressed), creating "misleading audit assurances," and **no integrated alerting/results storage/failure metadata**. https://datasettler.com/blog/post-4-dbt-pitfalls-in-practice/
- Structural limit for this project: **dbt tests run in the warehouse, after load.** They cannot gate ingestion unless you wrap them in a WAP flow (§5).

### 4.4 dbt-expectations — flag: unmaintained

The calogica repo carries "**Note: This package is no longer actively supported.**" Latest release **0.10.4, 2024-09-10**; 1.2k stars, 54 releases. Supports Postgres, Snowflake, BigQuery, DuckDB, Spark (experimental), Trino. Test categories: table shape, missing/unique/type, sets and ranges, string matching, aggregate functions, multi-column, distributional. https://github.com/calogica/dbt-expectations · The dbt package hub now lists it under `metaplane/dbt_expectations` — https://hub.getdbt.com/metaplane/dbt_expectations/latest/ (i.e. a fork/transfer exists; verify which one is live before adopting).

### 4.5 Pandera / Patito / Pointblank / Dataframely / Validoopsie

From the Posit survey (June 2025) with GitHub star counts at the time — https://posit-dev.github.io/pointblank/blog/validation-libs-2025/ :

| Library | Stars | Strength | Weakness |
|---|---|---|---|
| Pandera | 3,838 | statistical testing, schema-centric, mypy integration | stops at first failure; no comprehensive error report |
| Patito | 468 | Pydantic integration, model-based, row-level objects, mock data gen | limited to predefined validation types; no statistical testing |
| Pointblank | 173 | interactive HTML reports, threshold management, stakeholder comms, LLM-suggested validations | less statistical; schema-agnostic step-by-step rather than schema-first; **no action triggers on failure** (Turrell) |
| Validoopsie | 63 | only one with native structured logging (loguru); impact levels low/med/high + numeric thresholds | smallest community; predefined catalog only |
| Dataframely | 319 | **collection validation across multiple DataFrames**, soft validation with failure introspection | newest (early 2025), least battle-tested; complex API |

Note the self-interest: this survey is published by the Pointblank team. It is nonetheless the most detailed head-to-head available and it does list Pointblank's own weaknesses.

Turrell's split recommendation: **Pandera** for mixed-skill teams / non-production; **GX** for production *specifically* for the action triggers; **Pydantic** for input validation. Pandera's own maintainer (cosmicBboy, 2021-08-23): "Pandera is designed to be useful with zero configuration... One thing that Pandera offers that GE doesn't is data synthesis strategies" (i.e. generate fake data from a schema — directly useful for parser fixtures).

### 4.6 Deequ / PyDeequ

Strengths: Spark-native, incremental metric computation, metrics repository as a first-class concept, constraint *suggestion* from profiling. Real cost: **it's a JVM/Spark dependency**. PyDeequ is a py4j wrapper over the Scala library — you are running Spark whether or not your data justifies it. Bruno Gonzalez excluded Deequ from his 2023 comparison as "platform-specific." For a scraper fleet producing modest per-source volumes, Spark is a heavy tax; for a consolidated multi-TB harvest lake, it is the natural fit. Also note the community fork `nielsen-oss/python-deequ` exists — https://github.com/nielsen-oss/python-deequ

### 4.7 Elementary

dbt-native: captures dbt run/test artifacts into metadata tables and layers anomaly detection tests on top. OSS CLI + report; premium features in Cloud. https://github.com/elementary-data/elementary · https://github.com/elementary-data/dbt-data-reliability — Only relevant if dbt is already in the stack; it inherits dbt's "runs after load" limitation.

### 4.8 Commercial observability: real pricing

Vendr's marketplace data (as of **February 2026**, from anonymised transaction data — Monte Carlo publishes no list pricing):
- **Median buyer pays $61,925/year** across 74 purchases (avg 17.5–18% savings vs quote).
- Small/mid-market (30–100 tables, 2–3 sources): **$25k–$60k/yr**
- Mid-market (100–300 tables, 4–6 sources): **$60k–$120k/yr**
- Enterprise (300+ tables, 6+ sources): **$120k–$250k+/yr**
- Contract minimums typically **$25k–$30k/yr**; multi-year discounts 15–30%; TCO **+15–25%** once onboarding, support and warehouse compute are counted.

https://www.vendr.com/marketplace/monte-carlo

The structural mismatch for this project: these tools price on **tables and sources**, and a many-engine harvesting system is source-heavy by construction. They also do ML-driven anomaly detection on *warehouse tables* — they will not see a scraper returning a challenge page.

### 4.9 The minimalist "just write SQL/Python assertions" camp

Start Data Engineering, *How to Implement Data Quality Checks in Python Without Third-Party Tools* (**2026-03-08**) makes the case directly: vendor solutions run **$50k–$150k annually** and are unnecessarily complex. Core claim: *"Identifying data quality checks and working with stakeholders are the hard parts; keep the implementation simple."* Pattern: DuckDB + small reusable Python functions taking `(connection, table, column, allowed_values, pass_threshold=90%, sample_limit)`; query for violations, compute failure rate, return pass/fail plus sample offending rows. Explicitly endorses WAP and **threshold-based validation rather than 100% compliance**. https://www.startdataengineering.com/post/data-quality-with-python/

Lewis Hemens (Dataform) on the older "SQL assertions" view: a test is just a query that should return zero rows. https://medium.com/dataform/testing-data-quality-with-sql-assertions-2053755395e7

**The honest counter-argument** to minimalism, from Data Settler and Dataroots: what you lose isn't the assertion, it's the *scaffolding* — results storage over time, alert routing, failure metadata, historical baselines for anomaly detection, and a report a non-engineer can read. Rolling your own means also rolling that. That said, a metrics table + a plotting notebook is not a two-year project.

---

## 5. Architectural patterns

### 5.1 Write-Audit-Publish

Bigeye's taxonomy is the cleanest framing of the whole design space — https://www.bigeye.com/blog/strategies-for-handling-bad-data-in-data-pipelines :

**Strategy 1 — prevent bad data entering** ("no data is better than bad data")
1. **Circuit breakers** (coarse): "stop dependent tasks from executing if an error threshold is met."
2. **Inline fixup/drop** (fine): split into clean and dirty paths at ingestion.
3. **Dead letter / quarantine queues** (fine): rejected rows "sent to a separate quarantine stream," valid data keeps flowing.

**Strategy 2 — accept and refine later**
1. **YOLO**: straight to production, no gates.
2. **Staging inserts**: land in temp tables, check, then atomically promote.
3. **Branches**: "create a separate version of the table that isn't 'live' for most data consumers" until validation passes.

Classical ETL prioritises prevention; ELT accepts raw and defers cleanup, which buys reprocessing ability and tolerance of semi-structured data.

**Iceberg WAP mechanics** (Dremio, 2023-05-19 — **check for API drift since**):
```sql
ALTER TABLE glue.test.salesnew SET TBLPROPERTIES ('write.wap.enabled'='true');
ALTER TABLE glue.test.salesnew CREATE BRANCH ETL_0305;
```
```python
spark.conf.set('spark.wap.branch', 'ETL_0305')
```
then audit on the branch, then publish:
```sql
CALL glue.system.cherrypick_snapshot('test.salesnew', 5073925010883267751);
```
Cherry-pick is a **metadata-only operation** — no data movement, atomic visibility. https://www.dremio.com/blog/streamlining-data-quality-in-apache-iceberg-with-write-audit-publish-branching/ · JVM-free implementation: https://github.com/BauplanLabs/no-jvm-wap-with-iceberg · Telm.ai overview: https://www.telm.ai/blog/what-is-write-audit-publish-in-apache-iceberg-and-why-it-matters-for-data-quality/

Packkildurai distinguishes **two-phase WAP** (validate in staging, then promote) from **one-phase WAP** ("zero-copy data contract validation" on lakehouse branches — Iceberg/Hudi). https://www.dataengineeringweekly.com/p/an-engineering-guide-to-data-quality

### 5.2 Circuit break vs warn-and-continue

Monte Carlo markets circuit breakers explicitly as avoiding backfill costs — https://www.montecarlodata.com/blog-announcing-circuit-breakers-a-new-way-to-automatically-stop-broken-data-pipelines-and-avoid-backfilling-costs/ (vendor content). Airflow implementations use a check task that fails the DAG — https://blog.dataengineerthings.org/data-quality-with-airflow-circuit-breakers-a-step-by-step-guide-2ae30895778e

**The disagreement is real and unresolved.** Circuit-breaking is right when downstream consumers cannot tolerate wrong values and backfills are expensive. Warn-and-continue is right when partial data has value and a halted pipeline creates a *worse* failure (a months-long unattended harvest that circuit-breaks on Friday night and misses a source's only publication window is a data loss, not a data save). For a scraper fleet the practical resolution is **per-source, per-severity**: break on "this parser is clearly broken" (yield collapse, coverage collapse, finish-reason abnormal), warn on "this batch looks unusual" — and *always* quarantine rather than discard, because the raw payload is re-parseable (§5.4).

Note also Data Settler's warning that warn-and-continue degrades into ignore-and-continue via alert fatigue.

### 5.3 Dead-letter / quarantine mechanics

Databricks provides two native primitives worth knowing as prior art: `badRecordsPath` (malformed records written to a path with the source file and reason) and the **`_rescued_data` column** in Auto Loader / `from_json`, which captures fields that didn't match the expected schema instead of dropping them. https://learn.microsoft.com/en-us/azure/databricks/lakehouse-architecture/reliability/best-practices · https://community.databricks.com/t5/data-engineering/databricks-autoloader-badrecords-path-issue/td-p/126006

The generic streaming DLQ pattern is well documented — https://medium.com/@santhoshkumarv/handling-bad-records-in-streaming-pipelines-using-dead-letter-queues-in-pyspark-265e7a55eb29

The design rule everyone repeats: a DLQ that nobody reads is a `/dev/null` with extra storage costs. Give it a row count metric and an alert of its own.

### 5.4 Medallion (bronze/silver/gold) and storing the raw payload

For this project the important part of medallion isn't the three-layer discipline, it's **bronze as an immutable, append-only, byte-faithful record of what the source actually served** — store the raw HTML/JSON/CSV bytes, not just the parsed record. That gives you:
- reparse when the parser is fixed, without re-hitting the (possibly changed, possibly gone) source;
- forensic answers to "was the site wrong or were we wrong?";
- content hashing for change detection and dedupe;
- an archive that survives the source's own removal (§8.2).

Critiques of medallion: Ananth Packkildurai's is mild and mostly about naming ("data architecture is not a medal competition") plus the argument that three batch layers don't serve operational/real-time analytics, for which he proposes a "Platinum" layer. https://www.dataengineeringweekly.com/p/revisiting-medallion-architecture — Bright Data has a web-data-specific medallion writeup: https://brightdata.com/blog/web-data/medallion-architecture (vendor content).

---

## 6. Data contracts: what they mean, and whether they apply here

### 6.1 The canonical statement

Chad Sanderson, *The Rise of Data Contracts* (**2022-08-22**): contracts are "API-like agreements between Software Engineers who own services and Data Consumers." They specify semantic entities, events, attributes and relationships; include **versioned strongly-typed schemas, SLAs, and change-management protocols**; are enforced through CI/CD, Protobuf/Kafka schemas, and column-level lineage so producers can see downstream impact. The problem framing: the "GIGO cycle," where databases are treated as *non-consensual APIs*. https://dataproducts.substack.com/p/the-rise-of-data-contracts · follow-up: https://dataproducts.substack.com/p/the-consumer-defined-data-contract

### 6.2 The critiques

- **Toby Mao (Tobiko Data), *The False Promise of dbt Contracts* (2023-04-20)** — the sharpest technical critique. Four arguments: (1) YAML contracts **duplicate information already in the SQL**; (2) they don't answer the real question, "understand all the downstream consumers of my model, whether or not what I did was breaking or non-breaking for each of them"; (3) they are **logic-blind** — changing `1 + 1` to `1 + 2` passes the contract because the schema is unchanged, though it is a breaking change; (4) versioning creates tech debt — manual model copying plus YAML, producing "spaghetti code" and maintenance of legacy versions. Note Mao sells a competing product (SQLMesh). https://www.tobikodata.com/blog/the-false-promise-of-dbt-contracts
- **Daniel Beach, *Are Data Contracts Dead?* (2024-12-02, partly paywalled)** — subtitle "were they ever alive?" His visible argument is about the ecosystem generally: new tools are "the worm on the hook of a Company bent on selling you their hosted SaaS." https://dataengineeringcentral.substack.com/p/are-data-contracts-dead · earlier: https://dataengineeringcentral.substack.com/p/are-data-contracts-for-real/comments
- **Robert Allen (2026-04-29)** — as above, the enforcement gap: publishing a contract to a catalog is not enforcement. https://zircote.com/blog/2026/04/most-data-contract-tools-dont-enforce-contracts/
- **Pro side, for balance:** Snowplow, *Why data contracts are obviously a good idea* — https://snowplow.io/blog/why-data-contracts-are-obviously-a-good-idea (vendor). Sanderson's own 53-comment LinkedIn thread is a decent snapshot of the argument as it played out: https://www.linkedin.com/posts/chad-sanderson_the-rise-of-data-contracts-activity-6967515733471735808-VHOM

### 6.3 Does any of this apply when the producer is data.gov?

**The producer-consent premise does not hold.** Sanderson's mechanism is *social and organisational*: the producer signs up, CI blocks their breaking change, lineage shows them who they'd break. None of that exists when the producer is a state agency's Drupal site or an NIH API that will be redesigned without telling you.

What survives the translation, and is genuinely valuable:
1. **The schema artefact itself** — a versioned, machine-readable declaration of what you *expect* each source to yield, checked into version control next to the parser. This is the "expected" side of every drift check in §2–3.
2. **Explicit SLAs you assert on the source** — expected publication cadence, expected row-count band, expected field coverage. You can't make the agency honour them; you *can* make violations loud.
3. **Versioning discipline** — when a source changes shape, you bump the contract version and the parser version together, and every row records which version produced it (§8).
4. **`datacontract-cli`** is the pragmatic artefact: it compiles an ODCS contract into dbt tests / SodaCL checks that actually run in CI (per Allen). That's contract-as-codegen, not contract-as-agreement.

What does not survive: enforcement, negotiation, breaking-change notification, producer accountability. Calling your schema a "contract" when only one party ever saw it is a naming choice, not a mechanism — and it's worth being clear-eyed about that internally so nobody assumes a guarantee that isn't there.

---

## 7. Golden/canary records and parser regression testing

### 7.1 Canaries

Definition per Crawlex: "a small set of records whose correct values you know and that you re-scrape on every run" — verified prices, stable profile fields. Their key point: **canaries catch deliberate data poisoning and wrong-element extraction that structural checks cannot**, because the checks are on *known correct values*, not on shape.

Context.dev's operational spec: "Choose a stable page with predictable content, run it on a schedule **through the same browser, proxy, and parsing path as production jobs**, and verify a small set of expected values." The "same path" clause is the whole trick — a canary that bypasses your proxy pool tests nothing about your proxy pool.

They also warn against naive full-page content hashing: monitor **specific selectors and DOM structure** rather than whole-page diffs, or ads and timestamps generate constant false positives.

For government/scientific sources, good canary candidates are values that are *definitionally* stable: a historical statistical release that will never be revised, a legislative record from 1998, a completed clinical trial's registration date, a fixed geographic identifier.

### 7.2 Fixture-based regression testing

**scrapy-autounit** — record/replay for Scrapy callbacks. `AutounitMiddleware` + `Recorder` capture "cassettes" (binary fixtures of request/response plus callback outputs); a `Player` replays them and diffs current callback behaviour against the recording. Settings: `AUTOUNIT_MAX_FIXTURES_PER_CALLBACK` (default 10, minimum 10), `AUTOUNIT_DONT_TEST_OUTPUT_FIELDS` (skip timestamps and other dynamic values), `AUTOUNIT_DONT_RECORD_HEADERS` (Authorization auto-excluded), `AUTOUNIT_DONT_TEST_META`, `AUTOUNIT_RECORD_SETTINGS`. Update fixtures with `autounit update`. Best practice: keep `AUTOUNIT_ENABLED` off in production so you don't silently overwrite fixtures with broken output.

**Flag: the repo was archived by the owner on 2026-06-25 and is read-only.** https://github.com/scrapinghub/scrapy-autounit — the successor/alternative most often named is **Scrapy-Testmaster**: https://github.com/ThomasAitken/Scrapy-Testmaster

**VCR.py / pytest-recording / betamax** — the generic HTTP record-replay pattern. https://github.com/kiwicom/pytest-recording · https://code.kiwi.com/articles/pytest-cassettes-forget-about-mocks-or-live-requests/

### 7.3 The staleness problem, honestly

This is the known weak spot and it is *not* solved. VCR issue #746 (opened **2019-05-08**, still discussed) is the canonical complaint: with 200+ cassettes separated from their tests, re-recording is "a little tedious"; `:all` mode updates and adds interactions but **does not remove unused ones**, which combined with `allow_unused_http_interactions: false` produces false failures. The requested `:re_record` mode still hadn't landed as of the linked discussion. Notably, **TTL-based expiry was not proposed** in that thread. https://github.com/vcr/vcr/issues/746 · https://github.com/vcr/vcr/discussions/864

Practical mitigations practitioners use (synthesising the above, no single authoritative source):
- Cap fixtures per callback (autounit's default of 10) so the set stays refreshable.
- Exclude volatile fields from the diff (`AUTOUNIT_DONT_TEST_OUTPUT_FIELDS`) so tests fail on *structure*, not on today's date.
- Run a **separate scheduled live smoke test** against canary URLs — the fixture suite proves "the parser still does what it did"; only a live fetch proves "the site still looks like the fixture." Fixtures alone give false confidence precisely because they are frozen.
- Record the fixture's capture date and alert when it exceeds an age budget.
- Pandera's **data synthesis strategies** (generate synthetic frames from a schema) are useful for testing downstream code without any fixture at all — https://github.com/unionai-oss/pandera/discussions/598

---

## 8. Provenance

### 8.1 The per-row columns

There is no single authoritative "here are the eight columns" source; this is assembled from the practices described across Crawlex, Context.dev, Zyte and the WAP/medallion literature. The set that recurs:

- **source URL** (as requested) **and final URL** (after redirects) — the delta is itself the redirect-wall signal from §2.2
- **fetch timestamp** (UTC) and **source-declared last-updated** where available
- **HTTP status**, and response size
- **content hash** of the raw payload (dedupe, change detection, "did this actually change or did we just re-fetch it")
- **raw payload pointer** (path/key into the immutable bronze store)
- **parser version** / code commit SHA — without this you cannot answer "which rows came from the buggy parser" after a fix
- **schema/contract version**
- **run ID / job ID** — joins the row back to the job stats (finish reason, item counts, coverage)
- **validation status** and any validation error payload (Spidermon's `_validation` field pattern)

### 8.2 Why this matters more for government/scientific data than for e-commerce

Because the source can vanish, and has. *Rescuing US Government Data* (2025-03-19) documents federal dataset removal and modification beginning late 2024/early 2025, from both intentional deletion (climate, public health) and administrative dismemberment (agency closures orphaning data). Concrete example given: the NIEHS Climate Change and Human Health Literature Portal at `https://tools.niehs.nih.gov/cchhl/index.cfm` was **deleted in mid-February 2025**, evidenced via the Wayback Machine; ~100 GB of CDC datasets were archived in January 2025. Preservation infrastructure named: Internet Archive/Wayback, the Data Rescue Project, DataLumos (ICPSR), Harvard Dataverse, OpenICPSR, and Harvard Law School's data.gov mirror. https://atcoordinates.info/2025/03/19/rescuing-us-government-data/ · https://www.slaw.ca/2025/11/27/the-data-rescue-project-preserving-government-data-is-a-tech-community-issue/ · https://sr.ithaka.org/blog/preserving-access-for-at-risk-public-data/ · library guides: https://libguides.umn.edu/c.php?g=1449575&p=10778647 · https://libguides.macalester.edu/disappearing-data

The operational implication for this system is direct: **if you don't store the raw payload with a hash and a fetch timestamp, "what did the source say on 2025-02-01?" becomes unanswerable** — and for scientific/government data, that question is the reproducibility question.

### 8.3 W3C PROV

W3C PROV Primer, **W3C Working Group Note, 2013-04-30** (mature/stable, not stale in the sense of abandoned — it's a finished standard). Core model: **Entity** (a thing — a page, a dataset, a row), **Activity** (a process — a fetch, a parse), **Agent** (person, software, org, bearing responsibility). Core relations: `wasGeneratedBy` (activity brought entity into existence), `used` (activity consumed an entity), `wasAttributedTo` (entity → responsible agent), `wasDerivedFrom` (entity → source entity). https://www.w3.org/TR/prov-primer/

Mapping to a scraping pipeline is natural: raw payload = Entity; fetch = Activity (with start/end time, HTTP status as attributes); scraper version = Agent; parsed record `wasDerivedFrom` raw payload; raw payload `wasGeneratedBy` the fetch Activity.

Honest assessment: **full PROV serialisation is usually more than a harvesting system needs.** Its value is as a *vocabulary and completeness checklist* — if you can answer entity/activity/agent + used/generated/derived/attributed for every row, your provenance is adequate, whether or not you ever emit PROV-O RDF. Adopt the ontology proper only if you're publishing into a scientific data commons that expects it. Extensions and the wider ecosystem: https://blogs.ncl.ac.uk/paolomissier/2021/02/07/w3c-prov-some-interesting-extensions-to-the-core-standard/ · ProvONE for workflow provenance: https://purl.dataone.org/provone-v1-dev · biomedical reproducibility application (ProvCaRe): https://www.sciencedirect.com/science/article/abs/pii/S1386505618302697

### 8.4 Frictionless Data — relevant for government tabular sources

Open Knowledge's Frictionless Data / goodtables work targets exactly the open-government-portal case: validate at "the structural level, such as missing headers and blank rows, and at the data schema level, such as wrong data types and out of range values," with Table Schema as the portable schema artefact. Caveat: the specific case study I checked announces a pilot with the Western Pennsylvania Regional Data Center but **reports no quantitative failure rates**, so don't cite it for numbers. https://blog.okfn.org/2017/12/18/validation-for-open-data-portals-a-frictionless-data-case-study/ · https://frictionlessdata.io/blog/2018/07/16/validated-tabular-data/ · https://blog.okfn.org/2017/05/31/open-data-quality-the-next-shift-in-open-data/ · fast Python implementation: https://github.com/ezwelty/goodtables-pandas-py — **all 2017–2018, stale; verify the current Frictionless v5 tooling before adopting.**

---

## 9. Explicit disagreements to carry into the decision

1. **GX vs Soda vs dbt vs roll-your-own.** Turrell recommends GX for production because of action triggers; Dataroots (2023) recommends Soda as most versatile; Bruno Gonzalez (2023) notes Soda's smaller community and thinner source coverage; Start Data Engineering (2026) says all of it is overkill and to write Python functions over DuckDB. All four are defensible; the divergence is really about whether you value the *scaffolding* (results history, alerting, reports) enough to pay in configuration ceremony.
2. **Declarative vs programmatic.** The Data Letter frames this as the real axis, not "which is better": Soda = declarative simplicity, GX = programmatic power — and argues the gap narrowed via Soda's custom SQL and Python UDFs.
3. **Do contracts help?** Sanderson yes; Mao says schema contracts are logic-blind and duplicative; Beach implies vendor-driven hype; Allen says the tools mostly don't enforce anyway. And none of them are arguing about the case here, where the producer is a third party who never agreed.
4. **Circuit-break vs warn.** Bigeye's own framing presents both as legitimate ("no data is better than bad data" vs accept-and-refine); Monte Carlo (a vendor) pushes circuit breakers; Data Settler documents how warn-mode rots into ignore-mode. No consensus.
5. **PSI threshold.** 0.2 (Fiddler) vs 0.25 (common elsewhere) for "act on this." Both are credit-scoring folklore.
6. **Anomaly training windows.** Elementary defaults to 14 days; Soda's Prophet-based check defaults to a 1000-measurement window and needs only 4 measurements to start. A 14-day baseline is plainly wrong for a quarterly government release.
7. **Statistical anomaly detection vs deterministic rules.** Zyte's position is that semantic correctness still needs human QA; Crawlex argues for burn-rate alerting on yield over static thresholds; Deequ's paper argues for constraint suggestion from profiling. These aren't mutually exclusive but they imply very different budgets.

---

## 10. Gaps and stale material to re-verify

- **Soda anomaly detection docs are marked deprecated** in the v3 reference; Soda Core v4 restructured packages and repositioned around contracts. Verify current check syntax before committing.
- **dbt-expectations is explicitly unmaintained** (last release 2024-09-10); a `metaplane/` listing exists on the dbt hub — confirm which is live.
- **scrapy-autounit archived 2026-06-25.** Evaluate Scrapy-Testmaster or a homegrown fixture harness.
- **GX**: the 2023 critiques (unstable versioning, Pandas performance, CLI removal) predate the 1.x rewrite; the 2025 critiques (config burden, hosted-product emphasis, no native Polars) are current as of mid-2025. Re-check Polars support.
- **Zyte's QA framework (2018)** and **Frictionless/goodtables material (2017–18)** are structurally sound but the tooling has moved.
- **Dremio Iceberg WAP post is 2023-05-19** — Iceberg branching APIs have evolved; check against current Iceberg docs. Newer worked examples: https://ghostinthedata.info/posts/2026/2026-02-27-wap-iceberg-branching/ · https://www.guptaakashdeep.com/wap-via-apache-iceberg-on-aws/ · https://iceberglakehouse.com/iceberg/iceberg-wap-pattern/
- **Not found in this pass:** a rigorous, non-vendor benchmark of data-quality-tool overhead at scale; published null-rate/coverage thresholds from a government-data harvesting operation specifically; empirical data on how often fixture suites go stale.
- Web search budget was exhausted before I could dig further into freshness-SLA literature and raw-payload/content-hash ingestion architecture; both are covered above from adjacent sources but could be deepened.

---

## Sources

**Scraper-specific silent failure & monitoring**
- Crawlex, *Scraping observability: success metrics, block-rate dashboards, and silent failures* (2026-03-14) — https://blog.crawlex.net/blog/scraping-observability/
- Context.dev, *Scraper Monitoring in Production: A Practical Guide* (2026-08-16) — https://www.context.dev/blog/scraper-monitoring-in-production
- Ficstar, *Why Scraping Fails Silently and Why That's Worse Than Crashing* — https://www.ficstar.com/why-scraping-fails-silently
- DEV, *Stop Silent Scraper Failures: Using Pydantic for Instant Layout Change Detection* — https://dev.to/withatte/stop-silent-scraper-failures-using-pydantic-for-instant-layout-change-detection-4p1k
- Zyte, *Data quality assurance for enterprise web scraping* (2018-09-27) — https://www.zyte.com/blog/data-quality-assurance-for-enterprise-web-scraping/
- Zyte, *4 key steps to develop an Automated Data QA process* (2022-11-01) — https://www.zyte.com/blog/automated-data-qa/
- Spidermon monitors — https://spidermon.readthedocs.io/en/latest/monitors.html
- Spidermon item validation — https://spidermon.readthedocs.io/en/latest/item-validation.html
- PromptCloud, *Data Accuracy in Web Scraping* — https://www.promptcloud.com/blog/data-accuracy-in-web-scraping-challenges-2026/

**Validation libraries**
- Arthur Turrell, *The data validation landscape in 2025* (2025-03-05) — https://aeturrell.com/blog/posts/the-data-validation-landscape-in-2025/
- Posit/Pointblank, *Data Validation Libraries for Polars (2025 Edition)* (2025-06) — https://posit-dev.github.io/pointblank/blog/validation-libs-2025/ (mirror: https://opensource.posit.co/blog/2025-06-04_validation-libs-2025/)
- Pandera vs Great Expectations discussion #598 — https://github.com/unionai-oss/pandera/discussions/598
- endjin, *Data validation in Python: Pandera and Great Expectations* — https://endjin.com/blog/a-look-into-pandera-and-great-expectations-for-data-validation
- Real Python, *Validating Data With Pointblank in Python* — https://realpython.com/python-pointblank/
- GX Core overview (v1.22.0) — https://docs.greatexpectations.io/docs/core/introduction/gx_overview
- ftfy mojibake heuristics — https://ftfy.readthedocs.io/en/latest/heuristic.html · https://alexwlchan.net/notes/2025/ftfy-fix-and-explain/

**Tool comparisons & critiques**
- The Data Letter, *Tool Review: Soda Core vs. Great Expectations* (2025-10-20) — https://www.thedataletter.com/p/tool-review-soda-core-vs-great-expectations
- Dataroots, *State of Data Quality* (Nov 2023) — https://dataroots.io/blog/state-of-data-quality-october-2023
- Bruno Gonzalez, *A guide to open-source data quality tools in late 2023* (2023-09-11) — https://medium.com/@brunouy/a-guide-to-open-source-data-quality-tools-in-late-2023-f9dbadbc7948
- Data Settler, *Challenges with dbt Tests in Practice* (2023-11-26) — https://datasettler.com/blog/post-4-dbt-pitfalls-in-practice/
- Sifflet, *Soda Review: Is Declarative Data Quality Real Observability?* (2025-10-24, competitor) — https://www.siffletdata.com/blog/soda-review
- PipeCode, *Great Expectations vs dbt Tests vs Soda Core* — https://pipecode.ai/blogs/data-quality-frameworks-great-expectations-vs-dbt-tests-vs-soda-core
- Atlan, *Top open source data quality tools* — https://atlan.com/open-source-data-quality-tools/
- Start Data Engineering, *Data quality checks in Python without third-party tools* (2026-03-08) — https://www.startdataengineering.com/post/data-quality-with-python/
- Lewis Hemens, *Testing data quality with SQL assertions* — https://medium.com/dataform/testing-data-quality-with-sql-assertions-2053755395e7
- Datafold, *How to use dbt-expectations* — https://www.datafold.com/blog/dbt-expectations/

**Tool docs & repos**
- Soda Core (v4) — https://github.com/sodadata/soda-core/blob/main/README.md
- SodaCL freshness — https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/freshness.md
- SodaCL anomaly detection (marked deprecated) — https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/anomaly-detection.md
- SodaCL schema — https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/schema.md
- SodaCL reference (referential) — https://docs.soda.io/soda-documentation/soda-v3/sodacl-reference/reference.md
- dbt-expectations (unmaintained) — https://github.com/calogica/dbt-expectations · hub listing https://hub.getdbt.com/metaplane/dbt_expectations/latest/
- Elementary — https://github.com/elementary-data/elementary · https://github.com/elementary-data/dbt-data-reliability · https://docs.elementary-data.com/data-tests/dbt/dbt-package
- Elementary anomaly test parameters (Paradime mirror) — https://docs.paradime.io/app-help/documentation/integrations/observability/elementary-data/anomaly-detection-tests/anomaly-tests-parameters
- Elementary freshness anomalies — https://docs.paradime.io/app-help/documentation/integrations/observability/elementary-data/anomaly-detection-tests/freshness-anomalies
- Deequ — https://github.com/awslabs/deequ · anomaly example https://github.com/awslabs/deequ/blob/master/src/main/scala/com/amazon/deequ/examples/anomaly_detection_example.md · PyDeequ https://pypi.org/project/pydeequ/ · fork https://github.com/nielsen-oss/python-deequ
- Schelter et al., *Automating Large-Scale Data Quality Verification*, PVLDB 2018 — https://www.vldb.org/pvldb/vol11/p1781-schelter.pdf · https://dl.acm.org/doi/10.14778/3229863.3229867
- AWS, *Testing data quality at scale with PyDeequ* — https://aws.amazon.com/blogs/big-data/testing-data-quality-at-scale-with-pydeequ/

**Commercial observability pricing**
- Vendr, Monte Carlo pricing (Feb 2026) — https://www.vendr.com/marketplace/monte-carlo
- Bigeye G2 reviews — https://www.g2.com/products/bigeye/reviews
- Anomalo vs Monte Carlo — https://www.g2.com/compare/anomalo-vs-monte-carlo · https://www.peerspot.com/products/comparisons/anomalo_vs_monte-carlo
- Bigeye vs Monte Carlo — https://datastackindex.com/data-observability/compare/bigeye-vs-monte-carlo/

**Architecture: WAP, quarantine, medallion, circuit breakers**
- Bigeye, *Strategies for handling bad data in data pipelines* — https://www.bigeye.com/blog/strategies-for-handling-bad-data-in-data-pipelines
- Dremio, *Streamlining Data Quality in Apache Iceberg with WAP & branching* (2023-05-19) — https://www.dremio.com/blog/streamlining-data-quality-in-apache-iceberg-with-write-audit-publish-branching/
- Bauplan, no-JVM WAP with Iceberg — https://github.com/BauplanLabs/no-jvm-wap-with-iceberg
- Telm.ai, *What is Write–Audit–Publish?* — https://www.telm.ai/blog/what-is-write-audit-publish-in-apache-iceberg-and-why-it-matters-for-data-quality/
- Iceberg Lakehouse KB, WAP — https://iceberglakehouse.com/iceberg/iceberg-wap-pattern/
- WAP with Iceberg in Snowflake (2026-02-27) — https://ghostinthedata.info/posts/2026/2026-02-27-wap-iceberg-branching/
- Ananth Packkildurai, *An Engineering Guide to Data Quality — A Data Contract Perspective (Part 2)* — https://www.dataengineeringweekly.com/p/an-engineering-guide-to-data-quality
- Ananth Packkildurai, *Revisiting Medallion Architecture* — https://www.dataengineeringweekly.com/p/revisiting-medallion-architecture
- Bright Data, medallion for web data — https://brightdata.com/blog/web-data/medallion-architecture
- Azure Databricks reliability best practices (badRecordsPath, rescued data) — https://learn.microsoft.com/en-us/azure/databricks/lakehouse-architecture/reliability/best-practices
- PySpark dead-letter queues — https://medium.com/@santhoshkumarv/handling-bad-records-in-streaming-pipelines-using-dead-letter-queues-in-pyspark-265e7a55eb29
- Monte Carlo, circuit breakers (vendor) — https://www.montecarlodata.com/blog-announcing-circuit-breakers-a-new-way-to-automatically-stop-broken-data-pipelines-and-avoid-backfilling-costs/
- Airflow circuit breakers guide — https://blog.dataengineerthings.org/data-quality-with-airflow-circuit-breakers-a-step-by-step-guide-2ae30895778e

**Schemas & contracts**
- Confluent, Schema Evolution & Compatibility Types — https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html
- Conduktor, Avro vs Protobuf vs JSON Schema — https://www.conduktor.io/glossary/avro-vs-protobuf-vs-json-schema
- Chad Sanderson, *The Rise of Data Contracts* (2022-08-22) — https://dataproducts.substack.com/p/the-rise-of-data-contracts
- Chad Sanderson, *The Consumer-Defined Data Contract* — https://dataproducts.substack.com/p/the-consumer-defined-data-contract
- Toby Mao, *The False Promise of dbt Contracts* (2023-04-20) — https://www.tobikodata.com/blog/the-false-promise-of-dbt-contracts
- Daniel Beach, *Are Data Contracts Dead?* (2024-12-02) — https://dataengineeringcentral.substack.com/p/are-data-contracts-dead
- Robert Allen, *Most Data Contract Tools Don't Enforce Contracts* (2026-04-29) — https://zircote.com/blog/2026/04/most-data-contract-tools-dont-enforce-contracts/
- Snowplow, *Why data contracts are obviously a good idea* (vendor) — https://snowplow.io/blog/why-data-contracts-are-obviously-a-good-idea
- Bruno Gonzalez, *Implementing data contracts: schema validation* — https://medium.com/@brunouy/implementing-data-contracts-schema-validation-5aefa2b89332

**Parser regression testing**
- scrapy-autounit (archived 2026-06-25) — https://github.com/scrapinghub/scrapy-autounit
- Scrapy-Testmaster — https://github.com/ThomasAitken/Scrapy-Testmaster
- VCR issue #746, stale cassettes / re_record (2019-05-08) — https://github.com/vcr/vcr/issues/746 · discussion #864 https://github.com/vcr/vcr/discussions/864
- pytest-recording — https://github.com/kiwicom/pytest-recording · https://code.kiwi.com/articles/pytest-cassettes-forget-about-mocks-or-live-requests/

**Drift statistics**
- Fiddler, *Measuring Data Drift with PSI* — https://www.fiddler.ai/blog/measuring-data-drift-population-stability-index
- StatsTest, *Drift Detection: KS Test, PSI* — https://www.statstest.com/drift-detection-ks-test-psi-interpret-signals

**Provenance & government data**
- W3C PROV Primer (WG Note, 2013-04-30) — https://www.w3.org/TR/prov-primer/
- Paolo Missier, PROV extensions (2021-02-07) — https://blogs.ncl.ac.uk/paolomissier/2021/02/07/w3c-prov-some-interesting-extensions-to-the-core-standard/
- ProvONE workflow provenance — https://purl.dataone.org/provone-v1-dev
- ProvCaRe, biomedical reproducibility — https://www.sciencedirect.com/science/article/abs/pii/S1386505618302697
- *Rescuing US Government Data* (2025-03-19) — https://atcoordinates.info/2025/03/19/rescuing-us-government-data/
- Slaw, *The Data Rescue Project* (2025-11-27) — https://www.slaw.ca/2025/11/27/the-data-rescue-project-preserving-government-data-is-a-tech-community-issue/
- Ithaka S+R, *Preserving At-Risk Public Data* — https://sr.ithaka.org/blog/preserving-access-for-at-risk-public-data/
- UMN library guide, data & website rescue — https://libguides.umn.edu/c.php?g=1449575&p=10778647
- Macalester, Disappearing Government Data — https://libguides.macalester.edu/disappearing-data
- Frictionless Data validation case study (2017-12-18) — https://blog.okfn.org/2017/12/18/validation-for-open-data-portals-a-frictionless-data-case-study/
- Frictionless, *Validated tabular data* (2018-07-16) — https://frictionlessdata.io/blog/2018/07/16/validated-tabular-data/
- Open Knowledge, *Open data quality — the next shift?* (2017-05-31) — https://blog.okfn.org/2017/05/31/open-data-quality-the-next-shift-in-open-data/
- goodtables-pandas-py — https://github.com/ezwelty/goodtables-pandas-py



---

# Monitoring & Alerting for an Unattended, Long-Running Multi-Source Harvester

**Research date:** 2026-09-06. All pricing verified against vendor pricing pages on this date unless noted.
**Scope:** How a very small team (possibly one person) finds out that one of many government/scientific scrapers has broken, without watching constantly.
**Deliberately not doing:** recommending one stack. Trade-offs and disagreements are surfaced with attribution.

---

## 0. Source-quality warning (read first)

A large fraction of what ranks for "scraper monitoring" in 2026 is **vendor content marketing**, and it is mostly self-confirming. I have flagged each source below as CANONICAL / PRACTITIONER / VENDOR. The strongest, most-citable material on this topic is old (Google SRE 2016–2018, Ewaschuk ~2013, Prometheus docs) and is *not* scraping-specific; the scraping-specific material is mostly recent vendor blogs whose claims are plausible but rarely evidenced. Treat vendor numbers ("3–5 days to detect field-level failures") as illustrative, not measured.

---

## 1. The core insight: silent failure is the dangerous mode

### 1.1 The claim

For scrapers, the failure mode that matters is not the crash. It is the run that exits 0, returns HTTP 200, and writes zero or wrong rows. Process-level monitoring (exit codes, "did the job run?") is structurally blind to it.

### 1.2 Sources that make the point

**CANONICAL — Google SRE Book, "Monitoring Distributed Systems" (2016).** The Four Golden Signals definition of *Errors* explicitly includes the silent case: errors are "the rate of requests that fail, either explicitly (e.g., HTTP 500s), **implicitly (for example, an HTTP 200 success response, but coupled with the wrong content)**, or by policy." This is the earliest authoritative statement of the idea and it predates the scraping-blog genre by a decade.
https://sre.google/sre-book/monitoring-distributed-systems/

**VENDOR (Ficstar, 2026-06-29) — "Why Scraping Fails Silently and Why That's Worse Than Crashing."** Enumerates four silent modes: selector drift after redesign; soft blocks serving truncated/placeholder pages; JavaScript gaps (skeleton HTML captured before hydration); proxy/routing issues returning cached duplicates or disguised rate-limits. Argues the difference is *detection timing*: "A crash stops the pipeline before bad data spreads. A silent failure lets corrupted data move downstream." Recommends: validate critical fields every run, compare record counts to historical baselines, schema/format validators, **canary records with known correct values**, rolling baselines. Reframes the question from "Is the job running?" to "Is the data accurate?"
https://www.ficstar.com/why-scraping-fails-silently

**VENDOR (Context.dev, 2026-08-16) — "Scraper Monitoring in Production."** Three monitoring dimensions: output validation (required fields, types, row counts, null rates); **canary checks** on known URLs with predictable content; **structural diffing** of HTML across snapshots to catch redesigns *before* parsers break. Concrete threshold example: alert when null-price rate jumps from 2% to 70%. Explicitly contrasts process-focused error handling against output monitoring that "checks pages for changes even when scraping completes without an error."
https://www.context.dev/blog/scraper-monitoring-in-production

**VENDOR (Crawlex, 2026-03-14) — "Scraping observability."** The most technically specific of the vendor pieces. Argues standard RED (Rate/Errors/Duration) is insufficient and proposes **yield** — ratio of useful extracted records to attempted requests — as the extra signal. Its single best operational idea: **plot HTTP success rate and data yield on the same axis; "the gap is the silent failure."** Describes a nine-day outage where status codes stayed green while extraction went to zero. Proposes a cheap→expensive block-detection ladder: (1) status code, free; (2) final URL vs requested URL, free; (3) response body-size distribution collapse toward stub sizes, cheap — **this fires before yield metrics decline**; (4) structural validation that required elements exist, cheap; (5) anti-bot vendor marker detection, moderate; (6) canary re-scrape of records with known values, essential but costly.
https://blog.crawlex.net/blog/scraping-observability/

**VENDOR (PromptCloud, 2026-02-18) — "10 Web Scraping Monitoring and Observability Challenges."** Challenge #1 is "Job-Level Monitoring Creates False Safety": "A scrape can finish 'successfully' while the dataset is unusable." Challenge #8, "Freshness Without SLA Context," is the sharpest for this project: **"job runs on time" ≠ "data reflects current state"** — recommends tracking *last meaningful update* rather than last run, with per-source SLAs. Claims field-level completeness failures take 3–5 days to detect unmonitored vs minutes for infrastructure failures (unevidenced).
https://www.promptcloud.com/blog/web-scraping-monitoring-challenges/

**PRACTITIONER — Open States** (the closest real-world analogue: continuously scraping 50+ US state legislature sites for over a decade). Their documented approach is notable for what it *is not*: they do not maintain per-state unit tests. Instead: (a) scraper output is "verified against JSON schemas that protect against common regressions (missing sources, invalid formatted districts, etc.)"; (b) scrapers are **run nightly and the production run is the integration test**; (c) scrapers are deliberately **fragile by design** — they "raise an exception when they see unexpected input" rather than swallowing it. The docs candidly admit the limitation: this catches scrapers that *fail to run*, not scrapers that collect *bad* data.
https://docs.openstates.org/contributing/testing-scrapers/

> **Design lesson from Open States:** deliberately converting silent failures into loud ones (raise on unexpected input) is cheaper than building detectors for silent ones. This is an architectural choice, not a monitoring purchase.

### 1.3 The mapping to data-observability vocabulary

The data-engineering world named this problem first. **Barr Moses, "The 5 Pillars of Data Observability" (2020-12-23)**: Freshness ("Is my data up-to-date? Are there gaps in time when the data has not been updated?"), Distribution (null rates, field-level health), Volume (row counts, sudden drops/increases), Schema (fields added/removed/changed), Lineage.
https://montecarlo.ai/blog-introducing-the-5-pillars-of-data-observability
*Caveat: Monte Carlo is a vendor and this post defines the category it sells into. The taxonomy is nonetheless widely adopted and useful.*

For a harvester, **Freshness + Volume + Distribution are the three that matter**; Lineage is largely irrelevant at this scale.

---

## 2. Dead man's switch / heartbeat monitoring

### 2.1 Why it is the highest-value-per-effort control

A heartbeat inverts the failure logic: instead of the job telling you it failed, **silence is the alarm**. This is the only class of monitoring that catches the "job never ran at all" cases — host down, cron daemon dead, crontab wiped, disk full, container never rescheduled, cloud account suspended — which produce *no* signal of any kind.

**PRACTITIONER — Max Rozen (OnlineOrNot, solo SaaS founder), "Cron job monitoring," last updated 2026-03-03.** Three silent failure modes for cron: complete silence (crash, no notification); lost email (output to root mailbox nobody reads); buried logs (`/var/log/syslog`). Grace-period guidance: **30–60 min for daily jobs, 10 min for hourly.** Canonical invocation:
```bash
0 2 * * * /home/user/backup.sh && curl -fsS --retry 3 https://oonchk.com/abc123
```
The `&&` matters — the ping fires only on exit 0. Triage list: *always* monitor backups, billing, **data pipelines**, cert renewal; *probably* monitor reports and cleanup; *skip* low-impact jobs. Claims ~30 seconds of setup per job.
https://onlineornot.com/cron-job-monitoring-guide

### 2.2 Vendor comparison — pricing verified 2026-09-06

| Service | Free tier | Next tier | Notes |
|---|---|---|---|
| **Healthchecks.io** | **20 checks**, 100 log entries/check, no SMS/call credits, community support | Business **$20/mo** = 100 checks, 1000 log entries/check, 50 SMS/WhatsApp, 20 calls. Business Plus **$80/mo** = 1000 checks. "Supporter" $5/mo is a donation tier with *no* extra features. 20% off annual. | https://healthchecks.io/pricing/ |
| **Cronitor** | Hacker: **5 monitors**, email+Slack, basic status page, 5-min min frequency | Business: **$2/mo per monitor + $5/mo per user**, all 11 integrations, SMS, on-call, 6-mo retention, 30-sec frequency. Enterprise from $6,000/yr. | https://cronitor.io/pricing |
| **Dead Man's Snitch** | "The Lone Snitch" — **1 snitch** | Little Birdy $5/mo (3 snitches); Private Eye **$19/mo (100 snitches)**; Surveillance Van $49/mo (300 snitches, **Smart Alerts, Error Notices**) | https://deadmanssnitch.com/plans |
| **Better Stack** | **10 monitors & heartbeats**, 1 status page; 3 GB logs @ 3-day retention | +10 heartbeats = $17–20/mo; +50 monitors = $21–25/mo; Responder license $29–34/mo | https://betterstack.com/pricing |

**Free-tier reality for a solo operator with many scrapers:** this is the decisive constraint. If the design is one heartbeat per source and there are dozens of sources, **only Healthchecks.io's free tier (20) is remotely sufficient**, and it runs out fast. Cronitor's free 5 and DMS's free 1 are demo tiers. Cronitor's *per-monitor* $2/mo pricing scales badly for a fleet — 60 sources = $120/mo + $5 user = $125/mo, vs Healthchecks Business Plus at $80/mo for 1000 checks, vs **self-hosted Healthchecks at ~$0 plus a VPS**.

**Alternative framing:** don't create one heartbeat per source. Create **one heartbeat for the orchestrator** (did the whole sweep run?) plus **per-source freshness checks in your own database** (§4.3). This collapses the vendor-tier problem entirely and is a strong argument against paying per-monitor.

### 2.3 Healthchecks.io self-hosting

**CANONICAL — github.com/healthchecks/healthchecks.** BSD 3-clause, ~10.1k stars, actively developed. Requires Python 3.12+, Django 6.0, and PostgreSQL/MySQL/MariaDB. Dockerfile and pre-built images provided. Self-hosting requires managing the DB, SMTP, background processes (`sendalerts`, `sendreports`), static files, and a reverse proxy — moderate Django/Linux competence. No documented feature gating vs the hosted service; core functionality including 25+ notification integrations, team management and WebAuthn 2FA is in the OSS build.
https://github.com/healthchecks/healthchecks

**Mechanics worth knowing:** beyond the base ping URL, you can append `/start` (job began — enables runtime tracking), `/fail` (explicit failure), and `/<exitcode>`. stdout/stderr can be POSTed in the body and attached to the ping. **Slug-based URLs support auto-provisioning** — pinging a slug with no existing check creates the check automatically, which is the right primitive for a fleet where sources are added over time. A Management API exists for programmatic check creation.
https://healthchecks.io/docs/ · https://healthchecks.io/docs/monitoring_cron_jobs/

### 2.4 A real, documented limitation of the heartbeat model — DISAGREEMENT

**PRACTITIONER — Hacker News thread, 2022 (item 32038197).** User **ydant**, otherwise praising the service, described exactly the harvester's problem: they monitor periodic downloads that *frequently* time out or fail transiently, but only want an alert if **no successful completion occurs for over a day**. Healthchecks.io alerts immediately on a reported failure, so they abandoned it and **built custom monitoring logic instead**. Maintainer **cuu508** responded by proposing a `/log` endpoint that records events without alerting, letting the client distinguish crashes from routine failures — which ydant accepted while noting it "increases client-side complexity, though this is somewhat inevitable once moving beyond basic use cases." A Cronitor employee (**m1245**) pitched Cronitor as supporting more complex alerting rules.
https://news.ycombinator.com/item?id=32038197

> **This is the single most relevant disagreement in this research.** For a scraper fleet hitting flaky government sites, *transient failure is the normal state*. A naive heartbeat that pages on every failed run will be pure noise. The correct pattern is **ping only on success, with a grace window sized to tolerate N consecutive failures** — i.e. period = run interval, grace = (N × interval). That uses the dead-man's-switch semantics to implement "alert only after N consecutive failures" for free, with no extra logic.

### 2.5 Prometheus-native equivalents

**CANONICAL — Prometheus, "When to use the Pushgateway."** Official position: "We only recommend using the Pushgateway in certain limited cases." Three named downsides: (1) it becomes **a single point of failure and bottleneck** when fronting many instances; (2) you **lose the automatic `up` metric** that Prometheus generates per scrape — i.e. you lose exactly the health signal you wanted; (3) **it never forgets** — series pushed persist forever until manually deleted via its API, so renamed/removed sources leave stale metrics that make `absent()`-style alerts lie. Valid narrow case: *service-level* batch jobs not tied to a machine, and those should omit instance/machine labels. For machine-scoped jobs, use the Node Exporter **textfile collector** instead.
https://prometheus.io/docs/practices/pushing/

> The "never forgets" property is a genuine trap for a harvester: a source you retire keeps reporting its last-known `last_success_timestamp` forever, so a staleness alert on it will fire eternally, and a source you *rename* leaves a ghost that looks healthy.

**CANONICAL — Prometheus, "Alerting" best practices.** Explicitly credits Ewaschuk: "keep alerting simple, alert on symptoms, have good consoles to allow pinpointing causes, and avoid having pages where there is nothing to do." **The batch-job guidance is directly applicable:** page if a job "hasn't succeeded recently enough to avoid user-visible problems," and set the threshold to permit **at least two full job cycles** — their worked example: a job that runs every 4 hours and takes 1 hour should alert at roughly **10 hours** of no success.
https://prometheus.io/docs/practices/alerting/

**The Watchdog / DeadMansSnitch pattern (who watches the watcher).** kube-prometheus ships a `Watchdog` alert that is **designed to always fire**: "an alert meant to ensure that the entire alerting pipeline is functional… If not firing then it should alert external systems that this alerting system is no longer working." It is routed to an external heartbeat service; silence there means your *monitoring* died.
https://runbooks.prometheus-operator.dev/runbooks/general/watchdog/

**PRACTITIONER — Paul's Programming Notes, 2026-07-05, "A dead man's switch for a single-host monitoring stack."** A solo operator running Prometheus + Alertmanager + Grafana on one mini-PC. The failure that motivated it: **"Prometheus once sat dead for two days before I noticed."** Implementation: an alert rule on `vector(1)` (always true), routed *exclusively* to an external webhook (not to the normal Telegram channel), `repeat_interval: 5m`, `send_resolved: false` to keep heartbeat data clean, pointed at Healthchecks.io with a ~10-min period and ~5-min grace — total detection latency ~15 min.
https://www.paulsprogrammingnotes.com/2026/07/dead-mans-switch-single-host-monitoring.html

> **Direct relevance:** any self-hosted monitoring for an unattended harvester needs this, or the whole scheme is one host failure away from silently doing nothing. The external endpoint must be on infrastructure that does not share a failure domain with the harvester.

### 2.6 Self-hosted alternatives

**Uptime Kuma vs Healthchecks (self-hosted), for solo builders — futurion.blog, 2026-04-07 (updated 2026-08-25).** Kuma actively probes HTTP(S)/TCP/DNS and renders history charts; good at *visible* failures (unreachable, broken TLS). Healthchecks is for *silent* failures via heartbeats. Named blind spots: **Kuma will show green for a service returning 200 with corrupted data** unless you add semantic checks; **Healthchecks misses degradation** — a job can ping success after running six hours and doing damage. Kuma has "more moving parts" (richer UI); Healthchecks has a simpler schema. On the monitor paradox: "the thing that tells you other things are down must not go down with them." Recommends running both.
https://futurion.blog/self-hosting-uptime-kuma-vs-healthchecks-io-honest-trade-offs-for-solo-builders/
*Flag: low-authority blog, but the trade-off analysis is sound and matches the primary docs.*

**`deadmancheck`** (github.com/Kriss-V/deadmancheck) is interesting conceptually — explicitly markets "alerts when jobs run but do nothing," combining dead man's switch + **output assertions on the ping payload** (e.g. POST `{"rows_exported": 1523, "status": "ok"}` and assert `rows_exported > 0`) + duration anomaly detection via rolling average. **However: 0 stars, 0 forks, no releases. Do not depend on it.** Cited only because the payload-assertion pattern is the right idea and is trivially reimplementable.
https://github.com/Kriss-V/deadmancheck

---

## 3. Metrics: the four golden signals, translated

### 3.1 The originals

**CANONICAL — Google SRE Book, ch. 6 (2016):** Latency (distinguish successful vs failed request latency — "slow errors are worse than fast errors"); Traffic (demand: req/s); Errors (explicit / **implicit — 200 with wrong content** / by policy); Saturation (utilization; "many systems degrade in performance before they achieve 100% utilization").
https://sre.google/sre-book/monitoring-distributed-systems/

Same chapter, two under-quoted principles that matter more here than the signals themselves:
- **Keep it simple:** "Rules that catch real incidents most often should be as simple, predictable, and reliable as possible." Google removes rarely-used signals and avoids fragile dependency hierarchies.
- **The Bigtable case study:** excessive alerting, *even when technically accurate*, consumed engineering time without improving reliability; the fix was to temporarily *relax* SLO targets to free capacity to fix root causes.

### 3.2 Translation to a harvester

| Golden signal | Harvester equivalent |
|---|---|
| Traffic | requests/run; pages fetched per source |
| Errors (explicit) | HTTP 5xx, transport/DNS/TLS/timeout errors, **429 rate, 403 rate** (separate them — 429 is politeness, 403 is a block) |
| Errors (implicit) | **the important one:** items/run vs baseline, items per source, parse-success ratio, field-coverage %, null rate per field, canary mismatch, body-size distribution collapse, final-URL ≠ requested-URL |
| Latency | fetch latency distribution (bucketed, **not averaged** — per Crawlex), run duration vs rolling baseline |
| Saturation | proxy pool utilisation, **proxy cost/run**, bytes fetched, disk, rate-limit headroom |
| *(new)* Freshness | **staleness/lag per source: now − last_meaningful_update** |
| *(new)* Yield | useful records ÷ attempted requests (Crawlex's "fifth signal") |

### 3.3 A ready-made, battle-tested checklist: Spidermon

**CANONICAL-ish (Zyte/Scrapinghub OSS, in production across a commercial scraping business).** Spidermon's built-in monitors are effectively "the golden signals for scrapers," already enumerated with setting names. This is the most concrete artefact found in this research and is worth copying even if not using Scrapy:

| Monitor | Setting | Checks |
|---|---|---|
| ItemCountMonitor | `SPIDERMON_MIN_ITEMS` | minimum items extracted |
| ItemValidationMonitor | `SPIDERMON_MAX_ITEM_VALIDATION_ERRORS` | validation errors below limit |
| **FieldCoverageMonitor** | `SPIDERMON_FIELD_COVERAGE_RULES` | **per-field completeness %** |
| ErrorCountMonitor / WarningCountMonitor / CriticalCountMonitor | `SPIDERMON_MAX_ERRORS` / `_WARNINGS` / `_CRITICALS` | log-level counts |
| FinishReasonMonitor | `SPIDERMON_EXPECTED_FINISH_REASONS` | job ended for the right reason (default `finished`) |
| RetryCountMonitor | `SPIDERMON_MAX_RETRIES` | requests dropped after retry exhaustion |
| DownloaderExceptionMonitor | `SPIDERMON_MAX_DOWNLOADER_EXCEPTIONS` | timeouts / connection rejections |
| PeriodicExecutionTimeMonitor | `SPIDERMON_MAX_EXECUTION_TIME` | runtime cap |
| PeriodicItemCountMonitor | `SPIDERMON_ITEM_COUNT_INCREASE` | **items still increasing mid-run** (catches stalls) |
| UnwantedHTTPCodesMonitor | `SPIDERMON_UNWANTED_HTTP_CODES` | 403/429/etc. thresholds |
| SuccessfulRequestsMonitor / TotalRequestsMonitor | `SPIDERMON_MIN_SUCCESSFUL_REQUESTS` / `_MAX_REQUESTS_ALLOWED` | request-level floors/ceilings |
| **ZyteJobsComparisonMonitor** | `SPIDERMON_JOBS_COMPARISON` | **item count vs previous N jobs** — relative, not absolute |

https://spidermon.readthedocs.io/en/latest/monitors.html

**Documented weakness (Ian Kerins, 2022-01-12):** "Spidermon doesn't have any dashboard or user interface where you can see the output of your monitors." You review log files or build your own visualisation.
https://dev.to/iankerins/how-to-monitor-your-scrapy-spiders-5c9o
Worked config examples (thresholds, Schematics validation models, Slack delivery via `SendSlackMessageSpiderFinished`): https://scrapeops.io/python-scrapy-playbook/extensions/scrapy-spidermon-guide/

### 3.4 Where to put the metrics — pricing verified 2026-09-06

**Grafana Cloud** — https://grafana.com/pricing/
- Free: 10k active metric series, 50 GB logs, 50 GB traces, **14-day retention**, **3 active users**.
- Pro: **$19/mo platform fee** + usage. Metrics from **$6.50 per 1k series** (13-month retention). Logs/traces: $0.050/GB process + $0.400/GB write + $0.100/GB retain. Grafana users $8.00 each. IRM users $20.00 each.
- Enterprise: from **$25,000/yr commit**.

> For a harvester the free tier is genuinely usable: 10k series is a lot if you keep cardinality low (per-source labels are fine; per-URL labels are not). 14-day retention is the real constraint — it kills year-over-year seasonality comparison, which matters for government sources with legislative/fiscal-calendar rhythms.

**Datadog** — https://www.datadoghq.com/pricing/
- Infrastructure Pro **$15/host/mo** annual ($18 on-demand); Enterprise $23 ($27).
- Logs: **$0.10/GB ingest**; standard indexing **$1.70 per million events** annual ($2.55 on-demand); Flex storage $0.05/million.
- APM $31/host/mo annual with infra; $36 standalone.
- **Custom metrics: 100 included per host on Pro, 200 on Enterprise.**

**The custom-metrics trap, with arithmetic (SigNoz, 2026-07-17 — VENDOR, competitor, but the maths is checkable against Datadog's own docs):** a custom metric is "uniquely identified by the combination of a metric name **and its tag values, including the host tag**." Worked example: `request.latency` tagged by region (2) × status (2) = 4 metrics; add `user_id` with 1,250 values → **5,000 billable timeseries**. On Pro with 100 included, that's 4,900 overage units at $5 per 100 = **$245/mo, ~$2,940/yr, from a single over-tagged metric.**
https://signoz.io/blog/datadog-custom-metrics-pricing/

> **Directly applicable risk:** the natural scraper instrumentation is `items_scraped{source="...", status="...", error_class="..."}`. With 60 sources × 5 statuses × 10 error classes that is 3,000 series **per metric name**. On Datadog that is a four-figure annual line item; on self-hosted Prometheus it is free. This is the strongest single cost argument in the whole comparison, and it is a *design* issue (label cardinality), not just a vendor issue.

**Sentry** — https://sentry.io/pricing/
- Developer (free): **5k errors/mo**, 1 user, 30-day retention.
- Team **$26/mo**: 50k errors, unlimited users, 90-day lookback.
- Business **$80/mo**: same 50k errors, advanced quota management, SAML+SCIM.
- Pay-as-you-go overage from ~$0.0003625/error.

> **Caution for a scraper fleet:** 5k errors/month is ~167/day. A single broken source in a retry loop will exhaust the free tier in hours and then blind you to everything else. Sentry's value here is *exception grouping and stack traces for parser bugs*, not volume monitoring. Aggressive `before_send` sampling/filtering of expected network errors is mandatory.

**Better Stack** — https://betterstack.com/pricing — free: 10 monitors+heartbeats, 3 GB logs @ 3 days, 100k exceptions/mo, 5k session replays. Logs $0.10–0.15/GB standard ingest, $0.05–0.08/GB/mo retention; bundles from $25/mo (40 GB) to $420/mo (700 GB). Responder license $29–34/mo.

**Self-hosted (Prometheus + Grafana + Loki, or OpenObserve).** Consensus across the comparison literature is that self-hosting trades **direct cost near zero (one small VPS)** for **operator time and a new failure domain you must yourself monitor** (§2.5). The genuinely honest framing found: a solo operator's scarcest resource is attention, and a self-hosted TSDB+log stack is a second system to keep alive for months unattended. Sources are mostly vendor-adjacent comparison posts; treat directionally:
https://betterstack.com/community/comparisons/open-source-log-managament/ · https://www.groundcover.com/guides/saas-vs-self-hosted-observability-costs

---

## 4. Alerting philosophy

### 4.1 Ewaschuk — the foundational document

**CANONICAL — Rob Ewaschuk, "My Philosophy on Alerting"** (written at Google, ~2013; the Google Doc is the original but requires JS — a plain-text mirror is the practical citation). Key lines:

- "Every time my pager goes off, I should be able to react with a sense of urgency."
- "**Every page should be actionable**; simply noting 'this paged again' is not an action."
- "Every page should require intelligence to deal with: **no robotic, scriptable responses.**"
- On symptoms vs causes: users don't care that the MySQL servers are down; they care that queries fail. "The former is a proximate cause, the latter is a symptom." Where both exist, redundant alerts require separate tuning for no added value — catch symptoms first, use causes as supporting diagnostic detail.
- On cause-based rules: "**Use these sparingly**; don't write cause-based paging rules for symptoms you can catch otherwise."
- On sub-critical alerts: route to ticket systems, daily reports, or email — but **maintain triage discipline**, or they accumulate indefinitely.
- Summary: pages should be "urgent, important, actionable, and real," representing "either ongoing or imminent problems." Alert from the client/user perspective where possible. **Remove noisy alerts aggressively — "under-monitoring proves easier to fix than over-monitoring."**

Mirror: https://gist.github.com/msgodf/86a3fc7fcd3ce663ff37
Original (JS-gated): https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcApr8Q/preview
Also mirrored: https://linuxczar.net/sysadmin/philosophy-on-alerting/

> **Applied to a solo harvester operator, "every page should be actionable" is nearly decisive.** If nobody is on call, a "page" is just an interruption. The Ewaschuk framework mostly argues for pushing *everything* into the ticket/daily-report tier (§5).

### 4.2 SLO / error-budget alerting — and the counter-argument

**CANONICAL — Google SRE Workbook, "Alerting on SLOs."** Six escalating approaches: (1) target error rate over a short window — fast, imprecise; (2) longer window — precise, terrible reset time; (3) add a duration parameter — still fails for severe incidents (total outage alerts no faster than mild degradation); (4) burn-rate alerts; (5) multiple burn rates with different routing (page vs ticket); (6) **multiwindow, multi-burn-rate — recommended**: alert only when a long *and* a short window both exceed threshold, confirming burning is still active, which fixes reset time.

Recommended starting parameters for a 99.9% SLO:

| Severity | Long window | Short window | Burn rate | Budget consumed |
|---|---|---|---|---|
| Page | 1 hour | 5 min | 14.4 | 2% |
| Page | 6 hours | 30 min | 6 | 5% |
| Ticket | 3 days | 6 hours | 1 | 10% |

**Google's own stated caveat:** "Lots of parameters to specify, which can make alerting rules hard to manage."
https://sre.google/workbook/alerting-on-slos/

**DISAGREEMENT — Marcus Hill (Chronosphere), "Stop thinking in burn rates," 2025-07-01.** Direct attack on the recommended approach: "**Burn rates are a horrible thing to obsess over when configuring an SLO.**" Three arguments: (a) they obscure the thing you actually care about, which is user experience; (b) "Creating and managing SLOs is hard enough for all of these nontechnical reasons. Adding in burn rates and doing math in a language like PromQL, with its own gotchas, just makes it harder"; (c) conceptually, burn rates ignore how the service performed over the *entire* SLO window and fixate on the recent past. Proposes replacing the burn-rate abstraction with two plain questions — "What percentage of the budget should trigger an alert?" and "Over what time window?" — and notes teams should prefer "error rates and counts" because "they're more grokkable and your less SLO-familiar teammates will thank you."
https://chronosphere.io/learn/stop-thinking-in-burn-rates/
*Flag: Chronosphere is an observability vendor; this is partly positioning. The complexity critique is nonetheless substantive and echoes Google's own caveat.*

**Middle position — Roman Khavronenko & Mathias Palmersheim (VictoriaMetrics), "Alerting Best Practices," 2025-08-22.** Pragmatic and non-dogmatic. Route by a `severity` label so warnings and criticals go to different destinations. Emphasises the `for` parameter as "one of the most effective tools for reducing noisy alerts." Concrete sizing rule: **set lookbehind windows to at least 4× the scrape interval, and make `for` exceed the lookbehind window.** Mentions SLO/error-budget alerting only as an *option* offering more nuance than raw error counts — notably declining to make it the default.
https://victoriametrics.com/blog/alerting-best-practices/

> **Synthesis:** the Crawlex scraping-observability post (§1.2) recommends multiwindow multi-burn-rate alerting on *yield*. That is technically coherent but is the maximalist position: it imports Google's full apparatus into a one-person project. Chronosphere's and VictoriaMetrics' positions — plus Ewaschuk's own "keep alerting simple" — argue the opposite. **No consensus exists; the disagreement is real and is fundamentally about team size, not correctness.**

### 4.3 Freshness thresholds: the dbt pattern

The cleanest existing formalism for **per-source staleness thresholds with two severity tiers** is dbt's `source freshness`, and it is worth stealing wholesale even outside dbt:

```yaml
sources:
  - name: jaffle_shop
    config:
      freshness:
        warn_after:  { count: 12, period: hour }
        error_after: { count: 24, period: hour }
      loaded_at_field: _etl_loaded_at
    tables:
      - name: orders
        config:
          freshness:
            warn_after:  { count: 6,  period: hour }
            error_after: { count: 12, period: hour }
            filter: datediff('day', _etl_loaded_at, current_timestamp) < 2
      - name: product_skus
        config:
          freshness: null   # explicitly exempt
```
Key properties: source-level defaults with **per-table override**; **two tiers (warn vs error) mapping cleanly to digest vs alert**; `freshness: null` to explicitly exempt a source; `loaded_at_field` can be an expression (timezone conversion, casts); `loaded_at_query` (dbt ≥1.10) for custom max-timestamp SQL.
https://docs.getdbt.com/reference/resource-properties/freshness

> **This is the highest-leverage pattern in this document for a multi-source harvester.** Government sources have wildly heterogeneous natural cadences — a daily FDA feed, a quarterly BLS release, an irregular legislature calendar. A single global staleness threshold is useless; per-source `warn_after`/`error_after` in a config file, evaluated by one query against a `last_meaningful_update` column, is a few dozen lines of code and replaces most of a monitoring product.

### 4.4 Alert fatigue — evidence

**CANONICAL — Google SRE Book, "Being On-Call."** Hard numbers: "**the maximum number of incidents per day is 2 per 12-hour on-call shift**," justified because "dealing with the tasks involved in an on-call incident — root-cause analysis, remediation, and follow-up activities like writing a postmortem and fixing bugs — takes 6 hours" on average. Also: "we strive to invest at least 50% of SRE time into engineering: of the remainder, no more than 25% can be spent on-call." On cognitive impairment: "Stress hormones like cortisol and corticotropin-releasing hormone (CRH) are known to cause behavioral consequences — including fear — that can impair cognitive functions and cause suboptimal decision making."
https://sre.google/sre-book/being-on-call/

**Research aggregation — pingfatigue.com/research.** Collects citations across domains. The DevOps/SRE evidence base is thin; the strongest quantitative evidence is borrowed from healthcare alarm fatigue:
- Joint Commission Sentinel Event Alert 50 (2013): **85–99% false-positive ICU alarm rates.**
- AHRQ PSNet (2019): 72–99% false-positive across ICU unit types; alarm fatigue linked to patient harm.
- Cvach MM (2012) literature review: 86–99% false-alarm range.
- Gloria Mark (UC Irvine, 2004; follow-up 2023): ~23-minute refocus time after interruption.
- Microsoft Work Trend Index (2023): 57% report constant notification interruption.
https://pingfatigue.com/research
*Flag: this is a curated list on a small site; the underlying citations are real and well-known, but I did not verify each primary source. The healthcare→DevOps transfer is an analogy, not a finding.*

---

## 5. What people with small systems actually run

### 5.1 Digest over pages

The pattern that recurs — and which follows directly from Ewaschuk's "sub-critical alerts → tickets/daily reports" tier — is a **daily digest** rather than per-failure notification. The searchable literature on this is weak (mostly Zapier/Confluence how-tos, not engineering practice), so this is a **gap in the evidence**: it is widely practised and rarely written up rigorously. Better-documented instances:
- **metaodi** (HN, 2020) collecting COVID-19 data across Swiss cantons: **error notifications to Slack**, not paging. https://news.ycombinator.com/item?id=24732943
- Healthchecks.io ships `sendreports` as a first-class background process — periodic report emails are a built-in concept, not a bolt-on. https://github.com/healthchecks/healthchecks
- Homelab-scale worked example: https://peira.dev/blog/homelab-daily-digest-email/

### 5.2 "Alert only if N consecutive failures"

Three mechanically different ways to get this, all documented:

1. **Grace-window arithmetic on a heartbeat** (§2.4) — ping only on success; set grace = N × interval. Zero code.
2. **Prometheus/Grafana `for` clause.** Grafana's own guide warns this is subtler than it looks: `up == 0 for: 5m` means five consecutive failed scrapes, but "**a single recovery resets the alert, that's why `up == 0 for: 5m` can sometimes be unreliable**." Their recommended fix is **smoothing instead of consecutiveness**: `avg_over_time(up[10m]) < 0.8` fires when availability drops below 80% over 10 minutes and is *not* reset by intermittent recoveries — then lower the pending period to 0–1 min. https://grafana.com/docs/grafana/latest/alerting/guides/connectivity-errors/
3. **Prometheus batch-job rule** — alert on time-since-last-success exceeding ~2 job cycles (§2.5). https://prometheus.io/docs/practices/alerting/

> For flaky government sites, **smoothing (avg_over_time) is materially better than the `for` clause**, because scrapers routinely half-succeed. This is a concrete, citable correction to the obvious approach.

### 5.3 Output validation tooling

- **Spidermon** (§3.3) if Scrapy.
- **Great Expectations / Soda Core / dbt tests** for warehouse-side row-count, null-rate and schema assertions. Comparison: https://www.pistack.xyz/posts/self-hosted-data-quality-tools-great-expectations-soda-dbt-guide-2026/ and https://www.thedataletter.com/p/tool-review-soda-core-vs-great-expectations (both partly promotional; the feature comparisons are checkable against each project's docs).
- **dbt `source freshness`** (§4.3) for staleness specifically.
- **JSON Schema validation on scraper output**, per Open States — the cheapest option and proven over a decade on exactly this problem domain. https://docs.openstates.org/contributing/testing-scrapers/

### 5.4 Exceptions and logs

Sentry for parser exceptions (§3.4 — with the 5k/mo free-tier caveat and mandatory filtering). Log aggregation only if you will actually query it; a solo operator's realistic alternative is structured JSON logs written to disk/object storage with `jq`, which costs nothing and has no failure domain. The evidence base for "small teams need centralised logs" is entirely vendor-authored and should be discounted.

---

## 6. Anomaly detection on scrape volume

### 6.1 What practitioners actually use

**Ryan Kearns (Monte Carlo), "How To Build Your Own Data Anomaly Detectors Using SQL," published 2025-04-18, updated 2025-07-01.** Notable because a vendor is telling you to write SQL instead of buying. Two categories: **freshness** (examine timestamp columns, detect gaps in arrival) and **volume** (row counts against baseline — "when daily load of 100,000 customer records suddenly drops to zero, the anomaly detection query immediately alerts"), plus **distribution** (null rates, zero-value frequencies) as the subtle-degradation canary. Thresholds are baseline-relative, not fixed: values "outside 1.5× the interquartile range or multiple standard deviations." Their stated emphasis is **context-driven investigation over pure statistical outlier flagging** — "good data observability hinges on proper leverage of metadata" to determine *why*, not just *that*.
https://montecarlo.ai/blog-how-to-build-your-own-data-anomaly-detectors-using-sql

**Crawlex (2026-03-14)** recommends **EWMA or CUSUM** for seasonality-aware detection "rather than static thresholds" — the most sophisticated recommendation found, and correspondingly the least evidenced.
https://blog.crawlex.net/blog/scraping-observability/

**Spidermon's `ZyteJobsComparisonMonitor`** is the shipping, production-proven version of this idea and it is deliberately crude: compare this job's item count against the previous N jobs. From a company whose business is running scrapers.
https://spidermon.readthedocs.io/en/latest/monitors.html

**PromptCloud (2026-02-18)** frames the failure mode of naive anomaly detection precisely — challenge #6 "Missing Performance Baselines" (recommends **rolling 7/30-day bands, segmented per source, seasonality-adjusted**) and challenge #9 "Context-Free Anomaly Detection": statistical anomalies fire during *normal* business events. Recommends business-aware thresholds and seasonality calendars.
https://www.promptcloud.com/blog/web-scraping-monitoring-challenges/

### 6.2 Assessment and the honest gap

**I found no rigorous practitioner evidence that ML-based anomaly detection outperforms simple percentage-of-median or z-score rules at this scale.** What the sources consistently converge on:
- **Relative beats absolute.** Compare to the last N runs for *this source*, never a global constant (Spidermon, Monte Carlo, PromptCloud all agree).
- **Zero is special.** A drop to zero is unambiguous and deserves its own rule; it needs no statistics.
- **Government sources have real seasonality** (legislative sessions, fiscal calendars, quarterly releases) which will generate false positives in any naive z-score. This is the single strongest argument *against* over-engineering: a source that legitimately publishes nothing for three months breaks both z-score and static-threshold approaches, and the only robust fix is per-source configuration (§4.3), not a better algorithm.

---

## 7. Structured logging and OpenTelemetry

### 7.1 The complexity critique — DISAGREEMENT, well-attributed

**David Cramer (founder of Sentry), "The Problem with OpenTelemetry."** Scope creep: "It tries to build specifications and SDKs for not only span annotations, but universal transports for logs and metrics, and who knows what else." Learning curve: "There are *so many concepts* you have to master, and even as an experienced developer you're going to rightly question why some are relevant." Governance: "OpenTelemetry doesn't have an end state that we'd all agree upon" — design by committee without strategic alignment. Practical: "It doesn't work for customers" due to "version conflicts, specification incompatibilities, and generally speaking, code that tries to do too much." **Concession:** OTel has "actually done a really good job in some ecosystems of providing baked-in instrumentation for libraries," particularly Node.js.
https://cra.mr/the-problem-with-otel/
*Note: Cramer is the founder of a competing vendor. Disclose that when citing. No publication date visible on the page.*

**Moshe Shaham, "Distributed Tracing Is Overrated," 2026-01-14.** Argues tracing is "widely overused, misunderstood, and delivers a terrible noise-to-value ratio." Where it *does* work: low-volume request-driven systems, high fan-out architectures, latency-sensitive user requests, clear causal chains across services. Where it does not: **batch processing, streaming/queue-based systems, background workers, and ETL pipelines** — i.e. precisely the shape of a harvester. **Weakness of the piece:** he critiques effectively but offers little constructive alternative beyond gesturing at "metrics, logs, and profiling at appropriate abstraction levels."
https://mosheshaham.substack.com/p/distributed-tracing-is-overrated

**Counterweight:** the pro-OTel case is that vendor-neutral instrumentation is an *exit option* — instrument once, switch backends without re-instrumenting, which matters if Datadog bill shock (§3.4) later forces a migration. That is a real strategic argument and is why "no OTel" is not obviously correct even for a solo operator.

### 7.2 Assessment

The evidence points one direction for this workload: **a harvester is a batch/ETL system with a shallow call graph** (fetch → parse → write). Traces answer "which of my 40 microservices added the latency," a question this architecture does not raise. **Structured logging with a run_id and source_id per record delivers most of the debugging value at a fraction of the complexity**, and can be grep'd or loaded into SQLite/DuckDB. This is an assessment, not a citation — the literature on structured logging is dominated by vendor guides (e.g. https://www.dash0.com/guides/structured-logging-for-modern-applications) with little independent practitioner writing.

---

## 8. Runbooks, auto-remediation, and 3am

### 8.1 The reality for this system

For a months-long unattended harvester run by one person, **almost nothing justifies a 3am page**, and this follows from Ewaschuk's own criteria rather than from laziness:
- "Every page should be actionable." At 3am, the action for a broken parser is *go back to sleep and fix it tomorrow* — so it is not a page.
- "Every page should require intelligence... no robotic, scriptable responses." If the fix is "retry," automate the retry; do not page.
- Google's ceiling is **2 incidents per 12-hour shift** with a **50% ops cap** — for a solo operator with a day job, the sustainable rate is far lower.
- Government data sources are not real-time. A source going stale for 12 hours is essentially never worse than it going stale for 4.
https://gist.github.com/msgodf/86a3fc7fcd3ce663ff37 · https://sre.google/sre-book/being-on-call/

### 8.2 A defensible tiering

| Tier | Trigger | Channel |
|---|---|---|
| **Page (rare)** | Orchestrator heartbeat silent (nothing has run at all); storage full / write failures; credential or billing expiry that will cause irrecoverable data loss | SMS/phone |
| **Same-day** | A source past `error_after` staleness; yield collapse across *many* sources at once (suggests shared cause: proxy, network, egress IP blocked) | Slack/push |
| **Daily digest** | Individual source failures; sources past `warn_after`; field-coverage regressions; 429/403 rate changes; cost anomalies | Email |
| **Weekly review** | Trend dashboards, slow degradation, sources quietly declining in volume | Dashboard |

**The irrecoverable-loss criterion is the one genuinely harvester-specific rule** and is not in the general SRE literature: most scraper failures are recoverable by backfilling later, but some government sources publish transient data that is *gone* if missed (a real-time feed, a document replaced in place, a page with no archive). Only those justify urgency. Simon Willison's git-scraping approach is in effect a mitigation for this class — commit every observed state so nothing is lost even if you notice late. https://simonwillison.net/2020/Oct/9/git-scraping/

### 8.3 Runbook guidance

Runbook literature found is uniformly 2026 vendor content of modest quality — e.g. https://openobserve.ai/blog/on-call-runbook-template-sre/ and https://rootly.com/incident-response/runbooks. The durable point is Ewaschuk's: if a runbook step is mechanical, **automate it and stop paging**; a runbook that reduces to a script is a bug in the alert.

---

## 9. Real accounts of long-running unattended scraping

**Simon Willison, "Git scraping" (2020-10-09)** — commit scraped data to a git repo on a schedule via GitHub Actions; the commit history becomes an auditable changelog. Heavily used for government/civic data. **Monitoring gap, and it matters:** the article contains no discussion of failure detection, and the canonical workflow line `git commit -m ... || exit 0` **succeeds silently when there are no changes — which is indistinguishable from the scraper having broken.** This is a textbook silent failure baked into the most popular pattern for exactly this use case.
https://simonwillison.net/2020/Oct/9/git-scraping/ · series: https://simonwillison.net/series/git-scraping/

**Hacker News discussion of git scraping (2020, item 24732943)** — the richest source of unvarnished long-run experience found:
- **swyx**: "GH actions fail pretty regularly" — shared action logs showing repeated failures across a year, and recommended building retry capability to avoid data loss. *Directly relevant: the free scheduler is itself a source of silent gaps.*
- **simonw**: curl errors typically kill the script before the commit, so corrupted data doesn't land in the repo — a fail-loud-by-accident property.
- **arkadiyt**: tracking bug bounty programs hourly **for three years**.
- **13rac1**: air-quality time-lapse via Puppeteer screenshots, **accumulated 200 GB**.
- **johnnyapol**: daily 7:30am course-schedule scraping via Actions.
- **metaodi**: COVID-19 data across Swiss cantons with **error notifications to Slack**.
- **LukeEF** (TerminusDB): git has a pull:push bottleneck limiting writes to ~one commit per 30s — a hard scaling ceiling.
- **drhayes9** raised GitHub ToS concerns (Actions are for "production, testing, deployment, or publication of the software project"); **simonw** replied that GitHub employees had raised no objection in past discussions. *Unresolved — a real operational risk for a months-long unattended system on someone else's free compute.*
https://news.ycombinator.com/item?id=24732943

**Open States** (§1.2) — decade-plus, 50+ jurisdictions, nightly runs as integration tests, JSON-schema validation, deliberately fragile scrapers. https://docs.openstates.org/contributing/testing-scrapers/

**Paul's Programming Notes (2026-07-05)** — "Prometheus once sat dead for two days before I noticed." https://www.paulsprogrammingnotes.com/2026/07/dead-mans-switch-single-host-monitoring.html

**HN ydant (2022)** — abandoned Healthchecks.io's failure alerting for custom logic because transient failures are normal and only a full day without success should alert. https://news.ycombinator.com/item?id=32038197

> **Gap in the evidence:** I could not find a rigorous, long-form "lessons from running N scrapers for Y years" post with quantified monitoring outcomes. The genre exists mostly as vendor content. The HN thread above is the closest thing to primary evidence and it is six years old.

---

## 10. Open disagreements, restated

1. **Full SLO/burn-rate alerting vs simple thresholds.** Crawlex recommends multiwindow multi-burn-rate on yield; Google SRE Workbook provides the parameters *and* warns "lots of parameters to specify, which can make alerting rules hard to manage"; Marcus Hill (Chronosphere) says burn rates are "a horrible thing to obsess over"; VictoriaMetrics treats SLOs as optional; Ewaschuk says keep it simple. **Unresolved. Divides on team size, not correctness.**
2. **Heartbeat services vs roll-your-own.** ydant (HN) abandoned Healthchecks.io because it couldn't express "only alert if no success for a day"; maintainer cuu508 proposed `/log`; a Cronitor employee pitched richer rules. **The grace-window trick (§2.4) resolves this for free and neither party mentioned it** — worth noting as a finding rather than a citation.
3. **OpenTelemetry.** Cramer (Sentry founder, conflicted) and Shaham say it's too complex and wrong-shaped for batch/ETL; the vendor-neutrality/exit-option argument cuts the other way. **Genuine disagreement.**
4. **Hosted vs self-hosted.** Hosted has a real free tier (Grafana Cloud 10k series, Healthchecks 20 checks) but 14-day retention and cardinality cliffs; self-hosted is near-zero marginal cost but adds a failure domain that itself needs a dead man's switch. **No consensus; sources are mostly conflicted vendors.**
5. **Anomaly detection sophistication.** EWMA/CUSUM (Crawlex) vs compare-to-last-N-jobs (Spidermon, shipping in production) vs IQR/stddev SQL (Monte Carlo). **No evidence found that sophistication wins at this scale.**
6. **Per-source heartbeats vs one orchestrator heartbeat + DB freshness checks.** Vendor pricing pushes toward the former; cost and simplicity push toward the latter. Not directly debated in any source — **this is my synthesis, flagged as such.**

## 11. Stale-material flags

- **Ewaschuk (~2013)** and **Google SRE Book (2016)** are old but foundational and still the most-cited; not stale in substance.
- **Ian Kerins on Spidermon (2022-01-12)** — verify the dashboard limitation still holds against current Spidermon releases.
- **HN git-scraping thread (2020)** — six years old; GitHub Actions reliability, quotas and ToS have all changed since. swyx's reliability complaint may or may not still apply.
- **Simon Willison's git scraping (2020)** — the pattern is current (he maintains an ongoing series) but the original post predates most Actions changes.
- **Monte Carlo 5 Pillars (2020-12-23)** — taxonomy still standard.
- **Chronosphere (2025-07-01)**, **VictoriaMetrics (2025-08-22)** — recent enough.
- **All pricing** verified 2026-09-06; observability pricing changes frequently — **re-verify before committing.**
- **David Cramer's OTel post** — no visible date; OTel has moved considerably, so treat specific technical complaints as possibly dated while the structural critique likely stands.
- The **2026 vendor blogs** (Ficstar, Context.dev, Crawlex, PromptCloud, OneUptime, SigNoz) are current but promotional; their *taxonomies* are useful, their *statistics* are unevidenced.

---

## Sources

### Canonical / foundational
- Google SRE Book, Monitoring Distributed Systems (Four Golden Signals; implicit errors; keep it simple) — https://sre.google/sre-book/monitoring-distributed-systems/
- Google SRE Book, Being On-Call (2 incidents/12h shift; 50% ops cap) — https://sre.google/sre-book/being-on-call/
- Google SRE Workbook, Alerting on SLOs (six methods; multiwindow multi-burn-rate table; complexity caveat) — https://sre.google/workbook/alerting-on-slos/
- Rob Ewaschuk, My Philosophy on Alerting — mirror https://gist.github.com/msgodf/86a3fc7fcd3ce663ff37 · original https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcApr8Q/preview · mirror https://linuxczar.net/sysadmin/philosophy-on-alerting/
- Prometheus, Alerting best practices (batch jobs: ~2 cycles) — https://prometheus.io/docs/practices/alerting/
- Prometheus, When to use the Pushgateway (three downsides) — https://prometheus.io/docs/practices/pushing/
- kube-prometheus Watchdog runbook (always-firing dead man's switch) — https://runbooks.prometheus-operator.dev/runbooks/general/watchdog/
- Grafana Alerting, connectivity errors (`for` unreliability; `avg_over_time` smoothing) — https://grafana.com/docs/grafana/latest/alerting/guides/connectivity-errors/
- dbt, source freshness (`warn_after`/`error_after`, per-table override) — https://docs.getdbt.com/reference/resource-properties/freshness
- Spidermon, Monitors reference (built-in monitor/threshold catalogue) — https://spidermon.readthedocs.io/en/latest/monitors.html
- Healthchecks.io docs — https://healthchecks.io/docs/ · cron guide https://healthchecks.io/docs/monitoring_cron_jobs/
- healthchecks/healthchecks on GitHub (BSD-3, self-hosting reqs) — https://github.com/healthchecks/healthchecks
- Open States, Testing Scrapers — https://docs.openstates.org/contributing/testing-scrapers/

### Pricing (verified 2026-09-06)
- Healthchecks.io — https://healthchecks.io/pricing/
- Cronitor — https://cronitor.io/pricing
- Dead Man's Snitch — https://deadmanssnitch.com/plans
- Better Stack — https://betterstack.com/pricing
- Grafana Cloud — https://grafana.com/pricing/
- Datadog — https://www.datadoghq.com/pricing/
- Sentry — https://sentry.io/pricing/

### Practitioner accounts
- Simon Willison, Git scraping (2020-10-09) — https://simonwillison.net/2020/Oct/9/git-scraping/ · series https://simonwillison.net/series/git-scraping/
- HN: Git scraping discussion (2020) — https://news.ycombinator.com/item?id=24732943
- HN: healthchecks.io discussion, ydant/cuu508 exchange (2022) — https://news.ycombinator.com/item?id=32038197
- Max Rozen (OnlineOrNot), Cron job monitoring guide (upd. 2026-03-03) — https://onlineornot.com/cron-job-monitoring-guide
- Paul's Programming Notes, Dead man's switch for single-host monitoring (2026-07-05) — https://www.paulsprogrammingnotes.com/2026/07/dead-mans-switch-single-host-monitoring.html
- Ian Kerins, How to monitor your Scrapy spiders (2022-01-12) — https://dev.to/iankerins/how-to-monitor-your-scrapy-spiders-5c9o
- ScrapeOps, Spidermon guide (worked configs) — https://scrapeops.io/python-scrapy-playbook/extensions/scrapy-spidermon-guide/
- Apify, Why monitor long-running scraping projects (2023-04-15) — https://dev.to/apify/why-you-need-to-monitor-long-running-large-scale-scraping-projects-5flj

### Opinion / disagreement
- Marcus Hill (Chronosphere), Stop thinking in burn rates (2025-07-01) — https://chronosphere.io/learn/stop-thinking-in-burn-rates/
- Khavronenko & Palmersheim (VictoriaMetrics), Alerting best practices (2025-08-22) — https://victoriametrics.com/blog/alerting-best-practices/
- David Cramer (Sentry founder), The Problem with OpenTelemetry — https://cra.mr/the-problem-with-otel/
- Moshe Shaham, Distributed Tracing Is Overrated (2026-01-14) — https://mosheshaham.substack.com/p/distributed-tracing-is-overrated
- futurion.blog, Uptime Kuma vs Healthchecks.io for solo self-hosters (2026-04-07, upd. 2026-08-25) — https://futurion.blog/self-hosting-uptime-kuma-vs-healthchecks-io-honest-trade-offs-for-solo-builders/

### Vendor content (useful taxonomies, unevidenced statistics)
- Ficstar, Why scraping fails silently (2026-06-29) — https://www.ficstar.com/why-scraping-fails-silently
- Context.dev, Scraper monitoring in production (2026-08-16) — https://www.context.dev/blog/scraper-monitoring-in-production
- Crawlex, Scraping observability (2026-03-14) — https://blog.crawlex.net/blog/scraping-observability/
- PromptCloud, 10 web scraping monitoring challenges (2026-02-18) — https://www.promptcloud.com/blog/web-scraping-monitoring-challenges/
- Barr Moses (Monte Carlo), 5 Pillars of Data Observability (2020-12-23) — https://montecarlo.ai/blog-introducing-the-5-pillars-of-data-observability
- Ryan Kearns (Monte Carlo), Build your own data anomaly detectors using SQL (2025-04-18, upd. 2025-07-01) — https://montecarlo.ai/blog-how-to-build-your-own-data-anomaly-detectors-using-sql
- SigNoz, Datadog custom metrics pricing (2026-07-17) — https://signoz.io/blog/datadog-custom-metrics-pricing/
- Better Stack, Open source log management comparison — https://betterstack.com/community/comparisons/open-source-log-managament/
- Pi Stack, Self-hosted data quality tools compared (2026) — https://www.pistack.xyz/posts/self-hosted-data-quality-tools-great-expectations-soda-dbt-guide-2026/

### Research aggregation
- pingfatigue.com/research (alert-fatigue citations; healthcare 85–99% false-positive figures) — https://pingfatigue.com/research

### Referenced but not recommended
- github.com/Kriss-V/deadmancheck (0 stars, no releases; payload-assertion pattern only) — https://github.com/Kriss-V/deadmancheck



---

# 07 — Capacity & Resource Requirements for Continuous Multi-Source Harvesting

**Research date:** 2026-09-06. Every number below carries its source date. Figures that are vendor
marketing, secondary reporting, or my own arithmetic are labelled as such.

**Scope note:** the target system is many separate scrapers/engines against **government and
scientific** sites and APIs, running unattended for months. That matters, because those sources
(a) publish documented rate limits, (b) usually publish *bulk downloads* that make scraping partly
unnecessary, and (c) rarely require anti-bot evasion — which removes the two biggest cost drivers
(headless browsers and residential proxies) from the default design.

---

## 1. The dominant cost split: HTTP-only vs headless browser

This is the single largest lever, larger than any server-sizing decision.

### 1.1 Per-page cost, measured

| Metric | HTTP-only client | Headless browser | Ratio | Source |
|---|---|---|---|---|
| Pages per Apify Compute Unit (1 GB × 1 h) | **3,000** (Cheerio) | **300** (Puppeteer) | **10×** | [Apify: estimate CU usage](https://help.apify.com/en/articles/3470975-how-to-estimate-compute-unit-usage-for-your-project) |
| Speed claim (same doc) | — | — | "up to **20×** faster" for Cheerio | Apify, ibid. |
| Recommended memory | **128–512 MB** (Cheerio) | **1,024 MB minimum**, 4,096–16,384 MB for long runs | 2–32× | Apify, ibid. |
| Platform floor | none stated | **1,024 MB required**; ≥4,096 MB for complex sites (e.g. Google Maps) | — | [Apify: usage and resources](https://docs.apify.com/platform/actors/running/usage-and-resources) |
| Bytes on the wire per page | ~40–60 KB (HTML doc only, gzipped — derived from Common Crawl, §4) | **2,311 KB median mobile / 2,652 KB median desktop**, ~66–71 requests | **~40–60×** | [Web Almanac 2024, Page Weight](https://almanac.httparchive.org/en/2024/page-weight) (Oct 2024 data) |
| Latency per page | 0.1–0.5 s | 1–5 s | 10× | [crawlex, 2026-03-16](https://blog.crawlex.net/blog/headless-browser-tax/) — *secondary, ranges not controlled benchmarks* |

**Honest conclusion: the "10–100×" range in the brief is correct, but the multiple depends on what
you measure.** CPU/CU: ~10×. Memory: 2–32×. Bandwidth: ~40–60×. Latency: ~10×. Anyone quoting a
single number is picking one axis.

### 1.2 RAM per Chromium instance / context / page — this is where sources genuinely disagree

| Figure | Value | Date | Status |
|---|---|---|---|
| Browserless: "roughly **10 concurrent requests per GB** of memory" (⇒ ~100 MB/session) | ~100 MB | **2018-06-04** | [browserless blog](https://www.browserless.io/blog/observations-running-headless-browser) — vendor guidance, **8 years old** |
| Browserless default `CONCURRENT` | **10** | current | [browserless docs](https://docs.browserless.io/enterprise/open-source) |
| Browserless required `shm_size` | **2 GB** (Docker default 64 MB crashes Chrome) | current | ibid. |
| Cold headless browser, before page load | **50–150 MB** | 2026-03 | crawlex — secondary |
| Headless browser **rendering a typical page** | **300–500 MB** per concurrent instance | 2026-03 | crawlex — secondary |
| HTTP client, per connection | **1–10 MB** | 2026-03 | crawlex — secondary |
| Apify browser actor floor / recommended | **1 GB / 4 GB** | current | Apify docs |
| Playwright on contexts vs browsers | "fast and cheap to create and are completely isolated, even when running in a single browser" — **no numbers given** | current | [Playwright docs](https://playwright.dev/docs/browser-contexts) |

**The disagreement is 100 MB vs 300–500 MB vs 1,000 MB — a 3–10× spread, and it is not
reconcilable by averaging.** The most likely explanation: 100 MB/session is a *cold browser* number
from 2018, when the median page was a fraction of today's 2.3 MB / 66-request page. A browser that
has actually rendered a modern page and run 22–24 JS files is in the 300–500 MB band, and Apify —
who bill for this and therefore have an incentive to be accurate — will not let you start a browser
actor below 1 GB.

**Planning implication:** budget **300–500 MB per concurrently-rendering page**, not 100 MB. Treat
"10 browsers per GB" as an unachievable ceiling.

### 1.3 Concurrent browsers per vCPU

There is no authoritative published table. The only firm platform datum:

> "For every `4096MB` of memory, the Actor receives one full CPU core." — [Apify](https://docs.apify.com/platform/actors/running/usage-and-resources)

Combined with Apify's 1 GB floor / 4 GB recommendation for browser actors, Apify's own platform
economics imply **1–4 concurrent browsers per full core**. Apify also warns: "Node.js actors cannot
benefit from memory beyond `4096MB`" unless multi-threaded, because Node is single-threaded — so
scaling a browser fleet means **more processes**, not bigger ones.

Derived planning band (my arithmetic, from the figures above — *not* a cited benchmark):

| Box | Concurrent rendering pages (300–500 MB each, 1–4/core) | Concurrent HTTP fetches (1–10 MB each) |
|---|---|---|
| 2 vCPU / 4 GB | 2–8 | several hundred |
| 4 vCPU / 8 GB | 4–16 | ~1,000 |
| 8 vCPU / 32 GB | 8–32 | several thousand (fd/port-limited, §3.3) |
| 16 vCPU / 64 GB | 16–64 | port-limited before RAM-limited |

**Cost caveat added in 2026:** Hetzner attributed its June 2026 price rises to DRAM prices up
"roughly **171% year-over-year**" (§6.1). RAM-heavy designs got materially more expensive in 2026
relative to I/O-bound designs. The headless-browser tax is currently *rising*.

---

## 2. Concrete HTTP throughput benchmarks

| Benchmark | Result | Conditions | Source |
|---|---|---|---|
| `scrapy bench` | "Scrapy is able to crawl about **3,000 pages per minute**" (**50 pages/s**); sample output shows 4,200 / 3,840 / 3,360 / 2,880 pages/min | **Local link-generating mock server, no real network, 0 items parsed, hardware unspecified** | [Scrapy benchmarking docs](https://docs.scrapy.org/en/latest/topics/benchmarking.html) |
| Apify CheerioCrawler, 1,000 pages, example.com | 34 s ⇒ **29 pages/s** | real network, trivial pages | [Apify CU estimation](https://help.apify.com/en/articles/3470975-how-to-estimate-compute-unit-usage-for-your-project) |
| Apify CheerioCrawler, 1,000 pages, apify.com | 82 s ⇒ **12 pages/s** | real network, average pages | ibid. |
| Apify CheerioCrawler, 1,000 pages, amazon.com | 258 s ⇒ **3.9 pages/s** | real network, heavy pages | ibid. |
| Page-weight sensitivity | up to **7.5×** consumption swing | same crawler, different sites | ibid. |

**Flagged disagreement: 50 pages/s (Scrapy docs) vs 3.9–29 pages/s (Apify, real sites) — a
2–13× gap.** `scrapy bench` measures framework overhead against localhost with no parsing; it is a
*ceiling*, not a planning number. **Plan on 4–30 pages/s per worker process** for real HTTP-only
crawling of real sites, and toward the bottom of that band if you parse heavily.

### 2.1 Scrapy's own tuning numbers for broad crawls

From [Scrapy: Broad Crawls](https://docs.scrapy.org/en/2.11/topics/broad-crawls.html):

| Setting | Recommended | Why |
|---|---|---|
| `CONCURRENT_REQUESTS` | **100** ("a good starting point"), tune to **80–90% CPU utilisation** | throughput |
| `REACTOR_THREADPOOL_MAXSIZE` | **20** | "Scrapy does DNS resolution in a blocking way with usage of thread pool. With higher concurrency levels the crawling could be slow or even fail hitting DNS resolver timeouts." |
| `DOWNLOAD_TIMEOUT` | **15** s | stop stuck requests holding resources |
| `RETRY_ENABLED`, `REDIRECT_ENABLED`, `COOKIES_ENABLED` | **False** | CPU/memory |
| `LOG_LEVEL` | `INFO` | reduces CPU overhead |
| Scheduler | `DownloaderAwarePriorityQueue` | fairness across many domains |

Note the explicit statement that **CPU utilisation, not a fixed number, should set your concurrency
ceiling** — and that "increasing concurrency also increases memory usage."

### 2.2 Client library defaults you will hit

| Library | Setting | Default | Consequence |
|---|---|---|---|
| aiohttp | `TCPConnector(limit=…)` | **100** total | [docs](https://docs.aiohttp.org/en/stable/client_advanced.html) |
| aiohttp | `limit_per_host` | **0 = no limit** | **politeness footgun** — unlimited concurrent connections to one host unless you set it |
| aiohttp | `ttl_dns_cache` | **10 s** | far too short when polling the same N hosts for months; causes needless DNS load |
| httpx | `max_connections` | **100** | [docs](https://www.python-httpx.org/advanced/resource-limits/) |
| httpx | `max_keepalive_connections` | **20** | forces reconnects (TLS handshakes) above 20 |
| httpx | `keepalive_expiry` | **5 s** | idle connections dropped fast; re-handshake cost on slow-cadence polling |
| Crawlee (JS) | `maxConcurrency` | **200** (starts at 1, scales up) | [AutoscaledPoolOptions](https://crawlee.dev/js/api/core/interface/AutoscaledPoolOptions) |
| Crawlee | `desiredConcurrencyRatio` / step ratios | **0.90** / **0.05** | [ibid.](https://crawlee.dev/js/docs/guides/scaling-crawlers) |

### 2.3 Where the bottleneck actually moves

Ordered by when you hit it, for HTTP-only work:

1. **Per-domain politeness ceiling** (§3) — hit almost immediately. Binding constraint.
2. **DNS** — Scrapy names this explicitly as the thing that makes high-concurrency crawls "slow or
   even fail" ([broad-crawls docs](https://docs.scrapy.org/en/2.11/topics/broad-crawls.html)).
   aiohttp's 10-second DNS cache TTL makes it worse. Fix: local caching resolver.
3. **TLS handshakes** — only if keep-alive is misconfigured (httpx caps keep-alive at 20).
4. **Ephemeral ports** — Linux default `net.ipv4.ip_local_port_range` is **32768–60999**, i.e.
   **28,232 ports** ([kernel ip-sysctl docs](https://www.kernel.org/doc/html/latest/networking/ip-sysctl.html)).
   With `tcp_fin_timeout` default **60 s**, ~28k sockets churning per minute is the hard wall per
   source IP. At 1 req/s/domain you would need ~28,000 domains to reach it — so **not a real
   constraint for this workload**.
5. **File descriptors** — `ulimit -n` (commonly 1024 default) bites long before ports do; raise it.
6. **CPU (parsing)** — only if you parse heavily. Scrapy's advice to tune to 80–90% CPU exists
   because parsing, not fetching, is the CPU consumer.

---

## 3. Politeness, not hardware, is the binding constraint

### 3.1 Documented rate limits at real government/scientific sources

| Source | Documented limit | Effective req/s | Citation |
|---|---|---|---|
| **SEC EDGAR** | "our current maximum access rate is **10 requests per second**"; declared User-Agent with contact email required | 10 | [SEC webmaster FAQ](https://www.sec.gov/os/webmaster-faq) |
| **NCBI E-utilities** (PubMed etc.) | **3 req/s** without API key, **10 req/s** with key; "limit large jobs to either weekends or between 9:00 PM and 5:00 AM Eastern"; IP blocking for non-compliance | 3–10, **time-of-day restricted** | [NCBI NBK25497](https://www.ncbi.nlm.nih.gov/books/NBK25497/) |
| **arXiv API** | "make no more than **one request every three seconds**, and limit requests to a **single connection at a time**"; limit applies "collectively to all machines under your control" | **0.33** | [arXiv API ToU](https://info.arxiv.org/help/api/tou.html) |
| **Crossref REST API** (from **2025-12-01**) | Public pool: **5 req/s** single record / **1 req/s** list, **1 concurrent**. Polite pool (`mailto`): **10 req/s** single / **3 req/s** list, **3 concurrent** | 1–10 | [Crossref blog](https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/) |
| **api.data.gov** (fronts many federal APIs) | **1,000 requests/hour** per key = **0.28 req/s**; `DEMO_KEY` **30/hour, 50/day**; 429 on excess, rolling 1-hour unblock | **0.28** | [api.data.gov developer manual](https://api.data.gov/docs/developer-manual/) |
| **OpenAlex** | **100 req/s**, **100,000 credits/day** free (list request = 10 credits ⇒ **10,000 list calls/day**) | 100 burst, **daily-capped** | [OpenAlex docs](https://github.com/ourresearch/openalex-docs/blob/main/how-to-use-the-api/rate-limits-and-authentication.md) |

### 3.2 What established crawlers do by default

| Crawler | Default politeness | Effective req/s per host | Citation |
|---|---|---|---|
| **Apache Nutch** | `fetcher.server.delay` = **5.0 s**; `fetcher.threads.per.queue` = **1**; `fetcher.max.crawl.delay` = **30 s** (skip pages whose robots Crawl-Delay exceeds it) | **0.2** | [nutch-default.xml](https://raw.githubusercontent.com/apache/nutch/master/conf/nutch-default.xml) |
| **Heritrix** (Internet Archive) | "only a **single connection to a server at a time**" as its fundamental rule; `delay-factor` × last fetch duration, bounded by `min-delay-ms`/`max-delay-ms`; documented example targets "never more than one request every 2 seconds" | ~0.5 or less | [Heritrix politeness parameters](https://github.com/internetarchive/heritrix3/wiki/Politeness-parameters) |

**Note how far below any hardware limit these are.** Nutch's out-of-the-box default is **0.2
requests per second per host**. Heritrix — the crawler that built the Wayback Machine — holds
**one connection per server**.

### 3.3 The arithmetic

Total sustainable request budget = **Σ (per-source ceiling)**, independent of CPU.

| N sources | @ 0.2 req/s (Nutch default) | @ 1 req/s (common polite target) | @ 10 req/s (SEC-class ceiling) |
|---|---|---|---|
| 10 | 2 req/s → 173k req/day | 10 req/s → 864k/day | 100 req/s → 8.6M/day |
| 100 | 20 req/s → 1.7M/day | 100 req/s → 8.6M/day | 1,000 req/s → 86M/day |
| 1,000 | 200 req/s → 17.3M/day | 1,000 req/s → 86M/day | 10,000 req/s → 864M/day |
| 10,000 | 2,000 req/s → 173M/day | 10,000 req/s → 864M/day | 100,000 req/s → 8.6B/day |

Now compare against per-worker capacity (§2, real-site 4–30 pages/s):

- **100 sources × 1 req/s = 100 req/s.** That is *one* Scrapy process at Scrapy's own suggested
  `CONCURRENT_REQUESTS=100`, or ~4–25 worker processes if each sustains 4–30 pages/s. Fits on a
  2–4 vCPU box.
- **1,000 sources × 1 req/s = 1,000 req/s.** ~35–250 worker processes' worth of throughput. Still
  a small number of machines for HTTP-only — but at this point DNS, connection pooling and the
  scheduler are the engineering problem, not the CPU.
- **1,000 sources × 0.2 req/s (Nutch-default politeness) = 200 req/s.** Comfortably one machine.

**And crucially, nobody polls every source flat-out.** For change detection on government pages the
cadence is per-source (hourly, daily, weekly), so actual load is a small fraction of the politeness
ceiling. **The request budget, not the CPU, is what you plan against — and the request budget is set
by the publishers, not by you.**

### 3.4 The bigger lever: bulk downloads

Several of these sources explicitly tell you not to scrape them:

- **Crossref** recommends the annual **Public Data File**, **Metadata Plus** monthly snapshots, and
  "store results locally since most records don't change on a regular basis"
  ([source](https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/)).
- **NCBI** advises using the **Entrez History server** to "upload and download records in batches
  rather than making individual requests for each record"
  ([source](https://www.ncbi.nlm.nih.gov/books/NBK25497/)).
- **OpenAlex** documents OR-syntax batching to collapse many requests into one.

One bulk file can replace millions of requests. **This is a larger capacity lever than any hardware
choice in this document.**

---

## 4. Storage: real numbers

### 4.1 Common Crawl as the primary anchor

| Crawl | Pages | Uncompressed | WARC (gzip) | WAT | WET (text) | Hosts | Registered domains |
|---|---|---|---|---|---|---|---|
| **CC-MAIN-2026-34** (Aug 7–20, 2026) | **2.14 B** | **360 TiB** | **84.78 TiB** | 13.92 TiB | **5.84 TiB** | 40.2 M | 33.1 M |
| CC-MAIN-2026-05 (Jan 12–25, 2026) | 2.3 B | 398 TiB | 85.00 TiB | 15.85 TiB | 6.49 TiB | 44.9 M | 36.9 M |
| Aug 2025 (Aug 2–16, 2025) | 2.44 B | 424 TiB | 88.24 TiB | — | — | 47.5 M | 38.5 M |

Sources: [Aug 2026](https://commoncrawl.org/blog/august-2026-crawl-archive-now-available),
[Jan 2026](https://commoncrawl.org/blog/january-2026-crawl-archive-now-available),
[Aug 2025](https://commoncrawl.org/blog/august-2025-crawl-archive-now-available),
[blog index](https://commoncrawl.org/blog). Each crawl = 100,000 WARC files.

**Derived per-page averages (my arithmetic on the above):**

| Quantity | Aug 2026 | Jan 2026 | Aug 2025 |
|---|---|---|---|
| Uncompressed bytes/page | **~181 KiB** | ~186 KiB | ~187 KiB |
| **Compressed WARC bytes/page** | **~42.5 KiB** | ~39.7 KiB | ~38.8 KiB |
| gzip compression ratio | **4.25×** | 4.68× | 4.81× |
| Compressed **text-only** (WET) bytes/page | **~2.9 KiB** | ~3.0 KiB | — |
| **WARC → WET reduction** | **14.5×** | 13.1× | — |

**Headline numbers to plan with: ~40 KB per page stored (raw HTML, gzipped WARC), ~3 KB per page if
you keep extracted text only, ~4.2–4.8× gzip ratio on HTML.**

Also: Aug 2026 contained **0.7 B new URLs out of 2.14 B** — i.e. **~67% of fetches were re-visits**
to previously-seen URLs. That is the empirical basis for "content-hash dedup pays for itself."

### 4.2 gzip vs zstd on actual WARC data

Measured on a single Common Crawl WARC file (4,463,763,277 bytes raw):

| Codec | Compressed size | Time | vs gzip |
|---|---|---|---|
| gzip | 941,893,587 B | — | baseline |
| zstd -1 | 707,109,581 B | 10 s | **−25%** |
| **zstd -5 (default)** | **599,217,283 B** | 26 s | **−36%** |
| zstd -10 | 549,859,495 B | 69 s | −42% |
| zstd -12 | 536,512,569 B | 109 s | −43% |

Projected across the Feb 2017 crawl: **55.88 TB → 35.5 TB, ≈37% saving**. Decompression: zstd is
"about **2× as fast** as zlib, no matter which compression level."

Source: [Common Crawl mailing list, Ben Wills & Greg Lindahl, 2017-03-15](https://groups.google.com/g/common-crawl/c/bO6B6xQJnEE).
**Date flag: 2017.** zstd has improved since; treat 37% as a conservative floor. There is now a
formal [Zstandard Compression for WARC Files 1.0](https://iipc.github.io/warc-specifications/specifications/warc-zstd/)
spec from IIPC, so this is a supported path, not a hack.

### 4.3 Government-data anchors (directly relevant)

| Dataset | Size | Scale | Source |
|---|---|---|---|
| **Harvard LIL data.gov archive** | **16 TB** | **311,000+ datasets**, harvested 2024–2025, "updated daily as new datasets are added" | [LIL announcement, 2025-02-06](https://lil.law.harvard.edu/blog/2025/02/06/announcing-data-gov-archive/) |
| **End of Term Web Archive 2024** | **2.29 PB compressed** | 1,216,891 WARC files | [eotarchive.org/data](https://eotarchive.org/data/) |
| EOT 2020 | 266.04 TB | 239,811 WARCs | ibid. |
| EOT 2016 | 284 TB | 198,593 WARCs | ibid. |
| EOT 2012 | 41.42 TB | 78,509 WARCs | ibid. |
| EOT 2008 | 15.32 TB | 125,704 WARCs | ibid. |
| EOT 2004 | 6.42 TB | 58,977 WARCs | ibid. |

**Two important reads.** (1) data.gov: **~54 MB average per dataset** — government *data files* are
1,000× larger per item than HTML pages, so a dataset-harvesting system is storage-dominated in a way
an HTML scraper is not. (2) EOT: a comprehensive .gov crawl went **266 TB → 2.29 PB in one
four-year term (8.6×)**. If you crawl .gov broadly, storage growth is superlinear in wall-clock time,
not just in source count.

### 4.4 Per-source storage growth math (my arithmetic, using ~40 KB/page compressed)

| Pages/day/source | 1 source | 10 sources | 100 sources | 1,000 sources |
|---|---|---|---|---|
| 100 | 1.5 GB/yr | 15 GB/yr | 146 GB/yr | 1.5 TB/yr |
| 1,000 | 14.6 GB/yr | 146 GB/yr | **1.46 TB/yr** | 14.6 TB/yr |
| 10,000 | 146 GB/yr | 1.46 TB/yr | 14.6 TB/yr | 146 TB/yr |

Adjustments to that table:

- **zstd instead of gzip:** ×0.63 (§4.2) → 100 sources @ 1k pages/day ≈ **0.92 TB/yr**.
- **Text/extracted-fields only:** ×0.069 (WARC→WET, §4.1) → ≈ **100 GB/yr**.
- **Keep only changes (content-hash dedup):** Common Crawl's 67% re-visit rate is the only hard
  datum here; for slow-moving government pages the unchanged fraction on re-fetch is much higher
  than 67%, so dedup typically dominates every other storage decision. I found **no published
  measurement** of the byte-identical re-fetch rate for .gov pages — treat this as the biggest
  unquantified variable in the storage model.
- **Every version vs only changes:** at daily polling, keeping every version multiplies storage by
  the polling cadence (365× per year vs one snapshot); keeping only changes multiplies it by the
  actual change rate. The gap between those is 1–2 orders of magnitude and is the difference between
  "a 100 GB disk" and "an object-storage bill."

### 4.5 Object-storage request costs (the hidden line item)

| Provider | Storage | Write ops | Read ops | Egress |
|---|---|---|---|---|
| **AWS S3 Standard** (us-east-1) | **$0.023/GB-mo** = $23/TB-mo | PUT/COPY/POST **$0.005 per 1,000** = **$5/million** | GET $0.0004/1,000 = $0.40/M | **$0.09/GB** (see §6.3) |
| **Cloudflare R2** Standard | **$0.015/GB-mo** = $15/TB-mo | Class A **$4.50/million** | Class B $0.36/M | **$0 — free** |
| R2 Infrequent Access | $0.01/GB-mo | $9.00/M | $0.90/M | $0 |
| **Backblaze B2** | **$6.95/TB-mo** | free (Class A/B/C) | free | **free up to 3× stored**, then $0.01/GB |
| B2 Overdrive | $15/TB-mo | free | free | unlimited free |

Sources: [S3 pricing](https://aws.amazon.com/s3/pricing/),
[R2 pricing](https://developers.cloudflare.com/r2/pricing/),
[B2 pricing](https://www.backblaze.com/cloud-storage/pricing). Fetched 2026-09-06.

**Two traps.**

1. **One object per page is a tax.** At 100k pages/day = 3M PUTs/month: S3 **$15/mo**, R2 $13.50/mo
   — negligible. At 10M pages/day = 300M PUTs/month: S3 **$1,500/mo in PUTs alone**, before storing
   a byte. This is precisely why the archiving world batches into WARC files (Common Crawl:
   2.14 B pages → **100,000 WARC files**, i.e. ~21,000 pages per file).
2. **Minimum billable object size.** S3 Standard-IA and Glacier Instant Retrieval have a **128 KB
   minimum object size** ([S3 pricing](https://aws.amazon.com/s3/pricing/)). A 40 KB compressed page
   stored individually in S3-IA is billed as 128 KB — a **3.2× overcharge** that silently erases the
   tiering saving. Glacier Flexible adds 40 KB metadata overhead per object; minimum durations are
   30 days (IA), 90 days (Glacier IR/Flexible), 180 days (Deep Archive).

**Annual storage cost, 100 sources @ 1,000 pages/day, every version, gzipped (1.46 TB/yr):**
S3 $403/yr; R2 $263/yr; B2 $122/yr. With zstd + text-only extraction the same corpus is ~100 GB/yr
and costs **$8–$28/yr**. The compression/extraction decision is worth ~15–50× more than the
provider decision.

---

## 5. Database sizing

### 5.1 PostgreSQL hard limits

| Limit | Value | Note |
|---|---|---|
| Max table size | **32 TB** | with default 8,192-byte `BLCKSZ` |
| Max rows/table | limited by **4,294,967,295 pages** | depends on row width |
| Max field size | **1 GB** | a single scraped blob above this simply cannot be stored |
| Max columns/table | **1,600** | further constrained by the tuple fitting one 8 KB page |
| Max partition keys | 32 | recompile to raise |

Source: [PostgreSQL Appendix K, Limits](https://www.postgresql.org/docs/current/limits.html).

### 5.2 When Postgres stops being enough — the documented trigger

> "a rule of thumb is that **the size of the table should exceed the physical memory of the database
> server**" — [PostgreSQL: Table Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html)

That is the actual, official trigger: **not a row count, but table size vs RAM.** Also from the same
page:

- Declarative partitioning: "the query planner is generally able to handle partition hierarchies with
  up to **a few thousand partitions** fairly well, **provided that typical queries allow the query
  planner to prune all but a small number** of partitions."
- Legacy inheritance partitioning: "will work well with **up to perhaps a hundred child tables**;
  don't try to use many thousands of children."
- "the server's memory consumption may grow significantly over time, especially if many sessions
  touch large numbers of partitions… each partition requires its metadata to be loaded into the local
  memory of **each session** that touches it." — a real hazard for a long-running unattended system
  with a persistent connection pool.
- "Never just assume that more partitions are better than fewer partitions, nor vice-versa."

**Derived row-count band (my arithmetic, clearly labelled as such — no source publishes this):**
a page-metadata row (URL, hashes, timestamps, status, source id) is ~200–500 bytes plus index
entries. On a 32–64 GB database server, the RAM-crossover therefore lands somewhere around
**50–200 million rows** for the metadata table. Below that, plain Postgres. Above it, partition by
time and/or source. Any specific number you see quoted ("Postgres breaks at 100M rows") is a
row-width-and-RAM artefact, not a property of Postgres.

**The bigger architectural point:** the 32 TB table limit and the 1 GB field limit both say the same
thing — **do not put raw HTML in Postgres.** At ~40 KB/page, 32 TB is only **800 M pages**, and
you pay TOAST plus WAL write amplification for every byte. The standard split is metadata + extracted
fields in Postgres, raw payloads in object storage (§4.5) keyed by content hash.

### 5.3 TimescaleDB / TigerData

Documented: delta-of-delta encoding on timestamps compresses "a full timestamp of 8 bytes, or 64
bits, down to just a single bit, resulting in **64× compression**." Algorithms by type: delta /
delta-of-delta / simple-8b / RLE for integers and timestamps; XOR-based (Gorilla) for floats;
dictionary for others.
([TigerData: compression methods](https://www.tigerdata.com/docs/learn/columnar-storage/compression-methods))

**Flagged:** the widely-repeated "90–95% compression" claim for TimescaleDB is **not stated on the
current docs page I read**. The 64× figure applies to timestamp columns specifically, which is the
best case. Scraped-page metadata is mostly text and hashes — high-entropy, poorly compressible —
so do not assume time-series compression ratios transfer to this workload.

### 5.4 Parquet / DuckDB / object storage

- **Common Crawl's URL index in Parquet: "The index table of a single monthly crawl has about
  **300 GB**"** for a ~2–3 billion-row index — i.e. **~100 bytes/row** for URL + WARC file + offset
  + length + status, columnar and compressed. ([Common Crawl, 2018-03-01](https://commoncrawl.org/blog/index-to-warc-files-and-urls-in-columnar-format))
  **Date flag: 2018**, but the per-row figure is a good order-of-magnitude anchor for "URL index in
  Parquet."
- Same post: a per-domain query over that index scanned **2.12 MB** and "cost less than one cent" —
  the columnar-pruning argument in one datum.
- DuckDB over Parquet, NYC taxi (**21 M rows × 18 columns, ~360 MB across 3 files**):
  `COUNT(*)` 15 ms; date-predicate filter 45–50 ms; `GROUP BY` aggregation 220 ms.
  ([DuckDB, 2021-06-25](https://duckdb.org/2021/06/25/querying-parquet.html)) — **that post does not
  publish a Parquet-vs-JSON size ratio**, so I am not asserting one. The defensible statement is
  Common Crawl's 100 bytes/row for real crawl-index data.

**Practical staging:** Postgres for the operational/queue/metadata tier while the working set fits
RAM → partition by time+source when it doesn't → Parquet on object storage + DuckDB for the
analytical corpus, which removes the row-count question entirely.

---

## 6. Cost anchors (verified 2026-09-06 where possible)

### 6.1 Hetzner — **major 2026 repricing; treat all figures as secondary**

I could **not** verify Hetzner prices against hetzner.com directly — their cloud and dedicated
pricing pages are JavaScript-rendered and returned only marketing copy. Everything below is from
Hetzner's own 2024 press release (for specs) plus multiple independent secondary reports (for the
2026 changes). **Verify against the live price calculator before using.**

Hetzner's own 2024 launch specs ([Hetzner pressroom, 2024-06-06](https://www.hetzner.com/pressroom/new-cx-plans/)):

| Plan | vCPU | RAM | Disk | **Traffic** | Launch price (2024) |
|---|---|---|---|---|---|
| CX22 | 2 | 4 GB | 40 GB | **20 TB** | €3.79/mo |
| CX32 | 4 | 8 GB | 80 GB | 20 TB | €6.80/mo |
| CX42 | 8 | 16 GB | 160 GB | 20 TB | €16.40/mo |
| CX52 | 16 | 32 GB | 320 GB | 20 TB | €32.40/mo |

**The 20 TB included traffic is the number that matters for scraping** — it is 1,000× DigitalOcean's
smallest allowance and effectively unlimited for an ingress-heavy workload.

**2026 price increases** (two rounds — April 1 and June 15, 2026):

| Plan | Pre-April 2026 | April 1, 2026 | June 15, 2026 | June change |
|---|---|---|---|---|
| CCX13 (dedicated vCPU) | ~€12.40 | €15.99 | **€42.99** | **+169%** |
| CCX23 | ~€24.30 | €31.49 | **€85.99** | **+173%** |
| CCX33 | — | €62.49 | €138.49 | +122% |
| CCX63 | — | €374.49 | €853.49 | +128% |
| CPX22 (shared AMD) | — | €7.99 | **€19.49** | **+144%** |
| CPX32 | ~€10.40 | €13.99 | €34.99–35.49 | +150–154% |
| CPX52 | — | €36.49 | **€100.49** | **+175%** |
| CX23 (cost-optimised) | — | €3.99 | €5.49 | **+38%** |
| CX33 | ~€8.70 | €11.36 | €15.49 | +36% |
| CAX11 (ARM) | — | €4.49 | €5.99 | +33% |
| CAX21 (ARM) | ~€7.40 | €9.99 | €13.49 | +35% |

Sources: [Northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases),
[wz-it](https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/),
[bex.co](https://bex.co/blog/2026/08/21/hetzner-second-price-hike-ccx-cpx-113-percent-fleet-economics),
[cdnsun](https://blog.cdnsun.com/ovhcloud-and-hetzner-2026-hosting-price-increases-explained/).

**Sources agree on the June 15 2026 figures and on the effective date ("08:00 CEST", new orders and
rescales only — existing servers keep their price). They disagree on the pre-April baseline**
(bex.co gives CCX13 ≈ €12.40 pre-April; Northflank starts from €15.99). Northflank reports CPX32
at €35.49, bex.co at €34.99. Treat ±5% and the pre-April column as unreliable.

**The stated cause is the most important current signal in this whole document:** Hetzner attributed
the rises to "dramatically increased costs to operate infrastructure and to buy new hardware," with
**DRAM prices up roughly 171% year-over-year** driven by AI demand, plus NVMe/NAND increases
([wz-it](https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/),
[cdnsun](https://blog.cdnsun.com/ovhcloud-and-hetzner-2026-hosting-price-increases-explained/)).
Note the **shape** of the increase: cost-optimised shared lines +33–38%, dedicated-vCPU lines
+114–176%. As of late 2026, **memory and dedicated CPU are the expensive resources, and shared
vCPU is comparatively cheap** — which shifts the economics further against headless browsers
(RAM-bound) and in favour of HTTP-only crawling (I/O-bound, happy on shared vCPU).

**Hetzner dedicated (AX42):** AMD Ryzen 7 PRO 8700GE, 8 cores/16 threads, up to 64 GB DDR5 ECC,
2× 512 GB NVMe, 1 Gbit/s **unlimited traffic** (10G upgrades are metered above 20 TB/mo).
Price not rendered on the page. ([hetzner.com/dedicated-rootserver/ax42](https://www.hetzner.com/dedicated-rootserver/ax42/))
Secondary reports note dedicated-server setup fees were lowered and monthly rates raised in 2026.

### 6.2 DigitalOcean and OVHcloud

**DigitalOcean Droplets** ([pricing page](https://www.digitalocean.com/pricing/droplets), fetched
2026-09-06):

| vCPU | RAM | SSD | **Transfer** | $/mo | $/hr |
|---|---|---|---|---|---|
| 1 | 512 MiB | 10 GB | 500 GB | $4.00 | $0.00595 |
| 1 | 1 GB | 25 GB | 1 TB | $6.00 | $0.00893 |
| 2 | 2 GB | 60 GB | 3 TB | $18.00 | $0.02679 |
| 2 | 4 GB | 80 GB | 4 TB | $24.00 | $0.03571 |
| 4 | 8 GB | 160 GB | 5 TB | $48.00 | $0.07143 |
| 8 | 16 GB | 320 GB | 6 TB | $96.00 | $0.14286 |
| General Purpose 4/16 GB | 4 | 16 GB | 50 GB | 5 TB | $126.00 |
| General Purpose 8/32 GB | 8 | 32 GB | 100 GB | 6 TB | $252.00 |

Also: **per-second billing from 2026-01-01** (60 s / $0.01 minimum). DO's page does **not** state
the bandwidth overage rate; commonly $0.01/GB, **unverified here**.

**Compare:** DO 4 vCPU / 8 GB = **$48/mo with 5 TB**. Hetzner CX32 4 vCPU / 8 GB = **€6.80 (2024) /
~€9–11 (post-2026) with 20 TB**. That is a **4–7× price gap on comparable specs and 4× the
bandwidth**, and it is the single largest cost decision in this section.

**OVHcloud VPS** ([us.ovhcloud.com/vps](https://us.ovhcloud.com/vps/), fetched 2026-09-06):

| Plan | vCore | RAM | Storage | Bandwidth | $/mo |
|---|---|---|---|---|---|
| VPS-1 | 2 | 4 GB | 40 GB NVMe | 500 Mbps | $4.54 |
| VPS-2 | 4 | 8 GB | 75 GB NVMe | 1 Gbps | $8.50 |
| VPS-3 | 6 | 12 GB | 100 GB NVMe | 2 Gbps | $12.32 |
| VPS-4 | 8 | 24 GB | 200 GB NVMe | 3 Gbps | $23.37 |

"Unlimited traffic" on standard plans; **Asia-Pacific has quotas (1 TB VPS-1/2, 3 TB VPS-3/4)**.
Daily 24 h backup included.

**Flagged contradiction:** OVH's own page shows VPS-1 at **$4.54** and VPS-4 at **$23.37** today,
while cdnsun reports an **April 1, 2026** increase taking VPS-1 $4.90→**$7.60** (+55%) and VPS-4
$26.00→**$43.50** (+67%), plus IPv4 $2.00→$2.40
([cdnsun](https://blog.cdnsun.com/ovhcloud-and-hetzner-2026-hosting-price-increases-explained/)).
The most likely reconciliation is **promotional first-term vs renewal pricing** — a real trap for a
system meant to run for months. **Check the renewal price, not the headline price.**

### 6.3 AWS: egress is the story

**Data Transfer OUT to Internet (US/EU regions):**

| Tier | Price |
|---|---|
| First 100 GB/month | **free** (aggregated across all AWS services and regions) |
| Next 10 TB/month | **$0.09/GB** |
| Next 40 TB/month | $0.085/GB |
| Next 100 TB/month | $0.07/GB |
| Over 150 TB/month | $0.05/GB |
| Cross-AZ | $0.01/GB **each way** |
| NAT Gateway processing | $0.045/GB |

Free 100 GB confirmed on [AWS's own EC2 on-demand page](https://aws.amazon.com/ec2/pricing/on-demand/);
tier table from [EgressCost.com, "last verified 2026-06"](https://egresscost.com/aws/data-transfer-pricing/)
— **secondary, but consistent with the long-standing published AWS rates.** AWS's own pricing tables
are JS-rendered and I could not extract them directly.

**The documented complaint** ([Cloudflare, "AWS's Egregious Egress", 2021-07-23](https://blog.cloudflare.com/aws-egregious-egress/)):

- "Wholesale bandwidth is **93% less expensive** than 10 years ago" while "AWS's egress fees over
  that same period have fallen by only **25%**."
- "**Since 2018, the egress fees AWS charges in North America and Europe have not dropped a penny.**"
- Markups quoted up to "**357%**" for Seoul, described as *relatively modest*.
- Conversion factor: 1 Mbps sustained ≈ 0.3285 TB/month (0.3458 TB at 95th-percentile billing).

**Date flag: 2021.** But note that the $0.09/GB first-tier rate is still the verified rate in June
2026 — so Cloudflare's "not a penny since 2018" claim now covers roughly **eight years**.

**Why this matters less than usual for scraping:** a harvesting system is **ingress-dominated**.
Data transfer *in* to AWS is free, and Hetzner/OVH/DO meter primarily outbound. Egress only bites
when you (a) serve the corpus, (b) move it between clouds/regions, or (c) route through a NAT
Gateway at $0.045/GB — which, at 100k pages/day × 50 KB = 5 GB/day, is a trivial **$6.75/mo**, but
at headless-browser volumes (2.3 MB/page → 230 GB/day) becomes **$310/mo for the NAT Gateway alone**.

**R2's zero egress and B2's 3×-stored free egress** (§4.5) are the structural answers if the corpus
will ever be read out at volume.

### 6.4 Serverless: when it's cheaper and when it isn't

**AWS Lambda** ([pricing page](https://aws.amazon.com/lambda/pricing/)): free tier **1 M requests +
400,000 GB-seconds/month**; **$0.20 per million requests**; duration billed per GB-second at
**$0.0000166667/GB-s** (x86 standard rate as shown in the page's worked examples; the full tiered
table is JS-rendered and I could not extract it — tiers apply "up to 6 billion GB-seconds/month").

**Google Cloud Run** ([pricing page](https://cloud.google.com/run/pricing), us-central1):

| Billing model | CPU | Memory | Requests |
|---|---|---|---|
| Instance-based (CPU always allocated) | $0.000018/vCPU-s | $0.000002/GiB-s | — |
| Request-based, active | $0.000024/vCPU-s | $0.0000025/GiB-s | $0.40/M |
| Request-based, idle (min instances) | $0.0000025/vCPU-s | $0.0000025/GiB-s | — |

Free tier: 240,000 vCPU-s + 450,000 GiB-s (instance-based); 180,000 vCPU-s + 360,000 GiB-s +
2 M requests (request-based). Minimum billing **100 ms/invocation**. Egress: Premium tier rates,
**only 1 GiB/month free** in North America.

**Derived per-page costs (my arithmetic):**

| Scenario | Cost/page | Cost per 1 M pages |
|---|---|---|
| Lambda, HTTP fetch+parse, 512 MB, 1 s | $0.0000085 | **~$8.50** |
| Lambda, headless Chrome, 2,048 MB, 10 s | $0.00033 | **~$333** |
| Cloud Run instance-based, 1 vCPU + 2 GiB, always-on | — | **~$58/month** for the instance |
| Hetzner CX32 (4 vCPU/8 GB) doing 1 M pages/mo | ~$0.00001 | ~€7–11/**month total** |

**The crossover.** At 1 M pages/month, Lambda (~$8.50) and a small VPS (~€7–11) are roughly equal —
and Lambda wins on operational simplicity because you are not managing a machine. At 100 M
pages/month, Lambda is **~$850/month** against one or two servers costing ~€20–40. **Serverless wins
when duty cycle is low and bursty — which describes "poll 100 government sites once an hour" quite
well** — and loses badly at high sustained utilisation. Two further caveats: (1) Cloud Run's
always-on instance-based pricing (~$58/mo for 1 vCPU + 2 GiB) is **5–10× a comparable VPS**, so
"serverless" only saves money if you actually scale to zero; (2) Lambda's per-page cost for headless
Chrome (**$333/M pages**) is ~40× the HTTP-only figure, the same 10–100× tax as §1, now denominated
in dollars.

### 6.5 Managed scraping platforms, for reference

- **Zyte Scrapy Cloud:** "from **$9/unit per month**"; "**1 Scrapy Unit = 1 GB of RAM and 1
  concurrent crawl**"; docs say a unit is "1 computing unit, 1 GB of memory, and 2.5 GB of disk."
  Free tier: 1 hour crawl time, 1 concurrent crawl, 7-day retention.
  ([Zyte Scrapy Cloud](https://www.zyte.com/scrapy-cloud/), [units docs](https://docs.zyte.com/scrapy-cloud/usage/units.html))
- **Oxylabs Web Scraper API:** "from **$0.25/1K results**" = **$250 per million pages**
  ([Oxylabs pricing](https://oxylabs.io/pricing)).
- **Cloudflare Browser Rendering:** $0.09/browser-hour + $2.00/additional concurrent browser/month;
  free tier 10 browser-hours + 10 concurrent; paid ceiling 120 concurrent browsers — **July 2025
  pricing per a secondary source ([crawlex](https://blog.crawlex.net/blog/headless-browser-tax/));
  not verified against Cloudflare's own page.**

**Calibration:** $9/unit-month for 1 GB + 1 concurrency (Zyte) vs €3.79–5.49/month for 2 vCPU + 4 GB
+ 20 TB traffic (Hetzner CX22/CX23) is roughly a **10× premium** for the managed layer. Oxylabs at
$250/M pages vs Lambda at ~$8.50/M vs a VPS at ~$0.01/M is a **~25,000× spread** across the
build-vs-buy axis — which mostly reflects that you are buying anti-bot evasion and proxies, not
compute. For .gov/.edu sources you generally don't need either.

---

## 7. Proxy bandwidth: often the dominant line item

| Provider | Entry $/GB | Mid-tier $/GB | Enterprise $/GB |
|---|---|---|---|
| Bright Data | **$8.00** (PAYG) | $6.50 | negotiated |
| IPRoyal | $7.35 | $5.51 | negotiated |
| Oxylabs | $6.00 | $4.00 | **$2.50** |
| Decodo | $4.00 (PAYG) | $3.75 | $2.00 |
| Proxidize | $1.00 | $1.00 | $0.50 |
| DataImpulse | $1.00 | $0.80 | custom |

Source: [Proxidize, 2026-07-26](https://proxidize.com/blog/residential-proxy-pricing/) — **note this
is a vendor's own comparison table and shows Proxidize favourably; treat the competitor rows as
indicative.** Cross-check: [Oxylabs' own pricing page](https://oxylabs.io/pricing) says residential
"starts from **$2.5/GB**" with "$6/GB" for large-scale packages, and prices datacenter proxies at
**$0.7/IP** and ISP at **$1.2/IP** — i.e. per-IP, not per-GB, which is dramatically cheaper for
bandwidth-heavy work. A commonly cited overall range for tested residential providers is
**$1.50–$15/GB**.

**Range to carry forward: residential $1–$8/GB (enterprise $0.50–$2.50/GB); datacenter/ISP priced
per IP, effectively unmetered.**

### The arithmetic that makes this the dominant cost

At 100,000 pages/day:

| Mode | GB/month | Residential @ $4/GB | Residential @ $8/GB | vs a €5.49 Hetzner CX23 |
|---|---|---|---|---|
| HTTP-only, ~50 KB/page | **150 GB** | **$600/mo** | **$1,200/mo** | **~100–200× the server** |
| Headless, 2.3 MB/page | **6,900 GB** | **$27,600/mo** | **$55,200/mo** | **~5,000–10,000×** |

**Two conclusions.** (1) If residential proxies are in the design, **proxy bandwidth is the budget**,
and every other capacity decision is rounding error. (2) The combination of *headless browser* and
*residential proxies* multiplies both taxes — 46× more bytes at $1–8 per byte-GB — and is what turns
a €10/month problem into a five-figure monthly one.

**For this specific workload, both are usually avoidable.** SEC EDGAR *requires* a declared
User-Agent with a contact email; NCBI *requires* tool and email registration to be unblocked; arXiv
explicitly forbids spreading load across machines to evade its limit; Crossref *rewards* identifying
yourself with `mailto` by putting you in the faster polite pool. These sources want you identified,
not hidden. Rotating residential IPs is both unnecessary and, given arXiv's wording, arguably a ToS
violation.

---

## 8. Scaling curves: 10 → 100 → 1,000 → 10,000 sources

| | 10 sources | 100 sources | 1,000 sources | 10,000 sources |
|---|---|---|---|---|
| **Aggregate request budget @1 req/s** | 10 req/s (864k/day) | 100 req/s (8.6 M/day) | 1,000 req/s (86 M/day) | 10,000 req/s (864 M/day) |
| **Worker processes needed** (4–30 pages/s each) | <1 | 4–25, or **one** tuned Scrapy process (`CONCURRENT_REQUESTS=100`) | 35–250 | 350–2,500 |
| **HTTP-only compute** | 1 shared-vCPU VPS | 1 VPS (2–4 vCPU) | a few machines | small cluster |
| **Headless compute** (8–32 pages/box) | 1 box | 3–12 boxes | 30–125 boxes | 300–1,250 boxes |
| **Storage @1k pages/day/src, gzip** | 15 GB/yr | 1.5 TB/yr | 15 TB/yr | 146 TB/yr |
| **Bandwidth (ingress), HTTP-only** | 0.5 GB/day | 5 GB/day | 50 GB/day | 500 GB/day |
| **Bandwidth, headless** | 23 GB/day | 230 GB/day | 2.3 TB/day | 23 TB/day |
| **What newly breaks** | nothing; cron is fine | per-domain rate limiting, retry/backoff, scheduling, **parser breakage becomes a standing cost** | **DNS** (Scrapy names it explicitly); connection pooling; scheduler fairness; per-source config becomes data, not code | not a single-machine problem; this is Common-Crawl/EOT territory — a team, a cluster, and shallow-per-host crawling |
| **What is *not* the problem** | — | CPU, RAM, disk | CPU (still); ephemeral ports (28,232 is far away) | — |

**The 10,000 reality check.** Systems that genuinely touch ~10,000+ distinct sources are Common
Crawl (**33–40 M registered domains per monthly crawl**, on a Hadoop cluster, ~2.1 B pages,
85 TiB/crawl) and the End of Term Web Archive (**2.29 PB** for the 2024 .gov harvest, 1.2 M WARC
files). They achieve breadth by crawling each host **shallowly and infrequently** — not by polling
10,000 sources at 1 req/s. **If your design implies 864 M requests/day, the design is wrong, not
the hardware budget.**

### 8.1 What actually breaks first: human attention

The scaling term that does *not* get cheaper is per-source maintenance. The strongest citable frame
is Google's SRE definition of **toil**:

> "manual, repetitive, automatable, tactical, devoid of enduring value, and that **scales linearly as
> a service grows**." Google SRE aims to keep operational work "**below 50% of each SRE's time**."
> "An ideally managed service should grow by **at least one order of magnitude with zero additional
> work**, other than some one-time efforts to add resources."
> — [Google SRE Book, "Eliminating Toil"](https://sre.google/sre-book/eliminating-toil/)

Applied here: **N bespoke parsers = N independent breakage streams.** Machines scale superlinearly
cheap (one €10/month box absorbs 100 sources of HTTP-only crawling); attention scales strictly
linearly with source count unless the extraction layer is generic. The SRE book's own escape hatch —
absorbing 10× growth with zero extra work — only applies if you engineered for it up front. That is
the argument for declarative/config-driven source definitions, per-source health monitoring with
alerting on *silent* failure (a scraper returning 0 rows is the dangerous case, not one that
crashes), and schema/change detection — not for a bigger server.

Corroborating evidence from the sources above: the sources themselves are the moving parts.
**Crossref changed its rate limits on 2025-12-01. OpenAlex moved to a credit system. Hetzner
repriced twice in 2026. api.data.gov's 1,000 req/hour is per-key and per-agency-negotiable.** Over a
"months, unattended" horizon, the environment changes more than the workload does.

---

## 9. Summary of ranges and disagreements to report honestly

| Question | Honest answer | Spread / disagreement |
|---|---|---|
| RAM per headless page | **300–500 MB** for planning | 100 MB (browserless 2018) ↔ 300–500 MB (2026 secondary) ↔ 1 GB floor / 4 GB recommended (Apify). **3–10×.** |
| Concurrent browsers per core | **1–4** | No authoritative table exists; derived from Apify's 1 core/4 GB + 1 GB floor. |
| HTTP-vs-browser cost multiple | **10–100×** | 10× (Apify CU), 20× (Apify speed), ~40–60× (bandwidth), ~40× (Lambda $/page). Depends on axis. |
| Pages/s per HTTP worker | **4–30** on real sites | Scrapy docs say 50 pages/s — localhost mock server, no parsing. **2–13× optimistic.** |
| Compressed storage/page | **~40 KB** (gzip), **~25 KB** (zstd), **~3 KB** (text only) | Stable across three Common Crawl releases (38.8–42.5 KB). High confidence. |
| gzip ratio on HTML | **4.2–4.8×** | Consistent. zstd adds a further **25–43%** (2017 measurement — conservative). |
| Postgres row-count ceiling | **No such number.** Trigger is table size > server RAM | My derived band is 50–200 M metadata rows on 32–64 GB. Any quoted absolute is a row-width artefact. |
| TimescaleDB compression | **64× documented for timestamps only** | The popular "90–95%" figure is **not on the current docs page**. Do not assume it for text-heavy scrape metadata. |
| Hetzner prices | **Unverified against vendor** — JS-rendered pages | 4 secondary sources agree on June 2026 figures, disagree on pre-April baselines and by ±5% on some plans. |
| OVH VPS-1 | **$4.54 (own page) vs $7.60 (reported post-April-2026)** | Likely promo vs renewal. Check renewal price. |
| AWS egress | **$0.09/GB first 10 TB**, verified June 2026 | Cloudflare's 2021 critique still holds: unchanged in NA/EU since 2018 (~8 years). |
| Residential proxy $/GB | **$1–8 entry, $0.50–2.50 enterprise** | Best table is a competitor-published one; Oxylabs' own page says $2.50 starting vs $6.00 in that table. |
| Serverless crossover | **~1 M pages/month** for HTTP-only | Lambda ~$8.50/M pages ≈ a small VPS at that volume; 100× worse at 100 M pages/month. |

---

## Sources

**Headless browser resources**
- Browserless — Observations running 2 million headless browser sessions (2018-06-04): https://www.browserless.io/blog/observations-running-headless-browser
- Browserless — Open Source Docker Deployment docs: https://docs.browserless.io/enterprise/open-source
- Apify — Usage and resources (memory/CPU per actor, 1 core per 4096 MB): https://docs.apify.com/platform/actors/running/usage-and-resources
- Apify — How to estimate compute unit usage (3,000 vs 300 pages/CU; 1,000-page timings): https://help.apify.com/en/articles/3470975-how-to-estimate-compute-unit-usage-for-your-project
- Playwright — Browser contexts: https://playwright.dev/docs/browser-contexts
- Crawlee — Scaling crawlers: https://crawlee.dev/js/docs/guides/scaling-crawlers
- Crawlee — AutoscaledPoolOptions defaults: https://crawlee.dev/js/api/core/interface/AutoscaledPoolOptions
- crawlex — The headless-browser tax (2026-03-16, *secondary*): https://blog.crawlex.net/blog/headless-browser-tax/

**HTTP throughput and bottlenecks**
- Scrapy — Benchmarking (~3,000 pages/min): https://docs.scrapy.org/en/latest/topics/benchmarking.html
- Scrapy — Broad Crawls (CONCURRENT_REQUESTS=100, REACTOR_THREADPOOL_MAXSIZE=20, DNS): https://docs.scrapy.org/en/2.11/topics/broad-crawls.html
- aiohttp — Client advanced usage (limit=100, limit_per_host=0, ttl_dns_cache=10): https://docs.aiohttp.org/en/stable/client_advanced.html
- httpx — Resource limits (max_connections=100, keepalive=20/5 s): https://www.python-httpx.org/advanced/resource-limits/
- Linux kernel — ip-sysctl (ip_local_port_range 32768–60999, tcp_fin_timeout 60): https://www.kernel.org/doc/html/latest/networking/ip-sysctl.html
- HTTP Archive Web Almanac 2024 — Page Weight (median 2,311 KB mobile / 2,652 KB desktop; 66–71 requests): https://almanac.httparchive.org/en/2024/page-weight

**Politeness / documented rate limits**
- SEC EDGAR webmaster FAQ (10 req/s, User-Agent required): https://www.sec.gov/os/webmaster-faq
- NCBI E-utilities guidelines (3 or 10 req/s; 9 PM–5 AM ET for large jobs): https://www.ncbi.nlm.nih.gov/books/NBK25497/
- arXiv API Terms of Use (1 request / 3 s, single connection): https://info.arxiv.org/help/api/tou.html
- Crossref — Announcing changes to REST API rate limits (effective 2025-12-01): https://www.crossref.org/blog/announcing-changes-to-rest-api-rate-limits/
- api.data.gov developer manual (1,000 req/hour; DEMO_KEY 30/hr, 50/day): https://api.data.gov/docs/developer-manual/
- OpenAlex — Rate limits and authentication (100 req/s, 100k credits/day): https://github.com/ourresearch/openalex-docs/blob/main/how-to-use-the-api/rate-limits-and-authentication.md
- Apache Nutch — nutch-default.xml (fetcher.server.delay = 5.0 s): https://raw.githubusercontent.com/apache/nutch/master/conf/nutch-default.xml
- Heritrix — Politeness parameters (single connection per server): https://github.com/internetarchive/heritrix3/wiki/Politeness-parameters

**Storage**
- Common Crawl — August 2026 crawl archive (2.14 B pages, 360 TiB, 84.78 TiB WARC, 5.84 TiB WET): https://commoncrawl.org/blog/august-2026-crawl-archive-now-available
- Common Crawl — January 2026 crawl archive: https://commoncrawl.org/blog/january-2026-crawl-archive-now-available
- Common Crawl — August 2025 crawl archive: https://commoncrawl.org/blog/august-2025-crawl-archive-now-available
- Common Crawl blog index (Apr–Aug 2026 crawl sizes): https://commoncrawl.org/blog
- Common Crawl statistics: https://commoncrawl.github.io/cc-crawl-statistics/
- Common Crawl mailing list — zstd vs gzip on WARC (2017-03-15): https://groups.google.com/g/common-crawl/c/bO6B6xQJnEE
- IIPC — Zstandard Compression for WARC Files 1.0: https://iipc.github.io/warc-specifications/specifications/warc-zstd/
- Common Crawl — Index to WARC files and URLs in columnar format (~300 GB/crawl, 2018-03-01): https://commoncrawl.org/blog/index-to-warc-files-and-urls-in-columnar-format
- Harvard LIL — Announcing the data.gov archive (16 TB, 311k datasets, 2025-02-06): https://lil.law.harvard.edu/blog/2025/02/06/announcing-data-gov-archive/
- End of Term Web Archive — Data (2.29 PB in 2024; 266 TB in 2020): https://eotarchive.org/data/
- End of Term Web Archive: https://eotarchive.org/

**Databases**
- PostgreSQL — Appendix K, Limits (32 TB table, 1 GB field, 1,600 columns): https://www.postgresql.org/docs/current/limits.html
- PostgreSQL — Table Partitioning ("size should exceed physical memory"; a few thousand partitions): https://www.postgresql.org/docs/current/ddl-partitioning.html
- TigerData/TimescaleDB — Compression methods (64× on timestamps): https://www.tigerdata.com/docs/learn/columnar-storage/compression-methods
- DuckDB — Querying Parquet with precision (21 M rows, ~360 MB, 15–220 ms): https://duckdb.org/2021/06/25/querying-parquet.html

**Pricing**
- Hetzner — New CX plans press release (2024-06-06, specs + 20 TB traffic): https://www.hetzner.com/pressroom/new-cx-plans/
- Hetzner — AX42 dedicated (8c/16t, 64 GB DDR5, unlimited traffic): https://www.hetzner.com/dedicated-rootserver/ax42/
- Northflank — Hetzner cloud server price increases in 2026: https://northflank.com/blog/hetzner-cloud-server-price-increases
- wz-it — Hetzner price increase June 2026 (+176%; DRAM +171% YoY): https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/
- bex.co — Hetzner's second price hike of 2026 (April 1 and June 15 rounds): https://bex.co/blog/2026/08/21/hetzner-second-price-hike-ccx-cpx-113-percent-fleet-economics
- cdnsun — OVHcloud and Hetzner 2026 price increases explained: https://blog.cdnsun.com/ovhcloud-and-hetzner-2026-hosting-price-increases-explained/
- DigitalOcean — Droplet pricing: https://www.digitalocean.com/pricing/droplets
- OVHcloud US — VPS pricing: https://us.ovhcloud.com/vps/
- AWS — S3 pricing ($0.023/GB-mo, $0.005/1k PUT, 128 KB minimum for IA): https://aws.amazon.com/s3/pricing/
- AWS — EC2 on-demand pricing (100 GB free egress): https://aws.amazon.com/ec2/pricing/on-demand/
- EgressCost.com — AWS data transfer out tiers, verified 2026-06 (*secondary*): https://egresscost.com/aws/data-transfer-pricing/
- Cloudflare — AWS's Egregious Egress (2021-07-23): https://blog.cloudflare.com/aws-egregious-egress/
- Cloudflare — R2 pricing ($0.015/GB-mo, zero egress): https://developers.cloudflare.com/r2/pricing/
- Backblaze — B2 pricing ($6.95/TB-mo, 3× free egress): https://www.backblaze.com/cloud-storage/pricing
- AWS — Lambda pricing ($0.20/M requests, $0.0000166667/GB-s): https://aws.amazon.com/lambda/pricing/
- Google Cloud — Cloud Run pricing: https://cloud.google.com/run/pricing
- Zyte — Scrapy Cloud ($9/unit/mo; 1 unit = 1 GB + 1 concurrent crawl): https://www.zyte.com/scrapy-cloud/
- Zyte — Scrapy Cloud units docs: https://docs.zyte.com/scrapy-cloud/usage/units.html
- Oxylabs — Pricing (residential from $2.5/GB; DC $0.7/IP; API $0.25/1k results): https://oxylabs.io/pricing
- Proxidize — Residential proxy pricing comparison (2026-07-26, *vendor-published*): https://proxidize.com/blog/residential-proxy-pricing/

**Operational scaling**
- Google SRE Book — Eliminating Toil (toil scales linearly; <50% cap): https://sre.google/sre-book/eliminating-toil/



---

# Tools, Frameworks & Platforms for Large-Scale Government/Scientific Data Harvesting
## Real options, real trade-offs, real criticisms

**Research date: 2026-09-06.** All version numbers and prices below were verified on this date against
primary sources (PyPI JSON API, GitHub release pages, vendor pricing pages). Practitioner opinion is
drawn mostly from Hacker News (via the Algolia HN API) with usernames and dates attached, so you can
judge staleness yourself.

**No single stack is recommended.** This document deliberately presents the axes of disagreement.

---

## 0. Verification notes and methodology caveats (read this first)

Three methodological warnings, because they materially affect how you should read *any* tooling
comparison you find on this topic:

1. **Most search results for "best web scraping tool 2026" are vendor SEO content.** Searches for
   Scrapy/Crawlee/Playwright comparisons return, in order: Scrapfly, ScrapingBee, Bright Data, Apify,
   Octoparse, ScraperAPI, Oxylabs, Thunderbit, HasData. Every one of these companies sells a competing
   or complementary paid product. The Crawlee-vs-Scrapy comparison at
   <https://crawlee.dev/blog/scrapy-vs-crawlee> is published by Apify, who own Crawlee, and is dated
   **April 23, 2024** — it is both partisan and stale (see point 2). Treat this whole genre as
   marketing, not evidence.

2. **The most widely repeated criticism of Scrapy is now factually out of date.** The standard
   complaint ("it's stuck on Twisted, it's not really async, development has slowed") reflects
   2022–2024 reality. As of 2026 it is wrong — see §1.1. I verified this directly from
   `docs/news.rst` on Scrapy master and from PyPI, not from blog posts.

3. **Beware fetch-tool summarisation error.** Fetching <https://github.com/scrapy/scrapy/releases>
   returned "latest = 2.16.0 (2026-05-19)". The PyPI JSON API for the same package returns
   **2.18.0, uploaded 2026-08-20**, and Scrapy's own `news.rst` on master confirms a
   `Scrapy 2.18.0 (2026-08-20)` section header. The GitHub page render was incomplete. Wherever
   possible below I cite a machine-readable registry (PyPI) over a rendered HTML page. Apply the
   same scepticism to numbers you see quoted elsewhere.

---

## 1. FETCH / PARSE LAYER

### 1.1 Scrapy — actively maintained, and mid-migration off Twisted

**Verified status (PyPI, 2026-09-06):** latest **2.18.0, released 2026-08-20**.
Release history from the PyPI JSON API:

| Version | Release date |
|---|---|
| 2.13.0 | 2025-05-08 |
| 2.13.4 | 2025-11-17 |
| 2.14.0 | 2026-01-05 |
| 2.14.1 | 2026-01-12 |
| 2.14.2 | 2026-03-12 |
| 2.15.0 | 2026-04-09 |
| 2.15.1 | 2026-04-23 |
| 2.15.2 | 2026-04-28 |
| 2.16.0 | 2026-05-19 |
| 2.17.0 | 2026-07-07 |
| 2.18.0 | 2026-08-20 |

Source: `https://pypi.org/pypi/scrapy/json`. That is **11 releases in ~16 months** — a healthy cadence,
not a dying project.

**The Twisted question, from primary sources.** Scrapy's release notes (raw file:
`https://raw.githubusercontent.com/scrapy/scrapy/master/docs/news.rst`) show a deliberate,
multi-release migration away from Twisted internals:

- **2.13.0** added `scrapy.crawler.AsyncCrawlerProcess` and `AsyncCrawlerRunner` as coroutine-native
  counterparts to the classic `CrawlerProcess`/`CrawlerRunner` (news.rst line ~2342).
- **2.14.0 (2026-01-05)** headline: *"More coroutine-based replacements for Deferred-based APIs"*, and
  the default priority queue became `DownloaderAwarePriorityQueue`. Zyte's write-up
  (<https://www.zyte.com/blog/scrapy-in-2026-modern-async-crawling/>) puts it as: *"In 2.14.0, Scrapy
  replaces a huge chunk of these internals with native coroutines."* (Zyte employs core maintainers, so
  this is authoritative on intent but not neutral on enthusiasm.)
- A `HttpxDownloadHandler` exists (`scrapy.core.downloader.handlers._httpx.HttpxDownloadHandler`),
  first appearing in the 2.15/2.16 line, with **HTTP/2 and SOCKS proxy support added in 2.17.0**, and
  in **2.18.0 it moved to `httpx2`**. This matters a lot: it is an escape hatch from Twisted's
  networking stack entirely.
- **2.18.0** also un-flagged the Twisted-based HTTP/2 handler from "experimental", made `brotli` and
  Zstandard support mandatory (so `br`/`zstd` are always advertised in `Accept-Encoding`), and added
  a documented "optimization" page and a built-in stats reference.
- `scrapy.utils.asyncio.sleep` was added and explicitly *"works both with and without a Twisted
  reactor"*; `AsyncCrawlerProcess.start()` now uninstalls the `twisted.internet.reactor` import hook
  on exit.

**Honest reading:** Twisted has *not* been removed and no removal date is announced. You are therefore
in the worst window of a migration — two idioms coexist, `TWISTED_REACTOR` still exists as a setting,
and 2.14.0 introduced a genuine footgun: setting `TWISTED_REACTOR` to a non-asyncio value at spider
level may now require `FORCE_CRAWLER_PROCESS=True` to avoid a reactor-mismatch exception (news.rst,
2.14.0 backward-incompatible changes). That is exactly the kind of papercut that generates the
"Twisted is a liability" complaint.

**Documented criticisms:**
- *Learning curve / architectural complexity.* Apify's comparison: Scrapy's *"complex architecture"*
  makes it challenging for newcomers (<https://crawlee.dev/blog/scrapy-vs-crawlee>, Apr 2024).
- *No native browser rendering.* Requires `scrapy-playwright`; the same (partisan) source notes the
  plugin historically lacked good Windows support.
- *No built-in autoscaling* — you add Scrapyd / Scrapy Cloud / your own supervisor.
- *Churn tax.* The deprecation lists in 2.15.0–2.18.0 are long: `scrapy.FormRequest` deprecated in
  favour of the external `form2request` package (2.16.0); `start_requests()` removed in 2.16.0 after
  deprecation in 2.13.0; the `download_delay` and `max_concurrent_requests` spider attributes
  deprecated in favour of settings (2.18.0); `Spider.log()` deprecated (2.18.0). For an unattended
  multi-year fleet, this is real recurring work — pin versions and budget upgrade time.

**Verdict axis:** Scrapy's genuine strengths for *this* use case are the unglamorous ones — built-in
robots.txt handling, per-domain politeness/`AUTOTHROTTLE`, retry/backoff middleware, request
deduplication, resumable job state (`JOBDIR`), feed exports, and a stats/telemetry surface. Those are
precisely what you re-implement badly by hand across 50 scrapers.

### 1.2 Crawlee (Python and JS) — Apify-backed

**Verified:** `crawlee` on PyPI is **1.10.0 (2026-08-31)**; GitHub shows ~9.4k stars, with releases
roughly every 1–3 weeks through 2026 (1.7.0 May 12 → 1.9.1 Aug 6 → 1.10.0 Aug 31).
Sources: `https://pypi.org/pypi/crawlee/json`, <https://github.com/apify/crawlee-python/releases>.

**Strengths:** unified HTTP + headless-browser interface behind one API (so you can start with HTTP
and escalate a specific site to Playwright without rewriting the crawler), built-in autoscaling
(`AutoscaledPool`), persistent request queue, session/proxy rotation, pluggable storage, and
pre-built Docker images. Parser-agnostic (BeautifulSoup, Parsel, or Playwright).

**Criticisms:**
- **Vendor gravity.** Crawlee is built and maintained by Apify, and its defaults, storage
  abstractions and docs funnel toward the Apify platform. This is the clearest lock-in risk in the
  fetch layer. It is genuinely open source (Apache-2.0) and runs standalone, but the ergonomic path
  is Apify's.
- **Marketing overreach.** The README claims *"Your crawlers will appear almost human-like and fly
  under the radar of modern bot protections even with the default configuration."* For government
  and scientific targets this is irrelevant at best; as a claim about modern bot protection it is
  not credible for hard targets, and it signals a product positioned for adversarial scraping rather
  than for polite, long-horizon harvesting.
- **Younger ecosystem.** Far fewer third-party middlewares than Scrapy, and a much shorter track
  record of multi-year unattended operation. The Python port only reached 1.0 relatively recently.

### 1.3 Browser automation: Playwright vs Puppeteer vs Selenium

**Verified (PyPI, 2026-09-06):** `playwright` **1.62.0 (2026-07-31)**; `selenium` **4.48.0
(2026-08-27)**. Both actively maintained. Puppeteer remains Chrome/Chromium-centric and
Node-only-first.

**Practitioner trajectory — a well-documented migration path.** HN user **michael_j_x**
(2024-12-17, <https://news.ycombinator.com/item?id=42445575>) describes the canonical arc:
`requests` → Scrapy → Selenium → `undetected_chromedriver` → SeleniumBase → **Playwright**, which he
calls *"a much saner interface."* He criticised SeleniumBase's internals as having
*"1000 lines long methods, with 20-30 different flags and branches."* Note the rebuttal in-thread:
the **SeleniumBase maintainer** replied that the code sample cited predated 2020 and was not
representative — an explicit disagreement worth recording, and a reminder that "tool X has bad code"
claims decay fast.

**The critical trade-off for a months-long unattended fleet:** browsers are 10–100× the
CPU/RAM cost per page of an HTTP fetch, and they are the single largest source of flakiness (zombie
processes, memory leaks, driver/browser version skew after auto-updates). On the 100B-pages thread
(<https://news.ycombinator.com/item?id=17497184>), **lyjackal** noted *"headless browser resources are
pretty huge"* at scale, while **waprin** argued the opposite — start with a headless browser despite
overhead because it removes a class of reverse-engineering work. **hermanradtke** sided with parsing
speed for large-scale work. This is an unresolved, legitimate disagreement; the resolution is usually
"HTTP by default, browser as a per-site escalation", which is exactly what Crawlee and
scrapy-playwright are designed to express.

**For government/scientific targets specifically:** a large fraction serve server-rendered HTML,
static files (CSV/XML/JSON/NetCDF/PDF), OAI-PMH, CKAN/Socrata APIs, or bulk FTP/S3 dumps. The
browser tier is often *entirely avoidable* — and every page you can fetch without a browser is
~2 orders of magnitude cheaper to run for months.

### 1.4 The lean stack: httpx / aiohttp + selectolax / lxml / BeautifulSoup

**Verified (PyPI, 2026-09-06):**

| Library | Version | Released | Note |
|---|---|---|---|
| `httpx` | 0.28.1 | **2024-12-06** | last release of the 0.x line — ~21 months stale |
| `httpx2` | 2.12.0 | 2026-08-18 | *"The next generation HTTP client"*, now under Pydantic (`httpx2.pydantic.dev`) |
| `aiohttp` | 3.14.3 | 2026-07-23 | actively maintained |
| `lxml` | 6.1.3 | 2026-09-02 | very actively maintained |
| `selectolax` | 0.4.11 | 2026-07-15 | Modest/lexbor bindings, fastest HTML parsing |
| `beautifulsoup4` | 4.15.0 | 2026-06-07 | maintained |
| `curl_cffi` | ~0.15.x | active | TLS/JA3/HTTP2 impersonation; HTTP/3 since 0.15.0 |

**This is an important and under-reported finding:** **`httpx` 0.x has not had a release since
December 2024**, and the project's continuation is `httpx2` under the Pydantic org — which Scrapy
2.18.0 already adopted for its `HttpxDownloadHandler`. If you were planning to standardise on
`httpx`, you are standardising on a package in the middle of a name/ownership transition. `aiohttp`
is the conservative choice for pure-asyncio HTTP; `lxml` is the conservative choice for parsing;
`selectolax` is the speed choice.

**Criticisms of the lean stack:** you get *nothing* for free — no robots.txt, no per-domain
politeness, no retry/backoff policy, no dedup, no resumability, no stats, no feed export. See §2.

### 1.5 curl-impersonate / curl_cffi

**Verified:** `curl_cffi` — GitHub <https://github.com/lexiforest/curl_cffi>, ~6.4k stars, actively
developed (recent updates for Chrome 150/152 support), HTTP/3 since v0.15.0, min Python 3.10 since
v0.14. Described as *"Python binding for curl-impersonate fork via cffi. A http client that can
impersonate browser tls/ja3/http2 fingerprints."*

Note the ecosystem shape: original `curl-impersonate`, plus `curl_cffi` (Python), `brimp` (lightweight
browser), `impers` (Node), and **`impersonate.pro` (commercial support)**. The maintainer monetises
support, which is a stability signal *and* a hint that the free tier's roadmap follows paying users.

**Criticism / relevance caveat:** this is an *anti-anti-bot* tool. It solves JA3/TLS/HTTP2
fingerprint blocking. Government and scientific sites overwhelmingly do not fingerprint-block
polite, identified, rate-limited clients — and using impersonation against a `.gov` while spoofing a
browser User-Agent is both operationally unnecessary and a compliance and reputational liability.
Keep it in the toolbox for the rare Cloudflare-fronted state portal; do not make it the default.

### 1.6 Colly (Go) — fast, but the release signal is weak

<https://github.com/gocolly/colly> — ~25.5k stars, ~778 commits, 146 open issues, README claims
*">1k request/sec on a single core"*.

**The concern:** the releases page shows **v2.2.0 released 2024-03-27** as the newest tagged release,
after a 9-month gap from v2.1.0 (2023-06-08). That is **~2.5 years without a tagged release** as of
2026-09-06. (I could not confirm master-branch commit recency: `github.com/gocolly/colly/commits/master`
is robots-disallowed to my fetcher, and `proxy.golang.org` was outside this session's egress
allowlist. **Verify this yourself before adopting** — it is the single most important open question
in this section.) Go's module system means you can depend on a pseudo-version from master, but an
un-tagged project is a maintenance risk for a multi-year deployment.

**Strengths if it is alive:** single static binary (trivial deployment, no venv/interpreter drift over
months), excellent goroutine concurrency, low memory footprint, built-in rate limiting and proxy
switching. For a fleet of many small, long-lived, low-complexity harvesters, "one static binary per
scraper under systemd" is an extremely low-maintenance shape.

### 1.7 StormCrawler / Apache Nutch (JVM, genuinely large-scale)

<https://stormcrawler.apache.org/> — **Apache StormCrawler 3.7.0**, an *"open source SDK for building
distributed web crawlers based on Apache Storm"*. Apache-2.0, Java, operational since 2014. Built-in
robots.txt, delays and politeness settings; Apache Tika for document parsing; sitemap and feed
parsers; indexing modules for OpenSearch, Solr, or SQL. Key architectural difference from Nutch: it
processes *"URLs as streams across a whole Apache Storm cluster"* rather than in batches — lower
latency, continuous operation.

**Criticism / fit:** it drags in the entire JVM + Apache Storm operational surface (Nimbus,
supervisors, ZooKeeper). Nutch adds Hadoop batch semantics on top. Both are designed for
**broad, discovery-oriented crawls of the open web at billions-of-URLs scale**. A harvesting system
targeting a *known, enumerable* set of government and scientific endpoints has the opposite shape:
few hosts, deep structure, high per-record parsing specificity, strong politeness constraints. For
that shape, Storm/Hadoop is almost pure overhead — and it commits a small team to JVM cluster ops
for years. Consider these only if you later need genuine open-web discovery.

### 1.8 Katana — commonly mis-listed; it is a security tool

<https://github.com/projectdiscovery/katana> — ~17.4k stars, Go, active (~1,727 commits on `dev`).
It is *"a next-generation crawling and spidering framework"* built for **security reconnaissance**:
endpoint discovery, JS parsing for hidden endpoints, automatic form filling, secrets extraction,
technology detection, scope control. Requires Go 1.26+.

**This is not a data-harvesting tool** and appears in "best crawler" listicles largely through
keyword overlap. Its output is a URL/endpoint inventory, not structured records. It *could* be useful
once, as a reconnaissance step to enumerate the URL surface of an unfamiliar agency portal — but note
that pointing an automatic-form-filling, secrets-extracting pentest crawler at a government host is
a decision with legal implications well beyond scraping. Do not put it in the production path.

### 1.9 Scrapling — the "adaptive selector" pitch

<https://github.com/D4Vinci/Scrapling> — **0.4.15 (PyPI, 2026-08-23)**, ~69.1k stars, BSD-3-Clause,
Python 3.10+, 48 releases, claims 92% test coverage. Also covered by ScrapingBee
(<https://www.scrapingbee.com/blog/scrapling-adaptive-python-web-scraping/>).

The headline feature is directly relevant to a multi-year unattended fleet: *"smart element
tracking"* that will *"relocate elements after website changes using intelligent similarity
algorithms"* when you pass `adaptive=True`.

**Criticism:** the star count (69.1k) is wildly out of proportion to the project's age and to
comparable tools (Scrapy is far more depended-upon), which usually indicates a viral-launch
popularity spike rather than production adoption depth — treat stars as a marketing metric here, not
an adoption signal. More substantively, **adaptive selectors trade a loud failure for a quiet one**:
a broken selector raises an error you can alert on; a selector that silently "relocates" to a
*similar-looking but wrong* element produces plausible bad data. That is the single worst failure
mode for a research dataset. If you use it, you must pair it with hard output validation (see §9).

---

## 2. THE "FRAMEWORK VS PLAIN SCRIPTS" DEBATE

This argument is old, unresolved, and both sides are represented by people who have shipped. Direct
quotes with attribution:

**"Frameworks are overkill":**
- **krakensden** (HN, 2011-04-20): *"Scrapy is overkill for nearly everything. You'll probably have
  under a page of code using lxml and urllib."*
- **krakensden** again (2011-04-21) refined it usefully: the choice depends on *whether the scraper
  runs regularly and needs robustness, versus being a one-off utility*. This is the actual crux.
- **datalopers** (HN, 2022): *"stop adding in layers of Airflow/Dagster/Prefect when a simple cron
  would work"* — the same instinct applied one layer up.
- On the 100B-pages thread: *"Starting with a single threaded model allowed my team to scale quickly
  and with little additional overhead"*, and *"Start with single threaded and simple, move to
  multi-threaded scrapers when the juice is worth the squeeze."*

**"Hand-rolled reinvents it badly":**
- **cdr** (HN, 2011-04-20) rebutted krakensden directly: Scrapy lets you use simple *or* advanced
  features as needed, and dismissing it is like arguing *"jQuery is overkill for just about
  everything, you should use plain javascript."*
- **tekkk** (HN, 2017-10-24) is the most valuable data point because he changed his mind *with
  experience*: he initially thought Scrapy excessive, then found simple scripts *"failed miserably"*
  on larger projects requiring cookie handling and gentle scraping limits. His conclusion: Scrapy
  *"gives a lot more control on how gently you scrape and outputs nicely json/jl/csv with the data
  you want."*
- **PaulHoule** (HN, Mar 2025) offers the cynic's synthesis that applies to *every* layer of this
  stack: all of these tools hit endpoint limitations; the only difference is *when*.

**Honest synthesis for a many-scrapers, months-unattended system.** The "plain scripts" position is
strongest when N is small and lifetime is short — the inverse of the stated requirements. The
specific things a framework gives you that hand-rolled code reliably gets wrong on the 30th scraper
are: per-domain concurrency and politeness, robots.txt, retry/backoff with jitter, request dedup,
resumability after a crash mid-run, and uniform stats/logging. Note the middle path that several
practitioners land on and that nobody markets: **one thin shared internal library** (fetch + retry +
politeness + raw-response archiving + validation + emit) that all N scrapers import, with per-site
code kept to selectors and pagination. That gets you framework benefits without framework churn — at
the cost of owning it yourself forever.

---

## 3. LLM-BASED EXTRACTION

### 3.1 Verified cost data

The most concrete published cost breakdown I found is <https://webscraping.ai/blog/llm-web-scraping>
(**August 2026**; note the publisher sells a scraping API, so read the framing critically — but the
token arithmetic is checkable). For **1,000 pages** of *cleaned* text (~5,000 input + 200 output
tokens each):

| Model | Cost per 1,000 pages |
|---|---|
| Gemini 2.5 Flash Lite | **$0.58** |
| DeepSeek V4 Flash | $0.76 |
| Claude Haiku 4.5 | $6.00 |
| GPT-5.6-sol (flagship) | $31.00 |

**The dominant variable is not model choice — it is HTML cleaning.** Raw HTML runs 10–50× the token
count of extracted text; the same source estimates **~$306 per 1,000 pages** feeding raw HTML to a
flagship model versus **$0.58** feeding cleaned text to a value model. A ~500× spread. Typical cleaned
page: 2,000–8,000 tokens.

**Scale implication:** at 1M pages/month, cleaned-text extraction on a value model is ~$580/month —
affordable. The same volume on raw HTML through a flagship model is ~$306,000/month — not.
Any LLM-extraction plan without an aggressive HTML-reduction step in front of it is financially
unserious.

### 3.2 The honest critiques

From the same source, the failure modes are stated plainly:

1. **Silent errors** — *"Models return plausible but incorrect values, evading detection for weeks."*
   This is categorically worse than a crashed parser for a research dataset.
2. **Invisible markup data** — classes, attributes and images that encode ratings, status codes or
   units are unreadable to text-based extraction.
3. **Incomplete JSON** — plain JSON mode guarantees *syntax*, not *field completeness*; strict schema
   mode is required.
4. **No network layer** — *"The model never touches the network layer."* LLMs do nothing about
   blocking, rate limits, robots.txt or pagination. This is architectural and unfixable by prompting.

Recommended defences (also from that source): make nulls explicit and mandatory in the schema,
validate in **code** rather than in prompts, and spot-check production runs.

**HN sentiment is sparse and mostly tangential** — the Algolia comment corpus does not contain a
rich LLM-scraping debate, which is itself informative (this is a vendor-driven conversation more than
a practitioner one). What exists: **medivhX** (2026-01-16) promotes Crawl4AI as solving
*"LLM-scraping headaches"*; **l33t-mt** (2026-01-02) built browser-based
*"Web scraping → LLM processing → webhook pipelines"* specifically to avoid server-side data exposure.
I found **no** substantive HN thread arguing the "use the LLM to write the selector once, not to
extract every page" position, despite it being the position most experienced engineers hold. Flag
this as a genuine gap in available evidence rather than as consensus either way.

### 3.3 Products in this space

| Product | What it is | Verified pricing (2026-09-06) | Criticisms |
|---|---|---|---|
| **Firecrawl** | Crawl/scrape → LLM-ready markdown | Free 1k credits; Hobby **$16**/5k; Standard **$83**/100k; Growth **$333**/500k; Scale **$599**/1M. 1 credit ≈ 1 page. Annual discount 16.7–20%. <https://www.firecrawl.dev/pricing> | See below — cost, ToS, and crawler-ethics complaints |
| **Jina Reader** (`r.jina.ai`) | URL → clean markdown; PDF, image captioning | Free no-key: **20 RPM**. Free key: **500 RPM**, 10M free tokens. Paid: 500 RPM / 2M TPM. Premium: 5,000 RPM / 50M TPM. <https://jina.ai/reader/> | Explicitly *"does not actively circumvent... anti-bot systems"*; higher tiers raise rate limits only, they do **not** unblock sites |
| **ScrapeGraphAI** | LLM + graph logic extraction pipelines | OSS (MIT), ~29.5k stars, ~2,891 commits. Managed cloud API with credit pricing. <https://github.com/ScrapeGraphAI/Scrapegraph-ai> | **Its own README says it is *"meant to be used for data exploration and research purposes only"*** — read that as the maintainers declining to claim production-readiness. Collects anonymous telemetry by default (opt-out available). OSS version leaves you to manage browsers/proxies/scaling |
| **Diffbot** | ML extraction + Knowledge Graph | Free 10k credits @ 5 req/min; Startup **$299**/250k credits (overage $0.001/credit, 5 req/s); Plus **$899**/1M credits (overage $0.0009, 25 req/s). 1 page = 1 credit; KG entity export = 25 credits. <https://www.diffbot.com/pricing/> | Oldest player, most opaque model; rate limits are low at the cheap tiers |

**Firecrawl — the documented complaints (all from HN, with dates):**
- **Cost.** **machinelinux** (2026-03-04): *"Firecrawl charges ~$100/mo, Browserless ~$200/mo.
  Kryfto runs on a $5/mo VPS with zero per-request costs."* (Note: he is pitching his own product —
  discount accordingly, but the order-of-magnitude point about self-hosting stands.)
- **Terms of service / data rights.** **lukewarm707** (2026-04-29) quotes Firecrawl's terms as
  granting a *"worldwide, irrevocable, non-exclusive, royalty-free license to use, reproduce, modify,
  publish, translate and distribute any content."* **For a research or public-sector dataset, read
  the ToS before routing content through any managed extractor.** This is the kind of clause that
  creates downstream licensing problems you cannot unwind.
- **Crawler ethics.** **oasisbob** (2026-03-11): *"On one hand, their bots seem much more
  well behaved than others. However, running a crawler fleet which is deceptive and evasive in its
  identification and don't honor REP is no way to build a business."* Same user (2026-03-29) reports
  Firecrawl clients using a *"deceptive UserAgent"* while operating through residential proxies.
  If you are harvesting `.gov`/`.edu` and care about not being blocked (or about your institution's
  name), a vendor with that reputation is a liability you inherit.
- **Crowded field.** Multiple commenters list Firecrawl alongside Exa, Tavily, Parallel, BrowserUse,
  Browserbase, Context.dev — high churn risk. **TheYahiaBakour** (Context.dev founder, 2026-07-10):
  *"Firecrawl doesn't have a search index... Firecrawl is a wonderful company, it's fun to compete
  with them."*

**Verified library version:** `firecrawl-py` **4.41.0 (2026-08-31)** — a major-version-4 SDK moving
fast, which for an unattended multi-year system means pinning and periodic breakage.

---

## 4. ORCHESTRATION LAYER

### 4.1 Verified current versions (PyPI / official docs, 2026-09-06)

| Tool | Version | Released |
|---|---|---|
| `apache-airflow` | **3.3.1** | 2026-08-12 |
| `dagster` | **1.13.21** | 2026-09-03 |
| `prefect` | **3.8.5** | 2026-09-03 |
| `temporalio` (Python SDK) | **1.32.0** | 2026-08-24 |
| Nomad | **v2.0.4** | 2026-07-07 |

Airflow 3.x release line (<https://airflow.apache.org/docs/apache-airflow/stable/release_notes.html>):
3.0.0 (2025-04-22) → 3.1.0 (2025-09-25) → 3.1.8 (2026-03-11) → 3.2.0 (2026-04-07) →
3.2.1 (2026-04-21) → 3.2.2 (2026-05-29) → 3.3.0 (2026-07-06) → 3.3.1 (2026-08-12).

### 4.2 The paradigm question — the most important comparison for this use case

There are three genuinely different models, and the practitioner threads distinguish them clearly.

**(a) DAG orchestrators** (Airflow, Dagster, Prefect) — built for *scheduled batch analytics*: a
bounded graph of tasks over a time partition, with data lineage as a first-class concern.

**(b) Task queues** (Celery, RQ, Dramatiq, arq, Huey; on Redis/RabbitMQ/SQS/NATS) — built for *many
small independent jobs* with retries and horizontal workers, no graph.

**(c) Durable execution** (Temporal, Restate, Windmill) — built for *long-lived, stateful,
resumable workflows* where the engine persists execution history so a process can crash and resume
mid-flight.

**Attributed arguments (HN thread <https://news.ycombinator.com/item?id=39211186>, Feb 2024):**
- **jtmarmon** (2024-02-01): *"Temporal is entirely focused on the orchestration piece, and the
  others are much more focused on the data piece."* He values Temporal's multi-language SDKs and
  queueing for heterogeneous workers — e.g. *"a 10 part workflow where two parts run on a python
  service running with a GPU."*
- **jaydeegee** (2024-02-01): *"Cadence/Temporal are focused on code orchestration side rather than
  data orchestration."*
- **nerdponx** (2024-02-01) attacks the framing: Airflow is not really data-oriented, it is
  fundamentally **"Cron + Make"**, with data features secondary; its only genuinely
  data-specific concept is logical time intervals, which you can ignore entirely.
- **wharvle** (2024-02-02) is blunter: Airflow imposes unnecessary structure while being bad at
  passing data between tasks — *"half-baked"* despite its prevalence.

**Why this matters here.** A harvesting fleet is shape (b) with a dash of (c): N mostly-independent
per-source jobs, each of which may run long, page through thousands of records, and need to resume.
It is *not* a single dependency graph over a daily partition. The most-cited criticism of Airflow —
that it is a scheduler with a metadata DB bolted to a batch model — is a **structural** mismatch,
not a taste objection. **nerdponx**'s "Cron + Make" reduction is the useful lens: if that's all you
need, cron/systemd plus a queue may genuinely dominate.

### 4.3 Airflow — documented complaints

From <https://news.ycombinator.com/item?id=31480320> (Shopify's "Lessons learned running Airflow at
scale", 2022) and follow-on threads:

- **Scheduling overhead:** *"5-15 minutes of overhead between task scheduling"* pre-2.0.
- **Upgrades:** *"Upgrades have been an absolute nightmare and so disruptive... we are just done with
  it. We'll stay at 2.0 until we eventually move off airflow altogether."* Shopify themselves
  *"tried multiple times to upgrade past the 2.0 release and hit issues every time."*
- **Failure modes:** *"scaling issues at the k8s level, scheduling overhead in airflow, random race
  conditions deep in the airflow code."*
- **UI collapse at fan-out:** an operator running ~1,000 tasks in one DAG: *"the UI is completely
  useless, tree view has insane duplication, graph view is super slow."* **This is directly relevant
  — a fleet of many scrapers is exactly a high-fan-out DAG.**
- **Staffing cost:** *"You almost need a dedicated Airflow expert to handle minor version upgrades or
  figure out why a task isn't running when you think it should."*
- **niwtsol** (Mar 2025): *"it's just overly complex for what's really a pretty simple data flow."*
- **mushufasa** (2025): *"Airflow is a really dated choice for a greenfield project."*

**The strongest defence of Airflow, and it is a real argument** — **disgruntledphd2** (Mar 2025):
*"it sucks in well known and predictable ways."* For a system that must run unattended for months,
a tool whose failure modes are extensively documented, Stack-Overflowed and hireable-for has genuine
value over a more elegant tool with a thin community. Do not dismiss this.

**Airflow 3.x specifics.** 3.0 (2025-04-22) brought DAG Versioning (AIP-66), a Task Execution API and
Task SDK (AIP-72) that **removes worker tasks' direct access to the metadata DB**, asset-based
scheduling (AIP-74/75), an Edge Executor (AIP-69), a full React UI rewrite, REST API v2 replacing v1
entirely, a split CLI (AIP-81), and removal of SequentialExecutor and plugin-based executor
registration.

Criticisms of 3.0 (<https://danubedatalabs.com/apache-airflow-3-0-new-features-what-hurts-and-should-you-upgrade/>,
undated but 2025 — **flagged as stale**, given 3.3.1 shipped 2026-08-12): early-release bugs;
providers/operators lagging; *no* large-scale published migration case studies; and fragmented
managed support at the time (Astronomer day-zero, Cloud Composer 3 in preview, **AWS MWAA still on
2.x**, Azure none). Breaking change with teeth: tasks *"no longer have direct access to the metadata
database"* — which breaks a very common Airflow anti-pattern many teams rely on. **The managed-support
picture is the item most likely to have changed since; verify MWAA's current version before assuming
you can use it.**

Self-hosting requirements (<https://stribog.com/blog/self-hosted-workflow-orchestration-temporal-airflow-sovereign-scheduling>,
**2026-08-26**): multiple schedulers coordinating via DB row-level locks, an API server, workers, and
an **external** Postgres 13–17 or MySQL 8.0/8.4 (the Helm chart's bundled Postgres is *explicitly
discouraged* for production). Two operational landmines named: the **metadata database becomes a
bottleneck under load**, and **losing the Fernet key renders stored credentials unrecoverable**
(restore needs both a DB dump and the Fernet key).

### 4.4 Dagster

**Strengths:** asset-oriented rather than task-oriented — you declare the datasets you want to exist
and their dependencies. **maliciouspickle** (HN, 2026): *"geared for hooking into the data itself...
tracks data over time with nice UI."* For a harvesting system whose deliverable *is* a versioned
dataset per source, the asset model maps more naturally than Airflow's task model. Strong local
testing story and better Kubernetes integration than Airflow (per the 2022 Shopify thread).
**recursive4** (Mar 2025) notes the team is *"actively reducing the learning curve with each release."*

**Criticisms:**
- **vitorbaptistaa** (Mar 2025): *"more complex and more powerful"* than Luigi — the complexity is
  acknowledged even by people recommending it.
- **abelanger** (2025): *"more oriented towards data engineering"* — specialised rather than general.
- **Cost model risk.** Dagster+ (<https://dagster.io/pricing>): Solo **$10/mo** at
  **$0.040/credit**; Starter **$100/mo** at **$0.035/credit**; serverless compute **$0.010/minute**;
  Pro is contact-sales. Crucially: *"A credit is defined as the sum of asset materializations and ops
  executed"* — each asset materialisation is 1 credit, each executed op is 1 credit.
  **This pricing is actively hostile to a many-small-assets harvesting workload.** If you model each
  scraped source-partition as an asset and run them frequently, credits scale with your
  *fan-out × frequency*, not with your data volume or compute. Do the arithmetic before committing:
  1,000 assets materialised hourly = 720,000 credits/month ≈ **$25,200/month** at $0.035. Self-hosted
  Dagster OSS avoids this entirely — but then you own the ops.
- Tier limits bite early: Starter is 3 users / 5 code locations / 1 deployment.

### 4.5 Prefect — and the churn complaint

**Verified:** `prefect` **3.8.5 (2026-09-03)**. Prefect 3.0 went GA September 2024
(<https://www.prefect.io/blog/prefect-3-generally-available-september-3>).

**Strengths:** the lowest-friction authoring model of the DAG tools. **cicdw** (a Prefect employee,
Mar 2025) describes it as *"minimally invasive"*, needing *"only two lines to convert a script to a
flow"* — worth noting the affiliation. The 2022 Shopify thread praised *"sane deployment options, not
the insane file-based polling that airflow does"*, and called 2.0 *"a first-principles rewrite that
removes most friction."*

**Criticisms:**
- **The churn is the criticism.** Prefect 1 → 2 was a *"first-principles rewrite"*; 2 → 3 was another
  major break, with community threads and a dedicated migration guide
  (<https://docs.prefect.io/v3/how-to-guides/migrate/upgrade-to-prefect-3>) plus a long-running GitHub
  migration-guide issue (<https://github.com/PrefectHQ/prefect/issues/13979>) and typing regressions
  such as <https://github.com/PrefectHQ/prefect/issues/15275> (`my_flow.deploy()` mypy breakage on
  2.2→3). **Two rewrites in ~4 years is the central risk for a system meant to run untouched for
  months.**
- **aradox66** (Mar 2025): *"concurrency code needs to get somewhat invasive"* — the "just two lines"
  claim degrades once you actually need parallelism, which a scraping fleet does immediately.
- **Pricing shape** (<https://www.prefect.io/pricing>): Hobby free (2 users, 1 workspace, **5
  deployments**, 500 serverless min/mo); Starter **$100/mo** (3 users, **20 deployments**, 75 h
  serverless); Team **$100 per user/mo** (4–8 users, **100 deployments**, 225 h); Enterprise custom.
  Prefect markets *"pricing based on seats and workspaces, not usage"* — genuinely attractive versus
  Dagster's per-materialisation credits. **But the deployment caps are the catch for this use case:**
  if each of your ~200 sources is a deployment, Starter's 20 and Team's 100 are hard walls pushing you
  to Enterprise. Design around a few parameterised deployments, not one per source.

### 4.6 Temporal and durable execution

**Advocates:**
- **MrSaints** (2020-08-19) chose Temporal after evaluating Conductor, Zeebe and Cadence, for
  handling complex online workflows without DSL constraints.
- **mfateev** (Temporal's creator, 2021-01-02) argues imperative code beats declarative DSLs, with
  fault-tolerant state preservation.
- **emmanueloga_** (2026-01-12): durable-execution frameworks *reduce* boilerplate versus
  hand-rolled queue patterns that require implementing transactional-outbox yourself — with latency
  trade-offs.
- **bazizbaziz** (2025-03-22): workflows are *"table stakes"* for serious services, though he
  criticises the authoring experience.

**Critics — and these are specific and recent:**
- **graerg** (2026-05-30) is the most operationally concrete: self-hosting needs **4+ services**
  (history, matching, frontend, UI), the Helm charts introduce *"substantial ops burden"*, and he
  reports **occasional upgrade failures with data loss**.
- **BowBun** (2026-01-28): *"an AMAZING piece of software"* but excessive for simple tasks; being
  forced to use it at work creates *"a lot of boilerplate"* for small background jobs. Same user
  (2025-12-10) argues Temporal and Celery should **coexist** rather than compete.
- **pm90** (2025-04-22): the abstraction holds *"for as long as things go smoothly. When a thing
  breaks then ur hosed"*, citing self-hosting difficulty.
- **prosunpraiser** (2024-02-13): non-trivial state management, determinism constraints, and shared
  infrastructure creating a **single point of failure**.

**Hard technical limits** (<https://stribog.com/blog/self-hosted-workflow-orchestration-temporal-airflow-sovereign-scheduling>,
2026-08-26):
- **Event History warns at 10 MB and errors at 50 MB.** For a scraper that paginates through
  100,000 records inside one workflow, this is a *design constraint you will hit* — you must
  batch into child workflows or use continue-as-new.
- **Editing workflow definitions in flight causes non-determinism errors** — so deploying a fixed
  selector while long-running harvest workflows are live requires versioning discipline.
- **Search Attribute values are unencrypted:** *"the Server must read these values in plain text to
  support filtering."* Default attributes (WorkflowId, TaskQueue) leak execution metadata regardless
  of payload encryption.
- **History Shard count is immutable after initial configuration** — a capacity decision you make
  once, before you know your load.
- Persistence: Postgres/MySQL/Cassandra, plus a visibility store (Postgres 12+/MySQL 8.0.17+, or
  Elasticsearch recommended at high volume).

**Temporal Cloud pricing** (<https://temporal.io/pricing>): Essentials from **$100/mo** (1M actions,
1 GB active / 40 GB retained storage); Business from **$500/mo** (2.5M actions, 2.5 GB / 100 GB);
Enterprise custom (10M actions, 10 GB / 400 GB). Overage tiers: next 5M at **$50/M**, next 5M at
$45/M, next 10M at $40/M, next 30M at $35/M, next 50M at $30/M, next 100M at $25/M. Storage:
**active $0.042/GB-hour**, retained **$0.00105/GB-hour**. New users get **$1,000 in credits**.

**The pricing trap to note:** monthly fees are *"the greater of your monthly plan or 5%-10% of usage
as you scale"* — a percentage-of-usage support fee layered on top of consumption. And **active
storage at $0.042/GB-hour is ~$30/GB-month**, which is very expensive; keep scraped payloads *out*
of workflow history and pass references to object storage instead. An "action" is roughly every
workflow/activity state transition, so a chatty per-record workflow burns actions fast.

### 4.7 Restate, Windmill, Kestra, Argo

From <https://www.pkgpulse.com/guides/temporal-vs-restate-vs-windmill-durable-workflow-2026>
(**March 2026**):

| | Temporal | Restate | Windmill |
|---|---|---|---|
| Model | Event sourcing; workflows + activities | Journaling; services + virtual objects | Scripts as workflows + visual builder |
| State store | Postgres / MySQL / Cassandra (+ optional Elasticsearch) | RocksDB, **single binary, no external DB** | Postgres |
| Ops weight | Highest | Lowest | Medium (reuses existing Postgres) |
| Licence | **MIT** | **BSL** (Business Source License) | **AGPLv3** |
| Main criticism | Complexity; determinism constraints (no `Date.now()`, `Math.random()`, direct network calls in workflow code) | **Observability tooling less mature**; dashboard offers less introspection | Less suited than Temporal for enterprise mission-critical complex patterns |

**Licence matters for lock-in assessment:** Temporal is MIT (most permissive). **Restate is BSL** —
source-available, not OSI open source, with use restrictions; a real consideration if this system
sits in a public-sector or institutional context with open-source procurement rules. **Windmill is
AGPLv3** — fine to self-host, but a copyleft consideration if you ever redistribute a derived
service.

**Restate's single-binary, no-external-DB deployment is the most interesting property here for a
small team** — it removes the entire "operate Cassandra/Elasticsearch" burden that makes self-hosted
Temporal heavy. Weigh that against the immature-observability criticism and the BSL.

**Argo Workflows** — Kubernetes-native, and opinion is sharply split:
- **verdverm** (2026-03-21): *"a top choice for self hosted"* CI/CD given vendor pricing; but the
  *same user* (2025-11-12): *"Argo Workflows does not live up to what they advertise, it is so much
  more complex to setup and then build workflows for."* One person, both views, four months apart —
  a useful signal that it rewards investment but punishes casual adoption. He also hit Helm/Argo
  template-delimiter conflicts (2025-12-16).
- **odie5533** (2025-11-29) names the disqualifying issue for this use case: **poor startup speed
  makes it unsuitable as a Celery replacement; only viable for long-running workflows.** A harvesting
  fleet of many short jobs is precisely the wrong shape.
- **sleepybrett** (2026-08-06) reports 9 months of successful self-service automation use;
  **spacemule** (2026-06-18) runs it on a small k3s cluster.
- Everything about it presumes Kubernetes — see §7.

**Kestra** — YAML-declarative, JVM-based, positions itself against Windmill
(<https://kestra.io/vs/windmill>) and Temporal (<https://kestra.io/resources/infrastructure/temporal-alternatives>).
**Caution: nearly all comparison content for Kestra is published by Kestra**, and I found no
independent HN practitioner reports. Treat as unvalidated by this research.

### 4.8 Task queues — Celery, RQ, Dramatiq, and friends

From <https://judoscale.com/blog/choose-python-task-queue> (Jeff Morhous; **no publication date given
— flagged as undated**, and Judoscale sells autoscaling for queues):

| | Celery | RQ | Dramatiq / Huey |
|---|---|---|---|
| Strength | Feature-rich, built-in scheduling, flexible brokers, cross-language, better at scale | Simplicity, minimal API, no service beyond Redis | Between the two; Dramatiq stresses *"simplicity, reliability, performance"* |
| Weakness | *"with this power comes a learning curve and setup overhead"*; needs a separate broker (e.g. RabbitMQ) | Less feature-rich, Python-only, **51 s vs Celery's 12 s** in one cited benchmark; weaker reliability guarantees than Celery+RabbitMQ | Smaller communities |

The author's own recommendation is refreshingly unglamorous: *"pick RQ until it didn't work for me
anymore, then migrate to Celery."*

**Where queues win for this use case:** they are the native shape for "many small independent jobs".
Politeness is naturally expressed as **one queue per domain with a bounded worker count** — which is
how the practitioners actually do it. **mnmkng** (Crawlee/Apify, 2022-08-23): *"Create new queues
with one line of code and name them with hostname"* for per-domain rate limiting.

**Message-broker choices, with the actual advice:** **danudey** (2021-12-16) gives the pragmatic rule
that generalises well — you probably want *a* queue, but *"it can just be Redis instead of Kafka"*,
and the broker should be an implementation detail behind an interface, chosen by need rather than
architectural dogma. For a harvesting fleet: Redis is the default (simple, you likely already run it,
good enough); SQS if you want zero ops and are already on AWS (and accept the lock-in); RabbitMQ if
you need strong per-message delivery guarantees; **Kafka is almost certainly wrong** — it is a
partitioned log optimised for high-throughput streaming, not a job queue with per-domain concurrency
limits and long-tail retries, and it adds major ops weight. NATS sits between Redis and Kafka and is
operationally light, but has the smallest practitioner corpus of the four.

### 4.9 Cron / systemd timers — the baseline nobody markets

There is no vendor publishing "why cron is enough", which is exactly why it is under-represented in
search results. The practitioner support is nonetheless real:
- **rasmusab** (Mar 2025): plain Python scripts plus `make` for parallelisation, avoiding framework
  complexity entirely.
- Multiple users in the same thread advocate **Makefiles** as lightweight DAGs; **fforflo** suggests
  `make2graph` for visualisation.
- **vitorbaptistaa** (Mar 2025) recommends **Luigi** after 4+ years as *"the simplest to reason
  about"*, while conceding it *"lacks monitoring and advanced web ui"*. **Important caveat from the
  2022 thread: Luigi is described as *"no longer actively maintained"*** — do not start new work on it.
- **datalopers** (2022): *"stop adding in layers of Airflow/Dagster/Prefect when a simple cron would
  work."*

**systemd timers specifically** beat cron for this workload on several axes worth naming: per-unit
logging via `journalctl`, `Restart=`/`RestartSec=` policies, `RandomizedDelaySec` (jitter, which
matters for politeness across many sources), `Persistent=true` (catch-up after downtime — important
for months-long unattended operation), resource limits via cgroups, and dependency ordering.
The honest limits: no cross-machine scheduling, no built-in retry-with-backoff semantics beyond
restart, no DAG, no UI, and observability is whatever you build.

---

## 5. MANAGED SCRAPING PLATFORMS — verified pricing (2026-09-06)

All figures fetched from the vendors' own pricing pages on 2026-09-06. **Vendor pricing pages are
frequently geo-targeted, A/B tested, and JS-rendered; re-verify before budgeting.** Bright Data's
numbers in particular were showing a "50% off" promotion at fetch time.

| Platform | Published price | Notes |
|---|---|---|
| **Zyte API** <https://www.zyte.com/pricing/> | Pay-as-you-go: **HTTP $0.13–$1.27 / 1k requests**; **browser-rendered $1.01–$16.08 / 1k**. With $100/mo commit: HTTP $0.10–$0.95, browser $0.75–$12.00. $200/mo: HTTP $0.08–$0.76, browser $0.60–$9.60. $500/mo: HTTP $0.06–$0.61, browser $0.48–$7.68 | Five complexity tiers (Simple→Advanced) drive the range. *"No overage penalties"*. $5 free credit |
| **Zyte Scrapy Cloud** <https://www.zyte.com/scrapy-cloud/> | Free Starter (1 h crawl limit, 1 concurrent crawl, 7-day retention); Professional from **$9 per unit/month** (1 unit = 1 GB RAM + 1 concurrent crawl), unlimited crawl time, 120-day retention | Still an active product; no deprecation notice found |
| **Apify** <https://apify.com/pricing> | Free $0 ($5 credit); Starter **$19/mo**; Scale **$199/mo**; Business **$999/mo**. Compute units **$0.2** (Free/Starter) → $0.16 (Scale) → $0.13 (Business). Residential proxies **$8→$7/GB**; datacenter from **$1/IP** → $0.6/IP | Prepaid credit + PAYG overage |
| **Bright Data** <https://brightdata.com/pricing> | Residential **$2.5/GB** (listed as 50% off $5); datacenter **$0.9/IP**; **Web Unlocker from $1/1k req**; Scraping Browser **$5/GB**; Web Scraper APIs **$0.75/1k records** (from $1) | Promotional pricing at fetch time — verify |
| **Oxylabs Web Scraper API** <https://oxylabs.io/products/scraper-api/web> | Micro **$49/mo** (98k results, from $0.50/1k); Starter **$99/mo** (220k, from $0.45/1k); Advanced **$249/mo** (622.5k, from $0.40/1k). By target: Amazon $0.40–0.50/1k; **Google $0.80–1.00/1k**; other sites **$0.95–1.15/1k**; **JS-rendered $1.25–1.35/1k** | Free trial 2k results |
| **ScrapingBee** <https://www.scrapingbee.com/pricing/> | Hobby **$19**/75k credits/25 concurrent; Freelance **$49**/250k/50; Startup **$99**/1M/100; Business **$249**/3M/200; Business+ **$599**/8M/400 | Free 1k credits, no card. Per-request-type credit multipliers **not published on the pricing page** — ask before committing |
| **ScraperAPI** <https://www.scraperapi.com/pricing/> | Hobby **$49**/100k credits/20 threads; Startup **$149**/1M/50; Business **$299**/3M/100; Scaling **$475**/5M/200; Professional **$975**/10.5M/300; Advanced **$1,975**/21.5M/500 | 1 credit = standard page. Amazon 5; **Google/Bing 25**; LinkedIn 30; **bot-protected (Cloudflare/DataDome/PerimeterX) +10 per request** |
| **Firecrawl** | See §3.3 | LLM-markdown oriented |
| **Diffbot** | See §3.3 | ML extraction + Knowledge Graph |

### 5.1 When buying beats building

Buying wins when the *unblocking* problem dominates: adversarial targets, aggressive bot protection,
residential IP requirements, CAPTCHA. Building wins when the *parsing and curation* problem dominates
and targets are cooperative.

**This distinction is decisive for government/scientific harvesting, and it points away from the
expensive tier.** Note how the price structures above are built almost entirely around
anti-bot difficulty: Zyte's 5 complexity tiers spanning a ~10× range, ScraperAPI's
"+10 credits for Cloudflare/DataDome/PerimeterX", Oxylabs charging 2.5–3× more for JS-rendered pages,
Bright Data's Web Unlocker as a separate product. **You are being asked to pay for a problem most
`.gov`, `.edu`, and research-infrastructure endpoints do not pose.** Many such sources additionally
publish bulk downloads, OAI-PMH endpoints, CKAN/Socrata APIs, or S3/FTP dumps — where the correct
"tool" is `curl` plus a manifest differ, and any per-request pricing is pure waste.

Where a managed platform still earns its keep here: (a) the handful of state/municipal portals behind
Cloudflare; (b) geo-restricted sources needing egress from a specific country; (c) as a *fallback
path* when your own fetch fails, rather than the primary path; (d) Scrapy Cloud specifically as
*hosting/scheduling* for spiders you wrote, which is a different purchase from per-request unblocking.

### 5.2 Documented criticisms of the proxy vendors

This is the ugliest part of the landscape and it carries institutional risk:
- **55555** (2021-08-11): *"Companies like Bright Data sell access to what is essentially a legal
  botnet"* where innocent users unknowingly become infrastructure.
- **sroelants** (2025-10-27): providers embed SDKs in consumer apps that convert users' devices into
  crawler infrastructure. **This is the sourcing mechanism for "residential" IPs** — someone
  installed a free VPN or a game and their home connection became your exit node.
- **cute_boi** (2026-06-23): *"Bright data is formally Luminati proxy. They are well known for doing
  shady things."*
- **waterproof** (2025-11-05): Bright Data blocks its own domain when accessed through its own proxy
  service.
- Billing shape: **super256** (2023-05-14) cites residential bandwidth billed at **$9.45/GB**;
  **faizshah** (2025-04-13) cites ~**$4/GB** residential and **$1–1.3/1k requests** for scraping APIs
  — roughly corroborating the published numbers above. **vivzkestrel** (2026-08-22) questions whether
  rotating residential proxies justify the expense at all.
- Counterweight — people who shipped on them: **alephu5** (2021-06-26) ran a 2M pages/day project on
  Luminati; **telecomhacker** (2025-12-01) used Bright Data, Zyte and Oxylabs together successfully;
  **tomp** (2022-08-10) recommends Bright Data.

**For a research or public-sector project, routing traffic through consumer devices that did not
meaningfully consent is an ethics-review and reputational problem, not merely a procurement one** —
and it is almost never necessary for cooperative government and scientific sources.

---

## 6. STORAGE / DB — trade-offs for append-heavy, versioned scraped data

The requirement shape: append-heavy, rarely-updated, immutable-once-written snapshots; you want to
answer "what did this record look like on date X" and "what changed"; volumes grow monotonically for
months.

| Option | Strengths | Documented criticisms |
|---|---|---|
| **Postgres** | Transactional, familiar, excellent for job/queue state, metadata, dedup indexes, JSONB for semi-structured records; one thing to operate | Row-based, poor at analytical scans. **nikita** (2024-08-17): Postgres is *"at the bottom of the list and ~1000x slower than @duckdb and @ClickHouseDB"* on ClickBench. **physicles** (2024-01-27): *"DuckDB is column-based and Postgres is row-based. For analytics workloads, I'm having a hard time thinking of a scenario where Postgres wins."* **saisrirampur** (PeerDB/ClickHouse, 2024-08-18): *"The Postgres extension framework is complex, still maturing"* and Citus customers moved to purpose-built DBs |
| **DuckDB** (**1.5.5**, 2026-07-22) | **notpushkin** (2024-09-12): *"pretty much the SQLite for analytics."* Reads Parquet directly; zero ops; excellent SQL coverage — **wenc** (2023-02-11) notes better SQL than ClickHouse incl. lead/lag window functions | **Single-process, single-writer** — an embedded engine, not a concurrent server. Do **not** make it the shared write target for N concurrent scrapers. A v2.0 is in preview (<https://news.ycombinator.com/item?id=49330781>), so expect change |
| **ClickHouse** (`clickhouse-connect` **1.8.0**, 2026-09-03) | **atemerev** (2025-06-21): *"it is _really_ hard to ignore 50x (or more) speedup"* for logs, events, metrics and rarely-updated data — **which is exactly this workload's shape** | **atemerev**, same comment: if *"your data is highly mutable, or you cannot do batch writes... ClickHouse is a wrong choice."* **Batch writes are mandatory** — many small inserts is a known anti-pattern. **wenc**: supports only *"a subset of SQL"*. Real cluster ops burden |
| **Parquet on object storage** | Cheapest durable storage; open format = **lowest lock-in of any option**; natural partitioning by `source/date`; queryable by DuckDB, ClickHouse, Athena, Spark, pandas/Polars. **awill88** (Dec 2021) advocates Parquet + Arrow to *"democratize the dataset by using Athena / BigQuery"* | No transactions, no indexes, no updates — small-file problem if you write per-record; needs a compaction step. Metadata/catalogue is on you unless you add Iceberg/Delta (more moving parts) |
| **SQLite + Datasette** | Superb for per-source datasets and publishing; the whole Willison toolchain (`git-history`, `sqlite-utils`, `datasette`, `csv-diff`) is built for exactly this | **simonw** (Dec 2022) states the limits himself: *"Only send writes from a single process (maybe via a queue)... Don't use it if you're going to want to horizontally scale to handle more than 1,000 requests/second."* **di456** (Dec 2022): the hard question is *"how and where to persist the SQLite data cheaply"* once it outgrows a repo |
| **MongoDB** | Schema-flexible — genuinely useful when 200 heterogeneous sources have unstable shapes and you want to land raw records before normalising | Weakest analytical story of the set; largely superseded by Postgres JSONB for this purpose; notably absent from the practitioner threads I found — **no HN advocacy surfaced in this research**, which is itself a signal |

**The pattern the practitioners actually converge on** — and the strongest recommendation-shaped thing
in this document, because it appears independently in multiple threads:

**Store the raw response, immutably, in cheap object storage, and treat parsing as re-runnable.**
From the 100B-pages thread: practitioners *"recommended storing unparsed HTML and JSON responses to
enable replay during parser failures, treating network requests as expensive operations."*
**thoop** (Prerender.io founder, 2022-09-28) describes exactly this split at scale: *"All of the HTML
was stored in s3"* with Postgres (20 sharded RDS databases) for metadata only.

For a months-unattended government/scientific harvester this is close to essential: when you discover
in month 5 that a selector has been silently wrong since month 2, raw-response archives are the
difference between reparsing and re-scraping (which for many government sources is *impossible* —
the old page is gone). A defensible default: **raw responses → object storage (content-addressed,
partitioned by source/date); metadata, job state and dedup → Postgres; analytics → DuckDB or
ClickHouse over Parquet extracts.** The storage layer is also where lock-in is cheapest to avoid —
Parquet and SQLite are open formats you can walk away from.

---

## 7. DEPLOYMENT

### 7.1 The Kubernetes argument, both sides, with attribution

Thread: <https://news.ycombinator.com/item?id=46576224> — *"Kubernetes Was Overkill. We Moved to
Docker Compose and Saved 60 Hours"* (~Feb 2026, per "7 months ago" at fetch time).

**Against K8s for a small team:**
- The article's own claim, as summarised by **austin-cheney**: *"we're spending 60 hours a week
  managing Kubernetes instead of shipping features"* — with **eight engineers**.
- **jaynamburi**: *"YAML sprawl, slow feedback loops, and time spent maintaining the platform instead
  of the product."* He also argues *"simpler"* is sometimes the more senior engineering choice.
- **elthor89** names the risk that most matters for a small team running something for years:
  *"we'd built a dependency on one person's specialized knowledge. And that knowledge had nothing to
  do with our actual product."* **For a system that must run unattended for months, bus-factor on
  platform expertise is a first-order reliability concern.**

**For K8s:**
- **mmh0000** reframes it: Kubernetes is *"simply a distributed init system"* when used with
  restraint, and it provides automatic load-balancing between pods and automatic node failover and
  recovery — *"capabilities Docker Compose cannot match."*
- **superze** and **the_real_cher** argue the real problem is insufficient upskilling, not the tool.
- **halfmatthalfcat** calls the migration an *"overreaction"* driven by frustration rather than
  objective assessment.

**Related:** *"Replacing Kubernetes with systemd (2024)"*
(<https://news.ycombinator.com/item?id=43899236>), *"Zero-Downtime Deployments with Docker Compose –
No Kubernetes Required"* (<https://news.ycombinator.com/item?id=48665130>), and Podman Quadlets —
running containers as systemd units (<https://news.ycombinator.com/item?id=43456934>,
<https://news.ycombinator.com/item?id=45945200>). **Quadlets are the most under-appreciated option
here**: container images for reproducibility, systemd for supervision/restart/logging/timers, no
orchestrator at all.

### 7.2 Nomad as middle ground

**Verified:** Nomad **v2.0.4 (2026-07-07)**, with v2.0.1→v2.0.4 shipping May–July 2026 at roughly
2–3 week intervals, alongside maintained 1.11.x/1.10.x enterprise lines
(<https://github.com/hashicorp/nomad/releases>).

- **sofixa** (May 2026) defends orchestrators against the systemd position while conceding the point:
  *"Yes, not everyone needs Kubernetes, Nomad or other advanced orchestrators. But comparing them to
  running a Go binary with systemd is an unfair comparison."*
- **quickslowdown** (April 2025) is the honest middle: *"I've gotten much further with Nomad than
  Kubernetes, but I've kind of always gone back to ol' faithful, writing a docker compose file."*
- **Glemkloksdjf** (Dec 2025) argues the opposite — *"If you do anything professional, you better
  choose proven software like kubernetes or managed kubernetes"* — citing the ecosystem: IaC, cloud
  provider support, cert management, observability, ArgoCD, HA out of the box.

**Two gaps I could not close and you should:** (1) I found **no substantive HN criticism of Nomad's
BSL licence or ecosystem constraints** in this research — the comment corpus discusses K8s vs Compose
far more than Nomad specifically, so Nomad's practitioner evidence base is **thin**, which is itself
a risk for a multi-year bet. (2) I could not verify Nomad's current licence text from a primary
source in this session (the releases page did not state it). **HashiCorp products moved to BUSL in
2023 and HashiCorp was acquired by IBM — verify Nomad's current licence and its implications
directly before adopting.**

### 7.3 Serverless

Not well covered in the practitioner threads I found. The structural mismatch is worth stating
anyway: hard execution-time ceilings conflict with long paginated harvests; per-invocation billing
conflicts with continuous polling; cold starts conflict with browser automation; and egress IPs are
shared and often already rate-limited or blocked by government hosts. Serverless suits the
*git-scraping shape* (tiny, frequent, stateless snapshot jobs), not the *deep-harvest shape*.
**Flagged as an evidence gap** rather than a settled conclusion.

---

## 8. GIT SCRAPING — a genuinely different, genuinely cheap architecture, with a newly serious limit

**The pattern** (<https://simonwillison.net/2020/Oct/9/git-scraping/>, **2020-10-09**): *"a technique
where data is scraped from an external source into a Git repository in order to record changes to that
data over time."* Implementation is one GitHub Actions workflow file: fetch on a schedule (Willison
staggers at 6, 26, 46 past the hour), pretty-print (`curl` piped through `jq .`), commit only if
changed. The commit log becomes a free, complete changelog. His examples are almost all
government-adjacent: California fire incidents, San Francisco's tree inventory, PG&E power outages,
FARA registrations.

**Why it is a serious contender for this project specifically:** government and scientific sources are
disproportionately *small, slow-changing, snapshot-shaped* — a JSON endpoint, a CSV, a status table.
For those, git scraping gives you versioning, diffing, auditability and provenance **for free**, with
zero infrastructure. **pjot** (2024) ran power-company outage mapping to **67,000 commits**,
demonstrating sustained viability.

**The ecosystem:** `git-history` (converts git-scraped data into SQLite), `sqlite-utils`, `datasette`
(publish/explore), `shot-scraper` (headless-browser extraction), `csv-diff` (meaningful commit
messages), and a `git-scraper-template` repo. Willison's more recent work (2025) uses LLMs to generate
scrapers faster; his most recent tagged posts are Dec 2025 (`actions-latest`) and Mar 2025 (Ollama
models Atom feed, plus a NICAR 2025 talk on *"Cutting-edge web scraping techniques"*)
(<https://simonwillison.net/tags/gitscraping/>).

### 8.1 The limits — including one that has become severe *this year*

**This is the single most important current finding about git scraping, and it is not in any of the
tutorials.** GitHub Actions scheduled-cron reliability has degraded sharply in 2026. From
<https://github.com/orgs/community/discussions/156282>, GitHub staff member **nebuk89**
acknowledged on **2026-06-04**: *"We are aware that the drift on the start of our scheduled jobs has
got worse"*, attributing it to load balancing and noting *"scheduled drops have grown >30% in 2ish
months"*, with no committed timeline. Reported delays:

| Period | Reported cron delay |
|---|---|
| April 2025 | 20–30 minutes |
| May–June 2026 | 45 min to 2+ hours |
| Late June–July 2026 | **1–3 hours average, some >4 hours** |

One user documented a `06:30 UTC` job averaging **2h42m late** over seven days (April–July 2026).
The delay is **upstream of runner allocation** — in GitHub's event dispatch queue — so it affects
**self-hosted runners too**. It worsened markedly around February 2026.

**Consequence:** if your harvest cadence needs to be tighter than "a few times a day, whenever",
GitHub Actions cron is currently not a dependable scheduler. For sources where the *snapshot timing*
is part of the data (outage maps, live counts, auction/permit statuses), multi-hour drift corrupts the
time series. Mitigations: trigger via `workflow_dispatch` from an external scheduler you control, or
run the same scripts under your own cron/systemd and push to git.

**Other documented limits:**
- **Willison himself** (2022): *"I do worry that they'll change their policy with regards to free
  minutes for public repos at some point"*, and he cautions against *"saving huge binary files to free
  repositories."*
- **Workflows auto-disable after 60 days of repository inactivity** — a direct hazard for
  *unattended, months-long* operation, which is precisely this project's requirement. Keep-alive
  workarounds exist (<https://github.com/efrecon/gh-action-keepalive>,
  <https://github.com/PhrozenByte/gh-workflow-immortality>), which is a strong signal that the
  platform is being used against its grain.
- **Repository bloat.** Frequent commits and any binary content grow the repo unboundedly.
  **awill88** (Dec 2021) had to write *"intricate code to essentially perform a mitosis each month"*
  to avoid size limits.
- **di456** (Dec 2022): where to persist the data *cheaply* once it outgrows a repo is unsolved.
- Willison also notes practical needs for rate-limiting and handling blocked IP ranges — he used
  **Tailscale exit nodes** to get around Cloudflare blocks on GitHub Actions' IP ranges. Note the
  general problem: **GitHub Actions egress IPs are widely known and sometimes blocked**, and shared
  with every other Actions user.
- Also see GitHub's documented Actions limits: <https://docs.github.com/en/actions/reference/limits>.

**Honest scope:** git scraping is excellent for the *long tail* of small snapshot sources and for
provenance; it is not an architecture for high-volume, deep, paginated harvests. Nothing stops you
running **both** — git scraping for the 150 small sources, a real pipeline for the 20 big ones.

---

## 9. RETROSPECTIVES AND OPERATIONAL LESSONS FROM LONG RUNS

### 9.1 "Lessons learned scraping 100B product pages" (<https://news.ycombinator.com/item?id=17497184>, 2018)

Dated, but the operational lessons are the most durable material in this document:
- **Store raw HTML/JSON** to enable replay when parsers fail; treat network requests as expensive.
- **Caching infrastructure matters** — one company found aging caches couldn't handle scraper load and
  moved to CDN + Redis.
- **Proxy management is a real cost centre** — managing thousands of proxies was resource-intensive
  enough that teams switched to managed services.
- **Logging/monitoring is critical** — high-concurrency systems need *"highly available Logging and
  monitoring system"* to debug failures at all.
- **Language/concurrency:** one commenter reported **10K pages/second sustained on four servers in
  Go**; others advocated Elixir/BEAM for concurrency without complexity
  (`Task.async_stream`); several rejected GIL languages for CPU-bound work — *"I am never going back
  to languages with a global interpreter lock, ever."*
- **Counter-current on complexity:** *"Start with single threaded and simple, move to multi-threaded
  scrapers when the juice is worth the squeeze."*

### 9.2 The fragility disagreement — genuinely unresolved

From <https://news.ycombinator.com/item?id=24420120> (2020-09-09):
- **b6z**: *"Web scraping is fragile! Web sites change, web frameworks evolve, and just some subtle
  reordering of some `<divs>` or renaming of CSS classes, and your perfect scraping code from
  yesterday will break tomorrow."*
- **hansvm** flatly contradicts this from his own experience: website structures are relatively
  stable; he has *"wasted effort in hindsight since web pages are stable and I never have to update
  my scrapers."*

**Both are probably right about different populations of sites, and the split matters here.**
Government and scientific sites skew heavily toward hansvm's world — legacy CMSes, decade-old table
markup, standardised endpoints (OAI-PMH, CKAN, Socrata), and slow procurement cycles that mean
redesigns happen every 5–10 years, not quarterly. The counterweight: when a government site *does*
migrate, it often changes *everything* at once, including URL structure — a rare, total break rather
than continuous drift. **Design for rare catastrophic breakage plus silent content drift, not for
constant selector churn.**

**API stability, also disputed:** **hansvm** observes APIs are often *less* stable than page
structures when maintained as a secondary concern; **achillean** counters that companies which consume
their own public APIs maintain them well. For government data, the practical read: prefer documented
bulk/API endpoints, but *version-pin and monitor them like scrapers*, because "official" does not mean
"stable".

**Monitoring is the agreed-upon mitigation.** **joshxyz** (2020-09-09): add real-time validation
checks on scraped content and alert when unexpected formats appear. This is the one thing every thread
agrees on, and it is what the LLM-extraction critique (§3.2, "silent errors evading detection for
weeks") and the adaptive-selector critique (§1.9) both point at. **For a months-unattended system,
per-source output validation with alerting is not optional infrastructure — it is the difference
between a dataset and a liability.** Concretely: assert expected record counts within a tolerance
band, non-null rates per field, type/range/enum validity, and inter-run diff magnitude; alert on
*silence* too (a scraper returning zero rows without erroring).

### 9.3 The tool-churn retrospective

**michael_j_x** (2024-12-17): `requests` → Scrapy → Selenium → `undetected_chromedriver` →
SeleniumBase → Playwright, driven almost entirely by **detection avoidance** rather than by
capability. Each migration was forced by an external change (a Chrome update detecting
`undetected_chromedriver`).

**The lesson for this project is a hopeful one:** that churn cycle is driven by *adversarial*
targets. A harvester of cooperative government and scientific sources that identifies itself honestly,
respects robots.txt and rate-limits politely is **largely exempt from the treadmill that generates
most tool churn in this field** — and can therefore afford a much more boring, stable stack than the
scraping-vendor content assumes.

---

## 10. EXPLICIT DISAGREEMENTS — the summary table

| Question | Position A | Position B |
|---|---|---|
| Framework or plain scripts? | **krakensden** (2011): *"Scrapy is overkill for nearly everything. You'll probably have under a page of code using lxml and urllib."* **datalopers** (2022): *"stop adding in layers... when a simple cron would work"* | **cdr** (2011): dismissing it is like saying *"jQuery is overkill... you should use plain javascript."* **tekkk** (2017) changed his mind: simple scripts *"failed miserably"* on larger projects needing cookies and politeness |
| Are scrapers fragile? | **b6z** (2020): *"Web scraping is fragile!... your perfect scraping code from yesterday will break tomorrow"* | **hansvm** (2020): *"web pages are stable and I never have to update my scrapers"* |
| Headless browser by default? | **waprin** (2018): start with headless browsers despite the overhead | **lyjackal** (2018): *"headless browser resources are pretty huge"*; **hermanradtke**: parsing speed wins at scale |
| Is Airflow acceptable? | **niwtsol** (2025): *"overly complex for what's really a pretty simple data flow"*; **mushufasa** (2025): *"a really dated choice for a greenfield project"*; **wharvle** (2024): *"half-baked"* | **disgruntledphd2** (2025): *"it sucks in well known and predictable ways"* — predictability has real value for multi-year unattended operation |
| Is Temporal worth the weight? | **graerg** (2026-05): 4+ services, Helm *"substantial ops burden"*, upgrade failures with data loss. **BowBun** (2026-01): *"a lot of boilerplate"* for small jobs. **pm90** (2025): *"When a thing breaks then ur hosed"* | **jtmarmon** (2024): composability, multi-language SDKs, heterogeneous workers. **emmanueloga_** (2026-01): less boilerplate than hand-rolling transactional-outbox queue patterns |
| Argo Workflows? | **verdverm** (2025-11): *"does not live up to what they advertise, it is so much more complex"*; **odie5533** (2025-11): startup too slow to replace Celery | **verdverm** (2026-03), *same person*: *"a top choice for self hosted"*; **sleepybrett** (2026-08): 9 months of successful use |
| Kubernetes for a small team? | **jaynamburi**: *"YAML sprawl, slow feedback loops"*; **elthor89**: bus-factor on *"one person's specialized knowledge... nothing to do with our actual product"* | **mmh0000**: *"simply a distributed init system"*, gives pod load-balancing and node failover *"Docker Compose cannot match"*; **halfmatthalfcat**: the migration was an *"overreaction"* |
| Nomad as middle ground? | **quickslowdown** (2025-04): *"I've gotten much further with Nomad than Kubernetes, but I've kind of always gone back to... docker compose"* | **sofixa** (2026-05): comparing real orchestrators *"to running a Go binary with systemd is an unfair comparison"*; **Glemkloksdjf** (2025-12): *"better choose proven software like kubernetes"* |
| Postgres for analytics? | **nikita** (2024): *"~1000x slower than @duckdb and @ClickHouseDB"*; **physicles** (2024): row vs column store is decisive | Nobody in these threads defends Postgres *for analytics* — but its role as the metadata/job-state store is uncontested. **thoop** ran Postgres for metadata and S3 for HTML |
| ClickHouse for scraped data? | **atemerev** (2025-06): *"hard to ignore 50x (or more) speedup"* for rarely-updated event data | **atemerev**, *same comment*: wrong choice if data is *"highly mutable, or you cannot do batch writes"*; **wenc**: only *"a subset of SQL"* |
| LLM extraction? | Cheap on value models with cleaned HTML (**$0.58/1k pages**); handles heterogeneous sources without per-site code | *"Models return plausible but incorrect values, evading detection for weeks"*; can't read markup-encoded data; no network layer. **ScrapeGraphAI's own README: *"for data exploration and research purposes only"*** |
| Managed platform or self-host? | **telecomhacker** (2025-12), **alephu5** (2021, 2M pages/day) shipped on Bright Data/Zyte/Oxylabs | **machinelinux** (2026-03): *"Firecrawl charges ~$100/mo... Kryfto runs on a $5/mo VPS with zero per-request costs"*; **55555** (2021): residential proxies are *"essentially a legal botnet"*; **oasisbob** (2026-03): deceptive UA + ignoring REP is *"no way to build a business"* |
| Git scraping? | **pjot** (2024): 67k commits, sustained. Free hosting, free versioning, free provenance | GitHub cron drift **1–3 h in mid-2026, staff-acknowledged**; workflows auto-disable after 60 days idle; repo bloat; **Willison**: don't save *"huge binary files to free repositories"* |

---

## 11. Notable gaps and stale material — flagged

- **Colly's maintenance status is unresolved and is the biggest open question in §1.** Newest tagged
  release is v2.2.0 (2024-03-27). I could not check master commits (robots-disallowed) or the Go
  module proxy (outside this session's egress allowlist). **Verify before adopting.**
- **`httpx` 0.x last released 2024-12-06**; continuation is **`httpx2` (2.12.0, 2026-08-18)** under the
  Pydantic org. Any guide recommending `httpx` predates this transition.
- **The Crawlee-vs-Scrapy comparison everyone cites is Apify-authored and dated April 2024** — both
  partisan and stale relative to Scrapy 2.14–2.18.
- **The Airflow 3.0 "what hurts" critique is undated 2025 material** and 3.3.1 shipped 2026-08-12;
  its claim that **AWS MWAA is still on 2.x** is the most likely item to have changed.
- **The richest orchestration-paradigm thread is from Feb 2024** — predating Airflow 3.x, Prefect 3.x
  and Restate's maturation.
- **The 100B-pages retrospective is from 2018.** Its operational lessons (archive raw responses, log
  everything) are durable; its tool specifics are not.
- **No substantive HN debate found** on: LLM-generated-selectors vs runtime LLM extraction; Kestra
  (all comparison content is vendor-published); Nomad's BSL licence; serverless for scraping;
  MongoDB for scraped data. Treat these as un-evidenced, not as settled.
- **Judoscale's task-queue comparison carries no publication date** and the vendor sells queue
  autoscaling.
- **Vendor pricing is volatile and geo-targeted.** Bright Data showed promotional "50% off" pricing at
  fetch time. ScrapingBee does not publish per-request-type credit multipliers on its pricing page.
  Re-verify everything in §5 before budgeting.
- **I could not fetch `docs.scrapy.org`** (proxy returned 403 for that host), so Scrapy claims here
  rest on PyPI metadata and the raw `news.rst` from GitHub — arguably better primary sources anyway.

---

## 12. Sources

**Primary — versions and releases (verified 2026-09-06)**
- Scrapy release notes (raw): https://raw.githubusercontent.com/scrapy/scrapy/master/docs/news.rst
- Scrapy on PyPI (2.18.0, 2026-08-20): https://pypi.org/pypi/scrapy/json
- Scrapy releases (GitHub, render was incomplete): https://github.com/scrapy/scrapy/releases
- PyPI JSON API used for: crawlee, playwright, selenium, apache-airflow, dagster, prefect, temporalio,
  scrapling, httpx, httpx2, aiohttp, lxml, selectolax, beautifulsoup4, firecrawl-py, duckdb,
  clickhouse-connect — e.g. https://pypi.org/pypi/crawlee/json
- Airflow release notes: https://airflow.apache.org/docs/apache-airflow/stable/release_notes.html
- Airflow 3 GA announcement: https://airflow.apache.org/blog/airflow-three-point-oh-is-here/
- Crawlee for Python releases: https://github.com/apify/crawlee-python/releases
- Crawlee for Python repo: https://github.com/apify/crawlee-python
- Colly repo / releases: https://github.com/gocolly/colly · https://github.com/gocolly/colly/releases
- curl_cffi: https://github.com/lexiforest/curl_cffi
- Katana: https://github.com/projectdiscovery/katana
- Scrapling: https://github.com/D4Vinci/Scrapling
- ScrapeGraphAI: https://github.com/ScrapeGraphAI/Scrapegraph-ai
- Apache StormCrawler: https://stormcrawler.apache.org/ · https://stormcrawler.apache.org/docs/
- Nomad releases: https://github.com/hashicorp/nomad/releases
- Prefect 3.0 GA: https://www.prefect.io/blog/prefect-3-generally-available-september-3
- Prefect 3 upgrade guide: https://docs.prefect.io/v3/how-to-guides/migrate/upgrade-to-prefect-3
- Prefect migration issues: https://github.com/PrefectHQ/prefect/issues/13979 ·
  https://github.com/PrefectHQ/prefect/issues/15275

**Pricing (fetched 2026-09-06)**
- Zyte API: https://www.zyte.com/pricing/ · Scrapy Cloud: https://www.zyte.com/scrapy-cloud/
- Apify: https://apify.com/pricing
- Bright Data: https://brightdata.com/pricing
- Oxylabs Web Scraper API: https://oxylabs.io/products/scraper-api/web
- ScrapingBee: https://www.scrapingbee.com/pricing/
- ScraperAPI: https://www.scraperapi.com/pricing/
- Firecrawl: https://www.firecrawl.dev/pricing
- Diffbot: https://www.diffbot.com/pricing/
- Jina Reader: https://jina.ai/reader/
- Temporal Cloud: https://temporal.io/pricing
- Dagster+: https://dagster.io/pricing
- Prefect Cloud: https://www.prefect.io/pricing

**Practitioner discussion (Hacker News)**
- Lessons learned scraping 100B product pages (2018): https://news.ycombinator.com/item?id=17497184
- How we learnt to stop worrying and love web scraping (2020-09): https://news.ycombinator.com/item?id=24420120
- Scraping tool evolution → Playwright (2024-12): https://news.ycombinator.com/item?id=42445575
- Simplest data orchestration tool? (2025-03): https://news.ycombinator.com/item?id=43439939
- Lessons from running Airflow at scale (Shopify, 2022): https://news.ycombinator.com/item?id=31480320
- Airflow/Dagster/Prefect vs Temporal (2024-02): https://news.ycombinator.com/item?id=39211186
- Temporal as superset of DAG managers? (2022): https://news.ycombinator.com/item?id=31485170
- Perfect use case for Cadence/Temporal (2024-02): https://news.ycombinator.com/item?id=39210757
- Kubernetes Was Overkill → Docker Compose (~2026-02): https://news.ycombinator.com/item?id=46576224
- Replacing Kubernetes with systemd: https://news.ycombinator.com/item?id=43899236
- Zero-downtime deploys with Docker Compose: https://news.ycombinator.com/item?id=48665130
- Podman Quadlets: https://news.ycombinator.com/item?id=43456934 · https://news.ycombinator.com/item?id=45945200
- ClickHouse vs DuckDB: https://news.ycombinator.com/item?id=37610689
- DuckDB v2.0 preview: https://news.ycombinator.com/item?id=49330781
- Keeping scraped data up to date (2025-12): https://news.ycombinator.com/item?id=46395677
- HN Algolia comment search API (used for attributed quotes on Firecrawl, Scrapy-overkill, Nomad,
  Temporal, git scraping, ClickHouse/DuckDB/Postgres, Argo, Datasette/SQLite, proxy vendors, crawler
  architecture): https://hn.algolia.com/api/v1/search

**Git scraping**
- Simon Willison, "Git scraping" (2020-10-09): https://simonwillison.net/2020/Oct/9/git-scraping/
- Willison's gitscraping tag: https://simonwillison.net/tags/gitscraping/
- Willison's github-actions tag: https://simonwillison.net/tags/github-actions/
- **GitHub staff on worsening cron drift (2026-06-04)**: https://github.com/orgs/community/discussions/156282
- Scheduled workflows not running: https://github.com/orgs/community/discussions/185355
- GitHub Actions limits: https://docs.github.com/en/actions/reference/limits
- Workflow keep-alive workarounds: https://github.com/efrecon/gh-action-keepalive ·
  https://github.com/PhrozenByte/gh-workflow-immortality

**Vendor/third-party analysis (partisan or undated — flagged in text)**
- Zyte, "Scrapy in 2026: modern async crawling": https://www.zyte.com/blog/scrapy-in-2026-modern-async-crawling/
- Apify/Crawlee, "Scrapy vs Crawlee" (2024-04-23): https://crawlee.dev/blog/scrapy-vs-crawlee
- webscraping.ai, "LLM Web Scraping: Models, Cost, Pipelines" (2026-08): https://webscraping.ai/blog/llm-web-scraping
- Stribog, "Workflow Orchestration You Host: Temporal vs Airflow 3" (2026-08-26):
  https://stribog.com/blog/self-hosted-workflow-orchestration-temporal-airflow-sovereign-scheduling
- PkgPulse, "Temporal vs Restate vs Windmill" (2026-03): https://www.pkgpulse.com/guides/temporal-vs-restate-vs-windmill-durable-workflow-2026
- Danube Data Labs, "Airflow 3.0: What's New, What Hurts" (2025, undated): https://danubedatalabs.com/apache-airflow-3-0-new-features-what-hurts-and-should-you-upgrade/
- Judoscale, "Choosing the Right Python Task Queue" (undated): https://judoscale.com/blog/choose-python-task-queue
- ScrapingBee on Scrapling: https://www.scrapingbee.com/blog/scrapling-adaptive-python-web-scraping/
- Kestra comparison pages (vendor-published): https://kestra.io/vs/windmill ·
  https://kestra.io/resources/infrastructure/temporal-alternatives
- Windmill peer comparison (vendor-published): https://www.windmill.dev/docs/compared_to/peers
- Nutch vs StormCrawler (DZone): https://dzone.com/articles/the-battle-of-the-crawlers-apache-nutch-vs-stormcr



---

# Adversarial verification — batch A (checked 2026-09-06)

Note: this session's WebSearch budget was exhausted after claim 5, so everything from
claim 6 onward was verified by direct WebFetch against official URLs only. `curl` is
proxy-restricted to package registries (pypi.org worked; api.data.gov, docs.scrapy.org,
web.archive.org were refused by the gateway with 403 CONNECT).

| # | Claim | Verdict | Finding |
|---|-------|---------|---------|
| 1 | Hetzner 2026 double price rise; CX22 exists | **MIXED / mostly UNVERIFIED + one CORRECTION** | Prices unobtainable from official pages (JS-rendered, archive.org blocked). CX22 does **not** exist any more. |
| 2 | Common Crawl CC-MAIN-2026-34 | **CONFIRMED** | Every figure matches the announcement. |
| 3 | Crossref new rate limits 2025-12-01 | **UNVERIFIED** | No numbers or effective date on crossref.org. |
| 4 | OpenAlex API key required, credits, HTTP 409 | **CORRECTED** | Key not required; budget is dollar-denominated ($1/day keyed, $0.10/day keyless); error is 429. |
| 5 | SEC EDGAR 10 req/s + User-Agent | **CONFIRMED (partial)** | 10/s and User-Agent confirmed; "regardless of the number of machines" not found. |
| 6 | NCBI E-utilities limits | **CONFIRMED (partial)** | 3/s, 10/s with key, one key per account, weekend/9pm–5am all confirmed; the "does not allow scripting" quote not found. |
| 7 | arXiv 1 req / 3 s, single connection, collective | **CONFIRMED** | Verbatim match. |
| 8 | api.data.gov 1,000 req/hour | **CONFIRMED** | "Hourly Limit: 1,000 requests per hour". |
| 9 | Scrapy 2.18.0 (2026-08-20), asyncio migration | **CONFIRMED** | PyPI + release notes; httpx handler now uses httpx2. |
| 10 | httpx last release 0.28.1 (2024-12-06); httpx2 under Pydantic | **CONFIRMED with correction** | httpx2 2.12.0 (2026-08-18) under pydantic is real; but httpx has 1.0.dev pre-releases up to 2026-08-31. |

---

## 1. Hetzner Cloud pricing

**Verdict: price figures UNVERIFIED; CX22 claim CORRECTED.**

What was verified from official Hetzner sources:

- **CX22 no longer exists.** Hetzner Cloud API changelog (Oct 16 2025): "Starting on
  1 January 2026, the old server plans with shared Intel® vCPUs will no longer be
  available for order." Same changelog announced the replacements: "Four new cost
  optimized server types are now available: cx23 (ID 114), cx33 (ID 115), cx43 (ID 116),
  cx53 (ID 117)" and "Six new regular purpose server types … cpx12 (108), cpx22 (109),
  cpx32 (110), cpx42 (111), cpx52 (112), cpx62 (113)".
  Source: https://docs.hetzner.cloud/changelog
- **CX23** is the 2 vCPU / 4 GB / 40 GB NVMe / **20 TB traffic** plan (EU locations), per
  https://www.hetzner.com/cloud/cost-optimized/ — so the "2 vCPU / 4 GB / 40 GB with
  20 TB traffic" tier still exists, but as CX23, not CX22. 20 TB included traffic is
  confirmed for the whole CX/CAX cost-optimized line and for CCX13/CCX23.
- **CPX52** is itself a plan introduced only in Oct 2025 (12 vCPU / 24 GB / 480 GB /
  20 TB, per https://www.hetzner.com/cloud/regular-performance/), and the older CPX51
  (16 vCPU / 32 GB) still appears for US locations. A "€36.49 → €100.49" history for
  CPX52 is therefore doubtful on its face — the plan did not exist long enough to have a
  pre-April-2026 price of €36.49 under the old generation's pricing.
- **CCX13** is 2 vCPU / 8 GB / 80 GB / 20 TB (https://www.hetzner.com/cloud/general-purpose/).

What could **not** be verified:

- No euro prices at all. hetzner.com/cloud, /cloud/cost-optimized/, /cloud/general-purpose/,
  /cloud/regular-performance/ and /cloud/dedicated-vcpu/ all render prices client-side; the
  served HTML contains the spec tables but placeholders where prices go ("starting from
  max/mo").
- archive.org: a snapshot exists (http://web.archive.org/web/20260722210829/https://www.hetzner.com/cloud/,
  confirmed available via the Wayback availability API) but web.archive.org and archive.ph are
  both blocked to this session's fetcher.
- **No official announcement of any 2026 price increase was found.** hetzner.com/news/ shows
  nothing later than June 2024; hetzner.com/blog/ has no pricing posts; the Hetzner Cloud API
  changelog's 2026 entries (Mar–Aug 2026) are exclusively image deprecations, load-balancer
  and datacenter-endpoint changes — no pricing or new server types.
- The "April 1 and June 15 2026" dates, the €15.99→€42.99 / €36.49→€100.49 / €3.99→€5.49
  figures, the "33–38%" spread and the "DRAM up ~171% YoY" causal claim have **no primary
  source behind them here**. Treat all of them as unsupported. The DRAM figure in particular
  reads like a secondary/vendor-blog number.

**Recommendation:** do not publish any Hetzner price figure without a live check of
console.hetzner.com or an authenticated `GET /v1/pricing` call.

## 2. Common Crawl CC-MAIN-2026-34 — CONFIRMED

Source: https://commoncrawl.org/blog/august-2026-crawl-archive-now-available (Aug 24, 2026)

- Crawl ID: **CC-MAIN-2026-34**
- Date range: **August 7–20, 2026**
- **2.14 billion** web pages, **360 TiB** uncompressed
- **100,000 WARC files**, **84.78 TiB** gzipped WARC
- WET **5.84 TiB**; WAT 13.92 TiB (extra datum, not in the claim)
- 100 segments; >40 million hosts, 33.1 million registered domains, ~700 million
  previously-uncrawled URLs

Every figure in the claim matches. (Cross-check: the July 2026 crawl was also 2.14 B pages /
364.01 TiB, so the page count is not a copy error.)

## 3. Crossref REST API rate limits — UNVERIFIED

Tried:

- https://www.crossref.org/documentation/retrieve-metadata/rest-api/tips-for-using-the-crossref-rest-api/
  — discusses backing off on HTTP 429 but states **no numeric limits**.
- https://www.crossref.org/documentation/retrieve-metadata/rest-api/ — mentions the polite
  pool with `mailto` in the quick-start, no numbers, no effective date.
- https://www.crossref.org/blog/ and /blog/page/2/ — no post about rate limits, throttling or
  the polite pool in late 2025 or 2026. Most recent API-adjacent post concerns rebuilding the
  "Content System and REST API" monoliths.
- https://api.crossref.org/ — returned **HTTP 429** to this fetch (consistent with tight
  limits now being enforced, but not evidence of the specific numbers).
- https://api.crossref.org/swagger-ui/index.html — JS-only shell, no readable content.
- https://github.com/CrossRef/rest-api-doc — Crossref's own (now deprecated: "This
  documentation is outdated… Current docs are at https://api.crossref.org/") repo says limits
  are advertised in `X-Rate-Limit-Limit` / `X-Rate-Limit-Interval` headers and shows the
  historical example `X-Rate-Limit-Limit: 50` / `X-Rate-Limit-Interval: 1s`. It documents the
  public/polite pool split and `mailto`, and says nothing about concurrent connections.

So: the polite-pool mechanism is real and documented; the specific figures (5/s single-item,
1/s lists, 1 connection public; 10/s, 3/s, 3 connections polite) and the 2025-12-01
effective date are **not supported by any Crossref page found**. The only Crossref-published
number is the legacy 50/s header example. The claim's shape (per-second split by query type
plus a concurrency cap) does not match how Crossref documents limits at all. Verify by
reading `X-Rate-Limit-*` headers off a live request before publishing.

## 4. OpenAlex — CORRECTED (claim is wrong in three of four particulars)

Sources: https://help.openalex.org/api/authentication (last updated Aug 19, 2026),
https://help.openalex.org/api/, https://help.openalex.org/access/pricing/,
https://help.openalex.org/access/example-costs/

- **API key is NOT required.** "free to start, no key required"; "you can try it right now,
  no account needed." A key is recommended for scale: "it raises your daily budget 10× and
  lets you track your usage."
- **The quota is denominated in dollars, not "credits" of the size claimed.**
  "Every account gets **$1 of API usage per day for free**" (no payment method required),
  budget "resets every midnight UTC"; "Without a key you get $0.10/day."
  So the split is $0.10/day keyless vs $1.00/day keyed — a 10× ratio, not 100 vs 100,000.
- Per-1,000-call rates: single entity **free**; list+filter **$0.10**; search **$1**;
  semantic search **$1**; content download **$10**. The free $1/day therefore buys ~10,000
  list/filter calls, ~1,000 searches, or ~100 content downloads.
- **The error code is 429, not 409.** "Two things return `429 Too Many Requests`: exceeding
  your daily budget, or making more than **100 requests per second**." (Note also the
  per-second cap is 100/s, far above the old 10/s.)
- **The old docs no longer describe the polite pool** — docs.openalex.org paths
  (`/how-to-use-the-api/rate-limits-and-authentication`, `/how-to-use-the-api/api-overview`)
  now 302-redirect to help.openalex.org, and the new authentication page mentions neither
  `mailto` nor a polite pool. So the "its own docs still describe the polite pool"
  contradiction in the claim is stale.
- The "~2026-02-13" changeover date could not be verified; the relevant help pages are
  stamped Aug 11 and Aug 19, 2026.

## 5. SEC EDGAR — CONFIRMED (with one unverified phrase)

- https://www.sec.gov/about/webmaster-frequently-asked-questions — "Our current maximum
  access rate is 10 requests per second." Requires declaring a user agent, sample given as
  `User-Agent: Sample Company Name AdminContact@<sample company domain>.com`.
- https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data —
  "Current max request rate: 10 requests/second"; "Please declare your user agent in request
  headers".
- The quoted phrase **"regardless of the number of machines"** does not appear on either
  page. Neither page frames the limit as per-requester-across-machines. Drop the quotation
  or attribute it as a paraphrase. (Could not run a site search to look for it elsewhere on
  sec.gov — WebSearch budget was exhausted.)

## 6. NCBI E-utilities — CONFIRMED except the "scripting" quotation

Source: https://www.ncbi.nlm.nih.gov/books/NBK25497/ (Chapter 2, Usage Guidelines and
Requirements)

- Without a key: "any site (IP address) posting more than 3 requests per second to the
  E-utilities will receive an error message". CONFIRMED
- With a key: "a site can post up to 10 requests per second by default". CONFIRMED
- "Only one API key is allowed per NCBI account; however, a user may request a new key at
  any time." CONFIRMED
- Large jobs: limit to "either weekends or between 9:00 PM and 5:00 AM Eastern time during
  weekdays". CONFIRMED — and corroborated at
  https://www.ncbi.nlm.nih.gov/home/about/policies/ : "Run retrieval scripts on weekends or
  between 9 pm and 5 am Eastern Time weekdays for any series of more than 100 requests" and
  "Make no more than 3 requests every 1 second."
- **"NCBI does not allow scripting against our web pages" — NOT FOUND.** Absent from
  NBK25497, from NBK25501 (the E-utilities help TOC), and from the NCBI policies page. The
  policies page instead sets conditions for scripted use and directs traffic to
  `https://eutils.ncbi.nlm.nih.gov` "not the standard NCBI Web address". That is a
  don't-script-the-website-use-the-API instruction, which is probably what the quote is
  garbling, but the sentence as quoted is unsourced — paraphrase it.

## 7. arXiv API terms of use — CONFIRMED

Source: https://info.arxiv.org/help/api/tou.html

"make no more than one request every three seconds, and limit requests to a single
connection at a time", and the limits "apply to all of the machines under your control as a
whole." All three elements of the claim are verbatim-correct.

## 8. api.data.gov — CONFIRMED

Source: https://api.data.gov/docs/developer-manual/ (the /docs/rate-limits/ path is folded
into the developer manual)

"Hourly Limit: 1,000 requests per hour". Caveat worth carrying into the report: "Rate limits
may vary by service, but the defaults are…", and higher limits are obtained by contacting
the agency that owns the specific API — so 1,000/hr is the default, not a universal ceiling.

## 9. Scrapy — CONFIRMED

- PyPI JSON API (https://pypi.org/pypi/scrapy/json): `version` = **2.18.0**, first file
  uploaded **2026-08-20T15:15:37Z**. CONFIRMED.
- https://docs.scrapy.org/en/latest/news.html: 2.18.0 dated **August 20, 2026**. It adds
  `scrapy.utils.asyncio.sleep()` "which works both with and without a Twisted reactor" and
  `scrapy.utils.reactorless.uninstall_reactor_import_hook()`, "which `AsyncCrawlerProcess.start()`
  now uses to uninstall the `twisted.internet.reactor` import hook when it exits."
- 2.17.0 (July 7, 2026) added "support for HTTP/2 requests to `HttpxDownloadHandler`" and
  "support for SOCKS proxies to `HttpxDownloadHandler`" — so the httpx-based download handler
  predates 2.17 and was *extended* in 2.17/2.18 rather than introduced then. Minor wording
  fix for the report.
- 2.18.0 also: "`HttpxDownloadHandler` now uses [httpx2](https://httpx2.pydantic.dev/), the
  successor of `httpx`" — independent corroboration for claim 10.
- Twisted-optional operation is real: https://docs.scrapy.org/en/latest/topics/practices.html
  documents `TWISTED_REACTOR_ENABLED = False` with `AsyncCrawlerProcess` starting "an asyncio
  event loop for every spider run", and allows several `AsyncCrawlerProcess` instances per
  process — but flags it: "Running Scrapy without a Twisted reactor is experimental and has
  some limitations." Say "experimental", not "migrated".
- The specific attribution of `AsyncCrawlerProcess` to **2.13** carries no `versionadded`
  note on the practices page; 2.13.0 is dated May 8, 2025 in the news file. Low-stakes, but
  the 2.13 attribution is inferred rather than confirmed.

## 10. httpx / httpx2 — CONFIRMED, with an important correction

PyPI `httpx` (https://pypi.org/pypi/httpx/json):

- Latest **stable** version is indeed **0.28.1**, uploaded **2024-12-06T15:37:21Z**. CONFIRMED.
- **But the project is not dead on PyPI:** it has pre-releases `1.0.dev1` (2025-07-02),
  `1.0.dev2` (2025-08-04), `1.0.dev3` (2025-09-15), `1.0.dev4` (2026-08-19),
  `1.0.dev5` (2026-08-21), `1.0.dev6` (**2026-08-31**). 77 versions total. So "last release
  was 0.28.1 on 2024-12-06" is only true of stable releases — the last upload of any kind was
  six days before this check. Word it as "last stable release".
- Project URLs still point at github.com/encode/httpx.

PyPI `httpx2` (https://pypi.org/pypi/httpx2/json):

- Real package. Latest **2.12.0**, uploaded **2026-08-18T13:22:06Z**. CONFIRMED.
- Summary is identical to httpx's: "The next generation HTTP client."
- Homepage/Source: **https://github.com/pydantic/httpx2**; changelog at
  `pydantic/httpx2/blob/main/src/httpx2/CHANGELOG.md`. Pydantic ownership CONFIRMED.
- 16 versions; the 2.x line moves fast (2.8.0 2026-07-23 → 2.12.0 2026-08-18).
- Repo README states the relationship explicitly: "HTTPX2 is a continuation of the wonderful
  work started by @lovelydinosaur and the broader HTTPX community" and "With HTTPX itself
  seeing limited activity recently, Pydantic is picking up stewardship under the HTTPX2 name
  so that users have a reliably maintained path forward."
- Third-party corroboration from a large downstream: Scrapy 2.18.0 release notes call httpx2
  "the successor of `httpx`".
- Caveat for the report: this is a **third-party continuation under a new name**, not a
  handover of the `httpx` package itself — `httpx` on PyPI is still controlled by encode and
  still publishing 1.0 dev builds. Don't write "httpx was renamed to httpx2."



---

# Adversarial verification pass B — 2026-09-06

Method note: WebSearch budget for this session was exhausted (200/200) early in the run.
All verification below was done by direct WebFetch/curl against primary sources
(vendor pricing pages, docs sites, paper PDFs, govinfo court-opinion PDFs, GitHub docs source).
One sub-claim (GitHub community-discussion quote) could not be reached: github.com HTML is
blocked to curl by egress policy and GitHub's robots.txt disallows the discussions search path.

| # | Verdict | Detail |
|---|---------|--------|
| 1 | CONFIRMED (one date correction) | Uniform > proportional, non-monotonic optimum, >50% .edu/.gov static all confirmed. Study period is **Feb 17 – Jun 24, 1999**, not "~1999–2000". |
| 2 | CONFIRMED | 64-bit simhash, k=3, 8B pages, precision and recall "near 0.75" — verbatim. |
| 3 | SPLIT: docs CONFIRMED (with scope correction), staff quote UNVERIFIED | 60-day auto-disable applies **only to public repositories**. Delay/dropped-job wording confirmed. nebuk89 = Ben De St Paer-Gotch, GitHub staff PM for Actions (confirmed). The ">30% in 2ish months" quote and the 1–3h vs 20–30min figures: not reachable. |
| 4 | CONFIRMED | Credit definition verbatim; $0.040 (Solo, $10/mo base) / $0.035 (Starter, $100/mo base). 1,000 assets hourly = 720,000 credits/mo = **$25,200/mo** at $0.035, **$28,800/mo** at $0.040, plus base fee and serverless compute at $0.010/min. |
| 5 | CONFIRMED | 10 MB warning / 50 MB terminate (paired with 51,200 events); $0.042/GBh active storage; floor wording verbatim "the greater of your monthly plan or 5%-10% of usage". |
| 6 | CONFIRMED (two small corrections) | Healthchecks.io exact. Cronitor: free Hacker = 5 monitors; Business = **$2/monitor/mo (all monitors, not just additional)** + $5/user/mo. Dead Man's Snitch: free = 1, **$19/mo = 100 ("Private Eye")** — but note an intermediate $5/mo "Little Birdy" = 3 snitches sits between them. |
| 7 | CONFIRMED | All three rationales present verbatim on prometheus.io/docs/practices/pushing. |
| 8 | CONFIRMED | AUTOTHROTTLE_ENABLED=False, start 5.0s, max 60.0s, non-200 sentence verbatim. RETRY_TIMES=2; RETRY_HTTP_CODES=[500,502,503,504,522,524,408,429] — 403 and 404 absent. |
| 9 | CONFIRMED | fetcher.server.delay=5.0; db.fetch.schedule.class=DefaultFetchSchedule (Adaptive not default); fetcher.threads.per.queue=1. Heritrix wiki verbatim: "only open a single connection to a server at a time. The current Heritrix frontier always follows this rule." |
| 10 | CONFIRMED (framing corrections on both Bright Data cases) | hiQ: 9th Cir. 2019 + 31 F.4th 1180 (2022-04-18, GVR after Van Buren) confirmed; **2022-11-04** ruling found hiQ breached the User Agreement; consent judgment filed **2022-12-06**, **$500,000**, delete data + destroy code. Meta v. Bright Data: **2024-01-23**, Judge Edward M. Chen — holding is that logged-off scraping is not "use" and **the Terms do not bar logged-off scraping of public data**; it is NOT a "no contract was formed" holding. X Corp. v. Bright Data: preemption holding is in the **2024-05-09** Alsup order, which **dismissed** trespass to chattels for want of harm; the server-impairment trespass theory survived only later, in the **2024-11-26** order re second amended complaint. |

## Sources

1. http://oak.cs.ucla.edu/~cho/papers/cho-tods03.pdf — Theorem 5.1 "It is always true that F(S)u >= F(S)p"; "the synchronization frequency of e5 is zero, while it changes at the highest rate"; "it is better to give up synchronizing the elements that change too fast."
   http://oak.cs.ucla.edu/~cho/papers/cho-evol.pdf — "around 720,000 pages from 270 sites every day, from February 17th through June 24th, 1999"; "more than 50% of pages in those domains did not change at all for 4 months."
2. https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/33026.pdf — "For 8B web pages, 64-bit simhash fingerprints suffice."; "Choosing k = 3 is reasonable because both precision and recall are near 0.75."
3. https://raw.githubusercontent.com/github/docs/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md — "In a public repository, scheduled workflows are automatically disabled when no repository activity has occurred in 60 days."
   https://raw.githubusercontent.com/github/docs/main/data/reusables/actions/schedule-delay.md — "can be delayed during periods of high loads ... some queued jobs may be dropped."
   https://github.com/nebuk89 — GitHub staff, PM for Actions/Packages/Codespaces/npm.
   Not reachable: https://github.com/orgs/community/discussions (robots-disallowed search; curl blocked by egress policy).
4. https://dagster.io/pricing — "A credit is defined as the sum of asset materializations and ops executed."; $0.040/credit Solo, $0.035/credit Starter.
5. https://docs.temporal.io/cloud/limits — 51,200 Events or 10 MB (warning) / 50 MB (max); 2 MB single-request payload; 4 MB gRPC.
   https://temporal.io/pricing — "$0.042 GBh"; "Temporal Cloud Plans charge the greater of your monthly plan or 5%-10% of usage as you scale to maintain your level of support."
6. https://healthchecks.io/pricing/ — Hobbyist $0/20, Supporter $5/20, Business $20/100, Business Plus $80/1000.
   https://cronitor.io/pricing — Hacker free (5 monitors); Business $2/monitor/mo + $5/user/mo; Enterprise from $6,000/yr.
   https://deadmanssnitch.com/plans — Lone Snitch free/1; Little Birdy $5/3; Private Eye $19/100; Surveillance Van $49/300.
7. https://prometheus.io/docs/practices/pushing/ — "single point of failure and a potential bottleneck"; "You lose Prometheus's automatic instance health monitoring via the `up` metric"; "The Pushgateway never forgets series pushed to it".
8. https://docs.scrapy.org/en/latest/topics/autothrottle.html — "latencies of non-200 responses are not allowed to decrease the delay".
   https://docs.scrapy.org/en/latest/topics/downloader-middleware.html — RETRY_TIMES default 2; RETRY_HTTP_CODES default [500, 502, 503, 504, 522, 524, 408, 429].
9. https://raw.githubusercontent.com/apache/nutch/master/conf/nutch-default.xml
   https://github.com/internetarchive/heritrix3/wiki/Politeness%20parameters
10. https://cdn.ca9.uscourts.gov/datastore/opinions/2022/04/18/17-16783.pdf ; https://law.justia.com/cases/federal/appellate-courts/ca9/17-16783/17-16783-2022-04-18.html (31 F.4th 1180)
    https://natlawreview.com/article/hiq-and-linkedin-reach-proposed-settlement-landmark-scraping-case — "Judgment in the amount of $500,000 is entered against hiQ, with all other monetary relief waived."
    https://www.govinfo.gov/content/pkg/USCOURTS-cand-3_23-cv-00077/pdf/USCOURTS-cand-3_23-cv-00077-7.pdf — Meta v. Bright Data, 2024-01-23, Chen: "A 'user' of Facebook and Instagram is one who has an active account"; "the Facebook and Instagram Terms do not bar logged-off scraping of public data."
    https://www.govinfo.gov/content/pkg/USCOURTS-cand-3_23-cv-03698/pdf/USCOURTS-cand-3_23-cv-03698-1.pdf — X Corp. v. Bright Data, 2024-05-09, Alsup (preemption; trespass dismissed for no harm).
    https://www.govinfo.gov/content/pkg/USCOURTS-cand-3_23-cv-03698/pdf/USCOURTS-cand-3_23-cv-03698-3.pdf — 2024-11-26 order: "X Corp.'s motion for leave to amend its trespass to chattels claims is GRANTED."

