# Turborepo

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/turbo.json`

## Technical Overview

**Turborepo 2.4.4** orchestrates builds across the monorepo with intelligent caching and parallel execution.

## Configuration

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"],
      "env": ["NEXT_PUBLIC_*", "SENTRY_*"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

## Usage

```bash
turbo run build                    # Build all workspaces
turbo run build --filter=api       # Build specific workspace
turbo run dev                      # Start all in dev mode
```

## References
- **Configuration**: `/turbo.json`
- **External Docs**: [Turborepo](https://turbo.build/repo/docs)
