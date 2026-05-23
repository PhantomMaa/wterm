# syntax=docker/dockerfile:1

# ───────────────────────────────────────────────
# Builder stage
# ───────────────────────────────────────────────
FROM node:22 AS builder
WORKDIR /app

# Enable pnpm via corepack
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable && corepack prepare pnpm@11.1.3 --activate

# Copy monorepo root files
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY patches ./patches
COPY scripts ./scripts

# Copy workspace packages
COPY packages ./packages

# Copy the docker example
COPY examples/docker ./examples/docker

# Install dependencies (use cache mount for pnpm store)
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

# Rebuild node-pty native module for the Linux container
RUN cd /app/node_modules/.pnpm/node-pty@*/node_modules/node-pty && npx node-gyp rebuild

# Build the docker example and its workspace dependencies
RUN pnpm --filter docker build

# Compile the custom server
WORKDIR /app/examples/docker
RUN pnpm exec tsc -p tsconfig.server.json

# ───────────────────────────────────────────────
# Runner stage
# ───────────────────────────────────────────────
FROM node:22-slim AS runner
WORKDIR /app/examples/docker

# Install runtime tools: git, python, uv, bash
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    python3 \
    python3-pip \
    python3-venv \
    curl \
    ca-certificates \
    bash \
    && rm -rf /var/lib/apt/lists/*

# Install uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:${PATH}"

# Create the workspace directory that will be bind-mounted
RUN mkdir -p /workspace

# Copy app build artifacts
COPY --from=builder /app/examples/docker/.next ./.next
COPY --from=builder /app/examples/docker/dist ./dist
COPY --from=builder /app/examples/docker/public ./public
COPY --from=builder /app/examples/docker/package.json ./

# Copy node_modules — both root .pnpm store and local symlinks are required
COPY --from=builder /app/node_modules /app/node_modules
COPY --from=builder /app/examples/docker/node_modules ./node_modules

ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3000
ENV WORKSPACE_DIR=/workspace
ENV SHELL=/bin/bash

EXPOSE 3000
CMD ["node", "dist/server.js"]
