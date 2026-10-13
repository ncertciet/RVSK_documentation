```dockerfile
# =============================================================================
# RVSK Accreditation Platform - Multi-stage Dockerfile
# =============================================================================
# Stage 1: Development
# Stage 2: Build
# Stage 3: Production
# =============================================================================

# ---------------------------------------------------------------------------
# Stage 1: Development
# ---------------------------------------------------------------------------
FROM node:22-alpine AS development

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm ci

# Copy application source
COPY . .

# Expose API port
EXPOSE 3000

# Development command
CMD ["npm", "run", "start:dev"]


# ---------------------------------------------------------------------------
# Stage 2: Build
# ---------------------------------------------------------------------------
FROM node:22-alpine AS build

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm ci

# Copy application source and compile TypeScript
COPY . .
RUN npm run build

# Install production dependencies only
RUN npm ci --omit=dev && npm cache clean --force

# Create logs directory for the production image
RUN mkdir -p /app/logs


# ---------------------------------------------------------------------------
# Stage 3: Production
# ---------------------------------------------------------------------------
FROM node:22-alpine AS production

# Install wget for the health check
RUN apk add --no-cache wget

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app

# Create logs directory and set permissions
RUN mkdir -p /app/logs && \
    chown -R appuser:appgroup /app

# Copy production artifacts from the build stage
COPY --from=build --chown=appuser:appgroup /app/dist ./dist
COPY --from=build --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=build --chown=appuser:appgroup /app/package.json ./package.json
COPY --from=build --chown=appuser:appgroup /app/logs ./logs

# Run as non-root user
USER appuser

# Expose API port
EXPOSE 3000

# Container health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD wget --spider --quiet http://localhost:3000/accreditation/api/v1/health || exit 1

# Start production API
CMD ["node", "dist/main.js"]
```
