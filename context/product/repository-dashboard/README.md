# Repository Dashboard - Repository Visualization

## Overview

The Repository Dashboard domain provides the interface for visualizing and navigating the organization's GitHub repositories.

**Framework**: Next.js 15 (App Router)  
**Status**: ✅ Implemented  
**Coverage**: 60%

---

## Main Features

- ✅ Organization repository list
- ✅ Repository selector (RepositoriesSwitcher)
- ✅ Repository navigation
- ✅ Pages: README, About, Admin
- ❌ Filters/sort/search
- ❌ Statistics (commits, issues, PRs)
- ❌ Favorites/bookmarks

---

## Routes

- `/dashboard` - Redirect to first repository
- `/dashboard/repos/[repositoryId]` - General repository view
- `/dashboard/repos/[repositoryId]/readme` - README
- `/dashboard/repos/[repositoryId]/about` - About
- `/dashboard/repos/[repositoryId]/admin` - Admin

---

## Key Components

- `repositories-switcher.tsx` - Repository selection dropdown
- `connect-github.tsx` - Connect GitHub button
- `manage-repositories-modal.tsx` - Repository management modal

---

## Use Cases

### UC-1: View repository list

1. User arrives at `/dashboard`
2. useGithubRepositories() fetches repos via TanStack Query
3. If repos exist: redirect to first repo
4. If no repos: show "Connect GitHub"

### UC-2: Switch repository

1. User clicks on RepositoriesSwitcher
2. Dropdown displays repo list
3. User selects repo
4. Navigation to `/dashboard/repos/[newRepoId]`

---

## Business Rules

- **Repos filtered by organization**: Automatic orgId scoping
- **Automatic redirect**: If no repo selected, go to first
- **Empty state**: If no repos, display RepositoriesSwitcher

---

## Gaps & Limitations

- ❌ Repository filtering/sorting
- ❌ Repository search
- ❌ Favorites/bookmarks
- ❌ GitHub statistics (commits, issues, PRs)
- ❌ README/About content (pages exist but likely empty)

---

## References

- `apps/web/src/app/(protected)/dashboard/` - Dashboard pages
- `apps/web/src/components/github/` - GitHub components
- `apps/web/src/lib/data/github/use-github-repositories.tsx`

---

**Last updated**: 2025-10-17
