# 🟡 GCP — Complete Reference for Interviews

---

## 1. GCP Global Infrastructure

```
GCP Region (e.g. asia-south1 = Mumbai)
  ├── Zone: asia-south1-a
  ├── Zone: asia-south1-b
  └── Zone: asia-south1-c

GCP has 40+ regions worldwide
India: Mumbai (asia-south1), Delhi (asia-south2)

Note: GCP calls them "zones" not "Availability Zones"
Same concept — isolated data centers within a region
```

---

## 2. GCP Networking — Core Services

### VPC (Virtual Private Cloud)
```
GCP VPC is GLOBAL — one VPC spans all regions!
(AWS/Azure VPCs are regional — big difference)

Example GCP VPC:
┌──────────────────────────────────────────────────┐
│  my-vpc (Global)                                 │
│                                                  │
│  ┌───────────────────┐  ┌───────────────────┐   │
│  │ asia-south1       │  │ us-central1       │   │
│  │ (Mumbai)          │  │ (Iowa)            │   │
│  │ 10.0.1.0/24       │  │ 10.0.2.0/24       │   │
│  └───────────────────┘  └───────────────────┘   │
│                                                  │
│  Subnets are REGIONAL (not zonal like AWS)       │
└──────────────────────────────────────────────────┘

Benefit: Resources in different regions can communicate
         over GCP's private backbone without peering!
```

### Firewall Rules in GCP
```
GCP Firewall Rules = applied at VPC level, use tags or service accounts
(Not per-subnet like AWS NACLs)

Example:
  Name: allow-http-to-web
  Direction: INGRESS
  Target: instances with tag "web-server"
  Source: 0.0.0.0/0
  Ports: tcp:80, tcp:443
  Action: ALLOW

Use tags to group VMs → apply rules to tags (flexible!)
OR use Service Accounts as targets (more secure)
```

---

## 3. GCP Compute Services

| Service | Use Case | Azure / AWS Equivalent |
|---------|----------|----------------------|
| **Compute Engine** | Virtual machines | Azure VMs / EC2 |
| **Managed Instance Groups** | Auto-scaling VMs | VM Scale Sets / ASG |
| **App Engine** | PaaS for web apps | App Service / Elastic Beanstalk |
| **GKE (Google Kubernetes Engine)** | Managed Kubernetes | AKS / EKS |
| **Cloud Run** | Serverless containers | Container Apps / Fargate |
| **Cloud Functions** | Serverless functions | Azure Functions / Lambda |
| **Bare Metal Solution** | Physical servers in GCP | - |

### GKE — Why GCP is Famous for Kubernetes
```
Google INVENTED Kubernetes (open-sourced in 2014)
GKE = most mature managed K8s offering

GKE Autopilot:
  - Google manages nodes, you just deploy workloads
  - Pay per pod resource, not per node
  - Automatic security hardening

GKE Standard:
  - You manage node pools
  - More control over machine types
  - Traditional K8s experience
```

### Preemptible / Spot VMs
```
Preemptible VM (GCP) = Spot Instance (AWS) = Spot VM (Azure)
  - Up to 91% cheaper than regular VMs
  - Can be terminated by Google with 30-second notice
  - Max 24 hours runtime
  - Good for: batch jobs, ML training, fault-tolerant workloads
```

---

## 4. GCP Storage Services

| Service | Type | Use Case | AWS/Azure Equivalent |
|---------|------|----------|---------------------|
| **Cloud Storage** | Object storage | Files, backups, data lake | S3 / Blob Storage |
| **Persistent Disk** | Block storage | VM disks | EBS / Azure Managed Disks |
| **Filestore** | NFS file storage | Shared file system | EFS / Azure Files |
| **Cloud Storage Archive** | Archive | Long-term cheap storage | S3 Glacier / Azure Archive |

### Cloud Storage Classes
```
Standard   → Hot data, frequent access
Nearline   → Once per month access (30-day min)
Coldline   → Once per quarter access (90-day min)
Archive    → Less than once per year (365-day min, cheapest)

Autoclass feature → GCP automatically moves objects to right class!
```

---

## 5. GCP Database Services

| Service | Type | Use Case | AWS/Azure Equivalent |
|---------|------|----------|---------------------|
| **Cloud SQL** | Relational (MySQL/PG/SQL Server) | OLTP apps | RDS / Azure SQL |
| **Cloud Spanner** | Globally distributed relational | Planet-scale SQL | Aurora Global / - |
| **Firestore** | NoSQL document | Mobile/web apps | DynamoDB / Cosmos DB |
| **Bigtable** | NoSQL wide-column | Time-series, IoT | DynamoDB / - |
| **AlloyDB** | PostgreSQL-compatible, AI-ready | High performance OLTP | Aurora / - |
| **Memorystore** | Managed Redis/Memcached | Caching | ElastiCache / Azure Redis |
| **BigQuery** | Data warehouse | Analytics, ML | Redshift / Synapse |

### Cloud Spanner — Unique to GCP
```
- Globally distributed relational database
- ACID transactions across regions
- Scales horizontally (like NoSQL) but with SQL
- 99.999% SLA (five nines!)
- Use for: global banking, inventory, games
- Expensive but unique in the industry
```

### BigQuery — Data Warehouse Powerhouse
```
- Serverless (no infrastructure to manage)
- Petabyte-scale queries in seconds
- Pay per query (per TB scanned)
- Built-in ML (BQML — run ML with SQL!)
- Columnar storage (fast aggregations)
- Separates compute and storage

Common interview question: "How would you build a data pipeline?"
Answer: Pub/Sub → Dataflow → BigQuery → Looker Studio
```

---

## 6. GCP Security Services

| Service | Purpose | AWS/Azure Equivalent |
|---------|---------|---------------------|
| **Cloud IAM** | Identity & access management | IAM / Azure AD |
| **Cloud KMS** | Key management | KMS / Key Vault |
| **Secret Manager** | Store secrets | Secrets Manager / Key Vault |
| **Cloud Armor** | WAF + DDoS protection | Shield + WAF / Azure DDoS + WAF |
| **Security Command Center** | Security posture | Security Hub / Defender for Cloud |
| **VPC Service Controls** | Data exfiltration prevention | Azure Policy / AWS SCPs |
| **Binary Authorization** | Only deploy trusted containers | - |
| **Workload Identity Federation** | Keyless auth for external systems | Managed Identity / IRSA |

### GCP IAM — Key Concepts
```
Resource Hierarchy:
  Organization → Folders → Projects → Resources

IAM works by granting roles at any level:
  - Grant at Organization → applies to everything below
  - Grant at Project → applies only to that project

Role Types:
  Primitive (Basic): Owner, Editor, Viewer (too broad, avoid!)
  Predefined: roles/compute.instanceAdmin, roles/storage.objectViewer
  Custom: You define exact permissions needed

Best Practice:
  ✓ Use predefined roles, not primitive
  ✓ Use Workload Identity for GKE workloads (not service account keys)
  ✓ Least privilege always
  ✓ Audit with Cloud Audit Logs
```

---

## 7. GCP Architecture Patterns

### Pattern 1: 3-Tier Web App on GCP
```
Internet
   │
   ▼
[Cloud Armor + WAF]           ← DDoS, IP filtering, OWASP rules
   │
   ▼
[Cloud Load Balancing (HTTP)]  ← Global, anycast, L7
   │
   ▼
[Cloud Run / GKE]             ← Containers in private subnet
   │  Firewall: only LB traffic allowed in
   ▼
[Cloud SQL (Private IP)]       ← Private Service Connect only
   │  No public IP assigned

Supporting:
  Secret Manager ← credentials
  Cloud Monitoring + Logging ← observability
  Cloud Trace ← distributed tracing
```

### Pattern 2: Data / ML Pipeline (GCP Strength)
```
Data Sources
  │
  ▼
[Pub/Sub]                   ← Real-time event streaming
  │
  ▼
[Dataflow]                  ← ETL/ELT processing (Apache Beam)
  │
  ├── [BigQuery]             ← Data warehouse for analytics
  │       │
  │       ▼
  │   [Looker Studio]        ← Dashboards and reports
  │       │
  │       ▼
  │   [BigQuery ML]          ← ML models using SQL
  │
  └── [Vertex AI]            ← Advanced ML model training
          │
          ▼
      [Model deployment]     ← Serve predictions via API

This is GCP's strongest differentiation from Azure/AWS!
```

### Pattern 3: Multi-Region on GCP
```
Users worldwide
   │
   ▼
[Cloud Load Balancing]       ← Already global! Single anycast IP
   │                            Routes to nearest healthy backend
   ├── [us-central1 backends]
   ├── [europe-west1 backends]
   └── [asia-south1 backends]

Same VPC spans all regions → simpler network than Azure/AWS
Cloud Spanner → global database, no read replicas needed
Cloud Storage → multi-region bucket automatically

GCP advantage: Global LB is built-in, not an add-on like Azure Front Door
```

---

## 8. GCP DevOps Services

| Service | Purpose | AWS/Azure Equivalent |
|---------|---------|---------------------|
| **Cloud Source Repositories** | Git hosting | CodeCommit / Azure Repos |
| **Cloud Build** | CI/CD build system | CodeBuild / Azure Pipelines |
| **Artifact Registry** | Container + package registry | ECR / Azure ACR |
| **Cloud Deploy** | Managed deployment pipelines | CodeDeploy / Azure Pipelines |
| **Config Connector** | K8s-native GCP resource management | - |
| **Cloud Monitoring** | Metrics, dashboards, alerts | CloudWatch / Azure Monitor |
| **Cloud Logging** | Centralized log management | CloudWatch Logs / Log Analytics |
| **Cloud Trace** | Distributed tracing | X-Ray / App Insights |
| **Error Reporting** | App error tracking | - |

---

## 9. GCP Interview Q&A

**Q: How is GCP VPC different from AWS VPC?**
> GCP VPC is global — one VPC spans all regions, subnets are regional.
> AWS VPC is regional — need separate VPCs per region, peering or Transit Gateway for multi-region.
> GCP VPC is simpler for multi-region: resources in different regions in same VPC communicate privately.

**Q: What is Cloud Spanner and when would you use it?**
> Globally distributed relational database with horizontal scaling.
> Use when you need: SQL semantics + global scale + strong consistency across regions.
> Use cases: global financial systems, inventory, SaaS platforms needing multi-region SQL.
> Expensive, so use Cloud SQL for regional apps, Spanner for global.

**Q: What is BigQuery and how is pricing different?**
> Serverless data warehouse. No clusters to manage.
> Pricing: Pay per TB of data scanned (on-demand) or flat rate (slots).
> Columnar storage makes it very efficient. Use SELECT only needed columns to reduce cost.
> Partitioning and clustering reduce data scanned → reduce cost.

**Q: What is Workload Identity in GKE?**
> Allows K8s pods to authenticate to GCP services without service account keys.
> K8s service account linked to GCP service account.
> Pod automatically gets tokens. No JSON key files needed.
> Much more secure than mounting service account keys in pods.

**Q: What is Cloud Armor?**
> GCP's WAF + DDoS protection service.
> Applied at Cloud Load Balancer level.
> Features: OWASP rule sets, rate limiting, geo-blocking, IP allow/deny lists, adaptive protection (ML-based DDoS).
> Equivalent to AWS Shield + WAF or Azure DDoS Protection + WAF.

---

*Next: Read `04-multi-cloud-comparison/comparison.md` for side-by-side comparison*
