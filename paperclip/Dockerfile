# =============================================================================
# Paperclip — AI Company OS
# https://github.com/paperclipai/paperclip
#
# Builds Paperclip from source by cloning the official repo.
# Based on the official Paperclip Dockerfile structure.
# =============================================================================

FROM node:lts-trixie-slim AS base
RUN apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates curl git \
  && rm -rf /var/lib/apt/lists/*
RUN corepack enable

# Clone Paperclip source at build time
FROM base AS source
WORKDIR /app
RUN git clone --depth 1 https://github.com/paperclipai/paperclip.git .

# Install dependencies
FROM base AS deps
WORKDIR /app
COPY --from=source /app/package.json /app/pnpm-workspace.yaml /app/pnpm-lock.yaml /app/.npmrc ./
COPY --from=source /app/cli/package.json cli/
COPY --from=source /app/server/package.json server/
COPY --from=source /app/ui/package.json ui/
COPY --from=source /app/packages/shared/package.json packages/shared/
COPY --from=source /app/packages/db/package.json packages/db/
COPY --from=source /app/packages/adapter-utils/package.json packages/adapter-utils/
COPY --from=source /app/packages/adapters/claude-local/package.json packages/adapters/claude-local/
COPY --from=source /app/packages/adapters/codex-local/package.json packages/adapters/codex-local/
COPY --from=source /app/packages/adapters/cursor-local/package.json packages/adapters/cursor-local/
COPY --from=source /app/packages/adapters/gemini-local/package.json packages/adapters/gemini-local/
COPY --from=source /app/packages/adapters/openclaw-gateway/package.json packages/adapters/openclaw-gateway/
COPY --from=source /app/packages/adapters/opencode-local/package.json packages/adapters/opencode-local/
COPY --from=source /app/packages/adapters/pi-local/package.json packages/adapters/pi-local/
COPY --from=source /app/packages/plugins/sdk/package.json packages/plugins/sdk/
COPY --from=source /app/patches/ patches/
RUN pnpm install --frozen-lockfile

# Build
FROM base AS build
WORKDIR /app
COPY --from=deps /app /app
COPY --from=source /app /app
RUN pnpm --filter @paperclipai/ui build
RUN pnpm --filter @paperclipai/plugin-sdk build
RUN pnpm --filter @paperclipai/server build
RUN test -f server/dist/index.js || (echo "ERROR: server build output missing" && exit 1)

# Production image
FROM base AS production
WORKDIR /app

LABEL org.opencontainers.image.source="https://github.com/nikushasta2026/stilgar-infra"
LABEL org.opencontainers.image.description="Paperclip AI Company OS"

COPY --chown=node:node --from=build /app /app
RUN npm install --global --omit=dev @anthropic-ai/claude-code@latest @openai/codex@latest opencode-ai \
  && mkdir -p /paperclip \
  && chown node:node /paperclip

ENV NODE_ENV=production \
  HOME=/paperclip \
  HOST=0.0.0.0 \
  PORT=3100 \
  SERVE_UI=true \
  PAPERCLIP_HOME=/paperclip \
  PAPERCLIP_INSTANCE_ID=default \
  PAPERCLIP_CONFIG=/paperclip/instances/default/config.json \
  PAPERCLIP_DEPLOYMENT_MODE=authenticated \
  PAPERCLIP_DEPLOYMENT_EXPOSURE=private

VOLUME ["/paperclip"]
EXPOSE 3100

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=90s --retries=3 \
    CMD curl -f http://localhost:3100/api/health || exit 1

USER node
CMD ["node", "--import", "./server/node_modules/tsx/dist/loader.mjs", "server/dist/index.js"]
