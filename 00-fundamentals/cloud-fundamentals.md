# ☁️ Cloud Fundamentals — Every Engineer Must Know

---

## 1. Cloud Service Models

```
┌─────────────────────────────────────────────────────┐
│                    SaaS                             │
│         (Gmail, Salesforce, Office 365)             │
├─────────────────────────────────────────────────────┤
│                    PaaS                             │
│     (Azure App Service, AWS Elastic Beanstalk)      │
├─────────────────────────────────────────────────────┤
│                    IaaS                             │
│         (Azure VMs, AWS EC2, GCP Compute)           │
├─────────────────────────────────────────────────────┤
│              On-Premises (You manage ALL)           │
└─────────────────────────────────────────────────────┘

You manage MORE ↑ in IaaS, LESS ↑ in SaaS
Cloud manages MORE ↑ in SaaS, LESS ↑ in IaaS
```

### What you manage vs what cloud manages:

| Layer | On-Prem | IaaS | PaaS | SaaS |
|-------|---------|------|------|------|
| Application | You | You | You | Cloud |
| Runtime | You | You | Cloud | Cloud |
| OS | You | You | Cloud | Cloud |
| Virtualization | You | Cloud | Cloud | Cloud |
| Hardware | You | Cloud | Cloud | Cloud |

---

## 2. Cloud Deployment Models

| Model | Description | Example Use Case |
|-------|-------------|-----------------|
| **Public Cloud** | Resources shared, owned by provider | Startups, SaaS products |
| **Private Cloud** | Dedicated resources, owned by you | Banks, Government |
| **Hybrid Cloud** | Mix of on-prem + public cloud | Enterprise core banking |
| **Multi-Cloud** | Multiple cloud providers | Avoid vendor lock-in |

---

## 3. Core Networking Concepts (All Clouds)

### Virtual Network (VNet/VPC)
- Your private network INSIDE the cloud
- Isolated from other customers
- You define the IP address range (CIDR)

```
Your VNet: 10.0.0.0/16  (65,536 IP addresses)
  ├── Subnet 1: 10.0.1.0/24  (256 IPs) ← Web tier
  ├── Subnet 2: 10.0.2.0/24  (256 IPs) ← App tier
  └── Subnet 3: 10.0.3.0/24  (256 IPs) ← Database tier
```

### CIDR Notation (You WILL be asked this)
```
10.0.0.0/8   = 16 million IPs  (huge network)
10.0.0.0/16  = 65,536 IPs      (large — typical VNet)
10.0.0.0/24  = 256 IPs         (small — typical subnet)
10.0.0.0/32  = 1 IP            (single host)

Rule: /8 < /16 < /24 < /32
Smaller number = MORE IPs = BIGGER network
```

### Subnets
- Subdivisions of your VNet
- Each subnet lives in ONE availability zone (usually)
- Resources in same subnet can talk freely
- Between subnets → firewall rules (NSG/Security Groups) apply

### Public vs Private Subnet
```
PUBLIC subnet  → Has route to Internet Gateway → Internet-facing
PRIVATE subnet → No internet route → Internal only (databases, APIs)

RULE: Databases should NEVER be in a public subnet!
```

---

## 4. Key Networking Components

| Concept | Azure Name | AWS Name | GCP Name |
|---------|-----------|----------|----------|
| Virtual Network | VNet | VPC | VPC |
| Subnet | Subnet | Subnet | Subnet |
| Firewall rules | NSG | Security Group | Firewall Rules |
| Load Balancer (L4) | Azure LB | NLB | Cloud LB |
| Load Balancer (L7) | App Gateway | ALB | Cloud LB (HTTP) |
| Global LB + CDN | Front Door | CloudFront + ALB | Cloud Armor + LB |
| Private connection | Private Link | PrivateLink | Private Service Connect |
| VPN to on-prem | VPN Gateway | Site-to-Site VPN | Cloud VPN |
| Dedicated line | ExpressRoute | Direct Connect | Cloud Interconnect |
| NAT for outbound | NAT Gateway | NAT Gateway | Cloud NAT |
| DNS | Azure DNS | Route 53 | Cloud DNS |

---

## 5. Availability Concepts (Critical for Interviews)

### Regions, Availability Zones, Data Centers
```
Region (e.g. East US)
  ├── Availability Zone 1 (AZ1) ← separate data center
  ├── Availability Zone 2 (AZ2) ← separate data center
  └── Availability Zone 3 (AZ3) ← separate data center

AZs are connected by low-latency fiber
If AZ1 fails → AZ2 and AZ3 still run
If Region fails → you need another region (disaster recovery)
```

### SLA / Uptime Numbers (Memorize These!)
```
99%     = 87.6 hours downtime/year    (NOT acceptable for prod)
99.9%   = 8.76 hours downtime/year    (basic production)
99.95%  = 4.38 hours downtime/year    (good production)
99.99%  = 52.6 minutes downtime/year  (high availability)
99.999% = 5.26 minutes downtime/year  (five nines — very rare)
```

### RTO vs RPO (Disaster Recovery)
```
RPO = Recovery Point Objective
    → How much DATA can you lose?
    → "Our RPO is 1 hour" = we can lose max 1 hour of data
    → Achieved by: backups, replication frequency

RTO = Recovery Time Objective
    → How long can system be DOWN?
    → "Our RTO is 4 hours" = system back up within 4 hours
    → Achieved by: standby systems, automation, runbooks

RULE: Lower RPO/RTO = more expensive to build
```

---

## 6. Security Fundamentals

### Defense in Depth (Layers of Security)
```
Layer 1: Physical security (data center)
Layer 2: Network security (firewall, DDoS)
Layer 3: Perimeter (WAF, Front Door/CloudFront)
Layer 4: Network controls (NSG, Security Groups)
Layer 5: Compute (patching, antivirus)
Layer 6: Application (secure code, OWASP)
Layer 7: Data (encryption at rest + in transit)
Layer 8: Identity (MFA, least privilege, IAM)
```

### Encryption
```
At Rest   → Data stored on disk is encrypted (AES-256)
In Transit → Data moving over network is encrypted (TLS 1.2+)
In Use    → Data being processed (Confidential Computing — advanced)

Key Management:
  Azure → Key Vault (HSM)
  AWS   → KMS (Key Management Service)
  GCP   → Cloud KMS
```

### IAM Concepts (All Clouds)
```
Authentication = Who are you? (login, MFA)
Authorization  = What can you do? (permissions, roles)

Least Privilege = Give MINIMUM permissions needed
Zero Trust     = "Never trust, always verify" — even internal traffic

Service Identity:
  Azure → Managed Identity
  AWS   → IAM Role (for EC2/Lambda)
  GCP   → Service Account
```

---

## 7. Containers & Kubernetes Basics

### Docker → Container
```
Code + Dependencies + Runtime = Container Image
Container = Running instance of an image
"Works on my machine" problem → SOLVED by containers
```

### Kubernetes (K8s) Key Terms
```
Cluster     → Group of machines running K8s
Node        → A single machine in the cluster
Pod         → Smallest unit — 1 or more containers
Deployment  → Manages replicas of your pod
Service     → Network endpoint to reach your pods
Ingress     → HTTP/HTTPS routing into the cluster
Namespace   → Logical isolation within a cluster
ConfigMap   → Non-secret configuration
Secret      → Sensitive config (passwords, keys)
HPA         → Horizontal Pod Autoscaler (scale by CPU/memory)
```

### Managed Kubernetes
```
Azure → AKS  (Azure Kubernetes Service)
AWS   → EKS  (Elastic Kubernetes Service)
GCP   → GKE  (Google Kubernetes Engine)
```

---

## 8. Infrastructure as Code (IaC)

| Tool | Language | Works With | Best For |
|------|----------|-----------|---------|
| **Terraform** | HCL | All clouds | Multi-cloud IaC |
| **Bicep** | Bicep DSL | Azure only | Azure native |
| **CloudFormation** | YAML/JSON | AWS only | AWS native |
| **Pulumi** | Python/TS/Go | All clouds | Developers |
| **Ansible** | YAML | Any server | Configuration mgmt |
| **Helm** | YAML templates | Kubernetes | K8s app packaging |

### Terraform Workflow
```
terraform init     ← Download providers
terraform plan     ← Preview what will change
terraform apply    ← Create/update resources
terraform destroy  ← Delete all resources

State file (.tfstate) → tracks what Terraform created
Store state in:  Azure Blob / AWS S3 / Terraform Cloud
```

---

## 9. GitOps Concepts

```
Traditional:   Engineer runs commands → changes infra
GitOps:        Push to Git → pipeline → infra updates automatically

Git = Single source of truth for ALL infrastructure

Tools:
  ArgoCD    → GitOps for Kubernetes
  Flux      → GitOps for Kubernetes (CNCF official)
  Atlantis  → GitOps for Terraform PRs
```

---

## 10. Common Interview Architecture Questions

> These come up in 90% of cloud/DevOps interviews:

1. **"How would you make this application highly available?"**
   → Multi-AZ deployment, load balancer, health checks, auto-scaling

2. **"How would you secure this architecture?"**
   → No public IPs on databases, private subnets, WAF, encryption, IAM least privilege

3. **"How would you reduce costs?"**
   → Right-sizing, reserved instances, spot/preemptible VMs, auto-scaling down

4. **"How would you handle a region outage?"**
   → Multi-region active/passive or active/active, traffic manager, geo-replication, tested DR plan

5. **"Walk me through your CI/CD pipeline"**
   → Code commit → build → unit test → security scan → deploy to staging → smoke test → deploy to prod

---

*Next: Read `01-azure/` or `02-aws/` or `03-gcp/` for cloud-specific details*
