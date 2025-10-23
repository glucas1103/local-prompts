# Research: User Management Domain

## Recherche complétée le: 2025-10-17

## Statut: ✅ COMPLÉTÉ

---

## Findings Summary

### User Data Model

**Schema Table**: `users`

**Fields**:

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

**Relations**:

- `organizations`: many organizationMembers (user can belong to multiple orgs)

**Zod Schema**: `packages/types/src/user.ts`

---

## 1. User Profile

### Implémentation

**API Endpoint**: `GET /api/users/`

**Code**:

```typescript
userRoutes.get(
  "/",
  validateResponse(userSchema),
  authMiddleware,
  async (req, res) => {
    const { userId } = req.auth;
    const [user] = await req.tx
      .select()
      .from(users)
      .where(eq(users.id, userId))
      .limit(1);

    if (!user) {
      return res.status(404).json({ error: "USER_NOT_FOUND" });
    }

    res.json(user);
  }
);
```

### Données du Profil

**Champs disponibles**:

- ✅ `id` - Clerk user ID (read-only)
- ✅ `email` - Email address (synced from Clerk)
- ✅ `firstName` - First name (synced from Clerk)
- ✅ `lastName` - Last name (synced from Clerk)
- ✅ `photoUrl` - Profile photo URL (synced from Clerk)
- ✅ `isAdmin` - Admin role flag (synced from Clerk metadata)
- ✅ `deleted` - Soft-delete flag (read-only)
- ✅ `deletedAt` - Deletion timestamp (read-only)
- ✅ `createdAt` - Creation timestamp (read-only)
- ✅ `updatedAt` - Last update timestamp (read-only)

### Édition du Profil

❓ **Aucun endpoint UPDATE trouvé** dans `apps/api/src/routes/users.ts`

**Hypothèse**: Édition gérée via Clerk:

- User met à jour son profil dans Clerk UI (Account Portal)
- Clerk envoie webhook `user.updated`
- Backend sync automatiquement via webhook handler

**Champs éditables** (via Clerk):

- First name
- Last name
- Email (avec vérification)
- Photo
- Password (si email/password activé)

**Champs non éditables** (contrôlés par admin):

- `isAdmin` - Via Clerk metadata
- `deleted` - Via soft-delete process
- `id` - Immutable

### Validation

**Zod Schema**:

```typescript
export const userSchema = z.object({
  id: z.string(),
  email: z.string().email(), // Email validation
  isAdmin: z.boolean(),
  firstName: z.string(),
  lastName: z.string(),
  photoUrl: z.string(),
  deleted: z.boolean(),
  deletedAt: z.date().nullable(),
  createdAt: z.date(),
  updatedAt: z.date(),
});
```

**Validation des réponses**: Middleware `validateResponse(userSchema)`

### Règles Métier

- **Email unique**: Géré par Clerk
- **Prénom/nom**: Pas de contraintes de longueur visibles
- **Photo par défaut**: Générée si absente (via `generateSeedPhotoUrl()`)
- **Sync uni-directionnel**: Clerk → App (pas de update direct en DB)

---

## 2. User Roles

### Système de Rôles

**Rôle disponible**: `isAdmin` (boolean)

**Pas de système granulaire** de permissions. Seulement:

- Regular user (`isAdmin = false`)
- Admin (`isAdmin = true`)

### Affectation du Rôle Admin

**Source**: `public_metadata.isAdmin` dans Clerk

**Code extraction**:

```typescript
const isAdmin = data.public_metadata.isAdmin === true;
```

**Process**:

1. Admin set `public_metadata.isAdmin = true` dans Clerk Dashboard
2. Clerk envoie webhook `user.updated`
3. Backend extrait et sauvegarde `isAdmin` field
4. User a maintenant accès aux routes admin

### Vérification Admin

**Middleware**:

```typescript
export function authIsAdminMiddleware(req, res, next) {
  if (!isAdmin(req)) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}

function isAdmin(req) {
  return req.auth.user?.isAdmin;
}
```

**Usage**:

```typescript
router.delete(
  "/admin/route",
  authMiddleware,
  authIsAdminMiddleware, // ← Admin check
  handler
);
```

### Permissions Admin

**Actions réservées aux admins**:

- ❓ Routes `/api/admin/*` (non analysées en détail yet)
- ❓ Repository admin actions
- ❓ System management

### Limites du Système Actuel

❌ **Pas de rôles intermédiaires** (viewer, editor, etc.)  
❌ **Pas de permissions granulaires** (RBAC)  
❌ **Admin global uniquement** (pas par organization)  
❌ **Pas de custom roles**

💡 **Amélioration future**: Système RBAC complet

---

## 3. User Lifecycle

### Création

**Trigger**: User sign-up via Clerk

**Flow**:

1. User s'inscrit via Clerk (`/sign-up`)
2. Clerk crée user dans leur système
3. Clerk envoie webhook `user.created`
4. Backend reçoit webhook:
   ```typescript
   case 'user.created': {
     const userInfo = extractUserInfo(data);
     await req.tx.insert(users).values({
       ...userInfo,
     });
     break;
   }
   ```
5. User inséré dans table `users` PostgreSQL
6. Création terminée

**Données initiales**:

- `id`: Clerk user ID
- `email`: From Clerk
- `firstName`, `lastName`: From Clerk ou empty string
- `photoUrl`: From Clerk ou generated
- `isAdmin`: `false` (default)
- `deleted`: `false` (default)
- `createdAt`, `updatedAt`: `now()`

### Mise à Jour

**Trigger**: User modifie profil dans Clerk

**Flow**:

1. User édite profil dans Clerk Account Portal
2. Clerk envoie webhook `user.updated`
3. Backend reçoit webhook:
   ```typescript
   case 'user.updated': {
     await req.tx
       .update(users)
       .set({
         ...extractUserInfo(data),
         updatedAt: new Date(),
       })
       .where(eq(users.id, data.id));
     break;
   }
   ```
4. User updated dans DB

**Champs mis à jour**:

- `email`
- `firstName`, `lastName`
- `photoUrl`
- `isAdmin` (si metadata changed)
- `updatedAt` (auto)

### Suppression (Soft-Delete)

**Trigger**: User supprimé dans Clerk (ou par admin)

**Flow**:

1. User deleted dans Clerk
2. Clerk envoie webhook `user.deleted`
3. Backend reçoit webhook:
   ```typescript
   case 'user.deleted':
     await req.tx
       .update(users)
       .set({
         deleted: true,
         deletedAt: new Date(),
       })
       .where(eq(users.id, data.id));
     break;
   ```
4. User marqué `deleted = true`

**Comportement**:

- ❓ **User peut-il encore se connecter** (probablement non via Clerk)
- ❓ **Données conservées** (oui, soft-delete)
- ❓ **Restauration possible** (non visible dans code)
- ❓ **Hard delete planifié** (non implémenté)

### États du Cycle de Vie

| État    | `deleted` | Comportement               |
| ------- | --------- | -------------------------- |
| Active  | `false`   | User normal, plein accès   |
| Deleted | `true`    | Soft-deleted, accès refusé |

**Pas d'états intermédiaires** (suspended, pending, etc.)

---

## 4. User Deletion

### Soft-Delete

**Implémentation**: Fields `deleted` et `deletedAt`

**Process**: Cf. section 3 (User Lifecycle > Suppression)

**Règles**:

- Données préservées en DB
- User ne peut plus se connecter (géré par Clerk)
- ❓ Réversible ou non (non implémenté)

### Hard-Delete

❌ **Pas implémenté** dans le code actuel

**Serait nécessaire pour**:

- Compliance RGPD (droit à l'effacement)
- Nettoyage des données après X temps
- Suppression définitive sur demande

💡 **Amélioration future**:

- Scheduled job pour hard-delete après X jours
- User data export avant suppression (RGPD)
- Anonymisation alternative

### Cascade Behavior

**Relations affectées**:

- `organizationMembers`: Membership with organizations

❓ **Que devient organizationMembership quand user deleted**:

- Option 1: Soft-delete cascade (marquer deleted aussi)
- Option 2: Hard-delete membership (lien cassé)
- Option 3: Rien (membership reste)

**Non implémenté explicitement dans code**

### RGPD Compliance

❓ **Data Export**: Pas implémenté
❓ **Right to be Forgotten**: Soft-delete seulement
❓ **Data Portability**: Non implémenté

💡 **Recommandation**: Ajouter endpoints RGPD

---

## 5. Historique d'Activité

### Logging

❌ **Pas de table `user_activity` ou `audit_log`**

**Logging actuel**:

- Structured logging via Pino (`@/utils/logger`)
- Logs applicatifs (requêtes, erreurs)
- Pas de tracking d'activité user spécifique

### Données Tracées

✅ **Timestamps**:

- `createdAt`: User creation
- `updatedAt`: Last profile update
- `deletedAt`: Deletion timestamp

❌ **Non tracé**:

- Last login
- Login history
- Actions performed
- IP addresses
- Device information
- Activity timeline

### Amélioration Future

💡 **Activity Log Table**:

```typescript
{
  id: string,
  userId: string,
  action: string,         // 'login', 'profile_update', etc.
  metadata: jsonb,        // Additional context
  ipAddress: string,
  userAgent: string,
  createdAt: timestamp,
}
```

---

## External Integrations

### Clerk

**User Management Features Used**:

- User creation & updates
- Profile management
- Email management
- OAuth connections
- Account Portal (user self-service)

**Webhooks**:

- `user.created`
- `user.updated`
- `user.deleted`

### Pas d'autres intégrations

User management entièrement géré via Clerk + local DB

---

## Code Examples

### Get Current User

```typescript
// In API route
router.get("/", authMiddleware, async (req, res) => {
  const user = req.auth.user; // Already loaded by middleware
  res.json(user);
});
```

### Check if Admin

```typescript
const isUserAdmin = req.auth.user?.isAdmin;

if (isUserAdmin) {
  // Admin-only logic
}
```

### Query Users (Admin Only)

❌ **Pas implémenté** - Pas de route pour lister tous les users

💡 **À implémenter**:

```typescript
router.get(
  "/admin/users",
  authMiddleware,
  authIsAdminMiddleware,
  async (req, res) => {
    const allUsers = await req.tx
      .select()
      .from(users)
      .where(eq(users.deleted, false));
    res.json(allUsers);
  }
);
```

---

## Testing

**User Fixtures**:

```typescript
import { createUserFixture } from "@boilerplate-turbo/types";

const testUser = createUserFixture(organizationId);
```

**Mock Auth with User**:

```typescript
mockAuthMiddleware(isAdmin: boolean = false)
```

---

## Gaps & Unknowns

### Profile Editing

❓ **Frontend pour éditer profil**: Pas trouvé dans codebase

- Likely: Clerk Account Portal (hosted by Clerk)
- Alternative: Custom profile page (non implémenté)

### User Search

❓ **Recherche d'utilisateurs**: Non implémentée
❓ **Liste de tous les users (admin)**: Non implémentée
❓ **Filtrage par role/organization**: Non implémenté

### Advanced Features

❌ **User Preferences**: Non implémenté (theme, language, notifications)
❌ **User Avatar Upload**: Géré via Clerk uniquement
❌ **Email Notifications**: Non implémenté (welcome email, etc.)
❌ **User Invitations**: Via organization membership

### Security

❓ **Password Policy**: Géré par Clerk
❓ **Account Lockout**: Géré par Clerk
❓ **Brute Force Protection**: Géré par Clerk

---

## Recommendations

### Short Term

1. **Add GET /admin/users** - Liste tous les utilisateurs (admin only)
2. **Add Last Login tracking** - Ajouter `lastLoginAt` field
3. **RGPD Compliance** - User data export endpoint

### Medium Term

1. **Activity Logging** - Table audit_log
2. **User Preferences** - Table user_preferences
3. **Advanced User Search** - Filtering, pagination

### Long Term

1. **RBAC System** - Remplacer isAdmin boolean
2. **Custom Roles** - Définir des rôles personnalisés
3. **Granular Permissions** - Permission-based access control

---

## References

### Main Files

- `apps/api/src/db/schema.ts` - users table (lines 11-26)
- `apps/api/src/routes/users.ts` - GET user profile (41 lines)
- `apps/api/src/routes/clerk.ts` - User webhooks (lines 69-107)
- `packages/types/src/user.ts` - User schema & types (17 lines)
- `apps/api/src/middlewares/authMiddleware.ts` - Auth & admin checks (90 lines)

### Schema Definition

```typescript
// apps/api/src/db/schema.ts
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

### Relations

```typescript
export const usersRelations = relations(users, ({ many }) => ({
  organizations: many(organizationMembers),
}));
```

---

**Status**: ✅ RECHERCHE COMPLÉTÉE  
**Coverage**: ~75% (75% implemented, 25% missing features)  
**Files Analyzed**: 5  
**LOC Reviewed**: ~180

