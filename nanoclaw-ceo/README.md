# NanoClaw CEO — OpenClaw Nanoclaw Node

An OpenClaw nanoclaw node running the CEO persona. This agent acts as the strategic lead of the AI company running on Stilgar, delegating to specialized sub-agents via Paperclip and reporting to the board (Marcus) via Slack.

- **Port:** `3000` (OpenClaw gateway)
- **Workspace volume:** `/agent/workspace`

## CEO Persona

| Attribute | Value |
|---|---|
| **Name** | CEO (Aria) |
| **Role** | Chief Executive Officer |
| **Reports to** | Board (Marcus Safar) via Slack |
| **Delegates to** | CTO, CMO, CFO, CPO (via Paperclip) |
| **Never does** | Writes code, runs infrastructure commands |
| **Focus** | Strategy, P&L, board alignment |

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `OPENCLAW_AGENT_TOKEN` | ✅ | — | OpenClaw agent auth token |
| `ANTHROPIC_API_KEY` | ✅ | — | Anthropic API key for Claude |
| `PAPERCLIP_GATEWAY_URL` | ✅ | `http://paperclip:3100` | Paperclip instance URL |
| `SLACK_BOT_TOKEN` | ✅ | — | Slack bot token for board updates |
| `SLACK_BOARD_CHANNEL` | ❌ | — | Slack channel ID for board posts |
| `OPENCLAW_PORT` | ❌ | `3000` | OpenClaw gateway port |

## Persona Files

The Dockerfile bakes in these persona files at `/agent/workspace/`:

| File | Purpose |
|---|---|
| `SOUL.md` | CEO identity, principles, decision framework |
| `AGENTS.md` | Workspace rules and delegation protocol |
| `HEARTBEAT.md` | Active monitoring checklist |

These are **baked into the image** as defaults. Mount a volume at `/agent/workspace` to override them or persist the CEO's evolving memory.

## Connecting to Paperclip

1. Paperclip must be running and healthy first
2. Set `PAPERCLIP_GATEWAY_URL=http://paperclip:3100` (docker-compose handles this)
3. The CEO will connect to Paperclip on startup

## Build

```bash
docker build -t nanoclaw-ceo ./nanoclaw-ceo
```
