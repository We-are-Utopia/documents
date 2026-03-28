# Utopia — Documentation Hub

> **Version**: 1.0.0 · **Author**: Vox · **Last Updated**: 2026-03-27

Tài liệu kỹ thuật toàn diện cho dự án **Utopia** — personal production-grade DevOps platform.

---

## Đọc Ở Đâu Trước?

| Bạn muốn… | Bắt đầu tại |
|------------|-------------|
| Hiểu dự án là gì | [PROJECT-CHARTER.md](01-project/PROJECT-CHARTER.md) |
| Setup môi trường dev | [GETTING-STARTED.md](08-onboarding/GETTING-STARTED.md) |
| Bắt đầu code | [DEVELOPMENT-WORKFLOW.md](08-onboarding/DEVELOPMENT-WORKFLOW.md) |
| Hiểu kiến trúc tổng quan | [C4-CONTEXT.md](02-architecture/C4-CONTEXT.md) → [C4-CONTAINER.md](02-architecture/C4-CONTAINER.md) |
| Tìm convention/chuẩn | [CODING-STANDARD.md](00-standards/CODING-STANDARD.md) |
| Tra API endpoint | [API-CONTRACTS.md](07-api/API-CONTRACTS.md) |

---

## Reading Order (Recommended)

```
 1. PROJECT-CHARTER          → Vision, goals, scope
 2. TECH-STACK-DECISION      → Why each technology was chosen
 3. GLOSSARY                 → Terminology reference
 │
 4. C4-CONTEXT               → System boundary (Level 1)
 5. C4-CONTAINER             → Containers & ports (Level 2)
 6. C4-COMPONENT             → Internal components (Level 3)
 7. DATA-ARCHITECTURE        → Database schemas, caching
 8. INTEGRATION-ARCHITECTURE → REST API, events, external systems
 │
 9. ADR-0001 → ADR-0008      → Key architectural decisions
 │
10. KUBERNETES-ARCHITECTURE  → K3d cluster, namespaces, resources
11. DOCKER-STRATEGY          → Image builds, registry
12. TERRAFORM-STRUCTURE      → IaC modules, state
13. NETWORKING               → DNS, ingress, TLS, NetworkPolicy
14. STORAGE-BACKUP           → PVCs, backup/restore
 │
15. CI-PIPELINE              → GitHub Actions workflows
16. CD-PIPELINE              → ArgoCD GitOps deployment
17. ENVIRONMENT-STRATEGY     → dev/staging config differences
18. MONITORING-OBSERVABILITY → Prometheus, Grafana, Loki, Tempo
```

---

## Folder Structure

### [`00-standards/`](00-standards/) — Coding & Documentation Standards

Quy tắc chung áp dụng cho toàn bộ dự án.

| File | Mô tả |
|------|--------|
| [DOCUMENTATION-STANDARD.md](00-standards/DOCUMENTATION-STANDARD.md) | Chuẩn viết tài liệu, metadata header, RFC 2119 |
| [CODING-STANDARD.md](00-standards/CODING-STANDARD.md) | Convention cho .NET, TypeScript, Terraform, Docker, Git |
| [SECURITY-STANDARD.md](00-standards/SECURITY-STANDARD.md) | ISO 27001 controls, OWASP Top 10, auth, crypto |
| [VERSIONING-STANDARD.md](00-standards/VERSIONING-STANDARD.md) | SemVer 2.0, Docker tagging, API versioning |
| [REVIEW-CHECKLIST.md](00-standards/REVIEW-CHECKLIST.md) | PR review checklist (code, security, IaC, DB) |

### [`01-project/`](01-project/) — Project Definition

| File | Mô tả |
|------|--------|
| [PROJECT-CHARTER.md](01-project/PROJECT-CHARTER.md) | Vision, 8 goals, scope, hardware, risk matrix |
| [GLOSSARY.md](01-project/GLOSSARY.md) | 50+ thuật ngữ A–Z |
| [TECH-STACK-DECISION.md](01-project/TECH-STACK-DECISION.md) | Full stack với versions, so sánh, resource estimation |

### [`02-architecture/`](02-architecture/) — System Architecture

| File | Mô tả |
|------|--------|
| [C4-CONTEXT.md](02-architecture/C4-CONTEXT.md) | Level 1 — actors, system boundary |
| [C4-CONTAINER.md](02-architecture/C4-CONTAINER.md) | Level 2 — 16 containers, ports, namespace layout |
| [C4-COMPONENT.md](02-architecture/C4-COMPONENT.md) | Level 3 — modules, middleware pipeline |
| [DATA-ARCHITECTURE.md](02-architecture/DATA-ARCHITECTURE.md) | ER diagrams, EF Core, Redis, events, encryption |
| [INTEGRATION-ARCHITECTURE.md](02-architecture/INTEGRATION-ARCHITECTURE.md) | REST API, MassTransit events, Keycloak/SMTP integration |

### [`03-adr/`](03-adr/) — Architecture Decision Records

| File | Decision |
|------|----------|
| [ADR-TEMPLATE.md](03-adr/ADR-TEMPLATE.md) | Template for new ADRs |
| [ADR-0001](03-adr/ADR-0001-modulith-architecture.md) | Modulith over Microservices |
| [ADR-0002](03-adr/ADR-0002-postgresql-over-mssql.md) | PostgreSQL over MSSQL |
| [ADR-0003](03-adr/ADR-0003-nextjs-frontend.md) | Next.js for Frontend |
| [ADR-0004](03-adr/ADR-0004-keycloak-identity.md) | Keycloak for Identity |
| [ADR-0005](03-adr/ADR-0005-github-actions-cicd.md) | GitHub Actions for CI/CD |
| [ADR-0006](03-adr/ADR-0006-k3s-local-kubernetes.md) | K3s (K3d) for Local Kubernetes |
| [ADR-0007](03-adr/ADR-0007-terraform-iac.md) | Terraform for IaC |
| [ADR-0008](03-adr/ADR-0008-gitops-argocd.md) | ArgoCD for GitOps |

### [`04-security/`](04-security/) — Security & Compliance (ISO 27001)

| File | Mô tả |
|------|--------|
| [ISMS-SCOPE.md](04-security/ISMS-SCOPE.md) | ISO 27001 Clause 4.3, asset inventory |
| [RISK-ASSESSMENT.md](04-security/RISK-ASSESSMENT.md) | 25 risks, 5×5 matrix, treatment plan |
| [ACCESS-CONTROL-POLICY.md](04-security/ACCESS-CONTROL-POLICY.md) | RBAC roles, Keycloak, K8s RBAC, DB access |
| [SECURE-DEVELOPMENT-POLICY.md](04-security/SECURE-DEVELOPMENT-POLICY.md) | SSDLC, STRIDE, quality gates, OPA policies |
| [CRYPTOGRAPHY-POLICY.md](04-security/CRYPTOGRAPHY-POLICY.md) | Algorithms, TLS, Vault key management |
| [INCIDENT-RESPONSE-PLAN.md](04-security/INCIDENT-RESPONSE-PLAN.md) | 7-phase IRP, severity levels, playbooks |
| [SUPPLY-CHAIN-SECURITY.md](04-security/SUPPLY-CHAIN-SECURITY.md) | Dependency rules, SLSA Level 2, Harbor |
| [OWASP-ASVS-CHECKLIST.md](04-security/OWASP-ASVS-CHECKLIST.md) | 115 ASVS Level 2 requirements |

### [`05-infrastructure/`](05-infrastructure/) — Infrastructure

| File | Mô tả |
|------|--------|
| [KUBERNETES-ARCHITECTURE.md](05-infrastructure/KUBERNETES-ARCHITECTURE.md) | K3d cluster, 9 namespaces, PSS, HPA, Helm |
| [DOCKER-STRATEGY.md](05-infrastructure/DOCKER-STRATEGY.md) | Multi-stage builds, Harbor registry, Trivy |
| [TERRAFORM-STRUCTURE.md](05-infrastructure/TERRAFORM-STRUCTURE.md) | 6 modules, state management, CI/CD |
| [NETWORKING.md](05-infrastructure/NETWORKING.md) | DNS, Traefik, NetworkPolicy, TLS, CORS |
| [STORAGE-BACKUP.md](05-infrastructure/STORAGE-BACKUP.md) | PVCs, backup CronJob, restore procedures |

### [`06-devops/`](06-devops/) — CI/CD & Operations

| File | Mô tả |
|------|--------|
| [CI-PIPELINE.md](06-devops/CI-PIPELINE.md) | GitHub Actions: backend/frontend/IaC workflows, 20+ quality gates |
| [CD-PIPELINE.md](06-devops/CD-PIPELINE.md) | ArgoCD GitOps, ApplicationSet, sync waves, rollback |
| [ENVIRONMENT-STRATEGY.md](06-devops/ENVIRONMENT-STRATEGY.md) | dev/staging, config differences, promotion flow |
| [MONITORING-OBSERVABILITY.md](06-devops/MONITORING-OBSERVABILITY.md) | Prometheus, Grafana, Loki, Tempo, OTel, alerting, SLOs |

#### [`06-devops/runbooks/`](06-devops/runbooks/) — Operational Runbooks

| File | Mô tả |
|------|--------|
| [RUNBOOK-TEMPLATE.md](06-devops/runbooks/RUNBOOK-TEMPLATE.md) | Standard runbook structure |
| [RUNBOOK-DATABASE-FAILOVER.md](06-devops/runbooks/RUNBOOK-DATABASE-FAILOVER.md) | PostgreSQL recovery (4 scenarios) |

### [`07-api/`](07-api/) — API Specification

| File | Mô tả |
|------|--------|
| [API-CONTRACTS.md](07-api/API-CONTRACTS.md) | Endpoint specs: Identity (6) + Catalog (11), validation, rate limits |
| [`contracts/`](07-api/contracts/) | Auto-generated OpenAPI 3.1 specs (from CI) |

### [`08-onboarding/`](08-onboarding/) — Developer Onboarding

| File | Mô tả |
|------|--------|
| [GETTING-STARTED.md](08-onboarding/GETTING-STARTED.md) | Full setup guide: tools, K3d, DNS, Terraform, services |
| [DEVELOPMENT-WORKFLOW.md](08-onboarding/DEVELOPMENT-WORKFLOW.md) | Daily workflow: branching, commits, testing, PR, deploy |

---

## Standards & Conventions

Tất cả tài liệu tuân theo:

- **Metadata header**: Version, Status, Author, Reviewer, Created, Updated
- **RFC 2119 keywords**: MUST, SHOULD, MAY (viết hoa = bắt buộc tuân thủ)
- **Diagrams**: Mermaid syntax (C4, flowchart, sequence, ER)
- **Versioning**: SemVer 2.0.0
- **Naming**: `UPPER-KEBAB-CASE.md`

Chi tiết: [DOCUMENTATION-STANDARD.md](00-standards/DOCUMENTATION-STANDARD.md)

---

## Tech Stack Summary

| Layer | Technology |
|-------|-----------|
| Backend | .NET 8 LTS, C# 12, Modulith, FastEndpoints, EF Core 8 |
| Frontend | Next.js 14+, TypeScript 5, Tailwind CSS 4, shadcn/ui |
| Database | PostgreSQL 16, Redis 7 |
| Messaging | RabbitMQ + MassTransit |
| Identity | Keycloak (OAuth 2.0 / OIDC) |
| CI/CD | GitHub Actions + ArgoCD (GitOps) |
| Infrastructure | K3d (K3s), Terraform, Docker, Helm |
| Security | Vault, Trivy, Semgrep, SonarQube, Gitleaks, OPA |
| Observability | Prometheus, Grafana, Loki, Tempo, OpenTelemetry |

---

## File Count

| Folder | Files |
|--------|-------|
| 00-standards | 5 |
| 01-project | 3 |
| 02-architecture | 5 |
| 03-adr | 9 |
| 04-security | 8 |
| 05-infrastructure | 5 |
| 06-devops | 4 + 2 runbooks |
| 07-api | 1 + contracts/ |
| 08-onboarding | 2 |
| **Total** | **44** |