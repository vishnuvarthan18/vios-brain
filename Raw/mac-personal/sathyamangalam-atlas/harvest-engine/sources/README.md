# Source registry

Sources are data, not code. Each file in this folder describes one harvest
*stream* (a group of related sources — e.g. all biodiversity APIs, or all
gazetteer inputs). A future stage's harvester Worker reads the relevant file
here at run time; no source URL is ever hardcoded into `src/streams/*.js`.

## File-per-stream

- `sources/<stream-name>.json` — one file per stream.
- `sources/sensitive-species.json` — cross-cutting config (not a stream):
  the list of species whose occurrence coordinates must be coarsened. See
  [../src/lib/coarsen.js](../src/lib/coarsen.js).

## Entry shape

Each stream file is a JSON object with a `stream` name and a `sources` array.
Every entry in `sources` must have:

| field         | meaning                                                              |
|---------------|-----------------------------------------------------------------------|
| `name`        | human-readable identifier, unique within the stream                   |
| `kind`        | `api` \| `pdf` \| `html` \| `archive` — matches `source.kind` in D1    |
| `base_url`    | root URL this source is fetched from                                  |
| `reserves`    | array of reserve slugs this source applies to (or `["*"]` for all)    |
| `license`     | license/terms the data is released under, as stated by the source     |
| `terms_url`   | link to that license or terms-of-use page                             |
| `rate_limit_ms` | minimum ms between requests to this host (see `src/lib/rate-limit.js`; must be ≥1000 for any `.gov.in` host unless `rate_limit_justification` is filled in) |
| `notes`       | anything a future implementer should know before writing the fetcher  |

See `dummy-stream.json` for a filled-out (placeholder) example.

## Adding a real stream in a later task

1. Add `sources/<stream-name>.json` describing every source in that stream.
2. Add `src/streams/<stream-name>.js` exporting a `run(env, reserveSlug)`
   function, following the shape of `src/streams/dummy.js`.
3. Register it in `src/streams/index.js`.
4. Add a cron trigger line in `wrangler.toml` if it needs its own schedule,
   or reuse the existing daily trigger and branch on stream name.
5. Do not add a new Worker per stream — one Worker, many streams, dispatched
   by `stream_name`.
