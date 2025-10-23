# Authentication - User Authentication and Sessions

## Overview

The Authentication domain manages user authentication and session management via **Clerk**. It includes sign-up, sign-in, logout, GitHub OAuth, and user data synchronization via webhooks.

**Provider**: Clerk (@clerk/nextjs + @clerk/express)  
**Status**: ✅ Implemented  
**Coverage**: 85%

---

## Main Features

### ✅ Sign-Up (Registration)

- Clerk form with GitHub OAuth
- Email/Password (depending on Clerk configuration)
- Automatic creation in DB via webhook
- Default profile photo generated

### ✅ Sign-In (Login)

- GitHub OAuth or Email/Password
- Sessions managed by Clerk (JWT)
- User/org verification in DB

### ✅ GitHub OAuth

- "Continue with GitHub" button
- Retrieval of email, name, photo
- Integration via Clerk (not directly GitHub)

### ✅ Session Management

- JWT tokens by Clerk
- Next.js middleware for protected routes (`/dashboard(.*)`)
- User + organization verification on each API request

### ✅ Logout

- Session invalidation on Clerk side
- Component: `logout-alert-dialog.tsx`

### ✅ Clerk Webhooks

- Automatic user/org data sync
- Events: `user.*`, `organization.*`, `organizationMembership.*`
- Soft-delete for user/org deleted

### ✅ Admin Role

- Field `isAdmin` (boolean)
- Source: `public_metadata.isAdmin` from Clerk
- Middleware: `authIsAdminMiddleware`

---

## Main Use Cases

### UC-1: New user signs up with GitHub

**Actor**: Visitor  
**Objective**: Create an account

**Flow**:

1. Visitor accesses `/sign-up`
2. Clicks on "Continue with GitHub"
3. Authorizes the app on GitHub
4. Clerk creates the account
5. Webhook `user.created` synchronizes to DB
6. Redirect to dashboard

**Detail file**: [sign-up-flow.md](./sign-up-flow.md)

---

### UC-2: User logs in

**Actor**: Registered user  
**Objective**: Access their account

**Flow**:

1. Accesses `/sign-in`
2. Authenticates (GitHub or Email/Password)
3. Clerk creates a session (JWT)
4. Middleware verifies token
5. API verifies user + org in DB
6. Access to dashboard

**Detail file**: [sign-in-flow.md](./sign-in-flow.md)

---

### UC-3: User logs out

**Actor**: Logged-in user  
**Objective**: End their session

**Flow**:

1. Clicks on "Logout" in sidebar
2. Confirmation via alert dialog
3. `signOut()` called (Clerk SDK)
4. Session invalidated
5. Redirect to public page

**Detail file**: [logout-flow.md](./logout-flow.md)

---

## Business Rules

### Rule 1: Organization required

- **Statement**: A user must have an organization context to access the dashboard
- **Verification**: `authMiddleware` verifies `req.auth.orgId`
- **Error**: `UNAUTHORIZED - ORG ID NOT FOUND`

### Rule 2: User must exist in DB

- **Statement**: The Clerk user must be synchronized in PostgreSQL database
- **Verification**: `authMiddleware` loads user from DB
- **Error**: `USER_NOT_FOUND`

### Rule 3: Unique email

- **Statement**: An email can only be associated with one account
- **Management**: By Clerk
- **Error**: Clerk returns duplication error

### Rule 4: Admin via metadata

- **Statement**: The admin role is defined via `public_metadata.isAdmin` in Clerk
- **Default**: `false`
- **Modification**: Manual in Clerk Dashboard only

### Rule 5: Soft-delete users

- **Statement**: User deletion is logical (`deleted = true`)
- **Trigger**: Webhook `user.deleted`
- **Data**: Preserved in DB

---

## Functional Dependencies

### Depends on:

- ❌ No dependencies (foundation domain)

### Required by:

- ✅ **User Management** - User profile management
- ✅ **Organization** - Multi-tenancy
- ✅ **All domains** - Authentication required everywhere

---

## External Integrations

### Clerk

- **Services used**: Authentication, User Management, Organization Management, Webhooks
- **SDKs**: `@clerk/nextjs`, `@clerk/express`
- **Required config**:
  - Clerk Application (Dashboard)
  - GitHub OAuth provider
  - Webhook URL + signing secret
- **Env vars**: `CLERK_WEBHOOK_SIGNING_SECRET`

### GitHub (OAuth only)

- **Usage**: Authentication via GitHub account
- **Managed by**: Clerk (no direct code)
- **Distinction**: GitHub OAuth ≠ GitHub App (repository integration)

---

## Code Examples

### Protect a Next.js route

```typescript
// apps/web/src/app/(protected)/dashboard/page.tsx
import { auth } from '@clerk/nextjs/server';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const { userId, orgId } = await auth();

  if (!userId) {
    redirect('/sign-in');
  }

  // Render protected content
  return <div>Dashboard</div>;
}
```

### Protect an API route

```typescript
// apps/api/src/routes/example.ts
import { authMiddleware } from "@/middlewares/authMiddleware";

router.get("/protected", authMiddleware, async (req, res) => {
  // req.auth.user and req.auth.organization available
  const user = req.auth.user;
  const org = req.auth.organization;

  res.json({ user, org });
});
```

### Admin-only route

```typescript
import {
  authMiddleware,
  authIsAdminMiddleware,
} from "@/middlewares/authMiddleware";

router.delete(
  "/admin/action",
  authMiddleware,
  authIsAdminMiddleware,
  async (req, res) => {
    // Only admins can access
    res.json({ success: true });
  }
);
```

---

## Gaps & Limitations

### ❓ External Clerk Configuration

The following elements are configured in Clerk Dashboard (not visible in code):

- Enabled providers (GitHub OAuth + Email/Password ?)
- MFA settings
- Session duration
- Password policy
- Email templates
- Rate limiting

### ❌ Non-Implemented Features

- **Forgot Password**: Managed by Clerk (no custom code)
- **Email Verification**: Managed by Clerk
- **Session Activity Log**: No connection tracking
- **Last Login tracking**: `lastLoginAt` field missing
- **Brute Force Protection**: Managed by Clerk
- **Device Management**: Not implemented

### ⚠️ Points of Attention

- **Session duration**: Not documented (defined in Clerk)
- **Refresh tokens**: Automatic via Clerk (no visible code)
- **Logout backend notification**: No webhook for logout

---

## Detail Files

| File                                             | Description                   | Status |
| ------------------------------------------------ | ----------------------------- | ------ |
| [sign-up-flow.md](./sign-up-flow.md)             | Complete registration process | ✅     |
| [sign-in-flow.md](./sign-in-flow.md)             | Login process                 | ✅     |
| [github-oauth.md](./github-oauth.md)             | GitHub OAuth integration      | ✅     |
| [session-management.md](./session-management.md) | Session and token management  | ✅     |
| [logout-flow.md](./logout-flow.md)               | Logout process                | ✅     |
| [webhooks-clerk.md](./webhooks-clerk.md)         | Webhook synchronization       | ✅     |

---

## Technical References

### Source Files

- `apps/web/src/middleware.ts` - Next.js route protection (19 lines)
- `apps/api/src/routes/clerk.ts` - Clerk webhooks (253 lines)
- `apps/api/src/middlewares/authMiddleware.ts` - Auth verification (90 lines)
- `apps/web/src/app/(public)/(auth)/` - Sign-in/sign-up pages

### DB Schema

- Table `users` in `apps/api/src/db/schema.ts` (lines 11-26)

### Types

- `packages/types/src/user.ts` - User Zod schema

### External Documentation

- [Clerk Docs](https://clerk.com/docs)
- [Clerk Webhooks](https://clerk.com/docs/integrations/webhooks/overview)
- [Clerk Next.js](https://clerk.com/docs/quickstarts/nextjs)

---

**Last updated**: 2025-10-17  
**Workflow**: audit-codebase-product  
**Search file**: `.speiros/artifacts/audit-product/research/authentication/`
