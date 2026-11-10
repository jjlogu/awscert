# Security, encryption & threat detection: AWS services explained for beginners
> [Home](../../README.md) · [Beginner track](../README.md) · [All services A–Z](README.md) · Deep dive: [Notes 07](../../notes/07-security-data-protection.md)

[Identity & governance](identity-governance.md) decides *who* may do *what*. This category covers everything else that keeps a workload safe: **encrypting data** (KMS, CloudHSM, ACM, Secrets Manager), **filtering traffic** before it reaches your app (WAF, Shield, Network Firewall, Firewall Manager), **detecting problems** after the fact (GuardDuty, Inspector, Macie, Detective, Security Hub), and **proving compliance** to auditors (Artifact, Audit Manager). Design Secure Architectures is 30% of the exam, the largest domain, so most of these services matter.

| Layer | What it protects | Services |
|---|---|---|
| Edge and network | Traffic coming in from the internet, and traffic inside your VPC (Virtual Private Cloud) | Shield, WAF, Network Firewall, Firewall Manager |
| Data | Data at rest and in transit, plus passwords and API keys | KMS, CloudHSM, ACM, Secrets Manager |
| Detection | Spotting attacks, weaknesses and exposed data | GuardDuty, Inspector, Macie, Detective, Security Hub |
| Compliance | Showing auditors that AWS and you meet the rules | Artifact, Audit Manager |

> **Encryption 101 (read this first)**
> - **At rest** means data sitting on a disk, in a bucket or in a database backup. **In transit** means data moving across a network. You need both. AWS encrypts at rest through KMS keys, and in transit with **TLS** (Transport Layer Security, the "S" in HTTPS).
> - A **symmetric key** is one secret that both locks and unlocks (AES-256, the Advanced Encryption Standard with 256-bit keys, is the usual one). It's fast and used for almost all data at rest. An **asymmetric key pair** has a public key to lock or verify and a private key to unlock or sign. TLS and code signing use these.
> - **Why key management matters:** the encryption maths is the easy part. The hard part is *who* can use the key, *proving* every use in a log, *rotating* it, and *never losing it*. Encryption turns "protect 10 TB of data" into "protect one key". If the key leaks, the data is exposed. If the key is deleted, the data is gone for good.
> - **Envelope encryption:** AWS encrypts your data with a small *data key*, then encrypts that data key with a *master key* that never leaves KMS. Think of locking a document in a box, then putting the box's key in a bank vault.

**Which detective service?** They all sound alike. Learn this table.

| Question you're asking | Service | Looks at |
|---|---|---|
| "Is someone attacking us or using stolen credentials right now?" | [Amazon GuardDuty](#amazon-guardduty) | API logs, network flow logs, DNS logs, EKS audit logs |
| "Which servers, container images or functions have known software vulnerabilities?" | [Amazon Inspector](#amazon-inspector) | EC2 (Elastic Compute Cloud) instances, ECR (Elastic Container Registry) images, Lambda functions |
| "Where is sensitive data, such as credit card numbers or personal data, stored in S3?" | [Amazon Macie](#amazon-macie) | Amazon S3 (Simple Storage Service) objects only |
| "This alert fired. What else did this IP address or role do?" | [Amazon Detective](#amazon-detective) | GuardDuty findings + logs, as a graph |
| "Show me all findings from every account, plus a compliance score" | [AWS Security Hub](#aws-security-hub) | Findings from the services above |
| "Is this resource configured correctly, and what did it look like last week?" | [AWS Config](management-monitoring.md#aws-config) | Resource configurations |
| "Who did what, when, and from where?" | [AWS CloudTrail](management-monitoring.md#aws-cloudtrail) | Every API call |

| Service | One-liner | Exam importance |
|---|---|---|
| [AWS KMS](#aws-kms) | Creates and guards the encryption keys most AWS services use | ★★★ |
| [AWS CloudHSM](#aws-cloudhsm) | Your own dedicated hardware key vault inside your VPC | ★★ |
| [AWS Certificate Manager (ACM)](#aws-certificate-manager-acm) | Free TLS certificates that renew themselves, for AWS load balancers and CloudFront | ★★ |
| [AWS Secrets Manager](#aws-secrets-manager) | Stores passwords and API keys and rotates them automatically | ★★★ |
| [AWS WAF](#aws-waf) | Web application firewall that blocks bad HTTP requests | ★★★ |
| [AWS Shield](#aws-shield) | DDoS (distributed denial of service) protection: free basic level, paid advanced level | ★★ |
| [AWS Firewall Manager](#aws-firewall-manager) | Pushes the same firewall rules to every account in an organization | ★★ |
| [AWS Network Firewall](#aws-network-firewall) | Managed firewall that inspects traffic entering, leaving or inside a VPC | ★★ |
| [Amazon GuardDuty](#amazon-guardduty) | Detects threats and compromised resources from your logs | ★★★ |
| [Amazon Inspector](#amazon-inspector) | Scans EC2, container images and Lambda for known vulnerabilities | ★★ |
| [Amazon Macie](#amazon-macie) | Finds sensitive data in S3 | ★★ |
| [Amazon Detective](#amazon-detective) | Helps investigate the root cause of a security finding | ★ |
| [AWS Security Hub](#aws-security-hub) | One dashboard for security findings and best-practice checks | ★★ |
| [AWS Artifact](#aws-artifact) | Download AWS's own compliance reports and agreements | ★ |
| [AWS Audit Manager](#aws-audit-manager) | Collects evidence for *your* audits (closed to new customers) | ★ |

---

## AWS KMS

> **In one line:** KMS (Key Management Service) creates, stores and controls the encryption keys that S3, EBS (Elastic Block Store), RDS (Relational Database Service) and most other AWS services use.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | EKS can encrypt Kubernetes Secrets in etcd with a KMS key. Everyday: a bank's key desk. You never take the master key home; you ask the clerk to lock or unlock for you, and every request is written in a ledger. | Per key per month (customer managed keys) + per API request |

**What it is.** Encryption needs keys, and keys need a safe home, access control, logs and rotation. KMS is that home. Its keys live inside hardware security modules (HSMs) and the key material never leaves KMS in plain form. When you tick "encrypt" on an S3 bucket, an EBS volume or an RDS database, the service quietly asks KMS for a data key using the key you picked. Without KMS you'd be running your own key servers and writing the audit trail yourself.

**Key ideas to know**
- **KMS key**: the master key that stays in KMS. You reference it by ID, ARN (Amazon Resource Name) or an **alias** such as `alias/orders-prod`.
- **Three ownership types**: **AWS owned** (invisible, used by default by some services), **AWS managed** (`aws/s3`, `aws/ebs`; visible but not editable) and **customer managed** (yours: you control its policy, rotation, sharing and deletion).
- **Key policy**: every key has one, and it's the main gate. Unusually, an IAM policy alone isn't enough: the key policy must allow access too, usually by delegating to the account's IAM.
- **Envelope encryption**: KMS only encrypts up to 4 KB directly. For real data, services call `GenerateDataKey`, encrypt locally with the data key, and store the encrypted data key next to the data.
- **Rotation**: automatic rotation (yearly by default) creates new key material but keeps the same key ID, so old data still decrypts and apps don't change.
- **Audit**: every use of a customer managed or AWS managed key appears in [AWS CloudTrail](management-monitoring.md#aws-cloudtrail).
- **Multi-Region keys**: copies of one key in several Regions, so data encrypted in one Region can be decrypted in another.
- **Scheduled deletion**: 7–30 days of waiting. Deleting a key destroys every piece of data encrypted with it, so disable it first if in doubt.

**Use it when / don't use it when**
- ✅ Encrypt data at rest in almost any AWS service, with audit and access control.
- ✅ Share encrypted snapshots or data across accounts → you need a **customer managed** key.
- ❌ Regulations demand a single-tenant HSM that only you control → [AWS CloudHSM](#aws-cloudhsm).
- ❌ Storing passwords or API keys → [AWS Secrets Manager](#aws-secrets-manager) (which uses KMS underneath).

**Exam hot spots**
- "Audit every use of the key / control who can decrypt / custom rotation" → **customer managed key**.
- AWS managed keys **can't be shared across accounts**. To share an encrypted snapshot, re-encrypt it with a customer managed key.
- You can't encrypt an existing unencrypted EBS volume or RDS database in place: snapshot → encrypted copy → restore.
- S3 with SSE-KMS (server-side encryption with KMS keys) at very high request rates hits KMS limits → turn on **S3 Bucket Keys**.

**Try it** (free, 5 min, read-only): open the **KMS** console → *AWS managed keys*. If you've used S3 or EBS encryption you'll see `aws/s3` or `aws/ebs`. Open one and read its key policy.

**Go deeper:** [Notes 07: AWS KMS](../../notes/07-security-data-protection.md#2-aws-kms) · related services: [Amazon S3](storage.md#amazon-s3) · [Amazon EBS](storage.md#amazon-ebs) · [AWS CloudHSM](#aws-cloudhsm)

---

## AWS CloudHSM

> **In one line:** a dedicated, single-tenant hardware security module (HSM) in your VPC, where only you can access the keys.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like renting your own private safe that only you have the combination to, instead of using the bank's shared vault (KMS) | Per HSM per hour (much more than KMS) |

**What it is.** KMS is a shared, multi-tenant service: AWS runs it, and you control access through policies. Some regulations or contracts require keys in hardware that *only you* control. CloudHSM gives you that: AWS racks, patches and backs up the device, but you create the crypto users and keys, and AWS can't see or recover them. Lose your credentials and the keys are gone.

**Key ideas to know**
- **Single-tenant**: the HSM is yours alone. Validated to **FIPS 140-3 Level 3** (FIPS is the U.S. Federal Information Processing Standards).
- **Cluster**: you add HSMs in **at least two AZs** (Availability Zones) for high availability. That's your design job.
- **Standard interfaces**: PKCS#11, JCE (Java Cryptography Extension), Microsoft CNG and OpenSSL. Your app talks to it directly.
- **KMS custom key store**: lets KMS-integrated services such as S3 and EBS use keys stored in your CloudHSM cluster.
- Typical uses: TLS offload for web servers, Oracle TDE (Transparent Data Encryption), signing.

**Use it when / don't use it when**
- ✅ "Single-tenant HSM", "exclusive control of keys", "FIPS 140-3 Level 3".
- ❌ Ordinary encryption with the least effort → [AWS KMS](#aws-kms).

**Exam hot spots**
- CloudHSM = **you** manage keys and HA; KMS = AWS manages the service.
- Need S3/EBS integration **and** a single-tenant HSM → KMS custom key store backed by CloudHSM.

**Go deeper:** [Notes 07: AWS CloudHSM](../../notes/07-security-data-protection.md#3-aws-cloudhsm) · related services: [AWS KMS](#aws-kms)

---

## AWS Certificate Manager (ACM)

> **In one line:** issues free public TLS certificates for AWS load balancers, CloudFront and API Gateway, and renews them automatically.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like cert-manager + Let's Encrypt in your cluster. With the AWS Load Balancer Controller you already reference an ACM certificate in an Ingress annotation (`alb.ingress.kubernetes.io/certificate-arn`). | Public certificates free for integrated services; private certificates (AWS Private CA) are paid |

**What it is.** HTTPS needs a TLS certificate that proves your domain is yours. Buying, installing and remembering to renew certificates causes outages when someone forgets. ACM issues certificates for free, attaches them to AWS services that terminate TLS, and renews them on its own.

**Key ideas to know**
- **Integrated services**: Elastic Load Balancing (the ALB, or Application Load Balancer, and the NLB, or Network Load Balancer), Amazon CloudFront, Amazon API Gateway and a few others. You pick the certificate in their settings.
- **DNS validation**: you add a CNAME record to prove you own the domain. Keep it in place and renewal is fully automatic. (Email validation needs a person to approve each renewal.)
- **Imported certificates**: you can upload a certificate from another authority, but ACM **won't renew it**.
- **Regional**: a certificate for **CloudFront must be in us-east-1**. A certificate for an ALB must be in the ALB's own Region.
- **AWS Private CA**: a paid private certificate authority for internal services and mTLS (mutual TLS).

**Use it when / don't use it when**
- ✅ HTTPS on a load balancer, CloudFront distribution or API Gateway.
- ❌ HTTPS terminated on your own EC2 web server → the classic exam answer is to put an ALB or CloudFront in front and use ACM there.

**Exam hot spots**
- "Certificate for CloudFront" → request it in **us-east-1**.
- "A certificate expired and caused an outage" → it was imported. Use ACM-issued with DNS validation.

**Go deeper:** [Notes 07: ACM and certificates](../../notes/07-security-data-protection.md#4-acm-and-certificates) · related services: [Elastic Load Balancing (ELB)](networking.md#elastic-load-balancing-elb) · [Amazon CloudFront](networking.md#amazon-cloudfront)

---

## AWS Secrets Manager

> **In one line:** stores database passwords, API keys and other secrets, encrypted, and can rotate them automatically.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | A Kubernetes `Secret`, done properly. In EKS you mount Secrets Manager values into pods with the Secrets Store CSI Driver (AWS provider) or sync them with External Secrets Operator. | Per secret per month + per 10,000 API calls |

**What it is.** Apps need passwords, and hardcoding them in code, config files or environment variables is how they leak. A Kubernetes `Secret` is only base64-encoded, which isn't encryption. Secrets Manager stores each secret encrypted with KMS, controls access with IAM, logs every read in CloudTrail and, most importantly, **rotates** secrets on a schedule so a leaked password soon stops working. Your app fetches the secret at runtime using its IAM role.

**Key ideas to know**
- **Secret**: a named value (often JSON, such as username + password), up to 64 KB.
- **Rotation**: a Lambda function changes the password in the database *and* in Secrets Manager on a schedule. AWS provides ready-made rotation for Amazon RDS, Aurora, Redshift and DocumentDB. RDS can also manage its master password in Secrets Manager for you.
- **Versions**: labels such as `AWSCURRENT` and `AWSPREVIOUS` keep apps working during rotation.
- **Replica secrets**: copies in other Regions for disaster recovery.
- **Resource policy**: lets another account read the secret.
- **Caching**: fetch once and cache in the app rather than calling the API on every request.

**Secrets Manager vs SSM Parameter Store.** AWS Systems Manager (SSM) has a feature called Parameter Store that also stores values, and the exam often makes you choose between them. Parameter Store's full description lives on the [AWS Systems Manager](management-monitoring.md#aws-systems-manager) page.

| | Secrets Manager | SSM Parameter Store |
|---|---|---|
| Best for | Secrets with a lifecycle (DB credentials, API keys) | Configuration, plus simple secrets |
| Encryption | Always (KMS) | Optional: `SecureString` uses KMS; `String` is plain text |
| Automatic rotation | ✅ Built in | ❌ None; you'd build it yourself |
| Cross-Region replication | ✅ | ❌ |
| Cost | Paid per secret | Standard tier free |

**Use it when / don't use it when**
- ✅ "Rotate database credentials automatically."
- ✅ "Replicate the secret to the DR (disaster recovery) Region."
- ❌ Plain config (feature flags, endpoints, AMI (Amazon Machine Image) IDs) or secrets that never rotate, on a budget → Parameter Store.
- ❌ Credentials for AWS itself → use an **IAM role**, not a stored access key.

**Exam hot spots**
- "**Rotate** automatically" → Secrets Manager. "Cheap, no rotation" → Parameter Store.
- Never put secrets in Lambda environment variables, user data or AMIs in plain text.

**Try it** (costs a few cents at most, because the per-secret charge is prorated): in the Secrets Manager console, choose *Store a new secret → Other type of secret*, add a key/value, save it, and look at the *Rotation* tab. Then schedule its deletion.

**Go deeper:** [Notes 07: Secrets and configuration](../../notes/07-security-data-protection.md#5-secrets-and-configuration) · related services: [AWS KMS](#aws-kms) · [Amazon RDS](databases.md#amazon-rds) · [AWS Systems Manager](management-monitoring.md#aws-systems-manager)

---

## AWS WAF

> **In one line:** a web application firewall (WAF) that inspects HTTP(S) requests and blocks malicious ones before they reach your app.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | Like ModSecurity rules on an NGINX Ingress controller. With the AWS Load Balancer Controller you attach one to an ALB Ingress via the `alb.ingress.kubernetes.io/wafv2-acl-arn` annotation. | Per web ACL per month + per rule + per million requests |

**What it is.** Security groups and network ACLs filter by IP address and port (layers 3–4). They can't see that a request contains a SQL injection attack in a form field. WAF works at layer 7 (HTTP): it reads URLs, headers, query strings and bodies, and blocks requests that match your rules. You'd otherwise run and tune a reverse proxy with security rules yourself.

**Key ideas to know**
- **Web ACL** (web access control list): your set of rules, attached to a resource. Each rule has an action: **Allow, Block, Count** (log only, for testing), CAPTCHA or Challenge.
- **Managed rule groups**: ready-made rules from AWS (core rule set, SQL injection, known bad inputs, IP reputation) or from AWS Marketplace sellers.
- **SQLi and XSS rules**: block SQL injection and cross-site scripting attacks.
- **Rate-based rules**: block an IP that sends more than a set number of requests in a time window, such as hammering `/login`.
- **Geo match**: allow or block by country.
- **Where it attaches**: Amazon CloudFront, ALB, API Gateway REST APIs, AWS AppSync, Cognito user pools, AWS App Runner and AWS Verified Access.

**Use it when / don't use it when**
- ✅ "Block SQL injection / XSS", "block requests from certain countries", "throttle IPs that send too many requests".
- ❌ Traffic through an **NLB** or straight to EC2 → WAF can't attach there. Put CloudFront or an ALB in front, or filter at the network level with [AWS Network Firewall](#aws-network-firewall).
- ❌ Large-scale DDoS protection on its own → combine with [AWS Shield](#aws-shield).

**Exam hot spots**
- "SQL injection" → **WAF**, never a security group or network ACL.
- WAF does **not** attach to an NLB.
- For many accounts with the same rules → [AWS Firewall Manager](#aws-firewall-manager).

**Go deeper:** [Notes 07: AWS WAF](../../notes/07-security-data-protection.md#61-aws-waf) · related services: [Amazon CloudFront](networking.md#amazon-cloudfront) · [Elastic Load Balancing (ELB)](networking.md#elastic-load-balancing-elb) · [Amazon API Gateway](integration.md#amazon-api-gateway)

---

## AWS Shield

> **In one line:** protection against DDoS (distributed denial of service) attacks: Shield Standard is free and always on; Shield Advanced is a paid upgrade.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like a city's flood defences: everyone is protected by the basic levee (Standard); you can pay for sandbags, a response crew and flood insurance (Advanced) | Standard: free. Advanced: monthly subscription with a 1-year commitment (expensive) |

**What it is.** A DDoS attack floods your service with traffic from many machines until real users can't get through. **Shield Standard** automatically protects every AWS customer against common network-layer floods at no cost. **Shield Advanced** adds detection and mitigation of application-layer (HTTP) attacks, 24/7 access to the AWS Shield Response Team, and refunds for the scaling bill an attack causes.

**Key ideas to know**
- **Layer 3/4 attacks**: network floods such as SYN floods and UDP reflection. Standard handles these.
- **Layer 7 attacks**: floods of valid-looking HTTP requests. Advanced can write WAF rules automatically to mitigate them.
- **SRT** (Shield Response Team): AWS experts who help during an attack (Advanced only, and it needs a qualifying AWS Support plan).
- **Cost protection**: credits for EC2, ELB, CloudFront and Route 53 charges caused by an attack (Advanced).
- **Protected resources (Advanced)**: CloudFront, Route 53 hosted zones, Global Accelerator, ELB and EC2 Elastic IPs.

**Use it when / don't use it when**
- ✅ Standard: always, automatically. Nothing to switch on.
- ✅ Advanced: business-critical public apps, "DDoS cost protection", "24/7 DDoS experts".
- ❌ Blocking SQL injection or bad bots → [AWS WAF](#aws-waf).

**Exam hot spots**
- "Enable Shield Standard" is never a needed step: it's already on.
- SRT access, cost protection and automatic L7 mitigation are **Advanced only**.
- The DDoS-resilient pattern: Route 53 + CloudFront + WAF + Shield in front of an ALB and Auto Scaling.

**Go deeper:** [Notes 07: AWS Shield](../../notes/07-security-data-protection.md#62-aws-shield) · related services: [Amazon CloudFront](networking.md#amazon-cloudfront) · [Amazon Route 53](networking.md#amazon-route-53)

---

## AWS Firewall Manager

> **In one line:** sets firewall and protection rules once and enforces them across every account in an AWS Organization, including accounts created later.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like a GitOps repo that pushes the same NetworkPolicies and Gatekeeper policies to every cluster in the fleet, and to every new cluster automatically | Per protection policy per Region, plus the WAF, Shield or firewall resources it creates |

**What it is.** With 50 accounts, configuring WAF rules or security groups account by account is slow and drifts out of sync. Firewall Manager lets a central security account define a policy ("every ALB tagged `prod` must have this web ACL") and applies it everywhere it matches, now and in future.

**Key ideas to know**
- **Policy types**: AWS WAF web ACLs, Shield Advanced protections, security groups (baselines and audits for over-permissive rules), AWS Network Firewall, Route 53 Resolver DNS Firewall, and network ACLs.
- **Prerequisites**: [AWS Organizations](identity-governance.md#aws-organizations) with all features, a Firewall Manager administrator account, and [AWS Config](management-monitoring.md#aws-config) enabled in member accounts.
- **Auto-remediation**: new accounts and resources that match the scope get the rules automatically.

**Use it when / don't use it when**
- ✅ "Same WAF rules in all accounts, including **future** ones."
- ❌ One account, one ALB → configure [AWS WAF](#aws-waf) directly.

**Exam hot spots**
- Keywords "organization-wide", "centrally", "new accounts automatically" + firewall → **Firewall Manager**.
- It needs Organizations **and** AWS Config.

**Go deeper:** [Notes 07: AWS Firewall Manager](../../notes/07-security-data-protection.md#63-aws-firewall-manager) · related services: [AWS WAF](#aws-waf) · [AWS Shield](#aws-shield)

---

## AWS Network Firewall

> **In one line:** a managed, stateful network firewall and intrusion prevention system that inspects traffic flowing into, out of and between your VPCs.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like an egress gateway with FQDN-based policies (as in Cilium or Istio), but as a routed appliance for the whole VPC rather than a per-pod rule. Everyday: a customs checkpoint every vehicle must drive through. | Per firewall endpoint per hour + per GB processed |

**What it is.** Security groups allow or deny by IP and port. Sometimes you need more: "servers may only reach `*.amazonaws.com` and `updates.example.com`", or "drop traffic matching known attack signatures". Network Firewall does this for a whole VPC. You'd otherwise run third-party firewall appliances on EC2 and handle their scaling and failover yourself.

**Key ideas to know**
- **Firewall endpoint**: placed in a dedicated **firewall subnet** in each AZ. You change **route tables** so traffic passes through it (e.g., private subnet → firewall endpoint → NAT gateway → internet).
- **Stateless rules**: simple per-packet checks (IP, port, protocol).
- **Stateful rules**: track connections, filter by **domain name / FQDN** (fully qualified domain name), and run **IPS** (intrusion prevention system) signatures in Suricata-compatible format.
- **TLS inspection**: optionally decrypt and inspect encrypted traffic.
- **Centralized inspection**: one inspection VPC behind an [AWS Transit Gateway](networking.md#aws-transit-gateway) can inspect traffic for many VPCs.

**Use it when / don't use it when**
- ✅ "Filter outbound traffic by domain name", "intrusion prevention", "inspect traffic between VPCs centrally".
- ❌ HTTP attacks like SQL injection on a public web app → [AWS WAF](#aws-waf).
- ❌ Simple port/IP filtering on one instance → a security group ([Amazon VPC](networking.md#amazon-vpc)).

**Exam hot spots**
- "Egress filtering by FQDN" or "IDS/IPS for the VPC" → **Network Firewall**.
- At scale: Transit Gateway + inspection VPC with Network Firewall.

**Go deeper:** [Notes 02: Network security layers](../../notes/02-networking-vpc.md#13-network-security-layers) · related services: [Amazon VPC](networking.md#amazon-vpc) · [AWS Firewall Manager](#aws-firewall-manager)

---

## Amazon GuardDuty

> **In one line:** an always-on threat detection service that reads your AWS logs and alerts you to attacks, compromised resources and stolen credentials.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★★ High | Like Falco plus a threat-intelligence feed, watching both your AWS account and your EKS clusters; or a building's security guard watching all the CCTV feeds | By volume of logs and events analysed; 30-day free trial |

**What it is.** Attacks show up as strange patterns: an EC2 instance talking to a known crypto-mining server, API calls from an unusual country, or instance credentials used from outside AWS. GuardDuty spots these with machine learning, anomaly detection and threat-intelligence lists. You turn it on with one click. You don't have to enable or store the logs it analyses yourself.

**Key ideas to know**
- **Foundational data sources**: CloudTrail management events, VPC Flow Logs and DNS query logs. GuardDuty reads its own copies, so you don't need to turn these on.
- **Protection plans** (optional extras): S3 data events, **EKS audit logs**, **Runtime Monitoring** (an agent for EKS, ECS on Fargate and EC2), Malware Protection for EC2 and S3, RDS login activity, Lambda network activity.
- **Finding**: an alert with a type (e.g., `CryptoCurrency:EC2/BitcoinTool.B`), a severity and the affected resource.
- **No auto-fix**: GuardDuty only detects. Send findings to [Amazon EventBridge](integration.md#amazon-eventbridge), which can trigger a Lambda function to isolate the instance or notify the team.
- **Multi-account**: a delegated administrator account can manage GuardDuty for the whole organization.

**Use it when / don't use it when**
- ✅ "Detect compromised instances, unusual API calls, crypto-mining, malicious IPs."
- ❌ Find known software vulnerabilities → [Amazon Inspector](#amazon-inspector).
- ❌ Find personal data in S3 → [Amazon Macie](#amazon-macie).

**Exam hot spots**
- GuardDuty doesn't require you to enable Flow Logs, DNS logs or a trail.
- Automatic response = **GuardDuty → EventBridge → Lambda** (or SSM Automation).
- It's a Regional service: enable it in every Region you use.

**Try it** (free trial): enable GuardDuty in one Region, then use *Settings → Generate sample findings* to see what alerts look like. Disable it before the trial ends if you don't want charges.

**Go deeper:** [Notes 07: Threat detection](../../notes/07-security-data-protection.md#7-threat-detection-and-security-posture) · related services: [Amazon Detective](#amazon-detective) · [AWS Security Hub](#aws-security-hub)

---

## Amazon Inspector

> **In one line:** continuously scans EC2 instances, container images in ECR and Lambda functions for known software vulnerabilities.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like Trivy or Grype scanning your images, except it keeps rescanning after deployment as new CVEs are published | Per EC2 instance, container image and Lambda function scanned, per month |

**What it is.** Software has known security holes, catalogued as **CVEs** (Common Vulnerabilities and Exposures). Inspector checks your operating system packages, container images and function dependencies against those lists and tells you what to patch, ranked by risk. It rescans automatically when a new CVE is published, so an image that was clean last month gets flagged today.

**Key ideas to know**
- **EC2 scanning**: uses the SSM Agent, or an agentless mode that scans EBS snapshots.
- **ECR scanning**: scans images when pushed to Amazon ECR (Elastic Container Registry) and again as new CVEs appear.
- **Lambda scanning**: checks function packages and, optionally, the code.
- **Network reachability**: flags EC2 ports that are reachable from the internet.
- **Risk score**: adjusts severity for your environment (e.g., lower if the instance isn't reachable).

**Use it when / don't use it when**
- ✅ "Scan EC2 / container images / Lambda for vulnerabilities continuously."
- ❌ Detect active attacks → [Amazon GuardDuty](#amazon-guardduty).
- ❌ Scan RDS databases → not supported; AWS patches the RDS engine for you.

**Exam hot spots**
- "CVE", "vulnerabilities", "patch" + EC2/ECR/Lambda → **Inspector**.
- Inspector scans; it doesn't patch. Patching is done with [AWS Systems Manager](management-monitoring.md#aws-systems-manager) Patch Manager.

**Go deeper:** [Notes 07: Threat detection](../../notes/07-security-data-protection.md#7-threat-detection-and-security-posture) · related services: [Amazon ECR](containers.md#amazon-ecr) · [Amazon EC2](compute.md#amazon-ec2)

---

## Amazon Macie

> **In one line:** uses pattern matching and machine learning to find sensitive data, such as personal or financial details, stored in S3.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Everyday: an auditor walking through the filing cabinets to find every document with a passport number in it | Per S3 bucket monitored + per GB of data inspected |

**What it is.** Companies often don't know where personal data ends up: a CSV export dropped in a bucket, a log file with credit card numbers. Regulations such as GDPR (the EU's General Data Protection Regulation) require you to know. Macie scans S3 objects for **PII** (personally identifiable information), financial data and credentials, and also reports risky bucket settings such as public or unencrypted buckets.

**Key ideas to know**
- **Managed data identifiers**: built-in detectors for names, addresses, credit card numbers, passport numbers, API keys and more.
- **Custom data identifiers**: your own regex patterns, such as employee IDs.
- **Automated discovery**: samples objects across all buckets continuously. **Sensitive data discovery jobs** do targeted, deeper scans.
- **Bucket inventory**: flags buckets that are public, unencrypted or shared with other accounts.
- Findings go to EventBridge and Security Hub.

**Use it when / don't use it when**
- ✅ "Find PII in S3", "data inventory for GDPR".
- ❌ Data in RDS, DynamoDB or EBS → Macie is **S3 only**.

**Exam hot spots**
- "Sensitive data / PII" + "S3" → **Macie**.

**Go deeper:** [Notes 07: Threat detection](../../notes/07-security-data-protection.md#7-threat-detection-and-security-posture) · related services: [Amazon S3](storage.md#amazon-s3)

---

## Amazon Detective

> **In one line:** turns logs and GuardDuty findings into a linked graph so you can investigate what happened and why.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | A detective's evidence board with photos connected by string | Per GB of data ingested |

**What it is.** GuardDuty tells you *something* is wrong. Detective helps answer *what else* that IP address, role or instance did, by building a behaviour graph from CloudTrail, VPC Flow Logs, EKS audit logs and GuardDuty findings.

**Exam hot spots**
- "Investigate / root cause / visualize a finding" → **Detective**. It investigates; it doesn't detect first.

**Go deeper:** [Notes 07: Threat detection](../../notes/07-security-data-protection.md#7-threat-detection-and-security-posture) · related services: [Amazon GuardDuty](#amazon-guardduty)

---

## AWS Security Hub

> **In one line:** one place to see security findings from all your accounts and services, plus automated checks against security best practices.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★★ Medium | Like a single dashboard that collects Falco alerts, Trivy results and kube-bench scores from every cluster; or a building's central security control room | Usage-based (checks run, findings and resources covered); see the pricing page |

**What it is.** Each detection service produces its own findings in its own console. Security Hub collects them from GuardDuty, Inspector, Macie, Firewall Manager, IAM Access Analyzer, AWS Config and partner tools, puts them in one common format, and lets you view them across every account and Region. It also runs **posture checks**: automated tests of your settings against security standards, giving you a score.

**Key ideas to know**
- **Aggregation**: findings from many services and accounts in one view, usually managed from a delegated administrator account.
- **Security standards**: AWS Foundational Security Best Practices, the CIS (Center for Internet Security) AWS Foundations Benchmark, PCI DSS (the payment card industry standard) and NIST 800-53.
- **Depends on AWS Config**: the posture checks use Config rules, so Config must be enabled.
- **Naming change**: in December 2025 AWS launched a broader, unified Security Hub that correlates signals across services, and renamed the original posture-check service **Security Hub CSPM** (cloud security posture management). For the exam, "Security Hub" means central aggregation plus standards-based checks.
- **Automation**: findings go to EventBridge for automatic responses.

**Use it when / don't use it when**
- ✅ "Single dashboard of security findings across accounts", "compliance score against CIS".
- ❌ Detecting threats itself → it relies on [Amazon GuardDuty](#amazon-guardduty) and others for that.

**Exam hot spots**
- "Central view of findings from multiple accounts and services" → **Security Hub**.
- Security Hub (like Firewall Manager) needs **AWS Config** enabled.

**Go deeper:** [Notes 07: Threat detection](../../notes/07-security-data-protection.md#7-threat-detection-and-security-posture) · related services: [AWS Config](management-monitoring.md#aws-config) · [Amazon GuardDuty](#amazon-guardduty)

---

## AWS Artifact

> **In one line:** a self-service portal to download AWS's own compliance reports and accept legal agreements such as the HIPAA BAA.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | Asking the landlord for the building's fire-safety certificate | Free |

**What it is.** Auditors want proof that AWS's data centres and services meet standards such as SOC (System and Organization Controls) 1/2/3, ISO 27001 and PCI DSS. Artifact lets you download those reports and accept agreements such as the HIPAA (U.S. health data law) **BAA** (Business Associate Addendum).

**Exam hot spots**
- "Download a SOC report / accept the BAA" → **Artifact**. It covers *AWS's* side of shared responsibility, not your resources.

**Go deeper:** [Notes 07: Audit Manager and Artifact](../../notes/07-security-data-protection.md#83-aws-audit-manager-and-aws-artifact) · related services: [AWS Audit Manager](#aws-audit-manager)

---

## AWS Audit Manager

> **In one line:** continuously collects evidence from your AWS accounts and maps it to compliance frameworks to prepare *your* audits.

| Exam importance | Kubernetes / everyday analogy | Pricing model (rough) |
|---|---|---|
| ★ Low | An evidence binder for your auditor that fills itself in | Per resource assessment (evidence collected) |

**What it is.** Audit Manager gathers evidence from AWS Config, CloudTrail and Security Hub and organizes it against frameworks such as PCI DSS, HIPAA and SOC 2, so you can produce an audit report. ⚠️ It moved to maintenance mode and has been **closed to new customers since April 30, 2026**. AWS points new users to AWS Config conformance packs instead.

**Exam hot spots**
- Artifact = **AWS's** compliance reports. Audit Manager = evidence about **your** workloads.

**Go deeper:** [Notes 07: Audit Manager and Artifact](../../notes/07-security-data-protection.md#83-aws-audit-manager-and-aws-artifact) · related services: [AWS Config](management-monitoring.md#aws-config) · [AWS Artifact](#aws-artifact)

---

**Next:** [Networking services](networking.md)
