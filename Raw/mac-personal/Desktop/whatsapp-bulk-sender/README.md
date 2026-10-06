# BlastDesk

Desktop WhatsApp broadcast app for small businesses. Sold as a one-time or monthly license; each customer runs their own copy offline.

**Marketing site:** https://blastdesk-web.web.app  
**Admin portal:** https://blastdesk-admin.vercel.app  
**App code:** `whatsapp-sender-app/`  
**Seller key tool (offline fallback):** `whatsapp-sender-app/keygen.js`  
**Brand config (domain one-liner):** `brand/config.js`  
**Ship summary:** `BRAND_RESULT.md`

## Domain (one-line swap)

When you buy a real domain, change:

1. `brand/config.js` → `SITE_DOMAIN`
2. `website/js/config.js` → `SITE_DOMAIN`
3. Vercel env `SITE_DOMAIN` / `ADMIN_SITE_DOMAIN`

## Sale flow (normal)

1. Collect payment (UPI / bank).
2. Open the admin portal → New Sale → generate key → Copy / Share via WhatsApp.
3. Send the Mac/Windows installer + key.
4. Buyer pastes the key on the BlastDesk activation screen.

## Sale flow (offline emergency)

```bash
cd whatsapp-sender-app
node keygen.js onetime --buyer "Name"
# or
node keygen.js subscription --days 30 --buyer "Name"
```

## Placeholders to swap

| What | File / env |
|------|------------|
| WhatsApp number | `website/js/config.js` → `WHATSAPP_NUMBER` (now `(removed)`) |
| Prices | `website/js/config.js` → `pricing` (₹4999 / ₹999) |
| Downloads | same file → `downloads` |
| Admin password | Vercel env `ADMIN_PASSWORD_HASH` |

## Product notes

- Offline HMAC license keys — no activation-limit enforcement.
- Subscription expiry uses the computer’s system clock.
- Contacts never leave the customer’s computer.
- Installers are unsigned until a code-signing certificate is added.
- Anti-ban defaults: 20–60s delay, 5 min pause every 50 messages.

## Quick start (dev)

```bash
cd whatsapp-sender-app
nvm use 20
npm install
npm run dev
```
