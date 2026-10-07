# Agent task — consolidated open items, 9 September

Hand this whole document to the build agent as one batch. It lists every
piece of outstanding code/server work known right now. Do the items in the
order listed — item 1 is the most urgent (it blocks the live end-to-end
test). Commit each item separately, with its own clear commit message.
**Do not push anywhere — commit locally only, same as always, until
Vishnu gives explicit word to push.**

If anything below turns out to already be done (check the repo first,
the repo wins over any doc claim), say so instead of redoing it, and
move to the next item.

---

## 1. Fix screenshot capture failing on real pages (most urgent)

Full detail already written up: see `claude/agent-task-screenshot-capture-bug.md`
in the project (copy below for convenience).

**Summary:** Live-tested on `halle-dev.webflow.io`. Every screenshot
attempt (both "Point at the problem" and "Screenshot" modes) fires over
1,170 separate fetch requests — every image/icon/font anywhere on the
whole page, not just what's visible on screen. It's slow, and it ends
with no picture attached at all ("(no picture)" in the report box), with
no error anywhere (matches the existing fail-silently-on-capture rule,
but the capture itself needs to work).

**Likely cause:** the capture step isn't scoped to the visible viewport —
it's walking the whole page DOM instead. Per the v2 spec (M6b:
"visible-window WebP"), it should only capture what's on screen.

**What to do:**
1. Confirm the capture is scoped to the visible viewport only.
2. Find why it's iterating every image on the page instead of only
   elements inside the visible capture region, and fix it.
3. Find the actual failure point after the fetches (or add a sane
   timeout) so a real page with many images doesn't silently fail.
   Consider an internal-only debug log (not shown to the tester) so
   future failures are traceable.
4. Test against a real marketing page with many images, not just local
   fixtures — that's what exposed this.
5. Confirm both "Point at the problem" and "Screenshot" modes work after
   the fix.

**Do not** remove the "never block a report on a failed screenshot" rule
— keep it. The fix is making capture succeed on a normal page.

---

## 2. Confirm (or re-run) the CSV token strip + 180-day retention fix

This was written up and handed over on 9 September
(`claude/agent-task-csv-retention-fix.md`) but there has never been a
report back confirming it was actually run. Check the repo first:

- Does the CSV export function strip the `t` query parameter from
  `page_url` before writing rows? Test: a `page_url` with
  `?ref=x&t=abc123&foo=bar` should export with `t` removed, others kept.
- Is the picture retention sweep set to 180 days (not 90)? Check the
  sweep test suite: a picture at 179 days should survive, one at 181
  days should be swept.

If both are already correct in the repo, say so — nothing to do. If
either is missing, implement it exactly as specified in
`claude/agent-task-csv-retention-fix.md` and confirm with tests.

---

## 3. Sync live server fixes back into the repo's `deploy/` files

Two sets of fixes exist only on the live server right now, never copied
back into the repo:

**a. The `AF_NETLINK` systemd fix** — long-standing, still not synced
into `deploy/halle-feedback.service`. (Ask Claude/Vishnu for the exact
fix that was applied on the server if it isn't already recorded
somewhere in the repo's history — check `claude/server-deployment-plan.md`
for the detail if needed.)

**b. The Apache vhost corrections from the HTTPS work** (9 September),
into `deploy/apache-halle-feedback-optional.conf`:
   - certbot copies `RequestHeader set X-Forwarded-Proto "http"` /
     `X-Forwarded-Port "80"` into the SSL vhost it generates, unchanged.
     Add a clear comment in the template warning that these must be
     hand-corrected to `"https"` / `"443"` after running certbot — this
     is what the admin login cookie depends on.
   - certbot does not add an http→https redirect. Add one to the `:80`
     vhost block in the template: `Redirect permanent /
     https://FEEDBACK_DOMAIN_HERE/`, with that vhost's `ProxyPass` /
     `ProxyPassReverse` lines commented out so there's no ambiguity about
     which directive wins once the redirect is added.

Full detail and exact wording of what was done live:
`claude/https-domain-live-record.md` (steps 5–6).

---

## 4. Set up backups, log rotation, and the scheduled screenshot-retention job

None of these exist yet on the live server:

- **Database backups** — a scheduled job (e.g. cron, or systemd timer)
  that dumps the production Postgres database on a regular schedule and
  keeps a reasonable number of recent backups.
- **Log rotation** — the app's logs (systemd journal or file logs,
  whichever this app uses) should not grow unbounded on the server.
- **Scheduled picture-retention sweep** — the 180-day retention logic
  (see item 2) needs to actually run on a schedule in production, not
  just exist as code. Confirm there's a cron/systemd timer wired up to
  run it, and add one if there isn't.

Write these as deploy artifacts in `deploy/` (a cron file, a systemd
timer unit, or whatever fits the existing deploy pattern) so they can be
installed the same way the rest of the server was set up, and document
the install step in `deploy/runbook.md`.

---

## Report back

For each of the 4 items above, say: already done / fixed now / blocked
on something, with a one-line reason. Do not commit anything until each
item's own fix is verified working (tests passing, or a manual check
where there's no test). Do not push to GitHub — that always needs
Vishnu's own explicit word at the time.
