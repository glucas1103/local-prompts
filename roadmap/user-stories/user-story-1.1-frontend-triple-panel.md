# **Story 1.1-Frontend : Composant Triple Panel pour Éditeur Markdown**

## **Status**
Draft

---

## **Story**

**En tant que** développeur utilisant l'application,  
**Je veux** un composant triple panel frontend pour gérer des fichiers Markdown,  
**Afin de** créer, éditer et organiser ma documentation avec l'aide d'un assistant IA dans une interface moderne et performante.

---

## **Acceptance Criteria**

1. Le panel gauche affiche une arborescence de fichiers/dossiers avec interface pour création, renommage, suppression et déplacement par drag & drop
2. Le panel central affiche et édite le contenu Markdown du fichier sélectionné avec TipTap
3. Le panel droit propose un chat avec un assistant IA capable de communiquer avec le backend
4. Les trois panels sont redimensionnables horizontalement avec sauvegarde des tailles en localStorage
5. L'interface respecte les guidelines du projet (TypeScript strict, fonctions pures, architecture Next.js)
6. Le composant est intégré dans le dashboard existant comme nouvelle page
7. Le code utilise TanStack Query pour tous les appels API
8. Le code est propre, minimal, et suit les conventions du repo

---

## **Tasks / Subtasks**

- [ ] **Task 1: Configuration initiale et dépendances** (AC: 5, 8)
  - [ ] Installer les dépendances requises (react-arborist, @tiptap/react, react-resizable-panels, zustand)
  - [ ] Configurer TypeScript strict selon les guidelines existantes
  - [ ] S'assurer que Tailwind CSS et Shadcn/UI sont configurés

- [ ] **Task 2: Définir les types TypeScript** (AC: 5, 8)
  - [ ] Créer `src/types/file-system.ts` avec les types pour fichiers/dossiers
  - [ ] Créer `src/types/markdown-editor.ts` avec les types pour l'éditeur
  - [ ] Créer `src/types/ai-chat.ts` avec les types pour le chat IA
  - [ ] Assurer la strictness TypeScript (pas de `any`)

- [ ] **Task 3: Créer les stores Zustand** (AC: 4, 5)
  - [ ] Créer `src/stores/file-system-store.ts` pour gérer l'état local de l'arborescence
  - [ ] Créer `src/stores/editor-store.ts` pour gérer l'état de l'éditeur
  - [ ] Créer `src/stores/chat-store.ts` pour gérer l'état du chat IA
  - [ ] Créer `src/stores/panel-store.ts` pour gérer les tailles des panels (localStorage)

- [ ] **Task 4: Créer les hooks TanStack Query** (AC: 7)
  - [ ] Créer `src/hooks/use-markdown-files.ts` pour récupérer la liste des fichiers
  - [ ] Créer `src/hooks/use-markdown-file-content.ts` pour récupérer le contenu d'un fichier
  - [ ] Créer `src/hooks/use-create-markdown-file.ts` pour la création
  - [ ] Créer `src/hooks/use-update-markdown-file.ts` pour la mise à jour
  - [ ] Créer `src/hooks/use-delete-markdown-file.ts` pour la suppression
  - [ ] Créer `src/hooks/use-rename-markdown-file.ts` pour le renommage
  - [ ] Créer `src/hooks/use-move-markdown-file.ts` pour le déplacement
  - [ ] Créer `src/hooks/use-ai-chat.ts` pour les messages IA

- [ ] **Task 5: Développer le FileExplorer (Panel Gauche)** (AC: 1)
  - [ ] Créer `src/components/triple-panel/file-explorer/index.tsx`
  - [ ] Implémenter `file-tree.tsx` avec react-arborist
  - [ ] Implémenter `file-node.tsx` pour le rendu des nœuds
  - [ ] Implémenter `file-actions.tsx` pour CRUD (créer, renommer, supprimer)
  - [ ] Ajouter le drag & drop pour déplacer les fichiers/dossiers
  - [ ] Intégrer lucide-react pour les icônes selon type de fichier
  - [ ] Connecter au fileSystemStore et aux hooks TanStack Query

- [ ] **Task 6: Développer le ContentEditor (Panel Central)** (AC: 2)
  - [ ] Créer `src/components/triple-panel/content-editor/index.tsx`
  - [ ] Implémenter `markdown-editor.tsx` avec TipTap
  - [ ] Configurer TipTap avec extensions de base (starter-kit, placeholder)
  - [ ] Ajouter la sauvegarde automatique du contenu (debounce 300ms)
  - [ ] Afficher le header avec nom et chemin du fichier
  - [ ] Gérer les états de chargement et erreurs
  - [ ] Connecter au editorStore et aux hooks TanStack Query

- [ ] **Task 7: Développer l'AIChat (Panel Droit)** (AC: 3)
  - [ ] Créer `src/components/triple-panel/ai-chat/index.tsx`
  - [ ] Implémenter `chat-messages.tsx` pour afficher l'historique
  - [ ] Implémenter `chat-input.tsx` pour la saisie utilisateur
  - [ ] Implémenter `message-bubble.tsx` pour le rendu des messages
  - [ ] Ajouter la gestion des états (envoi, réception, erreur)
  - [ ] Gérer les actions IA retournées par le backend (modifier fichier, créer fichier)
  - [ ] Connecter au chatStore et au hook useAIChat

- [ ] **Task 8: Assembler le TriplePanel avec layout redimensionnable** (AC: 4)
  - [ ] Créer `src/components/triple-panel/index.tsx`
  - [ ] Intégrer react-resizable-panels pour les 3 panels
  - [ ] Configurer les tailles par défaut [25%, 50%, 25%]
  - [ ] Implémenter la sauvegarde des tailles en localStorage via panelStore
  - [ ] Gérer les tailles min/max des panels

- [ ] **Task 9: Intégrer dans le Dashboard** (AC: 6)
  - [ ] Créer la page `src/app/(protected)/dashboard/repos/[repositoryId]/markdown-editor/page.tsx`
  - [ ] Ajouter l'entrée de navigation dans `nav-main.tsx`
  - [ ] Configurer la route dans le route-formatter

- [ ] **Task 10: Créer les hooks personnalisés d'orchestration** (AC: 5)
  - [ ] Créer `src/hooks/use-triple-panel.ts` pour synchronisation globale
  - [ ] Gérer la sélection d'un fichier → chargement dans ContentEditor
  - [ ] Gérer la modification dans ContentEditor → invalidation cache
  - [ ] Gérer les actions IA → mise à jour de l'arborescence

- [ ] **Task 11: Créer les utilitaires** (AC: 8)
  - [ ] Créer `src/lib/file-utils.ts` pour opérations sur fichiers
  - [ ] Créer `src/lib/editor-utils.ts` pour helpers éditeur
  - [ ] Créer `src/lib/chat-utils.ts` pour helpers chat
  - [ ] Assurer que toutes les fonctions sont pures quand possible

- [ ] **Task 12: Tests et documentation** (AC: 5, 8)
  - [ ] Écrire tests unitaires pour les stores
  - [ ] Écrire tests unitaires pour les hooks TanStack Query
  - [ ] Écrire tests de composants pour FileExplorer, ContentEditor, AIChat
  - [ ] Créer un README.md pour le composant TriplePanel
  - [ ] Documenter les props et types avec JSDoc

---

## **Dev Notes**

### **Architecture et Contraintes Techniques**

Ce composant doit être développé dans `apps/web` en suivant strictement les guidelines du fichier `.cursorrules`.

#### **Stack Technique Imposé**

**Existant dans le projet :**
- React 19.0.0 avec TypeScript strict
- Next.js 15.3.5 (App Router)
- Tailwind CSS + Shadcn/UI
- TanStack Query 5.71.1 pour data fetching
- Lucide-react pour les icônes
- clsx pour les classes CSS conditionnelles

**À installer :**
- `react-arborist` pour l'arborescence de fichiers
- `@tiptap/react` + `@tiptap/starter-kit` + `@tiptap/extension-placeholder` pour l'éditeur Markdown
- `react-resizable-panels` pour les panels redimensionnables
- `zustand` pour la gestion d'état locale
- `nanoid` pour la génération d'IDs

#### **Principes de Code**

1. **TypeScript strict** : Pas de type `any`, validation runtime avec Zod si nécessaire
2. **Programmation fonctionnelle** : Pas de classes JavaScript, utiliser fonctions et closures
3. **Fonctions pures** quand possible, structures de données immutables
4. **Principe de responsabilité unique** pour chaque composant/fonction
5. **Code auto-documenté** avec JSDoc pour les APIs publiques
6. **Gestion d'erreur complète** avec messages utilisateur clairs
7. **Logging structuré** pour faciliter le debug

#### **Conventions de Nommage**

- **PascalCase** : Components, Types, Interfaces
- **camelCase** : Functions, Variables, Properties
- **kebab-case** : Files, Folders
- **SCREAMING_SNAKE_CASE** : Constants

#### **Structure de Composants React**

✅ **BONNE pratique** :
```typescript
type UserProfileProps = {
  id: string;
};

export function UserProfile(props: UserProfileProps) {
  const { id } = props;
  // Logic here
  return <div>...</div>;
}
```

❌ **MAUVAISE pratique** :
- Pas de destructuring dans les paramètres
- Pas de typage inline des props
- Pas de classes React

#### **Structure de Fichiers**

Le composant doit être organisé ainsi dans `apps/web/src/` :

```
src/
├── app/(protected)/dashboard/repos/[repositoryId]/markdown-editor/
│   └── page.tsx
├── components/
│   └── triple-panel/
│       ├── index.tsx
│       ├── file-explorer/
│       │   ├── index.tsx
│       │   ├── file-tree.tsx
│       │   ├── file-node.tsx
│       │   └── file-actions.tsx
│       ├── content-editor/
│       │   ├── index.tsx
│       │   └── markdown-editor.tsx
│       └── ai-chat/
│           ├── index.tsx
│           ├── chat-messages.tsx
│           ├── chat-input.tsx
│           └── message-bubble.tsx
├── hooks/
│   ├── use-markdown-files.ts
│   ├── use-markdown-file-content.ts
│   ├── use-create-markdown-file.ts
│   ├── use-update-markdown-file.ts
│   ├── use-delete-markdown-file.ts
│   ├── use-rename-markdown-file.ts
│   ├── use-move-markdown-file.ts
│   ├── use-ai-chat.ts
│   └── use-triple-panel.ts
├── stores/
│   ├── file-system-store.ts
│   ├── editor-store.ts
│   ├── chat-store.ts
│   └── panel-store.ts
├── types/
│   ├── file-system.ts
│   ├── markdown-editor.ts
│   └── ai-chat.ts
└── lib/
    ├── file-utils.ts
    ├── editor-utils.ts
    └── chat-utils.ts
```

#### **Gestion d'État**

**Zustand** pour l'état local UI :
- Sélection de fichier en cours
- État du panel (tailles, visibilité)
- État temporaire de l'éditeur

**TanStack Query** pour l'état serveur :
- Liste des fichiers/dossiers
- Contenu des fichiers
- Mutations (CRUD)
- Invalidation et re-fetch automatiques

#### **Configuration TipTap**

L'éditeur Markdown doit utiliser :
- `StarterKit` pour les fonctionnalités de base
- `Placeholder` extension pour le placeholder
- Configuration éditable par défaut
- Sauvegarde automatique via debounce (300ms)

#### **Panels Redimensionnables**

Configuration recommandée pour `react-resizable-panels` :
- Tailles par défaut : `[25, 50, 25]` (pourcentages)
- Tailles minimales : `[15, 30, 15]`
- Tailles maximales : `[40, 70, 40]`
- Persistence via `localStorage` avec clé unique : `triple-panel-layout-${repositoryId}`

#### **Synchronisation entre Panels**

Le hook `use-triple-panel.ts` doit orchestrer :
1. Sélection d'un fichier dans FileExplorer → Chargement dans ContentEditor
2. Modification dans ContentEditor → Mutation et invalidation cache
3. Action IA dans AIChat → Mutation et mise à jour de l'arborescence

#### **Performance et Optimisation**

- Utiliser `React.memo` pour les composants lourds (FileNode, MessageBubble)
- Utiliser `useMemo` et `useCallback` pour éviter les re-renders
- Virtualisation de la liste des messages du chat si > 100 messages
- Debounce pour la sauvegarde automatique (300ms)
- TanStack Query gère le cache automatiquement

#### **Accessibilité**

- ARIA labels sur tous les boutons interactifs
- Navigation au clavier dans l'arborescence
- Focus management lors de la création/renommage de fichiers
- Indicateurs visuels pour les états de chargement

#### **Gestion des Erreurs**

Toute opération doit avoir :
- Un try/catch approprié dans les mutations
- Un message d'erreur utilisateur clair (sonner toast)
- Un log structuré pour le debug
- Un état d'erreur visible dans l'UI

### **Testing**

#### **Localisation des Tests**
- Tests unitaires : `apps/web/src/__tests__/unit/`
- Tests de composants : `apps/web/src/__tests__/components/`
- Tests d'intégration : `apps/web/src/__tests__/integration/`

#### **Standards de Test**
- Utiliser Vitest pour les tests unitaires
- React Testing Library pour les tests de composants
- Couverture minimale : 80% pour les stores et hooks
- Tests de chaque action CRUD
- Tests de la sauvegarde automatique du ContentEditor
- Tests de l'envoi de messages dans AIChat

#### **Frameworks et Patterns**
- Vitest avec configuration TypeScript existante
- React Testing Library avec custom renders
- Mock des stores Zustand pour l'isolation
- Mock de TanStack Query avec QueryClient de test

#### **Requirements Spécifiques**
- Tester tous les cas d'erreur (fichier inexistant, erreur réseau, etc.)
- Tester la persistence du layout dans localStorage
- Tester la synchronisation entre panels
- Tester le drag & drop de fichiers

### **API Client**

Utiliser le `httpClient` existant du projet :

```typescript
// Exemple de hook TanStack Query
export function useMarkdownFiles(repositoryId: string) {
  const httpClient = useHttpClient();
  
  return useQuery({
    queryKey: ['markdown-files', repositoryId],
    queryFn: async () => {
      const response = await httpClient.get(`/api/repositories/${repositoryId}/markdown-files`);
      return response.data;
    },
  });
}
```

### **Intégration avec le Dashboard Existant**

- Le composant doit s'intégrer dans la structure existante du dashboard
- Utiliser le layout `(protected)/dashboard/repos/[repositoryId]/`
- Ajouter l'entrée de navigation dans `nav-main.tsx` comme pour les pages `readme` et `about`
- Respecter le système de routing existant avec `route-formatter.ts`

### **Notes Importantes**

1. **TanStack Query** : Toutes les données serveur doivent être gérées via TanStack Query pour bénéficier du cache, de l'invalidation automatique et des optimistic updates.

2. **Extensibilité IA** : L'AIChat doit parser les réponses du backend pour identifier les actions IA :
```typescript
type AIAction = 
  | { type: 'modify_file'; fileId: string; content: string }
  | { type: 'create_file'; parentId: string | null; name: string; content: string }
  | { type: 'delete_file'; fileId: string };
```

3. **Markdown Uniquement** : Le composant ne gère que les fichiers `.md`. Pas besoin de Monaco Editor ou autre éditeur de code.

4. **Context Repository** : Tous les appels API doivent inclure le `repositoryId` du contexte actuel (via `useRepositoryIdParam()` existant).

5. **Sonner Toasts** : Utiliser `sonner` (déjà installé) pour les notifications utilisateur (succès, erreurs).

---

## **Change Log**

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-10-09 | 1.0 | Création initiale de la story | Bob (SM) |
| 2025-10-09 | 1.1 | Découpage frontend/backend | Agent |

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

