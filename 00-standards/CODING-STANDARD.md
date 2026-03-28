# Coding Standard

| Field         | Value                                                    |
|---------------|----------------------------------------------------------|
| **Version**   | 1.0.0                                                    |
| **Status**    | Draft                                                    |
| **Author**    | Vox                                                      |
| **Reviewer**  | Vox                                                      |
| **Created**   | 2026-03-27                                               |
| **Updated**   | 2026-03-27                                               |
| **Standard**  | ISO/IEC 25010, .NET Coding Conventions, ESLint Recommended |

---

## 1. Purpose

This document defines the coding standards and conventions for the **Utopia** project across all codebases: backend (.NET), frontend (TypeScript/Next.js), Infrastructure as Code (Terraform/Ansible), and CI/CD pipelines (GitHub Actions).

## 2. Scope

This standard applies to:

- All source code in the `backend/`, `frontend/`, `infrastructure/`, and `devops/` directories
- Configuration files (YAML, JSON, TOML, HCL)
- Scripts (PowerShell, Bash)
- Database migrations and seed data

## 3. General Principles

All code in Utopia MUST adhere to these principles regardless of language:

1. **Readability over cleverness** — Code is read more often than written
2. **Explicit over implicit** — Avoid magic; make behavior obvious
3. **Fail fast** — Validate at system boundaries, fail immediately with clear errors
4. **DRY at the right level** — Avoid premature abstraction; duplication is better than wrong abstraction
5. **Single Responsibility** — Each unit (function, class, module) does one thing well
6. **Secure by default** — Follow OWASP guidelines (see [SECURITY-STANDARD.md](./SECURITY-STANDARD.md))

## 4. Backend — .NET (C#)

### 4.1. Language Version & Framework

| Item | Value |
|------|-------|
| .NET Version | .NET 8 LTS |
| C# Version | C# 12 |
| Nullable Reference Types | MUST be enabled (`<Nullable>enable</Nullable>`) |
| Implicit Usings | MUST be enabled |
| Code Analysis | MUST enable `<EnableNETAnalyzers>true</EnableNETAnalyzers>` |
| Treat Warnings as Errors | MUST in CI (`<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`) |

### 4.2. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Namespace | PascalCase, match folder structure | `Utopia.Modules.Identity` |
| Class / Record / Struct | PascalCase, noun | `UserProfile`, `OrderCreatedEvent` |
| Interface | PascalCase with `I` prefix | `IUserRepository` |
| Method | PascalCase, verb phrase | `GetUserById()`, `CreateOrderAsync()` |
| Async method | PascalCase with `Async` suffix | `SendEmailAsync()` |
| Property | PascalCase | `FirstName`, `IsActive` |
| Private field | `_camelCase` with underscore prefix | `_userRepository` |
| Local variable | camelCase | `userCount`, `isValid` |
| Constant | PascalCase | `MaxRetryCount`, `DefaultPageSize` |
| Enum | PascalCase (singular) | `OrderStatus { Pending, Confirmed }` |
| Generic type parameter | `T` prefix | `TEntity`, `TResponse` |

### 4.3. Project Structure (Modulith)

Each module MUST follow this structure:

```
Modules/
└── Identity/
    ├── Utopia.Modules.Identity/            # Main module project
    │   ├── Domain/                          # Entities, Value Objects, Domain Events
    │   │   ├── Entities/
    │   │   ├── ValueObjects/
    │   │   ├── Events/
    │   │   └── Exceptions/
    │   ├── Application/                     # Use cases, DTOs, Interfaces
    │   │   ├── Commands/
    │   │   ├── Queries/
    │   │   ├── DTOs/
    │   │   ├── Interfaces/
    │   │   └── Validators/
    │   ├── Infrastructure/                  # EF Core, external services
    │   │   ├── Persistence/
    │   │   ├── Services/
    │   │   └── Configuration/
    │   ├── Endpoints/                       # API endpoints (Minimal API / FastEndpoints)
    │   └── ModuleServiceExtensions.cs       # DI registration
    └── Utopia.Modules.Identity.Tests/       # Unit + Integration tests
        ├── Unit/
        └── Integration/
```

### 4.4. Code Rules

#### 4.4.1. Classes & Methods

- Classes MUST NOT exceed **300 lines** (excluding using statements and namespace)
- Methods MUST NOT exceed **30 lines**
- Methods MUST NOT have more than **5 parameters** — use a request object instead
- Constructor injection MUST be used for dependencies
- Primary constructors SHOULD be used for simple DI

```csharp
// GOOD — Primary constructor
public class UserService(IUserRepository userRepository, ILogger<UserService> logger)
{
    public async Task<UserDto> GetByIdAsync(Guid id, CancellationToken ct)
    {
        var user = await userRepository.GetByIdAsync(id, ct)
            ?? throw new NotFoundException(nameof(User), id);

        return user.ToDto();
    }
}

// BAD — Too many parameters
public void CreateUser(string name, string email, string phone, 
    string address, string city, string country, int age) { }

// GOOD — Use request object
public void CreateUser(CreateUserRequest request) { }
```

#### 4.4.2. Error Handling

- MUST NOT use exceptions for flow control
- MUST use domain-specific exceptions inheriting from a base `DomainException`
- MUST use global exception handling middleware
- MUST NOT catch `Exception` without re-throwing or logging
- SHOULD use the Result pattern for expected failures

```csharp
// Domain exception hierarchy
public abstract class DomainException(string message) : Exception(message);
public class NotFoundException(string entity, object key) 
    : DomainException($"{entity} with key '{key}' was not found.");
public class BusinessRuleException(string message) : DomainException(message);
```

#### 4.4.3. Async/Await

- All I/O operations MUST be async
- Async methods MUST accept `CancellationToken` as the last parameter
- MUST NOT use `.Result` or `.Wait()` — always `await`
- MUST use `ConfigureAwait(false)` in library code only

#### 4.4.4. LINQ

- Prefer method syntax over query syntax
- MUST NOT use `ToList()` or `ToArray()` unnecessarily — use `IEnumerable<T>` when possible
- Complex LINQ MUST be split across multiple lines

```csharp
// GOOD
var activeUsers = users
    .Where(u => u.IsActive)
    .OrderBy(u => u.LastName)
    .Select(u => u.ToDto());

// BAD — Single line, hard to read
var activeUsers = users.Where(u => u.IsActive).OrderBy(u => u.LastName).Select(u => u.ToDto()).ToList();
```

### 4.5. Entity Framework Core

- MUST use **Fluent API** configuration, NOT data annotations
- MUST configure entities in separate `IEntityTypeConfiguration<T>` classes
- Each module MUST have its own `DbContext` with schema isolation
- Migrations MUST be named descriptively: `AddUserEmailIndex`, NOT `Migration001`
- MUST NOT use lazy loading

```csharp
// GOOD — Separate configuration
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("users", "identity");
        builder.HasKey(u => u.Id);
        builder.Property(u => u.Email).HasMaxLength(256).IsRequired();
        builder.HasIndex(u => u.Email).IsUnique();
    }
}
```

### 4.6. Testing (.NET)

| Type | Framework | Convention |
|------|-----------|-----------|
| Unit Test | xUnit + FluentAssertions | `MethodName_StateUnderTest_ExpectedBehavior` |
| Integration Test | xUnit + Testcontainers | `Should_ExpectedBehavior_When_Condition` |
| Fake Data | Bogus | Use `Faker<T>` for test data |
| Mocking | NSubstitute | Prefer over Moq for simplicity |

```csharp
// Unit test naming
[Fact]
public async Task GetByIdAsync_UserExists_ReturnsUserDto()
{
    // Arrange
    var user = new UserFaker().Generate();
    _userRepository.GetByIdAsync(user.Id, Arg.Any<CancellationToken>())
        .Returns(user);

    // Act
    var result = await _sut.GetByIdAsync(user.Id, CancellationToken.None);

    // Assert
    result.Should().NotBeNull();
    result.Email.Should().Be(user.Email);
}
```

## 5. Frontend — TypeScript / Next.js

### 5.1. Language & Framework

| Item | Value |
|------|-------|
| Node.js | LTS (v20+) |
| Package Manager | pnpm |
| TypeScript | Strict mode MUST be enabled |
| Framework | Next.js 14+ (App Router) |
| Linting | ESLint (flat config) + Prettier |

### 5.2. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| File (component) | kebab-case | `user-profile.tsx` |
| File (utility) | kebab-case | `format-date.ts` |
| Component | PascalCase | `UserProfile` |
| Hook | camelCase with `use` prefix | `useAuth`, `useUserQuery` |
| Function | camelCase | `formatCurrency()` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Type / Interface | PascalCase | `UserResponse`, `AuthState` |
| Enum | PascalCase | `OrderStatus` |
| CSS class (Tailwind) | kebab-case utility classes | `flex items-center gap-2` |

### 5.3. Project Structure

```
frontend/src/
├── app/                          # Next.js App Router
│   ├── (auth)/                     # Auth route group
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/                # Dashboard route group
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── layout.tsx                  # Root layout
│   ├── page.tsx                    # Home page
│   └── not-found.tsx
├── components/                   # Shared components
│   ├── ui/                         # shadcn/ui components
│   ├── forms/                      # Form components
│   └── layouts/                    # Layout components
├── hooks/                        # Custom hooks
├── lib/                          # Utilities, API client, configs
│   ├── api/                        # API client + typed fetchers
│   ├── utils/                      # Helper functions
│   └── validators/                 # Zod schemas
├── stores/                       # Zustand stores
├── types/                        # Shared TypeScript types
└── styles/                       # Global styles
```

### 5.4. Code Rules

#### 5.4.1. Components

- MUST use **function components** with arrow syntax
- Props MUST be typed with explicit interface (NOT inline)
- MUST NOT use `any` type — use `unknown` and narrow with type guards
- Components MUST NOT exceed **200 lines**
- SHOULD extract logic into custom hooks when component exceeds **100 lines**

```typescript
// GOOD
interface UserCardProps {
  user: User;
  onEdit: (id: string) => void;
}

export const UserCard = ({ user, onEdit }: UserCardProps) => {
  return (
    <div className="flex items-center gap-4 rounded-lg border p-4">
      <span>{user.name}</span>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
};
```

#### 5.4.2. State Management

- **Server state**: MUST use TanStack Query — MUST NOT use Zustand/Redux for API data
- **Client state**: Use Zustand for global UI state
- **Form state**: Use React Hook Form
- **URL state**: Use Next.js `searchParams`

#### 5.4.3. API Calls

- All API calls MUST go through a typed API client in `lib/api/`
- Response types MUST match backend DTOs
- Error handling MUST be centralized in the API client

### 5.5. Testing (Frontend)

| Type | Framework | Convention |
|------|-----------|-----------|
| Unit Test | Vitest | `*.test.ts` alongside source |
| Component Test | Vitest + Testing Library | `*.test.tsx` alongside source |
| E2E Test | Playwright | `e2e/*.spec.ts` |

## 6. Infrastructure as Code — Terraform

### 6.1. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Resource names | snake_case | `resource "azurerm_resource_group" "utopia_rg"` |
| Variable names | snake_case | `var.database_name` |
| Output names | snake_case | `output "cluster_endpoint"` |
| File names | kebab-case | `main.tf`, `variables.tf`, `outputs.tf` |
| Module names | kebab-case | `modules/kubernetes-cluster/` |

### 6.2. File Structure

Each Terraform root module MUST contain:

```
infrastructure/terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   ├── providers.tf
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
└── modules/
    ├── networking/
    ├── kubernetes/
    ├── database/
    └── monitoring/
```

### 6.3. Rules

- MUST use **remote state** with state locking
- MUST NOT hardcode secrets — use variables with `sensitive = true`
- All resources MUST have a `tags` block with: `Project`, `Environment`, `ManagedBy`
- MUST pin provider versions with `~>` constraint
- MUST run `terraform fmt` and `terraform validate` in CI

```hcl
# GOOD — Resource with required tags
resource "azurerm_resource_group" "utopia" {
  name     = "rg-utopia-${var.environment}"
  location = var.location

  tags = {
    Project     = "Utopia"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

## 7. CI/CD — GitHub Actions

### 7.1. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Workflow file | kebab-case | `backend-ci.yml`, `deploy-prod.yml` |
| Job name | kebab-case | `build-and-test` |
| Step name | Sentence case, descriptive | `Run unit tests` |
| Secret name | SCREAMING_SNAKE_CASE | `SONAR_TOKEN` |
| Environment variable | SCREAMING_SNAKE_CASE | `DOCKER_REGISTRY` |

### 7.2. Rules

- MUST pin action versions with full SHA, NOT tags: `uses: actions/checkout@<sha>`
- MUST use **least privilege** for `GITHUB_TOKEN` permissions
- MUST NOT store secrets in workflow files
- MUST use reusable workflows for shared logic
- MUST use job-level `timeout-minutes`

```yaml
# GOOD — Pinned action + explicit permissions
name: Backend CI
on:
  pull_request:
    paths: ['backend/**']

permissions:
  contents: read

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
```

## 8. Docker

### 8.1. Rules

- MUST use **multi-stage builds** to minimize image size
- MUST NOT run as root — use `USER` directive
- MUST use specific base image tags, NOT `latest`
- MUST use `.dockerignore` to exclude unnecessary files
- MUST include `HEALTHCHECK` instruction
- Image labels MUST follow OCI annotation spec

```dockerfile
# GOOD — Multi-stage, non-root, health check
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime
RUN adduser -D appuser
USER appuser
WORKDIR /app
COPY --from=build /app .
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:8080/health || exit 1
ENTRYPOINT ["dotnet", "Utopia.Api.dll"]
```

## 9. Git Conventions

### 9.1. Commit Messages

MUST follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `style` | Formatting, no code change |
| `refactor` | Code refactoring |
| `test` | Adding/updating tests |
| `chore` | Build, CI, tooling changes |
| `security` | Security fix or improvement |
| `infra` | Infrastructure changes |

Scope examples: `identity`, `catalog`, `frontend`, `terraform`, `ci`

### 9.2. Branch Naming

```
<type>/<ticket-or-short-description>

feat/user-registration
fix/login-token-expiry
infra/k8s-monitoring-stack
docs/security-standard
```

## 10. Code Quality Enforcement

| Tool | Purpose | Enforcement |
|------|---------|-------------|
| `.editorconfig` | Consistent formatting across IDEs | MUST be in repo root |
| Roslyn Analyzers | .NET code analysis | MUST in CI (warnings as errors) |
| ESLint + Prettier | TypeScript linting + formatting | MUST in CI (pre-commit hook) |
| `terraform fmt` | HCL formatting | MUST in CI |
| `terraform validate` | HCL validation | MUST in CI |
| `hadolint` | Dockerfile linting | MUST in CI |
| `actionlint` | GitHub Actions linting | MUST in CI |
| `commitlint` | Commit message validation | MUST via Git hook |

## 11. References

- [.NET Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [TypeScript Style Guide (Google)](https://google.github.io/styleguide/tsguide.html)
- [Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [ISO/IEC 25010 — Software Quality Model](https://www.iso.org/standard/35733.html)
- [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
