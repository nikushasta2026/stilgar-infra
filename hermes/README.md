# Hermes Agent — NousResearch

A containerized deployment of the [NousResearch Hermes Agent](https://github.com/NousResearch/hermes-agent) for testing the `hermes-local` adapter against the Stilgar AI stack.

- **Port:** `8080` (Hermes gateway)
- **Data volume:** `/home/hermes/.hermes`

## Purpose

The Hermes container serves as:
1. **Adapter testing** — validate the `hermes-local` adapter works end-to-end
2. **Model routing** — test different model backends (Anthropic, OpenRouter)
3. **Agent protocol testing** — verify agent-to-agent communication via Paperclip

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | ✅* | — | Anthropic API key (*or use OPENROUTER_API_KEY) |
| `OPENROUTER_API_KEY` | ✅* | — | OpenRouter key (alternative to Anthropic) |
| `HERMES_MODEL` | ❌ | `anthropic:claude-sonnet-4-6` | Model in `provider:model` format |
| `HERMES_PORT` | ❌ | `8080` | Port for Hermes gateway |

## Model Format

The `HERMES_MODEL` env var uses `provider:model-name` format:

```
anthropic:claude-sonnet-4-6          # Claude via Anthropic
anthropic:claude-opus-4-6            # Claude Opus
openrouter:anthropic/claude-3.7-sonnet  # Claude via OpenRouter
openrouter:nousresearch/hermes-3-llama-3.1-70b  # Hermes 3 via OpenRouter
```

## API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/health` | GET | Health check |
| `/v1/chat/completions` | POST | OpenAI-compatible chat |
| `/v1/agents/run` | POST | Run an agent task |

## Build

```bash
docker build -t hermes ./hermes
```

## Test

```bash
curl http://localhost:8080/health
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "hermes", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Notes

- No Slack integration needed — this is purely for testing the hermes-local adapter
- Connects to `paperclip-net` network so it can reach Paperclip at `http://paperclip:3100`
