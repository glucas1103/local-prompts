# Task: Audit Technical Context

## Objective

Analyze initial summary to identify ideal technical documentation structure and information gaps.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-technical/initial-summary.md`

## Steps

1. **Analyze what we have**
   - Review initial summary completeness
   - Identify: well-documented technical areas
   - Note: vague or missing technical information

2. **Identify technical domains**
   - Based on architecture, determine logical groupings
   - Examples: API, Database, Auth System, Error Handling, Tests
   - Map components to technical domains

3. **Define ideal structure**
   - For each domain, list required information:
     - Architecture & Design
     - Configuration & Setup
     - APIs & Interfaces
     - Patterns & Best Practices
     - Tests & Validation
   - Determine file hierarchy (context/, tools/, memory/)

4. **List information gaps**
   - What's missing from initial summary?
   - Which technical areas need deep research?
   - Which areas need user input (ADRs, decisions)?

5. **Create audit results**
   - Write to `.speiros/artifacts/audit-technical/audit-results.md`
   - Structure:
     - Identified Technical Domains (list)
     - Proposed File Structure (tree)
     - Information Gaps (categorized)
     - Recommended Next Steps

## Output

- `.speiros/artifacts/audit-technical/audit-results.md`

## Success Criteria

- Clear technical domain categorization
- Realistic file structure plan (context/, tools/, memory/)
- Specific technical gaps identified
- Ready for enrichment phase
