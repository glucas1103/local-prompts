# Task: Create Technical Context Folder

## Objective

Scan technical files and create initial technical context summary.

## Language Requirements

All generated content must be written in English.

## Inputs

- Architecture files (docs/architecture/)
- Configuration files (tsconfig, eslint, package.json, docker-compose, etc.)
- Technical documentation (README sections, ADRs, etc.)
- Test files (_.test.ts, _.spec.ts, vitest.config.ts, etc.)

## Steps

1. **Scan architecture documentation**
   - Read architecture files in docs/
   - Extract: architecture patterns, design decisions
   - Note: component diagrams, system design

2. **Scan configuration files**
   - package.json: dependencies, scripts, dev tools
   - tsconfig.json, eslint.config: code standards
   - docker-compose.yml, Dockerfile: deployment setup
   - Identify: build tools, testing frameworks, linters

3. **Scan test files**
   - Read test configuration (vitest.config, playwright.config)
   - Extract: test patterns, coverage requirements
   - Note: test utilities, fixtures, mocks

4. **Scan source code structure**
   - Analyze folder organization
   - Identify: architectural layers (api, services, routes, etc.)
   - Note: shared utilities, middleware, types

5. **Create initial summary**
   - Write to `.speiros/artifacts/audit-technical/initial-summary.md`
   - Structure:
     - Tech Stack Overview (languages, frameworks, tools)
     - Architecture Patterns (monorepo, layered, etc.)
     - Development Tools (linters, formatters, bundlers)
     - Testing Strategy (unit, integration, e2e)
     - Build & Deployment (scripts, containers, CI/CD)
     - Information Sources (files read)

## Output

- `.speiros/artifacts/audit-technical/initial-summary.md`

## Success Criteria

- Summary captures core technical setup
- All configuration and architecture files scanned
- Clear, structured format
- Ready for gap analysis
