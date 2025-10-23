# Epic 4 - Speiros Evolution: Monitoring, Automation & Web Platform

## Métadonnées

- **ID**: Epic-4
- **Titre**: Évolution de Speiros vers une plateforme complète de gestion de développement
- **Statut**: Draft
- **Priorité**: Haute
- **Estimation**: 89 points (à décomposer en user stories)
- **Créé le**: 2025-10-16

---

## Vision

Transformer Speiros d'un système de workflows documentés en une **plateforme complète de gestion de développement** avec :

1. **CLI standalone** pour automatiser les workflows
2. **Monitoring et debugging** en temps réel
3. **Web app de gestion** synchronisée avec l'IDE
4. **Error handling robuste** et reprise de workflows
5. **Analytics et métriques** pour mesurer l'efficacité

---

## Objectifs Stratégiques

### 🎯 Objectif 1 : Autonomie et Portabilité

- Rendre Speiros indépendant de tout IDE spécifique
- Permettre l'utilisation en ligne de commande
- Faciliter l'intégration CI/CD

### 🎯 Objectif 2 : Visibilité et Contrôle

- Vue d'ensemble de l'état du système
- Monitoring en temps réel des workflows
- Dashboard de métriques et analytics

### 🎯 Objectif 3 : Robustesse et Fiabilité

- Gestion d'erreurs explicite
- Reprise de workflows interrompus
- Validation automatique des outputs

### 🎯 Objectif 4 : Collaboration et Synchronisation

- Web app pour gérer contexte et roadmap
- Synchronisation bidirectionnelle IDE ↔ Web
- Gestion multi-utilisateurs

---

## User Stories (Roadmap)

### Phase 1 : CLI & Automation (21 points)

#### US-4.1 : Speiros CLI Core (8 points)

**Description** : Créer une CLI standalone pour exécuter les workflows Speiros

**Fonctionnalités** :

- `speiros init` - Initialiser un projet Speiros
- `speiros status` - Afficher l'état du système
- `speiros workflow run <workflow-name>` - Lancer un workflow
- `speiros agent <agent-name> <command>` - Invoquer un agent
- `speiros validate <artifact>` - Valider un artefact

**Acceptance Criteria** :

- CLI installable via npm/pip
- Fonctionne sans Cursor
- Supporte tous les workflows existants
- Output formaté (JSON, table, markdown)
- Documentation complète

**Technologies** :

- Node.js + Commander.js ou Python + Click
- YAML parser pour workflows
- Markdown parser pour agents
- Intégration avec LLM APIs (Claude, OpenAI)

---

#### US-4.2 : Workflow Orchestrator (8 points)

**Description** : Moteur d'orchestration pour exécuter les workflows automatiquement

**Fonctionnalités** :

- Parser de workflows YAML
- Exécution séquentielle des steps
- Gestion des dépendances entre steps
- Support des élicitations (mode interactif vs auto)
- Logging détaillé de chaque step

**Acceptance Criteria** :

- Exécute `speiros-workflow.yaml` de bout en bout
- Gère les handoffs entre agents
- Sauvegarde l'état après chaque step
- Permet la reprise en cas d'interruption
- Timeout configurable par step

**Architecture** :

```typescript
interface WorkflowEngine {
  loadWorkflow(path: string): Workflow;
  executeWorkflow(
    workflow: Workflow,
    options: ExecutionOptions
  ): Promise<Result>;
  resumeWorkflow(workflowId: string, fromStep: number): Promise<Result>;
  getWorkflowState(workflowId: string): WorkflowState;
}
```

---

#### US-4.3 : Agent Executor (5 points)

**Description** : Système pour charger et exécuter les agents avec leurs personas

**Fonctionnalités** :

- Parser de fichiers agents `.md`
- Chargement dynamique des personas
- Injection de contexte (product/technical)
- Exécution des commandes d'agents
- Gestion des templates

**Acceptance Criteria** :

- Charge n'importe quel agent depuis `.speiros/agents/`
- Applique la persona à l'IA
- Exécute les tasks associées
- Supporte les élicitations
- Logs structurés

**Exemple d'utilisation** :

```bash
speiros agent product-agent create-brief --input "Add notifications"
speiros agent spec-agent analyse --story user-story-4.1.md
speiros agent dev-agent implement --story user-story-4.1.md
```

---

### Phase 2 : Monitoring & Debugging (21 points)

#### US-4.4 : Status Dashboard CLI (5 points)

**Description** : Commande `speiros status` avec vue d'ensemble du système

**Output** :

```
┌─────────────────────────────────────────────────────────┐
│ SPEIROS STATUS                                          │
├─────────────────────────────────────────────────────────┤
│ 📊 System Health: ✅ Healthy                            │
│                                                         │
│ 📁 Contexts:                                            │
│   ✅ Product: 6 domains (last update: 2025-10-15)      │
│   ✅ Technical: 18 files (last update: 2025-10-14)     │
│   📝 ADRs: 12 decisions documented                     │
│                                                         │
│ 🚀 Active Workflows:                                    │
│   🚧 speiros-workflow-001 (step 5/8, 62% complete)     │
│      Story: US-4.2 - Workflow Orchestrator             │
│      Agent: spec-agent                                  │
│      Started: 2h ago                                    │
│                                                         │
│ 📋 Roadmap:                                             │
│   📝 3 briefs pending                                   │
│   📖 2 epics in progress (Epic-3, Epic-4)              │
│   ✅ 12 stories completed                              │
│   🚧 3 stories in progress                             │
│   📝 8 stories ready for implementation                │
│                                                         │
│ 🔒 File Locks:                                          │
│   🔐 src/components/Button.tsx (locked by US-4.3)      │
│                                                         │
│ ⚠️  Warnings:                                           │
│   • Workflow speiros-workflow-001 idle for 30min       │
│   • Context product/authentication/ not updated in 45d │
└─────────────────────────────────────────────────────────┘
```

**Acceptance Criteria** :

- Affiche l'état des contextes
- Liste les workflows actifs
- Montre la progression des stories
- Détecte les anomalies
- Export JSON pour intégration

---

#### US-4.5 : Workflow State Management (8 points)

**Description** : Système de sauvegarde et reprise d'état des workflows

**Fonctionnalités** :

- Sauvegarde automatique après chaque step
- État stocké dans `.speiros/state/`
- Commande `speiros resume <workflow-id>`
- Historique des exécutions
- Rollback possible

**Structure d'état** :

```yaml
# .speiros/state/workflow-speiros-workflow-001.yaml
workflow_id: speiros-workflow-001
workflow_name: speiros-workflow
status: in_progress
current_step: 5
total_steps: 8
started_at: 2025-10-16T10:30:00Z
last_updated: 2025-10-16T12:45:00Z
story_id: US-4.2
agent: spec-agent

steps_completed:
  - step: 1
    name: create-brief
    status: completed
    completed_at: 2025-10-16T10:35:00Z
    output: roadmap/briefs/brief-042.md

  - step: 2
    name: fetch-product-context
    status: completed
    completed_at: 2025-10-16T10:42:00Z
    output: .speiros/artifacts/speiros-workflow/briefs/brief-042-product-context.md

current_step_data:
  step: 5
  name: fetch-technical-context
  status: in_progress
  started_at: 2025-10-16T12:30:00Z
  elapsed_time: 900s

context:
  brief_id: brief-042
  story_id: US-4.2
  files_locked:
    - src/orchestrator/workflow-engine.ts
```

**Acceptance Criteria** :

- État sauvegardé automatiquement
- Reprise possible à tout moment
- Historique consultable
- Nettoyage des états anciens
- Export/import d'état

---

#### US-4.6 : Error Handling & Recovery (8 points)

**Description** : Gestion robuste des erreurs avec stratégies de récupération

**Fonctionnalités** :

- Détection d'erreurs par type
- Retry automatique (avec backoff)
- Fallback strategies
- Logs d'erreurs détaillés
- Notifications d'erreurs

**Types d'erreurs gérées** :

```typescript
enum ErrorType {
  CONTEXT_NOT_FOUND = "context_not_found",
  TEMPLATE_INVALID = "template_invalid",
  AGENT_TIMEOUT = "agent_timeout",
  ELICITATION_TIMEOUT = "elicitation_timeout",
  FILE_LOCK_CONFLICT = "file_lock_conflict",
  VALIDATION_FAILED = "validation_failed",
  LLM_API_ERROR = "llm_api_error",
  NETWORK_ERROR = "network_error",
}

interface ErrorHandler {
  handle(error: SpeirosError): Promise<RecoveryAction>;
  retry(step: Step, maxAttempts: number): Promise<Result>;
  fallback(step: Step): Promise<Result>;
  notify(error: SpeirosError): void;
}
```

**Stratégies de récupération** :

- **Retry** : Tentatives multiples avec délai exponentiel
- **Fallback** : Mode dégradé (ex: skip élicitation)
- **Rollback** : Retour au step précédent
- **Manual** : Demande intervention utilisateur
- **Abort** : Arrêt propre avec sauvegarde d'état

**Acceptance Criteria** :

- Toutes les erreurs catchées
- Logs structurés (JSON)
- Retry automatique (max 3 fois)
- Sauvegarde d'état avant abort
- Documentation des erreurs

---

### Phase 3 : Validation & Quality (13 points)

#### US-4.7 : Artifact Validators (5 points)

**Description** : Système de validation automatique des artefacts générés

**Validators** :

```yaml
# .speiros/validators/brief-validator.yaml
validator:
  name: brief-completeness
  artifact_type: brief

  checks:
    - name: metadata_present
      rule: has_field("ID") && has_field("Title")
      severity: error

    - name: objective_not_empty
      rule: field("Objective").length > 20
      severity: error

    - name: scope_defined
      rule: has_field("Scope") && field("Scope").includes("IN") && field("Scope").includes("OUT")
      severity: warning

# .speiros/validators/story-validator.yaml
validator:
  name: story-completeness
  artifact_type: user-story

  checks:
    - name: has_description
      rule: has_section("Description")
      severity: error

    - name: min_acceptance_criteria
      rule: section("Acceptance Criteria").count >= 3
      severity: warning

    - name: implementation_plan_present
      rule: has_section("Implementation Plan") || status != "Spec-Ready"
      severity: error

    - name: context_referenced
      rule: references_context("product") && references_context("technical")
      severity: warning

    - name: no_broken_links
      rule: all_links_valid()
      severity: warning
```

**Commandes** :

```bash
speiros validate brief roadmap/briefs/brief-042.md
speiros validate story roadmap/user-stories/user-story-4.2.md
speiros validate all --fix  # Auto-fix quand possible
```

**Acceptance Criteria** :

- Validators pour tous les types d'artefacts
- Règles configurables en YAML
- Auto-fix pour erreurs simples
- Rapport détaillé
- Intégration pre-commit

---

#### US-4.8 : Context Quality Checker (5 points)

**Description** : Vérification de la qualité et fraîcheur des contextes

**Métriques** :

- **Freshness** : Dernière mise à jour
- **Completeness** : Sections remplies
- **Consistency** : Pas de contradictions
- **Coverage** : Tous les domaines documentés
- **Link Health** : Pas de liens cassés

**Commandes** :

```bash
speiros check context product
speiros check context technical
speiros check context adr --outdated  # ADRs > 90 jours
```

**Output** :

```
📊 Context Quality Report: context/product/

✅ Freshness: Good (last update: 15 days ago)
⚠️  Completeness: 85% (missing: organization/roles)
✅ Consistency: No conflicts detected
⚠️  Coverage: 6/8 domains documented
❌ Link Health: 3 broken links found

Recommendations:
  • Update context/product/organization/ (not updated in 45 days)
  • Document missing domains: payments, notifications
  • Fix broken links in authentication/context.md
```

**Acceptance Criteria** :

- Analyse tous les contextes
- Détecte les problèmes
- Recommandations actionnables
- Rapport exportable
- Alertes configurables

---

#### US-4.9 : Metrics & Analytics (3 points)

**Description** : Collecte et analyse de métriques sur l'utilisation de Speiros

**Métriques collectées** :

```typescript
interface SpeirosMetrics {
  workflows: {
    total_executed: number;
    success_rate: number;
    average_duration: number;
    by_workflow: Map<string, WorkflowMetrics>;
  };

  stories: {
    total_completed: number;
    average_size: number;
    estimation_accuracy: number; // estimated vs actual
    by_epic: Map<string, StoryMetrics>;
  };

  agents: {
    invocations: Map<string, number>;
    average_response_time: Map<string, number>;
    elicitations_count: number;
  };

  context: {
    product_usage: Map<string, number>; // combien de fois chaque domaine est référencé
    technical_usage: Map<string, number>;
    adr_count: number;
  };

  quality: {
    validation_failures: number;
    retry_count: number;
    error_rate: number;
  };
}
```

**Commandes** :

```bash
speiros metrics show
speiros metrics export --format json
speiros metrics report --period last-30-days
```

**Acceptance Criteria** :

- Collecte automatique
- Stockage local (SQLite)
- Visualisation CLI
- Export JSON/CSV
- Anonymisation possible

---

### Phase 4 : Web Platform (34 points)

#### US-4.10 : Speiros Web App - Core (13 points)

**Description** : Application web pour monitorer et gérer Speiros

**Stack technique** :

- **Frontend** : Next.js 14 + React + TypeScript
- **Backend** : Next.js API Routes + tRPC
- **Database** : PostgreSQL (sync avec fichiers locaux)
- **Real-time** : WebSockets (pour sync IDE ↔ Web)
- **Auth** : Clerk (déjà utilisé dans le projet)

**Pages principales** :

##### 1. Dashboard (`/`)

```
┌─────────────────────────────────────────────────────┐
│ 🎯 Speiros Dashboard                    [Settings] │
├─────────────────────────────────────────────────────┤
│                                                     │
│ 📊 System Health                                    │
│ ┌─────────┬─────────┬─────────┬─────────┐          │
│ │ Contexts│ Workflows│ Stories │  ADRs   │          │
│ │   ✅ 2  │  🚧 1   │  ✅ 12  │  📝 15  │          │
│ └─────────┴─────────┴─────────┴─────────┘          │
│                                                     │
│ 🚀 Active Workflows                                 │
│ ┌───────────────────────────────────────┐          │
│ │ speiros-workflow-001        [View]    │          │
│ │ Story: US-4.10                        │          │
│ │ Progress: ████████░░ 62% (step 5/8)   │          │
│ │ Agent: spec-agent                     │          │
│ │ Started: 2h ago                       │          │
│ └───────────────────────────────────────┘          │
│                                                     │
│ 📋 Recent Activity                                  │
│ • US-4.9 completed by dev-agent (5min ago)         │
│ • Context technical/API updated (1h ago)           │
│ • Brief-043 created (2h ago)                       │
│                                                     │
│ ⚠️  Alerts                                          │
│ • Workflow idle for 30min                          │
│ • Context outdated: product/authentication         │
└─────────────────────────────────────────────────────┘
```

##### 2. Contexts (`/contexts`)

- Liste des contextes (product, technical, ADRs)
- Éditeur Markdown intégré
- Historique des modifications
- Recherche full-text
- Visualisation des liens entre contextes

##### 3. Roadmap (`/roadmap`)

- Vue Kanban des stories
- Filtres par epic, statut, priorité
- Drag & drop pour changer statuts
- Création/édition de briefs, epics, stories
- Visualisation des dépendances

##### 4. Workflows (`/workflows`)

- Liste des workflows disponibles
- Lancer un workflow
- Voir workflows actifs
- Historique des exécutions
- Logs en temps réel

##### 5. Analytics (`/analytics`)

- Graphiques de métriques
- Vélocité de l'équipe
- Temps moyen par story
- Taux de réussite des workflows
- Utilisation des contextes

**Acceptance Criteria** :

- Interface responsive
- Authentification Clerk
- Toutes les pages fonctionnelles
- Édition de contextes
- Gestion de roadmap
- Visualisation de workflows

---

#### US-4.11 : Sync Engine IDE ↔ Web (13 points)

**Description** : Synchronisation bidirectionnelle entre fichiers locaux et web app

**Architecture** :

```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│             │  Watch  │              │  Sync   │             │
│  IDE/Local  │────────▶│ Sync Engine  │◀───────▶│   Web App   │
│   Files     │◀────────│  (Daemon)    │         │ (Database)  │
│             │  Write  │              │         │             │
└─────────────┘         └──────────────┘         └─────────────┘
```

**Fonctionnalités** :

1. **File Watcher** (Local → Web)
   - Détecte changements dans `.speiros/`, `context/`, `roadmap/`
   - Parse les fichiers modifiés
   - Envoie les updates à la web app via WebSocket
   - Gère les conflits (merge ou demande résolution)

2. **Web Sync** (Web → Local)
   - Écoute les changements depuis la web app
   - Écrit les modifications dans les fichiers locaux
   - Respecte les locks de fichiers
   - Notifie l'IDE des changements

3. **Conflict Resolution**
   - Détection de conflits (même fichier modifié des 2 côtés)
   - Stratégies : last-write-wins, manual, merge
   - UI pour résoudre manuellement
   - Historique des versions

**Daemon CLI** :

```bash
# Démarrer le daemon de sync
speiros sync start

# Voir le statut
speiros sync status

# Arrêter
speiros sync stop

# Configuration
speiros sync config --strategy last-write-wins
```

**Protocol** :

```typescript
// WebSocket messages
interface SyncMessage {
  type: "file_changed" | "file_created" | "file_deleted" | "conflict";
  path: string;
  content?: string;
  timestamp: number;
  user_id: string;
  checksum: string;
}

interface SyncEngine {
  watch(paths: string[]): void;
  sync(file: File): Promise<SyncResult>;
  resolveConflict(conflict: Conflict, strategy: Strategy): Promise<void>;
  getStatus(): SyncStatus;
}
```

**Acceptance Criteria** :

- Sync bidirectionnel fonctionnel
- Détection de conflits
- Résolution de conflits
- Performance (< 1s de latence)
- Gestion multi-utilisateurs
- Logs de sync
- Reconnexion automatique

---

#### US-4.12 : Real-time Collaboration (8 points)

**Description** : Collaboration temps réel sur les contextes et stories

**Fonctionnalités** :

1. **Présence** : Voir qui est en ligne et sur quel fichier
2. **Live Cursors** : Voir les curseurs des autres utilisateurs
3. **Live Editing** : Édition collaborative (CRDT)
4. **Comments** : Commentaires sur les stories/contextes
5. **Notifications** : Alertes en temps réel

**Technologies** :

- **Yjs** : CRDT pour édition collaborative
- **WebRTC** : Communication peer-to-peer
- **WebSockets** : Fallback pour signaling
- **Presence API** : Suivi des utilisateurs

**UI** :

```
┌─────────────────────────────────────────────────────┐
│ 📝 Editing: user-story-4.12.md                      │
│                                                     │
│ 👥 Online (3):                                      │
│ • Lucas (you) - Editing Implementation Plan        │
│ • Alice - Reading                                   │
│ • Bob - Commenting                                  │
│                                                     │
│ ┌─────────────────────────────────────────────┐    │
│ │ ## Description                              │    │
│ │                                             │    │
│ │ En tant que développeur, je veux...        │    │
│ │                                             │    │
│ │ ## Implementation Plan                      │    │
│ │                                             │    │
│ │ 1. Create sync engine                       │    │
│ │    ▲ Alice's cursor                         │    │
│ │                                             │    │
│ │ 2. Implement WebSocket server               │    │
│ │    💬 Bob: "Should we use Socket.io?"       │    │
│ └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

**Acceptance Criteria** :

- Présence en temps réel
- Édition collaborative sans conflits
- Commentaires persistants
- Notifications push
- Performance (< 100ms latency)

---

## Architecture Technique

### Stack Global

```
┌─────────────────────────────────────────────────────────┐
│                    Speiros Platform                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │              │  │              │  │              │ │
│  │  Speiros CLI │  │  Sync Daemon │  │   Web App    │ │
│  │  (Node.js)   │  │  (Node.js)   │  │  (Next.js)   │ │
│  │              │  │              │  │              │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│         │                 │                  │         │
│         └─────────────────┼──────────────────┘         │
│                           │                            │
│  ┌────────────────────────▼─────────────────────────┐  │
│  │         Speiros Core Library                     │  │
│  │  - Workflow Engine                               │  │
│  │  - Agent Executor                                │  │
│  │  - Validators                                    │  │
│  │  - Parsers (YAML, Markdown)                      │  │
│  │  - State Management                              │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Data Layer                         │   │
│  │  - File System (.speiros/, context/, roadmap/)  │   │
│  │  - PostgreSQL (web app data)                    │   │
│  │  - SQLite (metrics, cache)                      │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │           External Services                     │   │
│  │  - LLM APIs (Claude, OpenAI)                    │   │
│  │  - Clerk (Auth)                                 │   │
│  │  - Sentry (Error tracking)                      │   │
│  │  - PostHog (Analytics)                          │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Monorepo Structure

```
monorepo-boucleai/
├── apps/
│   ├── speiros-web/           # 🆕 Web app Next.js
│   │   ├── src/
│   │   │   ├── app/
│   │   │   ├── components/
│   │   │   ├── lib/
│   │   │   └── server/
│   │   └── package.json
│   │
│   ├── speiros-cli/           # 🆕 CLI Node.js
│   │   ├── src/
│   │   │   ├── commands/
│   │   │   ├── orchestrator/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── speiros-daemon/        # 🆕 Sync daemon
│       ├── src/
│       │   ├── watcher/
│       │   ├── sync/
│       │   └── index.ts
│       └── package.json
│
├── packages/
│   ├── speiros-core/          # 🆕 Core library
│   │   ├── src/
│   │   │   ├── workflow/
│   │   │   ├── agent/
│   │   │   ├── validator/
│   │   │   ├── parser/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── speiros-types/         # 🆕 Shared types
│   │   └── src/
│   │       └── index.ts
│   │
│   └── speiros-db/            # 🆕 Database client
│       └── src/
│           ├── schema.ts
│           └── client.ts
│
├── .speiros/                  # Configuration Speiros
├── context/                   # Contextes
├── roadmap/                   # Roadmap
└── package.json
```

---

## Database Schema (PostgreSQL)

```sql
-- Users (via Clerk)
CREATE TABLE users (
  id UUID PRIMARY KEY,
  clerk_id TEXT UNIQUE NOT NULL,
  email TEXT NOT NULL,
  name TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Projects
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  path TEXT NOT NULL,  -- Local path
  owner_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Project Members
CREATE TABLE project_members (
  project_id UUID REFERENCES projects(id),
  user_id UUID REFERENCES users(id),
  role TEXT NOT NULL,  -- owner, editor, viewer
  PRIMARY KEY (project_id, user_id)
);

-- Contexts
CREATE TABLE contexts (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  type TEXT NOT NULL,  -- product, technical, adr
  path TEXT NOT NULL,
  content TEXT,
  checksum TEXT,
  last_synced_at TIMESTAMP,
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Stories
CREATE TABLE stories (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  story_id TEXT UNIQUE NOT NULL,  -- US-4.1
  title TEXT NOT NULL,
  status TEXT NOT NULL,
  priority TEXT,
  estimation INTEGER,
  epic_id TEXT,
  content TEXT,
  checksum TEXT,
  last_synced_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Workflows
CREATE TABLE workflow_executions (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  workflow_name TEXT NOT NULL,
  story_id TEXT,
  status TEXT NOT NULL,  -- pending, in_progress, completed, failed
  current_step INTEGER,
  total_steps INTEGER,
  state JSONB,  -- Full workflow state
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  error TEXT
);

-- Workflow Steps
CREATE TABLE workflow_steps (
  id UUID PRIMARY KEY,
  execution_id UUID REFERENCES workflow_executions(id),
  step_number INTEGER NOT NULL,
  name TEXT NOT NULL,
  status TEXT NOT NULL,
  agent TEXT,
  output TEXT,
  error TEXT,
  started_at TIMESTAMP,
  completed_at TIMESTAMP
);

-- Comments
CREATE TABLE comments (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  entity_type TEXT NOT NULL,  -- story, context, workflow
  entity_id UUID NOT NULL,
  user_id UUID REFERENCES users(id),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Metrics
CREATE TABLE metrics (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  metric_type TEXT NOT NULL,
  metric_name TEXT NOT NULL,
  value NUMERIC NOT NULL,
  metadata JSONB,
  recorded_at TIMESTAMP DEFAULT NOW()
);

-- File Locks
CREATE TABLE file_locks (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  file_path TEXT NOT NULL,
  locked_by UUID REFERENCES users(id),
  story_id TEXT,
  locked_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(project_id, file_path)
);

-- Sync Events
CREATE TABLE sync_events (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  event_type TEXT NOT NULL,  -- file_changed, file_created, file_deleted
  file_path TEXT NOT NULL,
  user_id UUID REFERENCES users(id),
  checksum TEXT,
  synced BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Roadmap de Développement

### Sprint 1-2 : CLI Foundation (2 semaines)

- ✅ US-4.1 : Speiros CLI Core
- ✅ US-4.2 : Workflow Orchestrator
- ✅ US-4.3 : Agent Executor

**Livrable** : CLI fonctionnelle pour exécuter workflows

---

### Sprint 3-4 : Monitoring & Debugging (2 semaines)

- ✅ US-4.4 : Status Dashboard CLI
- ✅ US-4.5 : Workflow State Management
- ✅ US-4.6 : Error Handling & Recovery

**Livrable** : Workflows robustes avec reprise et monitoring

---

### Sprint 5-6 : Quality & Validation (2 semaines)

- ✅ US-4.7 : Artifact Validators
- ✅ US-4.8 : Context Quality Checker
- ✅ US-4.9 : Metrics & Analytics

**Livrable** : Système de validation et métriques

---

### Sprint 7-10 : Web Platform Core (4 semaines)

- ✅ US-4.10 : Speiros Web App - Core
- ✅ US-4.11 : Sync Engine IDE ↔ Web
- ✅ US-4.12 : Real-time Collaboration

**Livrable** : Web app complète avec sync bidirectionnelle

---

## Métriques de Succès

### Adoption

- [ ] 100% des workflows exécutables via CLI
- [ ] 80% des stories créées via Speiros
- [ ] 5+ utilisateurs actifs sur la web app

### Performance

- [ ] Workflow complet < 10 minutes
- [ ] Sync latency < 1 seconde
- [ ] Web app load time < 2 secondes

### Qualité

- [ ] 95% de taux de réussite des workflows
- [ ] 0 perte de données lors de sync
- [ ] 90% de couverture de tests

### Productivité

- [ ] Réduction de 50% du temps de création de stories
- [ ] Réduction de 30% du temps de documentation
- [ ] 100% des décisions techniques documentées (ADRs)

---

## Risques et Mitigation

| Risque                                 | Impact | Probabilité | Mitigation                                  |
| -------------------------------------- | ------ | ----------- | ------------------------------------------- |
| Complexité de la sync bidirectionnelle | Élevé  | Haute       | POC early, utiliser lib éprouvée (Yjs)      |
| Performance de la web app              | Moyen  | Moyenne     | Caching agressif, pagination, lazy loading  |
| Adoption utilisateurs                  | Élevé  | Moyenne     | UX excellente, documentation, onboarding    |
| Coûts LLM APIs                         | Moyen  | Haute       | Caching, rate limiting, mode local possible |
| Conflits de merge                      | Moyen  | Haute       | Stratégies claires, UI de résolution        |
| Sécurité données                       | Élevé  | Faible      | Encryption, auth robuste, audit logs        |

---

## Dépendances

### Techniques

- Node.js 20+
- PostgreSQL 15+
- Next.js 14+
- Clerk Auth
- LLM APIs (Claude/OpenAI)

### Projets

- Epic-3 complété (restructuration Speiros)
- Contextes product/technical peuplés
- Au moins 5 stories complétées via Speiros

---

## Notes Techniques

### Choix d'Architecture

#### Pourquoi un Daemon de Sync ?

- Évite de bloquer l'IDE
- Gère la reconnexion automatique
- Permet le batch de changements
- Fonctionne en background

#### Pourquoi PostgreSQL + Files ?

- **Files** : Source de vérité, versionnable Git
- **PostgreSQL** : Performance, recherche, relations
- **Sync** : Best of both worlds

#### Pourquoi Next.js ?

- SSR pour SEO et performance
- API Routes pour backend simple
- Déjà utilisé dans le projet
- Écosystème riche

---

## Évolutions Futures (Post-Epic-4)

### Epic-5 : AI Enhancements

- Génération automatique de tests
- Suggestions de refactoring
- Détection de bugs potentiels
- Auto-completion de stories

### Epic-6 : Integrations

- GitHub Actions integration
- Jira/Linear sync
- Slack notifications
- VS Code extension

### Epic-7 : Enterprise Features

- Multi-tenancy
- RBAC avancé
- Audit logs
- SSO/SAML

---

## Conclusion

Cet epic transformera Speiros d'un système de documentation en une **plateforme complète de gestion de développement** avec :

✅ **Autonomie** : CLI standalone, indépendant de l'IDE  
✅ **Visibilité** : Monitoring temps réel, métriques, analytics  
✅ **Robustesse** : Error handling, reprise, validation  
✅ **Collaboration** : Web app, sync bidirectionnelle, temps réel

**Objectif final** : Une plateforme où le développement piloté par contexte devient fluide, traçable et collaboratif.

---

**Version:** 1.0  
**Créé le:** 2025-10-16  
**Auteur:** Lucas Gaillard  
**Estimation totale:** 89 points (~18 semaines pour 1 dev)
