---
type: system
status: stable
created: 2026-08-21
updated: 2026-08-21
tags: [vos, setup]
---

# 5. Optional: SilverBullet on a server

Only do this if you want to open your notes in a browser from any device, edit
them on your phone with a real editor rather than a text editor, and have them
work offline. It is the most powerful front end available, it is MIT-licensed,
and it costs a few euros a month for the box.

## What it is

**SilverBullet** — MIT licence, actively released through 2026. Your notes stay
**plain Markdown files in a folder** ("a Space") on the server's disk. The web
app is a front end over those files, with wiki links, a query language, and Lua
scripting written inside your Markdown.

Crucially, the maintainer confirms that editing those files directly from
outside the app — with git, a cron job, or your agent — is fine. That is exactly
the property that made it the pick over every other self-hosted option.

## Two hard requirements

1. **Run it on Linux, not macOS.** The docs explicitly discourage
   case-insensitive filesystems, and macOS's default APFS is case-insensitive.
2. **HTTPS is mandatory.** Browsers gate the service worker, crypto and clipboard
   APIs to secure origins, so offline mode and the phone app simply won't work
   over plain `http://`.

## Install (Docker)

```yaml
services:
  silverbullet:
    image: ghcr.io/silverbulletmd/silverbullet:latest
    restart: unless-stopped
    environment:
      - SB_USER=vishnu:CHANGE_THIS_PASSWORD
      - SB_SPACE_IGNORE=.git,*.db,90-System/audits
    volumes:
      - ./VOS:/space
    ports:
      - 127.0.0.1:3000:3000
```

It uses roughly 120 MB of RAM and near-zero CPU — the indexing work happens in
your browser, not on the server.

## Getting to it safely

Two free options, both avoiding open inbound ports:

- **Cloudflare Tunnel** (`cloudflared`) — makes an outbound-only connection, TLS
  terminates at Cloudflare's edge, so you need no reverse proxy and no
  certificates. Add Cloudflare Access in front for a second gate. Cost: a domain
  (~$10/year).
- **Caddy** — three lines and automatic Let's Encrypt certificates:
  ```
  notes.example.com {
      reverse_proxy 127.0.0.1:3000
  }
  ```

## On Android

Open the URL in Chrome or Firefox → menu → *Add to Home Screen*. You get an app
icon, a full offline copy of your notes in the browser's local storage, and
automatic sync when you reconnect.

## Where to host it

| Option | Cost | Notes |
|---|---|---|
| An old laptop or Mac mini at home + Cloudflare Tunnel | **$0** + electricity | Genuinely free, no vendor risk. Best option if you have spare hardware. |
| netcup VPS piko | ~€1.54/mo net | Cheapest credible rented box, 1 GB RAM — enough |
| Contabo Cloud VPS 10 | ~€4.50/mo | 8 GB RAM, most headroom per euro |
| Hetzner CAX11, IPv6-only | ~€5.99/mo net | 2 ARM vCPU / 4 GB. Skip the IPv4 charge since the tunnel is outbound-only. |
| Oracle Cloud Always Free | $0 | 2 ARM vCPU / 12 GB, but capacity is hard to get, the allowance was quietly halved in June 2026, and idle accounts can be reclaimed. **Fine as a spare copy, not as your only one.** |

## Three things to know before you commit

1. **Do not leave a page open on your phone while the agent writes to it.**
   SilverBullet does not merge concurrent edits — it keeps both and creates a
   conflict copy.
2. **Renaming a file from outside the app does not update wikilinks.** Rename
   inside SilverBullet, or have your agent rewrite the links itself.
3. **Keep the space under roughly 1,000-2,000 notes.** The first index on a
   phone takes minutes at that size, because the index is built client-side.

Keep the Space directory in git regardless. That is your backup and your undo.
