# Caddy — Stilgar Reverse Proxy

Caddy is the front-end proxy for all Stilgar services. It handles:
- **Automatic SSL** via Let's Encrypt (HTTPS out of the box)
- **Subdomain routing** — each service gets its own domain
- **HTTP/3** (QUIC) support
- **Internal-only services** — nothing else is exposed on a public port

## Domain Map

| Subdomain | Service | Internal Port |
|-----------|---------|---------------|
| `crm.<CADDY_DOMAIN>` | Twenty CRM | 3000 |
| `paperclip.<CADDY_DOMAIN>` | Paperclip AI OS | 3101 |
| `ceo.<CADDY_DOMAIN>` | NanoClaw CEO | 3000 |
| `hermes.<CADDY_DOMAIN>` | Hermes Agent | 8080 |
| `<CADDY_DOMAIN>` (root) | Redirect to CRM | — |

## Setup

1. **Point DNS** — Add A records for each subdomain to Stilgar's IP (77.42.7.12):
   ```
   crm.yourdomain.com        → 77.42.7.12
   paperclip.yourdomain.com  → 77.42.7.12
   ceo.yourdomain.com        → 77.42.7.12
   hermes.yourdomain.com     → 77.42.7.12
   yourdomain.com            → 77.42.7.12   (optional, for root redirect)
   ```
   Or use a wildcard: `*.yourdomain.com → 77.42.7.12`

2. **Set env vars** in `.env`:
   ```
   CADDY_DOMAIN=yourdomain.com
   CADDY_EMAIL=you@yourdomain.com
   ```

3. **Open firewall ports** on Stilgar: `80` and `443` (TCP + UDP for HTTP/3)

4. **Deploy** — Caddy auto-fetches SSL certs on first request:
   ```bash
   docker compose up -d caddy
   ```

## Adding a New Service

1. Add the service to `docker-compose.yml` with `expose:` (not `ports:`)
2. Add a block to `caddy/Caddyfile`:
   ```
   myservice.{$CADDY_DOMAIN} {
     reverse_proxy myservice-container:PORT
   }
   ```
3. Add an A record in DNS
4. `docker compose up -d --build caddy`

## Wildcard DNS (simplest setup)

If your DNS provider supports wildcards, add:
```
*.yourdomain.com → 77.42.7.12
```
Then any new subdomain just works — no DNS change needed per service.
