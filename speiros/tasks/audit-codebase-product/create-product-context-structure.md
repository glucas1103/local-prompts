# Task: Create Product Context Structure

## Objective

Build final product context file structure with pre-filled templates.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-product/initial-summary.md`
- `.speiros/artifacts/audit-product/audit-results.md`
- `.speiros/artifacts/audit-product/enrichment-qa.md`
- `.speiros/templates/audit-codebase/product-context-template.yaml`

## Steps

1. **Consolidate information**
   - Merge initial summary + enrichment Q&A
   - Map to identified domains
   - Prepare base data for templates

2. **Create directory structure**
   - For each domain from audit results:
     - Create `context/product/{domain}/`
     - Add `.gitkeep` if needed

3. **Generate structure plan**
   - Write to `.speiros/artifacts/audit-product/structure-plan.md`
   - For each domain:
     - Domain name
     - Target files to create
     - Information needed (what to research)
     - Priority (high/medium/low)

4. **Pre-fill templates**
   - For each domain, copy `product-context-template.yaml`
   - Fill metadata section with known info
   - Add placeholders where research needed
   - Save as `.speiros/artifacts/audit-product/templates/{domain}/context-template.yaml`

5. **Create research assignments**
   - List topics for sub-context-agent
   - One topic per domain
   - Include: search hints, key files, expected outputs

## Output

- `.speiros/artifacts/audit-product/structure-plan.md`
- `.speiros/artifacts/audit-product/templates/{domain}/context-template.yaml` (one per domain)

## Success Criteria

- All domains have dedicated folders
- Templates pre-filled with known data
- Clear research assignments ready
- Structure matches audit recommendations
