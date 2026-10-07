---
tags: chat
date: 2026-06-12
source: Claude personal account
uuid: ad4d266c-c5fe-40fc-ac82-869127fdd260
---
# Forms not working after webflow export

## Summary
**Conversation Overview**

The person is working on a Webflow site that has been exported for self-hosting, and encountered the common issue of forms breaking after export since Webflow's form handling only works on Webflow-hosted sites. The conversation focused on finding a free alternative solution to restore form functionality on the exported site. The person is working on a client site (SinoLink, email sinolink@sinolink.de) and needed a practical, easy-to-implement fix with no backend setup.

Claude recommended Web3Forms over Formspree due to its unlimited free tier submissions compared to Formspree's 50/month cap. The implementation approach discussed was a simple HTML edit: changing the form's `action` attribute to the Web3Forms endpoint and adding a hidden input field containing the access key. Claude also provided guidance on using Claude Code to automate the edits across all exported HTML files. A key consideration raised was handling Webflow's `.w-form-done` / `.w-form-fail` success/fail elements, which rely on Webflow's JS and won't trigger post-export, with a redirect field suggested as the workaround.

The person then asked about testing the form locally and how to direct submissions to the client's email rather than their own. Claude explained that Web3Forms access keys are tied to the email used during registration, and recommended registering a fresh key under the client's email for a clean handoff. The person's final question confirmed they wanted to know if the email destination could be changed later — Claude confirmed it can by generating a new key and swapping the `value` in the hidden input, noting that existing keys cannot be repointed to a different email but that generating a new one is quick.

## Chat

**Vishnu:** hey first i have the site in the webfloe now i exported the site the poblem is fomrs not it now working i need a aclternative way to fix is there any easy way free

**Claude:** Webflow forms break on export because Webflow's form handling backend only works on Webflow-hosted sites. Once exported, the `<form>` submits nowhere.

**Best free fix: Formspree**

1. Sign up at `formspree.io` (free tier: 50 submissions/month).
2. In your exported HTML, find the `<form>` tag and change it:

```html
<form action="https://formspree.io/f/YOUR_FORM_ID" method="POST">
```

3. Make sure each input has a `name` attribute (Webflow usually adds these already):

```html
<input type="email" name="email" required>
<textarea name="message"></textarea>
```

4. Remove Webflow's `data-name` / `action` leftovers if they conflict, and delete the `.w-form-done` / `.w-form-fail` reliance — or keep them, Formspree redirects by default.

Submissions land in your email. No backend needed.

**Alternatives if 50/mo is too low:**

- **Web3Forms** — unlimited submissions, free, no account (just an access key). Same pattern: `action="https://api.web3forms.com/submit"` plus a hidden `<input type="hidden" name="access_key" value="...">`.
- **Getform** — free tier 50/mo, similar setup.

Recommendation: **Web3Forms** — unlimited and no signup beats Formspree's cap for a client site.

Want me to rewrite your exported form's HTML? Paste the current `<form>` block.

**Vishnu:** which is best

**Claude:** **Web3Forms.**

- Unlimited submissions on free tier — Formspree caps at 50/month, which a real client site will blow through.
- No account required, just an access key tied to your email.
- Same drop-in setup, spam filtering included.

The only reason to pick Formspree instead is if you want a dashboard to view/manage submissions rather than just receiving emails. For SinoLink's contact form, you don't need that — Web3Forms wins.

**Vishnu:** how to install that to the code with the help of claude

**Claude:** Don't install anything — Web3Forms is just an edited form action plus a hidden key. Here's the exact flow with Claude Code:

**1. Get your access key**
Go to `web3forms.com`, enter `sinolink@sinolink.de`, and they email you an access key. Copy it.

**2. Open the exported site in Claude Code**

```bash
cd ~/path/to/exported-site
claude
```

**3. Tell Claude Code:**

> Find every `<form>` in the exported HTML. For each one:
> - Set `action="https://api.web3forms.com/submit"` and `method="POST"`
> - Add `<input type="hidden" name="access_key" value="YOUR_KEY_HERE">` as the first child
> - Make sure every input/textarea has a `name` attribute
> - Add `<input type="hidden" name="redirect" value="https://sinolink.de/thank-you">` if I want a redirect, otherwise leave it
> - Keep the existing Webflow markup and classes intact

That's the whole job — Claude Code edits the files in place. No package, no build step.

One thing to verify after: Webflow wraps forms in a `.w-form` div with `.w-form-done` / `.w-form-fail` success/fail messages that rely on Webflow's JS. With Web3Forms those won't trigger, so use the `redirect` field above to send users to a thank-you page instead.

Paste your `<form>` block here if you'd rather I rewrite it directly right now.

**Vishnu:** now i need to test that and aftet that i need to give that to the clinet email

**Claude:** Two separate things here — test first, then hand off. 

**Test it (local won't work, needs a live URL):**

Web3Forms requires the form to submit from a real domain, not a `file://` path. Two free options:

1. **Netlify Drop** — drag your exported folder to `app.netlify.com/drop`, get a live URL in seconds, submit the form, check `sinolink@sinolink.de` for the email. Fastest.
2. **Webflow staging** — won't help, since Web3Forms is for the exported version specifically.

Submit a real test entry, confirm the email arrives, check it's not in spam (mark "not spam" if so — improves future deliverability).

**Then the client handoff** — this is the part to get clear on. Who controls the access key matters:

The Web3Forms access key is tied to whatever email you entered. If you used `sinolink@sinolink.de`, submissions already go to the client — good. If you used your own email during testing, regenerate the key against the client's address before handing off, or they'll receive nothing.

What does the handoff actually involve? It changes the instructions significantly:

_[tool: ask_user_input_v0]_

**Vishnu:** Q: How is the client receiving the site?
A: I'm deploying it for them

**Claude:** Then deployment and handoff collapse into one clean flow. Recommendation: **Netlify** — free, custom domain support, and you keep deploy control.

**1. Deploy to Netlify**
- Drag the exported folder to `app.netlify.com/drop`, or connect a Git repo for future updates.
- You get a `*.netlify.app` URL immediately.

**2. Point sinolink.de at it**
- Netlify → Domain settings → add `sinolink.de` and `www.sinolink.de`.
- Netlify gives you DNS records (or nameservers). The client's domain registrar needs these updated — either you get registrar access, or you send the client the exact records to paste. SSL auto-provisions once DNS resolves.

**3. Verify the access key goes to the client**
- Confirm the Web3Forms key was generated against `sinolink@sinolink.de`, not your test email. If you tested with your own address, regenerate it now against the client's email and update the `value` in the hidden input before final deploy.

**4. Final live test**
- Once `sinolink.de` resolves, submit the contact form on the real domain and confirm `sinolink@sinolink.de` receives it. Test from the production URL, not the staging one — deliverability can differ.

**5. What to actually send the client**
A short email: the live URL, confirmation the contact form delivers to their inbox (tell them to check spam once and mark "not spam"), and where form submissions land. Don't hand over the Web3Forms key or Netlify login unless they asked — you're managing it.

One caveat worth flagging to them: Web3Forms free tier has no submission dashboard, so the email inbox *is* the record. If they want a searchable log of inquiries later, that's a paid upgrade or a different tool.

Want the handoff email drafted?

**Vishnu:** now i need to test that and aftet that i need to give that to the clinet email i am skong about fomrms

**Claude:** Two separate things — test first, then point submissions at the client.

**1. Test it**

Open the exported page in a browser (locally is fine — double-click the HTML or run a quick server):

```bash
python3 -m http.server 8000
```

Then go to `http://localhost:8000`, fill the form, submit. Web3Forms works from `localhost`, so you'll get the email within seconds. Check the inbox tied to your access key.

If it fails: open browser DevTools (F12) → Console/Network tab, submit again, and look at the response from `api.web3forms.com`. Usual culprits are a missing `name` attribute on a field or a wrong access key.

**2. Send submissions to the client's email**

The access key is tied to whatever email you registered with. You have two options:

- **Register the key with the client's email directly** — go to `web3forms.com`, enter the client's address, use that key. Submissions go straight to them.
- **Keep your key but redirect mail** — add a `replyto` and CC field so you both get it:

```html
<input type="hidden" name="ccemail" value="client@theircompany.com">
```

Recommendation: register a fresh key under the **client's own email** so the form belongs to them, not you — cleaner handoff, and they keep control if you stop working on the site. Generate it, swap the `access_key` value, re-test once, done.

One catch: the free tier key lives in the client's inbox-linked account. If they want a dashboard or higher limits later, they upgrade — your job is just wiring it correctly.

**Vishnu:** my only question is if i set this wiht a email can i change it later

**Claude:** Yes. The `access_key` is just a value in your HTML — swap it anytime and re-deploy.

Each key is tied to one email. To change the destination email later, you generate a **new key** under the new email at `web3forms.com`, then replace the `value` in the hidden input:

```html
<input type="hidden" name="access_key" value="NEW_KEY_HERE">
```

You can't repoint an existing key to a different email — but generating a new one takes 10 seconds, and editing one line is trivial. So you're never locked in.

For a clean client handoff: register with the client's email now so the key is theirs from the start. But if you set it up under your email today and switch later, that's completely fine.
