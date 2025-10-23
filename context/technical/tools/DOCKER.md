# Docker Containerization

## Metadata
- **Category**: Tool
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/docker-compose.yml`, `apps/*/Dockerfile`

## Technical Overview

Multi-stage Docker builds optimize image size using **node:22.14.0-alpine** base image.

## Docker Compose

```yaml
services:
  web:   # Next.js frontend (port 3000)
  api:   # Express backend (port 3001)
  db:    # PostgreSQL 17 (port 5432)
```

## Security

- Non-root users (`expressjs:1001`, `nextjs:1001`)
- Multi-stage builds reduce attack surface
- Only production dependencies in final image

## Usage

```bash
docker-compose up -d         # Start all services
docker-compose logs -f api   # View logs
docker-compose down          # Stop services
```

## References
- **Configuration**: `/docker-compose.yml`, `apps/*/Dockerfile`
- **External Docs**: [Docker](https://docs.docker.com/)
