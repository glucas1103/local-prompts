# PostHog Analytics

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/web/next.config.ts`

## Technical Overview

**PostHog** (posthog-js 1.258.2) provides analytics and feature flags, proxied through Next.js to avoid ad-blockers.

## Configuration

```typescript
// apps/web/next.config.ts
async rewrites() {
  return [
    {
      source: '/ingest/static/:path*',
      destination: 'https://eu-assets.i.posthog.com/static/:path*',
    },
    {
      source: '/ingest/:path*',
      destination: 'https://eu.i.posthog.com/:path*',
    },
  ];
},
skipTrailingSlashRedirect: true,  // Required for PostHog
```

## References
- **Configuration**: `apps/web/next.config.ts`
- **External Docs**: [PostHog](https://posthog.com/docs)
