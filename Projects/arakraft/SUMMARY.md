---
tags: project
status: active
owner: "[[People/Vishnu]]"
---
# PROJECT: araKraft Works website

## 1. What this project is
- **Goal:** A one-page website and product catalogue for araKraft Works, a laser engraving and CNC workshop in Kallady, Batticaloa, Sri Lanka.
- **Who it is for / client:** araKraft Works (part of araCreate Group). Customers order through WhatsApp.
- **Why it exists:** Show the four laser collections (photo frames, souvenirs, key tags, restaurant menus) with starting prices, plus a CNC gallery. Every "order" button opens a WhatsApp chat with the product and price filled in. No checkout, no accounts.

## 2. Status now (as of 2026-10-01)
- Site works locally and builds clean. 18 Playwright tests pass (Chrome + WebKit).
- Code follows the araCreate conventions (folders, file headers, Makefile, VERSION, LICENSE, semantic-release, param-case files, snake_case labels).
- Root now holds only Makefile, CHANGELOG.md, VERSION, .gitignore, LICENSE, README.md plus the convention folders. All website files live in `src/`.
- Code is pushed to GitHub `aracreate-group/arakraft-works` (last push `8eb6596`, 2026-10-01).
- Lighthouse (2026-09-26): phone 93 speed, desktop 100; accessibility, best practices, SEO all 100.
- Not done: Cloudflare Pages setup and domain (Vishnu does this himself). Old Netlify files still archived, waiting for Cloudflare to go live.
- Not done: analytics IDs, social links, exact workshop address and hours.
- Not done: signed ("Verified") commits.

## 3. Next steps
1. Vishnu: create the Cloudflare Pages project (root directory `src`), remove the old `arakraft.works` → `aracreate.group` redirect, connect `arakraft.works` and `www`.
2. After that, check the live site and clean up Netlify leftovers.
3. Set up SSH commit signing so new commits show "Verified".
4. Add GA4 / Meta Pixel IDs, social URLs, exact address and hours when confirmed.
5. Optional: add `llms.txt` (AI-readers score was 50).

## 4. Decisions
- 2026-09-26 — Apply the araCreate conventions to the whole repo — keep all araCreate projects the same shape. #decision
- 2026-09-26 — Commit author set to Aravinth Panch (as the conventions use) — follow the conventions. #decision
- 2026-09-26 — Code labels renamed to snake_case, except names React/browser need — conventions rule. #decision
- 2026-09-26 — Host on Cloudflare Pages instead of Netlify; Vishnu does the Cloudflare and domain steps himself — the domain is in a different Cloudflare account. #decision
- 2026-09-26 — Keep Vite + React instead of Astro (the conventions' default for static sites) — it is already a pre-rendered single page. Written as a deviation in the README. #decision
- 2026-09-26 — Removed unused files and images (public folder 30 MB → 14 MB); keep picture quality, no lossy cuts — quality first. #decision
- 2026-09-28 — Removed `docs/handoff.md` (old Netlify handoff) — not needed. #decision
- 2026-09-28 — When Vishnu asks for a "check", only report, do not change files. #decision
- 2026-10-01 — Only six files stay at the root; everything for the website goes in `src/` — Vishnu's reading of the conventions. #decision
- 2026-10-01 — Use SSH signing (a new key just for signing) for verified commits; old commits stay unsigned (no history rewrite). #decision

## 5. Timeline
- 2026-10-01 — Root structure cleaned; all website files moved into `src/`; pushed `8eb6596`. Asked how to get verified commits.
- 2026-09-28 — Handoff file removed and pushed. Servers stopped. Waiting on Cloudflare.
- 2026-09-26 — Ran locally. Applied conventions, health check, fixed share preview image, Lighthouse scores, removed unused files, light JPEG optimisation, first push to GitHub (4 commits). Cloudflare guide written.
- 2026-09-23 — Verification of CNC gallery, menu image fix, collection order (from repo docs).
- 2026-09-22 — Laser-first redesign done (from repo docs).

## 6. Key facts
- **People:** [[People/Vishnu]] — runs the project. Aravinth Panch — commit author per conventions.
- **Companies:** [[Companies/araCreate Group]] — parent company (araCreate Group).
- **Tools:** React 19, TypeScript, Vite 8, oxlint, Playwright + axe-core, make, semantic-release, Cloudflare Pages (was Netlify).
- **Links / repos:** github.com/aracreate-group/arakraft-works · live target https://arakraft.works · conventions repo aracreate-group/aracreate-conventions
- **Contact on site:** WhatsApp +94 71 669 7303, lk@aracreate.group, Kallady, Batticaloa.
- **Prices (starting):** photo frames LKR 1,200 · souvenirs LKR 2,000 · key tags LKR 180 · restaurant menus LKR 2,500. CNC is quote-only.
- **Local path:** `~/Downloads/arakraft-works` on the office Mac.

## 7. Files and documents
- `README.md` — what the site does, stack, layout, commands (repo root)
- `docs/cloudflare-hosting.md` — step-by-step Cloudflare Pages + domain guide
- `docs/launch-notes.md` — laser-first redesign notes (2026-09-22)
- `docs/verification.md` — test and Lighthouse results (2026-09-23)
- `docs/image-generation.md`, `docs/catalogue-prompts.md`, `docs/share-card-prompt.txt` — image notes and prompts

## 8. Open questions and problems
- Is the Cloudflare Pages project and domain live yet?
- Which Cloudflare account owns `arakraft.works`? (It is not in the account with sathyamangalam.online and vidivu.in.)
- Exact address, opening hours, analytics IDs and social links are still unknown.

## 9. All chats in this project
- Root structure into src, verified commits (archived: Projects/arakraft/claude-code/Downloads-arakraft-works__2026-10-01_e1378dbf.md) — 2026-10-01
- Root structure (empty start) (archived: Projects/arakraft/claude-code/Downloads-arakraft-works__2026-10-01_ebf9d469.md) — 2026-10-01
- Run locally, conventions, health, push (archived: Projects/arakraft/claude-code/Downloads-arakraft-works__2026-09-26_99f545a6.md) — 2026-09-26
