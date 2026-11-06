# 08 · High availability, DR, edge and operations

> [Home](../README.md) · [Study plan](../00-study-plan.md) · Next: [09 Data, analytics & migration](09-data-analytics-migration.md)

| Covers task statements | Est. study time | Hands-on lab | Practice |
|---|---|---|---|
| 2.2 (primary), 2.1, 3.4, 4.4 | ~4 hrs | [Lab 08](../labs/lab-08-iac-backup-dr.md) | [D2 questions](../practice/d2-resilient.md) |

Task 2.2 ("highly available and/or fault-tolerant architectures") has the widest service list in the exam guide. This module covers it end to end: resilience vocabulary, DR strategies, the global edge (Route 53, CloudFront, Global Accelerator), deployments and IaC, observability, and operating at scale. Compute HA (ELB and Auto Scaling) is in [module 03](03-compute-containers.md), and database HA (Multi-AZ, Aurora, read replicas) is in [module 05](05-databases.md).

---

## 1. Resilience vocabulary

### 1.1 HA vs fault tolerance vs DR

| Term | Meaning | AWS example |
|---|---|---|
| **High availability** | The system stays up through component failure, with **brief, automatic recovery** acceptable | RDS Multi-AZ failover (typically 1–2 minutes), ASG replacing an unhealthy instance |
| **Fault tolerance** | **No interruption or degradation** when a component fails. Needs active redundancy *with spare capacity*. | Aurora storage (6 copies across 3 AZs), an ALB + ASG sized to lose an AZ without scaling, DynamoDB |
| **Disaster recovery** | Recovering from a **large-scale event**, such as a Region impairment, data corruption, ransomware or a bad deployment. Measured by RPO/RTO. | Cross-Region backups, pilot light, Aurora Global Database |

- **RPO (recovery point objective):** how much data you can lose, measured in time back to the last good copy. It is driven by **replication and backup frequency**.
- **RTO (recovery time objective):** how long until service is restored. It is driven by **how much is already running** and how automated the failover is.
- ⚠️ Replication isn't a backup. Replicas copy corruption and deletes immediately, so logical errors need **point-in-time backups** (PITR, versioning, AWS Backup).

### 1.2 Designing out single points of failure

| Typical SPOF | Fix |
|---|---|
| One EC2 instance | ASG across ≥2 AZs behind an ALB |
| One NAT gateway | One NAT gateway per AZ, each AZ's private route table pointing at its own ([module 02](02-networking-vpc.md)) |
| Single-AZ database | RDS Multi-AZ or Aurora with replicas in other AZs |
| One VPN tunnel or one Direct Connect link | Use both VPN tunnels. For DX, add a second connection (ideally at a second location), or a VPN as backup. |
| State on the instance (sessions, uploads) | ElastiCache/DynamoDB for sessions, S3 or EFS for files |
| Hard-coded IPs | DNS names (Route 53), NLB static IPs, Global Accelerator |
| A cron job on one box | EventBridge Scheduler, or Systems Manager maintenance windows |

**Static stability**: provision so that losing an AZ needs *no* scaling actions to survive. Example: the app needs **at least 6 instances** at all times. Across 3 AZs, run **3 per AZ (9 total)**, so losing one AZ leaves 6. Across 2 AZs you'd need 12. Exam questions phrase this as "the minimum number of instances to stay available if one AZ fails".

### 1.3 Multi-AZ vs multi-Region

| | Multi-AZ | Multi-Region |
|---|---|---|
| Protects against | Instance, data center or AZ failure | Region-wide impairment, geo-regulatory needs, global latency |
| Replication | Synchronous is common (RDS Multi-AZ, Aurora storage, EFS Regional) | Usually **asynchronous** (CRR, global tables, Aurora Global Database), so RPO > 0 |
| Cost and complexity | Moderate. It is the **default for production**. | High: duplicate stacks, data transfer, failover runbooks |
| Exam signal | "Highly available", "survive an AZ failure" | "Survive a Region outage", "RPO/RTO in minutes across Regions", "users worldwide" |

---

## 2. Disaster recovery strategies

### 2.1 The four strategies

```
Cost / complexity  ────────────────────────────────────────────────────────────▶
  BACKUP & RESTORE       PILOT LIGHT              WARM STANDBY             MULTI-SITE ACTIVE/ACTIVE
  [backups only in       [data replicating live;  [whole stack running,    [full stack serving
   DR Region]             core infra defined;      scaled down; can take    production traffic
                          compute off]             traffic immediately]     in 2+ Regions]
  RPO/RTO: hours         RPO: minutes             RPO: seconds–minutes     RPO: ~0 to seconds
                         RTO: tens of minutes     RTO: minutes             RTO: ~0
◀──────────────────────────────────────────────────────── RPO and RTO get shorter
```

| | Backup & restore | Pilot light | Warm standby | Multi-site active/active |
|---|---|---|---|---|
| Running in DR Region | Nothing except backups (snapshots, AMIs, S3 copies) | **Data tier live** (cross-Region replica or global DB secondary). App tier defined but **off** (AMIs, IaC, ASG at 0). | **Every tier running at reduced capacity**, health-checked and ready | **Full capacity**, actively serving users |
| RPO | Hours (backup interval) | Minutes (replication lag) | Seconds to minutes | Near zero |
| RTO | Hours (rebuild with IaC, restore data) | Tens of minutes (start or scale compute, promote DB, switch DNS) | Minutes (scale out, promote DB, switch DNS) | Near zero (traffic shifts automatically) |
| Cost | $ | $$ | $$$ | $$$$ |
| Building blocks | AWS Backup cross-Region copy, S3 CRR, CloudFormation | RDS cross-Region read replica or Aurora Global Database, AMI copies, ASG desired 0 | Small ASG + ALB, replica DB, **Route 53 failover** | DynamoDB global tables, Aurora Global Database, Route 53 latency/weighted, Global Accelerator |

- **Pilot light vs warm standby** is the most-tested distinction. Pilot light **can't serve traffic** until you start or deploy compute. Warm standby **can serve traffic immediately**, only at lower capacity.
- Whatever the strategy, **automate failover with IaC and runbooks**, **pre-raise service quotas and reserve capacity in the DR Region** ([section 10](#10-service-quotas-and-throttling)), and **test regularly**. AWS Backup restore testing and game days both count.
- Prefer **data-plane** operations during a crisis (Route 53 health-check failover, Global Accelerator traffic dials, Route 53 ARC routing controls) over control-plane actions such as creating resources or editing DNS records, which can be impaired during regional events.

### 2.2 Cross-Region data replication by service

| Data | DR mechanism |
|---|---|
| S3 | **Cross-Region Replication** (versioning on both buckets; new objects only, so use **Batch Replication** for existing ones). **S3 Replication Time Control** gives an SLA of 99.99% of objects replicated within 15 minutes. |
| EBS / EC2 | Snapshot and AMI copies to another Region (Data Lifecycle Manager or AWS Backup) |
| RDS | **Cross-Region read replica** (promote on failover), cross-Region automated-backup replication, snapshot copy |
| Aurora | **Aurora Global Database**: storage-level replication with lag typically under 1 second, managed **switchover** (planned, no data loss) and **failover** (unplanned) to a secondary Region in about a minute |
| DynamoDB | **Global tables** (multi-active, replication typically within a second). Plus PITR and AWS Backup copies to protect against logical corruption. |
| EFS / FSx | EFS replication. AWS Backup copies. NetApp SnapMirror on FSx for ONTAP. |
| ElastiCache | Global Datastore (Valkey / Redis OSS) |
| Secrets and keys | Secrets Manager replica secrets, KMS multi-Region keys ([module 07](07-security-data-protection.md)) |
| Servers (on-premises, other clouds, EC2) | **AWS Elastic Disaster Recovery** (2.3) |

### 2.3 AWS Elastic Disaster Recovery (DRS)

- An agent performs **continuous block-level replication** of physical, virtual or cloud servers into a **low-cost staging area** in your AWS account (small replication servers and cheap EBS volumes). Full recovery instances are launched only for drills or a real failover.
- **RPO in seconds, RTO in minutes.** Point-in-time recovery helps against ransomware, and you can **fail back** to the source environment.
- Exam signal: "DR for **on-premises servers** (or EC2 in another Region) with **minimal infrastructure cost** while idle". Don't confuse it with **Application Migration Service (MGN)**, which uses similar technology for *one-way migration* ([module 09](09-data-analytics-migration.md)).

### 2.4 AWS Backup in DR

- **Backup plans** (schedule, window, lifecycle to cold storage, retention) assigned to resources **by tag**. It covers EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway, DocumentDB, Neptune, Redshift, VMware and more.
- **Cross-Region and cross-account copies** in the plan. **Organizations backup policies** enforce plans org-wide. **Vault Lock** gives immutability ([module 07](07-security-data-protection.md)). **Restore testing** validates RTO automatically. **Backup Audit Manager** reports compliance.
- It is the low-effort answer for "centralize and automate backups across services and accounts, with copies in another Region".

---

## 3. Amazon Route 53

### 3.1 Records: alias vs CNAME

Common types: A, AAAA, CNAME, MX, TXT, NS, SOA, CAA, SRV, PTR. **Alias** is a Route 53 extension.

| | CNAME | Alias (A/AAAA) |
|---|---|---|
| At the **zone apex** (`example.com`) | ❌ Not allowed | ✅ |
| Targets | Any DNS name | AWS resources: **ELB, CloudFront, API Gateway, S3 website endpoint, Global Accelerator, Elastic Beanstalk, VPC interface endpoints**, or another record in the same hosted zone |
| Query charges | Charged | **Free** for alias queries to AWS resources |
| TTL | You set it | Taken from the target |
| Health | Separate health check | **Evaluate target health** (e.g., an ALB with no healthy targets is treated as unhealthy) |
| Can't point to | | An EC2 instance's public DNS name (use an A record with its IP, or better, put an ALB in front) |

### 3.2 Routing policies

| Policy | How it answers | Health checks | Use |
|---|---|---|---|
| **Simple** | One record. With several values, all are returned in random order and the client picks. | ❌ | A single resource |
| **Weighted** | In proportion to weights (0–255; weight 0 stops traffic) | ✅ | Canary, blue/green, A/B, splitting across Regions |
| **Latency** | The Region with the lowest *measured* latency for the user's network | ✅ | Multi-Region app optimized for performance |
| **Failover** | Primary while healthy, otherwise secondary | ✅ (required on the primary) | **Active-passive DR**. The secondary can be an S3 static "maintenance" site. |
| **Geolocation** | By user location (continent, country, US state). Add a **default** record for unmatched locations. | ✅ | Compliance, localization, content rights. It is *not* for latency. |
| **Geoproximity** | By distance between user and resource, adjustable with a **bias** that grows or shrinks a resource's area | ✅ | Gradually shift traffic between Regions or locations |
| **Multivalue answer** | Up to **8 healthy** records, at random | ✅ | Simple client-side spreading with health checks. Not a replacement for ELB. |
| **IP-based** | By client source CIDR (CIDR collections you define) | ✅ | Route known ISP or customer ranges to specific endpoints |

You can **nest** policies (e.g., latency records whose targets are failover records in each Region), or build the tree visually with Traffic Flow.

### 3.3 Health checks

| Type | What it monitors | Use |
|---|---|---|
| **Endpoint** | HTTP, HTTPS or TCP checks from health checkers around the world. Interval 30 s, or 10 s ("fast"). Failure threshold is configurable (default 3). Optional string match on the response body. Healthy if more than 18% of checkers report healthy. | Public endpoints. Your SG/NACL must allow the Route 53 health-checker IP ranges. |
| **Calculated** | Combines child health checks with AND, OR, or "at least N healthy" | One status for a multi-component service |
| **CloudWatch alarm** | Follows the state of a CloudWatch alarm | **Private resources** that health checkers can't reach (e.g., in private subnets or on-premises), or any metric-based health (queue depth, error rate) |

Health checks don't send notifications by themselves. Alarm on their CloudWatch metric and send to SNS.

### 3.4 Private hosted zones, hybrid DNS, failover patterns

- **Private hosted zone:** resolves only inside the associated VPCs, which need `enableDnsSupport` and `enableDnsHostnames`. It can be associated with VPCs in other accounts (authorize the association via CLI/API). **Split-horizon DNS**: the same name in a public and a private zone gives different answers inside and outside the VPC.
- **Route 53 Resolver:** **inbound endpoints** let on-premises resolvers forward queries into AWS. **Outbound endpoints + forwarding rules** let VPCs resolve on-premises names. Share the rules across accounts with RAM. Details are in [module 02](02-networking-vpc.md).
- **Failover patterns:**
  - *Active-passive*: failover routing + health check on the primary.
  - *Active-active*: weighted or latency records, each with a health check. Unhealthy answers drop out.
  - **DNS caching delays failover.** Use low TTLs (e.g., 60 s) on records that fail over, and remember that some clients ignore TTLs. If that is unacceptable, use **Global Accelerator** (static IPs, no DNS change).
- **Route 53 Application Recovery Controller (ARC):** **zonal shift and zonal autoshift** move load balancer traffic away from an impaired AZ, and **routing controls** give a highly available switch for Region failover. Use it when the question stresses "reliable, controlled failover" for a multi-Region app.

---

## 4. Amazon CloudFront

### 4.1 Origins and origin security

| Origin | How to secure it |
|---|---|
| **S3 bucket (REST endpoint)** | **Origin Access Control (OAC)**: the bucket policy allows only the `cloudfront.amazonaws.com` service principal for your distribution's ARN. OAC supports SSE-KMS and writes. **OAI** is the legacy method. |
| S3 **static website endpoint** | Treated as a custom origin, HTTP only, and **can't use OAC**. Use a REST-endpoint origin + OAC when you need privacy. |
| ALB, NLB or EC2 with public IPs | Custom origin. Allow only the **CloudFront managed prefix list** in the SG, add a secret custom header and verify it, use HTTPS to the origin. |
| **VPC origin** | CloudFront reaches an ALB, NLB or EC2 instance in **private subnets**. No public exposure at all. |
| Any HTTP server (on-premises, other cloud), Lambda function URLs, media services | Custom origin. Lambda function URLs can also use OAC. |

### 4.2 Cache behaviours, TTLs and cache keys

- **Cache behaviours** match **path patterns** (e.g., `/api/*` → ALB with caching disabled, `/static/*` → S3 with long TTLs, `*` default). Each sets the viewer protocol policy (redirect HTTP to HTTPS), allowed methods, policies and function associations.
- **Cache policy** = TTLs (minimum, default, maximum) + **cache key** (which headers, cookies and query strings make an object unique). **Origin request policy** = extra values forwarded to the origin *without* adding them to the cache key. **Response headers policy** = CORS and security headers.
- **Cache hit ratio** is the lever for both performance and cost. Keep the cache key minimal (every extra header or cookie fragments the cache), normalize values with CloudFront Functions, use long TTLs with **versioned file names**, and add **Origin Shield** for a central caching layer in front of the origin.
- TTL logic: the origin's `Cache-Control`/`Expires` headers are honoured within the min/max bounds. The default TTL (24 hours unless changed) applies when the origin sends none.
- **Invalidations** remove objects before their TTL expires. Beyond a monthly free allowance they cost per path, and a wildcard counts as one path. Prefer versioned object names for routine releases.

### 4.3 Restricting access to content

| | Signed URL | Signed cookies |
|---|---|---|
| Scope | **One file** per URL | **Many files** (e.g., all of `/premium/*`) |
| URL changes | Yes | No, so existing URLs keep working |
| Clients | Any HTTP client, including ones that don't handle cookies | Browsers and cookie-aware clients |
| Policy options | Expiry, start time, source IP range (custom policy) | Same |

- Both are signed by your app with a private key whose public key is in a CloudFront **trusted key group**.
- An **S3 presigned URL** is different. It goes directly to S3, is signed with the signer's IAM credentials, and bypasses CloudFront.
- **Geo restriction**: allowlist or blocklist countries for the whole distribution (viewers get a 403). For finer rules, use WAF geo match.
- **Field-level encryption**: CloudFront encrypts specified POST form fields (e.g., card numbers) at the edge with your **public key**. Only the backend component holding the private key can read them, even if intermediate tiers log the request.

### 4.4 Resilience and edge compute

- **Origin failover:** an **origin group** with a primary and a secondary origin. CloudFront fails over on configured HTTP status codes (e.g., 500/502/503/504/403/404) or connection failure. It works **only for GET, HEAD and OPTIONS**, not writes. A common pair is an S3 bucket in Region A + its CRR replica in Region B.

| | CloudFront Functions | Lambda@Edge |
|---|---|---|
| Runtime | JavaScript, lightweight | Node.js, Python |
| Triggers | **Viewer request/response** only | Viewer **and origin** request/response |
| Execution | **Sub-millisecond**, massive scale, cheapest | Up to 5 s (viewer) or 30 s (origin) |
| Network/body access | ❌ No network calls, no request body | ✅ Can call other services and read the body |
| Use | Header manipulation, URL rewrites/redirects, cache-key normalization, simple token (JWT) validation, A/B cookie | Auth that calls an IdP, dynamic origin selection, content generation, image transformation at the origin |
| Deploy | Directly in CloudFront | Authored in **us-east-1**, replicated to edge locations |

- **Price classes:** All edge locations, Price Class 200, or **Price Class 100** (the cheapest regions, mainly North America and Europe). Lower classes cost less but give viewers elsewhere higher latency.
- **Regional edge caches** sit between edge locations and your origin, keeping less-popular objects longer. They are automatic and free. **Origin Shield** is an optional, paid extra layer.
- Also supported: HTTP/2 and HTTP/3, WebSockets, gRPC, TLS 1.3, and dynamic (uncacheable) content accelerated over the AWS backbone.

---

## 5. AWS Global Accelerator

- **Two static anycast IPv4 addresses** (dual-stack is optional). Users enter the AWS network at the nearest edge location, and traffic rides the **AWS backbone** to **endpoint groups** in one or more Regions. Endpoints are ALB, NLB, EC2 or Elastic IPs.
- **Health checks with fast failover** between endpoints and Regions, typically in seconds. Failover doesn't depend on DNS TTLs or client caches, because the IPs never change.
- **Traffic dials** per endpoint group (percentage of traffic to a Region) and **weights** per endpoint suit blue/green and gradual Region shifts. Client affinity is optional. Client IP is preserved for ALB and EC2 endpoints.
- **TCP and UDP**, so it serves **non-HTTP** workloads: gaming, VoIP, IoT/MQTT, and apps needing **static IPs for firewall allowlisting**. **Custom routing accelerators** map users to specific EC2 instances and ports (e.g., game sessions). Shield Standard is included.
- **No caching.** Cost is a fixed hourly fee per accelerator plus a per-GB data-transfer premium.

| | CloudFront | Global Accelerator |
|---|---|---|
| Layer and protocols | L7 HTTP(S), WebSockets, gRPC | **L4 TCP/UDP** |
| Caching | ✅ | ❌ |
| IP addresses | Many, changing (use DNS) | **2 static anycast IPs** |
| Best for | Static and dynamic web content, APIs, video, edge compute | **Non-HTTP**, static IPs, **fast multi-Region failover**, global TCP/UDP performance |
| Failover | Origin groups (GET/HEAD/OPTIONS) | Endpoint health, traffic dials, near-instant |
| Security | WAF, Shield, OAC, signed URLs/cookies | Shield. Endpoints keep their own SGs. |

---

## 6. Deployment strategies and immutable infrastructure

**Immutable infrastructure** means you never patch running servers in place. Build a new **golden AMI** (EC2 Image Builder) or container image, roll it out, and **replace** the old resources. This removes configuration drift and makes rollback a traffic switch.

| Strategy | How | Rollback | AWS tooling |
|---|---|---|---|
| All at once / in-place | Update every instance together | Redeploy the old version (downtime) | CodeDeploy in-place, Beanstalk "All at once" |
| Rolling | Update in batches | Roll back batch by batch | ASG **instance refresh**, Beanstalk "Rolling" / "Rolling with additional batch" |
| Immutable | New instances in a new ASG, then swap | Terminate the new ASG | Beanstalk "Immutable" |
| **Blue/green** | A complete new environment, then shift traffic all at once or gradually | Shift traffic back (fast) | Route 53 weighted, **ALB weighted target groups**, Beanstalk **swap environment URLs**, CodeDeploy blue/green (EC2, ECS), ECS native blue/green |
| **Canary / linear** | A small % first, then the rest (canary) or in equal steps (linear) | Automatic on CloudWatch alarms | **CodeDeploy for Lambda** (alias traffic shifting, e.g., 10% for 5 minutes), API Gateway canary releases, Lambda alias weights |

⚠️ DNS-based blue/green (Route 53 weighted) is slowed by client DNS caching. ALB weighted target groups or Global Accelerator weights shift traffic immediately.

---

## 7. Infrastructure as code and governed provisioning

### 7.1 AWS CloudFormation

| Feature | What to know |
|---|---|
| Templates and stacks | YAML/JSON: Parameters, Mappings, Conditions, Resources (required), Outputs. Use **nested stacks** for reuse, or **Exports + `Fn::ImportValue`** for cross-stack references. |
| **Change sets** | Preview exactly what an update will add, modify or **replace** before you execute it |
| **Drift detection** | Finds resources changed outside CloudFormation (console edits) |
| **StackSets** | Deploy one template to **many accounts and Regions** in a single operation. *Service-managed* permissions with Organizations can **auto-deploy to new accounts** in an OU. Concurrency and failure-tolerance settings control the rollout. |
| **DeletionPolicy** | `Delete` (the usual default), **`Retain`** (keep the resource when the stack is deleted), **`Snapshot`** (EBS, RDS, ElastiCache, Redshift and similar). RDS DB instances and clusters default to Snapshot. `UpdateReplacePolicy` does the same when an update replaces a resource. |
| Protection | **Termination protection**, **stack policies** (block updates to critical resources), rollback triggers on CloudWatch alarms |
| Bootstrapping | `cfn-init` (configure the instance), `cfn-signal` + `CreationPolicy` (wait until instances report healthy), custom resources (Lambda) for anything unsupported |
| Other | Dynamic references to Secrets Manager and Parameter Store. The IaC generator creates templates from existing resources. CloudFormation Hooks and Guard enforce policy before deployment. |

⚠️ Stack deletion fails on a **non-empty S3 bucket** unless it is retained or emptied first.

### 7.2 AWS CDK and AWS SAM

- **AWS CDK**: infrastructure defined in TypeScript, Python, Java, C# or Go using high-level constructs, which are **synthesized into CloudFormation**. Choose it for "developers want to define infrastructure in a programming language".
- **AWS SAM**: a CloudFormation transform with shorthand for serverless resources (functions, APIs, tables), plus local testing.

### 7.3 Service Catalog and Proton (briefly)

- **AWS Service Catalog**: admins publish **portfolios of approved products** (CloudFormation or Terraform templates) with **launch constraints**: an IAM role that does the provisioning, so end users don't need broad permissions. Portfolios can be shared across accounts. Exam signal: "let teams self-serve only approved, compliant stacks".
- **AWS Proton**: platform teams publish versioned environment and service templates for container and serverless apps, and developers deploy against them. It is still listed in the exam guide, but **AWS Proton reached end of support on Oct 7, 2026** (new sign-ups closed a year earlier). Know what it did in case it appears as an option, but for real projects use Service Catalog, CDK/CloudFormation with CodePipeline, or a platform like Backstage.

---

## 8. Observability

### 8.1 Amazon CloudWatch

| Feature | What to know |
|---|---|
| **Metrics** | EC2 **basic monitoring every 5 minutes**, **detailed monitoring every 1 minute** (paid). **Memory and disk usage aren't collected by default**: install the **CloudWatch agent**. Metrics are retained 15 months at decreasing resolution. |
| **Custom metrics** | `PutMetricData` or the Embedded Metric Format in logs. Standard resolution is 1 minute; **high-resolution goes down to 1 second**. Dimensions identify the series. |
| **Alarms** | States OK, ALARM, INSUFFICIENT_DATA. Actions: **SNS**, **EC2 Auto Scaling**, **EC2 actions** (stop, terminate, reboot, **recover**), Systems Manager OpsItems. Metric math and **anomaly detection** bands as thresholds. |
| **Composite alarms** | Combine alarms with AND/OR/NOT to **cut alert noise** (e.g., page only when high latency AND high error rate) |
| **Logs** | Log groups and streams. **Retention defaults to never expire**, so set it. **Metric filters** turn log patterns into metrics. **Subscription filters** stream to Lambda, Kinesis Data Streams or Firehose. Export to S3 for archives. Infrequent Access log class for cheaper storage. |
| **Logs Insights** | An interactive query language over log groups, for ad-hoc troubleshooting |
| **Agent** | The unified CloudWatch agent collects OS metrics and log files from **EC2 and on-premises servers**. Its config can be stored in Parameter Store. |
| **Dashboards** | Can be cross-account and cross-Region. Cross-account observability links monitoring accounts. |
| **Synthetics canaries** | Scheduled scripts that probe endpoints, APIs and UI flows **from outside**, catching failures before users do |
| Others | Container Insights, Lambda Insights, Application Signals (APM), RUM, Contributor Insights |

### 8.2 X-Ray, Managed Grafana, Managed Service for Prometheus, Health Dashboard

- **AWS X-Ray**: **distributed tracing** across API Gateway, Lambda, ECS, EC2 and SDK calls, giving a **service map**, latency breakdowns and errors per segment. Annotations are indexed for filtering. AWS now recommends instrumenting with **OpenTelemetry** (AWS Distro for OpenTelemetry) rather than the older X-Ray SDKs. Exam signal: "find **which microservice** adds latency".
- **Amazon Managed Grafana**: managed Grafana dashboards over many data sources (CloudWatch, Prometheus, OpenSearch, X-Ray, third-party), with SSO.
- **Amazon Managed Service for Prometheus**: Prometheus-compatible metric storage and PromQL for container (EKS/ECS) workloads, with no Prometheus servers to scale.
- **AWS Health Dashboard**: *Service health* (public status) and *Your account health*, i.e., events affecting **your** resources (scheduled maintenance, EC2 retirement, Regional issues). Health events go to **EventBridge** for automation. There is an organizational view, and the AWS Health API needs a Business-level support plan or higher.

---

## 9. AWS Systems Manager

| Capability | What it does | Exam signal |
|---|---|---|
| **Session Manager** | Browser or CLI shell and port forwarding to instances with **no SSH keys, no bastion, no inbound ports**. IAM-controlled, sessions logged to S3/CloudWatch Logs. | "Remove the bastion host", "audit shell access" |
| **Patch Manager** | Patch baselines (auto-approval rules), patch groups by tag, scheduled via **maintenance windows**, compliance reporting | "Patch fleets of EC2/on-premises servers on a schedule" |
| **Run Command** | Run scripts or commands on many nodes by tag, with rate control and error thresholds, output to S3 | "Run a command on 500 instances without SSH" |
| **Automation** | Multi-step **runbooks** (e.g., create AMI, restart, remediation), approvals, cross-account and cross-Region. **Config remediation** and EventBridge targets use it. | "Automate remediation / operational tasks" |
| **State Manager** | **Associations** that keep nodes in a desired state on a schedule (agent installed, config applied) | "Ensure a configuration stays applied" |
| **Fleet Manager** | Console UI to view and manage nodes (file system, processes, users, performance, RDP) | "Manage servers from the console" |
| **Parameter Store** | Config and secrets ([module 07](07-security-data-protection.md)) | |
| Inventory, OpsCenter | Software inventory, aggregation of operational issues | |

Requirements for managed nodes: the **SSM Agent** (preinstalled on many AMIs), **IAM permissions** (an instance profile with `AmazonSSMManagedInstanceCore`, or Default Host Management Configuration), and a **network path to the Systems Manager endpoints**. In private subnets with no NAT, create **interface VPC endpoints** for `ssm`, `ssmmessages` and `ec2messages`. On-premises servers register through **hybrid activations**.

---

## 10. Service quotas and throttling

- **Service Quotas** console/API: view quotas, request increases, alarm on usage (CloudWatch usage metrics). Organizations can apply quota request templates to new accounts. Trusted Advisor also flags quotas near their limit.
- **DR Region readiness:** quotas are **per account per Region**. **Pre-raise** EC2 vCPU quotas, Lambda concurrency and similar in the standby Region *before* you need them. To guarantee instance capacity, use **On-Demand Capacity Reservations** in specific AZs, which you can combine with Savings Plans or Regional RIs for the discount.
- **Retries:** use **exponential backoff with jitter** for throttling (`ThrottlingException`, HTTP 429, S3 `503 SlowDown`). AWS SDKs do this by default (standard and adaptive retry modes). Make operations **idempotent** (client tokens) so retries are safe.
- **API Gateway throttling:** a token bucket at the account level per Region (**10,000 rps** steady with a 5,000-request burst by default, adjustable), plus stage, route and method limits. **Usage plans + API keys** give per-client rate, burst and quotas (daily, weekly or monthly). Excess requests get **429**. API keys identify clients but don't authorize them.
- **Protect downstream systems:** buffer bursts with **SQS** ([module 06](06-decoupling-serverless.md)), cap Lambda with **reserved concurrency**, pool database connections with **RDS Proxy**, and cache with CloudFront or ElastiCache.

---

## 11. Making legacy apps reliable without code changes

| Weakness | Fix that needs no code change |
|---|---|
| Single EC2 server | Create an **AMI** → launch template → **ASG across ≥2 AZs behind an ALB**. If the app can't run as multiple copies, use an **ASG with min = max = 1** for self-healing. |
| Hardware failure of a pet instance | **EC2 automatic recovery** (CloudWatch alarm on `StatusCheckFailed_System` → recover; keeps instance ID, IPs and EBS) |
| Single-AZ database | **RDS Multi-AZ**. The endpoint DNS name stays the same after failover. Add **RDS Proxy** to speed up failover and pool connections. |
| Files on local disk shared by servers | **EFS** (NFS) mounted on every instance, or **FSx for Windows File Server** (SMB) |
| Sessions stored in memory | ALB **sticky sessions** as a stopgap, or point the framework's session store at ElastiCache or DynamoDB (configuration only) |
| Self-run ActiveMQ/RabbitMQ using JMS, AMQP, MQTT or STOMP | **Amazon MQ**: active/standby across AZs (ActiveMQ) or a 3-node cluster (RabbitMQ). It speaks the same protocols, so there is no rewrite to SQS/SNS. |
| Single Region | **Route 53 failover** to a warm-standby or pilot-light stack, with S3 CRR and RDS cross-Region replicas |
| Static assets served by the app server | Move them to S3 + CloudFront |
| Clients that need fixed IPs | NLB with Elastic IPs, or Global Accelerator |

---

## 12. AWS Well-Architected Framework

The exam guide is written through this lens. Know the **six pillars** and what each one pushes you toward:

| Pillar | Exam lens |
|---|---|
| Operational excellence | IaC, small reversible changes, observability, runbooks, learning from failures |
| Security | Least privilege, traceability, security at every layer, encryption, automated controls |
| Reliability | Automatic recovery, horizontal scaling, test recovery, stop guessing capacity, manage change through automation |
| Performance efficiency | Use managed and serverless services, go global in minutes, experiment, pick the right resource type |
| Cost optimization | Consumption model, measure efficiency, attribute spend, managed services ([module 10](10-cost-optimization.md)) |
| Sustainability | Maximize utilization, efficient hardware (e.g., Graviton), managed services, reduce downstream impact |

The **AWS Well-Architected Tool** (console, free) runs a structured review of a workload against the pillars and optional **lenses** (e.g., Serverless, SaaS). It records risks, milestones and an improvement plan.

---

## 13. How the exam tests this

| If the question says… | Think… |
|---|---|
| "RPO/RTO of hours, lowest cost" | **Backup & restore** (AWS Backup cross-Region copies + IaC) |
| "DB replicated, app servers off until disaster, RTO tens of minutes" | **Pilot light** |
| "Scaled-down copy running, RTO minutes" | **Warm standby** + Route 53 failover |
| "Near-zero RPO/RTO, users in multiple Regions" | **Multi-site active/active** (DynamoDB global tables / Aurora Global Database + Route 53 latency or Global Accelerator) |
| "DR for on-premises servers with minimal idle cost" | **Elastic Disaster Recovery** |
| "Point `example.com` at an ALB/CloudFront" | **Alias** record (CNAME is not allowed at the apex) |
| "Send users to the closest Region for performance" | **Latency** routing (or Global Accelerator) |
| "Users in a country must reach a specific endpoint" | **Geolocation** (+ default record) |
| "Gradually shift 10% of traffic to the new version" | **Weighted** routing / ALB weighted target groups / CodeDeploy canary |
| "Health-check a private resource" | Route 53 health check based on a **CloudWatch alarm** |
| "Private S3 content via CloudFront only" | **OAC** + bucket policy |
| "Premium members can access many files" | **Signed cookies**. One file → **signed URL**. |
| "Header rewrite/redirect at the edge, very high volume, lowest cost" | **CloudFront Functions**. Needs network calls or origin events → **Lambda@Edge**. |
| "UDP game traffic, static IPs, fast Regional failover" | **Global Accelerator** |
| "Same baseline stack in every account and Region" | **CloudFormation StackSets** (service-managed, Organizations) |
| "Keep the database when the stack is deleted" | `DeletionPolicy: Retain` (or `Snapshot`) |
| "Preview changes before updating production" | **Change set** |
| "Detect manual changes to stack resources" | **Drift detection** |
| "Memory utilization metrics for EC2" | **CloudWatch agent** |
| "Reduce alarm noise" | **Composite alarms** |
| "Find the slow microservice" | **X-Ray** |
| "Shell access without SSH or bastion" | **Session Manager** |
| "Patch on a schedule" | **Patch Manager** + maintenance windows |
| "Throttling errors from AWS APIs" | **Exponential backoff with jitter**; request quota increase |
| "Make a legacy app HA without code changes" | AMI → ASG + ALB, Multi-AZ RDS, EFS, Amazon MQ |

---

## 14. Common traps

- ⚠️ Trap: **Read replicas and cross-Region replicas aren't backups.** Corruption replicates too. Pair them with PITR or AWS Backup.
- ⚠️ Trap: **Pilot light** has compute *off* and **can't take traffic immediately**. Warm standby can.
- ⚠️ Trap: **CNAME at the zone apex isn't allowed.** Use an alias. Alias can't target an EC2 instance's DNS name.
- ⚠️ Trap: **Geolocation ≠ latency.** Geolocation is about *where the user is* (compliance and content). Latency is about *speed*.
- ⚠️ Trap: **Simple routing has no health checks.** Multivalue answer does, but it isn't a load balancer.
- ⚠️ Trap: Route 53 health checkers come from the **internet** and can't reach private IPs. Use CloudWatch-alarm-based checks.
- ⚠️ Trap: DNS failover is limited by **TTL and client caching**. "Failover without DNS changes" points to **Global Accelerator**.
- ⚠️ Trap: **OAC can't be used with an S3 website endpoint.** Use the REST endpoint.
- ⚠️ Trap: CloudFront **origin failover only covers GET/HEAD/OPTIONS**.
- ⚠️ Trap: **Lambda@Edge functions are created in us-east-1**, and CloudFront Functions can't make network calls.
- ⚠️ Trap: **Global Accelerator doesn't cache.** For static content, CloudFront is the answer.
- ⚠️ Trap: CloudWatch has **no memory metric for EC2** without the agent, and **log retention defaults to never expire** (a cost leak).
- ⚠️ Trap: Service quotas are **per Region**. A DR plan that doesn't pre-raise quotas in the standby Region can fail at the worst time.
- ⚠️ Trap: Session Manager in private subnets with no NAT needs **interface endpoints** (`ssm`, `ssmmessages`, `ec2messages`).
- ⚠️ Trap: Stack deletion **deletes resources by default** (except where Snapshot or Retain is the default or set). Set `DeletionPolicy` on stateful resources.

---

## 15. Quick self-check

1. Your app needs at least 4 instances to handle peak load and must survive the loss of one AZ without scaling. What's the minimum total across 3 AZs?
2. What's the practical difference between pilot light and warm standby?
3. Why would you choose Global Accelerator over Route 53 latency routing for multi-Region failover?
4. Which CloudFront feature lets an origin in a private subnet serve content, and which one restricts S3 to CloudFront only?
5. Name the Systems Manager capability for (a) shell access without SSH, (b) enforcing a desired configuration, (c) multi-step remediation runbooks.

<details>
<summary>Answers</summary>

1. **6** instances (2 per AZ). Losing one AZ leaves 4.
2. Pilot light runs only the **data tier** (and core pieces) in the DR Region, so compute must be started or deployed before it can serve traffic. Warm standby runs a **scaled-down copy of the full stack** that serves traffic immediately and is then scaled up.
3. Global Accelerator uses **static anycast IPs** and fails over at the network layer in seconds, so it **doesn't depend on DNS TTLs or client caching**. It also supports TCP/UDP non-HTTP traffic.
4. **VPC origins** for private ALB/NLB/EC2 origins. **Origin Access Control (OAC)** for S3.
5. (a) **Session Manager**, (b) **State Manager**, (c) **Automation**.

</details>

Practice: [D2 Resilient architectures questions](../practice/d2-resilient.md) · Lab: [Lab 08: IaC, backup & DR](../labs/lab-08-iac-backup-dr.md)
