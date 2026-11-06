# Numbers to Remember
> [Home](../README.md) · [Study plan](../00-study-plan.md) · [Decision tables](decision-tables.md) · [Exam keywords](exam-keywords.md)

> ⚠️ **Older exam questions and practice banks may still use previous limits (e.g., SQS 256 KB, gp3 16,000 IOPS). Reason from the scenario; when a question hinges on "message larger than the limit", the intended answer is usually S3 + pointer (the Extended Client Library pattern).**
>
> The same applies to other recently raised limits on this page (SNS, EventBridge and Lambda async payloads, Kinesis record size, Aurora storage and Global Database Regions). Learn the current value, but recognize the old one when a question is built around it.

These are the limits and specs that realistically show up on SAA-C03, each with **why it matters on the exam**. The exam tests *decisions*, so a number matters when it **rules an option in or out** ("job runs 20 minutes" rules out Lambda; "item is 600 KB" rules out storing it directly in DynamoDB). Values marked **approx.** are volatile or rounded. Check the AWS docs before relying on them in production.

**Sections:** [Exam](#the-exam-itself) · [S3](#s3) · [EBS](#ebs--instance-store) · [EFS/FSx](#efs--fsx-high-level) · [EC2/ASG](#ec2--auto-scaling) · [Lambda](#lambda) · [API Gateway](#api-gateway) · [SQS](#sqs) · [SNS](#sns) · [EventBridge](#eventbridge) · [Kinesis/Firehose](#kinesis-data-streams--data-firehose) · [Step Functions](#step-functions) · [DynamoDB](#dynamodb) · [RDS](#rds) · [Aurora](#aurora) · [ElastiCache/MemoryDB](#elasticache--memorydb) · [CloudFront/GA](#cloudfront--global-accelerator) · [Route 53](#route-53) · [VPC](#vpc) · [VPN/DX](#site-to-site-vpn--direct-connect) · [Transfer math & Snow](#data-transfer-math--snowball-edge) · [KMS/secrets/IAM](#kms-secrets--iam) · [Logging](#logging--monitoring) · [Pricing](#purchasing--pricing-levers) · [DR](#dr-strategy-typical-ranges)

---

## The exam itself

| Item | Value | Why it matters |
|---|---|---|
| Questions / time | 65 (50 scored + 15 unscored, unidentified) / 130 min | About 2 min per question. Treat every question as scored. |
| Passing score | **720** on a 100–1000 scale, compensatory | No per-domain minimum. Strong domains can carry weak ones. |
| Domain weights | D1 Secure 30% · D2 Resilient 26% · D3 High-performing 24% · D4 Cost 20% | Security is the biggest slice. Don't skimp on IAM/KMS. |
| Guessing penalty | None | Never leave a question blank. |

---

## S3

| Item | Value | Why it matters on the exam |
|---|---|---|
| Max object size | **50 TB** (since Dec 2025; previously 5 TB, and older questions may still say 5 TB) | Bigger datasets must be split into multiple objects. |
| Single PUT max | **5 GB** | Above this, **multipart upload is required**. |
| Multipart recommended | **>100 MB** | Faster, parallel, retryable uploads. "Large files fail on unreliable links" → multipart. |
| Multipart parts | Up to 10,000 parts, 5 MiB–5 GiB each (the last part can be smaller) | Rarely tested directly. 10,000 × 5 GiB is what caps a multipart upload at about 50 TB. |
| Request rate | **3,500 PUT/COPY/POST/DELETE** and **5,500 GET/HEAD** per second **per prefix** | Scale by spreading keys across prefixes. There's no limit on the number of prefixes. |
| Consistency | **Strong read-after-write** for all operations | Options that add a "consistency workaround" are unnecessary. |
| Durability | **11 9s** for all storage classes | One Zone classes are durable *within* an AZ but are lost if the AZ is destroyed. |
| Availability (design) | Standard 99.99% · Standard-IA, Intelligent-Tiering, Glacier IR 99.9% · One Zone-IA 99.5% | One Zone-IA is lower availability, not lower durability. |
| Minimum storage duration | Standard-IA / One Zone-IA **30 d** · Glacier IR **90 d** · Glacier Flexible **90 d** · Deep Archive **180 d** | Short-lived data in these classes is billed for the full minimum. |
| Minimum billable object size | **128 KB** (Standard-IA, One Zone-IA, Glacier IR) | Many tiny objects → keep them in Standard / Intelligent-Tiering. |
| Lifecycle to Standard-IA / One Zone-IA | Object must be **≥30 days** in its current class | "Move to IA after 7 days" isn't a valid lifecycle rule. |
| Glacier Flexible Retrieval | Expedited **1–5 min** · Standard **3–5 h** · Bulk **5–12 h** | Match the retrieval tier to the stated urgency. |
| Glacier Deep Archive | Standard **within 12 h** · Bulk **within 48 h** | "Retrieve within 24 hours, cheapest" → Deep Archive standard retrieval. |
| Glacier Instant Retrieval | **Milliseconds** | Archive pricing with instant access for quarterly reads. |
| Replication Time Control | **99.99% of objects within 15 min** (SLA-backed) | "Replicate within a predictable time / compliance SLA." |
| Presigned URL max lifetime | **7 days** (SigV4 with long-term credentials); shorter if signed with temporary credentials | Expires when the signing credentials expire. |
| Bucket quota | 10,000 general purpose buckets per account by default (raised from 100 in late 2024; adjustable) | Older material says 100. Bucket-per-customer designs are now feasible. |
| Object tags | Up to 10 per object | Tag-based lifecycle and access control. |
| New bucket defaults | Block Public Access **on**, ACLs **disabled** (bucket owner enforced), **SSE-S3** default encryption | "Make objects public via ACL" fails by default. Use bucket policies + CloudFront OAC. |

**Depth:** [notes/04 Storage](../notes/04-storage.md)

---

## EBS & instance store

| Item | Value | Why it matters on the exam |
|---|---|---|
| **gp3** | 1 GiB–**64 TiB**; baseline **3,000 IOPS + 125 MiB/s** at any size; up to **80,000 IOPS / 2,000 MiB/s** | Default choice. Performance is bought separately from size, and it's cheaper than gp2. |
| **gp2** | Up to 16 TiB; **3 IOPS/GiB**; max **16,000 IOPS / 250 MiB/s**; small volumes burst to 3,000 | Performance is tied to size. "Over-provisioned size just to get IOPS" → move to gp3. |
| **io2 Block Express** | 4 GiB–64 TiB; up to **256,000 IOPS / 4,000 MiB/s**; **99.999%** durability; Multi-Attach | Sub-ms latency, mission-critical DBs. Classic exam rule: **>16,000 IOPS on one volume or sub-ms → io2 BE**. gp3 now reaches 80,000, so read what the scenario actually needs. |
| io1 | Up to 64,000 IOPS / 1,000 MiB/s; Multi-Attach | Legacy. io2 is better at similar cost. |
| st1 / sc1 | st1 **500 MiB/s**, sc1 **250 MiB/s**; 125 GiB–16 TiB; **not bootable** | Throughput-oriented HDD. Never the boot volume. |
| Durability (gp2/gp3/io1/st1/sc1) | 99.8–99.9% (annual failure rate 0.1–0.2%) | Why snapshots matter. io2 is the 5-nines exception. |
| Multi-Attach | io1/io2 only, up to **16 Nitro instances**, **same AZ** | Not a cross-AZ shared disk. Needs a cluster-aware file system. |
| Scope | Volume lives in **one AZ** | Moving across AZs/Regions means snapshot → copy → restore. |
| Elastic Volumes | Change size/type/IOPS live; wait **≥6 h** between modifications of the same volume | gp2 → gp3 migration needs no downtime. |
| Snapshots | Incremental, stored regionally; copy across Regions/accounts; encrypt during copy | The way to encrypt an unencrypted volume and to do DR. |
| EBS Snapshots Archive | Up to ~75% cheaper storage; **90-day minimum**; restore takes **24–72 h** (approx.) | Long-term compliance snapshots rarely restored. |
| Fast Snapshot Restore | Removes first-read latency on volumes created from a snapshot (charged per AZ-hour) | "Volumes restored from snapshots must deliver full performance immediately." |
| Recycle Bin | Retention rules from 1 day to 1 year (approx.) | Recover accidentally deleted snapshots/AMIs. |
| Instance store | Survives **reboot**; lost on **stop, hibernate, terminate**, host failure | "Must persist" rules it out. |

**Depth:** [notes/04 Storage](../notes/04-storage.md)

---

## EFS & FSx (high level)

| Item | Value | Why it matters on the exam |
|---|---|---|
| EFS protocol / OS | **NFSv4.0/4.1**, **Linux only** (POSIX) | Windows clients → FSx for Windows. |
| EFS scope | Regional (multi-AZ mount targets) or One Zone | One Zone is cheaper but can't survive AZ loss. |
| EFS scale | Petabytes, **thousands of concurrent clients**, grows and shrinks automatically | No capacity planning, pay per GB used. |
| EFS storage classes | Standard · Infrequent Access · Archive (with lifecycle policies) | "Reduce EFS cost for files untouched for 30+ days" → lifecycle to IA/Archive. |
| EFS throughput modes | **Elastic** (recommended default), Provisioned, Bursting | Spiky throughput → Elastic. Steady high throughput on small data → Provisioned. |
| FSx for Lustre | Sub-ms latency, up to hundreds of GB/s, millions of IOPS; **Scratch** (no replication) vs **Persistent** | Short HPC jobs → Scratch (cheap). Long-lived → Persistent. |
| FSx for Windows | SMB, NTFS, **AD required**, Single-AZ or **Multi-AZ** | "Windows shares that survive an AZ failure" → Multi-AZ deployment. |
| FSx for NetApp ONTAP | NFS + SMB + iSCSI, Multi-AZ, snapshots/clones, SnapMirror | Multi-protocol or NetApp migration. |
| FSx for OpenZFS | NFS, very low latency, snapshots/clones | ZFS/Linux NAS migration. |

**Depth:** [notes/04 Storage](../notes/04-storage.md)

---

## EC2 & Auto Scaling

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Spot interruption notice** | **2 minutes** | Workloads must checkpoint or drain within 2 min. Stateful and long-uninterruptible work isn't for Spot. |
| Rebalance recommendation | Earlier, best-effort signal that a Spot instance is at elevated risk | ASG **Capacity Rebalancing** replaces at-risk instances proactively. |
| Spread placement group | **7 running instances per AZ** per group | "10 critical instances in one AZ, spread" won't fit. Use multiple AZs or groups. |
| Partition placement group | Up to **7 partitions per AZ**; hundreds of instances | Large distributed systems with rack awareness. |
| Cluster placement group | **Single AZ** | Lowest latency, but no AZ resilience. |
| Hibernation | RAM must be under 150 GiB; max 60 days hibernated; encrypted EBS root required (approx.) | "Resume in-memory state quickly after stop." |
| Monitoring | Basic **5-min** metrics (free) vs detailed **1-min** | Faster scaling reactions need detailed monitoring. |
| Memory / disk metrics | **Not collected** by default; need the **CloudWatch agent** | Classic trap: "scale on memory utilization." |
| ASG default cooldown | **300 s** | Applies to simple scaling. Target tracking uses instance warmup instead. |
| ASG health check grace period | 300 s (console default) | Too short → new instances killed before they finish booting. |
| Lifecycle hook default timeout | **1 hour** (extendable with heartbeats) | Time to bootstrap or drain before the transition completes. |
| Instance metadata | `169.254.169.254`; prefer **IMDSv2** (session tokens) | "Prevent SSRF from stealing role credentials" → require IMDSv2. |
| Public IPv4 | Billed **hourly** for all public IPv4 addresses, including in-use EIPs (approx. $0.005 per IP-hour) | Cost lever: fewer public IPs, NAT/ALB in front, IPv6. |

**Depth:** [notes/03 Compute & containers](../notes/03-compute-containers.md) · [notes/10 Cost](../notes/10-cost-optimization.md)

---

## Lambda

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Timeout** | Max **15 min** | Longer jobs → Fargate, Batch, EC2, or split with Step Functions. |
| Memory | **128 MB–10,240 MB**; CPU scales with memory (**1,769 MB ≈ 1 vCPU**) | "CPU-bound function too slow" → raise memory. |
| `/tmp` ephemeral storage | **512 MB–10,240 MB** | Larger scratch data → EFS mount or S3. |
| Env vars | **4 KB** total | Large config → Parameter Store / Secrets Manager / AppConfig. |
| Layers | **5** per function | Share libraries across functions. |
| Deployment package | **50 MB** zipped / **250 MB** unzipped; **container image up to 10 GB** | Big ML dependencies → container image. |
| Payload | **6 MB** synchronous · **1 MB** asynchronous (raised from 256 KB) | Bigger payloads → S3 + pointer. |
| Concurrency | **1,000** per Region by default (soft; new accounts may start lower) | Throttling under load → raise the quota, use reserved concurrency, or buffer with SQS. |
| Burst scaling | Each function adds up to 1,000 concurrent executions every 10 s (approx.) | Sudden huge spikes may throttle briefly. SQS absorbs the burst. |
| Async invocation retries | **2 retries** (3 attempts total); event age up to 6 h | Use destinations / DLQ for failed async events. |
| SQS event source batch size | Up to 10,000 (Standard) / 10 (FIFO) | Batch to cut invocations and cost. |
| In a VPC | Needs a **NAT gateway** (or VPC endpoints) for internet / AWS API access | "Lambda in VPC can't reach S3" → gateway endpoint or NAT. |
| DB connections | Each concurrent execution can open its own connection | Connection storms → **RDS Proxy**. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## API Gateway

| Item | Value | Why it matters on the exam |
|---|---|---|
| Account throttle | **10,000 rps** steady-state + **5,000 burst** per Region (soft) | 429 errors at scale → request an increase, use usage plans or caching. |
| Integration timeout | **29 s** default; REST (Regional/private) can be raised via quota, possibly lowering throttle; HTTP API max 30 s | Long-running backends → async pattern (return 202, process via SQS/Step Functions). |
| Payload size | **10 MB** | Bigger uploads → presigned S3 URL directly from the client. |
| Cache (REST) | TTL default 300 s, max 3,600 s | Cut backend load for repeated GETs. |
| WebSocket | Connection max **2 h**; idle timeout **10 min** | Clients must reconnect / send keep-alives. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## SQS

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Max message size** | **1 MiB** (raised from 256 KiB in Aug 2025) | Larger payloads → **Extended Client Library + S3 (up to 2 GB)**. Old questions say 256 KB, and the S3-pointer answer is still right. |
| Retention | Default **4 days**, configurable **1 min–14 days** | "Messages must be kept for 10 days" → raise retention. Over 14 days → not SQS alone. |
| Visibility timeout | Default **30 s**, max **12 h** | Duplicate processing → raise it to exceed processing time. |
| Delay | Max **15 min** (delay queue / per-message timer) | Longer delays → Step Functions Wait or EventBridge Scheduler. |
| Long polling | `WaitTimeSeconds` max **20 s** | Fewer empty receives, lower cost. |
| Batch | Max **10 messages** per send/receive/delete batch | Batching raises throughput and cuts cost. |
| FIFO throughput | **300 TPS** per API action without batching; **3,000 msg/s** with batching; high-throughput mode far higher (up to 70,000 TPS in the largest Regions) | Ordered + very high volume → high-throughput FIFO, or Kinesis. |
| FIFO deduplication window | **5 minutes** | Exactly-once processing within that window. |
| In-flight messages | ~120,000 (Standard) / ~20,000 (FIFO) (approx.) | Slow consumers with huge backlogs can hit this. |
| DLQ | `maxReceiveCount` sets how many receives happen before a message moves to the DLQ | Isolate poison messages. The DLQ type must match (FIFO ↔ FIFO). |
| Billing unit | Each 64 KB chunk = 1 request | Larger messages cost more requests. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## SNS

| Item | Value | Why it matters on the exam |
|---|---|---|
| Message size | **256 KB** classic limit (standard and FIFO topics); since Sep 2026, topics can **opt in** to up to 1 MiB (limited to SQS, Lambda and Firehose subscriptions) | Most questions assume 256 KB. Bigger → **Extended Client Library + S3 (up to 2 GB)**. |
| Delivery semantics | Push, at-least-once; no message storage | Needs a durable buffer → SNS → SQS fan-out. |
| Subscriber types | SQS, Lambda, HTTP/S, email, SMS, mobile push, Firehose | Email/SMS notifications → SNS. |
| Filtering | Subscription **filter policies** (attributes or body) | Route subsets of messages without consumer code. |
| FIFO topics | Ordered + deduplicated; subscribers are SQS FIFO (or standard) queues | Ordered fan-out. |
| Billing unit | Each 64 KB chunk = 1 request | Same as SQS. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## EventBridge

| Item | Value | Why it matters on the exam |
|---|---|---|
| Event size | **1 MB** (raised from 256 KB in Jan 2026) | Old questions assume 256 KB. Large details → S3 + reference. |
| Targets per rule | **5** | More targets → more rules, or fan out via SNS. |
| Retry policy | Retries for up to **24 h** / 185 attempts by default, then a DLQ if configured | Built-in resilience for delivery failures. |
| Archive & replay | Retain events indefinitely or for N days; replay to the bus | "Reprocess last week's events after a bug fix." |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## Kinesis Data Streams & Data Firehose

| Item | Value | Why it matters on the exam |
|---|---|---|
| **KDS write per shard** | **1 MB/s or 1,000 records/s** | Size the shard count: 5 MB/s ingest → at least 5 shards. Hot partition key → throttling. |
| **KDS read per shard** | **2 MB/s shared** across all standard consumers (5 GetRecords calls/s) | Many consumers starve each other → enhanced fan-out. |
| **Enhanced fan-out** | **2 MB/s per consumer per shard** (push over HTTP/2) | "Multiple consumers need low latency on the same stream." |
| **KDS retention** | **24 h default, up to 365 days** | Replay window. "Reprocess the last 7 days" → extend retention. |
| KDS record size | 1 MiB default; up to **10 MiB** opt-in (since Oct 2025) | Older questions assume 1 MB max. |
| KDS ordering | Per shard, determined by **partition key** | Order per customer/device → use that ID as the partition key. |
| KDS capacity modes | Provisioned (you manage shards) vs **on-demand** (auto) | Unpredictable stream volume → on-demand. |
| **Firehose buffering** | Interval classically **60–900 s** (default 300 s); size 1–128 MiB | **Near-real-time**, not real-time. Lower the interval for fresher data. |
| Firehose zero buffering | A 0-second buffer option exists for some destinations | Still treat Firehose as **near-real-time** on the exam. |
| Firehose replay | **None** (no retention) | Need replay → Kinesis Data Streams in front. |

**Depth:** [notes/09 Data & analytics](../notes/09-data-analytics-migration.md) · [notes/06 Decoupling](../notes/06-decoupling-serverless.md)

---

## Step Functions

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Standard** max duration | **Up to 1 year** | Long-running processes, human approval (`.waitForTaskToken`). |
| **Express** max duration | **Up to 5 min** | High-volume, short, idempotent workflows. |
| Semantics | Standard **exactly-once**; Express at-least-once (async) / at-most-once (sync) | Non-idempotent steps → Standard. |
| State payload | **256 KiB** per input/output | Large data → store in S3, pass the reference. |
| Standard history | 25,000 events per execution (approx.); history kept 90 days after close | Huge loops → child workflows or Distributed Map. |

**Depth:** [notes/06 Decoupling & serverless](../notes/06-decoupling-serverless.md)

---

## DynamoDB

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Max item size** | **400 KB** (names + values) | Larger blobs → S3 object + pointer in the item. |
| **RCU** | 1 strongly consistent read/s of up to **4 KB**, or **2** eventually consistent reads/s | Capacity math: 10 strongly consistent reads/s of 6 KB = 10 × 2 = 20 RCU. |
| **WCU** | 1 write/s of up to **1 KB** | 100 writes/s of 2.5 KB = 100 × 3 = 300 WCU. |
| Transactions | **Double** the capacity (2 RCU / 2 WCU per unit); up to 100 items / 4 MB per transaction | ACID across items costs twice as much. |
| Partition throughput | ~3,000 RCU and ~1,000 WCU per partition (approx.) | Why hot keys throttle even when table capacity is high. |
| **PITR** | Restore to any second in the last **1–35 days** (configurable) | "Recover from accidental writes in the past week." |
| On-demand backups | Kept until deleted (AWS Backup for cross-Region/account copies) | Long-term retention. |
| **LSI** | Must be created **at table creation**; max **5** per table; same partition key | "Add an index with a different sort key to an existing table" → GSI, not LSI. |
| **GSI** | **20** per table (default quota); can be added any time; eventually consistent | Alternate access patterns after launch. |
| Streams | Retain item changes for **24 h** | Trigger Lambda, replicate, audit changes. |
| TTL | Expired items deleted in the background, typically within a few days (approx.); no WCU cost | Don't rely on TTL for exact-time deletion. |
| Batch ops | BatchWriteItem 25 items · BatchGetItem 100 items / 16 MB | Fewer round trips. |
| Query/Scan page | **1 MB** per response (paginate) | Scans on big tables are slow and costly. Design keys for Query. |
| DAX | **Microsecond** reads | Read-heavy DynamoDB with hot items. |

**Depth:** [notes/05 Databases](../notes/05-databases.md)

---

## RDS

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Read replicas** | Up to **15** for MySQL, MariaDB, PostgreSQL (fewer for Oracle / SQL Server) | Read scaling. Async, so reads can be slightly stale. |
| **Automated backups** | Retention **up to 35 days** (0 disables); PITR to within ~5 min | Longer retention → manual snapshots or AWS Backup. |
| Manual snapshots | Kept until you delete them; copy across Regions/accounts | Long-term retention, DR, encrypting an unencrypted DB. |
| Max storage | **64 TiB** for most engines (verify per engine) | Bigger → Aurora (auto-grows) or sharding. |
| Multi-AZ DB instance failover | Typically **60–120 s** | HA, not zero-downtime. RDS Proxy shortens it. |
| Multi-AZ DB cluster failover | Typically **<35 s**; 2 readable standbys | Faster failover + reads for MySQL/PostgreSQL. |
| RDS Proxy | Pools connections; can cut failover time by up to 66% (approx.) | Lambda connection storms, faster failover. |
| IAM DB auth token | Valid **15 minutes** | Passwordless DB auth with IAM. |
| Storage autoscaling | Grows storage automatically up to a max you set | "Avoid running out of storage without manual work." |
| Encryption | Must be chosen **at creation**; to encrypt an existing DB, snapshot → encrypted copy → restore | A classic "how to encrypt an existing DB" question. |

**Depth:** [notes/05 Databases](../notes/05-databases.md)

---

## Aurora

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Aurora Replicas** | Up to **15** per cluster (shared storage, typically ms of lag) | Read scaling **and** failover targets. |
| Storage layout | **6 copies across 3 AZs**; writes need 4/6, reads 3/6 | Tolerates an AZ loss plus one more copy. Self-healing. |
| Max storage | **128 TiB** classically; **256 TiB** on supported engine versions (since mid-2025) | Auto-grows, so there's no storage provisioning. |
| Failover | Automatic, typically ~30 s or less (approx.) | Faster than RDS Multi-AZ instance failover. |
| **Global Database** | Up to **10 secondary Regions** (raised from 5 in May 2025); replication lag typically **<1 s**; RPO ~1 s, RTO ~1 min (approx.) | Cross-Region DR and local reads. Older material says 5 Regions. Max secondaries reduce primary-Region replicas (5 instead of 15 at 10 secondaries). |
| Backtrack (MySQL-compatible) | Rewind up to **72 h** | Undo a bad change without a restore. |
| Serverless v2 capacity | **0–256 ACUs** (0 = auto-pause on supported versions; older min 0.5) | Intermittent workloads can now pause to zero compute cost. |
| Cloning | Copy-on-write, minutes regardless of size | Fast test copies of production. |

**Depth:** [notes/05 Databases](../notes/05-databases.md)

---

## ElastiCache & MemoryDB

| Item | Value | Why it matters on the exam |
|---|---|---|
| Valkey / Redis OSS replicas | Up to **5** read replicas per shard; Multi-AZ automatic failover | HA cache. Memcached has no replication. |
| Cluster mode enabled | Data sharded across up to 500 shards (approx.) | Scale writes and memory horizontally. |
| Global Datastore (Valkey/Redis) | Primary + up to **2** secondary Regions (approx.) | Cross-Region cache reads and DR. |
| Memcached | Multi-threaded, no persistence, no replication | Simple, horizontally scaled object cache only. |
| MemoryDB | **µs reads**, single-digit ms writes, Multi-AZ durable log | Durable primary database with a Redis API. |
| Latency | ElastiCache **sub-ms** | "Sub-millisecond" → ElastiCache; "microsecond on DynamoDB" → DAX. |

**Depth:** [notes/05 Databases](../notes/05-databases.md)

---

## CloudFront & Global Accelerator

| Item | Value | Why it matters on the exam |
|---|---|---|
| Default TTL | **24 h** (86,400 s) unless headers or policy say otherwise | Stale content after deploy → invalidation or versioned file names. |
| ACM certificate Region for CloudFront | **us-east-1** | Classic trap: a cert in eu-west-1 can't be attached to CloudFront. |
| CloudFront Functions | Sub-ms runtime, 10 KB code, viewer request/response only | Header rewrites, redirects, simple auth at the lowest cost. |
| Lambda@Edge | Viewer triggers: 5 s; origin triggers: 30 s; authored in us-east-1 | Heavier edge logic and network calls. |
| Origin response timeout | 30 s default (adjustable) | Slow origins → 504s. |
| Global Accelerator | **2 static anycast IPv4** addresses; health-based failover in seconds (approx.) | Static IPs for allow-listing; failover without DNS TTL delays. |

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

---

## Route 53

| Item | Value | Why it matters on the exam |
|---|---|---|
| Health check interval | **30 s** standard or **10 s** fast (extra cost) | Faster detection → faster DNS failover. |
| Failure threshold | **3** consecutive checks (default) | Detection time ≈ interval × threshold. |
| TTL | Set per record; **alias records use the target's TTL** | Low TTL = faster failover but more queries. Clients may cache past the TTL. |
| Alias records | Free queries to AWS targets; work at the **zone apex** | Apex → ALB/CloudFront needs an alias, not a CNAME. |
| Multivalue answer | Up to **8** healthy records per response | Simple DNS-level spreading with health checks. |
| Private resources | Health checks can't reach private IPs directly → use a **CloudWatch alarm-based** health check | "Fail over based on a private resource's health." |

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

---

## VPC

| Item | Value | Why it matters on the exam |
|---|---|---|
| **Reserved IPs per subnet** | **5** (network, +1 router, +2 DNS, +3 reserved, broadcast) | A /28 gives 16 − 5 = **11 usable**. A /24 gives 251. |
| IPv4 CIDR size (VPC and subnet) | **/16** (65,536) max to **/28** (16) min | "Need 300 IPs in a subnet" → at least /23 (507 usable). |
| VPCs per Region | **5** default (soft) | Raise the quota. Not an architecture blocker. |
| IPv4 CIDR blocks per VPC | 5 by default (adjustable up to 50) | Add a secondary CIDR when a VPC runs out of IPs. |
| Internet gateway | **1 per VPC** | Highly available by design. Not a SPOF. |
| Security group rules | **60 inbound + 60 outbound** per SG (default); **5 SGs per ENI** default (up to 16) | Many CIDRs → managed prefix lists or reference SGs. |
| NACL rules | 20 per direction by default (up to 40); numbers **1–32766**, **lowest first, first match wins** | Put deny rules at lower numbers than broader allows. |
| NACL defaults | **Default NACL allows all**; a **new custom NACL denies all** until you add rules | "Created a NACL and everything broke." |
| Ephemeral ports | **1024–65535** | Stateless NACLs must allow return traffic on these. |
| VPC DNS resolver | VPC base address **+2** (also `169.254.169.253`) | Route 53 Resolver endpoints for hybrid DNS. |
| NAT gateway bandwidth | **5 Gbps**, scaling automatically to **100 Gbps** | Rarely a bottleneck. Per-AZ placement is about HA, not bandwidth. |
| VPC peering | Non-transitive; up to 125 per VPC (approx.) | Large meshes → Transit Gateway. |
| Gateway endpoints | **Free**; S3 and DynamoDB only | First cost fix for heavy S3 traffic through NAT. |

**Depth:** [notes/02 Networking & VPC](../notes/02-networking-vpc.md)

---

## Site-to-Site VPN & Direct Connect

| Item | Value | Why it matters on the exam |
|---|---|---|
| VPN tunnels | **2 per connection**, terminating in different AZs | Configure both on the customer gateway for HA. |
| VPN bandwidth | Up to **~1.25 Gbps per tunnel** (standard); **5 Gbps Large Bandwidth Tunnels** (Nov 2025, TGW/Cloud WAN only) | Above that → ECMP across tunnels on **TGW** (VGW doesn't do ECMP), or DX. |
| VPN setup time | Minutes | "Need hybrid connectivity this week" → VPN (then DX). |
| DX dedicated | **1, 10, 100 Gbps** (400 Gbps at select locations) | Large, steady bandwidth. |
| DX hosted | **50 Mbps–25 Gbps** via partners | Smaller or faster-to-order capacity. |
| DX lead time | **Weeks to months** (approx.) | Not an "urgent" answer. Bridge with VPN. |
| DX encryption | **None by default**; MACsec on dedicated 10/100/400 Gbps (supported locations); or IPsec VPN over DX | "Private **and** encrypted" → VPN over DX or MACsec. |
| DX virtual interfaces | **Private** (VPCs via VGW/DX GW) · **Public** (AWS public endpoints) · **Transit** (TGW via DX GW) | Many VPCs/Regions → transit VIF + DX Gateway + TGW. |
| DX resilience | Maximum: **2 connections at each of 2 locations**. Cost-effective: DX + VPN backup | Match the resilience tier to the requirement. |

**Depth:** [notes/02 Networking & VPC](../notes/02-networking-vpc.md)

---

## Data transfer math & Snowball Edge

Transfer time at **100% link utilization**. Real links reach maybe 70–80%, so add margin. 1 TB = 8 × 10¹² bits.

| Data | 100 Mbps | 1 Gbps | 10 Gbps |
|---|---|---|---|
| 1 TB | ~22 h | ~2.2 h | ~13 min |
| 10 TB | ~9 days | ~22 h | ~2.2 h |
| 100 TB | ~93 days | **~9 days** | ~22 h |
| 1 PB | ~2.5 years | ~93 days | ~9 days |

| Item | Value | Why it matters on the exam |
|---|---|---|
| Rule of thumb | Network transfer **> ~1 week**, or the link can't be spared → **offline** | "Limited bandwidth, 100s of TB" → Snowball Edge. |
| Snowball Edge Storage Optimized | ≈**210 TB** usable per device (approx., current model); multiple devices in parallel for PB | Device count = data ÷ ~210 TB (round up). |
| Service status | ⚠️ Closed to **new** customers since **Nov 7, 2025**; Snowcone and Snowmobile discontinued | Still in the exam guide, so answer Snowball Edge where the scenario fits. In real life: DataSync, Data Transfer Terminal or partners. |
| DataSync | Online; agent-based; encryption + integrity verification; scheduling | Network is adequate (or DX exists) → DataSync. |

**Depth:** [notes/09 Data, analytics & migration](../notes/09-data-analytics-migration.md)

---

## KMS, secrets & IAM

| Item | Value | Why it matters on the exam |
|---|---|---|
| **KMS Encrypt max** | **4 KB** of plaintext | Larger data → **envelope encryption**: `GenerateDataKey`, encrypt locally, store the encrypted data key. |
| Key deletion | Waiting period **7–30 days** (default 30) | Deletion is irreversible. Prefer **disabling** a key. |
| Automatic rotation | Customer managed: optional, **90–2,560 days** (default 365) + on-demand; AWS managed: **yearly** | Rotation keeps the key ID, so old ciphertext still decrypts. |
| Imported key material | No automatic rotation; can set expiry | BYOK trade-off. |
| S3 Bucket Keys | Cut S3→KMS request costs by up to 99% | "SSE-KMS costs too high at scale." |
| CloudHSM | **FIPS 140-3 Level 3**, single-tenant; run **≥2 HSMs in different AZs** for HA | Dedicated HSM requirement. You manage keys and users. |
| Secrets Manager rotation | Built in; schedules as often as every **4 h** | "Rotate DB credentials automatically every 30 days" → Secrets Manager. |
| Secrets Manager secret size | Up to **64 KB**; deletion recovery window **7–30 days** | Accidental deletion is recoverable within the window. |
| Parameter Store | Standard: **4 KB**, 10,000 params, **free**. Advanced: **8 KB**, 100,000 params, charged, parameter policies | Cheap config → Standard tier. Expiry policies → Advanced. |
| STS AssumeRole session | **15 min–12 h** (default 1 h; max set by the role) | Long jobs need a longer max session or credential refresh. |
| Role chaining | Max **1 h** session | Role-to-role hops are capped at 1 hour. |
| IAM access keys | Max **2** per user | Lets you rotate keys without downtime. |
| SCP scope | Doesn't affect the **management account**; never **grants** permissions | "SCP allows X" alone gives nobody access. IAM must allow it too. |

**Depth:** [notes/07 Security & data protection](../notes/07-security-data-protection.md) · [notes/01 IAM & multi-account](../notes/01-iam-multi-account.md)

---

## Logging & monitoring

| Item | Value | Why it matters on the exam |
|---|---|---|
| CloudTrail Event history | **90 days** of management events, free, no setup | Longer retention or org-wide → create a **trail** to S3 (or CloudTrail Lake). |
| CloudTrail delivery | Typically within about **5 min** of the API call (approx.) | Not real-time. For instant reaction, use EventBridge rules on CloudTrail events. |
| CloudTrail data events | **Off by default** (extra cost) | "Log every S3 object read" → enable data events. |
| CloudWatch metric retention | **15 months** (1-min data for 15 days, 5-min for 63 days, 1-h for 455 days) | Longer history → export metrics/logs. |
| High-resolution metrics | Down to **1 s** (custom metrics) | Sub-minute alarms. |
| CloudWatch Logs retention | **Never expire** by default; set 1 day–10 years | Cost lever: set retention + export to S3. |
| Config history | Kept up to 7 years by default (configurable) | Compliance timeline per resource. |
| AWS Backup Vault Lock | Compliance mode becomes immutable after a cooling-off period of at least 72 h (approx.) | WORM backups that even root can't delete. |
| Glacier Vault Lock | **24 h** to confirm the lock policy, then immutable | Vault-level WORM (legacy S3 Glacier vaults). |

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

---

## Purchasing & pricing levers

| Item | Value (approx.) | Why it matters on the exam |
|---|---|---|
| Spot discount | Up to **~90%** | Interruptible work only. |
| EC2 Instance Savings Plans / Standard RIs | Up to **~72%** | Steady, known family. |
| Compute Savings Plans | Up to **~66%** | Flexible across family/Region/Fargate/Lambda. |
| Term / payment | **1 or 3 years**; All, Partial or No Upfront | 3-yr All Upfront = deepest discount, least flexibility. |
| Database Savings Plans | 1-yr commitment across several managed DB services (Dec 2025) | Newer than most exam material. RDS reserved instances remain the classic answer. |
| Data transfer IN from internet | **Free** | Ingest-heavy designs aren't penalized. |
| Same-AZ traffic over private IP | Free | Co-locate chatty components. |
| Cross-AZ traffic | ~$0.01/GB **each direction** | Multi-AZ chatty microservices cost money. Cross-AZ NAT routing too. |
| NAT gateway | Hourly + ~$0.045/GB processed (us-east-1) | S3/DynamoDB through NAT → gateway endpoint saves that. |
| Gateway endpoint | **Free** | — |
| Interface endpoint | Hourly per AZ + per GB | Not free. Centralize in a shared-services VPC if many VPCs need them. |

**Depth:** [notes/10 Cost optimization](../notes/10-cost-optimization.md)

---

## DR strategy typical ranges

These are approximate ranges, for matching a stated RPO/RTO to a strategy, not SLAs.

| Strategy | Typical RPO | Typical RTO | Relative cost |
|---|---|---|---|
| Backup & restore | Hours (since the last backup) | Hours, up to ~24 h | $ |
| Pilot light | Minutes | Tens of minutes to a few hours | $$ |
| Warm standby | Seconds to minutes | Minutes | $$$ |
| Multi-site active/active | Near zero | Near zero | $$$$ |
| Elastic Disaster Recovery (tool) | Seconds | Minutes (approx. 5–20 min) | $–$$ |
| Aurora Global Database (tool) | ~1 s | ~1 min (managed failover) | $$–$$$ |

**Pick the cheapest row that meets both the RPO and the RTO.**

**Depth:** [notes/08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)
