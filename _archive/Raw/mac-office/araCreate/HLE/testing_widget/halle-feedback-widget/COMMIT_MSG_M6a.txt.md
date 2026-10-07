---
source: office Mac ~/araCreate/HLE/testing_widget/halle-feedback-widget/COMMIT_MSG_M6a.txt
---

feat: add M6a — screenshot storage and signed uploads

Gives the widget a place to put a screenshot: a local-disk store behind an
interface, a self-signed upload url in place of an S3 presign, and an upload
route that verifies everything before a byte reaches the filesystem.

Storage lives in lib/storage/ — put, get, delete, signed_upload_url — with
one implementation today (local disk) and nothing outside the folder ever
importing it directly. A key is only ever reports/<uuid>/<uuid>.webp, the
exact shape build_screenshot_key already produces at report-insert time;
anything else is rejected by construction rather than blocklisted attack by
attack. Upload urls are signed the same way a session cookie is — HMAC-SHA256
over the key and a 5-minute expiry, reusing SESSION_SECRET rather than a
second secret to manage — with the key itself bound into the signature, so a
url issued for one report can never authorise writing to another's. Single
use is enforced by the store, not the signature: local disk opens with
O_CREAT|O_EXCL, so a replayed-but-still-valid url's second write fails
atomically at the filesystem level regardless of its expiry.

Code review before this commit caught two gaps in that design, both fixed.
First: session cookies and upload tokens are both (secret removed) over
base64url(JSON) with the same SESSION_SECRET, and the only thing keeping one
from verifying as the other was that today's payload shapes don't overlap —
not an enforced invariant. Both sign() functions now hash a domain-prefixed
payload (upload:v1:..., session:v1:...) so neither can ever verify as the
other regardless of how either payload evolves; a cross-verification test
added in both directions. Second: put() wrote bytes directly into the final
key path after reserving it with O_EXCL, so a write that died part-way
(dropped connection, disk full, killed process) left a truncated file behind
that get() would serve as a corrupt image, and O_EXCL meant no retry could
ever replace it. put() now writes to a <key>.part file, fsyncs, writes
metadata, then rename()s onto the final path — a reader only ever sees a
complete object or nothing. A .part orphaned by a killed process (no code
path left to clean it up) is reclaimed by put() itself once it's older than
the upload token TTL plus a minute of grace, which is sound only because
MAX_UPLOAD_BYTES caps a body small enough that any live write finishes in
seconds; tested for the killed-mid-write case, the not-yet-stale case, and
the concurrent-reclaim race.

The session:v1: prefix changes the signature input, so cookies issued before
this commit no longer verify and everyone signs in again.

POST /api/v1/reports now returns a populated uploadUrl for the screenshot_key
set at insert time; that column is still never updated afterwards, since the
append-only trigger would refuse it. POST /api/v1/uploads verifies the
signature and expiry, caps the body at 2 MB (checked off Content-Length
before the body is read, and again against the real byte count), and checks
the RIFF/WEBP magic bytes directly rather than trusting Content-Type — every
rejection is a generic error code that writes nothing.

Retention (lib/retention.ts, make retention) reads which screenshots are due
across every project — the same kind of unscoped read find_project_by_
public_key already is — and deletes files only, in bounded batches, never
touching the reports row a key came from. Building the batch loop surfaced
two real findings, both fixed: an offset-based "next page" would have
silently skipped or repeated rows, since nothing can ever mark a row "already
swept" — replaced with a keyset cursor on (created_at, id); and the first
cut aborted a whole batch on one row with an invalid key instead of skipping
and counting it, which would have let one bad row block every valid one
queued behind it.

docs/build-plan.md still describes S3-compatible storage; the task message
given directly for this milestone replaces that with local disk and a
self-signed url, logged as a deviation in the new docs/blocked.md rather than
silently reconciled. M6b — capture, privacy stripping, consent, the widget's
dynamic-import loader — is a separate, reviewed step and has not been
started.
