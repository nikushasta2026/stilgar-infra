# =============================================================================
# Paperclip — AI Company OS
# https://github.com/paperclipai/paperclip
#
# Builds Paperclip from source by cloning the official repo.
# Mirrors the official Dockerfile structure exactly.
# =============================================================================

FROM node:lts-trixie-slim AS base
RUN apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates curl git \
  && rm -rf /var/lib/apt/lists/*
RUN corepack enable && corepack prepare pnpm@latest --activate

# Clone Paperclip source at build time into a temp location
FROM base AS source
WORKDIR /src
RUN git clone --depth 1 https://github.com/paperclipai/paperclip.git .

# Install dependencies (using cloned source)
FROM base AS deps
WORKDIR /app
COPY --from=source /src/package.json /src/pnpm-workspace.yaml /src/pnpm-lock.yaml /src/.npmrc ./
COPY --from=source /src/cli/package.json cli/
COPY --from=source /src/server/package.json server/
COPY --from=source /src/ui/package.json ui/
COPY --from=source /src/packages/shared/package.json packages/shared/
COPY --from=source /src/packages/db/package.json packages/db/
COPY --from=source /src/packages/adapter-utils/package.json packages/adapter-utils/
COPY --from=source /src/packages/adapters/claude-local/package.json packages/adapters/claude-local/
COPY --from=source /src/packages/adapters/codex-local/package.json packages/adapters/codex-local/
COPY --from=source /src/packages/adapters/cursor-local/package.json packages/adapters/cursor-local/
COPY --from=source /src/packages/adapters/gemini-local/package.json packages/adapters/gemini-local/
COPY --from=source /src/packages/adapters/openclaw-gateway/package.json packages/adapters/openclaw-gateway/
COPY --from=source /src/packages/adapters/opencode-local/package.json packages/adapters/opencode-local/
COPY --from=source /src/packages/adapters/pi-local/package.json packages/adapters/pi-local/
COPY --from=source /src/packages/plugins/sdk/package.json packages/plugins/sdk/
COPY --from=source /src/patches/ patches/
RUN pnpm install --frozen-lockfile

# Build
FROM base AS build
WORKDIR /app
COPY --from=deps /app /app
COPY --from=source /src .
RUN pnpm --filter @paperclipai/ui build
RUN pnpm --filter @paperclipai/plugin-sdk build
RUN pnpm --filter @paperclipai/server build
RUN test -f server/dist/index.js || (echo "ERROR: server build output missing" && exit 1)

# Production image — copy entire build tree (mirrors official Dockerfile)
FROM base AS production
WORKDIR /app

LABEL org.opencontainers.image.source="https://github.com/nikushasta2026/stilgar-infra"
LABEL org.opencontainers.image.description="Paperclip AI Company OS"
LABEL org.opencontainers.image.licenses="MIT"

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
  PAPERCLIP_DEPLOYMENT_EXPOSURE=private \
  PAPERCLIP_ALLOWED_HOSTNAMES=77.42.7.12,100.94.62.93,localhost

VOLUME ["/paperclip"]
EXPOSE 3100

HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=5 \
    CMD curl -f http://localhost:3100/api/health || wget -qO- http://localhost:3100/api/health || exit 1

USER node
CMD ["node", "--import", "./server/node_modules/tsx/dist/loader.mjs", "server/dist/index.js"]
