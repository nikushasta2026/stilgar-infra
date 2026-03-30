# Stilgar Deployment Guide — Coolify

Step-by-step guide to deploying the Stilgar AI stack on your Hetzner dedicated server using Coolify.

## Prerequisites

- Hetzner dedicated server (Stilgar) with:
  - Ubuntu 22.04 or Debian 12
  - 61 GB RAM
  - Docker installed
- Coolify installed on Stilgar
- GitHub account: `nikushasta2026`
- GHCR authenticated (see [registry.md](./registry.md))
- API keys: Anthropic, Slack Bot Token

---

## 1. Install Coolify on Stilgar

If not already installed:

```bash
# SSH into Stilgar
ssh root@<stilgar-ip>

# Install Coolify
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Coolify runs on port `8000` by default. Access it at `http://<stilgar-ip>:8000`.

---

## 2. Fork / Push the Repo

This repo should be on GitHub as `nikushasta2026/stilgar-infra`.

```bash
# Clone and push if needed
git clone https://github.com/nikushasta2026/stilgar-infra.git
cd stilgar-infra
```

---

## 3. Add GitHub Source in Coolify

1. Open Coolify UI → **Sources** → **Add Source**
2. Choose **GitHub App** or **GitHub PAT**
3. Connect to `nikushasta2026/stilgar-infra`

---

## 4. Deploy Each Service

Coolify supports Docker Compose deployments. Use the full stack `docker-compose.yml`.

### Option A: Docker Compose (recommended)

1. In Coolify: **Projects** → **New Project** → Name it `stilgar`
2. **New Resource** → **Docker Compose**
3. Select your GitHub source → `stilgar-infra` repo
4. Set compose file path: `docker-compose.yml`
5. Add environment variables (see section 5)
6. Deploy

### Option B: Individual Services

Deploy each service separately for independent scaling:

1. **New Resource** → **Docker Compose** → select `./paperclip/Dockerfile`
2. Repeat for `nanoclaw-ceo` and `hermes`

---

## 5. Environment Variables in Coolify

In Coolify's **Environment Variables** tab, add:

```
ANTHROPIC_API_KEY=sk-ant-api03-...
BETTER_AUTH_SECRET=<openssl rand -hex 32 output>
PAPERCLIP_URL=https://paperclip.yourdomain.com
NANOCLAW_CEO_TOKEN=oclaw_...
SLACK_BOT_TOKEN=xoxb-...
SLACK_BOARD_CHANNEL=C0...
HERMES_MODEL=anthropic:claude-sonnet-4-6
REGISTRY=ghcr.io/nikushasta2026
TAG=latest
```

**Never store secrets in the repo.** Coolify encrypts env vars at rest.

---

## 6. Domain / Reverse Proxy Setup

Coolify includes Traefik as a reverse proxy. For each service:

1. In the service settings → **Domains**
2. Set domain (e.g., `paperclip.yourdomain.com`)
3. Enable **HTTPS** (Coolify handles Let's Encrypt automatically)

Recommended domains:
- Paperclip: `paperclip.yourdomain.com` → port 3100
- NanoClaw CEO: `nanoclaw-ceo.yourdomain.com` → port 3000 (optional, may keep internal)
- Hermes: `hermes.yourdomain.com` → port 8080 (optional, may keep internal)

---

## 7. Verify Deployment

```bash
# SSH into Stilgar
ssh root@<stilgar-ip>

# Check all containers running
docker ps

# Check Paperclip health
curl http://localhost:3100/api/health

# Check Hermes health
curl http://localhost:8080/health

# Check logs
docker logs paperclip --tail 50
docker logs nanoclaw-ceo --tail 50
docker logs hermes-agent --tail 50
```

---

## 8. Connect NanoClaw CEO to Paperclip

The nanoclaw-ceo container connects to Paperclip via the `paperclip-net` Docker network. This is handled automatically by docker-compose (`PAPERCLIP_GATEWAY_URL=http://paperclip:3100`).

To verify the connection:
```bash
docker logs nanoclaw-ceo --tail 20
# Look for: "Connected to Paperclip gateway at http://paperclip:3100"
```

---

## 9. Test Hermes Adapter

```bash
# Basic health check
curl http://localhost:8080/health

# Test chat completion
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "hermes",
    "messages": [{"role": "user", "content": "Hello, Hermes!"}]
  }'
```

---

## 10. Build and Push Images to GHCR

To pre-build and push images (faster deploys):

```bash
# Login to GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u nikushasta2026 --password-stdin

# Build and push all
export REGISTRY=ghcr.io/nikushasta2026

docker build -t $REGISTRY/paperclip:latest ./paperclip
docker push $REGISTRY/paperclip:latest

docker build -t $REGISTRY/nanoclaw-ceo:latest ./nanoclaw-ceo
docker push $REGISTRY/nanoclaw-ceo:latest

docker build -t $REGISTRY/hermes:latest ./hermes
docker push $REGISTRY/hermes:latest
```

Or use the GitHub Actions workflow in `.github/workflows/docker-build.yml`.

---

## Resource Allocation (Stilgar 61GB RAM)

| Service | CPU Limit | Memory Limit | Memory Reserved |
|---|---|---|---|
| paperclip | 4 cores | 8 GB | 2 GB |
| nanoclaw-ceo | 2 cores | 4 GB | 1 GB |
| hermes | 2 cores | 4 GB | 512 MB |
| **Total** | **8 cores** | **16 GB** | **3.5 GB** |
| **Available for OS/other** | | **45 GB** | |

---

## Troubleshooting

### Paperclip won't start
- Check `BETTER_AUTH_SECRET` is set and 32+ chars
- Check `ANTHROPIC_API_KEY` is valid
- Check volume permissions: `docker exec paperclip ls -la /data/paperclip`

### NanoClaw CEO can't connect to Paperclip
- Ensure Paperclip is healthy first: `docker ps` → check `healthy` status
- Verify network: `docker network inspect paperclip-net`
- Check `PAPERCLIP_GATEWAY_URL=http://paperclip:3100` is set

### Hermes fails to start
- Check Python dependencies installed: `docker logs hermes-agent --tail 30`
- Verify `ANTHROPIC_API_KEY` is valid
- Try running in debug mode: `docker exec -it hermes-agent bash`

### Out of memory
- Coolify can set swap on Stilgar: Settings → Server → Enable Swap
- Reduce memory limits in docker-compose.yml
