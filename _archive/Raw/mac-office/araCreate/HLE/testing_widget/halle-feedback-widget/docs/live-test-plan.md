# LIVE SITE TEST PLAN — M2b

**The widget on B. Halle's real published Webflow site, served from Vishnu's
Mac through a tunnel.** No hosting account, no cloud service. The app,
database and screenshots all stay on the Mac.

This is the day surprises land. Everything before this ran on a test page in
one browser.

| Part | What it covers |
| --- | --- |
| [0 Security first](#0-security-first) | Read before opening a tunnel |
| [1 Setup](#1-setup) | What the agent builds |
| [2 Install on Webflow](#2-install-on-webflow) | The one line, and publishing |
| [3 The live checks](#3-the-live-checks) | Only the real site can answer these |
| [4 The Safari and iPad round](#4-the-safari-and-ipad-round) | Where capture breaks |
| [5 The one unverified risk](#5-the-one-unverified-risk) | Does capture disturb Webflow |
| [6 Results](#6-results) | What to send back |

---

## 0 Security first

**A tunnel puts your whole app on the public internet for as long as it is
running — including the dashboard login.** Anyone with the address can reach
the login page. Before opening one:

1. **Change all three account passwords to strong ones.** The shared demo
   password from `make demo` must never be live on a tunnel. Use
   `make user-create` with proper passwords, or ask the agent for a
   `make user-password` command.
2. **Only run the tunnel while you are actively testing.** Close it after.
   Every session gets a new address, so a leaked old one stops working.
3. **Do not put real personal data in during this round.** Testers come later,
   with real consent and a retention policy. Today is you, clicking.
4. The login lockout and the append-only rules are already in place, so the
   exposure is bounded — but the honest position is that a tunnel is for
   testing, not for running a real test round with 30 elderly people. That
   needs a proper decision about hosting.

---

## 1 Setup

The agent needs to do three things first — the prompt for it is in the chat.

1. **Serve `v1.js` and `capture.js` from the Next app**, not the separate
   static server, so one tunnel covers everything. Both need
   `Access-Control-Allow-Origin: *` — the capture chunk especially, because a
   cross-origin ES module fetch is refused without it, and a classic script
   tag is not. That difference is the most likely thing to break here.
2. **A `make tunnel` target** that opens the tunnel and prints the address.
3. **Print the exact script tag to paste**, with the real public key and the
   tunnel address already filled in.

---

## 2 Install on Webflow

The tag goes in **Site Settings → Custom Code → Footer**, then **Publish**.

```html
<script src="https://<tunnel>/v1.js" data-key="pk_live_xxxxxxxx"
        data-api="https://<tunnel>" defer></script>
```

Three things to know:

- **Custom code only runs on the PUBLISHED site**, never in Designer preview.
  Every change needs a publish. Budget for the cycle.
- **The tunnel address changes** each time you restart it, so the tag needs
  updating and the site republishing each session. Annoying but harmless.
- **Webflow's custom code field caps at 50,000 characters.** One script tag is
  nowhere near it.

---

## 3 The live checks

Open the site with a tester token on the end:
`https://halle-dev.webflow.io/?t=<token>`

1. **The widget appears at all.** If not, check: is the site published, is the
   tunnel up, and does the plan allow custom code.
2. **Without the token, the launcher does NOT appear.** This is what keeps a
   feedback button away from B. Halle's real visitors. Check it on two or
   three pages.
3. **A CMS product page.** Site-wide code should cover it, but confirm — the
   product pages are the bulk of the 49.
4. **The 404 page.** Visit a nonsense URL like
   `halle-dev.webflow.io/does-not-exist`. Does the widget appear? This has
   been an open question since the first day and nobody has checked it. The
   404 is one of the 49 pages.
5. **Clicking a real nav link selects it instead of navigating.** This worked
   on the test page. Webflow's real navigation is the harder case.
6. **The token survives moving between real pages.** Click through three
   pages using the site's own menu, then open the widget — it should still
   know who you are, and the address bar should still carry `t=`.
7. **Nothing is written to your browser.** In Safari or Chrome developer
   tools, check Local Storage, Session Storage and Cookies after a full
   report. All must be untouched by us. This is the German device-storage rule
   and the reason there is no cookie banner.
8. **A real page with real images.** File a report on a product page and look
   at the screenshot. Webflow serves images from its own CDN, and if the
   crossorigin handling is wrong they come out blank or missing.
9. **A lazy-loaded image.** Scroll down a long page so images load in, then
   capture. Are they in the picture?
10. **The sticky header.** Hover an element near the top, scroll slightly,
    then click. Does it select what you actually clicked?
11. **A cookie banner or embed, if the site has one.** Google Maps, YouTube,
    or a consent bar. These are cross-origin frames and cannot be pointed
    into or captured — that is a browser rule, not a bug. Confirm the widget
    fails gracefully rather than breaking.
12. **Page speed.** Does the site feel any slower with the script in? It is
    6 KB and deferred, so it should be imperceptible.
13. **Then look at the dashboard.** All your reports should be there, on the
    right pages, with the right element text.

---

## 4 The Safari and iPad round

Do the whole of section 3 again on Safari, and again on an iPad if you have
one. This is not optional padding — it is where screenshot code fails.

14. **Safari: the screenshot is a real image, not a blank white box.** The
    first capture on Safari is documented as blank across libraries; the code
    captures twice and keeps the second. This is where you find out whether
    that works.
15. **iPad: the two-tap confirm.** First tap highlights, asks "is that
    right?", second tap confirms. Elderly users on iPads mis-tap constantly,
    which is why it exists.
16. **iPad: are the buttons big enough for a real finger?**
17. **iPad: does the on-screen keyboard cover the Send button** when typing a
    note? A common and fatal little bug.

---

## 5 The one unverified risk

This is the item logged in `docs/blocked.md` that could not be checked
locally, and it needs a careful look.

To take a picture, the code briefly attaches a hidden copy of the page into
the page — about 150 to 200 milliseconds. On a plain test page that is
harmless. On a Webflow site it might not be, because Webflow's interactions
engine watches the page for changes.

18. **Take a screenshot on a page with animations** — anything that fades or
    slides in as you scroll. Watch the page while it happens. Does anything
    flicker, re-animate, jump or reset?
19. **Do it on a page with a slider or carousel.** Does it jump position?
20. **Check the browser console** for errors that are not ours.
21. **If the site has analytics**, check afterwards whether the capture
    generated any phantom events.

**If anything in 18 to 21 misbehaves, stop and tell me.** The fix is known —
render the copy inside a hidden iframe instead — but it is real work and we
should only do it if it is needed.

---

## 6 Results

Send me: which numbers passed, which failed, and for each failure what you did
and what happened. Screenshots of anything visual.

**The four that matter most:** 4 (does the 404 page work), 8 (do real images
appear in the screenshot), 14 (Safari), and 18 (does capture disturb the
Webflow page).

**After this round, the honest question becomes hosting.** A tunnel is fine
for you clicking around today. Thirty elderly testers over two weeks, on links
that must keep working, is not something a tunnel from your laptop can carry —
your Mac would have to be awake, online and running the app the whole time.
That is the decision this round should tee up.
