# BlastDesk (desktop) v2

Consent-first WhatsApp broadcast app (Electron + TypeScript). Contacts stay on the customer’s computer. Suppression, warm-up, daily caps, and pacing floors are always enforced.

## Develop

```bash
cd whatsapp-sender-app
npm install
npm run dev
```

Use mock WhatsApp transport:

```bash
BLASTDESK_TRANSPORT=mock npm run dev
```

## Verify

```bash
npm run verify
```

## License keys (seller)

```bash
npm run keygen -- onetime --buyer "Name"
npm run keygen -- subscription --days 30 --buyer "Name"
```

Compatible with v1 HMAC keys.

## Build installers

```bash
npm run build:mac
npm run build:win
```

Code signing / notarization require your certificates (see root `BUILD_REPORT.md` BLOCKED). Set `CSC_LINK` / `CSC_KEY_PASSWORD` (Windows) or Xcode signing identity (macOS) when available.

## Safety

- No contact scraping, number generation, or detection-evasion.
- Opt-out / suppression gate on every send.
- Pacing floors and warm-up cannot be set to zero or disabled.
