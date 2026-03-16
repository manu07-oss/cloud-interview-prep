# 🟠 AWS — Complete Reference for Interviews

---

## 1. AWS Global Infrastructure

```
AWS Region (e.g. ap-south-1 = Mumbai)
  ├── Availability Zone: ap-south-1a
  ├── Availability Zone: ap-south-1b
  └── Availability Zone: ap-south-1c

AWS has 33 regions, 105 AZs worldwide
India: Mumbai (ap-south-1), Hyderabad (ap-south-2)
```

---

## 2. AWS Networking — Core Services

### VPC (Virtual Private Cloud)
```
Your private network in AWS.

Example VPC for a FinTech app:
┌─────────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16                              │
│                                                  │
│  PUBLIC SUBNETS (have route to Internet GW)     │
│  ┌─────────────────┐  ┌─────────────────┐      │
│  │ Public-1a       │  │ Public-1b       │      │
│  │ 10.0.1.0/24     │  │ 10.0.2.0/24     │      │
│  │ (Load Balancer) │  │ (Load Balancer) │      │
│  └─────────────────┘  └─────────────────┘      │
│                                                  │
│  PRIVATE SUBNETS (no internet route)            │
│  ┌─────────────────┐  ┌─────────────────┐      │
│  │ Private-1a      │  │ Private-1b      │      │
│  │ 10.0.10.0/24    │  │ 10.0.11.0/24    │      │
│  │ (EC2, ECS)      │  │ (EC2, ECS)      │      │
│  └─────────────────┘  └─────────────────┘      │
│                                                  │
│  DATABASE SUBNETS                               │
│  ┌─────────────────┐  ┌─────────────────┐      │
│  │ DB-1a           │  │ DB-1b           │      │
│  │ 10.0.20.0/24    │  │ 10.0.21.0/24    │      │
│  │ (RDS, ElastiC.) │  │ (RDS, ElastiC.) │      │
│  └─────────────────┘  └─────────────────┘      │
└─────────────────────────────────────────────────┘
```

### Security Groups vs NACLs
```
Security Group (SG):                Network ACL (NACL):
- Attached to EC2/RDS/etc           - Attached to subnet
- Stateful (return traffic auto)    - Stateless (must allow both ways)
- Allow rules only                  - Allow AND deny rules
- Evaluated all rules               - Evaluated in order (rule number)
- Instance-level firewall           - Subnet-level firewall

USE: Security Groups for most cases
     NACLs for subnet-wide deny rules (e.g. block IP range)
```

### Internet Gateway vs NAT Gateway
```
Internet Gateway (IGW):
  - Allows public subnets to reach internet
  - Resources need a PUBLIC IP
  - Bidirectional (in and out)

NAT Gateway:
  - Allows PRIVATE subnets to reach internet (outbound only)
  - Private resources keep private IPs
  - No inbound connections from internet
  - Place in PUBLIC subnet, route private subnet traffic through it

Flow for private EC2 to reach internet:
Private EC2 → Private Subnet Route Table → NAT Gateway → IGW → Internet
```

---

## 3. AWS Compute Services

| Service | Use Case | Azure Equivalent |
|---------|----------|-----------------|
| **EC2** | Virtual machines | Azure VMs |
| **EC2 Auto Scaling** | Auto-scale VMs | VM Scale Sets |
| **Elastic Beanstalk** | PaaS for web apps | Azure App Service |
| **ECS (Fargate)** | Serverless containers | Azure Container Apps |
| **EKS** | Managed Kubernetes | AKS |
| **Lambda** | Serverless functions | Azure Functions |
| **AWS Batch** | Batch processing | Azure Batch |
| **Lightsail** | Simple VPS | - |

### EC2 Instance Types (Know These Categories)
```
General Purpose:    t3, t4g, m6i    → balanced CPU/memory
Compute Optimized:  c6i, c7g        → high CPU (web servers, HPC)
Memory Optimized:   r6i, x2idn      → high RAM (databases, caches)
Storage Optimized:  i3, d3          → high disk I/O
GPU:                p4, g5          → ML training, graphics

Pricing Models:
  On-Demand     → Pay per hour, no commitment (most expensive)
  Reserved      → 1 or 3 year commitment (up to 72% cheaper)
  Spot          → Unused capacity, can be terminated (up to 90% cheaper)
  Savings Plans → Flexible commitment (like Reserved but more flexible)
```

---

## 4. AWS Storage Services

| Service | Type | Use Case | Azure Equivalent |
|---------|------|----------|-----------------|
| **S3** | Object storage | Files, backups, static sites | Blob Storage |
| **EBS** | Block storage | EC2 disks | Azure Managed Disks |
| **EFS** | File storage (NFS) | Shared file system | Azure Files |
| **FSx** | Managed file systems | Windows/Lustre/NetApp | Azure Files Premium |
| **S3 Glacier** | Archive storage | Long-term backup | Azure Archive |
| **Storage Gateway** | Hybrid storage | On-prem to S3 bridge | Azure File Sync |

### S3 Storage Classes
```
Standard           → Frequently accessed → most expensive
Intelligent-Tiering→ Auto-moves between tiers based on access
Standard-IA        → Infrequent access → cheaper, retrieval fee
One Zone-IA        → Single AZ, 20% cheaper, less resilient
Glacier Instant    → Archive, millisecond retrieval
Glacier Flexible   → Archive, minutes/hours retrieval
Glacier Deep       → Archive, up to 12 hours retrieval (cheapest)

Lifecycle policies → auto-transition objects between classes
```

---

## 5. AWS Database Services

| Service | Type | Use Case | Azure Equivalent |
|---------|------|----------|-----------------|
| **RDS** | Relational (MySQL/PG/SQL) | OLTP apps | Azure SQL / PostgreSQL |
| **Aurora** | MySQL/PostgreSQL compatible | High performance OLTP | Azure SQL Hyperscale |
| **DynamoDB** | NoSQL key-value | Serverless, global scale | Cosmos DB |
| **ElastiCache** | In-memory (Redis/Memcached) | Caching, sessions | Azure Cache for Redis |
| **Redshift** | Data warehouse | Analytics, BI | Azure Synapse |
| **DocumentDB** | MongoDB compatible | Document store | Cosmos DB (MongoDB API) |
| **Neptune** | Graph database | Social networks, fraud | Cosmos DB (Gremlin) |
| **Timestream** | Time-series | IoT, metrics | - |

### RDS Multi-AZ vs Read Replicas
```
Multi-AZ:
  - Primary + Standby in different AZ
  - Synchronous replication
  - Automatic failover (~60-120 seconds)
  - For HIGH AVAILABILITY (not performance)
  - Standby can't be read by apps

Read Replicas:
  - Copy of primary for read traffic
  - Asynchronous replication (small lag)
  - Up to 5 replicas (15 for Aurora)
  - For PERFORMANCE (offload reads)
  - Can be in same region or cross-region

Use BOTH together for HA + performance
```

---

## 6. AWS Security Services

| Service | Purpose | Azure Equivalent |
|---------|---------|-----------------|
| **IAM** | Identity & access management | Azure AD / Entra ID |
| **KMS** | Key management | Azure Key Vault |
| **Secrets Manager** | Store secrets (with rotation) | Azure Key Vault |
| **WAF** | Web application firewall | Azure WAF |
| **Shield** | DDoS protection | Azure DDoS Protection |
| **GuardDuty** | Threat detection | Defender for Cloud |
| **Security Hub** | Security posture | Defender for Cloud |
| **Config** | Resource configuration compliance | Azure Policy |
| **CloudTrail** | API audit logs | Azure Activity Log |
| **VPC Endpoints** | Private access to AWS services | Azure Private Link |

### IAM Key Concepts
```
User     → Individual person (avoid for apps!)
Group    → Collection of users (assign policies to groups)
Role     → Identity for AWS services/apps (use instead of user for apps)
Policy   → JSON document defining permissions

Best Practices:
  ✓ Never use root account (create admin user instead)
  ✓ Enable MFA on all users
  ✓ Use Roles for EC2/Lambda (not access keys)
  ✓ Least privilege (only required permissions)
  ✓ Rotate access keys regularly
  ✓ Use AWS Organizations for multi-account setup
```

---

## 7. AWS Load Balancing

| Service | Layer | Use Case |
|---------|-------|---------|
| **ALB (Application LB)** | L7 HTTP/HTTPS | Web apps, microservices, path-based routing |
| **NLB (Network LB)** | L4 TCP/UDP | High performance, static IP needed, gaming |
| **CLB (Classic LB)** | L4/L7 | Legacy, avoid for new apps |
| **CloudFront** | Global CDN + L7 | Static content, global distribution |
| **Route 53** | DNS | Global traffic routing, health checks |
| **Global Accelerator** | Network L4 | Static anycast IP, global performance |

```
Pattern for global web app:
Users → Route 53 (DNS routing) → CloudFront (CDN + WAF)
     → ALB → EC2/ECS/Lambda

Pattern for global FinTech API:
Users → Route 53 → Global Accelerator → NLB → EC2 (low latency)
```

---

## 8. AWS Architecture Patterns

### Pattern 1: 3-Tier Web App on AWS
```
Internet
   │
   ▼
[Route 53]                    ← DNS, health routing
   │
   ▼
[CloudFront + WAF]            ← CDN, DDoS, WAF
   │
   ▼
[ALB - Application LB]        ← in Public Subnets
   │
   ▼
[EC2 Auto Scaling Group]      ← in Private Subnets
   │  Security Group: only ALB allowed in
   ▼
[RDS Aurora Multi-AZ]         ← in DB Subnets
   │  Security Group: only App tier allowed in
   │  No public accessibility

Supporting:
  AWS Secrets Manager ← connection strings
  CloudWatch ← logs, metrics, alarms
  AWS Config + Security Hub ← compliance
  AWS Backup ← automated backups
```

### Pattern 2: Microservices on EKS
```
Internet
   │
   ▼
[CloudFront]
   │
   ▼
[ALB Ingress Controller in EKS]
   │
   ├── [Payment Service Pod] ── [RDS Aurora]
   ├── [Auth Service Pod]    ── [ElastiCache Redis]
   └── [Order Service Pod]   ── [DynamoDB]
           │
           └── [SQS Queue] → [Lambda / Consumer]

Supporting:
  ECR ← container images
  AWS Secrets Manager + CSI driver ← secrets in pods
  IRSA (IAM Roles for Service Accounts) ← pod-level IAM
  X-Ray ← distributed tracing
  CloudWatch Container Insights ← monitoring
```

### Pattern 3: Serverless Architecture
```
Users
  │
  ▼
[API Gateway]           ← Managed API endpoint, auth, throttling
  │
  ▼
[Lambda Functions]      ← Stateless business logic, auto-scale
  │
  ├── [DynamoDB]        ← NoSQL data store
  ├── [S3]              ← File storage
  ├── [SQS]             ← Async processing queue
  └── [SNS]             ← Notifications fan-out

Event Sources for Lambda:
  S3 upload → Lambda (process image)
  DynamoDB stream → Lambda (react to data change)
  SQS message → Lambda (process queue item)
  API Gateway → Lambda (HTTP request)
  EventBridge → Lambda (scheduled or event-driven)

Cost: Pay only when Lambda runs (no idle cost)
```

### Pattern 4: Multi-Region Active/Passive
```
Users
  │
  ▼
[Route 53 - Failover routing]
  │
  ├── PRIMARY: us-east-1 (Active)
  │     ALB → EC2 ASG → RDS Primary
  │           ↓ replication
  └── SECONDARY: eu-west-1 (Passive)
        ALB → EC2 ASG → RDS Read Replica
              (promoted to primary on failover)

Route 53 health check → if primary fails, DNS switches to secondary
RTO: ~5-10 minutes (DNS TTL + health check interval)
RPO: seconds (RDS replication lag)
```

---

## 9. AWS DevOps Services

| Service | Purpose | Azure Equivalent |
|---------|---------|-----------------|
| **CodeCommit** | Git repository | Azure Repos |
| **CodeBuild** | Build service | Azure Pipelines Build |
| **CodeDeploy** | Deployment automation | Azure Pipelines Release |
| **CodePipeline** | CI/CD orchestration | Azure Pipelines |
| **ECR** | Container registry | Azure Container Registry |
| **CloudFormation** | IaC (AWS native) | Bicep / ARM Templates |
| **CDK** | IaC in code (TypeScript/Python) | Pulumi |
| **Systems Manager** | Server management, Patch | Azure Automation |
| **CloudWatch** | Monitoring and logs | Azure Monitor |

---

## 10. AWS Interview Q&A

**Q: What is the difference between S3 and EBS?**
> S3 = Object storage, accessed via HTTP, infinitely scalable, multi-AZ, used for files/backups/static sites
> EBS = Block storage, attached to ONE EC2 instance (usually), like a hard drive, used for OS and databases

**Q: What is VPC peering and its limitations?**
> VPC peering connects two VPCs so they can communicate privately.
> Limitations: NOT transitive (if A peers B, and B peers C, A cannot reach C via B), no overlapping CIDR ranges allowed.
> Alternative: AWS Transit Gateway (allows transitive routing, like a hub)

**Q: Difference between SQS and SNS?**
> SQS = Queue. One consumer reads each message. Message stays until consumed. For decoupled async processing.
> SNS = Topic. Fan-out to multiple subscribers simultaneously. Push-based. For notifications to many receivers.
> Use together: SNS fans out to multiple SQS queues (fan-out pattern)

**Q: What is IAM role vs IAM user?**
> User = permanent identity for a person. Has long-term credentials (password, access key).
> Role = temporary identity assumed by services or users. No long-term credentials. Tokens expire.
> Use Roles for EC2, Lambda, ECS tasks — never store access keys in code!

**Q: What is AWS PrivateLink?**
> Allows private connectivity to AWS services or your own services without traffic going over internet.
> Traffic stays within AWS network. Uses VPC Endpoint powered by PrivateLink.
> Example: Access S3 from EC2 without going through internet using VPC Endpoint.

**Q: Explain auto-scaling types in AWS**
> Target Tracking: maintain a metric (e.g. CPU at 70%) — simplest
> Step Scaling: scale by different amounts at different thresholds
> Scheduled: scale at specific times (e.g. scale up at 9am every weekday)
> Predictive: ML-based, predicts future load and scales proactively

---

*Next: Read `03-gcp/gcp-reference.md` for GCP services*
