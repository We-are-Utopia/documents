# ADR-0007: Terraform for Infrastructure as Code

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

Utopia requires an Infrastructure as Code (IaC) tool to define, provision, and manage all infrastructure declaratively. The tool must support multi-cloud providers, Kubernetes resources, and auxiliary services (Keycloak, Vault, databases). All infrastructure changes must be version-controlled, reviewed, and applied through automation.

## Decision Drivers

- **Multi-cloud support** — provider-agnostic, works with AWS, Azure, GCP, and non-cloud resources
- **Declarative** — define desired state, tool handles convergence
- **State management** — track infrastructure state for drift detection
- **Ecosystem** — rich provider library for all services in the stack
- **Community** — large adoption, extensive documentation, active development
- **CI/CD integration** — plan/apply in automated pipelines
- **Learning value** — most in-demand IaC skill in DevOps roles

## Considered Options

1. **Terraform / OpenTofu** — HashiCorp's declarative IaC tool (OpenTofu = open-source fork)
2. **Pulumi** — IaC using general-purpose programming languages
3. **AWS CDK / Azure Bicep** — cloud-specific IaC tools
4. **Ansible** — procedural configuration management (also evaluated for IaC)

## Decision

We will use **Terraform** (with **OpenTofu** as the open-source runtime) as the primary IaC tool for all infrastructure provisioning.

**Ansible** will complement Terraform for configuration management tasks that Terraform is not suited for (e.g., OS-level configuration, ad-hoc scripts).

## Comparison

| Criterion | Terraform/OpenTofu | Pulumi | AWS CDK / Bicep | Ansible |
|-----------|-------------------|--------|-----------------|---------|
| Multi-cloud | ✅ All providers | ✅ All providers | ❌ Single cloud | ✅ Via modules |
| Declarative | ✅ HCL | ⚠️ Imperative (code) | ⚠️ Mixed | ❌ Procedural |
| State management | ✅ Built-in | ✅ Built-in (Pulumi Cloud) | ✅ CloudFormation/ARM | ❌ Stateless |
| Provider ecosystem | ✅ 3000+ providers | ✅ Bridges Terraform | ❌ Cloud-specific | ⚠️ Modules |
| Keycloak provider | ✅ mrparkers/keycloak | ⚠️ Via bridge | ❌ None | ⚠️ REST API |
| Vault provider | ✅ Official | ✅ Via bridge | ❌ None | ✅ Module |
| K8s provider | ✅ hashicorp/kubernetes | ✅ Native | ❌ None | ⚠️ k8s module |
| Learning curve | ✅ Low (HCL) | ⚠️ Medium (needs PL) | ⚠️ Medium | ✅ Low (YAML) |
| Plan/preview | ✅ `terraform plan` | ✅ `pulumi preview` | ✅ Changeset | ❌ `--check` (limited) |
| Drift detection | ✅ State comparison | ✅ State comparison | ✅ Stack drift | ❌ Manual |
| CI/CD integration | ✅ Excellent | ✅ Good | ⚠️ Cloud-specific | ✅ Good |
| Industry adoption | ✅ Highest | ⚠️ Growing | ⚠️ Cloud-specific | ✅ High (config mgmt) |
| Open source | ✅ OpenTofu (MPL 2.0) | ⚠️ Open-core | ❌ Proprietary | ✅ GPL |

## Consequences

### Positive

- Single tool for all infrastructure: cloud resources, Kubernetes, Keycloak, Vault, databases
- `terraform plan` provides safe preview of all changes before apply
- State file enables drift detection and resource tracking
- HCL is domain-specific and accessible — no programming language knowledge required
- OpenTofu ensures long-term open-source availability
- Module system enables reusable, tested infrastructure components
- Most in-demand IaC skill — directly transferable to DevOps roles

### Negative

- State file management adds complexity (locking, remote storage) — mitigation: use remote backend (S3/MinIO + DynamoDB/PostgreSQL for locking)
- HCL has limitations compared to general-purpose languages — mitigation: acceptable for infrastructure definitions, use `for_each` and `dynamic` blocks for complex cases
- Provider version management requires attention — mitigation: pin versions with `~>` constraint

### Risks

- Terraform BSL license concerns — likelihood: Low, impact: Low — mitigation: OpenTofu as drop-in replacement
- State file corruption — likelihood: Low, impact: High — mitigation: versioned remote state with locking, state backups
- Provider bugs on upgrade — likelihood: Medium, impact: Medium — mitigation: test in dev environment first, pin versions

## References

- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [OpenTofu](https://opentofu.org/)
- [Terraform Keycloak Provider](https://registry.terraform.io/providers/mrparkers/keycloak/latest)
- [Terraform Kubernetes Provider](https://registry.terraform.io/providers/hashicorp/kubernetes/latest)
- [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) — Section 6: Terraform conventions

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
