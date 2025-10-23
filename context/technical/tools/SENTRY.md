# Sentry Error Tracking

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/instrument.ts`, `apps/web/sentry.server.config.ts`

## Technical Overview

**Sentry** tracks errors and performance on both API (@sentry/node 9.22.0) and Web (@sentry/nextjs 9.22.0).

## Backend Configuration

```typescript
// apps/api/src/instrument.ts
Sentry.init({
  dsn: process.env.SENTRY_API_DSN,
  release: process.env.COMMIT_SHA,
  tracesSampleRate: 1.0,
  environment: process.env.NODE_ENV,
});
```

## Frontend Configuration

```typescript
// apps/web/sentry.server.config.ts
Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_WEB_DSN,
  release: process.env.NEXT_PUBLIC_COMMIT_SHA,
  tracesSampleRate: 1,
});
```

## Source Maps

**API**:
```bash
npm run sentry:sourcemaps  # Upload source maps
```

## References
- **API Config**: `apps/api/src/instrument.ts`
- **Web Config**: `apps/web/sentry.server.config.ts`
- **External Docs**: [Sentry](https://docs.sentry.io/)
