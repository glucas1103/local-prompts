# Drizzle ORM

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/drizzle.config.ts`, `apps/api/src/db/schema.ts`

## Technical Overview

**Drizzle ORM 0.41.0** provides type-safe database operations with PostgreSQL.

## Configuration

```typescript
export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'postgresql',
  dbCredentials,
});
```

## Migration Workflow

```bash
npm run db:generate    # Generate migration from schema changes
npm run db:migrate     # Apply migrations
npm run db:init        # Migrate + seed (fresh database)
```

## Schema Example

```typescript
export const users = pgTable('users', {
  id: text('id').primaryKey(),
  email: text('email').notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
});
```

## References
- **Configuration**: `apps/api/drizzle.config.ts`
- **Schema**: `apps/api/src/db/schema.ts`
- **Migrations**: `apps/api/drizzle/*.sql`
- **External Docs**: [Drizzle ORM](https://orm.drizzle.team/)
