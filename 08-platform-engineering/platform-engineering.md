# 🏗️ Platform Engineering — Complete Reference

---

## 1. What is Platform Engineering?

```
Traditional DevOps: Every team manages their own infra
Problem: 10 teams × reinventing the wheel = waste + inconsistency

Platform Engineering Solution:
  Build an INTERNAL DEVELOPER PLATFORM (IDP)
  → "Paved road" for developers
  → Self-service infra provisioning
  → Standardized deployment patterns
  → Developers focus on code, not cloud config

Analogy:
  You don't build roads to drive.
  Platform team builds the roads (infra, tooling, standards).
  App teams just drive (build features, deploy apps).
```

### Platform Engineering vs DevOps vs SRE
```
DevOps Engineer:
  → Works WITH a product team
  → Builds and maintains CI/CD for that team
  → Manages infra for that product

SRE:
  → Focuses on reliability of services
  → SLIs/SLOs/Error budgets
  → Incident response

Platform Engineer:
  → Builds tools and platforms FOR other engineers
  → Internal customers = other dev teams
  → Creates golden paths and self-service portals
  → Reduces toil for ALL teams at once
```

---

## 2. Internal Developer Platform (IDP)

### What an IDP Provides
```
Self-service capabilities developers need:
  ✓ Spin up new environment (staging, preview, dev)
  ✓ Deploy application with one command
  ✓ View logs and metrics for their service
  ✓ Manage secrets and config
  ✓ Request cloud resources (DB, queue, cache)
  ✓ Onboard new service with standard template

Without IDP: Developer opens Jira ticket → waits 2 weeks for infra team
With IDP:    Developer types command → environment ready in 10 minutes
```

### Popular IDP Tools
```
Backstage (Spotify, open source):
  → Developer portal with service catalog
  → Software templates (create new service in minutes)
  → TechDocs (documentation as code)
  → Plugin ecosystem (integrate anything)
  → Industry standard for IDPs

Port:
  → Commercial Backstage alternative
  → Easier to set up than Backstage

Humanitec:
  → Platform orchestrator
  → Environment-as-a-Service
  → Score (open spec for workload config)

Crossplane:
  → K8s-native infrastructure provisioning
  → Developers request cloud resources via K8s CRDs
  → Platform team defines what's allowed
```

---

## 3. Golden Paths

```
Golden Path = The recommended, supported way to do something

Examples:
  Golden Path for new microservice:
    → Run: platform new-service --name my-service --lang python
    → Gets: GitHub repo, CI/CD pipeline, K8s namespace,
             monitoring dashboards, PagerDuty integration,
             all pre-configured and production-ready

  Golden Path for database:
    → YAML: kind: Database, spec: {engine: postgres, size: small}
    → Gets: PostgreSQL instance with: encryption, backups, monitoring,
             Private Endpoint, rotation of credentials — automatically

Rule: Make the right thing easy, make the wrong thing hard
  ✓ Golden path makes security automatic (can't forget encryption)
  ✗ Rolling your own = easy to misconfigure, skip security
```

---

## 4. Kubernetes Platform Patterns

### Namespace Strategy
```
Option 1: One namespace per team
  team-payments/
  team-auth/
  team-notifications/

Option 2: One namespace per environment per team
  payments-prod/
  payments-staging/
  payments-dev/

Option 3: One namespace per microservice
  payment-service/
  auth-service/
  notification-service/

Best Practice:
  Use Namespace + NetworkPolicy + RBAC together for isolation
  Each namespace has its own resource quotas (CPU/memory limits)
```

### Multi-Tenancy in Kubernetes
```
Soft multi-tenancy (same cluster, isolated):
  - Namespaces
  - NetworkPolicies (block cross-namespace traffic)
  - RBAC (teams can only see their namespace)
  - Resource Quotas (per-namespace CPU/memory limits)
  - Pod Security Admission (restrict privileged pods)

Hard multi-tenancy (separate clusters per team):
  - Complete isolation
  - More expensive
  - More management overhead
  - Used for: compliance isolation, very high security

Virtual Clusters (vcluster):
  - K8s cluster inside a namespace
  - Appears as real cluster to users
  - Shared underlying nodes
  - Good middle ground
```

### Platform Services on Kubernetes
```
Every cluster should have:

Networking:
  - Ingress controller (NGINX / Traefik / Gateway API)
  - Service mesh (Istio / Linkerd) for mTLS + observability
  - CNI plugin (Calico / Cilium) for NetworkPolicy

Security:
  - cert-manager (automatic TLS certificates)
  - External Secrets Operator (sync secrets from Vault/KV/SM)
  - OPA Gatekeeper / Kyverno (policy enforcement)
  - Falco (runtime security monitoring)

Observability:
  - Prometheus + Grafana (metrics)
  - Loki (logs)
  - Tempo / Jaeger (traces)
  - OpenTelemetry Collector (collect all signals)

GitOps:
  - ArgoCD or Flux (continuous deployment)

Autoscaling:
  - HPA (Horizontal Pod Autoscaler)
  - KEDA (event-driven autoscaling — queue depth, etc)
  - Cluster Autoscaler (scale nodes)
  - VPA (Vertical Pod Autoscaler — right-size resource requests)
```

---

## 5. Platform Team Metrics

```
What does a good platform team measure?

Developer Experience (DX):
  - Time to onboard new service (minutes, not days)
  - Time to spin up environment (< 10 min target)
  - Developer satisfaction score (quarterly survey)

Reliability:
  - Platform SLO (e.g. CI/CD pipeline 99.9% available)
  - Mean time to restore (MTTR) for platform issues

Adoption:
  - % of teams using golden paths
  - % of services using standard templates
  - % NOT using golden path (shows where improvement needed)

Efficiency:
  - Reduction in toil for app teams
  - Infrastructure cost per deployment
```

---

## 6. Platform Engineering Interview Q&A

**Q: What is the difference between Platform Engineering and DevOps?**
> DevOps = cultural practice where dev and ops collaborate, often embedded in product teams.
> Platform Engineering = builds internal platforms that other engineers USE.
> Platform team's customers are internal developers. They build self-service tools so every team doesn't reinvent infra.

**Q: What is Backstage and why would you use it?**
> Backstage is an open-source developer portal by Spotify.
> Provides: service catalog (see all microservices), software templates (create new services from templates), TechDocs (documentation in code repos).
> Reduces cognitive overhead for developers — one place to find everything.

**Q: What is Crossplane?**
> Kubernetes-native infrastructure provisioning tool.
> Platform team defines Composite Resources (XRDs) — templates for cloud resources.
> App developers request resources via standard K8s YAML.
> Example: developer creates a Database object → Crossplane provisions Azure SQL with all platform standards applied.

**Q: What is the difference between Kyverno and OPA Gatekeeper?**
> Both are K8s policy engines — enforce rules on K8s resources.
> OPA Gatekeeper: uses Rego language for policies, more powerful, steeper learning curve.
> Kyverno: uses YAML policies, easier to write and read, built for Kubernetes.
> Example policy: "All pods must have resource limits", "Images must come from approved registry only".

**Q: How do you handle secrets in Kubernetes?**
> Native K8s Secrets are base64 encoded (NOT encrypted at rest by default).
> Better options:
>   1. Seal secrets (Bitnami Sealed Secrets) — encrypted in Git
>   2. External Secrets Operator — sync from Key Vault/Secrets Manager
>   3. CSI Secrets Driver — mount secrets directly from Vault/KV/SM
>   4. Vault Agent Injector — inject secrets as env vars or files
> Best practice: never store raw secrets in Git, always use external secret store

---

*Next: Read `09-interview-qa/` for role-specific questions*
