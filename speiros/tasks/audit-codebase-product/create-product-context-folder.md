# Task: Create Product Context Folder

## Objective

Scan easily accessible documentation and create initial product context summary.

## Language Requirements

All generated content must be written in English.

## Inputs

- README.md
- Documentation files (markdown, docs/)
- Configuration files (package.json, etc.)

## Steps

1. **Scan root-level documentation**
   - Read README.md completely
   - Extract: project purpose, main features, target users
   - Note: tech stack, architecture mentions

2. **Scan configuration files**
   - package.json: name, description, scripts
   - Look for: env examples, deployment configs
   - Identify: external services referenced

3. **Scan docs/ folder if exists**
   - Read all markdown files
   - Extract: feature descriptions, user guides
   - Note: architecture diagrams, decision records

4. **Create initial summary**
   - Write to `.speiros/artifacts/audit-product/initial-summary.md`
   - Structure:
     - Project Overview (2-3 paragraphs)
     - Main Features (bullet list)
     - Target Users (who uses this?)
     - External Integrations (list of services)
     - Information Sources (files read)

## Output

- `.speiros/artifacts/audit-product/initial-summary.md`

## Success Criteria

- Summary captures core product information
- All easily accessible files scanned
- Clear, structured format
- Ready for gap analysis
