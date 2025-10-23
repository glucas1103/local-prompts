# Authentication & Authorization

## Metadata

- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/middlewares/authMiddleware.ts`, `apps/web/src/middleware.ts`, `apps/api/src/routes/clerk.ts`

## Technical Overview

Authentication is managed by **Clerk** (v6.24.0 frontend, v1.7.5 backend), providing a complete authentication and user management solution. The system implements a dual-layer authentication strategy: Clerk handles session management and token validation, while custom middleware enriches requests with database-synchronized user and organization data.

The frontend uses Clerk's Next.js SDK with edge middleware for route protection, automatically redirecting unauthenticated users to sign-in pages. The backend validates Clerk session tokens via Express middleware, then fetches corresponding user and organization records from PostgreSQL, ensuring that all authenticated requests have access to complete user context.

Clerk webhooks keep the local database synchronized with authentication events (user creation, updates, organization changes), maintaining a single source of truth for user data while enabling efficient queries without external API calls on every request.

## Architecture & Design

### Complete Authentication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     User Authentication Flow                     │
└─────────────────────────────────────────────────────────────────┘

1. User Access Request
   ↓
┌──────────────────────┐
│  Next.js Middleware  │ → Check Clerk session
│  (Edge Runtime)      │
└──────────────────────┘
   ↓
   ├─ No Session → Redirect to /sign-in
   │                ↓
   │     ┌─────────────────────┐
   │     │  Clerk Sign-In UI   │
   │     │  (Hosted by Clerk)  │
   │     └─────────────────────┘
   │                ↓
   │     Session Created → Cookie Set
   │
   └─ Session Valid → Allow Access to Protected Route
                       ↓
              ┌─────────────────────┐
              │   Frontend Makes    │
              │   API Request       │
              │   (with Clerk JWT)  │
              └─────────────────────┘
                       ↓
              ┌─────────────────────┐
              │  Express Backend    │
              │  clerkMiddleware()  │ → Validates JWT
              └─────────────────────┘
                       ↓
              ┌─────────────────────┐
              │  authMiddleware()   │
              │  Fetches User from  │
              │  PostgreSQL         │
              └─────────────────────┘
                       ↓
              req.auth.user ✓
              req.auth.organization ✓
                       ↓
              Route Handler Executes
```

### Frontend Auth Flow (Detailed)

**Step 1: Route Protection**

```typescript
// apps/web/src/middleware.ts
const isPrivateRoute = createRouteMatcher(["/dashboard(.*)"]);

export default clerkMiddleware(async (auth, request) => {
  if (isPrivateRoute(request)) {
    await auth.protect(); // Throws if not authenticated
  }
});
```

**Step 2: Route Groups**

- `(public)/` - Accessible without authentication
  - `(auth)/sign-in/` - Sign-in page
  - `(auth)/sign-up/` - Sign-up page
  - `page.tsx` - Landing page
- `(protected)/dashboard/` - Requires authentication
  - All sub-routes automatically protected

**Step 3: Client-Side Auth State**

```typescript
// Example component using Clerk hooks
'use client';

import { useUser, useOrganization } from '@clerk/nextjs';

export function UserProfile() {
  const { user, isLoaded } = useUser();
  const { organization } = useOrganization();

  if (!isLoaded) return <LoadingSpinner />;
  if (!user) return <SignInPrompt />;

  return (
    <div>
      <h1>Welcome {user.firstName}</h1>
      <p>Organization: {organization?.name}</p>
    </div>
  );
}
```

### Backend Auth Flow (Detailed)

**Step 1: Clerk Token Validation**

```typescript
// apps/api/src/server.ts
app.use(clerkMiddleware()); // Validates JWT, populates req.auth
```

After this middleware:

```typescript
req.auth = {
  userId: 'user_xyz',        // Clerk user ID
  orgId: 'org_abc',          // Clerk organization ID
  sessionId: 'sess_123',     // Session identifier
  sessionClaims: { ... }     // JWT claims
}
```

**Step 2: Database User Enrichment**

```typescript
// apps/api/src/middlewares/authMiddleware.ts
export async function authMiddleware(req, res, next) {
  // Validate Clerk provided userId
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

  // Check soft delete
  if (currentUser.deleted) {
    return res.status(401).json({ error: "USER_DELETED" });
  }

  // Attach to request
  req.auth.user = currentUser;

  // Fetch organization
  const [organization] = await req.tx
    .select()
    .from(organizations)
    .where(eq(organizations.id, req.auth.orgId))
    .limit(1);

  if (!organization) {
    return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
  }

  // Check soft delete
  if (organization.deleted) {
    return res.status(404).json({ error: "ORGANIZATION_DELETED" });
  }

  req.auth.organization = organization;

  next();
}
```

**Step 3: Role-Based Authorization**

```typescript
// apps/api/src/middlewares/authMiddleware.ts
export function authIsAdminMiddleware(req, res, next) {
  if (!req.auth.user?.isAdmin) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}
```

### Webhook Synchronization

**Clerk → Database Sync** via webhooks:

```typescript
// apps/api/src/routes/clerk.ts
clerkRouter.post(
  "/webhook",
  express.raw({ type: "application/json" }),
  async (req, res) => {
    // Verify webhook signature
    const evt = await verifyWebhook(req, {
      signingSecret: process.env.CLERK_WEBHOOK_SIGNING_SECRET,
    });

    const { type, data } = evt;

    switch (type) {
      case "user.created":
        await req.tx.insert(users).values({
          id: data.id,
          email: data.email_addresses[0].email_address,
          firstName: data.first_name || "",
          lastName: data.last_name || "",
          photoUrl: data.image_url || generateSeedPhotoUrl(data.id),
          isAdmin: data.public_metadata.isAdmin === true,
        });
        break;

      case "user.updated":
        await req.tx
          .update(users)
          .set({
            email: data.email_addresses[0].email_address,
            firstName: data.first_name || "",
            lastName: data.last_name || "",
            photoUrl: data.image_url,
            isAdmin: data.public_metadata.isAdmin === true,
            updatedAt: new Date(),
          })
          .where(eq(users.id, data.id));
        break;

      case "user.deleted":
        await req.tx
          .update(users)
          .set({ deleted: true, deletedAt: new Date() })
          .where(eq(users.id, data.id));
        break;

      case "organization.created":
        await req.tx.insert(organizations).values({
          id: data.id,
          name: data.name,
          plan: "FREE", // Default plan
        });
        break;

      case "organizationMembership.created":
        await req.tx.insert(organizationMembers).values({
          id: data.id,
          organizationId: data.organization.id,
          userId: data.public_user_data.user_id,
          role: data.role,
        });
        break;
    }

    res.json({ success: true });
  }
);
```

### Authentication States

**Frontend States**:

1. **Loading** - Clerk SDK initializing
2. **Unauthenticated** - No session, show sign-in
3. **Authenticated** - Valid session, show protected content
4. **Error** - Authentication error (token expired, etc.)

**Backend States**:

1. **No Token** - 401 Unauthorized
2. **Invalid Token** - 401 Unauthorized (Clerk validation failed)
3. **User Not in DB** - 401 User Not Found (webhook sync issue)
4. **User Deleted** - 401 User Deleted
5. **Authorized** - Request proceeds with full context

## Configuration & Setup

### Environment Variables

**Frontend** (`.env` or `.env.local`):

```bash
# Clerk Configuration
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx

# Clerk URLs
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FORCE_REDIRECT_URL=/dashboard
NEXT_PUBLIC_CLERK_SIGN_UP_FORCE_REDIRECT_URL=/dashboard
```

**Backend** (`apps/api/.env`):

```bash
# Clerk Configuration
CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx
CLERK_WEBHOOK_SIGNING_SECRET=whsec_xxx

# Frontend URL (for CORS)
FRONTEND_BASE_URL=http://localhost:3000
```

### Frontend Setup

**1. Clerk Provider** (`apps/web/src/app/layout.tsx`):

```typescript
import { ClerkProvider } from '@clerk/nextjs';

export default function RootLayout({ children }: { children: React.Node }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

**2. Middleware Configuration** (`apps/web/src/middleware.ts`):

```typescript
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

// Define protected routes
const isPrivateRoute = createRouteMatcher(["/dashboard(.*)"]);

export default clerkMiddleware(async (auth, request) => {
  // Protect dashboard and all sub-routes
  if (isPrivateRoute(request)) {
    await auth.protect(); // Redirects to sign-in if not authenticated
  }
});

export const config = {
  matcher: [
    // Skip Next.js internals and static files
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    // Always run for API routes
    "/(api|trpc)(.*)",
  ],
};
```

**3. Sign-In/Sign-Up Pages**:

```typescript
// apps/web/src/app/(public)/(auth)/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from '@clerk/nextjs';

export default function SignInPage() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <SignIn />
    </div>
  );
}
```

### Backend Setup

**1. Server Configuration** (`apps/api/src/server.ts`):

```typescript
import { clerkMiddleware } from "@clerk/express";

export const createServer = () => {
  const app = express();

  app
    .use(helmet())
    .use(cors({ origin: process.env.FRONTEND_BASE_URL }))
    .use(clerkMiddleware()) // ← Clerk validation
    .use(loggerMiddleware)
    .use(withTransaction())
    .use("/api", apiRoutes); // Routes use auth after this point

  return app;
};
```

**2. Auth Middleware** (`apps/api/src/middlewares/authMiddleware.ts`):

```typescript
import { users, organizations } from "@/db/schema";
import { eq } from "drizzle-orm";

export async function authMiddleware(req, res, next) {
  // Validate Clerk authentication
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

  if (currentUser.deleted) {
    return res.status(401).json({ error: "USER_DELETED" });
  }

  req.auth.user = currentUser;

  // Fetch organization
  const [organization] = await req.tx
    .select()
    .from(organizations)
    .where(eq(organizations.id, req.auth.orgId))
    .limit(1);

  if (!organization) {
    return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
  }

  if (organization.deleted) {
    return res.status(404).json({ error: "ORGANIZATION_DELETED" });
  }

  req.auth.organization = organization;

  next();
}

export function authIsAdminMiddleware(req, res, next) {
  if (!req.auth.user?.isAdmin) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}
```

**3. Webhook Endpoint** (`apps/api/src/routes/clerk.ts`):

```typescript
import { Router } from "express";
import express from "express";
import { verifyWebhook } from "@clerk/express/webhooks";

export const clerkRouter = Router();

clerkRouter.post(
  "/webhook",
  express.raw({ type: "application/json" }), // Raw body for signature verification
  async (req, res) => {
    try {
      const evt = await verifyWebhook(req, {
        signingSecret: process.env.CLERK_WEBHOOK_SIGNING_SECRET,
      });

      // Handle events (see Webhook Synchronization section above)
      // ...

      res.json({ success: true });
    } catch (error) {
      logger.error({
        msg: "Webhook verification failed",
        event: "clerk-webhook",
        metadata: { error },
      });
      res.status(400).json({ error: "INVALID_WEBHOOK" });
    }
  }
);
```

### Clerk Dashboard Setup

1. Create Clerk application at [clerk.com](https://clerk.com)
2. Configure **Redirect URLs**:
   - Sign-in: `/dashboard`
   - Sign-up: `/dashboard`
   - Sign-out: `/`
3. Enable **Organizations** feature
4. Configure **Webhooks**:
   - Endpoint: `https://your-api.com/api/clerk/webhook`
   - Events: `user.*`, `organization.*`, `organizationMembership.*`
   - Copy signing secret to `CLERK_WEBHOOK_SIGNING_SECRET`
5. Set **Public Metadata** schema (optional):
   ```json
   {
     "isAdmin": "boolean"
   }
   ```

````

## Patterns & Best Practices

### ✅ DO: Protect Routes with Middleware

```typescript
// Apply authentication to route groups
apiRoutes.use('/admin', authMiddleware, authIsAdminMiddleware, adminRouter);
apiRoutes.use('/user', authMiddleware, userRoutes);
apiRoutes.use('/organization', authMiddleware, organizationRoutes);
apiRoutes.use('/github', authMiddleware, githubRouter);

// Public routes (no auth required)
apiRoutes.use('/clerk', clerkRouter);  // Webhooks
````

### ✅ DO: Use Admin Middleware for Privileged Routes

```typescript
export function authIsAdminMiddleware(req, res, next) {
  if (!req.auth.user?.isAdmin) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}

// Usage
router.delete(
  "/users/:id",
  authMiddleware,
  authIsAdminMiddleware,
  deleteUserHandler
);
```

### ✅ DO: Check Organization Membership

```typescript
// Verify user belongs to organization
router.get("/organization/data", authMiddleware, async (req, res) => {
  const { organization, user } = req.auth;

  // Verify membership
  const [membership] = await req.tx
    .select()
    .from(organizationMembers)
    .where(
      and(
        eq(organizationMembers.organizationId, organization.id),
        eq(organizationMembers.userId, user.id)
      )
    )
    .limit(1);

  if (!membership) {
    return res.status(403).json({ error: "NOT_ORGANIZATION_MEMBER" });
  }

  // Proceed with authorized action
  res.json({ data: "..." });
});
```

### ✅ DO: Handle Soft Deleted Users

```typescript
// Always check deleted flag after fetching from database
if (currentUser.deleted) {
  return res.status(401).json({ error: "USER_DELETED" });
}

if (organization.deleted) {
  return res.status(404).json({ error: "ORGANIZATION_DELETED" });
}
```

### ✅ DO: Use Clerk Hooks on Frontend

```typescript
import { useUser, useOrganization, useAuth } from '@clerk/nextjs';

export function ProtectedComponent() {
  const { isLoaded, isSignedIn, user } = useUser();
  const { organization } = useOrganization();
  const { signOut } = useAuth();

  if (!isLoaded) return <LoadingSpinner />;
  if (!isSignedIn) return <SignInPrompt />;

  return (
    <div>
      <h1>Welcome {user.firstName}</h1>
      <p>Organization: {organization?.name}</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  );
}
```

### ❌ DON'T: Skip Authentication Middleware

```typescript
// BAD: No authentication
router.get("/sensitive-data", async (req, res) => {
  const data = await getSensitiveData();
  res.json(data);
});

// GOOD: Authenticated route
router.get("/sensitive-data", authMiddleware, async (req, res) => {
  const data = await getSensitiveData(req.auth.userId);
  res.json(data);
});
```

### ❌ DON'T: Trust Frontend Data for Authorization

```typescript
// BAD: Trust client-provided userId
router.get("/user/:userId/data", authMiddleware, async (req, res) => {
  const { userId } = req.params;
  const data = await getUserData(userId); // ❌ Can access any user's data!
  res.json(data);
});

// GOOD: Use authenticated user from req.auth
router.get("/user/data", authMiddleware, async (req, res) => {
  const { userId } = req.auth; // ✅ From validated session
  const data = await getUserData(userId);
  res.json(data);
});
```

### ❌ DON'T: Store Sensitive Data in Clerk Metadata

```typescript
// BAD: Storing sensitive data in Clerk
await clerkClient.users.updateUser(userId, {
  publicMetadata: {
    creditCard: "1234-5678-9012-3456", // ❌ NEVER!
  },
});

// GOOD: Store in database, reference by ID
const user = await req.tx.insert(users).values({
  id: userId,
  stripeCustomerId: "cus_xxx", // ✅ Reference, not sensitive data
});
```

## Usage Examples

### Complete Protected Route Example

```typescript
// apps/api/src/routes/users.ts
import { Router } from "express";
import { eq } from "drizzle-orm";
import { users } from "@/db/schema";
import { authMiddleware } from "@/middlewares/authMiddleware";
import { validateResponse } from "@/middlewares/validationMiddleware";
import { userSchema } from "@boilerplate-turbo/types";

export const userRoutes = Router();

// Get current user
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
      logger.error({
        msg: "Error fetching user",
        event: "get-user",
        metadata: { error, userId },
      });
      res.status(500).json({ error: "FAILED_TO_GET_USER" });
    }
  }
);
```

### Frontend Protected Component

```typescript
// apps/web/src/components/dashboard/user-profile.tsx
'use client';

import { useUser } from '@clerk/nextjs';
import { useQuery } from '@tanstack/react-query';

export function UserProfile() {
  const { isLoaded, isSignedIn, user } = useUser();

  // Fetch additional user data from API
  const { data: userData, isLoading } = useQuery({
    queryKey: ['user', user?.id],
    queryFn: () => fetch('/api/user').then(r => r.json()),
    enabled: isSignedIn,
  });

  if (!isLoaded || isLoading) {
    return <div>Loading...</div>;
  }

  if (!isSignedIn) {
    return <div>Please sign in</div>;
  }

  return (
    <div>
      <img src={user.imageUrl} alt={user.fullName} />
      <h1>{user.fullName}</h1>
      <p>{user.primaryEmailAddress?.emailAddress}</p>
      {userData?.isAdmin && <span className="badge">Admin</span>}
    </div>
  );
}
```

### Testing with Mock Auth

```typescript
// apps/api/src/tests/routes/users.test.ts
import { mockAuthMiddleware } from "@/middlewares/authMiddleware";
import request from "supertest";
import { createServer } from "@/server";

describe("User Routes", () => {
  let app;

  beforeEach(() => {
    app = createServer();
    // Replace real auth with mock
    app.use(mockAuthMiddleware());
  });

  it("should get current user", async () => {
    const response = await request(app).get("/api/user").expect(200);

    expect(response.body).toHaveProperty("id");
    expect(response.body).toHaveProperty("email");
  });

  it("should require admin role for admin routes", async () => {
    app.use(mockAuthMiddleware(false)); // Non-admin

    const response = await request(app).get("/api/admin/settings").expect(401);

    expect(response.body.error).toBe("UNAUTHORIZED - ADMIN ONLY");
  });
});
```

## Technical Dependencies

**Frontend**:

- `@clerk/nextjs`: ^6.24.0 - Clerk Next.js SDK
- `@clerk/testing`: ^1.9.3 - Testing utilities

**Backend**:

- `@clerk/express`: ^1.7.5 - Clerk Express SDK
- `drizzle-orm`: ^0.41.0 - Database queries
- `pg`: ^8.16.3 - PostgreSQL client

**Shared**:

- `@boilerplate-turbo/types` - Type definitions

## Tests & Validation

### E2E Authentication Test

**File**: `e2e-tests/auth.spec.ts`

```typescript
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
      identifier: "test@example.com",
      password: "testpassword123",
    },
  });

  // Should redirect to dashboard
  await expect(page).toHaveURL("/dashboard");

  // Verify authenticated UI elements
  const userMenu = page.getByTestId("user-menu");
  await expect(userMenu).toBeVisible();

  // Sign out
  await userMenu.click();
  await page.getByTestId("sign-out-button").click();

  // Should redirect to sign-in
  await expect(page).toHaveURL("/sign-in");
});
```

### Unit Test for Auth Middleware

```typescript
import { authMiddleware } from "@/middlewares/authMiddleware";
import { createDbConnection } from "@/db/db";

describe("authMiddleware", () => {
  it("should reject requests without userId", async () => {
    const req = { auth: {} };
    const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };
    const next = jest.fn();

    await authMiddleware(req, res, next);

    expect(res.status).toHaveBeenCalledWith(401);
    expect(res.json).toHaveBeenCalledWith({
      error: "UNAUTHORIZED - USER ID NOT FOUND",
    });
    expect(next).not.toHaveBeenCalled();
  });

  it("should attach user and organization to request", async () => {
    const req = {
      auth: { userId: "user_123", orgId: "org_456" },
      tx: mockDb,
    };
    const res = {};
    const next = jest.fn();

    await authMiddleware(req, res, next);

    expect(req.auth.user).toBeDefined();
    expect(req.auth.organization).toBeDefined();
    expect(next).toHaveBeenCalled();
  });
});
```

## Troubleshooting & FAQ

### Issue: "USER_NOT_FOUND" after sign-in

**Symptoms**: User can sign in to Clerk but API returns 401 USER_NOT_FOUND.

**Causes**:

- Webhook not configured or not firing
- Webhook signature verification failing
- User created in Clerk but not synced to database

**Solutions**:

1. Check webhook configuration in Clerk dashboard
2. Verify `CLERK_WEBHOOK_SIGNING_SECRET` is correct
3. Check API logs for webhook errors
4. Manually trigger webhook from Clerk dashboard
5. Verify user exists in database: `SELECT * FROM users WHERE id = 'user_xxx'`

---

### Issue: Infinite redirect loop

**Symptoms**: Page keeps redirecting between `/dashboard` and `/sign-in`.

**Causes**:

- Middleware misconfiguration
- Cookie not being set/read
- CORS issues preventing cookie transmission

**Solutions**:

1. Check middleware matcher configuration
2. Verify `NEXT_PUBLIC_CLERK_*` environment variables
3. Ensure frontend and backend URLs are correct
4. Check browser console for CORS errors
5. Clear cookies and try again

---

### Issue: Webhook signature verification fails

**Symptoms**: Clerk webhooks return 400 error.

**Causes**:

- Incorrect signing secret
- Raw body not preserved (body parser interfering)
- Secret from wrong environment (test vs production)

**Solutions**:

```typescript
// Ensure raw body for webhook route
app.use("/api/clerk/webhook", express.raw({ type: "application/json" }));

// Then parse JSON for other routes
app.use(express.json());
```

---

### Issue: Organization not found

**Symptoms**: API returns ORGANIZATION_NOT_FOUND.

**Causes**:

- User not associated with organization in Clerk
- Organization not synced via webhook
- Soft deleted organization

**Solutions**:

1. Check user's organization membership in Clerk dashboard
2. Verify organization exists in database
3. Check `deleted` flag on organization
4. Re-sync organization via webhook

---

### Issue: Admin middleware not working

**Symptoms**: Admin routes accessible by non-admin users.

**Causes**:

- Middleware order incorrect
- `isAdmin` flag not set in Clerk metadata
- Database not synced with Clerk metadata

**Solutions**:

1. Verify middleware order: `authMiddleware` before `authIsAdminMiddleware`
2. Set admin flag in Clerk dashboard: `publicMetadata.isAdmin = true`
3. Trigger user.updated webhook to sync
4. Check database: `SELECT is_admin FROM users WHERE id = 'user_xxx'`

## Historical Technical Decisions

### Decision: Dual-Layer Authentication (Clerk + Database)

**Date**: Project inception  
**Status**: Accepted

**Context**:  
Need efficient authentication with rich user data access.

**Decision**:  
Use Clerk for session management, but maintain user/organization data in PostgreSQL.

**Rationale**:

- ✅ Fast database queries without external API calls
- ✅ Join user data with application data easily
- ✅ Full control over user data structure
- ✅ Offline development possible
- ⚠️ Additional complexity (webhook synchronization)
- ⚠️ Potential sync delays (eventual consistency)

**Alternatives Considered**:

1. Clerk only (fetch from Clerk API every request)
   - ❌ Slower (network latency)
   - ❌ External dependency on every request
   - ❌ Harder to join with application data
2. Custom auth (JWT + database)
   - ❌ More code to maintain
   - ❌ No hosted UI
   - ❌ Have to handle password reset, MFA, etc.

---

### Decision: Soft Deletes for Users and Organizations

**Date**: Project inception  
**Status**: Accepted

**Rationale**:

- Maintains referential integrity
- Audit trail for deleted accounts
- Possible to restore accounts
- Syncs with Clerk's soft delete behavior

**[To be documented by CTO]**: Specific compliance or business requirements.

## References

- **Main Files**:
  - `apps/api/src/middlewares/authMiddleware.ts` - Backend auth middleware
  - `apps/web/src/middleware.ts` - Frontend route protection
  - `apps/api/src/routes/clerk.ts` - Webhook handling
- **Tests**:
  - `e2e-tests/auth.spec.ts` - E2E authentication flow
  - `apps/api/src/tests/routes/*.test.ts` - Unit tests with mock auth
- **Configuration**:
  - `.env` - Environment variables
  - `apps/web/src/app/layout.tsx` - Clerk Provider
- **External Docs**:
  - [Clerk Documentation](https://clerk.com/docs)
  - [Clerk Next.js SDK](https://clerk.com/docs/references/nextjs)
  - [Clerk Express SDK](https://clerk.com/docs/references/express)
  - [Clerk Webhooks](https://clerk.com/docs/integrations/webhooks)
