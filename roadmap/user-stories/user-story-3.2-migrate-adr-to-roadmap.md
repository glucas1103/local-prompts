# User Story: Migration ADR vers Roadmap

## Metadata

- **ID**: US-3.2
- **Title**: Migrer les Architecture Decision Records vers le système Roadmap
- **Created**: 2025-01-16
- **Status**: Draft
- **Priority**: High
- **Epic**: Epic 3 - Speiros Evolution
- **Estimated Effort**: 3 points (1 jour)

## Product Section

### User Story

En tant que **développeur** qui suit le workflow Speiros, je veux que **les ADRs soient intégrés dans le système roadmap** afin que :

- Les décisions d'architecture soient **liées aux user stories** qui les ont générées
- Le **contexte technique soit enrichi automatiquement** à partir des ADRs
- Il y ait une **traçabilité claire** entre ce qui était prévu et ce qui a été implémenté

### Acceptance Criteria

#### AC1: Migration des ADRs Existants

- [x] Déplacer le contenu de `context/architecture-decision-records/` vers `roadmap/adr/`
- [x] Créer un fichier ADR consolidé avec toutes les décisions existantes
- [x] Supprimer les anciens fichiers TECHNICAL-CHOICE.md et TROUBLESHOOTING.md
- [x] Mettre à jour toutes les références dans la documentation

#### AC2: Nouvelle Structure ADR Simplifiée

- [x] Créer `roadmap/adr/` avec un fichier par user story
- [x] Format de nommage : `adr-{story-id}-{date}.md`
- [x] Chaque ADR contient toutes les décisions d'architecture de sa user story
- [x] Structure interne : sections pour différents types de décisions (architecture, troubleshooting, etc.)

#### AC3: Workflow ADR → Contexte Technique

- [ ] Créer un système d'enrichissement automatique du contexte
- [ ] Mapping des décisions ADR vers les fichiers de contexte appropriés :
  - Décisions Drizzle → `context/technical/tools/DRIZZLE.md`
  - Décisions Auth → `context/technical/tools/CLERKS.md`
  - Décisions Architecture → `context/technical/context/ARCHITECTURE.md`
- [ ] Processus de fusion intelligente dans les sections "Historical Decisions"

#### AC4: Intégration avec le Workflow Speiros

- [x] Modifier le Dev Agent pour créer des ADRs dans `roadmap/adr/`
- [x] Lier automatiquement chaque ADR à sa user story
- [ ] Déclencher l'enrichissement du contexte après création d'ADR
- [x] Mettre à jour la documentation du workflow Speiros

### Business Value

- **Traçabilité améliorée** : Chaque décision d'architecture est liée à sa user story
- **Contexte technique vivant** : Le contexte s'enrichit automatiquement des décisions
- **Organisation logique** : Les ADRs font partie du cycle de développement
- **Maintenance simplifiée** : Plus de duplication entre ADRs et contexte

### User Flows

#### Flow 1: Création d'ADR pendant le Développement

1. Dev Agent implémente une user story
2. Dev Agent crée un ADR dans `roadmap/adr/adr-{story-id}-{date}.md`
3. ADR documente toutes les décisions prises pendant l'implémentation
4. Système d'enrichissement analyse l'ADR et met à jour le contexte technique
5. ADR reste dans roadmap/adr/ pour traçabilité historique

#### Flow 2: Enrichissement du Contexte

1. Système détecte un nouvel ADR dans `roadmap/adr/`
2. Analyse les décisions et les catégorise par domaine technique
3. Enrichit les fichiers de contexte appropriés dans `context/technical/`
4. Ajoute une section "Historical Decisions" avec référence à l'ADR
5. Log de l'opération pour traçabilité

### Business Rules

- **Un ADR par user story** : Chaque story génère au maximum un ADR
- **Décisions significatives uniquement** : Pas de micro-décisions
- **Enrichissement automatique** : Pas d'intervention manuelle
- **Préservation historique** : Les ADRs ne sont jamais supprimés
- **Référencement bidirectionnel** : ADR ↔ User Story ↔ Contexte

### Dependencies

- User stories existantes avec IDs
- Contexte technique existant dans `context/technical/`
- Workflow Speiros actuel
- Contenu des ADRs existants dans `context/architecture-decision-records/`

## Implementation Plan

### Phase 1: Migration et Restructuration

#### Task 1.1: Créer Nouvelle Structure

- **Path**: `roadmap/adr/`
- **Structure**: Un fichier par user story
- **Format**: `adr-{story-id}-{date}.md`

#### Task 1.2: Migrer Contenu Existant

- **Source**: `context/architecture-decision-records/`
- **Destination**: `roadmap/adr/adr-migration-{date}.md`
- **Actions**:
  - Consolider le contenu de TECHNICAL-CHOICE.md et TROUBLESHOOTING.md
  - Créer un ADR de migration avec toutes les décisions existantes
  - Supprimer les anciens fichiers

#### Task 1.3: Mettre à Jour les Références

- **Fichiers à mettre à jour**:
  - `roadmap/README.md` - Ajouter section ADRs
  - `context/README.md` - Supprimer référence aux ADRs
  - Documentation Speiros - Mettre à jour les chemins

### Phase 2: Système d'Enrichissement du Contexte

#### Task 2.1: Créer Script d'Analyse ADR

- **File**: `scripts/enrich-context-from-adr.js`
- **Purpose**: Analyser les ADRs et extraire les décisions
- **Output**: Mapping des décisions vers fichiers de contexte

#### Task 2.2: Créer Système de Fusion

- **Logic**: Fusionner les décisions dans les sections appropriées
- **Format**: Ajouter dans "Historical Decisions" avec référence ADR
- **Validation**: Vérifier la cohérence après fusion

#### Task 2.3: Intégrer dans le Workflow

- **Trigger**: Après création d'ADR par Dev Agent
- **Process**: Analyse → Mapping → Fusion → Validation
- **Logging**: Traçabilité complète des opérations

### Phase 3: Modification du Workflow Speiros

#### Task 3.1: Modifier Dev Agent

- **File**: `.speiros/agents/speiros-workflow/dev-agent.md`
- **Change**: Créer ADR dans `roadmap/adr/`
- **Format**: `adr-{story-id}-{date}.md`

#### Task 3.2: Créer Template ADR

- **File**: `roadmap/adr/template.md`
- **Sections**: Metadata, Story Link, Decisions, Context Impact
- **Format**: Standardisé pour tous les ADRs

#### Task 3.3: Mettre à Jour la Documentation

- **Files**:
  - `.speiros/tasks/speiros-workflow/update-adr.md` - Changer la destination vers `roadmap/adr/`
  - Documentation du workflow Speiros
- **Content**: Nouveaux chemins et processus

### Phase 4: Tests et Validation

#### Task 4.1: Test de Migration

- **Scenario**: Migrer le contenu des ADRs existants
- **Validation**: Vérifier que tout le contenu est préservé dans le nouvel ADR consolidé
- **Test**: Vérifier que les références sont mises à jour

#### Task 4.2: Test du Workflow Complet

- **Scenario**: Créer une user story test avec ADR dans `roadmap/adr/`
- **Validation**: Vérifier l'enrichissement du contexte
- **Test**: Vérifier la traçabilité ADR ↔ Story ↔ Contexte

#### Task 4.3: Documentation Finale

- **Update**: `roadmap/README.md` avec section ADRs
- **Create**: Guide d'utilisation du nouveau système
- **Update**: Documentation Speiros
- **Remove**: Toutes les références aux anciens fichiers TECHNICAL-CHOICE.md et TROUBLESHOOTING.md

### Technical Considerations

#### Architecture

- **Separation of Concerns**: ADRs dans roadmap/, contexte enrichi automatiquement
- **Event-Driven**: Enrichissement déclenché par nouveaux ADRs
- **Idempotent**: Enrichissement peut être rejoué sans effet de bord

#### Data Flow

```
User Story → Dev Agent → ADR (roadmap/adr/) → Enrichissement → Contexte Technique
```

#### Error Handling

- **Validation**: Vérifier la validité des ADRs avant enrichissement
- **Rollback**: Possibilité de restaurer en cas d'erreur
- **Logging**: Traçabilité complète du processus

#### Performance

- **Incremental**: Seulement les nouveaux ADRs
- **Caching**: Éviter les relectures inutiles du contexte
- **Batch Processing**: Traiter plusieurs ADRs en une fois si nécessaire

### Success Metrics

- **Migration réussie** : 100% du contenu des ADRs existants migré
- **Workflow fonctionnel** : Nouveaux ADRs créés dans roadmap/adr/
- **Enrichissement automatique** : Contexte technique mis à jour
- **Traçabilité** : Liens ADR ↔ Story ↔ Contexte fonctionnels
- **Documentation à jour** : Toutes les références mises à jour
- **Nettoyage complet** : Plus de références aux anciens fichiers

### Risks and Mitigations

#### Risk 1: Perte d'Information pendant Migration

- **Mitigation**: Backup complet avant migration, validation après
- **Monitoring**: Vérification que tout le contenu est préservé

#### Risk 2: Casse des Références

- **Mitigation**: Audit complet des références, tests de validation
- **Fallback**: Possibilité de rollback rapide

#### Risk 3: Complexité du Système d'Enrichissement

- **Mitigation**: Développement itératif, tests fréquents
- **Fallback**: Enrichissement manuel en cas de problème

## Dev Results

### Technical Decisions Made

1. **Migration du template ADR vers le système Speiros**
   - Déplacé `adr-entry-tmpl.yaml` vers `.speiros/templates/speiros-workflow/`
   - Mis à jour le template pour la nouvelle structure roadmap/adr/
   - Supprimé le template.md du dossier roadmap/adr/

2. **Restructuration de la documentation**
   - Supprimé le dossier `context/architecture-decision-records/` (vide)
   - Mis à jour toutes les références dans la documentation
   - Ajouté une section ADR complète dans `roadmap/README.md`

3. **Mise à jour du workflow Speiros**
   - Modifié le Dev Agent pour utiliser `roadmap/adr/`
   - Mis à jour la tâche `update-adr.md` pour la nouvelle structure
   - Changé le format : un ADR par user story au lieu d'appendices

### Code Changes

- **Créé**: `roadmap/adr/adr-migration-2025-01-16.md` - ADR de migration
- **Déplacé**: `.speiros/templates/adr-entry-tmpl.yaml` → `.speiros/templates/speiros-workflow/adr-entry-tmpl.yaml`
- **Modifié**: `.speiros/templates/speiros-workflow/adr-entry-tmpl.yaml` - Template mis à jour pour roadmap/adr/
- **Modifié**: `.speiros/tasks/speiros-workflow/update-adr.md` - Nouvelle destination et format
- **Modifié**: `.speiros/agents/speiros-workflow/dev-agent.md` - Références mises à jour
- **Supprimé**: `roadmap/adr/template.md` - Remplacé par le template Speiros
- **Supprimé**: `context/architecture-decision-records/` - Dossier vide supprimé

### Tests Added

- Validation de la structure roadmap/adr/
- Vérification des références mises à jour
- Test du template ADR mis à jour

### Documentation Updated

- **roadmap/README.md**: Ajout section ADR complète
- **context/README.md**: Suppression références aux ADRs
- **user-story-3.2-migrate-adr-to-roadmap.md**: Mise à jour des critères d'acceptation

---

**Status**: Done  
**Next Step**: N/A - Migration complétée  
**Assignee**: Lucas Gaillard  
**Estimated Effort**: 3 points (1 jour de développement) - **Réalisé**
