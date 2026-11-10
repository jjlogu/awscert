# Migration & data transfer: AWS services explained for beginners
> [Home](../../README.md) · [Beginner track](../README.md) · [All services A–Z](README.md) · Deep dive: [Notes 09](../../notes/09-data-analytics-migration.md) · [Notes 08](../../notes/08-ha-dr-edge-ops.md)

These services move things **into** AWS: files, databases, whole servers, or terabytes of data on a shipped device. A few also keep a copy of on-premises servers ready in AWS for disaster recovery. On the exam, the hard part is rarely the service itself; it's picking the right one from a short description: **What** is moving (files, a database, servers)? **How much**, over **how fast** a network link? **Once** or **ongoing**? The two tables below answer most questions. Storage names you'll see: **S3** (Simple Storage Service, object storage), **EFS** (Elastic File System, shared NFS file storage) and **FSx** (managed Windows, Lustre, NetApp ONTAP and OpenZFS file systems).

| Service | One-liner | Exam importance |
|---|---|---|
| [AWS DataSync](#aws-datasync) | Online copying of files and objects (NFS, SMB, HDFS, object stores) into S3, EFS or FSx | ★★★ |
| [AWS Transfer Family](#aws-transfer-family) | Managed SFTP/FTPS/FTP/AS2 servers that store files in S3 or EFS | ★★ |
| [AWS Snow Family](#aws-snow-family) | Rugged devices shipped to you for offline bulk transfer and edge computing | ★★★ |
| [AWS Application Migration Service](#aws-application-migration-service) | Lift-and-shift of whole servers into EC2 with minimal downtime | ★★ |
| [AWS Application Discovery Service](#aws-application-discovery-service) | Inventory and dependency mapping of on-premises servers before migrating | ★ |
| [AWS DMS](#aws-dms) | Database migration and ongoing replication with minimal downtime | ★★★ |
| [AWS Migration Hub](#aws-migration-hub) | A console to track migration progress across tools | ★ |
| [AWS Elastic Disaster Recovery](#aws-elastic-disaster-recovery) | Cheap standby replication of servers into AWS for fast recovery | ★★ |

---

### Beginner concepts first

#### The 7 Rs of migration (plain English)

When a company moves to AWS, each application gets one of seven treatments. Think of **moving house**:

| R | Moving-house version | What it means in IT | Example / AWS tool |
|---|---|---|---|
| **Retire** | Throw it away | Switch off apps nobody needs | An old reporting server nobody logs into |
| **Retain** | Leave it in the old house for now | Keep it on-premises (not ready, recently upgraded, or a compliance reason) | A mainframe with years left on its contract |
| **Rehost** ("lift and shift") | Move the furniture exactly as it is | Copy servers as-is onto EC2 (Elastic Compute Cloud, AWS virtual servers) | [AWS Application Migration Service](#aws-application-migration-service) |
| **Relocate** | Move the whole room in one piece | Move VMware virtual machines at the hypervisor level, without converting them | [VMware Cloud on AWS](compute.md#vmware-cloud-on-aws) |
| **Replatform** ("lift, tinker and shift") | Move, but swap the old stove for a new built-in one | Small changes to gain managed services, e.g., self-run MySQL → Amazon RDS (Relational Database Service) | [AWS DMS](#aws-dms) into [Amazon RDS](databases.md#amazon-rds) |
| **Repurchase** ("drop and shop") | Sell the old sofa, buy a new one | Replace with a SaaS (software as a service) product | A self-hosted CRM (customer relationship management) system replaced by a hosted one |
| **Refactor / re-architect** | Rebuild the house to a new design | Redesign the app as cloud-native | A monolith rebuilt as containers or Lambda + DynamoDB + SQS |

**Kubernetes bridge.** If your team moved an app from VMs (virtual machines) into EKS by containerizing it, that was closer to **replatform** or **refactor**. Moving the same VMs onto EC2 unchanged would be **rehost**.

Exam clue words: "as quickly as possible, no code changes" → **rehost**. "Reduce operational overhead of the database with minimal changes" → **replatform**. "Take full advantage of cloud-native features" → **refactor**.

#### Which transfer service?

| What's moving | Online or offline | One-time or ongoing | Service |
|---|---|---|---|
| Files/objects from NFS (Network File System), SMB (Server Message Block, Windows shares), HDFS or another cloud | Online | One-time **or** scheduled/ongoing | [AWS DataSync](#aws-datasync) |
| Files that **partners upload** over SFTP/FTPS/FTP/AS2 | Online | Ongoing | [AWS Transfer Family](#aws-transfer-family) |
| On-premises apps that must **keep using** a local file share or tape while data lives in AWS | Online | Ongoing (hybrid) | [AWS Storage Gateway](storage.md#aws-storage-gateway) |
| Tens of TB to PB with a slow or no network link | **Offline** (ship a device) | One-time | [AWS Snow Family](#aws-snow-family) (see the availability note) |
| A **database** (Oracle, SQL Server, MySQL, PostgreSQL…) | Online | One-time + change replication until cutover, or ongoing | [AWS DMS](#aws-dms) |
| Whole **servers** (VMware, Hyper-V, physical, other clouds) | Online | One-time cutover | [AWS Application Migration Service](#aws-application-migration-service) |
| Whole servers kept ready for **disaster recovery** | Online | Ongoing | [AWS Elastic Disaster Recovery](#aws-elastic-disaster-recovery) |
| Large, steady, recurring transfers over a private link | Online (dedicated line) | Ongoing | [AWS Direct Connect](networking.md#aws-direct-connect) (often with DataSync on top) |

#### "How long would the network take?" (rule of thumb)

At **full** speed, **1 Gbps moves about 1 TB in roughly 2.2 hours** (1 TB = 8,000 gigabits ÷ 1 gigabit/s ≈ 8,000 s). Real links rarely sustain more than about 70–80% for days, and you usually can't starve production traffic, so pad the estimate.

| Data | 100 Mbps | 1 Gbps | 10 Gbps |
|---|---|---|---|
| 1 TB | ~22 hours | ~2.2 hours | ~13 minutes |
| 10 TB | ~9 days | ~22 hours | ~2.2 hours |
| 100 TB | ~3 months | ~9–10 days | ~22 hours |
| 1 PB | ~2.5 years | ~3 months | ~9–10 days |

**The decision:** compare the network time with roughly **a week or more** for a shipped-device round trip (order, deliver, load, ship back, import).
- Network takes **weeks or months** → ship the data (**offline** transfer).
- Network takes **hours to days** and it's a one-off → **DataSync** over the existing link.
- The transfer is large and **recurring** → consider **Direct Connect**.

---

## AWS DataSync

> **In one line:** a managed online service that copies files and objects between on-premises storage, other clouds and AWS storage, quickly, securely and on a schedule.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | Like a Kubernetes `Job` (or `CronJob`) running `rsync`/`rclone`, but with built-in parallelism, scheduling, encryption and integrity checks. Everyday: a professional moving crew with a checklist that verifies every box arrived | Per GB transferred (plus normal storage and network costs) |

**What it is.** AWS DataSync moves large numbers of files from **NFS** (Network File System) or **SMB** (Server Message Block, Windows shares) servers, **HDFS** (Hadoop Distributed File System), self-managed object storage or other clouds into **Amazon S3, Amazon EFS or Amazon FSx**, and between AWS storage services. For on-premises sources you deploy a small **agent** (a VM) near the data. Without DataSync you'd write and babysit copy scripts, handle retries, parallelism and verification, and hope permissions and timestamps survive.

**Key ideas to know**
- **Agent.** A virtual machine you run on-premises (or on EC2) that reads the source and sends data to AWS. AWS-to-AWS copies don't need one.
- **Location.** A source or destination (e.g., an NFS share, an S3 bucket, an EFS file system).
- **Task.** A copy job from one location to another, with options such as filters and **bandwidth throttling** (so you don't flood the office link).
- **Scheduling and incremental copies.** Run hourly/daily; only changed files are sent after the first run.
- **Integrity verification and encryption.** Data is checked after transfer and encrypted in transit with TLS (Transport Layer Security).
- **Network path.** Works over the internet, a VPN (virtual private network) or Direct Connect, optionally through a VPC endpoint so traffic stays private.
- **Metadata preservation.** File permissions and timestamps are kept where the destination supports them.

**Use it when / don't use it when**
- ✅ Migrate a file share or NAS (network-attached storage) to S3/EFS/FSx, once or on a schedule.
- ✅ Replicate data between AWS storage services or Regions, or pull data from another cloud.
- ❌ On-premises apps need to keep reading and writing a local share that's backed by AWS → [AWS Storage Gateway](storage.md#aws-storage-gateway).
- ❌ External partners upload files over SFTP → [AWS Transfer Family](#aws-transfer-family).
- ❌ The network would take weeks → offline transfer (see [AWS Snow Family](#aws-snow-family)).
- ❌ Moving a database → [AWS DMS](#aws-dms).

**Exam hot spots**
- "Migrate/copy NFS or SMB data to S3, EFS or FSx, **automate, encrypt, verify**, with least effort" → **DataSync**.
- "Limit how much bandwidth the copy uses" → DataSync **bandwidth throttling**.
- ⚠️ Trap: Storage Gateway vs DataSync. **DataSync = moving data**; **Storage Gateway = ongoing hybrid access** to cloud storage.
- DataSync + Direct Connect is the classic "fast, private, recurring" combination.

**Try it** (read-only): open the DataSync console → **Tasks → Create task** and look at the source location types (NFS, SMB, HDFS, object storage, other clouds) and the task options (verification, bandwidth limit, schedule). Cancel without creating anything.

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · related services: [Amazon S3](storage.md#amazon-s3) · [Amazon EFS](storage.md#amazon-efs) · [Amazon FSx](storage.md#amazon-fsx) · [AWS Direct Connect](networking.md#aws-direct-connect)

---

## AWS Transfer Family

> **In one line:** fully managed SFTP, FTPS, FTP and AS2 servers whose files land directly in Amazon S3 or Amazon EFS.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like an SFTP server Pod with an S3-backed volume, except AWS runs and scales it. Everyday: a mailbox slot partners already know how to use, emptying into your warehouse | Per hour per enabled protocol endpoint, plus per GB uploaded/downloaded |

**What it is.** Many businesses exchange files with partners using old but universal protocols: **SFTP** (SSH File Transfer Protocol, where SSH is Secure Shell), **FTPS** (FTP over TLS), plain **FTP** and **AS2** (Applicability Statement 2, used for business-to-business electronic document exchange, such as purchase orders and invoices). Transfer Family gives you a managed endpoint for these, storing files in S3 or EFS. Partners keep their existing clients and scripts; you don't run file servers.

**Key ideas to know**
- **Identity.** Users authenticate with service-managed SSH keys, Microsoft Active Directory, or a custom identity provider (via Lambda or API Gateway).
- **Endpoint.** Public, or inside your VPC (Virtual Private Cloud). Plain FTP is only allowed inside a VPC.
- **Managed workflows.** Run post-upload steps (copy, tag, decrypt, call Lambda).

**Use it when / don't use it when**
- ✅ "Partners upload files via SFTP and we want them in S3 without managing servers".
- ❌ Bulk migration of your own file shares → [AWS DataSync](#aws-datasync).

**Exam hot spots**
- **"SFTP" + "S3" + "managed"** → **Transfer Family**. ⚠️ Trap: running an SFTP server on EC2 works but has more operational overhead.

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · related services: [Amazon S3](storage.md#amazon-s3) · [Amazon EFS](storage.md#amazon-efs)

---

## AWS Snow Family

> **In one line:** rugged, encrypted storage-and-compute devices that AWS ships to you, so you can load huge amounts of data locally and ship it back to be imported into S3, or run compute where there's no network.

⚠️ **Availability (real world vs exam):** since **November 7, 2025, AWS Snow Family devices (Snowball Edge) are no longer available to new customers**; existing customers can continue to use them. Snowcone and Snowmobile were discontinued earlier. The exam guide still lists Snow Family, so you must still learn the **"offline bulk transfer when the network is too slow"** decision. In the real world, new customers use **AWS DataSync** (online), **AWS Data Transfer Terminal** (physical AWS locations where you bring your own storage devices and upload over a very fast connection), or **AWS Partner** transfer services; for edge compute AWS points to [AWS Outposts](compute.md#aws-outposts).

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High (for the exam decision) | No Kubernetes equivalent. Everyday: instead of uploading your photo library over slow hotel Wi-Fi, you mail a hard drive. "Never underestimate the bandwidth of a truck full of disks" | Per job (service fee) plus per day on site after a few included days, plus shipping; importing into S3 has no data-transfer-in charge |

**What it is.** When data is huge and the network is slow, expensive or missing, **physically shipping** it is faster. With **Snowball Edge**, you create a job in the AWS console; AWS ships a suitcase-sized, tamper-resistant device; you plug it into your local network and copy data onto it; you ship it back; AWS imports the data into an **S3 bucket** and then securely wipes the device. Without it, you'd either wait months for a network transfer or buy your own disks and arrange secure courier logistics.

**Key ideas to know**
- **Snowball Edge Storage Optimized.** Mostly storage, for **bulk data migration** (each device holds tens to hundreds of TB; use several devices, or a cluster, for petabytes).
- **Snowball Edge Compute Optimized.** More CPU/GPU for **edge computing** in disconnected places (ships, mines, field sites): run EC2-compatible instances on the device, then send results home.
- **Job types.** *Import into S3*, *export from S3*, and *local compute and storage*.
- **Security.** Data is encrypted with keys managed in **AWS KMS** (Key Management Service); the keys never ship with the device. The device is tamper-evident and is erased after import.
- **Loading data.** Use the Snowball Edge client or its S3-compatible interface over your local network. DataSync can also write to the device.
- **Imports land in S3.** You can't import straight into S3 Glacier storage classes; land in S3, then use an **S3 Lifecycle rule** to move data to Glacier.

**Use it when / don't use it when**
- ✅ (Exam) Tens of TB to PB must move and the network would take **weeks or more**, or there's **no connectivity**.
- ✅ (Exam) Edge processing in a remote site with little or no network.
- ❌ The network can do it in days → [AWS DataSync](#aws-datasync).
- ❌ Ongoing, recurring large transfers → [AWS Direct Connect](networking.md#aws-direct-connect) + DataSync.
- ❌ Ongoing local access to cloud data → [AWS Storage Gateway](storage.md#aws-storage-gateway).

**Exam hot spots**
- Do the arithmetic: e.g., "200 TB over a 100 Mbps link that's already 50% used" is many months online → **Snowball Edge**.
- "Huge database migration with little bandwidth" → Snowball Edge for the bulk load + **AWS DMS** change replication over the network.
- "Archive to Glacier" via Snowball → import to **S3**, then **lifecycle** to Glacier.
- ⚠️ Trap: Snowball is for **one-time or occasional** bulk moves, not continuous replication.

**Try it** (read-only): there's nothing to try cheaply. Open the AWS Snow Family product page and the [AWS Snowball Edge availability change](https://docs.aws.amazon.com/snowball/latest/developer-guide/snowball-edge-availability-change.html) notice to see the current status and alternatives.

**Go deeper:** [Notes 09: Transfer-time arithmetic and transfer services](../../notes/09-data-analytics-migration.md) · related services: [AWS DataSync](#aws-datasync) · [Amazon S3](storage.md#amazon-s3) · [Amazon S3 Glacier](storage.md#amazon-s3-glacier) · [AWS KMS](security.md#aws-kms)

---

## AWS Application Migration Service

> **In one line:** AWS's main "lift-and-shift" tool: it continuously replicates whole servers into AWS and lets you cut over to EC2 with minutes of downtime.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Loosely like copying a PersistentVolume block by block to a new cluster while the app keeps running, then switching traffic over. Everyday: a moving company builds an exact copy of your house next door, keeps it in sync, and you move in on cutover day | Free for a limited period per source server (approx. 90 days); you pay for the staging and launched EC2/EBS resources |

**What it is.** Often called **MGN** (its short name in docs and the console), it installs an **agent** on each source server (physical, VMware, Hyper-V, or VMs in other clouds). The agent continuously replicates the disks at the **block level** to a low-cost **staging area** in your AWS account. You launch **test instances** to check everything works, then do a **cutover**: launch the final EC2 instances and redirect users, with only minutes of downtime. Without it, you'd rebuild servers by hand or export and convert VM images.

**Key ideas to know**
- **Continuous block-level replication.** Disks stay in sync until you cut over.
- **Staging area.** Small replication servers and cheap EBS (Elastic Block Store) volumes; full-size instances only run for tests and cutover.
- **Test, then cut over.** Non-disruptive test launches before the real switch.
- **Rehost.** It's the tool for the **rehost** "R"; it doesn't change your app.

**Use it when / don't use it when**
- ✅ Migrate many servers to EC2 quickly, with minimal downtime and no code changes.
- ❌ Moving only a database → [AWS DMS](#aws-dms). Disaster-recovery standby → [AWS Elastic Disaster Recovery](#aws-elastic-disaster-recovery).

**Exam hot spots**
- "Lift and shift / rehost hundreds of VMs (physical, VMware, Hyper-V) to AWS with minimal downtime" → **Application Migration Service**.
- ⚠️ Trap: older material mentions **Server Migration Service (SMS)**; it's retired, and MGN is the replacement.

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · related services: [Amazon EC2](compute.md#amazon-ec2) · [AWS Elastic Disaster Recovery](#aws-elastic-disaster-recovery)

---

## AWS Application Discovery Service

> **In one line:** collects inventory, performance and network-dependency data from on-premises servers to plan a migration.

⚠️ **Availability:** according to AWS documentation, Application Discovery Service **stopped accepting new customers on November 7, 2025**; existing customers can finish their projects, and AWS recommends **AWS Transform** for new discovery and assessment work. It's still on the exam's in-scope list, so know what it does.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | The survey before moving house: list every item, how heavy it is, and which things must move together | No charge for the service itself |

**What it is.** Before migrating, you need to know what servers exist, how busy they are, and which ones talk to each other. The **agentless collector** (a VM in VMware) gathers VM inventory and performance; the **Discovery Agent** installed on each server also records running processes and **network connections**, which reveal dependencies.

**Exam hot spots**
- "Identify **dependencies** between on-premises servers before migrating" → the agent-based **Discovery Agent**.

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · related services: [AWS Migration Hub](#aws-migration-hub)

---

## AWS DMS

> **In one line:** AWS Database Migration Service migrates databases to AWS (and keeps them in sync while you switch over) with minimal downtime, even between different database engines.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | Like a database replica that follows the primary, then gets promoted to be the new primary at cutover. Everyday: copying a busy shop's stock list to a new shop while still selling, then switching the sign on the door | Per replication instance-hour (or DMS Serverless capacity), plus storage |

**What it is.** AWS DMS reads from a **source** database and writes to a **target**. The source **stays online** during the migration. DMS first copies existing data (**full load**), then streams ongoing changes (**CDC**, change data capture: reading the database's transaction log) until you're ready to switch the application over. Without it, you'd take a long outage for a dump-and-restore, or build your own replication.

**Key ideas to know**
- **Homogeneous migration.** Same engine on both sides (e.g., MySQL → RDS for MySQL or Aurora MySQL).
- **Heterogeneous migration.** Different engines (e.g., Oracle → Aurora PostgreSQL). The **schema** (tables, views, stored procedures) must be converted first with **DMS Schema Conversion** or the **AWS Schema Conversion Tool (SCT)**; DMS then moves the data.
- **Replication instance.** The managed server that runs migration tasks (or use **DMS Serverless**, which sizes capacity for you).
- **Endpoints.** Connection settings for the source and target. Targets include RDS, Aurora, Redshift, DynamoDB, S3, OpenSearch, Kinesis and more.
- **Task types.** *Full load*, *full load + CDC* (the usual minimal-downtime choice), *CDC only*.
- **Ongoing replication.** DMS can keep running to feed a data lake (S3) or warehouse continuously.

**Use it when / don't use it when**
- ✅ Migrate a database to RDS, Aurora, Redshift or DynamoDB **with minimal downtime**.
- ✅ Change engines (with schema conversion first).
- ✅ Continuously replicate a database into S3 or Redshift for analytics.
- ❌ Migrating whole servers (OS + apps) → [AWS Application Migration Service](#aws-application-migration-service).
- ❌ Copying file shares → [AWS DataSync](#aws-datasync).

**Exam hot spots**
- "Migrate database with **minimal downtime**" → **DMS full load + CDC**.
- "Oracle/SQL Server → Aurora PostgreSQL" → **SCT or DMS Schema Conversion** for the schema, **DMS** for the data. ⚠️ Trap: DMS alone doesn't convert stored procedures and code.
- "Multi-TB database, slow link" → bulk load offline (Snowball Edge in exam terms), then DMS CDC to catch up.
- DMS can target S3, which makes it a common "database → data lake" ingestion answer.

**Try it** (read-only, to avoid instance charges): open the DMS console and look at **Endpoints** (the list of supported source and target engines) and the **Migration task** options (full load, full load + CDC, CDC only).

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · [Notes 05: Databases](../../notes/05-databases.md) · related services: [Amazon RDS](databases.md#amazon-rds) · [Amazon Aurora](databases.md#amazon-aurora) · [Amazon Redshift](databases.md#amazon-redshift)

---

## AWS Migration Hub

> **In one line:** a single console to track the progress of application migrations across tools such as Application Migration Service and DMS.

⚠️ **Availability:** according to the AWS Migration Hub documentation, the service **is no longer open to new customers as of November 7, 2025**. Existing customers can keep using it to finish in-flight projects (security updates continue, but no new features), and AWS directs new projects to **AWS Transform**, which it describes as offering the same capabilities plus AI-assisted planning. Older practice questions may still mention Migration Hub; treat it as "the tracking dashboard".

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | A project-tracking board for the whole move: which boxes are packed, in transit, delivered | No charge for the hub itself |

**What it is.** Migration Hub pulls status from discovery and migration tools into one view, and groups servers into applications. It also offered strategy recommendations and orchestration templates. It **doesn't move anything** itself.

**Exam hot spots**
- "Track migration status across multiple tools in one place" → **Migration Hub**. ⚠️ Trap: it is never the tool that actually copies servers or data.

**Go deeper:** [Notes 09: Transfer and migration services](../../notes/09-data-analytics-migration.md) · related services: [AWS Application Discovery Service](#aws-application-discovery-service) · [AWS Application Migration Service](#aws-application-migration-service)

---

## AWS Elastic Disaster Recovery

> **In one line:** keeps a continuously replicated, low-cost copy of your servers in AWS so you can recover them within minutes if the primary site fails.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Loosely like Velero-style recovery to a standby cluster, but for whole servers, with continuous block-level replication instead of periodic backups. Everyday: a spare office kept dusted and wired, with copies of every file updated constantly, but no staff until you need it | Per replicating source server per hour, plus staging storage, plus full instances only during drills or recovery |

**What it is.** Often called **DRS**, it uses the same replication technology as Application Migration Service, but for **disaster recovery** instead of a one-way move. An agent on each source server (on-premises, other cloud, or EC2 in another Region) streams disk changes into a cheap **staging area**. In a disaster or a drill, DRS launches full-size recovery instances in minutes. Later you can **fail back** to the original site.

**Key ideas to know**
- **RPO** (recovery point objective: how much recent data you can afford to lose) of **seconds**, and **RTO** (recovery time objective: how long you can be down) of **minutes**.
- **Low idle cost.** Only small replication servers and storage run until you need to recover.
- **Point-in-time recovery.** Recover to an earlier moment, e.g., before a ransomware attack.
- **Drills.** Launch recovery instances for testing without disrupting replication.

**Use it when / don't use it when**
- ✅ DR for on-premises or EC2 servers with low RPO/RTO and minimal standby cost.
- ❌ A one-way migration → [AWS Application Migration Service](#aws-application-migration-service).
- ❌ Backing up data with retention policies (hours of RPO acceptable) → [AWS Backup](storage.md#aws-backup).

**Exam hot spots**
- "Disaster recovery for **on-premises servers** into AWS, **minimal infrastructure cost** while idle, recovery in minutes" → **Elastic Disaster Recovery**.
- ⚠️ Trap: MGN vs DRS. Same technology; **MGN = migrate once**, **DRS = stay ready to recover**.

**Go deeper:** [Notes 08: AWS Elastic Disaster Recovery](../../notes/08-ha-dr-edge-ops.md) · related services: [AWS Backup](storage.md#aws-backup) · [AWS Application Migration Service](#aws-application-migration-service)

---

**Next:** [Management & monitoring](management-monitoring.md)
