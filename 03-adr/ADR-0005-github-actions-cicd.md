# ADR-0005: GitHub Actions for CI/CD

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Accepted                             |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |

---

## Context

Utopia requires a CI/CD platform for automated build, test, security scanning, and artifact publishing. The platform must support complex, multi-stage pipelines with quality gates, integrate with the DevSecOps toolchain (SonarQube, Trivy, Gitleaks), and work well with a GitOps deployment model via ArgoCD.

## Decision Drivers

- **Integration with source control** — tight coupling with Git workflow
- **DevSecOps tooling** — rich marketplace of security scanning actions
- **GitOps compatibility** — CI produces artifacts, CD handled by ArgoCD
- **Cost** — free tier for personal/open-source projects
- **Self-hosted runners** — option to run CI on local machine for resource-heavy tasks
- **Reusable workflows** — DRY pipeline definitions
- **Community adoption** — skills transferable to industry

## Considered Options

1. **GitHub Actions** — GitHub-native CI/CD
2. **GitLab CI/CD** — GitLab-native CI/CD
3. **Jenkins** — self-hosted open-source CI server
4. **Tekton** — Kubernetes-native CI/CD framework

## Decision

We will use **GitHub Actions** as the CI/CD platform for Utopia.

CI pipelines will handle: build, test, SAST, SCA, container scanning, and artifact publishing. CD (deployment to Kubernetes) will be handled by ArgoCD via GitOps — GitHub Actions will NOT directly deploy to clusters.

## Comparison

| Criterion | GitHub Actions | GitLab CI | Jenkins | Tekton |
|-----------|---------------|-----------|---------|--------|
| Source control integration | ✅ Native | ✅ Native (GitLab) | ⚠️ Plugin | ⚠️ Triggers |
| Free tier | ✅ 2000 min/month | ✅ 400 min/month | ✅ Self-hosted | ✅ Self-hosted |
| Self-hosted runners | ✅ Easy setup | ✅ Easy setup | ✅ Is self-hosted | ✅ K8s native |
| Marketplace / Actions | ✅ 20k+ actions | ⚠️ Templates | ⚠️ Plugins (legacy) | ❌ Limited |
| Reusable workflows | ✅ Native | ✅ includes | ⚠️ Shared libraries | ✅ Tasks |
| YAML config | ✅ | ✅ | ❌ Groovy/Declarative | ✅ |
| Security scanning actions | ✅ Rich ecosystem | ✅ Built-in | ⚠️ Plugin dependent | ❌ Build yourself |
| Container registry | ✅ GHCR included | ✅ Built-in | ❌ External | ❌ External |
| Learning curve | ✅ Low | ✅ Low | ❌ High | ❌ High |
| Industry adoption | ✅ Highest | ✅ High | ⚠️ Declining | ❌ Niche |

## Consequences

### Positive

- Zero infrastructure to manage — no CI server to maintain
- Native GitHub integration — PR checks, status badges, branch protection
- Rich marketplace — pre-built actions for SonarQube, Trivy, Gitleaks, Docker
- Reusable workflows enable DRY pipeline definitions across backend/frontend/infra
- Matrix builds for testing across multiple configurations
- GHCR (GitHub Container Registry) available as secondary registry
- Self-hosted runners available for local development when needed

### Negative

- Vendor lock-in to GitHub — mitigation: pipelines are YAML-based, concepts transfer to other platforms
- Free tier limits (2000 min/month) may be hit with heavy scanning — mitigation: self-hosted runner as fallback
- No built-in deployment tracking — mitigation: ArgoCD handles deployments and provides dashboard

### Risks

- GitHub Actions outage blocks CI — likelihood: Low, impact: Medium — mitigation: self-hosted runner fallback for critical builds
- Action supply chain attack (malicious action update) — likelihood: Low, impact: High — mitigation: pin actions by SHA, not tag (see [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md))

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Security Hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) — Section 7: CI/CD conventions
- [ADR-0008](./ADR-0008-gitops-argocd.md) — ArgoCD for CD/GitOps

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
