# Technical Context - Initial Summary

## Tech Stack Overview

### Languages & Runtimes

- **TypeScript 5.7.3**: Primary language for all applications
- **Node.js 22.14.0**: Backend runtime (LTS version)
- **React 19.0.0**: Frontend UI library
- **npm 10.9.2**: Package manager

### Frontend Stack

- **Next.js 15.3.5**: React framework with App Router
- **TanStack Query 5.71.1**: Server state management
- **Tailwind CSS 4.1.11**: Utility-first CSS framework
- **Radix UI**: Headless UI components
- **Shadcn/UI**: Pre-built component library
- **Lucide React 0.525.0**: Icon library
- **next-intl 4.3.4**: Internationalization
- **next-themes 0.4.6**: Dark mode support
- **Storybook 8.6.14**: Component development environment

### Backend Stack

- **Express 4.21.2**: Web server framework
- **Drizzle ORM 0.41.0**: Type-safe ORM
- **PostgreSQL 17**: Primary database (Alpine image)
- **Zod 3.25.67**: Schema validation
- **Pino 9.7.0 & pino-http 10.5.0**: Structured logging
- **Helmet 8.1.0**: Security headers
- **CORS 2.8.5**: Cross-origin resource sharing

### External Services & SDKs

- **Clerk (@clerk/nextjs 6.24.0, @clerk/express 1.7.5)**: Authentication & user management
- **Sentry (@sentry/nextjs 9.22.0, @sentry/node 9.22.0)**: Error tracking & performance monitoring
- **PostHog (posthog-js 1.258.2)**: Product analytics & feature flags
- **Octokit (@octokit/app 16.1.1)**: GitHub API integration
- **Svix 1.68.0**: Webhook management

## Architecture Patterns

### Monorepo Structure

- **Turborepo 2.4.4**: Build orchestration and caching
- **Workspaces**: npm workspaces for dependency management
- **Architecture**: Three main workspaces:
  - `apps/api`: Backend Express API
  - `apps/web`: Frontend Next.js application
  - `packages/types`: Shared TypeScript types

### Folder Organization

#### Backend (`apps/api/src/`)

```
src/
├── routes/          # API endpoints (admin, clerk, github, organization, users)
├── middlewares/     # Express middleware (auth, logger, transaction, validation)
├── services/        # Business logic (github)
├── db/              # Database layer
│   ├── schema.ts    # Drizzle schema definitions
│   ├── credentials.ts
│   └── seed/        # Database seed data
├── utils/           # Utility functions
├── types/           # Type definitions
├── tests/           # Test files and setup
├── server.ts        # Express app configuration
└── instrument.ts    # Sentry initialization
```

**Architectural Layers**:

- **Routes**: HTTP endpoints definition
- **Middlewares**: Request/response processing pipeline
- **Services**: Business logic and external API interactions
- **Database**: Data access layer with Drizzle ORM
- **Utils**: Shared utilities (logger, helpers)

#### Frontend (`apps/web/src/`)

```
src/
├── app/                  # Next.js App Router
│   ├── (protected)/      # Authenticated routes
│   │   └── dashboard/    # Main dashboard area
│   ├── (public)/         # Public routes
│   │   └── (auth)/       # Authentication pages
│   └── api/              # API routes
├── components/           # React components (61 files)
│   └── ui/               # Shadcn UI components
├── lib/                  # Utilities and helpers (19 files)
│   └── httpClient/       # API client
├── hooks/                # Custom React hooks
├── i18n/                 # Internationalization
└── middleware.ts         # Next.js middleware
```

**Architectural Patterns**:

- **App Router**: File-based routing with layouts
- **Route Groups**: `(protected)` and `(public)` for authentication separation
- **Component Architecture**: UI components separated from business logic
- **Data Fetching**: TanStack Query for server state
- **Styling**: Tailwind CSS with component variants (CVA)

#### Shared Types (`packages/types/src/`)

- **Purpose**: Shared TypeScript types across backend and frontend
- **Build**: Compiled to CommonJS for compatibility
- **Distribution**: Published internally within monorepo

### Database Schema

**Tables**:

- `users`: User accounts with soft delete
- `organizations`: Organization/company entities
- `organization_members`: Many-to-many relationship with roles
- `github_installations`: GitHub app installations per organization
- `github_installation_repositories`: Repositories linked to installations

**Patterns**:

- Relational design with foreign keys
- Soft deletes (`deleted` boolean, `deletedAt` timestamp)
- Timestamps (`createdAt`, `updatedAt`) on all tables
- ID generation: Text-based IDs with `generateShortId` utility

## Development Tools

### Build & Bundling

- **tsup 8.5.0**: TypeScript bundler for packages and API
- **Next.js built-in**: Webpack/Turbopack for frontend
- **Turbo**: Monorepo build orchestration with caching
- **tsx 4.20.3**: TypeScript execution for development

### Code Quality

- **ESLint 9.22.0**: Linting with flat config format
  - Backend: `@eslint/js`, `typescript-eslint`, `prettier`
  - Frontend: `next/core-web-vitals`, `next/typescript`, `storybook`
  - Rules: `no-console: error` across all projects
- **Prettier 3.6.2**: Code formatting
  - Plugin: `@trivago/prettier-plugin-sort-imports` (import sorting)
  - Plugin: `prettier-plugin-tailwindcss` (Tailwind class sorting)
- **Knip 5.46.0**: Dead code detection
- **TypeScript strict mode**: Enabled on all projects
- **Husky 9.1.7**: Git hooks for pre-commit checks

### Database Management

- **Drizzle Kit 0.31.4**: Schema migrations and introspection
- **Custom Scripts** (in `apps/api/db-management/scripts/`):
  - `dump.sh`: Database backup
  - `dump-external.sh`: External database backup
  - `restore.sh`: Database restore
  - `migrate.sh`: Migration execution

### Containerization

- **Docker**: Multi-stage builds for optimization
  - Base image: `node:22.14.0-alpine`
  - Backend: Express API with non-root user (`expressjs:1001`)
  - Frontend: Next.js standalone output with non-root user (`nextjs:1001`)
  - Database: PostgreSQL 17 Alpine
- **Docker Compose**: Local development environment
  - Services: `web`, `api`, `db`
  - Network: Internal networking between services
  - Volumes: Database persistence

## Testing Strategy

### Unit & Integration Testing

- **Vitest 3.0+**: Test runner for both API and web
- **API Configuration** (`apps/api/vitest.config.ts`):
  - Environment: Node.js
  - Setup file: `./src/tests/setup.ts`
  - Coverage: v8 provider with text/json/html reporters
  - Pool: forks (isolated test execution)
  - Hook timeout: 15 seconds
  - Path alias: `@/` → `./src/`
- **Web Configuration** (`apps/web/vitest.config.mts`):
  - Environment: jsdom (browser simulation)
  - Plugins: React, tsconfig paths
  - Globals enabled
- **Test Utilities**:
  - `@testing-library/react 16.3.0`: Component testing
  - `@testing-library/jest-dom 6.6.3`: DOM matchers
  - `supertest 7.1.1`: HTTP endpoint testing
  - `@testcontainers/postgresql 10.28.0`: Database containers for tests
- **Mocking Strategy**:
  - Clerk authentication mocked in tests
  - External services mocked (Google Cloud Storage, OpenAI, Mistral, pdf-parse)
  - Test environment: `.env.test` file

### E2E Testing

- **Playwright 1.53.2**: End-to-end testing framework
- **Configuration** (`playwright.config.ts`):
  - Test directory: `./e2e-tests`
  - Fully parallel execution
  - Browsers: Chromium, Firefox (webkit commented out)
  - Retries: 2 on CI, 0 locally
  - Workers: 1 on CI, unlimited locally
  - Reporter: HTML
  - Trace: on-first-retry
- **Clerk Integration**: `@clerk/testing/playwright 1.9.3`
  - Global setup for authentication
  - Test credentials: `nefeta1440@protonza.com`
- **Environment Support**:
  - Local: `E2E_TEST_ENV=local`
  - Staging: `E2E_TEST_ENV=staging`
  - Production: `E2E_TEST_ENV=production`

### Test Organization

- **API Tests**: `apps/api/src/**/*.test.ts`
  - `src/server.test.ts`: Server configuration tests
  - `src/db/db.test.ts`: Database tests
  - `src/tests/routes/admin.test.ts`: Route tests
- **Web Tests**: `apps/web/src/**/*.test.tsx`
- **E2E Tests**: `e2e-tests/*.spec.ts`
  - `auth.spec.ts`: Authentication flow tests

## Build & Deployment

### Build Scripts

**Root Level**:

- `npm run build`: Build all workspaces via Turbo
- `npm run build:packages`: Build only packages workspace
- `npm run dev`: Start all services in development
- `npm run clean`: Clean all build artifacts

**API Scripts** (`apps/api/package.json`):

- `dev`: `tsx watch src/index.ts` (hot reload)
- `build`: `tsup ./src/index.ts` (bundle to CommonJS)
- `start`: `node dist/index.js` (production)
- `db:generate`: Generate Drizzle migrations
- `db:migrate`: Apply migrations
- `db:seed`: Seed database
- `db:init`: Migrate + seed (for fresh databases)

**Web Scripts** (`apps/web/package.json`):

- `dev`: `next dev --turbopack` (fast refresh with Turbopack)
- `build`: `next build` (optimized production build)
- `start`: `next start` (production server)
- `storybook`: `storybook dev -p 6006`
- `build-storybook`: Static Storybook build

### Turbo Pipeline

**Tasks Configuration** (`turbo.json`):

- `build`: Depends on workspace builds, caches outputs (dist, .next)
- `test`: Independent execution, caches coverage
- `lint`: Depends on builds
- `dev`: No cache, persistent mode
- `typecheck`: Depends on workspace typechecks

### Deployment Configuration

- **Frontend Output**: `standalone` (Next.js self-contained)
- **Docker Images**: Multi-stage builds with `turbo prune`
- **Environment Variables**: Build-time and runtime injection
- **Source Maps**:
  - API: Generated for Sentry (`sourceRoot: "/"`)
  - Web: Widens client file upload for better stack traces
- **Security**: Non-root users in containers, Helmet security headers

### Monitoring & Observability

- **Sentry**:
  - Release tracking via `COMMIT_SHA` environment variable
  - Source map upload via `@sentry/cli 2.47.0`
  - Profiling: Node profiling integration on API
  - Trace sampling: 100% (should be adjusted for production)
  - Tunneling: Next.js route `/monitoring` to avoid ad-blockers
- **PostHog**:
  - Proxying through Next.js rewrites (`/ingest/*`)
  - Trailing slash support enabled
- **Logging**:
  - Structured JSON logs via Pino
  - HTTP request logging via pino-http
  - Pretty printing in development (pino-pretty)

## Information Sources

### Configuration Files Scanned

- `/package.json` - Root workspace configuration
- `/turbo.json` - Monorepo build orchestration
- `/docker-compose.yml` - Local development services
- `/playwright.config.ts` - E2E test configuration
- `/knip.json` - Dead code detection rules
- `apps/api/package.json` - Backend dependencies
- `apps/api/tsconfig.json` - Backend TypeScript config
- `apps/api/vitest.config.ts` - Backend test config
- `apps/api/eslint.config.mjs` - Backend linting rules
- `apps/api/drizzle.config.ts` - ORM configuration
- `apps/api/Dockerfile` - Backend containerization
- `apps/web/package.json` - Frontend dependencies
- `apps/web/tsconfig.json` - Frontend TypeScript config
- `apps/web/vitest.config.mts` - Frontend test config
- `apps/web/eslint.config.mjs` - Frontend linting rules
- `apps/web/next.config.ts` - Next.js configuration
- `apps/web/Dockerfile` - Frontend containerization
- `packages/types/package.json` - Shared types configuration

### Source Code Scanned

- `apps/api/src/server.ts` - Express app setup
- `apps/api/src/db/schema.ts` - Database schema
- `apps/api/src/instrument.ts` - Sentry instrumentation
- `apps/api/src/tests/setup.ts` - Test mocking setup
- `apps/web/sentry.server.config.ts` - Frontend monitoring
- `e2e-tests/auth.spec.ts` - E2E authentication tests
- `e2e-tests/global.setup.ts` - E2E global setup

### Project Structure Analyzed

- `apps/api/src/routes/` - 6 route files
- `apps/api/src/middlewares/` - 4 middleware files
- `apps/web/src/components/` - 61 component files
- `apps/web/src/lib/` - 19 library files
- `apps/web/src/app/` - App Router structure

### Documentation Reviewed

- `/README.md` - Project overview and getting started guide
- `apps/api/db-management/README.md` - Database management docs
