# User Story 3.1 - Restructurer les Livrables Speiros

## Métadonnées

- **ID**: US-3.1
- **Titre**: Restructurer l'organisation des livrables finaux de Speiros
- **Epic**: Epic 3 - Amélioration de l'Architecture Speiros
- **Statut**: Ready for Implementation
- **Priorité**: Haute
- **Estimation**: 5 points

## Description

En tant que **développeur utilisant Speiros**, je veux une **organisation claire des livrables finaux à la racine du projet** afin de **distinguer facilement la documentation de référence (contextes) des artefacts de développement (roadmap) et des artefacts intermédiaires de workflows**.

## Contexte

Actuellement, l'organisation des fichiers Speiros crée de la confusion :

1. **Mélange de concepts** : Le dossier `.speiros/context-engineering/` contient à la fois des artefacts intermédiaires et des user stories
2. **Dispersion des livrables** : Les contextes finaux sont à la racine (`product-context/`, `technical-context/`) mais la roadmap est enfouie dans `.speiros/context-engineering/roadmap/`
3. **Nomenclature confuse** : `context-engineering` suggère qu'il contient la documentation finale alors qu'il contient surtout des artefacts intermédiaires

## Objectifs

1. **Créer deux dossiers à la racine** pour tous les livrables finaux :
   - `context/` - Documentation de référence (product + technical)
   - `roadmap/` - Artefacts de développement actif (briefs, epics, user stories)

2. **Clarifier `.speiros/`** comme contenant uniquement :
   - Les définitions de workflows
   - Les agents
   - Les templates
   - Les artefacts intermédiaires des workflows

3. **Maintenir la cohérence** des références dans tous les fichiers

## Structure Cible

### Avant (Structure Actuelle)

```
monorepo-boucleai/
├── product-context/                      # ✅ Documentation produit finale
├── technical-context/                    # ✅ Documentation technique finale
└── .speiros/
    └── context-engineering/              # ⚠️ Nom confus
        ├── audit-codebase-product-artifacts/    # Artefacts intermédiaires
        ├── audit-codebase-technical-artifacts/  # Artefacts intermédiaires
        ├── product-context/              # ❌ SUPPRIMÉ (vide)
        ├── roadmap/                      # ❌ Mal placé
        │   ├── briefs/
        │   ├── epics/
        │   └── user-stories/
        ├── speiros-workflow-artifacts/   # Artefacts intermédiaires
        └── README.md
```

### Après (Structure Cible)

```
monorepo-boucleai/
│
├── context/                              # 🆕 NOUVEAU - Tous les contextes
│   ├── product/                          # 🔄 Déplacé depuis /product-context/
│   │   ├── authentication/
│   │   ├── organization/
│   │   ├── github-integration/
│   │   ├── user-management/
│   │   ├── administration/
│   │   ├── repository-dashboard/
│   │   └── README.md
│   ├── technical/                        # 🔄 Déplacé depuis /technical-context/
│   │   ├── context/
│   │   ├── tools/
│   │   ├── architecture-decision-records/
│   │   └── README.md
│   └── README.md                         # 🆕 Index des contextes
│
├── roadmap/                              # 🔄 Déplacé depuis .speiros/context-engineering/roadmap/
│   ├── briefs/
│   ├── epics/
│   ├── user-stories/
│   │   ├── user-story-1.1-frontend-triple-panel.md
│   │   ├── user-story-1.2-backend-markdown-api.md
│   │   ├── user-story-2.1-audit-codebase-product-workflow.md
│   │   ├── user-story-2.2-audit-codebase-technical-workflow.md
│   │   ├── user-story-3.1-restructure-speiros-deliverables.md  # Cette story
│   │   └── ...
│   └── README.md                         # 🆕 Guide de la roadmap
│
└── .speiros/                             # Configuration et artefacts intermédiaires uniquement
    ├── artifacts/                        # 🔄 Renommé depuis context-engineering/
    │   ├── audit-product/                # 🔄 Renommé depuis audit-codebase-product-artifacts/
    │   │   ├── initial-summary.md
    │   │   ├── audit-results.md
    │   │   ├── enrichment-qa.md
    │   │   ├── structure-plan.md
    │   │   ├── WORKFLOW-SUMMARY.md
    │   │   ├── COMPLETION-REPORT.md
    │   │   └── research/
    │   ├── audit-technical/              # 🔄 Renommé depuis audit-codebase-technical-artifacts/
    │   │   └── ...
    │   ├── speiros-workflow/             # 🔄 Renommé depuis speiros-workflow-artifacts/
    │   │   ├── briefs/
    │   │   └── user-stories/
    │   └── README.md                     # 🆕 Explique les artefacts intermédiaires
    │
    ├── workflows/
    │   ├── audit-codebase-product.yaml
    │   ├── audit-codebase-technical.yaml
    │   └── speiros-workflow.yaml
    │
    ├── agents/
    │   ├── audit-codebase/
    │   │   ├── context-agent.md
    │   │   └── sub-context-agent.md
    │   └── speiros-workflow/
    │       ├── product-agent.md
    │       ├── spec-agent.md
    │       └── dev-agent.md
    │
    ├── tasks/
    │   ├── audit-codebase-product/
    │   ├── audit-codebase-technical/
    │   └── speiros-workflow/
    │
    ├── templates/
    │   ├── audit-codebase/
    │   └── speiros-workflow/
    │
    ├── checklists/
    └── README.md                         # 🔄 Mettre à jour les chemins
```

## Bénéfices

### 1. Clarté Conceptuelle

| Dossier     | Rôle                                            | Audience        | Fréquence MAJ     |
| ----------- | ----------------------------------------------- | --------------- | ----------------- |
| `/context/` | Documentation de référence stable               | Tous            | Rare              |
| `/roadmap/` | Développement actif, planification              | PM, Dev         | Continue          |
| `.speiros/` | Configuration système, artefacts intermédiaires | Système Speiros | Rarement manipulé |

### 2. Navigation Simplifiée

```bash
# Documentation de référence
cd context/product/          # Comprendre les features
cd context/technical/        # Comprendre l'architecture

# Développement actif
cd roadmap/user-stories/     # Voir les stories en cours
cd roadmap/briefs/           # Voir les demandes

# Configuration système (rarement consulté)
cd .speiros/workflows/       # Voir les workflows
cd .speiros/artifacts/       # Voir les artefacts intermédiaires
```

### 3. Cohérence avec la Méthodologie

- **Livrables finaux** → Visibles à la racine
- **Configuration système** → Cachés dans `.speiros/`
- **Artefacts intermédiaires** → Archivés dans `.speiros/artifacts/`

## Acceptance Criteria

### AC1: Création du dossier `/context/` avec sous-structure

**Critères de validation :**

- [ ] Le dossier `/context/` existe à la racine du monorepo
- [ ] Le dossier `/context/product/` existe et contient tous les domaines de `/product-context/`
- [ ] Le dossier `/context/technical/` existe et contient toute la structure de `/technical-context/`
- [ ] Les fichiers `/product-context/README.md` sont déplacés vers `/context/product/README.md`
- [ ] Les fichiers `/technical-context/README.md` sont déplacés vers `/context/technical/README.md`
- [ ] Un nouveau fichier `/context/README.md` existe et explique l'organisation

**Tests de validation :**

```bash
# Vérifier la structure
ls -la context/
ls -la context/product/
ls -la context/technical/

# Vérifier que les domaines sont présents
ls context/product/authentication/
ls context/product/organization/
ls context/technical/context/
ls context/technical/tools/
```

### AC2: Création du dossier `/roadmap/` à la racine

**Critères de validation :**

- [ ] Le dossier `/roadmap/` existe à la racine du monorepo
- [ ] Le dossier `/roadmap/briefs/` existe
- [ ] Le dossier `/roadmap/epics/` existe
- [ ] Le dossier `/roadmap/user-stories/` existe et contient toutes les user stories
- [ ] Toutes les user stories sont déplacées depuis `.speiros/context-engineering/roadmap/user-stories/`
- [ ] Un fichier `/roadmap/README.md` existe et explique l'organisation de la roadmap

**Tests de validation :**

```bash
# Vérifier la structure
ls -la roadmap/
ls -la roadmap/user-stories/

# Vérifier le nombre de user stories
ls roadmap/user-stories/*.md | wc -l
# Doit retourner au moins 8 (les stories existantes + cette story)
```

### AC3: Renommage de `context-engineering/` en `artifacts/`

**Critères de validation :**

- [ ] Le dossier `.speiros/context-engineering/` n'existe plus
- [ ] Le dossier `.speiros/artifacts/` existe
- [ ] Le dossier `.speiros/artifacts/audit-product/` existe (renommé depuis `audit-codebase-product-artifacts/`)
- [ ] Le dossier `.speiros/artifacts/audit-technical/` existe (renommé depuis `audit-codebase-technical-artifacts/`)
- [ ] Le dossier `.speiros/artifacts/speiros-workflow/` existe (renommé depuis `speiros-workflow-artifacts/`)
- [ ] Le dossier `.speiros/artifacts/product-context/` n'existe plus (supprimé car vide)
- [ ] Le dossier `.speiros/artifacts/roadmap/` n'existe plus (déplacé vers `/roadmap/`)

**Tests de validation :**

```bash
# Vérifier que l'ancien dossier n'existe plus
[ ! -d .speiros/context-engineering ] && echo "✅ Ancien dossier supprimé"

# Vérifier la nouvelle structure
ls -la .speiros/artifacts/
ls -la .speiros/artifacts/audit-product/
ls -la .speiros/artifacts/audit-technical/
ls -la .speiros/artifacts/speiros-workflow/
```

### AC4: Mise à jour des références dans les workflows

**Critères de validation :**

- [ ] Le fichier `.speiros/workflows/audit-codebase-product.yaml` référence `/context/product/` au lieu de `/product-context/`
- [ ] Le fichier `.speiros/workflows/audit-codebase-technical.yaml` référence `/context/technical/` au lieu de `/technical-context/`
- [ ] Le fichier `.speiros/workflows/speiros-workflow.yaml` référence `/roadmap/` au lieu de `.speiros/context-engineering/roadmap/`
- [ ] Tous les chemins d'artefacts intermédiaires pointent vers `.speiros/artifacts/`

**Tests de validation :**

```bash
# Vérifier qu'il n'y a plus de références aux anciens chemins
grep -r "product-context/" .speiros/workflows/ && echo "❌ Références obsolètes trouvées" || echo "✅ Aucune référence obsolète"
grep -r "technical-context/" .speiros/workflows/ && echo "❌ Références obsolètes trouvées" || echo "✅ Aucune référence obsolète"
grep -r "context-engineering/roadmap" .speiros/workflows/ && echo "❌ Références obsolètes trouvées" || echo "✅ Aucune référence obsolète"
```

### AC5: Mise à jour des références dans les agents

**Critères de validation :**

- [ ] Le fichier `.speiros/agents/audit-codebase/context-agent.md` référence les nouveaux chemins
- [ ] Le fichier `.speiros/agents/audit-codebase/sub-context-agent.md` référence les nouveaux chemins
- [ ] Les agents du workflow Speiros référencent `/roadmap/` et `/context/`

**Tests de validation :**

```bash
# Vérifier les références dans les agents
grep -r "product-context/" .speiros/agents/ && echo "❌ Références obsolètes" || echo "✅ OK"
grep -r "technical-context/" .speiros/agents/ && echo "❌ Références obsolètes" || echo "✅ OK"
```

### AC6: Mise à jour des références dans les tasks

**Critères de validation :**

- [ ] Toutes les tasks dans `.speiros/tasks/audit-codebase-product/` référencent `/context/product/`
- [ ] Toutes les tasks dans `.speiros/tasks/audit-codebase-technical/` référencent `/context/technical/`
- [ ] Toutes les tasks dans `.speiros/tasks/speiros-workflow/` référencent `/roadmap/`
- [ ] Les tasks référencent `.speiros/artifacts/` pour les artefacts intermédiaires

**Tests de validation :**

```bash
# Vérifier les références dans les tasks
grep -r "product-context/" .speiros/tasks/ && echo "❌ Références obsolètes" || echo "✅ OK"
grep -r "technical-context/" .speiros/tasks/ && echo "❌ Références obsolètes" || echo "✅ OK"
grep -r "context-engineering/" .speiros/tasks/ && echo "❌ Références obsolètes" || echo "✅ OK"
```

### AC7: Création des fichiers README explicatifs

**Critères de validation :**

- [ ] `/context/README.md` existe et explique :
  - La distinction entre product et technical
  - Comment naviguer dans les contextes
  - Comment mettre à jour les contextes
  - La relation avec les workflows
- [ ] `/roadmap/README.md` existe et explique :
  - L'organisation des briefs, epics, user stories
  - Le workflow de création et suivi
  - Les conventions de nommage
  - Le lien avec le workflow Speiros
- [ ] `.speiros/artifacts/README.md` existe et explique :
  - La nature des artefacts intermédiaires
  - Quels artefacts sont créés par quel workflow
  - Pourquoi ils sont séparés des livrables finaux
  - Quand les consulter

**Tests de validation :**

```bash
# Vérifier l'existence des README
[ -f context/README.md ] && echo "✅ context/README.md existe"
[ -f roadmap/README.md ] && echo "✅ roadmap/README.md existe"
[ -f .speiros/artifacts/README.md ] && echo "✅ .speiros/artifacts/README.md existe"

# Vérifier qu'ils ne sont pas vides
[ $(wc -l < context/README.md) -gt 50 ] && echo "✅ context/README.md est documenté"
[ $(wc -l < roadmap/README.md) -gt 30 ] && echo "✅ roadmap/README.md est documenté"
```

### AC8: Mise à jour du README principal de `.speiros/`

**Critères de validation :**

- [ ] `.speiros/README.md` reflète la nouvelle structure
- [ ] Les exemples de chemins sont mis à jour
- [ ] Le diagramme de structure (si présent) est mis à jour
- [ ] Les instructions d'utilisation pointent vers les bons chemins

**Tests de validation :**

```bash
# Vérifier que le README est à jour
grep "context/product" .speiros/README.md && echo "✅ Nouvelle structure mentionnée"
grep "roadmap/" .speiros/README.md && echo "✅ Nouveau roadmap mentionné"
grep "artifacts/" .speiros/README.md && echo "✅ Nouveaux artefacts mentionnés"
```

### AC9: Suppression des anciens dossiers

**Critères de validation :**

- [ ] `/product-context/` n'existe plus à la racine
- [ ] `/technical-context/` n'existe plus à la racine
- [ ] `.speiros/context-engineering/` n'existe plus
- [ ] Aucun fichier orphelin n'est laissé derrière

**Tests de validation :**

```bash
# Vérifier que les anciens dossiers n'existent plus
[ ! -d product-context ] && echo "✅ product-context/ supprimé"
[ ! -d technical-context ] && echo "✅ technical-context/ supprimé"
[ ! -d .speiros/context-engineering ] && echo "✅ context-engineering/ supprimé"

# Vérifier qu'il n'y a pas de liens symboliques cassés
find . -type l ! -exec test -e {} \; -print
```

### AC10: Validation de la cohérence globale

**Critères de validation :**

- [ ] Tous les workflows s'exécutent sans erreur de chemin
- [ ] Les agents peuvent être activés et référencent les bons chemins
- [ ] Aucun lien mort dans les fichiers markdown
- [ ] La structure est documentée dans le README principal du projet

**Tests de validation :**

```bash
# Rechercher des références aux anciens chemins dans tout le projet
find . -type f -name "*.md" -o -name "*.yaml" | xargs grep -l "product-context/" | grep -v "node_modules"
find . -type f -name "*.md" -o -name "*.yaml" | xargs grep -l "technical-context/" | grep -v "node_modules"
find . -type f -name "*.md" -o -name "*.yaml" | xargs grep -l "context-engineering/roadmap" | grep -v "node_modules"

# Si aucune ligne n'est retournée, la migration est complète
```

## Tasks / Subtasks

### Tâche 1: Préparation et Backup (AC: Tous)

- [ ] **1.1** Créer une branche Git pour cette restructuration : `git checkout -b refactor/speiros-structure-v3`
- [ ] **1.2** Vérifier l'état actuel avec `git status`
- [ ] **1.3** Lister tous les fichiers qui seront déplacés/modifiés
- [ ] **1.4** Créer un backup des dossiers critiques :
  ```bash
  tar -czf backup-speiros-structure-$(date +%Y%m%d).tar.gz product-context/ technical-context/ .speiros/context-engineering/
  ```

### Tâche 2: Création du dossier `/context/` (AC: 1)

- [ ] **2.1** Créer la structure de base :
  ```bash
  mkdir -p context/product
  mkdir -p context/technical
  ```
- [ ] **2.2** Déplacer `/product-context/` vers `/context/product/` :
  ```bash
  mv product-context/* context/product/
  ```
- [ ] **2.3** Déplacer `/technical-context/` vers `/context/technical/` :
  ```bash
  mv technical-context/* context/technical/
  ```
- [ ] **2.4** Créer `/context/README.md` (voir template dans section Documentation)
- [ ] **2.5** Vérifier l'intégrité des fichiers déplacés
- [ ] **2.6** Supprimer les anciens dossiers vides :
  ```bash
  rmdir product-context/
  rmdir technical-context/
  ```

### Tâche 3: Création du dossier `/roadmap/` (AC: 2)

- [ ] **3.1** Créer la structure de base :
  ```bash
  mkdir -p roadmap/briefs
  mkdir -p roadmap/epics
  mkdir -p roadmap/user-stories
  ```
- [ ] **3.2** Déplacer les user stories :
  ```bash
  mv .speiros/context-engineering/roadmap/user-stories/* roadmap/user-stories/
  ```
- [ ] **3.3** Vérifier que les dossiers briefs/ et epics/ sont vides (les créer si nécessaire)
- [ ] **3.4** Créer `/roadmap/README.md` (voir template dans section Documentation)
- [ ] **3.5** Déplacer cette user story (user-story-3.1) vers `/roadmap/user-stories/`

### Tâche 4: Renommage de `context-engineering/` en `artifacts/` (AC: 3)

- [ ] **4.1** Créer la nouvelle structure :
  ```bash
  mkdir -p .speiros/artifacts
  ```
- [ ] **4.2** Renommer les sous-dossiers :
  ```bash
  mv .speiros/context-engineering/audit-codebase-product-artifacts .speiros/artifacts/audit-product
  mv .speiros/context-engineering/audit-codebase-technical-artifacts .speiros/artifacts/audit-technical
  mv .speiros/context-engineering/speiros-workflow-artifacts .speiros/artifacts/speiros-workflow
  ```
- [ ] **4.3** Déplacer le README de context-engineering (s'il existe) :
  ```bash
  mv .speiros/context-engineering/README.md .speiros/artifacts/README.md
  ```
- [ ] **4.4** Supprimer le dossier `roadmap/` maintenant vide :
  ```bash
  rmdir .speiros/context-engineering/roadmap/user-stories
  rmdir .speiros/context-engineering/roadmap/epics
  rmdir .speiros/context-engineering/roadmap/briefs
  rmdir .speiros/context-engineering/roadmap
  ```
- [ ] **4.5** Vérifier que `.speiros/context-engineering/product-context/` n'existe plus (déjà supprimé selon le contexte)
- [ ] **4.6** Supprimer le dossier `context-engineering/` maintenant vide :
  ```bash
  rmdir .speiros/context-engineering
  ```

### Tâche 5: Mise à jour du workflow `audit-codebase-product.yaml` (AC: 4)

- [ ] **5.1** Ouvrir `.speiros/workflows/audit-codebase-product.yaml`
- [ ] **5.2** Remplacer toutes les occurrences de `product-context/` par `context/product/`
- [ ] **5.3** Remplacer les chemins d'artefacts :
  - `product-context/initial-summary.md` → `.speiros/artifacts/audit-product/initial-summary.md`
  - `product-context/audit-results.md` → `.speiros/artifacts/audit-product/audit-results.md`
  - `product-context/enrichment-qa.md` → `.speiros/artifacts/audit-product/enrichment-qa.md`
  - `product-context/structure-plan.md` → `.speiros/artifacts/audit-product/structure-plan.md`
  - `product-context/**/research-*.md` → `.speiros/artifacts/audit-product/research/**/*.md`
- [ ] **5.4** Mettre à jour la section `artifacts.created` pour refléter la nouvelle organisation
- [ ] **5.5** Valider la syntaxe YAML du fichier
- [ ] **5.6** Sauvegarder le fichier

### Tâche 6: Mise à jour du workflow `audit-codebase-technical.yaml` (AC: 4)

- [ ] **6.1** Ouvrir `.speiros/workflows/audit-codebase-technical.yaml`
- [ ] **6.2** Remplacer toutes les occurrences de `technical-context/` par `context/technical/`
- [ ] **6.3** Remplacer les chemins d'artefacts :
  - `.speiros/context-engineering/technical-context-artifacts/` → `.speiros/artifacts/audit-technical/`
- [ ] **6.4** Mettre à jour la section `artifacts.created` pour refléter la nouvelle organisation
- [ ] **6.5** Valider la syntaxe YAML du fichier
- [ ] **6.6** Sauvegarder le fichier

### Tâche 7: Mise à jour du workflow `speiros-workflow.yaml` (AC: 4)

- [ ] **7.1** Ouvrir `.speiros/workflows/speiros-workflow.yaml`
- [ ] **7.2** Remplacer toutes les occurrences de `product-context/` par `context/product/`
- [ ] **7.3** Remplacer toutes les occurrences de `technical-context/` par `context/technical/`
- [ ] **7.4** Remplacer les références à la roadmap :
  - `.speiros/context-engineering/roadmap/briefs/` → `roadmap/briefs/`
  - `.speiros/context-engineering/roadmap/epics/` → `roadmap/epics/`
  - `.speiros/context-engineering/roadmap/user-stories/` → `roadmap/user-stories/`
- [ ] **7.5** Remplacer les chemins d'artefacts intermédiaires :
  - `.speiros/context-engineering/speiros-workflow-artifacts/` → `.speiros/artifacts/speiros-workflow/`
- [ ] **7.6** Valider la syntaxe YAML du fichier
- [ ] **7.7** Sauvegarder le fichier

### Tâche 8: Mise à jour de l'agent `context-agent.md` (AC: 5)

- [ ] **8.1** Ouvrir `.speiros/agents/audit-codebase/context-agent.md`
- [ ] **8.2** Rechercher et remplacer dans les sections `commands` et `description` :
  - `product-context/` → `context/product/`
  - `technical-context/` → `context/technical/`
- [ ] **8.3** Mettre à jour les exemples de chemins dans les descriptions de commandes
- [ ] **8.4** Sauvegarder le fichier

### Tâche 9: Mise à jour de l'agent `sub-context-agent.md` (AC: 5)

- [ ] **9.1** Ouvrir `.speiros/agents/audit-codebase/sub-context-agent.md`
- [ ] **9.2** Rechercher et remplacer les références aux contextes :
  - `product-context/` → `context/product/`
  - `technical-context/` → `context/technical/`
- [ ] **9.3** Sauvegarder le fichier

### Tâche 10: Mise à jour des agents du workflow Speiros (AC: 5)

- [ ] **10.1** Ouvrir `.speiros/agents/speiros-workflow/product-agent.md`
- [ ] **10.2** Remplacer `product-context/` par `context/product/`
- [ ] **10.3** Remplacer les références à roadmap : `.speiros/context-engineering/roadmap/` → `roadmap/`
- [ ] **10.4** Ouvrir `.speiros/agents/speiros-workflow/spec-agent.md`
- [ ] **10.5** Remplacer `technical-context/` par `context/technical/`
- [ ] **10.6** Ouvrir `.speiros/agents/speiros-workflow/dev-agent.md`
- [ ] **10.7** Remplacer les références aux contextes et roadmap
- [ ] **10.8** Sauvegarder tous les fichiers

### Tâche 11: Mise à jour des tasks `audit-codebase-product/` (AC: 6)

- [ ] **11.1** Pour chaque fichier dans `.speiros/tasks/audit-codebase-product/` :
- [ ] **11.2** `create-product-context-folder.md` :
  - Remplacer `product-context/` par `context/product/`
  - Mettre à jour le chemin de sortie : `.speiros/artifacts/audit-product/initial-summary.md`
- [ ] **11.3** `audit-product-context.md` :
  - Remplacer les chemins d'entrée/sortie avec `.speiros/artifacts/audit-product/`
- [ ] **11.4** `enrich-product-context.md` :
  - Mettre à jour les chemins d'artefacts
- [ ] **11.5** `create-product-context-structure.md` :
  - Mettre à jour les chemins d'artefacts et de sortie finale vers `context/product/`
- [ ] **11.6** `delegate-product-context.md` :
  - Mettre à jour les chemins
- [ ] **11.7** `populate-product-context.md` :
  - Mettre à jour le chemin de sortie finale : `context/product/**/*.md`
- [ ] **11.8** Sauvegarder tous les fichiers

### Tâche 12: Mise à jour des tasks `audit-codebase-technical/` (AC: 6)

- [ ] **12.1** Pour chaque fichier dans `.speiros/tasks/audit-codebase-technical/` :
- [ ] **12.2** `create-technical-context-folder.md` :
  - Remplacer `technical-context/` par `context/technical/`
  - Mettre à jour le chemin de sortie : `.speiros/artifacts/audit-technical/initial-summary.md`
- [ ] **12.3** `audit-technical-context.md` :
  - Remplacer les chemins d'entrée/sortie avec `.speiros/artifacts/audit-technical/`
- [ ] **12.4** `enrich-technical-context.md` :
  - Mettre à jour les chemins d'artefacts
- [ ] **12.5** `create-technical-context-structure.md` :
  - Mettre à jour les chemins d'artefacts et de sortie finale vers `context/technical/`
- [ ] **12.6** `delegate-technical-context.md` :
  - Mettre à jour les chemins
- [ ] **12.7** `populate-technical-context.md` :
  - Mettre à jour le chemin de sortie finale : `context/technical/**/*.md`
- [ ] **12.8** Sauvegarder tous les fichiers

### Tâche 13: Mise à jour des tasks `speiros-workflow/` (AC: 6)

- [ ] **13.1** Pour chaque fichier dans `.speiros/tasks/speiros-workflow/` :
- [ ] **13.2** `create-brief.md` :
  - Remplacer `.speiros/context-engineering/roadmap/briefs/` par `roadmap/briefs/`
  - Remplacer `.speiros/context-engineering/speiros-workflow-artifacts/briefs/` par `.speiros/artifacts/speiros-workflow/briefs/`
- [ ] **13.3** `fetch-product-context.md` :
  - Remplacer `product-context/` par `context/product/`
- [ ] **13.4** `fetch-technical-context.md` :
  - Remplacer `technical-context/` par `context/technical/`
- [ ] **13.5** `create-functional-story.md` et `create-technical-story.md` :
  - Remplacer les références roadmap : `.speiros/context-engineering/roadmap/user-stories/` → `roadmap/user-stories/`
  - Remplacer les références artefacts : `.speiros/context-engineering/speiros-workflow-artifacts/` → `.speiros/artifacts/speiros-workflow/`
- [ ] **13.6** `implement-user-story.md` :
  - Vérifier les références aux contextes
- [ ] **13.7** `update-adr.md` :
  - Remplacer `technical-context/memory/` par `context/architecture-decision-records/`
- [ ] **13.8** Sauvegarder tous les fichiers

### Tâche 14: Création de `/context/README.md` (AC: 7)

- [ ] **14.1** Créer le fichier `/context/README.md`
- [ ] **14.2** Ajouter la section "Vue d'Ensemble" expliquant les deux types de contexte
- [ ] **14.3** Ajouter la section "Structure" avec l'arborescence
- [ ] **14.4** Ajouter le tableau comparatif Product vs Technical
- [ ] **14.5** Ajouter la section "Comment Naviguer"
- [ ] **14.6** Ajouter la section "Comment Mettre à Jour" (via workflows)
- [ ] **14.7** Ajouter la section "Relation avec les Workflows Speiros"
- [ ] **14.8** Ajouter la section "Resources" avec liens vers workflows et templates
- [ ] **14.9** Sauvegarder le fichier

### Tâche 15: Création de `/roadmap/README.md` (AC: 7)

- [ ] **15.1** Créer le fichier `/roadmap/README.md`
- [ ] **15.2** Ajouter la section "Vue d'Ensemble" de la roadmap
- [ ] **15.3** Ajouter la section "Structure" expliquant briefs/epics/user-stories
- [ ] **15.4** Ajouter la section "Workflow de Création" (Product → Spec → Dev)
- [ ] **15.5** Ajouter la section "Conventions de Nommage"
  - Briefs : `brief-YYYY-MM-DD-nom-court.md`
  - Epics : `epic-X-nom-epic.md`
  - User Stories : `user-story-X.Y-nom-story.md`
- [ ] **15.6** Ajouter la section "États et Suivi" (Ready, In Progress, Done, etc.)
- [ ] **15.7** Ajouter la section "Lien avec le Workflow Speiros"
- [ ] **15.8** Sauvegarder le fichier

### Tâche 16: Création de `.speiros/artifacts/README.md` (AC: 7)

- [ ] **16.1** Créer le fichier `.speiros/artifacts/README.md`
- [ ] **16.2** Ajouter la section "Qu'est-ce qu'un Artefact Intermédiaire ?"
- [ ] **16.3** Ajouter la section "Organisation des Artefacts" avec l'arborescence
- [ ] **16.4** Créer un tableau des workflows et leurs artefacts :
      | Workflow | Dossier Artefacts | Livrable Final |
      |----------|-------------------|----------------|
      | audit-codebase-product | `.speiros/artifacts/audit-product/` | `/context/product/` |
      | audit-codebase-technical | `.speiros/artifacts/audit-technical/` | `/context/technical/` |
      | speiros-workflow | `.speiros/artifacts/speiros-workflow/` | `/roadmap/` |
- [ ] **16.5** Ajouter la section "Quand Consulter ces Artefacts ?"
- [ ] **16.6** Ajouter la section "Différence avec les Livrables Finaux"
- [ ] **16.7** Sauvegarder le fichier

### Tâche 17: Mise à jour de `.speiros/README.md` (AC: 8)

- [ ] **17.1** Ouvrir `.speiros/README.md`
- [ ] **17.2** Mettre à jour la section "File Structure" avec la nouvelle organisation
- [ ] **17.3** Remplacer toutes les occurrences de `product-context/` par `context/product/`
- [ ] **17.4** Remplacer toutes les occurrences de `technical-context/` par `context/technical/`
- [ ] **17.5** Remplacer les références à roadmap : `.speiros/context-engineering/roadmap/` → `roadmap/`
- [ ] **17.6** Mettre à jour les exemples de commandes avec les nouveaux chemins
- [ ] **17.7** Mettre à jour le diagramme mermaid (s'il existe)
- [ ] **17.8** Ajouter une section "Organisation des Livrables" expliquant :
  - `/context/` = Documentation de référence
  - `/roadmap/` = Développement actif
  - `.speiros/` = Configuration et artefacts intermédiaires
- [ ] **17.9** Sauvegarder le fichier

### Tâche 18: Mise à jour des README des contextes (AC: 1, 7)

- [ ] **18.1** Ouvrir `context/product/README.md`
- [ ] **18.2** Mettre à jour toutes les références de chemins :
  - `.speiros/context-engineering/audit-codebase-product-artifacts/` → `.speiros/artifacts/audit-product/`
  - `product-context/` → `context/product/`
- [ ] **18.3** Mettre à jour la section "Structure" pour refléter le nouveau dossier parent
- [ ] **18.4** Sauvegarder le fichier
- [ ] **18.5** Ouvrir `context/technical/README.md`
- [ ] **18.6** Mettre à jour toutes les références de chemins :
  - `.speiros/context-engineering/technical-context-artifacts/` → `.speiros/artifacts/audit-technical/`
  - `technical-context/` → `context/technical/`
- [ ] **18.7** Sauvegarder le fichier

### Tâche 19: Validation globale (AC: 10)

- [ ] **19.1** Exécuter la recherche de références obsolètes :
  ```bash
  find . -type f \( -name "*.md" -o -name "*.yaml" \) ! -path "*/node_modules/*" ! -path "*/.git/*" -exec grep -l "product-context/" {} \;
  ```
- [ ] **19.2** Vérifier qu'aucune référence à `technical-context/` ne subsiste (hors backup)
- [ ] **19.3** Vérifier qu'aucune référence à `context-engineering/roadmap` ne subsiste
- [ ] **19.4** Vérifier qu'aucune référence à `context-engineering/speiros-workflow-artifacts` ne subsiste
- [ ] **19.5** Tester la syntaxe YAML de tous les workflows :
  ```bash
  npx js-yaml .speiros/workflows/audit-codebase-product.yaml
  npx js-yaml .speiros/workflows/audit-codebase-technical.yaml
  npx js-yaml .speiros/workflows/speiros-workflow.yaml
  ```
- [ ] **19.6** Vérifier l'intégrité des fichiers déplacés (nombre de lignes identique)
- [ ] **19.7** Lister tous les fichiers non suivis par Git et vérifier qu'il n'y a pas d'orphelins

### Tâche 20: Tests et Validation Finale (AC: 10)

- [ ] **20.1** Créer un script de validation :

  ```bash
  #!/bin/bash
  echo "=== Validation de la Restructuration Speiros ==="

  # Test 1: Vérifier la structure
  [ -d "context/product" ] && echo "✅ context/product/ existe" || echo "❌ context/product/ manquant"
  [ -d "context/technical" ] && echo "✅ context/technical/ existe" || echo "❌ context/technical/ manquant"
  [ -d "roadmap" ] && echo "✅ roadmap/ existe" || echo "❌ roadmap/ manquant"
  [ -d ".speiros/artifacts" ] && echo "✅ .speiros/artifacts/ existe" || echo "❌ .speiros/artifacts/ manquant"

  # Test 2: Vérifier la suppression
  [ ! -d "product-context" ] && echo "✅ product-context/ supprimé" || echo "❌ product-context/ existe encore"
  [ ! -d "technical-context" ] && echo "✅ technical-context/ supprimé" || echo "❌ technical-context/ existe encore"
  [ ! -d ".speiros/context-engineering" ] && echo "✅ context-engineering/ supprimé" || echo "❌ context-engineering/ existe encore"

  # Test 3: Compter les user stories
  STORIES_COUNT=$(ls roadmap/user-stories/*.md 2>/dev/null | wc -l)
  echo "📊 Nombre de user stories: $STORIES_COUNT"

  # Test 4: Rechercher des références obsolètes
  echo ""
  echo "🔍 Recherche de références obsolètes..."
  OBSOLETE=$(find . -type f \( -name "*.md" -o -name "*.yaml" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/backup-*" -exec grep -l "product-context/\|technical-context/\|context-engineering/" {} \; 2>/dev/null | wc -l)

  if [ "$OBSOLETE" -eq 0 ]; then
    echo "✅ Aucune référence obsolète trouvée"
  else
    echo "⚠️  $OBSOLETE fichier(s) avec des références obsolètes"
    find . -type f \( -name "*.md" -o -name "*.yaml" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/backup-*" -exec grep -l "product-context/\|technical-context/\|context-engineering/" {} \; 2>/dev/null
  fi

  echo ""
  echo "=== Validation Terminée ==="
  ```

- [ ] **20.2** Exécuter le script de validation
- [ ] **20.3** Corriger les problèmes détectés
- [ ] **20.4** Relancer le script jusqu'à ce que tous les tests passent

### Tâche 21: Documentation et Commit (AC: Tous)

- [ ] **21.1** Mettre à jour le README principal du projet (`/README.md`) pour mentionner la nouvelle structure
- [ ] **21.2** Créer un fichier `CHANGELOG.md` ou mettre à jour l'existant avec cette restructuration
- [ ] **21.3** Vérifier le statut Git :
  ```bash
  git status
  ```
- [ ] **21.4** Ajouter tous les fichiers :
  ```bash
  git add -A
  ```
- [ ] **21.5** Commiter avec un message descriptif :

  ```bash
  git commit -m "refactor(speiros): restructure deliverables organization (US-3.1)

  - Move product-context/ to context/product/
  - Move technical-context/ to context/technical/
  - Move .speiros/context-engineering/roadmap/ to roadmap/
  - Rename .speiros/context-engineering/ to .speiros/artifacts/
  - Update all references in workflows, agents, and tasks
  - Add comprehensive README files for context/, roadmap/, and artifacts/

  Breaking Changes:
  - All paths referencing product-context/, technical-context/, and context-engineering/ need to be updated
  - Agents and workflows now use new paths

  Closes US-3.1"
  ```

- [ ] **21.6** Créer une Pull Request avec description détaillée
- [ ] **21.7** Demander une review

## Documentation

### Template `/context/README.md`

````markdown
# Context - Documentation de Référence

Ce dossier contient toute la documentation de référence du projet, organisée en deux catégories principales :

- **Product** (`/context/product/`) - Documentation fonctionnelle
- **Technical** (`/context/technical/`) - Documentation technique

## Vue d'Ensemble

| Type          | Contenu                                       | Audience                          | Mise à Jour                       |
| ------------- | --------------------------------------------- | --------------------------------- | --------------------------------- |
| **Product**   | Fonctionnalités, cas d'usage, règles métier   | PM, Product Agent, équipe métier  | Après features majeures           |
| **Technical** | Architecture, outils, patterns, configuration | Dev, Spec Agent, équipe technique | Après refactoring/nouveaux outils |

## Structure

\`\`\`
context/
├── README.md (ce fichier)
├── product/ # Documentation produit
│ ├── authentication/
│ ├── organization/
│ ├── github-integration/
│ ├── user-management/
│ ├── administration/
│ ├── repository-dashboard/
│ └── README.md
└── technical/ # Documentation technique
├── context/ # Domaines techniques (API, DB, Tests, etc.)
├── tools/ # Outils externes (Clerk, Drizzle, Sentry, etc.)
├── architecture-decision-records/ # Décisions techniques et troubleshooting
└── README.md
\`\`\`

## Contexte Produit (`product/`)

Le contexte produit répond à la question : **"Que fait le produit ?"**

- Cas d'usage et user journeys
- Fonctionnalités clés et leur valeur métier
- Règles métier et contraintes business
- Flux utilisateur détaillés
- Dépendances fonctionnelles

**Exemple** : `context/product/authentication/context.md` documente les parcours de login, les règles de validation, et les cas d'usage OAuth.

## Contexte Technique (`technical/`)

Le contexte technique répond à la question : **"Comment est-ce construit ?"**

- Architecture du système
- Patterns et best practices
- Configuration des outils
- APIs et interfaces
- Stratégie de tests
- Décisions techniques historiques (ADRs)

**Exemple** : `context/technical/tools/CLERKS.md` documente la configuration de Clerk, les APIs utilisées, et les patterns de code pour l'authentification.

## Comment Naviguer

### Pour Comprendre une Fonctionnalité

1. Commencez par `context/product/{domaine}/context.md` pour comprendre **quoi** et **pourquoi**
2. Consultez `context/technical/` pour comprendre **comment** c'est implémenté

### Pour Implémenter une Nouvelle Feature

1. Le **Product Agent** extrait le contexte de `context/product/`
2. Le **Spec Agent** extrait le contexte de `context/technical/`
3. Vous recevez une user story complète avec tout le contexte nécessaire

### Pour Onboarder sur le Projet

1. Lisez `context/product/README.md` pour la vue produit
2. Lisez `context/technical/README.md` pour la vue technique
3. Explorez les domaines qui vous intéressent

## Comment Mettre à Jour

### Via les Workflows Speiros (Recommandé)

Les contextes sont générés et mis à jour automatiquement via les workflows :

\`\`\`bash

# Mettre à jour le contexte produit

@context-agent \*run-workflow audit-codebase-product

# Mettre à jour le contexte technique

@context-agent \*run-workflow audit-codebase-technical
\`\`\`

Les workflows :

1. Scannent le codebase
2. Identifient les nouveaux domaines
3. Posent des questions pour enrichir
4. Génèrent/mettent à jour les fichiers dans `context/`

**Artefacts intermédiaires** : Les workflows créent des artefacts intermédiaires dans `.speiros/artifacts/audit-product/` et `.speiros/artifacts/audit-technical/`. La documentation finale est écrite ici dans `context/`.

### Édition Manuelle

Vous pouvez éditer manuellement n'importe quel fichier `.md` dans `context/`. Cependant, vos modifications risquent d'être écrasées si vous relancez les workflows.

**Recommandation** :

- Utilisez les workflows pour l'initialisation et les mises à jour majeures
- Effectuez des ajustements mineurs manuellement
- Documentez vos changements dans les commits Git

## Relation avec les Autres Dossiers

| Dossier               | Rôle                                              | Lien avec Context                                         |
| --------------------- | ------------------------------------------------- | --------------------------------------------------------- |
| `/roadmap/`           | Développement actif (briefs, epics, user stories) | Les user stories référencent le contexte pertinent        |
| `.speiros/`           | Configuration système et artefacts intermédiaires | Contient les workflows qui génèrent le contexte           |
| `.speiros/artifacts/` | Artefacts intermédiaires des workflows            | Étapes de travail avant génération finale dans `context/` |

## Workflows Speiros

Le contexte est utilisé à différentes phases du workflow Speiros :

1. **Phase Product** : Product Agent lit `context/product/` pour créer des user stories fonctionnelles
2. **Phase Spec** : Spec Agent lit `context/technical/` pour créer des plans d'implémentation
3. **Phase Dev** : Dev Agent reçoit tout le contexte dans la user story (pas besoin de lire directement)

Après implémentation, le **Dev Agent** peut mettre à jour `context/architecture-decision-records/` avec les nouvelles décisions techniques.

## Resources

- **Workflows** : `.speiros/workflows/audit-codebase-product.yaml`, `audit-codebase-technical.yaml`
- **Templates** : `.speiros/templates/audit-codebase/`
- **Agents** : `.speiros/agents/audit-codebase/`
- **Guide détaillé** : `.speiros/artifacts/README.md`

## Statut Actuel

**Product Context** :

- ✅ 6 domaines documentés (Authentication, Organization, GitHub Integration, User Management, Administration, Repository Dashboard)
- Dernière mise à jour : Voir `context/product/README.md`

**Technical Context** :

- ✅ 11 domaines techniques documentés
- ✅ 7 outils documentés
- ✅ 2 fichiers de mémoire (Décisions techniques, Troubleshooting)
- Dernière mise à jour : Voir `context/technical/README.md`

---

**Version:** 3.0 (Restructuration US-3.1)  
**Last Updated:** 2025-10-16
\`\`\`

### Template `/roadmap/README.md`

```markdown
# Roadmap - Planification et Développement

Ce dossier contient tous les artefacts de planification et développement du projet, organisés selon la méthodologie Speiros.

## Vue d'Ensemble

La roadmap suit une hiérarchie à trois niveaux :

1. **Briefs** - Demandes initiales de features ou problèmes à résoudre
2. **Epics** - Groupements de user stories liées (fonctionnalités majeures)
3. **User Stories** - Unités de travail détaillées avec spécifications techniques

## Structure

\`\`\`
roadmap/
├── README.md (ce fichier)
├── briefs/
│ └── brief-YYYY-MM-DD-nom.md
├── epics/
│ └── epic-X-nom-epic.md
└── user-stories/
├── user-story-X.Y-nom-story.md
├── user-story-X.Y.Z-sous-story.md
└── ...
\`\`\`

## Workflow de Création

### 1. Brief → Epic → User Story

\`\`\`mermaid
graph LR
A[Brief] --> B[Epic]
B --> C[User Story 1]
B --> D[User Story 2]
B --> E[User Story N]
C --> F[Implémentation]
D --> F
E --> F
\`\`\`

### 2. Workflow Speiros

Le système Speiros transforme un brief en code via trois phases :

\`\`\`
Brief → [Product Agent] → Functional Story
→ [Spec Agent] → Technical Story
→ [Dev Agent] → Code + Tests + ADR Update
\`\`\`

**Commandes** :

\`\`\`bash

# Phase 1: Créer la story fonctionnelle

@product-agent \*run-workflow

# Phase 2: Ajouter les spécifications techniques

@spec-agent \*run-workflow

# Phase 3: Implémenter

@dev-agent \*run-workflow
\`\`\`

## Conventions de Nommage

### Briefs

Format : `brief-YYYY-MM-DD-nom-court.md`

Exemples :

- `brief-2025-10-15-triple-panel-view.md`
- `brief-2025-10-16-realtime-notifications.md`

### Epics

Format : `epic-X-nom-epic.md`

Où `X` est un numéro séquentiel.

Exemples :

- `epic-1-triple-panel-interface.md`
- `epic-2-context-engineering-system.md`
- `epic-3-realtime-features.md`

### User Stories

Format : `user-story-X.Y-nom-story.md`

Où :

- `X` = Numéro de l'epic parent
- `Y` = Numéro séquentiel dans l'epic

Pour les sous-stories : `user-story-X.Y.Z-nom-sous-story.md`

Exemples :

- `user-story-1.1-frontend-triple-panel.md`
- `user-story-1.2-backend-markdown-api.md`
- `user-story-2.1-audit-codebase-product-workflow.md`
- `user-story-2.1.1-create-product-template.md` (sous-story)

## États et Suivi

Les user stories suivent ces états :

| État                         | Description                  | Marqueur |
| ---------------------------- | ---------------------------- | -------- |
| **Draft**                    | En cours de rédaction        | 📝       |
| **Ready for Review**         | Prêt pour relecture          | 👀       |
| **Ready for Implementation** | Approuvé, prêt à implémenter | ✅       |
| **In Progress**              | En cours d'implémentation    | 🚧       |
| **Done**                     | Implémenté et testé          | ✔️       |
| **Blocked**                  | Bloqué par une dépendance    | 🚫       |
| **Cancelled**                | Annulé                       | ❌       |

**Dans les fichiers** :

\`\`\`yaml

## Métadonnées

- **Statut**: Ready for Implementation
  \`\`\`

## Organisation par Epic

Utilisez le préfixe numérique pour regrouper les stories par epic :

\`\`\`
user-stories/
├── user-story-1.1-_.md # Epic 1
├── user-story-1.2-_.md
├── user-story-1.3-_.md
├── user-story-2.1-_.md # Epic 2
├── user-story-2.2-_.md
├── user-story-3.1-_.md # Epic 3
└── ...
\`\`\`

## Lien avec le Workflow Speiros

### Artefacts Intermédiaires

Pendant le développement d'une story, des artefacts intermédiaires sont créés dans :

**`.speiros/artifacts/speiros-workflow/`**

Ces artefacts incluent :

- Versions de travail du brief
- Itérations de la story durant les phases Product/Spec
- Notes et recherches

### Livrables Finaux

Une fois la story complétée, les livrables finaux sont :

- **User Story finale** → Sauvegardée dans `roadmap/user-stories/`
- **Code implémenté** → Dans les dossiers sources du projet (`apps/`, `packages/`)
- **Décisions techniques** → Documentées dans `context/architecture-decision-records/`
- **Contexte mis à jour** → Si nécessaire, mis à jour dans `context/`

## Référencement du Contexte

Les user stories doivent référencer le contexte pertinent :

### Référencement du Contexte Produit

\`\`\`markdown

## Contexte Produit

Cette story impacte les domaines suivants :

- \`context/product/authentication/\` - Gestion de l'authentification
- \`context/product/organization/\` - Hiérarchie des organisations
  \`\`\`

### Référencement du Contexte Technique

\`\`\`markdown

## Contexte Technique

Cette story utilise les composants techniques suivants :

- \`context/technical/tools/CLERKS.md\` - Service d'authentification
- \`context/technical/context/API.md\` - Patterns API
  \`\`\`

## Création d'une Nouvelle Story

### Méthode 1 : Via Speiros (Recommandé)

\`\`\`bash

# 1. Créer un brief

@product-agent \*create-brief

# > "Je veux ajouter des notifications en temps réel"

# 2. Transformer en user story

@product-agent \*run-workflow

# 3. Ajouter les specs techniques

@spec-agent \*run-workflow

# 4. Implémenter

@dev-agent \*run-workflow
\`\`\`

### Méthode 2 : Manuellement

1. Dupliquer un template de user story existant
2. Remplir les métadonnées (ID, titre, epic, statut, priorité, estimation)
3. Rédiger la description et les objectifs
4. Définir les Acceptance Criteria (AC)
5. Décomposer en tasks/subtasks
6. Sauvegarder dans `roadmap/user-stories/`

## Priorisation

Utilisez le champ "Priorité" dans les métadonnées :

- **Critique** : Bugs bloquants, failles de sécurité
- **Haute** : Features core, refactorings majeurs
- **Moyenne** : Améliorations, optimisations
- **Basse** : Nice-to-have, polish

## Estimation

Utilisez la suite de Fibonacci pour l'estimation :

- **1 point** : Très simple (< 1h)
- **2 points** : Simple (1-2h)
- **3 points** : Moyen (2-4h)
- **5 points** : Complexe (1 jour)
- **8 points** : Très complexe (2 jours)
- **13 points** : Epic (à décomposer)

Si une story > 8 points, décomposez-la en sous-stories.

## Resources

- **Workflows** : `.speiros/workflows/speiros-workflow.yaml`
- **Agents** : `.speiros/agents/speiros-workflow/`
- **Templates** : `.speiros/templates/speiros-workflow/`
- **Artefacts intermédiaires** : `.speiros/artifacts/speiros-workflow/`

## Statistiques

\`\`\`bash

# Compter les stories par état

grep -r "Statut:" roadmap/user-stories/ | sort | uniq -c

# Compter les stories par epic

ls roadmap/user-stories/user-story-\*.md | cut -d'-' -f3 | cut -d'.' -f1 | sort | uniq -c
\`\`\`

---

**Version:** 3.0 (Restructuration US-3.1)  
**Last Updated:** 2025-10-16
\`\`\`

## Définition de "Terminé" (DoD)

- [ ] La structure complète est créée et validée (AC: 1, 2, 3)
- [ ] Tous les anciens dossiers sont supprimés (AC: 9)
- [ ] Tous les workflows référencent les nouveaux chemins (AC: 4)
- [ ] Tous les agents référencent les nouveaux chemins (AC: 5)
- [ ] Toutes les tasks référencent les nouveaux chemins (AC: 6)
- [ ] Les trois README principaux existent et sont complets (AC: 7)
- [ ] Le README de `.speiros/` est mis à jour (AC: 8)
- [ ] Les README des contextes sont mis à jour (AC: 7)
- [ ] Aucune référence obsolète ne subsiste (AC: 10)
- [ ] Tous les tests de validation passent (AC: 10)
- [ ] Le commit est effectué avec un message clair
- [ ] Une PR est créée avec description détaillée

## Notes Techniques

### Points d'Attention

1. **Git et fichiers déplacés** : Utiliser `git mv` plutôt que `mv` pour préserver l'historique
2. **Recherche exhaustive** : S'assurer qu'aucune référence obsolète ne subsiste, y compris dans les commentaires
3. **Validation YAML** : Tous les workflows doivent être syntaxiquement valides après modification
4. **Backup** : Garder un backup avant de commencer (étape Tâche 1.4)
5. **Tests progressifs** : Tester après chaque grande étape plutôt qu'à la fin

### Ordre d'Exécution Recommandé

1. **Backup** (Tâche 1)
2. **Création des nouvelles structures** (Tâches 2, 3, 4) - Sans supprimer les anciennes encore
3. **Mise à jour des fichiers de configuration** (Tâches 5-13)
4. **Création des README** (Tâches 14-18)
5. **Validation** (Tâches 19-20)
6. **Suppression des anciennes structures** (Implicite dans Tâche 2, 3, 4)
7. **Commit** (Tâche 21)

### Commandes Git Utiles

\`\`\`bash

# Déplacer en préservant l'historique

git mv product-context context/product

# Voir les fichiers renommés

git status

# Voir le diff des fichiers modifiés

git diff .speiros/workflows/

# Vérifier qu'aucun fichier n'est orphelin

git status --short | grep "^??"
\`\`\`

## Risques et Mitigation

| Risque                                 | Impact   | Probabilité | Mitigation                                       |
| -------------------------------------- | -------- | ----------- | ------------------------------------------------ |
| Références obsolètes non détectées     | Élevé    | Moyen       | Scripts de validation automatisés (Tâche 19, 20) |
| Perte de données durant le déplacement | Critique | Faible      | Backup complet avant début (Tâche 1.4)           |
| Workflows cassés après restructuration | Élevé    | Moyen       | Tests YAML + validation manuelle (Tâche 19.5)    |
| Confusion durant la transition         | Moyen    | Élevé       | README détaillés + documentation de la migration |
| Historique Git perdu                   | Moyen    | Faible      | Utiliser `git mv` au lieu de `mv`                |

## Dépendances

- Aucune dépendance externe (refactoring interne uniquement)
- Tous les workflows, agents, et tasks existants doivent être mis à jour
- Cette story doit être complétée avant toute autre story utilisant les anciens chemins

## Évolutions Futures

- Automatiser la détection de références obsolètes (pre-commit hook)
- Créer un script de migration pour faciliter les futures restructurations
- Ajouter des tests d'intégration pour valider les chemins des workflows
- Documenter la structure dans un diagramme d'architecture

## Annexes

### Annexe A : Mapping Complet des Chemins

| Ancien Chemin                                                      | Nouveau Chemin                         | Type                |
| ------------------------------------------------------------------ | -------------------------------------- | ------------------- |
| `/product-context/`                                                | `/context/product/`                    | Dossier de contexte |
| `/technical-context/`                                              | `/context/technical/`                  | Dossier de contexte |
| `.speiros/context-engineering/`                                    | `.speiros/artifacts/`                  | Dossier d'artefacts |
| `.speiros/context-engineering/roadmap/`                            | `/roadmap/`                            | Dossier de roadmap  |
| `.speiros/context-engineering/audit-codebase-product-artifacts/`   | `.speiros/artifacts/audit-product/`    | Artefacts audit     |
| `.speiros/context-engineering/audit-codebase-technical-artifacts/` | `.speiros/artifacts/audit-technical/`  | Artefacts audit     |
| `.speiros/context-engineering/speiros-workflow-artifacts/`         | `.speiros/artifacts/speiros-workflow/` | Artefacts workflow  |
| `.speiros/context-engineering/product-context/`                    | _SUPPRIMÉ_                             | Dossier vide        |

### Annexe B : Checklist de Validation Post-Migration

\`\`\`
[ ] Structure des dossiers validée
[ ] Tous les workflows s'exécutent sans erreur
[ ] Tous les agents s'activent correctement
[ ] Aucune référence obsolète détectée
[ ] Les trois README principaux existent
[ ] Les README des contextes sont mis à jour
[ ] Le README de .speiros/ est mis à jour
[ ] Backup créé et archivé
[ ] Tests YAML passent
[ ] Tests de chemins passent
[ ] PR créée et approuvée
[ ] Branche mergée dans main
\`\`\`

---

**Version:** 1.0  
**Créé le:** 2025-10-16  
**Auteur:** Lucas Gaillard
```
````
