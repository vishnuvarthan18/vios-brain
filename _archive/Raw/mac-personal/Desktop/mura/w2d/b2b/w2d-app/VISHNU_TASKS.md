# W2D — Vishnu-Only Tasks

> Not for Cursor. These are things only you can do — an account action, a
> business decision, or something requiring credentials Cursor doesn't
> have. Nothing here should ever be handed to Cursor as a coding task.

Created: 2026-08-01

---

## Do this first, unrelated to tonight's build — safe to do right now

- [ ] **Git backup.** One commit exists (`Initial commit`, months old) —
  everything built since, including tonight's entire overnight run, exists
  only on this Mac. `git add -A && git commit -m "..."`, push to a private
  GitHub repo. Ten minutes, removes real risk to all of this work. Safe to
  do in a separate terminal window while Cursor is still running — it
  doesn't touch the app code Cursor is editing.

## Before tomorrow's demo

- [ ] **Device walkthrough** (already flagged) — Available/Needs/Post/
  Profile, My Catalog, the three settings rows, sign-in flow. Cursor
  can't test on a device; this is the only real confirmation any of
  tonight's work actually looks right.
- [ ] Review the summary section Cursor writes at the bottom of
  `BUILD_PLAN_v1.2.md` when the overnight run finishes — every judgment
  call it made unattended needs a human sanity check, not just the
  checkboxes.

## Already tracked elsewhere, listed here so nothing gets lost

- [ ] **Meta Business verification + WhatsApp template approval** — start
  this whenever ready; external approval, days-long, blocks real Phase 4.
  (Phase 0 in `BUILD_PLAN_v1.2.md`.)
- [ ] **Publish Privacy Policy to a public URL** (`wedding2day.com` or
  similar) — tonight's Phase D.2 makes it viewable in-app, which is not
  the same as a public URL, which Play Store submission requires
  separately.
- [ ] **Play Store Data Safety form** — filled out in Play Console at
  submission time, not an app feature, nothing for Cursor to build.
- [ ] **Play Store listing assets** — screenshots, feature graphic, short
  and long description, promo text. None of this is code; can be prepared
  in parallel with dev work rather than waiting until submission day.

## Decisions worth making, not urgent tonight

- [ ] **Account recovery policy** — what happens if a user loses access to
  their WhatsApp/phone number? Currently no path beyond a manual support
  request. Decide the approach (manual via support, or something more
  formal) before it's a real support ticket, not after.
- [ ] **Vendor "My Requirements" gap revisit** — currently a Vendor sees
  their own posted requirement only via the one-time redirect after
  posting (Phase 2's accepted workaround). Worth deciding if this needs
  fixing before My Listings (roadmap item 12) gets built properly, or if
  it can wait.

## Business, not app — no code involved

- [ ] Confirm whatever business-registration/tax implications apply to
  running a live trade platform, if not already sorted — outside app scope
  entirely, flagging only because it's easy to deprioritize while heads-down
  on the build.
