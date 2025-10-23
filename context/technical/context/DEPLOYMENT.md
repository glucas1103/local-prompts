# Build & Deployment

## Metadata

- **Category**: Guideline
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/turbo.json`, `/docker-compose.yml`, `apps/*/Dockerfile`, `apps/*/package.json`

## Technical Overview

Build and deployment leverages **Turborepo 2.4.4** for intelligent build orchestration with caching, combined with **Docker multi-stage builds** for optimized production images. The monorepo structure enables parallel builds of independent workspaces while respecting dependency graphs.

Deployment follows a **local → staging → production** flow, with local development using Docker Compose for service orchestration, and production deployments using standalone Docker containers. Database migrations are applied before code deployments to ensure schema compatibility, with rollback procedures available for failed migrations.

The build system optimizes for both development experience (fast rebuilds, hot reload) and production performance (minified bundles, tree-shaking, standalone outputs). All builds are reproducible with locked dependencies (package-lock.json) and versioned through commit SHA tracking for Sentry release monitoring.

## Architecture & Design

### Build Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Turborepo Build Flow                      │
└──────────────────────────────────────────────────────────────┘

npm run build (root)
     ↓
Turborepo Orchestrator
     ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: Build Dependencies                                 │
│  └─ packages/types → tsup → dist/index.cjs                  │
└─────────────────────────────────────────────────────────────┘
     ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: Build Applications (Parallel)                      │
│  ├─ apps/api → tsup → dist/index.js                         │
│  └─ apps/web → next build → .next/ (standalone)             │
└─────────────────────────────────────────────────────────────┘
     ↓
Build Artifacts Ready for Deployment
```

### Deployment Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    Local    │ →  │   Staging   │ →  │ Production  │
│ Development │    │   Testing   │    │   Release   │
└─────────────┘    └─────────────┘    └─────────────┘
      ↓                   ↓                   ↓
 docker-compose    Container Deploy    Container Deploy
 Hot Reload        Real Services       Optimized Images
 Mock Services     Real Database       Real Database
                   Integration Tests   Monitoring
```

## Configuration & Setup

### Turborepo Configuration

**File**: `/turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**", "public/dist/**"],
      "dependsOn": ["^build"], // Build dependencies first
      "env": [
        "NEXT_PUBLIC_CLIENT_API_HOST",
        "NEXT_PUBLIC_SENTRY_WEB_DSN",
        "SENTRY_AUTH_TOKEN",
        "NEXT_PUBLIC_COMMIT_SHA"
      ]
    },
    "test": {
      "outputs": ["coverage/**"],
      "dependsOn": [] // Independent
    },
    "lint": {
      "dependsOn": ["^build"] // Lint after build
    },
    "dev": {
      "cache": false, // Don't cache dev mode
      "persistent": true // Keep running
    },
    "typecheck": {
      "dependsOn": ["^typecheck"]
    }
  }
}
```

**Key Features**:

- **Dependency Graph**: Builds packages before apps
- **Caching**: Skips unchanged workspaces
- **Parallel Execution**: Builds independent workspaces simultaneously
- **Environment Awareness**: Passes required env vars to builds

### Build Scripts

**Root** (`package.json`):

```json
{
  "scripts": {
    "build": "turbo run build",
    "build:packages": "turbo run build --filter='./packages/*'",
    "dev": "turbo run dev",
    "clean": "turbo run clean",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "typecheck": "turbo run typecheck"
  }
}
```

**API** (`apps/api/package.json`):

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsup ./src/index.ts",
    "start": "NODE_ENV=production node dist/index.js",
    "db:migrate": "drizzle-kit migrate",
    "db:init": "npm run db:migrate && npm run db:seed",
    "sentry:sourcemaps": "npm run generate-sourcemaps && sentry-cli sourcemaps upload ..."
  }
}
```

**Web** (`apps/web/package.json`):

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start"
  }
}
```

## Build Pipeline

### Local Development Build

```bash
# Install dependencies
npm install

# Build packages first (if needed)
npm run build:packages

# Start all in development mode
npm run dev

# Or start specific app
cd apps/api && npm run dev
cd apps/web && npm run dev
```

**Development Features**:

- **Hot Reload**: tsx watch (API), turbopack (Web)
- **Fast Refresh**: React fast refresh
- **Source Maps**: Enabled for debugging
- **Pretty Logs**: pino-pretty for readable logs

### Production Build

```bash
# Build all workspaces
npm run build

# Outputs:
# - apps/api/dist/ (CommonJS bundle)
# - apps/web/.next/ (Next.js standalone)
# - packages/types/dist/ (CJS module)
```

**Build Optimizations**:

- **Tree Shaking**: Remove unused code
- **Minification**: Compress JavaScript
- **Source Maps**: Generated for Sentry
- **Standalone**: Next.js self-contained output

### Build Caching

Turborepo caches build outputs:

```bash
# Check cache hit/miss
turbo run build --summarize

# Force rebuild (skip cache)
turbo run build --force

# Clear cache
rm -rf .turbo
```

**Cache Keys**:

- File contents (hashed)
- Environment variables
- Dependencies
- Task configuration

## Docker Configuration

### Multi-Stage Build (API)

**File**: `apps/api/Dockerfile`

```dockerfile
FROM node:22.14.0-alpine AS base

FROM base AS builder
WORKDIR /app
RUN npm install -g turbo
COPY . .
RUN turbo prune api --docker

FROM base AS installer
WORKDIR /app
COPY --from=builder /app/out/json/ .
RUN npm install
COPY --from=builder /app/out/full/ .
RUN npm run build

FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 expressjs
RUN adduser --system --uid 1001 expressjs
USER expressjs
COPY --from=installer /app .
CMD ["node", "apps/api/dist/index.js"]
```

### Multi-Stage Build (Web)

**File**: `apps/web/Dockerfile`

```dockerfile
FROM node:22.14.0-alpine AS base

FROM base AS builder
WORKDIR /app
RUN npm install -g turbo
COPY . .
RUN turbo prune web --docker

FROM base AS installer
WORKDIR /app
COPY --from=builder /app/out/json/ .
RUN npm install
COPY --from=builder /app/out/full/ .

# Build arguments for Next.js
ARG NEXT_PUBLIC_CLIENT_API_HOST
ARG NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
ENV NEXT_PUBLIC_CLIENT_API_HOST=$NEXT_PUBLIC_CLIENT_API_HOST
ENV NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY

RUN npm run build

FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

CMD ["node", "apps/web/server.js"]
```

## Docker Compose

**File**: `docker-compose.yml`

```yaml
services:
  web:
    build:
      context: .
      dockerfile: apps/web/Dockerfile
    ports:
      - 3000:3000
    environment:
      - NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY}

  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile
    ports:
      - 3001:3001
    environment:
      - NODE_ENV=development
      - POSTGRES_HOST=db
    depends_on:
      - db

  db:
    image: postgres:17-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=db
```

## Deployment Flow

**Local Development**:

```bash
npm install
npm run dev
```

**Build for Production**:

```bash
npm run build
```

**Docker Deployment**:

```bash
docker-compose up -d
```

**Database Migrations**:

```bash
cd apps/api
npm run db:migrate
npm run db:seed
```

**[To be documented by CTO]**: CI/CD pipeline, deployment approval process, rollback procedures

## References

- **Configuration**: `/turbo.json`, `/docker-compose.yml`, `apps/*/Dockerfile`
- **Scripts**: `apps/*/package.json`
- **External Docs**: [Turborepo](https://turbo.build/), [Docker](https://docs.docker.com/)
