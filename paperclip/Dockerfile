# =============================================================================
# Paperclip — AI Company OS
# https://github.com/paperclipai/paperclip
# =============================================================================

# -----------------------------------------------------------------------------
# Stage 1: Builder
# -----------------------------------------------------------------------------
FROM node:22-alpine AS builder

# Install build dependencies
RUN apk add --no-cache git python3 make g++ libc6-compat

# Install pnpm
RUN npm install -g pnpm@latest

WORKDIR /build

# Clone Paperclip source
RUN git clone --depth 1 https://github.com/paperclipai/paperclip.git .

# Install dependencies
RUN pnpm install --frozen-lockfile || pnpm install

# Set build-time env vars (non-secret defaults for build)
ENV NODE_ENV=production
ENV SKIP_ENV_VALIDATION=1

# Build the application
RUN pnpm run build || pnpm build

# -----------------------------------------------------------------------------
# Stage 2: Runtime
# -----------------------------------------------------------------------------
FROM node:22-alpine AS runtime

LABEL org.opencontainers.image.source="https://github.com/nikushasta2026/stilgar-infra"
LABEL org.opencontainers.image.description="Paperclip AI Company OS"
LABEL org.opencontainers.image.licenses="MIT"

# Install runtime dependencies only
RUN apk add --no-cache wget curl

# Install pnpm for running the app
RUN npm install -g pnpm@latest

WORKDIR /app

# Copy built artifacts from builder
COPY --from=builder /build/.next /app/.next
COPY --from=builder /build/public /app/public
COPY --from=builder /build/package.json /app/package.json
COPY --from=builder /build/pnpm-lock.yaml /app/pnpm-lock.yaml
COPY --from=builder /build/node_modules /app/node_modules
COPY --from=builder /build/next.config.* /app/ 2>/dev/null || true
COPY --from=builder /build/prisma /app/prisma 2>/dev/null || true

# Create persistent data directory
RUN mkdir -p /data/paperclip && \
    chown -R node:node /data/paperclip /app

# Environment variables (overridable at runtime)
ENV NODE_ENV=production
ENV PORT=3100
ENV DATABASE_URL=file:/data/paperclip/db.sqlite
ENV NEXT_PUBLIC_APP_URL=http://localhost:3100
ENV PAPERCLIP_DATA_DIR=/data/paperclip

# Expose port
EXPOSE 3100

# Use non-root user
USER node

# Volume for persistent data
VOLUME ["/data/paperclip"]

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget -qO- http://localhost:3100/api/health || exit 1

# Start Paperclip
CMD ["sh", "-c", "npx prisma migrate deploy 2>/dev/null || true && pnpm start"]
