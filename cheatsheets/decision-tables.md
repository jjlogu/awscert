# Decision Tables: Last-Week Revision
> [Home](../README.md) · [Study plan](../00-study-plan.md) · [Exam keywords](exam-keywords.md) · [Numbers to remember](numbers-to-remember.md)

These are 25 "which one?" tables for the final week. Each one puts options in rows and the deciding criteria in columns, ending with a **Pick when…** column. One pass takes about 30–45 minutes. For the mechanisms behind any row, follow the **Depth** link to the notes module.

How to use it: cover the **Pick when…** column and predict it from the criteria. If you can't, re-read that module section. Bold marks exam-critical items, and **⚠️ Trap:** marks the common wrong turns.

**Contents:** [a S3 classes](#a-s3-storage-classes) · [b EBS](#b-ebs-volume-types--instance-store) · [c Storage service](#c-storage-service-choice) · [d Database](#d-database-choice) · [e RDS/Aurora HA](#e-rds--aurora-availability-and-read-scaling) · [f Load balancers](#f-load-balancer-choice) · [g Compute](#g-compute-choice) · [h EC2 purchasing](#h-ec2-purchasing-options) · [i Placement groups](#i-placement-groups) · [j Auto Scaling](#j-auto-scaling-policy-types) · [k Connectivity](#k-hybrid--inter-vpc-connectivity) · [l Network security](#l-security-group-vs-nacl-vs-network-firewall-vs-waf) · [m DR](#m-dr-strategies) · [n Route 53](#n-route-53-routing-policies) · [o Edge acceleration](#o-cloudfront-vs-global-accelerator-vs-s3-transfer-acceleration) · [p Messaging](#p-messaging--streaming) · [q Step Functions](#q-step-functions-standard-vs-express) · [r API Gateway](#r-api-gateway-rest-vs-http-vs-websocket) · [s Analytics](#s-analytics) · [t Transfer/migration](#t-data-transfer--migration) · [u Security services](#u-security-services-which-one) · [v KMS/CloudHSM](#v-kms-key-types-vs-cloudhsm) · [w Secrets](#w-secrets-manager-vs-parameter-store) · [x Identity](#x-directory--identity-options) · [y Cost tools](#y-cost-tools)

---

## a. S3 storage classes

| Class | AZs | Min storage duration | Min billable size | First-byte latency | Retrieval fee | Pick when… |
|---|---|---|---|---|---|---|
| **S3 Standard** | ≥3 | — | — | ms | No | Frequently accessed data. The default for active data, websites and content. |
| **S3 Intelligent-Tiering** | ≥3 | — | Objects <128 KB stay in the frequent tier (not monitored) | ms (frequent / infrequent / archive instant tiers); hours for the opt-in archive tiers | **No** (small per-object monitoring fee) | **Access pattern unknown or changing.** "Optimize cost automatically without retrieval fees." |
| **S3 Standard-IA** | ≥3 | **30 days** | 128 KB | ms | Per GB | Accessed about monthly, needs ms access and multi-AZ resilience (backups, DR copies). |
| **S3 One Zone-IA** | **1** | **30 days** | 128 KB | ms | Per GB | Infrequent data you can **re-create** (thumbnails, transcoded media, secondary backup copy). It is lost if the AZ is destroyed. |
| **S3 Express One Zone** | 1 (you choose the AZ) | Negligible | — | **Single-digit ms** | No | Latency-critical, very high request rates co-located with compute (ML training, interactive analytics). Uses *directory buckets*. |
| **S3 Glacier Instant Retrieval** | ≥3 | **90 days** | 128 KB | **ms** | Higher per GB | Accessed about once a quarter but still needs ms access (medical images, news archives). |
| **S3 Glacier Flexible Retrieval** | ≥3 | **90 days** | — | Expedited 1–5 min · Standard 3–5 h · Bulk 5–12 h (bulk is free) | Per GB (except bulk) | Archives read 1–2 times a year where minutes-to-hours is acceptable. |
| **S3 Glacier Deep Archive** | ≥3 | **180 days** | — | Standard within 12 h · Bulk within 48 h | Per GB | **Cheapest.** 7–10 year compliance retention, tape replacement. |

- **⚠️ Trap:** Lifecycle can't move objects to Standard-IA or One Zone-IA until they've been in their current class for **30 days**. Since 2024, lifecycle by default also skips transitioning objects smaller than 128 KB.
- **⚠️ Trap:** Deleting or transitioning an object before its minimum duration is still billed for the full minimum. Short-lived data belongs in Standard or Intelligent-Tiering.
- **Unknown access pattern → Intelligent-Tiering. Known pattern → lifecycle rules.**

**Depth:** [notes/04 Storage](../notes/04-storage.md)

---

## b. EBS volume types + instance store

| Type | Media | Size | Max IOPS / vol | Max throughput / vol | Boot | Multi-Attach | Pick when… |
|---|---|---|---|---|---|---|---|
| **gp3** | SSD | 1 GiB–64 TiB | **80,000** (baseline 3,000) | **2,000 MiB/s** (baseline 125) | ✅ | ❌ | **Default for almost everything.** IOPS and throughput are set independently of size, and it's cheaper than gp2. |
| **gp2** | SSD | 1 GiB–16 TiB | 16,000 (3 IOPS/GiB, burst credits for small vols) | 250 MiB/s | ✅ | ❌ | Legacy. If the answer is "migrate gp2 → gp3" for cost, that's usually right (Elastic Volumes, no downtime). |
| **io2 Block Express** | SSD | 4 GiB–64 TiB | **256,000** | **4,000 MiB/s** | ✅ | ✅ | **Sub-ms latency**, mission-critical DBs (SAP HANA, Oracle, SQL Server), 99.999% durability, or IOPS beyond gp3. |
| **io1** | SSD | 4 GiB–16 TiB | 64,000 | 1,000 MiB/s | ✅ | ✅ | Legacy provisioned IOPS. Prefer io2 Block Express. |
| **st1** | HDD | 125 GiB–16 TiB | 500 | 500 MiB/s | ❌ | ❌ | **Large sequential throughput**: big data, log processing, data warehouse staging, Kafka on EC2. |
| **sc1** | HDD | 125 GiB–16 TiB | 250 | 250 MiB/s | ❌ | ❌ | **Cold**, rarely accessed sequential data at the lowest EBS cost per GB. |
| **Instance store** | Local NVMe / disk on the host | Fixed by instance type | Very high (instance-dependent, can be millions) | Very high | Some types only (rare) | ❌ | **Temporary** buffers, caches, scratch, or data already replicated elsewhere (Cassandra, Kafka with replication). |

- **Exam rule of thumb:** a question needing **>16,000 IOPS on one volume, or sub-ms latency** expects **io2 Block Express**. Most questions were written when gp3 topped out at 16,000. Today gp3 reaches 80,000 IOPS, so in a newer, cost-focused question that needs, say, 30,000 IOPS without sub-ms latency or Multi-Attach, gp3 can be correct. Read what the scenario actually needs.
- **⚠️ Trap:** Instance store data **survives a reboot** but is **lost on stop, hibernate, terminate or host failure**. "Must persist" rules it out.
- **⚠️ Trap:** HDD volumes (st1/sc1) **cannot be boot volumes**.
- Multi-Attach (io1/io2) works only within **one AZ**, for up to 16 Nitro instances, and needs a cluster-aware file system.

**Depth:** [notes/04 Storage](../notes/04-storage.md) · [notes/03 Compute](../notes/03-compute-containers.md)

---

## c. Storage service choice

| Service | Type / protocol | Who can mount / access | OS fit | Resilience | Pick when… |
|---|---|---|---|---|---|
| **S3** | Object, HTTPS API | Any number of clients, any network path | Any | 11 9s durability, multi-AZ (except One Zone classes) | Unstructured objects, data lakes, static sites, backups, media. "Unlimited, cheap, durable." |
| **EBS** | Block | **One instance in one AZ** (Multi-Attach io1/io2 aside) | Any | Replicated within the AZ; snapshots go to S3 | Boot volumes, databases on EC2, low-latency block storage. |
| **Instance store** | Block, ephemeral | The host only | Any | **None** | Scratch, cache, buffers. |
| **EFS** | File, **NFSv4.1** (POSIX) | Thousands of **Linux** clients across AZs; on-premises via DX/VPN; ECS, EKS, Lambda | **Linux only** | Regional (multi-AZ) or One Zone | **Shared Linux file system** that grows and shrinks automatically: CMS uploads, home dirs, shared container storage. Lifecycle to IA and Archive. |
| **FSx for Windows File Server** | File, **SMB**, NTFS, **AD-integrated**, DFS | Windows (and Linux SMB clients) | **Windows** | Single-AZ or **Multi-AZ** | **Windows file shares**, lift-and-shift Windows apps, SharePoint, home drives with AD ACLs. |
| **FSx for Lustre** | Parallel POSIX file system | Linux compute fleets | Linux | Scratch (no replication) or Persistent | **HPC, ML training, rendering, genomics.** Hundreds of GB/s, sub-ms latency, **linked to S3** (lazy-load and export). |
| **FSx for NetApp ONTAP** | File + block: **NFS, SMB, iSCSI** | Linux, Windows, macOS | Multi-protocol | Single-AZ or Multi-AZ | **Migrate on-prem NetApp**, multi-protocol access, snapshots, clones, SnapMirror, dedupe and compression. |
| **FSx for OpenZFS** | File, NFS | Linux, macOS (and Windows NFS clients) | Mostly Linux | Single-AZ or Multi-AZ | Migrate ZFS or Linux NAS, very low latency, instant snapshots and clones. |
| **S3 File Gateway** | NFS/SMB on-prem → S3 objects | On-prem servers, with a local cache | Any | Data lives in S3 | **On-prem apps write files that should land in S3** (backups to S3, then lifecycle to Glacier). |
| **FSx File Gateway** | SMB local cache for FSx for Windows | On-prem Windows clients | Windows | Data lives in FSx | ⚠️ **No longer available to new customers (since Oct 2024).** It can still appear in older questions as "low-latency on-prem access to FSx for Windows shares". |
| **Volume Gateway – Cached** | iSCSI block | On-prem servers | Any | Primary data in S3, hot data cached locally | On-prem block storage that should live mainly in AWS, with a local cache. |
| **Volume Gateway – Stored** | iSCSI block | On-prem servers | Any | Primary data on-prem, async backup to AWS as EBS snapshots | Keep the full dataset local for low latency, and back it up to AWS for DR. |
| **Tape Gateway** | iSCSI VTL | Existing backup software | Any | Virtual tapes in S3, archived to Glacier Flexible Retrieval / Deep Archive | **Replace physical tapes without changing backup software.** |

- **Shared + Linux → EFS. Shared + Windows/SMB/AD → FSx for Windows. HPC + S3 dataset → FSx for Lustre. NetApp / multi-protocol → FSx for ONTAP.**
- **⚠️ Trap:** EFS doesn't support Windows clients, and EBS can't be shared across AZs.

**Depth:** [notes/04 Storage](../notes/04-storage.md) · [notes/09 Data & migration](../notes/09-data-analytics-migration.md)

---

## d. Database choice

| Service | Data model | Scaling model | Latency | HA built in | Pick when… |
|---|---|---|---|---|---|
| **RDS** | Relational: MySQL, PostgreSQL, MariaDB, **Oracle, SQL Server**, Db2 | Vertical + read replicas; storage autoscaling | ms | Multi-AZ (opt-in) | Lift-and-shift OLTP, **commercial engines** (Oracle/SQL Server), engine-specific features. **RDS Custom** when you need OS/DB host access. |
| **Aurora** | Relational, **MySQL/PostgreSQL-compatible** | Storage auto-grows; up to **15 replicas** | ms | **6 copies across 3 AZs**, fast failover | High-performance relational with HA, global reach (Global Database), fast cloning, Backtrack (MySQL). |
| **Aurora Serverless v2** | Same as Aurora | Capacity in ACUs scales in fine steps (can scale to 0 with auto-pause) | ms | Same as Aurora | **Spiky, intermittent or unpredictable relational load**, dev/test, many tenant DBs. |
| **DynamoDB** | Key-value / document | Horizontal, virtually unlimited; on-demand or provisioned + auto scaling | **Single-digit ms**; **µs with DAX** | Multi-AZ by default; **Global Tables** (multi-Region, multi-active) | **Massive scale, serverless, key-value access**: sessions, carts, gaming, IoT, metadata. Access patterns must be known up front. |
| **ElastiCache** (Valkey, Redis OSS, Memcached) | In-memory cache / data structures | Shards + replicas (Valkey/Redis); nodes (Memcached) | **Sub-ms** | Valkey/Redis: Multi-AZ auto-failover. Memcached: none. | **Cache DB reads**, session store, leaderboards (sorted sets), rate limiting. Memcached = simple multi-threaded cache with no persistence. |
| **MemoryDB** | Durable in-memory, Valkey/Redis-compatible | Shards + replicas | **µs reads**, single-digit ms writes | **Multi-AZ transaction log (durable)** | Redis/Valkey as the **primary database**, where durability is required. |
| **Redshift** | Columnar MPP data warehouse (SQL) | Provisioned RA3 or Serverless; concurrency scaling | Seconds (analytical queries) | RA3 Multi-AZ option, snapshots | **OLAP / BI** on structured data at TB–PB scale. **Spectrum** queries S3. Zero-ETL from Aurora/RDS/DynamoDB. |
| **DocumentDB** | Document (JSON), **MongoDB-compatible** | Up to 15 replicas, Aurora-style storage | ms | Multi-AZ | **"MongoDB compatible"** managed document DB. |
| **Neptune** | **Graph** (Gremlin, openCypher, SPARQL) | Up to 15 replicas; serverless option | ms | Multi-AZ | **Relationships**: social graphs, recommendations, fraud rings, knowledge graphs. |
| **Keyspaces** | Wide-column, **Cassandra (CQL)-compatible**, serverless | Automatic | Single-digit ms | Multi-AZ | **Migrate Cassandra** without managing clusters. |
| **OpenSearch Service** | Search / log-analytics index (also vector search) | Data nodes, UltraWarm/cold tiers; serverless option | ms–s | Multi-AZ (with standby) | **Full-text search**, log analytics, observability dashboards, search over DynamoDB data (zero-ETL). |

- **⚠️ Trap:** "Relational + need to scale writes horizontally without limit" usually means the question actually wants **DynamoDB**, so check whether it truly needs joins and SQL.
- **⚠️ Trap:** Amazon QLDB reached end of support in 2025. If an older question asks for an "immutable, cryptographically verifiable ledger", that's what it expected. Don't pick QLDB for new designs.

**Depth:** [notes/05 Databases](../notes/05-databases.md)

---

## e. RDS / Aurora availability and read scaling

| Option | Main purpose | Standby readable? | Replication | Failover | Cross-Region | Pick when… |
|---|---|---|---|---|---|---|
| **RDS Multi-AZ DB instance** | **HA only** | ❌ | **Synchronous** to one standby in another AZ | **Automatic**, typically 60–120 s; same endpoint | ❌ | "Survive an AZ failure with automatic failover, no app changes." Works for any RDS engine. |
| **RDS Multi-AZ DB cluster** | HA + some read capacity | ✅ (2 readable standbys in 2 other AZs) | Semi-synchronous | Automatic, typically **<35 s** | ❌ | MySQL/PostgreSQL needing faster failover *and* a reader endpoint. |
| **RDS read replica** | **Read scaling**; DR when cross-Region | ✅ | **Asynchronous** | **Manual promotion** (becomes standalone) | ✅ | Read-heavy apps, reporting, cross-Region DR copy. Up to 15 for MySQL/MariaDB/PostgreSQL. A replica can itself be Multi-AZ. |
| **Aurora Replica** | Read scaling **and** HA | ✅ | Shared cluster storage volume (no data copy) | **Automatic**, promotes a replica by priority tier, usually ~30 s or less (approx.) | ❌ (use Global Database) | Read scaling with the reader endpoint, auto-scaled replicas, fast failover. Up to 15. |
| **Aurora Global Database** | **Cross-Region DR** + low-latency global reads | ✅ in secondary Regions | Storage-level async, typically **<1 s lag** | Managed switchover / failover: RPO ~1 s, RTO ~1 min (approx.) | ✅ up to **10 secondary Regions** | Global users reading locally, Regional DR with tight RPO/RTO. Write forwarding is optional. |

- **⚠️ Trap:** A Multi-AZ *instance* standby **cannot serve reads**. "Improve read performance" means read replicas (or Aurora Replicas), not Multi-AZ.
- **⚠️ Trap:** Read replicas are async, so they **aren't** an HA mechanism with automatic failover in plain RDS.
- Cross-Region DR for RDS: a cross-Region read replica or cross-Region automated backup replication. For Aurora: **Global Database**.

**Depth:** [notes/05 Databases](../notes/05-databases.md) · [notes/08 HA/DR](../notes/08-ha-dr-edge-ops.md)

---

## f. Load balancer choice

| LB | Layer | Protocols | Targets | Distinguishing features | Pick when… |
|---|---|---|---|---|---|
| **ALB** | 7 | HTTP, HTTPS, **gRPC**, WebSocket | Instances, IPs, **Lambda**, containers (by IP) | **Path/host/header/query routing**, redirects, fixed responses, **OIDC/Cognito auth**, **WAF**, sticky sessions, mTLS | Web apps, microservices, containers, anything HTTP that needs content-based routing. |
| **NLB** | 4 | **TCP, UDP, TLS** | Instances, IPs, **ALB** | **Static IP / Elastic IP per AZ**, millions of req/s, very low latency, preserves client IP, **PrivateLink endpoint service** front end, security groups supported | **UDP**, non-HTTP protocols, **static IPs for allow-listing**, extreme performance, exposing a service via PrivateLink. |
| **GWLB** | 3 | IP packets (**GENEVE**, port 6081) | Virtual appliances (firewalls, IDS/IPS) | Transparent bump-in-the-wire via **GWLB endpoints**, scales appliance fleets | **Insert third-party security appliances** inline for inspection. |
| **CLB** (legacy) | 4/7 | HTTP, HTTPS, TCP, SSL | EC2 | None worth choosing | ⚠️ **Never for new designs.** Only appears as "the existing CLB", and the fix is to migrate to ALB/NLB. |

- Static IP **and** HTTP routing → **NLB in front of an ALB** (ALB-type target group), or **Global Accelerator in front of the ALB**.
- Cross-zone load balancing: ALB on by default (no charge). NLB/GWLB off by default (inter-AZ data charges apply when enabled).

**Depth:** [notes/03 Compute & containers](../notes/03-compute-containers.md) · [notes/02 Networking](../notes/02-networking-vpc.md)

---

## g. Compute choice

| Option | You manage | Scaling | Max run time | Pick when… | Watch out |
|---|---|---|---|---|---|
| **EC2** | OS upward | EC2 Auto Scaling | Unlimited | Full control, special hardware (GPU, high memory), licensing (BYOL, Dedicated Hosts), legacy apps. | Most ops overhead. |
| **Lambda** | Code only | Automatic per request | **15 min** | **Event-driven**, spiky, short tasks, glue between services, scheduled jobs. | 15-min cap, 10 GB memory cap, cold starts, concurrency limits, DB connection storms (→ RDS Proxy). |
| **ECS on Fargate** | Task definitions | Service Auto Scaling | Unlimited | **Containers without managing servers.** The usual "least overhead" container answer. | No GPUs or privileged containers; priced per vCPU/GB. |
| **ECS on EC2** | Tasks + container instances | Capacity providers + ASG | Unlimited | Containers needing GPUs, specific instance types, daemon tasks, or cost control with Spot/RIs at scale. | You patch and scale the hosts. |
| **EKS** | Kubernetes workloads (managed control plane; nodes via managed node groups, Fargate or Auto Mode) | Cluster Autoscaler / Karpenter / HPA | Unlimited | **"Already uses Kubernetes"**, portability, Helm/K8s ecosystem. | More ops than ECS. Don't pick it unless Kubernetes is stated or implied. |
| **ECS Anywhere / EKS Anywhere** | On-prem hardware | — | — | Run containers on-premises managed with AWS tooling. | You own the hardware. |
| **Batch** | Job definitions and queues | Managed compute environments (EC2, **Spot**, Fargate) | Unlimited | **Batch/HPC jobs**, array jobs, work longer than 15 min, queued workloads. | Not for request/response APIs. |
| **Elastic Beanstalk** | App code (platform managed) | Provisions ASG + ELB for you | Unlimited | "Developers want to deploy a web app quickly **without managing infrastructure**," keeping control of the underlying resources. | Less fashionable but still valid on the exam. |
| **Lightsail** | Simple VPS bundles | Limited | — | **Usually a distractor.** Only right for a tiny, simple site with predictable flat pricing. | Not for scalable, enterprise or HA designs. |
| **Outposts** | AWS racks in your data center | — | — | **Low latency to on-prem systems**, local data processing, data residency, with AWS APIs. | You provide space, power, network. |
| **Wavelength** | — | — | — | **Ultra-low latency to 5G mobile devices** (compute inside telecom networks). | Carrier/Region availability. |
| **Local Zones** | — | — | — | **Single-digit ms latency for users in a specific metro** (gaming, live media, VDI) or in-metro residency. | Subset of services. |

**Depth:** [notes/03 Compute & containers](../notes/03-compute-containers.md) · [notes/06 Serverless](../notes/06-decoupling-serverless.md)

---

## h. EC2 purchasing options

| Option | Commitment | Discount vs On-Demand (approx.) | Flexibility | Capacity guarantee | Pick when… |
|---|---|---|---|---|---|
| **On-Demand** | None | 0% | Full | ❌ | Short-term, unpredictable, can't be interrupted, or a new workload with an unknown baseline. |
| **Compute Savings Plans** | $/h for **1 or 3 yr** | Up to ~66% | **Any family, size, Region, OS, tenancy**; also covers **Fargate and Lambda** | ❌ | Steady spend where instance types, Regions or compute platform may change. |
| **EC2 Instance Savings Plans** | $/h for 1 or 3 yr | Up to ~72% | One instance family in one Region (size/OS/tenancy flexible) | ❌ | Steady usage of a known family in one Region. |
| **Standard Reserved Instances** | 1 or 3 yr | Up to ~72% | Can modify AZ/size within a family; **sellable on the RI Marketplace** | ✅ if zonal | Steady-state. Also the classic answer for **RDS, ElastiCache, Redshift, OpenSearch** reserved capacity. |
| **Convertible RIs** | 1 or 3 yr | Lower than Standard | **Exchange** for another family/OS/tenancy | ✅ if zonal | Steady-state, but you expect to change families. |
| **Spot** | None | **Up to ~90%** | Interrupted with a **2-min notice** | ❌ | **Fault-tolerant, stateless, flexible** work: batch, CI, big data, rendering, containers. Diversify instance types; use a mixed-instances ASG. |
| **Dedicated Instances** | None or reserved | Premium | Hardware dedicated to your account; no host visibility | ❌ | Compliance requires single-tenant hardware, with no per-socket license needs. |
| **Dedicated Hosts** | On-Demand or reserved | Premium | **Socket/core visibility**, host affinity | ✅ | **BYOL per-socket/per-core licenses** (Windows Server, SQL Server, Oracle), strict compliance. |
| **On-Demand Capacity Reservations** | **No term** | 0% (stack with Savings Plans / regional RIs for discount) | Specific AZ + instance type | ✅ | **Guarantee capacity** in an AZ for an event, DR or a critical launch. Billed whether used or not. |

- Steady baseline → Savings Plans/RIs. Peaks → On-Demand. Interruptible extras → Spot. That's the classic **mixed fleet** answer.
- Newer: **Database Savings Plans** (Dec 2025, 1-yr) cover several managed databases. For RDS questions, reserved DB instances remain the classic exam answer.

**Depth:** [notes/03 Compute](../notes/03-compute-containers.md) · [notes/10 Cost](../notes/10-cost-optimization.md)

---

## i. Placement groups

| Strategy | Layout | Limits | Benefit | Risk | Pick when… |
|---|---|---|---|---|---|
| **Cluster** | Packed close together in **one AZ** | Single AZ; best with one instance type launched together | **Lowest latency, highest per-flow throughput** between instances | One rack/AZ problem hits everything; capacity errors | **HPC, tightly coupled** MPI jobs, low-latency trading. |
| **Spread** | Each instance on **distinct hardware (racks)** | **Max 7 running instances per AZ** per group | Minimizes correlated failure | Small groups only | **A few critical instances** that must not fail together (e.g., primary/secondary nodes). |
| **Partition** | Instances split into partitions on separate racks | **Up to 7 partitions per AZ**; hundreds of instances | Rack-level fault isolation at scale; partition metadata visible to apps | — | **Large distributed, replicated systems**: HDFS, HBase, Cassandra, Kafka. |

**Depth:** [notes/03 Compute](../notes/03-compute-containers.md)

---

## j. Auto Scaling policy types

| Policy | How it decides | Best for | Pick when… |
|---|---|---|---|
| **Target tracking** | Keeps a metric at a target value (e.g., CPU 50%, ALB `RequestCountPerTarget`, SQS backlog per instance) | Most workloads | **Default answer.** "Maintain average CPU at X%." |
| **Step scaling** | Scales by different amounts depending on how far an alarm is breached | Graduated responses | Large breaches need bigger jumps than small ones. |
| **Simple scaling** | One adjustment, then waits out the cooldown | Legacy | Rarely the best answer. Step or target tracking wins. |
| **Scheduled** | Changes min/max/desired at set times | **Known, time-based** patterns | "Every weekday at 08:00 traffic triples." |
| **Predictive** | ML forecast from history; launches capacity **ahead** of demand | **Recurring cyclical** load, slow-booting apps | Daily/weekly patterns where reactive scaling is too late. Pair with target tracking. |
| *Related: warm pools* | Keeps pre-initialized (stopped/hibernated) instances ready | Long boot or initialization | "Reduce scale-out latency for an app with a 10-minute startup." |
| *Related: lifecycle hooks* | Pause launch/terminate to run custom actions | Bootstrapping, log draining | "Copy logs off before termination." |

- **Queue-driven workers:** scale on **backlog per instance** (queue depth ÷ running instances) with target tracking, not on CPU.

**Depth:** [notes/03 Compute](../notes/03-compute-containers.md)

---

## k. Hybrid & inter-VPC connectivity

| Option | Connects | Transitive? | Encrypted? | Bandwidth / setup | Pick when… |
|---|---|---|---|---|---|
| **VPC peering** | 2 VPCs (cross-account, cross-Region) | ❌ **No**; CIDRs **must not overlap** | Stays on the AWS network (inter-Region traffic encrypted) | No gateway bottleneck; no hourly fee | **A few VPCs**, lowest cost, simple. |
| **Transit Gateway** | Hub for **thousands of VPCs**, VPNs, DX (via DX Gateway); inter-Region TGW peering | ✅ (route tables control segmentation) | VPN attachments are IPsec | Per-attachment hourly + per-GB | **Many VPCs / hub-and-spoke**, centralized egress or inspection, share across accounts with RAM. |
| **PrivateLink** (endpoint service) | Consumer VPC → **one specific service** behind an NLB in the provider VPC | N/A (service-level, one direction) | TLS is up to the app | Hourly + per-GB | **Expose one service to many VPCs/accounts**, **overlapping CIDRs OK**, SaaS offerings. |
| **Gateway endpoint** | VPC → **S3 or DynamoDB only** | Not reachable from on-prem or peered VPCs | — | **Free**; route-table entry | Private S3/DynamoDB access from inside the VPC, **removing NAT data charges**. |
| **Interface endpoint** | VPC → most AWS services (and S3) via an **ENI with a private IP** | Reachable from on-prem (DX/VPN) and peered VPCs | TLS | Hourly per AZ + per-GB | Private access to AWS APIs (SQS, KMS, Secrets Manager…), **S3 access from on-prem** over DX/VPN. |
| **Site-to-Site VPN** | On-prem ↔ VGW or TGW | Via TGW | ✅ **IPsec** | **2 tunnels**, ~1.25 Gbps each (5 Gbps large tunnels on TGW/Cloud WAN); **minutes to set up**; runs over the internet | **Quick, cheap, encrypted** hybrid link; **backup for DX**. ECMP on TGW aggregates tunnels. |
| **Client VPN** | **Individual users** (OpenVPN-based client) → VPC | — | ✅ TLS | Per endpoint-hour + connection-hour | **Remote employees** reaching private resources. |
| **Direct Connect (DX)** | On-prem ↔ AWS over a **private dedicated circuit** | Via DX Gateway + TGW | ❌ **Not encrypted by default** | Dedicated 1/10/100/400 Gbps; hosted 50 Mbps–25 Gbps; **weeks to provision** | **Consistent latency, high steady throughput, lower egress cost.** |
| **DX + Site-to-Site VPN** | IPsec running over DX | — | ✅ IPsec | VPN tunnel limits apply | "**Private and encrypted**" hybrid connectivity. |
| **DX + MACsec** | Layer-2 encryption on the DX link | — | ✅ at line rate | Dedicated 10/100/400 Gbps at supported locations | Encryption on DX **without the IPsec throughput ceiling**. |
| **DX Gateway** | One DX connection → VGWs/TGWs **in many Regions and accounts** | Not VPC-to-VPC transit | — | Global object | Reach VPCs in multiple Regions over one DX. |

- **⚠️ Trap:** Peering isn't transitive. If A↔B and B↔C, A cannot reach C. The answer is a direct peering or a TGW.
- **⚠️ Trap:** A gateway endpoint can't be used from on-premises. Use an S3 **interface endpoint** over DX/VPN.
- **Maximum DX resilience:** two connections at each of two DX locations. **Cost-effective resilience:** DX + VPN backup.

**Depth:** [notes/02 Networking & VPC](../notes/02-networking-vpc.md)

---

## l. Security group vs NACL vs Network Firewall vs WAF

| Control | Applies to | Stateful? | Rule actions | Evaluation | Pick when… |
|---|---|---|---|---|---|
| **Security group** | ENI / instance (and ALB, RDS, endpoints, Lambda in VPC) | ✅ Stateful | **Allow only** | All rules evaluated | Default micro-segmentation. **Reference other SGs** ("only the app tier may reach the DB"). |
| **Network ACL** | **Subnet** | ❌ Stateless (allow **ephemeral return ports**) | **Allow + Deny** | **Numbered, lowest first, first match wins** | **Block specific IPs/CIDRs** at the subnet edge. Explicit deny. |
| **Network Firewall** | VPC traffic routed through firewall endpoints | Stateful + stateless engines | Allow, drop, alert; **Suricata IPS**, **domain (FQDN) filtering**, TLS inspection | Rule groups by priority | **Centralized egress filtering, IDS/IPS**, domain allow-lists, inspection VPC behind TGW. |
| **Route 53 Resolver DNS Firewall** | DNS queries from VPCs | — | Allow, block, alert by domain | Rule groups | Stop instances resolving known-bad domains (DNS exfiltration). |
| **WAF** | **L7** on CloudFront, ALB, API Gateway REST, AppSync, Cognito user pools, App Runner, Verified Access | — | Allow, block, count, CAPTCHA/challenge | Web ACL rule priority | **SQL injection, XSS, rate limiting, geo-blocking, bot control**, managed rule groups. |
| **Shield Standard / Advanced** | Edge and Regional resources | — | DDoS mitigation | Automatic | Standard: free L3/L4 protection. **Advanced**: large/L7 DDoS (with WAF), **Shield Response Team**, **DDoS cost protection**. |

- **⚠️ Trap:** You **can't deny** with a security group. "Block this one malicious IP" means NACL (or WAF for HTTP).
- **⚠️ Trap:** NACL return traffic needs **ephemeral ports 1024–65535** open outbound/inbound.

**Depth:** [notes/02 Networking](../notes/02-networking-vpc.md) · [notes/07 Security](../notes/07-security-data-protection.md)

---

## m. DR strategies

| Strategy | What runs in the DR Region | Typical RPO (approx.) | Typical RTO (approx.) | Cost | Pick when… |
|---|---|---|---|---|---|
| **Backup & restore** | Nothing; backups/snapshots copied cross-Region; IaC to rebuild | **Hours** (since last backup) | **Hours** (up to ~24 h) | $ | Non-critical systems, lowest cost, "RTO of a day is acceptable". |
| **Pilot light** | **Core data live** (replicated DB); app tier off or minimal (AMIs, IaC ready) | **Minutes** | **Tens of minutes to hours** | $$ | Must recover fairly fast, but you don't want to pay for idle app servers. |
| **Warm standby** | **Scaled-down, fully functional** copy serving or ready to serve | **Seconds to minutes** | **Minutes** | $$$ | Business-critical, minutes of downtime tolerable. Scale out on failover. |
| **Multi-site active/active** | **Full production in 2+ Regions**, taking traffic | **Near zero** | **Near zero** | $$$$ | Zero-downtime mandates. Route 53 / Global Accelerator, DynamoDB Global Tables, Aurora Global Database. |
| *Tool: Elastic Disaster Recovery (DRS)* | Low-cost staging area with continuous block replication; launches recovery instances on demand | **Seconds** | **Minutes** | $–$$ | Server-level DR for on-prem or cloud servers without building pilot light yourself. |

- **Pick the cheapest strategy that meets the stated RPO/RTO.** Over-delivering (active/active for a 4-hour RTO) is wrong on cost.
- Supporting pieces: **AWS Backup** (cross-Region/cross-account copies, Vault Lock), S3 CRR (+RTC for a 15-min SLA), Route 53 failover records with health checks.

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

---

## n. Route 53 routing policies

| Policy | Decides by | Health checks | Pick when… |
|---|---|---|---|
| **Simple** | One record (multiple values returned in random order) | ❌ (not on simple records) | Single resource, no routing logic. |
| **Weighted** | Assigned weights (e.g., 90/10) | ✅ | **Canary / blue-green**, A/B tests, gradual migration. |
| **Latency-based** | Lowest measured latency from the user's network to a Region | ✅ | **Multi-Region app, best performance** per user. |
| **Failover** | Primary unless unhealthy, then secondary | ✅ **required on primary** | **Active-passive DR.** |
| **Geolocation** | User's **continent / country / state** | ✅ | **Localization, licensing, data-sovereignty** rules ("EU users must hit the EU stack"). Needs a **default** record. |
| **Geoproximity** | Distance to resources, with a **bias** to grow or shrink regions | ✅ | **Shift traffic between Regions by geography** in a tunable way. |
| **Multivalue answer** | Returns up to **8 healthy** records at random | ✅ | Simple client-side load spreading with health checks (not a substitute for ELB). |
| **IP-based** | Client source **CIDR** | ✅ | Route by known ISP/corporate networks. |

- **Alias records:** point the **zone apex** (example.com) at ELB, CloudFront, S3 website, API Gateway or Global Accelerator. Queries to AWS targets are free. A CNAME can't sit at the apex.
- Private hosted zones resolve only inside associated VPCs. **Resolver inbound/outbound endpoints** handle hybrid DNS.

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

---

## o. CloudFront vs Global Accelerator vs S3 Transfer Acceleration

| Service | What it does | Protocols | Caching | Static IPs | Pick when… |
|---|---|---|---|---|---|
| **CloudFront** | CDN: caches at edge locations, terminates TLS near users | HTTP/HTTPS, WebSocket | ✅ | ❌ (DNS name) | **Static and dynamic web content** for global users, offload the origin, **OAC** for private S3, signed URLs/cookies, geo-restriction, WAF, edge functions. |
| **Global Accelerator** | Two **anycast static IPs**; routes over the AWS backbone to Regional endpoints (ALB, NLB, EC2, EIP) | **TCP, UDP** | ❌ | ✅ | **Non-HTTP** (gaming, VoIP, IoT over UDP), **static IP allow-listing**, **fast multi-Region failover not dependent on DNS TTLs**. |
| **S3 Transfer Acceleration** | Uploads/downloads to **one S3 bucket** enter AWS at the nearest edge | HTTPS (S3 API) | ❌ | ❌ | **Global users uploading large files to a central bucket.** Combine with multipart upload. |

- **⚠️ Trap:** Global Accelerator doesn't cache. "Reduce origin load / cache images" means CloudFront.
- **⚠️ Trap:** CloudFront's ACM certificate must be in **us-east-1**.

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md) · [notes/04 Storage](../notes/04-storage.md)

---

## p. Messaging & streaming

| Service | Model | Ordering | Delivery | Retention / replay | Consumers | Pick when… |
|---|---|---|---|---|---|---|
| **SQS Standard** | Queue, consumers **poll** | Best effort | **At-least-once** (handle duplicates) | 1 min–14 days (default 4 days); no replay once deleted | Competing consumers | **Decouple and buffer**, absorb spikes, nearly unlimited throughput. |
| **SQS FIFO** | Queue | **Strict within a message group** | **Exactly-once processing** (5-min dedup window) | Same as Standard | Competing consumers per group | **Order matters** (payments, inventory), dedup. 300 TPS, or 3,000 msg/s with batching; high-throughput mode goes higher. |
| **SNS** | **Pub/sub push** | None (FIFO topics: ordered) | At-least-once | **No storage** (retries, DLQ) | Many: SQS, Lambda, HTTP(S), email, SMS, mobile push, Firehose | **Fan-out** one event to many subscribers, notifications, message filtering. |
| **EventBridge** | Event bus with **content-based rules** | None | At-least-once | **Archive and replay** | Up to 5 targets per rule; many AWS targets; cross-account buses | **AWS service events, SaaS partner events**, routing by event content, Scheduler, Pipes. |
| **Kinesis Data Streams** | Streaming log in **shards** | **Per shard (partition key)** | At-least-once | **24 h–365 days, replayable** | **Multiple independent consumers**; enhanced fan-out | **Real-time** analytics, ordered per key, **replay**, many consumers of the same data. |
| **Data Firehose** | Managed **delivery** pipeline | — | — | Buffers only; **no replay** | Destinations: S3, Redshift, OpenSearch, Splunk, HTTP endpoints, Iceberg tables… | **Load streaming data into stores with no code**, **near-real-time**, optional Lambda transform and Parquet/ORC conversion. |
| **Amazon MQ** | Managed **ActiveMQ / RabbitMQ** broker | Broker semantics | Broker semantics | Broker-dependent | Queues/topics | **Migrate existing JMS, AMQP, MQTT, STOMP, OpenWire apps without code changes.** |
| **MSK** | Managed **Apache Kafka** (provisioned or serverless) | Per partition | At-least-once (Kafka semantics) | Configurable (Kafka retention) | Consumer groups | **Existing Kafka apps**, Kafka Connect ecosystem, open-source compatibility. |

- **SNS → multiple SQS queues** = durable fan-out (each consumer gets its own buffered copy).
- Kinesis Data Streams = **real-time** (custom consumers, ~sub-second). Firehose = **near-real-time** (buffered).

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md) · [notes/09 Data & analytics](../notes/09-data-analytics-migration.md)

---

## q. Step Functions Standard vs Express

| Workflow type | Max duration | Execution semantics | Pricing | Callback / `.sync` patterns | History | Pick when… |
|---|---|---|---|---|---|---|
| **Standard** | **Up to 1 year** | **Exactly-once** | Per state transition | ✅ `.waitForTaskToken` (human approval), `.sync` jobs | Full visual history (90 days) | **Long-running, auditable** business processes, **human approval**, non-idempotent steps, orchestration with retries/catch. |
| **Express** | **Up to 5 minutes** | At-least-once (async) / at-most-once (sync) | Per request + duration | ❌ | CloudWatch Logs | **High-volume, short, idempotent** work: IoT/stream event processing, API backends, data transforms. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## r. API Gateway REST vs HTTP vs WebSocket

| API type | Strengths | Missing / limits | Auth options | Pick when… |
|---|---|---|---|---|
| **REST API** | **API keys + usage plans** (per-client throttling/quotas), **caching**, request validation, mapping templates, **WAF**, **private APIs** (VPC-only), edge-optimized/Regional/private endpoints, resource policies | Higher cost and latency than HTTP API | IAM, **Cognito user pools**, Lambda authorizers | You need **any** of: usage plans, caching, WAF, private endpoint, request transformation. |
| **HTTP API** | **Cheaper, lower latency**, native **JWT/OIDC authorizers**, simple Lambda/HTTP proxies, private integrations via VPC link (ALB/NLB/Cloud Map) | No caching, no usage plans/API keys, no WAF | IAM, JWT (OIDC/OAuth 2.0), Lambda authorizers | **Simple proxy APIs at lowest cost.** |
| **WebSocket API** | **Persistent two-way connections**; routes by message (`$connect`, `$disconnect`, `$default`); server push via the `@connections` URL | Connection max 2 h; idle timeout 10 min | IAM, Lambda authorizer (on `$connect`) | **Chat, live dashboards, multiplayer, real-time notifications.** |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## s. Analytics

| Service | What it is | Ops model | Pricing driver | Pick when… |
|---|---|---|---|---|
| **Athena** | **Serverless SQL on S3** (Trino/Presto engine), federated queries | Serverless | **Per TB scanned**: partition, compress, use **Parquet/ORC** | **Ad-hoc SQL on S3** data and logs (CloudTrail, ALB, VPC Flow Logs) with no infrastructure. |
| **Redshift** | MPP **data warehouse** | Provisioned (RA3) or Serverless | Node-hours or RPU-hours | **Complex joins/aggregations, BI at PB scale**, consistent dashboard performance. Spectrum for S3 data. |
| **EMR** | Managed **Hadoop/Spark/Hive/Presto/HBase** | On EC2, on EKS, or Serverless | Instance-hours (Spot for task nodes) | **Big-data processing with open-source frameworks** and cluster control; migrating on-prem Hadoop. |
| **Glue** | **Serverless ETL (Spark/Python)**, **Data Catalog**, crawlers, DataBrew | Serverless | DPU-hours | **ETL without clusters**, schema discovery, the shared catalog for Athena/EMR/Redshift Spectrum. |
| **OpenSearch Service** | Search and log analytics + Dashboards | Managed cluster or Serverless | Instance/storage or OCU | **Full-text search**, near-real-time log analytics and observability. |
| **Amazon Quick** (Quick Sight; formerly QuickSight) | Serverless **BI dashboards**, SPICE in-memory engine, embedded analytics | Serverless | Per user / capacity | **Dashboards and visualizations for business users** on Athena, Redshift, RDS, S3 and more. |
| **Lake Formation** | **Data lake governance** on top of the Glue Data Catalog | Managed | Permissions are mostly free; you pay for the underlying services | **Fine-grained (column/row/cell) access control**, LF-Tags, **cross-account data sharing** for an S3 data lake. |
| *Managed Service for Apache Flink* | Real-time stream processing (SQL/Java/Python) | Serverless | KPU-hours | Windowed aggregations and anomaly detection on Kinesis/MSK streams. |

- The typical serverless data lake: **S3 + Glue (catalog/ETL) + Athena + Quick**, governed by **Lake Formation**.

**Depth:** [notes/09 Data, analytics & migration](../notes/09-data-analytics-migration.md)

---

## t. Data transfer & migration

| Tool | Moves | Online / offline | Ongoing or one-time | Pick when… |
|---|---|---|---|---|
| **DataSync** | Files/objects: NFS, SMB, HDFS, object storage, other clouds → **S3, EFS, FSx** (and between AWS storage) | **Online** (agent; over internet or DX) | Both (scheduled tasks) | **Migrate or sync large file datasets** with integrity checks, encryption and bandwidth throttling. |
| **Transfer Family** | **SFTP, FTPS, FTP, AS2** endpoints backed by S3/EFS | Online | Ongoing | **Partners keep using SFTP/FTP/AS2**, with B2B file exchange landing in S3. |
| **Snowball Edge** | Physical device (Storage Optimized ≈210 TB usable, approx.) | **Offline** (shipped) | One-time / periodic | **100s of TB–PB over limited bandwidth**, or edge compute when disconnected. ⚠️ Closed to new customers since **Nov 7, 2025** but still in exam scope. |
| **Storage Gateway** | NFS/SMB/iSCSI/VTL from on-prem → S3/EBS snapshots/Glacier | Online | **Ongoing hybrid access** | On-prem apps **keep using file/block/tape protocols** with AWS as the backend. |
| **DMS** | Databases (full load + **CDC**), homogeneous or heterogeneous (with **Schema Conversion**/SCT) | Online | Both | **Database migration with minimal downtime**, continuous replication. |
| **Application Migration Service (MGN)** | Whole servers (physical, VMware, Hyper-V, other clouds) → EC2, via continuous block replication | Online | One-time migration (rehost) | **Lift-and-shift servers** with minimal downtime cutover. |
| **Elastic Disaster Recovery (DRS)** | Servers → low-cost staging in AWS, continuous replication | Online | **Ongoing (DR)** | **DR for on-prem/cloud servers** with seconds of RPO and minutes of RTO, plus failback. |

- **Time math:** 100 TB over a fully used 1 Gbps link ≈ 9–10 days. If the network would take more than about a week, or the link can't be saturated, **think offline (Snowball Edge)**.
- **Migration vs hybrid:** "one-time move of files" → DataSync. "Apps keep using NFS on-prem with S3 behind" → S3 File Gateway.

**Depth:** [notes/09 Data, analytics & migration](../notes/09-data-analytics-migration.md)

---

## u. Security services "which one?"

| Service | Answers the question… | Input / scope | Pick when the stem says… |
|---|---|---|---|
| **GuardDuty** | "**Is something malicious happening?**" | CloudTrail, VPC Flow Logs, DNS logs, plus optional S3, EKS, RDS, Lambda, runtime and malware protection | Compromised instance, crypto-mining, unusual API calls, **threat detection** with no agents needed for the core log sources. |
| **Macie** | "**Where is sensitive data in S3?**" | S3 objects | **Discover PII / sensitive data** in S3. |
| **Inspector** | "**Which workloads have vulnerabilities?**" | EC2 (SSM agent or agentless), **ECR images**, **Lambda** | **CVE / software vulnerability scanning**, unintended network exposure. |
| **Detective** | "**What happened, and how far did it spread?**" | Graph built from GuardDuty, CloudTrail, VPC Flow Logs, EKS logs | **Investigate the root cause** of a finding. |
| **Security Hub** | "**What's our overall posture across accounts?**" | Findings from GuardDuty, Inspector, Macie, Config, partners; standards checks | **Single pane of glass**, aggregate findings, CIS / AWS Foundational best-practice checks. |
| **Config** | "**What did this resource look like, and is it compliant?**" | Resource configuration history | **Compliance rules** (e.g., "all EBS encrypted"), config timeline, **auto-remediation** with SSM Automation, conformance packs. |
| **CloudTrail** | "**Who did what, when, from where?**" | API calls (management events by default; data events optional) | **Audit API activity**, "who deleted the bucket", organization trail, log integrity validation. |
| **Shield** | "Are we protected from **DDoS**?" | Edge/Regional resources | Standard: automatic free L3/L4. **Advanced**: SRT support, cost protection, L7 with WAF. |
| **WAF** | "Can we filter **malicious HTTP requests**?" | CloudFront, ALB, API Gateway, AppSync… | **SQL injection, XSS, rate limiting, geo-blocking.** |
| **Firewall Manager** | "How do we **enforce firewall policy across all accounts**?" | Organizations | **Centrally apply WAF rules, Shield Advanced, SGs, Network Firewall, DNS Firewall**, auto-applied to new accounts/resources. |
| **Artifact** | "Where are **AWS's** compliance reports?" | AWS-provided documents | **Download SOC/PCI/ISO reports**, accept agreements (e.g., BAA). |
| **Audit Manager** | "How do we **collect evidence for our own** audit?" | Your accounts | **Continuously gather evidence** mapped to frameworks (PCI DSS, HIPAA, GDPR…). |

- **⚠️ Trap: CloudTrail vs Config.** CloudTrail = *who called which API*. Config = *what state the resource is in and whether that's compliant*. "Who changed the security group?" → CloudTrail. "Is any SG open to 0.0.0.0/0 and fix it automatically?" → Config rule + remediation.
- **⚠️ Trap:** Artifact = AWS's compliance evidence. Audit Manager = **your** compliance evidence.

**Depth:** [notes/07 Security & data protection](../notes/07-security-data-protection.md) · [notes/01 IAM & multi-account](../notes/01-iam-multi-account.md)

---

## v. KMS key types vs CloudHSM

| Option | Who controls it | Rotation | Cross-account use | Cost | Pick when… |
|---|---|---|---|---|---|
| **AWS owned keys** | AWS (not visible in your account) | AWS-managed | N/A | Free | Default encryption you never need to see or audit. |
| **AWS managed keys** (`aws/s3`, `aws/ebs`…) | AWS manages them in your account; you can't edit the key policy | **Automatic yearly** (mandatory) | ❌ | No monthly fee (usage charged) | Simple encryption with CloudTrail visibility and no key admin. |
| **Customer managed keys** | **You**: key policy, grants, enable/disable, scheduled deletion (7–30 days) | Optional automatic (90–2,560 days, default 365) + on-demand | ✅ via key policy | Monthly per key + requests | **Control and separation of duties**, cross-account access, disabling keys, auditing usage. **The usual exam answer.** |
| **Imported key material (BYOK)** | You generate key material outside AWS | **Manual only** | ✅ | As CMK | Regulation requires keys **generated on-prem**, or the ability to delete key material instantly / set expiry. |
| **Custom key store** (CloudHSM-backed or external key store) | You, in your own HSM, behind the KMS API | Manual | ✅ | CMK + HSM costs | Need **KMS integration with services** *and* keys kept in a single-tenant HSM you control. |
| **Multi-Region keys** | Same key material/ID replicated across Regions | Synced | ✅ | Per replica | **Encrypt in one Region, decrypt in another** (DR, global tables, client-side encryption). |
| **CloudHSM** | **You exclusively** (single-tenant HSM cluster in your VPC; AWS can't access keys) | You manage | Your app's choice | Per HSM-hour (expensive) | **Dedicated / single-tenant HSM**, **FIPS 140-3 Level 3** requirement stated with customer-exclusive control, PKCS#11/JCE/CNG apps, TLS offload, Oracle TDE. |

- **⚠️ Trap:** KMS's own HSMs are also FIPS-validated at Level 3. The phrases that point to CloudHSM are **"single-tenant", "dedicated HSM", "customer has exclusive control and AWS has no access"**.
- KMS Encrypt handles **≤4 KB** directly. Anything larger uses **envelope encryption** (GenerateDataKey).

**Depth:** [notes/07 Security & data protection](../notes/07-security-data-protection.md)

---

## w. Secrets Manager vs Parameter Store

| Criterion | **Secrets Manager** | **SSM Parameter Store** |
|---|---|---|
| Built for | Secrets with a lifecycle (DB creds, API keys, OAuth tokens) | Configuration values + secrets (`SecureString`, KMS-encrypted) |
| **Automatic rotation** | ✅ **Built in**: managed rotation for RDS/Aurora/Redshift/DocumentDB; Lambda for custom secrets | ❌ No native rotation (build it with EventBridge + Lambda) |
| Cross-Region replication | ✅ Replica secrets | ❌ |
| Cross-account sharing | ✅ Resource policies | Advanced-tier parameters via RAM |
| Size | Up to 64 KB | 4 KB (Standard) / 8 KB (Advanced) |
| Hierarchy / config | Flat names | ✅ Hierarchical paths (`/app/prod/db-url`), versioning, Advanced-tier parameter policies (expiry) |
| Cost | Per secret per month + API calls | **Standard tier free**; Advanced tier charged |
| **Pick when…** | "**Rotate credentials automatically**", RDS master password management, cross-Region DR of secrets | **Cheap/free config and simple secrets**, feature flags, AMI IDs, no rotation requirement |

**Depth:** [notes/07 Security & data protection](../notes/07-security-data-protection.md)

---

## x. Directory / identity options

| Option | Who it's for | Credentials | Pick when… |
|---|---|---|---|
| **IAM users** (+ groups) | Individual long-lived identities | **Long-term** keys/passwords | ⚠️ Avoid for humans. Only break-glass access or legacy tools that can't assume roles. |
| **IAM roles** | AWS services, workloads, cross-account, federated users | **Temporary (STS)** | EC2 instance profiles, Lambda execution roles, **cross-account access**, CI/CD via OIDC. |
| **IAM Identity Center** | **Workforce** SSO across **many AWS accounts** + SAML apps | Temporary, via permission sets | **Employees + Organizations**, central SSO. Identity source: built-in directory, AD (Managed AD / AD Connector) or an external IdP (Entra ID, Okta via SAML + SCIM). |
| **Cognito user pool** | **Customers of your app** | JWTs (ID/access/refresh tokens) | **App sign-up/sign-in**, MFA, social/SAML/OIDC login, auth on ALB or API Gateway. |
| **Cognito identity pool** | App users (authenticated or guest) needing **AWS access** | **Temporary AWS credentials** (IAM roles) | **Mobile/web clients call S3/DynamoDB directly**, often with per-user prefixes. |
| **AWS Managed Microsoft AD** | Workloads needing **real AD in AWS** | Kerberos/LDAP | AD-aware apps, **trusts with on-prem AD**, FSx for Windows, RDS SQL Server Windows auth, WorkSpaces. Multi-AZ domain controllers. |
| **AD Connector** | AWS apps using **existing on-prem AD** | Proxied to on-prem | Use on-prem AD for WorkSpaces / Identity Center / console **without syncing or storing directory data in AWS**. Needs connectivity (VPN/DX). |
| **Simple AD** | Small, basic AD-compatible directory (Samba-based) | LDAP/Kerberos | Low-cost basic AD **with no trusts, no MFA, no on-prem integration**. |
| **SAML 2.0 federation to IAM** | Existing corporate IdP → IAM roles (`AssumeRoleWithSAML`) | Temporary | Legacy/single-account federation. **Identity Center is preferred for multi-account.** |

- "Existing on-prem AD, don't want to replicate it" → **AD Connector**. "Need AD in AWS with a trust to on-prem" → **Managed Microsoft AD**.

**Depth:** [notes/01 IAM & multi-account](../notes/01-iam-multi-account.md) · [notes/07 Security](../notes/07-security-data-protection.md)

---

## y. Cost tools

| Tool | Looks | Does | Pick when… |
|---|---|---|---|
| **Cost Explorer** | Backward + short forecast | Visualize spend by service/account/tag; RI and Savings Plans recommendations, utilization and coverage | "**Analyze cost trends**", find the biggest spenders, SP/RI coverage. |
| **Budgets** | Forward | Cost/usage/RI/SP budgets with alerts (email/SNS); **Budget Actions** (apply IAM/SCP policy, stop EC2/RDS) | "**Alert or act** when spend crosses a threshold." |
| **CUR / Data Exports** | Backward, most granular | **Hourly line-item** billing data to S3 (CUR 2.0, FOCUS format); query with Athena, Quick, Redshift | **Detailed custom analysis, chargeback**, finance data pipelines. |
| **Cost allocation tags** | — | Activate user-defined and AWS-generated tags in Billing to split costs | "**Show cost per team/project/environment.**" Tags must be **activated** before they appear. |
| **Compute Optimizer** | Usage metrics | ML **rightsizing** for EC2, ASGs, EBS, Lambda, ECS on Fargate, RDS/Aurora, licenses | "**Over-provisioned instances**", right-size based on utilization. |
| **Trusted Advisor** | Current state | Best-practice checks: cost, performance, security, fault tolerance, **service quotas**, operational excellence (full checks need Business support or higher) | Idle/underutilized resources, open SG ports, quota warnings. |
| **Cost Anomaly Detection** | Continuous | **ML-based** detection of unusual spend, alerts via SNS/email | "**Unexpected spend spikes** without setting fixed thresholds." |
| **Pricing Calculator** | Before building | Estimate the cost of a planned architecture | "**Estimate costs before migrating/building.**" |
| *Cost Optimization Hub* | Current | Consolidates savings recommendations across accounts and Regions | One prioritized list of savings opportunities. |

- **Budgets = alert on the future. Cost Explorer = analyze the past. CUR = raw detail. Anomaly Detection = surprises.**

**Depth:** [notes/10 Cost optimization](../notes/10-cost-optimization.md)
