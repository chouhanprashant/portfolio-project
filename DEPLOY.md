# Deploying this static site to Cloudflare Pages

This repository is the complete deployable site. It has no build step: deploy the repository root (`.`). Do not select `dist`, `public`, or another output folder.

## Create the Pages project

1. In Cloudflare, open **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**.
2. Authorize GitHub if prompted and select `chouhanprashant/portfolio-project`, branch `main`.
3. In **Build settings**, select **None** (or leave the framework preset unset), leave **Build command** completely blank, and enter `.` for **Build output directory**.
4. Click **Save and Deploy**. Open the generated `*.pages.dev` URL and check `/`, `/about/`, `/experience/`, `/skills/`, `/blog/`, `/contact/`, and `/certifications/`.

`wrangler.toml` documents the same output directory for local Pages/Wrangler deployments; it does not deploy anything by itself.

## Attach siddeshmandhane.in

1. In the Pages project, open **Custom domains** and add `siddeshmandhane.in` (and `www.siddeshmandhane.in` only if it will be used).
2. Follow Cloudflare’s DNS instructions. If the zone is already on Cloudflare, it creates the required record; otherwise add the exact CNAME it gives you at your DNS provider.
3. Wait for the domain to become **Active**, then make `https://siddeshmandhane.in/` the canonical domain in the Pages dashboard and test it in a private window.
4. Only after the canonical domain works, configure the legacy `siddheshmandhane.in` spelling as a Cloudflare **Redirect Rule**: hostname equals `siddheshmandhane.in`, destination `https://siddeshmandhane.in/$1`, preserving path and query, status **301**. Do not put this host redirect in `_redirects`; keeping it at the edge avoids a DNS/canonical-domain redirect loop.

## Purge cached content

After a deployment or a DNS cutover, go to **Caching** → **Configuration** → **Purge Cache** and choose **Purge Everything**. Then load the domain in a private window and hard-refresh once. HTML is configured to revalidate within five minutes; `/assets/` files are deliberately long-cached and their `?v=` URL version must be bumped when they change.

## Roll back safely

From a clean checkout, revert the deployment commit and push the normal Git history:

```sh
git revert <bad-commit-sha>
git push origin main
```

Cloudflare Pages will deploy the revert automatically. Do not force-push. To return to a known older commit while preserving history, run `git revert <newest-sha>..<older-good-sha>` (or revert individual commits in reverse order), inspect the result, then push.

## Enable GitHub Actions later

No workflow is active in this repository today. When a Pages workflow is intentionally added or restored, first grant the GitHub CLI permission, then restore the workflow and push it:

```sh
gh auth refresh -s workflow
git restore --source=<commit-that-contained-the-workflow> -- .github/workflows/<workflow-file>.yml
git add .github/workflows/<workflow-file>.yml
git commit -m "ci: restore Pages deployment workflow"
git push origin main
```

Replace both placeholders with the relevant historical commit and workflow filename. A dashboard-connected Pages project does not require this workflow; Cloudflare deploys on pushes to `main`.

## Deployment audit notes

The repository contains `wrangler.toml` (Pages output directory `.`), `_redirects`, and a minimal `package.json` with only the local `check:seo` script. It has no `netlify.toml`, `vercel.json`, `CNAME`, or `.github/` workflow. A direct fetch on 2026-10-03 showed the public homepage byte-for-byte matches this repository’s pre-redesign commit `c08ccba`, not redesign commit `4860abc`. That is decisive evidence of a stale deployment (for example, a Pages project attached to an older branch/commit or a different deployment integration), rather than a cache of the redesign. The original import (`8e8ba4d`) also contains the older `#exp`, `#skills`, `#certs`, and `#contact` fragments; the current homepage preserves them for inbound links.

No `.assetsignore` was added: it is a Workers static-assets control rather than a documented Cloudflare Pages deployment filter. Pages treats the repository root as the output here, so `_headers` applies `noindex, nofollow` and `no-store` to the small authoring files (`scripts/`, `PLACEHOLDERS.md`, `package.json`, and `wrangler.toml`) instead. `node_modules` is not tracked and is excluded from the Pages upload by the platform.
