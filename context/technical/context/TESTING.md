# Testing Strategy

## Metadata

- **Category**: Guideline
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/vitest.config.ts`, `apps/web/vitest.config.mts`, `playwright.config.ts`, `apps/api/src/tests/**`

## Technical Overview

The testing strategy implements a **three-tier approach**: unit tests for isolated logic, integration tests with real database containers, and end-to-end tests for complete user flows. This ensures comprehensive coverage from function-level correctness to full-stack user scenarios.

**Vitest 3.0+** powers unit and integration testing with fast execution, watch mode, and coverage reporting. Backend tests use **Testcontainers** to spin up ephemeral PostgreSQL instances, providing true integration testing without mocking database behavior. Frontend tests run in jsdom environment with React Testing Library for component validation.

**Playwright 1.53.2** handles E2E testing across Chromium and Firefox browsers, with Clerk testing SDK enabling authentication flows. Tests can target multiple environments (local, staging, production) via configuration, with staging serving as the primary integration testing environment per CTO guidance: "Je testerai avec l'Environment staging sur lequel je deploy depuis local."

All external services (Clerk auth, Google Cloud, OpenAI, Mistral) are mocked in unit tests for speed and isolation, with real integration validated on staging deployments.

## Architecture & Design

### Test Pyramid Strategy

```
                    ┌────────────┐
                    │    E2E     │  ← Few, slow, high confidence
                    │ (Playwright)│    Critical user journeys
                    └────────────┘
                   ┌──────────────┐
                   │  Integration  │  ← Some, medium speed
                   │ (Testcontainers)│   Database + API together
                   └──────────────┘
              ┌──────────────────────┐
              │      Unit Tests       │  ← Many, fast, focused
              │   (Vitest + Mocks)   │    Individual functions
              └──────────────────────┘
```

### Testing Layers

**Unit Tests** (~70% of tests):

- Individual functions and utilities
- Mocked external dependencies
- Fast execution (milliseconds)
- High isolation

**Integration Tests** (~20% of tests):

- API routes with real database
- Testcontainers for PostgreSQL
- Medium execution (seconds)
- Tests interactions between layers

**E2E Tests** (~10% of tests):

- Complete user flows
- Real browser automation
- Slow execution (seconds to minutes)
- Tests entire system

### Test Organization

```
apps/api/src/
├── **/*.test.ts          # Co-located unit tests
├── server.test.ts        # Server configuration tests
├── db/db.test.ts         # Database connection tests
└── tests/
    ├── setup.ts          # Global test setup & mocks
    ├── setup-db.ts       # Testcontainers setup helper
    ├── routes/           # Integration tests for routes
    │   └── admin.test.ts
    └── fixtures/         # Test data

apps/web/src/
├── components/**/*.test.tsx   # Component tests
└── lib/**/*.test.ts           # Utility tests

e2e-tests/
├── auth.spec.ts          # E2E authentication tests
├── global.setup.ts       # Playwright global setup
└── utils.ts              # Test utilities (environment selection)
```

## Configuration & Setup

### Vitest Configuration (Backend)

**File**: `apps/api/vitest.config.ts`

```typescript
import path from "path";
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true, // Use global test functions
    environment: "node", // Node.js environment
    setupFiles: ["./src/tests/setup.ts"], // Global setup & mocks
    include: ["**/*.test.ts", "**/*.spec.ts"],
    exclude: ["node_modules", "dist"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: ["**/*.test.ts", "**/*.spec.ts", "**/tests/**"],
    },
    deps: {
      interopDefault: true,
    },
    pool: "forks", // Isolated test execution
    hookTimeout: 15000, // 15 seconds for setup/teardown
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### Vitest Configuration (Frontend)

**File**: `apps/web/vitest.config.mts`

```typescript
import react from "@vitejs/plugin-react";
import tsconfigPaths from "vite-tsconfig-paths";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: "jsdom", // Browser-like environment
    globals: true,
    setupFiles: ["./src/tests/setup.ts"],
  },
});
```

### Playwright Configuration

**File**: `playwright.config.ts`

```typescript
import { defineConfig, devices } from "@playwright/test";
import { getBaseUrl } from "./e2e-tests/utils";

export default defineConfig({
  testDir: "./e2e-tests",
  fullyParallel: true, // Run tests in parallel
  forbidOnly: !!process.env.CI, // No .only in CI
  retries: process.env.CI ? 2 : 0, // Retry on CI
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",

  use: {
    baseURL: getBaseUrl(), // Dynamic based on E2E_TEST_ENV
    trace: "on-first-retry",
  },

  projects: [
    {
      name: "global setup",
      testDir: "./e2e-tests",
      testMatch: /global\.setup\.ts/,
    },
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
      dependencies: ["global setup"],
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
      dependencies: ["global setup"],
    },
  ],
});
```

### Test Environment Selection

**File**: `e2e-tests/utils.ts`

```typescript
export function getBaseUrl() {
  switch (process.env.E2E_TEST_ENV) {
    case "staging":
      return "https://staging.boucle.ai";
    case "production":
      return "https://boucle.ai";
    default:
      return "http://localhost:3000";
  }
}
```

**Usage**:

```bash
npm run test:e2e              # Local (http://localhost:3000)
npm run test:e2e:staging      # Staging environment
npm run test:e2e:production   # Production environment
```

### Global Test Setup

**File**: `apps/api/src/tests/setup.ts`

```typescript
import { config } from "dotenv";
import { vi } from "vitest";

// Mock Clerk authentication
vi.mock("@clerk/express", () => ({
  clerkMiddleware: vi.fn().mockImplementation(() => {
    return (req, res, next) => {
      next();
    };
  }),
}));

// Mock external services
vi.mock("@google-cloud/storage", () => ({
  Storage: vi.fn().mockImplementation(() => ({
    bucket: vi.fn().mockImplementation(() => ({
      exists: vi.fn().mockResolvedValue([true]),
      file: vi.fn().mockImplementation(() => ({
        save: vi.fn(),
      })),
    })),
  })),
}));

vi.mock("openai", () => ({
  default: vi.fn().mockImplementation(() => ({
    chat: {
      completions: {
        create: vi.fn().mockResolvedValue({
          choices: [{ message: { content: "Mock response from OpenAI" } }],
        }),
      },
    },
  })),
}));

vi.mock("@mistralai/mistralai", () => ({
  Mistral: vi.fn().mockImplementation(() => ({
    ocr: {
      process: vi.fn().mockResolvedValue({
        pages: [{ markdown: "# Mock content" }],
      }),
    },
  })),
}));

// Load test environment variables
config({ path: ".env.test" });
```

## Patterns & Best Practices

### ✅ DO: Unit Test with Mocked Dependencies

```typescript
// apps/api/src/db/db.test.ts
import { beforeEach, describe, expect, it, vi } from "vitest";
import { createDbConnection } from "./db";

vi.mock("pg", () => ({
  Pool: vi.fn(() => ({
    connect: vi.fn(),
    on: vi.fn(),
  })),
}));

describe("Database", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("should create database connection with custom options", () => {
    const customOptions = {
      host: "localhost",
      port: 5432,
      database: "test_db",
      user: "test_user",
      password: "test_password",
      ssl: false,
    };

    const { pool, config } = createDbConnection(customOptions);

    expect(pool).toBeDefined();
    expect(config).toEqual({
      ...customOptions,
      max: 5,
      connectionTimeoutMillis: 10000,
    });
  });
});
```

### ✅ DO: Integration Test with Testcontainers

```typescript
// apps/api/src/tests/routes/admin.test.ts
import { NodePgDatabase } from "drizzle-orm/node-postgres";
import express from "express";
import request from "supertest";
import { beforeAll, describe, expect, it } from "vitest";
import { seedUnitTestingUsers } from "@/db/seed/fixtures";
import { adminRouter } from "@/routes/admin";
import { setupAppWithDatabase } from "../setup-db";

describe("Admin Routes", () => {
  let app: express.Application;
  let db: NodePgDatabase;
  let teardownDatabase: () => Promise<void>;

  beforeAll(async () => {
    // Setup Testcontainer with real PostgreSQL
    ({ app, db, teardownDatabase } = await setupAppWithDatabase());

    // Seed test data
    await seedUnitTestingUsers(db);

    // Mount routes
    app.use("/api/admin", adminRouter);

    return async () => {
      await teardownDatabase(); // Cleanup container
    };
  });

  describe("Get organizations", () => {
    it("should return all organizations", async () => {
      const response = await request(app).get("/api/admin/organizations");

      expect(response.status).toBe(200);
      expect(response.body.length).toEqual(1);
    });
  });
});
```

### ✅ DO: E2E Test with Clerk Authentication

```typescript
// e2e-tests/auth.spec.ts
import { test, expect } from "@playwright/test";
import { clerk, setupClerkTestingToken } from "@clerk/testing/playwright";

test("log in and log out", async ({ page }) => {
  await setupClerkTestingToken({ page });
  await page.goto("/");

  // Should redirect to sign-in
  await expect(page).toHaveURL("/sign-in");

  // Sign in with test credentials
  await clerk.signIn({
    page,
    signInParams: {
      strategy: "password",
      identifier: "nefeta1440@protonza.com",
      password: "testing-ACCOUNT-password-123!",
    },
  });

  // Should redirect to dashboard
  await expect(page).toHaveURL("/dashboard");

  // Verify authenticated UI
  const repositoriesSwitcher = await page.getByTestId("repositories-switcher");
  await expect(repositoriesSwitcher).toBeVisible();

  // Sign out flow
  await page.getByTestId("sidebar-user-button").click();
  await page.getByTestId("sidebar-logout-button").click();
  await page.getByTestId("sidebar-confirm-logout-button").click();

  // Should redirect to sign-in
  await expect(page).toHaveURL("/sign-in");
});
```

### ✅ DO: Mock Auth for Route Tests

```typescript
import { mockAuthMiddleware } from "@/middlewares/authMiddleware";
import request from "supertest";
import { createServer } from "@/server";

describe("User Routes", () => {
  let app;

  beforeEach(() => {
    app = createServer();
    app.use(mockAuthMiddleware()); // Mock authentication
  });

  it("should get current user", async () => {
    const response = await request(app).get("/api/user").expect(200);

    expect(response.body).toHaveProperty("id");
    expect(response.body).toHaveProperty("email");
  });

  it("should require admin for admin routes", async () => {
    app.use(mockAuthMiddleware(false)); // Non-admin user

    const response = await request(app).get("/api/admin/settings").expect(401);

    expect(response.body.error).toBe("UNAUTHORIZED - ADMIN ONLY");
  });
});
```

### ✅ DO: Test with Fixtures

```typescript
import {
  createUserFixture,
  createOrganisationFixture,
} from "@boilerplate-turbo/types";

describe("User Operations", () => {
  it("should create user with organization", async () => {
    const org = createOrganisationFixture();
    const user = createUserFixture(org.id);

    const [createdUser] = await db.insert(users).values(user).returning();

    expect(createdUser.id).toBe(user.id);
    expect(createdUser.organizationId).toBe(org.id);
  });
});
```

### ❌ DON'T: Test Implementation Details

```typescript
// BAD: Testing internal state
it("should set internal flag", () => {
  const service = new UserService();
  service.createUser(data);
  expect(service._internalFlag).toBe(true); // ❌
});

// GOOD: Test observable behavior
it("should create user", async () => {
  const user = await createUser(data);
  expect(user.id).toBeDefined(); // ✅
});
```

### ❌ DON'T: Share State Between Tests

```typescript
// BAD: Shared state
let sharedUser;

it("test 1", () => {
  sharedUser = createUser(); // ❌ Affects other tests
});

it("test 2", () => {
  expect(sharedUser).toBeDefined(); // ❌ Depends on test 1
});

// GOOD: Isolated tests
it("test 1", () => {
  const user = createUser(); // ✅ Local state
  expect(user).toBeDefined();
});

it("test 2", () => {
  const user = createUser(); // ✅ Independent
  expect(user).toBeDefined();
});
```

## Usage Examples

### Complete Integration Test Example

```typescript
// apps/api/src/tests/routes/organizations.test.ts
import { setupAppWithDatabase } from '../setup-db';
import request from 'supertest';

describe('Organization API', () => {
  let app, db, teardown;

  beforeAll(async () => {
    ({ app, db, teardownDatabase: teardown } = await setupAppWithDatabase());
    app.use(mockAuthMiddleware());

    return () => teardown();
  });

  it('should create organization', async () => {
    const orgData = {
      name: 'Test Corp',
      plan: 'FREE',
    };

    const response = await request(app)
      .post('/api/organizations')
      .send(orgData)
      .expect(201);

    expect(response.body).toMatchObject(orgData);
    expect(response.body.id).toBeDefined();

    // Verify in database
    const [org] = await db
      .select()
      .from(organizations)
      .where(eq(organizations.id, response.body.id))
      .limit(1);

    expect(org).toBeDefined();
    expect(org.name).toBe(orgData.name);
  });

  it('should soft delete organization', async () => {
    const [org] = await db.insert(organizations).values({...}).returning();

    await request(app)
      .delete(`/api/organizations/${org.id}`)
      .expect(200);

    // Verify soft delete
    const [deleted] = await db
      .select()
      .from(organizations)
      .where(eq(organizations.id, org.id))
      .limit(1);

    expect(deleted.deleted).toBe(true);
    expect(deleted.deletedAt).toBeDefined();
  });
});
```

### Frontend Component Test

```typescript
// apps/web/src/components/ui/button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './button';

describe('Button Component', () => {
  it('should render with label', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should call onClick handler', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);

    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled', () => {
    render(<Button disabled>Click</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### E2E Complete User Journey

```typescript
test("complete user journey: sign up → dashboard → repos", async ({ page }) => {
  await setupClerkTestingToken({ page });

  // 1. Sign up
  await page.goto("/sign-up");
  await clerk.signUp({
    page,
    signUpParams: {
      emailAddress: "new@example.com",
      password: "SecurePass123!",
    },
  });

  // 2. Should land on dashboard
  await expect(page).toHaveURL("/dashboard");

  // 3. Connect GitHub
  await page.getByTestId("connect-github-button").click();
  // ... GitHub OAuth flow

  // 4. Add repository
  await page.getByTestId("add-repo-button").click();
  await page.getByTestId("repo-checkbox-1").check();
  await page.getByTestId("confirm-add").click();

  // 5. Verify repository appears
  await expect(page.getByTestId("repo-card-1")).toBeVisible();
});
```

## Patterns & Best Practices

### Testing Strategies

**Unit Test Characteristics**:

- Fast (< 100ms per test)
- Isolated (mocked dependencies)
- Focused on single function/module
- No database, no network

**Integration Test Characteristics**:

- Medium speed (< 5s per test)
- Real database (Testcontainers)
- Tests multiple layers together
- May include external services on staging

**E2E Test Characteristics**:

- Slow (5-60s per test)
- Full stack with real browser
- Tests complete user flows
- Runs on staging for integration validation

### Mock Strategy

**What to Mock in Unit Tests**:

- ✅ External APIs (Clerk, GitHub, OpenAI)
- ✅ Database connections
- ✅ File system operations
- ✅ Network requests

**What NOT to Mock in Integration Tests**:

- ❌ Database (use Testcontainers)
- ❌ Internal business logic
- ❌ Middleware pipeline

**What NOT to Mock in E2E Tests**:

- ❌ Frontend rendering
- ❌ API endpoints
- ❌ Authentication flow (use Clerk testing SDK)
- ⚠️ External services (use staging environment)

## Technical Dependencies

**Unit/Integration Testing**:

- `vitest`: ^3.2.4 - Test runner
- `@vitest/coverage-v8`: ^3.2.4 - Coverage reporter
- `@testcontainers/postgresql`: ^10.28.0 - Database containers
- `supertest`: ^7.1.1 - HTTP testing
- `@testing-library/react`: ^16.3.0 - React testing
- `@testing-library/jest-dom`: ^6.6.3 - DOM matchers
- `jsdom`: ^26.1.0 - Browser simulation

**E2E Testing**:

- `@playwright/test`: ^1.53.2 - Browser automation
- `@clerk/testing`: ^1.9.3 - Clerk test utilities

**Fixtures**:

- `@boilerplate-turbo/types` - Test fixtures and utilities

## Tests & Validation

### Running Tests

**All Tests**:

```bash
npm test                    # Run all unit/integration tests
npm run test:watch          # Watch mode
npm run test:coverage       # With coverage report
```

**Specific Tests**:

```bash
# Specific file
npm test -- apps/api/src/routes/users.test.ts

# Pattern matching
npm test -- --testNamePattern="should create user"

# Specific workspace
cd apps/api && npm test
```

**E2E Tests**:

```bash
npm run test:e2e              # Run on local
npm run test:e2e:ui           # Interactive UI mode
npm run test:e2e:staging      # Test staging environment
npm run test:e2e:production   # Test production (use carefully)
```

### Coverage Reports

```bash
npm run test:coverage

# Opens HTML report in browser
open coverage/index.html
```

**Coverage Thresholds**:

```typescript
// vitest.config.ts
coverage: {
  provider: 'v8',
  reporter: ['text', 'json', 'html'],
  // [To be defined by CTO]: Coverage thresholds
  // thresholds: {
  //   lines: 80,
  //   functions: 80,
  //   branches: 75,
  //   statements: 80,
  // },
}
```

### Test Database Setup

**Testcontainers Helper** (`apps/api/src/tests/setup-db.ts`):

```typescript
import { PostgreSqlContainer } from "@testcontainers/postgresql";
import { drizzle } from "drizzle-orm/node-postgres";
import { migrate } from "drizzle-orm/node-postgres/migrator";
import { Pool } from "pg";
import express from "express";
import { createServer } from "@/server";

export async function setupAppWithDatabase() {
  // Start PostgreSQL container
  const container = await new PostgreSqlContainer()
    .withDatabase("test_db")
    .withUsername("test_user")
    .withPassword("test_password")
    .start();

  // Create connection pool
  const pool = new Pool({
    connectionString: container.getConnectionUri(),
  });

  // Run migrations
  const db = drizzle(pool);
  await migrate(db, { migrationsFolder: "./drizzle" });

  // Create Express app with test database
  const app = createServer();
  app.use((req, res, next) => {
    req.tx = db;
    next();
  });

  const teardownDatabase = async () => {
    await pool.end();
    await container.stop();
  };

  return { app, db, teardownDatabase };
}
```

## Historical Technical Decisions

### Decision: Vitest over Jest

**Date**: Project inception  
**Status**: Accepted

**Rationale**:

- Faster test execution
- Better TypeScript support
- Native ESM support
- Vite ecosystem integration
- Same API as Jest (easy migration)

### Decision: Testcontainers for Integration Tests

**Date**: Project inception  
**Status**: Accepted

**Rationale**:

- Real PostgreSQL instance (not mocked)
- Isolated test environment
- Automatic cleanup
- Same database version as production

**Trade-offs**:

- ⚠️ Requires Docker running
- ⚠️ Slower than mocked tests
- ✅ Higher confidence in database behavior

### Decision: Staging for Real Integration Testing

**Date**: Project inception (from CTO)  
**Status**: Accepted

**Rationale** (from CTO):

> "Je testerai avec l'Environment staging sur lequel je deploy depuis local"

**Approach**:

- Unit tests: Fully mocked
- Integration tests (Testcontainers): Real DB, mocked services
- E2E tests on staging: Real everything

**[To be documented by CTO]**:

- Test coverage targets
- Testing philosophy priorities
- CI/CD integration strategy

## Troubleshooting & FAQ

### Issue: Tests timeout

**Cause**: Database container slow to start or operations taking too long.

**Solution**:

```typescript
// Increase timeout
test('slow operation', async () => {
  // ...
}, { timeout: 30000 });  // 30 seconds

// Or in vitest.config.ts
hookTimeout: 30000,
```

---

### Issue: Testcontainers fails to start

**Cause**: Docker not running or insufficient resources.

**Solution**:

```bash
# Check Docker
docker ps

# Start Docker Desktop
open -a Docker

# Check Docker resources (at least 4GB RAM recommended)
```

---

### Issue: E2E tests flaky

**Cause**: Timing issues, network latency, or state pollution.

**Solution**:

```typescript
// Use proper waits
await page.waitForSelector('[data-testid="user-menu"]');

// Or wait for network idle
await page.goto("/dashboard", { waitUntil: "networkidle" });

// Reset state between tests
beforeEach(async () => {
  await setupClerkTestingToken({ page });
  // Clear any application state
});
```

---

### Issue: Mock not applied

**Cause**: Mock defined after import or wrong module path.

**Solution**:

```typescript
// Mock BEFORE imports
vi.mock("@clerk/express", () => ({
  clerkMiddleware: vi.fn(),
}));

// THEN import
import { clerkMiddleware } from "@clerk/express";
```

## References

- **Configuration**:
  - `apps/api/vitest.config.ts` - Backend test config
  - `apps/web/vitest.config.mts` - Frontend test config
  - `playwright.config.ts` - E2E test config
- **Test Files**:
  - `apps/api/src/**/*.test.ts` - Backend unit tests
  - `apps/api/src/tests/routes/*.test.ts` - Integration tests
  - `apps/web/src/**/*.test.tsx` - Frontend tests
  - `e2e-tests/*.spec.ts` - E2E tests
- **Setup**:
  - `apps/api/src/tests/setup.ts` - Global mocks
  - `apps/api/src/tests/setup-db.ts` - Testcontainers helper
  - `e2e-tests/global.setup.ts` - Playwright setup
- **External Docs**:
  - [Vitest](https://vitest.dev/)
  - [Playwright](https://playwright.dev/)
  - [Testcontainers](https://node.testcontainers.org/)
  - [React Testing Library](https://testing-library.com/react)
  - [Clerk Testing](https://clerk.com/docs/testing)
