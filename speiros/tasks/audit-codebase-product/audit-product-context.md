# Task: Audit Product Context

## Objective

Analyze initial summary to identify ideal documentation structure and information gaps.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-product/initial-summary.md`

## Steps

1. **Analyze what we have**
   - Review initial summary completeness
   - Identify: well-documented areas
   - Note: vague or missing information

2. **Identify product domains**
   - Based on features, determine logical groupings
   - Examples: Auth, Dashboard, Integrations, Admin
   - Map features to domains

3. **Define ideal structure**
   - For each domain, list required information:
     - Use cases
     - User flows
     - Business rules
     - Dependencies
   - Determine file hierarchy

4. **List information gaps**
   - What's missing from initial summary?
   - Which domains need deep research?
   - Which areas need user input?

5. **Create audit results**
   - Write to `.speiros/artifacts/audit-product/audit-results.md`
   - Structure:
     - Identified Domains (list)
     - Proposed File Structure (tree)
     - Information Gaps (categorized)
     - Recommended Next Steps

## Output

- `.speiros/artifacts/audit-product/audit-results.md`

## Success Criteria

- Clear domain categorization
- Realistic file structure plan
- Specific gaps identified
- Ready for enrichment phase
