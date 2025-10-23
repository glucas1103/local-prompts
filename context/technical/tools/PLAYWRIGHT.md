# Playwright E2E Testing

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/playwright.config.ts`, `/e2e-tests/**`

## Technical Overview

**Playwright 1.53.2** runs E2E tests against Chromium and Firefox with Clerk authentication.

## Configuration

```typescript
export default defineConfig({
  testDir: './e2e-tests',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  projects: [
    { name: 'global setup', testMatch: /global\.setup\.ts/ },
    { name: 'chromium', use: devices['Desktop Chrome'], dependencies: ['global setup'] },
    { name: 'firefox', use: devices['Desktop Firefox'], dependencies: ['global setup'] },
  ],
});
```

## Usage

```bash
npm run test:e2e              # Run tests
npm run test:e2e:ui           # Interactive mode
npm run test:e2e:staging      # Test staging environment
```

## References
- **Configuration**: `/playwright.config.ts`
- **Tests**: `/e2e-tests/**`
- **External Docs**: [Playwright](https://playwright.dev/)
