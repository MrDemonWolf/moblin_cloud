# MrDemonWolf Moblin Cloud fork

Upstream: https://github.com/eerimoq/moblin_cloud (`upstream`). Fork: https://github.com/MrDemonWolf/moblin_cloud (`origin`).

Keep upstream changes reviewable and branding changes concentrated in the frontend landing component, assets and runtime configuration. Use pull requests; production remains on its existing pinned images until Nathanial explicitly requests a deployment.

Runtime browser configuration is public. Never include OAuth secrets, session secrets or device keys. UI visibility controls do not replace backend authentication or Cloudflare Access.

See `docs/fork-maintenance.md` for upstream updates and frontend settings. Validate frontend edits with `npm ci`, `npm test` and `npm run build` in `frontend/`. Preserve unrelated work and upstream attribution.
