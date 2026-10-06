# W2D — UX Simplification Pass (post-v1.2 role split)

> Why this exists: v1.2 added a real permission system (Vendor/Manufacturer)
> on top of an app that was previously flat — everyone saw the same thing.
> That's inherently more complex. Nothing in the UI currently explains the
> split to the user, which is what's reading as "confusing." This is a plan
> to fix the *explaining*, not to redo the visual design.

Created: 2026-08-01

---

## What's actually causing the confusion

| Symptom | Root cause |
|---|---|
| Manufacturer signup feels heavier | Categories multi-select added as a signup step — a "nice for later" field blocking entry to the app |
| Two users compare phones, see different apps | Vendor and Manufacturer see different tabs/Post options with zero explanation of why |
| "Vendor" / "Manufacturer" picker doesn't self-explain | No subtext under the labels — user has to guess what each role lets them do, same mistake `DECISIONS.md` §5 already flagged for sell-new vs catalog and fixed there, not here |
| Posting a requirement feels like a dead end | Vendor gets redirected to their own post once, then has no way back to it (accepted gap when this was designed — now visibly a real complaint) |

## Reference patterns

Same evidence base your own `DECISIONS.md` §5 already uses for the
Available/Needs split — extending it here:

- **IndiaMART**: buyer/seller roles are explained at signup with one line
  each, not just a label picker.
- **Facebook Marketplace**: "For Sale" vs "Wanted" — the split is
  structural, but each surface has a persistent list, never a one-time view.
- **General mobile onboarding practice**: collect only what's needed to
  start using the app; defer anything optional (like categories) to
  Settings, prompted contextually later, not gated at signup.

## Recommended fixes, in priority order

1. **Move categories out of signup** (agreed above) — Profile/Settings only,
   optional, prompted later. Removes friction now, no functionality lost.
2. **Add one line of subtext under each role option** on the signup picker
   — e.g. "Vendor — buy, sell, and post what you need" / "Manufacturer —
   buy, sell, and respond to what vendors need." Same fix pattern already
   used for sell-new vs catalog; just wasn't applied here. Cheap, high
   impact.
3. **Build the cold-start 3-screen intro now, not later.** It was already
   planned (`DECISIONS.md` §15) but sequenced as general polish. Bump it up
   — it's no longer just explaining Available/Needs, it's the only place
   that can explain the two-role split at all. This is the single biggest
   lever for "confusing."
4. **Reconsider the Vendor "dead end" after posting a requirement.** The
   original build-plan decision (redirect to the post's own detail page,
   Phase 2 §2.1) was the cheap option under the assumption it was a minor,
   temporary gap. Now that it's live and reads as confusing in practice,
   worth the small extra scope: a minimal "My Requirements" list
   (Vendor-only, just their own posts) instead of a one-time redirect. Not
   as big as full "My Listings" (roadmap item 5) — just enough that a
   Vendor can find their own post again.

## What this is NOT

Not a visual redesign, not new navigation patterns, not new screens beyond
item 3/4 above. The goal is explaining the existing structure better, not
building more of it.

---

**Needs your call before this becomes BUILD_PLAN tasks:** items 1–2 are
small enough to just do. Item 3 (cold-start intro) and item 4 (My
Requirements list) are real scope additions — say go on both, one of them,
or neither and we leave 4 as the known accepted gap it already was.
