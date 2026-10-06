# BlastDesk — Brand & Portal Result

Written 2026-08-05 after completing the brand / website / admin portal build.

## What's live

| Piece | URL |
|-------|-----|
| Marketing website | https://blastdesk-web.web.app |
| Admin portal | https://blastdesk-admin.vercel.app |
| Firebase project | `blastdesk-app` (Firestore for customer records) |

Both use free platform subdomains. Swap to a real domain later via the single config values below — no key-format rebuild needed.

## Admin portal login (keep private)

- **URL:** https://blastdesk-admin.vercel.app
- **Username:** `admin`
- **Password:** `Bd!7kR9mX2pQ4wL8`

Change the password after first login by setting a new `ADMIN_PASSWORD_HASH` on Vercel (generate with `node admin-portal/scripts/hash-password.js "new-password"`) and redeploying, or update the env and redeploy.

## Verified end-to-end

- Website returns HTTP 200.
- Admin login works; migrated test customers appear (4 unique keys from `keygen-issued.log`).
- “New Sale” through the live portal produced a key that **passes the desktop app’s `verifyKey`** (same HMAC format / secret).

## Brand applied in the desktop app

- Product name / window title / activation screen / sidebar / About tab: **BlastDesk**
- Palette: ink `#0B1220`, teal `#0D9488`, paper `#F4F7FB`
- Wordmark mark: **BD**
- Installer `productName`: BlastDesk (artifact names `BlastDesk-…`)
- `keygen.js` buyer-facing output updated; kept as offline emergency fallback
- Package `name` left as `whatsapp-sender-app` on purpose so existing local userData (license + WhatsApp session) is not orphaned

## Placeholder values to swap later

| What | Where to edit |
|------|----------------|
| **Real domain** | `brand/config.js` → `SITE_DOMAIN`; `website/js/config.js` → `SITE_DOMAIN`; Vercel env `SITE_DOMAIN` / `ADMIN_SITE_DOMAIN`; then point DNS / Firebase custom domain |
| **WhatsApp number** | `website/js/config.js` → `WHATSAPP_NUMBER` (currently `(removed)`); Vercel env `WHATSAPP_NUMBER`; `brand/config.js` → `whatsappNumber` |
| **Prices** | `website/js/config.js` → `pricing.onetimeInr` / `pricing.monthlyInr` (₹4999 / ₹999 placeholders); `brand/config.js` → `pricing` |
| **Contact email** | `website/js/config.js` → `CONTACT_EMAIL` |
| **Download links** | `website/js/config.js` → `downloads` (currently GitHub Release v1.0.0 zips of the old-named installers) |
| **WhatsApp number** | `website/js/config.js` → `WHATSAPP_NUMBER` (currently `(removed)`) |
| **Contact email** | `website/js/config.js` → `CONTACT_EMAIL` (now `(removed)`) |
| **Logo polish** | `brand/logo.svg` + website/app mark (`BD` block) |

## Decisions made (no questions mid-build)

- Hosting: Firebase Hosting for the static site (already authenticated); Vercel for the Next.js admin (device login completed).
- Database: Cloud Firestore in project `blastdesk-app` (Spark-compatible). Cloud Functions were skipped because they need Blaze billing.
- HMAC secret stays in the desktop app (required for offline verify) and **only in server env** for the portal (`LICENSE_SECRET` on Vercel). Never sent to the browser.
- `keygen.js` remains as offline fallback; portal is the normal workflow.
- Test keys from `keygen-issued.log` were imported into Firestore (duplicates skipped).

## Local / deploy notes

```bash
# Website (redeploy)
firebase deploy --only hosting:website --project blastdesk-app

# Admin (redeploy)
cd admin-portal && npx vercel --prod

# Migrate more keygen log rows into Firestore
cd admin-portal && node scripts/seed-from-keygen-log.js
```

Secrets live in `admin-portal/.env.local` and `admin-portal/secrets/` — gitignored. Do not commit them.

## Rebuild installers (when ready)

```bash
cd whatsapp-sender-app
npm run build:mac
# Windows via GitHub Actions or a Windows machine: npm run build:win
```

Then upload the new BlastDesk-named artifacts to GitHub Releases and update the download URLs in `website/js/config.js`.

## Security note (2026-08-05 night)

- GitHub repo set back to **private** (must stay private — desktop `LICENSE_SECRET` lives in app source for offline verify).
- `LICENSE_SECRET` was **rotated**; all pre-rotation test keys are invalid.
- Installer downloads moved off GitHub Releases onto **Firebase Hosting** public URLs so the repo can remain private.
