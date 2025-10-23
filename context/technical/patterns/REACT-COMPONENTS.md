# React Component Patterns

## Metadata
- **Category**: Pattern
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/web/src/components/**`, `/.cursorrules`

## Technical Overview

Components follow functional programming patterns with no classes, using TypeScript for type safety and Shadcn/UI for UI primitives.

## Patterns & Best Practices

### ✅ DO: Use Functional Components with Type Definitions

```typescript
type UserProfileProps = {
  id: string;
};

export function UserProfile(props: UserProfileProps) {
  const { id } = props;
  const { data } = useQuery(['user', id], () => getUser(id));
  
  return data ? <div>{data.name}</div> : null;
}
```

### ✅ DO: Extract Props to Destructure in Body

```typescript
// GOOD: Props defined as type, destructured in body
type ButtonProps = {
  label: string;
  onClick: () => void;
};

export function Button(props: ButtonProps) {
  const { label, onClick } = props;
  return <button onClick={onClick}>{label}</button>;
}
```

### ❌ DON'T: Use Classes

```typescript
// BAD
class Profile extends React.Component {
  render() { return <div>{this.props.name}</div> }
}

// GOOD
export function Profile(props: ProfileProps) {
  const { name } = props;
  return <div>{name}</div>;
}
```

### ❌ DON'T: Inline Props or Destructure in Parameters

```typescript
// BAD: Inline props typing
export function Profile({ name }: { name: string }) {
  return <div>{name}</div>;
}

// BAD: Destructuring in parameters
export function Profile({ name }: ProfileProps) {
  return <div>{name}</div>;
}

// GOOD: Type defined, destructured in body
type ProfileProps = { name: string };
export function Profile(props: ProfileProps) {
  const { name } = props;
  return <div>{name}</div>;
}
```

## Component Organization

```
apps/web/src/components/
├── ui/                      # Shadcn UI primitives (44 components)
│   ├── button.tsx
│   ├── input.tsx
│   ├── dialog.tsx
│   └── ...
├── layout/                  # Layout components
│   └── sidebar/
└── github/                  # Feature components
```

## Naming Conventions

- **Files**: kebab-case (`user-profile.tsx`)
- **Components**: PascalCase (`UserProfile`)
- **Props Types**: PascalCase with `Props` suffix (`UserProfileProps`)

## References
- **Components**: `apps/web/src/components/**`
- **Guidelines**: `/.cursorrules` (Component Rules)
