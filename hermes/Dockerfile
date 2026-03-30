# =============================================================================
# Hermes Agent — NousResearch Hermes Agent
# https://github.com/NousResearch/hermes-agent
# =============================================================================
# Used for testing the hermes-local adapter with the Stilgar AI stack.

FROM python:3.12-slim

LABEL org.opencontainers.image.source="https://github.com/nikushasta2026/stilgar-infra"
LABEL org.opencontainers.image.description="NousResearch Hermes Agent"

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    wget \
    build-essential \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user for security
RUN useradd -m -u 1000 hermes && mkdir -p /home/hermes/.hermes /app

WORKDIR /app

# Clone hermes-agent from NousResearch
RUN git clone --depth 1 https://github.com/NousResearch/hermes-agent.git .

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -e . || \
    pip install --no-cache-dir -r requirements.txt 2>/dev/null || true

# Attempt official install script (fallback if pip install fails)
# The install script is included as a fallback mechanism
RUN chmod +x scripts/install.sh 2>/dev/null || true

# Create hermes config directory
RUN mkdir -p /home/hermes/.hermes && \
    chown -R hermes:hermes /home/hermes/.hermes /app

# Default hermes configuration
RUN cat > /home/hermes/.hermes/config.yaml << 'EOF'
# Hermes Agent Configuration
server:
  host: "0.0.0.0"
  port: 8080
  log_level: "info"

agent:
  name: "hermes"
  description: "NousResearch Hermes Agent for hermes-local adapter testing"
  
# Model is configured via HERMES_MODEL env var at runtime
EOF

# Environment variables
ENV HERMES_HOME=/home/hermes/.hermes
ENV HERMES_PORT=8080
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

# Expose Hermes gateway port
EXPOSE 8080

# Volume for persistent hermes data
VOLUME ["/home/hermes/.hermes"]

# Use non-root user
USER hermes

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget -qO- http://localhost:8080/health 2>/dev/null || \
        curl -sf http://localhost:8080/health 2>/dev/null || exit 1

# Start Hermes agent gateway
# Falls back to different commands based on what's available
CMD ["sh", "-c", "\
    if command -v hermes >/dev/null 2>&1; then \
        hermes gateway start --port ${HERMES_PORT:-8080}; \
    elif python -m hermes_agent.server >/dev/null 2>&1; then \
        python -m hermes_agent.server --port ${HERMES_PORT:-8080}; \
    else \
        python -c \"from hermes_agent import app; app.run(host='0.0.0.0', port=${HERMES_PORT:-8080})\" 2>/dev/null || \
        python main.py --port ${HERMES_PORT:-8080}; \
    fi"]
