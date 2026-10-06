# Play Console — Content Rating questionnaire: answers to submit

**For Vishnu to submit manually. I cannot submit this — it is a Play Console
form behind your Google account, and it has legal weight, so it needs a human
to press the button.**

Prepared 2026-08-21 as part of §14 item 2's release-gate work (audit item D5).
Every answer below was checked against the actual code, not assumed. Where the
answer depends on a judgement call rather than a fact, that is said explicitly.

---

## Before you start

Play Console → your app → **Policy → App content → Content rating** → Start
questionnaire.

Two things it asks first:

| Field | Answer | Why |
|---|---|---|
| Email address | the developer contact you use for the listing | See "Open item" at the bottom — this is currently a personal Gmail |
| **Category** | **Utility, Productivity, Communication, or Other** | W2D is a B2B trade tool. It is **not** a "Social" app: there is no chat, no inbox, no feed of user-to-user posts *about each other*, no follows and no comments (`DECISIONS.md` §10, §18 — the social layer is drafted and deliberately not built). Choosing "Social" would pull in a much stricter question set that does not describe this app |

> The category choice drives which questions you see. If your screen does not
> match the sections below, you almost certainly picked a different category —
> go back and change it rather than answering questions that do not apply.

---

## Section-by-section answers

### Violence

| Question | Answer |
|---|---|
| Does the app contain violence? | **No** |
| Realistic violence, blood, or gore? | **No** |
| Violence towards, or death of, humans or animals? | **No** |

There is no game content, no imagery we generate, and no editorial content. The
only images in the app are photos businesses upload of stock and portfolio work.

### Sexuality

| Question | Answer |
|---|---|
| Sexual or suggestive content? | **No** |
| Nudity? | **No** |

### Language

| Question | Answer |
|---|---|
| Profanity or crude humour? | **No** |

The app has no editorial copy of that kind. Users can type free text into a
post title and description — see "User-generated content" below, which is where
that risk is actually declared.

### Controlled substance

| Question | Answer |
|---|---|
| References to, or use of, drugs, alcohol or tobacco? | **No** |

Nothing in the 29 categories (`DECISIONS.md` §9) is a controlled substance. Note
category 16 is "Ice Cream and Beeda" — *beeda*/paan here means the sweet
after-meal betel-leaf preparation served at weddings. If the reviewer queries it,
the honest answer is that we do not verify what any individual business lists,
which is exactly what the user-generated-content declaration below covers.

### Gambling and contests

| Question | Answer |
|---|---|
| Simulated gambling? | **No** |
| Real-money gambling, contests or sweepstakes? | **No** |
| In-app purchases? | **No** |

There are no in-app purchases and no payment surface of any kind
(`DECISIONS.md` §10 — in-app payments are explicitly out of scope; §6 — the app
connects two businesses and they transact off-platform).

### User-generated content and interaction — **the section that matters**

Answer this one carefully. It is the only section where W2D is not a flat "no",
and understating it is the kind of mismatch that gets an app pulled after launch
rather than rejected before it.

| Question | Answer | What it refers to in the app |
|---|---|---|
| Does the app allow users to interact or exchange content with each other? | **Yes** | Businesses post listings and requirements that other businesses see, and respond to each other's posts |
| Can users share their own generated content with other users? | **Yes** | Post titles, descriptions, prices, and up to 3 photos per post; catalog entries; portfolio photos |
| Does the app let users communicate directly? | **No, not inside the app** | There is deliberately **no in-app chat, inbox or messaging** (`DECISIONS.md` §6, §10). The app reveals a phone number and the two businesses talk on phone or WhatsApp, entirely outside W2D. If the form only offers yes/no with no nuance, answer **Yes** — users do end up in contact with each other, and overstating our controls here is the worse error |
| Can users share their location with others? | **No** | Users pick a district from a fixed list of 38. There is no GPS, and the app never requests location permission — verified: no location permission in the merged Android manifest |
| Can users share personal information with each other? | **Yes** | A phone number, business name and district. This is the app's core mechanic, not a side effect |
| Is the content moderated? | **Yes** | Every new post is `status: 'pending'` and does not appear in any feed until an admin approves it (`w2d-admin`'s moderation queue). Users can report a post, and reports go to a manual admin queue (§15). Admins can suspend an account, which is enforced in security rules, not just the UI |
| Is there a way for users to report content? | **Yes** | Report button on every post, with a reason; plus block-a-business |
| Is there a way for users to block other users? | **Yes** | The `blocks` collection, user-initiated (§15) |

**Do not claim automated filtering or AI moderation.** There is none.
Moderation is a human approving a queue (§15: "Manual, no committed SLA"). Say
"manually reviewed before publication" — which is both true and stronger than
most apps can claim.

### Miscellaneous

| Question | Answer |
|---|---|
| Does the app share user location with third parties? | **No** |
| Does the app contain ads? | **No** — there is no ad SDK in the dependency tree |
| Is the app a news app? | **No** |
| Does the app promote or sell restricted products? | **No** |

---

## Target audience and content

A separate form from the rating questionnaire, and it is the one that must say
18+.

| Field | Answer | Why |
|---|---|---|
| Target age group | **18 and over, only** | ToS section 2 requires users to be 18+ and running a genuine wedding-trade business. Privacy Policy: "not intended for children, and we do not knowingly collect data from anyone under 18" |
| Could the app appeal to children? | **No** | It is a wholesale sourcing tool. There is no game content, no cartoon styling, no child-directed feature |
| Do you want the app in the "Designed for Families" programme? | **No** | Deliberately. Opting in would subject a B2B app that exchanges adults' phone numbers to Families policy, which it should not be in, and would be a mismatch with the 18+ audience above |
| Does the app collect data from children? | **No** |

Answering "18 and over" here is what keeps the app clear of Families policy
entirely. Do not be tempted to widen the audience for reach.

**Expected outcome:** the questionnaire should return a rating in the
"Rated for 3+ / Everyone" family on *content*, while the target-audience form
holds the app at 18+ on *audience*. Those two are not in conflict — the first
describes what the app depicts and the second who it is for. If the content
rating comes back higher than "Teen", re-read your answers in the UGC section;
something has probably been over-declared.

---

## Cross-checks I ran before writing this

Not assumptions — each of these was checked in the code.

| Claim | How it was checked | Result |
|---|---|---|
| No ads | grepped the dependency tree for an ad SDK | none |
| No in-app purchases or payments | no billing dependency; no payment code path | confirmed |
| No location access | read the merged `AndroidManifest.xml` after `expo prebuild` | no location permission of any kind |
| No contacts, SMS or calendar access | same | none present |
| Posts are moderated before publication | `createListing` writes `status: 'pending'`; `fetchListings` returns only `approved` | confirmed |
| Reporting exists | `reportListing` + the `reports` collection + the admin queue | confirmed |
| Blocking exists | the `blocks` collection and its security rules | confirmed |
| 18+ stated in-app | ToS section 2 and the Privacy Policy preamble | confirmed |
| Personal information is exchanged between users | the reveal mechanic (§6) is the app's core loop | confirmed — declared as Yes |

---

## Permissions, as they will actually ship

Read from the generated `AndroidManifest.xml` after `npx expo prebuild -p
android` on 2026-08-21, rather than from what we think we asked for. The Play
Console asks you to justify what ships.

| Permission | Ships? | Justification if asked |
|---|---|---|
| `INTERNET` | yes | Core function |
| `POST_NOTIFICATIONS` | yes | Alerting a business when someone responds to its post, and when a pending post is approved |
| `CAMERA` | yes — merged in from `expo-image-picker`'s own manifest, which is why it is not listed in `app.json` | Photographing stock, catalog items and portfolio work. Used in three screens, always after an explicit tap |
| `READ_EXTERNAL_STORAGE` | yes | Choosing an existing photo. Superseded by the system photo picker on API 33+ |
| `VIBRATE` | yes | Notification vibration |
| `WRITE_EXTERNAL_STORAGE` | **NO — removed 2026-08-21** | The app never writes to shared storage. Stripped with `tools:node="remove"` via `expo.android.blockedPermissions`, and verified present in the generated manifest |
| `SYSTEM_ALERT_WINDOW` | **NO — removed 2026-08-21** | See the note below |
| `RECORD_AUDIO` | **NO — removed 2026-08-21** | See the note below |
| Location, Contacts, SMS | never present | Nothing requests them |

**About the last two.** Neither was ours. Expo's prebuild template seeds
`SYSTEM_ALERT_WINDOW`, `RECORD_AUDIO`, `READ_EXTERNAL_STORAGE` and
`WRITE_EXTERNAL_STORAGE` into the app manifest as "optional permissions, remove
whatever you do not need" — and nobody had removed them. The app records no
audio and draws no overlay, and `SYSTEM_ALERT_WINDOW` ("Display over other
apps") is one of the permissions Play scrutinises hardest. Both are now blocked.

> ⚠️ **One thing to sanity-check on your next dev build.** React Native's own
> *debug* manifest declares `SYSTEM_ALERT_WINDOW` separately, for the dev
> overlay. Build-type manifests outrank the main manifest in Android's merger,
> so a debug build should keep it while release builds do not — which is exactly
> what we want. If the dev menu misbehaves after the rebuild, the revert is one
> line: drop `"android.permission.SYSTEM_ALERT_WINDOW"` from
> `expo.android.blockedPermissions` in `app.json`. Release builds are the ones
> Play sees, so do not leave it blocked-and-broken; either it works or you
> revert it.

---

## Open item this document cannot close

**The developer contact address is still a personal Gmail**
(`(removed)`), and it appears in the ToS section 14, the Privacy
Policy section 11, and `app/(auth)/_lib/support.ts`. Whatever you put in the
Play Console listing becomes publicly visible next to the app. The support
WhatsApp number in that same file also still carries a `TODO`.

This is a decision, not a code change — it needs an inbox and a number that
exist. When you have them, all three places change together, and the two legal
documents regenerate with `npm run docs:legal`.

---

## Everything else on the submission list

This document covers the content rating and target audience only. The rest of
the release-gate list, from the 2026-08-19 Play Store audit, is unchanged and
still needs you:

1. **A reviewer sign-in path.** Every screen except `/p/[slug]` is behind
   phone-OTP auth, and a Google reviewer has no Indian mobile number. Configure
   a Firebase Auth test phone number with a fixed OTP, then fill in Play Console
   → App access → "All functionality is restricted" with that number, the code
   and a one-line walkthrough. This is the most commonly fatal item on the list
   and nothing in the repo can do it for you.
2. **A hosted Privacy Policy URL**, and an account-deletion request URL. The
   text is ready — `docs/PRIVACY_POLICY.md` is generated from the same source as
   the in-app screen, so it cannot drift from what users see. It needs somewhere
   to live. One hosting decision solves this, the deletion URL, and §6a's real
   browser URL for public profiles.
3. **Store assets**: 512×512 store icon, 1024×500 feature graphic, at least two
   phone screenshots.
4. **The Data Safety form.** The inventory is in the 2026-08-19 notes under Task
   16 §C, and the Privacy Policy rewritten on 2026-08-20 now matches it — note
   in particular that phone number, business name, district, category, role and
   portfolio photos are **publicly accessible with no account required** for any
   business that creates a public profile (§6a). That is not "shared with third
   parties"; it is "made public", and understating it is the mismatch that gets
   an app pulled after launch rather than rejected before it.
5. **The support contact**, per the open item above.
