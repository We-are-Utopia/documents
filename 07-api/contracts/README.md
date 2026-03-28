# API Contracts — Auto-Generated OpenAPI Specs

This directory contains auto-generated OpenAPI 3.1 specification files, exported by the CI pipeline.

## Files

| File | Description |
|------|-------------|
| `utopia-api-v1.yaml` | Full OpenAPI spec for all `/api/v1/` endpoints |

## How It Works

1. The backend CI workflow ([CI-PIPELINE.md](../../06-devops/CI-PIPELINE.md)) builds the .NET API
2. Swagger/NSwag generates the OpenAPI spec at build time
3. The spec file is committed to this directory as a CI artifact

## Usage

- **Frontend developers**: Use this spec to generate TypeScript API clients
- **API review**: Compare spec diffs in pull requests to catch breaking changes
- **Documentation**: Import into Swagger UI or Redoc for interactive docs

## Rules

- Do NOT manually edit files in this directory
- Files are overwritten on every CI run
- See [API-CONTRACTS.md](../API-CONTRACTS.md) for human-readable endpoint documentation
- See [INTEGRATION-ARCHITECTURE.md](../../02-architecture/INTEGRATION-ARCHITECTURE.md) for API design guidelines
