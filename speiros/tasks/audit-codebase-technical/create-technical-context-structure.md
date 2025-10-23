# Task: Create Technical Context Structure

## Objective

Build final technical context file structure plan with research assignments.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-technical/initial-summary.md`
- `.speiros/artifacts/audit-technical/audit-results.md`
- `.speiros/artifacts/audit-technical/enrichment-qa.md`
- `.speiros/templates/audit-codebase/technical-context-template.yaml`

## Steps

1. **Consolidate information**
   - Merge initial summary + enrichment Q&A
   - Map to identified technical domains
   - Prepare base data for templates

2. **Define directory structure**
   - Plan `context/technical/` hierarchy:
     - `context/` - Core technical domains (API, Database, Architecture, etc.)
     - `tools/` - Tool-specific documentation (Clerk, Drizzle, Sentry, etc.)
     - `memory/` - Historical decisions, troubleshooting, ADRs
   - No need to create directories yet (done in populate phase)

3. **Generate structure plan**
   - Write to `.speiros/artifacts/audit-technical/structure-plan.md`
   - For each technical domain:
     - Domain name and category (context/tools/memory)
     - Target file to create
     - Information needed (what to research)
     - Priority (high/medium/low)
     - Key files to examine

4. **Create research assignments**
   - List topics for sub-context-agent
   - One topic per technical domain
   - Include:
     - Search hints (directories, file patterns)
     - Key files to examine
     - Expected technical outputs
     - Template sections to fill

## Output

- `.speiros/artifacts/audit-technical/structure-plan.md`

## Success Criteria

- Clear technical domain categorization (context/tools/memory)
- Detailed research assignments ready
- Structure matches audit recommendations
- Template reference included
