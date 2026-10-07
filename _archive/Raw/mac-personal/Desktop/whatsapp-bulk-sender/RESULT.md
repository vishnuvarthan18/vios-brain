# RESULT.md — WhatsApp Bulk Sender v1.0.0

Shipped 2026-08-05. Hardening pass completed same day (see `AUDIT_RESULTS.md`).

## Market-ready status

| Area | Status |
|------|--------|
| Mac installer | Ready to send — rebuilt DMG with correct auto-update owner (`vishnuvarthan18`) |
| Windows installer | **Built via CI; not yet run on a real Windows PC** — open risk until one smoke install |
| Licensing (offline HMAC) | Ready — clock-skew bypass accepted & documented |
| Connect / Broadcast / CSV | Hardened (disconnect safety, CSV validation, crash interrupt) |
| Code signing | Not done — Gatekeeper / SmartScreen warnings expected |
| Auto-update | Wired; needs a GitHub Release to actually deliver updates |
| Privacy | No shipped debug log of message bodies; activity log masks phones |

**Verdict:** Safe to sell to early Mac customers now. For Windows buyers, do one manual install test of the `.exe` before calling it verified. Prefer getting a code-signing cert soon for trust.

## What you have

| Piece | Path / link | Status |
|-------|-------------|--------|
| Mac installer (arm64 DMG) | `whatsapp-sender-app/dist-installer/WhatsApp Bulk Sender-1.0.0-arm64.dmg` | Built, unsigned |
| Windows installer (NSIS) | CI artifact + `dist-installer/WhatsApp Bulk Sender Setup 1.0.0.exe` | Built; **not runtime-tested** |
| Seller key generator | `whatsapp-sender-app/keygen.js` | `node keygen.js onetime\|subscription` |
| GitHub repo | https://github.com/vishnuvarthan18/whatsapp-bulk-sender | Private |
| Pre-launch audit | `AUDIT_RESULTS.md` | Gaps fixed + accepted limitations listed |

## Windows installer — download

**Run page:** https://github.com/vishnuvarthan18/whatsapp-bulk-sender/actions/runs/30973897537

1. Open the run (sign in as `vishnuvarthan18`).
2. **Artifacts** → **windows-nsis-installer**.
3. Unzip → `WhatsApp Bulk Sender Setup 1.0.0.exe`.

Or use the local copy under `whatsapp-sender-app/dist-installer/`.  
Rebuild: push to `main` or run workflow https://github.com/vishnuvarthan18/whatsapp-bulk-sender/actions/workflows/build-windows.yml

## License flow

- **One-time:** never expires after activation.
- **Subscription:** expiry embedded in the key; checked against the **system clock** on launch (turning the clock back can extend access — known offline limitation).
- Invalid/tampered keys rejected. No activation-limit enforcement (offline tradeoff).

## What I do for each sale

1. Collect payment (UPI / bank).
2. `node keygen.js onetime --buyer "Name"` or `node keygen.js subscription --days 30 --buyer "Name"`.
3. Send Mac `.dmg` or Windows `.exe` + the key.
4. Buyer activates → Connect → scan QR once.

## Customer-facing blurb

Download the installer for your computer (Mac `.dmg` or Windows `.exe`) and open it. Paste the license key we sent after payment and click Activate (works offline). Open Connect and scan the QR with WhatsApp → Linked Devices. Upload a CSV with Name and Phone, write your message with tags like `{Name}`, then launch from Broadcast. Messages are spaced automatically. One-time licenses never expire; monthly licenses expire on the date in your key — pay again for a fresh key when that happens.
