# ADR-0004: Keycloak for Identity and Access Management

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

Utopia requires an Identity and Access Management (IAM) solution that provides OAuth 2.0 / OpenID Connect (OIDC) for both the frontend (Next.js) and backend (.NET API). The solution must support self-hosting in Kubernetes, integrate with the DevSecOps pipeline, and follow the security requirements defined in [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md).

## Decision Drivers

- **Standards compliance** — full OAuth 2.0 / OpenID Connect implementation
- **Self-hosted** — must run on local K8s without cloud dependencies
- **Production proven** — battle-tested in enterprise environments
- **DevOps friendly** — containerized, configurable via IaC, Kubernetes-native
- **Feature completeness** — SSO, MFA, RBAC, user management, social login
- **Open source** — no licensing costs

## Considered Options

1. **Keycloak** — open-source IAM by Red Hat / CNCF
2. **Duende IdentityServer** — .NET-native OIDC server (commercial license)
3. **Authentik** — open-source identity provider
4. **Custom implementation** — ASP.NET Core Identity + custom OAuth

## Decision

We will use **Keycloak** as the Identity and Access Management solution for Utopia.

Keycloak will be deployed in Kubernetes, configured via Terraform (realm, clients, roles), and integrated with:
- **Next.js frontend** via Auth.js (NextAuth) with OIDC provider
- **.NET backend** via JWT bearer authentication middleware

## Comparison

| Criterion | Keycloak | Duende IdentityServer | Authentik | Custom |
|-----------|----------|-----------------------|-----------|--------|
| Open source | ✅ Apache 2.0 | ❌ Commercial ($) | ✅ Open-source | ✅ |
| OAuth 2.0 / OIDC | ✅ Full | ✅ Full | ✅ Full | ⚠️ Partial |
| MFA support | ✅ TOTP, WebAuthn | ✅ Via ASP.NET | ✅ TOTP, WebAuthn | ❌ Build yourself |
| Admin UI | ✅ Full admin console | ❌ None (code only) | ✅ Modern UI | ❌ None |
| K8s deployment | ✅ Official Helm chart | ⚠️ Custom container | ✅ Helm chart | ⚠️ Custom |
| Terraform provider | ✅ mrparkers/keycloak | ❌ None | ❌ None | ❌ None |
| User federation | ✅ LDAP, AD, custom | ❌ Via ASP.NET | ✅ LDAP, AD | ❌ Custom |
| Community & docs | ✅ Large (CNCF) | ⚠️ Medium | ⚠️ Growing | ❌ None |
| .NET integration | ✅ JWT middleware | ✅ Native | ✅ JWT middleware | ✅ Native |
| Container image size | ⚠️ ~400 MB | ✅ ~100 MB | ⚠️ ~500 MB | ✅ ~100 MB |
| Learning value | ✅ Industry IAM skill | ⚠️ .NET specific | ⚠️ Niche | ❌ Low |

## Consequences

### Positive

- Full-featured IAM out of the box — SSO, MFA, user management, social login, RBAC
- Terraform provider enables IaC for realm configuration, clients, roles
- Official Helm chart with production-ready deployment patterns
- Admin console reduces operational overhead for user management
- CNCF incubating project — strong community and long-term viability
- Industry-relevant skill — Keycloak is widely used in enterprise environments
- Separates identity concerns from application code

### Negative

- Relatively heavy container image (~400 MB) — mitigation: acceptable given 40 GB RAM, no impact on CI
- Java-based, requires JVM tuning for optimal memory usage — mitigation: configure `-Xms` and `-Xmx` in K8s resource limits
- Configuration complexity for initial setup — mitigation: Terraform provider automates realm setup
- Upgrade path can be complex — mitigation: pin version, test upgrades in staging

### Risks

- Keycloak major version upgrades may break Terraform provider compatibility — likelihood: Low, impact: Medium — mitigation: pin provider version, test before upgrading
- Memory consumption in resource-constrained environments — likelihood: Low (with 40 GB RAM), impact: Low — mitigation: configure JVM heap limits

## References

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Keycloak Helm Chart](https://github.com/bitnami/charts/tree/main/bitnami/keycloak)
- [Terraform Keycloak Provider](https://registry.terraform.io/providers/mrparkers/keycloak/latest)
- [Auth.js Keycloak Provider](https://authjs.dev/getting-started/providers/keycloak)
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) — Section 4: Authentication & Authorization

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
