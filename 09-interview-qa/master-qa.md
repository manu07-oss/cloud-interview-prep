# ❓ Interview Q&A Master Sheet — All 4 Roles

> Read this the night before your interview. These are the most commonly asked questions.

---

## 🔧 DevOps Engineer Questions

**Q: Walk me through your CI/CD pipeline**
> Start with code commit to Git. Pipeline triggers automatically.
> Build stage: compile, unit tests, Docker image build, security scan (Trivy, SAST).
> Test stage: deploy to staging, integration tests, DAST scan (OWASP ZAP).
> Deploy stage: Blue/Green or Canary to production, smoke tests, monitoring verification.
> Tools I've used: Azure DevOps / GitHub Actions, ArgoCD for K8s deployments.

**Q: How do you manage infrastructure as code?**
> Terraform for all cloud resources. HCL configuration, state in Azure Blob/S3.
> Module structure: networking, compute, database modules separately.
> Changes via PR → Atlantis runs plan → review → merge → auto-apply.
> Never manual changes in console — everything via code.

**Q: How do you handle a failed deployment?**
> Automated rollback if health checks fail after deployment.
> For Blue/Green: switch traffic back to Blue environment immediately.
> For K8s rolling: kubectl rollout undo deployment/myapp.
> Post-mortem: what failed, fix pipeline to catch it earlier.

**Q: What is your experience with Kubernetes?**
> Deployed and managed AKS/EKS/GKE clusters.
> Use Helm for application packaging, ArgoCD for GitOps deployment.
> Configure: HPA for autoscaling, NetworkPolicies for isolation, RBAC per team.
> Monitoring: Prometheus + Grafana, logs to centralized Log Analytics/CloudWatch.

**Q: How do you handle secrets?**
> Never in code or environment variables as plain text.
> Use managed identity / IRSA / Workload Identity to access secret stores.
> Azure Key Vault / AWS Secrets Manager / GCP Secret Manager.
> In K8s: External Secrets Operator syncs from vault to K8s Secrets.
> Rotation: automatic rotation configured in secret store.

**Q: How do you monitor your deployments?**
> Pre-deployment: set baseline metrics (error rate, latency).
> During: watch dashboards, automated rollback on error rate spike.
> Post: compare metrics before/after, check logs for errors.
> Alerts: if error rate > 1% for 5 minutes → auto-rollback + page on-call.

---

## 🏛️ Cloud Architect Questions

**Q: Design a highly available 3-tier application**
> Avoid single points of failure at every tier.
> Web tier: App Service with zone redundancy OR VM Scale Set across 3 AZs. Load balancer with health checks.
> App tier: Same — multiple instances, auto-scaling, health checks.
> Data tier: Multi-AZ database (RDS Multi-AZ / Azure SQL zone redundant / Cloud SQL HA).
> Layer: DDoS protection → WAF → Global LB → Regional LB → App tiers → Private DB.
> Supporting: Key Vault for secrets, centralized monitoring, auto-scaling policies.

**Q: How would you migrate an on-premises application to the cloud?**
> Assess: 6 Rs framework — Rehost (lift-shift), Replatform, Repurchase, Refactor, Retain, Retire.
> Rehost first (fastest, least risk): migrate VMs as-is using Azure Migrate / AWS MGN.
> Replatform next: swap databases for managed services, use managed K8s.
> Refactor eventually: redesign as microservices, serverless where appropriate.
> Hybrid during transition: ExpressRoute/Direct Connect for private connectivity.
> Test in stages, blue/green switch with rollback plan.

**Q: How do you design for cost optimization?**
> Right-size: monitor CPU/memory, downsize over-provisioned resources.
> Reserved/committed: 1-3 year commitments for baseline load (save 40-70%).
> Spot/Preemptible: for non-critical batch, dev/test workloads (save 60-90%).
> Auto-scaling: scale down nights/weekends for dev/test.
> Storage tiers: lifecycle policies to move old data to cheaper storage.
> Review: monthly FinOps review, tag all resources for cost allocation.

**Q: How do you design for security in the cloud (Zero Trust)?**
> Never trust, always verify — even internal traffic.
> Identity: MFA for all users, Managed Identity for services (no passwords).
> Network: private endpoints for PaaS, NSGs/SGs on every subnet, WAF on public endpoints.
> Data: encryption at rest (CMK in Key Vault) and in transit (TLS 1.2+).
> Least privilege: minimum permissions for every identity and service.
> Audit: all actions logged, SIEM alerts on anomalies, regular access reviews.

**Q: How do you handle disaster recovery?**
> Define RTO (how long can we be down) and RPO (how much data can we lose).
> Strategies by cost/complexity:
>   Backup/Restore: cheapest, longest RTO (hours)
>   Pilot Light: core services always running, scale up on DR
>   Warm Standby: scaled-down copy always running, scale up on DR
>   Active/Active: full capacity in both regions, instant failover
> Test DR quarterly: run chaos experiments, document runbooks.

**Q: Explain Hub-Spoke network topology**
> Hub VNet contains shared services: Firewall, Bastion, DNS, VPN Gateway.
> Spoke VNets contain applications, connected to Hub via VNet Peering.
> Benefits: centralize security (one firewall), reduce cost (shared Bastion), isolate apps.
> All traffic between spokes routes through Hub Firewall — full inspection.
> Use with Azure/AWS Transit Gateway or GCP VPC for transitive routing.

---

## 📈 SRE Questions

**Q: What are SLIs, SLOs, and SLAs?**
> SLI = measurement (e.g. 99.95% requests succeed, p99 latency 200ms).
> SLO = internal target (e.g. 99.9% availability per month).
> SLA = customer contract (e.g. 99.5% — lower than SLO to give buffer).
> Error budget = 100% - SLO. When budget is full → move fast. When low → slow down.

**Q: Walk me through how you handle a production incident**
> Detect (alert or user report) → Acknowledge → Create incident channel.
> Assess severity → notify stakeholders if SEV1/2.
> Mitigate FIRST (restore service) → then investigate root cause.
> Check: what changed recently? Dashboards, logs, traces.
> Post-mortem within 48 hours: blameless, 5 Whys, action items.

**Q: How do you reduce toil?**
> Identify: track what manual tasks engineers do repeatedly.
> Quantify: measure hours per week spent on toil.
> Automate: build runbooks → auto-remediation, self-healing systems.
> Example: restarting pods → liveness probe auto-restarts. Manual scaling → HPA.
> Target: keep toil below 50% of SRE time.

**Q: What is Chaos Engineering and how do you do it?**
> Intentionally inject failures to verify resilience.
> Process: hypothesis → controlled experiment → observe → fix weaknesses.
> Start with low-impact experiments (kill pod) → graduate to region outage.
> Tools: Azure Chaos Studio, AWS FIS, Chaos Mesh for K8s.
> Run quarterly DR drills — validate RTO/RPO actually work.

**Q: How do you set alerts properly?**
> Alert on symptoms (user impact) not causes (high CPU).
> Every alert must be actionable — if not actionable, it's noise.
> Each alert has a runbook linked.
> Review alerts monthly: which fired but required no action? Delete or downgrade those.
> Use multi-window multi-burn-rate alerts for SLO alerting.

---

## 🏗️ Platform Engineer Questions

**Q: What is an Internal Developer Platform?**
> A set of tools and workflows that enable developers to self-serve their infrastructure needs.
> Without IDP: open ticket → wait for infra team → get environment in 2 weeks.
> With IDP: run command → environment ready in 10 minutes.
> Components: service catalog (Backstage), templates, CI/CD, monitoring, secret management — all pre-integrated.

**Q: How do you implement multi-tenancy in Kubernetes?**
> Soft: namespaces + NetworkPolicies + RBAC + ResourceQuotas.
> Hard: separate clusters per tenant (for strict compliance needs).
> Virtual clusters (vcluster): K8s cluster inside namespace, good middle ground.
> Each team gets their namespace, can only see/modify their own resources.

**Q: What is GitOps and how do you implement it?**
> Git as single source of truth for all configuration.
> ArgoCD or Flux continuously syncs cluster state to match Git.
> Changes via PR → review → merge → auto-applied to cluster.
> Audit trail in Git, easy rollback via git revert.
> No direct kubectl in production — all changes through Git.

**Q: How do you enforce security policies in Kubernetes?**
> OPA Gatekeeper or Kyverno for admission control.
> Policies: "all pods must have resource limits", "images from approved registry only", "no privileged containers", "all pods must have labels".
> Pod Security Admission (built-in K8s) for baseline/restricted profiles.
> Falco for runtime security monitoring (detects suspicious behavior in running containers).

**Q: How do you handle secret management at scale across many services?**
> Central secret store: Azure Key Vault / AWS Secrets Manager / HashiCorp Vault.
> External Secrets Operator: syncs secrets from central store to K8s Secrets.
> Workload Identity / IRSA: pods authenticate without stored credentials.
> Automatic rotation configured in secret store.
> Audit: log all secret access, alert on unexpected access patterns.
> Never in Git, never in container images, never in environment variable files.

---

## 🎯 Behavioral Questions (All Roles)

**Q: Tell me about a production incident you handled**
> Use STAR: Situation, Task, Action, Result.
> Describe: what broke, how you detected it, how you mitigated, root cause, prevention.
> Show: calm under pressure, systematic debugging, communication skills, learning mindset.

**Q: How do you keep up with cloud/DevOps technology?**
> Read: AWS/Azure/GCP blogs, CNCF updates, Martin Fowler's blog, The Register.
> Hands-on: personal cloud account, build side projects, experiment with new tools.
> Community: KubeCon talks (free on YouTube), local meetups, GitHub trending.
> Certifications: currently working toward [your target cert].

**Q: Tell me about a time you improved a process**
> Example: manual deployments taking 3 hours → built pipeline → 15 minutes.
> Example: on-call alerts firing 50/week (90% noise) → tuned alerts → 8/week (all actionable).
> Example: new service onboarding taking 2 weeks → built service template → 1 day.

**Q: How do you handle disagreements with the team about technical decisions?**
> Bring data, not opinions. Show benchmarks, proof of concepts.
> Understand their concerns — often valid constraints you didn't consider.
> Propose time-boxed experiment: try approach A for 2 weeks, then evaluate.
> Document the decision and reasoning (Architecture Decision Record / ADR).
> Accept team decision even if you disagree — alignment > being right.

---

## 📚 Architecture Design Tips for Interviews

```
When asked to design something, always think about:

1. REQUIREMENTS first
   → How many users? Requests per second?
   → What's the data model?
   → SLO requirements? (99.9% or 99.99%?)
   → Budget constraints?

2. SCALABILITY
   → Where are the bottlenecks?
   → How does it scale at 10x? 100x?
   → Stateless services (scale horizontally)
   → Database (read replicas, caching, sharding)

3. RELIABILITY
   → Single points of failure?
   → Multi-AZ deployment?
   → Circuit breakers between services?
   → Graceful degradation (if DB slow, serve cached data)

4. SECURITY
   → Authentication + Authorization
   → Encryption in transit + at rest
   → Network isolation (private subnets)
   → Least privilege IAM

5. OBSERVABILITY
   → How will you know it's broken?
   → Logs, metrics, traces
   → Alerts on SLO breach

6. COST
   → Any over-engineered parts?
   → Right-sizing opportunities?
   → Appropriate storage tiers?

DRAW IT:
   Always draw architecture on whiteboard/paper.
   Start simple, add complexity layer by layer.
   Talk through your thinking out loud.
```

---

*Good luck in your interview! You've got this 💪*
*Remember: interviewers want to understand HOW you think, not just WHAT you know.*
