# W2D — Dev Agent Prompts, Batch 1 v2 (Foundation work — Tasks 1–5)

**This replaces the earlier Batch 1** (sent before today's repo/doc
reconciliation — that batch contained inaccuracies the dev agent correctly
flagged and refused). This version was generated directly against the
corrected, reconciled `DECISIONS.md` (last updated 2026-08-19, now also on
GitHub at `https://github.com/vishnuvarthan18/wedding2day-app`), verified
against the actual current code via an independent audit pass.

**How to use this:** paste each task's prompt into your VS Code AI coding
agent ONE AT A TIME, in order. Do not paste multiple tasks at once. Review
the agent's output/diff before accepting, and before starting the next
task. Each prompt assumes the agent has NOT seen this document — it
re-states enough context to stand alone.

**Fixed scope reminder for you (the PM):** these 5 tasks build the
foundation only — the category migration, security-rules fix, and the
public profile layer's data model + route. They do NOT yet touch UI polish
for existing screens beyond what's needed to not break, nor the admin
dashboard beyond what item 5 covers. This matches `DECISIONS.md` §14's
"Item 0: Foundation work," sequenced first, before anything else in the
roadmap. Do not let the agent jump ahead into later roadmap items even if
it offers to.

**Before Task 1:** make sure the agent's working copy is up to date with
the GitHub remote (`git pull origin main`) so it's reading the current
`DECISIONS.md`, not a stale local copy.

---

## Task 1 — Read project memory, confirm understanding, no code changes yet

```
Before writing any code, read these two files in full from the project
root: DECISIONS.md and PRODUCT_CONTEXT.md. Also run `git log --oneline -5`
so you know this repo has real GitHub history now
(https://github.com/vishnuvarthan18/wedding2day-app) — DECISIONS.md is
tracked there and this is the canonical version.

This project went through a reconciliation pass today (2026-08-19) —
DECISIONS.md §0 documents exactly what changed and why. Read §0 first, then
the rest of the file. This file is the current source of truth and
supersedes anything you might infer from existing code patterns. If
existing code in this repo contradicts DECISIONS.md, DECISIONS.md is right
and the code is what needs to change, not the other way around — UNLESS
you find a specific place where DECISIONS.md's claim doesn't match what's
actually in the code, in which case stop and tell me the discrepancy rather
than picking one side to trust. This exact situation happened once already
today (see §0, §19 point 7) — a previous version of this file contained
inaccurate claims that were caught by an AI agent's own verification, which
is exactly the kind of check I want you to keep doing.

Do not write or modify any code in this task. Instead, reply with:
1. A one-paragraph summary in your own words of what changed today (§0)
   and why (see PRODUCT_CONTEXT.md §0 for the founder-interview reasoning)
2. A list of every file/folder in the current codebase that references:
   the `userType` field, the old 10-item decor category constant (likely
   in a file like `constants/listings.ts` — confirm the actual path), the
   `users` collection, the `listings` collection, and `firestore.rules` —
   I need to know what currently exists before we touch it
3. Confirm whether a `profiles` collection, route, or any public-facing
   (no-auth) screen already exists anywhere in the codebase — I expect the
   answer is no, per DECISIONS.md §6a, but verify rather than assume
4. Confirm whether `firestore.rules` still contains the Vendor-only
   requirement-creation gate (DECISIONS.md §7 references it as being around
   line 80) — quote the actual rule text you find

Do not make assumptions about anything marked DRAFT in DECISIONS.md (§18).
If you're unsure whether something is in scope for this task, ask rather
than guessing.
```

---

## Task 2 — Migrate the category model (schema + constants, no UI screens yet)

```
Read DECISIONS.md in full before starting, specifically §3, §7, and §9 —
this task depends on getting those exactly right. If you already read
DECISIONS.md in a prior session today, re-read these three sections now
since this task implements them directly.

Your task: replace the old `userType` field (2-value: vendor/manufacturer)
and the old 10-item decor-only category constant with the single 29-item
`category` field defined in DECISIONS.md §9.

1. Find the existing category constant (likely `constants/listings.ts` or
   similar — confirm the actual file from your Task 1 findings) and add
   the new 29-item list from DECISIONS.md §9 as a new exported constant
   (e.g. `CATEGORIES` or similar — use a name that doesn't collide with
   the old one during migration). Use the exact category names listed
   there, in the exact order, verbatim — do not paraphrase, reorder, or
   "clean up" the names (e.g. keep "Banana Tree" and "Green Panthal"
   exactly as written, even if they look unusual to you).
2. This `category` field applies to BOTH: (a) a business's `users`
   document, and (b) individual `listings` documents (as `sellerCategory`,
   replacing the old `sellerUserType` — see DECISIONS.md §3). Confirm both
   use the same fixed list — do not create two different category enums.
3. Do NOT restrict which categories can post which `postType`. Every
   category must be equally able to post/browse listings — this was a
   deliberate decision (DECISIONS.md §7, §4), not an oversight. Do not add
   any validation that limits listing creation by category.
4. Keep the `listings` collection name unchanged — do not rename it to
   `posts` or anything else (DECISIONS.md §3, §12 — rejected twice now,
   for reasons that still apply).
5. Keep the `requirement` post type intact within `postType` — do not
   remove or weaken it (DECISIONS.md §4).
6. Do NOT remove the old `userType` field from existing documents yet —
   this task adds the new `category` field alongside it. A full migration
   (existing accounts prompted to pick a category) is a separate,
   later task — do not build the migration UI/prompt in this task.

Do NOT yet build the public profile entity — that's Task 4, a separate
step. This task is schema/constants only for the category consolidation.

Do NOT touch any UI/screen code in this task, except where a screen would
otherwise fail to compile/run because it references a constant you renamed
or removed. If a screen currently references the old `userType` field or
old category list and will need real UI rework later (not just a rename),
list those files at the end of your response as "screens that will need
updating in a later task" — do not build that UI now.

Show me your exact plan (which files, what specific changes) before
running anything. If you're unsure how to handle any existing seed/test
data with the old field format, stop and ask rather than guessing at a
migration strategy — DECISIONS.md §3 explicitly says not to guess a
mapping from `vendor`/`manufacturer` to a specific category.
```

---

## Task 3 — Update `firestore.rules`: remove the Vendor-only requirement gate

```
Read DECISIONS.md §7 before starting — this task implements one specific,
concrete change flagged there as "a real code change, not just a doc
update."

Your task: find the rule in `firestore.rules` (DECISIONS.md §7 says it's
around line 80, confirm the actual location) that gates `requirement`
post creation to accounts where `userType == 'vendor'`. Remove or change
this rule so that ANY authenticated user, regardless of `userType` or the
new `category` field (from Task 2), can create a `requirement` post.

1. Show me the exact rule as it currently exists, and your proposed
   replacement, before applying the change — security rules are easy to
   get subtly wrong, so I want to review the diff specifically.
2. Do not touch any other rules in this file unless they also reference
   the old Vendor/Manufacturer gating logic — if you find other rules
   that reference `userType` for permission purposes (not just as a
   readable field), flag them and ask before changing them, since
   DECISIONS.md doesn't explicitly cover every possible instance.
3. After changing the rule, if the Firebase emulator is set up in this
   project (DECISIONS.md §13 mentions ports Auth 9099 / Firestore 8080 /
   Storage 9199 / UI 4000), tell me how to verify the rule change works
   (e.g. via the emulator's rules playground or a quick test script) —
   don't just assume it's correct without a way to check.

This task does NOT touch the category migration from Task 2 beyond
referencing the field name — if Task 2 isn't done yet, stop and tell me
rather than proceeding on an assumption of what the schema looks like.
```

---

## Task 4 — Build the `profiles` collection + public security rules

```
Read DECISIONS.md §6a in full before starting — this task implements the
"public profile / showcase layer" described there. Also read the note in
§6a confirming (as of 2026-08-19) that the public profile shows identity,
contact, and portfolio photos ONLY — catalog is explicitly excluded.

Your task: create the data model and Firestore security rules for a new
public profile entity, per DECISIONS.md:

1. Every registered business gets one public profile document in a new
   `profiles` collection (DECISIONS.md §3). Fields needed (at minimum):
   business name, category (using the same 29-item list from Task 2 —
   reuse it, do not create a separate list), district, phone/WhatsApp
   contact, a photo/portfolio array (photos only — no catalog items, no
   pricing, no product data), and a shareable link/slug or ID usable in a
   public URL.
2. CRITICAL: contact info (phone/WhatsApp) must be readable by
   UNAUTHENTICATED users. This is a deliberate decision (DECISIONS.md
   §6a) — the public profile is NOT reveal-gated like the trade-side
   listings. Anyone with the link should see the phone number
   immediately, no login, no tap-to-reveal. Write the Firestore security
   rules to explicitly allow public read access to profile documents
   (the fields listed above), while keeping write access restricted to
   the profile's own authenticated owner.
3. Do NOT apply this same public-read pattern to the `listings` collection
   or the `catalogItems` subcollection — both stay behind the existing
   auth-gated rules, unchanged. This was explicitly confirmed today
   (DECISIONS.md §6a, §17 change log) — catalog is NOT part of the public
   profile. If you're unsure whether a field belongs to the public
   profile or the trade-side data, ask rather than assuming.
4. Before writing final security rules, show me a plain-English summary of
   exactly what an unauthenticated user CAN and CANNOT read/write under
   your proposed rules, so I can confirm before you finalize them.
   Security rules are hard to get right and easy to get wrong in a way
   that's not obvious from the code — I want to sanity check the summary,
   not just the rules syntax.

Do not build the actual profile SCREEN/UI yet — that's a separate task,
not in this batch. This task is data model + security rules only.
```

---

## Task 5 — Update `w2d-admin` for the category migration

```
Read DECISIONS.md §11 before starting — this task covers the admin
dashboard's side of the category migration from Task 2.

Prerequisite: Task 2 (category model migration in the main app) should be
done first. If it isn't, stop and tell me rather than building on an
assumption of what the schema looks like.

Your task: the `w2d-admin` web dashboard (Vite/React, same Firebase
project) currently has `userType`-dependent code (2-value
Vendor/Manufacturer), per DECISIONS.md §11. Update it to work with the new
`category` field (the same 29-item list from Task 2/DECISIONS.md §9)
instead:

1. Find every place in `w2d-admin` that reads or displays `userType` —
   check the Users screen specifically, and any filtering/sorting logic
   that assumes a 2-value field.
2. Update those to read/display the new `category` field. Where the UI
   shows a dropdown/filter for userType, replace it with one for the
   29-item category list — reuse the same constant from Task 2 if
   `w2d-admin` can import it, or recreate it verbatim if it's a separate
   codebase with no shared package (confirm which is the case before
   assuming).
3. Also update `w2d-admin/scripts/seed-admin.mjs` (DECISIONS.md §13
   flags this) to seed the new `category` field instead of `userType`.
4. Do NOT remove `userType` from anywhere yet — same as Task 2, this is
   additive until a full migration is separately planned and approved.
5. Reports and Requirements screens are flagged in DECISIONS.md §11 as
   "still near-empty stubs" — do not build these out as part of this
   task, that's separate scope. Only touch them if they specifically
   reference `userType` and would break otherwise.

Show me the list of files you plan to touch before making changes.
```

---

## After Batch 1 is done

Come back and tell me:
- Which tasks completed cleanly vs. which the agent pushed back on or
  flagged as ambiguous
- Any place the agent asked YOU a question mid-task — bring those to me
  before answering the agent directly if they touch product decisions,
  not just implementation details
- **Push to GitHub after each task is confirmed working** — don't batch
  all 5 tasks into one uncommitted pile. `git add -A && git commit -m
  "..." && git push` after Task 1 (no-op, but confirms baseline), and
  after each of Tasks 2-5 once you've reviewed and accepted the diff.

I'll review and generate Batch 2 (public profile screen UI, existing
profile-screen editing UI, the one-time "pick your category" migration
prompt for existing accounts) once this foundation is confirmed working
and pushed.
