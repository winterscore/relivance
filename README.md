# relivance.app

Static "coming soon" landing page for **Relivance**, designed to mirror the
iOS app's login screen one-for-one. Hosted on GitHub Pages, connected to the
custom domain `relivance.app`.

No frameworks. No build step. Edit a file, push to `main`, GitHub serves it.

## What's inside

```
relivance.app/
├── index.html              ← the page (single file)
├── styles.css              ← all visual styling
├── CNAME                   ← tells GitHub Pages which domain to serve
├── assets/
│   ├── logo-light.svg      ← gradient disc, light-mode variant (from the app)
│   ├── logo-dark.svg       ← silver disc, dark-mode variant (from the app)
│   ├── favicon.svg         ← compact logo for browser tabs
│   └── og-image.svg        ← 1200×630 social-preview card
└── README.md
```

The four SVG assets are the same source files the iOS app ships, copied here
so the brand mark is identical pixel-for-pixel between the app and the web.
Any future logo refresh should update both places.

## Design rationale

The login screen (`AuthView.swift` in the iOS app) is a calm three-line brand
moment: gradient disc → "Relivance" wordmark in Roboto SemiBold 40pt → slogan
in Roboto Regular 17pt, vertically centred on a pure background. This page
preserves all of that one-for-one and adds exactly one element — a small
"Coming soon" capsule below the brand block — plus a tiny footer.

Decisions worth calling out:

- **Same SVGs as the app.** `logo-light.svg` and `logo-dark.svg` are byte-for-
  byte copies of `Assets.xcassets/LaunchLogo.imageset/*`. Light mode renders
  the blue gradient disc; dark mode renders the silver gradient disc. The
  switch is driven by `<picture><source media="(prefers-color-scheme: dark)">`,
  which is native browser behaviour — no JS, no flash-of-wrong-theme.
- **Same colour tokens.** Every CSS variable in `:root` is the sRGB hex
  equivalent of a `wv*` colour-set in the app's asset catalog
  (`wvBackground`, `wvPrimary`, `wvSecondary`, `wvSurface`, `wvBorder`,
  `wvAccent`). Light and dark values mirror the asset catalog's two
  appearance variants. Reference table is inline in `styles.css`.
- **Same typography.** Roboto is loaded from Google Fonts at the same two
  weights the app uses (`400` for slogan / body, `600` for the wordmark).
  System fonts cover the brief moment before Roboto loads.
- **Same vertical rhythm.** The 24pt logo-to-title gap and 10pt title-to-
  slogan gap from `AuthView` are preserved as CSS variables, and the same
  -6pt optical-centre nudge on the logo is here too.
- **The slogan stands alone.** The brief allowed an optional supporting
  line; the slogan is already the strongest single line, so the supporting
  line is omitted. The page reads as elegant, not wordy.
- **The "Coming soon" capsule is calm, not loud.** Surface fill, secondary
  text, a 7px accent-blue dot with a 2.4s gentle pulse. Respects
  `prefers-reduced-motion`.
- **No JavaScript.** A single one-liner updates the footer year. Disabling
  JS leaves the page intact.

## Local preview

Any static-file server works. Two options:

```sh
# Python (no install needed on macOS)
cd relivance.app
python3 -m http.server 5173
# open http://localhost:5173
```

```sh
# npx, if you have Node installed
cd relivance.app
npx --yes serve@latest -l 5173
```

Toggle macOS or iOS into dark mode while the page is open to see the dark
variant. The logo and palette swap together; no reload required.

## GitHub Pages — exact setup

The repo can sit either at the root of your existing iOS repo or in its
own dedicated repo. A dedicated repo is cleaner for a marketing site and is
the assumption below.

### 1. Push to GitHub

```sh
cd relivance.app
git init -b main
git add .
git commit -m "Initial coming-soon page"
# Create the empty repo on GitHub first (https://github.com/new),
# then add it as origin:
git remote add origin git@github.com:<your-org>/relivance.app.git
git push -u origin main
```

### 2. Enable Pages

1. On GitHub, open the repo → **Settings** → **Pages**.
2. **Source**: `Deploy from a branch`.
3. **Branch**: `main`, **folder**: `/ (root)`. Click **Save**.
4. Wait ~30–60 seconds. Pages will show the temporary URL
   `https://<your-org>.github.io/relivance.app/`. Open it — the page is
   already live there.

### 3. Connect the custom domain

The `CNAME` file in this repo already contains `relivance.app`, so GitHub
Pages will pick it up automatically on the next deploy. Double-check:

1. On GitHub, **Settings** → **Pages** → **Custom domain**.
2. It should already read `relivance.app`. If not, type it in and **Save**.
3. GitHub will check the DNS, then show a green checkmark and
   "Your site is published at https://relivance.app/".
4. Once DNS verifies, tick **Enforce HTTPS**. (Available a few minutes
   after DNS resolves — GitHub provisions a free Let's Encrypt cert.)

### 4. DNS configuration at your registrar

`relivance.app` is on the [`.app` TLD](https://get.app), which is
HSTS-preloaded — HTTPS is **mandatory**. GitHub Pages serves HTTPS by
default, so we're fine.

Add these records at your registrar (Google Domains / Squarespace / Cloudflare
/ Namecheap / etc.). The exact UI varies, the records do not:

**Apex (`relivance.app`)** — four A records pointing at GitHub Pages:

| Type | Host | Value         | TTL  |
| ---- | ---- | ------------- | ---- |
| A    | `@`  | `185.199.108.153` | 3600 |
| A    | `@`  | `185.199.109.153` | 3600 |
| A    | `@`  | `185.199.110.153` | 3600 |
| A    | `@`  | `185.199.111.153` | 3600 |

Optional, for IPv6 (recommended):

| Type | Host | Value                   | TTL  |
| ---- | ---- | ----------------------- | ---- |
| AAAA | `@`  | `2606:50c0:8000::153`   | 3600 |
| AAAA | `@`  | `2606:50c0:8001::153`   | 3600 |
| AAAA | `@`  | `2606:50c0:8002::153`   | 3600 |
| AAAA | `@`  | `2606:50c0:8003::153`   | 3600 |

**`www` redirect** (so `www.relivance.app` works too):

| Type  | Host  | Value                       | TTL  |
| ----- | ----- | --------------------------- | ---- |
| CNAME | `www` | `<your-org>.github.io.`     | 3600 |

Replace `<your-org>` with your actual GitHub username/org. The trailing dot
is fine; some registrars strip it automatically.

DNS propagation usually takes 5–30 minutes. Worst case: a few hours. You
can spot-check from the terminal:

```sh
dig +short relivance.app
# expected: 185.199.108.153  (and the other three)
```

When all four A records resolve, GitHub's domain check will go green and
the HTTPS toggle becomes available.

### 5. Cloudflare users — one extra setting

If your DNS is on Cloudflare, set the proxy mode for the four A records to
**DNS only** (grey cloud), not **Proxied** (orange cloud), during the
initial cert provisioning. Once GitHub has issued the Let's Encrypt cert
and "Enforce HTTPS" is toggled on, you can switch the records back to
Proxied if you want Cloudflare's CDN in front. With Proxied on day one,
GitHub's ACME challenge can't reach the origin and the cert never issues.

## Updating the page

```sh
# Edit index.html or styles.css
git add .
git commit -m "Tweak slogan size"
git push
# Pages redeploys in ~30 seconds; hard-refresh the browser to see it.
```

## Optional: better social-preview compatibility

`assets/og-image.svg` is an SVG OG card. Twitter renders SVG OG images
correctly; some other platforms (older Facebook crawlers, LinkedIn) still
prefer raster. If you want maximum compatibility, rasterise it to PNG:

```sh
# Requires librsvg installed (brew install librsvg)
rsvg-convert -w 1200 -h 630 assets/og-image.svg > assets/og-image.png
```

Then change the `og:image` and `twitter:image` `<meta>` tags in `index.html`
from `og-image.svg` to `og-image.png`. Commit and push.

## What this page deliberately does not do

- No newsletter capture, no waitlist form, no "Notify me" button. The brief
  asked for elegant and calm; collection forms read as marketing.
- No App Store / TestFlight link. The app isn't shipping yet, and a dead
  link reads worse than no link.
- No animation beyond the 2.4s status-dot pulse (and that respects
  `prefers-reduced-motion`).
- No tracking pixels, no analytics. Add later if you want — Plausible or
  Vercel Analytics drop in cleanly as a single script tag.
