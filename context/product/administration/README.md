# Administration - Administration Features

## Overview

The Administration domain groups features reserved for application administrators.

**Status**: ⚠️ Partial  
**Coverage**: 40%

---

## Main Features

- ✅ Admin role verification (`authIsAdminMiddleware`)
- ✅ API routes `/admin/*`
- ✅ Repository admin page per repository
- ❌ Global admin dashboard
- ❌ Audit logs
- ❌ System metrics

---

## Routes

**API**:

- `/api/admin/*` - Routes admin (contenu non analysé en détail)

**Frontend**:

- `/dashboard/repos/[repositoryId]/admin` - Admin par dépôt

---

## Business Rules

- **Admin role required**: Verification via `authIsAdminMiddleware`
- **Global admin**: Boolean `isAdmin` (no granularity)
- **Admin actions**: ❓ Not documented

---

## Gaps & Limitations

- ❌ Admin actions not listed
- ❌ Global admin dashboard missing
- ❌ Audit logs not implemented
- ❌ System metrics missing
- ❌ User/org management by admin

---

## References

- `apps/api/src/routes/admin.ts` - Admin routes
- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/admin/page.tsx`
- `apps/api/src/middlewares/authMiddleware.ts` - authIsAdminMiddleware

---

**Last updated**: 2025-10-17
