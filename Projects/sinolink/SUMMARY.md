---
tags: project
status: active
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: SinoLink website (now SinoLink Europe)

## 1. What this project is
- **Goal:** Build and run the SinoLink website (designed in Webflow, now self-hosted from a git repo), live at www.sinolink.de and sinolink.pt.
- **Who it is for / client:** [[Companies/Sinolink]] (SinoLink Deutschland), a consulting company for China sourcing, technology transfer and inter-cultural competence. Since Sep 2026 branded "SinoLink Europe"; the company is now Portuguese (earlier: SinoLink Deutschland, Dresden). Contact: [[People/Achim]] (Achim Neu).
- **Why it exists:** Client work by araCreate. Footer credit: "Design & Development by araCreate Group".

## 2. Status now (as of 2026-09-25)
- Full site live on sinolink.de and sinolink.pt (26 pages checked on 2026-09-25). Served from a git repo: Claude pushes to `dev`, Vishnu pushes `main` (live).
- New "SinoLink Europe" logo (client SVG) on every page, loader and icons (2026-09-23). Preloader script self-hosted.
- New Impressum and Privacy pages for both sites in EN/DE/PT/CN (8 pages), in sitemaps and footers. Footer: "© 2026 SinoLink Europe".
- Client "no names" rule: all personal names (incl. Achim Neu), "SinoLink Deutschland" and Dresden removed from the site. Impressum shows the .pt company email; EU dispute paragraph removed.
- Earlier (June): under-development page, services 3×2 card grid, mobile nav with language switcher only, Webflow MCP connected; site exported from Webflow; forms broke and Web3Forms was planned.
- Not sure: whether the contact forms were fixed after the export.

## 3. Next steps
1. Confirm the contact form works (Web3Forms plan from June, key under client email; success redirect).
2. Check the legal pages at phone width (ran off the right edge; not sure if fixed).
3. Small issues from the 2026-09-25 code check were left on purpose — revisit only if the client asks.

## 4. Decisions
- 2026-06-02 — Under-dev page is minimal: no countdown, no progress bar, no email form, no extra sections — keep it clean. #decision
- 2026-06-02 — Contact button opens the mail app to the company email; LinkedIn removed. #decision
- 2026-06-03 — Legal pages as own HTML pages, not PDF links — better for the site. #decision
- 2026-06-07 — Use Webflow MCP `/mcp` endpoint with http transport, project scope in `~/Desktop/sinolink` — `/sse` gives HTTP 400. #decision
- 2026-06-08 — Services as 3×2 card grid (client changed design), Figtree 28/500 headings, 16/400 body, 5% side padding, sharp corners. #decision
- 2026-06-08 — All custom CSS scoped under `#sinolink-services` with `ssv-` classes — earlier code broke the site layout. #decision
- 2026-06-11 — On tablet and mobile show only the language switcher in the nav. #decision
- 2026-06-12 — Web3Forms over Formspree for exported forms — free with no monthly cap. #decision
- 2026-09-23 — Ask clients for SVG logos; self-host the Webflow preloader script; screenshots and test pages never go into git. #decision
- 2026-09-23 — Claude pushes to `dev` only; Vishnu pushes `main` (live) himself. #decision
- 2026-09-25 — Readable email on legal pages is the .pt one; .de stays hidden for the contact form — company is now Portuguese. #decision
- 2026-09-25 — No personal names anywhere on the site (client rule). #decision
- 2026-09-25 — Minor findings from the full check left as they are ("leave all, lets deploy"). #decision

## 5. Timeline
- 2026-06-02 — Maintenance page and "under development" page built from sinolink-dev.webflow.io branding.
- 2026-06-03 — Under-dev page deployed; legal pages converted from PDF.
- 2026-06-07 — Webflow MCP connected in terminal; hero and homepage copy changes.
- 2026-06-08 — Services slider → sticky scroll try → final 3×2 card grid.
- 2026-06-09 — Navbar logo colour on mobile scroll problem (not sure this was SinoLink).
- 2026-06-11 — Animation code review, mobile language-only nav, legal pages updated.
- 2026-06-12 — Exported site; forms fix with Web3Forms planned.
- 2026-09-23 — New SinoLink Europe logo (client SVG) on every page, loader and icons; live on both domains.
- 2026-09-25 — New legal pages for .de and .pt in 4 languages; footer "SinoLink Europe"; .pt email on Impressum; all names removed; 26 live pages checked.

## 6. Key facts
- **People:** [[People/Achim]] — client contact (Achim Neu; no longer named on the site); [[People/Vishnu]] — design and build.
- **Companies:** [[Companies/Sinolink]], [[Companies/araCreate Group]]
- **Tools:** [[Tools/Webflow]], [[Tools/Claude Code]], [[Tools/Clockify]] (tag #slk), Web3Forms
- **Links / repos / servers / file paths:** live www.sinolink.de and sinolink.pt; Webflow staging sinolink-dev.webflow.io; repo folder `araCreate/SLK/www.sinolink.de` (office Mac, branches `dev` and `main`); MCP folder `~/Desktop/sinolink` (`.mcp.json`). Webflow site/workspace IDs (not saved here).
- **Dev history:** [[Projects/sinolink/DEV-LOG]] (Claude Code sessions, office Mac). Chats index: [[Projects/sinolink/chats/INDEX]].
- **Design:** new logo grey #7C7E7F + yellow #FBB404 (Sep 2026); earlier brand yellow #F6C506; legal pages use #f7f5f0 bg, #1a1a1a header/footer, gold #b5843a, Cormorant Garamond + DM Sans; nav uses the Walsh cloneable template classes (`walsh-nav-...-2`).
- **Services (6 cards):** Source Inspection, Source Identification, Preparation, Partner Network, Forwarding, Fill the Gap.
- **Related:** [[Projects/aracreate/SUMMARY]] (SinoLink LinkedIn launch post)

## 7. Files and documents
- `sinolink-coming-soon.html` — under-development page (chat output).
- Impressum and Privacy Policy HTML pages (chat output, updated 2026-06-11).
- Services grid embed code (`#sinolink-services`), nav/animation code split into head and footer.

## 8. Open questions and problems
- Forms after export: fixed and tested? Key moved to client email?
- Where is the site hosted (served from the repo — host not written down)?
- Legal page at phone width ran off the right edge — fixed?
- Navbar logo colour on mobile (#6A0BB2) was never fixed; unclear if this is the SinoLink site (not sure).
- 2026-06-02 "Axiom" service template (from Fleet template) — was it for SinoLink? (not sure)

## 9. All chats in this project
- [[Projects/sinolink/chats/2026-06-02 Responsive maintenance page HTML|Responsive maintenance page HTML]] — 2026-06-02
- [[Projects/sinolink/chats/2026-06-02 Website under development template|Website under development template]] — 2026-06-02
- [[Projects/sinolink/chats/2026-06-03 Converting PDF legal pages to responsive website|Converting PDF legal pages to responsive website]] — 2026-06-03
- [[Projects/sinolink/chats/2026-06-07 Connecting Webflow MCP to Claude in terminal|Connecting Webflow MCP to Claude in terminal]] — 2026-06-07
- [[Projects/sinolink/chats/2026-06-08 Converting slider cards to vertical scroll|Converting slider cards to vertical scroll]] — 2026-06-08
- [[Projects/sinolink/chats/2026-06-09 Webflow navbar logo visibility on mobile scroll|Webflow navbar logo visibility on mobile scroll]] — 2026-06-09
- [[Projects/sinolink/chats/2026-06-11 Website full animation review|Website full animation review]] — 2026-06-11
- [[Projects/sinolink/chats/2026-06-12 Forms not working after webflow export|Forms not working after webflow export]] — 2026-06-12
