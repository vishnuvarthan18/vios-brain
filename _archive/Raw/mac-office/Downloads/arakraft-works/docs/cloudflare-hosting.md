# Cloudflare Pages hosting

The website is built and hosted by Cloudflare Pages from the GitHub repo
`aracreate-group/arakraft-works`, and served at https://arakraft.works.
Every push to `main` builds and publishes the site automatically.

Do the steps below in the Cloudflare account that owns the `arakraft.works`
domain.

## 1 Create the Pages project

1. Open the Cloudflare dashboard and go to **Workers & Pages**.
2. Click **Create** → **Pages** → **Connect to Git**.
3. Choose **GitHub**, allow access to the `aracreate-group` organisation, and
   pick the repo **arakraft-works**.
4. Fill in the build settings:

   | Setting | Value |
   | --- | --- |
   | Project name | `arakraft-works` |
   | Production branch | `main` |
   | Framework preset | None |
   | Build command | `npm run build` |
   | Build output directory | `dist` |
   | Root directory (advanced) | `src` |

5. Click **Save and Deploy** and wait for the build to finish (about 2 minutes).
6. Open the `arakraft-works.pages.dev` link it shows and check the site loads.

The website, its `package.json` and its config all live in `src/`, so the
root directory must be `src`. The build output directory is relative to it
(`src/dist`). The Node version comes from `src/.node-version` (22), and the
public address comes from `src/.env.production` (`https://arakraft.works`),
so no environment variables need to be added in the dashboard. If the build
log shows a Node version other than 22, add `NODE_VERSION` = `22` under
**Settings** → **Variables and Secrets**.

## 2 Remove the old redirect

`arakraft.works` currently sends visitors to `aracreate.group`. That rule must
go first, or it will keep overriding the website.

1. Go to **Websites** → **arakraft.works**.
2. Look in **Rules** → **Redirect Rules**, **Rules** → **Page Rules** and
   **Rules** → **Bulk Redirects** for the rule pointing to `aracreate.group`.
3. Delete or turn it off.

## 3 Connect the domain

1. Back in **Workers & Pages** → **arakraft-works** → **Custom domains**.
2. Click **Set up a custom domain**, enter `arakraft.works`, and confirm.
   Cloudflare updates the DNS record for you.
3. Do the same again for `www.arakraft.works`.
4. Wait until both show **Active** (usually a few minutes).

## 4 Send www to the main address

1. Go to **Websites** → **arakraft.works** → **Rules** → **Redirect Rules**.
2. Click **Create rule** and pick the template **Redirect from WWW to root**.
3. Check it redirects `www.arakraft.works` to `https://arakraft.works` with
   status **301**, keeping the path and query string, then **Deploy**.

## 5 Check it works

| Open | Expect |
| --- | --- |
| https://arakraft.works | The website |
| https://www.arakraft.works | Moves to https://arakraft.works |
| https://arakraft.works/anything | The "This page hasn't been made" page |
| Share `https://arakraft.works` in WhatsApp | The "Your ideas. Our craft." preview |

WhatsApp and Facebook save link previews for a while. The
[Sharing Debugger](https://developers.facebook.com/tools/debug/) refreshes
Facebook's copy straight away.

## Other domains

The archived [`netlify.toml`](../.archives/netlify-host/netlify.toml) also lists `arakraft.lk` and `arakraftworks.com` as redirects
to `arakraft.works`. That file is not read by Cloudflare. If those domains are bought later, add each to Cloudflare and
give it a Redirect Rule to `https://arakraft.works`.

## Previous host

Netlify (https://arakraft-works.netlify.app, see
[netlify-hosting.md](../.archives/netlify-host/netlify-hosting.md)) keeps serving the old build until it
is switched off. Leave it running until the Cloudflare site is confirmed
working, then delete the Netlify project.
