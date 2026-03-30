# Stilgar Infrastructure

Infrastructure-as-code for the **Stilgar** Hetzner dedicated server — Docker containers for the AI company stack managed by Coolify.

```
   ╔══════════════════════════════════════════════════════════════╗
   ║              STILGAR — AI Company Infrastructure             ║
   ║              Hetzner Dedicated Server (61 GB RAM)            ║
   ╚══════════════════════════════════════════════════════════════╝

   ┌──────────────────────────────────────────────────────────────┐
   │                      Coolify (host)                          │
   │          Reverse proxy (Traefik) + Container mgmt            │
   └─────────────────────────┬────────────────────────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
   ┌─────────────┐   ┌──────────────┐   ┌──────────────┐
   │  paperclip  │   │ nanoclaw-ceo │   │    hermes    │
   │  :3100      │◄──│  :3000       │   │  :8080       │
   │             │   │              │   │              │
   │  AI Company │   │  CEO Agent   │   │  Hermes      │
   │  OS         │   │  (OpenClaw   │   │  Agent       │
   │             │   │   nanoclaw)  │   │  (NousRes)   │
   └──────┬──────┘   └──────────────┘   └──────────────┘
          │                  ▲
          │         PAPERCLIP_GATEWAY_URL
          │         http://paperclip:3100
          │
   ┌──────▼──────────────────────────────────────────────┐
   │                  paperclip-net                       │
   │              (shared Docker network)                 │
   └─────────────────────────────────────────────────────┘

          ▲
          │  Board Reports
   ┌──────┴──────┐
   │   Slack     │
   │  (Marcus)   │
   └─────────────┘
```

---

## Containers

### 🗂️ Paperclip — AI Company OS
**Port:** `3100` | **Source:** [paperclipai/paperclip](https://github.com/paperclipai/paperclip)

The central workspace for the AI company. Provides:
- Project/task management for AI agents
- Agent collaboration workspace
- Authentication via BetterAuth
- SQLite (default) or Postgres persistence

### 🤖 NanoClaw CEO — OpenClaw Nanoclaw Node
**Port:** `3000` | **Connects to:** Paperclip via `paperclip-net`

An OpenClaw nanoclaw node running the CEO persona:
- **Name:** CEO (Aria)
- **Focus:** Strategic delegation, P&L, board reporting
- **Never:** Writes code or runs infrastructure commands
- **Reports to:** Board (Marcus) via Slack
- **Delegates to:** CTO, CMO, CFO, CPO via Paperclip

### 🐍 Hermes — NousResearch Hermes Agent
**Port:** `8080` | **Source:** [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

The Hermes agent container for testing the `hermes-local` adapter:
- No Slack integration (testing only)
- OpenAI-compatible API (`/v1/chat/completions`)
- Configurable model backend (Anthropic, OpenRouter)

---

## Quick Start

### 1. Clone and Configure

```bash
git clone https://github.com/nikushasta2026/stilgar-infra.git
cd stilgar-infra

# Copy env template
cp .env.example .env
nano .env   # Fill in your secrets
```

### 2. Generate Secrets

```bash
# Generate BETTER_AUTH_SECRET
openssl rand -hex 32
```

### 3. Build and Start

```bash
# Build all images
docker compose build

# Start the full stack
docker compose up -d

# Check status
docker compose ps
docker compose logs -f
```

### 4. Verify

```bash
# Paperclip
curl http://localhost:3100/api/health

# Hermes
curl http://localhost:8080/health
```

---

## Deploy via Coolify

See [docs/deployment.md](docs/deployment.md) for the full step-by-step Coolify deployment guide.

**Quick summary:**
1. Add `nikushasta2026/stilgar-infra` as a source in Coolify
2. Create a new Docker Compose resource pointing to `docker-compose.yml`
3. Add environment variables from `.env.example`
4. Deploy

---

## Connecting NanoClaw CEO to Paperclip

The nanoclaw-ceo container connects automatically via the `paperclip-net` Docker network. The `PAPERCLIP_GATEWAY_URL=http://paperclip:3100` env var tells the CEO agent where to find Paperclip.

**Requirements:**
1. Paperclip must be healthy before nanoclaw-ceo starts (handled by `depends_on`)
2. `OPENCLAW_AGENT_TOKEN` must be set (obtain from your OpenClaw gateway)

**To verify connection:**
```bash
docker logs nanoclaw-ceo --tail 20
```

---

## Environment Variables

| Variable | Service | Required | Description |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | all | ✅ | Anthropic API key for Claude |
| `BETTER_AUTH_SECRET` | paperclip | ✅ | Random hex secret (run `openssl rand -hex 32`) |
| `PAPERCLIP_URL` | paperclip | ❌ | Public URL (default: `http://localhost:3100`) |
| `NANOCLAW_CEO_TOKEN` | nanoclaw-ceo | ✅ | OpenClaw agent auth token |
| `SLACK_BOT_TOKEN` | nanoclaw-ceo | ✅ | Slack bot token for board updates |
| `SLACK_BOARD_CHANNEL` | nanoclaw-ceo | ❌ | Slack channel ID for board posts |
| `HERMES_MODEL` | hermes | ❌ | Model to use (default: `anthropic:claude-sonnet-4-6`) |
| `HERMES_PORT` | hermes | ❌ | Hermes port (default: `8080`) |
| `REGISTRY` | all | ❌ | Docker registry prefix (default: `ghcr.io/nikushasta2026`) |
| `TAG` | all | ❌ | Docker image tag (default: `latest`) |

See [.env.example](.env.example) for the full list with descriptions.

---

## Docker Registry

Images are hosted on **GitHub Container Registry (ghcr.io)**:
- `ghcr.io/nikushasta2026/paperclip:latest`
- `ghcr.io/nikushasta2026/nanoclaw-ceo:latest`
- `ghcr.io/nikushasta2026/hermes:latest`

See [docs/registry.md](docs/registry.md) for registry options and the recommendation rationale.

---

## Resource Usage (Stilgar 61GB RAM)

| Service | CPU | Memory |
|---|---|---|
| paperclip | 4 cores max | 8 GB max / 2 GB reserved |
| nanoclaw-ceo | 2 cores max | 4 GB max / 1 GB reserved |
| hermes | 2 cores max | 4 GB max / 512 MB reserved |
| **Stack total** | **8 cores** | **16 GB max / 3.5 GB reserved** |

Stilgar has ample headroom (45 GB+ free) for additional services.

---

## Repo Structure

```
stilgar-infra/
├── README.md                   # This file
├── docker-compose.yml          # Full stack compose
├── .env.example                # Environment variable template
├── .gitignore
├── paperclip/
│   ├── Dockerfile              # Paperclip container (multi-stage Node.js)
│   └── README.md
├── nanoclaw-ceo/
│   ├── Dockerfile              # NanoClaw CEO container
│   └── README.md
├── hermes/
│   ├── Dockerfile              # Hermes Agent container (Python 3.12)
│   └── README.md
└── docs/
    ├── deployment.md           # Step-by-step Coolify deployment guide
    └── registry.md             # Docker registry options and recommendation
```

---

## License

MIT — see upstream projects for their respective licenses.
