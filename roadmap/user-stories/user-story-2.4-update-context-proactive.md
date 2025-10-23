# User Story 2.4 - Proactive Context Update Workflow

## Status

Draft

## Story

**As a** developer using Speiros,  
**I want** an incremental update workflow for product and technical context,  
**so that** I can keep documentation synchronized with codebase changes without recreating everything from scratch or losing manual edits.

## Context

After initial context creation with `audit-codebase-product` and `audit-codebase-technical`, the codebase evolves through new features, refactoring, and tool changes. Currently, re-running these workflows would overwrite existing documentation, losing manual improvements. We need an intelligent update workflow that:

- Detects what changed in the codebase
- Compares with existing context documentation
- Proposes only necessary updates (new domains, modified domains)
- Preserves existing documentation for unchanged areas
- Allows user review and selection before applying changes

This is a **proactive** approach where developers manually trigger updates after significant changes.

## Acceptance Criteria

1. **AC1: Existing Context Scanning**
   - Workflow scans both `product-context/` and `technical-context/` directories
   - Extracts metadata from existing files (last_update, status, version)
   - Creates inventory of documented domains with their current state

2. **AC2: Codebase Change Detection**
   - Compares current codebase structure vs documented domains
   - Detects: new domains (new features, tools, components)
   - Detects: modified domains (significant file changes >30%)
   - Detects: deprecated domains (source files deleted)
   - Generates change impact report

3. **AC3: Interactive Update Plan Review**
   - Presents user with categorized changes:
     - ✅ NEW: Domains to create
     - 🔄 UPDATE: Domains to refresh
     - 🗑️ ARCHIVE: Domains to deprecate
     - ⏭️ SKIP: Unchanged domains
   - Allows user to select/deselect individual changes
   - Allows prioritization (high/medium/low)
   - Confirms plan before execution

4. **AC4: Incremental Update Execution**
   - For NEW domains: Run full research + populate workflow
   - For UPDATE domains: Load existing file, merge new information while preserving manual edits
   - For ARCHIVE domains: Move to `archived/` folder with timestamp
   - For SKIP domains: No action taken
   - Only touches selected domains

5. **AC5: Smart Merge Strategy**
   - Auto-generated sections (elicit: false): Refresh automatically
   - User-edited sections (elicit: true): Preserve unless explicitly marked for update
   - Add new sections without removing custom content
   - Maintain formatting and cross-references

6. **AC6: Update Report Generation**
   - Generate `CHANGELOG.md` with:
     - Files created (with reason)
     - Files updated (with section-level diff summary)
     - Files archived
     - Files skipped
   - Include recommendations for next update cycle

7. **AC7: Both Contexts Updated**
   - Single workflow updates both product-context/ and technical-context/
   - Maintains consistency between product and technical views
   - Cross-references updated if domains span both contexts

8. **AC8: Idempotent and Safe**
   - Can be run multiple times safely
   - Never overwrites without user confirmation
   - Preserves manual edits and improvements
   - Can be aborted at any step

## Tasks / Subtasks

- [ ] **Task 1: Create update-context.yaml workflow** (AC: 1,2,7)
  - [ ] Define workflow structure with 5 steps
  - [ ] Configure inputs and outputs
  - [ ] Set up artifact directories in `.speiros/context-engineering/update-artifacts/`
  - [ ] Add workflow to `context-agent.md` dependencies

- [ ] **Task 2: Implement scan-existing-contexts task** (AC: 1)
  - [ ] Create `.speiros/tasks/update-context/scan-existing-contexts.md`
  - [ ] Scan product-context/ directory recursively
  - [ ] Scan technical-context/ directory recursively
  - [ ] Extract metadata from each file (parse YAML frontmatter or metadata section)
  - [ ] Build inventory map: domain → {path, last_update, status, category}
  - [ ] Output to `update-artifacts/existing-state.md`

- [ ] **Task 3: Implement detect-codebase-changes task** (AC: 2)
  - [ ] Create `.speiros/tasks/update-context/detect-codebase-changes.md`
  - [ ] Scan current codebase structure (routes, services, tools, configs)
  - [ ] Compare with existing-state inventory
  - [ ] Classify changes:
    - NEW: Domain exists in code but not in context
    - UPDATE: Domain exists but significant code changes detected
    - DEPRECATE: Domain in context but source files deleted
    - UNCHANGED: Domain exists and code stable
  - [ ] Generate impact scores (high/medium/low based on file count, LOC changed)
  - [ ] Output to `update-artifacts/detected-changes.md`

- [ ] **Task 4: Implement review-update-plan task** (AC: 3)
  - [ ] Create `.speiros/tasks/update-context/review-update-plan.md`
  - [ ] Load detected-changes report
  - [ ] Present changes to user in categorized format (NEW/UPDATE/ARCHIVE/SKIP)
  - [ ] For each change, show:
    - Domain name and category
    - Change type and reason
    - Affected files list
    - Impact score
  - [ ] Elicit user selection (accept/reject each change)
  - [ ] Elicit priority for selected changes
  - [ ] Confirm final selection
  - [ ] Output to `update-artifacts/update-plan.md`

- [ ] **Task 5: Implement execute-incremental-updates task** (AC: 4,5)
  - [ ] Create `.speiros/tasks/update-context/execute-incremental-updates.md`
  - [ ] Load update-plan
  - [ ] For each selected domain (iterative):
    - If NEW: Delegate to sub-context-agent for research, then populate template
    - If UPDATE: Load existing file, run research, merge sections intelligently
    - If ARCHIVE: Move file to `archived/{timestamp}/` with metadata
  - [ ] Smart merge logic:
    - Identify sections: auto-generated vs user-edited (check for elicit: true marker)
    - Auto-generated: Replace content
    - User-edited: Preserve unless "force-update" flag set
    - New sections: Append to file
  - [ ] Update metadata (last_update, version increment)
  - [ ] Output updated files to product-context/ and technical-context/

- [ ] **Task 6: Implement generate-update-report task** (AC: 6)
  - [ ] Create `.speiros/tasks/update-context/generate-update-report.md`
  - [ ] Collect execution results from step 5
  - [ ] Generate `update-artifacts/CHANGELOG.md`:
    - Summary statistics (X created, Y updated, Z archived, W skipped)
    - Detailed change log per file with diff summary
    - Warnings or issues encountered
    - Recommendations for next update
  - [ ] Generate `update-artifacts/update-summary.json` (machine-readable)

- [ ] **Task 7: Update context-agent.md** (AC: 8)
  - [ ] Add command: `*update-context` → Execute update-context workflow
  - [ ] Add workflow dependency: `update-context.yaml`
  - [ ] Add task dependencies (6 new tasks)
  - [ ] Update help text to explain update vs initial audit

- [ ] **Task 8: Create documentation and examples** (AC: 8)
  - [ ] Add section to `.speiros/context-engineering/README.md`:
    - When to use update vs audit
    - Example update session workflow
    - Merge strategy explanation
    - Troubleshooting common issues
  - [ ] Create example CHANGELOG output
  - [ ] Document archive structure

- [ ] **Task 9: Write tests for update workflow**
  - [ ] Test scenario: New domain detected
  - [ ] Test scenario: Existing domain updated
  - [ ] Test scenario: Domain archived
  - [ ] Test scenario: User rejects all changes (no-op)
  - [ ] Test scenario: Manual edits preserved during merge
  - [ ] Test scenario: Run twice (idempotency check)

## Dev Notes

### Workflow Structure

```yaml
workflow:
  name: Update Context (Proactive)
  id: update-context
  description: Incremental update of product and technical context
  whenToUse: After major changes, refactoring, or new features added

sequence:
  - step: 1
    name: Scan existing contexts
    agent: context-agent
    action: scan-existing-contexts
    output:
      - .speiros/context-engineering/update-artifacts/existing-state.md

  - step: 2
    name: Detect codebase changes
    agent: context-agent
    action: detect-codebase-changes
    requires:
      - .speiros/context-engineering/update-artifacts/existing-state.md
    output:
      - .speiros/context-engineering/update-artifacts/detected-changes.md

  - step: 3
    name: Review update plan
    agent: context-agent
    action: review-update-plan
    requires:
      - .speiros/context-engineering/update-artifacts/detected-changes.md
    output:
      - .speiros/context-engineering/update-artifacts/update-plan.md
    elicit: true

  - step: 4
    name: Execute incremental updates
    agent: context-agent
    action: execute-incremental-updates
    requires:
      - .speiros/context-engineering/update-artifacts/update-plan.md
    uses_subagent: sub-context-agent
    output:
      - product-context/**/*.md (only updated)
      - technical-context/**/*.md (only updated)
    iterative: true

  - step: 5
    name: Generate update report
    agent: context-agent
    action: generate-update-report
    output:
      - .speiros/context-engineering/update-artifacts/CHANGELOG.md
      - .speiros/context-engineering/update-artifacts/update-summary.json
```

### Change Detection Heuristics

**NEW Domain Detection:**

- New routes: `src/routes/{domain}.ts` not in product-context
- New tools: package.json dependency not in technical-context/tools
- New components: src/components/{domain}/ folder not documented

**UPDATE Domain Detection (significant change >30%):**

- File count changed by >2 files in domain
- Total LOC in domain changed by >30%
- New public exports added (functions, classes, types)
- Configuration file modified (package.json, tsconfig, etc.)

**DEPRECATE Domain Detection:**

- All source files listed in context file are deleted
- Domain marked as "Deprecated" in metadata for >30 days

### Smart Merge Algorithm

```typescript
function mergeSection(existing: Section, new: Section): Section {
  // Check if section is user-edited
  if (existing.metadata?.elicit === true || existing.hasManualEdits) {
    // Preserve user content, append new info as subsection
    return {
      ...existing,
      content: existing.content,
      appendix: new.content,
      updated: false
    };
  } else {
    // Auto-generated section, replace
    return {
      ...new,
      updated: true
    };
  }
}
```

### Archive Structure

```
technical-context/
├── archived/
│   ├── 2025-10-15/
│   │   └── OLD-AUTH.md
│   └── 2025-10-20/
│       └── LEGACY-API.md
```

### Testing

**Test Framework:** Vitest (already used in project)

**Test Location:** `.speiros/tasks/update-context/__tests__/`

**Key Test Cases:**

1. Mock existing context + mock codebase → Verify correct change detection
2. User selects subset of changes → Verify only selected domains updated
3. Run update twice with no code changes → Verify no changes detected (idempotent)
4. Manual edit in elicit:true section → Verify preserved after update

### Related Files

- Existing workflows: `.speiros/workflows/audit-codebase-*.yaml`
- Existing tasks: `.speiros/tasks/audit-codebase-*/`
- Templates: `.speiros/templates/*-context-template.yaml`
- Agent: `.speiros/agents/context-agent.md`

## Change Log

| Date       | Version | Description   | Author |
| ---------- | ------- | ------------- | ------ |
| 2025-10-15 | 1.0     | Initial draft | Sarah  |

## Dev Agent Record

### Agent Model Used

_To be filled during development_

### Debug Log References

_To be filled during development_

### Completion Notes List

_To be filled during development_

### File List

_To be filled during development_

## QA Results

_To be filled after implementation_
