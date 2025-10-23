# Technical Choices & Decisions

## Metadata
- **Category**: Memory
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: User interviews, code analysis

## Technical Overview

This document records key technical decisions made during the project, following an ADR (Architecture Decision Record) style.

## Technology Stack Decisions

### Decision: Technology Selection Based on Familiarity
**Date**: Project inception  
**Status**: Accepted

**Context**:  
Needed to choose technology stack for new AI Context Management platform.

**Decision**:  
Selected technologies based on developer familiarity:
- Drizzle ORM (vs Prisma, TypeORM)
- Express (vs Fastify, Hono)
- Turborepo (vs Nx, Lerna)
- Next.js App Router (vs Pages Router)
- Clerk (vs Auth0, NextAuth)
- Pino (vs Winston, Bunyan)

**Rationale** (from CTO):  
> "Tous mes choix ont été faits car je m'étais déjà servi de ces technologies, donc c'était plus simple pour moi de faire le setup."

**Consequences**:
- ✅ Faster project setup
- ✅ Reduced learning curve
- ✅ Immediate productivity
- ⚠️ May not be optimal choices for all use cases
- ⚠️ Limited evaluation of alternatives

---

### Decision: Backend Independence from Next.js
**Date**: Project inception  
**Status**: Accepted

**Context**:  
Deciding whether to use Next.js API routes or separate backend.

**Decision**:  
Separate Express API from Next.js frontend.

**Rationale** (from CTO):  
> "Je ne veux pas être trop dépendant de Next au cas où de nouvelles technologies arrivent en back-end."

**Consequences**:
- ✅ Backend can evolve independently
- ✅ Easy to migrate to different frameworks
- ✅ Clear separation of concerns
- ⚠️ Additional deployment complexity
- ⚠️ Cannot use Next.js API routes directly

---

### Decision: REST API (not GraphQL)
**Date**: Project inception  
**Status**: Accepted

**Context**:  
Choosing API communication pattern.

**Decision**:  
Use REST API for backend-frontend communication.

**Rationale** (from CTO):  
Simple, well-understood pattern. No immediate need for GraphQL complexity.

**Consequences**:
- ✅ Simple and straightforward
- ✅ Easy to understand and debug
- ✅ Good tooling support
- ⚠️ May require multiple requests for complex data
- ⚠️ Less flexible than GraphQL for frontend needs

---

### Decision: Modular Monolith (not Microservices)
**Date**: Project inception  
**Status**: Accepted

**Context**:  
Architecture choice for scalability.

**Decision**:  
Build as modular monolith, no microservices.

**Rationale** (from CTO):  
> "Pas pour le moment" - current scale doesn't require microservices.

**Consequences**:
- ✅ Simpler deployment
- ✅ Lower operational overhead
- ✅ Easier local development
- ✅ Atomic transactions across domain boundaries
- ⚠️ May need refactoring at scale
- ⚠️ All services scale together

---

## Database Design Decisions

### **[To be documented by CTO]**: Soft Delete Strategy
**Question**: Why use soft deletes instead of hard deletes?

Possible reasons to explore:
- Audit trail requirements
- Compliance needs (GDPR, data retention)
- Recovery scenarios
- Business analytics needs

**Action**: Document rationale and use cases.

---

### **[To be documented by CTO]**: Text ID Generation Strategy
**Question**: Why use `generateShortId('prefix')` vs UUIDs or auto-increment integers?

Observed pattern:
```typescript
id: text('id').primaryKey().$defaultFn(() => generateShortId('org_mem'))
```

Possible reasons to explore:
- URL-friendly IDs
- Readability in logs
- External system compatibility
- Sortable IDs

**Action**: Document rationale and ID generation strategy.

---

## Testing Decisions

### Decision: Mocked Unit Tests + Staging Integration Tests
**Date**: Project inception  
**Status**: Accepted

**Context**:  
How to test external service integrations.

**Decision**:  
Mock services in unit tests, test real integrations on staging.

**Rationale** (from CTO):  
> "Je testerai avec l'Environment staging sur lequel je deploy depuis local"

**Consequences**:
- ✅ Fast unit tests
- ✅ Real integration validation on staging
- ⚠️ No integration tests in CI/CD
- ⚠️ Staging must be available for integration testing

**[To be documented by CTO]**: Test coverage targets and testing philosophy.

---

## Development Workflow Decisions

### **[To be documented by CTO]**: Git Workflow
**Questions**:
- Git Flow, trunk-based, or other?
- Branch naming conventions?
- Code review process?
- PR approval requirements?

**Action**: Document team's Git workflow.

---

### **[To be documented by CTO]**: Deployment Process
**Questions**:
- CI/CD pipeline details
- Deployment approval process
- Rollback procedures
- Database migration timing

**Action**: Document deployment workflow.

---

## Operational Decisions

### **[To be documented by CTO]**: Backup & Disaster Recovery
**Questions**:
- Backup frequency?
- Backup retention policy?
- Disaster recovery procedures?
- RTO/RPO targets?

**Action**: Document backup and recovery procedures.

**Responsibility**: CTO (Lucas Gaillard)

---

### **[To be documented by CTO]**: On-Call & Incident Response
**Questions**:
- On-call rotation?
- Alerting strategy?
- Incident response procedures?
- Postmortem process?

**Action**: Document operational procedures.

**Responsibility**: CTO (Lucas Gaillard)

---

## Product Vision Context

### Decision: Customer-Driven Development
**Date**: Project inception  
**Status**: Accepted

**Context**:  
Product development strategy.

**Decision**:  
Build features based on customer interviews and feedback.

**Rationale** (from CTO):  
> "Nous allons rajouter des features après interviews clients."

**Product Vision**:  
> "Le produit veut être la meilleure solution de gestion de contexte pour les IA (donc des workflows, agents, prompts, etc qui sont des markdownfiles) avec une observabilités, gouvernance, collaboration, versionage, etc."

**Consequences**:
- ✅ Features aligned with real user needs
- ✅ Avoid building unused features
- ✅ Iterative product development
- ⚠️ Requires active customer engagement
- ⚠️ May delay some technical improvements

---

## Future Considerations

**No Immediate Plans** (from CTO):  
> "Rien de prévu pour le moment, tout fonctionne bien."

**When to Revisit**:
- Performance bottlenecks emerge
- Scaling challenges arise
- Customer needs drive architectural changes
- New technologies provide significant advantages

---

## References

- **User Inputs**: `.speiros/artifacts/audit-technical/enrichment-qa.md`
- **Code Analysis**: Various source files
- **Owner**: CTO (Lucas Gaillard)
