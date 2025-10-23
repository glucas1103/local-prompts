# API Route Organization

## Metadata

- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/routes/**`, `/.cursorrules`

## Technical Overview

API routes follow REST conventions with named router exports, consistent error handling, and SCREAMING_SNAKE_CASE error codes.

## Patterns & Best Practices

### ✅ DO: Use Named Router Exports

```typescript
import { Router } from "express";

export const userRoutes = Router();

userRoutes.get("/", async (req, res) => {
  try {
    const result = await req.tx.select().from(users);
    res.json(result);
  } catch (error) {
    logger.error({
      msg: "Error fetching users",
      event: "get-users",
      metadata: { error },
    });
    res.status(500).json({ error: "FAILED_TO_GET_USERS" });
  }
});
```

### ✅ DO: Use SCREAMING_SNAKE_CASE for Error Codes

```typescript
res.status(404).json({ error: "USER_NOT_FOUND" });
res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
res.status(500).json({ error: "FAILED_TO_CREATE_ORGANIZATION" });
```

### ✅ DO: Always Use `req.tx` for Database Operations

```typescript
// GOOD
const [user] = await req.tx
  .select()
  .from(users)
  .where(eq(users.id, userId))
  .limit(1);
```

### ✅ DO: Apply Middleware in Correct Order

```typescript
router.post(
  "/",
  validateRequestBody(schema), // 1. Validate input
  validateResponse(schema), // 2. Validate output
  authMiddleware, // 3. Authenticate
  handler // 4. Handle request
);
```

## Route Organization

```
apps/api/src/routes/
├── index.ts          # Main router (applies auth middleware)
├── users.ts          # User endpoints
├── organization.ts   # Organization endpoints
├── github.ts         # GitHub endpoints
├── clerk.ts          # Clerk webhooks (public)
└── admin.ts          # Admin endpoints (double auth)
```

**Route Grouping** (`apps/api/src/routes/index.ts`):

```typescript
export const apiRoutes = express.Router();

// Admin (auth + admin middleware)
apiRoutes.use("/admin", authMiddleware, authIsAdminMiddleware, adminRouter);

// Protected routes (auth middleware only)
apiRoutes.use("/organization", authMiddleware, organizationRoutes);
apiRoutes.use("/user", authMiddleware, userRoutes);
apiRoutes.use("/github", authMiddleware, githubRouter);

// Public routes (no auth)
apiRoutes.use("/clerk", clerkRouter);
```

## Usage Examples

### Complete CRUD Route

```typescript
export const organizationRoutes = Router();

// GET /api/organization
organizationRoutes.get(
  "/",
  validateResponse(organizationSchema),
  async (req, res) => {
    res.json(req.auth.organization);
  }
);

// GET /api/organization/members
organizationRoutes.get(
  "/members",
  validateResponse(organizationMemberSchema.array()),
  async (req, res) => {
    const members = await req.tx
      .select()
      .from(organizationMembers)
      .leftJoin(users, eq(organizationMembers.userId, users.id))
      .where(eq(organizationMembers.organizationId, req.auth.orgId));

    res.json(members);
  }
);
```

## References

- **Routes**: `apps/api/src/routes/**`
- **Guidelines**: `/.cursorrules` (Router Rules)
- **See Also**: `patterns/MIDDLEWARE.md`, `context/API.md`
