# ⚙️ DevOps & CI/CD — Complete Reference

---

## 1. CI/CD Pipeline — Full Flow

```
Developer writes code
        │
        ▼
  git push → feature branch
        │
        ▼
  Pull Request / Merge Request opened
        │
        ▼
  ┌─────────────────────────────────┐
  │  CI — Continuous Integration   │
  │                                 │
  │  1. Code checkout               │
  │  2. Dependency install          │
  │  3. Compile / Build             │
  │  4. Unit tests                  │
  │  5. Code coverage check         │
  │  6. Static analysis (SonarQube) │
  │  7. SAST security scan          │
  │  8. Build Docker image          │
  │  9. Container security scan     │
  │     (Trivy / Snyk / Grype)      │
  │  10. Push to container registry │
  └─────────────┬───────────────────┘
                │
                ▼
  ┌─────────────────────────────────┐
  │  CD — Continuous Delivery      │
  │                                 │
  │  11. Deploy to DEV/TEST env     │
  │  12. Integration tests          │
  │  13. DAST scan (OWASP ZAP)      │
  │  14. Performance tests          │
  │  15. Deploy to STAGING          │
  │  16. Smoke tests                │
  │  17. Manual approval gate       │  ← optional
  │  18. Deploy to PRODUCTION       │
  │  19. Health check / verify      │
  │  20. Notify team (Slack/Email)  │
  └─────────────────────────────────┘
```

---

## 2. CI/CD Tools Comparison

| Tool | Type | Best For |
|------|------|---------|
| **Azure Pipelines** | Managed | Azure ecosystem, YAML pipelines |
| **GitHub Actions** | Managed | GitHub repos, open source |
| **GitLab CI** | Self/Managed | All-in-one DevOps platform |
| **Jenkins** | Self-hosted | Legacy, full control, plugins |
| **AWS CodePipeline** | Managed | AWS native workflows |
| **Google Cloud Build** | Managed | GCP native, fast builds |
| **ArgoCD** | GitOps | Kubernetes deployments |
| **Flux** | GitOps | Kubernetes (CNCF official) |
| **Tekton** | Kubernetes-native | Cloud-native CI/CD on K8s |

---

## 3. Deployment Strategies — CRITICAL Interview Topic

### Blue/Green Deployment
```
BLUE = Current production (live)
GREEN = New version (deployed, not live yet)

Step 1: Deploy new version to GREEN environment
Step 2: Run tests on GREEN
Step 3: Switch traffic from BLUE → GREEN (instant cutover)
Step 4: Monitor GREEN (if issues → switch back to BLUE instantly)
Step 5: After confidence → decommission BLUE

Pros:  Zero downtime, instant rollback
Cons:  Expensive (double infrastructure during switch)

Azure: App Service Deployment Slots
AWS:   ALB target groups swap, CodeDeploy
GCP:   Cloud Run traffic splitting
```

### Canary Deployment
```
Release new version to SMALL % of users first

  100% → old version (v1)

  After canary:
  95%  → v1 (old)
   5%  → v2 (new) ← canary

  If metrics are good → gradually increase:
  80%  → v1
  20%  → v2

  Full rollout:
  0%   → v1
  100% → v2

Monitor: error rate, latency, business metrics
Rollback: send 100% back to v1 if issues detected

Azure: Azure Front Door + Traffic Manager weights
AWS:   ALB weighted target groups, AppMesh
GCP:   Cloud Run traffic splitting, Istio
Kubernetes: Argo Rollouts, Flagger
```

### Rolling Deployment
```
Update instances one by one (or in batches)

Pods: [v1] [v1] [v1] [v1] [v1] [v1]

Rolling:
[v2] [v1] [v1] [v1] [v1] [v1]  ← 1 updated
[v2] [v2] [v1] [v1] [v1] [v1]  ← 2 updated
[v2] [v2] [v2] [v1] [v1] [v1]  ← 3 updated
[v2] [v2] [v2] [v2] [v2] [v2]  ← all updated

Pros:  No extra infrastructure needed
Cons:  Both versions run simultaneously (compatibility issues possible)

Kubernetes: default deployment strategy
maxSurge: 25% (how many extra pods during update)
maxUnavailable: 25% (how many pods can be down)
```

### Feature Flags
```
Deploy code to production but disable features for most users

if (featureFlag.isEnabled("new-payment-ui", user)) {
    showNewPaymentUI();
} else {
    showOldPaymentUI();
}

Tools: LaunchDarkly, Azure App Configuration, Unleash, Flagsmith

Benefits:
  - Separate deployment from release
  - Test with internal users first
  - Kill switch if issues
  - A/B testing capability
```

---

## 4. Infrastructure as Code (IaC) — Terraform Deep Dive

### Terraform Structure
```
project/
├── main.tf           ← main resources
├── variables.tf      ← input variables
├── outputs.tf        ← output values
├── provider.tf       ← cloud provider config
├── terraform.tfvars  ← variable values (don't commit secrets!)
└── modules/
    ├── networking/   ← reusable VNet/VPC module
    ├── compute/      ← reusable VM/K8s module
    └── database/     ← reusable DB module
```

### Terraform Workflow in Teams
```
Developer creates/modifies .tf files
        │
        ▼
git push → opens Pull Request
        │
        ▼
Atlantis (or CI pipeline) runs:
  terraform fmt    ← format check
  terraform validate ← syntax check
  terraform plan   ← shows what will change
        │
        ▼
Plan output added as PR comment
        │
        ▼
Team reviews plan
        │
        ▼
PR approved + merged
        │
        ▼
Atlantis runs: terraform apply
        │
        ▼
Infrastructure updated!
        │
        ▼
State stored in: Azure Blob / AWS S3 / Terraform Cloud
```

### Terraform State Management
```
State file = Terraform's memory of what it created

Local state: terraform.tfstate (local file)
  - NEVER for teams (conflicts!)
  - NEVER commit to Git (contains secrets)

Remote state: Blob/S3/GCS + state locking
  # Azure backend
  terraform {
    backend "azurerm" {
      resource_group_name  = "tfstate-rg"
      storage_account_name = "tfstatestorage"
      container_name       = "tfstate"
      key                  = "prod.terraform.tfstate"
    }
  }

State locking: prevents two people applying simultaneously
  Azure: Blob lease
  AWS:   DynamoDB table
  GCP:   GCS bucket
```

---

## 5. GitOps — ArgoCD Deep Dive

```
GitOps Philosophy:
  "If it's not in Git, it doesn't exist"
  "Git is the single source of truth"

ArgoCD Flow:
  Git Repo (K8s manifests)
        │
        ▼ ArgoCD watches for changes
  ArgoCD detects drift
        │
        ▼
  ArgoCD syncs cluster to match Git
        │
        ▼
  Kubernetes cluster = exactly what's in Git

Benefits:
  - Audit trail (Git history = change history)
  - Easy rollback (git revert)
  - No manual kubectl commands in production
  - Developers can deploy via PRs (no cluster access needed)
```

---

## 6. Container Best Practices

### Dockerfile Best Practices
```dockerfile
# Use specific version tags (not latest!)
FROM node:20.10-alpine

# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy package files first (Docker layer caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Switch to non-root
USER appuser

# Don't use root port
EXPOSE 3000

CMD ["node", "server.js"]
```

### Container Security Scanning
```
Trivy:    trivy image myapp:latest
Snyk:     snyk container test myapp:latest
Grype:    grype myapp:latest

What they check:
  - OS package vulnerabilities (CVEs)
  - Application dependency vulnerabilities
  - Secrets accidentally baked into image
  - Configuration issues

Block deployment if CRITICAL CVEs found in CI!
```

---

## 7. DevOps Interview Q&A

**Q: What is the difference between CI and CD?**
> CI (Continuous Integration) = automatically build and test code every commit. Goal: find bugs early.
> CD can mean two things:
>   Continuous Delivery = automatically deploy to staging, manual approval for production.
>   Continuous Deployment = automatically deploy all the way to production with no manual step.

**Q: How do you handle secrets in CI/CD pipelines?**
> Never hardcode secrets in code or pipeline YAML.
> Store in: Azure Key Vault, AWS Secrets Manager, HashiCorp Vault, GitHub Encrypted Secrets.
> Access in pipeline: use managed identity / OIDC federation (no stored passwords).
> Rotate secrets regularly. Audit who accesses them.

**Q: What is GitOps?**
> GitOps is a practice where Git is the single source of truth for infrastructure and application config.
> An operator (ArgoCD/Flux) continuously syncs the live state to match what's in Git.
> Changes are made via PRs, not kubectl commands. Provides audit trail, rollback capability.

**Q: How do you do zero-downtime deployments?**
> Choose a strategy: Blue/Green (instant cutover), Canary (gradual), Rolling (one by one).
> In Kubernetes: Deployment rolling update + readiness probes ensure new pods are healthy before old ones stop.
> Health checks must pass before routing traffic to new version.

**Q: What is the 12-Factor App methodology?**
> 12 principles for building modern cloud apps:
> 1. Codebase: One repo, many deploys
> 2. Dependencies: Explicitly declare
> 3. Config: Store in environment variables
> 4. Backing services: Treat as attached resources
> 5. Build/release/run: Strictly separate stages
> 6. Processes: Stateless, share nothing
> 7. Port binding: Export services via port
> 8. Concurrency: Scale out via process model
> 9. Disposability: Fast startup, graceful shutdown
> 10. Dev/prod parity: Keep environments similar
> 11. Logs: Treat as event streams
> 12. Admin processes: Run as one-off processes

---

*Next: Read `07-sre-observability/sre-reference.md`*
