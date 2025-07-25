# =========================
# Stage 1: Base Build Image
# =========================
FROM node:18-alpine AS base

WORKDIR /app

# Install OS deps & pnpm
RUN apk add --no-cache bash curl python3 make g++ && \
    corepack enable && corepack prepare pnpm@8.6.0 --activate

COPY . .

# Install dependencies with cache support
RUN pnpm install --frozen-lockfile

# ============================
# Stage 2: Build Editor UI
# ============================
FROM base AS build-ui

RUN pnpm run build:ui

# ==============================
# Stage 3: Build CLI (core app)
# ==============================
FROM base AS build-cli

RUN pnpm run build

# ================================
# Stage 4: Runtime Slim Container
# ================================
FROM node:18-alpine AS final

WORKDIR /app

# Install only runtime dependencies
RUN apk add --no-cache dumb-init

COPY --from=build-cli /app/packages/cli /app/packages/cli
COPY --from=build-ui /app/packages/editor-ui /app/packages/editor-ui
COPY --from=base /app/package.json /app/package.json

# Only copy required node_modules
COPY --from=base /app/node_modules /app/node_modules

ENV NODE_ENV=production
EXPOSE 5678

ENTRYPOINT ["dumb-init", "pnpm", "start"]