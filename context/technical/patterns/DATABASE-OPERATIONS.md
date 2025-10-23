# Database Operation Patterns

## Metadata
- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/db/**`, `/.cursorrules`

## Technical Overview

All database operations use Drizzle ORM via `req.tx` transaction context, with automatic transaction management for write operations.

## Patterns & Best Practices

### ✅ DO: Always Use `req.tx`

```typescript
// GOOD: Using transaction context
router.post('/users', authMiddleware, async (req, res) => {
  try {
    const [user] = await req.tx
      .insert(users)
      .values(req.body)
      .returning();
    
    res.status(201).json(user);
  } catch (error) {
    // Transaction automatically rolled back
    logger.error({ msg: 'Failed to create user', event: 'create-user', metadata: { error } });
    res.status(500).json({ error: 'CREATE_USER_FAILED' });
  }
});
```

### ✅ DO: Use Soft Deletes

```typescript
// GOOD: Soft delete
await req.tx
  .update(users)
  .set({ 
    deleted: true, 
    deletedAt: new Date() 
  })
  .where(eq(users.id, userId));
```

### ✅ DO: Update Timestamps

```typescript
// GOOD: Always update updatedAt
await req.tx
  .update(organizations)
  .set({ 
    name: 'New Name',
    updatedAt: new Date()
  })
  .where(eq(organizations.id, orgId));
```

### ❌ DON'T: Use Raw SQL or Manual Transactions

```typescript
// BAD: Raw SQL
await client.query('INSERT INTO users VALUES ($1, $2)', [id, email]);

// BAD: Manual transactions
const client = await pool.connect();
try {
  await client.query('BEGIN');
  // ... operations
  await client.query('COMMIT');
} catch {
  await client.query('ROLLBACK');
}

// GOOD: Use Drizzle with req.tx
await req.tx.insert(users).values({ id, email });
```

## Common Operations

**Insert**:
```typescript
const [user] = await req.tx.insert(users).values({...}).returning();
```

**Select**:
```typescript
const [user] = await req.tx.select().from(users).where(eq(users.id, id)).limit(1);
```

**Update**:
```typescript
const [updated] = await req.tx.update(users).set({...}).where(eq(users.id, id)).returning();
```

**Join**:
```typescript
const results = await req.tx
  .select()
  .from(organizationMembers)
  .leftJoin(users, eq(organizationMembers.userId, users.id))
  .where(eq(organizationMembers.organizationId, orgId));
```

## References
- **Database Layer**: `apps/api/src/db/**`
- **Schema**: `apps/api/src/db/schema.ts`
- **Guidelines**: `/.cursorrules` (Database Access)
