# Vaizle Technologies — Go‑Live Guide

Deploy the optimized website to **Cloudflare Pages** (free) and point **vaizletech.com**
(registered at GoDaddy) at it. Budget ~20 minutes of clicking + up to a few hours for DNS to
propagate the first time.

The site you're deploying is the `vaizletech-site` folder (also provided as
`vaizletech-site.zip`). Everything is static — no build step, no server.

---

## What's in the build

```
index.html          The website
404.html            Branded "page not found" page
robots.txt          Tells search engines they can crawl + where the sitemap is
sitemap.xml          Sitemap (update <lastmod> when you change the site)
site.webmanifest    PWA manifest (name, icons, theme colour)
_headers            Security + caching headers (Cloudflare reads this automatically)
_redirects          Sends www.vaizletech.com → vaizletech.com
assets/             Images, favicons, social-share (OG) image
```

You never edit `_headers` or `_redirects` in the dashboard — Cloudflare Pages applies them
from these files on every deploy.

---

## Part A — Deploy to Cloudflare Pages

There are two ways. **Direct Upload** is the fastest and needs no GitHub account. Use it unless
you already keep the site in Git.

### Option 1 — Direct Upload (recommended, ~5 min)

1. Go to **https://dash.cloudflare.com** and sign up / log in (free).
2. Left sidebar → **Compute (Workers & Pages)** → **Create** → **Pages** tab →
   **Upload assets**.
3. Project name: `vaizletech` (this becomes `vaizletech.pages.dev`).
4. **Drag the *contents* of the `vaizletech-site` folder** into the upload box —
   i.e. drop `index.html`, `assets/`, `_headers`, etc. directly.
   ⚠️ Do **not** drop the outer folder itself, or your site will live at
   `/vaizletech-site/index.html` instead of the root.
5. Click **Deploy site**. In a few seconds you'll get a live URL like
   `https://vaizletech.pages.dev` — open it and confirm everything looks right.

To publish an update later: same project → **Create new deployment** → drag the contents again.

### Option 2 — Git (if you want version control)

1. Put the `vaizletech-site` contents in a GitHub repo (root of the repo = the site).
2. Cloudflare → **Workers & Pages** → **Create** → **Pages** → **Connect to Git** →
   pick the repo.
3. Build settings: **Framework preset = None**, **Build command = (leave empty)**,
   **Build output directory = `/`**. Deploy.
4. Every `git push` now redeploys automatically.

---

## Part B — Connect vaizletech.com

Cloudflare needs to serve your custom domain. There are two routes. **Route 1 is strongly
recommended** — it makes the apex domain (`vaizletech.com` with no `www`), `www`, SSL, and the
`_redirects`/`_headers` all work correctly and gives you Cloudflare's CDN. Route 2 keeps DNS at
GoDaddy but is more fiddly for the apex domain.

### Route 1 — Move DNS to Cloudflare (recommended)

You keep the domain **registered** at GoDaddy; only the DNS (nameservers) moves to Cloudflare.

**Step 1 — Add the domain to Cloudflare**
1. Cloudflare dashboard → **Add a site** (or **+ Add** → **Existing domain**).
2. Enter `vaizletech.com`. Choose the **Free** plan.
3. Cloudflare scans your current DNS and shows the records it found. Review them — make sure
   anything you rely on is present, especially **email (MX) records** if you receive mail at
   `@vaizletech.com`. Add any that are missing (see the note below).
4. Cloudflare shows you **two nameservers**, e.g.
   `adam.ns.cloudflare.com` and `zara.ns.cloudflare.com` (yours will differ). Copy both.

**Step 2 — Point GoDaddy at Cloudflare's nameservers**
1. Log in to **GoDaddy** → **My Products** → find `vaizletech.com` → **DNS** (or the
   three‑dot menu → **Manage DNS**).
2. Scroll to **Nameservers** → **Change** → **Enter my own nameservers (Custom)**.
3. Delete GoDaddy's nameservers and paste **Cloudflare's two** nameservers.
4. **Save**. GoDaddy will warn you it may take time — that's normal. Propagation is usually
   under an hour but can take up to 24–48 h.
5. Back in Cloudflare, click **Check nameservers**. When it flips to **Active**, DNS is live on
   Cloudflare. (Cloudflare also emails you.)

**Step 3 — Attach the domain to your Pages project**
1. Cloudflare → **Workers & Pages** → your `vaizletech` project → **Custom domains** tab →
   **Set up a custom domain**.
2. Enter `vaizletech.com` → **Continue** → **Activate domain**. Because DNS is now on
   Cloudflare, it creates the record and issues the SSL certificate automatically (usually a
   couple of minutes, occasionally up to ~15).
3. Repeat and add `www.vaizletech.com` as well. The included `_redirects` file then bounces
   `www` → the apex, so both work and search engines see one canonical address.
4. Visit **https://vaizletech.com** — you should see the site on HTTPS with a valid padlock.

> **Email note:** Moving nameservers moves *all* DNS to Cloudflare. If you use Google Workspace,
> Microsoft 365, GoDaddy email, or any mailbox at `@vaizletech.com`, re‑create those **MX**
> records (and any **SPF/DKIM/DMARC TXT** records) in Cloudflare's DNS tab, or mail will stop.
> Cloudflare's initial scan usually imports them — just verify before you switch. This does not
> affect the FormSubmit contact form, which sends *from* Cloudflare, not through your MX.

### Route 2 — Keep DNS at GoDaddy (alternative)

Use this only if you can't move nameservers.

1. In your Pages project → **Custom domains** → add **`www.vaizletech.com`**. Cloudflare will
   show a target like `vaizletech.pages.dev` and ask you to create a CNAME.
2. In **GoDaddy → Manage DNS**, add a record:
   - **Type:** CNAME **Name:** `www` **Value:** `vaizletech.pages.dev` **TTL:** 1 hour
3. For the **apex** `vaizletech.com`: GoDaddy does **not** allow a CNAME on the root domain, so
   use **GoDaddy → Domain Settings → Forwarding → Add** and forward
   `vaizletech.com` → `https://www.vaizletech.com` with a **permanent (301)** redirect.
4. SSL for `www` is issued automatically by Cloudflare Pages once the CNAME resolves.
   Downside: the apex relies on GoDaddy forwarding and you don't get Cloudflare's CDN/`_headers`
   on the apex. This is why Route 1 is preferred.

---

## Part C — After it's live (5‑minute checklist)

- [ ] **Activate the contact form.** It posts to **FormSubmit** (`formsubmit.co`). The **first**
      time someone submits, FormSubmit emails `hello@vaizletech.com` a one‑time confirmation
      link — click it once and the form works forever after. Send a test message yourself to
      trigger it.
- [ ] **Force HTTPS.** Cloudflare → your domain → **SSL/TLS** → **Edge Certificates** → turn on
      **Always Use HTTPS**. Set **SSL/TLS mode** to **Full** (not Flexible).
- [ ] **Test on your phone** and on `www.` vs no‑`www` — both should land on
      `https://vaizletech.com`.
- [ ] **Submit the sitemap** to Google: **Google Search Console** → add `vaizletech.com` →
      verify (Cloudflare makes the TXT verification easy) → **Sitemaps** → submit
      `https://vaizletech.com/sitemap.xml`.
- [ ] **Check the social preview:** paste `https://vaizletech.com` into
      https://www.opengraph.xyz to see the share card (the OG image is already set).

---

## What changed from your original file (why it's faster now)

- **Removed the Tailwind CDN** — it was a development‑only build that printed a
  "don't use in production" warning and blocked page rendering; your CSS is hand‑written and
  didn't need it.
- **Removed the Font Awesome CDN** — all ~35 icons are now inline SVG, so there's no external
  request and no flash of missing icons.
- **Pulled the images out of the HTML.** They were embedded as base64, which made a single
  **956 KB** HTML file that couldn't cache. HTML is now **~136 KB** and images are separate,
  cacheable files (set to cache for a year via `_headers`).
- **Added SEO + social:** Open Graph / Twitter cards, a branded share image, canonical URL,
  JSON‑LD Organization schema, `sitemap.xml`, `robots.txt`, favicons, and a web manifest.
- **Security headers** (CSP, HSTS, X‑Content‑Type‑Options, etc.) via `_headers`.
- **UI polish:** consistent focus rings for keyboard users, subtle button interactions,
  lazy‑loaded below‑the‑fold image, and reduced‑motion support — the visual design and all your
  content are unchanged.

*The layout, copy, projects, and interactive demo are exactly as you designed them — this pass
optimised delivery and added the production plumbing, not the look.*

---

## Handy links

- Cloudflare dashboard: https://dash.cloudflare.com
- Cloudflare Pages docs (custom domains): https://developers.cloudflare.com/pages/configuration/custom-domains/
- GoDaddy — change nameservers: https://www.godaddy.com/help/change-nameservers-for-my-domains-664
- FormSubmit: https://formsubmit.co
