# Storage: AWS services explained for beginners
> [Home](../../README.md) · [Beginner track](../README.md) · [All services A–Z](README.md) · Deep dive: [Notes 04](../../notes/04-storage.md)

Storage services keep your data somewhere durable. AWS offers three basic *shapes* of storage (object, block and file) and you pick the shape before you pick the service. On top of those sit services for archiving (S3 Glacier), connecting on-premises systems (Storage Gateway) and backing everything up (AWS Backup).

| Service | One-liner | Exam importance |
|---|---|---|
| [Amazon S3](#amazon-s3) | Unlimited object storage over HTTPS | ★★★ |
| [Amazon S3 Glacier](#amazon-s3-glacier) | The archive storage classes of S3: very cheap, slower to read | ★★★ |
| [Amazon EBS](#amazon-ebs) | Network-attached block disks for EC2, in one AZ | ★★★ |
| [EC2 instance store](#ec2-instance-store) | Fast local disks that vanish when the instance stops | ★★ |
| [Amazon EFS](#amazon-efs) | Managed, elastic NFS file system for Linux, multi-AZ | ★★★ |
| [Amazon FSx](#amazon-fsx) | Managed Windows, Lustre, NetApp ONTAP and OpenZFS file systems | ★★★ |
| [AWS Storage Gateway](#aws-storage-gateway) | On-premises appliance that fronts cloud storage with NFS/SMB/iSCSI | ★★★ |
| [AWS Backup](#aws-backup) | Central, policy-based backups across AWS services | ★★ |

### Object vs block vs file, in plain English

Think of a self-storage business:
- **Object storage (Amazon S3)** is a **valet storage desk**. You hand over a whole box with a label, and later ask for that box back by its label. You can't open the box at the desk and change one page; you replace the whole box. There's no limit on how many boxes you can store, and you talk to the desk over HTTPS.
- **Block storage (Amazon EBS, instance store)** is **a private hard drive bolted to one server**. The operating system formats it and edits any byte in place. It's fast, but only one machine uses it at a time.
- **File storage (Amazon EFS, Amazon FSx)** is **a shared network drive** (like a team's `S:` drive). Many machines mount the same folders at once and see each other's changes.

**The Kubernetes angle.** You've already met two of these through PersistentVolumes on EKS:
- An **EBS-backed PV** is **ReadWriteOnce**: one node mounts it, and it lives in one Availability Zone (AZ, an isolated data center within a Region).
- An **EFS-backed PV** is **ReadWriteMany**: pods on many nodes, in different AZs, mount it at the same time.
- S3 isn't a PV at all. Your app calls it through the AWS SDK, like calling any other API.

| | **Object** (S3) | **Block** (EBS, instance store) | **File** (EFS, FSx) |
|---|---|---|---|
| How you access it | HTTPS API: PUT/GET/DELETE whole objects | Attached disk; OS formats it (ext4, XFS, NTFS) | Mount over a network protocol: NFS (Linux) or SMB (Windows) |
| Who can use it at once | Anyone with permission, from anywhere | One instance (except io1/io2 Multi-Attach) | Thousands of clients |
| Edit in place? | ❌ Replace the whole object | ✅ | ✅ |
| Capacity | Unlimited, pay per GB stored | You provision a size and pay for all of it | EFS grows automatically; FSx is provisioned |
| Scope | A Region (data copied across ≥3 AZs) | One AZ | Multi-AZ (EFS Regional, FSx Multi-AZ) or one AZ |
| K8s equivalent | None (API call) | ReadWriteOnce PV | ReadWriteMany PV |
| Typical use | Backups, media, data lakes, static websites, logs | Boot disks, databases on one server | Shared content, home directories, lift-and-shift apps |

---

## Amazon S3

> **In one line:** Amazon Simple Storage Service (S3) stores any number of files ("objects") of almost any size and serves them over HTTPS with very high durability.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | The MinIO you might run in a cluster, but infinitely large and run by AWS. Everyday: a valet storage desk | Per GB-month stored (varies by storage class) + per request + data transfer out |

**What it is.** S3 is the default place to put data on AWS: uploads, backups, logs, images, data-lake files, static websites. You don't provision disks or capacity; you just write objects and pay for what's stored. Without it you'd run and replicate your own storage cluster, and keep adding disks.

### Buckets, objects and keys (no real folders)

- A **bucket** is a container for objects. Its name is **globally unique** across all AWS accounts, but the bucket lives in **one Region** you choose.
- An **object** is the data plus metadata, addressed by a **key**: the full "path", such as `invoices/2026/11/inv-42.pdf`.
- There are **no real folders**. The console shows `invoices/2026/` as a folder, but it's just a shared key **prefix**. Prefixes still matter: lifecycle rules, replication and event filters can target them, and request-rate limits apply per prefix (3,500 writes and 5,500 reads per second).
- S3 has **strong read-after-write consistency**: after a successful write, every read sees the new data.

### Storage classes

You pay less per GB when you promise to read data less often. Every class has the same 11-nines durability.

| Class | Plain-English use | Copies across | Minimum storage duration | Read speed |
|---|---|---|---|---|
| **S3 Standard** | Hot data read often | ≥3 AZs | None | Milliseconds |
| **S3 Intelligent-Tiering** | You don't know the access pattern; S3 moves objects between tiers for a small monitoring fee | ≥3 AZs | None | Milliseconds (optional archive tiers are slower) |
| **S3 Standard-IA** (Infrequent Access) | Read about monthly, must be instant (backups, DR copies) | ≥3 AZs | 30 days | Milliseconds, per-GB retrieval fee |
| **S3 One Zone-IA** | Same, but data you can re-create | **1 AZ** | 30 days | Milliseconds, per-GB retrieval fee |
| **S3 Glacier Instant Retrieval** | Archive read about quarterly, still instant | ≥3 AZs | 90 days | Milliseconds |
| **S3 Glacier Flexible Retrieval** | Archive; minutes to hours is fine | ≥3 AZs | 90 days | Minutes to hours (restore first) |
| **S3 Glacier Deep Archive** | Compliance archive kept for years; cheapest | ≥3 AZs | 180 days | Up to 12–48 hours (restore first) |
| **S3 Express One Zone** | Ultra-low latency for compute in the same AZ | **1 AZ** | None | Single-digit milliseconds |

### Versioning

Turn on **versioning** and S3 keeps every version of an object. Overwrites create a new version; a DELETE just adds a "delete marker" on top, so you can recover from mistakes and ransomware. Once enabled, versioning can be suspended but not removed. It's required for replication and Object Lock (write-once-read-many protection).

### Lifecycle rules

**Lifecycle rules** automate cost savings. A **transition** moves objects to a cheaper class after N days (for example Standard → Standard-IA at 30 days → Glacier Flexible Retrieval at 90). An **expiration** deletes them (for example logs after 365 days). Rules can target a prefix or tags, and can clean up old versions and abandoned multipart uploads.

### Permissions

Access is checked in layers, a bit like Kubernetes RBAC plus a NetworkPolicy:
- **Block Public Access**: an on-by-default safety switch at account and bucket level that overrides any policy trying to make data public.
- **IAM policies** (on users and roles): what *your* identities may do in S3.
- **Bucket policies** (on the bucket, in JSON): who may access *this bucket*, including other AWS accounts, a specific VPC endpoint, or only over HTTPS.
- ACLs are legacy and **disabled by default** on new buckets. **Presigned URLs** give someone temporary access to one object without AWS credentials.

### Encryption

- **SSE-S3** (server-side encryption with S3-managed keys) is **on by default** for every new object, at no extra cost.
- **SSE-KMS** uses a key in [AWS KMS](security.md#aws-kms) (Key Management Service), so you control who can decrypt and every use is logged. Pick it for compliance or separation of duties.
- **SSE-C** (customer-provided keys, sent with each request) is **disabled by default on new buckets since April 2026**; you must enable it explicitly.
- Encryption **in transit** is HTTPS; a bucket policy can deny plain HTTP.

### Static website hosting + CloudFront

S3 can serve a static site (HTML, JS, images) from a **website endpoint**, but that endpoint is **HTTP only** and needs a public bucket. The production pattern is to keep the bucket private and put [Amazon CloudFront](networking.md#amazon-cloudfront) (the AWS CDN, content delivery network) in front of it with **Origin Access Control (OAC)**. That gives you HTTPS, custom domains, edge caching and no public bucket.

### Durability vs availability

- **Durability** (will my data still exist?) is **99.999999999%, "11 nines"**, for every class. Store 10 million objects and you might lose one every 10,000 years.
- **Availability** (can I read it right now?) differs by class: 99.99% designed for Standard, lower for IA and One Zone classes.
- One Zone classes can lose data if that AZ is destroyed.

### Size limits

- Max object size is **50 TB** (raised from 5 TB in Dec 2025; older questions may say 5 TB).
- A single PUT can upload at most **5 GB**. Use **multipart upload** above about 100 MB; it's required above 5 GB.

### Event notifications

S3 can announce "object created/deleted/restored" to [AWS Lambda](compute.md#aws-lambda), [Amazon SQS](integration.md#amazon-sqs), [Amazon SNS](integration.md#amazon-sns) or [Amazon EventBridge](integration.md#amazon-eventbridge). This is the basis of patterns like "upload an image → Lambda makes a thumbnail".

**Use it when / don't use it when**
- ✅ Any amount of unstructured data: media, backups, logs, data lakes, static assets.
- ✅ Sharing data across many apps, Regions or accounts.
- ❌ A disk for an OS or database that edits blocks in place → [Amazon EBS](#amazon-ebs).
- ❌ A shared POSIX file system that apps mount → [Amazon EFS](#amazon-efs).

**Exam hot spots**
- **Pick the storage class from the access pattern**: unknown → Intelligent-Tiering; known decay → lifecycle rules; re-creatable → One Zone-IA.
- **Private bucket + CloudFront OAC** beats a public bucket for websites.
- Cross-account access needs **both** a bucket policy and the caller's IAM permission; SSE-KMS adds the KMS key policy.
- "Protect against accidental deletes" → **versioning** (+ MFA Delete or Object Lock for stronger guarantees).
- Fast long-distance uploads → **S3 Transfer Acceleration** or multipart upload.

**Try it** (≤10 min, free-tier level): S3 console → **Create bucket** (keep Block Public Access on) → upload a file → open **Properties** (versioning, default encryption) and **Permissions** (bucket policy) → enable versioning, upload the same file again and view **Show versions** → delete the bucket.

**Go deeper:** [Notes 04: S3 essentials](../../notes/04-storage.md#2-s3-essentials) · related services: [Amazon CloudFront](networking.md#amazon-cloudfront) · [Amazon Athena](analytics.md#amazon-athena) · [AWS KMS](security.md#aws-kms)

---

## Amazon S3 Glacier

> **In one line:** the three archive storage classes inside Amazon S3, for data you must keep but rarely read, at a fraction of S3 Standard's price.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | No K8s equivalent. Everyday: boxes moved from your garage to an off-site warehouse; cheap to keep, and you wait for a truck to bring one back | Very low per GB-month + per-GB retrieval fees + minimum storage duration (90 or 180 days) |

**What it is.** Today "S3 Glacier" means **storage classes you choose inside a normal S3 bucket**, either by uploading straight into them or by moving objects there with lifecycle rules. There was also an older, separate vault-based service (vaults and archives, its own API); it **stopped accepting new customers in December 2025** and is legacy. Exam questions mostly mean the S3 storage classes, though some still mention vaults and Vault Lock.

**Retrieval times**

| Class | Restore needed? | Retrieval options | Minimum storage duration |
|---|---|---|---|
| **S3 Glacier Instant Retrieval** | ❌ No, read directly | Milliseconds | 90 days |
| **S3 Glacier Flexible Retrieval** | ✅ Yes | **Expedited 1–5 min · Standard 3–5 h · Bulk 5–12 h** (bulk is free) | 90 days |
| **S3 Glacier Deep Archive** | ✅ Yes | **Standard within 12 h · Bulk within 48 h** | 180 days |

**Key ideas to know**
- **Restore**: for Flexible Retrieval and Deep Archive you call a restore request; S3 makes a temporary readable copy for the number of days you ask.
- **Minimum storage duration**: delete or move an object early and you're still billed for the remaining days.
- **Small objects cost extra** in Flexible/Deep Archive (per-object overhead), so bundle tiny files (zip/tar) first.
- **Object Lock** (on the bucket) gives write-once-read-many retention for compliance; **Vault Lock** was the equivalent for old vaults.

**Use it when / don't use it when**
- ✅ Compliance archives (7–10 years), old backups, tape replacement.
- ❌ Data needed instantly and often → S3 Standard or Standard-IA.

**Exam hot spots**
- Match the **retrieval time** in the question: "within milliseconds" → Instant Retrieval; "within minutes" → Flexible with Expedited; "within 12 hours", "lowest cost" → Deep Archive.
- "Retained 7 years, rarely accessed, cheapest" → lifecycle rule to **Deep Archive**.

**Go deeper:** [Notes 04: S3 storage classes](../../notes/04-storage.md#3-s3-storage-classes) · related services: [AWS Storage Gateway](#aws-storage-gateway) (Tape Gateway archives here)

---

## Amazon EBS

> **In one line:** Amazon Elastic Block Store (EBS) provides network-attached virtual disks (volumes) for EC2 instances, living in one Availability Zone.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | The disk behind a **ReadWriteOnce** PersistentVolume (via the EBS CSI driver). Everyday: an external USB drive you can unplug and plug into another PC in the same room | Per GB-month provisioned (+ provisioned IOPS/throughput for some types) + snapshot storage |

**What it is.** EBS volumes are the hard drives of EC2: boot volumes and data disks for databases and apps. They're network-attached, so they survive an instance stopping and can be detached and re-attached. Data is replicated within its AZ automatically.

**Volume types**

| Type | Kind | Max IOPS | Max throughput | Notes |
|---|---|---|---|---|
| **gp3** | General purpose SSD | **80,000** (3,000 baseline at any size) | 2,000 MiB/s | **The default.** Set IOPS and throughput independently of size |
| gp2 | General purpose SSD (older) | 16,000 (3 IOPS per GiB) | 250 MiB/s | Performance tied to size; migrate to gp3 to save money |
| **io2 Block Express** | Provisioned IOPS SSD | **256,000** | 4,000 MiB/s | Sub-millisecond latency, 99.999% durability, **Multi-Attach** |
| io1 | Provisioned IOPS SSD (older) | 64,000 | 1,000 MiB/s | Multi-Attach; prefer io2 |
| **st1** | Throughput-optimized HDD | n/a | 500 MiB/s | Big sequential reads (logs, big data). **Can't boot** |
| **sc1** | Cold HDD | n/a | 250 MiB/s | Cheapest; rarely accessed data. **Can't boot** |

**Key ideas to know**
- **AZ-scoped**: a volume lives in **one AZ** and can only attach to instances in that AZ. To move it, take a snapshot and create a new volume in the other AZ. (This is why an EBS-backed pod can't be rescheduled to another AZ.)
- **Snapshots**: point-in-time, **incremental** backups stored in S3 behind the scenes. They're Regional, can be copied to other Regions and shared with other accounts.
- **Encryption** with KMS covers the volume, its snapshots and volumes made from them. You can't encrypt an existing volume in place: snapshot → encrypted copy → new volume.
- **Elastic Volumes**: grow size or change type/IOPS while in use (no shrinking).
- **DeleteOnTermination**: the root volume is deleted with the instance by default; extra volumes are kept.
- **Multi-Attach** (io1/io2 only): one volume on up to 16 instances in the same AZ; needs a cluster-aware file system.

**Use it when / don't use it when**
- ✅ Boot disks, self-managed databases, any app needing a low-latency disk on one instance.
- ❌ Many instances or AZs sharing files → [Amazon EFS](#amazon-efs) or [Amazon FSx](#amazon-fsx).
- ❌ Temporary scratch at maximum speed → [EC2 instance store](#ec2-instance-store).

**Exam hot spots**
- "Lower EBS cost, same performance" → **gp2 to gp3**.
- Classic rule: **>16,000 IOPS or sub-ms latency → io2 Block Express** (gp3 now goes to 80,000 IOPS, so read newer questions carefully).
- Moving data to another AZ or Region → **snapshot** (copy for cross-Region).
- st1/sc1 are **not bootable**.

**Try it** (read-only): EC2 console → **Elastic Block Store → Volumes**. Note each volume's type, size, IOPS and **Availability Zone**.

**Go deeper:** [Notes 04: Amazon EBS](../../notes/04-storage.md#12-amazon-ebs) · related services: [Amazon EC2](compute.md#amazon-ec2) · [AWS Backup](#aws-backup)

---

## EC2 instance store

> **In one line:** disks physically attached to the server your EC2 instance runs on: extremely fast, but the data disappears when the instance stops.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like an `emptyDir` or local PV on a node's own SSD: fast, but gone if the node goes away. Everyday: a hotel room safe; it's empty again when you check out | Included in the instance price (only some instance types have it) |

**What it is.** Some instance types (storage-optimized families and types with a `d` in the name) come with local NVMe/SSD disks. They offer the highest IOPS and lowest latency on EC2, but nothing is replicated or backed up for you.

**Key ideas to know**
- Data **survives a reboot** but is **lost on stop, hibernate, terminate or host failure**.
- You can't snapshot, detach or resize it.
- Good for caches, buffers, scratch files, and databases that replicate data across nodes themselves (Cassandra, Kafka).

**Exam hot spots**
- "Highest IOPS, temporary data or data replicated elsewhere" → **instance store**.
- "Data must persist after stop" → **EBS**, not instance store.

**Go deeper:** [Notes 04: Instance store](../../notes/04-storage.md#13-instance-store) · related services: [Amazon EC2](compute.md#amazon-ec2)

---

## Amazon EFS

> **In one line:** Amazon Elastic File System (EFS) is a managed, auto-growing NFS file system that many Linux machines across AZs can mount at once.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | The backend of a **ReadWriteMany** PersistentVolume (EFS CSI driver). Everyday: a shared team drive | Per GB-month actually stored (cheaper classes for cold files) + data transferred with Elastic throughput |

**What it is.** EFS gives you a shared POSIX file system without running NFS servers. You don't provision size; it grows and shrinks with your files. EC2, ECS, EKS, Fargate, Lambda and on-premises servers (over VPN or Direct Connect) can all mount it.

**Key ideas to know**
- **NFS, Linux only.** Windows clients need FSx for Windows File Server.
- **Mount targets**: one per AZ in your VPC subnets, protected by security groups (allow TCP 2049).
- **Regional vs One Zone**: Regional keeps data across multiple AZs; One Zone is cheaper for non-critical data.
- **Storage classes + lifecycle**: Standard, Infrequent Access and Archive; lifecycle moves untouched files to cheaper classes automatically.
- **Throughput modes**: **Elastic** (default, scales automatically), Provisioned, or legacy Bursting.
- **Access points**: per-app entry points that force a user ID and root directory; handy for containers.
- Costs several times more per GB than EBS gp3, so use it when sharing is the point.

**Use it when / don't use it when**
- ✅ Many Linux instances or pods need the same files (CMS uploads, shared config, ML data).
- ❌ Windows/SMB → [Amazon FSx](#amazon-fsx). One instance only → [Amazon EBS](#amazon-ebs) (cheaper).

**Exam hot spots**
- "Shared file system, Linux, multiple AZs, scales automatically" → **EFS**.
- Persistent shared storage for **Fargate** tasks or EKS pods → EFS.
- Reduce cost for rarely read files → **EFS lifecycle to IA/Archive**.

**Go deeper:** [Notes 04: Amazon EFS](../../notes/04-storage.md#14-amazon-efs) · related services: [AWS Fargate](containers.md#aws-fargate) · [Amazon EKS](containers.md#amazon-eks)

---

## Amazon FSx

> **In one line:** fully managed versions of four popular third-party file systems (Windows File Server, Lustre, NetApp ONTAP and OpenZFS).

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | Specialised RWX storage backends (each has a CSI driver). Everyday: renting a ready-made, branded file server instead of building one | Per GB-month of provisioned storage + throughput capacity (+ backups) |

**What it is.** When EFS doesn't fit (Windows, extreme HPC speed, or an existing NetApp/ZFS setup), FSx gives you that exact file system as a managed service. AWS handles hardware, patching, replication and backups; you get the familiar features and protocols.

| Flavour | Speaks | Pick it when the question says… |
|---|---|---|
| **FSx for Windows File Server** | SMB, Active Directory, NTFS permissions, DFS | "Windows", "SMB", "Active Directory", lift-and-shift of a Windows file share; Multi-AZ available |
| **FSx for Lustre** | Lustre (Linux, parallel) | "HPC", "ML training", "sub-millisecond, hundreds of GB/s"; can **link to an S3 bucket** and load objects as files. Scratch (cheap, temporary) or Persistent |
| **FSx for NetApp ONTAP** | NFS, SMB **and** iSCSI | "Migrate NetApp", "multi-protocol", SnapMirror, data deduplication |
| **FSx for OpenZFS** | NFS | "Migrate ZFS/Linux NFS servers", snapshots and instant clones |

**Use it when / don't use it when**
- ✅ The workload names a specific file system or protocol from the table.
- ❌ Plain shared Linux NFS with automatic scaling → [Amazon EFS](#amazon-efs) is simpler.

**Exam hot spots**
- Windows + AD → **FSx for Windows**. HPC on data in S3 → **FSx for Lustre**.
- "NFS + SMB on the same data" or NetApp → **FSx for NetApp ONTAP**.

**Go deeper:** [Notes 04: Amazon FSx family](../../notes/04-storage.md#15-amazon-fsx-family) · related services: [AWS Directory Service](identity-governance.md#aws-directory-service)

---

## AWS Storage Gateway

> **In one line:** a virtual appliance you run on-premises that lets local servers use AWS storage through ordinary file, block or tape protocols.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | No direct K8s match. Everyday: a local post office branch; you drop things off locally, and a truck carries them to the central warehouse (AWS) | Per GB stored in AWS + requests and data transfer (details vary by gateway type) |

**What it is.** Many companies have on-premises apps that expect a file share, an iSCSI disk or a tape library. Storage Gateway runs as a VM (or hardware appliance) in their data center, keeps a **local cache** for speed, and stores the data in S3, FSx or EBS snapshots. It's for **ongoing hybrid** use, not one-off migrations.

| Type | On-premises sees | Stored in AWS as | Typical use |
|---|---|---|---|
| **S3 File Gateway** | NFS / SMB file share | Objects in S3 (one file = one object) | File shares backed by S3; backups into S3 |
| **FSx File Gateway** | SMB share | FSx for Windows File Server | Branch-office cache. ⚠️ Closed to new customers since Oct 2024 |
| **Volume Gateway: cached** | iSCSI disk | S3, with EBS snapshots; only hot data kept locally | Shrink on-prem storage |
| **Volume Gateway: stored** | iSCSI disk | Full copy on-prem, async backups as EBS snapshots | Keep everything local, back up to AWS |
| **Tape Gateway** | Virtual tape library | S3, archived to **Glacier / Deep Archive** | Replace physical tapes without changing backup software |

**Exam hot spots**
- "Replace tapes, keep existing backup software" → **Tape Gateway**.
- "On-premises apps need NFS/SMB access to S3" → **S3 File Gateway**.
- One-time bulk migration → [AWS DataSync](migration-transfer.md#aws-datasync) or [AWS Snow Family](migration-transfer.md#aws-snow-family), **not** Storage Gateway.

**Go deeper:** [Notes 04: AWS Storage Gateway](../../notes/04-storage.md#16-aws-storage-gateway) · related services: [AWS Direct Connect](networking.md#aws-direct-connect)

---

## AWS Backup

> **In one line:** one place to schedule, retain, copy and audit backups for many AWS services, across accounts and Regions.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like Velero schedules, but for AWS resources (EBS, RDS, EFS, DynamoDB, S3…) | Per GB of backup storage (+ restores, cross-Region copy transfer) |

**What it is.** Instead of setting snapshot schedules separately in EC2, RDS, EFS and DynamoDB, you write a **backup plan** (schedule, retention, copies) and assign resources to it by tag. Backups land in encrypted **backup vaults**, and AWS Backup can copy them to another Region or account for disaster recovery.

**Key ideas to know**
- **Backup plan** → **resource assignment** (by tag or ID) → **backup vault**.
- **Cross-Region and cross-account copies** protect against Regional outages and a compromised account.
- **Vault Lock** (compliance mode) makes backups undeletable, even by root, until retention ends: a ransomware defence.
- With AWS Organizations, **backup policies** apply plans across every account.

**Exam hot spots**
- "Centrally manage and audit backups across services and accounts" → **AWS Backup**.
- "Backups must be immutable" → **AWS Backup Vault Lock** (or S3 Object Lock for objects).

**Go deeper:** [Notes 04: AWS Backup](../../notes/04-storage.md#17-aws-backup) · related services: [AWS Organizations](identity-governance.md#aws-organizations) · [AWS Elastic Disaster Recovery](migration-transfer.md#aws-elastic-disaster-recovery)

---

**Next:** [Databases](databases.md)
