# User Management - User Management

## Overview

The User Management domain manages the user lifecycle, their profiles, roles and personal data. User data is synchronized from Clerk and stored locally in PostgreSQL.

**Provider**: Clerk (source) + PostgreSQL (local storage)  
**Status**: ✅ Implemented  
**Coverage**: 75%

---

## Main Features

### ✅ User Profile

- User profile retrieval (GET `/api/users/`)
- Data: email, firstName, lastName, photoUrl, isAdmin
- Editing via Clerk Account Portal (no custom API)

### ✅ User Roles

- `isAdmin` role (boolean)
- Source: `public_metadata.isAdmin` from Clerk
- Middleware: `authIsAdminMiddleware`

### ✅ User Lifecycle

- **Creation**: Via webhook `user.created`
- **Update**: Via webhook `user.updated`
- **Deletion**: Via webhook `user.deleted` (soft-delete)

### ✅ Soft-Delete

- Fields: `deleted` (boolean), `deletedAt` (timestamp)
- User marked as deleted but data preserved

### ❌ Hard-Delete

- Not implemented
- ❓ GDPR compliance (right to erasure)

### ❌ Activity Logging

- No user activity tracking
- Only timestamps: `createdAt`, `updatedAt`, `deletedAt`

---

## Main Use Cases

### UC-1: User consults their profile

**Actor**: Logged-in user  
**Objective**: View their personal information

**Flow**:

1. Frontend calls GET `/api/users/`
2. Backend verifies authentication
3. Backend retrieves user by `userId`
4. Returns complete profile
5. Frontend displays the data

**API**: `GET /api/users/`  
**Detail file**: [user-profile.md](./user-profile.md)

---

### UC-2: User modifies their profile

**Actor**: Logged-in user  
**Objective**: Update their information

**Flow**:

1. User accesses Clerk Account Portal
2. Modifies firstName, lastName, email or photo
3. Clerk saves the changes
4. Clerk sends webhook `user.updated`
5. Backend synchronizes to local DB
6. Changes visible in the app

**Note**: No custom UPDATE API, managed by Clerk  
**Detail file**: [user-profile.md](./user-profile.md)

---

### UC-3: Admin deletes a user

**Actor**: Admin  
**Objective**: Delete a user account

**Flow**:

1. Admin deletes user in Clerk Dashboard
2. Clerk sends webhook `user.deleted`
3. Backend marks `deleted = true`, `deletedAt = now()`
4. User can no longer log in (Clerk)
5. Data preserved in DB

**Note**: Soft-delete only  
**Detail file**: [user-deletion.md](./user-deletion.md)

---

## Business Rules

### Rule 1: Unique email

- **Statement**: An email can only be associated with one user
- **Management**: By Clerk
- **Validation**: Zod schema `z.string().email()`

### Rule 2: Unidirectional sync

- **Statement**: Clerk is the source of truth, local DB is a mirror
- **Direction**: Clerk → App (not the reverse)
- **Mechanism**: Clerk webhooks

### Rule 3: Default photo

- **Statement**: If no photo provided, generate a default profile photo
- **Function**: `generateSeedPhotoUrl(firstName + lastName)`
- **Usage**: For users without avatar

### Rule 4: Admin role immutable per user

- **Statement**: Only an admin can modify a user's admin status
- **Mechanism**: Via `public_metadata.isAdmin` in Clerk Dashboard
- **Default**: `false`

### Rule 5: Soft-delete preserves data

- **Statement**: Deleted user keeps all their data in DB
- **Flag**: `deleted = true`
- **Timestamp**: `deletedAt = deletion date`
- **Restoration**: ❓ Not implemented

---

## User Data

### Schema Table `users`

```typescript
{
  id: text('id').primaryKey(),                  // Clerk user ID
  email: text('email').notNull(),
  isAdmin: boolean('is_admin').notNull().default(false),
  firstName: text('first_name').notNull(),
  lastName: text('last_name').notNull(),
  photoUrl: text('photo_url').notNull(),
  deleted: boolean('deleted').notNull().default(false),
  deletedAt: timestamp('deleted_at'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
}
```

### Editable Fields (via Clerk)

| Field       | Editable | By whom | How                        |
| ----------- | -------- | ------- | -------------------------- |
| `email`     | ✅       | User    | Clerk Account Portal       |
| `firstName` | ✅       | User    | Clerk Account Portal       |
| `lastName`  | ✅       | User    | Clerk Account Portal       |
| `photoUrl`  | ✅       | User    | Clerk Account Portal       |
| `isAdmin`   | ✅       | Admin   | Clerk Dashboard (metadata) |
| `id`        | ❌       | -       | Immutable                  |
| `deleted`   | ❌       | System  | Via webhook                |
| `createdAt` | ❌       | System  | Auto                       |
| `updatedAt` | ❌       | System  | Auto                       |

---

## Functional Dependencies

### Depends on:

- ✅ **Authentication** - Clerk authentication, webhooks

### Required by:

- ✅ **Organization** - Organization members (relation)
- ✅ **Administration** - Admin role verification

---

## External Integrations

### Clerk

**User Management Features Used**:

- User profile management
- Email management
- Photo upload
- Account Portal (self-service)

**Webhooks**:

- `user.created` - User creation in DB
- `user.updated` - Profile update
- `user.deleted` - Soft-delete user

---

## Code Examples

### Get current user

```typescript
// In API route (after authMiddleware)
router.get("/protected", authMiddleware, async (req, res) => {
  const user = req.auth.user; // Already loaded
  res.json({
    email: user.email,
    name: `${user.firstName} ${user.lastName}`,
    isAdmin: user.isAdmin,
  });
});
```

### Check if admin

```typescript
const isUserAdmin = req.auth.user?.isAdmin;

if (isUserAdmin) {
  // Admin-only logic
} else {
  return res.status(403).json({ error: "FORBIDDEN" });
}
```

### User fixture for tests

```typescript
import { createUserFixture } from "@boilerplate-turbo/types";

const testUser = createUserFixture(organizationId);
// Returns mock user with all fields
```

---

## Gaps & Limitations

### ❌ Non-Implemented Features

**Profile Management**:

- No custom API endpoint to edit profile
- No custom frontend to edit profile (Clerk Portal only)

**User Administration**:

- No GET `/admin/users` route (list all users)
- No user search
- No filtering by role/organization

**Advanced Features**:

- **User Preferences**: Not implemented (theme, language, notifications)
- **Last Login tracking**: `lastLoginAt` field missing
- **Activity Logging**: No audit_log table
- **User Invitations**: Managed via organization membership
- **Email Notifications**: No custom emails (welcome, etc.)

**GDPR Compliance**:

- **Data Export**: Not implemented
- **Right to be Forgotten**: Soft-delete only (no hard-delete)
- **Data Portability**: Not implemented

### ⚠️ Points of Attention

- **Cascade behavior**: What happens to organizationMembership when user deleted? (not documented)
- **Restoration**: Can soft-deleted user be restored? (not implemented)
- **Password policy**: Managed by Clerk (not visible in code)

---

## Recommendations

### Short Term

1. **Add GET /admin/users** - List all users (admin only)
2. **Add Last Login tracking** - Add `lastLoginAt` field
3. **GDPR Compliance** - User data export endpoint

### Medium Term

1. **Activity Logging** - `user_activity` or `audit_log` table
2. **User Preferences** - `user_preferences` table
3. **User Search** - Filtering, pagination, sorting

### Long Term

1. **RBAC System** - Replace boolean `isAdmin` with permission system
2. **Custom Roles** - Define custom roles
3. **Granular Permissions** - Permission-based access control

---

## Detail Files

| File                                     | Description                          | Status |
| ---------------------------------------- | ------------------------------------ | ------ |
| [user-profile.md](./user-profile.md)     | User profile management              | ✅     |
| [user-roles.md](./user-roles.md)         | Role system (admin)                  | ✅     |
| [user-lifecycle.md](./user-lifecycle.md) | Lifecycle (creation, update, delete) | ✅     |
| [user-deletion.md](./user-deletion.md)   | Soft-delete and deletion             | ✅     |

---

## Technical References

### Source Files

- `apps/api/src/db/schema.ts` - users table (lines 11-26)
- `apps/api/src/routes/users.ts` - GET user profile (41 lines)
- `apps/api/src/routes/clerk.ts` - User webhooks (lines 69-107)
- `apps/api/src/middlewares/authMiddleware.ts` - Auth & admin checks (lines 41-54)

### Types

- `packages/types/src/user.ts` - User schema & types

### DB Schema

```typescript
export const users = pgTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull(),
  isAdmin: boolean("is_admin").notNull().default(false),
  firstName: text("first_name").notNull(),
  lastName: text("last_name").notNull(),
  photoUrl: text("photo_url").notNull(),
  deleted: boolean("deleted").notNull().default(false),
  deletedAt: timestamp("deleted_at"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

---

**Last updated**: 2025-10-17  
**Workflow**: audit-codebase-product  
**Search file**: `.speiros/artifacts/audit-product/research/user-management/`
