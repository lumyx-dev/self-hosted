# Lumyx self-hosted

Run [Lumyx](https://github.com/lumyx-dev/sfu-core) on your own server: the SFU, the
[dashboard](https://github.com/lumyx-dev/dashboard), and [Caddy](https://caddyserver.com) in front
for automatic HTTPS.

```
                 ┌──────── Caddy :443 ────────┐
browser ──HTTPS──┤ DASHBOARD_DOMAIN → dashboard:3001
                 │ SFU_DOMAIN /ws, /health → sfu:3000
                 └────────────────────────────┘
browser ──UDP 40000-40099──────────────────────► sfu (media)
```

## Requirements

- Docker with Compose v2
- Two DNS records (dashboard and SFU) pointing at the server
- Open ports: `80/tcp`, `443/tcp+udp`, and the media range (`40000-40099/udp` by default)

## Install

```sh
git clone https://github.com/lumyx-dev/self-hosted.git lumyx && cd lumyx
cp .env.example .env   # set the domains and PUBLIC_IP
docker compose up -d
```

The dashboard is at `https://$DASHBOARD_DOMAIN`. Clients connect their WebSocket to
`wss://$SFU_DOMAIN/ws`.

## Configuration

Everything is in `.env`:

| Variable | What it does |
|---|---|
| `DASHBOARD_DOMAIN` / `SFU_DOMAIN` | Public hostnames. Caddy gets a certificate for each |
| `PUBLIC_IP` | IP advertised to browsers in ICE candidates. Must be reachable over UDP |
| `UDP_PORT_MIN` / `UDP_PORT_MAX` | Media port range, one port per connected peer: it caps concurrent participants |
| `DASHBOARD_VERSION` / `SFU_VERSION` | Image tags, pinned to a release of each repo |
| `SFU_REGION`, `SFU_LOG` | Passed through to the SFU |

The SFU's `/metrics` endpoint is not exposed publicly; it stays on the Compose network.

## Upgrading

Bump `DASHBOARD_VERSION` / `SFU_VERSION` in `.env`, then:

```sh
docker compose pull && docker compose up -d
```

## Local test

```sh
cp .env.example .env
# DASHBOARD_DOMAIN=localhost  SFU_DOMAIN=sfu.localhost  PUBLIC_IP=127.0.0.1
docker compose up -d
```

Caddy issues a locally-trusted certificate; your browser may ask you to accept it once.

To build the images from sibling checkouts of `sfu-core` and `dashboard` instead of pulling
them:

```sh
docker compose -f compose.yaml -f compose.build.yaml up -d --build
```

## Known limits

- The dashboard has no authentication yet. It only shows mock data today; put it behind a VPN
  or Caddy `basic_auth` until it talks to the SFU.
- No Postgres yet: the SFU's telemetry persistence is not wired in.
- Images are `linux/amd64` only for now.
