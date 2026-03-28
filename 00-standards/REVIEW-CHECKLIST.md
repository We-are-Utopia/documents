# Review Checklist

| Field         | Value                                                    |
|---------------|----------------------------------------------------------|
| **Version**   | 1.0.0                                                    |
| **Status**    | Draft                                                    |
| **Author**    | Vox                                                      |
| **Reviewer**  | Vox                                                      |
| **Created**   | 2026-03-27                                               |
| **Updated**   | 2026-03-27                                               |
| **Standard**  | ISO 27001:2022, OWASP ASVS, ISO/IEC 25010               |

---

## 1. Purpose

This document provides standardized review checklists for the **Utopia** project. These checklists MUST be used during Pull Request reviews to ensure consistency, quality, and security across all code and documentation changes.

## 2. Scope

This checklist applies to all Pull Request reviews covering:

- Code changes (backend, frontend, IaC, CI/CD)
- Documentation changes
- Configuration changes
- Database migrations
- Dependency updates

## 3. How To Use

1. Reviewers MUST copy the relevant checklist section(s) into the PR review comment
2. Check each item as reviewed — mark `[x]` for pass, leave `[ ]` for not applicable
3. Any unchecked **MUST** item blocks the PR merge
4. **SHOULD** items are recommended but do NOT block merge
5. Add comments for any concerns or suggestions

---

## 4. Documentation Review Checklist

Use when reviewing changes in `documents/` or any README files.

Reference: [DOCUMENTATION-STANDARD.md](./DOCUMENTATION-STANDARD.md)

### 4.1. Structure & Metadata

- [ ] Document has required metadata header (Version, Status, Author, Reviewer, Created, Updated)
- [ ] Status is set correctly (`Draft` for new docs)
- [ ] Version follows SemVer (see [VERSIONING-STANDARD.md](./VERSIONING-STANDARD.md))
- [ ] Required sections are present: Purpose, Scope, Body, References, Changelog
- [ ] Changelog entry is added for the change

### 4.2. Content Quality

- [ ] Written in English
- [ ] Uses active voice and present tense
- [ ] RFC 2119 keywords (MUST, SHOULD, MAY) are in UPPERCASE when used as requirements
- [ ] Acronyms are defined on first use
- [ ] No ambiguous or vague statements
- [ ] Content is accurate and technically correct

### 4.3. Formatting

- [ ] Headings use ATX-style (`#`), no skipped levels
- [ ] Code blocks use fenced syntax with language identifier
- [ ] Tables have header rows
- [ ] Links use relative paths for internal documents
- [ ] Diagrams use Mermaid syntax with titles

### 4.4. Cross-References

- [ ] Related documents are linked
- [ ] ADR references are included where design decisions are explained
- [ ] No broken links

---

## 5. Code Review Checklist — General

Use for ALL code changes regardless of language.

### 5.1. Correctness

- [ ] Code does what the PR description says it does
- [ ] Edge cases are handled
- [ ] No off-by-one errors
- [ ] No race conditions or concurrency issues
- [ ] No logical errors in conditions

### 5.2. Readability

- [ ] Code is self-documenting — variable and method names are descriptive
- [ ] No unnecessary comments (code explains itself)
- [ ] Complex logic has explanatory comments
- [ ] Consistent formatting matches project standards
- [ ] No dead code or commented-out code

### 5.3. Design

- [ ] Single Responsibility — each unit does one thing
- [ ] No premature abstraction or over-engineering
- [ ] Changes are minimal — no unrelated modifications
- [ ] No unnecessary dependencies added
- [ ] Backward compatibility is maintained (or breaking change is documented)

### 5.4. Security (OWASP)

- [ ] **No hardcoded secrets** — no passwords, API keys, tokens in code
- [ ] **Input validation** — all external input is validated at system boundary
- [ ] **Authorization** — endpoints have proper authorization checks
- [ ] **SQL injection** — parameterized queries only (no string concatenation)
- [ ] **XSS** — no unsafe HTML rendering, output encoding in place
- [ ] **Sensitive data** — not logged, not in error messages, not in URLs
- [ ] **Error handling** — errors do NOT leak internal details to clients
- [ ] **Dependencies** — from trusted sources, no known critical vulnerabilities
- [ ] **CORS** — no wildcard (`*`) origins
- [ ] **Rate limiting** — considered for public/authenticated endpoints

### 5.5. Testing

- [ ] New code has accompanying tests
- [ ] Tests are meaningful (not just for coverage)
- [ ] Edge cases are tested
- [ ] Test naming follows convention: `Method_Condition_Expected`
- [ ] No flaky tests (no `Thread.Sleep`, no time-dependent assertions)
- [ ] Integration tests use Testcontainers (not mocked infrastructure)

### 5.6. Performance

- [ ] No N+1 query problems
- [ ] No unnecessary allocations in hot paths
- [ ] Database queries are efficient (proper indexes considered)
- [ ] No blocking calls in async context
- [ ] Response payloads are appropriately sized

---

## 6. Code Review Checklist — Backend (.NET)

Use in addition to Section 5 for backend changes.

Reference: [CODING-STANDARD.md](./CODING-STANDARD.md)

### 6.1. .NET Specific

- [ ] Naming follows conventions (PascalCase for public, `_camelCase` for private fields)
- [ ] Async methods have `Async` suffix and accept `CancellationToken`
- [ ] No `.Result` or `.Wait()` calls on async methods
- [ ] Nullable reference types are handled (no `!` operator without justification)
- [ ] Constructor injection used for dependencies
- [ ] No service locator pattern (`IServiceProvider.GetService` in business logic)

### 6.2. Module Structure

- [ ] Changes respect module boundaries — no cross-module direct references
- [ ] Inter-module communication uses events/contracts, not direct calls
- [ ] Domain entities are not exposed outside the module
- [ ] DTOs are used for API responses

### 6.3. Database

- [ ] EF Core configuration uses Fluent API (not data annotations)
- [ ] Migrations are named descriptively
- [ ] Schema changes follow expand-contract pattern for breaking changes
- [ ] Indexes are added for frequently queried columns
- [ ] No lazy loading
- [ ] Raw SQL uses parameterized queries

### 6.4. API Endpoints

- [ ] Endpoints use appropriate HTTP methods (GET, POST, PUT, DELETE)
- [ ] Response status codes are correct (201 for create, 204 for delete, etc.)
- [ ] Request validation via FluentValidation
- [ ] Endpoint authorization is configured
- [ ] API versioning is consistent

---

## 7. Code Review Checklist — Frontend (TypeScript / Next.js)

Use in addition to Section 5 for frontend changes.

### 7.1. TypeScript Specific

- [ ] No `any` type usage — use `unknown` with type guards if needed
- [ ] Interfaces/types are properly defined (not inline)
- [ ] No type assertions (`as`) without justification
- [ ] Strict mode compliance (no `@ts-ignore` without comment)

### 7.2. Components

- [ ] Function components with arrow syntax
- [ ] Props are typed with explicit interface
- [ ] Component does NOT exceed 200 lines
- [ ] Complex logic extracted to custom hooks
- [ ] No inline styles — use Tailwind classes

### 7.3. State Management

- [ ] Server state uses TanStack Query (not local state for API data)
- [ ] Client state uses Zustand appropriately
- [ ] No prop drilling beyond 2 levels — use composition or context
- [ ] State updates are immutable

### 7.4. Performance

- [ ] Large lists use virtualization
- [ ] Images use Next.js `<Image>` component
- [ ] No unnecessary re-renders (check dependency arrays)
- [ ] Dynamic imports for heavy components
- [ ] No memory leaks in effects (cleanup functions present)

### 7.5. Accessibility

- [ ] Interactive elements are keyboard accessible
- [ ] Images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator of state
- [ ] ARIA attributes used correctly

---

## 8. Code Review Checklist — Infrastructure as Code

Use for Terraform, Ansible, Kubernetes manifests, and Docker changes.

### 8.1. Terraform

- [ ] `terraform fmt` has been run
- [ ] `terraform validate` passes
- [ ] Provider versions are pinned with `~>` constraint
- [ ] No hardcoded values — variables are used
- [ ] Sensitive variables marked with `sensitive = true`
- [ ] Resources have required tags (Project, Environment, ManagedBy)
- [ ] State is configured for remote backend with locking
- [ ] Outputs are documented

### 8.2. Kubernetes / Helm

- [ ] Pods run as non-root with security context
- [ ] Resource requests and limits are set
- [ ] Network policies are in place
- [ ] Secrets are not stored in plain text (Sealed Secrets / External Secrets)
- [ ] Liveness and readiness probes configured
- [ ] Pod disruption budgets defined for production
- [ ] No `latest` image tags

### 8.3. Docker

- [ ] Multi-stage build for minimal image size
- [ ] Runs as non-root user
- [ ] Specific base image tag (not `latest`)
- [ ] `.dockerignore` excludes unnecessary files
- [ ] `HEALTHCHECK` instruction present
- [ ] No secrets in build args or layers
- [ ] `hadolint` passes

### 8.4. CI/CD (GitHub Actions)

- [ ] Action versions pinned with full SHA (not tags)
- [ ] `permissions` block with least privilege
- [ ] `timeout-minutes` set on jobs
- [ ] No secrets in workflow logs
- [ ] Reusable workflows for shared logic
- [ ] `actionlint` passes

---

## 9. Dependency Update Review Checklist

Use when reviewing Dependabot/Renovate PRs or manual dependency updates.

- [ ] Changelog/release notes reviewed for breaking changes
- [ ] No known vulnerabilities in new version
- [ ] License is compatible (MIT, Apache 2.0, BSD — see approved list)
- [ ] Tests pass with new version
- [ ] No unnecessary transitive dependencies added
- [ ] Lock file is updated and committed

---

## 10. Database Migration Review Checklist

- [ ] Migration is forward-only (no down migration for production)
- [ ] Migration is idempotent or handles existing state
- [ ] Breaking schema changes use expand-contract pattern
- [ ] Large data migrations are batched (not full table locks)
- [ ] Indexes are added/removed intentionally
- [ ] Migration name is descriptive
- [ ] Rollback strategy is documented if applicable

---

## 11. Review Severity Levels

Reviewers SHOULD categorize feedback using these prefixes:

| Prefix | Meaning | Blocks Merge? |
|--------|---------|---------------|
| `blocking:` | Must be fixed before merge | Yes |
| `suggestion:` | Improvement recommended | No |
| `nitpick:` | Minor style/formatting | No |
| `question:` | Needs clarification | Depends |
| `praise:` | Good work, worth highlighting | No |

Example:
```
blocking: This endpoint is missing authorization — any authenticated user can delete other users.

suggestion: Consider using a guard clause here to reduce nesting.

nitpick: Variable name `x` could be more descriptive — maybe `retryCount`?
```

## 12. References

- [DOCUMENTATION-STANDARD.md](./DOCUMENTATION-STANDARD.md)
- [CODING-STANDARD.md](./CODING-STANDARD.md)
- [SECURITY-STANDARD.md](./SECURITY-STANDARD.md)
- [VERSIONING-STANDARD.md](./VERSIONING-STANDARD.md)
- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [Google Engineering Practices — Code Review](https://google.github.io/eng-practices/review/)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
