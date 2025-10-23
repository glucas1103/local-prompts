# User Story 2.5 - Reactive Context Update from Git Diff

## Status

Draft

## Story

**As a** developer or CI/CD system,  
**I want** a workflow that automatically updates context documentation based on a git diff or merged PR,  
**so that** context stays synchronized with code changes without manual intervention and can be integrated into the development workflow.

## Context

After implementing proactive context updates (US-2.4), we need a **reactive** approach triggered by actual code changes. This workflow analyzes git diffs (from PRs, commits, or branches) to determine which context domains are impacted and updates only those domains.

Key differences from proactive update:

- **Targeted**: Only analyzes files in the diff, not entire codebase
- **Automatic**: Can run in CI/CD pipeline on PR merge
- **Fast**: 2-5 min vs 10-20 min (smaller scope)
- **PR-aware**: Can comment on PRs with context impact analysis

Use cases:

1. Post-merge: Update context after PR is merged to main
2. Pre-release: Update context before creating a release
3. Manual: Developer runs on feature branch to preview impact
4. CI/CD: Automated context maintenance in GitHub Actions

## Acceptance Criteria

1. **AC1: Git Diff Parsing**
   - Accept input: branch comparison (main..feature), commit range (HEAD~5..HEAD), or PR number
   - Parse git diff to extract: files added, modified, deleted
   - Group changes by type: source files, tests, configs, docs
   - Calculate change magnitude per file (LOC added/removed)

2. **AC2: Diff-to-Domain Mapping**
   - For each changed file, determine impacted context domains:
     - New route file → product-context domain + technical-context/API.md
     - Modified tool config → technical-context/tools/{TOOL}.md
     - New test file → Update technical-context/TESTS.md examples
     - package.json dependency → technical-context/tools/{NEW_PACKAGE}.md
   - Classify impact level:
     - MAJOR: New domain needed (>5 new files or >500 LOC)
     - MINOR: Update existing domain (1-5 files or 50-500 LOC)
     - TRIVIAL: No update needed (<50 LOC)

3. **AC3: Smart Context Update Strategy**
   - NEW domain (no existing context file):
     - Run full research workflow for that domain only
     - Create new context file from template
   - MAJOR update (>30% of domain files changed):
     - Re-run research for that domain
     - Smart merge with existing file
   - MINOR update (<30% of domain files changed):
     - Append new examples/patterns to existing sections
     - Update metadata (last_update, add note about change)
   - TRIVIAL change:
     - Skip context update (log only)

4. **AC4: Context Impact Report Generation**
   - Generate `PR-CONTEXT-IMPACT.md` report containing:
     - Summary: X domains created, Y updated, Z unchanged
     - Detailed impact per domain with file links
     - Suggested manual edits (for complex changes)
     - Preview of updated sections
   - Format suitable for posting as PR comment

5. **AC5: Workflow Modes**
   - **Preview mode** (--dry-run): Generate impact report only, don't update files
   - **Auto mode**: Update context files automatically without confirmation
   - **Interactive mode**: Show impact, ask for confirmation before updating

6. **AC6: CI/CD Integration**
   - GitHub Actions workflow file provided
   - Triggers on: PR merge to main, manual workflow_dispatch
   - Runs update-context-from-diff
   - Commits updated context files with message: "chore: update context from PR #XXX"
   - Creates commit on main or opens PR with updates

7. **AC7: Performance Optimization**
   - Only scan/research domains actually impacted by diff
   - Parallel processing for multiple domain updates
   - Cache research results during single run
   - Complete update in <5 minutes for typical PR (<50 files)

8. **AC8: Rollback and Safety**
   - Validate all updated files before committing
   - Provide rollback command if update introduces issues
   - Dry-run mode always available for preview
   - Never run automatically on force-push or rebase operations

## Tasks / Subtasks

- [ ] **Task 1: Create update-context-from-diff.yaml workflow** (AC: 1,5,7)
  - [ ] Define workflow structure with 4 steps
  - [ ] Add input parameters: git_diff_source, mode (preview/auto/interactive)
  - [ ] Configure outputs in `.speiros/context-engineering/diff-artifacts/`
  - [ ] Set up parallel execution for multiple domains

- [ ] **Task 2: Implement parse-git-diff task** (AC: 1)
  - [ ] Create `.speiros/tasks/update-context-from-diff/parse-git-diff.md`
  - [ ] Accept various input formats:
    - Branch comparison: "main..feature/billing"
    - Commit range: "HEAD~5..HEAD"
    - PR number: "PR#123" (fetch diff from GitHub API)
  - [ ] Execute `git diff --stat` and `git diff --numstat` to get file changes
  - [ ] Parse diff output:
    - Files added (A), modified (M), deleted (D)
    - LOC added/removed per file
    - File types (source, test, config, doc)
  - [ ] Group changes by directory/domain
  - [ ] Calculate change magnitude scores
  - [ ] Output to `diff-artifacts/parsed-diff.md`

- [ ] **Task 3: Implement map-diff-to-domains task** (AC: 2)
  - [ ] Create `.speiros/tasks/update-context-from-diff/map-diff-to-domains.md`
  - [ ] Load parsed-diff and existing context inventory
  - [ ] Define mapping rules:
    ```
    src/routes/{domain}.ts → product-context/{domain}/ + technical-context/context/API.md
    src/services/{domain}.ts → product-context/{domain}/ + technical-context/context/ARCHITECTURE.md
    package.json (+{tool}) → technical-context/tools/{TOOL}.md
    src/middleware/{name}.ts → technical-context/context/MIDDLEWARE.md
    *.test.ts in {domain} → technical-context/TESTS.md (examples section)
    ```
  - [ ] For each changed file, determine:
    - Impacted context files (can be multiple)
    - Impact type (NEW_DOMAIN / MAJOR_UPDATE / MINOR_UPDATE / TRIVIAL)
    - Confidence score (high/medium/low)
  - [ ] Handle special cases:
    - Refactoring: File moved but logic unchanged → TRIVIAL
    - Deletion: File removed → Flag for archival review
  - [ ] Output to `diff-artifacts/impact-analysis.md`

- [ ] **Task 4: Implement smart-context-update task** (AC: 3,7)
  - [ ] Create `.speiros/tasks/update-context-from-diff/smart-context-update.md`
  - [ ] Load impact-analysis
  - [ ] For each impacted domain (process in parallel):
    - If NEW_DOMAIN:
      - Delegate to sub-context-agent for targeted research
      - Scope research to files in diff only (optimization)
      - Create new context file from template
    - If MAJOR_UPDATE:
      - Load existing context file
      - Re-run research for changed files
      - Smart merge sections (preserve user edits)
      - Update metadata (last_update, add changelog entry)
    - If MINOR_UPDATE:
      - Load existing context file
      - Extract new code examples from diff
      - Append to "Examples" section
      - Update "Last Updated" metadata
    - If TRIVIAL:
      - Log change but skip file update
  - [ ] Validate updated files (markdown syntax, no broken links)
  - [ ] Output updated files to product-context/ and technical-context/

- [ ] **Task 5: Implement generate-pr-report task** (AC: 4)
  - [ ] Create `.speiros/tasks/update-context-from-diff/generate-pr-report.md`
  - [ ] Load execution results
  - [ ] Generate `diff-artifacts/PR-CONTEXT-IMPACT.md`:
    - 📊 Summary section (statistics)
    - 📝 Detailed changes per domain
    - 🔗 Links to updated context files
    - ⚠️ Manual review suggestions (for complex changes)
    - 📦 Files changed in this update
  - [ ] Format for GitHub PR comment (use markdown tables, emojis, collapsible sections)
  - [ ] Generate `diff-artifacts/impact-summary.json` (machine-readable)

- [ ] **Task 6: Create GitHub Actions workflow** (AC: 6)
  - [ ] Create `.github/workflows/update-context.yml`
  - [ ] Trigger conditions:
    - workflow_dispatch (manual trigger with inputs)
    - pull_request: types: [closed] if merged (automatic on PR merge)
  - [ ] Steps:
    1. Checkout code
    2. Identify diff source (PR number or branch)
    3. Run update-context-from-diff workflow
    4. If preview mode: Post PR-CONTEXT-IMPACT.md as PR comment
    5. If auto mode: Commit updated files and push
  - [ ] Configure permissions (write to repo, comment on PRs)
  - [ ] Add job summary with impact report

- [ ] **Task 7: Implement workflow modes** (AC: 5)
  - [ ] Add `--mode` parameter to workflow (preview/auto/interactive)
  - [ ] Preview mode:
    - Generate impact report
    - Output to console/file
    - Exit without modifying files
  - [ ] Auto mode:
    - Run full update without confirmation
    - Suitable for CI/CD
  - [ ] Interactive mode:
    - Show impact analysis
    - Elicit user confirmation
    - Allow per-domain selection

- [ ] **Task 8: Update context-agent.md** (AC: 5)
  - [ ] Add command: `*update-from-diff {source} [mode]`
  - [ ] Examples:
    - `*update-from-diff main..feature/billing preview`
    - `*update-from-diff HEAD~3..HEAD auto`
    - `*update-from-diff PR#123 interactive`
  - [ ] Add workflow dependency: `update-context-from-diff.yaml`
  - [ ] Add task dependencies (5 new tasks)

- [ ] **Task 9: Documentation and examples** (AC: 6)
  - [ ] Add section to `.speiros/context-engineering/README.md`:
    - Diff-based update vs proactive update
    - GitHub Actions setup guide
    - Example PR comment output
    - Troubleshooting CI/CD issues
  - [ ] Create example impact reports
  - [ ] Document mapping rules (file patterns → context domains)

- [ ] **Task 10: Testing**
  - [ ] Test with real PR diffs from project history
  - [ ] Test branch comparisons (main..develop)
  - [ ] Test commit ranges (HEAD~5..HEAD)
  - [ ] Test edge cases:
    - Empty diff (no changes)
    - Large diff (>100 files)
    - Refactoring only (files moved)
    - Dependency updates only
  - [ ] Test all three modes (preview/auto/interactive)
  - [ ] Test GitHub Actions workflow in sandbox

## Dev Notes

### Workflow Structure

```yaml
workflow:
  name: Update Context from Diff
  id: update-context-from-diff
  description: Update context based on git diff or PR changes
  whenToUse: After PR merge, before release, CI/CD integration

inputs:
  - git_diff_source:
      type: string
      description: "Branch comparison, commit range, or PR number"
      examples: ["main..develop", "HEAD~5..HEAD", "PR#123"]
  - mode:
      type: choice
      choices: [preview, auto, interactive]
      default: preview

sequence:
  - step: 1
    name: Parse git diff
    agent: context-agent
    action: parse-git-diff
    input:
      - git diff {source}
    output:
      - .speiros/context-engineering/diff-artifacts/parsed-diff.md

  - step: 2
    name: Map diff to context domains
    agent: context-agent
    action: map-diff-to-domains
    requires:
      - .speiros/context-engineering/diff-artifacts/parsed-diff.md
    output:
      - .speiros/context-engineering/diff-artifacts/impact-analysis.md

  - step: 3
    name: Smart context update
    agent: context-agent
    action: smart-context-update
    requires:
      - .speiros/context-engineering/diff-artifacts/impact-analysis.md
    uses_subagent: sub-context-agent
    output:
      - product-context/**/*.md (affected only)
      - technical-context/**/*.md (affected only)
    parallel: true

  - step: 4
    name: Generate PR context report
    agent: context-agent
    action: generate-pr-report
    output:
      - .speiros/context-engineering/diff-artifacts/PR-CONTEXT-IMPACT.md
      - .speiros/context-engineering/diff-artifacts/impact-summary.json
```

### File Pattern → Context Domain Mapping

```yaml
mapping_rules:
  # Product Context
  - pattern: "src/routes/{domain}/*.ts"
    target: "product-context/{domain}/"
    impact_type: "MAJOR"

  - pattern: "src/app/(protected)/{domain}/**/*.tsx"
    target: "product-context/{domain}/"
    impact_type: "MAJOR"

  # Technical Context - API
  - pattern: "src/routes/*.ts"
    target: "technical-context/context/API.md"
    section: "Patterns & Best Practices"
    impact_type: "MINOR"

  # Technical Context - Tools
  - pattern: "package.json"
    extract: "dependencies.{tool}"
    target: "technical-context/tools/{TOOL}.md"
    impact_type: "NEW_DOMAIN" # if tool is new

  # Technical Context - Database
  - pattern: "src/db/schema.ts"
    target: "technical-context/context/DATABASE.md"
    section: "Schema Design"
    impact_type: "MAJOR"

  # Technical Context - Tests
  - pattern: "**/*.test.ts"
    target: "technical-context/context/TESTS.md"
    section: "Examples"
    impact_type: "MINOR"

  # Technical Context - Middleware
  - pattern: "src/middlewares/*.ts"
    target: "technical-context/context/MIDDLEWARE.md"
    impact_type: "MINOR"
```

### Impact Classification Algorithm

```typescript
function classifyImpact(diff: ParsedDiff, domain: string): ImpactType {
  const filesChanged = diff.getFilesInDomain(domain);
  const totalLOC = diff.getTotalLOCChanged(domain);
  const existingContext = loadContextFile(domain);

  if (!existingContext) {
    return filesChanged.length > 3 ? "NEW_DOMAIN" : "MINOR_UPDATE";
  }

  const domainFiles = getDomainSourceFiles(domain);
  const changePercentage = filesChanged.length / domainFiles.length;

  if (changePercentage > 0.3 || totalLOC > 500) {
    return "MAJOR_UPDATE";
  } else if (totalLOC > 50) {
    return "MINOR_UPDATE";
  } else {
    return "TRIVIAL";
  }
}
```

### GitHub Actions Workflow

```yaml
# .github/workflows/update-context.yml
name: Update Context from PR

on:
  pull_request:
    types: [closed]
    branches: [main, develop]
  workflow_dispatch:
    inputs:
      diff_source:
        description: "Git diff source (e.g., main..feature)"
        required: true
      mode:
        description: "Update mode"
        type: choice
        options: [preview, auto, interactive]
        default: preview

jobs:
  update-context:
    if: github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Need full history for diff

      - name: Determine diff source
        id: diff
        run: |
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            echo "source=PR#${{ github.event.pull_request.number }}" >> $GITHUB_OUTPUT
          else
            echo "source=${{ github.event.inputs.diff_source }}" >> $GITHUB_OUTPUT
          fi

      - name: Run context update
        run: |
          # This would invoke the Speiros agent
          # For now, placeholder for the actual command
          echo "@context-agent *update-from-diff ${{ steps.diff.outputs.source }} auto"

      - name: Commit updated context
        if: success()
        run: |
          git config user.name "Context Update Bot"
          git config user.email "bot@speiros.dev"
          git add product-context/ technical-context/
          git commit -m "chore: update context from ${{ steps.diff.outputs.source }}" || echo "No changes"
          git push

      - name: Post PR comment
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = fs.readFileSync('.speiros/context-engineering/diff-artifacts/PR-CONTEXT-IMPACT.md', 'utf8');

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            });
```

### Example PR Impact Report

````markdown
# 📊 Context Impact Report - PR #123

## Summary

- ✅ **2 domains created**
- 🔄 **3 domains updated**
- ⏭️ **12 domains unchanged**

---

## 📝 Detailed Changes

### ✅ NEW: `technical-context/tools/STRIPE.md`

**Reason:** New dependency detected in package.json

**Files analyzed:**

- `package.json` (+1 dependency)
- `src/services/stripe.ts` (NEW, 145 LOC)
- `src/routes/billing.ts` (NEW, 89 LOC)

**Sections created:**

- Technical Overview
- Configuration & Setup
- API & Interfaces
- Usage Examples

[View file →](../technical-context/tools/STRIPE.md)

---

### 🔄 UPDATE: `technical-context/context/API.md`

**Reason:** New API patterns detected (3 new routes)

**Files changed:**

- `src/routes/billing.ts` (NEW, 89 LOC)
- `src/routes/subscription.ts` (NEW, 67 LOC)

**Sections updated:**

- Patterns & Best Practices (added billing validation pattern)
- Usage Examples (added 2 new examples)

**Diff preview:**

```diff
+ ### Billing API Pattern
+
+ For billing routes, always use Stripe webhook signature validation:
+ ...
```
````

[View file →](../technical-context/context/API.md)

---

## ⚠️ Manual Review Suggested

1. **product-context/billing/** - New domain created with limited information
   - Consider adding user journey details
   - Business rules may need clarification

---

**Generated:** 2025-10-15 14:32:00  
**PR:** #123 - Add Stripe billing integration  
**Files in diff:** 8 changed (5 added, 3 modified)

```

### Performance Optimization

- **Targeted research**: Only scan files in diff, not entire codebase
- **Parallel processing**: Update multiple domains concurrently
- **Caching**: Share research findings across domains in single run
- **Early exit**: Skip TRIVIAL changes immediately
- **Target time**: <5 minutes for typical PR (<50 files)

### Testing Strategy

**Unit tests:**
- Diff parsing with various git output formats
- Mapping rules (file pattern → context domain)
- Impact classification algorithm

**Integration tests:**
- Real PR diffs from project history
- End-to-end workflow execution
- GitHub Actions workflow (in test repo)

**Test data:**
- Mock git diffs (small, medium, large)
- Sample PRs: feature addition, refactoring, dependency update, bug fix

### Related Files

- Proactive update workflow: `update-context.yaml` (US-2.4)
- Existing context workflows: `audit-codebase-*.yaml`
- Agent: `.speiros/agents/context-agent.md`
- GitHub Actions: `.github/workflows/`

## Change Log

| Date       | Version | Description      | Author |
| ---------- | ------- | ---------------- | ------ |
| 2025-10-15 | 1.0     | Initial draft    | Sarah  |

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

```
