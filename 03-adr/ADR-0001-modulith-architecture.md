# ADR-0001: Modulith Architecture for Backend

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

Utopia needs a backend architecture that is maintainable, testable, and demonstrates production-grade design patterns. The architecture must support multiple business domains (Identity, Catalog, etc.) while remaining simple to deploy and operate — especially important for a personal project running on a single machine.

## Decision Drivers

- **Maintainability** — clear module boundaries and separation of concerns
- **Deploy simplicity** — single deployment unit, minimal operational overhead
- **Future scalability** — ability to extract modules into microservices if needed
- **Learning value** — demonstrates patterns relevant to enterprise systems
- **DevOps friendliness** — single CI pipeline, single container image initially
- **Resource efficiency** — must run comfortably on local development machine

## Considered Options

1. **Modulith (Modular Monolith)** — single deployable with internal module boundaries
2. **Microservices** — independent services per domain, separate deployments
3. **Traditional Monolith** — single application without explicit module boundaries

## Decision

We will use a **Modulith (Modular Monolith)** architecture using .NET 8.

Each business domain (Identity, Catalog) is implemented as an independent module with its own Domain, Application, Infrastructure, and Endpoints layers. Modules communicate via domain events through MassTransit/RabbitMQ. A shared kernel contains only cross-cutting abstractions.

## Comparison

| Criterion | Modulith | Microservices | Traditional Monolith |
|-----------|----------|---------------|---------------------|
| Deploy simplicity | ✅ Single unit | ❌ Multiple services | ✅ Single unit |
| Module boundaries | ✅ Explicit | ✅ Enforced by network | ❌ Easily violated |
| Operational overhead | ✅ Low | ❌ High (service mesh, tracing) | ✅ Low |
| Future extraction | ✅ Designed for it | ✅ Already separated | ❌ Major refactor |
| Local dev resources | ✅ Low | ❌ High (many containers) | ✅ Low |
| Testing complexity | ✅ Moderate | ❌ High (contract tests) | ✅ Low |
| Learning value | ✅ High (DDD patterns) | ✅ High | ❌ Low |
| CI/CD complexity | ✅ Single pipeline | ❌ Multiple pipelines | ✅ Single pipeline |

## Consequences

### Positive

- Single deployment artifact simplifies CI/CD and Kubernetes deployment
- Clear module boundaries enforce separation of concerns at the code level
- Inter-module communication via events prepares for future microservice extraction
- Reduced infrastructure cost — one process, one database (with schema separation)
- Easier debugging and tracing compared to distributed systems

### Negative

- Module boundaries are convention-enforced, not physically enforced — mitigation: ArchUnit tests to validate module dependencies
- All modules share the same runtime — a failure in one module can affect others — mitigation: proper error isolation and health checks
- Database schema isolation requires discipline — mitigation: separate `DbContext` per module with schema prefixes

### Risks

- Module coupling creep over time — likelihood: Medium, impact: High — mitigation: automated architecture tests, code review checklist
- Performance bottleneck in shared process — likelihood: Low, impact: Medium — mitigation: module extraction path is designed in

## References

- [Modular Monolith with DDD](https://github.com/kgrzybek/modular-monolith-with-ddd)
- [.NET Modulith Template](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) — Module structure definition

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
