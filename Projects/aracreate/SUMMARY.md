---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: araCreate — website copy and design system

## 1. What this project is
- **Goal:** (1) Get the araCreate website out of Webflow and running locally / self-hosted. (2) Build one end-to-end design system for all araCreate websites.
- **Who it is for / client:** [[Companies/araCreate Group]] — the main site aracreate.group plus future sites (araCreate Academy, Meditate, service spin-off landing pages, all under araCreate Group).
- **Why it exists:** The site was built in [[Tools/Webflow]] from the bought Duotint template. The company will have many websites, so it needs one shared design system.

## 2. Status now (as of 2026-10-06)
- Paused since 2026-08-20. No new website work recorded. Design-system work continued in [[Projects/ac-ds/SUMMARY]] (phase 2 started 2026-10-06 from the same Webflow exports).
- **Website copy — done.** `~/araCreate/AC/araCreate Website`: 19 pages, 428 files, 0 missing, CMS content included (crawled from the live site). Open with "Start website".
- **Template copy — done.** `~/araCreate/AC/araCreate Template`: 53 pages, 442 files, 0 missing (crawled from the published template demo).
- Menu bug fixed (invisible click-blocker from `index.html` links); 2,636 links changed to folder links.
- **Design system — built and pushed.** Repo `~/araCreate/AC/aracreate-design-system` (follows aracreate-conventions), 24 component families + 11 page sections, deck (10 slides), brand facts, assets from ACDS, 7 test gates passing. Pushed to Claude Design as a new project "araCreate Design System" (90 files) on 2026-08-20.
- Later (21–24 Aug) this design system was merged into ACDS — see [[Projects/ac-ds/SUMMARY]].
- Not done:
  - Contact forms (26) do not send — needs a Formspree link from Vishnu.
  - 8 menu links go to pages that don't exist (`/business`, `/training`, `/ventures`, `/archive/services-old-2`) — Vishnu said leave them.
  - Site search does not work outside Webflow.
  - Nothing committed to git in the design-system repo (needs Vishnu's explicit word).
  - Real screen-reader pass not done.
  - Vishnu noticed logos missing in the Claude Design version — not resolved in chat (thumbnail or broken image paths, not sure).

## 3. Next steps
1. Get a Formspree link → wire all 26 forms (one line in `js/aracreate-forms.js`).
2. Decide what happens to the local repo `aracreate-design-system` now that ACDS is the single source.
3. Commit the drafted commits (only on Vishnu's instruction).
4. Bigger moves when ready: build Academy from the starter kit; rebuild aracreate.group on the system and leave Webflow.
5. Empty `_to_delete` folders (Desktop and repo).

## 4. Decisions
- 2026-08-19 — UI first; skip CMS crawl and forms at first. #decision
- 2026-08-19 — Skip the unlinked pages: Meet Us / Meet Aravinth / Meet Shyam / Search and the 3 unfinished `/etc/` pages. — not needed. #decision
- 2026-08-19 — Leave the 8 dead menu links as they are. — those pages will be built later. #decision
- 2026-08-19 — Website lives in `~/araCreate/AC/` (Vishnu moved it off the Desktop). #decision
- 2026-08-19 — Template copied from the published demo `aracreate-template.webflow.io` (Starter plan has no export). #decision
- 2026-08-19 — Design system in Claude Design + code, not Storybook. Lock the look as-is; rebuild template code from scratch (Duotint licence = one site only). #decision
- 2026-08-19 — Greys merged to 4; type locked exactly; dark theme skipped for now; full component set. #decision
- 2026-08-19 — Signature edge: dotted 1px, on every panel by default, square corners; add scroll progress bar, horizontal gallery, sticky side text. #decision
- 2026-08-19 — Follow https://github.com/aracreate-group/aracreate-conventions for the repo. #decision
- 2026-08-19 — Everything sharp (all corners 0), even against the brand guideline (cards 20px etc.). — "that is the only change". #decision
- 2026-08-20 — Take deck, logos, company facts and colour corrections from ACDS; not their React components or failing colours. #decision
- 2026-08-20 — Put our system in Claude Design as a separate new project; do not replace or update ACDS. #decision

## 5. Timeline
- 2026-08-18 — Webflow export ZIP uploaded; local build made (jQuery, fonts, libraries self-hosted). CMS lists came out empty.
- 2026-08-19 — Crawler built; full site with CMS content downloaded (19 pages, 373 assets). Port and stylesheet-fingerprint bugs fixed.
- 2026-08-19 — Template demo downloaded (52 pages). Menu click-blocker bug found and fixed. Everything saved to project notes.
- 2026-08-19 — Design system planned (8 phases), audit of 72 pages, foundations, signature edge, repo on conventions.
- 2026-08-19 — Component library built; "do fully": a11y gate, proof page, starter kit, guidance. Everything made sharp.
- 2026-08-20 — Audit vs team's ACDS (table). Took facts, colours, logos, deck from ACDS.
- 2026-08-20 — `/design consent` given; new Claude Design project "araCreate Design System" created (90 files). Vishnu said logos look missing.

## 6. Key facts
- **People:** [[People/Vishnu]] — owner, not a tech person, wants numbered simple steps. Ara — owner of ACDS team system, org default (probably [[People/Aravinth Panch]], not sure). [[People/Aravinth Panch]] and [[People/Shyam]] — have "Meet" booking pages on the site. Andreas Kissling — blog author on the site.
- **Companies:** [[Companies/araCreate Group]], [[Companies/araCreate India]], Deep Tech Foundry (DTF, sub-brand on the same site)
- **Tools:** [[Tools/Webflow]], [[Tools/Claude Design]], [[Tools/Claude]], [[Tools/GitHub]], [[Tools/Formspree]] (planned), Duotint template, Playwright, Python
- **Links / repos / servers / file paths:**
  - Live site https://aracreate.group (Webflow site id `63780fb6eec282197fc5547f`)
  - Template demo https://aracreate-template.webflow.io
  - `~/araCreate/AC/araCreate Website` (+ `Rebuild tools/`: Build my website, Get the template)
  - `~/araCreate/AC/araCreate Template`
  - `~/araCreate/AC/aracreate-design-system` (repo; open with `scripts/open-design-system.command`; `make test`)
  - Conventions: https://github.com/aracreate-group/aracreate-conventions
  - Brand: yellow/Golden Sun `#f9bf3b`, Graphite Gray `#555555`, Black `#222222`, Stroke `#cecece`, Canvas `#f6f6f6`; Poppins; Monument Extended for logo only. HQ Berlin (Hubertusstr. 5). "300+ clients".
- **Group entities (group map v0.3):** araCreate GmbH (AC, Germany), araCreate India Pvt Ltd (ACI), araCreate Lanka Pvt Ltd (ACL), BatchOne GmbH, DreamSpace Pvt Ltd, Hybrid 360 Art Tech, Genuine Products. Root domain aracreate.group.
- **PRD template (ACG)** is used for client projects; supplier entity araCreate GmbH, Berlin.
- **araCreate VPS (my-vps):** root SSH is key-only; Kishor's key added 5 Oct 2026. The root password was once shared in chat — rotate it.
- **Clockify Sept 2026:** 206 h logged; projects AC, ARM, DSA, Future State, Halle, SinoLink (see [[Projects/clockify/SUMMARY]]).
- **ISO 27001 (ACI):** see [[Projects/ac-iso-27001-isms/SUMMARY]].
- **Related:** [[Projects/ac-ds/SUMMARY]]

## 7. Files and documents
- `claude/webflow-export-status.md` — website export state (claude.ai project notes)
- `claude/design-system-plan.md`, `design-system-status.md`, `design-system-licence-policy.md`, `design-system-audit-findings.md`, `design-system-signature-devices.md`, `design-system-audit-vs-acds.md`, `design-system-claude-bundle.md` — design system notes (claude.ai project notes)
- `READ ME.txt` — plain-English guide in each website folder (Mac)
- `docs/brand-facts.md`, `docs/assets.md`, `docs/guidance.md`, `docs/decisions.md`, `docs/licence-policy.md` — in the design-system repo
- `releases/claude-design-system/` — the Claude Design bundle (`brand.md` = facts and voice; `CLAUDE.md` name is reserved)
- `ds-audit-comparison.md` — ours vs ACDS audit (sent in chat)

## 8. Open questions and problems
- Contact forms send nowhere (Formspree link missing).
- 8 dead menu links on the live site too.
- Logos missing in the Claude Design version — cause not confirmed.
- Domains for Academy etc. not decided.
- Two password-locked Webflow pages (`/dtf/dtf`, `/archive/services-old-2`) cannot be copied.
- Copies are snapshots — re-run "Build my website" after Webflow changes.
- Monument Extended is a commercial font — never in a public repo.

## 9. All chats in this project
- Index: INDEX (archived: Projects/aracreate/chats/INDEX.md) (11 chats; 9 imported from the personal account on 2026-10-06)
- Website export from Webflow (archived: Projects/aracreate/chats/2026-08-19 Website export from Webflow.md) — 2026-08-19
- Design system planning (archived: Projects/aracreate/chats/2026-08-19 Design system planning.md) — 2026-08-19
