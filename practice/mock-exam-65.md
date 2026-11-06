# Full mock exam: 65 questions (SAA-C03 style)
> [Home](../README.md) · [Practice index](README.md) · [Exam strategy](../01-exam-strategy.md)

## Instructions
- Timebox **130 minutes**, in one sitting, with no notes. Use a paper sheet to mark answers and flag questions.
- Domain mix matches the exam: D1 20 · D2 17 · D3 15 · D4 13.
- The real exam has 15 unscored questions that you can't identify; here every question counts.
- Pass target: **≥ 80% (52/65)** before booking the real exam (the real passing scaled score of 720 isn't a simple percentage).
- 12 questions are multiple response ("Choose two" or "Choose three"). Mark one of those correct only if you picked exactly the right set.
- These questions are harder and more integrated than the per-domain drills. Most hinge on a qualifier such as **MOST cost-effective**, **LEAST operational overhead**, or **minimal changes**, so read the last sentence of the stem first.
- Take this mock once, under exam conditions. A second attempt mostly measures memory, so use the domain drill sets for re-testing.

---

## Questions

### Q1
A company uses AWS Organizations. The central platform team wants application team leads in each workload account to create IAM roles for their own Lambda functions and ECS tasks without filing tickets. The security team requires that no role a team lead creates can ever have more permissions than a platform-defined set (for example, access to the team's own DynamoDB tables and S3 prefixes), and team leads must not be able to escalate their own privileges. Which solution meets these requirements with the LEAST ongoing operational overhead?

- A. Attach an SCP to the workload OUs that denies all IAM actions except `iam:CreateRole`, `iam:AttachRolePolicy`, and `iam:PassRole`.
- B. Allow team leads to create roles freely, and run IAM Access Analyzer weekly to flag and delete any roles that grant excessive permissions.
- C. Allow team leads to create and modify roles only when the `iam:PermissionsBoundary` condition key matches a platform-managed boundary policy, and deny changes to or removal of that boundary.
- D. Give team leads an IAM Identity Center permission set with the `AdministratorAccess` managed policy, assigned only to their own workload account.

### Q2
An e-commerce company processes order lifecycle events (created, paid, shipped, cancelled) from an Amazon SQS standard queue. Consumers occasionally process the same event twice, and sometimes process "shipped" before "paid" for the same order, which corrupts order state. Events for different orders are independent, and the company must keep processing thousands of orders in parallel. Which solution meets these requirements with the LEAST development effort?

- A. Keep the standard queue, record processed message IDs in a DynamoDB table, and have consumers sort each received batch by event timestamp before processing it.
- B. Move to an SQS FIFO queue, set the message group ID to the order ID, and enable content-based deduplication (or have producers supply a deduplication ID).
- C. Move to an SQS FIFO queue and use one fixed message group ID for all events so that ordering is guaranteed across the whole queue.
- D. Publish the events to an Amazon SNS standard topic that fans out to a separate SQS standard queue for each consumer service.

### Q3
A retailer stores 20 TB of curated, structured sales data in Amazon S3 as Parquet. About 300 business users open Amazon Quick Sight dashboards throughout the day, and each refresh runs complex, join-heavy SQL across several large fact and dimension tables. The team currently uses Amazon Athena, but dashboard latency is inconsistent during peak hours and the query volume keeps growing. Which solution will provide the MOST consistent performance for these frequent, high-concurrency queries?

- A. Load the curated tables into Amazon Redshift, choose distribution and sort keys for the main joins, and connect the dashboards to Redshift.
- B. Keep Athena, and convert the Parquet files to compressed CSV so that each dashboard query reads fewer bytes from S3.
- C. Migrate the data to Amazon RDS for PostgreSQL, and add read replicas to serve the dashboard queries during peak hours.
- D. Launch a transient Amazon EMR cluster with Spark SQL for each dashboard refresh, and terminate it after the queries finish.

### Q4
A company runs a data platform in AWS that produces about 4 TB of processed results every day. The results are sent to the on-premises data center over an AWS Site-to-Site VPN on the internet, and this pattern will continue for at least 3 more years. Data transfer out to the internet is now one of the largest lines on the bill, and VPN throughput varies during the day. Which solution is the MOST cost-effective over the long term?

- A. Add more VPN tunnels on an AWS Transit Gateway and enable equal-cost multi-path (ECMP) routing to spread the traffic across them.
- B. Copy each day's results to AWS Snowball Edge devices and ship them to the data center.
- C. Enable S3 Transfer Acceleration on the results bucket, and download the files through the accelerated endpoint.
- D. Provision an AWS Direct Connect connection, and route the results traffic over a private or transit virtual interface.

### Q5
A company uses Amazon GuardDuty in all accounts. When GuardDuty reports that an EC2 instance is querying a domain associated with cryptocurrency mining, the security team wants the instance isolated from the network within minutes, its EBS volumes snapshotted for forensics, and the on-call engineer notified. The response must not depend on someone reviewing findings manually. Which solution meets these requirements with the LEAST operational overhead?

- A. Export GuardDuty findings to Amazon S3, run a scheduled Athena query each night to find the affected instances, and have a script isolate them and email the team.
- B. Create an Amazon EventBridge rule for that GuardDuty finding type that invokes a Lambda function to swap the instance's security groups for an isolation group, snapshot its volumes, and notify through Amazon SNS.
- C. Enable Amazon Inspector in every account, and configure it to stop any instance that has a critical finding and to notify the team through Amazon SNS.
- D. Enable the GuardDuty option that blocks DNS queries to known mining domains in the affected VPCs, and have it send an alert to the team.

### Q6
A media company runs video-transcoding workers on EC2 instances that poll an Amazon SQS standard queue. Each job takes 8–12 minutes. The queue uses the default visibility timeout, and the team sees the same job being transcoded by several workers at once. A few corrupt source files fail every time and keep returning to the queue, consuming worker capacity indefinitely. Which combination of actions will resolve both problems? (Choose two.)

- A. Reduce the queue's message retention period to 1 minute so that messages that keep failing expire quickly.
- B. Set the visibility timeout above the maximum job duration (for example, 15 minutes), or have workers extend it with `ChangeMessageVisibility` while a job runs.
- C. Convert the queue to an SQS FIFO queue so that each message is delivered and processed exactly once.
- D. Enable long polling by setting the queue's receive message wait time to 20 seconds.
- E. Configure a dead-letter queue with a redrive policy that moves a message after a small `maxReceiveCount`, such as 3.

### Q7
A data team in Account A runs an analytics job under an IAM role. The job must read objects from an S3 bucket in Account B. All objects in the bucket are encrypted with SSE-KMS using a customer managed key that Account B owns. The job currently receives `AccessDenied` errors. Which combination of steps should the company take to grant access while following the principle of least privilege? (Choose three.)

- A. In Account B, add a bucket policy statement that allows the Account A role to perform `s3:GetObject` (and `s3:ListBucket` if needed) on the bucket.
- B. In Account B, change the bucket's Object Ownership setting to "Object writer" so that principals in Account A can read the objects.
- C. In Account B, update the KMS key policy to allow the Account A role (or Account A) to use `kms:Decrypt` with the key.
- D. In Account B, turn off S3 Block Public Access for the bucket so that principals outside the account can reach it.
- E. In Account B, change the bucket's default encryption to SSE-S3 so that the KMS key is no longer needed to read the objects.
- F. In Account A, attach an identity-based policy to the role that allows `s3:GetObject` on the bucket and `kms:Decrypt` on the Account B key ARN.

### Q8
A payments company wants to detect fraud rings during card authorization. It must find, in milliseconds, whether a new account shares a device, phone number, shipping address, or payment card with known fraudulent accounts up to four or five hops away. The current relational design needs many recursive self-joins and times out. Which database should a solutions architect recommend?

- A. Amazon DynamoDB, with a global secondary index on each shared attribute
- B. Amazon Redshift, with materialized views refreshed every few minutes
- C. Amazon Neptune, queried with Gremlin or openCypher traversals
- D. Amazon OpenSearch Service, with a nested document for each account

### Q9
A company runs its customer portal on Amazon Aurora MySQL in us-east-1. Leadership requires a disaster recovery plan for a full Regional outage with an RPO of about 1 second and an RTO of a few minutes. The company also wants users in Europe to get low-latency reads from eu-west-1 during normal operation. Which solution meets these requirements with the LEAST operational overhead?

- A. Convert the cluster to an Aurora global database with a secondary cluster in eu-west-1, and use global database failover to promote eu-west-1 if us-east-1 fails.
- B. Create an Aurora MySQL cross-Region read replica that uses binary log replication, and promote it manually during an outage.
- C. Add Aurora Replicas in three Availability Zones in us-east-1, and set failover priorities so the best replica is promoted automatically.
- D. Use AWS Backup to copy hourly cluster snapshots to eu-west-1, and restore a new cluster from the latest copy during an outage.

### Q10
A company runs two workloads on Amazon ECS with AWS Fargate. A customer-facing API runs 24/7 at a steady baseline of about 40 tasks. A nightly image-reprocessing job runs about 200 tasks for 3 hours; the job is idempotent, checkpoints its progress to S3, and can be retried at any time. Which combination of actions will reduce cost the MOST without affecting the API's availability? (Choose two.)

- A. Run the reprocessing tasks with a capacity provider strategy that uses `FARGATE_SPOT`.
- B. Purchase an EC2 Instance Savings Plan that covers the API's baseline task usage.
- C. Run all API tasks on `FARGATE_SPOT`, and rely on ECS to replace any interrupted tasks.
- D. Purchase a Compute Savings Plan sized to the API's steady baseline usage.
- E. Purchase 1-year Fargate Reserved Instances for the API tasks.

### Q11
A payment processor is moving its card-tokenization service to AWS. Its regulator requires that cryptographic keys be generated and stored in single-tenant hardware security modules that are validated at FIPS 140-3 Level 3 and managed only by the company, with no AWS access to the key material. The existing Java application performs cryptographic operations through the PKCS #11 and JCE interfaces, and the company does not want to rewrite it. Which solution meets these requirements?

- A. Create AWS KMS customer managed keys with imported key material, and change the application to call the KMS `Encrypt` and `Decrypt` APIs.
- B. Create AWS KMS multi-Region keys with automatic rotation, and grant the application's IAM role permission to use them.
- C. Store the keys as secrets in AWS Secrets Manager, encrypted with a customer managed key, and rotate them every 30 days.
- D. Deploy an AWS CloudHSM cluster with HSMs in at least two Availability Zones, and use the CloudHSM client SDK's PKCS #11 and JCE providers.

### Q12
A company's on-premises Linux applications write images and documents to an NFS share on a NAS that is almost full. The company wants new files stored as objects in Amazon S3 so that it can apply S3 Lifecycle policies and query the data with Athena. The applications must keep using NFS without code changes, and users need low-latency access to recently written files. Which solution meets these requirements?

- A. Deploy a Volume Gateway in cached mode, and mount its iSCSI volumes on the application servers.
- B. Deploy an S3 File Gateway on premises, create an NFS file share backed by the S3 bucket, and mount it on the servers.
- C. Deploy Amazon FSx for Windows File Server, and use AWS DataSync to copy its files to S3 every night.
- D. Create an Amazon EFS file system, and mount it on the on-premises servers over AWS Direct Connect.

### Q13
A gaming company runs a multiplayer backend that uses a custom TCP protocol on port 7777. The backend runs behind Network Load Balancers in us-west-2 and eu-central-1. The game client connects to fixed IP addresses that are compiled into each release, so the company cannot depend on DNS changes. If one Region's endpoints become unhealthy, players must be routed to the other Region automatically and quickly, and players worldwide should get lower latency and jitter. Which solution meets these requirements?

- A. Create Amazon Route 53 failover records with health checks and a 30-second TTL that point to the two NLBs.
- B. Create an Amazon CloudFront distribution with an origin group that contains the two NLBs.
- C. Create an AWS Global Accelerator standard accelerator with a TCP listener on port 7777 and an endpoint group for each Region's NLB.
- D. Assign Elastic IP addresses to both NLBs, and use a Lambda function to move the addresses to the healthy Region during an outage.

### Q14
A company sends all outbound internet traffic from its workload VPCs through AWS Transit Gateway to a central egress VPC that contains NAT gateways. A new compliance rule says workloads may make outbound HTTPS connections only to an approved list of about 50 domains (such as package repositories and SaaS APIs), and all other outbound traffic must be blocked and logged. The company wants a managed service and no proxy servers on EC2. Which solution meets these requirements?

- A. Deploy AWS Network Firewall in the egress VPC with a stateful domain-list rule group that allows the approved domains based on TLS SNI and HTTP Host headers and drops everything else.
- B. Update the security groups in every workload VPC with outbound rules that reference the approved domain names, and remove the default allow-all outbound rule.
- C. Add network ACL rules to the workload subnets that allow outbound traffic only to the IP addresses that the approved domains resolve to today.
- D. Create an AWS WAF web ACL that allows only the approved domains, and associate the web ACL with the NAT gateways in the egress VPC.

### Q15
A hospital network keeps 500 TB of medical images in Amazon S3 Standard. After the first 30 days, a radiologist opens a given study about once per quarter, but when a study is requested it must be displayed within milliseconds. Images must be retained for 7 years and stored redundantly across multiple Availability Zones. Which storage strategy is the MOST cost-effective?

- A. Use an S3 Lifecycle rule to transition the objects to S3 Standard-IA after 30 days.
- B. Use an S3 Lifecycle rule to transition the objects to S3 Glacier Flexible Retrieval after 30 days, and use expedited retrievals when a study is needed.
- C. Use an S3 Lifecycle rule to transition the objects to S3 One Zone-IA after 30 days.
- D. Use an S3 Lifecycle rule to transition the objects to S3 Glacier Instant Retrieval after 30 days.

### Q16
A monolithic web application runs on EC2 instances in an Auto Scaling group behind an Application Load Balancer. User sessions are stored in each instance's memory, and the ALB uses sticky sessions. Users are logged out whenever an instance is terminated by scale-in or fails a health check. The company wants the instances to be stateless so that they can scale freely, and session reads must stay at sub-millisecond latency. Which solution meets these requirements with the fewest changes to the application?

- A. Increase the ALB stickiness duration to 7 days, and enable connection draining (deregistration delay) on the target group.
- B. Store sessions in an Amazon ElastiCache (Valkey or Redis OSS) replication group with Multi-AZ enabled, and turn off stickiness.
- C. Store session files on an EBS io2 volume with Multi-Attach enabled, and attach the volume to every instance in the group.
- D. Store sessions in a table in an Amazon RDS for MySQL Multi-AZ DB instance, and turn off stickiness.

### Q17
A company uses AWS Organizations. The networking team manages VPCs, subnets, route tables, NAT gateways, and Transit Gateway attachments in a central network account. Application teams in a dozen separate accounts must launch EC2 instances, RDS databases, and load balancers into centrally managed subnets, while the networking team keeps sole control of routing and network ACLs. The company wants to avoid creating and peering a VPC in every application account. Which solution meets these requirements?

- A. Use AWS Resource Access Manager (AWS RAM) to share specific subnets from the network account's VPCs with the application accounts in the organization.
- B. Create a VPC in each application account, and connect each one to a central VPC with VPC peering connections that the networking team manages.
- C. Create IAM roles in the network account that the application teams assume to launch their resources directly in the network account.
- D. Publish the VPC as a product in AWS Service Catalog, and share the portfolio with each of the application accounts.

### Q18
A company runs image-processing workers in an EC2 Auto Scaling group. The workers pull jobs from an Amazon SQS queue. Each instance processes about 20 jobs per minute, and the business wants no job to wait more than 5 minutes, so a backlog of about 100 jobs per instance is acceptable. The group currently scales on average CPU utilization, which reacts too slowly to traffic spikes and keeps idle instances running during quiet periods. Which combination of steps should a solutions architect take? (Choose two.)

- A. Add scheduled scaling actions that increase capacity every weekday at 9 AM and reduce it at 6 PM.
- B. Define a backlog-per-instance metric that divides `ApproximateNumberOfMessagesVisible` by the number of InService instances, as a custom CloudWatch metric or with metric math.
- C. Create a target tracking scaling policy that keeps the backlog-per-instance metric at a target value of 100.
- D. Create a target tracking scaling policy that keeps the queue's `ApproximateNumberOfMessagesVisible` metric at a target value of 100.
- E. Create a step scaling policy that adds one instance whenever average CPU utilization stays above 70% for 5 minutes.

### Q19
A company has 60 AWS accounts in AWS Organizations and adds new accounts every month. Auditors require that API activity from every account and every Region, including future accounts, be delivered to a dedicated log-archive account. The log files must be immutable for 7 years so that no one, including the root user of the log-archive account, can delete or overwrite them. Which combination of steps meets these requirements with the LEAST operational overhead? (Choose two.)

- A. Create an organization trail in AWS CloudTrail from the management account (or a delegated administrator account) that logs all Regions and delivers to an S3 bucket in the log-archive account.
- B. Use CloudFormation StackSets to create a multi-Region trail in each member account that delivers to the log-archive bucket.
- C. Enable S3 Versioning and MFA delete on the log-archive bucket.
- D. Enable S3 Object Lock on the log-archive bucket in governance mode with a 7-year default retention period.
- E. Enable S3 Object Lock on the log-archive bucket in compliance mode with a 7-year default retention period.

### Q20
An online retailer runs in us-east-1 and wants a disaster recovery environment in us-west-2 with an RTO of 15 minutes and an RPO of under 1 minute. The database is Amazon Aurora. Even when launched from a prebuilt AMI, a new application instance needs about 30 minutes to load reference data and warm its caches before it can serve traffic. The company wants the lowest-cost design that still meets the RTO and RPO. Which design should a solutions architect recommend?

- A. Backup and restore: copy AWS Backup recovery points to us-west-2, and rebuild the environment from them during a disaster.
- B. Pilot light: replicate the database to us-west-2 with an Aurora global database, keep the application Auto Scaling group at zero instances, and scale it out during a disaster.
- C. Warm standby: replicate the database with an Aurora global database, run a small, always-warm application fleet in us-west-2, use Route 53 failover routing, and scale the fleet out during a disaster.
- D. Multi-site active-active: run a full-size application fleet in both Regions, and migrate the database to DynamoDB global tables.

### Q21
A company has 45 VPCs across six accounts in one AWS Region, plus two on-premises sites connected with Site-to-Site VPN. Almost every VPC needs to reach most other VPCs and both sites. A full-mesh VPC peering design would need about 990 connections, and route table changes have already caused several outages. Two analytics VPCs exchange tens of terabytes every month with each other. Which solution meets the connectivity needs with the LEAST operational overhead while keeping data transfer cost under control?

- A. Keep the full-mesh VPC peering design, and use a Lambda function to update the route tables automatically whenever a peering connection is added.
- B. Build a transit VPC with third-party router appliances on EC2 instances, and connect every VPC and both VPN sites to it.
- C. Create AWS PrivateLink endpoint services in every VPC, and create interface endpoints in each consumer VPC that needs to reach them.
- D. Create a transit gateway, share it with the other accounts by using AWS RAM, attach all VPCs and both VPNs, and keep a direct VPC peering connection between the two analytics VPCs.

### Q22
A company runs 300 microservices that produce about 2 TB of application logs per day. Engineers need full-text search across log messages with sub-second responses for the most recent 14 days, plus interactive dashboards of error trends. Logs older than 14 days are queried a few times a month but must be kept for 1 year at low cost. Which solution BEST meets these requirements?

- A. Send all logs to Amazon CloudWatch Logs with 1-year retention, and use CloudWatch Logs Insights for both the searches and the dashboards.
- B. Stream the logs with Amazon Data Firehose into Amazon OpenSearch Service, use OpenSearch Dashboards, and move indexes older than 14 days to UltraWarm or cold storage.
- C. Write the logs to Amazon S3 in JSON format, and run Amazon Athena queries to serve both the searches and the dashboards.
- D. Store each log line as an item in Amazon DynamoDB, with a global secondary index on log level and timestamp.

### Q23
An application needs about 40 configuration values (feature flags, endpoint URLs, and tuning parameters) and one Amazon RDS database password. Security policy requires the password to be rotated automatically every 30 days without downtime. The configuration values are not sensitive and change rarely. Which solution meets these requirements MOST cost-effectively?

- A. Store the configuration values as standard parameters in AWS Systems Manager Parameter Store, and store the password in AWS Secrets Manager with managed rotation turned on.
- B. Store all 41 values as secrets in AWS Secrets Manager, and turn on rotation only for the database password secret.
- C. Store all values, including the password, as SecureString parameters in Parameter Store, and rotate the password with a custom Lambda function on an EventBridge schedule.
- D. Store the configuration values in the EC2 user data, and store the password in an S3 object encrypted with SSE-KMS that the application reads at startup.

### Q24
A company is migrating an on-premises Java order system to AWS. The system's producers and consumers use JMS with Apache ActiveMQ over the OpenWire and STOMP protocols. The company wants to avoid code changes, stop patching and operating brokers itself, and keep the messaging layer available if an Availability Zone fails. Which solution meets these requirements?

- A. Rewrite the producers and consumers to use Amazon SQS queues and Amazon SNS topics instead of the ActiveMQ broker.
- B. Run ActiveMQ on EC2 instances in an Auto Scaling group across two Availability Zones, with an Amazon EFS file system for message persistence.
- C. Deploy an Amazon MQ for ActiveMQ broker in an active/standby deployment across two Availability Zones, and update the clients' connection URLs.
- D. Deploy Amazon MSK, and use Kafka Connect to bridge the existing JMS clients to Kafka topics.

### Q25
A company's security team needs continuous vulnerability scanning of both its EC2 instances and the container images in Amazon ECR across 40 accounts in AWS Organizations. Findings must be re-evaluated automatically when new CVEs are published, without anyone re-running scans, and the results must be visible from one account. Which solution meets these requirements with the LEAST operational overhead?

- A. Enable Amazon GuardDuty Runtime Monitoring in every account, and aggregate its findings in a delegated administrator account.
- B. Enable Amazon Macie in every account, and schedule sensitive data discovery jobs for the ECR repositories and EBS volumes.
- C. Turn on basic scan-on-push for every ECR repository, and run an open-source scanner on each EC2 instance on a schedule through SSM Run Command.
- D. Enable Amazon Inspector from a delegated administrator account for the organization, with EC2 scanning and ECR enhanced (continuous) scanning auto-enabled for member accounts.

### Q26
A research institute must move 200 TB of genomics files from an on-premises NAS to Amazon S3 within 4 weeks. Its only internet connection is 100 Mbps and is shared with other users. The institute already uses AWS Snowball Edge for other projects. While the migration is under way, and until cutover, scientists add about 50 GB of new files per day, and those files must also arrive in S3. Which combination of actions meets these requirements? (Choose two.)

- A. Enable S3 Transfer Acceleration on the bucket, and copy the full dataset with parallel multipart uploads.
- B. Set up an AWS Site-to-Site VPN connection, and copy the full dataset through it to keep the transfer private.
- C. Order AWS Snowball Edge Storage Optimized devices, copy the existing dataset to them, and ship them back for import into S3.
- D. Install an AWS DataSync agent on premises, and run scheduled DataSync tasks that transfer new and changed files to S3.
- E. Deploy a Tape Gateway, and back up the NAS to virtual tapes that are archived in S3 Glacier.

### Q27
A company has 120 development and test EC2 instances and 15 RDS DB instances, all tagged `Environment=dev`. They are used only from 8 AM to 7 PM on weekdays in a single time zone, but they run On-Demand 24/7. The company wants the largest cost reduction with the LEAST operational overhead and no impact on developers during working hours. Which solution should a solutions architect recommend?

- A. Deploy the Instance Scheduler on AWS solution, and configure a tag-based schedule that stops the EC2 and RDS instances outside working hours and starts them each weekday morning.
- B. Purchase 1-year Standard Reserved Instances for all of the development and test EC2 and RDS instances.
- C. Convert the development and test instances to Spot Instances, and keep them running 24/7 so developers are never blocked.
- D. Apply AWS Compute Optimizer recommendations to downsize each development and test instance by at least one size.

### Q28
A media company ingests about 50,000 clickstream events per second. Three independent applications (a real-time dashboard, a fraud model, and an archiver) must each read every event, in order, for a given user session. After a bug fix, a new data science team needs to reprocess the last 3 days of events from the beginning without affecting the other consumers. Which solution meets these requirements?

- A. Send the events to an Amazon SQS standard queue, and have each application read from the queue with its own consumer group.
- B. Send the events to Amazon Kinesis Data Streams in on-demand mode with the session ID as the partition key, set retention to 7 days, and use enhanced fan-out consumers.
- C. Publish the events to an Amazon SNS standard topic that fans out to three Amazon SQS standard queues with 14-day message retention.
- D. Send the events to an Amazon EventBridge custom event bus with an archive, and replay the archive for the new team.

### Q29
For compliance reasons, a workload runs in a VPC that has no internet gateway and no NAT gateway. An application on EC2 instances in private subnets must send messages to Amazon SQS and call AWS KMS to decrypt data keys. Traffic must stay on the AWS network, and the developers do not want to change the SDK endpoint configuration in the application code. Which combination of actions meets these requirements? (Choose two.)

- A. Create a gateway VPC endpoint for Amazon SQS, and add it to the private subnets' route tables.
- B. Create an interface VPC endpoint for Amazon SQS with private DNS enabled.
- C. Add a NAT gateway, and limit its route to the published AWS IP address ranges for SQS and KMS.
- D. Create an interface VPC endpoint for AWS KMS with private DNS enabled.
- E. Create an AWS Direct Connect public virtual interface so that the VPC can reach the public SQS and KMS endpoints.

### Q30
An IoT company runs a self-managed Apache Cassandra cluster on 30 EC2 instances to store device telemetry. The team spends significant time on patching, node replacement, repairs, and capacity planning. The applications use Cassandra Query Language (CQL) drivers, and the company wants a serverless database with minimal application changes. Which solution meets these requirements?

- A. Migrate to Amazon DynamoDB, and rewrite the data access layer to use the AWS SDK.
- B. Migrate to Amazon DocumentDB, and store each telemetry reading as a JSON document.
- C. Migrate to Amazon RDS for PostgreSQL, and use time-based table partitioning for the telemetry.
- D. Migrate to Amazon Keyspaces (for Apache Cassandra), and point the existing CQL drivers at its endpoint.

### Q31
A company runs an API on EC2 instances in an Auto Scaling group behind an Application Load Balancer. For each release, the company wants to deploy the new version on new instances, send 10% of production traffic to it, watch error rates for an hour, and then move all traffic, with the ability to roll back in seconds. Many API clients cache DNS records for long periods, so the solution must not rely on DNS changes. Which solution meets these requirements?

- A. Create a second ALB for the new version, and use Amazon Route 53 weighted records to send 10% of the traffic to it.
- B. Use an Auto Scaling group instance refresh with a minimum healthy percentage of 90% to replace instances with the new launch template version.
- C. Create a new Auto Scaling group and target group for the new version, and use weighted target groups in the ALB listener rule to shift traffic from 90/10 to 0/100.
- D. Deploy the new version to a separate AWS Elastic Beanstalk environment, and swap the environment URLs after testing.

### Q32
A company runs its production database on Amazon RDS for PostgreSQL, using a memory-optimized instance in a Multi-AZ deployment. The database has run 24/7 at steady utilization for 2 years and must remain highly available. The company expects no change to the engine, instance family, or Region for at least 3 more years, and finance wants the largest possible discount. Which option should the company choose?

- A. Purchase a 3-year Compute Savings Plan that covers the database's hourly spend.
- B. Purchase 3-year RDS Reserved Instances that match the engine, instance family, Region, and Multi-AZ deployment, and pay all upfront.
- C. Migrate the database to Aurora Serverless v2 so that the company pays only for the capacity that it uses.
- D. Convert the deployment to Single-AZ, and rely on automated backups and point-in-time recovery to handle instance failures.

### Q33
A company has 25 AWS accounts. Developers sometimes add inbound rules that allow SSH (port 22) from 0.0.0.0/0 to security groups. The security team wants such rules detected and removed automatically within minutes, wants a compliance history for auditors, and wants to write as little custom code as possible. Which solution meets these requirements?

- A. Deploy the AWS Config managed rule `restricted-ssh` across the organization, and configure automatic remediation with an AWS Systems Manager Automation runbook that removes the offending rule.
- B. Create an EventBridge rule for CloudTrail `AuthorizeSecurityGroupIngress` events, and write a Lambda function that parses each event and revokes any rule that allows 0.0.0.0/0.
- C. Enable Amazon GuardDuty in all accounts, and configure it to remove security group rules that expose SSH to the internet.
- D. Attach an SCP to all accounts that denies `ec2:AuthorizeSecurityGroupIngress` for every principal except the security team's role.

### Q34
An ad-tech company runs a real-time bidding service that must handle millions of requests per second over TCP, absorb sudden traffic spikes, and add as little latency as possible. The bidding servers terminate TLS themselves and must see each client's source IP address. Exchange partners allow only a fixed set of IP addresses through their firewalls, with one address per Availability Zone. Which load balancing solution should a solutions architect choose?

- A. An Application Load Balancer with AWS WAF and cross-zone load balancing enabled
- B. A Gateway Load Balancer that distributes the traffic to the bidding instances
- C. A Network Load Balancer with TCP listeners and an Elastic IP address assigned in each Availability Zone
- D. A Classic Load Balancer with TCP listeners and connection draining enabled

### Q35
A company keeps a disaster recovery copy of its application in us-west-2 and recreates the compute tier from a CloudFormation template during failover. In a recent game day, the stack failed for two reasons: EC2 reported that the account had exceeded its On-Demand vCPU limit in us-west-2, and the launch template referenced an AMI ID that exists only in us-east-1. Which combination of actions will prevent these failures during a real event? (Choose two.)

- A. Use Service Quotas to request increases in us-west-2 ahead of time so that the On-Demand vCPU quotas cover production peak capacity.
- B. Subscribe to AWS Health Dashboard notifications so that the team is alerted about quota issues during a failover.
- C. Have the image pipeline copy each new AMI to us-west-2 (for example, with EC2 Image Builder distribution settings), and resolve the Region-specific AMI ID from an SSM parameter.
- D. Change the DR Auto Scaling group to use Spot Instances, which do not count against the On-Demand vCPU quota.
- E. Reference the us-east-1 AMI ID in the us-west-2 template by using a CloudFormation cross-stack export.

### Q36
An e-learning company streams course videos through Amazon CloudFront using HLS. Each video consists of a manifest and hundreds of small segment files under the same path. Only logged-in, paying subscribers may watch, and access must expire after 12 hours. The third-party video player cannot be modified to add query strings to each segment request. Which solution meets these requirements?

- A. Generate a CloudFront signed URL for every segment file, and embed the signed URLs in a rewritten manifest for each viewer.
- B. Generate S3 presigned URLs for the manifest and segments, and let the player download the files from S3 directly.
- C. Add an AWS WAF rule to the distribution that allows requests only when the Referer header matches the company's website.
- D. After login, issue CloudFront signed cookies with a custom policy that covers the course's path with a wildcard and expires after 12 hours.

### Q37
A company with 30 accounts in AWS Organizations needs a monthly chargeback for each team and application. Finance wants hourly, resource-level data, including amortized Reserved Instance and Savings Plans costs, and wants to run ad hoc SQL queries against it. All resources carry a `CostCenter` tag. Which combination of actions meets these requirements? (Choose two.)

- A. Use the default AWS Cost Explorer view, and filter each month's costs by linked account.
- B. Create an AWS Budgets budget for each team that sends an alert when its spending exceeds its forecast.
- C. Review the AWS Trusted Advisor cost optimization checks each month, and export the results to a spreadsheet.
- D. Activate `CostCenter` as a user-defined cost allocation tag in the management account.
- E. Create a CUR 2.0 export with AWS Data Exports that delivers Parquet files with resource IDs to S3, and query the export with Amazon Athena.

### Q38
A company has a data lake in Amazon S3 that is cataloged in the AWS Glue Data Catalog and queried with Amazon Athena. Marketing analysts must see only the non-PII columns of the `customers` table, and only the rows for their own country. Finance analysts must see all columns and rows. The same tables must also be shared with a subsidiary's AWS account. The company wants to manage these permissions centrally rather than through many bucket policies and copies of the data. Which solution meets these requirements?

- A. Split the data into separate S3 prefixes for each country and column set, and grant access to each prefix with bucket policies and IAM policies.
- B. Register the data lake locations with AWS Lake Formation, grant table permissions with data filters for column-level and row-level security, and share the tables with the subsidiary through Lake Formation.
- C. Use Amazon Macie to discover the PII columns, and configure it to mask those columns automatically in the marketing analysts' query results.
- D. Create a separate Athena workgroup for each team, and use IAM policy conditions on the workgroups to limit which columns each team can query.

### Q39
An order API uses API Gateway, Lambda, six microservices on Amazon ECS, DynamoDB, and a third-party payment API. After a release, p99 latency rose from 300 ms to 2 seconds. CloudWatch metrics show that every service is healthy and none is near its CPU or memory limits. The team needs to find which downstream call adds the latency on each request path, with minimal effort. Which solution meets these requirements?

- A. Enable AWS X-Ray active tracing on API Gateway and Lambda, instrument the ECS services with AWS Distro for OpenTelemetry to send traces to X-Ray, and analyze the service map and traces.
- B. Enable VPC Flow Logs on all of the application subnets, and query the logs with Athena to find the slowest connections between the services.
- C. Enable CloudTrail data events for DynamoDB and Lambda, and search the event history for the API calls that took the longest to complete.
- D. Switch all of the services' CloudWatch metrics to 1-second high-resolution metrics, and build a dashboard that compares their latencies.

### Q40
A bank connects its data center to AWS through a 1 Gbps hosted AWS Direct Connect connection from a partner. A new regulation requires all traffic between the data center and its VPCs to be encrypted in transit at the network layer. The bank wants to keep the predictable performance of Direct Connect. Which solution meets these requirements?

- A. Enable MACsec on the hosted connection, and configure a MACsec key (CKN/CAK pair) on the on-premises router.
- B. Make no changes, because Direct Connect is a private circuit and AWS encrypts its traffic by default.
- C. Run an AWS Site-to-Site VPN (IPsec) connection over the Direct Connect connection, such as a private IP VPN to a transit gateway through a transit virtual interface.
- D. Replace the Direct Connect connection with an AWS Client VPN endpoint that the data center's routers connect to.

### Q41
A product catalog application uses MongoDB drivers, flexible JSON documents, secondary indexes, and aggregation pipelines. It runs on a self-managed MongoDB replica set on EC2 that the team struggles to patch, back up, and scale for reads. The company wants a managed AWS database that is compatible with the MongoDB API and can add read replicas quickly, with minimal code changes. Which solution meets these requirements?

- A. Migrate to Amazon DynamoDB, and rewrite the queries and aggregations to use the DynamoDB API.
- B. Migrate to Amazon Keyspaces (for Apache Cassandra), and model each document as a wide row.
- C. Migrate to Amazon Aurora PostgreSQL, and store the documents in JSONB columns.
- D. Migrate to Amazon DocumentDB (with MongoDB compatibility) by using AWS DMS.

### Q42
A development VPC spans three Availability Zones, and each AZ has its own NAT gateway. Instances in the private subnets only occasionally download operating system and package updates, and total traffic is low. The team accepts that the development environment can lose internet access during a rare AZ outage. The company wants to reduce cost while still using a managed service. Which solution meets these requirements?

- A. Remove the NAT gateways, move the instances to public subnets, and assign each instance a public IPv4 address.
- B. Keep one NAT gateway in a single AZ, and point the default route of every private subnet's route table to it.
- C. Replace each NAT gateway with an interface VPC endpoint for the public package repositories.
- D. Keep all three NAT gateways, and purchase a Compute Savings Plan that covers their hourly charges.

### Q43
A company serves static web assets and downloads through Amazon CloudFront from an S3 bucket in us-east-1. If the bucket becomes unavailable or returns server errors, CloudFront must automatically serve the content from a copy in another Region, without DNS changes or manual steps. Which combination of actions meets these requirements? (Choose two.)

- A. Create a second CloudFront distribution for a bucket in us-west-2, and use Amazon Route 53 failover records to switch between the two distributions.
- B. Configure S3 Cross-Region Replication from the us-east-1 bucket to a bucket in us-west-2.
- C. Enable S3 Transfer Acceleration on the us-east-1 bucket so that CloudFront reaches it through the edge network.
- D. Create an AWS Global Accelerator accelerator with the two S3 buckets as endpoints in separate endpoint groups.
- E. Create a CloudFront origin group with the us-east-1 bucket as the primary origin and the us-west-2 bucket as the secondary origin, with failover on 5xx status codes.

### Q44
A company wants its employees to sign in to the AWS access portal through IAM Identity Center with their existing on-premises Active Directory credentials. Security policy says that the company must not run or maintain a separate directory in AWS, and no passwords may be synchronized to or stored in AWS. The data center already connects to AWS over Direct Connect. Which solution meets these requirements MOST cost-effectively?

- A. Deploy AD Connector in a VPC that can reach the on-premises domain controllers, and set it as the identity source for IAM Identity Center.
- B. Deploy AWS Managed Microsoft AD, create a two-way forest trust with the on-premises domain, and set it as the identity source.
- C. Deploy Simple AD, and configure it to synchronize users and groups from the on-premises domain every hour.
- D. Create an Amazon Cognito user pool that federates with the on-premises AD FS server, and use it to sign in to the AWS access portal.

### Q45
A genomics team runs thousands of containerized analysis jobs every night. Each job runs 10–90 minutes, later stages depend on earlier ones, and CPU and memory needs vary by job. The team wants job queuing, automatic retries, dependency handling, and automatic provisioning of right-sized compute that can include Spot capacity, without operating its own scheduler. Which solution meets these requirements with the LEAST operational overhead?

- A. Trigger a Lambda function from S3 events for each stage, and pass intermediate results between the stages through S3.
- B. Build a Slurm cluster on EC2 instances, and manage the head node and an Auto Scaling group of worker nodes.
- C. Use AWS Batch with managed compute environments that mix Spot and On-Demand capacity, job queues, and job dependencies.
- D. Run the jobs as Amazon ECS services, and use scheduled scaling to set the desired task count each night.

### Q46
A company is concerned about ransomware. It wants backups of the EBS volumes, RDS databases, DynamoDB tables, and EFS file systems in all production accounts to be copied out of the production accounts and out of the primary Region. Nobody, including administrators of the production accounts, may delete those copies before their retention period ends, and backup policies must be managed centrally. Which solution meets these requirements?

- A. Create Amazon Data Lifecycle Manager policies in each production account that take EBS snapshots and copy them to a second Region every day.
- B. Turn on RDS automated backup replication to a second Region, and enable DynamoDB point-in-time recovery and EFS automatic backups in each account.
- C. Configure S3 Cross-Region Replication to copy each production account's snapshots to a bucket in a central backup account in a second Region.
- D. Use AWS Backup policies in AWS Organizations with copy rules to vaults in a central backup account and a second Region, and lock the destination vaults with AWS Backup Vault Lock in compliance mode.

### Q47
An audit found that teams in a new account are creating unencrypted EBS volumes, and that an existing production Amazon RDS for MySQL DB instance is unencrypted. The company requires all new EBS volumes to be encrypted automatically without changes to existing launch templates, and requires the production database to be encrypted with a customer managed KMS key. Which combination of actions meets these requirements? (Choose two.)

- A. Enable EBS encryption by default in each Region that the account uses, and set the customer managed key as the default key.
- B. Modify the RDS DB instance to turn on encryption with the customer managed key during the next maintenance window.
- C. Create an encrypted read replica of the unencrypted DB instance, and promote the replica to a standalone DB instance.
- D. Take a snapshot of the DB instance, copy it with encryption enabled using the customer managed key, restore a new DB instance from the encrypted copy, and move the application to it.
- E. Attach an SCP that denies `ec2:CreateVolume` unless the volume is encrypted, and ask the teams to update their launch templates.

### Q48
A content management system stores 40 TB on Amazon EFS in the Standard storage class. Analysis shows that about 85% of files are never read again 30 days after they are written, but any file must still be readable immediately, through the same path, if an editor opens it. The company wants to reduce storage cost without application changes. Which solution meets these requirements?

- A. Run a nightly script that moves files older than 30 days to S3 Glacier Deep Archive and replaces each one with a link.
- B. Enable EFS lifecycle management to move files to the Infrequent Access storage class after 30 days without access, and to move them back to Standard on first access.
- C. Convert the file system to EFS One Zone storage to lower the price of every file in it.
- D. Change the file system's throughput mode from Elastic to Provisioned so that throughput is billed at a fixed rate.

### Q49
A company is migrating a 30 TB on-premises NFS share to Amazon EFS. Until cutover in 2 months, files that change on premises must be copied to EFS every night. Transfers must be encrypted in transit, the copied data must be verified for integrity, and bandwidth use must be limited during business hours. The data center has a 1 Gbps Direct Connect connection. Which solution meets these requirements with the LEAST operational overhead?

- A. Mount the EFS file system on an EC2 instance, and run an `rsync` cron job from an on-premises server to the instance over SSH.
- B. Order an AWS Snowball Edge device for the initial copy, and order a new device for each night's changes.
- C. Deploy an AWS DataSync agent on premises, and create a scheduled DataSync task from the NFS share to EFS with data verification and a bandwidth limit.
- D. Deploy an S3 File Gateway on premises to copy the files to S3, and use a Lambda function to copy each new object from S3 to EFS.

### Q50
A company receives daily files from 200 partners through an on-premises SFTP server, which is a single point of failure. The company wants a highly available, fully managed SFTP endpoint that stores incoming files in Amazon S3 so that each upload can trigger processing. Partners must keep using SFTP with their existing SSH keys and the same host name. Which solution meets these requirements with the LEAST operational overhead?

- A. Create an AWS Transfer Family SFTP server with an S3 bucket as storage, add each partner as a service-managed user with their SSH public key, and point the existing host name at the server.
- B. Run an SFTP server on EC2 instances in an Auto Scaling group across two Availability Zones with a shared EFS file system, and sync the files to S3 every few minutes.
- C. Create an IAM user for each partner, and ask the partners to upload their files directly to the S3 bucket with the AWS CLI instead of SFTP.
- D. Deploy an S3 File Gateway in a VPC, and give the partners access to its file share over the internet.

### Q51
A company has 300 software projects, and new projects are created every week. Developers sign in through IAM Identity Center. Each developer may start, stop, and reboot only the EC2 instances, and read only the Secrets Manager secrets, that belong to their own project. The current approach of one IAM policy per project is reaching policy size limits and requires an update for every new project. Which solution scales with the LEAST administrative effort?

- A. Create one IAM role for each project that lists the ARNs of that project's instances and secrets, and update the role whenever the project's resources change.
- B. Move each project into its own AWS account so that the account boundary isolates its instances and secrets from other projects.
- C. Attach a resource-based policy to each EC2 instance and each secret that allows access only to the developers of the owning project.
- D. Pass a `Project` attribute from the identity provider as a session tag, and use one permission set whose conditions require `aws:ResourceTag/Project` to equal `${aws:PrincipalTag/Project}`.

### Q52
A reporting application on EC2 instances in Availability Zone B runs large queries against an Amazon RDS for MySQL read replica in Availability Zone A. The queries transfer about 40 TB per month, and inter-AZ data transfer has become a significant cost. The production primary must remain a Multi-AZ deployment. Which change is the MOST cost-effective way to reduce this charge?

- A. Enable Multi-AZ on the read replica so that a standby copy of the replica also exists in Availability Zone B.
- B. Create a new read replica in Availability Zone B, point the reporting application at it, and delete the replica in Availability Zone A.
- C. Route the reporting traffic through a NAT gateway in Availability Zone A so that it is processed in the same AZ as the replica.
- D. Attach both subnets to a transit gateway, and send the reporting traffic through the transit gateway.

### Q53
A company runs an in-memory analytics engine that loads a 900 GB dataset into RAM on each node. The workload uses moderate CPU but runs on compute-optimized instances, where it constantly pages to disk and queries are slow. Which instance choice will improve performance the MOST?

- A. Move to the largest instance size in the same compute-optimized family.
- B. Move to storage-optimized instances, and use their local NVMe drives as swap space.
- C. Move to memory-optimized instances (the R family, or the X family for the largest memory footprints) with enough RAM to hold the dataset.
- D. Move to burstable instances in unlimited mode so that the workload can use more CPU credits.

### Q54
A company has 400 remote employees who need secure access from their laptops to internal web applications and RDS databases in private subnets of a VPC. Employees must authenticate with the company's SAML 2.0 identity provider, including MFA. The company wants a managed, scalable solution and no bastion hosts. Which solution meets these requirements?

- A. Create an AWS Client VPN endpoint that uses SAML-based federated authentication, associate it with the VPC's subnets, and add authorization rules for the internal networks.
- B. Create an AWS Site-to-Site VPN connection to each employee's home router, and attach all of the connections to a virtual private gateway.
- C. Launch bastion hosts in public subnets, and distribute SSH keys to employees so that they can use port forwarding.
- D. Expose the internal applications through an internet-facing ALB, and restrict its security group to the employees' home IP addresses.

### Q55
A company runs Amazon RDS for PostgreSQL as a Multi-AZ DB instance deployment. Failovers during patching take 1–2 minutes, which the business considers too long. The company also wants to send read-only queries to the standby capacity instead of paying for separate read replicas. The database must stay on RDS for PostgreSQL because of extensions that the application depends on. Which solution meets these requirements?

- A. Add a cross-Region read replica, and promote it if the primary DB instance fails.
- B. Put Amazon RDS Proxy in front of the existing Multi-AZ DB instance deployment.
- C. Migrate the database to Amazon Aurora PostgreSQL, and use Aurora Replicas for the read-only queries.
- D. Migrate to an RDS Multi-AZ DB cluster deployment, which has a writer instance and two readable standby instances in three Availability Zones.

### Q56
A company must move 60 TB of archive data, one time, from its data center to Amazon S3 within 4 weeks. The data center has a 1 Gbps internet connection, and about half of that bandwidth is free around the clock. The company has no Direct Connect connection. Which solution is the MOST cost-effective way to meet the deadline?

- A. Order AWS Snowball Edge devices, load the data onto them, and ship the devices back to AWS for import.
- B. Copy the data over the existing internet connection with parallel multipart uploads (for example, `aws s3 sync`), throttled to the free bandwidth.
- C. Order a new AWS Direct Connect connection for the migration, and cancel it when the copy finishes.
- D. Enable S3 Transfer Acceleration on the bucket, and upload the data through the accelerated endpoint.

### Q57
A company serves a website and file downloads from a private S3 bucket through Amazon CloudFront by using a legacy origin access identity (OAI). A compliance rule now requires all objects to be encrypted with SSE-KMS using a customer managed key, and since the change, CloudFront returns 403 errors for the encrypted objects. The bucket must stay private and reachable only through this distribution. Which solution meets these requirements?

- A. Make the bucket publicly readable, and add a bucket policy condition that allows requests only when they carry the website's Referer header.
- B. Re-encrypt the objects with SSE-S3, keep the OAI, and record a formal exception to the new compliance rule.
- C. Replace the OAI with origin access control (OAC), allow the CloudFront service principal in the bucket policy with a condition on the distribution's ARN, and allow it to use `kms:Decrypt` in the key policy.
- D. Change the origin to the bucket's S3 website endpoint, and have CloudFront add a secret custom header that the bucket policy checks.

### Q58
An e-commerce application uses Amazon RDS for MySQL. Every night, and several times during the day, analysts run heavy reporting queries that drive the database's CPU to 95% and slow down checkout. The reports can use data that is a few seconds old. The company wants to fix the problem quickly with minimal changes. Which solution meets these requirements?

- A. Create an RDS read replica, and point the reporting tool at the replica's endpoint.
- B. Scale the primary DB instance up to the next larger instance class.
- C. Enable Multi-AZ, and point the reporting tool at the standby DB instance.
- D. Put an Amazon ElastiCache cluster in front of the database to cache the reporting query results.

### Q59
A company wants disaster recovery in AWS for 80 on-premises VMware virtual machines that run a mix of Windows and Linux applications and databases. It needs an RPO of seconds and an RTO of minutes, wants to pay as little as possible for idle AWS resources, and wants to run regular recovery drills without interrupting replication. Which solution meets these requirements?

- A. Use AWS Backup with a Backup gateway to back up the VMs every day, and restore them to EC2 during a disaster.
- B. Use AWS Application Migration Service to replicate the VMs and cut them over to EC2 permanently.
- C. Run duplicate EC2 instances 24/7 in AWS, and replicate the databases to them with AWS DMS.
- D. Use AWS Elastic Disaster Recovery to replicate the servers continuously to a low-cost staging area, and launch recovery instances for drills or a disaster.

### Q60
A company has 80 accounts in AWS Organizations across four Regions. The CISO wants a single view of findings from GuardDuty, Inspector, and Macie, plus automated checks against the AWS Foundational Security Best Practices and CIS benchmarks, with new accounts enrolled automatically. Which combination of actions meets these requirements with the LEAST operational overhead? (Choose two.)

- A. Create a cross-account IAM role in every account, and run a scheduled Lambda function in the security account that collects findings from each service every hour.
- B. Enable Amazon Detective in all accounts, and use it as the central compliance dashboard for the benchmark checks.
- C. Designate a security tooling account as the AWS Security Hub delegated administrator, and use central configuration to enable Security Hub and its standards in all current and new accounts.
- D. Enable CloudTrail Insights in every account so that insecure resource configurations are detected automatically.
- E. Configure Security Hub cross-Region aggregation with a single aggregation (home) Region.

### Q61
A company has 12 AWS accounts that each pay their own bills. Each account bought its own Compute Savings Plans. Some accounts now have unused commitment every hour, while other accounts pay On-Demand rates for similar compute. The company wants to use its existing commitments as fully as possible with minimal effort. Which solution meets these requirements?

- A. Sell the unused Savings Plans commitments on the Reserved Instance Marketplace.
- B. Move all of the accounts into one AWS Organization with consolidated billing, and keep Savings Plans discount sharing turned on.
- C. Exchange the Compute Savings Plans for Convertible Reserved Instances in the accounts that have On-Demand usage.
- D. Purchase additional Savings Plans in the accounts that are paying On-Demand rates.

### Q62
A company runs a self-managed Apache Kafka cluster on premises. Its producers and consumers use the Kafka client libraries, Kafka Connect, and Kafka Streams. The company wants to move the platform to AWS, stop managing brokers, and avoid application code changes. Existing topics and their data must be migrated with minimal downtime. Which solution meets these requirements?

- A. Migrate to Amazon Kinesis Data Streams, and rewrite the producers and consumers to use the Kinesis Producer Library and the Kinesis Client Library.
- B. Migrate to Amazon MQ, and convert the Kafka topics to ActiveMQ virtual topics.
- C. Create an Amazon MSK cluster, and use MirrorMaker 2 (for example, on MSK Connect) to replicate topics from the on-premises cluster before cutting the clients over.
- D. Run Kafka on EC2 instances in an Auto Scaling group with EBS volumes, and copy the topic data with a custom consumer application.

### Q63
An IoT platform receives sensor readings through API Gateway, which invokes a Lambda function that writes each reading to Amazon RDS for PostgreSQL. During traffic spikes of up to 20 times normal volume, Lambda scales out, exhausts the database's connections, and readings are lost. Readings can be stored up to a few minutes after they arrive, but none may be lost. Which solution meets these requirements with the LEAST operational overhead?

- A. Have API Gateway send each reading to an Amazon SQS queue through a direct service integration, and process the queue with a Lambda event source mapping that has a maximum concurrency setting.
- B. Increase the Lambda function's reserved concurrency so that more instances of the function are available to absorb the traffic spikes.
- C. Move the database to a larger instance class with a higher `max_connections` value so that it can accept more concurrent connections.
- D. Have API Gateway publish each reading to an Amazon SNS topic, and subscribe the existing Lambda function to the topic.

### Q64
A global application encrypts sensitive fields on the client side before writing them to a DynamoDB global table that is replicated between us-east-1 and ap-southeast-2. The application in each Region must decrypt items that were written in the other Region. It must not make cross-Region KMS calls, and it must keep working if the other Region is unavailable. Which solution meets these requirements with the LEAST complexity?

- A. Create a separate single-Region KMS key in each Region, and use DynamoDB Streams with a Lambda function to re-encrypt each item with the local key after replication.
- B. Create a KMS multi-Region primary key in us-east-1 and a replica key in ap-southeast-2, and have each Regional application use its local key with the client-side encryption library.
- C. Create one single-Region KMS key in us-east-1, and have the application in both Regions call that key for encryption and decryption.
- D. Rely on DynamoDB encryption at rest with an AWS owned key, and remove the client-side field encryption from the application.

### Q65
A fleet of EC2 instances in private subnets writes about 30 TB per month to DynamoDB and reads large objects from Amazon S3 in the same Region. All of this traffic currently goes through NAT gateways, and NAT gateway data processing charges are now the largest networking cost. Which solution reduces this cost the MOST with the least effort?

- A. Replace the NAT gateways with NAT instances that run on burstable instance types.
- B. Create interface VPC endpoints for DynamoDB and S3 in each Availability Zone.
- C. Move the instances to public subnets, and assign them public IPv4 addresses so that they bypass the NAT gateways.
- D. Create gateway VPC endpoints for DynamoDB and S3, and associate them with the private subnets' route tables.

---

## Answer key & explanations
> Don't open until you've finished all 65.

<details>
<summary>Q1: Answer C</summary>

**Domain:** D1 · **Task:** 1.1

**Why C:** A **permissions boundary** sets the maximum permissions a role can ever have: its effective permissions are the identity policy ∩ the boundary (∩ any SCPs). If you allow `iam:CreateRole`, `iam:PutRolePolicy`, `iam:AttachRolePolicy` and similar actions only when `iam:PermissionsBoundary` equals the platform boundary ARN, then every role a team lead creates carries the cap. Also deny `iam:DeleteRolePermissionsBoundary` and any edits to the boundary policy itself (`iam:CreatePolicyVersion`, `iam:DeletePolicy`) to close the escalation path. After that one-time setup the process is self-service.

**Why not A / B / D:**
- A: The SCP limits which IAM APIs can be called, not which permissions the new roles get. A lead could still attach `AdministratorAccess` to a new role and pass it to a Lambda function.
- B: This control is detective and weekly. An over-privileged role can be abused long before the review runs.
- D: `AdministratorAccess` lets leads do anything in the account, including widening their own access.

⚠️ Trap: SCPs and boundaries both *limit* permissions and neither *grants* any. SCPs apply to whole accounts or OUs. Boundaries apply to specific IAM principals, which is the classic answer for delegated administration.

**Review:** [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q2: Answer B</summary>

**Domain:** D2 · **Task:** 2.1

**Why B:** An SQS FIFO queue keeps messages in order **within a message group** and removes duplicates sent within the 5-minute deduplication interval. With the order ID as the group ID, each order's events arrive in sequence, while different orders (different groups) are consumed in parallel by many workers. If throughput becomes a concern, high-throughput FIFO mode raises the limits a lot.

**Why not A / C / D:**
- A: You build deduplication yourself, and sorting within one batch doesn't help when the "paid" event arrives in a *different* batch from the "shipped" event.
- C: A single group ID serializes the whole queue to one in-flight message group, which destroys the parallelism the stem requires.
- D: SNS standard and SQS standard topics and queues give at-least-once delivery with no ordering, which is the original problem.

**Review:** [Decoupling & serverless](../notes/06-decoupling-serverless.md)

</details>

<details>
<summary>Q3: Answer A</summary>

**Domain:** D3 · **Task:** 3.5

**Why A:** Repeated, complex, join-heavy BI queries from hundreds of concurrent users is the textbook Amazon Redshift workload. It provides columnar MPP storage, result caching, materialized views and concurrency scaling. Distribution and sort keys co-locate the big joins. Redshift Serverless is an option if the team doesn't want to size a cluster. Athena is excellent for ad hoc or infrequent queries on S3, but each query scans S3 and is billed per TB scanned, so a dashboard estate that refreshes all day gets steadier latency, and often a lower bill, from a warehouse.

**Why not B / C / D:**
- B: CSV is a row format. Queries read *more* bytes and run slower. Parquet is already the better format.
- C: RDS for PostgreSQL is a row-store OLTP engine. 20 TB of analytical joins would be slow, and read replicas don't change the engine's nature.
- D: Starting a cluster for each refresh adds minutes of latency and a lot of moving parts.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q4: Answer D</summary>

**Domain:** D4 · **Task:** 4.4

**Why D:** The traffic flows *out* of AWS. **Data transfer out over Direct Connect is billed at lower per-GB rates than internet data transfer out**, so a steady 4 TB/day for 3+ years easily pays for the port-hour charges. DX also gives consistent bandwidth and latency, which fixes the throughput swings.

**Why not A / B / C:**
- A: ECMP across more tunnels increases throughput, but every GB is still billed at internet data transfer out rates, plus VPN connection-hour charges and transit gateway data processing charges.
- B: Shipping devices every day isn't practical. Also, AWS Snowball Edge has been closed to new customers since Nov 2025.
- C: Transfer Acceleration *adds* a per-GB fee. It speeds up long-distance transfers to and from S3 and doesn't lower cost.

⚠️ Trap: Data transfer *into* AWS is free over the internet, so "DX saves money" is about data **out** at a **steady** volume. For occasional or small transfers, VPN is cheaper.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q5: Answer B</summary>

**Domain:** D1 · **Task:** 1.2

**Why B:** GuardDuty sends every finding to Amazon EventBridge automatically. A rule that filters on the finding `type` (and optionally on severity) can invoke a Lambda function or an SSM Automation runbook. That target swaps the instance's security groups for a quarantine group, snapshots the volumes and notifies through SNS. This is the standard event-driven, serverless auto-remediation pattern: response within minutes, with no servers or schedules to manage.

**Why not A / C / D:**
- A: A nightly batch job responds hours late.
- C: Inspector finds software vulnerabilities and network exposure. It doesn't act on GuardDuty threat findings, and it doesn't stop instances.
- D: GuardDuty is **detective only** and has no blocking option. To block domains, use Route 53 Resolver DNS Firewall or AWS Network Firewall.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q6: Answer B, E</summary>

**Domain:** D2 · **Task:** 2.1

**Why B:** While a message is in flight it is hidden for the visibility timeout (30 s by default, 12 h max). Here the timeout expires long before a 12-minute job finishes, so the message becomes visible again and a second worker picks it up. Set the timeout above the worst-case duration, or use a heartbeat that calls `ChangeMessageVisibility`.

**Why E:** A dead-letter queue with `maxReceiveCount` takes "poison" messages out of circulation after N failed attempts. You can then inspect them, fix them and redrive them.

**Why not A / C / D:**
- A: A 1-minute retention period also deletes good messages that are waiting in a backlog.
- C: FIFO queues use the same visibility timeout mechanics, so a 30-second timeout still causes re-delivery. "Exactly-once" refers to deduplication on send. A poison message would also block its whole message group.
- D: Long polling reduces empty receives and cost. It doesn't affect duplicate processing or poison messages.

**Review:** [Decoupling & serverless](../notes/06-decoupling-serverless.md)

</details>

<details>
<summary>Q7: Answer A, C, F</summary>

**Domain:** D1 · **Task:** 1.1

**Why A, C, F:** Cross-account access needs an allow on **both sides**: the resource policy in the owning account and the identity policy in the calling account. With SSE-KMS there are **two resources**, the bucket and the key, and each has its own resource policy. So you need:
- A: a bucket policy that allows the Account A role.
- C: a key policy that allows `kms:Decrypt` for the Account A role or account.
- F: an identity policy in Account A that allows both `s3:GetObject` and `kms:Decrypt` on the key ARN.

**Why not B / D / E:**
- B: Object Ownership controls who owns uploaded objects and whether ACLs apply. The default (bucket owner enforced) is correct here, and the setting doesn't grant read access.
- D: Granting a specific external principal in a bucket policy isn't "public", so Block Public Access doesn't block it. Turning BPA off only adds risk.
- E: Default encryption applies only to *new* objects. Existing objects still need the KMS key.

⚠️ Trap: Objects encrypted with the **AWS managed key** (`aws/s3`) can't be shared across accounts, because you can't edit its key policy. Use a customer managed key.

**Review:** [Security & data protection](../notes/07-security-data-protection.md) · [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q8: Answer C</summary>

**Domain:** D3 · **Task:** 3.3

**Why C:** Amazon Neptune is a purpose-built graph database. Multi-hop relationship traversals (account → device → account → card → …) run in milliseconds with Gremlin, openCypher or SPARQL. Fraud-ring detection is the canonical graph use case.

**Why not A / B / D:**
- A: GSIs find direct matches, but every extra hop is another round trip, and the fan-out explodes by hop 4–5.
- B: Redshift is an OLAP engine with second-to-minute queries and refresh lag. It doesn't fit inline authorization.
- D: OpenSearch is built for search and log analytics and has no native multi-hop graph traversal.

**Review:** [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q9: Answer A</summary>

**Domain:** D2 · **Task:** 2.2

**Why A:** An Aurora global database replicates at the storage layer with typical lag under a second (RPO ≈ 1 s). The secondary Region serves local reads in eu-west-1. In an outage, **global database failover** promotes the secondary in minutes, accepting the small replication lag as data loss. For planned drills, **switchover** moves the primary with no data loss. All of this is managed, so operational overhead is low.

**Why not B / C / D:**
- B: Binlog replication puts load on the primary, its lag grows under load, and promotion is a manual process.
- C: Everything is in one Region, so a Regional outage takes it all down.
- D: RPO is up to an hour, and restoring a large cluster takes far longer than a few minutes.

**Review:** [Databases](../notes/05-databases.md) · [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q10: Answer A, D</summary>

**Domain:** D4 · **Task:** 4.2

**Why A:** Fargate Spot runs tasks on spare capacity at a deep discount, with a 2-minute interruption warning. An idempotent, checkpointing batch job is the ideal fit.

**Why D:** **Compute Savings Plans cover Fargate** (as well as EC2 and Lambda), so a commitment sized to the API's 24/7 baseline discounts it without any availability risk.

**Why not B / C / E:**
- B: EC2 Instance Savings Plans apply only to EC2 usage in one instance family and Region, not to Fargate.
- C: Interruptions on a customer-facing API put availability at risk. A common pattern keeps a `FARGATE` base and adds `FARGATE_SPOT` only for burst capacity.
- E: Fargate has no Reserved Instances.

**Review:** [Compute & containers](../notes/03-compute-containers.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q11: Answer D</summary>

**Domain:** D1 · **Task:** 1.3

**Why D:** AWS CloudHSM gives you **single-tenant** HSMs in your VPC. You create and manage the HSM users and keys, and AWS has no access to the key material. The current HSM type is validated at FIPS 140-3 Level 3. CloudHSM supports the standard **PKCS #11, JCE** and OpenSSL/KSP/CNG interfaces, so the Java app needs only a provider change. Put HSMs in at least two AZs for high availability, because keys sync across the cluster.

**Why not A / B / C:**
- A: Importing key material controls where the key comes from, but the KMS HSMs are still multi-tenant and AWS-operated. The app would also have to be rewritten for the KMS API.
- B: Multi-Region keys solve a cross-Region problem, not tenancy or control.
- C: Secrets Manager isn't an HSM. The keys would be released into application memory.

⚠️ Trap: AWS KMS HSMs are also FIPS 140-3 Level 3 validated, so the FIPS level alone doesn't decide this question. The deciding signals are **single-tenant**, **exclusive customer control** and **PKCS #11/JCE**. If the app should keep the KMS API but needs CloudHSM-backed keys, the answer is a KMS custom key store.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q12: Answer B</summary>

**Domain:** D3 · **Task:** 3.1

**Why B:** S3 File Gateway presents an NFS (or SMB) share on premises and stores each file as a native S3 object. Lifecycle rules, Athena and event notifications all work on those objects. A local cache keeps recently used data at low latency, and the apps need no code changes.

**Why not A / C / D:**
- A: Volume Gateway exposes iSCSI block volumes, and its data lands in AWS as volume data and EBS snapshots, not as individual S3 objects you can query. The apps also expect NFS.
- C: FSx for Windows File Server is an SMB file system in AWS. It doesn't solve the on-premises NFS problem, and a nightly copy leaves a gap.
- D: EFS over DX is NFS, but the data lives in EFS, not as S3 objects, and every access crosses the WAN with no local cache.

**Review:** [Storage](../notes/04-storage.md)

</details>

<details>
<summary>Q13: Answer C</summary>

**Domain:** D2 · **Task:** 2.2

**Why C:** AWS Global Accelerator gives you **two static anycast IP addresses** for TCP or UDP. It health-checks the endpoints in each endpoint group and stops sending traffic to an unhealthy Region automatically, in less than a minute according to AWS, with **no DNS involved**. Traffic enters the AWS backbone at the nearest edge location, which reduces latency and jitter. Traffic dials allow controlled shifts during drills.

**Why not A / B / D:**
- A: Route 53 failover is DNS-based. The clients use hard-coded IPs and never re-resolve, and resolver caching delays any change anyway.
- B: CloudFront handles HTTP and HTTPS (and WebSocket), not an arbitrary TCP protocol on port 7777.
- D: Elastic IPs are Regional and can't move between Regions. You also can't move an NLB's addresses this way.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q14: Answer A</summary>

**Domain:** D1 · **Task:** 1.2

**Why A:** AWS Network Firewall stateful **domain list** rule groups match on TLS SNI and the HTTP Host header. Allowlist mode with a default drop action, plus alert and flow logs, satisfies "allow 50 domains, block and log the rest". A central egress VPC with the firewall endpoints placed between the transit gateway attachments and the NAT gateways is the AWS reference design.

**Why not B / C / D:**
- B: Security groups accept CIDRs, prefix lists and security group references, not domain names.
- C: NACLs are stateless and IP-based. SaaS and CDN IP addresses change often, so the rules would break.
- D: AWS WAF protects *inbound* HTTP(S) to resources such as CloudFront, ALB and API Gateway. You can't attach it to a NAT gateway, and it isn't an egress filter.

Route 53 Resolver DNS Firewall is a useful complement because it blocks name resolution, but a workload that connects directly to an IP address bypasses it.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q15: Answer D</summary>

**Domain:** D4 · **Task:** 4.1

**Why D:** S3 Glacier Instant Retrieval returns objects **in milliseconds**, like Standard-IA, but at a lower storage price with a higher per-GB retrieval charge. That trade-off suits data that is read about once a quarter. Its 90-day minimum storage duration doesn't matter with 7-year retention, and it stores data across multiple AZs.

**Why not A / B / C:**
- A: Standard-IA also works, but its storage costs more. It suits data that is read about monthly.
- B: Expedited retrieval takes 1–5 minutes, not milliseconds, and costs extra on every request.
- C: One Zone-IA stores data in a single AZ, which breaks the multi-AZ requirement, and it still costs more than Glacier Instant Retrieval.

⚠️ Trap: "Glacier" doesn't always mean slow. **Instant Retrieval returns objects in milliseconds**. Flexible Retrieval and Deep Archive take minutes to hours.

**Review:** [Storage](../notes/04-storage.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q16: Answer B</summary>

**Domain:** D2 · **Task:** 2.1

**Why B:** Moving session state out to ElastiCache (Valkey or Redis OSS) makes the web tier stateless. Reads take under a millisecond, and a Multi-AZ replication group fails over automatically. Most web frameworks have a Redis session provider, so the change is mostly configuration. With state off the instances, turn off stickiness so the ALB spreads load evenly and scale-in no longer logs users out.

**Why not A / C / D:**
- A: Longer stickiness concentrates users on particular instances and does nothing when an instance dies.
- C: EBS Multi-Attach works only for io1 and io2 volumes **within one AZ**, needs a cluster-aware file system and can't span the ASG's AZs.
- D: RDS stores sessions durably, but its latency is in milliseconds, not sub-millisecond, and it puts connection pressure on the database. DynamoDB would be a valid durable store, and DAX would bring it into microseconds.

**Review:** [Databases](../notes/05-databases.md) · [Compute & containers](../notes/03-compute-containers.md)

</details>

<details>
<summary>Q17: Answer A</summary>

**Domain:** D1 · **Task:** 1.2

**Why A:** With **VPC sharing**, the network account uses AWS RAM to share subnets with participant accounts in the same organization. Participants create, own and pay for their own EC2, RDS and ELB resources in those subnets. They can't modify the VPC, subnets, route tables or NACLs, which stay with the owner. This reduces the number of VPCs, peering connections and TGW attachments while keeping networking centrally controlled.

**Why not B / C / D:**
- B: A VPC in every account brings back the sprawl the company wants to avoid. Peering isn't transitive, and each account controls its own routing.
- C: The resources would be owned and billed in the network account, which breaks the account isolation and cost ownership that multi-account setups exist for.
- D: Service Catalog shares templates that people use to provision resources. It can't share existing subnets.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q18: Answer B, C</summary>

**Domain:** D3 · **Task:** 3.2

**Why B and C:** This is AWS's documented **backlog per instance** pattern. The acceptable backlog equals the acceptable wait divided by the processing time per job: 300 s ÷ 3 s = **100**. Divide queue depth by the number of InService instances, either as a custom metric or with metric math inside the target tracking policy, and target 100. Capacity then scales in proportion to the actual work.

**Why not A / D / E:**
- A: Scheduled scaling suits predictable patterns, and spikes aren't predictable.
- D: Target tracking assumes the metric falls in proportion as capacity is added. Raw queue depth doesn't reflect fleet size, so AWS advises against tracking it directly.
- E: Scaling on CPU has the same lag problem the company already has.

**Review:** [Compute & containers](../notes/03-compute-containers.md)

</details>

<details>
<summary>Q19: Answer A, E</summary>

**Domain:** D1 · **Task:** 1.3

**Why A:** An **organization trail** applies to every current and future member account in all Regions. Member accounts can see the trail but can't stop or delete it. It can deliver to a bucket in the log-archive account, whose bucket policy must allow CloudTrail. Turn on log file integrity validation as well.

**Why E:** **Object Lock in compliance mode** means no user, *including the root user*, can delete or overwrite a locked object version, or shorten its retention, until the retention period expires.

**Why not B / C / D:**
- B: You would have to maintain a trail in every account and every new account, and member-account admins could stop their own trails.
- C: MFA delete still lets the root user, with MFA, delete versions, so the logs aren't immutable.
- D: In governance mode, anyone with `s3:BypassGovernanceRetention` can remove the protection.

**Review:** [Security & data protection](../notes/07-security-data-protection.md) · [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q20: Answer C</summary>

**Domain:** D2 · **Task:** 2.2

**Why C:** A **warm standby** is a scaled-down but fully working copy that is already warm. At failover, Route 53 shifts traffic, the warm instances serve right away and the ASG scales out behind them. The Aurora global database supplies an RPO of about 1 s and promotes in minutes. This is the cheapest design that meets a 15-minute RTO when new instances need 30 minutes to warm up.

**Why not A / B / D:**
- A: Backup and restore has an RTO measured in hours and an RPO equal to the backup interval.
- B: A pilot light is cheaper, but every instance would need about 30 minutes to warm up after launch, which pushes recovery past 15 minutes.
- D: Active-active meets the targets, but it costs the most and needs a database re-platform.

| Strategy | Typical RTO | What runs in the DR Region |
|---|---|---|
| Backup & restore | Hours | Nothing (only backups) |
| Pilot light | Tens of minutes | Data replication; compute off |
| Warm standby | Minutes | A scaled-down, working stack |
| Multi-site active-active | Near zero | A full stack serving traffic |

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q21: Answer D</summary>

**Domain:** D4 · **Task:** 4.4

**Why D:** A transit gateway turns the full mesh into a hub and spoke: one attachment per VPC, plus the VPN attachments, transitive routing and route tables for segmentation. AWS RAM shares it across accounts. A transit gateway charges per attachment-hour and **per GB processed**. For the one high-volume pair, a direct peering connection avoids the per-GB processing fee. Peering has no processing charge; traffic within one AZ is free, and normal inter-AZ rates apply otherwise. Mixing TGW and peering like this is common and correct.

**Why not A / B / C:**
- A: The design still needs 990 connections. Peering isn't transitive, and each VPC would need its own VPN.
- B: A transit VPC is a legacy pattern: you run the appliances, their HA and their licences.
- C: PrivateLink exposes specific services in one direction (consumer to provider). It isn't any-to-any routing, and the endpoint costs multiply across VPCs.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q22: Answer B</summary>

**Domain:** D3 · **Task:** 3.5

**Why B:** OpenSearch Service keeps an inverted index, so full-text search over hot data is sub-second, and OpenSearch Dashboards provides the visualizations. Amazon Data Firehose handles buffered, retried delivery. Index State Management moves indexes older than 14 days to **UltraWarm** (S3-backed) and then **cold** storage, which keeps them queryable at a fraction of the hot-storage price.

**Why not A / C / D:**
- A: Logs Insights is good for ad hoc queries, but each query scans the log data, so it isn't sub-second across 14 days at 2 TB/day, and ingestion costs at this volume are high.
- C: Athena is a reasonable choice for occasional queries on archived logs, but it scans S3 without a full-text index, so queries take seconds to minutes.
- D: DynamoDB has no full-text search.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q23: Answer A</summary>

**Domain:** D1 · **Task:** 1.2

**Why A:** Standard Parameter Store parameters cost nothing extra and are the right home for non-secret configuration, with hierarchy and versioning. Secrets Manager charges per secret, but it has **built-in managed rotation** for RDS credentials, with single-user or alternating-user strategies that avoid downtime. RDS can also manage the master password in Secrets Manager for you. Pay for Secrets Manager only where you need rotation.

**Why not B / C / D:**
- B: You would pay for 40 secrets that don't need rotation.
- C: Parameter Store has no native rotation. Writing and coordinating the database password change yourself costs more effort and adds risk.
- D: User data isn't a configuration store, and anyone with the right describe permission can read it. A password in an S3 object has no rotation.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q24: Answer C</summary>

**Domain:** D2 · **Task:** 2.1

**Why C:** Amazon MQ runs managed ActiveMQ (or RabbitMQ) brokers that speak the industry protocols: JMS/OpenWire, STOMP, AMQP, MQTT and WebSocket. The migration is lift and shift: change the connection URL and keep the code. An **active/standby** broker spans two AZs and fails over automatically when clients use a failover transport URL.

**Why not A / B / D:**
- A: Moving to SQS and SNS means rewriting the producers and consumers.
- B: You would still patch and run the brokers yourself.
- D: Kafka is a different protocol and model. A bridge adds components and still needs changes.

⚠️ Trap: SQS is the default decoupling answer *unless* the stem mentions existing JMS, AMQP, MQTT or STOMP clients and "no code changes". Then the answer is Amazon MQ.

**Review:** [Decoupling & serverless](../notes/06-decoupling-serverless.md)

</details>

<details>
<summary>Q25: Answer D</summary>

**Domain:** D1 · **Task:** 1.2

**Why D:** Amazon Inspector scans automatically and continuously. It covers EC2 (through the SSM Agent, or agentless EBS snapshot scanning), ECR images (enhanced scanning re-evaluates images on push and when new CVEs are published) and Lambda. A delegated administrator in Organizations can turn it on automatically for new member accounts and see all findings, which also flow to Security Hub.

**Why not A / B / C:**
- A: GuardDuty Runtime Monitoring detects threats at runtime, such as suspicious processes. It isn't a CVE inventory.
- B: Macie discovers sensitive data in S3 only.
- C: Basic scanning runs on push or on demand, not continuously, and the self-run scanners are overhead you would have to maintain.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q26: Answer C, D</summary>

**Domain:** D3 · **Task:** 3.5

**Why C:** Do the math first. 200 TB over 100 Mbps takes about **185 days** even with the link fully used, so the bulk copy has to travel physically. Snowball Edge Storage Optimized devices are loaded on premises and imported into S3.

**Why D:** After the import, DataSync compares the source and destination and moves only new or changed files on a schedule. About 50 GB/day takes roughly an hour at 100 Mbps.

**Why not A / B / E:**
- A: Transfer Acceleration optimizes long-distance network paths, but it can't make a 100 Mbps link faster.
- B: A VPN encrypts traffic over the same internet link and adds no bandwidth.
- E: Tape Gateway is for backup software that writes to virtual tapes. The data wouldn't be usable as S3 objects.

⚠️ AWS Snowball Edge has been **closed to new customers since Nov 7, 2025**. Existing customers can still order devices, which is why the stem says the institute already uses it. The exam guide still lists the Snow Family, so learn the decision rule. In real projects, new customers look at AWS Data Transfer Terminal, partner solutions, or DataSync over a faster link.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q27: Answer A</summary>

**Domain:** D4 · **Task:** 4.2

**Why A:** The instances are used for 55 of 168 hours a week (about 33%), so stopping them off-hours removes about **two thirds** of the instance-hours. Instance Scheduler on AWS is an AWS solution that stops and starts EC2 and RDS instances according to tag-based schedules, deployed once. EBS and RDS storage are still billed while instances are stopped. RDS also starts a stopped instance automatically after 7 days, which a daily schedule handles.

**Why not B / C / D:**
- B: Reserved Instances discount every hour of the term, including hours you don't need. At a 33% duty cycle, stopping saves more than any RI discount.
- C: RDS doesn't run on Spot, Spot dev instances can be interrupted, and they would still run all week.
- D: Rightsizing helps a little but leaves everything running 24/7.

**Review:** [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q28: Answer B</summary>

**Domain:** D2 · **Task:** 2.1

**Why B:** Kinesis Data Streams **retains** records (24 h by default, extendable to 7 days and up to 365 days). Each consumer tracks its own position and can re-read from a timestamp or sequence number, which provides **replay**. Using the session ID as the partition key keeps ordering per session within a shard. Enhanced fan-out gives each consumer dedicated read throughput, and on-demand mode scales to the ingest rate.

**Why not A / C / D:**
- A: SQS has no consumer groups. A message is gone once it's processed and deleted, so there's no replay.
- C: Each queue gets a copy, but consumed messages are deleted, so the new team can't replay them. Standard SNS topics and SQS queues don't guarantee order.
- D: EventBridge archive and replay **doesn't guarantee ordering**. A replay works through the archive one minute of event time at a time, and events aren't necessarily replayed in the order they arrived, so per-session order can be lost. (The enhanced custom event bus launched in Sept 2026 adds ordered FIFO subscribers and replay from a past position. Exam questions still expect Kinesis Data Streams for high-volume, ordered, replayable streams.)

**Review:** [Decoupling & serverless](../notes/06-decoupling-serverless.md) · [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q29: Answer B, D</summary>

**Domain:** D1 · **Task:** 1.2

**Why B and D:** Interface VPC endpoints (PrivateLink) for SQS and KMS put ENIs in your subnets. With **private DNS** enabled, the services' default public hostnames resolve to those private IPs, so the SDK needs no endpoint changes. Secure the endpoints with a security group that allows 443 from the app, and with endpoint policies.

**Why not A / C / E:**
- A: Gateway endpoints exist **only for S3 and DynamoDB**.
- C: A NAT gateway is an internet egress path, which the design explicitly forbids.
- E: A public VIF connects on-premises routers to AWS public endpoints. It does nothing for a VPC that has no internet gateway.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md)

</details>

<details>
<summary>Q30: Answer D</summary>

**Domain:** D3 · **Task:** 3.3

**Why D:** Amazon Keyspaces is a serverless, **CQL-compatible** database. Existing Cassandra drivers connect over TLS with SigV4 or service-specific credentials, and there are no nodes to patch, repair or scale. You choose on-demand or provisioned capacity.

**Why not A / B / C:**
- A: DynamoDB is serverless, but moving to it means rewriting the data access layer.
- B: DocumentDB speaks the MongoDB API, so the CQL applications would have to be rewritten.
- C: A relational database means a rewrite plus instance management.

**Review:** [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q31: Answer C</summary>

**Domain:** D2 · **Task:** 2.2

**Why C:** An ALB listener rule can **forward to several target groups with weights**, so traffic shifts at the load balancer and DNS never changes. The blue ASG and target group stay intact, and rollback means setting the weights back, which takes seconds. Releases are immutable because new code runs only on new instances. Add target group stickiness if each client should stay on one version during the canary.

**Why not A / B / D:**
- A: Route 53 weighted routing is DNS, and clients that cache DNS ignore both the shift and the rollback.
- B: An instance refresh replaces instances on a rolling basis. It has no 10% traffic canary, and rolling back means running another refresh.
- D: An Elastic Beanstalk URL swap is a CNAME (DNS) change.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q32: Answer B</summary>

**Domain:** D4 · **Task:** 4.3

**Why B:** RDS Reserved Instances give the **largest discount** for a steady database that won't change: a 3-year term, All Upfront, matching the engine, Region, deployment type (Multi-AZ) and instance family. Size flexibility within the family applies to PostgreSQL.

**Why not A / C / D:**
- A: Compute Savings Plans don't cover RDS.
- C: Aurora Serverless v2 bills per ACU-hour. A steady 24/7 load usually costs more than reserved provisioned capacity, and the migration takes effort.
- D: Single-AZ is cheaper but breaks the high-availability requirement.

Note: AWS launched **Database Savings Plans** in Dec 2025. They cover RDS, Aurora, DynamoDB and other database services and are flexible across engines, families and Regions. At launch they had a 1-year term, no upfront option and smaller discounts than 3-year RIs. Choose them when flexibility matters more than the maximum discount, and check the current terms.

**Review:** [Cost optimization](../notes/10-cost-optimization.md) · [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q33: Answer A</summary>

**Domain:** D1 · **Task:** 1.2

**Why A:** The AWS Config managed rule `restricted-ssh` re-evaluates security groups whenever they change. Automatic remediation runs an SSM Automation runbook, such as `AWS-DisablePublicAccessForSecurityGroup`, that removes the rule. Config keeps a **compliance timeline** for auditors. Deploy it org-wide from a delegated administrator as an organization conformance pack, which carries the remediation action to every account (organization Config rules alone don't include remediation). You write no custom code.

**Why not B / C / D:**
- B: This works, but you write and maintain the parsing and revocation code, and you get no compliance history.
- C: GuardDuty analyzes logs for threats. It doesn't evaluate security group configuration or remediate it.
- D: This blocks *every* ingress change by developers, including legitimate ones. SCPs can't easily single out rules that allow only 0.0.0.0/0.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q34: Answer C</summary>

**Domain:** D3 · **Task:** 3.4

**Why C:** A Network Load Balancer works at layer 4. It handles millions of requests per second at ultra-low latency, absorbs sudden spikes without pre-warming, and supports **one static IP (an Elastic IP) per AZ**. It also preserves the client source IP for instance targets. With TCP listeners, the servers terminate TLS themselves, as the stem requires.

**Why not A / B / D:**
- A: An ALB works at layer 7, adds more latency and has no static IP addresses. WAF is irrelevant here.
- B: A Gateway Load Balancer inserts virtual appliances such as firewalls into the traffic path (GENEVE). It isn't an application front end.
- D: The Classic Load Balancer is legacy, with no static IPs and lower scale.

⚠️ Trap: Static IPs plus HTTP routing features → put the ALB behind Global Accelerator (or an NLB with an ALB target). Static IPs plus extreme TCP performance → NLB.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md)

</details>

<details>
<summary>Q35: Answer A, C</summary>

**Domain:** D2 · **Task:** 2.2

**Why A:** Service quotas apply **per account, per Region**. A DR Region that never runs production usually sits at the default quotas, so raise them in advance. In Organizations, quota request templates can do this for new accounts.

**Why C:** AMIs are **Regional**. Copy them to the DR Region as part of the build (EC2 Image Builder distribution settings do this). Store each Region's AMI ID in Parameter Store and resolve it in CloudFormation with an SSM parameter type.

**Why not B / D / E:**
- B: A notification doesn't raise a quota.
- D: Spot has its own vCPU quota, and capacity may be scarce or interrupted at exactly the moment a Regional event drives demand. Don't build DR on Spot.
- E: Cross-stack exports are Region-scoped, and an AMI ID from another Region can't launch instances anyway.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q36: Answer D</summary>

**Domain:** D1 · **Task:** 1.2

**Why D:** **Signed cookies** authorize many files at once without changing their URLs. A custom policy with a wildcard path covers the manifest and every segment, and it can set an expiry time (and optionally a start time and an IP range). Sign with a trusted key group. Serve the distribution from an alternate domain under the company's domain so the browser sends the cookies.

**Why not A / B / C:**
- A: Signed URLs work per file and need query strings, which the player can't add. Rewriting manifests for every viewer is fragile.
- B: Presigned URLs bypass CloudFront, losing caching and edge delivery, and they are still per-object query strings.
- C: The client controls the Referer header and can spoof it trivially. It isn't authentication.

Rule of thumb: one file → **signed URL**; many files, or URLs that must not change → **signed cookies**.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q37: Answer D, E</summary>

**Domain:** D4 · **Task:** 4.1

**Why D:** User-defined tags appear in billing data only after you **activate them as cost allocation tags** in the management (payer) account.

**Why E:** A CUR 2.0 export from AWS Data Exports gives hourly, resource-level line items with amortized RI and Savings Plans columns, delivered as Parquet to S3. You can query it with Athena or visualize it in Amazon Quick Sight.

**Why not A / B / C:**
- A: Cost Explorer is good for trends and tag filters once tags are activated, but it has no SQL, and hourly or resource-level views cover only a short recent window.
- B: Budgets sends alerts. It doesn't produce chargeback data.
- C: Trusted Advisor makes recommendations. It doesn't allocate costs.

**Review:** [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q38: Answer B</summary>

**Domain:** D3 · **Task:** 3.5

**Why B:** AWS Lake Formation manages permissions centrally on Data Catalog resources. It grants access at the database, table and column level, and **data filters** add row-level and cell-level security. LF-Tags help permissions scale, and cross-account sharing uses AWS RAM under the hood. Athena, Redshift Spectrum, EMR and Glue all enforce these permissions.

**Why not A / C / D:**
- A: Copying data per audience and maintaining policies per prefix doesn't scale, and S3 policies can't restrict columns.
- C: Macie *discovers* sensitive data. It doesn't enforce access or mask query results.
- D: Workgroups separate query settings and costs. IAM can't restrict access by column.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q39: Answer A</summary>

**Domain:** D2 · **Task:** 2.1

**Why A:** AWS X-Ray traces each request across every service. The service map shows latency and errors on each edge, and each trace breaks down the time spent in every downstream call, including DynamoDB and the third-party HTTP API. API Gateway and Lambda turn on active tracing with a single setting. The ECS services send traces through an ADOT collector.

Note: The X-Ray SDKs and daemon entered maintenance mode in Feb 2026, and AWS recommends OpenTelemetry (ADOT) for instrumentation. The traces still go to X-Ray, and exam questions still say "X-Ray" for distributed tracing.

**Why not B / C / D:**
- B: Flow Logs record IP addresses, ports, byte counts and accept/reject decisions, not per-request latency.
- C: CloudTrail is an audit trail of API calls. It doesn't break request latency down by call.
- D: The per-service metrics already look healthy. Higher resolution can't attribute latency to a call path.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q40: Answer C</summary>

**Domain:** D1 · **Task:** 1.3

**Why C:** Direct Connect traffic **isn't encrypted by default**. There are two ways to encrypt it: MACsec (layer 2) on supported dedicated connections, or an IPsec Site-to-Site VPN over DX. The VPN can terminate on public VPN endpoints through a public VIF, or run as a private IP VPN to a transit gateway through a transit VIF. A partner's hosted connection can't use MACsec, so the answer is IPsec over DX.

**Why not A / B / D:**
- A: MACsec is available only on certain *dedicated* high-speed connections, not on hosted connections.
- B: A private circuit isn't an encrypted one.
- D: Client VPN is for individual users' devices, not for site-to-site connectivity.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md)

</details>

<details>
<summary>Q41: Answer D</summary>

**Domain:** D3 · **Task:** 3.3

**Why D:** Amazon DocumentDB is managed and MongoDB API-compatible, so existing drivers and most queries and aggregations keep working. Its storage grows automatically and is replicated six ways across three AZs. Read replicas (up to 15) share that storage, so you can add them in minutes. AWS DMS supports MongoDB-to-DocumentDB migrations with a full load followed by ongoing change data capture.

**Why not A / B / C:**
- A: Moving to DynamoDB means rewriting the queries and aggregation pipelines.
- B: Keyspaces uses a wide-column model and CQL, which means a rewrite.
- C: JSONB can store the documents, but the queries and aggregation pipelines would have to be rewritten in SQL.

**Review:** [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q42: Answer B</summary>

**Domain:** D4 · **Task:** 4.4

**Why B:** NAT gateways bill **per hour per gateway** plus per GB processed. For low-traffic non-production, one shared NAT gateway removes two thirds of the hourly charges. The trade-offs are an AZ-failure blast radius, which the team has accepted, and small cross-AZ data charges from the other AZs. In production, keep one NAT gateway per AZ.

Note: The newer **regional NAT gateway** mode (Nov 2025) spans AZs automatically and needs no public subnet. Exam questions still frame the choice as one shared zonal NAT gateway versus one per AZ.

**Why not A / C / D:**
- A: Every public IPv4 address is billed hourly, and the instances would become directly internet-addressable.
- C: Interface endpoints reach AWS services and PrivateLink services, not public package repositories on the internet.
- D: Savings Plans don't apply to NAT gateways.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q43: Answer B, E</summary>

**Domain:** D2 · **Task:** 2.2

**Why E:** A CloudFront **origin group** has a primary and a secondary origin. When the primary returns one of the configured status codes (such as 500, 502, 503 or 504) or times out, CloudFront retries **that request** against the secondary. This happens per request, with no DNS involved. Origin failover applies to GET, HEAD and OPTIONS requests.

**Why B:** S3 Cross-Region Replication keeps the secondary bucket's content in sync. Use OAC on both origins.

**Why not A / C / D:**
- A: This fails over through DNS across two distributions. It has more moving parts and can't fail over per request.
- C: Transfer Acceleration speeds up client uploads and downloads to S3. It has nothing to do with failover.
- D: Global Accelerator endpoints are ALBs, NLBs, EC2 instances and Elastic IPs, not S3 buckets.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md) · [Storage](../notes/04-storage.md)

</details>

<details>
<summary>Q44: Answer A</summary>

**Domain:** D1 · **Task:** 1.1

**Why A:** AD Connector is a **proxy**. It forwards authentication requests to the on-premises domain controllers and doesn't store or cache any directory data in AWS. It is a supported identity source for IAM Identity Center and the cheapest directory option. It needs network connectivity to the domain controllers, which Direct Connect already provides. IAM Identity Center syncs user and group attributes into its identity store for assignments, but never passwords.

**Why not B / C / D:**
- B: Managed Microsoft AD *is* a separate directory running in AWS, which policy forbids, and it costs more. Choose it when AWS workloads need a real AD in AWS: AD-aware applications, trusts, or independence from on-premises connectivity.
- C: Simple AD is a standalone, Samba-based directory. It can't synchronize with or trust an on-premises AD.
- D: Cognito user pools hold customer and application identities. They don't sign workforce users in to the AWS access portal.

**Review:** [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q45: Answer C</summary>

**Domain:** D3 · **Task:** 3.2

**Why C:** AWS Batch provides job queues with priorities, job dependencies, array jobs and retry strategies. Managed compute environments choose instance types for you and can use Spot with allocation strategies, or run on Fargate or EKS. There's no scheduler for you to operate.

**Why not A / B / D:**
- A: Lambda has a 15-minute maximum, and these jobs run up to 90 minutes.
- B: You would operate the scheduler, head node and scaling yourself.
- D: ECS *services* keep long-running tasks alive. They have no job queue, dependencies or retries.

**Review:** [Compute & containers](../notes/03-compute-containers.md)

</details>

<details>
<summary>Q46: Answer D</summary>

**Domain:** D2 · **Task:** 2.2

**Why D:** AWS Backup manages backups centrally across EBS, RDS, DynamoDB, EFS and more. **Backup policies** in Organizations push backup plans to every account. Copy rules send recovery points to vaults in a central backup account and in a second Region. **Vault Lock in compliance mode** makes them immutable, so no one, including the root user, can delete them before retention ends. One copy rule can target a vault that is in another account *and* another Region, and RDS and Aurora snapshots now support this combined copy too. DynamoDB supports cross-account and cross-Region copies only after you turn on AWS Backup's advanced DynamoDB features. Logically air-gapped vaults are another option.

**Why not A / B / C:**
- A: Data Lifecycle Manager covers EBS only, is configured per account and keeps copies in the same account.
- B: This is piecemeal, keeps everything in the same account, and admins there can delete the backups.
- C: EBS and RDS snapshots aren't stored in your own S3 buckets, so CRR can't copy them.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q47: Answer A, D</summary>

**Domain:** D1 · **Task:** 1.3

**Why A:** **EBS encryption by default** is a per-Region, per-account setting. Once it's on, every new volume is encrypted with the chosen key, with no template changes needed.

**Why D:** You **can't encrypt an existing unencrypted RDS instance in place**. Take a snapshot, copy it with encryption and the customer managed key, restore a new instance from the copy and cut over. To keep downtime short, keep the new instance in sync with AWS DMS change data capture until cutover.

**Why not B / C / E:**
- B: The console doesn't offer encryption for an existing unencrypted DB instance.
- C: RDS can't create an encrypted read replica from an unencrypted DB instance.
- E: This is a useful guardrail, but it breaks launches until teams change their templates, and the requirement says no template changes.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q48: Answer B</summary>

**Domain:** D4 · **Task:** 4.1

**Why B:** EFS lifecycle management moves files to **Infrequent Access** once they have gone N days without access (30 here). A second policy moves a file back to Standard on its first access. Everything stays in the same file system under the same paths, so the application changes nothing. IA costs much less to store but adds a per-GB access charge. For data read only a few times a year, the **Archive** class can be added as a further tier.

**Why not A / C / D:**
- A: The application would have to change, and Deep Archive retrieval takes hours, not an immediate read.
- C: You can't convert an existing file system to One Zone; you would have to create a new one and copy the data. One Zone also stores data in a single AZ, and IA saves more on cold files anyway.
- D: Throughput mode doesn't affect storage price, and Provisioned throughput usually adds cost.

**Review:** [Storage](../notes/04-storage.md)

</details>

<details>
<summary>Q49: Answer C</summary>

**Domain:** D3 · **Task:** 3.5

**Why C:** DataSync is a managed online transfer service. The on-premises agent reads the NFS share, and DataSync encrypts traffic with TLS, verifies data integrity and transfers only what changed. Tasks run on schedules with bandwidth limits that you can adjust during a run, and they write directly to EFS. DataSync can also run privately over DX through a VPC endpoint.

**Why not A / B / D:**
- A: You would build and maintain custom scripts, error handling and monitoring, and the single EC2 instance becomes a bottleneck.
- B: Shipping a device every night for incremental changes takes days per round trip and isn't practical.
- D: Files would be copied twice through S3 into EFS, with custom glue code in between.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q50: Answer A</summary>

**Domain:** D2 · **Task:** 2.2

**Why A:** AWS Transfer Family provides fully managed, highly available SFTP (plus FTPS, FTP and AS2) endpoints backed by S3 or EFS. Service-managed users authenticate with their SSH public keys, and a custom host name (Route 53 alias or CNAME) means partners change nothing. A VPC-hosted endpoint with Elastic IPs supports partner IP allowlists. S3 event notifications or Transfer Family managed workflows trigger the processing.

**Why not B / C / D:**
- B: You would still run, patch and scale SFTP servers, and syncing to S3 adds lag.
- C: This breaks the requirement to keep SFTP and hands long-term credentials to outside parties.
- D: File Gateway exposes NFS or SMB to clients on the local network, not SFTP to partners on the internet.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q51: Answer D</summary>

**Domain:** D1 · **Task:** 1.1

**Why D:** **ABAC** grants access when the principal's tags match the resource's tags. One policy serves 300 projects or 3,000, and a new project needs only a new tag value. IAM Identity Center passes IdP attributes as session tags ("attributes for access control"). Add `aws:RequestTag` conditions so new resources must be tagged, and deny changes to the `Project` tag.

**Why not A / B / C:**
- A: Per-project roles (RBAC) cause policy sprawl and need an update every time resources change.
- B: More than 300 accounts is far too heavy for this requirement.
- C: EC2 instances don't support resource-based policies, and maintaining a policy per secret doesn't scale.

**Review:** [IAM & multi-account](../notes/01-iam-multi-account.md)

</details>

<details>
<summary>Q52: Answer B</summary>

**Domain:** D4 · **Task:** 4.4

**Why B:** Data that crosses AZs is charged per GB in each direction, while traffic over private IPs **within one AZ is free**. Put the reporting replica in the same AZ as its consumers. RDS doesn't charge for replication traffic between the source and a read replica in the same Region, so moving the replica adds no replication cost. The primary keeps its Multi-AZ deployment.

**Why not A / C / D:**
- A: A Multi-AZ replica adds a standby that can't serve reads, so you pay more and the traffic stays the same.
- C: A NAT gateway adds a per-GB processing charge, and the traffic still crosses AZs.
- D: A transit gateway adds a per-GB processing charge, and both subnets are already in the same VPC.

**Review:** [Cost optimization](../notes/10-cost-optimization.md) · [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q53: Answer C</summary>

**Domain:** D3 · **Task:** 3.2

**Why C:** Memory-optimized instances have a high memory-to-vCPU ratio, about 8 GiB per vCPU in the R family. The X family goes much higher, up to multiple TiB per instance, for in-memory databases and analytics. Holding the whole dataset in RAM removes the paging.

**Why not A / B / D:**
- A: Compute-optimized instances have a low memory-to-vCPU ratio, and even the largest size can't hold 900 GB.
- B: NVMe swap is still orders of magnitude slower than RAM.
- D: Burstable instances are for low, spiky CPU and have small amounts of memory.

**Review:** [Compute & containers](../notes/03-compute-containers.md)

</details>

<details>
<summary>Q54: Answer A</summary>

**Domain:** D1 · **Task:** 1.2

**Why A:** AWS Client VPN is a managed, elastically scaling remote-access VPN (OpenVPN-based). It supports **SAML 2.0 federated authentication**, with MFA enforced by the IdP, as well as Active Directory or mutual-certificate authentication. Authorization rules restrict access by network and group, and security groups apply to the endpoint's network interfaces.

**Why not B / C / D:**
- B: Site-to-Site VPN connects networks through customer gateway devices, not individual laptops.
- C: Bastion hosts are servers to patch and keys to distribute, with no native SAML or MFA.
- D: This exposes internal apps to the internet, home IP addresses change, and the network layer has no MFA.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md)

</details>

<details>
<summary>Q55: Answer D</summary>

**Domain:** D2 · **Task:** 2.2

**Why D:** An **RDS Multi-AZ DB cluster** (MySQL or PostgreSQL) has one writer and **two readable standbys** in three AZs, with semi-synchronous replication. It fails over faster than a Multi-AZ DB instance (typically well under a minute), and a reader endpoint serves read-only queries.

**Why not A / B / C:**
- A: A cross-Region replica is for DR and Regional reads. Promoting it is manual, replication is asynchronous, and it doesn't speed up in-Region failover.
- B: RDS Proxy reduces the impact of failover on clients by holding connections, but the standby still can't serve reads.
- C: The stem rules out Aurora.

⚠️ Trap: A Multi-AZ **DB instance** standby can't be read. A Multi-AZ **DB cluster's** standbys can.

**Review:** [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q56: Answer B</summary>

**Domain:** D4 · **Task:** 4.4

**Why B:** Do the math. 60 TB at about 500 Mbps takes about **11 days** of transfer, or about 2 weeks with protocol overhead, which fits the 4-week deadline. Data transfer *into* AWS is free, so you pay only for S3 requests. DataSync would also work if you want scheduling and verification, but it adds a per-GB fee.

**Why not A / C / D:**
- A: Snowball works, but it adds device fees, shipping and days of logistics. It pays off when the network transfer would take too long. It has also been closed to new customers since Nov 2025.
- C: A new DX connection takes weeks to provision and has port-hour charges, which is overkill for a one-time copy.
- D: Transfer Acceleration adds a per-GB fee and mainly helps over long distances.

⚠️ Trap: "Large data, so Snowball" is a reflex the exam punishes. Do the math every time.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q57: Answer C</summary>

**Domain:** D1 · **Task:** 1.2

**Why C:** **OAC** replaces OAI. It signs requests with SigV4, works in all Regions, supports PUT and DELETE and, crucially, **supports SSE-KMS**. The bucket policy allows the `cloudfront.amazonaws.com` service principal with `AWS:SourceArn` set to the distribution's ARN. The KMS key policy allows the same principal `kms:Decrypt` (plus `kms:Encrypt` and `kms:GenerateDataKey*` if you add uploads) under the same condition.

**Why not A / B / D:**
- A: The bucket would be public and the Referer header is spoofable, which violates "private".
- B: This breaks the compliance requirement.
- D: The S3 website endpoint requires a publicly readable bucket, accepts only HTTP from the origin side, and can't serve SSE-KMS objects.

**Review:** [Security & data protection](../notes/07-security-data-protection.md) · [Storage](../notes/04-storage.md)

</details>

<details>
<summary>Q58: Answer A</summary>

**Domain:** D3 · **Task:** 3.3

**Why A:** A read replica receives asynchronous replication and has its own endpoint. It offloads heavy reporting from the primary, and a few seconds of lag are acceptable here. The only change is the reporting tool's connection string.

**Why not B / C / D:**
- B: A bigger instance costs more, and the reports and checkout would still compete for the same database.
- C: The standby of a Multi-AZ **DB instance** can't serve reads.
- D: Ad hoc reports rarely repeat the same query, so a cache would get few hits, and adding one needs code changes.

**Review:** [Databases](../notes/05-databases.md)

</details>

<details>
<summary>Q59: Answer D</summary>

**Domain:** D2 · **Task:** 2.2

**Why D:** AWS Elastic Disaster Recovery replicates block-level changes continuously into a **low-cost staging area** of small replication servers and EBS volumes. That gives an RPO of seconds, and full-size recovery instances launch in minutes. Drills don't interrupt replication, and failback to on premises is supported. DRS works for VMware, physical servers and other clouds.

**Why not A / B / C:**
- A: Daily backups mean an RPO of up to 24 hours, and restores take hours.
- B: Application Migration Service is for a one-way migration, not ongoing DR with drills and failback.
- C: A duplicate fleet running 24/7 is the most expensive option, and DMS replicates only databases.

**Review:** [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

</details>

<details>
<summary>Q60: Answer C, E</summary>

**Domain:** D1 · **Task:** 1.1

**Why C:** Name a Security Hub **delegated administrator** through Organizations. **Central configuration** policies then turn on Security Hub, its standards (FSBP, CIS) and its controls across accounts and OUs, and enroll new accounts automatically. GuardDuty, Inspector and Macie findings flow into Security Hub.

**Why E:** **Cross-Region aggregation** collects findings from every linked Region into one home Region, giving the single view.

**Why not A / B / D:**
- A: A custom collector is brittle and is exactly the work Security Hub does for you.
- B: Detective helps investigate the root cause of findings. It doesn't run compliance standards.
- D: CloudTrail Insights detects unusual API call volumes or error rates, not insecure configurations.

Note: Since Dec 2025, AWS has offered a new unified **Security Hub**, and the original service, which runs the standards checks, is now called **Security Hub CSPM**. The delegated-administrator and home-Region aggregation pattern applies to both. Exam questions just say "Security Hub".

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q61: Answer B</summary>

**Domain:** D4 · **Task:** 4.2

**Why B:** With **consolidated billing**, RI and Savings Plans discounts apply across all accounts in the organization. Sharing is on by default and can be turned off for specific accounts. Each commitment applies first to the account that bought it, and any unused part then covers the other accounts. Usage also aggregates for volume pricing tiers.

**Why not A / C / D:**
- A: Savings Plans can't be resold. Only EC2 Standard RIs can be sold on the Reserved Instance Marketplace.
- C: You can't exchange a Savings Plan for RIs.
- D: Buying more commitment while existing commitment goes unused adds waste.

**Review:** [Cost optimization](../notes/10-cost-optimization.md)

</details>

<details>
<summary>Q62: Answer C</summary>

**Domain:** D3 · **Task:** 3.5

**Why C:** Amazon MSK is managed Apache Kafka with the same APIs. Producers, consumers, Kafka Streams and Kafka Connect (self-run or on MSK Connect) keep working with a new bootstrap broker string. MirrorMaker 2 replicates topics and consumer-group offsets from the on-premises cluster, so the cutover needs little downtime. MSK Serverless is an option if the throughput profile fits.

**Why not A / B / D:**
- A: Moving to Kinesis means rewriting every producer and consumer.
- B: Amazon MQ runs ActiveMQ or RabbitMQ brokers and doesn't speak the Kafka protocol.
- D: You would still manage the brokers yourself, and the custom copy tool is extra risk.

**Review:** [Data, analytics & migration](../notes/09-data-analytics-migration.md)

</details>

<details>
<summary>Q63: Answer A</summary>

**Domain:** D2 · **Task:** 2.1

**Why A:** Put a **queue between the spike and the database**. API Gateway writes directly to SQS with no Lambda in front, and SQS stores every reading durably. The Lambda event source mapping's **maximum concurrency** setting caps how many functions run at once, which caps the database connections. Add a DLQ for failures, and consider RDS Proxy for connection pooling.

**Why not B / C / D:**
- B: More concurrency means more database connections, which makes the problem worse.
- C: A larger instance buys headroom, but the next spike exhausts connections again, and it costs more.
- D: SNS invokes Lambda asynchronously with no concurrency control. After the retry window, failed events are dropped unless you configure an on-failure destination.

**Review:** [Decoupling & serverless](../notes/06-decoupling-serverless.md)

</details>

<details>
<summary>Q64: Answer B</summary>

**Domain:** D1 · **Task:** 1.3

**Why B:** **Multi-Region keys** are related keys in several Regions that share the same key ID and key material. Data encrypted under the primary in one Region can be decrypted *locally* with the replica in another Region, with no cross-Region call and no dependency on the other Region. This is exactly the use case for client-side encryption with global tables (the encryption SDKs support MRKs). Each replica keeps its own key policy, grants and aliases.

**Why not A / C / D:**
- A: A re-encryption pipeline is complex, and items can't be decrypted until it runs.
- C: The app would depend on another Region, add latency, and stop decrypting during an outage there.
- D: This drops the field-level encryption requirement. Encryption at rest doesn't protect fields from anyone who can read the table.

⚠️ Trap: Multi-Region keys aren't the default. Use them only when ciphertext has to move between Regions, because some compliance regimes prefer keys that stay in one Region.

**Review:** [Security & data protection](../notes/07-security-data-protection.md)

</details>

<details>
<summary>Q65: Answer D</summary>

**Domain:** D4 · **Task:** 4.4

**Why D:** **Gateway VPC endpoints** for S3 and DynamoDB have **no hourly or per-GB charge**. They add prefix-list routes to the route tables, keep traffic on the AWS network, and support endpoint policies. Same-Region traffic to S3 and DynamoDB then bypasses the NAT gateway completely.

**Why not A / B / C:**
- A: NAT instances remove the NAT gateway fees but add management work, a bottleneck and a single point of failure.
- B: Interface endpoints for S3 and DynamoDB exist and are useful for on-premises or peered access, but they charge per hour per AZ and per GB. Gateway endpoints are free.
- C: Each public IPv4 address is billed hourly, and the instances become internet-exposed.

**Review:** [Networking & VPC](../notes/02-networking-vpc.md) · [Cost optimization](../notes/10-cost-optimization.md)

</details>

---

## Score by domain
| Domain | Questions | Your correct | % |
|---|---|---|---|
| D1 Secure | Q1, Q5, Q7, Q11, Q14, Q17, Q19, Q23, Q25, Q29, Q33, Q36, Q40, Q44, Q47, Q51, Q54, Q57, Q60, Q64 (20) | | |
| D2 Resilient | Q2, Q6, Q9, Q13, Q16, Q20, Q24, Q28, Q31, Q35, Q39, Q43, Q46, Q50, Q55, Q59, Q63 (17) | | |
| D3 High-Performing | Q3, Q8, Q12, Q18, Q22, Q26, Q30, Q34, Q38, Q41, Q45, Q49, Q53, Q58, Q62 (15) | | |
| D4 Cost-Optimized | Q4, Q10, Q15, Q21, Q27, Q32, Q37, Q42, Q48, Q52, Q56, Q61, Q65 (13) | | |
| **Total** | 65 | | |

### What your score means
| Total score | Action |
|---|---|
| **≥ 80% (52+)** | Book the exam. First, review every question you got wrong *and* every one you guessed correctly. |
| **70–79% (46–51)** | Spend one more week on your two weakest domains: re-read their notes, then work the matching drill set. Don't retake this mock, because you'll remember the answers. |
| **< 70% (≤ 45)** | Go back to the [study plan](../00-study-plan.md) and redo the week 3–4 modules for your weak domains, then take the domain drills again. |

Where to go for each weak domain:

| Weak domain | Re-read | Drill |
|---|---|---|
| D1 Secure (below 14/20) | [IAM & multi-account](../notes/01-iam-multi-account.md), [Networking & VPC](../notes/02-networking-vpc.md), [Security & data protection](../notes/07-security-data-protection.md) | [D1 drills](d1-secure.md) |
| D2 Resilient (below 12/17) | [Decoupling & serverless](../notes/06-decoupling-serverless.md), [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md) | [D2 drills](d2-resilient.md) |
| D3 High-Performing (below 11/15) | [Compute & containers](../notes/03-compute-containers.md), [Storage](../notes/04-storage.md), [Databases](../notes/05-databases.md), [Data, analytics & migration](../notes/09-data-analytics-migration.md) | [D3 drills](d3-high-performing.md) |
| D4 Cost-Optimized (below 9/13) | [Cost optimization](../notes/10-cost-optimization.md) | [D4 drills](d4-cost-optimized.md) |

The per-domain cut-offs are about 70%. Scoring on the real exam is compensatory, so one weak domain won't fail you by itself, but a weak D1 (30% of the exam) costs the most points.
