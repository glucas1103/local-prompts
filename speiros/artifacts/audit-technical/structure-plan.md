# Technical Context Structure Plan

## Overview

This document outlines the complete technical documentation structure for the monorepo. Each file will be created following the `technical-context-template.yaml` and populated with information from codebase analysis and user inputs.

---

## Directory Structure

```
context/technical/
├── context/              # Core technical domains (8 files)
├── tools/                # External tools and integrations (9 files)
├── patterns/             # Coding patterns and conventions (6 files)
└── memory/               # Historical decisions and troubleshooting (4 files)

Total: 27 documentation files
```

---

## Detailed Structure Plan

### 📁 context/ - Core Technical Domains

#### 1. ARCHITECTURE.md

- **Category**: Architecture
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Overall system architecture and design principles

**Information to Document**:

- Monorepo structure and workspace organization
- Separation of concerns (API vs Web)
- Technology independence strategy
- Communication patterns (REST API)
- Modular monolith approach
- Future evolution considerations

**Key Files to Examine**:

- `/package.json`, `/turbo.json`
- `/apps/api/src/server.ts`
- `/apps/web/next.config.ts`
- `/docker-compose.yml`
- `/.cursorrules`

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Patterns & Best Practices ✓
- Technical Dependencies ✓
- Historical Technical Decisions (user input)

---

#### 2. API.md

- **Category**: Architecture
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Backend API architecture, routes, and conventions

**Information to Document**:

- Express server configuration
- Middleware pipeline (order and purpose)
- Route organization patterns
- Request/response lifecycle
- Error handling in API
- REST conventions
- Transaction management via `req.tx`

**Key Files to Examine**:

- `/apps/api/src/server.ts`
- `/apps/api/src/routes/*.ts`
- `/apps/api/src/middlewares/*.ts`
- `/.cursorrules` (Router Rules)

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- API & Interfaces ✓
- Patterns & Best Practices ✓
- Configuration & Setup ✓
- Usage Examples ✓

---

#### 3. DATABASE.md

- **Category**: Architecture
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Database schema, ORM usage, and data access patterns

**Information to Document**:

- PostgreSQL database design
- Drizzle ORM schema definitions
- Table structure and relationships
- Soft delete pattern
- ID generation strategy (`generateShortId`)
- Migration workflow
- Transaction patterns
- Seed data management

**Key Files to Examine**:

- `/apps/api/src/db/schema.ts`
- `/apps/api/drizzle.config.ts`
- `/apps/api/drizzle/*.sql` (migrations)
- `/apps/api/src/db/seed/*.ts`
- `/apps/api/db-management/scripts/*.sh`

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Configuration & Setup ✓
- Patterns & Best Practices ✓
- Usage Examples ✓
- Historical Technical Decisions **[To be documented - soft delete & ID rationale]**

---

#### 4. FRONTEND.md

- **Category**: Architecture
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Frontend architecture, routing, and component patterns

**Information to Document**:

- Next.js App Router structure
- Route groups (protected/public)
- Component organization (61 components)
- State management (TanStack Query)
- Styling system (Tailwind + Shadcn)
- Data fetching patterns
- Client vs Server components
- Internationalization (next-intl)

**Key Files to Examine**:

- `/apps/web/next.config.ts`
- `/apps/web/src/app/**/page.tsx`
- `/apps/web/src/components/**/*.tsx`
- `/apps/web/src/lib/**/*.ts`
- `/apps/web/components.json`

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Patterns & Best Practices ✓
- Configuration & Setup ✓
- Usage Examples ✓

---

#### 5. AUTHENTICATION.md

- **Category**: Pattern
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Authentication and authorization implementation

**Information to Document**:

- Clerk integration (frontend and backend)
- Auth middleware implementation
- Protected routes (frontend and backend)
- Session management
- Role-based access control
- Sign-in/sign-up flows
- Token validation

**Key Files to Examine**:

- `/apps/api/src/middlewares/authMiddleware.ts`
- `/apps/web/src/middleware.ts`
- `/apps/web/src/app/(protected)/**`
- `/apps/web/src/app/(public)/(auth)/**`
- `/e2e-tests/auth.spec.ts`

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Configuration & Setup ✓
- Patterns & Best Practices ✓
- Usage Examples ✓
- Tests & Validation ✓

---

#### 6. ERROR-HANDLING.md

- **Category**: Pattern
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Error management, logging, and monitoring

**Information to Document**:

- Error propagation patterns (db → services → routes)
- Structured logging (Pino)
- Error classification and standard responses
- Sentry integration
- Error middleware
- Frontend error boundaries
- Logging best practices (from .cursorrules)

**Key Files to Examine**:

- `/apps/api/src/server.ts` (error handler)
- `/apps/api/src/middlewares/loggerMiddleware.ts`
- `/apps/api/src/utils/logger.ts`
- `/apps/api/src/instrument.ts`
- `/apps/web/src/app/global-error.tsx`
- `/.cursorrules` (error handling rules)

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Patterns & Best Practices ✓
- Configuration & Setup ✓
- Usage Examples ✓

---

#### 7. TESTING.md

- **Category**: Guideline
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Testing strategy, patterns, and utilities

**Information to Document**:

- Test infrastructure (Vitest + Playwright)
- Unit testing patterns
- Integration testing with Testcontainers
- E2E testing with Clerk
- Mocking strategies
- Test organization
- Staging environment testing
- Test utilities and fixtures

**Key Files to Examine**:

- `/apps/api/vitest.config.ts`
- `/apps/web/vitest.config.mts`
- `/playwright.config.ts`
- `/apps/api/src/tests/setup.ts`
- `/e2e-tests/**/*.spec.ts`
- `/apps/api/src/tests/routes/*.test.ts`
- `/.cursorrules` (testing structure)

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Configuration & Setup ✓
- Patterns & Best Practices ✓
- Usage Examples ✓
- Technical Roadmap **[Test coverage targets - to be defined by CTO]**

---

#### 8. DEPLOYMENT.md

- **Category**: Guideline
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Build, deployment, and operations

**Information to Document**:

- Build pipeline (Turborepo)
- Docker containerization (multi-stage)
- Environment variable management
- Source map generation
- Deployment flow (local → staging → production)
- Database migrations in deployment
- Health checks and readiness

**Key Files to Examine**:

- `/turbo.json`
- `/docker-compose.yml`
- `/apps/api/Dockerfile`
- `/apps/web/Dockerfile`
- `/apps/api/package.json` (scripts)
- `/apps/web/package.json` (scripts)

**Template Sections**:

- Technical Overview ✓
- Architecture & Design ✓
- Configuration & Setup ✓
- Patterns & Best Practices ✓
- Usage Examples ✓
- Troubleshooting & FAQ **[To be documented from experience]**

---

### 🔧 tools/ - External Tools and Integrations

#### 9. TURBOREPO.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Monorepo build orchestration

**Information to Document**:

- Turbo pipeline configuration
- Task dependencies
- Caching strategy
- Workspace management
- Build optimization

**Key Files**: `/turbo.json`, `/package.json`

---

#### 10. DRIZZLE.md

- **Category**: Tool
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: ORM configuration and database migrations

**Information to Document**:

- Drizzle Kit setup
- Schema management
- Migration generation and execution
- Type generation
- Database connection configuration

**Key Files**: `/apps/api/drizzle.config.ts`, `/apps/api/src/db/**`

---

#### 11. CLERK.md

- **Category**: Tool
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Authentication service integration

**Information to Document**:

- Clerk SDK configuration (Next.js + Express)
- Webhook handling
- Environment variables
- Testing with Clerk Testing
- User management flows

**Key Files**:

- `/apps/api/src/routes/clerk.ts`
- `/apps/web/src/middleware.ts`
- `/e2e-tests/global.setup.ts`

---

#### 12. SENTRY.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Error tracking and performance monitoring

**Information to Document**:

- Sentry configuration (API + Web)
- Source map upload
- Error tracking
- Performance monitoring
- Release tracking

**Key Files**:

- `/apps/api/src/instrument.ts`
- `/apps/web/sentry.server.config.ts`
- `/apps/web/sentry.edge.config.ts`
- `/apps/web/next.config.ts` (Sentry plugin)

---

#### 13. POSTHOG.md

- **Category**: Tool
- **Priority**: LOW
- **Status**: Active
- **Purpose**: Analytics and feature flags

**Information to Document**:

- PostHog configuration
- Proxy setup (Next.js rewrites)
- Analytics tracking
- Feature flags usage

**Key Files**: `/apps/web/next.config.ts`

---

#### 14. GITHUB-INTEGRATION.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: GitHub API and webhooks

**Information to Document**:

- Octokit app setup
- GitHub app permissions
- Webhook handling
- Repository integration
- Installation management

**Key Files**:

- `/apps/api/src/routes/github.ts`
- `/apps/api/src/services/github.ts`
- `/apps/api/src/db/schema.ts` (github tables)

---

#### 15. DOCKER.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Containerization

**Information to Document**:

- Multi-stage Docker builds
- Docker Compose setup
- Service orchestration
- Non-root users
- Volume management

**Key Files**:

- `/docker-compose.yml`
- `/apps/api/Dockerfile`
- `/apps/web/Dockerfile`

---

#### 16. PLAYWRIGHT.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: E2E testing framework

**Information to Document**:

- Playwright configuration
- Browser setup
- Global setup patterns
- Clerk testing integration
- Multi-environment testing

**Key Files**:

- `/playwright.config.ts`
- `/e2e-tests/**/*.ts`

---

#### 17. VITEST.md

- **Category**: Tool
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Unit testing framework

**Information to Document**:

- Vitest configuration (API + Web)
- Coverage reporting
- Test environment setup
- Mocking strategies
- Test utilities

**Key Files**:

- `/apps/api/vitest.config.ts`
- `/apps/web/vitest.config.mts`
- `/apps/api/src/tests/setup.ts`

---

### 🎨 patterns/ - Coding Patterns and Conventions

#### 18. MIDDLEWARE.md

- **Category**: Pattern
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Express middleware patterns

**Information to Document**:

- Middleware pipeline order
- Transaction middleware (`req.tx`)
- Logger middleware
- Validation middleware
- Auth middleware
- Custom middleware development

**Key Files**: `/apps/api/src/middlewares/*.ts`

---

#### 19. REACT-COMPONENTS.md

- **Category**: Pattern
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Component architecture and organization

**Information to Document**:

- Component structure (from .cursorrules)
- Props typing patterns
- Functional components (no classes)
- Shadcn UI usage
- Component composition
- Naming conventions

**Key Files**:

- `/apps/web/src/components/**/*.tsx`
- `/.cursorrules` (Component Rules)

---

#### 20. API-ROUTES.md

- **Category**: Pattern
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: API route organization and conventions

**Information to Document**:

- Route file structure
- Router export patterns
- Error handling in routes
- Request/response patterns
- Status codes
- REST conventions

**Key Files**:

- `/apps/api/src/routes/*.ts`
- `/.cursorrules` (Router Rules)

---

#### 21. DATABASE-OPERATIONS.md

- **Category**: Pattern
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: Data access patterns and conventions

**Information to Document**:

- Using `req.tx` for all operations
- Transaction patterns
- Query building with Drizzle
- Relations and joins
- Soft delete patterns
- Seed data patterns

**Key Files**:

- `/apps/api/src/db/**/*.ts`
- `/.cursorrules` (Database Access)

---

#### 22. ERROR-PROPAGATION.md

- **Category**: Pattern
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Error handling conventions

**Information to Document**:

- try/catch patterns
- Error logging standards
- Error response format
- Error codes (SCREAMING_SNAKE_CASE)
- Error propagation layers

**Key Files**:

- `/apps/api/src/routes/*.ts`
- `/.cursorrules` (error handling examples)

---

#### 23. TYPE-SAFETY.md

- **Category**: Pattern
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: TypeScript usage patterns

**Information to Document**:

- Strict TypeScript configuration
- No `any` types rule
- Zod validation integration
- Shared types package
- Runtime validation
- Type-safe database queries

**Key Files**:

- `/apps/api/tsconfig.json`
- `/apps/web/tsconfig.json`
- `/packages/types/src/**/*.ts`
- `/.cursorrules` (Type Safety)

---

### 💾 memory/ - Historical Decisions and Troubleshooting

#### 24. TECHNICAL-CHOICES.md

- **Category**: Memory
- **Priority**: HIGH
- **Status**: Active
- **Purpose**: ADR-style technical decisions

**Information to Document**:

- Technology stack selection rationale
- Architecture decisions (API separation)
- Database design decisions
- Testing strategy choices
- Deployment approach decisions

**Sources**:

- User Q&A responses
- Patterns observed in codebase
- **[Sections requiring CTO input marked]**

---

#### 25. TROUBLESHOOTING.md

- **Category**: Memory
- **Priority**: MEDIUM
- **Status**: Active
- **Purpose**: Common issues and solutions

**Information to Document**:

- Common development issues
- Database connection problems
- Port conflicts
- Module resolution issues
- Docker troubleshooting
- Migration issues

**Sources**:

- README troubleshooting section
- Error patterns in code
- **[To be enriched with operational experience]**

---

#### 26. MIGRATIONS.md

- **Category**: Memory
- **Priority**: LOW
- **Status**: Active
- **Purpose**: Database migration history and notes

**Information to Document**:

- Migration naming conventions
- Migration execution workflow
- Production migration procedures
- Rollback strategies
- Migration history

**Key Files**:

- `/apps/api/drizzle/*.sql`
- `/apps/api/drizzle/meta/*.json`
- `/apps/api/db-management/**`

---

#### 27. REFACTORING-LOG.md

- **Category**: Memory
- **Priority**: LOW
- **Status**: Active
- **Purpose**: Major refactoring notes and lessons learned

**Information to Document**:

- Major refactorings performed
- Lessons learned
- Future refactoring considerations
- Technical debt tracking

**Sources**:

- **[To be populated as refactorings occur]**
- **[CTO to document major changes]**

---

## Research Assignments for Sub-Agent

### High Priority (Process First)

1. **API** - Express architecture and middleware
2. **DATABASE** - Schema and ORM patterns
3. **AUTHENTICATION** - Clerk integration and patterns
4. **ERROR-HANDLING** - Logging and error management
5. **MIDDLEWARE** - Express middleware patterns
6. **API-ROUTES** - Route organization
7. **DATABASE-OPERATIONS** - Data access patterns
8. **DRIZZLE** - ORM configuration
9. **CLERK** - Authentication tool integration

### Medium Priority

10. **FRONTEND** - Next.js App Router architecture
11. **TESTING** - Test strategies and utilities
12. **DEPLOYMENT** - Build and containerization
13. **GITHUB-INTEGRATION** - API and webhooks
14. **DOCKER** - Containerization setup
15. **PLAYWRIGHT** - E2E testing
16. **VITEST** - Unit testing
17. **REACT-COMPONENTS** - Component patterns
18. **ERROR-PROPAGATION** - Error handling conventions
19. **TYPE-SAFETY** - TypeScript patterns
20. **SENTRY** - Error tracking setup

### Low Priority (Document Later or As Needed)

21. **ARCHITECTURE** - High-level overview (can compile from other docs)
22. **TURBOREPO** - Build orchestration
23. **POSTHOG** - Analytics configuration
24. **TECHNICAL-CHOICES** - Compile from user input + observations
25. **TROUBLESHOOTING** - Aggregate from README + experience
26. **MIGRATIONS** - Database migration log
27. **REFACTORING-LOG** - Future documentation

---

## Template Application Strategy

### For Each File

1. **Read** relevant source files
2. **Extract** patterns, configurations, examples
3. **Apply** technical-context-template.yaml
4. **Fill** available sections with factual information
5. **Mark** missing information with standardized placeholders:
   - `**[To be documented by CTO]**`
   - `**[Requires business input]**`
   - `**[To be enriched with operational experience]**`
6. **Include** code examples from actual codebase
7. **Cross-reference** related documentation files

### Placeholder Format

```markdown
### Section Name

**[Information Gap]**
This section requires input from the CTO or further operational experience.

- **What**: Brief description of missing information
- **Why Important**: Impact on development/operations
- **Owner**: CTO (Lucas Gaillard) / Team
- **Priority**: High / Medium / Low
```

---

## Success Criteria

- ✅ All 27 files defined with clear purpose
- ✅ Priority levels assigned
- ✅ Key source files identified
- ✅ Research scope defined
- ✅ Template sections mapped
- ✅ Placeholder strategy defined
- ✅ User validated approach

---

## Next Steps

1. **Step 5**: Delegate research to sub-agent for each domain (iterative)
2. **Step 6**: Populate templates with research findings (iterative)
3. **Step 7**: Cleanup temporary artifacts

**Estimated Time**: 15-25 hours of research and documentation generation
