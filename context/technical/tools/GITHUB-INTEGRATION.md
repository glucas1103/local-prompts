# GitHub Integration

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/api/src/routes/github.ts`, `apps/api/src/services/github.ts`

## Technical Overview

GitHub integration uses **Octokit** (@octokit/app 16.1.1) for GitHub App authentication and API access.

## Configuration

```typescript
// apps/api/src/services/github.ts
const app = new App({
  appId: process.env.GH_APP_ID,
  privateKey: process.env.GH_PRIVATE_KEY,
});

async function getInstallationOctokit(installationId: number) {
  return await app.getInstallationOctokit(installationId);
}

export async function getRepositories(installationId: string) {
  const octokit = await getInstallationOctokit(Number(installationId));
  const repositories = await octokit.request('GET /installation/repositories');
  return repositories.data.repositories;
}
```

## API Endpoints

```typescript
POST   /api/github/installation           # Create installation
GET    /api/github/installation           # Get installation
GET    /api/github/available-repositories # List available repos
POST   /api/github/repositories           # Add repository
DELETE /api/github/repositories/:id       # Remove repository
GET    /api/github/repositories           # List added repos
```

## References
- **Routes**: `apps/api/src/routes/github.ts`
- **Service**: `apps/api/src/services/github.ts`
- **External Docs**: [Octokit](https://github.com/octokit/octokit.js)
