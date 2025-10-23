# Error Propagation Patterns

## Metadata
- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/routes/**`, `/.cursorrules`

## Technical Overview

Errors propagate through layers using try/catch with structured logging, SCREAMING_SNAKE_CASE error codes, and appropriate HTTP status codes.

## Patterns & Best Practices

### ✅ DO: Use try/catch with Structured Logging

```typescript
router.get('/user', authMiddleware, async (req, res) => {
  try {
    const [user] = await req.tx
      .select()
      .from(users)
      .where(eq(users.id, req.auth.userId))
      .limit(1);
    
    if (!user) {
      return res.status(404).json({ error: 'USER_NOT_FOUND' });
    }
    
    res.json(user);
  } catch (error) {
    logger.error({
      msg: 'Error fetching user',
      event: 'get-user',
      metadata: { error, userId: req.auth.userId },
    });
    res.status(500).json({ error: 'FAILED_TO_GET_USER' });
  }
});
```

### ✅ DO: Use Appropriate HTTP Status Codes

```typescript
res.status(200).json(data);                      // Success
res.status(201).json(created);                   // Created
res.status(400).json({ error: 'BAD_REQUEST' }); // Validation error
res.status(401).json({ error: 'UNAUTHORIZED' }); // Authentication required
res.status(404).json({ error: 'NOT_FOUND' });   // Resource not found
res.status(500).json({ error: 'SERVER_ERROR' }); // Internal error
```

### ✅ DO: Use SCREAMING_SNAKE_CASE for Error Codes

```typescript
res.status(404).json({ error: 'USER_NOT_FOUND' });
res.status(500).json({ error: 'FAILED_TO_CREATE_ORGANIZATION' });
```

### ❌ DON'T: Use console.log

```typescript
// BAD
console.log('Error:', error);

// GOOD
logger.error({
  msg: 'Database operation failed',
  event: 'database-error',
  metadata: { error },
});
```

## Error Flow

```
Database Error
     ↓
try/catch in Route
     ↓
Structured Logging (Pino)
     ↓
Sentry (if 500 error)
     ↓
HTTP Error Response
     ↓
Frontend Error Handling
```

## References
- **Routes**: `apps/api/src/routes/**`
- **Logger**: `apps/api/src/utils/logger.ts`
- **Guidelines**: `/.cursorrules`
