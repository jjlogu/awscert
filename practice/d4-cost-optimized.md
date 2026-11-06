# Domain 4 practice: Design Cost-Optimized Architectures (15 questions)
> [Home](../README.md) · [Practice index](README.md) · Notes: [Cost optimization](../notes/10-cost-optimization.md) · [Storage](../notes/04-storage.md) · [Compute & containers](../notes/03-compute-containers.md) · [Databases](../notes/05-databases.md) · [Networking & VPC](../notes/02-networking-vpc.md)

Untimed. Read each explanation, including why the wrong options are wrong. Log misses in your error log.

---

### Q1. Lifecycle rules for logs with a known access pattern
An application writes about 2 TB of log files per day to an Amazon S3 bucket in S3 Standard. Engineers query the logs often during the first 30 days. From day 31 to day 365, the logs are accessed about once a quarter for audits and must be available within milliseconds when requested. After 1 year the logs are almost never read, but they must be kept for 7 years in total, and retrieval within 48 hours is acceptable. Logs must be deleted after 7 years.

Which solution is the MOST cost-effective?

- A. Store all logs in S3 Intelligent-Tiering with the optional Deep Archive Access tier turned on, and add a lifecycle rule that expires objects after 7 years.
- B. Add a lifecycle rule that transitions objects to S3 Standard-IA after 30 days and to S3 Glacier Flexible Retrieval after 365 days, and expires them after 7 years.
- C. Add a lifecycle rule that transitions objects to S3 Glacier Flexible Retrieval after 30 days and to S3 Glacier Deep Archive after 365 days, and expires them after 7 years.
- D. Add a lifecycle rule that transitions objects to S3 Glacier Instant Retrieval after 30 days and to S3 Glacier Deep Archive after 365 days, and expires them after 7 years.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** When the access pattern is **known**, lifecycle rules are the cheapest option. **Glacier Instant Retrieval** is designed for data read about **once a quarter** that still needs **millisecond** access. Its storage price is lower than Standard-IA's, with a higher per-GB retrieval charge, and its 90-day minimum is easily met here. **Deep Archive** is the cheapest storage class, and its retrievals (standard within 12 hours, bulk within 48 hours) fit the 48-hour window. The expiration action deletes objects at 7 years.

**Why not the others:**
- **A**: Intelligent-Tiering is for **unknown or changing** patterns. You pay a per-object monitoring fee for movement you could schedule yourself, and the quarterly audit reads move objects back to the frequent tier.
- **B**: Standard-IA works for days 31–365 but costs more to store than Glacier IR for quarterly access. Glacier Flexible Retrieval costs more than Deep Archive when 48 hours is acceptable.
- **C**: Glacier Flexible Retrieval needs a **restore** before you can read an object (minutes to hours), which breaks the millisecond requirement for days 31–365.

**Task statement:** 4.1 · **Review:** [Storage: storage classes and lifecycle rules](../notes/04-storage.md)
</details>

---

### Q2. Unpredictable access across a data lake
A company's data lake in Amazon S3 holds petabytes of objects, most of them larger than 1 MB. Access patterns are unpredictable: some datasets go untouched for months and then become heavily used again when a new project starts. All data must be readable within milliseconds at any time. The company doesn't have staff to analyze access patterns and wants to avoid retrieval fees.

Which storage approach is the MOST cost-effective?

- A. Add a lifecycle rule that transitions all objects to S3 Standard-IA after 30 days.
- B. Store the objects in S3 Intelligent-Tiering, using only its default, automatic access tiers.
- C. Add a lifecycle rule that transitions all objects to S3 One Zone-IA after 30 days.
- D. Add a lifecycle rule that transitions all objects to S3 Glacier Instant Retrieval after 90 days.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Intelligent-Tiering** moves each object automatically between the Frequent, Infrequent (after 30 days without access), and Archive Instant Access (after 90 days) tiers. All three serve reads in **milliseconds**, and **there are no retrieval fees**. An object that becomes hot again moves back to the Frequent tier. You pay a small monitoring fee per object, and objects smaller than 128 KB aren't monitored and are always billed at the Frequent rate (most objects here are larger). Leave the optional asynchronous archive tiers off, because they would break the millisecond requirement.

**Why not the others:**
- **A**: When a dataset becomes busy again, you pay per-GB retrieval fees on every read, and the objects stay in IA unless you move them back.
- **C**: One Zone-IA keeps data in a single AZ and also charges retrieval fees.
- **D**: Reactivated datasets would pay Glacier IR's higher retrieval charges on every read.

**Task statement:** 4.1 · **Review:** [Storage: S3 Intelligent-Tiering](../notes/04-storage.md)
</details>

---

### Q3. Making data consumers pay for their downloads
A research institute shares a 400 TB genomics dataset in an Amazon S3 bucket with partner universities. Every partner has its own AWS account, and partners regularly download large parts of the dataset. The institute's grant covers storage costs only, so each partner must pay the request and data transfer costs of its own downloads.

Which solution meets these requirements with the LEAST operational overhead?

- A. Put an Amazon CloudFront distribution in front of the bucket, and give each partner signed URLs.
- B. Configure S3 Cross-Region Replication from the bucket to a bucket in each partner's account.
- C. Enable Requester Pays on the bucket, grant the partner accounts access in the bucket policy, and have the partners include the requester-pays parameter in their requests.
- D. Create an IAM user in the institute's account for each partner, and send each partner a monthly invoice based on S3 server access logs.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** With **Requester Pays**, the **requester's AWS account** pays for the requests and the data transfer, and the bucket owner pays only for storage. Requesters must confirm that they accept the charges, with the `x-amz-request-payer` header or `--request-payer requester` in the CLI. ⚠️ Trap: **Anonymous requests aren't allowed** on a Requester Pays bucket, because AWS must know which account to bill.

**Why not the others:**
- **A**: CloudFront charges go to the distribution owner, which is the institute.
- **B**: The **source** bucket owner pays for replication requests and inter-Region transfer, and the 400 TB would be stored again in every partner account.
- **D**: This means long-term credentials and a manual billing process. AWS can do the billing for you.

**Task statement:** 4.1 · **Review:** [Storage: Requester Pays](../notes/04-storage.md)
</details>

---

### Q4. Cutting EBS cost without downtime
A company runs 300 EC2 instances, each with a 1 TiB gp2 Amazon EBS data volume. Monitoring shows that each volume needs about 3,000 IOPS and no more than 125 MiB/s of throughput. The company wants to lower its EBS cost without downtime or any loss of performance.

Which solution meets these requirements?

- A. Modify each volume in place to gp3 with Elastic Volumes, keeping the gp3 baseline of 3,000 IOPS and 125 MiB/s.
- B. Change the volumes to st1, because throughput-optimized HDD volumes cost less per GiB.
- C. Create snapshots of the volumes, and restore them as io2 volumes with 3,000 provisioned IOPS.
- D. Shrink each gp2 volume in place to 500 GiB to reduce the cost of each volume.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **gp3** costs less per GiB than gp2 (approx. 20% less) and includes a **baseline of 3,000 IOPS and 125 MiB/s at any size**, which matches this workload exactly. With **Elastic Volumes**, you change the type **online**: no detaching and no downtime. Each modification must complete before the next one starts, and a volume can be modified at most four times in a rolling 24-hour period, so get the target settings right the first time and roll out a large fleet in waves.

**Why not the others:**
- **B**: st1 is an HDD type for large sequential I/O. Its random IOPS are far below 3,000.
- **C**: io2 charges for provisioned IOPS on top of storage, so it costs more, and restoring from snapshots is unnecessary work.
- **D**: ⚠️ Trap: EBS volumes **can't be shrunk** in place, only grown. A 500 GiB gp2 volume would also have only a 1,500 IOPS baseline.

**Task statement:** 4.1 · **Review:** [Storage: EBS volume types](../notes/04-storage.md)
</details>

---

### Q5. Replacing a physical tape library
A company's on-premises backup software writes weekly full backups to a physical tape library through an iSCSI virtual tape library (VTL) interface, and staff ship tapes to an offsite vault. The company wants to stop managing physical tapes and the offsite vault, while keeping its existing backup software and tape-based workflows unchanged. Backups must be retained for 7 years, and restoring an archived backup within 48 hours is acceptable.

Which solution is the MOST cost-effective?

- A. Deploy Amazon S3 File Gateway, reconfigure the backup software to write to an NFS share, and add a lifecycle rule that transitions objects to S3 Glacier Deep Archive.
- B. Deploy AWS Storage Gateway Tape Gateway, and archive ejected virtual tapes to the S3 Glacier Flexible Retrieval pool.
- C. Use AWS DataSync to copy the backup files from the backup server to an S3 bucket in the S3 Glacier Deep Archive storage class every week.
- D. Deploy AWS Storage Gateway Tape Gateway, and archive ejected virtual tapes to the S3 Glacier Deep Archive pool.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **Tape Gateway** presents an iSCSI **VTL** (a media changer and tape drives) to the existing backup software, so the workflows don't change. Active virtual tapes are stored in S3. When the software **ejects** a tape, it moves to the archive pool you chose. The **Deep Archive pool** is the lowest-cost option, and retrieving an archived tape takes hours, well within the 48-hour window.

**Why not the others:**
- **A**: Writing to a file share changes the tape-based workflow, which the company wants to keep.
- **B**: This works, but the Flexible Retrieval pool costs more to store. Its faster retrieval isn't needed with a 48-hour window.
- **C**: DataSync doesn't understand tapes, and the backup software would have to change to disk-based backups.

**Task statement:** 4.1 · **Review:** [Storage: Storage Gateway](../notes/04-storage.md)
</details>

---

### Q6. Cheapest compute for a fault-tolerant batch
A company renders animation frames in a nightly batch that runs for about 5 hours on a fleet of On-Demand EC2 instances. Each frame is an independent job that checkpoints its progress, and failed jobs are retried automatically from a queue. The work must finish within an 8-hour window. The company wants to reduce compute cost as much as possible.

Which combination of actions meets these requirements? **(Choose two.)**

- A. Run the batch on Spot Instances, for example through an AWS Batch managed compute environment or an Auto Scaling group that uses Spot capacity.
- B. Purchase 3-year Standard Reserved Instances that cover the full size of the batch fleet.
- C. Allow several instance types and sizes across multiple Availability Zones, and use the price-capacity-optimized allocation strategy.
- D. Run the batch on Dedicated Hosts so that capacity is always available during the window.
- E. Request Spot Instances with a defined duration (Spot blocks) of 6 hours so that they aren't interrupted.

<details>
<summary>Show answer</summary>

**Answer: A, C**

**Why A and C:** Independent jobs that checkpoint and retry are the textbook **Spot** workload. Spot can cost up to about 90% less than On-Demand, and an interruption (with its 2-minute warning) only costs a retry. **Diversifying** across instance types, sizes, and AZs gives the fleet more capacity pools to draw from. The **price-capacity-optimized** strategy picks pools that are both low-priced and least likely to be interrupted. The 3-hour slack in the window absorbs any interruptions.

**Why not the others:**
- **B**: Reserved Instances bill around the clock. A fleet that runs 5 hours a day would waste about 80% of the commitment.
- **D**: Dedicated Hosts are for licensing or compliance needs and are the most expensive way to get capacity.
- **E**: ⚠️ Trap: **Spot blocks are no longer offered.** Older practice material may still mention them.

**Task statement:** 4.2 · **Review:** [Compute & containers: purchasing options and Spot](../notes/03-compute-containers.md)
</details>

---

### Q7. A commitment that follows workloads as they change
A company has a steady compute baseline that it expects to keep for at least 3 years. Today the baseline runs on EC2 instances in several instance families across two AWS Regions. Over the next year, the company plans to move some workloads to newer instance families, some to Amazon ECS on AWS Fargate, and some to AWS Lambda. The company wants a commitment-based discount that keeps applying as the workloads change.

Which purchasing option meets these requirements?

- A. Compute Savings Plans
- B. EC2 Instance Savings Plans
- C. Standard Reserved Instances
- D. Convertible Reserved Instances

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** With **Compute Savings Plans**, you commit to a spend per hour (in USD) for 1 or 3 years. The discount applies automatically to EC2 in **any instance family, size, AZ, Region, operating system, or tenancy**, and also to **Fargate and Lambda**. In exchange for that flexibility, the discount is somewhat smaller than with EC2 Instance Savings Plans or Standard RIs.

**Why not the others:**
- **B**: An EC2 Instance Savings Plan is tied to **one instance family in one Region** and covers EC2 only.
- **C**: Standard RIs are tied to an instance family and Region (size flexibility applies only within a family for Regional Linux RIs) and cover EC2 only.
- **D**: Convertible RIs can be exchanged for other EC2 configurations, but exchanges are manual and they **never cover Fargate or Lambda**.

**Task statement:** 4.2 · **Review:** [Cost optimization: Savings Plans and Reserved Instances](../notes/10-cost-optimization.md)
</details>

---

### Q8. A low-traffic API running all day
An internal API receives about 5,000 requests per day, mostly during business hours, and each request completes in under 500 ms. The API runs 24 hours a day on two On-Demand EC2 instances in different Availability Zones behind an Application Load Balancer. The instances' average CPU utilization is under 5%. The company wants to reduce cost and operational overhead while keeping the API highly available.

Which solution meets these requirements MOST cost-effectively?

- A. Purchase 1-year Reserved Instances for the two EC2 instances, and keep the ALB.
- B. Replace the two instances with a single larger Spot Instance behind the ALB.
- C. Rebuild the API on Amazon API Gateway (HTTP API) with AWS Lambda functions.
- D. Move the API to Amazon ECS on AWS Fargate with two tasks running at all times behind the ALB.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** At a few thousand short requests a day, **pay-per-request** pricing beats paying for idle capacity by a wide margin. Lambda and API Gateway are **Multi-AZ by default**, and there are no instances to patch. HTTP APIs cost less than REST APIs and are enough for a simple internal API.

**Why not the others:**
- **A**: RIs lower the hourly rate, but you still pay 24/7 for instances that are 95% idle, plus the ALB's hourly charge.
- **B**: A single Spot Instance can be interrupted at any time, and one instance isn't highly available.
- **D**: Fargate removes server management, but two tasks running all the time plus the ALB still bill continuously.

**Task statement:** 4.2 · **Review:** [Cost optimization: serverless vs always-on compute](../notes/10-cost-optimization.md)
</details>

---

### Q9. Rightsizing across an organization
A company has about 800 EC2 instances across 40 accounts in AWS Organizations. Many instances were sized years ago, and the company suspects that many of them are overprovisioned. The company wants recommendations for better-fitting instance types based on actual utilization history, across all accounts, with the LEAST effort.

Which solution meets these requirements?

- A. Deploy the AWS Config managed rule that checks for desired instance types in every account, and review the noncompliant instances.
- B. Opt in to AWS Compute Optimizer for all accounts in the organization, install the Amazon CloudWatch agent to publish memory utilization, and review the instance recommendations.
- C. Build CloudWatch dashboards of CPU utilization in each account, and have engineers choose new instance types manually.
- D. Review the Savings Plans purchase recommendations in AWS Cost Explorer, and buy a Savings Plan that covers the current instances.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Compute Optimizer** analyzes utilization history with machine learning and recommends better-fitting configurations for EC2 instances, Auto Scaling groups, EBS volumes, Lambda functions, and more. You can opt in the **whole organization** from the management account, and then manage recommendations from a delegated administrator account. EC2 doesn't report memory by default, so publishing memory metrics with the **CloudWatch agent** makes the recommendations much more accurate. Cost Optimization Hub can bring these recommendations together with other savings opportunities.

**Why not the others:**
- **A**: That Config rule checks instances against an approved list of types. It says nothing about whether an instance is the right size.
- **C**: Doing this by hand across 40 accounts is a lot of effort, and CPU alone is incomplete data.
- **D**: ⚠️ Trap: Committing before rightsizing **locks in spending on oversized instances**. Rightsize first, then commit.

**Task statement:** 4.2 · **Review:** [Cost optimization: Compute Optimizer](../notes/10-cost-optimization.md)
</details>

---

### Q10. Alerts and automatic guardrails on spend
A company gives its developers a sandbox AWS account for experiments. Finance wants the team lead to receive an email when the forecasted spend for the month exceeds $2,000. If actual spend exceeds $2,500, developers must be prevented automatically from launching new EC2 instances until the finance team reviews the spend. Nobody should have to write custom code.

Which solution meets these requirements?

- A. Create an Amazon CloudWatch billing alarm on the `EstimatedCharges` metric, and invoke an AWS Lambda function that terminates all EC2 instances in the account.
- B. Turn on AWS Cost Anomaly Detection with an email subscription, and set the alert threshold to $2,500.
- C. Schedule a daily AWS Cost Explorer report to the team lead, and have the team lead remove developers' permissions manually when spend exceeds $2,500.
- D. Create an AWS Budgets cost budget with a forecasted-spend alert at $2,000 that emails the team lead, and a budget action at $2,500 of actual spend that applies an SCP or IAM policy denying `ec2:RunInstances`.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **AWS Budgets** alerts on **actual or forecasted** spend, by email or through Amazon SNS. **Budget actions** can apply an IAM policy or an SCP, or stop specific EC2 or RDS instances, when a threshold is crossed. They can run automatically or wait for approval. After finance reviews the spend, an administrator reverses the action. None of this requires code.

**Why not the others:**
- **A**: Billing alarms can only use **actual** charges (no forecast), the Lambda function is custom code, and terminating every instance is destructive.
- **B**: Anomaly Detection uses machine learning to flag **unusual** spending patterns. It isn't designed for fixed budget thresholds, and it can't enforce anything.
- **C**: This is manual, slow, and enforces nothing automatically.

**Task statement:** 4.2 · **Review:** [Cost optimization: AWS Budgets](../notes/10-cost-optimization.md)
</details>

---

### Q11. Chargeback by cost center
The management account of a company's organization in AWS Organizations receives a consolidated bill for 25 member accounts. Several product teams share some of these accounts. Every team already tags its resources with a `CostCenter` tag, but the finance team can't see costs by `CostCenter` in any billing report. Finance needs a monthly chargeback report of costs grouped by `CostCenter`.

Which combination of steps meets these requirements? **(Choose two.)**

- A. Create a tag policy in AWS Organizations that requires the `CostCenter` tag on all resources.
- B. Turn on AWS Config in every member account to record resource tags.
- C. Turn on AWS CloudTrail data events so that every API call on tagged resources is logged.
- D. Activate `CostCenter` as a user-defined cost allocation tag in the Billing and Cost Management console of the management account.
- E. Use AWS Cost Explorer to group and filter costs by the `CostCenter` tag, and save the view as a report that finance reviews monthly.

<details>
<summary>Show answer</summary>

**Answer: D, E**

**Why D and E:** Resource tags appear in billing data only after they're **activated as cost allocation tags**. In an organization, the **management account** does this. Activated tags can take up to 24 hours to appear. Then **Cost Explorer** can group and filter costs by the tag, and you can save that view as a report. For line-item detail, use the Cost and Usage Report or Data Exports. For mapping rules that are more complex than one tag, use Cost Categories.

**Why not the others:**
- **A**: Tag policies standardize tag keys and values, which helps keep the data clean, but they don't make tags appear in billing data.
- **B**: Config records resource configuration. It has no cost data.
- **C**: CloudTrail records API activity, not costs.

**Task statement:** 4.2 · **Review:** [Cost optimization: cost allocation tags and Cost Explorer](../notes/10-cost-optimization.md)
</details>

---

### Q12. DynamoDB capacity mode for predictable traffic
An Amazon DynamoDB table backs an internal order-tracking application that has run for 2 years. Traffic follows a predictable daily curve: it rises gradually in the morning, stays steady during business hours, and drops to a low level overnight. There are no sudden spikes. The table uses on-demand capacity mode, and the company wants to lower its DynamoDB bill without affecting performance.

Which solution is the MOST cost-effective?

- A. Keep on-demand capacity mode, because it always costs less than provisioned capacity for workloads with daily variation.
- B. Switch the table to provisioned capacity mode with auto scaling based on target utilization, and buy reserved capacity for the steady overnight baseline.
- C. Change the table class to DynamoDB Standard-Infrequent Access (Standard-IA), and keep on-demand capacity mode.
- D. Put a DynamoDB Accelerator (DAX) cluster in front of the table to reduce the number of read requests.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** For **predictable, gradually changing** traffic, **provisioned capacity with auto scaling** follows the curve at high utilization and costs less per request than on-demand. **Reserved capacity** (a 1- or 3-year commitment) discounts the baseline further. On-demand is the better fit for new, unknown, very spiky, or mostly idle tables. AWS cut on-demand prices significantly in late 2024, which narrowed the gap, so model both modes with your own numbers.

**Why not the others:**
- **A**: That claim is false. On-demand is simpler but generally costs more for steady, well-understood traffic.
- **C**: Standard-IA lowers **storage** cost and raises **request** cost. For a table whose bill is mostly throughput, it increases the total.
- **D**: DAX adds node-hour charges and only helps read-heavy, cache-friendly workloads. It does nothing for writes.

**Task statement:** 4.3 · **Review:** [Databases: DynamoDB capacity modes](../notes/05-databases.md)
</details>

---

### Q13. Intermittent dev and test databases
A development team uses 15 MySQL-compatible databases for development and testing. Each database is busy for a few hours at irregular times during the workday, with occasional heavy bursts during load tests, and is idle on most evenings and weekends. The databases currently run on provisioned Amazon RDS for MySQL instances that are sized for the load tests. The team wants to reduce cost and administrative effort.

Which solution meets these requirements MOST cost-effectively?

- A. Move the databases to Amazon Aurora Serverless v2 (MySQL-compatible) with a low minimum capacity, using auto-pause to 0 ACUs where the engine version supports it, and a maximum capacity sized for the load tests.
- B. Move the databases to Multi-AZ RDS for MySQL deployments on smaller instance classes.
- C. Move the databases to provisioned Amazon Aurora MySQL instances, and purchase 1-year Reserved Instances for them.
- D. Keep the RDS for MySQL instances, and use a scheduled AWS Lambda function to stop them at 7 PM and start them at 7 AM on weekdays.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Aurora Serverless v2** scales capacity in fine-grained ACU steps within seconds, and you pay per ACU-second actually used. A low minimum (or **auto-pause to 0 ACUs** on supported versions) removes most of the idle cost, and the maximum covers the load-test bursts. The trade-off of auto-pause is that the first connection after a pause waits several seconds while the database resumes, which is fine for dev and test.

**Why not the others:**
- **B**: Multi-AZ roughly doubles the instance cost, which dev and test rarely need. The instances still run all the time, and smaller classes may fail the load tests.
- **C**: RIs commit you to paying for always-on capacity that sits idle most of the week.
- **D**: A fixed schedule doesn't match irregular use, the instances stay sized for peak load, and RDS automatically restarts an instance that has been stopped for 7 days.

**Task statement:** 4.3 · **Review:** [Databases: Aurora Serverless v2](../notes/05-databases.md)
</details>

---

### Q14. NAT gateway charges for AWS service traffic
Data processing jobs on EC2 instances in private subnets read several terabytes per day from Amazon S3 and write results to an Amazon DynamoDB table in the same AWS Region. The instances reach both services through a NAT gateway. Cost Explorer shows that NAT gateway data processing charges are now one of the largest items on the bill. The company wants to reduce this cost with minimal changes and without exposing the instances to the internet.

Which combination of actions meets these requirements MOST cost-effectively? **(Choose two.)**

- A. Replace the NAT gateway with a NAT instance on a large EC2 instance type.
- B. Create a gateway VPC endpoint for Amazon S3, and associate it with the private subnets' route tables.
- C. Create interface VPC endpoints for Amazon S3 and Amazon DynamoDB in each Availability Zone used by the private subnets.
- D. Move the instances to public subnets, and assign them public IPv4 addresses.
- E. Create a gateway VPC endpoint for Amazon DynamoDB, and associate it with the private subnets' route tables.

<details>
<summary>Show answer</summary>

**Answer: B, E**

**Why B and E:** **S3 and DynamoDB are the only services with gateway endpoints**, and gateway endpoints have **no hourly or per-GB charge**. They add prefix-list routes to the route tables, so traffic to those services bypasses the NAT gateway with no application changes. Gateway endpoints only serve traffic that starts inside the VPC in the same Region, which is the case here.

**Why not the others:**
- **A**: A NAT instance avoids the per-GB NAT gateway charge, but you take on patching, scaling, and failover, it's a single point of failure, and its bandwidth is limited by the instance size. Free gateway endpoints are cheaper and simpler.
- **C**: Interface endpoints keep traffic private too, but they're billed per hour per AZ and per GB processed. Use them when you need on-premises or cross-Region access.
- **D**: This exposes the instances to the internet, and public IPv4 addresses are billed hourly.

**Task statement:** 4.4 · **Review:** [Networking & VPC: VPC endpoints and NAT gateway cost](../notes/02-networking-vpc.md)
</details>

---

### Q15. Reducing S3 data transfer out for global downloads
A software company distributes installer files from an Amazon S3 bucket directly to customers worldwide through public object URLs. Data transfer out from S3 to the internet has become a large part of the bill, and customers far from the bucket's Region complain about slow downloads. Downloads are anonymous.

Which solution will reduce cost and improve download speed?

- A. Enable S3 Transfer Acceleration on the bucket, and publish the accelerate endpoint URLs to customers.
- B. Replicate the bucket to four other Regions with Cross-Region Replication, and use Route 53 latency-based routing to send customers to the nearest bucket.
- C. Create an Amazon CloudFront distribution with the bucket as its origin, restrict the bucket with origin access control (OAC), and publish CloudFront URLs.
- D. Enable Requester Pays on the bucket so that customers pay for their own downloads.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** Data transfer from S3 **to CloudFront is free**. Caching at edge locations cuts the number of S3 GET requests. CloudFront's always-free tier covers the first 1 TB of transfer out each month, and its per-GB rates for North America and Europe are lower than S3's, with volume discounts. Users also download from a nearby edge location, which is faster. **Price classes** can limit which edge locations are used if you need to control cost further.

**Why not the others:**
- **A**: Transfer Acceleration adds a per-GB fee **on top of** normal transfer charges and caches nothing.
- **B**: You'd pay for five copies of the data and for replication, and each bucket still pays normal internet transfer rates.
- **D**: ⚠️ Trap: Requester Pays **doesn't work with anonymous requests**. Requesters must be authenticated AWS accounts.

**Task statement:** 4.4 · **Review:** [Cost optimization: data transfer and CloudFront](../notes/10-cost-optimization.md)
</details>

---

## Answer summary

<details>
<summary>Show all answers</summary>

| Q | Answer | Topic | Task |
|---|---|---|---|
| 1 | D | Lifecycle: Glacier IR → Deep Archive → expire | 4.1 |
| 2 | B | S3 Intelligent-Tiering for unknown patterns | 4.1 |
| 3 | C | S3 Requester Pays | 4.1 |
| 4 | A | gp2 → gp3 with Elastic Volumes | 4.1 |
| 5 | D | Tape Gateway → Deep Archive pool | 4.1 |
| 6 | A, C | Spot + diversified, price-capacity-optimized | 4.2 |
| 7 | A | Compute Savings Plans (EC2 + Fargate + Lambda) | 4.2 |
| 8 | C | API Gateway + Lambda for a low-traffic API | 4.2 |
| 9 | B | Compute Optimizer for rightsizing | 4.2 |
| 10 | D | AWS Budgets alerts + budget actions | 4.2 |
| 11 | D, E | Activate cost allocation tags + Cost Explorer | 4.2 |
| 12 | B | DynamoDB provisioned + auto scaling + reserved | 4.3 |
| 13 | A | Aurora Serverless v2 for intermittent dev/test | 4.3 |
| 14 | B, E | Gateway endpoints for S3 and DynamoDB | 4.4 |
| 15 | C | CloudFront to cut S3 data transfer out | 4.4 |

</details>

**Scoring:** 13+/15 strong · 10–12 review misses · <10 re-read the module
