# API Architecture

## Metadata

- **Category**: Architecture
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/server.ts`, `apps/api/src/routes/*.ts`

## Technical Overview

The backend API is built with Express.js following a layered architecture with clear separation between HTTP handling, business logic, and data access. The API uses a middleware pipeline to handle cross-cutting concerns like authentication, logging, and transaction management automatically.

All routes follow REST conventions with consistent error handling, validation, and response formats. The architecture emphasizes type safety through TypeScript and Zod validation, ensuring API contracts are enforced at runtime.

Database access is managed through a transaction middleware that automatically handles BEGIN/COMMIT/ROLLBACK based on HTTP methods, eliminating manual transaction management and reducing error-prone boilerplate code.

## Architecture & Design

### Layered Architecture

```
HTTP Request
     ↓
┌─────────────────────────────────────┐
│   Middleware Pipeline               │
│   - Helmet (security)               │
│   - CORS                            │
│   - Clerk Auth                      │
│   - Logger                          │
│   - Transaction (req.tx)            │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│   Routes Layer                      │
│   - Route definitions               │
│   - Request validation (Zod)        │
│   - Response validation (Zod)       │
│   - Error handling                  │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│   Services Layer (optional)         │
│   - Business logic                  │
│   - External API calls              │
│   - Complex operations              │
└─────────────────────────────────────┘
     ↓
┌─────────────────────────────────────┐
│   Database Layer                    │
│   - Drizzle ORM queries             │
│   - Schema definitions              │
│   - Relations                       │
└─────────────────────────────────────┘
     ↓
PostgreSQL Database
```

### Server Configuration

From `apps/api/src/server.ts`:

```typescript
export const createServer = () => {
  const app = express();

  app
    .disable("x-powered-by") // Security
    .use(helmet()) // Security headers
    .use(urlencoded({ extended: true })) // Form parsing
    .use("/api/clerk/webhook", express.raw()) // Raw body for webhooks
    .use(json()) // JSON parsing
    .use(
      cors({
        origin: process.env.FRONTEND_BASE_URL,
        credentials: true,
      })
    )
    .use(clerkMiddleware()) // Authentication
    .use(loggerMiddleware) // Logging
    .use(withTransaction()) // Transaction management
    .use("/api", apiRoutes) // Application routes
    .get("/status", statusHandler); // Health check

  Sentry.setupExpressErrorHandler(app); // Error tracking

  app.use(onError); // Global error handler

  return app;
};
```

### Route Organization

All routes are organized under `/api` prefix:

```
/api
├── /admin           # Admin-only routes (protected)
├── /user            # User profile routes (protected)
├── /organization    # Organization management (protected)
├── /github          # GitHub integration (protected)
└── /clerk           # Clerk webhooks (public)
```

**Route Index** (`apps/api/src/routes/index.ts`):

```typescript
export const apiRoutes = express.Router();

// Admin Routes (double protection)
apiRoutes.use("/admin", authMiddleware, authIsAdminMiddleware, adminRouter);

// Authenticated Routes
apiRoutes.use("/organization", authMiddleware, organizationRoutes);
apiRoutes.use("/user", authMiddleware, userRoutes);
apiRoutes.use("/github", authMiddleware, githubRouter);

// Public Routes
apiRoutes.use("/clerk", clerkRouter);
```

## Configuration & Setup

### Environment Variables

**Required**:

```bash
# Server
API_PORT=3001
NODE_ENV=development|production
FRONTEND_BASE_URL=http://localhost:3000

# Database
POSTGRES_HOST=localhost
POSTGRES_USER=user
POSTGRES_PASSWORD=password
POSTGRES_DB=db
POSTGRES_PORT=5432

# Authentication
CLERK_PUBLISHABLE_KEY=pk_xxx
CLERK_SECRET_KEY=sk_xxx
CLERK_WEBHOOK_SIGNING_SECRET=whsec_xxx

# GitHub Integration
GH_APP_ID=123456
GH_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"

# Monitoring
SENTRY_API_DSN=https://xxx@xxx.ingest.sentry.io/xxx
COMMIT_SHA=abc123 (for release tracking)
```

### Starting the Server

```typescript
// apps/api/src/index.ts
const server = createServer();
const PORT = process.env.API_PORT || 3001;

server.listen(PORT, () => {
  logger.info({
    msg: "Server started",
    event: "server-start",
    metadata: { port: PORT },
  });
});
```

## API & Interfaces

### REST Conventions

**HTTP Methods**:

- `GET` - Retrieve resources (no transaction)
- `POST` - Create resources (transaction)
- `PUT` - Replace resources (transaction)
- `PATCH` - Update resources (transaction)
- `DELETE` - Remove resources (transaction)

**Status Codes**:

- `200` - Success
- `201` - Created
- `400` - Bad Request (validation error)
- `401` - Unauthorized
- `404` - Not Found
- `500` - Internal Server Error

**Response Format**:

Success:

```json
{
  "id": "usr_123",
  "email": "user@example.com",
  "firstName": "John"
}
```

Error:

```json
{
  "error": "USER_NOT_FOUND"
}
```

Validation Error:

```json
{
  "error": "REQUEST_VALIDATION_ERROR",
  "details": [
    {
      "code": "invalid_type",
      "message": "Expected string, received number",
      "path": ["email"]
    }
  ]
}
```

### Request/Response Validation

**Validation Middleware Usage**:

```typescript
import {
  validateRequestBody,
  validateResponse,
} from "@/middlewares/validationMiddleware";
import { userSchema, createUserSchema } from "@boilerplate-turbo/types";

// Validate request body
router.post("/", validateRequestBody(createUserSchema), handler);

// Validate response
router.get("/", validateResponse(userSchema), handler);

// Validate both
router.put(
  "/:id",
  validateRequestBody(updateUserSchema),
  validateResponse(userSchema),
  handler
);
```

## Patterns & Best Practices

### ✅ DO: Use Named Router Exports

```typescript
import { Router } from "express";

export const userRoutes = Router();

userRoutes.get("/", async (req, res) => {
  // Handler logic
});
```

### ✅ DO: Always Use `req.tx` for Database Access

```typescript
userRoutes.get("/", authMiddleware, async (req, res) => {
  try {
    const [user] = await req.tx
      .select()
      .from(users)
      .where(eq(users.id, req.auth.userId))
      .limit(1);

    if (!user) {
      return res.status(404).json({ error: "USER_NOT_FOUND" });
    }

    res.json(user);
  } catch (error) {
    logger.error({
      msg: "Error fetching user",
      event: "get-user",
      metadata: { error },
    });
    res.status(500).json({ error: "FAILED_TO_GET_USER" });
  }
});
```

### ✅ DO: Use SCREAMING_SNAKE_CASE for Error Codes

```typescript
res.status(404).json({ error: "USER_NOT_FOUND" });
res.status(500).json({ error: "FAILED_TO_CREATE_ORGANIZATION" });
res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
```

### ✅ DO: Apply Middleware in Correct Order

```typescript
// Correct order
router.post(
  "/",
  validateRequestBody(schema), // 1. Validate input
  validateResponse(schema), // 2. Validate output
  authMiddleware, // 3. Authenticate
  authIsAdminMiddleware, // 4. Authorize
  handler // 5. Handle request
);
```

### ❌ DON'T: Manually Manage Transactions

```typescript
// BAD: Don't do this
const client = await pool.connect();
try {
  await client.query("BEGIN");
  // ... operations
  await client.query("COMMIT");
} catch (error) {
  await client.query("ROLLBACK");
} finally {
  client.release();
}

// GOOD: Use req.tx
const result = await req.tx.insert(table).values(data);
```

### ❌ DON'T: Use console.log

```typescript
// BAD
console.log("User created:", user);

// GOOD
logger.info({
  msg: "User created",
  event: "user-create",
  metadata: { userId: user.id },
});
```

## Usage Examples

### Complete CRUD Route Example

```typescript
import { Router } from "express";
import { eq } from "drizzle-orm";
import { organizations } from "@/db/schema";
import { authMiddleware } from "@/middlewares/authMiddleware";
import {
  validateRequestBody,
  validateResponse,
} from "@/middlewares/validationMiddleware";
import {
  organizationSchema,
  createOrganizationSchema,
} from "@boilerplate-turbo/types";
import { logger } from "@/utils/logger";

export const organizationRoutes = Router();

// GET /api/organization - Get current organization
organizationRoutes.get(
  "/",
  validateResponse(organizationSchema),
  authMiddleware,
  async (req, res) => {
    const { organization } = req.auth;
    res.json(organization);
  }
);

// POST /api/organization - Create organization
organizationRoutes.post(
  "/",
  validateRequestBody(createOrganizationSchema),
  validateResponse(organizationSchema),
  authMiddleware,
  async (req, res) => {
    try {
      const [org] = await req.tx
        .insert(organizations)
        .values(req.body)
        .returning();

      res.status(201).json(org);
    } catch (error) {
      logger.error({
        msg: "Failed to create organization",
        event: "create-organization",
        metadata: { error },
      });
      res.status(500).json({ error: "CREATE_ORG_FAILED" });
    }
  }
);

// PATCH /api/organization/:id - Update organization
organizationRoutes.patch(
  "/:id",
  validateRequestBody(updateOrganizationSchema),
  validateResponse(organizationSchema),
  authMiddleware,
  async (req, res) => {
    const { id } = req.params;
    try {
      const [org] = await req.tx
        .update(organizations)
        .set({ ...req.body, updatedAt: new Date() })
        .where(eq(organizations.id, id))
        .returning();

      if (!org) {
        return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
      }

      res.json(org);
    } catch (error) {
      logger.error({
        msg: "Failed to update organization",
        event: "update-organization",
        metadata: { error, organizationId: id },
      });
      res.status(500).json({ error: "UPDATE_ORG_FAILED" });
    }
  }
);

// DELETE /api/organization/:id - Soft delete organization
organizationRoutes.delete(
  "/:id",
  authMiddleware,
  authIsAdminMiddleware, // Admin only
  async (req, res) => {
    const { id } = req.params;
    try {
      await req.tx
        .update(organizations)
        .set({ deleted: true, deletedAt: new Date() })
        .where(eq(organizations.id, id));

      res.json({ success: true });
    } catch (error) {
      logger.error({
        msg: "Failed to delete organization",
        event: "delete-organization",
        metadata: { error, organizationId: id },
      });
      res.status(500).json({ error: "DELETE_ORG_FAILED" });
    }
  }
);
```

### Webhook Route Example

```typescript
// Special handling for webhooks (raw body)
clerkRouter.post(
  "/webhook",
  express.raw({ type: "application/json" }), // Raw body for signature verification
  async (req, res) => {
    try {
      const evt = await verifyWebhook(req, {
        signingSecret: process.env.CLERK_WEBHOOK_SIGNING_SECRET,
      });

      const { type, data } = evt;

      switch (type) {
        case "user.created":
          await req.tx.insert(users).values(extractUserInfo(data));
          break;
        case "user.updated":
          await req.tx
            .update(users)
            .set(extractUserInfo(data))
            .where(eq(users.id, data.id));
          break;
        // ... more cases
      }

      res.json({ success: true });
    } catch (error) {
      logger.error({
        msg: "Webhook processing failed",
        event: "clerk-webhook",
        metadata: { error },
      });
      res.status(400).json({ error: "INVALID_WEBHOOK" });
    }
  }
);
```

## Technical Dependencies

**Core Framework**:

- `express`: ^4.21.2 - Web framework
- `typescript`: ^5.7.3 - Type system

**Middleware**:

- `helmet`: ^8.1.0 - Security headers
- `cors`: ^2.8.5 - CORS handling
- `@clerk/express`: ^1.7.5 - Authentication
- `pino-http`: ^10.5.0 - Request logging

**Validation**:

- `zod`: ^3.25.67 - Schema validation

**Database**:

- `drizzle-orm`: ^0.41.0 - ORM
- `pg`: ^8.16.3 - PostgreSQL client

**Monitoring**:

- `@sentry/node`: ^9.22.0 - Error tracking

## Tests & Validation

**Test Example**:

```typescript
import request from "supertest";
import { createServer } from "@/server";
import { mockAuthMiddleware } from "@/middlewares/authMiddleware";

describe("User API", () => {
  let app;

  beforeEach(() => {
    app = createServer();
    app.use(mockAuthMiddleware());
  });

  it("should get current user", async () => {
    const response = await request(app).get("/api/user").expect(200);

    expect(response.body).toHaveProperty("id");
    expect(response.body).toHaveProperty("email");
  });

  it("should return 404 for non-existent user", async () => {
    const response = await request(app)
      .get("/api/user/non-existent")
      .expect(404);

    expect(response.body.error).toBe("USER_NOT_FOUND");
  });
});
```

## Troubleshooting & FAQ

**Issue**: CORS errors in development

- **Cause**: `FRONTEND_BASE_URL` not set correctly
- **Solution**: Set to `http://localhost:3000` in `.env`

**Issue**: Transactions not rolling back

- **Cause**: Error status < 300
- **Solution**: Use status codes ≥ 300 for errors

**Issue**: Webhook signature verification fails

- **Cause**: Incorrect webhook secret or raw body not parsed
- **Solution**: Ensure `express.raw()` middleware before webhook route

**[To be documented by CTO]**

- API versioning strategy
- Rate limiting configuration
- Caching strategy
- Performance benchmarks

## References

- **Main Files**:
  - `apps/api/src/server.ts` - Server setup
  - `apps/api/src/routes/index.ts` - Route organization
  - `apps/api/src/routes/*.ts` - Individual route handlers
- **Tests**:
  - `apps/api/src/server.test.ts` - Server tests
  - `apps/api/src/tests/routes/*.test.ts` - Route tests

- **Configuration**:
  - `.cursorrules` - Router rules and patterns

- **External Docs**:
  - [Express.js](https://expressjs.com/)
  - [Zod](https://zod.dev/)
  - [Clerk Express SDK](https://clerk.com/docs/references/express)
