# ☁️ Azure vs AWS vs GCP — Side-by-Side Comparison

> This is the most important file for interviews. Companies ask: "How does X in AWS compare to Azure?"

---

## 1. Core Service Equivalents

| Category | Azure | AWS | GCP |
|----------|-------|-----|-----|
| **Virtual Network** | VNet | VPC | VPC (global) |
| **VM** | Virtual Machine | EC2 | Compute Engine |
| **VM Auto-scale** | VM Scale Sets | Auto Scaling Group | Managed Instance Group |
| **Managed K8s** | AKS | EKS | GKE |
| **Serverless Container** | Container Apps | Fargate / App Runner | Cloud Run |
| **Serverless Function** | Azure Functions | Lambda | Cloud Functions |
| **PaaS Web App** | App Service | Elastic Beanstalk | App Engine |
| **Object Storage** | Blob Storage | S3 | Cloud Storage |
| **Block Storage** | Managed Disks | EBS | Persistent Disk |
| **File Storage** | Azure Files | EFS | Filestore |
| **Relational DB** | Azure SQL | RDS | Cloud SQL |
| **PostgreSQL** | Azure DB for PG | RDS PostgreSQL / Aurora | AlloyDB / Cloud SQL PG |
| **NoSQL** | Cosmos DB | DynamoDB | Firestore / Bigtable |
| **In-memory Cache** | Azure Redis Cache | ElastiCache | Memorystore |
| **Data Warehouse** | Synapse Analytics | Redshift | BigQuery |
| **Global DB** | Cosmos DB | Aurora Global | Cloud Spanner |
| **Identity** | Azure AD (Entra ID) | IAM + Cognito | Cloud IAM |
| **Secret Store** | Key Vault | Secrets Manager + KMS | Secret Manager + Cloud KMS |
| **WAF** | Azure WAF | AWS WAF | Cloud Armor |
| **DDoS** | Azure DDoS Protection | AWS Shield | Cloud Armor |
| **CDN + Global LB** | Azure Front Door | CloudFront + Global Accelerator | Cloud CDN + Global LB |
| **Regional LB (L7)** | Application Gateway | ALB | Cloud Load Balancing |
| **DNS** | Azure DNS | Route 53 | Cloud DNS |
| **Private Connectivity** | Private Link | PrivateLink + VPC Endpoints | Private Service Connect |
| **Dedicated Network** | ExpressRoute | Direct Connect | Cloud Interconnect |
| **VPN** | VPN Gateway | Site-to-Site VPN | Cloud VPN |
| **Container Registry** | Azure Container Registry | ECR | Artifact Registry |
| **CI/CD** | Azure Pipelines | CodePipeline | Cloud Build + Cloud Deploy |
| **IaC Native** | Bicep / ARM | CloudFormation / CDK | Deployment Manager / Config Connector |
| **IaC Multi-cloud** | Terraform (works on all) | Terraform | Terraform |
| **Monitoring** | Azure Monitor | CloudWatch | Cloud Monitoring |
| **Logging** | Log Analytics | CloudWatch Logs | Cloud Logging |
| **Distributed Tracing** | Application Insights | X-Ray | Cloud Trace |
| **SIEM** | Microsoft Sentinel | Security Hub + Detective | Security Command Center |
| **Message Queue** | Service Bus / Storage Queue | SQS | Pub/Sub |
| **Event Streaming** | Event Hubs | Kinesis | Pub/Sub |
| **Notification Fan-out** | Event Grid | SNS | Pub/Sub |
| **API Management** | APIM | API Gateway | Apigee / Cloud Endpoints |
| **Service Mesh** | Open Service Mesh / Istio on AKS | App Mesh / Istio on EKS | Anthos Service Mesh / Istio on GKE |

---

## 2. Networking Deep Comparison

### VPC / VNet Scope
```
AWS VPC:   REGIONAL — one VPC per region, subnets in AZs
Azure VNet: REGIONAL — VNet per region, subnets span AZs
GCP VPC:   GLOBAL   — one VPC across ALL regions, subnets are regional

GCP advantage: simpler multi-region networking
AWS/Azure: need peering or transit hub for multi-region
```

### Firewall / Security Groups Comparison
```
AWS:
  Security Groups → stateful, instance level, allow only
  NACLs          → stateless, subnet level, allow + deny

Azure:
  NSG            → stateful, subnet OR NIC level, allow + deny
                   (similar to combining both AWS options)

GCP:
  Firewall Rules → stateful, VPC level, use tags or service accounts
                   (more flexible targeting than AWS/Azure)
```

### Private Connectivity to Managed Services
```
All three clouds let you access managed services (DB, Storage) privately:

Azure: Private Endpoint → gives PaaS service a private IP in your VNet
AWS:   VPC Endpoint    → Interface Endpoint (PrivateLink) or Gateway Endpoint
GCP:   Private Service Connect → similar to Azure Private Link

All prevent traffic from going over internet to managed services
```

---

## 3. Kubernetes Comparison

| Feature | AKS (Azure) | EKS (AWS) | GKE (GCP) |
|---------|------------|-----------|-----------|
| **Control plane cost** | Free | $0.10/hr (~$73/mo) | Free (Standard), Free (Autopilot) |
| **Autopilot/Serverless nodes** | Yes (Virtual Nodes) | Yes (Fargate) | Yes (Autopilot — best in class) |
| **Managed node upgrades** | Yes | Managed node groups | Yes (GKE Autopilot fully auto) |
| **Workload identity** | Workload Identity (OIDC) | IRSA | Workload Identity Federation |
| **GitOps integration** | Azure Arc + GitOps | EKS anywhere | Anthos Config Management |
| **GPU support** | Yes | Yes | Yes (TPUs unique to GCP) |
| **Best known for** | Azure AD integration | AWS service integration | Most advanced K8s (inventor) |

---

## 4. Serverless Comparison

| Feature | Azure Functions | AWS Lambda | GCP Cloud Functions / Run |
|---------|----------------|-----------|--------------------------|
| **Max timeout** | 230 seconds (Consumption) | 15 minutes | 60 min (Cloud Run) |
| **Max memory** | 1.5 GB | 10 GB | 32 GB (Cloud Run) |
| **Cold start** | Moderate | Moderate | Low (Cloud Run — fast) |
| **Container support** | Yes (Container Apps) | Yes (Lambda container) | Yes (Cloud Run — best) |
| **Concurrency** | Auto | Per-function limit | Per-instance |
| **VNet integration** | Yes (Premium plan) | Yes (ENI — adds latency) | Yes (Serverless VPC connector) |

---

## 5. Database Comparison

### Relational Database
```
Feature              | Azure SQL        | AWS RDS Aurora    | GCP AlloyDB
---------------------|------------------|-------------------|------------------
Engine               | SQL Server       | MySQL/PostgreSQL   | PostgreSQL only
Multi-AZ HA          | Zone redundant   | Multi-AZ          | Multi-zone
Read replicas        | Yes              | Up to 15          | Yes
Global replication   | Active Geo-rep   | Aurora Global DB  | Cross-region replica
Max storage          | 4 TB (Gen5)      | 128 TB (Aurora)   | 64 TB
Serverless option    | Azure SQL        | Aurora Serverless | No
Unique feature       | SQL Server compat| Aurora DSQL (new) | AI-ready (pgvector)
```

### NoSQL Database
```
Feature              | Cosmos DB        | DynamoDB          | Firestore
---------------------|------------------|-------------------|------------------
Data model           | Multi-model      | Key-value         | Document
Global write         | Yes              | Global Tables     | Multi-region (new)
Consistency          | 5 levels         | Strong/Eventual   | Strong/Eventual
Auto-scaling         | Yes              | Yes (On-demand)   | Yes
Unique feature       | 5 APIs supported | Single-digit ms   | Real-time updates
Pricing              | Per RU/s         | Per RU            | Per operation
```

---

## 6. Cost Comparison Principles

```
General rule: All three clouds are similar in price for equivalent services
Differences matter in specific scenarios:

AWS advantages:
  - Largest spot/preemptible market (most options)
  - Most mature Reserved Instance options
  - S3 pricing very competitive

Azure advantages:
  - Hybrid Benefit: use existing Windows/SQL Server licenses
  - Dev/Test pricing for non-prod
  - Better Microsoft stack integration (SQL Server, .NET, Active Directory)

GCP advantages:
  - Sustained use discounts (automatic, no commitment needed)
  - Preemptible VMs cheapest (91% off)
  - BigQuery on-demand pricing (pay per query, no idle cluster)
  - Committed use discounts (no upfront payment like AWS Reserved)

IaC tool Terraform works on ALL three → same skill applies
```

---

## 7. When to Choose Which Cloud

```
Choose AZURE when:
  ✓ Company is Microsoft-heavy (.NET, SQL Server, Active Directory)
  ✓ Enterprise with existing Microsoft agreements (EA)
  ✓ Need tight Azure AD / Entra ID integration
  ✓ Hybrid cloud with Windows workloads
  ✓ Financial services in regulated markets (strong compliance)

Choose AWS when:
  ✓ Startup / greenfield (largest ecosystem, most services)
  ✓ Need widest range of managed services
  ✓ Largest community / most certifications / most tutorials
  ✓ Multi-cloud strategy with AWS as primary
  ✓ Most cloud engineers know AWS → easier hiring

Choose GCP when:
  ✓ Data analytics and ML/AI are core to the product
  ✓ Using Kubernetes heavily (GKE is most mature)
  ✓ Need BigQuery for data warehouse
  ✓ Want global networking simplicity (global VPC)
  ✓ Google Workspace integration needed
```

---

## 8. Certification Path by Role

| Role | Azure | AWS | GCP |
|------|-------|-----|-----|
| **DevOps Engineer** | AZ-400 DevOps Expert | AWS DevOps Pro | Professional Cloud DevOps |
| **Cloud Architect** | AZ-305 Solutions Architect | AWS Solutions Architect Pro | Professional Cloud Architect |
| **SRE** | AZ-400 + AZ-700 | AWS SysOps Admin + DevOps | Professional Cloud DevOps |
| **Platform Engineer** | AZ-400 + AKS certs | EKS + DevOps Pro | Professional Cloud Developer |
| **Entry Level** | AZ-900 (fundamentals) | AWS Cloud Practitioner | Cloud Digital Leader |

---

*Next: Read `05-app-architectures/` for detailed architecture patterns*
