# Getting the CMS content — client logos, team, testimonials, services

## Why this is a separate step

The Webflow code export you sent me contains the *layout* of every Collection
list but none of the *content*. That's a deliberate limit on Webflow's side, not
a setting you can flip. Your site has 14 Collections, so it hits hard: client
logos, team members, testimonials, the services and tools lists, blog posts —
all of it comes out of the ZIP empty.

The content does exist in one place we can read: the published site. This tool
loads every page of aracreate.group the way a browser does, so Webflow has
already filled the Collections in, and saves the result as plain HTML.

It only reads. Nothing in your Webflow project is touched.

---

## How to run it

1. Unzip this folder somewhere easy to find — the Desktop is fine.
2. Make sure you're online.
3. Double-click **`Get full website.command`**.

A Terminal window opens and lists each page as it downloads. It takes a few
minutes. When it says **Finished**, you'll have a new folder next to the script:

```
aracreate-website-full/
```

Open that folder and double-click **`Start website.command`** to view the site
at <http://localhost:8000>.

If macOS refuses to open the file ("unidentified developer"), right-click it →
**Open** → **Open**. You only need to do that once.

If it says Python isn't installed, run `xcode-select --install` in Terminal
once, then try again.

---

## What you get

Every page of the live site, with the CMS content baked in, plus every image,
stylesheet, script and font it uses — including the CMS images that live on
Webflow's CDN and were never in the export. All links are rewritten to relative
paths, so the folder works locally, on any web host, or moved anywhere.

It also picks up three SVGs on the About page (`Frame-2420-1.svg`, `image-25.svg`,
`image-27.svg`) that the Webflow export references but forgot to include — they
404 in the ZIP version and will be correct here.

Forms get the same handler as the other build: they validate and show your
designed success message, and start delivering as soon as you put an endpoint in
`js/aracreate-forms.js`.

---

## The trade-off, stated plainly

This is a **snapshot**. It captures the content as it stands the day you run it.

- Publishing a new blog post or adding a client logo in Webflow will *not* show
  up in this folder. Run the script again to refresh it — that's the intended
  workflow, and it's why it's a script and not a one-off.
- Anything driven by Webflow's servers rather than by rendered HTML still won't
  work: site search, password-protected pages, and form delivery through
  Webflow.
- Collection *filtering and sorting* that happens on the server (the Finsweet
  CMS filters on the services pages) may behave differently, since there's no
  Webflow backend answering. The visible content will be there; the interactive
  filtering is worth a click-through to confirm.

If you'd rather not re-run a script every time content changes, the alternative
is pulling the Webflow CMS API and generating pages from it. More setup, but it
becomes a repeatable build you can point at a real host. Say the word and I'll
put that together instead.

---

## If it doesn't go smoothly

The script prints anything it couldn't download at the end. A handful of
failures is normal and harmless — third-party embeds, tracking pixels, and the
`/business`, `/training`, `/ventures` and `/services` links, which 404 on your
live site too because those pages don't exist in Webflow yet.

If something looks wrong, copy the output and send it to me. The messages say
exactly which URL failed and why.

Useful options, if you want them:

```bash
python3 fetch_live_site.py --max-pages 15     # quick trial run
python3 fetch_live_site.py --verbose          # more detail
python3 fetch_live_site.py --site https://aracreate.webflow.io   # staging site
```
