---
tags: project
status: paused
owner: "[[People/Vishnu]]"
updated: 2026-10-06
---
# PROJECT: BlastDesk (WhatsApp bulk sender)

## 1. What this project is
- **Goal:** BlastDesk: a WhatsApp broadcast desktop app for small businesses. The Claude project "whatsapp api" (made 2026-08-04) is this project (confirmed 2026-10-06).
- **Who it is for:** Small businesses that send WhatsApp broadcasts. Sold as a one-time or monthly licence.
- **How it works:** Contacts stay on the buyer's computer. Anti-ban delays built in (20–60 s between messages, pause every 50 messages). Offline HMAC licence keys; the seller makes keys in an admin portal (Vercel) or with `keygen.js`. Marketing site on Firebase Hosting.

## 2. Status now (as of 2026-10-06)
- Paused. Last commit 2026-08-07.
- v1.0.0 shipped 2026-08-05: Mac DMG ready (unsigned); Windows installer built by CI but not tested on a real PC.
- v2 rebuild in TypeScript started 2026-08-07: sql.js, Baileys transport, consent checkbox on CSV import, safety floors that cannot be turned off.
- The Claude project itself has no instructions, files or real work chats; the facts above come from the Mac repo.
- Project = BlastDesk; folder renamed to `blastdesk` (confirmed 2026-10-06).

## 3. Next steps
1. Get a code-signing certificate.
2. Get a real domain.
3. Test the Windows installer on a real PC.

## 4. Decisions
- 2026-08 — Contacts stay on the buyer's computer; anti-ban delays built in. #decision
- 2026-08 — Licences are offline HMAC keys (one-time or monthly). #decision
- 2026-10-06 — Old "whatsapp api" Claude project = BlastDesk; folder named `blastdesk` (Vishnu confirmed). #decision
- 2026-08-07 — v2 in TypeScript with safety floors that cannot be turned off and a consent checkbox on CSV import. #decision

## 5. Timeline
- 2026-08-04 — Claude project "whatsapp api" created.
- 2026-08-05 — BlastDesk v1.0.0 shipped (Mac DMG; Windows installer via CI).
- 2026-08-07 — v2 TypeScript rebuild; last commit.
- 2026-10-02 — Two viOS move attempts on the Claude project; no content found.
- 2026-10-06 — BlastDesk repo found on the personal Mac; Vishnu confirmed project = BlastDesk.

## 6. Key facts
- **Repo:** `vishnuvarthan18/whatsapp-bulk-sender`; Mac `~/Desktop/whatsapp-bulk-sender`.
- **Tools:** [[Tools/WhatsApp]], [[Tools/Firebase]] (hosting), [[Tools/Vercel]] (admin portal), Baileys, sql.js, [[Tools/Claude]].
- **Possible link:** a Cloudflare R2 bucket `blastdesk-downloads` exists (seen in [[Projects/india-data-atlas/SUMMARY]] open questions).
- **Related:** [[Projects/vios/SUMMARY]]; maybe [[Projects/hr-leads-bala/SUMMARY]] (leads come by WhatsApp, not sure).

## 7. Files and documents
- Repo `whatsapp-bulk-sender` (code, `keygen.js`).

## 8. Open questions and problems
- App is unsigned; no real domain; Windows install never tested.
- Is the v2 rebuild finished?

## 9. All chats in this project
- Moving project to viOS (3) (archived: Projects/blastdesk/chats/2026-10-02 Moving project to viOS (3).md) — 2026-10-02
- Moving project to viOS (6) (archived: Projects/blastdesk/chats/2026-10-02 Moving project to viOS (6).md) — 2026-10-02
