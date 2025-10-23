# Database Architecture

## Metadata

- **Category**: Architecture
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/db/schema.ts`, `apps/api/drizzle.config.ts`

## Technical Overview

The application uses **PostgreSQL 17** as the primary relational database, with **Drizzle ORM** for type-safe database operations. The schema follows a normalized design with clear relationships between entities, supporting the core domain of user management, organizations, and GitHub integrations.

All tables implement **soft deletes** and **timestamp tracking** (`createdAt`, `updatedAt`) for audit trails. Database access is exclusively through Drizzle ORM queries using the `req.tx` transaction context provided by middleware, ensuring all operations are properly managed and automatically rolled back on errors.

## Architecture & Design

### Entity-Relationship Diagram

```
┌─────────────┐         ┌──────────────────────┐         ┌──────────────────┐
│    users    │◄───────┤ organization_members  ├────────►│  organizations   │
└─────────────┘         └──────────────────────┘         └──────────────────┘
                                                                   │
                                                                   │
                                                                   ▼
                                                          ┌──────────────────────┐
                                                          │ github_installations │
                                                          └──────────────────────┘
                                                                   │
                                                                   │
                                                                   ▼
                                                 ┌────────────────────────────────────┐
                                                 │ github_installation_repositories │
                                                 └────────────────────────────────────┘
```

### Schema Definition

**Users Table**:

```typescript
export const users = pgTable("users", {
  id: text("id").primaryKey(), // Clerk user ID
  email: text("email").notNull(),
  isAdmin: boolean("is_admin").notNull().default(false),
  firstName: text("first_name").notNull(),
  lastName: text("last_name").notNull(),
  photoUrl: text("photo_url").notNull(),
  deleted: boolean("deleted").notNull().default(false),
  deletedAt: timestamp("deleted_at"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

**Organizations Table**:

```typescript
export const organizations = pgTable("organizations", {
  id: text("id").primaryKey(), // Clerk organization ID
  name: text("company").notNull(),
  plan: text("plan").notNull(), // FREE, PRO, ENTERPRISE
  deleted: boolean("deleted").notNull().default(false),
  deletedAt: timestamp("deleted_at"),
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

**Organization Members (Junction Table)**:

```typescript
export const organizationMembers = pgTable("organization_members", {
  id: text("id")
    .primaryKey()
    .$defaultFn(() => generateShortId("org_mem")),
  organizationId: text("organization_id")
    .notNull()
    .references(() => organizations.id),
  userId: text("user_id")
    .notNull()
    .references(() => users.id),
  role: text("role").notNull(), // admin, member, etc.
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

**GitHub Installations**:

```typescript
export const githubInstallations = pgTable("github_installations", {
  id: text("id").primaryKey(), // GitHub installation ID
  organizationId: text("organization_id")
    .notNull()
    .references(() => organizations.id),
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

**GitHub Installation Repositories**:

```typescript
export const githubInstallationRepositories = pgTable(
  "github_installation_repositories",
  {
    id: text("id").primaryKey(), // GitHub repository ID
    installationId: text("installation_id")
      .notNull()
      .references(() => githubInstallations.id),
    name: text("name").notNull(),
    createdAt: timestamp("created_at").notNull().defaultNow(),
    updatedAt: timestamp("updated_at").notNull().defaultNow(),
  }
);
```

### Drizzle Relations

```typescript
// User to Organizations (through members)
export const usersRelations = relations(users, ({ many }) => ({
  organizations: many(organizationMembers),
}));

// Organization to Members and Installations
export const organizationRelations = relations(organizations, ({ many }) => ({
  members: many(organizationMembers),
  githubInstallations: many(githubInstallations),
}));

// Member relations
export const organizationMemberRelations = relations(
  organizationMembers,
  ({ one }) => ({
    organization: one(organizations, {
      fields: [organizationMembers.organizationId],
      references: [organizations.id],
    }),
    user: one(users, {
      fields: [organizationMembers.userId],
      references: [users.id],
    }),
  })
);

// GitHub Installation relations
export const githubInstallationRelations = relations(
  githubInstallations,
  ({ one, many }) => ({
    organization: one(organizations, {
      fields: [githubInstallations.organizationId],
      references: [organizations.id],
    }),
    repositories: many(githubInstallationRepositories),
  })
);
```

## Configuration & Setup

### Drizzle Configuration

**File**: `apps/api/drizzle.config.ts`

```typescript
import { defineConfig } from "drizzle-kit";
import { dbCredentials } from "./src/db/credentials";

export default defineConfig({
  out: "./drizzle", // Migration output directory
  schema: "./src/db/schema.ts", // Schema source
  dialect: "postgresql",
  dbCredentials, // Connection config
});
```

### Database Connection

**File**: `apps/api/src/db/db.ts`

```typescript
import { Pool } from "pg";
import { dbCredentials } from "./credentials";

export function createDbConnection(options = dbCredentials) {
  const poolConfig = {
    ...options,
    max: 5, // Max connections
    connectionTimeoutMillis: 10000, // 10s timeout
  };

  return { pool: new Pool(poolConfig), config: poolConfig };
}

const { pool } = createDbConnection();

// Error handling
pool.on("error", (err) => {
  logger.error({
    msg: "Unexpected error on idle database client",
    event: "database-error",
    metadata: { error: err },
  });
});

// Validate connection on startup
async function validateConnection() {
  const client = await pool.connect();
  client.release();
  logger.info({
    msg: "Database connection established",
    event: "database-connected",
  });
}

if (process.env.NODE_ENV !== "test") {
  validateConnection().catch((error) => {
    logger.error({
      msg: "Database initialization failed",
      event: "database-init-failed",
      metadata: { error },
    });
    process.exit(-1);
  });
}
```

### Migration Workflow

**Generate Migration**:

```bash
cd apps/api
npm run db:generate
# Creates migration file in drizzle/ directory
```

**Apply Migrations**:

```bash
npm run db:migrate
# Applies pending migrations to database
```

**Initialize Database** (fresh setup):

```bash
npm run db:init
# Runs migrations + seeds test data
```

## Patterns & Best Practices

### ✅ DO: Always Use `req.tx`

```typescript
// GOOD: Using transaction context
router.post("/users", authMiddleware, async (req, res) => {
  try {
    const [user] = await req.tx.insert(users).values(req.body).returning();

    res.status(201).json(user);
  } catch (error) {
    // Transaction automatically rolled back on error
    logger.error({
      msg: "Failed to create user",
      event: "create-user",
      metadata: { error },
    });
    res.status(500).json({ error: "CREATE_USER_FAILED" });
  }
});
```

### ✅ DO: Use Soft Deletes

```typescript
// GOOD: Soft delete pattern
await req.tx
  .update(users)
  .set({
    deleted: true,
    deletedAt: new Date(),
  })
  .where(eq(users.id, userId));

// Query with soft delete filter
const activeUsers = await req.tx
  .select()
  .from(users)
  .where(eq(users.deleted, false));
```

### ✅ DO: Update Timestamps

```typescript
// GOOD: Always update updatedAt
await req.tx
  .update(organizations)
  .set({
    name: "New Name",
    updatedAt: new Date(), // Update timestamp
  })
  .where(eq(organizations.id, orgId));
```

### ✅ DO: Use Type-Safe Queries

```typescript
// GOOD: Type-safe with Drizzle
const [org] = await req.tx
  .select()
  .from(organizations)
  .where(eq(organizations.id, orgId))
  .limit(1);

// org is typed as Organization | undefined
if (!org) {
  return res.status(404).json({ error: "ORGANIZATION_NOT_FOUND" });
}
```

### ❌ DON'T: Use Raw SQL

```typescript
// BAD: Raw SQL (avoid unless absolutely necessary)
await client.query("INSERT INTO users VALUES ($1, $2)", [id, email]);

// GOOD: Use Drizzle query builder
await req.tx.insert(users).values({ id, email });
```

### ❌ DON'T: Hard Delete

```typescript
// BAD: Hard delete loses audit trail
await req.tx.delete(users).where(eq(users.id, userId));

// GOOD: Soft delete preserves history
await req.tx
  .update(users)
  .set({ deleted: true, deletedAt: new Date() })
  .where(eq(users.id, userId));
```

## Usage Examples

### Basic CRUD Operations

**Insert**:

```typescript
const [user] = await req.tx
  .insert(users)
  .values({
    id: "usr_123",
    email: "user@example.com",
    firstName: "John",
    lastName: "Doe",
    photoUrl: "https://example.com/photo.jpg",
    isAdmin: false,
  })
  .returning();
```

**Select**:

```typescript
const [user] = await req.tx
  .select()
  .from(users)
  .where(eq(users.id, userId))
  .limit(1);
```

**Update**:

```typescript
const [updated] = await req.tx
  .update(users)
  .set({
    firstName: "Jane",
    updatedAt: new Date(),
  })
  .where(eq(users.id, userId))
  .returning();
```

**Soft Delete**:

```typescript
await req.tx
  .update(users)
  .set({
    deleted: true,
    deletedAt: new Date(),
  })
  .where(eq(users.id, userId));
```

### Joins and Relations

**Left Join**:

```typescript
const results = await req.tx
  .select()
  .from(organizationMembers)
  .leftJoin(users, eq(organizationMembers.userId, users.id))
  .where(eq(organizationMembers.organizationId, orgId));

const members = results.map(({ organization_members, users }) => ({
  ...users,
  ...organization_members,
}));
```

**Inner Join**:

```typescript
const results = await req.tx
  .select()
  .from(githubInstallationRepositories)
  .innerJoin(
    githubInstallations,
    eq(githubInstallationRepositories.installationId, githubInstallations.id)
  )
  .where(eq(githubInstallations.organizationId, orgId));
```

### Complex Queries

**Multiple Conditions**:

```typescript
import { and, eq, ne } from "drizzle-orm";

const activeMembers = await req.tx
  .select()
  .from(organizationMembers)
  .leftJoin(users, eq(organizationMembers.userId, users.id))
  .where(
    and(
      eq(organizationMembers.organizationId, orgId),
      eq(users.deleted, false) // Only active users
    )
  );
```

**Conditional Soft Delete Check**:

```typescript
const repository = await req.tx
  .select()
  .from(githubInstallationRepositories)
  .innerJoin(
    githubInstallations,
    eq(githubInstallationRepositories.installationId, githubInstallations.id)
  )
  .where(
    and(
      eq(githubInstallationRepositories.id, repositoryId),
      eq(githubInstallations.organizationId, orgId)
    )
  )
  .limit(1);
```

## Technical Dependencies

**ORM & Database**:

- `drizzle-orm`: ^0.41.0 - Type-safe ORM
- `drizzle-kit`: ^0.31.4 - Migration tool
- `pg`: ^8.16.3 - PostgreSQL client
- `postgres` (Docker image): 17-alpine

**Utilities**:

- `@boilerplate-turbo/types` - Shared ID generation

## Tests & Validation

**Test Setup with Testcontainers**:

```typescript
import { PostgreSqlContainer } from '@testcontainers/postgresql';
import { drizzle } from 'drizzle-orm/node-postgres';

describe('Database Tests', () => {
  let container;
  let db;

  beforeAll(async () => {
    container = await new PostgreSqlContainer().start();
    const pool = new Pool({ connectionString: container.getConnectionUri() });
    db = drizzle(pool);
    // Run migrations
  });

  afterAll(async () => {
    await container.stop();
  });

  it('should insert and retrieve user', async () => {
    const [user] = await db.insert(users).values({...}).returning();
    expect(user.id).toBeDefined();
  });
});
```

## Historical Technical Decisions

**[To be documented by CTO]**

- **Soft Delete Rationale**: Why soft deletes instead of hard deletes?
  - Compliance requirements?
  - Audit trail needs?
  - Data recovery scenarios?

- **Text ID Strategy**: Why text IDs with `generateShortId` vs UUIDs or integers?
  - Readability in logs?
  - External system compatibility?
  - URL-friendly IDs?

## Troubleshooting & FAQ

**Issue**: Migration fails with "relation already exists"

- **Cause**: Migration already applied or manual schema changes
- **Solution**: Check migration history, reset database or create rollback migration

**Issue**: Connection pool exhausted

- **Cause**: Too many concurrent requests or connections not released
- **Solution**: Check transaction middleware, increase pool size, or investigate slow queries

**Issue**: Type errors with Drizzle queries

- **Cause**: Schema changes not reflected in TypeScript types
- **Solution**: Restart TypeScript server or regenerate types

## References

- **Main Files**:
  - `apps/api/src/db/schema.ts` - Schema definitions
  - `apps/api/src/db/db.ts` - Connection management
  - `apps/api/drizzle.config.ts` - Drizzle configuration
  - `apps/api/drizzle/*.sql` - Migration files

- **Scripts**:
  - `apps/api/db-management/scripts/` - Database management scripts

- **External Docs**:
  - [Drizzle ORM](https://orm.drizzle.team/)
  - [PostgreSQL](https://www.postgresql.org/docs/)
  - [pg (node-postgres)](https://node-postgres.com/)
