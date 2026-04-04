# Backend Development Dockerfile

# UV source image
FROM ghcr.nju.edu.cn/astral-sh/uv:0.7.20 AS uv-source

# Node.js source image (copy binary instead of using nodesource apt repo)
FROM docker.1ms.run/node:22-slim AS node-source

FROM docker.1ms.run/python:3.12-slim-bookworm

# Use China apt mirror
RUN sed -i "s|deb.debian.org|mirrors.aliyun.com|g" /etc/apt/sources.list.d/debian.sources 2>/dev/null || true; \
    sed -i "s|deb.debian.org|mirrors.aliyun.com|g" /etc/apt/sources.list 2>/dev/null || true

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    build-essential \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js (provides npx for MCP servers) from node-source stage
COPY --from=node-source /usr/local/bin/node /usr/local/bin/node
COPY --from=node-source /usr/local/include/node /usr/local/include/node
COPY --from=node-source /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

# Install Docker CLI (for DooD: allows starting sandbox containers via host Docker socket)
COPY --from=docker.1ms.run/docker:cli /usr/local/bin/docker /usr/local/bin/docker

# Install uv (source image overridable via UV_IMAGE build arg)
COPY --from=uv-source /uv /uvx /usr/local/bin/

# Set working directory
WORKDIR /app

# Copy frontend source code
COPY backend ./backend

# Install dependencies with cache mount
RUN --mount=type=cache,target=/root/.cache/uv \
    sh -c "cd backend && UV_INDEX_URL=https://mirrors.aliyun.com/pypi/simple uv sync"

# Expose ports (gateway: 8001, langgraph: 2024)
EXPOSE 8001 2024

# Default command (can be overridden in docker-compose)
CMD ["sh", "-c", "cd backend && PYTHONPATH=. uv run uvicorn app.gateway.app:app --host 0.0.0.0 --port 8001"]
