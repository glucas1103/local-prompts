# Task: Populate Technical Context

## Objective

Fill technical context templates with research findings to create final technical documentation.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-technical/research/**/*.md` (research files)
- `.speiros/templates/audit-codebase/technical-context-template.yaml` (template structure)

## Steps

1. **For each technical domain** (iterative):

   a. **Load research data**
   - Read all research files for domain
   - Extract: findings, code examples, patterns, configs
   - Organize by template section

   b. **Fill template sections**
   - **Metadata**: category, files, last update, status
   - **Technical Overview**: synthesize from research (2-3 paragraphs)
   - **Architecture & Design**: patterns, design choices
   - **Configuration & Setup**: env vars, setup steps, configs
   - **API & Interfaces**: public APIs, contracts, interfaces
   - **Patterns & Best Practices**: code conventions, standards
   - **Usage Examples**: concrete code examples from codebase
   - **Technical Dependencies**: packages, services, modules
   - **Tests & Validation**: test strategy, examples, coverage
   - **References**: link to main files, tests, configs, specs

   c. **Handle elicit sections**
   - **Historical Technical Decisions**: use enrichment Q&A data
   - **Technical Roadmap**: use enrichment Q&A data
   - **Troubleshooting & FAQ**: extract from code comments, logs
   - If missing: mark as "To be documented"

   d. **Generate final document**
   - Save to appropriate location:
     - `context/technical/context/{DOMAIN}.md`
     - `context/technical/tools/{TOOL}.md`
     - `context/technical/memory/{TOPIC}.md`
   - Format: clean markdown, no YAML
   - Include: code snippets, file references, diagrams
   - Ensure: self-contained and readable

2. **Create technical index**
   - Generate `context/technical/README.md`
   - List all technical domains with brief descriptions
   - Organize by category (context/tools/memory)
   - Link to each domain's documentation

3. **Validate completeness**
   - Check: all template sections filled
   - Verify: no placeholders left
   - Ensure: concrete examples and configs included

## Output

- `context/technical/context/*.md` (technical domain docs)
- `context/technical/tools/*.md` (tool-specific docs)
- `context/technical/memory/*.md` (decisions, troubleshooting)
- `context/technical/README.md` (index)

## Success Criteria

- All technical domains have complete documentation
- Each section has concrete, actionable technical content
- Code references, configs, and examples included
- Documentation self-contained and navigable
- Index provides clear entry point by category
