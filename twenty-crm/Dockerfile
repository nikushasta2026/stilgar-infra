# =============================================================================
# Twenty CRM — Stilgar Deployment
# =============================================================================
# Official unified image — serves both frontend and backend from port 3000.
# Docs: https://twenty.com/developers/self-hosting/self-hosting-guide/docker-compose
# =============================================================================

FROM twentycrm/twenty:latest

# Expose the app port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=5 \
  CMD wget -qO- http://localhost:3000/api/health || curl -f http://localhost:3000/api/health || exit 1
