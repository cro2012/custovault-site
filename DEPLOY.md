# Custovault landing — deployment runbook

_Last updated: 13 Jul 2026. Site source: this folder (`11 Website/`). Primary domain: **custovault.com**._

## TL;DR

The site is a static folder — no build step. Recommended host is **Netlify** (free), keeping DNS
on **Porkbun** so the `andy@custovault.com` mailbox is never touched. Two things only Andy can do
(they need a login/payment and can't be automated): **create the Netlify account** and **edit the
Porkbun DNS records**. Everything else in this folder is deploy-ready.

Total time ≈ 15–20 min. Cost: **€0** (Netlify free tier covers a landing page many times over).

---

## What's already prepared in this folder

| File | Purpose |
|---|---|
| `index.html` | The landing page. Self-contained; fonts are self-hosted (no Google Fonts CDN → no visitor IPs sent to Google, cleaner for an EU-sovereignty brand + GDPR). |
| `fonts/` | Schibsted Grotesk (400/500/600/700) + IBM Plex Mono (400/500), woff2. |
| `og-image.png` | 1200×630 social-share card (branded). Shown when the link is pasted into email/LinkedIn/Slack. |
| `404.html` | Branded not-found page. |
| `robots.txt`, `sitemap.xml` | SEO basics; sitemap points at `https://custovault.com/sitemap.xml`. |
| `_redirects` | `www.custovault.com` → apex 301 (Netlify reads this automatically). |
| `_headers` | Security headers (HSTS, CSP, X-Frame-Options, etc.) + long-cache for fonts. |
| `netlify.toml` | Site config (only relevant if later wired to Git for auto-deploy). |

---

## Recommended path — Netlify + Porkbun DNS (email-safe)

### Step 1 — Deploy the files (Andy)
1. Go to **app.netlify.com** → sign up (email or GitHub). *(Account creation is Andy's step.)*
2. **Add new site → Deploy manually.** Drag the **`11 Website`** folder onto the drop zone.
   - Netlify gives an instant HTTPS URL like `random-name-123.netlify.app`. Note this subdomain — it's used in the DNS below. (Optional: **Site settings → Change site name** to something tidy like `custovault`.)
3. Confirm the temp URL renders correctly (hero, fonts, sections).

### Step 2 — Attach the domain in Netlify (Andy)
1. **Domain management → Add a domain** → enter `custovault.com`.
2. Netlify will say it's registered elsewhere and show the DNS target — proceed with **external DNS** (do **not** switch to Netlify DNS; we keep Porkbun so email stays put).

### Step 3 — DNS at Porkbun (Andy)
In Porkbun → **Details** for `custovault.com` → **DNS Records**:

1. **Delete** the existing parking record — the `A` record on host `@` pointing to `44.230.85.241`.
   *(You cannot have both `A` and `ALIAS` on the root.)*
2. **Add** — apex → Netlify:
   - Type `ALIAS` · Host *(blank = root)* · Answer `apex-loadbalancer.netlify.com` · TTL `600`
3. **Add** — www → your site:
   - Type `CNAME` · Host `www` · Answer `<your-site-name>.netlify.app` · TTL `600`
4. **Leave every other record alone** — especially the `MX` and any `TXT`/`CNAME` for email
   (SPF/DKIM). Those keep `andy@custovault.com` working.

### Step 4 — HTTPS + verify (Andy, mostly automatic)
- Back in Netlify domain management, wait for DNS to verify (minutes, occasionally up to a few hours),
  then Netlify auto-provisions a Let's Encrypt certificate. Set **custovault.com** as the **primary
  domain** so `www` 301-redirects to it.
- Check: `https://custovault.com` loads with a padlock; `http://` and `www.` both redirect to it.

---

## Redirect the other domains → custovault.com

`custovault.eu` is registered at Porkbun (and `custovault.ai` if/where it was registered). Point them
at the primary with a free 301 so no second site is needed:

- **Porkbun → the domain → URL Forwarding** → forward `https://custovault.eu` → `https://custovault.com`,
  **301 permanent**, include path/wildcard on. Porkbun issues the redirect SSL automatically.
- If `custovault.ai` is at a different registrar, set the equivalent forward there.

> **Brand note (decision for Andy):** the site, deck and email all use **.com**, so .com is primary here.
> If the EU-sovereignty optics of a **.eu** primary matter more, we can flip it — but that's a broader
> brand call (email address, deck, LOIs all reference .com today). Left as-is unless you say otherwise.

---

## Alternative — Cloudflare Pages (only if we later want the CDN/WAF)

Cloudflare Pages is excellent, but a **bare apex domain requires moving nameservers to Cloudflare**
(verified — Cloudflare Pages can't attach an apex over external DNS). Moving nameservers drags the
**email records with it**, so the Porkbun MX/SPF/DKIM would have to be re-created inside Cloudflare or
the mailbox breaks. Not worth that risk for a static landing right now. Revisit if/when we want
Cloudflare's CDN, caching or WAF in front of the app.

---

## After it's live — checklist

- [ ] `https://custovault.com` loads, padlock valid, `www`/`http` redirect to it.
- [ ] Paste the URL into the **Meta/LinkedIn Post Inspector** and **opengraph.xyz** — confirm the OG card renders.
- [ ] Test the `404` (visit `custovault.com/nope`).
- [ ] **Replace the placeholder CTA.** `index.html` currently mails `andywhitecyprus@icloud.com`
      (search the file for it) — swap to `andy@custovault.com` once that mailbox is set, for a
      consistent domain identity in prospect-facing links.
- [ ] Add a legal footer before wider promotion: imprint + a short privacy line
      (self-hosted fonts + no third-party scripts means there's very little to disclose, but a
      one-line privacy/contact note is good form for EU B2B).
- [ ] Optional: privacy-friendly analytics (Plausible/Umami, EU-hosted) — deferred; not needed for LOI-stage traffic.

## Updating the site later

Edit files in this folder and **re-drag it onto the same Netlify site** (Deploys tab → drag to
redeploy). For hands-off updates, connect this folder as a Git repo and Netlify auto-deploys on push
(`netlify.toml` already set for that).
