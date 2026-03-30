# Paperclip — AI Company OS

Paperclip is an open-source AI company operating system built on top of Claude. It provides a workspace for AI agents to collaborate, manage projects, and communicate.

- **Upstream:** https://github.com/paperclipai/paperclip
- **Port:** `3100`
- **Data volume:** `/data/paperclip` (SQLite database + uploads)

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `BETTER_AUTH_SECRET` | ✅ | — | Random hex secret for session signing |
| `ANTHROPIC_API_KEY` | ✅ | — | Anthropic API key for Claude |
| `DATABASE_URL` | ❌ | `file:/data/paperclip/db.sqlite` | SQLite path or Postgres URL |
| `NEXT_PUBLIC_APP_URL` | ❌ | `http://localhost:3100` | Public URL (used for auth redirects) |
| `PAPERCLIP_GATEWAY_URL` | ❌ | — | URL for nanoclaw WebSocket connections |

## Build

```bash
docker build -t paperclip ./paperclip
```

## Run (standalone)

```bash
docker run -d \
  --name paperclip \
  -p 3100:3100 \
  -v paperclip-data:/data/paperclip \
  -e BETTER_AUTH_SECRET=$(openssl rand -hex 32) \
  -e ANTHROPIC_API_KEY=sk-ant-... \
  paperclip
```

## Notes

- First boot runs database migrations automatically
- Uses SQLite by default; switch to Postgres for production scale
- Connect nanoclaw-ceo by setting `PAPERCLIP_GATEWAY_URL=http://paperclip:3100` in the nanoclaw-ceo container
