# Contact form: saved in Cloudflare

The Contact panel (slides in from the side on every page) posts to `/api/contact` ([functions/api/contact.js](../functions/api/contact.js)).
Every message is **saved in a Cloudflare D1 database** named `semmozhi-contact` (table `messages`, created from [db/contact_schema.sql](../db/contact_schema.sql),
connected through [wrangler.toml](../wrangler.toml)). Each row records which site it came from (`www...` = production, `dev...` = staging).
Limits: at most 5 messages per 10 minutes per visitor, a hidden honeypot field, length limits. Only a salted hash of the visitor's address is kept.

## Read the messages
```
wrangler d1 execute semmozhi-contact --remote --command "SELECT id, created_at, site, name, contact, message FROM messages ORDER BY id DESC LIMIT 50"
```
(or Cloudflare dashboard -> Storage & Databases -> D1 -> semmozhi-contact -> Explore data).

## WhatsApp copy (optional, not required)
If the secrets below are also set, each saved message is forwarded to WhatsApp as well. Without them nothing is lost: the message is already saved.
`WHATSAPP_TO` is already set. Missing: `WHATSAPP_TOKEN`, `WHATSAPP_PHONE_ID` (from the Meta app, steps below).

## WhatsApp setup (only if you want the copy)
1. **Meta:** create a Meta developer app with the *WhatsApp* product, add a business phone number (this is the number that *sends*; it can be a new SIM or number, not the one you read messages on), and create a **permanent access token** (System User token with `whatsapp_business_messaging`).
2. **Template (recommended):** in WhatsApp Manager create a *Utility* message template, for example named `site_contact`, with three body variables: `New website message from {{1}} ({{2}}): {{3}}`. Wait for approval.
   Without a template, only plain text can be sent, and WhatsApp delivers it only if *you* messaged the business number in the last 24 hours.
3. **Cloudflare:** Workers & Pages -> `semmozhi` (and `semmozhi-dev` for staging) -> Settings -> Variables and Secrets. Add as **Secrets**:

| Name | Value |
|---|---|
| `WHATSAPP_TOKEN` | the permanent access token |
| `WHATSAPP_PHONE_ID` | the *Phone number ID* shown in the Meta app (not a dialable number) |
| `WHATSAPP_TO` | your receiving number, digits with country code, e.g. `9198xxxxxxxx` |
| `WHATSAPP_TEMPLATE` | `site_contact` (omit to send plain text) |
| `WHATSAPP_LANG` | template language code, default `en` |

4. Redeploy (push to `dev`, or Run workflow for production). Send a test message from the Contact page.

## What protects it
- Only the server sees the number and token.
- A hidden honeypot field drops most bots; wrong `Origin` is refused; name and message are required and length-limited.
- Errors from WhatsApp are never shown to visitors.
- Not included: rate limiting and CAPTCHA. If spam appears, add Cloudflare Turnstile (free) in front of the function.

## Where the function is deployed
The `functions/` folder at the repository root is picked up by the production and staging deploys (`pages deploy _site` runs from the repo root). The private admin site deploys from inside `_engine`, so it does not get the function.
