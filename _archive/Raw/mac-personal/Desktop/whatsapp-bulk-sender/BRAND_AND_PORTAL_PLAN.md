# BlastDesk — Brand, Website & Admin Portal Plan

Written 2026-08-05. This is the plan to review before any building starts — nothing below is built yet.

## What's changing vs. what stays the same

**Stays exactly the same (don't rebuild):**
- The desktop app's core engine: Baileys WhatsApp connection, CSV/tags, sending with delays — already built and proven working.
- The customer's experience of licensing: paste a key, it works offline forever (one-time) or until the embedded expiry (subscription). No internet check in the app, ever.

**What's new:**
1. A brand name and look — **BlastDesk**.
2. A public one-page marketing website.
3. A hosted admin portal that replaces the `node keygen.js` terminal step — you log in from any browser to generate keys and see your customers.
4. A visual refresh of the desktop app to match the brand.

**Important architecture note:** the admin portal is the *only* new server/hosting piece. It exists purely for your sales bookkeeping and key generation. The product itself — what runs on a customer's computer — remains fully offline. This is not a reversal of the "no server for the product" decision, it's a separate small internal tool for you.

---

## Phase 1 — Brand foundations

- Name: **BlastDesk** (chosen after checking for conflicts with existing WhatsApp tools like AiSensy, WATI, Interakt, Gupshup, DoubleTick — clear).
- Tagline direction: something like *"Send WhatsApp broadcasts from your own number — no monthly API fees, no approval process, install once and own it."* (differentiates from the official-API competitors, who all charge recurring API fees).
- Register a domain — check availability yourself on Namecheap/GoDaddy for `blastdesk.com`, `blastdesk.app`, or `blastdesk.io` (a couple minutes, only you can buy it). Rough cost: $10-15/year.
- Simple color palette + wordmark logo (no need for a paid designer to start — a clean text-based logo in 1-2 brand colors is enough for v1; can upgrade later).
- Rename throughout: app product name, installer file names, `keygen.js` buyer-facing text, README, GitHub repo (optional — repo name can stay technical, nobody but you sees it).

## Phase 2 — Marketing website (simple one-pager)

Single page, sections:
- Hero: name, tagline, a "Get it on WhatsApp" button that opens a `wa.me` chat link to you directly (this replaces a checkout — you don't need a payment gateway, you're already selling manually).
- What it does: CSV upload, `{Name}` tags, safe automatic delays, works with your own WhatsApp number.
- Pricing: your one-time price and monthly price, side by side.
- Download buttons for Mac/Windows — **safe to make public**, since the app is useless without a license key; the installer itself can be a marketing tool (someone downloads it, tries the locked license screen, messages you to buy).
- FAQ: install steps, refund policy, "will this get my WhatsApp banned?" (answer honestly using the anti-ban delay explanation already written), data privacy (contacts never leave their computer).
- Footer: WhatsApp/email contact.

Hosting: a static site on Vercel/Netlify/Cloudflare Pages — free tier is enough at this scale.

## Phase 3 — Admin portal (hosted, replaces the terminal)

This is the biggest new piece. Features:
- Login screen — just for you, one account, strong password (no public sign-up).
- "New Sale" form: buyer name + phone/email + plan (one-time / subscription + days) → generates the signed license key server-side (same HMAC scheme as today, but the secret now lives only in the server's environment, never on your laptop or in any file you could lose).
- Shows the generated key with a copy button and a one-click "send via WhatsApp" link pre-filled with the install + activation message.
- Customer list: every key you've issued, with plan type, issue date, expiry (if any), status (active/expired).
- One-click "renew" for a subscription customer — generates a fresh key against their existing record instead of you re-typing their name.
- Optional: log the amount paid per sale, so this doubles as your sales record — no separate spreadsheet needed.

Tech (for Cursor to build, not a decision you need to make): a small backend + a lightweight hosted database (e.g. Supabase or Neon's free Postgres tier), deployed on Vercel/Render. Estimated cost: **$0-15/month** at your current scale — free tiers are genuinely enough for one seller issuing keys by hand.

## Phase 4 — App UI refresh

- Apply the BlastDesk name, logo, and color palette inside the Electron app: title bar, sidebar, activation screen, an "About" section.
- Rebuild both installers under the new name.
- Point auto-update at the (possibly renamed) release feed.

## Phase 5 — Migration & cutover

- Carry over any keys already issued via `keygen.js` into the new portal's customer list (low volume right now — just your own test entries).
- Keep `keygen.js` as an offline emergency fallback (e.g. if the portal is down and you need to make a sale right now), but the portal becomes the normal way you work.

## Phase 6 — Final QA before calling this done

- Full flow test: website → WhatsApp contact → you generate a key in the portal → send installer + key → they activate → connect → send. On both Mac and Windows, since the rebrand touches UI on both.
- Security check: confirm the HMAC secret never appears in the portal's browser-visible code or network responses (it must only exist server-side), and that the admin login can't be easily guessed/brute-forced.

## Rough cost picture

| Item | Cost |
|------|------|
| Domain | ~$12/year |
| Website hosting | Free tier |
| Admin portal hosting + database | $0-15/month |
| Code signing certs (optional, recommended once branded) | Windows ~$100-250/yr, Mac (Apple Developer Program) $99/yr |
| Logo/design polish (optional) | $0 if DIY, variable if you hire a designer |

The infrastructure itself is genuinely cheap — most of the "investment" here is build time, not ongoing cost. The one area worth spending real money on, if you want the brand to feel trustworthy immediately, is code signing (removes the scary "unknown publisher" warnings).

## Decisions still open (yours to make, not urgent)

- Exact domain extension (.com vs .app vs .io) — depends on availability.
- Whether to get code-signing certs now or after your first few sales.
- Whether you want a designed logo (paid) or a simple text wordmark (free) for v1.

## Suggested build order for Cursor

1. Brand assets + renaming pass across the existing app/repo.
2. Marketing website (static, one-pager).
3. Admin portal backend + database + auth + key generation (the biggest task).
4. App UI refresh to match brand.
5. Migration of existing keys + cutover.
6. End-to-end QA on both platforms.
