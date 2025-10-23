# Organization - Organization Management (Multi-tenant)

## Overview

The Organization domain manages the application's multi-tenant system. Each organization represents an isolated tenant with its own members, plans, and data.

**Provider**: Clerk (source) + PostgreSQL (local storage)  
**Status**: ✅ Implemented  
**Coverage**: 70%

---

## Main Features

- ✅ Organization creation/deletion (via Clerk)
- ✅ Member management (organization_members)
- ✅ Plans: FREE, PRO
- ✅ Roles per member (string field)
- ✅ Organization selector (UI)
- ✅ Soft-delete organizations
- ✅ Multi-tenancy (orgId scoping)

---

## Use Cases

### UC-1: Create an organization

- User creates org in Clerk
- Webhook `organization.created` syncs to DB
- User becomes first member (owner likely)

### UC-2: Invite a member

- Managed via Clerk Organizations
- Webhook `organizationMembership.created`
- Member added to local DB

### UC-3: Switch organization

- User uses organization selector (UI)
- Context orgId changes
- All data filtered by new orgId

---

## Business Rules

- **Organization context required**: `authMiddleware` verifies `orgId`
- **Available plans**: FREE (default), PRO
- **Plan limits**: ❓ Not implemented in code
- **Member roles**: String field (not strongly typed)
- **Soft-delete organizations**: `deleted = true`, `deletedAt = now()`

---

## Schema

**Table `organizations`**:

```typescript
{
  (id, name, plan, deleted, deletedAt, createdAt, updatedAt);
}
```

**Table `organization_members`**:

```typescript
{
  (id, organizationId, userId, role, createdAt, updatedAt);
}
```

---

## Gaps & Limitations

- ❌ Plan limits not implemented
- ❌ Billing system missing
- ❌ Custom invitation (managed by Clerk)
- ❌ Roles not explicitly defined (free string)
- ❌ Ownership transfer not documented

---

## References

- `apps/api/src/db/schema.ts` - organizations, organization_members
- `apps/api/src/routes/organization.ts` - GET org, GET members
- `apps/api/src/routes/clerk.ts` - Org webhooks
- `apps/web/src/app/(protected)/dashboard/org-selector.tsx`

---

**Last updated**: 2025-10-17
