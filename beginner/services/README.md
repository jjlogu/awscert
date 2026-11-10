# All AWS Services A–Z (Beginner Reference)
> [Home](../../README.md) · [Beginner track](../README.md) · [Glossary](../glossary.md)

Every service covered in the beginner track: **139 services**. Each links to its plain-English section (what it is, an analogy, key ideas, when to use it, exam hot spots). Sorted by the distinctive part of the name, so *Amazon S3* is under **S** and *AWS Lambda* under **L**.

**Exam importance:** ★★★ core · ★★ appears regularly · ★ recognition only (know the one-liner)

**Browse by category:** [Identity & governance](identity-governance.md) · [Security](security.md) · [Networking](networking.md) · [Compute](compute.md) · [Containers](containers.md) · [Storage](storage.md) · [Databases](databases.md) · [Integration](integration.md) · [Analytics](analytics.md) · [Migration & transfer](migration-transfer.md) · [Management & monitoring](management-monitoring.md) · [Cost management](cost-management.md) · [AI & ML](ai-ml.md)

**Jump to:** [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [V](#v) · [W](#w) · [X](#x)

## A

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Amplify](integration.md#aws-amplify) | A toolkit and hosting service that lets front-end and mobile developers build, deploy and host full-stack apps quickly. | ★ | [Integration](integration.md) |
| [Amazon API Gateway](integration.md#amazon-api-gateway) | A fully managed "front door" that publishes HTTP, REST and WebSocket APIs and forwards each call to Lambda, containers, other AWS services or any HTTP backend. | ★★★ | [Integration](integration.md) |
| [AWS App Runner](containers.md#aws-app-runner) | A fully managed service that turned a container image or source repo into a running HTTPS web app with scaling, with almost no configuration. | ★ | [Containers](containers.md) |
| [Amazon AppFlow](integration.md#amazon-appflow) | A no-code service that moves data between SaaS (software as a service) apps such as Salesforce, SAP, ServiceNow or Zendesk and AWS services such as S3 and Redshift. | ★ | [Integration](integration.md) |
| [AWS Application Discovery Service](migration-transfer.md#aws-application-discovery-service) | Collects inventory, performance and network-dependency data from on-premises servers to plan a migration. | ★ | [Migration & transfer](migration-transfer.md) |
| [AWS Application Migration Service](migration-transfer.md#aws-application-migration-service) | AWS's main "lift-and-shift" tool: it continuously replicates whole servers into AWS and lets you cut over to EC2 with minutes of downtime. | ★★ | [Migration & transfer](migration-transfer.md) |
| [AWS Artifact](security.md#aws-artifact) | A self-service portal to download AWS's own compliance reports and accept legal agreements such as the HIPAA BAA. | ★ | [Security](security.md) |
| [Amazon Athena](analytics.md#amazon-athena) | A serverless SQL query service that reads data directly from files in S3; there's nothing to load or run, and you pay per amount of data scanned. | ★★★ | [Analytics](analytics.md) |
| [AWS Audit Manager](security.md#aws-audit-manager) | Continuously collects evidence from your AWS accounts and maps it to compliance frameworks to prepare *your* audits. | ★ | [Security](security.md) |
| [Amazon Aurora](databases.md#amazon-aurora) | A MySQL- and PostgreSQL-compatible relational database rebuilt by AWS for the cloud, with a shared storage layer copied across three AZs. | ★★★ | [Databases](databases.md) |
| [AWS Auto Scaling](compute.md#aws-auto-scaling) | A single console and "scaling plan" to configure scaling across several resource types at once: EC2 Auto Scaling groups, ECS services, DynamoDB tables, Aurora replicas and Spot Fleets. | ★ | [Compute](compute.md) |

## B

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Backup](storage.md#aws-backup) | One place to schedule, retain, copy and audit backups for many AWS services, across accounts and Regions. | ★★ | [Storage](storage.md) |
| [AWS Batch](compute.md#aws-batch) | A fully managed service that queues batch jobs and runs them on automatically provisioned compute, scaling to zero when the work is done. | ★★ | [Compute](compute.md) |
| [Amazon Bedrock](ai-ml.md#amazon-bedrock) | A fully managed API for using and customizing **generative AI foundation models** (from Amazon and other model providers) without running any GPUs yourself. | ★ | [AI & ML](ai-ml.md) |
| [AWS Budgets](cost-management.md#aws-budgets) | Set a spending or usage limit and get an alert (or trigger an automatic action) when actual or **forecasted** spend crosses it. | ★★★ | [Cost management](cost-management.md) |

## C

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Certificate Manager (ACM)](security.md#aws-certificate-manager-acm) | Issues free public TLS certificates for AWS load balancers, CloudFront and API Gateway, and renews them automatically. | ★★ | [Security](security.md) |
| [AWS CLI](management-monitoring.md#aws-cli) | A command-line tool (`aws ...`) that can call every AWS API from your terminal or scripts. | ★ | [Management & monitoring](management-monitoring.md) |
| [AWS Client VPN](networking.md#aws-client-vpn) | A managed, OpenVPN-based VPN that lets individual users connect their laptops securely to a VPC. | ★ | [Networking](networking.md) |
| [AWS CloudFormation](management-monitoring.md#aws-cloudformation) | Infrastructure as code (IaC): you describe AWS resources in a YAML or JSON template, and CloudFormation creates, updates and deletes them as one unit. | ★★★ | [Management & monitoring](management-monitoring.md) |
| [Amazon CloudFront](networking.md#amazon-cloudfront) | A content delivery network (CDN) that serves your website, API or video from hundreds of edge locations near your users, caching what it can. | ★★★ | [Networking](networking.md) |
| [AWS CloudHSM](security.md#aws-cloudhsm) | A dedicated, single-tenant hardware security module (HSM) in your VPC, where only you can access the keys. | ★★ | [Security](security.md) |
| [AWS CloudShell](management-monitoring.md#aws-cloudshell) | A terminal in your browser, opened from the console, with the AWS CLI already installed and logged in as you. | ★ | [Management & monitoring](management-monitoring.md) |
| [AWS CloudTrail](management-monitoring.md#aws-cloudtrail) | A tamper-evident log of every API call made in your AWS account: who made it, when, from which IP address, and whether it succeeded. | ★★★ | [Management & monitoring](management-monitoring.md) |
| [Amazon CloudWatch](management-monitoring.md#amazon-cloudwatch) | AWS's built-in monitoring service: it collects metrics and logs, raises alarms, and draws dashboards. | ★★★ | [Management & monitoring](management-monitoring.md) |
| [Amazon Cognito](identity-governance.md#amazon-cognito) | A managed user directory and login service for *your application's* customers, plus a way to give them temporary AWS credentials. | ★★★ | [Identity & governance](identity-governance.md) |
| [Amazon Comprehend](ai-ml.md#amazon-comprehend) | Natural-language processing (NLP) for text: sentiment, entities, key phrases, language detection, topics and **PII detection**. | ★ | [AI & ML](ai-ml.md) |
| [AWS Compute Optimizer](management-monitoring.md#aws-compute-optimizer) | Uses machine learning on your CloudWatch history to tell you which resources are over- or under-sized, and what size to use instead. | ★★ | [Management & monitoring](management-monitoring.md) |
| [AWS Config](management-monitoring.md#aws-config) | Records how each AWS resource is configured over time, and checks those configurations against rules you define. | ★★★ | [Management & monitoring](management-monitoring.md) |
| [AWS Control Tower](identity-governance.md#aws-control-tower) | A wizard and ongoing governance layer that sets up a secure, best-practice multi-account AWS environment for you. | ★★ | [Identity & governance](identity-governance.md) |
| [Cost allocation tags](cost-management.md#cost-allocation-tags) | Key-value tags (like `team=payments`) that, once **activated** in Billing, let you split your AWS bill by team, application or environment. | ★★ | [Cost management](cost-management.md) |
| [AWS Cost and Usage Report](cost-management.md#aws-cost-and-usage-report) | The most detailed billing data AWS offers, with line items per resource per hour or day, delivered as files to an S3 bucket for analysis. | ★★ | [Cost management](cost-management.md) |
| [AWS Cost Anomaly Detection](cost-management.md#aws-cost-anomaly-detection) | Machine learning watches your spending patterns and alerts you to unusual spikes, with a hint about the root cause. | ★★ | [Cost management](cost-management.md) |
| [AWS Cost Explorer](cost-management.md#aws-cost-explorer) | Interactive charts of what you've spent, broken down any way you like, plus forecasts and recommendations for saving money. | ★★★ | [Cost management](cost-management.md) |

## D

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Data Exchange](analytics.md#aws-data-exchange) | A catalog where you find, subscribe to and receive third-party datasets (files, APIs, Redshift data shares) in AWS. | ★ | [Analytics](analytics.md) |
| [Amazon Data Firehose](analytics.md#amazon-data-firehose) | A fully managed delivery service that buffers streaming data and loads it into S3, Redshift, OpenSearch, Splunk or HTTP endpoints, with no servers or shards to manage. | ★★★ | [Analytics](analytics.md) |
| [AWS Data Pipeline](analytics.md#aws-data-pipeline) | A legacy service for scheduling data movement and processing between AWS services and on-premises sources. | ★ | [Analytics](analytics.md) |
| [AWS DataSync](migration-transfer.md#aws-datasync) | A managed online service that copies files and objects between on-premises storage, other clouds and AWS storage, quickly, securely and on a schedule. | ★★★ | [Migration & transfer](migration-transfer.md) |
| [Amazon Detective](security.md#amazon-detective) | Turns logs and GuardDuty findings into a linked graph so you can investigate what happened and why. | ★ | [Security](security.md) |
| [AWS Device Farm](integration.md#aws-device-farm) | An app-testing service that runs your tests on real mobile devices and desktop browsers hosted by AWS. | ★ | [Integration](integration.md) |
| [AWS Direct Connect](networking.md#aws-direct-connect) | A private, dedicated network connection from your data center to AWS that bypasses the public internet. | ★★★ | [Networking](networking.md) |
| [AWS Directory Service](identity-governance.md#aws-directory-service) | Gives you Microsoft Active Directory in AWS, or connects AWS to the Active Directory you already run on-premises. | ★★ | [Identity & governance](identity-governance.md) |
| [AWS DMS](migration-transfer.md#aws-dms) | AWS Database Migration Service migrates databases to AWS (and keeps them in sync while you switch over) with minimal downtime, even between different database engines. | ★★★ | [Migration & transfer](migration-transfer.md) |
| [Amazon DocumentDB](databases.md#amazon-documentdb) | A managed document database compatible with the MongoDB API, for storing and querying JSON-like documents. | ★★ | [Databases](databases.md) |
| [Amazon DynamoDB](databases.md#amazon-dynamodb) | A fully managed, serverless NoSQL key-value and document database that delivers single-digit-millisecond responses at virtually any scale. | ★★★ | [Databases](databases.md) |

## E

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon EBS](storage.md#amazon-ebs) | Amazon Elastic Block Store (EBS) provides network-attached virtual disks (volumes) for EC2 instances, living in one Availability Zone. | ★★★ | [Storage](storage.md) |
| [Amazon EC2](compute.md#amazon-ec2) | Rent virtual machines ("instances") in seconds, choose their CPU, memory, storage and networking, and pay only while they run. | ★★★ | [Compute](compute.md) |
| [Amazon EC2 Auto Scaling](compute.md#amazon-ec2-auto-scaling) | Keeps a group of EC2 instances at the right size, replacing unhealthy ones and adding or removing instances as load changes. | ★★★ | [Compute](compute.md) |
| [EC2 instance store](storage.md#ec2-instance-store) | Disks physically attached to the server your EC2 instance runs on: extremely fast, but the data disappears when the instance stops. | ★★ | [Storage](storage.md) |
| [Amazon ECR](containers.md#amazon-ecr) | Amazon Elastic Container Registry (ECR) is a private, managed Docker/OCI (Open Container Initiative) image registry integrated with IAM. | ★★ | [Containers](containers.md) |
| [Amazon ECS](containers.md#amazon-ecs) | Amazon Elastic Container Service (ECS) is AWS's own container orchestrator: you describe containers in a task definition, and ECS keeps the right number of copies running. | ★★★ | [Containers](containers.md) |
| [Amazon ECS Anywhere](containers.md#amazon-ecs-anywhere) | Run ECS tasks on your own on-premises servers or VMs while the ECS control plane stays in an AWS Region. | ★ | [Containers](containers.md) |
| [Amazon EFS](storage.md#amazon-efs) | Amazon Elastic File System (EFS) is a managed, auto-growing NFS file system that many Linux machines across AZs can mount at once. | ★★★ | [Storage](storage.md) |
| [Amazon EKS](containers.md#amazon-eks) | Amazon Elastic Kubernetes Service (EKS) runs the Kubernetes control plane for you, and plugs Kubernetes into AWS networking, load balancing, storage and identity. | ★★★ | [Containers](containers.md) |
| [Amazon EKS Anywhere](containers.md#amazon-eks-anywhere) | Tooling to create and operate EKS-compatible Kubernetes clusters on your own infrastructure (VMware vSphere, bare metal, Nutanix and others). | ★ | [Containers](containers.md) |
| [Amazon EKS Distro](containers.md#amazon-eks-distro) | The open-source Kubernetes distribution (same versions and security patches) that EKS runs, which you can install anywhere yourself. | ★ | [Containers](containers.md) |
| [AWS Elastic Beanstalk](compute.md#aws-elastic-beanstalk) | A platform as a service (PaaS): upload your application code, and Beanstalk creates and manages the EC2 instances, load balancer, scaling and monitoring for it. | ★★ | [Compute](compute.md) |
| [AWS Elastic Disaster Recovery](migration-transfer.md#aws-elastic-disaster-recovery) | Keeps a continuously replicated, low-cost copy of your servers in AWS so you can recover them within minutes if the primary site fails. | ★★ | [Migration & transfer](migration-transfer.md) |
| [Elastic Load Balancing (ELB)](networking.md#elastic-load-balancing-elb) | Managed load balancers that receive client traffic and spread it across healthy targets in several AZs. | ★★★ | [Networking](networking.md) |
| [Amazon ElastiCache](databases.md#amazon-elasticache) | A managed in-memory data store (Valkey, Redis OSS or Memcached) used mostly as a cache and session store. | ★★★ | [Databases](databases.md) |
| [Amazon EMR](analytics.md#amazon-emr) | Managed clusters for big-data frameworks such as Apache Spark, Hive, HBase, Presto/Trino and Flink. | ★★ | [Analytics](analytics.md) |
| [Amazon EventBridge](integration.md#amazon-eventbridge) | A serverless event bus that receives events from AWS services, SaaS apps and your own code, and routes each one to targets based on what's inside it. | ★★★ | [Integration](integration.md) |

## F

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Fargate](containers.md#aws-fargate) | A serverless compute engine for containers: you say how much CPU and memory each task or pod needs, and AWS finds somewhere to run it. | ★★★ | [Containers](containers.md) |
| [AWS Firewall Manager](security.md#aws-firewall-manager) | Sets firewall and protection rules once and enforces them across every account in an AWS Organization, including accounts created later. | ★★ | [Security](security.md) |
| [Amazon Forecast](ai-ml.md#amazon-forecast) | Time-series forecasting (sales, demand, capacity) from historical data. **Closed to new customers since July 29, 2024.** | ★ | [AI & ML](ai-ml.md) |
| [AWS Free Tier](cost-management.md#aws-free-tier) | The free starter offer for new AWS accounts: credits to spend while you learn, plus a set of always-free services. | ★ | [Cost management](cost-management.md) |
| [Amazon FSx](storage.md#amazon-fsx) | Fully managed versions of four popular third-party file systems (Windows File Server, Lustre, NetApp ONTAP and OpenZFS). | ★★★ | [Storage](storage.md) |

## G

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Global Accelerator](networking.md#aws-global-accelerator) | Gives your application two fixed (anycast) IP addresses worldwide and carries user traffic over the AWS backbone to the closest healthy Regional endpoint. | ★★ | [Networking](networking.md) |
| [AWS Glue](analytics.md#aws-glue) | A serverless data integration service: a central data catalog, crawlers that discover schemas, and Spark-based ETL jobs. | ★★★ | [Analytics](analytics.md) |
| [Amazon GuardDuty](security.md#amazon-guardduty) | An always-on threat detection service that reads your AWS logs and alerts you to attacks, compromised resources and stolen credentials. | ★★★ | [Security](security.md) |

## H

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Health Dashboard](management-monitoring.md#aws-health-dashboard) | Shows AWS service problems and scheduled maintenance, and specifically which ones affect **your** resources. | ★★ | [Management & monitoring](management-monitoring.md) |

## I

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS IAM](identity-governance.md#aws-iam) | IAM (Identity and Access Management) decides who can call which AWS action on which resource, in one AWS account. | ★★★ | [Identity & governance](identity-governance.md) |
| [AWS IAM Identity Center](identity-governance.md#aws-iam-identity-center) | One login for employees that gives them the right access in many AWS accounts (the successor to AWS SSO). | ★★★ | [Identity & governance](identity-governance.md) |
| [Amazon Inspector](security.md#amazon-inspector) | Continuously scans EC2 instances, container images in ECR and Lambda functions for known software vulnerabilities. | ★★ | [Security](security.md) |

## K

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Kendra](ai-ml.md#amazon-kendra) | ML-powered enterprise search that answers natural-language questions across company documents. **Closed to new customers since July 30, 2026.** | ★ | [AI & ML](ai-ml.md) |
| [Amazon Keyspaces](databases.md#amazon-keyspaces) | A serverless, managed database compatible with Apache Cassandra, so Cassandra apps run without Cassandra clusters. | ★★ | [Databases](databases.md) |
| [Amazon Kinesis Data Streams](analytics.md#amazon-kinesis-data-streams) | A managed, real-time data stream that stores ordered records for 24 hours to 365 days so several applications can each read and replay them. | ★★★ | [Analytics](analytics.md) |
| [Amazon Kinesis Video Streams](analytics.md#amazon-kinesis-video-streams) | Securely ingests, stores and plays back video and other time-encoded media from cameras and devices. | ★ | [Analytics](analytics.md) |
| [AWS KMS](security.md#aws-kms) | KMS (Key Management Service) creates, stores and controls the encryption keys that S3, EBS (Elastic Block Store), RDS (Relational Database Service) and most other AWS services use. | ★★★ | [Security](security.md) |

## L

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS Lake Formation](analytics.md#aws-lake-formation) | Builds and governs a data lake on S3, giving you database-style permissions (table, column, row, cell) on top of the Glue Data Catalog. | ★★ | [Analytics](analytics.md) |
| [AWS Lambda](compute.md#aws-lambda) | Run a function in response to an event, without provisioning or managing any servers, and pay only for the milliseconds it runs. | ★★★ | [Compute](compute.md) |
| [Amazon Lex](ai-ml.md#amazon-lex) | Builds **conversational chatbots and voice bots** that understand what the user wants and collect the details. | ★ | [AI & ML](ai-ml.md) |
| [AWS License Manager](management-monitoring.md#aws-license-manager) | Tracks your software licenses (Windows Server, SQL Server, Oracle and so on) across AWS and on-premises, and can block launches that would break license terms. | ★ | [Management & monitoring](management-monitoring.md) |
| [Amazon Lightsail](compute.md#amazon-lightsail) | Simple virtual private servers (VPS) with storage, networking and a fixed monthly price bundled together. | ★ | [Compute](compute.md) |
| [AWS Local Zones](compute.md#aws-local-zones) | AWS infrastructure placed in large metro areas, close to end users, as an extension of a parent Region. | ★ | [Compute](compute.md) |

## M

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Macie](security.md#amazon-macie) | Uses pattern matching and machine learning to find sensitive data, such as personal or financial details, stored in S3. | ★★ | [Security](security.md) |
| [Amazon Managed Grafana](management-monitoring.md#amazon-managed-grafana) | Fully managed Grafana workspaces for building dashboards over CloudWatch, Prometheus, X-Ray, OpenSearch and third-party data sources. | ★ | [Management & monitoring](management-monitoring.md) |
| [Amazon Managed Service for Apache Flink](analytics.md#amazon-managed-service-for-apache-flink) | Runs Apache Flink applications (Java, Python, Scala or SQL) for real-time, stateful processing of streaming data, without managing servers. | ★★ | [Analytics](analytics.md) |
| [Amazon Managed Service for Prometheus](management-monitoring.md#amazon-managed-service-for-prometheus) | A Prometheus-compatible, fully managed place to store and query container metrics with PromQL. | ★ | [Management & monitoring](management-monitoring.md) |
| [AWS Management Console](management-monitoring.md#aws-management-console) | The web interface where you click around to create, view and change AWS resources. | ★ | [Management & monitoring](management-monitoring.md) |
| [Amazon MemoryDB](databases.md#amazon-memorydb) | A durable, Valkey- and Redis OSS-compatible in-memory database that you can use as your primary database, not just a cache. | ★ | [Databases](databases.md) |
| [AWS Migration Hub](migration-transfer.md#aws-migration-hub) | A single console to track the progress of application migrations across tools such as Application Migration Service and DMS. | ★ | [Migration & transfer](migration-transfer.md) |
| [Amazon MQ](integration.md#amazon-mq) | Managed Apache ActiveMQ and RabbitMQ message brokers, so existing apps that use standard messaging protocols can move to AWS without code changes. | ★★ | [Integration](integration.md) |
| [Amazon MSK](analytics.md#amazon-msk) | Amazon Managed Streaming for Apache Kafka: fully managed Kafka clusters (or serverless Kafka) for apps that use the Kafka APIs. | ★★ | [Analytics](analytics.md) |

## N

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Neptune](databases.md#amazon-neptune) | A managed graph database for data where the relationships matter as much as the records. | ★★ | [Databases](databases.md) |
| [AWS Network Firewall](security.md#aws-network-firewall) | A managed, stateful network firewall and intrusion prevention system that inspects traffic flowing into, out of and between your VPCs. | ★★ | [Security](security.md) |

## O

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon OpenSearch Service](databases.md#amazon-opensearch-service) | Managed OpenSearch (the open-source fork of Elasticsearch) for full-text search, log analytics and dashboards. | ★★ | [Databases](databases.md) |
| [AWS Organizations](identity-governance.md#aws-organizations) | Groups many AWS accounts into a tree, with one combined bill and guardrails that apply to whole branches. | ★★★ | [Identity & governance](identity-governance.md) |
| [AWS Outposts](compute.md#aws-outposts) | AWS-owned, AWS-managed racks or servers installed in your own data center, running EC2 and other AWS services with the same APIs. | ★★ | [Compute](compute.md) |

## P

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Personalize](ai-ml.md#amazon-personalize) | Builds real-time **recommendations** ("customers who viewed this also viewed…") from your users' interaction data. | ★ | [AI & ML](ai-ml.md) |
| [Amazon Polly](ai-ml.md#amazon-polly) | Turns **text into lifelike speech** in many voices and languages. | ★ | [AI & ML](ai-ml.md) |
| [AWS Pricing Calculator](cost-management.md#aws-pricing-calculator) | A free web tool for estimating the monthly cost of an architecture before you build it. | ★ | [Cost management](cost-management.md) |
| [AWS PrivateLink](networking.md#aws-privatelink) | Lets a VPC reach one specific service (an AWS service, a SaaS product, or another team's app) through private IPs, without connecting the two networks. | ★★ | [Networking](networking.md) |
| [AWS Proton](management-monitoring.md#aws-proton) | A service where platform teams published versioned environment and service templates that developers deployed containers and serverless apps against. **AWS ended support on Oct 7, 2026.** | ★ | [Management & monitoring](management-monitoring.md) |

## Q

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Quick](analytics.md#amazon-quick) | AWS's serverless business intelligence (BI) service for interactive dashboards, reports and natural-language questions about your data. | ★★ | [Analytics](analytics.md) |

## R

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon RDS](databases.md#amazon-rds) | Amazon Relational Database Service (RDS) runs a standard relational database engine for you, handling installation, patching, backups and failover. | ★★★ | [Databases](databases.md) |
| [Amazon Redshift](databases.md#amazon-redshift) | A fully managed, columnar data warehouse that runs SQL analytics over terabytes to petabytes of data. | ★★★ | [Databases](databases.md) |
| [Amazon Rekognition](ai-ml.md#amazon-rekognition) | Analyzes **images and video** to find objects, scenes, faces, celebrities, text in photos, and unsafe content. | ★ | [AI & ML](ai-ml.md) |
| [AWS Resource Access Manager (AWS RAM)](identity-governance.md#aws-resource-access-manager-aws-ram) | Shares resources you own, such as subnets or a Transit Gateway, with other AWS accounts without copying them. | ★★ | [Identity & governance](identity-governance.md) |
| [Amazon Route 53](networking.md#amazon-route-53) | AWS's highly available DNS service: it registers domains, answers DNS queries, checks endpoint health and can steer users by health, latency, location or weight. | ★★★ | [Networking](networking.md) |

## S

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon S3](storage.md#amazon-s3) | Amazon Simple Storage Service (S3) stores any number of files ("objects") of almost any size and serves them over HTTPS with very high durability. | ★★★ | [Storage](storage.md) |
| [Amazon S3 Glacier](storage.md#amazon-s3-glacier) | The three archive storage classes inside Amazon S3, for data you must keep but rarely read, at a fraction of S3 Standard's price. | ★★★ | [Storage](storage.md) |
| [Amazon SageMaker AI](ai-ml.md#amazon-sagemaker-ai) | A managed platform for data scientists to build, train, tune and deploy their **own** machine learning models. | ★★ | [AI & ML](ai-ml.md) |
| [Savings Plans](cost-management.md#savings-plans) | Promise to spend a fixed amount per hour on compute (or databases) for 1 or 3 years, and AWS discounts that usage heavily. | ★★★ | [Cost management](cost-management.md) |
| [AWS Secrets Manager](security.md#aws-secrets-manager) | Stores database passwords, API keys and other secrets, encrypted, and can rotate them automatically. | ★★★ | [Security](security.md) |
| [AWS Security Hub](security.md#aws-security-hub) | One place to see security findings from all your accounts and services, plus automated checks against security best practices. | ★★ | [Security](security.md) |
| [AWS Serverless Application Repository](compute.md#aws-serverless-application-repository) | A catalog where you can find, share and deploy ready-made serverless applications built from Lambda functions and related resources. | ★ | [Compute](compute.md) |
| [AWS Service Catalog](management-monitoring.md#aws-service-catalog) | A curated "app store" where admins publish approved infrastructure templates and teams launch them self-service. | ★★ | [Management & monitoring](management-monitoring.md) |
| [AWS Shield](security.md#aws-shield) | Protection against DDoS (distributed denial of service) attacks: Shield Standard is free and always on; Shield Advanced is a paid upgrade. | ★★ | [Security](security.md) |
| [AWS Site-to-Site VPN](networking.md#aws-site-to-site-vpn) | Encrypted IPsec tunnels over the public internet between your office or data center and your VPCs. | ★★ | [Networking](networking.md) |
| [AWS Snow Family](migration-transfer.md#aws-snow-family) | Rugged, encrypted storage-and-compute devices that AWS ships to you, so you can load huge amounts of data locally and ship it back to be imported into S3, or run compute where there's no network. | ★★★ | [Migration & transfer](migration-transfer.md) |
| [Amazon SNS](integration.md#amazon-sns) | A managed publish/subscribe service: publish a message to a topic once, and SNS pushes a copy to every subscriber. | ★★★ | [Integration](integration.md) |
| [Amazon SQS](integration.md#amazon-sqs) | A fully managed message queue that lets one part of your system hand work to another without waiting. | ★★★ | [Integration](integration.md) |
| [AWS Step Functions](integration.md#aws-step-functions) | A serverless workflow engine that runs a multi-step process as a visual state machine, with retries, branching and waiting built in. | ★★ | [Integration](integration.md) |
| [AWS Storage Gateway](storage.md#aws-storage-gateway) | A virtual appliance you run on-premises that lets local servers use AWS storage through ordinary file, block or tape protocols. | ★★★ | [Storage](storage.md) |
| [AWS STS](identity-governance.md#aws-sts) | STS (Security Token Service) hands out temporary credentials when someone assumes an IAM role. | ★★ | [Identity & governance](identity-governance.md) |
| [AWS Systems Manager](management-monitoring.md#aws-systems-manager) | A toolbox for operating fleets of servers (EC2 and on-premises): shell access without SSH, patching, running commands, automation runbooks, and a parameter store. | ★★★ | [Management & monitoring](management-monitoring.md) |

## T

| Service | What it is | Exam | Category |
|---|---|---|---|
| [Amazon Textract](ai-ml.md#amazon-textract) | Reads scanned documents and PDFs and extracts not just the text, but **form fields and tables** with their structure. | ★ | [AI & ML](ai-ml.md) |
| [Amazon Timestream](databases.md#amazon-timestream) | A managed time-series database for data points stamped with a time, such as sensor readings and metrics. | ★ | [Databases](databases.md) |
| [Amazon Transcribe](ai-ml.md#amazon-transcribe) | Converts **speech to text**, from recordings or live audio streams. | ★ | [AI & ML](ai-ml.md) |
| [AWS Transfer Family](migration-transfer.md#aws-transfer-family) | Fully managed SFTP, FTPS, FTP and AS2 servers whose files land directly in Amazon S3 or Amazon EFS. | ★★ | [Migration & transfer](migration-transfer.md) |
| [AWS Transit Gateway](networking.md#aws-transit-gateway) | A managed regional hub that connects many VPCs, VPNs and Direct Connect links, with transitive routing between them. | ★★ | [Networking](networking.md) |
| [Amazon Translate](ai-ml.md#amazon-translate) | Neural machine translation of text between languages. | ★ | [AI & ML](ai-ml.md) |
| [AWS Trusted Advisor](management-monitoring.md#aws-trusted-advisor) | Runs automated checks on your account and flags waste, security gaps, resilience risks and service limits you're about to hit. | ★★ | [Management & monitoring](management-monitoring.md) |

## V

| Service | What it is | Exam | Category |
|---|---|---|---|
| [VMware Cloud on AWS](compute.md#vmware-cloud-on-aws) | VMware's full virtualization stack (vSphere, vSAN, NSX) running on dedicated bare-metal hosts inside AWS Regions. | ★ | [Compute](compute.md) |
| [Amazon VPC](networking.md#amazon-vpc) | A logically isolated private network that you define inside one AWS Region, where your servers, containers, databases and load balancers get their IP addresses. | ★★★ | [Networking](networking.md) |

## W

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS WAF](security.md#aws-waf) | A web application firewall (WAF) that inspects HTTP(S) requests and blocks malicious ones before they reach your app. | ★★★ | [Security](security.md) |
| [AWS Wavelength](compute.md#aws-wavelength) | AWS compute and storage embedded inside telecom providers' 5G networks, for ultra-low latency to mobile devices. | ★ | [Compute](compute.md) |
| [AWS Well-Architected Tool](management-monitoring.md#aws-well-architected-tool) | A free console questionnaire that reviews one of your workloads against AWS's six Well-Architected pillars and produces an improvement plan. | ★ | [Management & monitoring](management-monitoring.md) |

## X

| Service | What it is | Exam | Category |
|---|---|---|---|
| [AWS X-Ray](management-monitoring.md#aws-x-ray) | Distributed tracing that follows a single request through API Gateway, Lambda, containers and databases, and shows where the time and errors are. | ★★ | [Management & monitoring](management-monitoring.md) |

**Next:** [Identity & governance services](identity-governance.md)
