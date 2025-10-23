# Architecture

## Metadata

- **Category**: Architecture
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/package.json`, `/turbo.json`, `/docker-compose.yml`, `/.cursorrules`

## Technical Overview

The application follows a **monorepo architecture** using Turborepo, with clear separation between frontend (Next.js) and backend (Express API). This architectural choice prioritizes technology independence, allowing the backend to evolve independently from frontend framework decisions.

The system is designed as a **modular monolith** - a single codebase organized into well-defined workspaces with clear boundaries. This approach balances simplicity with maintainability, avoiding the operational complexity of microservices while maintaining good separation of concerns.

Communication between frontend and backend follows a **RESTful API pattern**, with the backend exposing HTTP endpoints consumed by the frontend. The architecture supports containerized deployment via Docker, with orchestration handled locally through Docker Compose and in production through container platforms.

## Architecture & Design

### High-Level Architecture

```
monorepo-boucleai/
├── apps/
│   ├── api/                  # Backend Express API (Port 3001)
│   │   ├── src/
│   │   │   ├── routes/       # HTTP endpoints
│   │   │   ├── services/     # Business logic
│   │   │   ├── middlewares/  # Request processing
│   │   │   ├── db/           # Data access layer
│   │   │   └── utils/        # Shared utilities
│   │   └── drizzle/          # Database migrations
│   │
│   └── web/                  # Frontend Next.js (Port 3000)
│       └── src/
│           ├── app/          # Next.js App Router
│           ├── components/   # React components
│           ├── lib/          # Client utilities
│           └── hooks/        # React hooks
│
├── packages/
│   └── types/                # Shared TypeScript types
│
└── e2e-tests/                # End-to-end tests
```

### Design Principles

**1. Backend Independence**

- **Rationale**: "Je ne veux pas être trop dépendant de Next au cas où de nouvelles technologies arrivent en back-end."
- **Implementation**: Backend is a standalone Express application
- **Benefit**: Can migrate to different frameworks (Fastify, Hono, etc.) without touching frontend

**2. Type Safety Across Stack**

- **Shared Types Package**: Common TypeScript definitions used by both apps
- **Runtime Validation**: Zod schemas ensure type safety at API boundaries
- **No `any` Types**: Strict TypeScript configuration enforced

**3. Separation of Concerns**

```
Frontend (apps/web)           Backend (apps/api)
├── Presentation Layer        ├── HTTP Layer (routes)
├── State Management          ├── Business Logic (services)
├── Client-Side Logic         ├── Data Access (db)
└── UI Components             └── External Integrations
```

**4. Functional Programming Approach**

- No classes (use functions and closures)
- Pure functions when possible
- Immutable data structures
- Single responsibility principle

### Communication Architecture

**REST API Communication**:

```
┌─────────────────┐         HTTP/REST          ┌─────────────────┐
│                 │   ────────────────────>    │                 │
│   Next.js Web   │                            │   Express API   │
│   (Port 3000)   │   <────────────────────    │   (Port 3001)   │
│                 │         JSON Response       │                 │
└─────────────────┘                             └─────────────────┘
         │                                               │
         │                                               │
         ├─ TanStack Query (data fetching)             ├─ PostgreSQL
         ├─ Client State                                ├─ Clerk Auth
         └─ React Components                            └─ External APIs
```

**Data Flow**:

1. Frontend makes HTTP requests via TanStack Query
2. Backend routes handle requests
3. Middleware processes authentication, logging, transactions
4. Services implement business logic
5. Database layer manages data persistence
6. Response flows back through middleware to frontend

### Workspace Organization

**Turborepo Workspaces**:

```json
{
  "workspaces": [
    "apps/*", // Applications (api, web)
    "packages/*" // Shared packages (types)
  ]
}
```

**Dependencies**:

- **apps/api** → depends on **packages/types**
- **apps/web** → depends on **packages/types**
- **packages/types** → no dependencies (foundational)

**Build Order** (managed by Turborepo):

1. Build `packages/types` first
2. Build `apps/api` and `apps/web` in parallel

## Configuration & Setup

### Monorepo Configuration

**Turborepo Pipeline** (`turbo.json`):

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"], // Build dependencies first
      "outputs": ["dist/**", ".next/**"],
      "env": ["NEXT_PUBLIC_*", "SENTRY_*"]
    },
    "dev": {
      "cache": false, // Don't cache dev mode
      "persistent": true // Keep running
    },
    "test": {
      "outputs": ["coverage/**"],
      "dependsOn": [] // Independent
    }
  }
}
```

### Environment Configuration

**Development**:

- Local database via Docker Compose
- Hot reload on both apps (tsx watch, turbopack)
- Pretty-printed logs
- Mock external services in tests

**Staging** (deployed from local):

- Real database instance
- Integration testing environment
- Real external services
- Source maps enabled

**Production**:

- Containerized deployment
- Environment variables injected at runtime
- Optimized builds
- Health checks and monitoring

### Port Allocation

```
3000  → Frontend (Next.js)
3001  → Backend (Express API)
5432  → PostgreSQL (Docker)
6006  → Storybook (optional)
```

## Patterns & Best Practices

### ✅ DO: Maintain Clear Boundaries

```typescript
// Backend: Expose well-defined API contracts
export const userRoutes = Router();
userRoutes.get("/", validateResponse(userSchema), authMiddleware, handler);

// Frontend: Consume via type-safe client
const { data } = useQuery(["user"], () => apiClient.get<User>("/api/user"));
```

### ✅ DO: Share Types, Not Implementation

```typescript
// packages/types/src/user.ts
export const userSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  firstName: z.string(),
});

export type User = z.infer<typeof userSchema>;

// Used in both apps, but implementation differs
```

### ✅ DO: Version APIs for Breaking Changes

```typescript
// Future consideration (not yet implemented)
apiRoutes.use("/v1", v1Routes);
apiRoutes.use("/v2", v2Routes);
```

### ❌ DON'T: Couple Frontend to Backend Implementation

```typescript
// BAD: Frontend directly imports backend code
import { getUserFromDatabase } from "@api/db/users"; // ❌

// GOOD: Frontend uses API contract
const response = await fetch("/api/users"); // ✅
```

### ❌ DON'T: Duplicate Business Logic

```typescript
// BAD: Validation logic duplicated
// frontend/validation.ts
const validateEmail = (email) => /regex/.test(email);

// backend/validation.ts
const validateEmail = (email) => /different-regex/.test(email);

// GOOD: Share validation via Zod schemas in types package
import { emailSchema } from "@boilerplate-turbo/types";
```

## Usage Examples

### Starting the Monorepo

```bash
# Install all dependencies
npm install

# Start all apps in development
npm run dev

# Start specific app
cd apps/api && npm run dev
cd apps/web && npm run dev

# Build all apps for production
npm run build

# Run all tests
npm test
```

### Adding a New Workspace

```bash
# Create new package
mkdir -p packages/new-package
cd packages/new-package
npm init -y

# Add to workspace (automatic with npm workspaces)
# Update root package.json if needed
```

### Cross-Workspace Dependencies

```json
// apps/api/package.json
{
  "dependencies": {
    "@boilerplate-turbo/types": "*" // Workspace dependency
  }
}
```

## Technical Dependencies

**Monorepo Management**:

- `turbo`: ^2.4.4 - Build orchestration
- `npm workspaces` - Dependency management

**Shared Infrastructure**:

- `typescript`: ^5.7.3 - Type system
- `eslint`: ^9.22.0 - Linting
- `prettier`: ^3.6.2 - Code formatting
- `husky`: ^9.1.7 - Git hooks

**Containerization**:

- Docker (runtime)
- Docker Compose (local orchestration)

## Tests & Validation

**Testing Strategy**:

- **Unit Tests**: Per workspace (Vitest)
- **Integration Tests**: API with Testcontainers
- **E2E Tests**: Cross-app flows (Playwright)

**Running Tests**:

```bash
# All tests
npm test

# Specific workspace
npm test -- apps/api/src/routes/users.test.ts

# E2E tests
npm run test:e2e
```

## Historical Technical Decisions

**Decision: Monorepo vs Polyrepo**

- **Date**: Project inception
- **Rationale**: Simplified dependency management, atomic changes, shared tooling
- **Trade-offs**:
  - ✅ Easier refactoring across apps
  - ✅ Single source of truth for types
  - ⚠️ Larger repository size
  - ⚠️ More complex CI/CD setup

**Decision: Separate API and Web Apps**

- **Date**: Project inception
- **Rationale**: Backend technology independence (user input)
- **Trade-offs**:
  - ✅ Can change backend framework without frontend impact
  - ✅ Clear separation of concerns
  - ⚠️ Cannot use Next.js API routes directly
  - ⚠️ Additional deployment complexity

**Decision: Modular Monolith (not Microservices)**

- **Date**: Project inception
- **Rationale**: "Pas pour le moment" - current scale doesn't require microservices
- **Trade-offs**:
  - ✅ Simpler deployment and operations
  - ✅ Lower operational overhead
  - ✅ Easier local development
  - ⚠️ Future scaling may require architectural changes

**[To be documented by CTO]**

- Performance benchmarks and scaling triggers
- Plans for future microservices migration (if any)
- CI/CD pipeline details

## Troubleshooting & FAQ

**Issue**: Changes in types package not reflected in apps

- **Cause**: Types package not rebuilt
- **Solution**: Run `npm run build:packages` or `turbo build --filter='./packages/*'`

**Issue**: Ports already in use

- **Cause**: Previous instances still running
- **Solution**: Kill processes on ports 3000/3001 or use different ports

**Issue**: Docker Compose services not communicating

- **Cause**: Service names in docker-compose.yml don't match hostnames
- **Solution**: Use service names as hostnames (e.g., `api:3001`, not `localhost:3001`)

**Issue**: npm install fails with workspace errors

- **Cause**: Workspace dependencies out of sync
- **Solution**: Delete all `node_modules` and run `npm install` from root

## References

- **Main Files**:
  - `/package.json` - Root workspace configuration
  - `/turbo.json` - Build pipeline
  - `/docker-compose.yml` - Local services orchestration
  - `/.cursorrules` - Development guidelines

- **Configuration**:
  - `/tsconfig.json` - Base TypeScript config
  - `/apps/api/tsconfig.json` - Backend TypeScript
  - `/apps/web/tsconfig.json` - Frontend TypeScript

- **External Docs**:
  - [Turborepo Documentation](https://turbo.build/repo/docs)
  - [npm Workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces)
  - [Docker Compose](https://docs.docker.com/compose/)

## Product Vision Context

The application is an **AI Context Management Platform** enabling teams to:

- Manage **Workflows** (Markdown-based configurations)
- Configure **Agents** (AI agent definitions)
- Version **Prompts** (Prompt templates)

**Key Capabilities**:

- **Observability**: Track and monitor AI operations
- **Governance**: Control and audit AI usage
- **Collaboration**: Team-based context sharing
- **Versioning**: Version control for AI configurations

**Development Approach**: Customer-driven feature development based on user interviews and feedback.
