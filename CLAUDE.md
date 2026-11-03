# lumyx-dev/self-hosted

Docker Compose stack to self-host Lumyx: `sfu-core` + `dashboard` behind Caddy (automatic HTTPS). No application code here.

- Files: `compose.yaml` (pulls `ghcr.io/lumyx-dev/*` images), `compose.build.yaml` (builds from sibling `sfu-core` / `dashboard` checkouts), `Caddyfile`, `.env.example`.
- Check: `docker compose config` must pass; test locally per README ("Local test").
- All configuration goes through `.env`; never commit `.env`. Keep README's variable table in sync with `compose.yaml`.
- The SFU advertises its own UDP ports in ICE candidates: host and container port ranges must stay identical.
- Commits and PRs never mention Claude: no `Co-Authored-By: Claude` trailer, no "Generated with Claude Code" line, no reference to AI in messages or descriptions.
