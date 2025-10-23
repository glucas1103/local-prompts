# Type Safety Patterns

## Metadata
- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/packages/types/src/**`, `/.cursorrules`

## Technical Overview

Strict TypeScript configuration with Zod for runtime validation ensures type safety across the entire stack.

## Patterns & Best Practices

### ✅ DO: Define Zod Schemas in Shared Types

```typescript
// packages/types/src/user.ts
import { z } from 'zod';

export const userSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  firstName: z.string(),
  lastName: z.string(),
  isAdmin: z.boolean(),
});

export type User = z.infer<typeof userSchema>;
```

### ✅ DO: Validate at API Boundaries

```typescript
// Backend
import { userSchema } from '@boilerplate-turbo/types';

router.get('/', 
  validateResponse(userSchema),
  async (req, res) => {
    const user = await getUser();
    res.json(user);  // Validated before sending
  }
);
```

### ✅ DO: Use Strict TypeScript

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true
  }
}
```

### ❌ DON'T: Use `any` Type

```typescript
// BAD
const user: any = await getUser();

// GOOD
const user: User = await getUser();
```

### ❌ DON'T: Skip Runtime Validation

```typescript
// BAD: Only TypeScript validation
const data: User = req.body;

// GOOD: Runtime validation with Zod
const data = userSchema.parse(req.body);
```

## Type Organization

```
packages/types/src/
├── user.ts           # User types & schemas
├── organization.ts   # Organization types
├── github.ts         # GitHub integration types
└── common.ts         # Shared utilities
```

## Naming Conventions

- **Types**: PascalCase (`User`, `Organization`)
- **Schemas**: camelCase with `Schema` suffix (`userSchema`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_FILE_SIZE`)

## References
- **Types Package**: `/packages/types/src/**`
- **Guidelines**: `/.cursorrules` (Type Safety)
- **External Docs**: [Zod](https://zod.dev/), [TypeScript](https://www.typescriptlang.org/)
