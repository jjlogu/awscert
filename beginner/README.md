# Beginner Track: Start Here If You're New to AWS
> [Home](../README.md) · [Study plan](../00-study-plan.md) · [All services A–Z](services/README.md) · [Glossary](glossary.md)

The [notes modules](../README.md#file-index) are written at exam depth and move fast. This track comes **before** them. It explains AWS from first principles, so that each notes module builds on things you already understand.

**If you've only touched AWS through `kubectl` on EKS, you know more than you think.** Containers, scaling, declarative config, service discovery and health checks all carry over. What's new is the AWS layer underneath: networks (VPC), permissions (IAM), managed databases and storage, and cost. This track maps your Kubernetes knowledge onto AWS and fills the gaps.

---

## Reading order

| # | Read | Time | What you'll be able to do afterwards |
|---|---|---|---|
| 1 | [Cloud & AWS basics](01-cloud-and-aws-basics.md) | ~3 h | Explain Regions/AZs, the shared responsibility model, how AWS bills you, and follow a request through a typical AWS architecture |
| 2 | [Networking basics](02-networking-basics.md) | ~4 h | Read CIDR blocks, explain routing, NAT, DNS, TLS, firewalls and load balancing, and picture a 2-AZ VPC |
| 3 | [Kubernetes → AWS map](03-kubernetes-to-aws.md) | ~1.5 h | Translate what you know from EKS into AWS services, and know where the analogy breaks |
| 4 | [AWS services, one page per category](services/README.md) | ~6 h for ★★★ services | Say what each exam service does, when to use it, and how the exam tests it |
| — | [Glossary](glossary.md) | as needed | Look up any term you hit in the notes or practice questions |

Total: about **15 hours**, which is **Week 0** in the [study plan](../00-study-plan.md#week-0-foundations-new-to-aws).

---

## How to use the service pages

Every AWS service in the exam has its own section with the same layout: a one-line summary, an **exam importance** rating, a Kubernetes or everyday analogy, the key ideas, when to use it and when not to, exam hot spots, an optional 10-minute "try it", and a link to the deep-dive notes.

| Rating | Meaning | How to study it |
|---|---|---|
| ★★★ | Core: appears in many questions | Read fully in Week 0. Re-read before its notes module. |
| ★★ | Appears regularly | Read before its notes module. |
| ★ | Recognition only | Know the one-liner, so you can eliminate it (or pick it) in an answer list. |

| Category page | Read before | Services |
|---|---|---|
| [Identity & governance](services/identity-governance.md) | [Notes 01](../notes/01-iam-multi-account.md) | IAM, STS, IAM Identity Center, Organizations, Control Tower, RAM, Directory Service, Cognito |
| [Networking & content delivery](services/networking.md) | [Notes 02](../notes/02-networking-vpc.md), [08](../notes/08-ha-dr-edge-ops.md) | VPC, ELB, Route 53, CloudFront, Global Accelerator, PrivateLink, Transit Gateway, VPN, Direct Connect |
| [Compute](services/compute.md) | [Notes 03](../notes/03-compute-containers.md) | EC2, Auto Scaling, Lambda, Elastic Beanstalk, Batch, Outposts, Local Zones, Wavelength… |
| [Containers](services/containers.md) | [Notes 03](../notes/03-compute-containers.md) | EKS, ECS, Fargate, ECR, App Runner, ECS/EKS Anywhere |
| [Storage](services/storage.md) | [Notes 04](../notes/04-storage.md) | S3, S3 Glacier, EBS, instance store, EFS, FSx, Storage Gateway, AWS Backup |
| [Databases](services/databases.md) | [Notes 05](../notes/05-databases.md) | RDS, Aurora, DynamoDB, ElastiCache, MemoryDB, Redshift, DocumentDB, Neptune, Keyspaces, OpenSearch… |
| [Integration & messaging](services/integration.md) | [Notes 06](../notes/06-decoupling-serverless.md) | SQS, SNS, EventBridge, Step Functions, MQ, API Gateway, AppFlow, Amplify |
| [Security](services/security.md) | [Notes 07](../notes/07-security-data-protection.md) | KMS, CloudHSM, ACM, Secrets Manager, WAF, Shield, GuardDuty, Inspector, Macie, Security Hub… |
| [Management & monitoring](services/management-monitoring.md) | [Notes 08](../notes/08-ha-dr-edge-ops.md) | CloudWatch, CloudTrail, Config, Systems Manager, CloudFormation, X-Ray, Trusted Advisor… |
| [Analytics](services/analytics.md) | [Notes 09](../notes/09-data-analytics-migration.md) | Kinesis, Data Firehose, MSK, Glue, Athena, Lake Formation, EMR, Amazon Quick… |
| [Migration & transfer](services/migration-transfer.md) | [Notes 09](../notes/09-data-analytics-migration.md) | DataSync, Transfer Family, Snow Family, Application Migration Service, DMS, Elastic Disaster Recovery… |
| [AI & ML](services/ai-ml.md) | [Notes 09](../notes/09-data-analytics-migration.md) | SageMaker AI, Rekognition, Textract, Comprehend, Transcribe, Translate, Polly, Lex |
| [Cost management](services/cost-management.md) | [Notes 10](../notes/10-cost-optimization.md) | Free Tier, Pricing Calculator, Budgets, Cost Explorer, CUR, Savings Plans, cost allocation tags |

---

## Tips for learning AWS from zero
- **Click around before you read deeply.** Open each service's console once, read-only, so the names have a picture attached. The *Try it* boxes tell you where to look.
- **Draw it.** After each networking or architecture section, sketch the boxes (Region → VPC → AZ → subnet → resource) from memory.
- **Don't memorize everything.** The exam rewards choosing the right service for a scenario, not knowing every setting. The ★ ratings tell you where to spend time.
- **Use the [glossary](glossary.md) freely.** Every acronym the guide uses is defined there.
- **Protect your wallet.** Read [Lab safety](../labs/README.md) and set a budget alert before creating anything.

**Next:** [Cloud & AWS basics](01-cloud-and-aws-basics.md)
