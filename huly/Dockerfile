# Huly CRM — Open-source business operations platform
# Based on official huly/huly image

FROM huly/huly:latest

LABEL maintainer="Stilgar Infrastructure"
LABEL description="Huly CRM for developer outreach and kanban management"

# Environment defaults (override via docker-compose or .env)
ENV NODE_ENV=production
ENV PORT=3300
ENV MONGO_URL=mongodb://huly-mongo:27017
ENV ELASTIC_URL=http://huly-elastic:9200
ENV MINIO_ACCESS_KEY=minioadmin
ENV MINIO_SECRET_KEY=minioadmin
ENV MINIO_ENDPOINT=http://huly-minio:9000

# Ensure the application can bind to the container's network interface
ENV HOST=0.0.0.0

EXPOSE 3300

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=5 \
    CMD wget -qO- http://localhost:3300 || exit 1
