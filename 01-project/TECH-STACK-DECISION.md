# Tech Stack Decision

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |

---

## 1. Purpose

This document consolidates all technology choices for the **Utopia** project with rationale, alternatives considered, and references to detailed Architecture Decision Records (ADRs).

## 2. Scope

All technology selections across backend, frontend, infrastructure, DevOps, DevSecOps, and observability.

## 3. Stack Summary

### 3.1. Application Layer

| Category | Choice | Version | ADR |
|----------|--------|---------|-----|
| Backend Runtime | .NET | 8 LTS | [ADR-0001](../03-adr/ADR-0001-modulith-architecture.md) |
| Backend Architecture | Modulith (Modular Monolith) | — | [ADR-0001](../03-adr/ADR-0001-modulith-architecture.md) |
| API Style | Minimal API + FastEndpoints | — | [ADR-0001](../03-adr/ADR-0001-modulith-architecture.md) |
| ORM | EF Core + Dapper | 8.x | — |
| Validation | FluentValidation | Latest | — |
| Object Mapping | Mapster | Latest | — |
| Frontend Framework | Next.js (App Router) | 14+ | [ADR-0003](../03-adr/ADR-0003-nextjs-frontend.md) |
| Frontend Language | TypeScript | 5.x (strict) | [ADR-0003](../03-adr/ADR-0003-nextjs-frontend.md) |
| UI Components | shadcn/ui + Tailwind CSS | 4.x | [ADR-0003](../03-adr/ADR-0003-nextjs-frontend.md) |
| State Management | Zustand + TanStack Query | v5 | — |
| Form Handling | React Hook Form + Zod | — | — |

### 3.2. Data & Messaging Layer

| Category | Choice | Version | ADR |
|----------|--------|---------|-----|
| Primary Database | PostgreSQL | 16 | [ADR-0002](../03-adr/ADR-0002-postgresql-over-mssql.md) |
| Cache | Redis (Valkey) | 7.x | — |
| Message Broker | RabbitMQ | 3.13+ | — |
| Message Framework | MassTransit | Latest | — |

### 3.3. Identity & Security Layer

| Category | Choice | Version | ADR |
|----------|--------|---------|-----|
| Identity Provider | Keycloak | Latest | [ADR-0004](../03-adr/ADR-0004-keycloak-identity.md) |
| Auth Protocol | OAuth 2.0 / OIDC | — | [ADR-0004](../03-adr/ADR-0004-keycloak-identity.md) |
| Secret Management | HashiCorp Vault | Latest | — |
| SAST | SonarQube Community + Semgrep | — | — |
| SCA / Container Scan | Trivy | Latest | — |
| Secret Scanning | Gitleaks + TruffleHog | — | — |
| DAST | OWASP ZAP | Latest | — |
| Policy Engine | OPA / Gatekeeper | — | — |

### 3.4. Infrastructure Layer

| Category | Choice | Version | ADR |
|----------|--------|---------|-----|
| IaC | Terraform (OpenTofu) | ≥1.7 | [ADR-0007](../03-adr/ADR-0007-terraform-iac.md) |
| Configuration Mgmt | Ansible | Latest | — |
| Container Runtime | Docker | Latest | — |
| Container Registry | Harbor | Latest | — |
| Orchestration | Kubernetes (K3s) | ≥1.28 | [ADR-0006](../03-adr/ADR-0006-k3s-local-kubernetes.md) |
| Local K8s | K3d (K3s in Docker) | Latest | [ADR-0006](../03-adr/ADR-0006-k3s-local-kubernetes.md) |
| Package Manager | Helm 3 | ≥3.14 | — |
| Ingress Controller | Traefik | Bundled with K3s | — |

### 3.5. CI/CD & GitOps Layer

| Category | Choice | Version | ADR |
|----------|--------|---------|-----|
| CI/CD Platform | GitHub Actions | — | [ADR-0005](../03-adr/ADR-0005-github-actions-cicd.md) |
| GitOps | ArgoCD | Latest | [ADR-0008](../03-adr/ADR-0008-gitops-argocd.md) |
| Git Strategy | Trunk-Based Development | — | — |
| Commit Convention | Conventional Commits | 1.0.0 | — |

### 3.6. Observability Layer

| Category | Choice | Version | Purpose |
|----------|--------|---------|---------|
| Metrics | Prometheus | Latest | Time-series metrics collection |
| Visualization | Grafana | Latest | Dashboards, alerting |
| Logs | Loki | Latest | Log aggregation + querying |
| Traces | Tempo | Latest | Distributed tracing |
| Instrumentation | OpenTelemetry | Latest | Unified telemetry SDK |
| .NET Logging | Serilog | Latest | Structured logging |

### 3.7. Testing Layer

| Category | Choice | Target |
|----------|--------|--------|
| .NET Unit Testing | xUnit + FluentAssertions + NSubstitute | Backend |
| .NET Integration Testing | Testcontainers + Bogus | Backend |
| Frontend Unit Testing | Vitest + Testing Library | Frontend |
| Frontend E2E Testing | Playwright | Frontend |
| IaC Testing | `terraform validate` + `terraform plan` | Infrastructure |
| Dockerfile Linting | hadolint | Docker |
| CI Linting | actionlint | GitHub Actions |

## 4. Decision Criteria

All technology choices were evaluated against these criteria:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Production Readiness** | High | Proven in production environments |
| **Community & Ecosystem** | High | Active community, good documentation, tooling |
| **DevOps Compatibility** | High | Easy to containerize, automate, observe |
| **Security** | High | Secure defaults, active vulnerability patches |
| **Open Source** | Medium | Prefer open-source to avoid vendor lock-in |
| **Learning Value** | Medium | Industry-relevant skills for DevOps/DevSecOps |
| **Resource Efficiency** | Medium | Runs within 40 GB RAM local environment |
| **Developer Experience** | Medium | Fast feedback loops, good IDE support |

## 5. Resource Estimation (Local Development)

| Component | Estimated RAM | Notes |
|-----------|---------------|-------|
| K3d cluster (3 nodes) | 4–6 GB | Configurable per node |
| PostgreSQL 16 | 1 GB | With default config |
| Redis 7 | 0.5 GB | Caching workload |
| RabbitMQ | 0.5 GB | Moderate message volume |
| Keycloak | 1 GB | With PostgreSQL backend |
| SonarQube | 2 GB | Community edition |
| HashiCorp Vault | 0.5 GB | Dev mode |
| Prometheus + Grafana + Loki | 1.5 GB | Basic retention |
| ArgoCD | 1 GB | Namespace deployed |
| Harbor | 2 GB | With Trivy scanner |
| Docker Desktop + WSL2 | 2 GB | Runtime overhead |
| IDE + Browser | 4 GB | VS Code + Chrome |
| OS (Windows) | 4 GB | Base usage |
| **Total Estimated** | **~25 GB** | **~15 GB headroom** |

## 6. References

- Architecture Decision Records: [03-adr/](../03-adr/)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md)
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
