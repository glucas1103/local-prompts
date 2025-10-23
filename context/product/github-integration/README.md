# GitHub Integration - GitHub App Integration

## Overview

The GitHub Integration domain manages integration with GitHub App to access repositories. Distinct from GitHub OAuth used for authentication.

**Provider**: GitHub App via Octokit  
**Status**: ✅ Implemented  
**Coverage**: 80%

---

## Main Features

- ✅ GitHub App installation on GitHub account/org
- ✅ Retrieval of available repositories
- ✅ Add/remove repositories to follow
- ✅ Storage of installations and repos in DB
- ❌ GitHub webhooks (not implemented)
- ❌ Automatic sync (manual only)

---

## Use Cases

### UC-1: Install GitHub App

1. User clicks "Connect GitHub"
2. Redirected to GitHub to authorize app
3. GitHub callback with installationId
4. Frontend POST `/installation` with installationId
5. Backend creates installation linked to orgId

### UC-2: Add a repository

1. User fetches available repos (GET `/available-repositories`)
2. Selects a repository
3. Frontend POST `/repositories` with repositoryId
4. Backend saves repo in DB

### UC-3: Remove a repository

1. User clicks "Remove repository"
2. Frontend DELETE `/repositories/:repositoryId`
3. Backend removes from DB

---

## Business Rules

- **One installation per organization**: Link installation ↔ org
- **Available repos via GitHub API**: Real-time request
- **Manual sync**: No webhooks, user fetches manually
- **GitHub permissions**: Defined in GitHub App config (not in code)

---

## Schema

**Table `github_installations`**:

```typescript
{
  (id, organizationId, createdAt, updatedAt);
}
```

**Table `github_installation_repositories`**:

```typescript
{
  (id, installationId, name, createdAt, updatedAt);
}
```

---

## Gaps & Limitations

- ❌ GitHub webhooks not implemented
- ❌ Automatic sync missing
- ❌ GitHub permissions not documented
- ❌ Installation revocation management
- ❓ Public/private repos handling difference

---

## References

- `apps/api/src/services/github.ts` - GitHub service (Octokit)
- `apps/api/src/routes/github.ts` - GitHub routes
- `apps/web/src/components/github/` - UI components

---

**Last updated**: 2025-10-17
