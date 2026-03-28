# ADR-0006: K3s for Local Kubernetes

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

Utopia needs a local Kubernetes environment for developing and testing Helm charts, ArgoCD deployments, network policies, and Kubernetes-native tooling (Prometheus, Loki, etc.). The cluster must be production-like while running efficiently on a developer workstation with 40 GB RAM.

## Decision Drivers

- **Resource efficiency** — must coexist with other services (PostgreSQL, Redis, etc.)
- **Production parity** — K8s API compatible, supports standard manifests
- **Startup speed** — fast cluster creation and teardown for development
- **Docker integration** — works well with Docker Desktop on Windows
- **Feature completeness** — ingress, storage, RBAC, network policies
- **Ease of use** — minimal configuration to get started

## Considered Options

1. **K3d** (K3s in Docker) — lightweight K8s via Docker containers
2. **minikube** — local K8s cluster in a VM or container
3. **kind** (Kubernetes in Docker) — K8s nodes as Docker containers
4. **Docker Desktop Kubernetes** — built-in K8s in Docker Desktop

## Decision

We will use **K3d** (K3s running inside Docker containers) for local Kubernetes development.

K3d provides K3s clusters as Docker containers, enabling fast creation/deletion, multi-node clusters, and Docker registry integration — all within Docker Desktop on Windows.

## Comparison

| Criterion | K3d | minikube | kind | Docker Desktop K8s |
|-----------|-----|----------|------|--------------------|
| Resource usage | ✅ ~500 MB/node | ⚠️ ~1 GB (VM) | ✅ ~500 MB/node | ⚠️ ~1.5 GB |
| Startup time | ✅ ~20 seconds | ❌ ~2 minutes | ✅ ~30 seconds | ⚠️ ~1 minute |
| Multi-node support | ✅ Native | ⚠️ Limited | ✅ Native | ❌ Single node |
| Docker integration | ✅ Native | ⚠️ Requires config | ✅ Native | ✅ Native |
| Bundled ingress | ✅ Traefik included | ⚠️ Addon | ❌ Manual | ❌ Manual |
| Local registry | ✅ Built-in support | ⚠️ Addon | ⚠️ Manual | ❌ Manual |
| Windows support | ✅ Via Docker Desktop | ✅ Hyper-V/WSL2 | ✅ Via Docker Desktop | ✅ Native |
| Helm support | ✅ Full | ✅ Full | ✅ Full | ✅ Full |
| Network policies | ✅ Via Calico/Cilium | ✅ Via CNI plugin | ✅ Via CNI plugin | ❌ Limited |
| Production parity | ✅ K3s = certified K8s | ✅ Full K8s | ✅ Full K8s | ✅ Full K8s |
| Cluster management | ✅ CLI (create/delete fast) | ✅ CLI | ✅ CLI | ❌ GUI only |

## Consequences

### Positive

- Cluster creation/deletion in ~20 seconds — rapid iteration
- Multi-node clusters simulate production topology (1 server + 2 agents)
- Traefik ingress controller bundled — immediate ingress support
- Built-in local registry integration — push images without external registry
- Minimal resource usage (~500 MB per node) — fits within 40 GB RAM budget
- Same K8s API as production — Helm charts and manifests work identically
- K3s is CNCF-certified — not a toy, a production-grade distribution

### Negative

- K3s uses SQLite by default instead of etcd — mitigation: acceptable for local dev, production can use embedded etcd or external
- Some K8s features may differ from full K8s (e.g., storage drivers) — mitigation: test critical features in staging with full K8s
- Docker-in-Docker networking adds complexity — mitigation: K3d handles this transparently

### Risks

- K3d version lag behind K3s — likelihood: Low, impact: Low — mitigation: K3d releases closely follow K3s
- Port conflicts with other Docker services — likelihood: Medium, impact: Low — mitigation: use K3d port mapping configuration

## References

- [K3d Documentation](https://k3d.io/)
- [K3s Documentation](https://docs.k3s.io/)
- [K3d Quick Start](https://k3d.io/v5.7.3/#quick-start)
- [ADR-0008](./ADR-0008-gitops-argocd.md) — ArgoCD deployment on K3s

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial decision     |
