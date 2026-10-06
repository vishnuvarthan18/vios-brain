---
tags: project
status: active
owner: "[[People/Vishnu]]"
---
# PROJECT: Adobe Subscription Status Checker (web-bala)

## 1. What this project is
- **Goal:** A simple website where customers type their email and see their Adobe Creative Cloud (education plan) subscription status, with own branding. Plus an admin panel to add, remove and manage customers.
- **Who it is for / client:** A reseller business of education Adobe plans (Vishnu calls it his business / client "Bala" — not sure). The business is a vendor under a main reseller company.
- **Why it exists:** The main company gives no official way for customers to check status. Only the organisation name is pulled live from the main company; all other data comes from the own admin panel.

## 2. Status now (as of 2026-06-08)
- Built and working on local Mac.
- Live on Hostinger at airdigital.store, after fixing the env variables.
- All 7 tests passed: valid lookup, wrong email, admin wrong/right password, add user, remove user, full flow.
- Search and filter added to the admin user table.
- Push to GitHub was blocked because `.env.production` with real secrets was committed. Fix steps given: remove the file from git, add to .gitignore, reset the last 2 commits.
- A later step (from cowork.md): "Client X" dashboard with credits, approve/revoke requests and activation emails (Resend). Not sure if this was built or deployed.

## 3. Next steps
1. Remove `.env.production` from git history and push clean.
2. Rotate the database password and Supabase keys (they were pasted in chat).
3. Decide on and test the Client X / credits feature (see cowork.md).
4. Make all screens fully mobile responsive.

## 4. Decisions
- 2026-06-08 — Use the main company's status endpoint only for the organisation name; all other data from own admin — main company has no official API. #decision
- 2026-06-08 — Stack: Next.js + Tailwind + Supabase (Postgres) — simple and fits Hostinger Node.js hosting. #decision
- 2026-06-08 — Use VS Code with Claude in VS Code for coding — Vishnu is new to coding. #decision
- 2026-06-08 — Host on Hostinger (with GitHub), not Vercel — Vishnu already has Hostinger business hosting and the domain there. #decision
- 2026-06-08 — UI should match shadcn/ui style, look "not AI-made". #decision

## 5. Timeline
- 2026-06-08 — Project planned; main company status endpoint checked.
- 2026-06-08 — Built locally: status page, admin login, dashboard, API routes.
- 2026-06-08 — Many deploy failures (Hostinger GitHub deploy, Vercel try, "supabaseKey is required" error).
- 2026-06-08 — Fixed by adding SUPABASE_SERVICE_ROLE_KEY in Hostinger env; site works live.
- 2026-06-08 — Full test passed; search/filter added; push blocked by GitHub secret scanning.

## 6. Key facts
- **People:** [[People/Vishnu]] — developer and owner; GitHub user vishnuvarthan18
- **Companies:** Hostinger (hosting + domain), Supabase, main reseller company (status site at reseller.ado-besoft.com)
- **Tools:** [[Tools/Next.js]], [[Tools/Tailwind CSS]], [[Tools/PostgreSQL]], [[Tools/GitHub]], [[Tools/VS Code]], [[Tools/Claude Code]], [[Tools/Vercel]] (tried), [[Tools/shadcn-ui]], Supabase, Resend (planned)
- **Links / repos / servers / file paths:** Live site: airdigital.store. Local folder: ~/adobe-project/adobe-status. Admin login, DB password, Supabase keys (secret, not saved).
- **Related:** [[Projects/hr-leads-bala/SUMMARY]] (same client, not sure)

## 7. Files and documents
- `cowork.md` — steps to add Client X dashboard, credits, request approve/revoke and emails (docs/)

## 8. Open questions and problems
- Secrets were pasted in chat and committed to git. They should be changed.
- Using the main company's endpoint without an official path may break or cause issues later.
- Is the Client X / credits feature live?

## 9. All chats in this project
- [[Projects/web-bala/chats/2026-06-08 Adobe account reseller admin panel and user portal|Adobe account reseller admin panel and user portal]] — 2026-06-08
