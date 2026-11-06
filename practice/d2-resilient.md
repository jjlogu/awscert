# Domain 2 practice: Design Resilient Architectures (15 questions)
> [Home](../README.md) · [Practice index](README.md) · Notes: [Compute & containers](../notes/03-compute-containers.md) · [Databases](../notes/05-databases.md) · [Decoupling & serverless](../notes/06-decoupling-serverless.md) · [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md)

Untimed. Read each explanation, including why the wrong options are wrong. Log misses in your error log.

---

### Q1. Decoupling an order-processing tier
An e-commerce company runs a web tier and an order-processing tier in separate Auto Scaling groups of EC2 instances. The web tier calls the order-processing tier synchronously over HTTP. During flash sales the processing tier can't keep up, requests time out, and some orders are lost. Orders don't need to be processed in a strict sequence, and the processing code is idempotent.

Which solution will prevent lost orders and let each tier scale independently?

- A. Replace the synchronous call with an Amazon SNS topic, and subscribe the processing tier's HTTP endpoint to the topic.
- B. Have the web tier send each order to an Amazon SQS standard queue. Have the processing tier poll the queue, and scale the processing Auto Scaling group on the queue backlog per instance.
- C. Move the processing tier to larger instance types, and set the Auto Scaling group's minimum capacity to the peak capacity seen during the last flash sale.
- D. Put a Network Load Balancer in front of the processing tier, and turn on cross-zone load balancing to spread requests more evenly.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** An **SQS queue** is a durable buffer. The web tier's write succeeds even when consumers are saturated, and messages wait (up to 14 days with a longer retention setting) until a consumer processes and deletes them. The consumers **pull at their own pace**. Scale them with target tracking on a custom **backlog per instance** metric (`ApproximateNumberOfMessagesVisible` ÷ running instances). Add a **dead-letter queue** for messages that keep failing. A standard queue delivers **at least once** with best-effort ordering, which is fine because the code is idempotent. Use FIFO when you need strict ordering or deduplication.

**Why not the others:**
- **A**: SNS **pushes** messages. It retries HTTP endpoints according to its delivery policy but doesn't hold a backlog for a slow consumer, so the overloaded tier still drops work.
- **C**: This is over-provisioning. It costs more, the tiers stay tightly coupled, and the next sale's peak is unknown.
- **D**: A load balancer still makes the call synchronous. It spreads load but buffers nothing.

**Task statement:** 2.1 · **Review:** [Decoupling & serverless: SQS](../notes/06-decoupling-serverless.md)
</details>

---

### Q2. One event, many independent consumers
When a customer places an order, an order service must notify three independent systems: inventory, shipping, and analytics. Each system must receive every order, process it at its own pace, and retry failures without affecting the others. The company expects to add more consuming systems later and doesn't want to change the order service when it does.

Which architecture meets these requirements?

- A. Have the order service send each order to a single Amazon SQS queue that all three systems poll.
- B. Have the order service send each order to three separate Amazon SQS queues, one per system.
- C. Have the order service start an AWS Step Functions workflow that calls the three systems in a Parallel state.
- D. Have the order service publish each order to an Amazon SNS topic, and subscribe a separate Amazon SQS queue for each system to the topic.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** This is the **SNS → SQS fan-out** pattern. The producer publishes once. Each consumer gets its **own durable queue**, so it processes at its own pace and handles retries and its own DLQ independently. Adding a consumer means adding a subscription, with no change to the producer. Use **subscription filter policies** if a consumer only needs some messages. Each queue's access policy must allow the topic to send messages, and if the queue uses a KMS key, the key policy must allow SNS too. EventBridge with one rule per consumer is the other common way to do this.

**Why not the others:**
- **A**: Consumers of one queue **compete**: each message goes to only one of them, so no system would see every order.
- **B**: This works today, but the producer is coupled to every consumer. Each new system means a code change in the order service.
- **C**: Orchestration ties the systems together. One failing branch fails the Parallel state unless you handle it, and adding a consumer means editing the workflow.

**Task statement:** 2.1 · **Review:** [Decoupling & serverless: SNS fan-out](../notes/06-decoupling-serverless.md)
</details>

---

### Q3. Workflow with retries and a human approval step
A bank is automating loan applications. The workflow validates documents with an AWS Lambda function, then calls an external credit bureau API that sometimes times out. For loans above a threshold, it then waits for a loan officer to approve or reject the application, which can take up to 5 days. The bank wants built-in retries with exponential backoff, a visual history of each execution, and the LEAST amount of custom code.

Which solution meets these requirements?

- A. Build an AWS Step Functions Standard workflow. Add a Retry policy with backoff to the credit check task, a Choice state for the threshold, and a task that uses the wait-for-callback (task token) pattern to pause until the officer responds.
- B. Build an AWS Step Functions Express workflow with the same states, and use the wait-for-callback (task token) pattern for the approval step.
- C. Chain Lambda functions with Amazon SQS queues between them. For approval, have a Lambda function poll an Amazon DynamoDB table in a loop until the officer's decision is recorded.
- D. Have each Lambda function invoke the next one asynchronously, and use Amazon EventBridge Scheduler to check for the officer's decision every hour.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Standard workflows** can run for up to **1 year**, keep a full visual execution history, and have exactly-once execution semantics. `Retry` (with `IntervalSeconds`, `BackoffRate`, and `MaxAttempts`) and `Catch` are declarative, so you don't write retry code. The **`.waitForTaskToken`** callback pattern pauses the execution and sends a token to the approver, for example through SNS email or an internal app. The officer's action calls `SendTaskSuccess` or `SendTaskFailure` with that token, and the workflow continues.

**Why not the others:**
- **B**: ⚠️ Trap: **Express** workflows run for at most **5 minutes** and **don't support** the `.waitForTaskToken` callback pattern. They suit high-volume, short event processing.
- **C**: A Lambda function runs for at most 15 minutes, so a loop can't wait 5 days. It would also cost money while waiting and is all custom code.
- **D**: This is hand-written orchestration: you'd have to build retries, state tracking, and execution history yourself.

**Task statement:** 2.1 · **Review:** [Decoupling & serverless: Step Functions](../notes/06-decoupling-serverless.md)
</details>

---

### Q4. Protecting Lambda concurrency from a bursty caller
A company exposes a REST API through Amazon API Gateway that invokes an AWS Lambda function, which reads from Amazon DynamoDB. The API must return results synchronously. One partner sometimes sends large bursts of requests that use up most of the account's Lambda concurrency. During these bursts, other business-critical Lambda functions in the same account and Region are throttled.

Which solution meets these requirements with the LEAST operational overhead?

- A. Configure provisioned concurrency on the API's Lambda function equal to the account's concurrency quota, so that the function never throttles.
- B. Change the integration so that API Gateway sends requests to an Amazon SQS queue, and have the Lambda function process messages from the queue.
- C. Request a higher Lambda concurrency quota for the account and Region, and make no other changes.
- D. Create an API Gateway usage plan with an API key for the partner that sets throttling limits, and set reserved concurrency on the API's Lambda function to cap how much of the account's concurrency it can use.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** Fix it at two layers. A **usage plan** sets per-client **rate and burst limits** (and quotas if needed), so the partner gets HTTP 429 responses instead of flooding the backend. **Reserved concurrency** both **caps** the API function and **guarantees** it that capacity, so it can never take the account pool away from other functions. You can also reserve concurrency for the critical functions. API keys identify clients for usage plans; they aren't an authentication mechanism.

**Why not the others:**
- **A**: Provisioned concurrency draws from the same account pool. Allocating the whole quota would starve every other function and cost a lot.
- **B**: Queueing is a good pattern for spiky **asynchronous** work, but this API must answer synchronously.
- **C**: A higher ceiling doesn't isolate anything. The partner's bursts can still consume it.

**Task statement:** 2.1 · **Review:** [Decoupling & serverless: API Gateway throttling and Lambda concurrency](../notes/06-decoupling-serverless.md)
</details>

---

### Q5. Reacting to events from a SaaS partner
A company uses a SaaS help desk platform that is an Amazon EventBridge partner. When a ticket is created with the priority set to "urgent", the company wants to invoke an AWS Lambda function that pages the on-call engineer, and also send the ticket details to an Amazon SQS queue for a reporting system. The solution should react within seconds, avoid polling the SaaS API, and need the LEAST custom code.

Which solution meets these requirements?

- A. Schedule a Lambda function to call the SaaS platform's REST API every minute, filter for urgent tickets, and then invoke the paging function and send a message to the queue.
- B. Set up the SaaS partner event source in EventBridge and associate it with a partner event bus. Create a rule with an event pattern that matches urgent tickets, with the Lambda function and the SQS queue as targets.
- C. Create an Amazon AppFlow flow that runs every hour to copy new tickets to Amazon S3, and use S3 event notifications to invoke the Lambda function.
- D. Create an Amazon Kinesis data stream, give the SaaS vendor an IAM access key to write ticket events to it, and have the Lambda function consume the stream.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** EventBridge **partner event sources** deliver the SaaS provider's events into a **partner event bus** in your account, with no polling and no integration code. A **rule** filters on event content (here, `priority = urgent`) and sends matching events to several **targets** at once, with built-in retries and an optional DLQ. For outbound calls to third-party HTTP APIs, EventBridge **API destinations** are the matching feature.

**Why not the others:**
- **A**: Polling is custom code, adds up to a minute of delay, and uses up the SaaS API's rate limits.
- **C**: An hourly flow misses the "within seconds" requirement.
- **D**: This means a custom integration on the vendor's side and a long-term AWS credential handed to a third party.

**Task statement:** 2.1 · **Review:** [Decoupling & serverless: EventBridge](../notes/06-decoupling-serverless.md)
</details>

---

### Q6. Aurora read scaling and fast failover
A company runs an Amazon Aurora MySQL DB cluster with a single writer instance. A new reporting tool runs heavy read queries that push the writer's CPU above 90%, slowing down the application's writes. The company also wants the database to fail over automatically and quickly if the writer instance or its Availability Zone fails.

Which combination of actions meets these requirements? **(Choose two.)**

- A. Add two Aurora Replicas in two Availability Zones other than the writer's.
- B. Enable a Multi-AZ standby for the writer instance, and point the reporting tool at the standby.
- C. Scale the writer instance up to the next larger instance class.
- D. Point the reporting tool at the cluster endpoint, so that Aurora balances its queries across instances.
- E. Point the reporting tool at the cluster's reader endpoint.

<details>
<summary>Show answer</summary>

**Answer: A, E**

**Why A and E:** **Aurora Replicas** share the cluster's storage volume, which already keeps six copies across three AZs. They serve reads with typically low replica lag and **are also the failover targets**. Aurora promotes one according to its failover priority tier, which is much faster than recreating a writer when no replica exists. The **reader endpoint** spreads connections across the replicas. A **custom endpoint** can pin reporting to specific replicas so that it doesn't affect application reads.

**Why not the others:**
- **B**: Aurora doesn't use a separate "standby": the replicas fill that role. In RDS (non-Aurora) Multi-AZ *instance* deployments, the standby serves **no** reads. Readable standbys exist only in RDS Multi-AZ *DB cluster* deployments, which aren't Aurora.
- **C**: A bigger writer buys some headroom but adds no failover target and still mixes reporting with OLTP.
- **D**: ⚠️ Trap: The **cluster endpoint always points to the writer**, so this changes nothing.

**Task statement:** 2.2 · **Review:** [Databases: Aurora replicas and endpoints](../notes/05-databases.md)
</details>

---

### Q7. Lambda exhausting database connections
An API built on Amazon API Gateway and AWS Lambda reads and writes data in an Amazon RDS for MySQL DB instance. During traffic spikes, thousands of concurrent Lambda invocations open new database connections, and the database returns "too many connections" errors. The data must stay in RDS for MySQL, and the team wants to keep code changes to a minimum.

Which solution meets these requirements?

- A. Scale the DB instance up to a larger instance class to raise its `max_connections` limit.
- B. Add an RDS read replica, and send half of the Lambda functions' connections to the replica.
- C. Create an RDS Proxy for the DB instance, and change the Lambda functions' connection string to the proxy endpoint.
- D. Add an Amazon ElastiCache cluster in front of the database to cache query results.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** **RDS Proxy** keeps a warm **pool** of database connections and **multiplexes** many client connections over it, so connection storms from Lambda don't reach the database. It can take credentials from Secrets Manager or use IAM authentication, and it shortens failover time. The only code change is the endpoint. Also open the connection **outside the handler** so that each execution environment reuses it.

**Why not the others:**
- **A**: `max_connections` grows with instance memory, so this buys some headroom at a higher cost, but the next spike exceeds it again.
- **B**: Writes still have to go to the primary, and replicas lag. Splitting connections doesn't stop the churn of opening and closing them.
- **D**: Caching only helps reads, and only after you write cache logic (which means code changes). Writes still need database connections.

**Task statement:** 2.1 · **Review:** [Databases: RDS Proxy](../notes/05-databases.md)
</details>

---

### Q8. Multi-Region active-active data
A gaming company runs its player profile service in us-east-1, eu-west-1, and ap-southeast-1. Players must be able to read and write their profiles with low latency in their nearest Region. If one Region becomes unavailable, players must be able to keep reading and writing in the remaining Regions. The company wants the LEAST operational overhead.

Which solution meets these requirements?

- A. Use an Amazon Aurora global database with the primary cluster in us-east-1, and turn on write forwarding in the secondary clusters.
- B. Use an Amazon DynamoDB table in us-east-1, and deploy a DynamoDB Accelerator (DAX) cluster in each of the other Regions.
- C. Create an Amazon DynamoDB table in each Region, and use DynamoDB Streams with AWS Lambda functions to copy every change to the other two tables.
- D. Create an Amazon DynamoDB global table with replicas in all three Regions, and have each Regional deployment use its local replica.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **Global tables** are **multi-active**: every replica accepts reads and writes, and AWS replicates changes between Regions asynchronously, typically within about a second. Conflicts resolve as last writer wins. If a Region fails, the other replicas keep working without a failover step. A newer multi-Region strong consistency mode exists for workloads that can't accept eventual consistency, at the cost of higher write latency.

**Why not the others:**
- **A**: An Aurora global database has **one writer Region**. Write forwarding sends writes to the primary, which adds cross-Region latency, and if us-east-1 fails, writes stop until you fail over.
- **B**: DAX is an in-VPC cache in front of a table in the same Region. It doesn't replicate data across Regions.
- **C**: You'd be building replication yourself, including loop prevention and conflict handling. That is what global tables manage for you.

**Task statement:** 2.2 · **Review:** [Databases: DynamoDB global tables](../notes/05-databases.md)
</details>

---

### Q9. Choosing a DR strategy
A company runs a revenue-critical application in us-east-1 on EC2 instances in an Auto Scaling group behind an Application Load Balancer, with an Amazon Aurora PostgreSQL database. The DR requirements are an RPO of under 1 minute and an RTO of 10 minutes. After a failover, the DR Region must serve production traffic immediately, even at reduced capacity, while it scales out. The budget doesn't allow running full production capacity in a second Region.

Which DR strategy meets these requirements MOST cost-effectively?

- A. Backup and restore: copy AWS Backup recovery points of the database and the AMIs to us-west-2 every hour, and rebuild the stack with AWS CloudFormation during a disaster.
- B. Pilot light: replicate the database to us-west-2 with an Aurora global database, and keep AMIs and CloudFormation templates ready to launch the application tier during a disaster.
- C. Warm standby: replicate the database to us-west-2 with an Aurora global database, and run a scaled-down copy of the application tier there that scales out during a disaster.
- D. Multi-site active/active: run full production capacity in both Regions with an Aurora global database, and route users to both Regions with Route 53 latency-based routing.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** An Aurora global database typically replicates with **under a second** of lag, which meets the RPO. A **warm standby** is a smaller but **fully working** copy of the stack that is already running. It can take traffic as soon as you promote the secondary cluster and switch DNS, and then it scales out. That fits an RTO of minutes and the requirement to serve traffic immediately. The usual summary: backup and restore takes **hours**, pilot light **tens of minutes**, warm standby **minutes**, and active/active **near zero**.

**Why not the others:**
- **A**: Hourly copies mean an RPO of up to an hour, and rebuilding the stack takes hours.
- **B**: The data is ready, but the application tier isn't running. Launching, warming up, and configuring it puts the RTO at risk, and the DR site can't serve traffic "immediately".
- **D**: This meets the objectives but runs double capacity, which the budget rules out. Aurora global databases also have a single writer Region, so writes need extra routing design.

**Task statement:** 2.2 · **Review:** [HA, DR, edge & ops: DR strategies](../notes/08-ha-dr-edge-ops.md)
</details>

---

### Q10. DNS failover to a static site
A company's website runs on an Application Load Balancer in us-east-1. A static "we'll be right back" version of the site is hosted in an Amazon S3 bucket configured for static website hosting in us-west-2. All users must reach the ALB while it is healthy and be sent to the static site automatically when the ALB's targets are unhealthy. The domain is hosted in Amazon Route 53.

Which configuration meets these requirements?

- A. Create one simple routing record with two values, the ALB and the S3 website endpoint, and rely on clients to retry the second value if the first one fails.
- B. Create failover routing records: a primary alias record for the ALB with Evaluate Target Health set to Yes, and a secondary alias record for the S3 website endpoint.
- C. Create latency-based routing records for the ALB and the S3 website endpoint, each associated with its own Region.
- D. Create geolocation routing records that send North American users to the ALB, and a default record that sends all other users to the S3 website endpoint.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Failover routing** is active-passive. Route 53 answers with the primary record while it is healthy and with the secondary record when it isn't. For an alias record that points to an ALB, **Evaluate Target Health** treats the ALB as unhealthy when it has no healthy targets, so no separate health check is needed. For an S3 website alias, the bucket name must match the record name. The website endpoint is HTTP only, so put CloudFront in front of it if you need HTTPS.

**Why not the others:**
- **A**: Simple routing has no health checks, and an alias record can't hold two targets anyway.
- **C**: Latency routing sends users to whichever Region is closer, so West Coast users could land on the static site while the ALB is perfectly healthy.
- **D**: Geolocation routing chooses by user location, not by health. Most users outside North America would always get the static site.

**Task statement:** 2.2 · **Review:** [HA, DR, edge & ops: Route 53 routing policies](../notes/08-ha-dr-edge-ops.md)
</details>

---

### Q11. Self-healing across Availability Zones
A web application runs on EC2 instances in an Auto Scaling group that uses subnets in a single Availability Zone. The instances are registered with an Application Load Balancer target group, and the Auto Scaling group uses the default EC2 health checks. Last week an application process hung: the instance kept passing EC2 status checks but returned HTTP 500 errors, and it wasn't replaced. The company also wants the application to survive the loss of an Availability Zone.

Which combination of changes meets these requirements? **(Choose two.)**

- A. Increase the Auto Scaling group's desired capacity so that more instances run in the current Availability Zone.
- B. Add subnets in two more Availability Zones to the Auto Scaling group, and make sure the ALB is enabled in the same three Availability Zones.
- C. Turn on detailed monitoring for the instances so that Amazon CloudWatch reports their metrics every minute.
- D. Turn on Elastic Load Balancing health checks for the Auto Scaling group, so that instances that fail the target group's health check are replaced.
- E. Launch the instances into a cluster placement group to improve network performance between them.

<details>
<summary>Show answer</summary>

**Answer: B, D**

**Why B and D:** Across several AZs, the Auto Scaling group spreads instances evenly and replaces capacity in the remaining AZs if one fails. Size the group so that losing one AZ still leaves enough capacity (with three AZs, two-thirds remain). Adding the **ELB health check type** makes the group treat instances that fail the target group's health check as unhealthy too, so it terminates and replaces them. Set a sensible **grace period**, and choose a health check path that actually exercises the application.

**Why not the others:**
- **A**: More instances in the same AZ still fail together with that AZ, and the hung instance still wouldn't be replaced.
- **C**: Detailed monitoring gives more frequent metrics but replaces nothing.
- **E**: A cluster placement group keeps all instances in **one AZ**, which works against the resilience goal.

**Task statement:** 2.2 · **Review:** [Compute & containers: Auto Scaling health checks](../notes/03-compute-containers.md)
</details>

---

### Q12. Monolith to containers without servers
A company runs a Java monolith on self-managed EC2 instances and plans to split it into about ten containerized microservices. The operations team doesn't want to patch operating systems or manage cluster capacity. Each microservice must scale independently and be reachable through a single public endpoint that uses path-based routing.

Which solution meets these requirements with the LEAST operational overhead?

- A. Push the images to Amazon ECR, and run each microservice as an Amazon ECS service on AWS Fargate behind an Application Load Balancer with path-based listener rules. Configure ECS Service Auto Scaling for each service.
- B. Push the images to Amazon ECR, and run each microservice as an Amazon ECS service on the EC2 launch type with an Auto Scaling group capacity provider, behind an Application Load Balancer.
- C. Run the microservices on an Amazon EKS cluster with self-managed node groups, and expose them through a Kubernetes service backed by a Network Load Balancer.
- D. Deploy all of the containers to a single AWS Elastic Beanstalk Docker environment behind an Application Load Balancer.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Fargate** runs containers without instances, so there are no AMIs to patch and no cluster capacity to manage. Each microservice is its own **ECS service** with its own task definition and **Service Auto Scaling** (for example, target tracking on CPU or on ALB requests per target). One **ALB** routes `/orders/*`, `/users/*`, and other paths to each service's target group, using IP targets with `awsvpc` networking.

**Why not the others:**
- **B**: With the EC2 launch type, you manage the container instances yourself, including patching and scaling them.
- **C**: You'd operate Kubernetes, and self-managed nodes need patching. An NLB works at Layer 4 and can't do path-based routing.
- **D**: One environment scales as one unit, so the services can't scale independently.

**Task statement:** 2.1 · **Review:** [Compute & containers: ECS, EKS and Fargate](../notes/03-compute-containers.md)
</details>

---

### Q13. Shared file storage for a CMS
A company runs a content management system on Linux EC2 instances in an Auto Scaling group across two Availability Zones. Editors upload images and documents through the CMS, which saves them to the local file system. Because each upload lands on only one instance, users often see broken images. The application needs a shared POSIX file system that all instances can mount at the same time, that survives the loss of an Availability Zone, and that grows automatically.

Which solution meets these requirements?

- A. Attach a single io2 EBS volume with Multi-Attach enabled to all instances in the Auto Scaling group.
- B. Mount an Amazon S3 bucket on each instance with Mountpoint for Amazon S3, and point the CMS upload directory to it.
- C. Create a Regional (not One Zone) Amazon EFS file system with a mount target in each Availability Zone, and mount it on every instance at launch.
- D. Create an Amazon FSx for Windows File Server file system, and map it as a network drive on each instance.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** **EFS** is a managed NFS file system with POSIX semantics. A **Regional** file system stores data redundantly across multiple AZs; One Zone file systems keep it in a single AZ. Instances in each AZ mount it through that AZ's **mount target**, and capacity grows and shrinks automatically. Mount it from the launch template's user data (or `/etc/fstab` with the EFS mount helper) so that new instances get it automatically.

**Why not the others:**
- **A**: Multi-Attach works only within **one AZ**, for a limited number of Nitro instances, and it needs a cluster-aware file system. Instances in the second AZ can't attach the volume.
- **B**: Mountpoint presents objects as files but doesn't provide full POSIX semantics. Editing files in place and file locking, for example, aren't supported, which a CMS may rely on.
- **D**: FSx for Windows File Server serves SMB for Windows workloads. For Linux with POSIX semantics, EFS is the native fit.

**Task statement:** 2.2 · **Review:** [Storage: Amazon EFS](../notes/04-storage.md)
</details>

---

### Q14. High availability for a legacy app with no code changes
A company runs a legacy line-of-business web application on a single EC2 instance. The application keeps user sessions in memory and writes generated reports to a local directory, `/data/reports`, from which users download them later. The vendor no longer exists, so the code can't be changed. The company needs the application to keep serving users if an instance or an Availability Zone fails. Users having to sign in again after a failure is acceptable.

Which combination of steps meets these requirements? **(Choose two.)**

- A. Modify the application to store sessions in Amazon ElastiCache, so that any instance can serve any user.
- B. Use Amazon Data Lifecycle Manager to create an AMI of the instance every hour, and launch a new instance from the latest AMI in another Availability Zone if the instance fails.
- C. Run the application on multiple instances in an Auto Scaling group across two Availability Zones, behind an Application Load Balancer with sticky sessions turned on.
- D. Move the application to a larger instance type, and turn on EC2 automatic recovery for the instance.
- E. Create an Amazon EFS file system, and mount it at `/data/reports` on every instance.

<details>
<summary>Show answer</summary>

**Answer: C, E**

**Why C and E:** **Sticky sessions** (a load balancer-generated cookie) keep each user on the instance that holds their in-memory session, which works without touching the code. The **Auto Scaling group across two AZs** replaces failed instances and survives the loss of an AZ. Users on a failed instance sign in again, which the business accepts. Mounting **EFS at the same path** makes every report visible from every instance. This is purely an infrastructure change, done in the launch template's user data.

**Why not the others:**
- **A**: This is a code change, which isn't possible here. (Without that constraint, it would be the better design.)
- **B**: This is a manual recovery process. It loses data written since the last AMI, and you have to notice the failure and launch the replacement yourself.
- **D**: It's still one instance in one AZ. Auto recovery handles underlying host failures, not an AZ outage or a hung application.

**Task statement:** 2.2 · **Review:** [Compute & containers: ALB, Auto Scaling and sticky sessions](../notes/03-compute-containers.md)
</details>

---

### Q15. Replicating S3 objects for DR
A media company stores critical assets in an Amazon S3 bucket in us-east-1. For DR, it needs a copy of every object in a bucket in us-west-2. New objects must be replicated within 15 minutes under an SLA, and the operations team needs metrics and alerts when replication falls behind. About 200 million objects already exist in the source bucket and must be copied too.

Which solution meets these requirements?

- A. Enable versioning on both buckets, configure S3 Cross-Region Replication with S3 Replication Time Control (S3 RTC), and use S3 Batch Replication to copy the existing objects.
- B. Enable versioning on both buckets, and configure S3 Cross-Region Replication with default settings. Existing objects are replicated automatically when the rule is created.
- C. Configure S3 Same-Region Replication to a second bucket, and enable S3 Versioning and MFA Delete on both buckets.
- D. Create an AWS Backup plan for the bucket that runs every hour and copies the recovery points to a backup vault in us-west-2.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **S3 RTC** is designed to replicate 99.99% of new objects **within 15 minutes**, is **backed by an SLA**, and turns on **replication metrics** and event notifications (for example, when an object misses the threshold). Live replication only applies to objects written **after** the rule exists, so **S3 Batch Replication** copies the existing objects and can also retry ones that failed. Both buckets need versioning.

**Why not the others:**
- **B**: Without RTC there's no 15-minute SLA. ⚠️ Trap: Existing objects are **not** replicated by a new rule.
- **C**: A second bucket in the same Region doesn't protect against a Regional outage.
- **D**: Hourly backups can't meet a 15-minute target, and recovering means restoring data, not just switching to a live copy.

**Task statement:** 2.2 · **Review:** [HA, DR, edge & ops: S3 replication for DR](../notes/08-ha-dr-edge-ops.md)
</details>

---

## Answer summary

<details>
<summary>Show all answers</summary>

| Q | Answer | Topic | Task |
|---|---|---|---|
| 1 | B | SQS to decouple tiers, scale on backlog | 2.1 |
| 2 | D | SNS → SQS fan-out | 2.1 |
| 3 | A | Step Functions Standard, retries, task token | 2.1 |
| 4 | D | API Gateway usage plan + Lambda reserved concurrency | 2.1 |
| 5 | B | EventBridge partner event source | 2.1 |
| 6 | A, E | Aurora Replicas + reader endpoint | 2.2 |
| 7 | C | RDS Proxy for Lambda connections | 2.1 |
| 8 | D | DynamoDB global tables (multi-active) | 2.2 |
| 9 | C | Warm standby DR | 2.2 |
| 10 | B | Route 53 failover routing | 2.2 |
| 11 | B, D | Multi-AZ Auto Scaling group + ELB health checks | 2.2 |
| 12 | A | ECS on Fargate behind an ALB | 2.1 |
| 13 | C | EFS shared across AZs | 2.2 |
| 14 | C, E | Legacy app: sticky sessions + EFS | 2.2 |
| 15 | A | S3 CRR with RTC + Batch Replication | 2.2 |

</details>

**Scoring:** 13+/15 strong · 10–12 review misses · <10 re-read the module
