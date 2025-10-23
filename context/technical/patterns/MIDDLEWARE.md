# Middleware Patterns

## Metadata

- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/middlewares/*.ts`

## Technical Overview

The API uses Express middleware to handle cross-cutting concerns like authentication, logging, transaction management, and validation. Middleware functions are composed in a specific order to ensure proper request processing and error handling.

The middleware pipeline follows a layered approach:

1. **Infrastructure Layer**: Helmet, CORS, body parsing
2. **Observability Layer**: Logging (Pino HTTP)
3. **Authentication Layer**: Clerk middleware
4. **Transaction Layer**: Database transaction management
5. **Business Logic Layer**: Route-specific validation and authorization

## Architecture & Design

### Middleware Pipeline Order

From `apps/api/src/server.ts`:

```typescript
app
  .disable('x-powered-by')                    // Security
  .use(helmet())                              // Security headers
  .use(urlencoded({ extended: true }))        // Body parsing
  .use('/api/clerk/webhook', express.raw())   // Raw body for webhooks
  .use(json())                                // JSON body parsing
  .use(cors({ ... }))                         // CORS configuration
  .use(clerkMiddleware())                     // Authentication
  .use(loggerMiddleware)                      // Request/response logging
  .use(withTransaction())                     // Transaction management
  .use('/api', apiRoutes)                     // Application routes
```

**Critical**: The order matters. Authentication must come before logging sensitive user data, and transactions must be available before routes execute.

### Transaction Middleware (`req.tx`)

The transaction middleware automatically manages database transactions based on HTTP method:

**Write Operations** (POST, PUT, PATCH, DELETE):

- BEGIN transaction before route handler
- COMMIT on success (status 2xx)
- ROLLBACK on error (status ≥ 300)
- Always release connection

**Read Operations** (GET, HEAD, OPTIONS):

- No transaction started
- Direct database access via `req.tx`
- Connection released after response

**Key Implementation** (`transactionMiddleware.ts`):

```typescript
export function withTransaction(poolOverride?: Pool) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const pool = poolOverride || defaultPool;
    const client = await pool.connect();

    try {
      // Start transaction for write operations
      if (["POST", "PUT", "PATCH", "DELETE"].includes(req.method)) {
        await client.query("BEGIN");
      }

      // Attach Drizzle instance to request
      req.tx = drizzle(client);

      // Override res.end to handle commit/rollback
      const originalEnd = res.end;
      res.end = function (...args) {
        Promise.resolve().then(async () => {
          try {
            if (["POST", "PUT", "PATCH", "DELETE"].includes(req.method)) {
              if (res.statusCode >= 200 && res.statusCode < 300) {
                await client.query("COMMIT");
              } else {
                await client.query("ROLLBACK");
              }
            }
          } finally {
            client.release();
          }
        });
        return originalEnd.call(this, ...args);
      };

      next();
    } catch (error) {
      if (["POST", "PUT", "PATCH", "DELETE"].includes(req.method)) {
        await client.query("ROLLBACK");
      }
      client.release();
      res.status(500).json({ error: "DATABASE_ERROR" });
    }
  };
}
```

## Configuration & Setup

### Logger Middleware

**Configuration** (`loggerMiddleware.ts`):

```typescript
export const loggerMiddleware = pinoHttp({
  logger: baseLogger,
  autoLogging: {
    ignore: (req) => req.method === "OPTIONS", // Skip OPTIONS requests
  },
  serializers: {
    req(req) {
      return {
        id: req.id,
        method: req.method,
        url: req.url,
        body: req.method === "POST" ? req.raw.body : undefined,
        query: req.query,
        params: req.params,
        headers: req.headers,
      };
    },
  },
  redact: [
    "req.headers.authorization", // Redact sensitive data
    "req.headers.cookie",
    "*.password",
    "*.token",
    "*.secret",
  ],
  quietReqLogger: isDevelopment, // Less verbose in dev
  quietResLogger: isDevelopment,
});
```

### Auth Middleware

**Purpose**: Validate Clerk authentication and fetch user/organization from database.

**Implementation** (`authMiddleware.ts`):

```typescript
export async function authMiddleware(
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) {
  // Check Clerk authentication
  if (!req.auth.userId) {
    return res.status(401).json({ error: "UNAUTHORIZED - USER ID NOT FOUND" });
  }
  if (!req.auth.orgId) {
    return res.status(401).json({ error: "UNAUTHORIZED - ORG ID NOT FOUND" });
  }

  // Fetch user from database
  const [currentUser] = await req.tx
    .select()
    .from(users)
    .where(eq(users.id, req.auth.userId))
    .limit(1);

  if (!currentUser) {
    return res.status(401).json({ error: "USER_NOT_FOUND" });
  }

  req.auth.user = currentUser; // Attach to request

  // Fetch organization from database
  const [organization] = await req.tx
    .select()
    .from(organizations)
    .where(eq(organizations.id, req.auth.orgId))
    .limit(1);

  if (!organization) {
    return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
  }

  req.auth.organization = organization; // Attach to request

  next();
}
```

**Admin Middleware**:

```typescript
export function authIsAdminMiddleware(
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) {
  if (!req.auth.user?.isAdmin) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}
```

### Validation Middleware

**Three types of validation**:

1. **Request Body**: `validateRequestBody(schema)`
2. **Query Parameters**: `validateQueryParams(schema)`
3. **Response**: `validateResponse(schema)`

**Implementation** (`validationMiddleware.ts`):

```typescript
export function validateRequestBody<T extends z.ZodType>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = schema.parse(req.body); // Validate and transform
      next();
    } catch (error) {
      if (isZodError(error)) {
        const errorMessages = formatZodError(error);
        logger.error({
          msg: "Request validation error",
          event: "request-validation",
          metadata: { errorMessages },
        });
        res.status(400).json({
          error: "REQUEST_VALIDATION_ERROR",
          details: errorMessages,
        });
      } else {
        res.status(500).json({ error: "Internal Server Error" });
      }
    }
  };
}
```

**Response Validation**:

- Only validates on success (status < 300)
- Prevents invalid data from reaching clients
- Catches schema drift between backend and shared types

## Patterns & Best Practices

### ✅ DO: Use Middleware Composition

```typescript
// Route with multiple middleware
userRoutes.get(
  "/",
  validateResponse(userSchema), // Validate response
  authMiddleware, // Require authentication
  async (req, res) => {
    // Handler logic
  }
);

// Admin-only route
adminRoutes.post(
  "/settings",
  authMiddleware, // Authenticate first
  authIsAdminMiddleware, // Then check admin
  validateRequestBody(settingsSchema),
  async (req, res) => {
    // Handler logic
  }
);
```

### ✅ DO: Always Use `req.tx` for Database Access

```typescript
router.post("/resource", async (req, res) => {
  try {
    // req.tx provides managed database access
    const result = await req.tx.insert(table).values(data);
    res.json(result);
  } catch (error) {
    logger.error({
      msg: "Operation failed",
      event: "create-resource",
      metadata: { error },
    });
    res.status(500).json({ message: "FAILED_TO_CREATE_RESOURCE" });
  }
});
```

### ❌ DON'T: Manually Manage Transactions

```typescript
// BAD: Don't use raw pool or client
router.post("/resource", async (req, res) => {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    const result = await client.query("INSERT INTO...");
    await client.query("COMMIT");
    res.json(result);
  } catch (error) {
    await client.query("ROLLBACK");
  } finally {
    client.release();
  }
});
```

### Middleware Testing

**Mock Auth** (`authMiddleware.ts`):

```typescript
export function mockAuthMiddleware(isAdmin: boolean = false) {
  return (req, res, next) => {
    req.auth = {
      userId: "test-user-id",
      user: createUserFixture(organization.id),
      orgId: "test-org-id",
      organization: createOrganisationFixture(),
      sessionId: "test-session-id",
      sessionClaims: { metadata: { isAdmin } },
    };
    next();
  };
}
```

## Usage Examples

### Standard Protected Route

```typescript
export const userRoutes = Router();

userRoutes.get(
  "/",
  validateResponse(userSchema),
  authMiddleware,
  async (req, res) => {
    const { userId } = req.auth;
    try {
      const [user] = await req.tx
        .select()
        .from(users)
        .where(eq(users.id, userId))
        .limit(1);

      if (!user) {
        return res.status(404).json({ error: "USER_NOT_FOUND" });
      }

      res.json(user);
    } catch (error) {
      captureExceptionWithContext({
        req,
        error,
        msg: "Error fetching user",
        event: "get-user",
        metadata: { userId },
      });
      res.status(500).json({ error: "FAILED_TO_GET_USER" });
    }
  }
);
```

### Write Operation with Validation

```typescript
router.post(
  "/organizations",
  validateRequestBody(createOrgSchema),
  validateResponse(organizationSchema),
  authMiddleware,
  async (req, res) => {
    try {
      // Transaction automatically started (POST method)
      const [org] = await req.tx
        .insert(organizations)
        .values(req.body)
        .returning();

      // Transaction auto-committed on success (status 200)
      res.status(201).json(org);
    } catch (error) {
      // Transaction auto-rolled back on error
      logger.error({
        msg: "Failed to create org",
        event: "create-org",
        metadata: { error },
      });
      res.status(500).json({ error: "CREATE_ORG_FAILED" });
    }
  }
);
```

### Query Parameter Validation

```typescript
const listQuerySchema = z.object({
  page: z.string().transform(Number).pipe(z.number().min(1)).default("1"),
  limit: z
    .string()
    .transform(Number)
    .pipe(z.number().min(1).max(100))
    .default("20"),
});

router.get(
  "/items",
  validateQueryParams(listQuerySchema),
  authMiddleware,
  async (req, res) => {
    const { page, limit } = req.query; // Already validated and transformed
    // Use page and limit...
  }
);
```

## Technical Dependencies

**Middleware Dependencies**:

- `express`: ^4.21.2
- `@clerk/express`: ^1.7.5 (authentication)
- `pino-http`: ^10.5.0 (logging)
- `helmet`: ^8.1.0 (security)
- `cors`: ^2.8.5 (CORS)
- `drizzle-orm`: ^0.41.0 (database)
- `pg`: ^8.16.3 (PostgreSQL client)
- `zod`: ^3.25.67 (validation)

**Internal Dependencies**:

- `@/db/db`: Database connection
- `@/utils/logger`: Structured logging
- `@/utils/format-zod-error`: Error formatting
- `@/utils/sentry-utils`: Error tracking

## Tests & Validation

**Testing Approach**:

- Mock middleware in tests using provided `mockAuthMiddleware`
- Integration tests use Testcontainers for real database
- E2E tests use Clerk testing SDK

**Test Example**:

```typescript
import { mockAuthMiddleware } from "@/middlewares/authMiddleware";

describe("User Routes", () => {
  it("should fetch current user", async () => {
    app.use(mockAuthMiddleware()); // Mock authentication

    const response = await request(app).get("/api/users");

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty("id");
  });
});
```

## Troubleshooting & FAQ

### Common Issues

**Issue**: `req.tx is undefined`

- **Cause**: Transaction middleware not applied or ordering issue
- **Solution**: Ensure `withTransaction()` is applied before routes

**Issue**: Transaction not rolling back on error

- **Cause**: Response status < 300 even on error
- **Solution**: Ensure proper status codes (≥ 300) for errors

**Issue**: Validation error but response still succeeds

- **Cause**: Validation middleware not applied or wrong order
- **Solution**: Apply validation middleware before route handler

**Issue**: Authentication passes but user not found

- **Cause**: User exists in Clerk but not in database
- **Solution**: Check webhook synchronization from Clerk

## References

- **Main Files**:
  - `apps/api/src/server.ts` - Middleware pipeline setup
  - `apps/api/src/middlewares/authMiddleware.ts` - Authentication
  - `apps/api/src/middlewares/transactionMiddleware.ts` - Transactions
  - `apps/api/src/middlewares/loggerMiddleware.ts` - Logging
  - `apps/api/src/middlewares/validationMiddleware.ts` - Validation

- **Tests**:
  - `apps/api/src/tests/routes/*.test.ts` - Integration tests

- **Configuration**:
  - `.cursorrules` - Middleware patterns and best practices

- **External Docs**:
  - [Express Middleware](https://expressjs.com/en/guide/using-middleware.html)
  - [Pino HTTP](https://github.com/pinojs/pino-http)
  - [Clerk Express SDK](https://clerk.com/docs/references/express)
  - [Zod](https://zod.dev/)
