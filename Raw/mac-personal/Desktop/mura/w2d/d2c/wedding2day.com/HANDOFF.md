# HANDOFF — 2026-07-28

## Tickets completed (this run)

| Ticket | Commit | Summary |
| --- | --- | --- |
| W-035 | `72388c3` | Admin package image CRUD (upload / reorder / delete / set-primary) reusing `src/lib/r2/`; required alt; 5MB MIME/size via existing `image-upload` schemas; public detail gallery renders when `R2_PUBLIC_BASE_URL` is set |
| W-033 | `d6fcc82` | `inquiry_rate_limits` table + migration; 5 inquiries / IP / rolling hour; clear form error; wired only in `performSubmitPackageInquiry` |

**Pre-check:** `git log` had no `W-035` or `W-033` implementation commits (only doc mentions). Proceeded.

**Verification:** After each ticket: `npm run build`, `npm run lint`, `npm test` pass. Final suite: **132 tests**, 33 files.

## Where stopped and why

Queue finished (W-035 → W-033). No further tickets in the prompt. Did not touch `wrangler.toml`. Schema changes limited to the new `inquiry_rate_limits` table (+ seed `clearAll` delete). Did not run wrangler `--remote`, pages deploy, or force-push.

Local migration applied with `npm run db:migrate` (local-only). **Production / remote still needs you to apply `0001_inquiry_rate_limits.sql` when ready** — not done here by design.

## Decisions that weren’t fully specified

1. **W-035 image cap** — Reused `MAX_IMAGES_PER_PACKAGE = 15` already in `src/lib/r2/constants.ts` (same as venues). 5MB confirmed identical to mandaps via `MAX_IMAGE_BYTES` / `requestImageUploadSchema`.
2. **W-035 public render** — Updated `PackageGallery` to mirror `MandapGallery` (real `<img>` when public base URL exists; branded placeholder otherwise). Browse cards still use placeholders — ticket only required the detail page.
3. **W-033 table shape** — One row per IP (`ip` PK, `window_start`, `count`). Window resets when `now - window_start >= 3600`. Simpler than multi-row hourly buckets; matches suggested columns.
4. **W-033 limit** — **5 per hour** as suggested. Three seeded packages in one sitting stay under the limit (covered by test).
5. **W-033 IP source** — `cf-connecting-ip`, then first `x-forwarded-for` hop, then `x-real-ip`, else `"unknown"` (shared bucket for missing headers).
6. **W-033 phone limit** — Not implemented; ticket said “optionally per phone.” IP-only keeps one counter path.

## Possible doc / follow-up notes

- BACKLOG W-033 still says “needs schema migration approval” — approved and done this run; wording can be updated.
- Remote D1 must get migration `0001_inquiry_rate_limits` before production inquiries are rate-limited (table missing would error on submit).
- Uncommitted local noise left alone: `data/`, `docs/PLAN.md`, `docs/go-to-market/`.

## Uncertain / half-finished

- **No live browser upload** of a package image against real R2 (presign mocked in tests, same as W-018).
- Package browse cards still show gradient placeholders, not primary images.
- Rate-limit race under concurrent requests from the same IP is best-effort (read-then-write); D1 has no strong row lock — fine for spam deterrence, not a perfect mutex.
- Optional phone-based limit still open if spam shifts off shared IPs (NATs / mobile carriers).

## What should happen next

1. Apply `0001_inquiry_rate_limits` on staging/production when you deploy (or via W-036 CI).
2. Set `R2_PUBLIC_BASE_URL` (and R2 S3 secrets) so package images display on `/packages/[slug]`.
3. Optionally update browse `PackageCard` to resolve primary image URLs the same way as the detail gallery.
4. Next backlog candidates: W-031 deploy, W-038 (needs approval), W-032 SEO — not started here.
