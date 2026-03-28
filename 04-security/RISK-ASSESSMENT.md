# Risk Assessment

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | ISO/IEC 27001:2022, Clause 6.1.2; ISO/IEC 27005 |

---

## 1. Purpose

This document identifies, analyzes, and evaluates information security risks for the **Utopia** project. It defines the risk assessment methodology, risk register, and treatment plan in accordance with ISO 27001:2022, Clause 6.1.2.

## 2. Scope

All information assets within the ISMS scope as defined in [ISMS-SCOPE.md](./ISMS-SCOPE.md).

## 3. Risk Assessment Methodology

### 3.1. Risk Identification

Risks are identified through:

- Threat modeling of architecture components
- OWASP Top 10 mapping
- Supply chain analysis
- Infrastructure vulnerability assessment
- Historical incident analysis (industry)

### 3.2. Risk Analysis

Each risk is assessed using a **Likelihood × Impact** matrix.

#### Likelihood Scale

| Level | Score | Description |
|-------|-------|-------------|
| **Rare** | 1 | Unlikely to occur (once per year or less) |
| **Unlikely** | 2 | Could occur but not expected (quarterly) |
| **Possible** | 3 | Might occur (monthly) |
| **Likely** | 4 | Expected to occur (weekly) |
| **Almost Certain** | 5 | Will occur frequently (daily) |

#### Impact Scale

| Level | Score | Description |
|-------|-------|-------------|
| **Negligible** | 1 | No meaningful disruption |
| **Minor** | 2 | Minor inconvenience, quick recovery |
| **Moderate** | 3 | Service degradation, hours of recovery |
| **Major** | 4 | Service outage, data exposure, days of recovery |
| **Critical** | 5 | Complete system compromise, data breach, weeks of recovery |

#### Risk Score Matrix

```
Impact →        1        2        3        4        5
Likelihood ↓   Negl.   Minor    Mod.    Major   Crit.
─────────────────────────────────────────────────────
5 Almost Cert.   5       10       15      20      25
4 Likely          4        8       12      16      20
3 Possible        3        6        9      12      15
2 Unlikely        2        4        6       8      10
1 Rare            1        2        3       4       5
```

| Score Range | Risk Level | Required Action |
|-------------|-----------|-----------------|
| 1–4 | **Low** | Accept or monitor |
| 5–9 | **Medium** | Mitigate within 30 days |
| 10–15 | **High** | Mitigate within 7 days |
| 16–25 | **Critical** | Mitigate immediately |

### 3.3. Risk Treatment Options

| Option | When to Use |
|--------|-------------|
| **Mitigate** | Implement controls to reduce likelihood or impact |
| **Accept** | Risk is within acceptable threshold (score ≤ 4) |
| **Transfer** | Shift risk to third party (e.g., cloud provider managed service) |
| **Avoid** | Eliminate the activity that introduces the risk |

## 4. Risk Register

### 4.1. Application Security Risks

| ID | Risk | Threat | Asset | L | I | Score | Level | Treatment | Controls |
|----|------|--------|-------|---|---|-------|-------|-----------|----------|
| R-APP-001 | SQL Injection | External attacker | PostgreSQL databases | 2 | 5 | 10 | High | Mitigate | Parameterized queries (EF Core), SAST (SonarQube), DAST (ZAP) |
| R-APP-002 | Cross-Site Scripting (XSS) | External attacker | Frontend / API responses | 2 | 4 | 8 | Medium | Mitigate | React auto-escaping, CSP headers, output encoding |
| R-APP-003 | Broken Authentication | External attacker | User accounts | 2 | 5 | 10 | High | Mitigate | Keycloak (battle-tested), MFA, rate limiting, session management |
| R-APP-004 | Broken Access Control | External attacker | Protected resources | 3 | 4 | 12 | High | Mitigate | RBAC, server-side authorization, architecture tests |
| R-APP-005 | Sensitive Data Exposure | External attacker | PII, credentials | 2 | 5 | 10 | High | Mitigate | TLS 1.3, encryption at rest, Vault secrets, no PII in logs |
| R-APP-006 | Mass Assignment | External attacker | API endpoints | 3 | 3 | 9 | Medium | Mitigate | Explicit DTOs, FluentValidation, no entity binding |
| R-APP-007 | Insecure Deserialization | External attacker | API endpoints | 1 | 4 | 4 | Low | Accept | System.Text.Json (safe defaults), no polymorphic deserialization |
| R-APP-008 | Server-Side Request Forgery | External attacker | Backend API | 2 | 4 | 8 | Medium | Mitigate | URL allowlisting, no user-controlled URLs in server requests |

### 4.2. Infrastructure Risks

| ID | Risk | Threat | Asset | L | I | Score | Level | Treatment | Controls |
|----|------|--------|-------|---|---|-------|-------|-----------|----------|
| R-INF-001 | Container Image Vulnerability | Known CVE | Docker images | 4 | 3 | 12 | High | Mitigate | Trivy scanning in CI, minimal base images (Alpine/distroless) |
| R-INF-002 | Kubernetes Misconfiguration | Human error | K8s cluster | 3 | 4 | 12 | High | Mitigate | OPA Gatekeeper, Pod Security Standards, IaC review |
| R-INF-003 | Exposed Secrets in Git | Human error | API keys, passwords | 3 | 5 | 15 | High | Mitigate | Gitleaks pre-commit, TruffleHog CI scan, Vault for secrets |
| R-INF-004 | Terraform State Exposure | Misconfiguration | Cloud resources | 2 | 5 | 10 | High | Mitigate | Encrypted remote state, access-controlled backend, state locking |
| R-INF-005 | Unauthorized K8s Access | External/Internal | Cluster resources | 2 | 5 | 10 | High | Mitigate | RBAC, namespace isolation, no cluster-admin for apps |
| R-INF-006 | Container Escape | External attacker | Host system | 1 | 5 | 5 | Medium | Mitigate | Non-root containers, seccomp profiles, read-only rootfs |
| R-INF-007 | Database Backup Failure | System failure | Data assets | 2 | 4 | 8 | Medium | Mitigate | Automated backup validation, monitoring alerts |
| R-INF-008 | Resource Exhaustion (DoS) | External attacker | All services | 2 | 3 | 6 | Medium | Mitigate | Rate limiting, resource limits in K8s, monitoring |

### 4.3. Supply Chain Risks

| ID | Risk | Threat | Asset | L | I | Score | Level | Treatment | Controls |
|----|------|--------|-------|---|---|-------|-------|-----------|----------|
| R-SUP-001 | Malicious Dependency | Supply chain attack | NuGet/npm packages | 2 | 5 | 10 | High | Mitigate | SCA (Trivy), lock files, Dependabot alerts |
| R-SUP-002 | Compromised CI Action | Supply chain attack | GitHub Actions | 2 | 5 | 10 | High | Mitigate | Pin actions by SHA, minimal permissions, OIDC federation |
| R-SUP-003 | Compromised Base Image | Supply chain attack | Docker images | 2 | 4 | 8 | Medium | Mitigate | Official images only, image signing, digest pinning |
| R-SUP-004 | Vulnerable Transitive Dependency | Known CVE | Application runtime | 3 | 3 | 9 | Medium | Mitigate | Deep SCA scanning, lock file analysis, automated updates |

### 4.4. Operational Risks

| ID | Risk | Threat | Asset | L | I | Score | Level | Treatment | Controls |
|----|------|--------|-------|---|---|-------|-------|-----------|----------|
| R-OPS-001 | Monitoring Blind Spot | System failure | Observability stack | 2 | 3 | 6 | Medium | Mitigate | Prometheus self-monitoring, Grafana alerts on data gaps |
| R-OPS-002 | Certificate Expiry | Misconfiguration | TLS certificates | 2 | 4 | 8 | Medium | Mitigate | cert-manager auto-renewal, expiry monitoring |
| R-OPS-003 | Drift Between IaC and Reality | Human error | Infrastructure state | 3 | 3 | 9 | Medium | Mitigate | Terraform plan in CI, ArgoCD drift detection |
| R-OPS-004 | Log Data Loss | System failure | Audit trail | 2 | 3 | 6 | Medium | Mitigate | Loki persistence, log shipping redundancy |
| R-OPS-005 | Single Point of Failure (Operator) | Bus factor = 1 | Entire project | 5 | 2 | 10 | High | Accept | Comprehensive documentation, runbooks, IaC reproducibility |

## 5. Risk Heat Map

```mermaid
quadrantChart
    title Risk Heat Map
    x-axis Low Impact --> High Impact
    y-axis Low Likelihood --> High Likelihood
    quadrant-1 Monitor Closely
    quadrant-2 Mitigate Immediately
    quadrant-3 Accept / Monitor
    quadrant-4 Mitigate (Plan)
    R-APP-004: [0.65, 0.55]
    R-INF-001: [0.52, 0.72]
    R-INF-003: [0.85, 0.55]
    R-SUP-001: [0.85, 0.38]
    R-APP-001: [0.85, 0.35]
    R-OPS-005: [0.35, 0.88]
    R-INF-002: [0.68, 0.55]
    R-APP-006: [0.52, 0.55]
    R-OPS-003: [0.52, 0.55]
```

## 6. Risk Treatment Plan

### 6.1. High / Critical Risks — Action Plan

| Risk ID | Risk | Action | Control Reference | Target Date | Status |
|---------|------|--------|-------------------|-------------|--------|
| R-INF-003 | Secrets in Git | Implement Gitleaks pre-commit + CI scan | [SECURITY-STANDARD](../00-standards/SECURITY-STANDARD.md) §7 | Phase 3 (CI setup) | Not Started |
| R-APP-001 | SQL Injection | Enforce parameterized queries, SAST rules | [SECURITY-STANDARD](../00-standards/SECURITY-STANDARD.md) §5.2 | Phase 2 (Backend) | Not Started |
| R-APP-003 | Broken Auth | Deploy Keycloak, configure MFA, rate limits | [ACCESS-CONTROL-POLICY](./ACCESS-CONTROL-POLICY.md) | Phase 2 (Backend) | Not Started |
| R-APP-004 | Broken Access Control | Implement RBAC middleware, architecture tests | [ACCESS-CONTROL-POLICY](./ACCESS-CONTROL-POLICY.md) | Phase 2 (Backend) | Not Started |
| R-APP-005 | Data Exposure | Enable TLS everywhere, Vault for secrets | [CRYPTOGRAPHY-POLICY](./CRYPTOGRAPHY-POLICY.md) | Phase 3 (CI setup) | Not Started |
| R-INF-001 | Container CVEs | Trivy scanning in CI, Alpine base images | [SUPPLY-CHAIN-SECURITY](./SUPPLY-CHAIN-SECURITY.md) | Phase 3 (CI setup) | Not Started |
| R-INF-002 | K8s Misconfig | OPA Gatekeeper policies, PSS restricted | [SECURITY-STANDARD](../00-standards/SECURITY-STANDARD.md) §9.2 | Phase 6 (K8s) | Not Started |
| R-INF-004 | TF State Exposure | Encrypted remote backend with locking | [SECURITY-STANDARD](../00-standards/SECURITY-STANDARD.md) §7 | Phase 5 (IaC) | Not Started |
| R-SUP-001 | Malicious Dependency | SCA in CI, lock files, Dependabot | [SUPPLY-CHAIN-SECURITY](./SUPPLY-CHAIN-SECURITY.md) | Phase 3 (CI setup) | Not Started |
| R-SUP-002 | Compromised CI Action | Pin by SHA, least-privilege permissions | [CODING-STANDARD](../00-standards/CODING-STANDARD.md) §7 | Phase 3 (CI setup) | Not Started |

### 6.2. Accepted Risks

| Risk ID | Risk | Score | Justification |
|---------|------|-------|---------------|
| R-APP-007 | Insecure Deserialization | 4 (Low) | System.Text.Json safe defaults; no polymorphic deserialization used |
| R-OPS-005 | Single Operator | 10 (High) | Inherent to personal project; mitigated by documentation and automation |

## 7. Risk Review Schedule

| Activity | Frequency | Trigger |
|----------|-----------|---------|
| Full risk assessment review | Every 6 months | Scheduled |
| Risk register update | When architecture changes | Architecture ADR |
| Post-incident risk review | After each security incident | Incident closure |
| Dependency risk review | Monthly | Dependabot/Renovate reports |
| Threat model update | When new modules are added | Module creation |

## 8. References

- [ISO/IEC 27001:2022](https://www.iso.org/standard/27001) — Clause 6.1.2: Information security risk assessment
- [ISO/IEC 27005:2022](https://www.iso.org/standard/80585.html) — Information security risk management
- [OWASP Top 10 (2021)](https://owasp.org/www-project-top-ten/)
- [ISMS-SCOPE.md](./ISMS-SCOPE.md) — ISMS scope and assets
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) — Control implementations
- [SUPPLY-CHAIN-SECURITY.md](./SUPPLY-CHAIN-SECURITY.md) — Supply chain controls

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
