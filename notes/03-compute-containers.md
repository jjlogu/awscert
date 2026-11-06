# 03 · Compute and containers

> [Home](../README.md) · [Study plan](../00-study-plan.md) · Next: [04 Storage](04-storage.md)

| Covers task statements | Est. study time | Hands-on lab | Practice |
|---|---|---|---|
| 2.1, 3.2, 4.2 (plus compute parts of 2.2) | ~4 hrs | [Lab 03](../labs/lab-03-alb-asg.md) | [D2 questions](../practice/d2-resilient.md) · [D3 questions](../practice/d3-high-performing.md) · [D4 questions](../practice/d4-cost-optimized.md) |

You've launched EC2 instances and written Lambda functions. The exam asks a different question: given a workload profile (steady or spiky, stateless or stateful, latency-sensitive, fault-tolerant, licence-bound, on-prem-bound), **which compute shape, purchasing model and scaling mechanism fits best**, and what is the trap in each? This module goes from EC2 internals to Auto Scaling and load balancing, then serverless and containers, and ends with hybrid/edge compute and rightsizing.

---

## 1. EC2 instance families and naming

### 1.1 Families

| Category | Families | Shape | Typical workloads |
|---|---|---|---|
| **General purpose** | **M**, **T** (burstable), Mac | Balanced CPU:memory (M ≈ 1 vCPU : 4 GiB) | Web/app servers, small/medium DBs, dev/test, microservices |
| **Compute optimized** | **C** | High CPU:memory (≈ 1:2) | Batch processing, HPC, game servers, video encoding, scientific modeling, CPU-based ML inference, high-traffic web front ends |
| **Memory optimized** | **R**, **X**, High Memory (**U**), z1d | Large RAM (R ≈ 1:8, X ≈ 1:16 or more) | In-memory databases and caches, SAP HANA, real-time big-data analytics |
| **Storage optimized** | **I** (local NVMe SSD), **D** (dense local HDD), H | Very high local disk IOPS or throughput | NoSQL (Cassandra, MongoDB), OLTP, data warehousing, Kafka, HDFS/MapReduce, search |
| **Accelerated computing** | **P** (GPU training/HPC), **G** (graphics, GPU inference), **Inf** (Inferentia), **Trn** (Trainium), F (FPGA) | Hardware accelerators | ML training and inference, rendering, video transcoding, genomics |
| **HPC optimized** | Hpc | Many cores + EFA networking | Tightly coupled CFD, weather modeling |

Mnemonic: **C**ompute, **R**AM, **I**OPS, **D**ense storage, **G**raphics, **P**arallel GPU, **Inf**erence, **Tr**ai**n**ing.

### 1.2 Decoding a name

`m7gd.2xlarge` = family **m** · generation **7** · attributes **g** (Graviton) + **d** (local NVMe instance store) · size **2xlarge**.

| Attribute letter | Meaning |
|---|---|
| `a` | AMD processor |
| `g` | AWS **Graviton** (Arm64) |
| `i` | Intel processor |
| `d` | Local NVMe **instance store** included |
| `n` | Enhanced networking bandwidth |
| `e` | Extra memory or storage |
| `z` | High CPU frequency |
| `b` | Higher EBS (block storage) performance |
| `flex` | Lower-cost variant for workloads that don't need full sustained CPU (e.g., `m7i-flex`) |
| `.metal` | Bare metal: no hypervisor, direct hardware access (licensing, custom hypervisors) |

Within a family, each size step roughly doubles vCPU, memory, and price. Newer generations generally give better price-performance than older ones at the same size.

### 1.3 Graviton

AWS's Arm-based processors deliver the best **price-performance** for many workloads (AWS cites up to approx. 40% better than comparable x86 for many workloads) and lower energy use. Graviton is available for EC2, **Lambda (arm64)**, **Fargate**, RDS, Aurora, ElastiCache, OpenSearch Service and EMR. The catch: software must run on Arm64. Interpreted languages and JVM apps usually just work; native binaries and container images need Arm builds (multi-arch images). Exam cue: "improve price-performance with minimal effort" for a Java/Python/Node.js workload → **move to Graviton**.

### 1.4 Burstable T instances and CPU credits

- Each size has a **baseline** CPU percentage. Below baseline, the instance **earns CPU credits**; above it, it **spends** them. One credit = one vCPU at 100% for one minute.
- **Standard mode:** when credits run out, the CPU is throttled to baseline.
- **Unlimited mode** (default for T3, T3a, T4g): it can burst beyond its credits. If average usage over 24 hours exceeds the baseline, you pay a flat charge for **surplus credits**.
- Watch `CPUCreditBalance` and `CPUSurplusCreditBalance` in CloudWatch.
- Exam cue: a T instance that runs **consistently high CPU** is mis-sized. Move to **M or C**, which is cheaper than paying unlimited-mode surplus and faster than being throttled.

---

## 2. Purchasing options

### 2.1 Comparison

| Option | Max discount vs On-Demand (approx.) | Commitment | Flexibility | Capacity guaranteed? | Best for |
|---|---|---|---|---|---|
| **On-Demand** | 0% | None; per-second billing (60 s minimum) for Linux and Windows | Full | ❌ | Short-term, spiky, unpredictable; new workloads you haven't profiled |
| **Compute Savings Plans** | ~66% | $/hour for 1 or 3 years | **Any** family, size, AZ, **Region**, OS, tenancy; also **Fargate and Lambda** | ❌ | Steady baseline spend while the architecture keeps evolving |
| **EC2 Instance Savings Plans** | ~72% | $/hour, 1 or 3 years, **one family in one Region** | Size, OS, tenancy, AZ within that family/Region | ❌ | Steady use of a known family in a known Region |
| **Standard RI** | ~72% | 1 or 3 years, specific instance attributes | Modify AZ, scope, size (same family, Linux); **sell on the RI Marketplace** | Only **zonal** RIs | Very stable workloads. RIs also exist for **RDS, ElastiCache, OpenSearch, Redshift**, which Savings Plans for compute don't cover. |
| **Convertible RI** | ~66% | 1 or 3 years | **Exchange** for a different family, OS or tenancy of equal or greater value; can't be sold | Only zonal | Stable, but the instance type may change |
| **Spot** | ~90% | None | Can be **interrupted with a 2-minute notice** | ❌ | Fault-tolerant, stateless, flexible, checkpointed work |
| **Dedicated Host** | Varies (On-Demand, reservation, or Savings Plans) | Optional | A whole physical server you control | ✅ (the host) | **BYOL** per-socket/per-core licences, compliance needing host control |
| **Dedicated Instance** | Premium over On-Demand | None | Instances on hardware dedicated to your account | ❌ | Compliance requiring single-tenant hardware, without host management |
| **On-Demand Capacity Reservation** | None by itself (combine with Savings Plans or regional RIs) | None; billed at On-Demand rates **whether used or not** | Specific instance type in a **specific AZ** | ✅ | Guaranteed capacity for events, DR or regulated workloads |

Payment options for Savings Plans and RIs: **All Upfront > Partial Upfront > No Upfront** in discount depth. A 3-year term discounts more than 1-year. SageMaker AI has its own Savings Plans. Database commitments are covered in [module 05](05-databases.md).

### 2.2 Reserved Instance details

- **Regional RI**: discount applies to matching usage in **any AZ** of the Region, with **instance size flexibility** within the family (Linux/Unix, default tenancy). **No capacity reservation.**
- **Zonal RI**: discount plus a **capacity reservation** in one AZ, but no size flexibility.
- AWS generally recommends **Savings Plans over RIs** for EC2 because they're simpler and more flexible. RIs remain relevant for non-EC2 services and for capacity guarantees with zonal RIs.

### 2.3 Spot in depth

- You pay the current Spot price, which moves gradually with long-term supply and demand. No bidding is needed. An optional maximum price defaults to the On-Demand price.
- **Interruption notice: 2 minutes**, through instance metadata and an EventBridge event. An earlier **rebalance recommendation** signal often comes first. Interruption behavior: **terminate** (default), **stop** or **hibernate**.
- **Allocation strategies** (EC2 Fleet, Spot Fleet, Auto Scaling groups):

| Strategy | Picks | Use |
|---|---|---|
| **price-capacity-optimized** | Pools with the lowest interruption risk, then the lowest price among them | **Recommended default** for most workloads |
| capacity-optimized (and -prioritized) | Pools with the most spare capacity (lowest interruption risk); the prioritized variant honors your instance-type order | Workloads where interruption is expensive (long jobs, big state) |
| lowest-price | The cheapest pools | ⚠️ Highest interruption risk; not recommended |
| diversified (Spot Fleet) | Spread across all specified pools | Long-running fleets that want broad spread |

- Best practice: **diversify** across many instance types (or use **attribute-based instance type selection**: "any type with 4–8 vCPUs and 16–32 GiB") and across AZs. Make the work **idempotent and checkpointed**.
- **Spot Fleet** is the older API. **EC2 Fleet** and **Auto Scaling groups with a mixed instances policy** are the modern approaches. (Spot blocks, the fixed-duration Spot option, are discontinued.)
- Spot shows up elsewhere too: **Fargate Spot**, EMR task nodes, **AWS Batch** Spot compute environments, EKS node groups and Karpenter.
- ⚠️ Not for databases, stateful singletons, or anything that can't tolerate a 2-minute termination.

### 2.4 Dedicated Hosts vs Dedicated Instances

| | Dedicated Host | Dedicated Instance |
|---|---|---|
| Hardware not shared with other AWS accounts | ✅ | ✅ |
| Visibility of sockets, cores, host ID | ✅ | ❌ |
| Control instance placement, **host affinity** | ✅ | ❌ |
| BYOL for per-socket/per-core/per-VM licences (Windows Server, SQL Server, Oracle) | ✅ (with **License Manager**) | Limited |
| Billing | Per host | Per instance, plus a per-Region dedicated fee |
| Exam cue | "Existing server-bound software licences", "licence compliance", "need core count visibility" | "Must run on single-tenant hardware" without licensing requirements |

### 2.5 Capacity guarantees

- **On-Demand Capacity Reservations (ODCRs):** reserve a specific instance type in a specific AZ for as long as you like, with no term commitment. You pay On-Demand rates whether you use it or not, so combine it with a **Savings Plan or regional RI** to get a discount. Exam cue: "guarantee capacity in an AZ for a two-week launch event", "DR capacity must be available when we fail over".
- **Capacity Blocks for ML:** reserve GPU instances for a future time window, for short ML training runs.
- Zonal RIs reserve capacity but lock you into a 1- or 3-year term.

### 2.6 Layering strategy (how architects actually buy)

| Layer of the load curve | Buy with |
|---|---|
| Always-on baseline | **Savings Plans** (Compute for flexibility, EC2 Instance for depth) or RIs |
| Capacity that must exist (DR, events) | **ODCR** (+ Savings Plan for the discount) |
| Predictable daily/weekly peaks | On-Demand via **scheduled/predictive scaling**, possibly covered by Savings Plans |
| Burst, batch, fault-tolerant | **Spot** (price-capacity-optimized, diversified) |
| Licence-bound | **Dedicated Hosts** |

**Rightsize first, then commit.** Buying a 3-year plan for an oversized fleet locks in waste ([section 14](#14-rightsizing-compute-optimizer)).

---

## 3. Placement groups

| | **Cluster** | **Spread** | **Partition** |
|---|---|---|---|
| Layout | Packed close together in **one AZ** on a high-bisection network segment | Each instance on **distinct hardware (rack)** | Instances grouped into **partitions**; each partition is on its own set of racks |
| AZs | One | Can span AZs | Can span AZs |
| Limits | Capacity risk: launch all instances at once, same type | **Max 7 running instances per AZ** per group | **Up to 7 partitions per AZ**; hundreds of instances per partition |
| Benefit | **Lowest latency, highest throughput** between instances (best single-flow bandwidth) | Minimizes **correlated hardware failure** | Contains failures to one partition; **topology exposed** to the app via metadata |
| Downside | A rack or AZ problem hits everything | Small scale only | — |
| Use cases | HPC, tightly coupled MPI, low-latency trading, distributed ML training (with EFA) | A few critical instances: primary/secondary pairs, domain controllers, quorum nodes | Large distributed, replicated systems: **HDFS, HBase, Cassandra, Kafka** |

If a cluster placement group launch fails with "insufficient capacity", stop and start all instances together (they may move to hardware with room), or retry later. Don't mix instance types.

---

## 4. Network interfaces: ENI, ENA, EFA

| | ENI (elastic network interface) | ENA (enhanced networking) | EFA (Elastic Fabric Adapter) |
|---|---|---|---|
| What | A virtual NIC object | SR-IOV-based high-performance networking | ENA plus an **OS-bypass** interface (libfabric) |
| Gives you | Primary and secondary private IPs, EIP, MAC address, SGs | Higher bandwidth (up to 100 Gbps or more on large current-generation types), higher packets per second, lower latency | Low-latency, high-throughput inter-node communication for **MPI** and **NCCL** |
| Notes | Lives in one subnet/AZ. **Can be detached and moved** to another instance in the same AZ (failover keeps the IP and MAC). Multiple ENIs give dual-homed or management networks. | On by default on current Nitro instance types; no extra cost | Supported instance types only. OS-bypass traffic stays within one subnet. Pair it with a **cluster placement group**. |
| Exam cue | "Move the IP to a standby instance", "licence tied to a MAC address", "separate management network" | "Higher network performance at no extra cost" | "Tightly coupled HPC", "distributed ML training", "MPI" |

(ENA Express, an optional feature on supported instances, uses AWS's SRD protocol to raise single-flow bandwidth and cut tail latency between instances.)

---

## 5. Instance store vs EBS

| | Instance store | EBS |
|---|---|---|
| Attachment | Physically attached to the host (NVMe SSD or HDD) | Network-attached block storage |
| Persistence | Survives reboot. **Lost on stop, hibernate, terminate, or host failure.** | Persists independently of the instance (unless *Delete on termination* is set, which is the default for root volumes) |
| Performance | Highest IOPS and lowest latency (millions of IOPS on I-family) | Up to 256,000 IOPS per volume (io2 Block Express) |
| Cost | Included in the instance price | Per GB-month (+ provisioned IOPS/throughput) |
| Backup | Your job (replicate or copy elsewhere) | **Snapshots** (incremental, stored in S3), Data Lifecycle Manager, AWS Backup |
| Resize / detach | ❌ | ✅ (Elastic Volumes; detach and attach in the same AZ) |
| Use for | Caches, buffers, scratch space, temp files, **data replicated across nodes** (Cassandra, Kafka with replication) | Boot volumes, databases, anything that must survive |

EBS volume types, Multi-Attach and snapshot details are in [module 04](04-storage.md).

---

## 6. AMIs, user data, hibernation and instance lifecycle

### 6.1 AMIs and golden images

- An AMI is **Regional**. **Copy** it to other Regions (DR, multi-Region deployments) and **share** it with accounts, OUs or the org through launch permissions.
- Sharing an **encrypted** AMI requires a **customer managed KMS key** shared with the target account. AMIs encrypted with the default `aws/ebs` key can't be shared.
- **EC2 Image Builder** automates golden AMI pipelines: build, harden, patch, test, and distribute across Regions and accounts on a schedule.
- Protect images with AMI **deregistration protection** and the **Recycle Bin**.

| | Golden AMI ("bake") | User data / configuration at boot ("bootstrap") |
|---|---|---|
| Boot-to-ready time | **Fast** | Slower (installs at launch) |
| Flexibility | Rebuild the AMI to change anything | Edit the script anytime |
| Consistency | Immutable, tested artifact | Can drift; depends on external repositories at boot |
| Use when | **Auto Scaling must react quickly**; compliance needs identical images | Small environment-specific config |

The usual answer is a hybrid: bake the OS, runtime and dependencies into the AMI, and at boot pull environment config and secrets from **Parameter Store / Secrets Manager**. Exam cue: "instances take 15 minutes to become ready during scale-out" → **golden AMI** (and/or **warm pools**, [7.5](#75-warm-pools-and-lifecycle-hooks)).

### 6.2 User data and instance metadata

- **User data** runs **once, at first boot by default**, as root (cloud-init on Linux, EC2Launch on Windows). The limit is **16 KB**.
- User data is readable by anyone with access to instance metadata or `DescribeInstanceAttribute`. **Never put secrets in it.**
- The **instance metadata service (IMDS)** at `169.254.169.254` provides the instance ID, role credentials, tags and user data. **Require IMDSv2** (token-based) to block SSRF-style credential theft. You can enforce it account-wide or with an SCP ([module 01](01-iam-multi-account.md)).

### 6.3 Stop, hibernate, terminate

| Action | Instance store | EBS root | RAM | Public IPv4 | Private IP | Billing |
|---|---|---|---|---|---|---|
| Reboot | Kept | Kept | Lost | Kept | Kept | Continues |
| Stop / start | **Lost** | Kept | Lost | **Changes** (EIP kept) | Kept | Instance charges stop (EBS still billed) |
| **Hibernate** / start | Lost | Kept | **Saved to the EBS root volume and restored** | Changes (EIP kept) | Kept | As for stop |
| Terminate | Lost | Deleted by default | Lost | Released | Released | Stops |

**Hibernation** requirements: enabled at launch, an **encrypted EBS root volume** large enough to hold RAM, supported instance families, RAM below approx. 150 GB, and no more than 60 days hibernated. Use it for apps with long in-memory warm-up (caches, JVMs) that you stop and start often. Spot instances can hibernate on interruption.

**Recovery:** for supported instance types, EC2 automatically recovers an instance whose *system* status check fails onto new hardware, keeping its instance ID, IPs and EBS volumes. A CloudWatch alarm with the **recover** action does the same explicitly. In an ASG, the ASG replaces unhealthy instances instead.

---

## 7. EC2 Auto Scaling

### 7.1 Building blocks

- **Launch template** (versioned): AMI, instance type(s), key pair, SGs, IAM instance profile, user data, EBS, network interfaces, placement group, Spot options. **Launch configurations are legacy**: new accounts can't create them (since Oct 2024) and they don't support newer instance types. Always pick launch templates.
- **Capacity**: **min / max / desired**. Scaling policies change *desired* within min and max. Spread the group across **multiple AZs** in one Region; an ASG never spans Regions.
- **Health checks**: **EC2 status checks** by default. **Turn on ELB health checks** so instances that are running but failing at the application level get replaced. A **health check grace period** gives new instances time to boot before they're judged. Custom health signals can be sent with `SetInstanceHealth`.
- The ASG itself is free; you pay for the instances.

### 7.2 Scaling policies

| Policy | How it works | Use when |
|---|---|---|
| **Target tracking** | Keep a metric at a target (e.g., average CPU 50%, `ALBRequestCountPerTarget` 1,000). The ASG creates and manages the CloudWatch alarms. | **Default choice** whenever a utilization metric moves proportionally with capacity |
| **Step scaling** | CloudWatch alarm with steps sized by how far the metric breaches (e.g., +2 at 60%, +4 at 80%). Keeps responding while the alarm stays in breach. | You need bigger reactions to bigger breaches |
| **Simple scaling** | One adjustment per alarm, then it **waits for the cooldown** | Legacy. Prefer step or target tracking. |
| **Scheduled** | Set min/max/desired at given times (cron) | **Known** patterns: business hours, a Monday 9 a.m. spike, a nightly batch window |
| **Predictive** | ML forecast from history (needs at least 24 h of data, uses up to 14 days) that **launches capacity ahead** of the predicted load. *Forecast only* mode lets you evaluate it first. | **Recurring daily/weekly cycles**, especially with slow boot times. Combine with target tracking for the unexpected. |

- **Cooldown** (default 300 s) applies to **simple scaling** only. For target tracking and step scaling, set the **default instance warm-up** so new instances don't count toward metrics until they're ready (prevents overshooting).
- Scale out fast, scale in cautiously. Target tracking can be configured to never scale in, if another process handles that.
- **AWS Auto Scaling** and **Application Auto Scaling** apply the same policy types to other resources: ECS services, DynamoDB capacity, Aurora replicas, Lambda provisioned concurrency, Spot Fleets, EMR, SageMaker endpoints.

### 7.3 Scaling a queue worker fleet on SQS backlog

Raw queue depth is the wrong metric: 1,000 messages is a lot for 2 instances and nothing for 200. Use **backlog per instance**:

- Metric: `ApproximateNumberOfMessagesVisible` ÷ number of **InService** instances (a custom metric, or metric math inside the target tracking policy).
- Target: **acceptable latency ÷ average processing time per message**.
- Example: each message takes 0.5 s and the SLA is 50 s, so the target is **100 messages per instance**. With 3,000 messages visible, target tracking drives the group toward 30 instances.

### 7.4 Mixed instances policy

One ASG can combine **multiple instance types** (listed or via attribute-based selection) and **On-Demand + Spot**:
- **On-Demand base capacity** (e.g., the first 2 instances always On-Demand), then an **On-Demand percentage above base** (e.g., 20% On-Demand / 80% Spot).
- A Spot allocation strategy (**price-capacity-optimized**), and an On-Demand allocation strategy (lowest-price or prioritized).
- **Capacity Rebalancing** proactively replaces Spot instances that receive a rebalance recommendation.
- Exam cue: "cost-optimize a stateless web tier but keep a guaranteed minimum" → mixed instances policy with an On-Demand base (covered by a Savings Plan) and Spot above it.

### 7.5 Warm pools and lifecycle hooks

- **Warm pools** keep pre-initialized instances in a **Stopped** (pay only for EBS), **Hibernated**, or **Running** state. On scale-out, the ASG moves them into service in seconds instead of booting from scratch. Use them for apps with long initialization that can't be fixed with a better AMI.
- **Lifecycle hooks** pause an instance in **`Pending:Wait`** (after launch, before InService) or **`Terminating:Wait`** (before termination). The default heartbeat timeout is 1 hour (extendable). Your automation (EventBridge → Lambda or SSM Automation) finishes with `CompleteLifecycleAction`.
  - Launch hook examples: install agents, pull config, register with an external system, warm a cache.
  - Terminate hook examples: **drain work, copy logs off the instance**, deregister from licensing or DNS.

### 7.6 Termination policy, rebalancing and refresh

- **Default termination policy:** first pick the **AZ with the most instances** (keep AZs balanced), then the instance that best restores the On-Demand/Spot and instance-type allocation, then the one with the **oldest launch template or configuration**, then the one **closest to its next billing hour**. Custom options include `OldestInstance`, `NewestInstance`, `OldestLaunchTemplate`, and a Lambda-based custom policy.
- **Scale-in protection** on specific instances (e.g., one running a long job). **Standby** state takes an instance out of service for troubleshooting. You can **suspend processes** (e.g., `ReplaceUnhealthy`, `AZRebalance`) during maintenance.
- **AZ rebalancing:** after an AZ recovers, the ASG launches in the depleted AZ first and then terminates in the others.
- **Instance refresh:** rolling replacement onto a new launch template/AMI with a minimum healthy percentage, checkpoints, skip-matching, and automatic rollback.

---

## 8. Elastic Load Balancing

### 8.1 ALB vs NLB vs GWLB

| | **Application Load Balancer** | **Network Load Balancer** | **Gateway Load Balancer** |
|---|---|---|---|
| Layer | **7**: HTTP, HTTPS, HTTP/2, **gRPC**, WebSockets | **4**: TCP, UDP, TLS | **3**: IP packets, **GENEVE** (port 6081) |
| Routing | Rules on **path, host, HTTP header, method, query string, source IP**; weighted target groups; redirects; fixed responses | By listener port/protocol to a target group (flow hash) | Flow hash across an appliance fleet, with flow stickiness |
| Targets | Instances, IPs (including on-prem over DX/VPN), **Lambda functions**, ECS tasks (IP) | Instances, IPs, **an ALB** | Instances or IPs running virtual appliances |
| **Static IP** | ❌ (DNS name only) | ✅ **one per AZ, or your EIPs** | n/a |
| TLS | Terminates TLS (ACM, **SNI**), **mutual TLS** | TLS termination **or TCP pass-through** for end-to-end encryption | n/a |
| Client IP visible to target | `X-Forwarded-For` header | **Preserved** (or Proxy Protocol v2) | Preserved (transparent) |
| Built-in user authentication | ✅ **Cognito user pools / any OIDC IdP** | ❌ | ❌ |
| AWS WAF | ✅ | ❌ | ❌ |
| Security groups | ✅ | ✅ (must be attached at creation) | ❌ |
| Cross-zone default | **On**, no inter-AZ charge | **Off**; inter-AZ charges if turned on | Off |
| PrivateLink | Behind an NLB | **Backs endpoint services** | Backs **GWLB endpoints** |
| Performance | Scales automatically; pre-warming rarely needed | **Millions of requests/s, ultra-low latency**, handles sudden spikes | Line rate for inline appliances |
| Typical use | Web apps, microservices, containers, serverless APIs | Non-HTTP, **gaming, IoT, VoIP**, extreme performance, **static IPs for allow-lists**, PrivateLink | **Third-party firewalls, IDS/IPS, deep packet inspection** inserted transparently |

The **Classic Load Balancer** is legacy: no SNI, no Lambda targets, no path routing. "Migrate from CLB" → ALB or NLB.

### 8.2 Choosing a load balancer

| Requirement | Choose |
|---|---|
| Path- or host-based routing to microservices | **ALB** |
| Static IP addresses for customer firewalls | **NLB** (EIP per AZ), or **Global Accelerator** in front of an ALB |
| Both static IPs and L7 routing | **NLB → ALB target**, or Global Accelerator → ALB |
| UDP or non-HTTP TCP | **NLB** |
| Millions of rps with very low latency | **NLB** |
| TLS must terminate on the instances (end-to-end) | **NLB TCP listener (pass-through)**, or ALB re-encrypting to HTTPS targets |
| Sign users in before they reach the app | **ALB + Cognito/OIDC** |
| Invoke Lambda over HTTP without API Gateway | **ALB → Lambda target** |
| Expose a service privately to other VPCs/accounts | **NLB** + endpoint service (PrivateLink) |
| Inspect traffic with vendor appliances | **GWLB** |
| gRPC | **ALB** |

### 8.3 Behavior details the exam probes

- **Cross-zone load balancing:** spreads traffic evenly across all registered targets in all enabled AZs. When it's off, each LB node only sends to targets in its own AZ, so uneven target counts per AZ mean uneven load. On by default for ALB (can be turned off per target group, no charge). Off by default for NLB and GWLB (inter-AZ data charges when on).
- **Sticky sessions:** ALB duration-based cookie (`AWSALB`) or application-based cookie; NLB stickiness by source IP. Stickiness causes uneven load and lost sessions when a target dies. The better architecture is **stateless app servers with sessions in ElastiCache or DynamoDB**.
- **Deregistration delay** (connection draining on CLB): when a target is deregistered or scaled in, the LB stops new requests and waits for in-flight ones (**default 300 s**, range 0–3,600 s). Lower it for short requests to speed up deployments.
- **SNI:** ALB and NLB serve **multiple TLS certificates on one listener**, choosing by hostname. ACM certificates for an ELB must be in the **same Region** as the load balancer (CloudFront's must be in us-east-1).
- **Health checks** are per target group (ALB: HTTP path and expected codes; NLB: TCP, HTTP or HTTPS). ⚠️ If **all** targets are unhealthy, an ALB **fails open** and routes to all of them.
- **Slow start** ramps traffic to new targets gradually. Algorithms include round robin (default), least outstanding requests, and weighted random.
- **Blue/green and canary** at the LB: ALB **weighted target groups** shift a percentage of traffic between two versions.

---

## 9. AWS Lambda

### 9.1 Limits worth memorizing

| Limit | Value |
|---|---|
| Memory | **128 MB – 10,240 MB**. CPU scales with memory: **1,769 MB ≈ 1 vCPU**, approx. 6 vCPUs at the maximum. |
| Timeout | **15 minutes** max |
| `/tmp` ephemeral storage | 512 MB – 10,240 MB |
| Environment variables | 4 KB total |
| Layers | 5 per function |
| Deployment package | Zip **50 MB** zipped / **250 MB** unzipped; **container image up to 10 GB** |
| Invocation payload | **6 MB** synchronous, **1 MB** asynchronous |
| Concurrency | **1,000 per Region** by default (new accounts may start lower), adjustable |
| Scaling rate | Each function can add approx. 1,000 concurrent executions every 10 seconds |

Anything over 15 minutes, needing more than 10 GB of memory, or needing a GPU doesn't belong in Lambda → **Fargate, ECS, AWS Batch, or EC2**.

**Memory tuning:** more memory means more CPU, so a CPU-bound function often runs **faster and cheaper** at higher memory (you pay GB-seconds). Tools like Lambda Power Tuning find the sweet spot. **arm64 (Graviton)** functions have a lower GB-second price (approx. 20% lower). **Compute Savings Plans** also cover Lambda.

### 9.2 Concurrency controls

| | **Reserved concurrency** | **Provisioned concurrency** |
|---|---|---|
| What it does | **Reserves and caps** concurrency for one function | Keeps **pre-initialized** execution environments ready |
| Cold starts | Doesn't prevent them | **Eliminates** them up to the provisioned amount |
| Cost | Free | Charged for the provisioned time (plus a lower execution rate) |
| Use | Guarantee capacity for a critical function; **protect a downstream** resource (e.g., a database) by capping; **set to 0 to stop** all invocations | **Latency-sensitive** APIs, predictable traffic spikes |
| Scaling | Static setting | **Application Auto Scaling** (scheduled or target tracking on utilization) |
| Applies to | The function | A published **version or alias** (not `$LATEST`) |

Reserved concurrency is subtracted from the account pool, so other functions share what's left. For **SQS-triggered** functions, cap concurrency with the event source mapping's **maximum concurrency** setting instead, because throttled messages go back to the queue and burn receive counts ([module 06](06-decoupling-serverless.md)). When concurrency is exhausted, synchronous calls get **throttled (429)**. Asynchronous events are retried by Lambda for a period. Invocation models, retries and destinations are covered in [module 06](06-decoupling-serverless.md).

### 9.3 Cold starts and SnapStart

A **cold start** happens when Lambda creates a new execution environment: it downloads the code, starts the runtime and runs your initialization code. Mitigations:
- **Provisioned concurrency** (the definitive fix, at a cost).
- **SnapStart**: Lambda snapshots the initialized environment when you publish a version and resumes new environments from that snapshot. Supported for **Java, Python and .NET** runtimes (Java at no extra cost; Python and .NET pay for snapshot caching and restores). It can't be combined with provisioned concurrency, EFS, or ephemeral storage above 512 MB. Code must handle uniqueness (random seeds, connections) after a restore, via runtime hooks.
- Smaller packages, lazy-loaded dependencies, and **initializing SDK clients and DB connections outside the handler** so warm invocations reuse them.

Exam cue: "Java function has multi-second cold starts; minimize latency at **low cost**" → **SnapStart**. "Must have consistently low latency at predictable peak times" → **provisioned concurrency** (scheduled scaling).

### 9.4 Lambda in a VPC

- By default, Lambda runs in an AWS-managed VPC with internet access. To reach **private resources** (RDS, ElastiCache, internal APIs), attach the function to **your VPC subnets and security groups**.
- Lambda then creates **Hyperplane ENIs** in your subnets, shared per subnet + SG combination and created when you configure the function, not per invocation. The old big VPC cold-start penalty is gone.
- ⚠️ A VPC-attached function **has no public IP**. Even in a public subnet it can't reach the internet. For internet or AWS public endpoints, use private subnets with a route to a **NAT gateway**, or **VPC endpoints** for AWS services.
- The execution role needs ENI permissions (the `AWSLambdaVPCAccessExecutionRole` managed policy).
- For relational databases, put **RDS Proxy** in between to pool connections, so thousands of concurrent functions don't exhaust database connections ([module 05](05-databases.md)).

### 9.5 Lambda@Edge vs CloudFront Functions

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript | Node.js, Python |
| Triggers | **Viewer request, viewer response** only | Viewer request/response **and origin request/response** |
| Max execution time | **Sub-millisecond** | 5 s (viewer triggers), 30 s (origin triggers) |
| Memory / package | 2 MB / 10 KB | 128 MB on viewer triggers, up to 10 GB on origin triggers / 1 MB (viewer) or 50 MB (origin) |
| Network calls, file system, request body | ❌ | ✅ |
| Scale and price | Millions of requests/s, **much cheaper** | Lower scale, higher price |
| Runs at | All edge locations | Regional edge caches; authored in us-east-1 |
| Use for | Header manipulation, **URL rewrites/redirects**, cache-key normalization, simple token (JWT) validation, A/B routing by cookie (with KeyValueStore for data) | Calling external services, dynamic origin selection, image resizing, body inspection, heavier auth logic |

Rule of thumb: **lightweight, high-volume header/URL logic → CloudFront Functions**; anything needing network access, the body, origin-side triggers or more than a millisecond → **Lambda@Edge**.

---

## 10. Containers

### 10.1 Amazon ECS

ECS is AWS's own container orchestrator: **task definitions** (containers, CPU/memory, networking, IAM roles, logging), **tasks**, and **services** (keep N tasks running, register them with a load balancer, roll out deployments).

| | ECS on **EC2** | ECS on **Fargate** |
|---|---|---|
| Hosts | You manage the instances (AMI, patching, scaling through an ASG **capacity provider**) | **Serverless**: no instances to manage |
| Billing | The EC2 instances, whether or not tasks fill them | **Per task vCPU and memory per second** |
| Isolation | Tasks share instances | Each task gets its own isolated compute environment and kernel |
| Networking modes | `awsvpc`, bridge, host | **`awsvpc` only** (each task gets its own ENI and **security group**) |
| Storage | EBS, instance store, EFS | Ephemeral 20 GiB by default (configurable up to 200 GiB), **EFS**, EBS volumes attached to tasks |
| Fit | Large steady fleets with good bin-packing, **GPUs**, special instance types, daemon tasks, host-level access | Variable or spiky workloads, small teams, per-task isolation, **least operational overhead** |
| Cost levers | Savings Plans/RIs, Spot capacity providers, Graviton | **Fargate Spot** (up to approx. 70% off, 2-minute warning), Compute Savings Plans, Graviton (arm64) |

(Amazon ECS Managed Instances, a newer option, has AWS provision and patch the EC2 capacity for you while you keep EC2 instance choice. It's unlikely to be tested.)

**Task execution role vs task role** (a favourite trap):

| | **Task execution role** | **Task role** |
|---|---|---|
| Used by | The ECS agent / Fargate platform, **to start the task** | **Your application code** inside the container |
| Typical permissions | **Pull images from ECR**, write logs to CloudWatch Logs, **fetch Secrets Manager secrets / SSM parameters** referenced in the task definition | Whatever the app calls: S3, DynamoDB, SQS… |
| Failure symptom | Task never starts: `CannotPullContainerError`, can't retrieve secret | App runs but gets `AccessDenied` calling AWS APIs |

On the EC2 launch type there's also the **container instance role** (an instance profile the ECS agent uses to register with the cluster). Never give app permissions through it, because every task on the host would inherit them.

**Scaling ECS:**
- **Service auto scaling** (Application Auto Scaling): target tracking on `ECSServiceAverageCPUUtilization`, `ECSServiceAverageMemoryUtilization` or `ALBRequestCountPerTarget`; step; scheduled; predictive.
- On the EC2 launch type, the **cluster** also needs capacity: an **ASG capacity provider with managed scaling** grows the instance fleet to fit pending tasks. Fargate has no cluster capacity to manage.
- Deployments: rolling update (minimum healthy / maximum percent) with the **deployment circuit breaker** for automatic rollback; blue/green either natively in ECS (newer) or with CodeDeploy.
- Use **ECS Exec** for shell access into containers (no SSH). **Service Connect / Cloud Map** handle service discovery.

### 10.2 Amazon EKS

- **Choose EKS when Kubernetes itself is a requirement**: existing manifests, Helm charts and operators, K8s skills, multi-cloud or hybrid portability, the Kubernetes ecosystem. If none of that applies, ECS is simpler and the usual "least operational overhead" answer.
- AWS runs the **control plane** across multiple AZs for an hourly per-cluster fee (approx. $0.10/h). Kubernetes versions get approx. 14 months of standard support, then paid extended support.

| Data plane option | What it is | Notes |
|---|---|---|
| **Managed node groups** | EKS-managed ASGs of EC2 nodes | One-click updates, On-Demand or Spot; the common default |
| Self-managed nodes | Your own ASGs and AMIs | Maximum control, maximum effort |
| **Fargate profiles** | Pods matching namespace/label selectors run on Fargate | No nodes to manage; no DaemonSets, GPUs or privileged pods |
| Karpenter | Open-source just-in-time node provisioner | Picks right-sized (often Spot) instances fast and consolidates nodes to cut cost |
| **EKS Auto Mode** (newer) | AWS manages nodes, scaling, load balancing and storage add-ons | Closest to "serverless Kubernetes" with EC2 flexibility |

- Pod permissions: **EKS Pod Identity** or **IRSA** ([module 01](01-iam-multi-account.md)), not node instance roles.
- Networking: the VPC CNI gives each pod a **VPC IP address**, so plan large subnets. Fixes for IP exhaustion are a secondary CIDR (e.g., `100.64.0.0/10`) with custom networking, or prefix delegation.
- Load balancing: the **AWS Load Balancer Controller** turns an Ingress into an ALB and a `LoadBalancer` Service into an NLB.
- Storage: EBS CSI driver (single-AZ volumes) or **EFS CSI driver** (shared, multi-AZ).

### 10.3 Amazon ECR

- Private registry with IAM and **repository policies** (cross-account pulls), KMS encryption, **tag immutability**, and **lifecycle policies** (expire untagged or old images to cut storage cost).
- **Image scanning**: basic scanning, or **enhanced scanning through Amazon Inspector** (continuous, OS and language packages).
- **Cross-Region and cross-account replication**; **pull-through cache** rules for upstream registries (Docker Hub, ECR Public, Quay, GitHub).
- Private subnets with no NAT need interface endpoints **`ecr.api` and `ecr.dkr`** plus the **S3 gateway endpoint** (image layers are stored in S3), and a CloudWatch Logs endpoint for logging.

### 10.4 Containers on-premises and at the edge

| Option | Control plane | Where workloads run | Use when |
|---|---|---|---|
| **ECS Anywhere** | ECS in an AWS Region (managed) | **Your servers/VMs**, registered through the SSM Agent (`EXTERNAL` launch type) | You want ECS on existing on-prem hardware with a central AWS control plane; needs connectivity to the Region |
| **EKS Anywhere** | **You run it** on your infrastructure (VMware vSphere, bare metal, Nutanix and others), with EKS tooling; optional AWS support subscription | On-prem | Kubernetes on-prem that's consistent with EKS; can run **disconnected/air-gapped** |
| **EKS Distro** | Open-source Kubernetes distribution that EKS uses; you install and operate everything | Anywhere | DIY Kubernetes with the same tested versions and patches as EKS |
| EKS / ECS on **Outposts** | Region (EKS can also run a local control plane on the Outpost) | AWS-managed Outposts hardware in your data center | Low latency or data residency with AWS-managed hardware |

### 10.5 AWS App Runner (brief)

A fully managed service that deploys a **web app or API from source code or a container image** and handles the build, deployment, load balancing, TLS and auto scaling, with a VPC connector for private resources. It's even less configuration than ECS on Fargate, at the cost of control. It isn't in the SAA-C03 in-scope service list, so it rarely appears; check its current availability before designing with it.

### 10.6 Migrating a monolith to containers

1. **Containerize as-is** ("lift and containerize"): write a Dockerfile, or use **AWS App2Container** to generate one for existing Java and .NET apps.
2. Push the image to **ECR**. Run it on **ECS with Fargate** (least ops) behind an **ALB**, across at least two AZs.
3. **Externalize state**: sessions → ElastiCache/DynamoDB; files → S3/EFS; config and secrets → Parameter Store/Secrets Manager; logs → CloudWatch Logs.
4. **Strangler fig**: carve out functions into separate services and route to them with ALB **path-based rules**, retiring parts of the monolith over time.
5. **Decouple** the services with SQS, SNS or EventBridge ([module 06](06-decoupling-serverless.md)).

(Rehosting VMs unchanged onto EC2 is Application Migration Service territory: [module 09](09-data-analytics-migration.md).)

### 10.7 Compute decision table

| Scenario | Pick |
|---|---|
| Event-driven, short (< 15 min), spiky, pay per use | **Lambda** |
| Containerized services, no servers to manage | **ECS on Fargate** |
| Large steady container fleet, cost-optimized, GPUs | **ECS on EC2** (Savings Plans + Spot capacity providers) |
| Kubernetes required (Helm, portability, existing K8s) | **EKS** (managed node groups, Fargate, Karpenter or Auto Mode) |
| Upload code, get a managed web environment, keep access to the resources | **Elastic Beanstalk** |
| Large-scale batch jobs, queues, job dependencies, Spot | **AWS Batch** |
| Full OS control, legacy software, special licensing | **EC2** |
| Event-triggered work that runs longer than 15 minutes | **ECS/Fargate task** (via EventBridge or Step Functions) or **Batch** |
| Same container tooling on-prem | **ECS Anywhere / EKS Anywhere** |

---

## 11. AWS Batch

- **Fully managed batch computing.** You submit jobs (container images + commands) to **job queues** (with priorities). Batch provisions **compute environments** sized to the queued work and **scales to zero** afterwards.
- Compute environment types: managed **EC2 On-Demand or Spot**, **Fargate / Fargate Spot**, or **EKS**. Spot with a retry strategy is the cost-optimized default for fault-tolerant jobs.
- Features: **array jobs** (run the same job across thousands of inputs), **job dependencies**, **multi-node parallel jobs** (MPI-style HPC), scheduling policies for fair share.
- **No additional charge** for Batch itself; you pay for the compute.
- How it compares: **Lambda** has a 15-minute limit and no GPUs. **Step Functions** orchestrates workflows (and can submit Batch jobs). **EMR** is for Spark/Hadoop frameworks. **Glue** is for serverless ETL.
- Exam cue: "hundreds of thousands of containerized compute jobs, run nightly, minimize cost and management" → **Batch with a Spot compute environment**.

---

## 12. AWS Elastic Beanstalk

- A **PaaS** that takes your code and provisions and manages **EC2, an ASG, an ELB, CloudWatch alarms** and (optionally) a database, while leaving the underlying resources visible and adjustable. Beanstalk is free; you pay for the resources.
- Platforms: Java, .NET (Windows/Linux), Node.js, Python, Ruby, PHP, Go, and Docker.
- **Environment tiers:** a **web server** environment (ELB + ASG) and a **worker** environment (an SQS queue plus a daemon that POSTs each message to your app; `cron.yaml` for periodic tasks). Offload long-running work from web to worker.
- Customize with `.ebextensions`, platform hooks, and saved configurations.
- ⚠️ An RDS database created *inside* an environment is tied to that environment's lifecycle. For production, create RDS separately and pass the connection details in.

**Deployment policies:**

| Policy | Downtime | Capacity during the deploy | Rollback | Extra cost | Notes |
|---|---|---|---|---|---|
| **All at once** | ✅ Yes, briefly | Drops to zero | Redeploy the old version | None | Fastest; dev/test |
| **Rolling** | None | **Reduced** by one batch at a time | Redeploy | None | Two versions run side by side during the rollout |
| **Rolling with additional batch** | None | **Full** (an extra batch is launched first) | Redeploy | Small (one batch) | Production without capacity loss |
| **Immutable** | None | Full: new instances in a temporary ASG | **Fast**: terminate the new instances | Double capacity temporarily | Safest in-place option |
| **Traffic splitting** | None | Full | Fast | Double temporarily | **Canary**: a percentage of traffic goes to the new version for an evaluation period |
| **Blue/green** (swap environment URLs) | None | A separate full environment | **Swap the CNAME back** | A whole second environment | Done outside the deployment policies; DNS-based cutover |

---

## 13. Hybrid and edge compute

| | **Outposts** | **Local Zones** | **Wavelength Zones** | **VMware Cloud on AWS** |
|---|---|---|---|---|
| What | AWS-owned, AWS-managed **racks (42U) or servers (1U/2U)** in your data center | AWS infrastructure in a **metro area**, an extension of a parent Region | AWS compute and storage **inside telco 5G networks** | VMware's SDDC stack (vSphere, vSAN, NSX) on dedicated bare-metal hosts in AWS Regions, operated and sold by VMware (Broadcom) |
| Latency target | Low latency to **on-prem systems** | Single-digit ms to **metro end users** | Ultra-low latency to **5G mobile devices** | — |
| Classic use | **Data residency**, local data processing, low latency to factory/hospital/trading systems, a migration stepping stone | Media and entertainment rendering, real-time gaming, live video, latency-sensitive hybrid apps | AR/VR, connected vehicles, live video at the edge, edge ML inference | **Move a VMware estate without re-platforming**, data center exit, VMware-based DR |
| Services | EC2, EBS, S3 on Outposts, ECS, EKS, RDS, ElastiCache, EMR, ALB (a subset) | EC2, EBS, ECS, EKS, ALB, FSx and more (varies by zone) | EC2, EBS, ECS, EKS (a subset) | VMware tooling, plus access to native AWS services from the SDDC |
| Connectivity | A service link to the parent Region (needs reliable connectivity) | Opt in; create a subnet in the Local Zone in your VPC | Extend a VPC subnet into the zone; a **carrier gateway** for mobile traffic | A connected VPC; VMware HCX for migration |
| Capacity | **Fixed**: what you ordered is what you have | Elastic within the zone | Elastic within the zone | Host-based |

Exam cues: "data must stay on-premises for regulatory reasons, but use AWS APIs and services" → **Outposts**. "Low-latency game servers for players in one city" → **Local Zone**. "Single-digit-ms latency to devices on a carrier's 5G network" → **Wavelength**. "Migrate 500 vSphere VMs quickly, keep VMware tools and skills" → **VMware Cloud on AWS**.

---

## 14. Rightsizing: Compute Optimizer

- **AWS Compute Optimizer** uses ML on CloudWatch utilization history (14 days by default; paid **enhanced infrastructure metrics** extend the lookback to approx. 3 months) to classify resources as **over-provisioned, under-provisioned or optimized**, with projected savings and performance risk.
- Covers **EC2 instances, Auto Scaling groups, EBS volumes, Lambda functions, ECS services on Fargate**, RDS/Aurora DB instances (newer), commercial software licences, and idle resources. Recommendations include **Graviton** migration options.
- ⚠️ **Memory utilization isn't collected by default** for EC2. Install the **CloudWatch agent** so memory-bound instances aren't recommended for downsizing.
- Opt in per account or org-wide (delegated administrator). The base service is free.
- Related tools: **Cost Optimization Hub** (consolidated recommendations), Cost Explorer rightsizing and Savings Plans recommendations, Trusted Advisor low-utilization checks, and scheduled stop/start of dev instances. All of it is in [module 10](10-cost-optimization.md).
- Order of operations for cost: **eliminate idle → rightsize (Compute Optimizer) → pick the right model (Graviton, Spot, serverless) → commit (Savings Plans)**.

---

## 15. How the exam tests this

| If the question says… | Think… |
|---|---|
| "Steady 24/7 workload, 3 years, may change instance families or move to Fargate" | **Compute Savings Plan** |
| "Steady workload on one family in one Region, maximum discount" | **EC2 Instance Savings Plan** (or Standard RI) |
| "Fault-tolerant batch/CI/rendering, minimize cost" | **Spot** (diversified, price-capacity-optimized) |
| "Guarantee capacity in a specific AZ for an event" | **On-Demand Capacity Reservation** |
| "Existing per-core licences (Oracle, SQL Server, Windows Server)" | **Dedicated Hosts** (+ License Manager) |
| "Single-tenant hardware, no licensing need" | **Dedicated Instances** |
| "HPC, lowest latency between nodes" | **Cluster placement group** + **EFA** |
| "Cassandra/Kafka/HDFS, rack-aware failure isolation" | **Partition placement group** |
| "A few critical instances must not share hardware" | **Spread placement group** |
| "T instance constantly out of CPU credits" | Move to **M/C** (or accept unlimited mode surplus charges) |
| "Scale on a known daily pattern" | **Scheduled** (or **predictive**) scaling |
| "Keep average CPU at 50%" | **Target tracking** |
| "Scale workers on SQS queue length" | Target tracking on **backlog per instance** |
| "Instances take a long time to boot during scale-out" | **Golden AMI**, **warm pools**, predictive scaling |
| "Run a script/drain before an instance terminates" | **Lifecycle hook** (`Terminating:Wait`) |
| "ASG doesn't replace instances whose app is broken" | Turn on **ELB health checks** for the ASG |
| "Static IPs for a load-balanced app" | **NLB** (EIPs) or **Global Accelerator** |
| "Route /api and /images to different services" | **ALB** path-based routing |
| "Authenticate users at the load balancer" | **ALB + Cognito/OIDC** |
| "Third-party firewall appliances, transparent insertion" | **GWLB** |
| "Many HTTPS domains on one load balancer" | **SNI** with multiple ACM certificates |
| "Users lose their session when an instance is replaced" | Store sessions in **ElastiCache/DynamoDB** (not stickiness) |
| "Lambda cold starts on Java, cost-sensitive" | **SnapStart** |
| "Lambda needs consistently low latency at peak" | **Provisioned concurrency** |
| "One Lambda function exhausts the DB / other functions get throttled" | **Reserved concurrency** (cap) + **RDS Proxy** |
| "VPC-attached Lambda can't reach the internet" | Private subnet + **NAT gateway** (or VPC endpoints) |
| "Rewrite URLs/headers at the edge, very high volume" | **CloudFront Functions** |
| "Edge logic that calls an external service or changes the origin" | **Lambda@Edge** |
| "Containers, least operational overhead" | **ECS on Fargate** |
| "Container can't pull its image or read its secret" | **Task execution role** permissions |
| "Container app gets AccessDenied on S3" | **Task role** permissions |
| "Must use Kubernetes / already have Helm charts" | **EKS** |
| "Run containers on existing on-prem servers, managed from AWS" | **ECS Anywhere** (or EKS Anywhere for K8s) |
| "Thousands of batch jobs with dependencies" | **AWS Batch** |
| "Upload code; AWS handles capacity, LB, scaling, health" | **Elastic Beanstalk** |
| "Deploy with zero downtime and fast rollback in Beanstalk" | **Immutable** or **blue/green (swap URLs)** |
| "Data must stay on-prem but use AWS services" | **Outposts** |
| "Ultra-low latency for 5G mobile users" | **Wavelength** |
| "Identify over-provisioned instances" | **Compute Optimizer** (+ CloudWatch agent for memory) |

---

## 16. Common traps

- ⚠️ Trap: **Instance store data is lost on stop or hibernate**, not only on terminate. Never put the only copy of data there.
- ⚠️ Trap: **Spread placement groups max out at 7 running instances per AZ.** "Place 20 instances on distinct hardware in one AZ" can't be done with one spread group.
- ⚠️ Trap: **Cluster placement groups are single-AZ.** Great performance, bad for HA.
- ⚠️ Trap: **Regional RIs don't reserve capacity**; only zonal RIs (and ODCRs) do. Savings Plans never reserve capacity.
- ⚠️ Trap: **Convertible RIs can't be sold** on the RI Marketplace; Standard RIs can.
- ⚠️ Trap: **Compute Savings Plans don't cover RDS.** Use RDS Reserved Instances ([module 05](05-databases.md)).
- ⚠️ Trap: **Spot isn't for databases or stateful singletons.** "Lowest cost" + "can't be interrupted" means On-Demand + Savings Plans, not Spot.
- ⚠️ Trap: An **ASG with only EC2 health checks** keeps a running-but-broken instance forever. Enable ELB health checks.
- ⚠️ Trap: **Cooldown periods apply to simple scaling.** Target tracking and step scaling use instance warm-up.
- ⚠️ Trap: **Scaling on raw SQS queue depth** is wrong. Use backlog per instance.
- ⚠️ Trap: **ALB has no static IP.** Put an NLB or Global Accelerator in front if allow-listing is required.
- ⚠️ Trap: **WAF can't attach to an NLB.** Use an ALB or CloudFront for WAF.
- ⚠️ Trap: **An ALB with all targets unhealthy fails open**, routing to all of them.
- ⚠️ Trap: **NLB cross-zone is off by default** and costs inter-AZ data when enabled. ALB cross-zone is on and free.
- ⚠️ Trap: **Lambda's 15-minute timeout is a hard limit.** Long jobs belong on Fargate, ECS, Batch or Step Functions orchestration.
- ⚠️ Trap: **Reserved concurrency doesn't remove cold starts**; provisioned concurrency does.
- ⚠️ Trap: **SnapStart can't be combined with provisioned concurrency.**
- ⚠️ Trap: **Lambda in a public subnet still has no internet access.** It needs NAT from a private subnet.
- ⚠️ Trap: **Task role ≠ task execution role.** The execution role starts the container; the task role is for your code.
- ⚠️ Trap: **Fargate supports only `awsvpc` networking**, and EKS Fargate pods can't run DaemonSets.
- ⚠️ Trap: **User data runs once** at first boot by default, not on every start, and must never contain secrets.
- ⚠️ Trap: **Launch configurations are legacy.** If an answer offers launch templates, take it.
- ⚠️ Trap: **Outposts capacity is fixed.** It doesn't burst into the Region by itself.

---

## 17. Quick self-check

1. A nightly genomics pipeline runs thousands of independent containerized jobs, each 30–90 minutes, and may be retried. What is the most cost-effective, low-ops design?
2. Why might a Compute Savings Plan beat an EC2 Instance Savings Plan for a team that's migrating from EC2 to Fargate next year?
3. An ASG processes SQS messages. Each message takes 2 s, and the business accepts 60 s of latency. What target value do you set, and on which metric?
4. An app needs static IPs for customer firewalls **and** host-based routing. Name two architectures.
5. ECS tasks on Fargate fail to start with `CannotPullContainerError` in a private subnet without a NAT gateway. Name the two likely fixes.

<details>
<summary>Answers</summary>

1. **AWS Batch** with a managed **Spot** (or Fargate Spot) compute environment, array jobs and a retry strategy. Batch scales to zero when the queue is empty. Lambda is out because of the 15-minute limit.
2. A **Compute Savings Plan** applies across instance families, Regions, **Fargate and Lambda**, so the commitment keeps paying off after the migration. An EC2 Instance Savings Plan covers only one EC2 family in one Region.
3. **30 messages per instance** (60 s ÷ 2 s), using target tracking on **backlog per instance** = `ApproximateNumberOfMessagesVisible` ÷ InService instances.
4. **NLB (with EIPs) → ALB** as the NLB's target, or **Global Accelerator → ALB** (static anycast IPs, plus multi-Region failover).
5. Give the **task execution role** ECR pull permissions (and CloudWatch Logs), and add VPC endpoints for **`ecr.api`, `ecr.dkr`, the S3 gateway endpoint** (image layers) and CloudWatch Logs, or add a NAT gateway route.

</details>

Practice: [D2 Resilient architectures questions](../practice/d2-resilient.md) · [D3 High-performing questions](../practice/d3-high-performing.md) · [D4 Cost-optimized questions](../practice/d4-cost-optimized.md) · Lab: [Lab 03 ALB + Auto Scaling](../labs/lab-03-alb-asg.md)
