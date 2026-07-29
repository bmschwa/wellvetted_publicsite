# wellvetted.co

Static one-page site, deployed to GitHub Pages via GitHub Actions.

## 1. Repo & Pages setup

1. Push this directory to a new GitHub repo (e.g. `wellvetted/site`).
2. In the repo: **Settings → Pages → Build and deployment → Source** → set to **GitHub Actions**.
   (Not "Deploy from a branch" — the workflow here uses the newer Pages Actions
   deployment, which is what lets you gate it on a specific branch/merge event.)
3. Push to `main` (or edit `.github/workflows/deploy.yml` to watch a different
   branch, e.g. `release`, if you want `main` to stay a working branch and
   only release on merges into `release`).
4. Check the **Actions** tab — the `Deploy site` workflow should run and
   finish green. The **Environments → github-pages** entry will show the live
   URL.

## 2. Custom domain (wellvetted.co)

The `CNAME` file in this repo already tells GitHub Pages to serve
`wellvetted.co`. You still need to point DNS at GitHub:

At your DNS provider, on the apex domain (`wellvetted.co`), add these four
`A` records (GitHub Pages' current IPs — double check
[GitHub's docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
in case they've changed):

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

If you also want `www.wellvetted.co` to work, add a `CNAME` record for `www`
pointing to `<your-github-username>.github.io`, then in repo **Settings →
Pages** set the custom domain field to `wellvetted.co` and check **Enforce
HTTPS** once the certificate provisions (can take up to ~24h after DNS
propagates).

## 3. Matomo integration

You mentioned self-hosting Matomo behind Traefik on your own VPS — that part
is independent of this repo. Once that instance is up (e.g. at
`matomo.yourdomain.com`):

1. In Matomo, create a new **Website** for `wellvetted.co` and note its
   **Site ID**.
2. In `index.html`, find the tracking snippet near the bottom and replace:
   - `MATOMO_URL` → your Matomo domain, e.g. `matomo.yourdomain.com`
   - `MATOMO_SITE_ID` → the numeric site ID Matomo assigned
3. Commit and push — the next deploy will carry the real tracking code.

Notes on the cross-domain setup (Pages-hosted site → self-hosted Matomo):
- The tracker just sends a request to `matomo.php` on your Matomo domain; it
  doesn't need to *read* a cross-origin response, so no CORS configuration
  is required for basic pageview tracking.
- Make sure Traefik terminates TLS for your Matomo domain (matching
  `https://` in the snippet) — mixed-content browsers will otherwise block
  the request since `wellvetted.co` will be served over HTTPS by GitHub
  Pages.
- If you later add a `Content-Security-Policy` header anywhere in front of
  the site, you'll need a `script-src` / `connect-src` allowance for your
  Matomo domain, or the tracker will be silently blocked.
- If you want cookie-based visit tracking to work well across the two
  domains, set Matomo's **Website → General Settings → Cookie domain** and
  review its cross-domain linking docs — not required for basic analytics,
  only for stitching sessions if a visitor moves between the two domains.

## 4. Editing content

Everything is in the single `index.html` file — no build step, no
dependencies. Section markers (`about`, `services`, `contact`) are plain
IDs you can restructure freely. Fonts are loaded from Google Fonts via
`<link>` tags in the `<head>`.
