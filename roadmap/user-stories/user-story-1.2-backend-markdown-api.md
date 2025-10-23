# **Story 1.2-Backend : API pour Gestion de Fichiers Markdown**

## **Status**
Draft

---

## **Story**

**En tant que** backend de l'application,  
**Je veux** une API REST pour gérer les fichiers Markdown et le chat IA,  
**Afin de** permettre au frontend de créer, éditer, organiser la documentation et d'interagir avec un assistant IA.

---

## **Acceptance Criteria**

1. API REST pour CRUD des fichiers/dossiers Markdown (GET, POST, PUT, DELETE)
2. Endpoint pour récupérer l'arborescence complète des fichiers
3. Endpoint pour déplacer/renommer des fichiers
4. API de chat avec assistant IA capable de retourner des actions structurées
5. Stockage des fichiers en base de données PostgreSQL avec Drizzle ORM
6. Validation des entrées avec Zod
7. Authentification et autorisation via Clerk middleware existant
8. L'API respecte les guidelines du projet (TypeScript strict, req.tx, logging structuré)
9. Tests unitaires et d'intégration pour tous les endpoints

---

## **Tasks / Subtasks**

- [ ] **Task 1: Définir le schéma de base de données** (AC: 5, 8)
  - [ ] Créer `src/db/schema/markdown-files.ts` pour la table `markdown_files`
  - [ ] Créer `src/db/schema/markdown-folders.ts` pour la table `markdown_folders`
  - [ ] Créer les relations entre files, folders, repositories et organizations
  - [ ] Ajouter les index nécessaires pour les performances
  - [ ] Créer la migration Drizzle

- [ ] **Task 2: Créer les types et validateurs Zod** (AC: 6, 8)
  - [ ] Créer `packages/types/src/markdown-file.ts` avec types et schémas Zod
  - [ ] Créer `packages/types/src/markdown-folder.ts` avec types et schémas Zod
  - [ ] Créer `packages/types/src/ai-chat.ts` avec types et schémas Zod
  - [ ] Exporter tous les types dans `packages/types/src/index.ts`

- [ ] **Task 3: Créer les routes de gestion des fichiers** (AC: 1, 7, 8)
  - [ ] Créer `src/routes/markdown-files.ts`
  - [ ] Implémenter `GET /api/repositories/:repositoryId/markdown-files` (liste)
  - [ ] Implémenter `GET /api/repositories/:repositoryId/markdown-files/:fileId` (détail)
  - [ ] Implémenter `POST /api/repositories/:repositoryId/markdown-files` (création)
  - [ ] Implémenter `PUT /api/repositories/:repositoryId/markdown-files/:fileId` (mise à jour)
  - [ ] Implémenter `DELETE /api/repositories/:repositoryId/markdown-files/:fileId` (suppression)
  - [ ] Ajouter middleware d'authentification sur toutes les routes
  - [ ] Ajouter validation Zod sur les body/params

- [ ] **Task 4: Créer les routes de gestion des dossiers** (AC: 1, 7, 8)
  - [ ] Implémenter `GET /api/repositories/:repositoryId/markdown-folders` (liste)
  - [ ] Implémenter `POST /api/repositories/:repositoryId/markdown-folders` (création)
  - [ ] Implémenter `PUT /api/repositories/:repositoryId/markdown-folders/:folderId` (mise à jour)
  - [ ] Implémenter `DELETE /api/repositories/:repositoryId/markdown-folders/:folderId` (suppression)
  - [ ] Ajouter middleware d'authentification sur toutes les routes
  - [ ] Ajouter validation Zod sur les body/params

- [ ] **Task 5: Créer l'endpoint d'arborescence** (AC: 2, 8)
  - [ ] Implémenter `GET /api/repositories/:repositoryId/markdown-tree`
  - [ ] Retourner une structure hiérarchique complète (folders + files)
  - [ ] Optimiser la requête SQL pour éviter les N+1
  - [ ] Ajouter le cache si nécessaire

- [ ] **Task 6: Créer l'endpoint de déplacement/renommage** (AC: 3, 8)
  - [ ] Implémenter `PATCH /api/repositories/:repositoryId/markdown-files/:fileId/move`
  - [ ] Implémenter `PATCH /api/repositories/:repositoryId/markdown-files/:fileId/rename`
  - [ ] Implémenter `PATCH /api/repositories/:repositoryId/markdown-folders/:folderId/move`
  - [ ] Implémenter `PATCH /api/repositories/:repositoryId/markdown-folders/:folderId/rename`
  - [ ] Valider que le déplacement est valide (pas de cycle pour les dossiers)
  - [ ] Gérer les transactions pour garantir la cohérence

- [ ] **Task 7: Créer l'API de chat IA** (AC: 4, 8)
  - [ ] Créer `src/routes/ai-chat.ts`
  - [ ] Implémenter `POST /api/repositories/:repositoryId/ai-chat/message`
  - [ ] Intégrer un service IA (OpenAI, Anthropic, etc.)
  - [ ] Parser les réponses IA pour extraire les actions structurées
  - [ ] Retourner un format JSON standardisé avec message + actions
  - [ ] Ajouter gestion d'erreur robuste pour les appels IA

- [ ] **Task 8: Créer les services métier** (AC: 8)
  - [ ] Créer `src/services/markdown-file-service.ts` pour logique métier fichiers
  - [ ] Créer `src/services/markdown-folder-service.ts` pour logique métier dossiers
  - [ ] Créer `src/services/ai-chat-service.ts` pour logique IA
  - [ ] Implémenter la validation des permissions (user appartient à l'org du repository)
  - [ ] Assurer que toutes les fonctions sont pures quand possible

- [ ] **Task 9: Ajouter les routes dans l'index principal** (AC: 8)
  - [ ] Ajouter `markdownFilesRouter` dans `src/routes/index.ts`
  - [ ] Ajouter `markdownFoldersRouter` dans `src/routes/index.ts`
  - [ ] Ajouter `aiChatRouter` dans `src/routes/index.ts`
  - [ ] Assurer que les middlewares sont appliqués correctement

- [ ] **Task 10: Implémenter les permissions** (AC: 7)
  - [ ] Vérifier que l'utilisateur a accès au repository
  - [ ] Vérifier que le repository appartient à l'organisation de l'utilisateur
  - [ ] Créer un middleware `checkRepositoryAccess` réutilisable
  - [ ] Ajouter des tests pour les cas d'accès non autorisé

- [ ] **Task 11: Ajouter le logging structuré** (AC: 8)
  - [ ] Logger toutes les opérations CRUD avec métadonnées
  - [ ] Logger les appels IA avec durée et tokens utilisés
  - [ ] Logger les erreurs avec contexte complet
  - [ ] Utiliser le logger existant du projet (pino)

- [ ] **Task 12: Tests et documentation** (AC: 9)
  - [ ] Écrire tests unitaires pour les services
  - [ ] Écrire tests d'intégration pour tous les endpoints
  - [ ] Tester les cas d'erreur (fichier inexistant, permissions, etc.)
  - [ ] Tester les transactions et rollbacks
  - [ ] Documenter l'API avec JSDoc
  - [ ] Créer un README.md pour les endpoints

---

## **Dev Notes**

### **Architecture et Contraintes Techniques**

Ce backend doit être développé dans `apps/api` en suivant strictement les guidelines du fichier `.cursorrules`.

#### **Stack Technique Imposé**

**Existant dans le projet :**
- Node.js 22.14.0 avec TypeScript strict
- Express 4.21.2
- Drizzle ORM 0.41.0 avec PostgreSQL
- Clerk Express 1.7.5 pour l'authentification
- Pino pour le logging
- Zod pour la validation

**À installer (si nécessaire) :**
- SDK OpenAI ou Anthropic pour l'assistant IA
- Bibliothèques de parsing si nécessaire

#### **Principes de Code**

1. **TypeScript strict** : Pas de type `any`, validation runtime avec Zod
2. **Programmation fonctionnelle** : Pas de classes JavaScript, utiliser fonctions et closures
3. **Fonctions pures** quand possible, structures de données immutables
4. **Principe de responsabilité unique** pour chaque fonction/service
5. **Code auto-documenté** avec JSDoc pour les APIs publiques
6. **Gestion d'erreur complète** avec messages clairs et codes HTTP appropriés
7. **Logging structuré** avec pino pour faciliter le debug

#### **Conventions de Nommage**

- **PascalCase** : Types, Interfaces
- **camelCase** : Functions, Variables, Properties, Router exports
- **kebab-case** : Files, Folders
- **SCREAMING_SNAKE_CASE** : Constants, Error codes

#### **Database Access**

**OBLIGATOIRE** : Toutes les opérations de base de données DOIVENT utiliser `req.tx`

✅ **BONNE pratique** :
```typescript
export const markdownFilesRouter = Router();

markdownFilesRouter.post('/', async (req, res) => {
  try {
    // req.tx provides managed database access
    const result = await req.tx.insert(markdownFiles).values(data).returning();
    res.json(result);
  } catch (error) {
    logger.error({
      msg: 'Failed to create markdown file',
      event: 'markdown-file-create-error',
      metadata: { error },
    });
    res.status(500).json({ message: 'MARKDOWN_FILE_CREATE_FAILED' });
  }
});
```

❌ **MAUVAISE pratique** :
```typescript
// Ne jamais utiliser pool.connect() ou db directement
const client = await pool.connect();
```

#### **Router Guidelines**

- Export named router instance using camelCase
- Use async/await with proper error handling
- Always use req.tx for database operations
- Implement structured logging for errors
- Return consistent JSON responses
- Use uppercase error codes in responses
- Group related endpoints in dedicated router files
- Use typed request/response objects

#### **Structure de Fichiers**

```
apps/api/src/
├── db/
│   └── schema/
│       ├── markdown-files.ts
│       └── markdown-folders.ts
├── routes/
│   ├── index.ts (mise à jour)
│   ├── markdown-files.ts
│   ├── markdown-folders.ts
│   └── ai-chat.ts
├── services/
│   ├── markdown-file-service.ts
│   ├── markdown-folder-service.ts
│   └── ai-chat-service.ts
├── middlewares/
│   └── check-repository-access.ts
└── tests/
    └── routes/
        ├── markdown-files.test.ts
        ├── markdown-folders.test.ts
        └── ai-chat.test.ts

packages/types/src/
├── markdown-file.ts
├── markdown-folder.ts
└── ai-chat.ts
```

#### **Schéma de Base de Données**

**Table `markdown_folders`** :
```typescript
export const markdownFolders = pgTable('markdown_folders', {
  id: text('id').primaryKey().$defaultFn(() => generateShortId()),
  repositoryId: text('repository_id').notNull().references(() => repositories.id),
  parentId: text('parent_id').references((): AnyPgColumn => markdownFolders.id),
  name: text('name').notNull(),
  path: text('path').notNull(), // Chemin complet pour faciliter les requêtes
  deleted: boolean('deleted').notNull().default(false),
  deletedAt: timestamp('deleted_at'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
});
```

**Table `markdown_files`** :
```typescript
export const markdownFiles = pgTable('markdown_files', {
  id: text('id').primaryKey().$defaultFn(() => generateShortId()),
  repositoryId: text('repository_id').notNull().references(() => repositories.id),
  folderId: text('folder_id').references(() => markdownFolders.id),
  name: text('name').notNull(),
  path: text('path').notNull(), // Chemin complet
  content: text('content').notNull().default(''),
  deleted: boolean('deleted').notNull().default(false),
  deletedAt: timestamp('deleted_at'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
});
```

**Index** :
- Index sur `repository_id` pour les deux tables
- Index sur `parent_id` pour `markdown_folders`
- Index sur `folder_id` pour `markdown_files`
- Index composite sur `repository_id, deleted` pour filtrer efficacement

#### **Validation Zod**

Exemples de schémas :

```typescript
// packages/types/src/markdown-file.ts
export const markdownFileSchema = z.object({
  id: z.string(),
  repositoryId: z.string(),
  folderId: z.string().nullable(),
  name: z.string().min(1).max(255),
  path: z.string(),
  content: z.string(),
  deleted: z.boolean(),
  deletedAt: z.date().nullable(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

export const createMarkdownFileSchema = z.object({
  name: z.string().min(1).max(255).regex(/\.md$/),
  folderId: z.string().nullable(),
  content: z.string().default(''),
});

export const updateMarkdownFileSchema = z.object({
  content: z.string(),
});

export type MarkdownFile = z.infer<typeof markdownFileSchema>;
export type CreateMarkdownFile = z.infer<typeof createMarkdownFileSchema>;
export type UpdateMarkdownFile = z.infer<typeof updateMarkdownFileSchema>;
```

#### **Format de Réponse API Chat IA**

```typescript
type AIChatResponse = {
  message: string; // Réponse textuelle de l'IA
  actions: AIAction[]; // Actions à exécuter
};

type AIAction = 
  | { type: 'modify_file'; fileId: string; content: string; explanation: string }
  | { type: 'create_file'; folderId: string | null; name: string; content: string; explanation: string }
  | { type: 'delete_file'; fileId: string; explanation: string };
```

Le backend doit parser la réponse de l'IA (potentiellement en JSON ou en utilisant function calling) et retourner un format standardisé.

#### **Gestion des Permissions**

Middleware à créer :

```typescript
export async function checkRepositoryAccess(
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) {
  try {
    const { repositoryId } = req.params;
    const userId = req.auth.userId;
    
    // Vérifier que le repository existe et appartient à une org de l'utilisateur
    const repository = await req.tx
      .select()
      .from(repositories)
      .where(eq(repositories.id, repositoryId))
      .limit(1);
    
    if (!repository.length) {
      return res.status(404).json({ message: 'REPOSITORY_NOT_FOUND' });
    }
    
    // Vérifier l'appartenance à l'organisation
    const membership = await req.tx
      .select()
      .from(organizationMembers)
      .where(
        and(
          eq(organizationMembers.userId, userId),
          eq(organizationMembers.organizationId, repository[0].organizationId)
        )
      )
      .limit(1);
    
    if (!membership.length) {
      return res.status(403).json({ message: 'ACCESS_DENIED' });
    }
    
    next();
  } catch (error) {
    logger.error({
      msg: 'Repository access check failed',
      event: 'repository-access-check-error',
      metadata: { error },
    });
    res.status(500).json({ message: 'ACCESS_CHECK_FAILED' });
  }
}
```

#### **Performance et Optimisation**

- Utiliser des index sur les colonnes fréquemment requêtées
- Éviter les requêtes N+1 pour l'arborescence (utiliser une seule requête avec jointures)
- Mettre en cache l'arborescence si elle devient volumineuse
- Limiter la taille des contenus markdown (ex: max 1MB par fichier)

#### **Sécurité**

- Valider que les noms de fichiers ne contiennent pas de caractères dangereux
- Limiter la profondeur de l'arborescence (ex: max 10 niveaux)
- Rate limiting sur les endpoints IA pour éviter l'abus
- Sanitizer les inputs pour éviter les injections

#### **Logging Structuré**

Tous les logs doivent suivre ce format :

```typescript
logger.info({
  msg: 'Markdown file created successfully',
  event: 'markdown-file-created',
  metadata: {
    fileId: result.id,
    repositoryId,
    userId: req.auth.userId,
  },
});

logger.error({
  msg: 'Failed to create markdown file',
  event: 'markdown-file-create-error',
  metadata: {
    error: error.message,
    stack: error.stack,
    repositoryId,
    userId: req.auth.userId,
  },
});
```

### **Testing**

#### **Localisation des Tests**
- Tests unitaires : `apps/api/src/tests/unit/`
- Tests d'intégration : `apps/api/src/tests/routes/`

#### **Standards de Test**
- Utiliser Vitest pour tous les tests
- Utiliser Supertest pour les tests d'endpoints
- Utiliser TestContainers pour la base de données de test
- Couverture minimale : 80% pour les services et routes
- Tests de tous les cas d'erreur et edge cases

#### **Frameworks et Patterns**
- Vitest avec configuration TypeScript existante
- Supertest pour les requêtes HTTP
- TestContainers PostgreSQL pour l'isolation des tests
- Fixtures pour les données de test

#### **Requirements Spécifiques**
- Tester tous les endpoints avec et sans authentification
- Tester les permissions (accès non autorisé)
- Tester les validations Zod
- Tester les transactions et rollbacks
- Tester la gestion des dossiers (cycle, profondeur)
- Tester l'API IA avec des mocks

### **Intégration IA**

Pour la première version, utiliser OpenAI GPT-4 ou Claude :

1. **Prompt Engineering** : Créer un prompt système qui demande à l'IA de retourner un JSON structuré avec le message et les actions
2. **Function Calling** : Utiliser la fonctionnalité de function calling d'OpenAI pour extraire les actions
3. **Parsing** : Parser la réponse et valider avec Zod avant de retourner au frontend

Exemple de prompt système :
```
Tu es un assistant de documentation Markdown. Quand l'utilisateur te demande de modifier
des fichiers, tu dois retourner une réponse JSON avec :
- message: ta réponse textuelle
- actions: un tableau d'actions à exécuter

Actions possibles :
- modify_file: { type: 'modify_file', fileId: string, content: string, explanation: string }
- create_file: { type: 'create_file', folderId: string | null, name: string, content: string, explanation: string }
- delete_file: { type: 'delete_file', fileId: string, explanation: string }
```

### **Migration Database**

Créer une migration Drizzle :

```bash
npm run db:generate
```

Cela créera un fichier SQL dans `apps/api/drizzle/`.

### **Notes Importantes**

1. **Transactions** : Le middleware `withTransaction()` existant gère déjà les transactions pour les POST/PUT/PATCH/DELETE. Utiliser `req.tx` partout.

2. **Soft Delete** : Utiliser le champ `deleted` pour les suppressions logiques, jamais de DELETE SQL réel.

3. **Path Management** : Le champ `path` doit être maintenu à jour lors des déplacements/renommages pour faciliter les requêtes.

4. **Cascade Delete** : Lors de la suppression d'un dossier, soft-delete tous les enfants (récursif).

5. **Validation Noms** : Les noms de fichiers doivent se terminer par `.md`.

---

## **Change Log**

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-10-09 | 1.0 | Création initiale de la story | Bob (SM) |
| 2025-10-09 | 1.2 | Découpage frontend/backend | Agent |

---

## **Dev Agent Record**

_Section à remplir par l'agent de développement lors de l'implémentation_

### **Agent Model Used**
_À compléter_

### **Debug Log References**
_À compléter_

### **Completion Notes List**
_À compléter_

### **File List**
_À compléter_

---

## **QA Results**

_Section à remplir par l'agent QA lors de la revue_

