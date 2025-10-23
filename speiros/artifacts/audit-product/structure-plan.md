# Structure Plan - Product Context Documentation

## Vue d'ensemble

Ce plan définit la structure complète de la documentation produit qui sera créée dans `context/product/`.

**Approche** : Documentation par domaine, avec un fichier README.md principal par domaine et des fichiers détaillés par fonctionnalité.

**Total** : 7 domaines, ~50 fichiers à créer

---

## Structure des Dossiers

```
context/product/
├── README.md                           # Index global + guide de navigation
│
├── authentication/
│   ├── README.md                       # [PRIORITY: HIGH]
│   ├── sign-up-flow.md
│   ├── sign-in-flow.md
│   ├── github-oauth.md
│   ├── session-management.md
│   ├── logout-flow.md
│   └── webhooks-clerk.md
│
├── user-management/
│   ├── README.md                       # [PRIORITY: HIGH]
│   ├── user-profile.md
│   ├── user-roles.md
│   ├── user-lifecycle.md
│   └── user-deletion.md
│
├── organization/
│   ├── README.md                       # [PRIORITY: HIGH]
│   ├── organization-creation.md
│   ├── organization-members.md
│   ├── organization-roles.md
│   ├── organization-plans.md
│   ├── organization-selector.md
│   ├── organization-lifecycle.md
│   └── multi-tenancy.md
│
├── github-integration/
│   ├── README.md                       # [PRIORITY: HIGH]
│   ├── github-app-setup.md
│   ├── installation-flow.md
│   ├── repository-access.md
│   ├── repository-selection.md
│   ├── repository-sync.md
│   └── github-webhooks.md
│
├── repository-dashboard/
│   ├── README.md                       # [PRIORITY: MEDIUM]
│   ├── dashboard-layout.md
│   ├── repository-list.md
│   ├── repository-switcher.md
│   ├── repository-navigation.md
│   ├── readme-view.md
│   ├── about-view.md
│   └── admin-view.md
│
├── markdown-editor/
│   ├── README.md                       # [PRIORITY: LOW - EN DÉVELOPPEMENT]
│   ├── file-explorer.md
│   ├── content-editor.md
│   ├── ai-chat.md
│   ├── file-operations.md
│   ├── drag-and-drop.md
│   ├── auto-save.md
│   ├── ai-actions.md
│   ├── panel-resize.md
│   └── markdown-format.md
│
└── administration/
    ├── README.md                       # [PRIORITY: MEDIUM]
    ├── admin-roles.md
    ├── admin-routes.md
    ├── repository-admin.md
    └── system-management.md
```

---

## Détails par Domaine

### 1. Authentication (Priorité: HIGH)

**Dossier** : `context/product/authentication/`

**Fichiers à créer** :

| Fichier                 | Description                     | Recherche nécessaire                |
| ----------------------- | ------------------------------- | ----------------------------------- |
| `README.md`             | Vue d'ensemble authentification | ✅ Routes Clerk, Middleware Next.js |
| `sign-up-flow.md`       | Processus d'inscription complet | ✅ Clerk config, webhooks           |
| `sign-in-flow.md`       | Processus de connexion          | ✅ Clerk routes, sessions           |
| `github-oauth.md`       | OAuth GitHub intégration        | ✅ Clerk GitHub provider            |
| `session-management.md` | Gestion sessions et tokens      | ✅ Middleware auth, Clerk JWT       |
| `logout-flow.md`        | Processus de déconnexion        | ✅ Clerk logout, UI                 |
| `webhooks-clerk.md`     | Synchronisation via webhooks    | ✅ `/routes/clerk.ts`               |

**Information à rechercher** :

- Configuration Clerk exacte (providers, settings)
- Webhooks implémentés (user.created, user.updated, etc.)
- Gestion des erreurs d'authentification
- Durée de session et refresh
- MFA configuré ou non

**Fichiers sources clés** :

- `apps/web/src/middleware.ts`
- `apps/api/src/routes/clerk.ts`
- `apps/api/src/middlewares/authMiddleware.ts`
- `.env.sample` files

---

### 2. User Management (Priorité: HIGH)

**Dossier** : `context/product/user-management/`

**Fichiers à créer** :

| Fichier             | Description                         | Recherche nécessaire          |
| ------------------- | ----------------------------------- | ----------------------------- |
| `README.md`         | Vue d'ensemble gestion utilisateurs | ✅ Schema users, routes       |
| `user-profile.md`   | Gestion du profil utilisateur       | ✅ User fields, édition       |
| `user-roles.md`     | Système de rôles (admin/user)       | ✅ isAdmin field, permissions |
| `user-lifecycle.md` | Cycle de vie utilisateur            | ✅ Création, sync, updates    |
| `user-deletion.md`  | Soft-delete et suppression          | ✅ Deleted flag, deletedAt    |

**Information à rechercher** :

- Champs éditables du profil
- Validation des données utilisateur
- Processus de soft-delete
- Rôles et permissions admin
- Historique d'activité (si existe)

**Fichiers sources clés** :

- `apps/api/src/db/schema.ts` (users table)
- `apps/api/src/routes/users.ts`
- `packages/types/src/user.ts`

---

### 3. Organization (Priorité: HIGH)

**Dossier** : `context/product/organization/`

**Fichiers à créer** :

| Fichier                     | Description                  | Recherche nécessaire         |
| --------------------------- | ---------------------------- | ---------------------------- |
| `README.md`                 | Vue d'ensemble organisations | ✅ Schema, multi-tenant      |
| `organization-creation.md`  | Création d'organisation      | ✅ Qui peut créer, processus |
| `organization-members.md`   | Gestion des membres          | ✅ Ajout, retrait, rôles     |
| `organization-roles.md`     | Rôles au niveau organisation | ✅ Permissions par rôle      |
| `organization-plans.md`     | Plans (free/pro/enterprise)  | ✅ Limites, changement       |
| `organization-selector.md`  | Sélecteur d'organisation UI  | ✅ Composant, switching      |
| `organization-lifecycle.md` | Cycle de vie organisation    | ✅ Création, update, delete  |
| `multi-tenancy.md`          | Architecture multi-tenant    | ✅ Scoping, isolation        |

**Information à rechercher** :

- Rôles disponibles et permissions exactes
- Limites par plan (membres, dépôts, etc.)
- Système de facturation (si existe)
- Processus d'invitation de membres
- Gestion soft-delete organisations

**Fichiers sources clés** :

- `apps/api/src/db/schema.ts` (organizations, organization_members)
- `apps/api/src/routes/organization.ts`
- `apps/web/src/app/(protected)/dashboard/org-selector.tsx`
- `packages/types/src/organization.ts`

---

### 4. GitHub Integration (Priorité: HIGH)

**Dossier** : `context/product/github-integration/`

**Fichiers à créer** :

| Fichier                   | Description                       | Recherche nécessaire   |
| ------------------------- | --------------------------------- | ---------------------- |
| `README.md`               | Vue d'ensemble intégration GitHub | ✅ GitHub App, Octokit |
| `github-app-setup.md`     | Configuration GitHub App          | ✅ Permissions, scopes |
| `installation-flow.md`    | Flux d'installation GitHub        | ✅ UI flow, callback   |
| `repository-access.md`    | Gestion permissions dépôts        | ✅ API GitHub, access  |
| `repository-selection.md` | Ajout/suppression de dépôts       | ✅ Routes, UI          |
| `repository-sync.md`      | Synchronisation données GitHub    | ✅ Polling, webhooks   |
| `github-webhooks.md`      | Webhooks GitHub (si impl.)        | ✅ Events, handlers    |

**Information à rechercher** :

- Scopes/permissions GitHub App
- Webhooks implémentés (push, PR, etc.)
- Synchronisation (manuelle/auto/webhooks)
- Gestion révocation installation
- Différence public/private repos

**Fichiers sources clés** :

- `apps/api/src/services/github.ts`
- `apps/api/src/routes/github.ts`
- `apps/api/src/db/schema.ts` (github_installations, github_installation_repositories)
- `apps/web/src/components/github/`
- `packages/types/src/github.ts`

---

### 5. Repository Dashboard (Priorité: MEDIUM)

**Dossier** : `context/product/repository-dashboard/`

**Fichiers à créer** :

| Fichier                    | Description                     | Recherche nécessaire  |
| -------------------------- | ------------------------------- | --------------------- |
| `README.md`                | Vue d'ensemble dashboard dépôts | ✅ Layout, navigation |
| `dashboard-layout.md`      | Structure du dashboard          | ✅ Sidebar, layout    |
| `repository-list.md`       | Vue liste des dépôts            | ✅ Affichage, données |
| `repository-switcher.md`   | Sélecteur de dépôt              | ✅ Composant, UX      |
| `repository-navigation.md` | Navigation par dépôt            | ✅ Routes, tabs       |
| `readme-view.md`           | Page README                     | ✅ Affichage README   |
| `about-view.md`            | Page About                      | ✅ Infos dépôt        |
| `admin-view.md`            | Page Admin dépôt                | ✅ Actions admin      |

**Information à rechercher** :

- Fonctionnalités de filtrage/tri/recherche
- Statistiques affichées (commits, issues, etc.)
- Favoris ou archivage
- Permissions au niveau dépôt
- Source des données (API GitHub directe ou cache)

**Fichiers sources clés** :

- `apps/web/src/app/(protected)/dashboard/`
- `apps/web/src/components/github/repositories-switcher.tsx`
- `apps/web/src/lib/data/github/`
- `apps/web/src/lib/route-formatter.ts`

---

### 6. Markdown Editor (Priorité: LOW - EN DÉVELOPPEMENT)

**Dossier** : `context/product/markdown-editor/`

**Fichiers à créer** :

| Fichier              | Description                 | Recherche nécessaire      |
| -------------------- | --------------------------- | ------------------------- |
| `README.md`          | Vue d'ensemble éditeur      | ✅ User Story 1.1         |
| `file-explorer.md`   | Panel gauche (arborescence) | ✅ react-arborist, CRUD   |
| `content-editor.md`  | Panel central (éditeur)     | ✅ TipTap, auto-save      |
| `ai-chat.md`         | Panel droit (assistant IA)  | ✅ Backend IA, actions    |
| `file-operations.md` | CRUD fichiers/dossiers      | ✅ API, permissions       |
| `drag-and-drop.md`   | Drag & drop                 | ✅ react-arborist         |
| `auto-save.md`       | Sauvegarde automatique      | ✅ Debounce, TanStack     |
| `ai-actions.md`      | Actions IA (modifier/créer) | ✅ API IA, parsing        |
| `panel-resize.md`    | Redimensionnement panels    | ✅ react-resizable-panels |
| `markdown-format.md` | Format et conventions MD    | ✅ Extensions TipTap      |

**Information à rechercher** :

- Stockage des fichiers (PostgreSQL, S3, GitHub ?)
- Backend IA utilisé (OpenAI, Anthropic, autre ?)
- Versioning/historique fichiers
- Collaboration temps réel
- Implémentation actuelle (code existe ?)

**Fichiers sources clés** :

- `/roadmap/user-stories/user-story-1.1-frontend-triple-panel.md`
- `/roadmap/user-stories/user-story-1.2-backend-markdown-api.md` (si existe)
- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/markdown-editor/` (si existe)
- API routes pour markdown (à chercher)

---

### 7. Administration (Priorité: MEDIUM)

**Dossier** : `context/product/administration/`

**Fichiers à créer** :

| Fichier                | Description                   | Recherche nécessaire   |
| ---------------------- | ----------------------------- | ---------------------- |
| `README.md`            | Vue d'ensemble administration | ✅ Routes admin, rôles |
| `admin-roles.md`       | Rôles et permissions admin    | ✅ isAdmin, checks     |
| `admin-routes.md`      | Routes admin disponibles      | ✅ /admin routes       |
| `repository-admin.md`  | Administration par dépôt      | ✅ /repos/[id]/admin   |
| `system-management.md` | Gestion système (si appl.)    | ✅ Métriques, logs     |

**Information à rechercher** :

- Actions admin disponibles
- Vérifications d'autorisation
- Dashboard admin global (si existe)
- Logs d'audit
- Métriques système

**Fichiers sources clés** :

- `apps/api/src/routes/admin.ts`
- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/admin/page.tsx`
- `apps/api/src/middlewares/authMiddleware.ts` (vérif admin)

---

## Assignments pour Sub-Context-Agent

### Assignment 1: Authentication Domain

**Topic**: Authentication and Session Management  
**Priority**: HIGH  
**Target Folder**: `context/product/authentication/`

**Research Instructions**:

1. Analyze Clerk configuration and webhooks implementation
2. Document sign-up, sign-in, logout flows
3. Extract session management logic
4. Identify all authentication-related business rules
5. Map GitHub OAuth integration details

**Key Files to Analyze**:

- `apps/web/src/middleware.ts`
- `apps/api/src/routes/clerk.ts`
- `apps/api/src/middlewares/authMiddleware.ts`

**Expected Outputs**: 7 markdown files

---

### Assignment 2: User Management Domain

**Topic**: User Lifecycle and Profile Management  
**Priority**: HIGH  
**Target Folder**: `context/product/user-management/`

**Research Instructions**:

1. Analyze users table schema and relations
2. Document user profile fields and editability
3. Extract user roles and permissions logic
4. Document soft-delete mechanism
5. Identify user validation rules

**Key Files to Analyze**:

- `apps/api/src/db/schema.ts` (users table)
- `apps/api/src/routes/users.ts`
- `packages/types/src/user.ts`

**Expected Outputs**: 5 markdown files

---

### Assignment 3: Organization Domain

**Topic**: Multi-tenant Organizations and Members  
**Priority**: HIGH  
**Target Folder**: `context/product/organization/`

**Research Instructions**:

1. Analyze organizations and organization_members schemas
2. Document organization creation and lifecycle
3. Extract member management logic and roles
4. Identify plan system (free/pro/enterprise) implementation
5. Document multi-tenancy architecture and scoping
6. Analyze organization selector UI component

**Key Files to Analyze**:

- `apps/api/src/db/schema.ts` (organizations, organization_members)
- `apps/api/src/routes/organization.ts`
- `apps/web/src/app/(protected)/dashboard/org-selector.tsx`
- `packages/types/src/organization.ts`

**Expected Outputs**: 8 markdown files

---

### Assignment 4: GitHub Integration Domain

**Topic**: GitHub App Integration and Repository Management  
**Priority**: HIGH  
**Target Folder**: `context/product/github-integration/`

**Research Instructions**:

1. Analyze GitHub service implementation (Octokit usage)
2. Document installation flow and permissions
3. Extract repository selection and management logic
4. Identify synchronization mechanism (polling/webhooks)
5. Document GitHub webhooks if implemented
6. Map all GitHub API calls and their purposes

**Key Files to Analyze**:

- `apps/api/src/services/github.ts`
- `apps/api/src/routes/github.ts`
- `apps/api/src/db/schema.ts` (github_installations, github_installation_repositories)
- `apps/web/src/components/github/`
- `packages/types/src/github.ts`

**Expected Outputs**: 7 markdown files

---

### Assignment 5: Repository Dashboard Domain

**Topic**: Dashboard UI and Repository Visualization  
**Priority**: MEDIUM  
**Target Folder**: `context/product/repository-dashboard/`

**Research Instructions**:

1. Analyze dashboard layout and structure
2. Document repository list rendering and data flow
3. Extract repository switcher component logic
4. Map repository navigation routes and tabs
5. Identify README, About, Admin view implementations
6. Check for filtering, sorting, search features

**Key Files to Analyze**:

- `apps/web/src/app/(protected)/dashboard/page.tsx`
- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/`
- `apps/web/src/components/github/repositories-switcher.tsx`
- `apps/web/src/lib/data/github/`
- `apps/web/src/lib/route-formatter.ts`

**Expected Outputs**: 8 markdown files

---

### Assignment 6: Markdown Editor Domain

**Topic**: Triple Panel Markdown Editor (Planned/In Development)  
**Priority**: LOW  
**Target Folder**: `context/product/markdown-editor/`

**Research Instructions**:

1. Read User Story 1.1 completely
2. Check if any implementation exists (components, routes, API)
3. Document planned architecture from user story
4. Identify file storage strategy (if mentioned in code)
5. Identify AI backend (if any config exists)
6. Document planned vs. implemented features
7. Mark everything clearly as PLANNED or IMPLEMENTED

**Key Files to Analyze**:

- `/roadmap/user-stories/user-story-1.1-frontend-triple-panel.md`
- `/roadmap/user-stories/user-story-1.2-backend-markdown-api.md` (if exists)
- Search for: `markdown-editor`, `triple-panel`, `ai-chat` in codebase

**Expected Outputs**: 10 markdown files (mostly PLANNED status)

---

### Assignment 7: Administration Domain

**Topic**: Admin Roles and System Management  
**Priority**: MEDIUM  
**Target Folder**: `context/product/administration/`

**Research Instructions**:

1. Analyze admin routes and available actions
2. Document admin authorization checks
3. Extract repository admin page functionality
4. Check for system-wide admin dashboard
5. Identify audit logging if implemented
6. Document admin-specific business rules

**Key Files to Analyze**:

- `apps/api/src/routes/admin.ts`
- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/admin/page.tsx`
- `apps/api/src/middlewares/authMiddleware.ts`
- Search for: `isAdmin`, `admin` checks in codebase

**Expected Outputs**: 5 markdown files

---

## Timeline Estimation

| Domaine              | Priority | Files | Est. Time | Cumulative |
| -------------------- | -------- | ----- | --------- | ---------- |
| Authentication       | HIGH     | 7     | 15 min    | 15 min     |
| User Management      | HIGH     | 5     | 10 min    | 25 min     |
| Organization         | HIGH     | 8     | 20 min    | 45 min     |
| GitHub Integration   | HIGH     | 7     | 20 min    | 65 min     |
| Repository Dashboard | MEDIUM   | 8     | 15 min    | 80 min     |
| Administration       | MEDIUM   | 5     | 10 min    | 90 min     |
| Markdown Editor      | LOW      | 10    | 15 min    | 105 min    |

**Total Estimated Time**: ~105 minutes (~1h45)

---

## Success Criteria

✅ All 7 domain folders created  
✅ 50 markdown files generated  
✅ Each file follows product-context-template structure  
✅ Information extracted from code sources with references  
✅ PLANNED vs. IMPLEMENTED clearly marked  
✅ Unknown/missing information flagged for future research  
✅ Cross-references between domains documented

---

**Date de création**: 2025-10-17  
**Workflow**: audit-codebase-product  
**Étape**: 4/7 - Create Product Context Structure  
**Statut**: ✅ Plan créé, prêt pour délégation

