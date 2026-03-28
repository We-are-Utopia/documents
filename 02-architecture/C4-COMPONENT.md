# C4 Component Diagram — Utopia

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | C4 Model — Level 3                   |

---

## 1. Purpose

This document describes the **Component Diagrams** (C4 Level 3) for the Utopia platform. It zooms into each container from the [container diagram](./C4-CONTAINER.md), showing the major structural building blocks (components) and their interactions.

## 2. Scope

Internal components of:

- Backend API (.NET Modulith)
- Frontend (Next.js)

## 3. Backend API — Component Overview

```mermaid
graph TB
    TRAEFIK["🌍 Traefik Ingress"]

    subgraph "Backend API Container (.NET 8 Modulith)"
        subgraph "Host Layer"
            HOST["Utopia.Api<br/><i>ASP.NET Host</i><br/><i>Program.cs, middleware pipeline</i>"]
            MW_AUTH["Auth Middleware<br/><i>JWT Bearer validation</i>"]
            MW_ERR["Global Error Handler<br/><i>Exception → ProblemDetails</i>"]
            MW_LOG["Request Logging<br/><i>Serilog + Correlation ID</i>"]
            MW_RATE["Rate Limiter<br/><i>Per-user, global limits</i>"]
            MW_SEC["Security Headers<br/><i>CSP, HSTS, X-Frame-Options</i>"]
            HEALTH["Health Checks<br/><i>/health, /ready</i>"]
            METRICS["/metrics Endpoint<br/><i>Prometheus exporter</i>"]
        end

        subgraph "Shared Kernel"
            SHARED["Utopia.Shared<br/><i>Base entities, interfaces,<br/>domain event abstractions,<br/>common value objects</i>"]
        end

        subgraph "Identity Module"
            ID_EP["Identity Endpoints<br/><i>User profile, account mgmt</i>"]
            ID_CMD["Commands<br/><i>UpdateProfile, ChangePassword</i>"]
            ID_QRY["Queries<br/><i>GetUserProfile, ListUsers</i>"]
            ID_VAL["Validators<br/><i>FluentValidation rules</i>"]
            ID_SVC["Domain Services<br/><i>Business logic</i>"]
            ID_ENT["Domain Entities<br/><i>User, Role, Permission</i>"]
            ID_EVT["Domain Events<br/><i>UserRegistered, RoleAssigned</i>"]
            ID_REPO["Repositories<br/><i>IUserRepository</i>"]
            ID_INFRA["Infrastructure<br/><i>EF Core DbContext,<br/>Keycloak integration</i>"]
        end

        subgraph "Catalog Module"
            CT_EP["Catalog Endpoints<br/><i>Products, Categories</i>"]
            CT_CMD["Commands<br/><i>CreateProduct, UpdateCategory</i>"]
            CT_QRY["Queries<br/><i>GetProduct, SearchProducts</i>"]
            CT_VAL["Validators<br/><i>FluentValidation rules</i>"]
            CT_SVC["Domain Services<br/><i>Business logic</i>"]
            CT_ENT["Domain Entities<br/><i>Product, Category, Price</i>"]
            CT_EVT["Domain Events<br/><i>ProductCreated, PriceChanged</i>"]
            CT_REPO["Repositories<br/><i>IProductRepository</i>"]
            CT_INFRA["Infrastructure<br/><i>EF Core DbContext,<br/>Redis cache</i>"]
        end
    end

    PG["🗄️ PostgreSQL"]
    REDIS["💾 Redis"]
    RMQ["📨 RabbitMQ"]
    KC["🔐 Keycloak"]
    VAULT["🔑 Vault"]

    TRAEFIK --> HOST
    HOST --> MW_SEC --> MW_LOG --> MW_RATE --> MW_AUTH --> MW_ERR

    MW_ERR --> ID_EP
    MW_ERR --> CT_EP
    HOST --> HEALTH
    HOST --> METRICS

    ID_EP --> ID_VAL --> ID_CMD
    ID_EP --> ID_QRY
    ID_CMD --> ID_SVC --> ID_ENT
    ID_SVC --> ID_REPO
    ID_SVC --> ID_EVT
    ID_REPO --> ID_INFRA
    ID_INFRA --> PG
    ID_INFRA --> KC

    CT_EP --> CT_VAL --> CT_CMD
    CT_EP --> CT_QRY
    CT_CMD --> CT_SVC --> CT_ENT
    CT_SVC --> CT_REPO
    CT_SVC --> CT_EVT
    CT_REPO --> CT_INFRA
    CT_INFRA --> PG
    CT_INFRA --> REDIS

    ID_EVT -->|"MassTransit"| RMQ
    CT_EVT -->|"MassTransit"| RMQ
    RMQ -->|"Consume"| CT_SVC
    RMQ -->|"Consume"| ID_SVC

    ID_CMD --> SHARED
    CT_CMD --> SHARED
    HOST --> VAULT
```

## 4. Backend Components Detail

### 4.1. Host Layer — `Utopia.Api`

The host project is the entry point. It composes all modules and configures the ASP.NET middleware pipeline.

| Component | Responsibility |
|-----------|---------------|
| **Program.cs / Host** | Module registration, DI composition, middleware ordering |
| **Auth Middleware** | Validates JWT tokens against Keycloak JWKS endpoint |
| **Global Error Handler** | Catches unhandled exceptions, maps to RFC 7807 ProblemDetails |
| **Request Logging** | Structured logging with correlation ID propagation via Serilog |
| **Rate Limiter** | Fixed-window and sliding-window rate limits per endpoint |
| **Security Headers** | Adds CSP, HSTS, X-Frame-Options, X-Content-Type-Options |
| **Health Checks** | `/health` (liveness), `/ready` (readiness) — checks DB, Redis, RabbitMQ |
| **Metrics Endpoint** | `/metrics` — exposes Prometheus-format metrics |

### 4.2. Shared Kernel — `Utopia.Shared`

Minimal shared code used by all modules. MUST NOT contain business logic.

| Component | Contents |
|-----------|----------|
| **Base Entities** | `Entity<TId>`, `AggregateRoot<TId>` |
| **Value Objects** | `Money`, `Email`, `DateRange` |
| **Interfaces** | `IRepository<T>`, `IDomainEvent`, `IUnitOfWork` |
| **Event Bus Abstractions** | `IEventPublisher`, `IEventHandler<T>` |
| **Common Exceptions** | `DomainException`, `NotFoundException`, `BusinessRuleException` |
| **Results** | `Result<T>`, `PagedResult<T>` |

### 4.3. Identity Module — `Utopia.Modules.Identity`

| Layer | Components | Responsibility |
|-------|-----------|---------------|
| **Endpoints** | `GET /api/v1/users/{id}`, `PUT /api/v1/users/{id}`, `GET /api/v1/users/me` | HTTP API surface — routing, authorization attributes |
| **Commands** | `UpdateProfileCommand`, `ChangePasswordCommand` | Write operations (CQRS command side) |
| **Queries** | `GetUserProfileQuery`, `ListUsersQuery` | Read operations (CQRS query side) |
| **Validators** | `UpdateProfileValidator`, `ChangePasswordValidator` | Input validation via FluentValidation |
| **Domain Services** | `UserService`, `RoleService` | Business logic orchestration |
| **Entities** | `User`, `Role`, `Permission` | Domain model — rich entities |
| **Value Objects** | `Email`, `FullName` | Immutable typed values |
| **Domain Events** | `UserRegistered`, `UserProfileUpdated`, `RoleAssigned` | Published via MassTransit to RabbitMQ |
| **Repositories** | `IUserRepository` | Data access abstraction |
| **Infrastructure** | `IdentityDbContext`, `UserRepository`, `KeycloakSyncService` | EF Core persistence (schema: `identity`), Keycloak admin API |

### 4.4. Catalog Module — `Utopia.Modules.Catalog`

| Layer | Components | Responsibility |
|-------|-----------|---------------|
| **Endpoints** | `GET /api/v1/products`, `POST /api/v1/products`, `GET /api/v1/categories` | HTTP API surface |
| **Commands** | `CreateProductCommand`, `UpdateProductCommand`, `CreateCategoryCommand` | Write operations |
| **Queries** | `GetProductQuery`, `SearchProductsQuery`, `ListCategoriesQuery` | Read operations with Redis caching |
| **Validators** | `CreateProductValidator`, `UpdateProductValidator` | Input validation |
| **Domain Services** | `ProductService`, `CategoryService`, `PricingService` | Business logic |
| **Entities** | `Product`, `Category`, `ProductVariant` | Domain model |
| **Value Objects** | `Money`, `Sku`, `ProductName` | Typed values |
| **Domain Events** | `ProductCreated`, `ProductUpdated`, `PriceChanged` | Published to event bus |
| **Repositories** | `IProductRepository`, `ICategoryRepository` | Data access |
| **Infrastructure** | `CatalogDbContext`, `ProductRepository`, `CachedProductRepository` | EF Core (schema: `catalog`), Redis cache decorator |

## 5. Frontend — Component Overview

```mermaid
graph TB
    USER["👤 End User"]

    subgraph "Frontend Container (Next.js 14+)"
        subgraph "App Router"
            LAYOUT["Root Layout<br/><i>HTML shell, providers,<br/>global styles</i>"]
            AUTH_PAGES["(auth) Route Group<br/><i>/login, /register</i>"]
            DASH_PAGES["(dashboard) Route Group<br/><i>/dashboard, /products,<br/>/profile</i>"]
            PUBLIC_PAGES["Public Pages<br/><i>/, /about</i>"]
        end

        subgraph "Core"
            AUTH_LIB["Auth Provider<br/><i>Auth.js + Keycloak OIDC</i>"]
            API_CLIENT["API Client<br/><i>Typed fetch wrapper</i>"]
            QUERY_HOOKS["Query Hooks<br/><i>TanStack Query</i>"]
            STORES["State Stores<br/><i>Zustand</i>"]
            VALIDATORS["Validators<br/><i>Zod schemas</i>"]
        end

        subgraph "Components"
            UI_LIB["UI Components<br/><i>shadcn/ui + Radix</i>"]
            FORMS["Form Components<br/><i>React Hook Form</i>"]
            LAYOUTS_C["Layout Components<br/><i>Sidebar, Header, Nav</i>"]
        end
    end

    API["⚙️ Backend API"]
    KC["🔐 Keycloak"]

    USER -->|"HTTPS"| LAYOUT
    LAYOUT --> AUTH_PAGES
    LAYOUT --> DASH_PAGES
    LAYOUT --> PUBLIC_PAGES

    AUTH_PAGES --> AUTH_LIB
    DASH_PAGES --> QUERY_HOOKS
    DASH_PAGES --> FORMS
    DASH_PAGES --> UI_LIB
    DASH_PAGES --> LAYOUTS_C

    AUTH_LIB -->|"OIDC"| KC
    QUERY_HOOKS --> API_CLIENT
    FORMS --> VALIDATORS
    API_CLIENT -->|"REST + JWT"| API
    STORES --> DASH_PAGES
```

### 5.1. Frontend Components Detail

| Component | Responsibility |
|-----------|---------------|
| **Root Layout** | HTML shell, font loading, theme provider, auth session provider, global error boundary |
| **(auth) Route Group** | Login, register, forgot password pages — server-side rendered, redirects authenticated users |
| **(dashboard) Route Group** | Protected pages requiring authentication — products, categories, user profile |
| **Auth Provider** | Auth.js (NextAuth) with Keycloak OIDC provider — manages tokens, session, refresh |
| **API Client** | Typed fetch wrapper with interceptors — attaches JWT, handles errors, retries |
| **Query Hooks** | TanStack Query hooks per API resource — `useProducts()`, `useUser()` — manages cache, loading, error states |
| **State Stores** | Zustand stores for client-only state — UI preferences, sidebar state, filters |
| **Validators** | Zod schemas matching backend DTOs — client-side validation before API call |
| **UI Components** | shadcn/ui components — Button, Dialog, Table, Form, Card, etc. |
| **Form Components** | React Hook Form wrappers with Zod resolver — `ProductForm`, `ProfileForm` |
| **Layout Components** | Sidebar navigation, header with user menu, breadcrumbs, mobile responsive nav |

## 6. Module Interaction Rules

### 6.1. Allowed Dependencies

```mermaid
graph TD
    HOST["Utopia.Api (Host)"]
    SHARED["Utopia.Shared"]
    ID["Identity Module"]
    CT["Catalog Module"]

    HOST --> ID
    HOST --> CT
    HOST --> SHARED
    ID --> SHARED
    CT --> SHARED

    ID -.->|"❌ FORBIDDEN"| CT
    CT -.->|"❌ FORBIDDEN"| ID
```

### 6.2. Rules

| Rule | Description |
|------|-------------|
| **No cross-module references** | Module A MUST NOT reference Module B's classes, interfaces, or namespaces directly |
| **Shared Kernel only** | Modules MAY only depend on `Utopia.Shared` for common abstractions |
| **Event-based communication** | Inter-module communication MUST use domain events via MassTransit/RabbitMQ |
| **No shared database tables** | Each module MUST own its own schema — no cross-schema JOINs |
| **Host composes** | Only the Host project (`Utopia.Api`) MAY reference all modules for DI registration |
| **Architecture tests** | These rules MUST be verified by automated architecture tests (ArchUnit / NetArchTest) |

## 7. Request Flow Example

### 7.1. Create Product (Authenticated)

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant T as Traefik
    participant API as Backend API
    participant VAL as Validator
    participant SVC as ProductService
    participant DB as PostgreSQL
    participant RMQ as RabbitMQ
    participant CACHE as Redis

    U->>FE: Fill product form + submit
    FE->>FE: Zod validation (client-side)
    FE->>T: POST /api/v1/products (JWT in header)
    T->>API: Forward request
    API->>API: Auth middleware (validate JWT)
    API->>API: Rate limit check
    API->>VAL: FluentValidation (server-side)
    VAL-->>API: Validation passed
    API->>SVC: CreateProductCommand
    SVC->>SVC: Business logic (check duplicates, etc.)
    SVC->>DB: INSERT INTO catalog.products
    DB-->>SVC: Product created
    SVC->>RMQ: Publish ProductCreated event
    SVC-->>API: Result<ProductDto>
    API->>CACHE: Invalidate product cache
    API-->>T: 201 Created + ProductDto
    T-->>FE: Response
    FE->>FE: TanStack Query cache update
    FE-->>U: Show success + redirect
```

## 8. References

- [C4 Model — Component Diagram](https://c4model.com/#ComponentDiagram)
- [C4-CONTEXT.md](./C4-CONTEXT.md) — Level 1
- [C4-CONTAINER.md](./C4-CONTAINER.md) — Level 2
- [DATA-ARCHITECTURE.md](./DATA-ARCHITECTURE.md) — Database schemas
- [INTEGRATION-ARCHITECTURE.md](./INTEGRATION-ARCHITECTURE.md) — Event contracts
- [ADR-0001](../03-adr/ADR-0001-modulith-architecture.md) — Modulith architecture
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) — Module structure

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
