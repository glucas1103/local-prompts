# Task: Populate Product Context

## Objective

Fill product context templates with research findings to create final documentation.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-product/research/{domain}/research-*.md` (research files)
- `.speiros/templates/audit-codebase/product-context-template.yaml` (template structure)

## Steps

1. **For each domain** (iterative):

   a. **Load research data**
   - Read all `research-*.md` files for domain
   - Extract: findings, code examples, patterns
   - Organize by template section

   b. **Fill template sections**
   - **Metadata**: domain name, files, last update
   - **Overview**: synthesize from research (2-3 paragraphs)
   - **Main Use Cases**: extract from code/tests (3-5 items)
   - **Key Features**: list with business value
   - **User Flows**: document step-by-step from code
   - **Business Rules**: extract validations, constraints
   - **Functional Dependencies**: identify imports, integrations
   - **External Integrations**: list third-party services
   - **References**: link to main files, tests, docs

   c. **Handle elicit sections**
   - **Historical Decisions**: use enrichment Q&A data
   - **Future Evolutions**: use enrichment Q&A data
   - If missing: mark as "To be documented"

   d. **Generate final document**
   - Save as `context/product/{domain}/README.md`
   - Format: clean markdown, no YAML
   - Include: code snippets, file references
   - Ensure: self-contained and readable

2. **Create domain index**
   - Generate `context/product/README.md`
   - List all domains with brief descriptions
   - Link to each domain's README

3. **Validate completeness**
   - Check: all template sections filled
   - Verify: no placeholders left
   - Ensure: concrete examples included

## Output

- `context/product/{domain}/README.md` (final documentation, one per domain)
- `context/product/README.md` (index)

## Success Criteria

- All domains have complete documentation
- Each section has concrete, actionable content
- Code references and examples included
- Documentation self-contained and navigable
- Index provides clear entry point
