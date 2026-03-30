# Docker Registry Options for Stilgar

## Overview

This document covers Docker registry options for hosting the Stilgar container images:
- `ghcr.io/nikushasta2026/paperclip`
- `ghcr.io/nikushasta2026/nanoclaw-ceo`
- `ghcr.io/nikushasta2026/hermes`

---

## Option 1: GitHub Container Registry (ghcr.io) ⭐ RECOMMENDED

**URL:** `ghcr.io/nikushasta2026/<image>:<tag>`

### Pricing
- **Free** for public repositories (unlimited pulls)
- **Free** for private repos with GitHub Free (1 GB storage, 10 GB/month transfer)
- **GitHub Pro ($4/mo):** 2 GB storage, 10 GB/month transfer included
- **GitHub Team ($4/user/mo):** More storage, team access controls
- Additional storage: $0.008/GB/month; additional transfer: $0.50/GB

### Pros
- ✅ Already authenticated via `nikushasta2026` GitHub account
- ✅ Free for public images (zero cost for open-source stack)
- ✅ Tightly integrated with GitHub Actions for CI/CD
- ✅ Supports multi-platform images (amd64/arm64)
- ✅ Access controlled via GitHub PAT or Actions token
- ✅ `GITHUB_TOKEN` auto-auth in Actions (no manual secret setup)
- ✅ No pull rate limits for authenticated users

### Cons
- ❌ Private repos consume storage quota
- ❌ Requires GitHub account (obvious but worth noting)

### Setup

```bash
# Login
echo $GITHUB_TOKEN | docker login ghcr.io -u nikushasta2026 --password-stdin

# Build and push
docker build -t ghcr.io/nikushasta2026/paperclip:latest ./paperclip
docker push ghcr.io/nikushasta2026/paperclip:latest
```

---

## Option 2: Hetzner Registry

Hetzner offers a **Container Registry** as part of Hetzner Cloud, but availability and pricing vary by region.

### Current Status (2025)
- Hetzner has a managed container registry service in beta/limited availability
- Part of Hetzner Cloud infrastructure (not dedicated server by default)
- Pricing is typically very competitive (~€0.01/GB/month)
- Available via the Hetzner Cloud Console at `registry.hetzner.com`

### Pros
- ✅ Co-located with Hetzner infrastructure (fast pull speeds from Stilgar)
- ✅ Data stays in EU (important for GDPR if applicable)
- ✅ Low latency for pulls from Hetzner dedicated servers
- ✅ Competitive pricing

### Cons
- ❌ Not as mature as ghcr.io or Docker Hub
- ❌ Less community documentation
- ❌ Availability may vary by account/region
- ❌ Separate credentials to manage

### Check Availability
Log into [console.hetzner.cloud](https://console.hetzner.cloud) → Look for "Container Registry" in the sidebar.

---

## Option 3: Docker Hub

**URL:** `docker.io/nikushasta2026/<image>:<tag>`

### Pricing
- **Free:** 1 private repo, unlimited public repos
- **Pro ($5/mo):** Unlimited private repos, 50,000 pulls/6h rate limit
- Public images: 100 pulls/6h (unauthenticated), 200/6h (free authenticated)

### Pros
- ✅ Most widely used registry
- ✅ Default in Docker — no registry prefix needed for public images

### Cons
- ❌ **Aggressive pull rate limits** — can break CI/CD on free tier
- ❌ Private images limited on free tier
- ❌ Less integrated with GitHub workflow
- ❌ Docker Desktop requires paid subscription for commercial use

---

## Option 4: Self-Hosted on Stilgar

Run a Docker Registry v2 or Portainer on Stilgar itself.

### Options
- **Docker Registry v2:** `registry:2` official image — simple, lightweight
- **Portainer + Registry:** GUI management layer on top
- **Harbor:** Enterprise-grade, heavy but full-featured

### Pros
- ✅ Zero external dependency
- ✅ Fastest pull speeds (local to Stilgar)
- ✅ No storage limits (bounded by Stilgar disk)
- ✅ Complete control

### Cons
- ❌ You own availability — if Stilgar goes down, no images
- ❌ No CDN — slow pulls from outside Stilgar
- ❌ Requires TLS setup (Let's Encrypt or self-signed)
- ❌ More operational overhead

### Quick Setup (for reference)

```yaml
# docker-compose registry addition
registry:
  image: registry:2
  container_name: stilgar-registry
  restart: unless-stopped
  ports:
    - "5000:5000"
  volumes:
    - registry-data:/var/lib/registry
  environment:
    - REGISTRY_STORAGE_DELETE_ENABLED=true
```

---

## 🏆 Recommendation: GitHub Container Registry (ghcr.io)

**Use `ghcr.io/nikushasta2026/` as the primary registry.**

**Rationale:**

1. **Already authenticated** — the `nikushasta2026` GitHub account is set up; zero additional setup
2. **Free for this stack** — these images can be public (no secrets in images), making storage/transfer free
3. **CI/CD ready** — GitHub Actions can build and push automatically using `GITHUB_TOKEN`
4. **No pull limits** — authenticated pulls from Stilgar won't hit rate limits
5. **Reliability** — GitHub's infrastructure is battle-tested

**For Coolify:** Set `REGISTRY=ghcr.io/nikushasta2026` in Coolify's environment variables. Pre-pull images by running `docker compose pull` before starting services.

**Future consideration:** If Hetzner Registry becomes generally available and you're sensitive to data residency, it's worth switching for the latency improvements (images co-located with server).

---

## CI/CD: GitHub Actions Workflow

Here's a minimal workflow to build and push all three images:

```yaml
# .github/workflows/docker-build.yml
name: Build and Push Docker Images

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_PREFIX: ghcr.io/${{ github.repository_owner }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    strategy:
      matrix:
        service: [paperclip, nanoclaw-ceo, hermes]

    steps:
      - uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push ${{ matrix.service }}
        uses: docker/build-push-action@v5
        with:
          context: ./${{ matrix.service }}
          push: ${{ github.event_name != 'pull_request' }}
          tags: |
            ${{ env.IMAGE_PREFIX }}/${{ matrix.service }}:latest
            ${{ env.IMAGE_PREFIX }}/${{ matrix.service }}:${{ github.sha }}
```
