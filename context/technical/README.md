# Technical Context Documentation

## Overview

This directory contains comprehensive technical documentation for the monorepo. All documentation follows a standardized template and is organized into four main categories.

**Last Updated**: 2025-10-20  
**Status**: Active Development  
**Owner**: CTO (Lucas Gaillard)

---

## Directory Structure

```
technical/
├── context/        # Core technical domains (8 files)
├── tools/          # External tools and integrations (9 files)
├── patterns/       # Coding patterns and conventions (6 files)
└── memory/         # Historical decisions and troubleshooting (4 files)
```

---

## 📁 context/ - Core Technical Domains

High-level technical architecture and design documentation.

| File                  | Purpose                                            | Status      |
| --------------------- | -------------------------------------------------- | ----------- |
| **ARCHITECTURE.md**   | Overall system architecture and monorepo structure | ✅ Complete |
| **API.md**            | Backend API architecture, routes, and conventions  | ✅ Complete |
| **DATABASE.md**       | Database schema, ORM usage, and data patterns      | ✅ Complete |
| **FRONTEND.md**       | Frontend architecture, routing, and components     | ✅ Complete |
| **AUTHENTICATION.md** | Auth flow and security implementation              | ✅ Complete |
| **ERROR-HANDLING.md** | Error management, logging, and monitoring          | ✅ Complete |
| **TESTING.md**        | Testing strategy, patterns, and utilities          | ✅ Complete |
| **DEPLOYMENT.md**     | Build, deployment, and operational procedures      | ✅ Complete |

---

## 🔧 tools/ - External Tools & Integrations

Tool-specific configuration and usage documentation.

| File                      | Purpose                                   | Status      |
| ------------------------- | ----------------------------------------- | ----------- |
| **TURBOREPO.md**          | Monorepo build orchestration              | ✅ Complete |
| **DRIZZLE.md**            | ORM configuration and migrations          | ✅ Complete |
| **CLERK.md**              | Authentication service integration        | ✅ Complete |
| **SENTRY.md**             | Error tracking and performance monitoring | ✅ Complete |
| **POSTHOG.md**            | Analytics and feature flags               | ✅ Complete |
| **GITHUB-INTEGRATION.md** | GitHub API and webhooks                   | ✅ Complete |
| **DOCKER.md**             | Containerization and orchestration        | ✅ Complete |
| **PLAYWRIGHT.md**         | E2E testing framework                     | ✅ Complete |
| **VITEST.md**             | Unit testing framework                    | ✅ Complete |

---

## 🎨 patterns/ - Coding Patterns & Conventions

Implementation patterns and coding standards.

| File                       | Purpose                                        | Status      |
| -------------------------- | ---------------------------------------------- | ----------- |
| **MIDDLEWARE.md**          | Express middleware patterns and `req.tx` usage | ✅ Complete |
| **REACT-COMPONENTS.md**    | Component architecture and organization        | ✅ Complete |
| **API-ROUTES.md**          | API route organization and REST conventions    | ✅ Complete |
| **DATABASE-OPERATIONS.md** | Data access patterns with Drizzle              | ✅ Complete |
| **ERROR-PROPAGATION.md**   | Error handling conventions across layers       | ✅ Complete |
| **TYPE-SAFETY.md**         | TypeScript patterns and Zod validation         | ✅ Complete |

---

## 💾 memory/ - Historical Knowledge

Historical decisions, troubleshooting, and lessons learned.

| File                     | Purpose                                   | Status            |
| ------------------------ | ----------------------------------------- | ----------------- |
| **TECHNICAL-CHOICES.md** | ADR-style technical decision log          | ✅ Complete       |
| **TROUBLESHOOTING.md**   | Common issues and solutions               | ✅ Complete       |
| **MIGRATIONS.md**        | Database migration history and procedures | ✅ Complete       |
| **REFACTORING-LOG.md**   | Major refactoring notes and learnings     | 📝 To be enriched |

---

## Quick Links

### For New Developers

1. Start with **ARCHITECTURE.md** for system overview
2. Read **API.md** and **FRONTEND.md** for application structure
3. Review **MIDDLEWARE.md** and **DATABASE-OPERATIONS.md** for core patterns
4. Check **TESTING.md** for test conventions

### For Feature Development

1. **API.md** - Adding backend endpoints
2. **FRONTEND.md** - Adding frontend features
3. **DATABASE.md** - Schema changes
4. **AUTHENTICATION.md** - Protected routes

### For Deployment & Operations

1. **DEPLOYMENT.md** - Build and release process
2. **DOCKER.md** - Containerization
3. **TROUBLESHOOTING.md** - Common issues
4. **MIGRATIONS.md** - Database updates

### For Debugging

1. **ERROR-HANDLING.md** - Logging and error tracking
2. **SENTRY.md** - Error monitoring
3. **TROUBLESHOOTING.md** - Known issues

---

## Documentation Standards

All technical documentation follows these principles:

### Content Structure

Each file includes:

- **Metadata**: Category, status, owner, source files
- **Technical Overview**: Concise description (2-3 paragraphs)
- **Architecture & Design**: Patterns and design choices
- **Configuration & Setup**: Required setup and configuration
- **API & Interfaces**: Public interfaces and contracts (when applicable)
- **Patterns & Best Practices**: Code conventions and standards
- **Usage Examples**: Concrete code examples from codebase
- **Technical Dependencies**: Package and internal dependencies
- **Tests & Validation**: Testing approach and examples
- **Troubleshooting & FAQ**: Common issues and solutions
- **References**: Links to source files and external docs

### Information Gaps

When information is unavailable, it's marked as:

- **[To be documented by CTO]** - Requires CTO input
- **[Requires business input]** - Needs product/business context
- **[To be enriched with operational experience]** - Will be documented over time

### Code Examples

- All examples come from actual codebase
- File paths and line numbers referenced when possible
- Both ✅ good and ❌ bad examples shown

---

## Maintenance

### Updating Documentation

- Update **Last Update** date in file metadata
- Add examples from actual implementation
- Mark resolved information gaps
- Cross-reference related documentation

### Responsibility

- **CTO (Lucas Gaillard)**: Architecture decisions, operational procedures
- **Development Team**: Technical patterns, code examples
- **All Contributors**: Keep documentation in sync with code

---

## Technology Stack Summary

### Frontend

- **React 19** + **TypeScript 5.7**
- **Next.js 15** (App Router)
- **TanStack Query 5.71** (data fetching)
- **Tailwind CSS 4** + **Shadcn/UI** (styling)

### Backend

- **Node.js 22.14** + **TypeScript 5.7**
- **Express 4.21** (web framework)
- **Drizzle ORM 0.41** (database)
- **PostgreSQL 17** (database)
- **Zod 3.25** (validation)

### Development Tools

- **Turborepo 2.4** (monorepo)
- **Vitest 3** (unit/integration tests)
- **Playwright 1.53** (E2E tests)
- **ESLint 9** + **Prettier 3** (linting)
- **Docker** + **Docker Compose** (containerization)

### External Services

- **Clerk** (authentication)
- **Sentry** (error tracking)
- **PostHog** (analytics)
- **GitHub** (version control + API integration)

---

## Contributing

When making significant technical changes:

1. Update relevant documentation files
2. Add code examples from your implementation
3. Document any new patterns or conventions
4. Update troubleshooting section if you encountered issues
5. Cross-reference related files

---

## Product Context

The application is an **AI Context Management Platform** for teams working with:

- **Workflows** (Markdown-based)
- **Agents** (AI agent configurations)
- **Prompts** (Prompt templates and versioning)

**Key Capabilities**:

- **Observability**: Track and monitor AI workflows
- **Governance**: Control and audit AI usage
- **Collaboration**: Team-based context management
- **Versioning**: Version control for AI configurations

**Development Approach**: Customer-driven feature development based on interviews and feedback.

---

## References

- **Root Configuration**: `/package.json`, `/turbo.json`, `/.cursorrules`
- **API Source**: `/apps/api/src/**`
- **Web Source**: `/apps/web/src/**`
- **Shared Types**: `/packages/types/src/**`
- **E2E Tests**: `/e2e-tests/**`
- **Product Context**: `/context/product/**`
- **Roadmap**: `/roadmap/**`

---

## Support

For questions or clarifications:

- **Technical Decisions**: Contact CTO (Lucas Gaillard)
- **Architecture Questions**: Review ARCHITECTURE.md, then ask CTO
- **Operational Issues**: Check TROUBLESHOOTING.md first
- **Code Patterns**: Review relevant pattern documentation in `patterns/`

---

_This documentation is living documentation—keep it updated as the codebase evolves._
