# Research: Authentication Domain

## Recherche complétée le: 2025-10-17

## Statut: ✅ COMPLÉTÉ

---

## Findings Summary

### Configuration Clerk

**Provider utilisé**: Clerk (@clerk/nextjs + @clerk/express)

**Configuration identifiée**:

- Middleware Next.js avec protection des routes privées (`/dashboard(.*)`)
- Webhooks Clerk implémentés pour synchronisation
- Authentification basée sur Clerk Components (SignIn, SignUp)
- Support GitHub OAuth (via Clerk)

**Fichiers sources**:

- `apps/web/src/middleware.ts` - Protection des routes
- `apps/api/src/routes/clerk.ts` - Webhooks
- `apps/api/src/middlewares/authMiddleware.ts` - Vérifications auth
- `apps/web/src/app/(public)/(auth)/` - Pages auth

---

## 1. Sign-Up Flow

### Implémentation

**Frontend**:

- Page: `/sign-up`
- Composant: `<SignUp />` de `@clerk/nextjs`
- Wrapper: `<AuthLayoutWrapper>` pour mise en page

**Backend - Webhook `user.created`**:

```typescript
case 'user.created': {
  const userInfo = extractUserInfo(data);
  await req.tx.insert(users).values({
    ...userInfo,
  });
  break;
}
```

**Données extraites lors de la création**:

```typescript
function extractUserInfo(data: UserJSON) {
  const firstName = data.first_name || "";
  const lastName = data.last_name || "";
  const photoUrl =
    data.image_url || generateSeedPhotoUrl(`${firstName}${lastName}`);
  const isAdmin = data.public_metadata.isAdmin === true;

  return {
    id: data.id,
    email: data.email_addresses[0].email_address,
    firstName,
    lastName,
    photoUrl,
    isAdmin,
  };
}
```

### Processus

1. **Utilisateur arrive sur `/sign-up`**
2. **Clerk affiche le formulaire d'inscription** (géré par Clerk)
   - GitHub OAuth button visible
   - Ou Email/Password (selon config Clerk)
3. **Utilisateur remplit et soumet**
4. **Clerk crée l'utilisateur** dans leur système
5. **Clerk envoie webhook `user.created`** à `/api/clerk/webhook`
6. **Backend reçoit le webhook**:
   - Vérifie la signature avec `CLERK_WEBHOOK_SIGNING_SECRET`
   - Extrait les infos utilisateur
   - Insère dans table `users` PostgreSQL
   - Génère photo par défaut si absente
7. **Utilisateur est redirigé** vers dashboard

### Règles Métier

- **Email obligatoire**: Extrait de `data.email_addresses[0].email_address`
- **Nom/prénom**: Optionnels (chaîne vide par défaut)
- **Photo**: Générée automatiquement via `generateSeedPhotoUrl()` si non fournie
- **Rôle admin**: Défini via `public_metadata.isAdmin` (default: false)
- **Soft-delete support**: Fields `deleted` et `deletedAt` disponibles

### Questions non résolues

- ✅ **GitHub OAuth activé** (via Clerk, composant visible)
- ❓ **Email/Password activé** (non visible dans le code, dépend config Clerk)
- ❓ **Validation email requise** (géré par Clerk)
- ❓ **MFA disponible** (non visible dans le code)

---

## 2. Sign-In Flow

### Implémentation

**Frontend**:

- Page: `/sign-in`
- Composant: `<SignIn />` de `@clerk/nextjs`
- Wrapper: `<AuthLayoutWrapper>` pour mise en page

**Backend**: Aucun webhook spécifique pour sign-in. Clerk gère la session.

### Processus

1. **Utilisateur arrive sur `/sign-in`**
2. **Clerk affiche le formulaire de connexion**
   - GitHub OAuth button
   - Ou Email/Password (selon config)
3. **Utilisateur s'authentifie**
4. **Clerk crée une session**
5. **Clerk émet JWT token** avec claims:
   - `userId`
   - `orgId` (si organization sélectionnée)
   - `sessionId`
   - `sessionClaims`
6. **Middleware Next.js vérifie le token**:
   - Route protégée (`/dashboard(.*)`)?
   - Oui → `auth.protect()` vérifie la session
   - Non → Accès autorisé
7. **Backend vérifie avec `authMiddleware`**:
   - Vérifie `req.auth.userId` présent
   - Vérifie `req.auth.orgId` présent
   - Charge user et organization depuis DB
   - Attache à `req.auth.user` et `req.auth.organization`

### Règles Métier

- **userId et orgId requis**: Pour accéder aux routes protégées
- **User doit exister en DB**: Sinon erreur `USER_NOT_FOUND`
- **Organization doit exister en DB**: Sinon erreur `ORGANIZATION_NOT_FOUND`
- **Session gérée par Clerk**: Pas de logique custom

### Code Middleware Auth

```typescript
export async function authMiddleware(req, res, next) {
  if (!req.auth.userId) {
    return res.status(401).json({ error: "UNAUTHORIZED - USER ID NOT FOUND" });
  }
  if (!req.auth.orgId) {
    return res.status(401).json({ error: "UNAUTHORIZED - ORG ID NOT FOUND" });
  }
  const [currentUser] = await req.tx
    .select()
    .from(users)
    .where(eq(users.id, req.auth.userId))
    .limit(1);
  if (!currentUser) {
    return res.status(401).json({ error: "USER_NOT_FOUND" });
  }
  req.auth.user = currentUser;
  const [organization] = await req.tx
    .select()
    .from(organizations)
    .where(eq(organizations.id, req.auth.orgId))
    .limit(1);
  if (!organization) {
    return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
  }
  req.auth.organization = organization;
  next();
}
```

### Questions non résolues

- ❓ **Durée de session** (géré par Clerk, non configuré dans code)
- ❓ **Refresh token** (géré par Clerk automatiquement)
- ❓ **Tentatives de connexion échouées** (géré par Clerk)

---

## 3. GitHub OAuth

### Implémentation

**Configuration**: Via Clerk Dashboard (external)

**Flow**:

1. Utilisateur clique sur "Continue with GitHub" dans Clerk component
2. Clerk redirige vers GitHub OAuth
3. GitHub demande permissions
4. Utilisateur autorise
5. GitHub redirige vers Clerk callback
6. Clerk crée/met à jour l'utilisateur
7. Webhook `user.created` ou `user.updated` envoyé
8. Backend synchronise en DB

### Données récupérées

- **Email**: `data.email_addresses[0].email_address`
- **Nom/Prénom**: `data.first_name`, `data.last_name`
- **Photo**: `data.image_url` (avatar GitHub)
- **GitHub ID**: Stocké par Clerk (pas dans notre DB)

### Règles Métier

- **Email GitHub prioritaire**: Utilisé pour l'email principal
- **Photo GitHub utilisée**: Si disponible
- **Pas de GitHub App confusion**: L'OAuth GitHub pour auth ≠ GitHub App pour intégration dépôts

### Questions non résolues

- ❓ **Scopes OAuth demandés** (configuré dans Clerk Dashboard)
- ❓ **Que se passe-t-il si user révoque GitHub OAuth** (probablement géré par Clerk)

---

## 4. Session Management

### Implémentation

**Gestion par Clerk**:

- JWT tokens émis par Clerk
- Tokens vérifiés par middleware Clerk Express
- Session Claims disponibles dans `req.auth`

**Structure Session**:

```typescript
req.auth = {
  userId: string,
  user: User, // Chargé depuis DB
  orgId: string,
  organization: Organization, // Chargé depuis DB
  sessionId: string,
  sessionClaims: {
    metadata: {
      isAdmin: boolean,
    },
  },
};
```

### Middleware Next.js

```typescript
const isPrivateRoute = createRouteMatcher(["/dashboard(.*)"]);

export default clerkMiddleware(async (auth, request) => {
  if (isPrivateRoute(request)) {
    await auth.protect();
  }
});
```

**Comportement**:

- Routes publiques: Pas de vérification
- Routes `/dashboard(.*)`: Require session valide
- Si pas de session: Redirection automatique vers `/sign-in`

### Règles Métier

- **Session requise pour `/dashboard`**: Toutes sous-routes protégées
- **Organization context obligatoire**: API backend require `orgId`
- **User et Org chargés à chaque requête**: Par `authMiddleware`

### Questions non résolues

- ❓ **Durée token** (définie dans Clerk Dashboard)
- ❓ **Refresh automatique** (géré par Clerk SDK)
- ❓ **Session persistante** (cookies Clerk)

---

## 5. Logout Flow

### Implémentation

**Frontend**: Likely via `<UserButton />` de Clerk ou `signOut()` method

**Code trouvé**:

- `apps/web/src/components/layout/sidebar/logout-alert-dialog.tsx` (composant exist)

**Backend**: Aucun webhook pour logout. Session invalidée côté Clerk uniquement.

### Processus

1. Utilisateur clique sur "Logout" (dans sidebar likely)
2. Frontend appelle `signOut()` de Clerk
3. Clerk invalide la session
4. Cookies Clerk cleared
5. Redirect vers page publique (likely `/sign-in`)

### Règles Métier

- **Session invalidée immédiatement**: Côté Clerk
- **Pas de soft-logout**: Session complètement terminée
- **Pas de webhook**: Backend n'est pas notifié

### Questions non résolues

- ✅ **Composant logout exists**: `logout-alert-dialog.tsx`
- ❓ **Confirmation requise** (probable vu le nom "alert-dialog")

---

## 6. Webhooks Clerk

### Webhooks Implémentés

**Endpoint**: `POST /api/clerk/webhook`

**Events gérés**:

| Event                            | Action                     | Notes                                 |
| -------------------------------- | -------------------------- | ------------------------------------- |
| `user.created`                   | Insert user dans DB        | Sync initial                          |
| `user.updated`                   | Update user dans DB        | Sync des modifications                |
| `user.deleted`                   | Soft-delete user           | `deleted = true`, `deletedAt = now()` |
| `organization.created`           | Insert org + sync members  | Fetch members via Clerk API           |
| `organization.updated`           | Update org dans DB         | Sync des modifications                |
| `organization.deleted`           | Soft-delete org            | `deleted = true`, `deletedAt = now()` |
| `organizationMembership.created` | Insert org member          | Link user ↔ org                      |
| `organizationMembership.updated` | Update org member          | Update role                           |
| `organizationMembership.deleted` | **Hard delete** org member | Suppression définitive                |

### Sécurité

**Vérification signature**:

```typescript
const evt = await verifyWebhook(req, {
  signingSecret: CLERK_WEBHOOK_SIGNING_SECRET,
});
```

**Env var requise**: `CLERK_WEBHOOK_SIGNING_SECRET`

### Gestion d'erreurs

**Cas gérés**:

- Secret manquant: Throw error
- Signature invalide: 400 + `INVALID_WEBHOOK`
- Organization inexistante lors de membership.created: Log error, skip
- Event order issues: Handled gracefully

**Logging**:

- Success: `logger.info()` avec event type
- Errors: `captureExceptionWithContext()` (Sentry)

### Particularités

**Organization.created webhook**:

- Fetch members via `clerkClient.organizations.getOrganizationMembershipList()`
- Insert members manquants en DB
- Gère le cas où events arrivent dans le désordre

**organizationMembership.deleted**:

- Hard delete (pas de soft-delete)
- Contrairement à user.deleted et organization.deleted

### Questions non résolues

- ❓ **Rate limiting webhooks** (non implémenté dans code)
- ❓ **Retry logic** (géré par Clerk)
- ❓ **Webhook queue** (synchrone pour l'instant)

---

## 7. Admin Role

### Implémentation

**Field**: `users.isAdmin` (boolean, default: false)

**Source**: `data.public_metadata.isAdmin` from Clerk

**Verification**:

```typescript
export function authIsAdminMiddleware(req, res, next) {
  if (!isAdmin(req)) {
    return res.status(401).json({ error: "UNAUTHORIZED - ADMIN ONLY" });
  }
  next();
}
```

### Règles Métier

- **isAdmin dans public_metadata**: Doit être set dans Clerk Dashboard
- **Middleware dédié**: `authIsAdminMiddleware` pour routes admin
- **Check simple**: Boolean check, pas de granular permissions

### Questions non résolues

- ❓ **Comment set isAdmin** (manuellement dans Clerk Dashboard likely)
- ❓ **Super admin existe** (pas visible dans code)
- ❓ **Admin par organization** (non, global seulement)

---

## External Integrations

### Clerk

**Services utilisés**:

- Authentication (sign-up, sign-in, sessions)
- User management
- Organization management
- Webhooks pour synchronisation

**Configuration requise**:

- Clerk Application (Dashboard)
- GitHub OAuth provider (dans Clerk)
- Webhook URL configurée
- `CLERK_WEBHOOK_SIGNING_SECRET` dans env vars

**SDKs utilisés**:

- `@clerk/nextjs` - Frontend
- `@clerk/express` - Backend
- `@clerk/express/webhooks` - Webhook verification

### GitHub (for OAuth only)

**Usage**: GitHub OAuth pour authentication (via Clerk)

**Distinction importante**:

- GitHub OAuth (auth) ≠ GitHub App (repo integration)
- Deux systèmes séparés

---

## Code Examples

### Protect API Route

```typescript
import { authMiddleware } from "@/middlewares/authMiddleware";

router.get("/protected", authMiddleware, async (req, res) => {
  // req.auth.user et req.auth.organization disponibles
  const user = req.auth.user;
  const org = req.auth.organization;
  // ...
});
```

### Protect Admin Route

```typescript
import {
  authMiddleware,
  authIsAdminMiddleware,
} from "@/middlewares/authMiddleware";

router.delete(
  "/admin/delete-user/:id",
  authMiddleware,
  authIsAdminMiddleware,
  async (req, res) => {
    // Only admins can access
  }
);
```

### Frontend: Check Auth

```typescript
import { auth } from "@clerk/nextjs/server";

export default async function Page() {
  const { userId, orgId } = await auth();

  if (!userId) {
    redirect("/sign-in");
  }

  // Render protected content
}
```

---

## Testing

**Test mocks disponibles**:

```typescript
// Mock authenticated user
mockAuthMiddleware(isAdmin: boolean = false)

// Mock unauthenticated
mockAuthPublicMiddleware()
```

---

## Gaps & Unknowns

### Configuration Clerk (External)

❓ Les éléments suivants sont configurés dans Clerk Dashboard (pas visible dans code):

- Providers activés (GitHub OAuth + Email/Password ?)
- MFA settings
- Session duration
- Token refresh strategy
- Password requirements (si email/password activé)
- Email templates
- Redirect URLs
- Rate limiting

### Missing Features

❓ **Forgot Password**: Pas de code custom, géré par Clerk
❓ **Email Verification**: Géré par Clerk
❓ **Two-Factor Auth**: Configurable dans Clerk Dashboard
❓ **Session Logging**: Pas d'audit trail des connexions

### Future Improvements

💡 **Suggestions**:

- Add session activity logging
- Track login attempts (for security)
- Add "last login" field to users table
- Add user activity audit trail
- Consider role-based permissions beyond isAdmin boolean

---

## References

### Main Files

- `apps/web/src/middleware.ts` - Route protection
- `apps/api/src/routes/clerk.ts` - Webhooks (253 lignes)
- `apps/api/src/middlewares/authMiddleware.ts` - Auth verification
- `apps/web/src/app/(public)/(auth)/sign-in/[[...sign-in]]/page.tsx`
- `apps/web/src/app/(public)/(auth)/sign-up/[[...sign-up]]/page.tsx`
- `apps/web/src/components/layout/sidebar/logout-alert-dialog.tsx`

### Schema

- `apps/api/src/db/schema.ts` - users table (lines 11-22)

### Types

- `packages/types/src/user.ts` - User types

### External Documentation

- Clerk Docs: https://clerk.com/docs
- Clerk Webhooks: https://clerk.com/docs/integrations/webhooks/overview
- Clerk Next.js: https://clerk.com/docs/quickstarts/nextjs

---

**Status**: ✅ RECHERCHE COMPLÉTÉE  
**Coverage**: ~85% (85% implemented, 15% external Clerk config)  
**Files Analyzed**: 6  
**LOC Reviewed**: ~400

