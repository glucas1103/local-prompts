# Vitest Unit Testing

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/vitest.config.ts`, `apps/web/vitest.config.mts`

## Technical Overview

**Vitest 3.0+** runs unit and integration tests with coverage reporting.

## Backend Configuration

```typescript
export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./src/tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
    pool: 'forks',
    hookTimeout: 15000,
  },
});
```

## Frontend Configuration

```typescript
export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: 'jsdom',
    globals: true,
  },
});
```

## Usage

```bash
npm test                    # Run all tests
npm run test:watch          # Watch mode
npm run test:coverage       # With coverage
```

## References
- **API Config**: `apps/api/vitest.config.ts`
- **Web Config**: `apps/web/vitest.config.mts`
- **External Docs**: [Vitest](https://vitest.dev/)
