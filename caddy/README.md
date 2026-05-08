# Caddy + Tailscale Sidecar — Chelsea Girl Quiz

This Docker Compose stack serves the static `index.html` (the Chelsea Girl Quiz) from the repo root via [Caddy](https://github.com/caddyserver/caddy-docker), exposed only on your private Tailnet through a [Tailscale](https://tailscale.com/) sidecar.

## What runs

- `tailscale` — sidecar container that joins this service to your Tailnet as `${SERVICE}.${TAILNET}.ts.net`.
- `application` — Caddy, sharing the Tailscale network namespace (`network_mode: service:tailscale`), serving `../index.html` at `/srv/index.html`.

## First-time setup

1. Copy the env template and fill it in:

   ```bash
   cp .env.example .env
   ```

   Required values:
   - `SERVICE` — Tailscale hostname for this node (default `chelsea-girl-quiz`).
   - `TAILNET` — your tailnet name (find it at <https://login.tailscale.com/admin/dns>).
   - `TS_AUTHKEY` — generate at <https://login.tailscale.com/admin/settings/keys>.
   - `TZ` — your timezone.

2. Bring it up:

   ```bash
   docker compose up -d
   ```

3. Visit `http://chelsea-girl-quiz.<your-tailnet>.ts.net` from any device on your Tailnet.

## Enabling HTTPS via Tailscale

If you have [HTTPS](https://tailscale.com/kb/1153/enabling-https) and [MagicDNS](https://tailscale.com/kb/1081/magicdns) enabled in your Tailscale admin console, you can get an automatic Let's Encrypt cert:

1. In `Caddyfile`, drop the `http://` prefix from the site address.
2. In `compose.yaml`, uncomment the two `tailscaled.sock` / `./tailscale/tmp` volume lines (in both the `tailscale` and `application` services).
3. `docker compose up -d --force-recreate`.

The cert takes ~20s to be procured on first visit. See [Caddy certificates on Tailscale](https://tailscale.com/kb/1190/caddy-certificates).

## Editing the site

`index.html` is mounted read-only from the repo root. Edit it in place and refresh — no rebuild needed.
