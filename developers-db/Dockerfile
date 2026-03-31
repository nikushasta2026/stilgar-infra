FROM postgres:16-alpine

# Copy init scripts — postgres runs these alphabetically on first boot
COPY initdb/ /docker-entrypoint-initdb.d/

EXPOSE 5432

HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
  CMD pg_isready -U ${POSTGRES_USER:-developers} -d ${POSTGRES_DB:-developers}
