# Glossary: cloud, AWS and exam terms A–Z

> [Home](../README.md) · [Beginner track](README.md)

Plain-English definitions of the terms you'll meet in SAA-C03 material. Where a Kubernetes analogy helps, it's in *italics*; treat analogies as bridges, not exact equivalents. Arrows (→) point to the beginner service page or the deep-dive notes module.

Service names themselves (EC2, S3, SQS…) aren't listed here; see the [services index](services/README.md).

[A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [Z](#z)

---

## A

**Access key**: A long-term credential pair (access key ID + secret access key) that lets a program or the CLI call AWS APIs as an IAM user or the root user. Treat it like a password; prefer temporary credentials from roles or IAM Identity Center. → [IAM](services/identity-governance.md#aws-iam)

**ACL (access control list)**: A list of allow/deny rules attached to a resource. In AWS you'll meet network ACLs (subnet firewalls) and S3 ACLs (a legacy permission mechanism, disabled by default on new buckets). → [Amazon VPC](services/networking.md#amazon-vpc)

**Active-active**: A setup where two or more sites or copies all serve live traffic at the same time. Failure of one just shifts load to the others. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**Active-passive**: A setup where one copy serves traffic and a standby waits to take over if it fails. RDS Multi-AZ is a classic example. *Like a leader/follower StatefulSet where only the leader takes writes.*

**Alias record**: A Route 53-specific DNS record that points a name (including the bare domain, e.g., `example.com`) at an AWS resource such as a load balancer or CloudFront distribution. It's free for queries to AWS resources and follows IP changes automatically. → [Amazon Route 53](services/networking.md#amazon-route-53)

**ALB (Application Load Balancer)**: A load balancer that understands HTTP/HTTPS (Layer 7) and can route by path, hostname or header. *The AWS Load Balancer Controller creates one from a Kubernetes Ingress.* → [ELB](services/networking.md#elastic-load-balancing-elb)

**AMI (Amazon Machine Image)**: A template for launching EC2 instances: the OS disk snapshot plus launch settings. AMIs are Regional; copy them to use elsewhere. *Like a container image, but for a whole virtual machine.* → [Amazon EC2](services/compute.md#amazon-ec2)

**API (application programming interface)**: The set of operations a service exposes for programs to call. Every AWS action (console, CLI, SDK, IaC) is an API call. *Like the Kubernetes API server: every client talks to it.*

**ARN (Amazon Resource Name)**: A unique ID for any AWS resource, shaped like `arn:aws:service:region:account-id:resource`. You use ARNs in IAM policies to say exactly which resource a rule applies to. *Like a fully qualified `namespace/kind/name`, but globally unique.*

**Assume role**: The act of taking on an IAM role to get temporary credentials for it, done through AWS STS. → [IAM](services/identity-governance.md#aws-iam)

**Asynchronous (async)**: The caller hands off work and doesn't wait for the result. Queues and event buses make systems asynchronous, which improves resilience. → [Integration services](services/integration.md)

**Auto Scaling group (ASG)**: A fleet of EC2 instances that EC2 Auto Scaling keeps at a desired size, replaces when unhealthy, and grows or shrinks by policy. *Like a Deployment's ReplicaSet plus Cluster Autoscaler, for VMs.* → [Compute](services/compute.md)

**Availability**: The percentage of time a system is up and usable (e.g., 99.99%). Not the same as durability. See *Durability vs availability*.

**Availability Zone (AZ)**: One or more physically separate data centers within a Region, with independent power, cooling and networking. Spreading across AZs is the basic high-availability move. *The `topology.kubernetes.io/zone` label on your nodes is an AZ.* → [Cloud & AWS basics](01-cloud-and-aws-basics.md#5-global-infrastructure)

**AZ ID**: A consistent identifier for a physical AZ (e.g., `use1-az1`). AZ names like `us-east-1a` are mapped differently per account, so use AZ IDs when coordinating across accounts.

## B

**Backoff (exponential backoff)**: Retrying a failed call with increasing waits (e.g., 1 s, 2 s, 4 s…), usually with random "jitter". It stops retries from overwhelming a struggling service. AWS SDKs do this automatically on throttling errors. *Like the growing delay in a `CrashLoopBackOff`.*

**Bastion host (jump box)**: A hardened server in a public subnet that admins SSH into first, then hop to private servers. Modern AWS answers prefer Systems Manager Session Manager, which needs no open inbound ports. → [Management & monitoring](services/management-monitoring.md)

**Blast radius**: How much is affected when something fails or is compromised. Multiple accounts, AZs and least privilege all shrink the blast radius.

**Block storage**: Storage presented as a raw disk that an OS formats with a file system. EBS is AWS's block storage. *A PersistentVolume backed by EBS is block storage.* → [Storage](services/storage.md)

**Blue/green deployment**: Running the old version (blue) and new version (green) side by side, then switching traffic to green all at once (e.g., by DNS or load balancer). Rollback is just switching back. *Like two Deployments behind a Service whose selector you flip.*

**Bucket**: A top-level container for objects in S3. Bucket names must be unique across all AWS accounts. → [Amazon S3](services/storage.md#amazon-s3)

**Bucket policy**: A JSON resource-based policy attached to an S3 bucket that says who can do what with it, including other accounts. → [Amazon S3](services/storage.md#amazon-s3)

**Burstable instance**: An EC2 instance type (the T family) with a low baseline CPU that can burst higher using CPU credits. Good for spiky, mostly idle workloads. → [Amazon EC2](services/compute.md#amazon-ec2)

## C

**Cache**: A fast store holding copies of frequently used data so you don't fetch it from the slower source every time. CloudFront caches at the edge; ElastiCache caches in memory near your app. → [Databases](services/databases.md)

**Cache hit ratio**: The share of requests answered from the cache instead of the origin. Higher is better: faster responses and less load on the origin. → [Amazon CloudFront](services/networking.md#amazon-cloudfront)

**Canary deployment**: Sending a small percentage of traffic to a new version first, watching for errors, then gradually increasing. *Like progressive delivery with Argo Rollouts or Flagger.*

**CapEx / OpEx**: Capital expenditure (big up-front purchases like servers) vs operating expenditure (ongoing costs like a monthly bill). The cloud shifts IT spending from CapEx to OpEx.

**CDN (content delivery network)**: A network of servers around the world that caches content close to users to cut latency. CloudFront is AWS's CDN. → [Amazon CloudFront](services/networking.md#amazon-cloudfront)

**CIDR (Classless Inter-Domain Routing)**: Notation for an IP address range, like `10.0.0.0/16`. The number after the slash says how many leading bits are fixed; a smaller number means a bigger range. → [Networking basics](02-networking-basics.md)

**Cold start**: Extra latency when a serverless function runs on a fresh execution environment that must be initialized first. *Like the delay of pulling an image and starting a new Pod.* → [Compute](services/compute.md)

**Compliance**: Meeting legal, regulatory or contractual rules (e.g., PCI DSS for card data, HIPAA for health data, GDPR for EU personal data). AWS Artifact provides AWS's compliance reports. → [Security](services/security.md)

**Concurrency**: How many requests or executions are being processed at the same moment. Lambda limits concurrency per Region. → [Compute](services/compute.md)

**Consistency (strong vs eventual)**: Strong consistency means a read always returns the latest write. Eventual consistency means a read may briefly return old data, but all copies converge soon. S3 is strongly consistent; DynamoDB offers both.

**Consolidated billing**: An AWS Organizations feature that combines all member accounts into one bill and pools usage for volume discounts. → [AWS Organizations](services/identity-governance.md)

**Container**: A packaged application with its dependencies, run in isolation on a shared OS kernel. *You know these: Pods run containers.* → [Containers](services/containers.md)

**Control plane vs data plane**: The control plane creates and configures things (API calls like "create a load balancer"); the data plane does the actual work (the load balancer passing traffic). AWS designs data planes to keep working even if the control plane has problems. *In Kubernetes, the API server and controllers are the control plane; kubelets, kube-proxy and your Pods are the data plane.*

**Credentials**: Proof of identity used to sign API calls: a password, access keys, or temporary security credentials (key + secret + session token).

**Cross-Region replication (CRR)**: Automatically copying data (e.g., S3 objects) to another Region for disaster recovery, compliance or lower latency. → [Amazon S3](services/storage.md#amazon-s3)

**Customer managed key**: A KMS key you create and control (rotation, key policy, deletion), as opposed to an AWS managed key that AWS creates for a service on your behalf. → [Security](services/security.md)

## D

**Data lake**: A central store (usually S3) holding large amounts of raw and processed data in open formats, which many tools can query. → [Analytics](services/analytics.md)

**Data residency**: A requirement that data stays in a particular country or region. You meet it by choosing Regions (and guardrails like SCPs that block others).

**Data transfer out (DTO)**: Data leaving AWS to the internet, or crossing AZs or Regions. It's charged per GB; data coming in from the internet is generally free. → [Cost management](services/cost-management.md)

**Data warehouse**: A database optimized for analytical queries over large historical datasets (OLAP). Redshift is AWS's data warehouse. → [Databases](services/databases.md)

**DDoS (distributed denial of service)**: An attack that floods a service with traffic from many sources to knock it offline. AWS Shield protects against it. → [Security](services/security.md)

**Dead-letter queue (DLQ)**: A queue where messages go after failing processing too many times, so they don't block the main queue and can be inspected later. → [Integration services](services/integration.md)

**Decoupling**: Designing components so they don't depend on each other being up at the same moment, usually with a queue or event bus in between. If the consumer slows down, work waits instead of failing. → [Notes 06](../notes/06-decoupling-serverless.md)

**Defense in depth**: Layering several security controls (WAF, security groups, NACLs, IAM, encryption) so one failure doesn't expose everything.

**Disaster recovery (DR)**: Plans and architecture for restoring service after a major failure, such as losing a Region. The four classic strategies, from cheapest/slowest to most expensive/fastest: backup and restore, pilot light, warm standby, multi-site active-active. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**DNS (Domain Name System)**: The internet's phone book: it turns names like `www.example.com` into IP addresses. *CoreDNS does this inside your cluster.* → [Amazon Route 53](services/networking.md#amazon-route-53)

**Drift**: When real resources no longer match their IaC definition, usually because someone changed them by hand. CloudFormation can detect drift. *Like a resource edited with `kubectl edit` that no longer matches Git, except nothing reconciles it back automatically.*

**Durability vs availability**: Durability is the chance your data is *not lost* (S3 Standard is designed for 99.999999999%, "eleven nines"). Availability is the chance you can *access it right now*. Data can be durable but briefly unavailable.

## E

**East-west traffic**: Traffic between services inside your environment (service to service). North-south traffic enters or leaves the environment (users to your app). *Pod-to-Pod traffic is east-west; Ingress traffic is north-south.*

**Edge location**: A small AWS site near users, used by CloudFront, Route 53 and Global Accelerator to serve or route requests with low latency. There are far more edge locations than Regions. → [Amazon CloudFront](services/networking.md#amazon-cloudfront)

**Egress / ingress**: Egress is traffic leaving a network or resource; ingress is traffic entering it. Security group rules are split into inbound (ingress) and outbound (egress). *Not to be confused with the Kubernetes Ingress object, which is an HTTP routing resource.*

**Elastic IP address**: A static public IPv4 address you own in your account and can move between instances. Like all public IPv4 addresses, it's billed hourly. → [Amazon EC2](services/compute.md#amazon-ec2)

**Elasticity**: Automatically adding and removing capacity to match demand, so you pay for what you need. *HPA and Cluster Autoscaler are elasticity.*

**Encryption at rest**: Encrypting stored data (disks, objects, databases, backups). Usually done with KMS keys and turned on with a setting. → [Security](services/security.md)

**Encryption in transit**: Encrypting data as it moves over the network, usually with TLS (HTTPS).

**Endpoint (VPC endpoint)**: A private connection from your VPC to an AWS service, so traffic doesn't go over the internet. Gateway endpoints (S3, DynamoDB) are free route-table entries; interface endpoints are ENIs powered by PrivateLink. → [Amazon VPC](services/networking.md#amazon-vpc)

**ENI (elastic network interface)**: A virtual network card in a subnet, with a private IP address and security groups. EC2 instances, Lambda functions in a VPC, load balancers and interface endpoints all use ENIs. *With the EKS VPC CNI, Pods get IPs from ENIs on the node.* → [Amazon VPC](services/networking.md#amazon-vpc)

**Envelope encryption**: Encrypting data with a data key, then encrypting that data key with a master key (a KMS key). Only the small data key travels to KMS, so large data can be encrypted locally and fast. → [Security](services/security.md)

**ETL (extract, transform, load)**: Pulling data from sources, cleaning or reshaping it, and loading it into a target such as a data warehouse or data lake. AWS Glue is AWS's serverless ETL service. → [Analytics](services/analytics.md)

**Event-driven architecture**: Components react to events ("order placed", "file uploaded") instead of calling each other directly. → [Integration services](services/integration.md)

**Eventual consistency**: See *Consistency (strong vs eventual)*.

## F

**Failover**: Automatically switching to a standby component when the primary fails, such as an RDS Multi-AZ standby taking over or Route 53 routing to a backup Region.

**Fan-out**: Sending one message to many receivers at once, typically SNS publishing to several SQS queues so each consumer processes its own copy. → [Integration services](services/integration.md)

**Fault tolerance**: See *High availability (HA) vs fault tolerance*.

**Federation**: Letting users sign in with an existing identity provider (a corporate directory, Google, Okta) instead of creating separate AWS users. They receive temporary AWS credentials. → [Identity & governance](services/identity-governance.md)

**FIFO (first in, first out)**: Ordering where messages are processed in the order they were sent. SQS FIFO queues guarantee order and exactly-once processing within limits. → [Integration services](services/integration.md)

**File storage**: Storage accessed as a shared file system over the network (NFS, SMB). EFS and FSx are AWS's file storage. *An EFS-backed PersistentVolume with ReadWriteMany is file storage.* → [Storage](services/storage.md)

**Firewall**: A control that allows or blocks network traffic based on rules. On AWS: security groups, network ACLs, AWS Network Firewall and AWS WAF, each at a different layer.

## G

**Gateway endpoint**: A free VPC endpoint type for S3 and DynamoDB that works through route-table entries. See *Endpoint*. → [Amazon VPC](services/networking.md#amazon-vpc)

**Global service**: A service not tied to one Region, such as IAM, Route 53 and CloudFront.

**Golden image**: A pre-configured, tested AMI (or container image) used as the standard starting point for servers, which speeds up launches and keeps them consistent.

**Governance**: The rules and controls for how an organization uses AWS: account structure, guardrails, tagging, audit. → [Identity & governance](services/identity-governance.md)

**Guardrail**: A preventive or detective rule applied across accounts, such as an SCP blocking a Region or an AWS Config rule flagging unencrypted volumes. AWS Control Tower bundles common ones. *Like an admission policy (OPA Gatekeeper/Kyverno) across many clusters.*

## H

**Health check**: A periodic test (e.g., an HTTP request to `/health`) that decides whether a target is healthy enough to receive traffic. Load balancers, Auto Scaling and Route 53 all use them. *Like readiness and liveness probes.* → [ELB](services/networking.md#elastic-load-balancing-elb)

**High availability (HA) vs fault tolerance**: High availability means the system recovers quickly from failure with little downtime (e.g., a few seconds of failover). Fault tolerance means it keeps running with *no* interruption, because redundant parts are already doing the work. Fault tolerance usually costs more.

**Horizontal vs vertical scaling**: Horizontal scaling (scaling out/in) adds or removes machines. Vertical scaling (scaling up/down) makes one machine bigger or smaller. Horizontal is preferred for HA and has no hard ceiling. *HPA is horizontal; changing a Pod's resource requests (VPA) is vertical.*

**Hybrid cloud**: Running workloads across your own data center and a public cloud, connected by VPN or a dedicated link. → [Networking basics](02-networking-basics.md)

**Hypervisor**: Software that runs virtual machines on physical hardware. AWS's is the Nitro System. AWS is responsible for it under the shared responsibility model.

## I

**IaaS (Infrastructure as a Service)**: Renting raw building blocks (VMs, disks, networks) and managing the OS and everything above it yourself. EC2 is the classic example. → [Cloud & AWS basics](01-cloud-and-aws-basics.md#1-what-the-cloud-actually-is)

**IaC (Infrastructure as Code)**: Defining infrastructure in text files (CloudFormation, Terraform, AWS CDK) that a tool applies to create and update resources. *Like YAML manifests with `kubectl apply` or a Helm chart.* → [Management & monitoring](services/management-monitoring.md)

**IAM policy**: A JSON document listing allowed or denied actions on resources, optionally under conditions. Identity-based policies attach to users/groups/roles; resource-based policies attach to resources like buckets. *Like an RBAC Role's rules, but with explicit Deny and conditions.* → [IAM](services/identity-governance.md#aws-iam)

**IAM role**: An identity with permissions but no long-term credentials. Users, services or other accounts assume it and get temporary credentials. *Like a ServiceAccount bound to RBAC permissions; IRSA and EKS Pod Identity link the two directly.* → [IAM](services/identity-governance.md#aws-iam)

**IAM user**: A named identity with long-term credentials (password and/or access keys). For people, AWS now prefers IAM Identity Center. → [IAM](services/identity-governance.md#aws-iam)

**Idempotent**: An operation that has the same effect whether it runs once or many times. Important because queues may deliver a message more than once and clients retry. *`kubectl apply` is idempotent; `kubectl create` is not.*

**Identity provider (IdP)**: A system that authenticates users and vouches for them to others, such as Microsoft Entra ID, Okta or Google. AWS trusts it through SAML or OIDC.

**Immutable infrastructure**: Never patching running servers in place; instead, build a new image and replace the servers. Rollback means redeploying the old image. *Exactly how you treat containers: you don't SSH in and patch a Pod.*

**Instance**: A running virtual machine on EC2 (or a database server on RDS).

**Instance profile**: The container that attaches an IAM role to an EC2 instance, so software on it gets temporary credentials automatically. → [Amazon EC2](services/compute.md#amazon-ec2)

**Instance type**: The size and hardware family of an EC2 instance, like `m7g.large`: family (m = general purpose), generation (7), extra attributes (g = Graviton), size (large). → [Amazon EC2](services/compute.md#amazon-ec2)

**Interface endpoint**: A VPC endpoint that places an ENI with a private IP in your subnet to reach an AWS service or a partner service privately, powered by PrivateLink. It has an hourly and per-GB charge. → [Amazon VPC](services/networking.md#amazon-vpc)

**Internet gateway (IGW)**: The VPC component that connects public subnets to the internet. A subnet is "public" when its route table sends `0.0.0.0/0` to an IGW. → [Amazon VPC](services/networking.md#amazon-vpc)

**IOPS (input/output operations per second)**: How many read/write operations a disk can do per second. Matters for databases and lots of small random reads/writes. → [Storage](services/storage.md)

**IPv4 / IPv6**: The two versions of IP addressing. IPv4 addresses are scarce (public ones are billed hourly on AWS); IPv6 addresses are plentiful and public by nature. → [Networking basics](02-networking-basics.md)

## J

**JSON (JavaScript Object Notation)**: A text data format of keys and values. IAM policies, bucket policies and many API payloads are JSON; CloudFormation templates can be JSON or YAML.

## K

**Key pair**: A public/private key pair used to SSH into Linux EC2 instances (or decrypt the Windows admin password). AWS keeps the public key; you keep the private key. → [Amazon EC2](services/compute.md#amazon-ec2)

**Key-value store**: A database that stores items looked up by a key, with very fast reads and writes at any scale. DynamoDB is AWS's main one. → [Databases](services/databases.md)

**KMS key**: A logical encryption key in AWS KMS (Key Management Service) whose key material never leaves KMS unencrypted. Services use it to encrypt data keys (envelope encryption). Access is controlled by a key policy. → [Security](services/security.md)

## L

**Latency**: The delay before a response starts, usually in milliseconds. Distance, network hops and disk speed all add latency. Lower is better.

**Launch template**: A saved set of EC2 launch settings (AMI, instance type, security groups, user data) that Auto Scaling groups use to create instances. *Like the Pod template inside a Deployment.* → [Compute](services/compute.md)

**Layer 4 vs Layer 7**: Layers of the OSI network model. Layer 4 handles TCP/UDP connections and ports (Network Load Balancer); Layer 7 understands HTTP requests, paths and headers (Application Load Balancer). → [Networking basics](02-networking-basics.md)

**Least privilege**: Granting only the permissions needed for a task, and nothing more. The most important IAM principle on the exam. *Same idea as narrow RBAC Roles instead of `cluster-admin`.* → [IAM](services/identity-governance.md#aws-iam)

**Lifecycle policy**: Rules that automatically move data to cheaper storage or delete it after a set age, such as S3 objects to Glacier after 90 days. → [Amazon S3](services/storage.md#amazon-s3)

**Lift and shift (rehost)**: Moving an application to the cloud mostly unchanged, typically onto EC2. Fast, but doesn't use cloud-native benefits. → [Migration & transfer](services/migration-transfer.md)

**Load balancer**: A component that spreads incoming traffic across several healthy targets. *A Service of type LoadBalancer on EKS creates an AWS load balancer.* → [ELB](services/networking.md#elastic-load-balancing-elb)

**Long polling**: A consumer asks a queue for messages and waits (up to 20 seconds in SQS) for one to arrive, instead of getting an empty answer immediately. Cuts empty requests and cost. → [Integration services](services/integration.md)

**Loose coupling**: See *Decoupling*.

## M

**Managed service**: A service where AWS runs the servers, patching, scaling and availability, and you just use it (RDS, DynamoDB, SQS). Usually the "least operational overhead" answer. *Like using EKS instead of running your own control plane.*

**MFA (multi-factor authentication)**: Requiring a second proof of identity (an authenticator app, a security key) in addition to a password. Always enable it for the root user.

**Microservices**: An architecture where an application is split into small, independently deployable services that talk over APIs or messages. *Usually one Deployment per microservice.*

**Multi-AZ**: Running copies of a resource in two or more Availability Zones so it survives an AZ failure. For RDS, it specifically means a synchronous standby that takes over automatically. → [Amazon RDS](services/databases.md#amazon-rds)

**Multi-Region**: Running in two or more AWS Regions for disaster recovery, compliance or lower latency for global users. Costlier and more complex than Multi-AZ. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**Multi-tenant**: Many customers (tenants) share the same underlying infrastructure or application, with isolation between them. Most AWS services are multi-tenant behind the scenes; Dedicated Hosts are the single-tenant opposite. *Like many teams sharing one cluster, separated by namespaces.*

**Multipart upload**: Uploading a large S3 object in parallel parts, then combining them. Recommended above about 100 MB and required above 5 GB. → [Amazon S3](services/storage.md#amazon-s3)

## N

**NAT (network address translation)**: Rewriting IP addresses as traffic passes through a device, typically so many private addresses can share one public address for outbound internet access. → [Networking basics](02-networking-basics.md)

**NAT gateway**: AWS's managed NAT service. It sits in a public subnet and lets instances in private subnets reach the internet for outbound connections, while blocking unsolicited inbound ones. Charged per hour and per GB. → [Amazon VPC](services/networking.md#amazon-vpc)

**Network ACL (NACL)**: A stateless firewall at the subnet boundary with numbered allow/deny rules. Because it's stateless, return traffic must be explicitly allowed. → [Amazon VPC](services/networking.md#amazon-vpc)

**NLB (Network Load Balancer)**: A Layer 4 load balancer for TCP/UDP/TLS with very high performance and static IP addresses per AZ. → [ELB](services/networking.md#elastic-load-balancing-elb)

**NoSQL**: Databases that don't use the traditional relational table model, such as key-value, document, graph or wide-column stores. DynamoDB, DocumentDB and Neptune are AWS examples. → [Databases](services/databases.md)

## O

**Object storage**: Storage where each file is an object (data + metadata) addressed by a key in a flat namespace, accessed over HTTP APIs rather than mounted as a disk. S3 is AWS's object storage. → [Amazon S3](services/storage.md#amazon-s3)

**OIDC (OpenID Connect)**: A modern identity protocol built on OAuth 2.0, using JSON tokens. Used for web/mobile sign-in, and by EKS IRSA so Pods can assume IAM roles. *The ServiceAccount tokens your Pods get are OIDC tokens.*

**OLTP vs OLAP**: OLTP (online transaction processing) is many small, fast reads and writes, like placing orders (RDS, Aurora, DynamoDB). OLAP (online analytical processing) is big queries over lots of historical data, like monthly sales reports (Redshift, Athena). → [Databases](services/databases.md)

**On-Demand**: The default EC2 pricing: pay per second or hour with no commitment. Most flexible, most expensive per hour. → [Amazon EC2](services/compute.md#amazon-ec2)

**On-premises (on-prem)**: Running in your own data center rather than in a cloud.

**Origin**: The source a CDN fetches content from when it isn't cached, such as an S3 bucket or a load balancer. → [Amazon CloudFront](services/networking.md#amazon-cloudfront)

**Origin Access Control (OAC)**: A CloudFront feature that lets only your distribution read a private S3 bucket, so users can't bypass the CDN. It replaces the older Origin Access Identity (OAI). → [Amazon CloudFront](services/networking.md#amazon-cloudfront)

**OU (organizational unit)**: A folder of accounts in AWS Organizations, used to apply policies to many accounts at once. → [Identity & governance](services/identity-governance.md)

## P

**PaaS (Platform as a Service)**: The provider runs the OS and runtime; you deploy your code and data. Elastic Beanstalk is the classic AWS example. → [Cloud & AWS basics](01-cloud-and-aws-basics.md#1-what-the-cloud-actually-is)

**Parquet**: An open, columnar file format for analytics. It's compressed and lets queries read only the needed columns, so Athena queries on it are faster and cheaper than on CSV or JSON. → [Analytics](services/analytics.md)

**Partition key**: The attribute DynamoDB (or Kinesis) hashes to decide where an item or record is stored. A key with many distinct, evenly used values spreads load; a "hot" key causes throttling. → [Databases](services/databases.md)

**Peering (VPC peering)**: A private, direct network connection between two VPCs (same or different accounts or Regions). It's non-transitive: if A peers with B and B with C, A can't reach C through B. → [Amazon VPC](services/networking.md#amazon-vpc)

**PII (personally identifiable information)**: Data that identifies a person: names, emails, ID numbers, addresses. Amazon Macie finds PII in S3. → [Security](services/security.md)

**Pilot light**: A DR strategy where only the core pieces (usually a replicated database) run in the recovery Region, and everything else is started or scaled up when disaster strikes. Cheaper than warm standby, but slower to recover. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**Point-in-time recovery (PITR)**: Restoring a database to any second within a retention window, using continuous backups. RDS and DynamoDB support it.

**Polling**: A consumer repeatedly asking "anything new?" rather than being pushed events. SQS consumers poll; SNS pushes.

**Presigned URL**: A temporary URL that grants time-limited access to a specific private S3 object, signed with the creator's credentials. Good for letting users download or upload without AWS credentials. → [Amazon S3](services/storage.md#amazon-s3)

**Principal**: The "who" in a policy: a user, role, account, or AWS service making a request.

**Private subnet**: A subnet with no route to an internet gateway. Resources there can't be reached from the internet. They use a NAT gateway for outbound access. → [Amazon VPC](services/networking.md#amazon-vpc)

**Provisioned capacity**: Reserving a fixed amount of capacity (IOPS, throughput, read/write units) in advance, as opposed to on-demand modes that scale and bill per use.

**Pub/sub (publish/subscribe)**: A messaging pattern where publishers send messages to a topic and every subscriber receives a copy. SNS and EventBridge are pub/sub. → [Integration services](services/integration.md)

**Public subnet**: A subnet whose route table sends internet traffic (`0.0.0.0/0`) to an internet gateway. Load balancers and NAT gateways usually live here. → [Amazon VPC](services/networking.md#amazon-vpc)

## Q

**Queue**: A buffer where producers put messages and consumers take them off later at their own pace. The basic tool for decoupling. SQS is AWS's queue service. → [Integration services](services/integration.md)

**Quota (service quota / limit)**: A maximum AWS sets per account and Region, such as the number of VPCs or Lambda concurrency. Many can be raised on request through Service Quotas. *Like a namespace ResourceQuota, but set by AWS.*

## R

**Read replica**: A read-only copy of a database, updated asynchronously, used to offload read traffic. Unlike a Multi-AZ standby, it serves traffic, but it isn't an automatic failover target by default. → [Amazon RDS](services/databases.md#amazon-rds)

**Region**: A separate geographic area of AWS (e.g., `eu-west-1`) containing several AZs. Most resources live in one Region. → [Cloud & AWS basics](01-cloud-and-aws-basics.md#5-global-infrastructure)

**Regional service**: A service whose resources live in one Region and that AWS spreads across that Region's AZs for you, such as S3, DynamoDB, SQS and Lambda.

**Replication (synchronous vs asynchronous)**: Synchronous replication confirms a write only after every copy has it (no data loss, a bit slower; RDS Multi-AZ). Asynchronous replication copies after confirming (faster, may lose the last few seconds; read replicas, cross-Region replication).

**Reserved Instance (RI)**: A 1- or 3-year commitment to a specific instance configuration in return for a large discount vs On-Demand. Still the classic exam answer for steady databases. → [Cost management](services/cost-management.md)

**Resource-based policy**: A permission policy attached to a resource (an S3 bucket, SQS queue, KMS key) saying who may access it, including principals in other accounts. → [IAM](services/identity-governance.md#aws-iam)

**Rolling deployment**: Replacing instances or containers with the new version a few at a time, so the service stays up. *The default Kubernetes Deployment strategy.*

**Root user**: The identity created with the AWS account (the sign-up email) that has unrestricted access. Protect it with MFA and use it only for the few tasks that need it. → [Cloud & AWS basics](01-cloud-and-aws-basics.md#3-the-aws-account)

**Route table**: A set of rules in a VPC deciding where traffic for each destination IP range goes (local, internet gateway, NAT gateway, peering connection…). Each subnet is associated with one. → [Amazon VPC](services/networking.md#amazon-vpc)

**RPO (recovery point objective)**: How much data loss is acceptable, measured in time. An RPO of 15 minutes means you can lose at most the last 15 minutes of data. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**RTO (recovery time objective)**: How long the system may be down before it must be restored. An RTO of 1 hour means service must be back within an hour. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

## S

**SaaS (Software as a Service)**: Finished software you use over the internet without managing any of it, like email or CRM services.

**SAML (Security Assertion Markup Language) 2.0**: An older, XML-based standard for single sign-on, widely used by corporate identity providers to federate users into AWS. → [Identity & governance](services/identity-governance.md)

**Savings Plans**: A 1- or 3-year commitment to a dollar amount of compute usage per hour, in exchange for discounts. More flexible than Reserved Instances. → [Cost management](services/cost-management.md)

**Scaling policy**: The rule that tells Auto Scaling when to add or remove capacity, such as "keep average CPU at 50%" (target tracking). *Like an HPA target.*

**SCP (service control policy)**: An AWS Organizations policy that sets the maximum permissions for accounts in an OU. It grants nothing by itself; it only limits. → [Identity & governance](services/identity-governance.md)

**Security group (SG)**: A stateful virtual firewall on an ENI (an instance, load balancer, database) with allow-only rules. Return traffic is allowed automatically. *Similar in spirit to a NetworkPolicy, applied per network interface.* → [Amazon VPC](services/networking.md#amazon-vpc)

**Serverless**: Running code or using services without managing servers, with automatic scaling and pay-per-use pricing (often scaling to zero). Lambda, Fargate, DynamoDB and S3 are serverless. → [Notes 06](../notes/06-decoupling-serverless.md)

**Sharding**: Splitting data across several databases or partitions by a key so each holds only part of it, to scale beyond one machine.

**SLA (service level agreement)**: A provider's formal promise of availability (e.g., 99.99% monthly), with service credits if it's missed. It describes compensation, not a guarantee that failures won't happen.

**Snapshot**: A point-in-time backup of a volume or database. EBS snapshots are stored in S3 and are incremental. *A VolumeSnapshot via the EBS CSI driver creates one.* → [Storage](services/storage.md)

**Spot Instance**: Spare EC2 capacity at a large discount that AWS can reclaim with a two-minute warning. Ideal for fault-tolerant, interruptible work like batch jobs. → [Amazon EC2](services/compute.md#amazon-ec2)

**SSE (server-side encryption)**: The storage service encrypts data when it writes it and decrypts when you read it. For S3: SSE-S3 (the default), SSE-KMS, DSSE-KMS and SSE-C. → [Amazon S3](services/storage.md#amazon-s3)

**SSO (single sign-on)**: Logging in once to reach many applications or accounts. On AWS this is IAM Identity Center. → [Identity & governance](services/identity-governance.md)

**Stateful vs stateless**: A stateless component keeps no session data between requests, so any copy can serve any request and copies can be added or killed freely. A stateful one remembers data between requests. Design app servers stateless; keep state in a database or cache. *A Deployment vs a StatefulSet.* Security groups are *stateful* (they remember connections); NACLs are *stateless*.

**Sticky sessions (session affinity)**: Making a load balancer send a user's requests to the same target every time. A workaround for stateful apps; external session storage is usually better. *Like `sessionAffinity: ClientIP` on a Service.*

**Strong consistency**: See *Consistency (strong vs eventual)*.

**Subnet**: A slice of a VPC's IP range that lives in exactly one AZ. You place resources in subnets to choose their AZ and whether they are public or private. → [Amazon VPC](services/networking.md#amazon-vpc)

**Synchronous**: The caller waits for the result before continuing, like a normal HTTP request. Simple, but couples the caller's availability to the callee's.

## T

**Tag**: A key-value label on an AWS resource (e.g., `env=prod`, `team=payments`), used for cost tracking, automation and access control. *Like Kubernetes labels.* → [Cost management](services/cost-management.md)

**Target group**: The set of targets (instances, IPs, Lambda functions) a load balancer sends traffic to, along with the health check settings. *Like the Endpoints behind a Service.* → [ELB](services/networking.md#elastic-load-balancing-elb)

**Temporary credentials**: Short-lived credentials (access key, secret key, session token) issued by AWS STS when a role is assumed. They expire automatically, which makes them safer than access keys.

**Tenancy**: Whether EC2 instances share hardware with other customers (default, shared), run on hardware dedicated to your account (Dedicated Instances), or on a whole physical server you control (Dedicated Hosts, useful for per-socket software licenses). → [Amazon EC2](services/compute.md#amazon-ec2)

**Throttling**: A service rejecting requests because you exceeded a rate limit, returning errors like `ThrottlingException` or HTTP 429. Handle it with retries and exponential backoff, or raise the quota.

**Throughput**: How much data is moved per second (MB/s, Gbps). Large sequential workloads like log processing care about throughput more than IOPS.

**TLS (Transport Layer Security)**: The encryption protocol behind HTTPS, protecting data in transit. AWS Certificate Manager issues free public TLS certificates for AWS load balancers and CloudFront. → [Security](services/security.md)

**Transit gateway**: A regional network hub that connects many VPCs and on-premises networks in a hub-and-spoke design, replacing a mesh of peering connections. → [Networking](services/networking.md)

**TTL (time to live)**: How long a cached item (a DNS answer, a CDN object, a cache entry) is considered fresh before it must be fetched again. DynamoDB also uses TTL to auto-delete old items.

## U

**Unmanaged (self-managed)**: Something you install and run yourself, usually on EC2, such as your own database or Kafka cluster. More control, more work. Exam answers usually prefer the managed option unless a requirement rules it out.

**User data**: A script passed to an EC2 instance that runs at first boot, often used to install software. *Similar in purpose to an init container.* → [Amazon EC2](services/compute.md#amazon-ec2)

## V

**Versioning**: Keeping every version of an S3 object, so overwrites and deletes can be undone. → [Amazon S3](services/storage.md#amazon-s3)

**Virtual machine (VM)**: A software-defined computer with its own OS running on shared physical hardware. An EC2 instance is a VM. *Your EKS worker nodes are VMs.*

**VPC (virtual private cloud)**: Your own isolated private network in an AWS Region, with your IP range, subnets, route tables and firewalls. *Your EKS cluster's nodes and Pods live inside a VPC.* → [Amazon VPC](services/networking.md#amazon-vpc)

**VPN (virtual private network)**: An encrypted tunnel across the public internet. Site-to-Site VPN links an office network to a VPC; Client VPN links individual laptops. → [Networking](services/networking.md)

## W

**Warm standby**: A DR strategy where a scaled-down but fully working copy runs in another Region, ready to scale up and take traffic. Faster recovery than pilot light, cheaper than active-active. → [Notes 08](../notes/08-ha-dr-edge-ops.md)

**Well-Architected Framework**: AWS's best-practice guide organized into six pillars: operational excellence, security, reliability, performance efficiency, cost optimization and sustainability. → [Cloud & AWS basics](01-cloud-and-aws-basics.md#10-the-well-architected-framework)

**Workload**: A collection of resources and code that together deliver business value, such as "the checkout system".

**WORM (write once, read many)**: Storage where data can't be changed or deleted for a set period, for compliance. S3 Object Lock and S3 Glacier Vault Lock provide it. → [Storage](services/storage.md)

## Z

**Zonal service**: A resource that lives in one specific AZ, such as an EC2 instance, an EBS volume or a subnet. You build HA by having copies in more than one AZ.

**Zero trust**: A security approach that never trusts a request just because of its network location; every request is authenticated and authorized. On AWS that means strong IAM, encryption everywhere and private connectivity.

---

**Next:** [Beginner track index](README.md)
