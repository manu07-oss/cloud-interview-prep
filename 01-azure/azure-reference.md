# 🔷 Azure — Complete Reference for Interviews

---

## 1. Azure Global Infrastructure

```
Azure Geography (e.g. India)
  └── Region (e.g. Central India — Mumbai)
        ├── Availability Zone 1
        ├── Availability Zone 2
        └── Availability Zone 3

Azure has 60+ regions worldwide
India regions: Central India, South India, West India
```

---

## 2. Azure Networking — Core Services

### Virtual Network (VNet)
```
Your private network in Azure.

Example VNet design for a FinTech app:
┌─────────────────────────────────────────┐
│  VNet: 10.0.0.0/16                     │
│                                         │
│  ┌─────────────────┐                   │
│  │ Web Subnet      │  10.0.1.0/24      │
│  │ App Service     │                   │
│  └─────────────────┘                   │
│  ┌─────────────────┐                   │
│  │ App Subnet      │  10.0.2.0/24      │
│  │ APIs / Backend  │                   │
│  └─────────────────┘                   │
│  ┌─────────────────┐                   │
│  │ Data Subnet     │  10.0.3.0/24      │
│  │ SQL / Redis     │                   │
│  └─────────────────┘                   │
└─────────────────────────────────────────┘
```

### Network Security Group (NSG)
- Firewall rules at subnet or NIC level
- Rules have: Priority, Source, Destination, Port, Allow/Deny
- Lower number = higher priority (100 beats 200)

```
Example NSG rules for Web subnet:
Priority | Source        | Dest    | Port | Action
100      | AppGateway    | WebTier | 443  | ALLOW
200      | LoadBalancer  | WebTier | *    | ALLOW
4096     | *             | *       | *    | DENY  ← default deny all
```

### Hub-Spoke Network Topology
```
                ┌──────────────┐
                │   Hub VNet   │
                │  (Shared     │
                │  Services)   │
                │  - Firewall  │
                │  - Bastion   │
                │  - DNS       │
                └──────┬───────┘
           ┌───────────┼───────────┐
           │           │           │
    ┌──────┴───┐ ┌─────┴────┐ ┌───┴──────┐
    │ Spoke 1  │ │ Spoke 2  │ │ Spoke 3  │
    │ (App A)  │ │ (App B)  │ │ (App C)  │
    └──────────┘ └──────────┘ └──────────┘

Why Hub-Spoke?
- Centralize security (one firewall for all)
- Reduce cost (shared Bastion, DNS)
- Isolate apps from each other
```

---

## 3. Azure Compute Services

| Service | Use Case | When to use |
|---------|----------|------------|
| **Virtual Machines** | Full OS control | Lift-and-shift, legacy apps |
| **VM Scale Sets** | Auto-scaling VMs | Variable load workloads |
| **App Service** | Web apps, APIs | No infra management wanted |
| **AKS** | Container orchestration | Microservices at scale |
| **Container Apps** | Serverless containers | Simple containerized apps |
| **Azure Functions** | Serverless code | Event-driven, short tasks |
| **Azure Batch** | Large parallel jobs | HPC, rendering, ML training |

---

## 4. Azure Storage Services

| Service | Use Case | Equivalent |
|---------|----------|-----------|
| **Blob Storage** | Files, images, backups | AWS S3 |
| **Azure Files** | SMB/NFS file share | AWS EFS |
| **Azure Disks** | VM OS/data disks | AWS EBS |
| **Table Storage** | NoSQL key-value | AWS DynamoDB (basic) |
| **Queue Storage** | Simple message queue | AWS SQS |

### Blob Storage Tiers
```
Hot    → Frequently accessed → Higher storage cost, lower access cost
Cool   → Infrequently accessed → Lower storage, higher access
Cold   → Rarely accessed → Even lower storage cost
Archive → Almost never → Cheapest storage, hours to retrieve

Lifecycle policy: auto-move blobs between tiers based on age
```

---

## 5. Azure Database Services

| Service | Type | Use Case |
|---------|------|---------|
| **Azure SQL Database** | Relational (SQL Server) | OLTP applications |
| **Azure SQL Managed Instance** | Relational (full SQL Server) | Migrate on-prem SQL |
| **Cosmos DB** | NoSQL (multi-model) | Global scale, low latency |
| **Azure Database for PostgreSQL** | Relational (PostgreSQL) | Open source preference |
| **Azure Database for MySQL** | Relational (MySQL) | LAMP stack apps |
| **Azure Cache for Redis** | In-memory cache | Session store, caching |
| **Azure Synapse** | Data warehouse + analytics | Big data, reporting |

### Cosmos DB — Important Interview Points
```
- Multi-region write capability
- 5 consistency levels (Strong → Eventual)
- Automatic indexing
- 99.999% SLA with multi-region
- Partitioned by partition key
- Pay per RU (Request Unit)
```

---

## 6. Azure Security Services

| Service | What it does |
|---------|-------------|
| **Azure Active Directory (Entra ID)** | Identity provider, SSO, MFA |
| **Key Vault** | Store secrets, keys, certificates |
| **Azure Firewall** | Network-level traffic filtering |
| **WAF** | Web app attack protection (OWASP) |
| **DDoS Protection** | Volumetric attack mitigation |
| **Defender for Cloud** | Security posture + threat detection |
| **Microsoft Sentinel** | SIEM — security event management |
| **Private Link** | Access PaaS services privately |
| **Managed Identity** | App identity without passwords |

### Managed Identity — Very Common Interview Topic
```
PROBLEM: App needs to access database → where to store password?
WRONG:   Hard-code password in code ← NEVER do this!
WRONG:   Store in config file ← risky
RIGHT:   Use Managed Identity

HOW IT WORKS:
App Service (with Managed Identity enabled)
  → Azure says "this app IS this identity"
  → App can get token from Azure AD automatically
  → No password needed anywhere!
  → Key Vault grants access to this identity
  → App reads secrets without ever seeing a password
```

---

## 7. Azure Load Balancing — Which One to Use?

```
Question: Does traffic come from Internet or internal?
  └── Internet → Need public-facing LB
        ├── Global (multi-region)?   → Azure Front Door
        └── Regional only?
              ├── HTTP/HTTPS (L7)?   → Application Gateway
              └── TCP/UDP (L4)?      → Azure Load Balancer

  └── Internal → Application Gateway (internal) or Azure LB (internal)
```

| Service | Layer | Scope | WAF | SSL |
|---------|-------|-------|-----|-----|
| **Azure Load Balancer** | L4 | Regional | No | No |
| **Application Gateway** | L7 | Regional | Yes | Yes |
| **Azure Front Door** | L7 | Global | Yes | Yes |
| **Traffic Manager** | DNS | Global | No | No |

---

## 8. Azure DevOps Services

| Service | Purpose |
|---------|---------|
| **Azure DevOps Pipelines** | CI/CD — build, test, deploy |
| **Azure Repos** | Git repository hosting |
| **Azure Artifacts** | Package registry (npm, Maven, NuGet) |
| **Azure Boards** | Work item tracking (like Jira) |
| **Azure Container Registry** | Docker image storage |

### CI/CD Pipeline Flow (Azure DevOps)
```
Developer pushes code to Azure Repos / GitHub
           │
           ▼
    Azure Pipeline triggers
           │
    ┌──────┴──────────────────┐
    │  BUILD STAGE            │
    │  - Compile code         │
    │  - Run unit tests       │
    │  - Build Docker image   │
    │  - Push to ACR          │
    └──────┬──────────────────┘
           │
    ┌──────┴──────────────────┐
    │  SECURITY STAGE         │
    │  - Trivy (container)    │
    │  - OWASP ZAP (DAST)     │
    │  - SonarQube (SAST)     │
    │  - Dependency scan      │
    └──────┬──────────────────┘
           │
    ┌──────┴──────────────────┐
    │  DEPLOY: STAGING        │
    │  - Deploy to staging    │
    │  - Run smoke tests      │
    │  - Integration tests    │
    └──────┬──────────────────┘
           │
    ┌──────┴──────────────────┐
    │  DEPLOY: PRODUCTION     │
    │  - Blue/Green swap OR   │
    │  - Canary release OR    │
    │  - Rolling update       │
    └─────────────────────────┘
```

---

## 9. Azure Architecture Patterns

### Pattern 1: 3-Tier Web Application
```
Internet
   │
   ▼
[DDoS Protection]
   │
   ▼
[Azure Front Door + WAF]     ← Global, SSL, CDN
   │
   ▼
[Application Gateway v2]     ← Regional L7 LB
   │
   ▼
[Web Tier - App Service]     ← Frontend (10.0.1.0/24)
   │  NSG: only App GW allowed in
   ▼
[App Tier - APIs + APIM]     ← Business logic (10.0.2.0/24)
   │  NSG: only Web tier allowed in
   ▼
[Data Tier - SQL + Redis]    ← Data (10.0.3.0/24)
   │  NSG: only App tier allowed in
   │  Private Endpoint only, no internet

Supporting:
  Key Vault ← secrets, no hard-coded passwords
  Azure Monitor ← logs, metrics, alerts
  Azure Firewall ← east-west traffic control
```

### Pattern 2: Microservices on AKS
```
Internet
   │
   ▼
[Azure Front Door]
   │
   ▼
[AKS Cluster]
   │  ┌────────────────────────────────┐
   │  │  Namespace: payments           │
   │  │  ┌──────────┐ ┌────────────┐  │
   │  │  │ Payment  │ │    Auth    │  │
   │  │  │ Service  │ │  Service   │  │
   │  │  └──────────┘ └────────────┘  │
   │  │  Istio mTLS between all pods   │
   │  └────────────────────────────────┘
   │
   ├── [Azure Container Registry]   ← images
   ├── [Key Vault CSI driver]        ← secrets
   ├── [Service Bus]                 ← async messaging
   └── [Cosmos DB]                   ← data (Private EP)

DevOps:
  ArgoCD → GitOps, auto-deploy from Git
  Prometheus + Grafana → monitoring
  Workload Identity → no stored credentials
```

### Pattern 3: Hybrid (On-Premises + Azure)
```
On-Premises DC                Azure
┌─────────────┐               ┌─────────────────┐
│ Core Banking│               │  Hub VNet        │
│ (Finacle)   │               │  - Firewall      │
│             │──ExpressRoute─│  - Bastion       │
│ Mainframe   │  (Private,    │  - DNS Resolver  │
│ SQL Server  │   1 Gbps)     └────────┬─────────┘
└─────────────┘                        │ VNet Peering
                               ┌────────┴─────────┐
                               │  Spoke VNet       │
                               │  - APIM           │
                               │  - API Apps       │
                               │  - Azure SQL      │
                               └──────────────────┘

Key: ExpressRoute = private dedicated line (not internet!)
     Never use public internet for core banking data
```

### Pattern 4: Multi-Region High Availability
```
Users worldwide
      │
      ▼
[Azure Traffic Manager]  ← DNS-based, routes to nearest healthy region
      │
      ├──────────────────────────┐
      ▼                          ▼
[East US 2 - Primary]    [West Europe - Secondary]
  App Service (3 AZs)      App Service (3 AZs)
  SQL Primary               SQL Geo-Replica
  Redis Primary             Redis Geo-Replica
  Cosmos Write              Cosmos Write (multi-write)
      │                          │
      └──── Async replication ───┘
                RPO: ~5 seconds
                RTO: < 1 hour (auto-failover)
```

---

## 10. Azure Interview Q&A

**Q: What is the difference between NSG and Azure Firewall?**
> NSG = simple allow/deny rules at subnet/NIC level, free, stateful
> Azure Firewall = fully managed, FQDN filtering, IDPS, TLS inspection, centralized, costs money
> Use BOTH: NSG for micro-segmentation, Firewall for centralized control

**Q: What is a Private Endpoint?**
> A Private Endpoint gives a PaaS service (like Azure SQL) a private IP address inside your VNet.
> Traffic never leaves Microsoft's network. Prevents data exfiltration.
> Example: Azure SQL with Private Endpoint → accessible only from inside your VNet

**Q: Explain Managed Identity**
> System-assigned Managed Identity = Azure creates an identity for your resource (App Service, VM).
> App authenticates to Key Vault, SQL, Storage using this identity.
> No passwords stored anywhere. Azure handles token rotation automatically.

**Q: What is the difference between Azure Front Door and Application Gateway?**
> Front Door = Global (anycast), CDN, multi-region failover, WAF globally
> App Gateway = Regional, Layer-7, SSL termination, session affinity
> Often used TOGETHER: Front Door globally routes, App Gateway handles regional LB

**Q: How do you implement zero-downtime deployments in Azure?**
> Option 1: Deployment Slots (App Service) — deploy to staging, then swap
> Option 2: Blue/Green — run two identical environments, switch traffic
> Option 3: Canary — send 5% of traffic to new version, monitor, increase gradually
> Option 4: Rolling — update instances one by one

**Q: What is Azure Policy and why use it?**
> Azure Policy enforces rules on resources at scale
> Example policies: "All VMs must use managed disks", "No public IP on databases", "All resources must have tags"
> Assign at Management Group → applies to all subscriptions below

---

*Next: Read `02-aws/aws-reference.md` for AWS equivalents*
