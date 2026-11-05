# MrDemonWolf fork maintenance

`origin` is MrDemonWolf/moblin_cloud; `upstream` is eerimoq/moblin_cloud. Work through pull requests into `main`. Production remains on its existing pinned images until a separate deployment is approved.

## Upstream updates

The Upstream updates workflow runs daily at 09:23 UTC and can be started manually after it is merged into the default branch. It mirrors upstream main onto `codex/upstream-sync` and opens one update PR. It explicitly dispatches CI when that branch changes, because GitHub does not start ordinary PR workflows for PRs created by `GITHUB_TOKEN`. It never merges or deploys. Review conflicts and merge only after verification. Repository Actions settings must allow Actions to create pull requests. Inactive public forks may have scheduled workflows paused by GitHub. Upstream history rewrites require manual review; the workflow never force-pushes.

The inherited GitHub Pages deployment is restricted to the original upstream repository. Image publishing remains tag-only; do not create release tags until ready to publish fork images. An upstream merge that changes workflows needs review too.

Manual equivalent:

```sh
git fetch upstream
git switch -c codex/upstream-update origin/main
git merge upstream/main
# Resolve conflicts, run checks, then push and open a PR.
```

## Landing page and public frontend settings

Edit `frontend/src/components/CloudLanding.tsx` for the landing page. Shared header/sign-in rendering lives in `frontend/src/components/PageParts.tsx`; assets live in `frontend/public`. Keep branding changes concentrated here to reduce upstream conflicts. Visual branding is a separate next step.

The frontend container writes public `/config.js` at startup. In Dokploy, put these flags on the Environment page and explicitly pass them to the frontend service:

```yaml
environment:
  BACKEND_HOST: ${BACKEND_HOST:?Set the backend host}
  TWITCH_SIGN_IN_ENABLED: ${TWITCH_SIGN_IN_ENABLED:-true}
  KICK_SIGN_IN_ENABLED: ${KICK_SIGN_IN_ENABLED:-false}
  LANDING_DETAILS_ENABLED: ${LANDING_DETAILS_ENABLED:-true}
  TIPPING_ENABLED: ${TIPPING_ENABLED:-false}
```

Only `true` and `false` are accepted. Restart/redeploy the frontend after editing its environment; no image rebuild is needed. The image defaults preserve upstream behavior (both providers and the How it works section visible). The example disables Kick because this instance has no Kick credentials. Set both sign-in flags false to hide all shared sign-in buttons, including the header, landing CTAs and account-link prompts.

Provider secrets belong only in the backend. Match the explicit sign-in flags to the providers configured there; the frontend does not receive or inspect OAuth credentials. Sign-in button visibility is presentation only: keep Cloudflare Access and backend authentication. Hiding sign-in does not invalidate sessions or disable login routes.

Pass `TIPPING_ENABLED` to the backend too, through its existing `env_file` or an explicit environment entry. Set the same value in both containers. `false` hides public tipping pages and share links and returns 404 for public viewer page APIs, including new tip, spin and effect orders. Phone connections, dashboard APIs, page-address discovery, payment webhooks and captures for existing orders remain available. Payment reconciliation continues. Already-open pages may remain visible until their next request or reload. Missing Stripe/PayPal credentials alone do not disable the whole page: non-payment interactions can still be useful. The explicit switch controls the entire public viewer page.

For local Vite development, add the camelCase booleans to `frontend/public/config.js`; the container entrypoint is not used by Vite. Never put secrets there. `config.js` is served with `Cache-Control: no-store` in the container.

Checks: `cd frontend && npm ci && npm test && npm run build`. Test startup configuration defaults, disabled providers and invalid values before deploying a new image. Build frontend and backend from the same reviewed revision when adopting upstream updates.
