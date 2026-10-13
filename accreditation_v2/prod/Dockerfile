# =============================================================================
# RVSK Accreditation Platform - Multi-stage Dockerfile
# =============================================================================
# Stage 1: Development (full dev dependencies, hot-reload)
# Stage 2: Build (compile TypeScript)
# Stage 3: Production (minimal image, non-root user)
# =============================================================================

# ---------------------------------------------------------------------------
# Stage 1: Development
# ---------------------------------------------------------------------------
FROM node:22-alpine AS development

WORKDIR /app

# Install dependencies first for better layer caching
COPY package.json package-lock.json ./
RUN npm ci

# Copy source code
COPY . .

# Expose API port
EXPOSE 3000

# Default command for development (hot-reload)
CMD ["npm", "run", "start:dev"]

# ---------------------------------------------------------------------------
# Stage 2: Build
# ---------------------------------------------------------------------------
FROM node:22-alpine AS build

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Prune dev dependencies after build
RUN npm ci --omit=dev && npm cache clean --force

# ---------------------------------------------------------------------------
# Stage 3: Production
# ---------------------------------------------------------------------------
FROM node:22-alpine AS production

# Security: add non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app

RUN mkdir -p /app/logs && \
    chown -R appuser:appgroup /app

# Copy only production artifacts
COPY --from=build --chown=appuser:appgroup /app/dist ./dist
COPY --from=build --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=build --chown=appuser:appgroup /app/package.json ./package.json
COPY --from=build --chown=appuser:appgroup /app/logs ./logs

# Security: run as non-root user
USER appuser

# Expose API port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
CMD wget --spider --quiet http://localhost:3000/accreditation/api/v1/health || exit 1

# Default command for production API
CMD ["node", "dist/main.js"]
