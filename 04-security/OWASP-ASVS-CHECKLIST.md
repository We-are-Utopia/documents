# OWASP ASVS Checklist

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | OWASP ASVS v4.0.3                   |

---

## 1. Purpose

This document maps the OWASP Application Security Verification Standard (ASVS) v4.0.3 requirements to the Utopia project. It serves as a verification checklist to ensure all applicable security requirements are addressed during development and testing.

## 2. Target Level

Utopia targets **ASVS Level 2** (Standard) — appropriate for applications that handle sensitive data and require defense against most common application security risks.

| Level | Description | Target |
|-------|-------------|--------|
| Level 1 | Opportunistic — basic security | Baseline |
| **Level 2** | **Standard — defense against most risks** | **✅ Target** |
| Level 3 | Advanced — high-value, critical systems | Future |

## 3. Verification Checklist

### Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Implemented / Addressed |
| 🔲 | Not yet implemented |
| ➖ | Not applicable |
| L1 / L2 / L3 | Required at that ASVS level |

---

### V1 — Architecture, Design, and Threat Modeling

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V1.1.1 | Secure SDLC in use | | ✅ | 🔲 | [SECURE-DEVELOPMENT-POLICY.md](./SECURE-DEVELOPMENT-POLICY.md) |
| V1.1.2 | Threat modeling for design changes | | ✅ | 🔲 | STRIDE methodology defined |
| V1.1.3 | All user stories include security requirements | | ✅ | 🔲 | Template includes security section |
| V1.1.5 | All application components defined and known | ✅ | ✅ | ✅ | [C4-CONTAINER.md](../02-architecture/C4-CONTAINER.md) |
| V1.1.6 | High-level architecture defined | ✅ | ✅ | ✅ | [C4-CONTEXT.md](../02-architecture/C4-CONTEXT.md) |
| V1.1.7 | All components independently deployable or verifiable | | ✅ | ✅ | Modulith module isolation |
| V1.2.1 | Communications between components are encrypted (TLS) | | ✅ | 🔲 | TLS 1.3 required everywhere |
| V1.2.3 | Authentication pathway is documented | | ✅ | ✅ | [ACCESS-CONTROL-POLICY.md](./ACCESS-CONTROL-POLICY.md) §4 |
| V1.2.4 | Access control centrally enforced | | ✅ | 🔲 | Keycloak + RBAC middleware |
| V1.4.1 | Trusted enforcement points (server-side) | ✅ | ✅ | 🔲 | Backend API authorization |
| V1.4.4 | All access controls enforced at a trusted level | ✅ | ✅ | 🔲 | Server-side only |
| V1.5.1 | Input validation at a trusted level | ✅ | ✅ | 🔲 | FluentValidation (backend) |
| V1.5.3 | Output encoding happens close to the interpreter | ✅ | ✅ | 🔲 | React auto-escaping, API encoding |
| V1.6.1 | Cryptographic key management documented | | ✅ | ✅ | [CRYPTOGRAPHY-POLICY.md](./CRYPTOGRAPHY-POLICY.md) |
| V1.6.2 | Consumers can rotate keys without downtime | | ✅ | 🔲 | Vault key rotation + grace period |
| V1.7.1 | Common logging format and approach | | ✅ | 🔲 | Serilog structured JSON |
| V1.7.2 | Logs processed securely (no injection) | | ✅ | 🔲 | Structured logging prevents injection |
| V1.8.1 | Sensitive data classified | | ✅ | ✅ | [ISMS-SCOPE.md](./ISMS-SCOPE.md) §6, [DATA-ARCHITECTURE.md](../02-architecture/DATA-ARCHITECTURE.md) |
| V1.8.2 | Protection requirements for each classification | | ✅ | ✅ | [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) §10 |
| V1.11.1 | Business logic has documented limits and controls | | ✅ | 🔲 | Per-module design docs (future) |
| V1.14.1 | Segregation of components (different trust levels) | | ✅ | ✅ | K8s namespaces, module boundaries |

### V2 — Authentication

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V2.1.1 | Passwords at least 12 characters | ✅ | ✅ | 🔲 | Keycloak password policy |
| V2.1.2 | Passwords at least 128 characters allowed | ✅ | ✅ | 🔲 | Keycloak default |
| V2.1.5 | Users can change password | ✅ | ✅ | 🔲 | Keycloak account management |
| V2.1.7 | Check passwords against breached password list | ✅ | ✅ | 🔲 | Keycloak HaveIBeenPwned integration |
| V2.1.9 | No password composition rules beyond minimum length | ✅ | ✅ | 🔲 | Per NIST 800-63B |
| V2.1.12 | User can view last login | | ✅ | 🔲 | Keycloak login events |
| V2.2.1 | Anti-automation controls (rate limiting) | ✅ | ✅ | 🔲 | Keycloak brute force protection |
| V2.2.3 | Generic error messages for authentication | ✅ | ✅ | 🔲 | "Invalid credentials" only |
| V2.5.2 | Password reset via secure mechanism | ✅ | ✅ | 🔲 | Keycloak email reset flow |
| V2.7.1 | OTP delivered securely (not SMS) | ✅ | ✅ | 🔲 | TOTP (Authenticator app) |
| V2.8.1 | Time-based OTP | | ✅ | 🔲 | Keycloak TOTP |
| V2.9.1 | Cryptographic keys for authentication (server-side) | | ✅ | 🔲 | RS256 JWT from Keycloak |
| V2.10.1 | No shared or default secrets in service authentication | ✅ | ✅ | 🔲 | Vault-managed unique secrets |
| V2.10.4 | Disable default credentials | ✅ | ✅ | 🔲 | Change all defaults in setup |

### V3 — Session Management

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V3.1.1 | Application never reveals session tokens in URL | ✅ | ✅ | 🔲 | httpOnly cookies, Bearer header |
| V3.2.1 | Sufficient session token entropy (128-bit) | ✅ | ✅ | 🔲 | Keycloak-generated tokens |
| V3.2.3 | Session tokens use approved algorithms | ✅ | ✅ | 🔲 | JWT RS256 |
| V3.3.1 | Logout invalidates session | ✅ | ✅ | 🔲 | Keycloak session invalidation |
| V3.3.2 | Idle timeout (30 min) | ✅ | ✅ | 🔲 | Keycloak SSO Session Idle = 30m |
| V3.3.3 | Absolute timeout (10 hours) | | ✅ | 🔲 | Keycloak SSO Session Max = 10h |
| V3.4.1 | Cookie-based sessions use Secure attribute | ✅ | ✅ | 🔲 | Secure, httpOnly, SameSite=Strict |
| V3.4.2 | Cookie-based sessions use httpOnly attribute | ✅ | ✅ | 🔲 | Next.js cookie configuration |
| V3.4.3 | Cookie-based sessions use SameSite attribute | ✅ | ✅ | 🔲 | SameSite=Strict |
| V3.4.5 | Cookie path set appropriately | ✅ | ✅ | 🔲 | Path=/ |
| V3.5.2 | Application only uses stateless session tokens (JWT) | | ✅ | 🔲 | JWT + refresh token rotation |
| V3.7.1 | Concurrent sessions limited or managed | | ✅ | 🔲 | Keycloak session limits |

### V4 — Access Control

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V4.1.1 | Trusted enforcement point for access control | ✅ | ✅ | 🔲 | Backend API middleware |
| V4.1.2 | All attributes used for access control cannot be manipulated | ✅ | ✅ | 🔲 | Server-side JWT claims extraction |
| V4.1.3 | Principle of least privilege | ✅ | ✅ | 🔲 | RBAC with minimal defaults |
| V4.1.5 | Access controls fail securely (deny by default) | ✅ | ✅ | 🔲 | `[Authorize]` default policy |
| V4.2.1 | Sensitive data and APIs protected against IDOR | ✅ | ✅ | 🔲 | Ownership check on resource access |
| V4.2.2 | Application enforces strong anti-CSRF mechanism | ✅ | ✅ | 🔲 | SameSite cookies + CSRF tokens |
| V4.3.1 | Admin interfaces use appropriate authorization | ✅ | ✅ | 🔲 | `super-admin` / `admin` roles |
| V4.3.3 | Directory listing disabled | ✅ | ✅ | 🔲 | API-only backend; static assets via CDN |

### V5 — Validation, Sanitization, and Encoding

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V5.1.1 | HTTP parameter pollution protection | ✅ | ✅ | 🔲 | ASP.NET Core model binding |
| V5.1.2 | Framework protects against mass assignment | ✅ | ✅ | 🔲 | Explicit DTOs, no entity binding |
| V5.1.3 | All input validated (server-side positive validation) | ✅ | ✅ | 🔲 | FluentValidation on all endpoints |
| V5.1.5 | URL redirects only to allowed destinations | ✅ | ✅ | 🔲 | Redirect URL allowlist |
| V5.2.1 | All untrusted HTML input sanitized | ✅ | ✅ | 🔲 | React auto-escaping; no dangerouslySetInnerHTML |
| V5.2.2 | Unstructured data sanitized | ✅ | ✅ | 🔲 | Input sanitization middleware |
| V5.3.1 | Output encoding relevant for the interpreter | ✅ | ✅ | 🔲 | Context-aware encoding |
| V5.3.3 | Context-aware output escaping (HTML, JS, CSS, URL) | ✅ | ✅ | 🔲 | React JSX + System.Text.Encodings.Web |
| V5.3.4 | Database queries use parameterized queries | ✅ | ✅ | 🔲 | EF Core + Dapper parameterized |
| V5.3.7 | Application protects against OS command injection | ✅ | ✅ | 🔲 | No shell commands from user input |
| V5.3.8 | Application protects against SSRF | ✅ | ✅ | 🔲 | URL allowlisting |
| V5.5.2 | JSON deserialization uses safe defaults | ✅ | ✅ | 🔲 | System.Text.Json (no TypeNameHandling) |

### V6 — Stored Cryptography

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V6.1.1 | Regulated data encrypted at rest | | ✅ | 🔲 | Volume encryption + Vault Transit |
| V6.1.2 | PII encrypted or hashed | | ✅ | 🔲 | Vault Transit for PII fields |
| V6.2.1 | Approved cryptographic modules used | ✅ | ✅ | 🔲 | System.Security.Cryptography |
| V6.2.2 | Industry-proven algorithms used | ✅ | ✅ | ✅ | [CRYPTOGRAPHY-POLICY.md](./CRYPTOGRAPHY-POLICY.md) §3 |
| V6.2.5 | Key rotation possible without downtime | | ✅ | 🔲 | Vault versioned keys |
| V6.2.6 | Random values generated by CSPRNG | ✅ | ✅ | 🔲 | RandomNumberGenerator |
| V6.3.1 | All random values generated server-side | | ✅ | 🔲 | No client-side crypto |
| V6.4.1 | Key management solution in use | | ✅ | 🔲 | HashiCorp Vault |
| V6.4.2 | Key material not exposed to application | | ✅ | 🔲 | Vault Transit (encrypt/decrypt API) |

### V7 — Error Handling and Logging

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V7.1.1 | Generic error message for users | ✅ | ✅ | 🔲 | RFC 7807 ProblemDetails without stack traces |
| V7.1.2 | Application does not expose stack traces in production | ✅ | ✅ | 🔲 | `ASPNETCORE_ENVIRONMENT=Production` |
| V7.1.3 | Security logging covers successful and unsuccessful events | | ✅ | 🔲 | Serilog + Loki for auth events |
| V7.2.1 | Each log event includes enough info for investigation | | ✅ | 🔲 | Structured: timestamp, level, correlationId, userId |
| V7.2.2 | All authentication decisions logged | | ✅ | 🔲 | Keycloak events + API authorization logs |
| V7.3.1 | Sensitive data not logged (PII, tokens, secrets) | | ✅ | 🔲 | Serilog destructuring policy, masking |
| V7.3.3 | Security logs protected from injection | | ✅ | 🔲 | Structured logging (no string interpolation) |
| V7.4.1 | Generic error message shown when unexpected error occurs | ✅ | ✅ | 🔲 | Global exception handler middleware |

### V8 — Data Protection

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V8.1.1 | Application protects sensitive data in HTTP responses | ✅ | ✅ | 🔲 | Cache-Control, no caching sensitive data |
| V8.1.2 | No sensitive data stored in browser storage | ✅ | ✅ | 🔲 | No localStorage for tokens |
| V8.2.1 | Application sets sufficient anti-caching headers | ✅ | ✅ | 🔲 | `Cache-Control: no-store` for sensitive responses |
| V8.2.2 | Data in authenticated responses not cached by shared caches | ✅ | ✅ | 🔲 | `Cache-Control: private` |
| V8.3.1 | Sensitive data sent via HTTP message body (not URL) | ✅ | ✅ | 🔲 | POST/PUT for sensitive data |
| V8.3.4 | All sensitive data server-side processes are associated with user | | ✅ | 🔲 | Correlation ID + user context |

### V9 — Communication

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V9.1.1 | TLS used for all connections | ✅ | ✅ | 🔲 | TLS 1.3 everywhere |
| V9.1.2 | Trusted TLS configuration (strong cipher suites) | ✅ | ✅ | 🔲 | [CRYPTOGRAPHY-POLICY.md](./CRYPTOGRAPHY-POLICY.md) §4.1 |
| V9.1.3 | Only latest recommended TLS versions | ✅ | ✅ | 🔲 | TLS 1.3 preferred, 1.2 minimum |
| V9.2.1 | Connections to external systems use TLS | | ✅ | 🔲 | All outbound via HTTPS |

### V10 — Malicious Code

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V10.1.1 | Source code does not contain time bombs or backdoors | | ✅ | 🔲 | Code review |
| V10.2.1 | Application does not request unnecessary permissions | ✅ | ✅ | 🔲 | K8s RBAC, GitHub Actions permissions |
| V10.3.1 | Application has auto-update mechanism controlled securely | ✅ | ✅ | 🔲 | ArgoCD GitOps (no auto-download) |

### V12 — Files and Resources

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V12.1.1 | Application does not accept large files (configurable limit) | ✅ | ✅ | 🔲 | MaxRequestBodySize configuration |
| V12.1.2 | File type validated against allowlist (not content type header only) | ✅ | ✅ | 🔲 | Magic bytes validation + extension check |
| V12.3.1 | File name metadata not used directly (path traversal prevention) | ✅ | ✅ | 🔲 | UUID rename on upload |
| V12.3.6 | Application does not include user-submitted content as SSI | ✅ | ✅ | 🔲 | No SSI used |

### V13 — API and Web Service

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V13.1.1 | All API responses use proper content type | ✅ | ✅ | 🔲 | `Content-Type: application/json` |
| V13.1.3 | API URLs do not expose sensitive info | ✅ | ✅ | 🔲 | No tokens/keys in URLs |
| V13.2.1 | HTTP methods restricted appropriately | ✅ | ✅ | 🔲 | Explicit route mapping |
| V13.2.2 | JSON schema validation on input | ✅ | ✅ | 🔲 | FluentValidation + Zod |
| V13.2.5 | REST API checks content type (application/json) | ✅ | ✅ | 🔲 | Content negotiation middleware |
| V13.4.1 | GraphQL or data-layer query limits | | ✅ | ➖ | No GraphQL used |

### V14 — Configuration

| ID | Requirement | L1 | L2 | Status | Implementation |
|----|-------------|----|----|--------|---------------|
| V14.1.1 | Application build is reproducible | | ✅ | 🔲 | Lock files, pinned deps, multi-stage Docker |
| V14.2.1 | All components are up-to-date | ✅ | ✅ | 🔲 | Dependabot weekly scans |
| V14.2.2 | Unnecessary features disabled | ✅ | ✅ | 🔲 | Minimal Alpine images |
| V14.2.3 | API does not expose debug endpoints in production | ✅ | ✅ | 🔲 | `ASPNETCORE_ENVIRONMENT=Production` |
| V14.3.2 | HTTP methods restricted to expected list | ✅ | ✅ | 🔲 | Explicit route configuration |
| V14.4.1 | HSTS header included | ✅ | ✅ | 🔲 | `Strict-Transport-Security: max-age=31536000; includeSubDomains` |
| V14.4.2 | CSP header included | ✅ | ✅ | 🔲 | Content-Security-Policy: strict |
| V14.4.3 | X-Content-Type-Options header | ✅ | ✅ | 🔲 | `nosniff` |
| V14.4.4 | Referrer-Policy header | ✅ | ✅ | 🔲 | `strict-origin-when-cross-origin` |
| V14.4.5 | X-Frame-Options or CSP frame-ancestors | ✅ | ✅ | 🔲 | `frame-ancestors 'none'` |
| V14.5.1 | Server sends X-Content-Type-Options: nosniff | ✅ | ✅ | 🔲 | Security headers middleware |
| V14.5.3 | CSP policies restrict inline script execution | ✅ | ✅ | 🔲 | `script-src 'self'` with nonce |

## 4. Compliance Summary

| Section | Total L2 Requirements | Addressed | Implemented | Score |
|---------|----------------------|-----------|-------------|-------|
| V1 Architecture | 18 | 18 | 8 | 44% |
| V2 Authentication | 14 | 14 | 0 | 0% |
| V3 Session Management | 12 | 12 | 0 | 0% |
| V4 Access Control | 8 | 8 | 0 | 0% |
| V5 Validation | 12 | 12 | 0 | 0% |
| V6 Cryptography | 9 | 9 | 1 | 11% |
| V7 Error Handling | 8 | 8 | 0 | 0% |
| V8 Data Protection | 6 | 6 | 0 | 0% |
| V9 Communication | 4 | 4 | 0 | 0% |
| V10 Malicious Code | 3 | 3 | 0 | 0% |
| V12 Files | 4 | 4 | 0 | 0% |
| V13 API | 5 | 5 | 0 | 0% |
| V14 Configuration | 12 | 12 | 0 | 0% |
| **Total** | **115** | **115** | **9** | **8%** |

> **Note**: Current score reflects documentation-first phase. Implementation score will increase as backend and frontend code is developed.

## 5. Verification Schedule

| Phase | Modules Verified | Target ASVS Sections |
|-------|-----------------|---------------------|
| Phase 2 — Backend | Identity module | V2, V3, V4, V7 |
| Phase 2 — Backend | Catalog module | V4, V5, V13 |
| Phase 3 — Frontend | Next.js app | V3, V5, V8, V14 |
| Phase 3 — CI/CD | Pipelines | V10, V14 |
| Phase 4 — Security | Cross-cutting | V1, V6, V9, V12 |
| Pre-release | Full application | All V1–V14 |

## 6. References

- [OWASP ASVS v4.0.3](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Top 10 (2021)](https://owasp.org/www-project-top-ten/)
- [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md)
- [RISK-ASSESSMENT.md](./RISK-ASSESSMENT.md)
- [ACCESS-CONTROL-POLICY.md](./ACCESS-CONTROL-POLICY.md)
- [SECURE-DEVELOPMENT-POLICY.md](./SECURE-DEVELOPMENT-POLICY.md)
- [CRYPTOGRAPHY-POLICY.md](./CRYPTOGRAPHY-POLICY.md)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
