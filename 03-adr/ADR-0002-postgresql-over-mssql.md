# ADR-0002: PostgreSQL Over MSSQL

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

Utopia requires a relational database that supports multiple schemas (one per module), is production-proven, has strong DevOps tooling support, and can run efficiently in containers for both local development and Kubernetes deployment.

## Decision Drivers

- **Open source** — no licensing costs, community-driven
- **Container support** — official lightweight Docker images
- **Schema isolation** — native support for multiple schemas per database
- **IaC support** — Terraform providers for major cloud managed services (AWS RDS, Azure Database, GCP Cloud SQL)
- **Performance** — proven at scale with strong query optimizer
- **Ecosystem** — tooling, extensions, community resources
- **EF Core support** — first-class Entity Framework Core provider

## Considered Options

1. **PostgreSQL 16** — open-source relational database
2. **SQL Server (MSSQL)** — Microsoft's relational database
3. **MySQL 8** — open-source relational database

## Decision

We will use **PostgreSQL 16** as the primary database for all modules.

Each module will have its own schema within a single PostgreSQL database (e.g., `identity.*`, `catalog.*`), managed by separate `DbContext` instances.

## Comparison

| Criterion | PostgreSQL | MSSQL | MySQL |
|-----------|-----------|-------|-------|
| Open source | ✅ Fully | ❌ Licensed (Express limited) | ✅ Dual-licensed |
| Container image size | ✅ ~80 MB (Alpine) | ❌ ~1.5 GB | ✅ ~150 MB |
| Schema per module | ✅ Native schemas | ✅ Native schemas | ❌ Schema = Database |
| JSON support | ✅ jsonb (indexed) | ⚠️ JSON (limited) | ⚠️ JSON (basic) |
| Terraform providers | ✅ All clouds | ✅ Azure only (managed) | ✅ All clouds |
| EF Core provider | ✅ Npgsql (mature) | ✅ Official | ✅ Pomelo (community) |
| K8s operators | ✅ CloudNativePG, Zalando | ⚠️ Limited | ✅ Oracle operator |
| Extensions | ✅ PostGIS, pg_trgm, etc. | ❌ Limited | ❌ Limited |
| Developer tooling | ✅ pgAdmin, psql, DBeaver | ✅ SSMS, Azure Data Studio | ✅ MySQL Workbench |
| Cloud managed options | ✅ RDS, Cloud SQL, Azure | ✅ Azure SQL | ✅ RDS, Cloud SQL |

## Consequences

### Positive

- Zero licensing cost, even in production
- Lightweight container (~80 MB Alpine) — fast CI builds and local dev
- Native schema support enables clean module isolation
- Rich extension ecosystem (PostGIS for future geo features, pg_trgm for text search)
- Excellent Kubernetes operators for production HA (CloudNativePG)
- Strong JSON support via `jsonb` for semi-structured data needs

### Negative

- Team members more familiar with MSSQL may need onboarding — mitigation: comprehensive documentation and EF Core abstracts most differences
- Some .NET-specific tooling assumes MSSQL — mitigation: Npgsql provider is mature and well-documented
- Windows native development requires container (no native PG install) — mitigation: Docker Compose handles this

### Risks

- EF Core Npgsql provider has subtle differences from MSSQL provider — likelihood: Low, impact: Low — mitigation: integration tests with Testcontainers validate all queries

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/16/)
- [Npgsql EF Core Provider](https://www.npgsql.org/efcore/)
- [CloudNativePG Operator](https://cloudnative-pg.io/)
- [ADR-0001](./ADR-0001-modulith-architecture.md) — Schema-per-module strategy

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
