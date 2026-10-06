# Harvest Engine

The backend harvester for the [Sathyamangalam Record](../README.md). Pulls
data from external sources (APIs, PDFs, archives) into D1/R2 with full
provenance, then later stages read from here to build out the public
record. Runs entirely on Cloudflare's free tier: D1, R2, Workers, Cron
Triggers, and Queues.

This is a **separate database from `data/atlas.db`** (the SQLite file the
public site reads from). Harvest Engine is the write/staging side; a future
stage is responsible for reviewing and syncing rows from here into
`atlas.db`. Nothing in this folder touches the site or `atlas.db` directly.

Designed as a template: nothing in the schema or application code hardcodes
"Sathyamangalam" except the one seed row in the initial migration. A second
reserve is a second row in the `reserve` table, not a fork of this codebase.

## Stage 0 (Foundation) — what's here

- D1 schema (`migrations/0001_init.sql`): `reserve`, `source`, `place`,
  `taxon`, `occurrence`, `document`, `historical_passage`,
  `legal_instrument`, `news_event`, `observation_layer`, `claim`, `job_run`.
  Seeded with one `reserve` row for Sathyamangalam Tiger Reserve.
- R2-backed raw storage (`src/lib/raw-storage.js`) — `saveRaw()` is the one
  function every future harvester calls to store an unmodified fetch,
  content-addressed by SHA-256.
- Shared helpers used by every future source, not reimplemented per source:
  - `src/lib/robots.js` — robots.txt checker + honest User-Agent builder.
  - `src/lib/rate-limit.js` — per-host minimum request interval (1 req/s
    default, enforced floor for any `.gov.in` host).
  - `src/lib/hash.js` — request/content hashing and dedupe-by-hash lookup.
  - `src/lib/coarsen.js` — the ~5km coordinate coarsening math for
    sensitive species (the real enforcement is a D1 trigger — see below).
  - `src/lib/job-run.js` — `job_run` bookkeeping + the per-stream error
    boundary (`withJobRun`).
- One working dummy stream (`src/streams/dummy.js`) that fetches
  `https://example.com/` — a safe, trivial, real fetch — to prove the whole
  pipeline: fetch → robots check → hash check → save raw to R2 → write a
  `source` row → write a `job_run` row → show up in the dashboard.
- A source registry structure (`sources/*.json`) — sources are data, not
  code. See `sources/README.md`.
- A password-protected dashboard (`src/index.js` + `dashboard/render.js`)
  showing job run history, a "run now" button per stream, and live
  per-table row counts.

### Why sensitive coordinates are safe by construction

Rule: no occurrence for a sensitive species (tiger, leopard, elephant,
pangolin, vulture, star tortoise, sandalwood — see
`sources/sensitive-species.json`, not hardcoded in application code) may
ever expose full-precision coordinates automatically.

This is enforced with SQLite triggers on the `occurrence` table
(`trg_occurrence_coarsen_insert`, `trg_occurrence_coarsen_update`), not by
application code remembering to call a coarsening function. Every INSERT
or UPDATE that touches `occurrence` is intercepted by D1 itself: if the
row's `taxon_id` points to a taxon with `is_sensitive = 1`, `public_lat` /
`public_lon` are overwritten with values snapped to a ~5km grid
(`ROUND(lat / 0.045) * 0.045`) and `public_precision_m` is forced to 5000 —
regardless of which harvester, migration, or manual `INSERT` wrote the row.
Non-sensitive species get a passthrough copy into the same public_* columns
so downstream readers never need an "is this coarsened?" branch — they
always read `public_lat`/`public_lon`, never `lat`/`lon`, when building
anything public-facing.

This was verified directly against a local D1/SQLite instance during Stage
0 development (insert a tiger occurrence → coarsened; insert a bird
occurrence → exact; re-point an existing occurrence to a sensitive taxon →
retroactively coarsened).

### What's deliberately NOT in Stage 0

- No real data harvesters — biodiversity, gazetteer, literature, legal,
  news streams are separate, later stages, one at a time.
- No Workers AI / OCR integration — `confidence` and `derived_by` columns
  exist on `claim` and `observation_layer` so this isn't awkward to add
  later, but nothing calls Workers AI yet.
- No public-facing API for external researchers.

## Stage 1 (Biodiversity streams) — what's here

Three real occurrence-data streams, built directly on Stage 0's shared
libs — no new infrastructure, same registry/dispatch pattern:

- `src/streams/gbif.js` — GBIF Occurrence Search API. Public, no auth.
  Scoped to each reserve's bbox via a WKT polygon. Paginates up to
  `MAX_PAGES_PER_RUN` (20) pages of 300 records per run; already-fetched
  pages are skipped by request hash on the next run, so a reserve with more
  occurrences than one run covers is picked up incrementally over several
  scheduled runs.
- `src/streams/inaturalist.js` — iNaturalist API v1 observations search.
  Public, no auth. Scoped by bbox (no iNaturalist place exists for this
  reserve). Uses small pages (30 records) — iNaturalist's own max (200) was
  measured taking 70+ seconds per page against this reserve's bbox due to
  embedded photo/identification metadata, too slow to page through
  reliably. `sources/inaturalist.json` documents why its otherwise-blanket
  `Disallow: /*?` robots.txt is treated as superseded by iNaturalist's own
  published, rate-limited API terms for this exact access pattern.
- `src/streams/ebird.js` — eBird API 2.0 recent-observations-by-geo.
  Requires `env.EBIRD_API_KEY` (free key from ebird.org/api/keygen) via the
  `X-eBirdApiToken` header. eBird has no bbox endpoint — the reserve's bbox
  is converted to a covering center point + radius at run time.
  `sources/ebird.json` documents why its blanket `Disallow: /` robots.txt is
  treated as superseded by eBird's own key-gated API Terms of Use.
- `src/lib/taxon.js` — shared `findOrCreateTaxon()`, used by all three
  streams: looks up or inserts a `taxon` row and sets `is_sensitive` from
  `sources/sensitive-species.json` by scientific name. Occurrence coordinate
  coarsening (the D1 trigger) reads that flag, so every stream that can
  produce occurrences must create taxa through this helper.
- `src/lib/fetch-timeout.js` — `fetchWithTimeout()`, a thin `AbortController`
  wrapper used by every external fetch (including the robots.txt check
  itself). Guards rule #4: a hung upstream host must fail a `job_run` like
  any other fetch error, not leave it stuck at `status = 'running'` forever.

All three were run against their live APIs during development (not
mocked): real `job_run` rows, real R2 raw copies, real occurrence rows with
full provenance, and — since GBIF and iNaturalist both turned up real
sensitive-species records for this reserve during that run (Indian Vulture,
Sandalwood) — real confirmation that the Stage 0 coarsening trigger fires
correctly on live data, not just a synthetic test row.

## Deploying to a Cloudflare account

Requires a Cloudflare account (free tier is sufficient) and Node.js.

```bash
cd harvest-engine
npm install

# Authenticate wrangler against your Cloudflare account
npx wrangler login

# Create the D1 database, then paste the returned database_id into
# wrangler.toml (replacing REPLACE_WITH_D1_DATABASE_ID)
npm run db:create

# Create the R2 bucket and both queues
npm run r2:create
npm run queue:create

# Apply the schema
npm run db:migrate:remote

# Set secrets (you'll be prompted for each value)
npx wrangler secret put DASHBOARD_PASSWORD
npx wrangler secret put SESSION_SECRET      # any long random string
npx wrangler secret put CONTACT_EMAIL       # a real address — used in the bot's User-Agent

# Deploy
npm run deploy
```

After deploying, visit the Worker's URL, log in with
`DASHBOARD_PASSWORD`, and click "Run now" next to the `dummy` stream to
confirm the pipeline works against your account's live D1/R2/Queues before
building a real stream on top of it.

### Local development

```bash
cp .dev.vars.example .dev.vars   # fill in local-only secret values
npm run db:migrate:local
npx wrangler dev --local
```

`wrangler dev --local` simulates D1, R2, and Queues entirely on your
machine — no Cloudflare account calls are made, and nothing here touches
production data. `.dev.vars` is gitignored; never commit it.

## Adding a new harvester stream (for later stages)

Stage 0 ships one dummy stream. A real stream — biodiversity, gazetteer,
literature, legal, news — follows the same shape:

1. **Describe the sources as data.** Add `sources/<stream-name>.json`
   listing every source in the stream (name, base URL, kind, license,
   rate limit, which reserves it applies to). See `sources/README.md` for
   the exact shape and `sources/dummy-stream.json` for a filled example.
   Do not put URLs directly in JS files.
2. **Write the stream function.** Add `src/streams/<stream-name>.js`
   exporting `run(env, reserveSlug)`, modeled on `src/streams/dummy.js`:
   load the registry file, and for each source — check robots.txt
   (`checkRobotsAllowed`), respect rate limits (`RateLimiter`), compute a
   request hash and skip if already processed (`requestHash` +
   `findExistingSource`), fetch, save the raw response with `saveRaw()`
   *before* parsing anything, then write a `source` row and whatever
   content rows the parsed data produces — each with `source_id` and
   `retrieved_at` set. Wrap the whole thing in `withJobRun()` so one
   source's failure can't take down the others or crash the run (rule #4
   in the project brief).
3. **Register it.** Add one line to the `STREAMS` map in
   `src/streams/index.js`. Nothing else needs to change — the dashboard,
   the queue consumer, and the cron dispatcher all iterate that map.
4. **Sensitive species.** If the stream can produce `occurrence` rows,
   make sure the `taxon.is_sensitive` flag is set correctly against
   `sources/sensitive-species.json` before the occurrence is inserted —
   the D1 trigger handles the coordinate coarsening itself, but it can only
   act on whatever `is_sensitive` says.
5. **Disputed facts.** If a stream extracts a fact that might conflict with
   one already in `claim` (e.g. two sources disagreeing on a reserve's core
   area), insert a new `claim` row — never update or overwrite an existing
   one, and never average or pick a "winner." Leave `conflicts_with_claim_id`
   for a later review stage to annotate, if useful.
6. **Give it a schedule.** The existing daily cron trigger already
   dispatches every registered stream for every reserve via the queue —
   a new stream doesn't need its own trigger unless it needs a different
   cadence, in which case add a new `crons` entry in `wrangler.toml` and
   branch on it in the `scheduled()` handler.

## Repo layout

```
harvest-engine/
├── migrations/         D1 schema migrations (0001_init.sql = Stage 0 schema + seed)
├── src/
│   ├── index.js         Worker entrypoint: fetch (dashboard/login/API), scheduled (cron), queue (consumer)
│   ├── queue-consumer.js  Processes enqueued stream runs
│   ├── lib/              Shared helpers: raw storage, robots, rate limiting, hashing, coarsening, job_run, auth
│   └── streams/           One file per harvest stream + the dispatch table
├── sources/             Source registry (data, not code) + sensitive-species config
├── dashboard/           Dashboard HTML rendering
├── wrangler.toml        Cloudflare bindings: D1, R2, Queues, Cron
└── package.json          wrangler dev/deploy/migration scripts
```
