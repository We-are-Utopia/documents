# Getting Started

| Field         | Value                                |
|---------------|--------------------------------------|
| **Version**   | 1.0.0                                |
| **Status**    | Draft                                |
| **Author**    | Vox                                  |
| **Reviewer**  | Vox                                  |
| **Created**   | 2026-03-27                           |
| **Updated**   | 2026-03-27                           |
| **Standard**  | DOCUMENTATION-STANDARD.md            |

---

## 1. Purpose

This guide walks through setting up the Utopia development environment from scratch on a Windows machine. After completing these steps, you will have the full stack running locally.

## 2. Hardware & OS Requirements

| Requirement | Minimum | Recommended (Utopia) |
|-------------|---------|----------------------|
| **OS** | Windows 10 21H2+ / Windows 11 | Windows 11 |
| **CPU** | 4 cores | 8+ cores (i9) |
| **RAM** | 16 GB | 40 GB |
| **Disk** | 256 GB SSD | 1 TB NVMe SSD |
| **Network** | Broadband | Required for initial setup |

## 3. Prerequisites

Install the following tools in order. Check each with the verification command.

### 3.1. Core Tools

| Tool | Version | Install | Verify |
|------|---------|---------|--------|
| **Git** | 2.40+ | [git-scm.com](https://git-scm.com/) | `git --version` |
| **Docker Desktop** | 4.25+ | [docker.com](https://www.docker.com/products/docker-desktop/) | `docker --version` |
| **WSL 2** | Latest | `wsl --install` | `wsl --status` |
| **kubectl** | 1.28+ | `winget install Kubernetes.kubectl` | `kubectl version --client` |
| **Helm** | 3.13+ | `winget install Helm.Helm` | `helm version` |
| **k3d** | 5.6+ | `winget install k3d-io.k3d` | `k3d version` |

### 3.2. Backend (.NET)

| Tool | Version | Install | Verify |
|------|---------|---------|--------|
| **.NET SDK** | 8.0 LTS | [dot.net](https://dot.net/download) | `dotnet --version` |
| **EF Core CLI** | 8.x | `dotnet tool install --global dotnet-ef` | `dotnet ef --version` |

### 3.3. Frontend (Node.js)

| Tool | Version | Install | Verify |
|------|---------|---------|--------|
| **Node.js** | 20 LTS | [nodejs.org](https://nodejs.org/) (via nvm-windows recommended) | `node --version` |
| **pnpm** | 8.x+ | `npm install -g pnpm` | `pnpm --version` |

### 3.4. Infrastructure

| Tool | Version | Install | Verify |
|------|---------|---------|--------|
| **Terraform** | 1.6+ | `winget install Hashicorp.Terraform` | `terraform version` |
| **Ansible** | Latest | Via WSL: `pip install ansible` | `ansible --version` (in WSL) |
| **yq** | 4.x | `winget install MikeFarah.yq` | `yq --version` |

### 3.5. IDE

| Tool | Extensions |
|------|------------|
| **VS Code** | C# Dev Kit, ESLint, Prettier, Tailwind CSS IntelliSense, Docker, Kubernetes, GitLens, Terraform |
| **JetBrains Rider** (optional) | Bundled .NET support |

## 4. Clone the Repository

```powershell
cd D:\
git clone https://github.com/vox/utopia.git Utopia
cd Utopia
```

## 5. Docker Desktop Configuration

Open Docker Desktop → Settings:

| Setting | Value |
|---------|-------|
| **WSL 2 backend** | Enabled |
| **Resources → Memory** | 16 GB |
| **Resources → CPUs** | 6 |
| **Resources → Disk** | 100 GB |
| **Kubernetes** | Disabled (using K3d instead) |

## 6. Create K3d Cluster

```powershell
# Create the cluster with port mappings
k3d cluster create utopia `
  --servers 1 `
  --agents 2 `
  --port "80:80@loadbalancer" `
  --port "443:443@loadbalancer" `
  --api-port 6550 `
  --k3s-arg "--disable=traefik@server:0" `
  --wait

# Verify
kubectl cluster-info
kubectl get nodes
```

Expected output: 1 server node + 2 agent nodes in `Ready` state.

See [KUBERNETES-ARCHITECTURE.md](../05-infrastructure/KUBERNETES-ARCHITECTURE.md) for full cluster specification.

## 7. Configure Local DNS

Edit `C:\Windows\System32\drivers\etc\hosts` (run editor **as Administrator**):

```
# Utopia local development
127.0.0.1  api.utopia.local
127.0.0.1  api-staging.utopia.local
127.0.0.1  app.utopia.local
127.0.0.1  app-staging.utopia.local
127.0.0.1  auth.utopia.local
127.0.0.1  grafana.utopia.local
127.0.0.1  argocd.utopia.local
127.0.0.1  harbor.utopia.local
127.0.0.1  vault.utopia.local
127.0.0.1  sonarqube.utopia.local
127.0.0.1  rabbitmq.utopia.local
```

See [NETWORKING.md](../05-infrastructure/NETWORKING.md) for full DNS mapping.

## 8. Deploy Infrastructure (Terraform)

```powershell
cd infrastructure/terraform

# Initialize
terraform init

# Plan and apply (creates namespaces, Helm releases)
terraform plan -var-file="environments/dev.tfvars"
terraform apply -var-file="environments/dev.tfvars" -auto-approve
```

This deploys: Traefik, cert-manager, Keycloak, Vault, Harbor, ArgoCD, observability stack.

See [TERRAFORM-STRUCTURE.md](../05-infrastructure/TERRAFORM-STRUCTURE.md) for module details.

## 9. Initialize Services

### 9.1. Vault

```powershell
# Initialize Vault (first time only)
kubectl exec -n vault vault-0 -- vault operator init -key-shares=1 -key-threshold=1

# Save the unseal key and root token securely
# Unseal Vault
kubectl exec -n vault vault-0 -- vault operator unseal <unseal-key>
```

### 9.2. Keycloak

```powershell
# Get admin password
kubectl get secret -n identity keycloak-admin -o jsonpath='{.data.password}' | 
  ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

# Access admin console
# https://auth.utopia.local/admin
```

Create the `utopia-dev` realm with settings from [ACCESS-CONTROL-POLICY.md](../04-security/ACCESS-CONTROL-POLICY.md).

### 9.3. Harbor

```powershell
# Access Harbor UI: https://harbor.utopia.local
# Default admin: admin / Harbor12345 (change immediately)

# Create project "utopia"
# Configure robot account for CI
```

## 10. Run Backend (Development Mode)

```powershell
cd backend/src/Utopia.Api

# Restore dependencies
dotnet restore

# Apply database migrations
dotnet ef database update --project ../Utopia.Infrastructure

# Run the API
dotnet run

# Or with watch (hot reload)
dotnet watch run
```

API will be available at `https://localhost:5001`. Swagger UI at `https://localhost:5001/swagger`.

### 10.1. Using Docker Compose (Alternative)

```powershell
cd backend
docker compose up -d

# Includes: API + PostgreSQL + Redis + RabbitMQ
```

## 11. Run Frontend (Development Mode)

```powershell
cd frontend

# Install dependencies
pnpm install

# Run dev server
pnpm dev
```

Frontend will be available at `http://localhost:3000`.

### 11.1. Environment Variables

```powershell
# Copy environment template
Copy-Item .env.example .env.local

# Edit .env.local with local values
# NEXT_PUBLIC_API_URL=https://api.utopia.local/api/v1
# NEXT_PUBLIC_AUTH_URL=https://auth.utopia.local
```

## 12. Verify Everything Works

Run this checklist after initial setup:

| Check | Command / URL | Expected |
|-------|---------------|----------|
| K3d cluster | `kubectl get nodes` | 3 nodes Ready |
| All pods | `kubectl get pods -A` | All Running/Completed |
| Backend API | `curl -k https://api.utopia.local/health/ready` | `{"status":"Healthy"}` |
| Frontend | `http://localhost:3000` | Next.js app loads |
| Keycloak | `https://auth.utopia.local` | Login page |
| Grafana | `https://grafana.utopia.local` | Dashboards |
| ArgoCD | `https://argocd.utopia.local` | App list |
| Harbor | `https://harbor.utopia.local` | Projects |
| Vault | `https://vault.utopia.local` | Sealed/Unsealed status |
| RabbitMQ | `https://rabbitmq.utopia.local` | Management UI |

## 13. Common Issues

| Issue | Solution |
|-------|----------|
| Docker Desktop not starting | Ensure WSL 2 is installed: `wsl --install` |
| K3d port conflict on 80/443 | Stop IIS or other services using those ports |
| `kubectl` cannot connect | `k3d kubeconfig merge utopia --kubeconfig-merge-default` |
| DNS not resolving `*.utopia.local` | Check hosts file, flush DNS: `ipconfig /flushdns` |
| .NET restore fails | Check NuGet feeds: `dotnet nuget list source` |
| `pnpm install` fails | Clear store: `pnpm store prune`, retry |
| Vault is sealed after restart | Unseal: `kubectl exec -n vault vault-0 -- vault operator unseal <key>` |
| Terraform state conflict | `terraform state pull`, check backend config |
| Pods stuck in Pending | Check resources: `kubectl describe pod <name> -n <ns>` |
| Image pull errors from Harbor | Verify robot account credentials and Docker login |

## 14. Next Steps

After the environment is running:

1. Read the [DEVELOPMENT-WORKFLOW.md](./DEVELOPMENT-WORKFLOW.md) for day-to-day coding workflow
2. Review [CODING-STANDARD.md](../00-standards/CODING-STANDARD.md) for code conventions
3. Review [SECURITY-STANDARD.md](../00-standards/SECURITY-STANDARD.md) for security requirements
4. Check [CI-PIPELINE.md](../06-devops/CI-PIPELINE.md) to understand quality gates

## 15. References

- [PROJECT-CHARTER.md](../01-project/PROJECT-CHARTER.md) — Project vision and scope
- [TECH-STACK-DECISION.md](../01-project/TECH-STACK-DECISION.md) — Full technology stack
- [KUBERNETES-ARCHITECTURE.md](../05-infrastructure/KUBERNETES-ARCHITECTURE.md) — K3d cluster details
- [DOCKER-STRATEGY.md](../05-infrastructure/DOCKER-STRATEGY.md) — Container build strategy
- [NETWORKING.md](../05-infrastructure/NETWORKING.md) — DNS and ingress configuration

## Changelog

| Version | Date       | Author | Description          |
|---------|------------|--------|----------------------|
| 1.0.0   | 2026-03-27 | Vox    | Initial draft        |
