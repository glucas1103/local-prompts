# Clerk Authentication

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/routes/clerk.ts`, `apps/web/src/middleware.ts`

## Technical Overview

**Clerk** manages authentication on both frontend (@clerk/nextjs 6.24.0) and backend (@clerk/express 1.7.5).

## Frontend Integration

```typescript
// apps/web/src/middleware.ts
const isPrivateRoute = createRouteMatcher(['/dashboard(.*)']);

export default clerkMiddleware(async (auth, request) => {
  if (isPrivateRoute(request)) {
    await auth.protect();
  }
});
```

## Backend Integration

```typescript
// apps/api/src/server.ts
app.use(clerkMiddleware())
   .use(authMiddleware);  // Fetches user from database
```

## Webhook Handling

**File**: `apps/api/src/routes/clerk.ts`
```typescript
clerkRouter.post('/webhook', express.raw({ type: 'application/json' }), async (req, res) => {
  const evt = await verifyWebhook(req, {
    signingSecret: process.env.CLERK_WEBHOOK_SIGNING_SECRET,
  });
  
  // Handle user.created, user.updated, organization.created, etc.
});
```

## References
- **Backend**: `apps/api/src/routes/clerk.ts`
- **Frontend**: `apps/web/src/middleware.ts`
- **External Docs**: [Clerk](https://clerk.com/docs)
