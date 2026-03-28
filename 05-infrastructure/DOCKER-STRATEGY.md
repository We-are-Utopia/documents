# Docker Strategy

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | CIS Docker Benchmark; ISO/IEC 27001:2022 |

---

## 1. Purpose

This document defines the Docker containerization strategy for the Utopia project, including Dockerfile standards, image management, multi-stage build patterns, and local development configuration via Docker Compose.

## 2. Base Image Strategy

### 2.1. Approved Base Images

| Image | Version | Source | Use Case | Size |
|-------|---------|--------|----------|------|
| `mcr.microsoft.com/dotnet/aspnet` | `8.0-alpine` | Microsoft Container Registry | .NET runtime | ~100 MB |
| `mcr.microsoft.com/dotnet/sdk` | `8.0-alpine` | Microsoft Container Registry | .NET build | ~400 MB |
| `node` | `20-alpine` | Docker Hub Official | Next.js build/runtime | ~130 MB |
| `postgres` | `16-alpine` | Docker Hub Official | PostgreSQL | ~80 MB |
| `redis` | `7-alpine` | Docker Hub Official | Redis cache | ~30 MB |

### 2.2. Base Image Rules

- MUST use official images from verified publishers only
- MUST use Alpine variants to minimize attack surface
- MUST pin to specific minor version (e.g., `8.0-alpine`, NOT `latest`)
- SHOULD pin to digest for production deployments
- MUST NOT use `ubuntu`, `debian` unless Alpine is not available
- MUST NOT use community or unverified images

## 3. Dockerfile Standards

### 3.1. General Rules

| Rule | Requirement |
|------|-------------|
| Multi-stage builds | REQUIRED for all application images |
| Non-root user | REQUIRED — `USER 1001:1001` in runtime stage |
| `COPY` vs `ADD` | Use `COPY` only; `ADD` prohibited (no remote URL fetch) |
| `HEALTHCHECK` | REQUIRED in every Dockerfile |
| `.dockerignore` | REQUIRED — exclude `.git`, `node_modules`, `*.md`, `.env` |
| Labels | REQUIRED — `maintainer`, `version`, `description` |
| No secrets | NEVER use `ENV`, `ARG`, or `COPY` for secrets |
| Minimal layers | Combine `RUN` commands with `&&` to reduce layers |
| Package cleanup | Remove caches in the same `RUN` instruction |
| Pinned packages | Pin OS package versions where possible |

### 3.2. .NET Backend Dockerfile

```dockerfile
# ============================================
# Stage 1: Build
# ============================================
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
WORKDIR /src

# Copy solution and project files first (layer caching)
COPY *.sln .
COPY src/Utopia.Host/*.csproj src/Utopia.Host/
COPY src/Utopia.SharedKernel/*.csproj src/Utopia.SharedKernel/
COPY src/Utopia.Identity/*.csproj src/Utopia.Identity/
COPY src/Utopia.Catalog/*.csproj src/Utopia.Catalog/
COPY tests/Utopia.Tests/*.csproj tests/Utopia.Tests/

# Restore with locked mode
RUN dotnet restore --locked-mode

# Copy remaining source
COPY src/ src/
COPY tests/ tests/

# Build and publish
RUN dotnet publish src/Utopia.Host/Utopia.Host.csproj \
    -c Release \
    -o /app/publish \
    --no-restore \
    /p:UseAppHost=false

# ============================================
# Stage 2: Runtime
# ============================================
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime

# Labels
LABEL maintainer="Vox"
LABEL version="1.0.0"
LABEL description="Utopia Backend API"

# Security: Non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

WORKDIR /app
COPY --from=build --chown=1001:1001 /app/publish .

# Security: Non-root execution
USER 1001:1001

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health/live || exit 1

# Entrypoint
ENTRYPOINT ["dotnet", "Utopia.Host.dll"]
```

### 3.3. Next.js Frontend Dockerfile

```dockerfile
# ============================================
# Stage 1: Dependencies
# ============================================
FROM node:20-alpine AS deps
WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml ./
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# ============================================
# Stage 2: Build
# ============================================
FROM node:20-alpine AS build
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build with standalone output
ENV NEXT_TELEMETRY_DISABLED=1
RUN corepack enable pnpm && pnpm build

# ============================================
# Stage 3: Runtime
# ============================================
FROM node:20-alpine AS runtime

LABEL maintainer="Vox"
LABEL version="1.0.0"
LABEL description="Utopia Frontend"

WORKDIR /app

# Security: Non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# Copy standalone output
COPY --from=build --chown=1001:1001 /app/.next/standalone ./
COPY --from=build --chown=1001:1001 /app/.next/static ./.next/static
COPY --from=build --chown=1001:1001 /app/public ./public

USER 1001:1001

EXPOSE 3000

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/ || exit 1

CMD ["node", "server.js"]
```

### 3.4. .dockerignore Template

```
# Version control
.git
.gitignore

# Documentation
*.md
documents/
LICENSE

# IDE
.vscode/
.idea/
*.swp
*.swo

# Environment
.env
.env.*

# Node
node_modules/
.next/
coverage/

# .NET
bin/
obj/
*.user

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Terraform
.terraform/
*.tfstate*

# Tests
tests/
*.test.*
*.spec.*
```

## 4. Image Tagging Strategy

### 4.1. Tag Format

| Tag Format | Example | Usage |
|------------|---------|-------|
| `<semver>` | `1.2.3` | Release version (production) |
| `<semver>-<sha7>` | `1.2.3-abc1234` | Traceable release |
| `<branch>-<sha7>` | `main-abc1234` | Branch build (pre-release) |
| `sha-<sha7>` | `sha-abc1234` | Immutable commit reference |

### 4.2. Tagging Rules

- NEVER use `latest` tag in any environment
- Production deployments MUST use SemVer tags
- ArgoCD MUST reference immutable tags or digests
- CI MUST tag with both SemVer and SHA for traceability
- Old tags MUST be cleaned up (Harbor garbage collection)

## 5. Image Registry

### 5.1. Harbor Configuration

| Setting | Value |
|---------|-------|
| **Registry URL** | `harbor.utopia.local` |
| **Project** | `utopia` (private) |
| **Auto-scan** | Enabled (Trivy) |
| **Vulnerability policy** | Prevent pull on critical severity |
| **Garbage collection** | Weekly, keep last 10 tags per repo |
| **Replication** | None (single instance) |

### 5.2. Repository Structure

```
harbor.utopia.local/utopia/
├── backend-api        # .NET Backend
├── frontend           # Next.js Frontend
└── migration          # Database migration job
```

### 5.3. Robot Accounts

| Account | Scope | Permissions |
|---------|-------|-------------|
| `ci-push` | `utopia` project | push, pull |
| `k8s-pull` | `utopia` project | pull only |

## 6. Docker Compose (Local Development)

### 6.1. Development Compose

```yaml
# docker-compose.dev.yaml
version: "3.9"

services:
  postgres:
    image: postgres:16-alpine
    container_name: utopia-postgres
    environment:
      POSTGRES_USER: utopia
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: utopia
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./infrastructure/docker/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U utopia"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: utopia-redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    container_name: utopia-rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: utopia
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "check_running"]
      interval: 10s
      timeout: 5s
      retries: 5

  keycloak:
    image: quay.io/keycloak/keycloak:24.0
    container_name: utopia-keycloak
    command: start-dev --import-realm
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/utopia
      KC_DB_SCHEMA: identity_kc
      KC_DB_USERNAME: utopia
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD}
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD}
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./infrastructure/keycloak/realm-export.json:/opt/keycloak/data/import/realm.json

  mailpit:
    image: axllent/mailpit:latest
    container_name: utopia-mailpit
    ports:
      - "8025:8025"  # Web UI
      - "1025:1025"  # SMTP

volumes:
  postgres-data:
  redis-data:
  rabbitmq-data:
```

### 6.2. Environment Variables

```bash
# .env.example (committed to git — no real values)
POSTGRES_PASSWORD=change-me-in-local
REDIS_PASSWORD=change-me-in-local
RABBITMQ_PASSWORD=change-me-in-local
KEYCLOAK_ADMIN_PASSWORD=change-me-in-local
```

> **.env** file MUST be in `.gitignore`. Only `.env.example` is committed.

## 7. Image Scanning

### 7.1. Scan Pipeline

| Stage | Tool | When | Threshold |
|-------|------|------|-----------|
| **CI — filesystem** | Trivy `fs` | Every PR | 0 critical, 0 high (fixable) |
| **CI — image** | Trivy `image` | After build | 0 critical, 0 high (fixable) |
| **Registry** | Harbor (Trivy) | On push | Block pull on critical |
| **Runtime** | Trivy Operator | Weekly | Alert on new findings |

### 7.2. Trivy CI Configuration

```yaml
# .github/workflows/scan.yaml (excerpt)
- name: Scan image
  uses: aquasecurity/trivy-action@<sha>
  with:
    image-ref: harbor.utopia.local/utopia/backend-api:${{ github.sha }}
    format: sarif
    output: trivy-results.sarif
    severity: CRITICAL,HIGH
    exit-code: 1
```

## 8. Build Optimization

### 8.1. Layer Caching Strategy

| Language | Strategy |
|----------|----------|
| .NET | Copy `.csproj` + `sln` first → `dotnet restore` → copy source → `dotnet publish` |
| Node.js | Copy `package.json` + `pnpm-lock.yaml` first → `pnpm install` → copy source → `pnpm build` |

### 8.2. Build Performance Tips

- Use BuildKit (`DOCKER_BUILDKIT=1`) for parallel stage execution
- Cache mount for package managers: `--mount=type=cache,target=/root/.nuget`
- Use `.dockerignore` to minimize build context size
- Keep frequently-changing layers at the bottom of Dockerfile

## 9. References

- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [Docker Best Practices](https://docs.docker.com/build/building/best-practices/)
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md)
- [SUPPLY-CHAIN-SECURITY.md](../04-security/SUPPLY-CHAIN-SECURITY.md)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md)
- [KUBERNETES-ARCHITECTURE.md](./KUBERNETES-ARCHITECTURE.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
