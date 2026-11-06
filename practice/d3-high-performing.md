# Domain 3 practice: Design High-Performing Architectures (15 questions)
> [Home](../README.md) · [Practice index](README.md) · Notes: [Compute & containers](../notes/03-compute-containers.md) · [Storage](../notes/04-storage.md) · [Databases](../notes/05-databases.md) · [HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md) · [Data, analytics & migration](../notes/09-data-analytics-migration.md)

Untimed. Read each explanation, including why the wrong options are wrong. Log misses in your error log.

---

### Q1. Large uploads from distant continents
A video production company has editors in Asia, Europe, and South America who upload raw footage files of 20–60 GB each to a single Amazon S3 bucket in us-east-1. Uploads are slow, and when a connection drops, the upload restarts from the beginning. A central editing team in us-east-1 needs each file in that bucket as soon as its upload completes.

Which combination of actions will improve upload speed and reliability? **(Choose two.)**

- A. Upload each file with a single `PutObject` request, and increase the client's request timeout.
- B. Use S3 multipart upload, so that parts are uploaded in parallel and only failed parts are retried.
- C. Create a bucket in each editor's nearest Region, and configure S3 Cross-Region Replication to the us-east-1 bucket.
- D. Enable S3 Transfer Acceleration on the bucket, and have the editors upload to the accelerate endpoint.
- E. Ship the footage to AWS on AWS Snowball Edge devices every week.

<details>
<summary>Show answer</summary>

**Answer: B, D**

**Why B and D:** **Multipart upload** splits a file into parts that upload **in parallel**. A failed part is retried on its own instead of restarting the whole file. It's **required above 5 GB** and recommended above about 100 MB. **Transfer Acceleration** sends traffic into the nearest edge location and then across the AWS backbone to the bucket's Region, which helps most over long distances. Its per-GB fee is charged only when the transfer is actually faster. Also add a lifecycle rule that **aborts incomplete multipart uploads**, because orphaned parts are billed as storage.

**Why not the others:**
- **A**: A single PUT can't exceed **5 GB**, so 20–60 GB files can't be uploaded this way at all.
- **C**: The files would reach us-east-1 later, through replication, and you'd pay extra storage and replication costs.
- **E**: Snowball Edge suits one-off bulk transfers, or sites with poor connectivity. A weekly shipment adds days of delay.

**Task statement:** 3.1 · **Review:** [Storage: S3 performance and Transfer Acceleration](../notes/04-storage.md)
</details>

---

### Q2. A single volume for very high IOPS
A company is migrating a self-managed OLTP database to Amazon EC2. The database needs a single Amazon EBS volume that delivers 150,000 sustained IOPS with consistent sub-millisecond latency. The data is critical, and the company wants the highest durability that EBS offers.

Which EBS volume type should a solutions architect choose?

- A. A gp3 volume with IOPS provisioned to the volume type's maximum.
- B. A gp2 volume sized large enough to reach 150,000 IOPS through its per-GiB baseline.
- C. An io2 Block Express volume provisioned with 150,000 IOPS, attached to an instance type that supports that level of EBS performance.
- D. An st1 volume, because it offers the highest throughput of the HDD-backed volume types.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** **io2 Block Express** scales to **256,000 IOPS** and 4,000 MiB/s per volume, with **sub-millisecond latency** and **99.999% durability**, the highest of any EBS type. The rule of thumb: more than 80,000 IOPS on one volume (the current gp3 maximum), consistent sub-ms latency, or the highest durability points to io2 Block Express. Older questions use 16,000 as the threshold, from when that was gp3's limit. ⚠️ Trap: The **instance** must also support that level of EBS IOPS and bandwidth, otherwise the instance becomes the bottleneck.

**Why not the others:**
- **A**: gp3 tops out at **80,000 IOPS** per volume.
- **B**: gp2 is capped at **16,000 IOPS** no matter how large the volume is.
- **D**: st1 is a throughput-optimized HDD for large sequential I/O. Its random IOPS are very low.

**Task statement:** 3.1 · **Review:** [Storage: EBS volume types](../notes/04-storage.md)
</details>

---

### Q3. HPC scratch file system backed by S3
A research lab stores a 300 TB genomics dataset in Amazon S3. For each analysis run, the lab launches a cluster of several hundred EC2 instances that must read the data through a shared POSIX file system with sub-millisecond latency and hundreds of GB/s of aggregate throughput. Results must be written back to S3, and the file system is deleted when the run finishes.

Which storage solution meets these requirements?

- A. Create an Amazon FSx for Lustre file system linked to the S3 bucket as its data repository, and export the results back to S3 at the end of the run.
- B. Copy the dataset to an Amazon EFS file system that uses Elastic throughput, and mount it on all instances.
- C. Mount the S3 bucket directly on every instance with Mountpoint for Amazon S3.
- D. Copy the dataset to an io2 Block Express volume with Multi-Attach enabled, and attach the volume to all instances.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **FSx for Lustre** is a parallel file system built for HPC, with sub-millisecond latency and throughput that scales with the file system's size. When it's **linked to S3**, it presents the bucket's objects as files and loads their contents on first access, so you don't copy 300 TB up front. You then **export** results back to S3. A **scratch** deployment type is the cheaper choice for short-lived processing; **persistent** keeps data replicated for longer-running workloads.

**Why not the others:**
- **B**: EFS is a general-purpose NFS file system. Its latency and per-client throughput don't match Lustre for tightly parallel HPC, and copying 300 TB in for every run is slow and expensive.
- **C**: Mountpoint gives high sequential throughput, but it lacks full POSIX semantics and has S3's per-request latency, not sub-millisecond file system latency.
- **D**: Multi-Attach supports only a small number of instances in **one AZ** and needs a cluster-aware file system. A single volume also tops out at 64 TiB.

**Task statement:** 3.1 · **Review:** [Storage: Amazon FSx family](../notes/04-storage.md)
</details>

---

### Q4. Shared storage for a Windows application
A company is moving a .NET application from its data center to EC2 instances running Windows Server in two Availability Zones. The application stores files on an SMB file share and relies on NTFS permissions tied to the company's Active Directory users and groups. The company needs a fully managed, highly available shared file system.

Which solution meets these requirements?

- A. Create an Amazon EFS file system with mount targets in both Availability Zones, and mount it on the Windows instances.
- B. Create an Amazon FSx for Lustre file system, and share it with the Windows instances as a mapped drive.
- C. Deploy Amazon S3 File Gateway on an EC2 instance, and present an S3 bucket to the application as an SMB share.
- D. Create a Multi-AZ Amazon FSx for Windows File Server file system joined to the company's Active Directory, and map the share on the instances.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **FSx for Windows File Server** is native Windows file storage. It provides **SMB**, **NTFS ACLs**, and **Active Directory** integration (with AWS Managed Microsoft AD or a self-managed AD), plus features such as shadow copies and DFS namespaces. A **Multi-AZ** deployment keeps a standby file server in a second AZ and fails over automatically.

**Why not the others:**
- **A**: EFS is NFS only and isn't supported on Windows instances.
- **B**: FSx for Lustre is for Linux clients running HPC and ML workloads, not Windows SMB.
- **C**: File Gateway is for giving **on-premises** applications access to S3. A gateway on a single EC2 instance is a single point of failure, and S3 objects behind it don't behave like a native NTFS file system.

**Task statement:** 3.1 · **Review:** [Storage: Amazon FSx family](../notes/04-storage.md)
</details>

---

### Q5. Lowest-latency node-to-node networking
An engineering firm runs a tightly coupled computational fluid dynamics (CFD) simulation that uses MPI across 64 EC2 instances. Nodes exchange data constantly, so the simulation needs the lowest possible node-to-node latency and the highest network throughput. Running all nodes in one Availability Zone is acceptable.

Which solution meets these requirements?

- A. Launch the instances in a cluster placement group in one Availability Zone, using an instance type that supports Elastic Fabric Adapter (EFA).
- B. Launch the instances in a spread placement group across three Availability Zones.
- C. Launch the instances in a partition placement group, with one partition per rack.
- D. Launch the instances in an Auto Scaling group that spans three Availability Zones behind a Network Load Balancer.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** A **cluster placement group** packs instances close together inside one AZ for the lowest latency and highest per-flow throughput. **EFA** adds OS-bypass networking that MPI libraries use directly. To reduce insufficient-capacity errors, launch all the instances in **one request** with the same instance type, or use a Capacity Reservation.

**Why not the others:**
- **B**: Spread placement groups put each instance on distinct hardware to **reduce correlated failures**, and they allow at most 7 running instances per AZ per group. Spreading across AZs also increases latency.
- **C**: Partition placement groups give **rack-level fault isolation** for distributed, replicated systems such as HDFS, Cassandra, and Kafka. They aren't designed for the lowest latency.
- **D**: Cross-AZ traffic adds latency, and a load balancer plays no part in node-to-node MPI traffic.

**Task statement:** 3.2 · **Review:** [Compute & containers: placement groups](../notes/03-compute-containers.md)
</details>

---

### Q6. Speeding up a CPU-bound Lambda function
An AWS Lambda function computes checksums and thumbnails for uploaded images. It is configured with 512 MB of memory, uses about 150 MB, and is CPU-bound. Each invocation takes about 8 seconds, and users want results faster. The team also wants to avoid increasing cost where possible.

Which change is MOST likely to meet these requirements?

- A. Increase the function's timeout so that invocations have more time to finish.
- B. Increase the function's memory setting, which also allocates proportionally more CPU, and measure duration and cost at several memory sizes to find the best setting.
- C. Configure provisioned concurrency for the function so that invocations run on pre-initialized execution environments.
- D. Decrease the function's memory setting to 256 MB, because the function uses only about 150 MB.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** Lambda allocates **CPU in proportion to memory**: about **1,769 MB gives one full vCPU**, and the maximum of 10,240 MB gives up to 6 vCPUs. You pay for GB-seconds, so if more memory cuts the duration enough, the cost stays about the same or even drops. Test several settings (the open-source Lambda Power Tuning tool automates this). Code only benefits from more than one vCPU if it is multithreaded. Switching to the arm64 (Graviton) architecture can also improve price-performance.

**Why not the others:**
- **A**: A longer timeout doesn't make the function run any faster.
- **C**: Provisioned concurrency removes **cold-start** (initialization) latency. This function's time is spent in its handler, and provisioned concurrency adds cost.
- **D**: ⚠️ Trap: Less memory means **less CPU**, so a CPU-bound function runs slower and may even cost more.

**Task statement:** 3.2 · **Review:** [Compute & containers: Lambda performance](../notes/03-compute-containers.md)
</details>

---

### Q7. One domain for static and dynamic content
An e-commerce website keeps its static assets (images, JavaScript, CSS) in an Amazon S3 bucket and serves a dynamic API from an Application Load Balancer in us-east-1. Customers in Europe and Asia report slow page loads. The company wants a single domain name, static content cached close to users, faster API calls even though the API responses can't be cached, and no direct user access to the S3 bucket.

Which solution meets these requirements?

- A. Enable S3 Transfer Acceleration for the static assets, and put AWS Global Accelerator in front of the ALB.
- B. Replicate the S3 bucket to Regions in Europe and Asia with Cross-Region Replication, and use Route 53 latency-based routing to send users to the nearest bucket.
- C. Create an AWS Global Accelerator accelerator with the S3 bucket and the ALB as endpoints.
- D. Create an Amazon CloudFront distribution with two origins: the S3 bucket, restricted with origin access control (OAC), for static paths with caching enabled, and the ALB for `/api/*` with caching disabled.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** One **CloudFront** distribution can have several origins, with **cache behaviors** chosen by path pattern. Static content is cached at edge locations. Uncacheable API calls still get faster, because TLS terminates near the user and CloudFront reuses persistent connections to the origin over the AWS backbone. **OAC** plus a bucket policy lets only this distribution read the bucket.

**Why not the others:**
- **A**: Transfer Acceleration speeds up transfers to and from a bucket but doesn't cache anything. This design also ends up with two separate hostnames.
- **B**: You'd pay for several full copies, the API wouldn't get any faster, and S3 website endpoints don't serve HTTPS.
- **C**: Global Accelerator endpoints are ALBs, NLBs, EC2 instances, and Elastic IPs. **S3 isn't a supported endpoint**, and Global Accelerator doesn't cache.

**Task statement:** 3.4 · **Review:** [HA, DR, edge & ops: CloudFront](../notes/08-ha-dr-edge-ops.md)
</details>

---

### Q8. Key-value access at massive scale
A company is building a shopping cart service for a global sales event. Every request reads or writes one cart by customer ID; there are no joins or ad hoc queries. Traffic is expected to reach millions of requests per second at peak, with sudden spikes, and every request needs single-digit millisecond latency. The company wants the database to scale without capacity planning or sharding work by the team.

Which database should a solutions architect recommend?

- A. Amazon Aurora MySQL with the maximum number of Aurora Replicas, to scale both reads and writes.
- B. Amazon RDS for MySQL, sharded by customer ID across several DB instances, with a sharding layer in the application.
- C. Amazon DynamoDB, with customer ID as the partition key, using on-demand capacity mode.
- D. Amazon Redshift, with customer ID as the distribution key.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** **DynamoDB** partitions data by key and scales horizontally to virtually any request rate for this kind of access pattern, with single-digit millisecond latency. A high-cardinality partition key such as customer ID spreads the load evenly. **On-demand** mode requires no capacity planning. For a known launch event, you can also **pre-warm** the table (warm throughput) so that it's ready for the first spike.

**Why not the others:**
- **A**: ⚠️ Trap: Aurora Replicas scale **reads only**. A cluster still has one writer instance.
- **B**: Sharding works, but it is exactly the operational work the company wants to avoid.
- **D**: Redshift is an analytics (OLAP) warehouse, not a low-latency key-value store.

**Task statement:** 3.3 · **Review:** [Databases: choosing a database](../notes/05-databases.md)
</details>

---

### Q9. Microsecond reads on DynamoDB
A retail company stores its product catalog in an Amazon DynamoDB table. The catalog API is read-heavy, and the same popular items are read millions of times per hour. Reads currently take single-digit milliseconds, but a new personalization feature needs microsecond read latency. The developers want as few code changes as possible, and eventually consistent reads are acceptable.

Which solution meets these requirements?

- A. Deploy an Amazon ElastiCache (Redis OSS) cluster, and add cache-aside logic to the application for every read path.
- B. Deploy a DynamoDB Accelerator (DAX) cluster in the VPC, and replace the DynamoDB client in the application with the DAX client.
- C. Switch all reads to strongly consistent reads so that the application always gets the latest data faster.
- D. Switch the table to on-demand capacity mode so that DynamoDB scales automatically for peak read traffic.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **DAX** is an in-memory, write-through cache built for DynamoDB. It is **API-compatible**, so the main code change is swapping the client. Cached eventually consistent reads return in **microseconds**. It caches both individual items and query or scan results, and it runs inside your VPC.

**Why not the others:**
- **A**: ElastiCache can be just as fast, but you'd have to write and maintain caching logic for every read path. That's more code than swapping a client.
- **C**: Strongly consistent reads aren't faster, they consume twice the read capacity, and DAX passes them straight through to the table.
- **D**: Capacity mode affects throughput and throttling, not per-request latency.

**Task statement:** 3.3 · **Review:** [Databases: DynamoDB and DAX](../notes/05-databases.md)
</details>

---

### Q10. Caching a read-heavy relational database
An API serves product details from an Amazon RDS for PostgreSQL database. About 95% of the queries are reads of the same few thousand products. The company already added read replicas, but database CPU is still high and p99 latency is too slow. Product data changes a few times a day, and showing data that is up to 5 minutes old is acceptable.

Which combination of actions will improve read performance? **(Choose two.)**

- A. Deploy an Amazon ElastiCache cluster and implement lazy loading: read from the cache first, and on a cache miss, read from the database and write the result to the cache.
- B. Implement write-through caching only, without any expiration, so that items stay in the cache indefinitely.
- C. Convert the DB instance to a Multi-AZ deployment so that the standby instance shares the read load.
- D. Migrate the database to Amazon Redshift to use its columnar storage for faster queries.
- E. Set a time to live (TTL) of about 5 minutes on cached items, so that stale entries expire and are reloaded from the database.

<details>
<summary>Show answer</summary>

**Answer: A, E**

**Why A and E:** **Lazy loading** (cache-aside) caches only data that is actually requested, which suits a hot set of a few thousand items. If a cache node fails, the application falls back to the database. A **TTL** that matches the business's staleness tolerance limits how old data can get and keeps memory use in check. You can add write-through for the hottest keys if you need to, but lazy loading with a TTL is the standard pairing.

**Why not the others:**
- **B**: Write-through only fills the cache **on writes**, so items that haven't changed since the cache started are always misses. With no expiry, updates made outside the application leave stale data, and memory fills with items nobody reads.
- **C**: A Multi-AZ **instance** standby serves no reads. It exists for failover.
- **D**: Redshift is a data warehouse for analytics, not an OLTP backend for an API.

**Task statement:** 3.3 · **Review:** [Databases: ElastiCache caching strategies](../notes/05-databases.md)
</details>

---

### Q11. Static IPs and fast failover for a UDP game
A gaming company runs multiplayer game servers on EC2 instances behind Network Load Balancers in three AWS Regions. The game uses a custom UDP protocol. Players worldwide experience jitter on the public internet, and some corporate and ISP firewalls require a small, fixed set of IP addresses to allow the traffic. If a Region fails, players must be redirected to a healthy Region within seconds, without waiting for DNS caches to expire.

Which solution meets these requirements?

- A. Create an AWS Global Accelerator standard accelerator with a UDP listener and an endpoint group in each Region that contains that Region's NLB.
- B. Create an Amazon CloudFront distribution with the three NLBs as origins in an origin group with failover.
- C. Create Amazon Route 53 latency-based records for the three NLBs, with health checks and a TTL of 60 seconds.
- D. Assign Elastic IP addresses to every game server, and publish the list of addresses to players and ISPs.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Global Accelerator** gives you **two static anycast IP addresses** that are announced from AWS edge locations. Player traffic enters AWS at the nearest edge and crosses the AWS backbone, which cuts jitter. It supports **TCP and UDP**. Health checks shift traffic between endpoint groups **in seconds**, and because the IP addresses never change, DNS caching isn't involved. Traffic dials and endpoint weights let you drain or shift traffic deliberately.

**Why not the others:**
- **B**: ⚠️ Trap: CloudFront handles HTTP(S) and WebSocket. It **doesn't carry UDP**.
- **C**: DNS failover depends on resolvers and clients honoring the TTL, and many cache longer. The IP addresses also aren't a small, fixed set.
- **D**: This produces a long list of addresses that changes whenever servers change, with no failover and no accelerated path.

**Task statement:** 3.4 · **Review:** [HA, DR, edge & ops: Global Accelerator](../notes/08-ha-dr-edge-ops.md)
</details>

---

### Q12. Real-time clickstream with several consumers and replay
A media website generates about 50,000 clickstream events per second. Three applications must consume the same events: a fraud detector that needs sub-second latency, a personalization engine, and an archiver that writes to Amazon S3. Events for the same user session must be processed in order. After a bug fix, the team must be able to reprocess the last 3 days of events.

Which solution meets these requirements?

- A. Send the events to an Amazon SQS standard queue that all three applications poll.
- B. Send the events to an Amazon Kinesis data stream with the session ID as the partition key, extend the retention period to at least 3 days, and register each application as an enhanced fan-out consumer.
- C. Send the events to an Amazon Data Firehose stream that delivers to Amazon S3, and have each application read the delivered files from S3.
- D. Publish the events to an Amazon SNS standard topic, and subscribe a separate Amazon SQS standard queue for each application to the topic.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Kinesis Data Streams** keeps records **ordered within a shard**, and the partition key decides the shard, so each session's events stay in order. Many applications can read the **same** stream independently. **Enhanced fan-out** gives each consumer its own read throughput per shard, delivered by push with low latency. Retention defaults to 24 hours and can be extended up to 365 days. To **replay**, a consumer starts reading from a timestamp. On-demand capacity mode removes the need to size shards.

**Why not the others:**
- **A**: Each SQS message goes to **one** consumer and is deleted after processing, so there's no replay, and a standard queue doesn't guarantee order.
- **C**: Firehose (formerly Kinesis Data Firehose) is a **delivery** service with no consumer API. Reading files from S3 isn't sub-second, and ordering per session isn't preserved.
- **D**: Fan-out works, but standard topics and queues don't guarantee order, and once a message is consumed it's gone, so there's no replay.

**Task statement:** 3.5 · **Review:** [Data, analytics & migration: Kinesis Data Streams](../notes/09-data-analytics-migration.md)
</details>

---

### Q13. Streaming JSON into Parquet on S3
Several applications emit JSON events that analysts query with Amazon Athena. To reduce query cost and time, the company wants the data stored in Amazon S3 as Apache Parquet in date-based prefixes, and available within about 5 minutes of being generated. The company doesn't want to manage servers or write data conversion code.

Which solution meets these requirements?

- A. Send the events to an Amazon Kinesis data stream, and run a Spark application on an EC2 Auto Scaling group that converts the records to Parquet and writes them to S3.
- B. Send the events to an Amazon Data Firehose stream that delivers JSON to S3, and run an AWS Glue ETL job each night that converts the day's files to Parquet.
- C. Send the events to an Amazon Data Firehose stream with record format conversion turned on, using a table schema in the AWS Glue Data Catalog, and deliver Parquet files to S3.
- D. Invoke an AWS Lambda function for each event that converts the event to Parquet and writes it to S3 as a separate object.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** Firehose has built-in **record format conversion** from JSON to **Parquet or ORC**. It reads the schema from a table in the **Glue Data Catalog**. The buffering size and interval settings control how quickly files land, so a few minutes is easy to meet. By default, objects are written under time-based `YYYY/MM/DD/HH` prefixes, and **dynamic partitioning** can partition by fields in the records instead. It's fully managed, with no code to write.

**Why not the others:**
- **A**: You'd manage servers and write the conversion code yourself, which are the two things the company wants to avoid.
- **B**: A nightly batch misses the 5-minute requirement and adds a second pipeline to run.
- **D**: One object per event creates millions of tiny files, which Athena handles badly, and you'd still be writing code.

**Task statement:** 3.5 · **Review:** [Data, analytics & migration: Firehose](../notes/09-data-analytics-migration.md)
</details>

---

### Q14. Ad hoc SQL on data in S3
A company stores several terabytes of CSV and JSON log files in Amazon S3, organized in date-based prefixes. Analysts need to run ad hoc SQL queries a few times a week. New prefixes are added every day, and the schema occasionally gains new columns. The company wants to avoid managing infrastructure and to pay only when queries run.

Which combination of steps meets these requirements? **(Choose two.)**

- A. Load the data into a provisioned Amazon Redshift cluster every night, and have the analysts query the cluster.
- B. Launch an Amazon EMR cluster with Apache Hive, and keep it running for the analysts' queries.
- C. Schedule an AWS Glue crawler to scan the S3 prefixes and keep the table definitions and partitions in the AWS Glue Data Catalog up to date.
- D. Use S3 Select to run SQL queries that join and aggregate data across all of the log files.
- E. Query the data with Amazon Athena by using the tables in the AWS Glue Data Catalog.

<details>
<summary>Show answer</summary>

**Answer: C, E**

**Why C and E:** **Athena** is serverless SQL over S3. With on-demand pricing you pay per amount of data scanned, so occasional queries cost very little. It reads table definitions from the **Glue Data Catalog**. A scheduled **Glue crawler** infers schemas, picks up new columns, and registers new partitions as prefixes appear. (Partition projection is an alternative to crawling for the partitions.) To cut cost and latency further, convert the data to **Parquet** and partition it by date.

**Why not the others:**
- **A**: A provisioned cluster runs (and bills) all the time, and nightly loads are extra pipeline work. Redshift fits frequent, heavy BI workloads better.
- **B**: A cluster that's always running means infrastructure to manage and pay for between queries.
- **D**: S3 Select filters the contents of **one object at a time**. It can't join or aggregate across files, and AWS no longer offers it to new customers.

**Task statement:** 3.5 · **Review:** [Data, analytics & migration: Athena and Glue](../notes/09-data-analytics-migration.md)
</details>

---

### Q15. Faster Spark jobs on EMR at the same cost
A company runs a nightly Apache Spark job on an Amazon EMR cluster of On-Demand Instances. The job takes 4 hours, and the company wants it to finish sooner without increasing cost. The cluster stores intermediate data in HDFS on the core nodes. The Spark job tolerates the loss of executors, which are rerun automatically.

Which solution meets these requirements?

- A. Run the primary, core, and task nodes all on Spot Instances to get the largest discount across the whole cluster.
- B. Run the core nodes on Spot Instances and the task nodes on On-Demand Instances.
- C. Replace the EMR cluster with a single, very large EC2 On-Demand Instance that runs Spark in local mode.
- D. Keep the primary and core nodes on On-Demand Instances, and add task nodes that run on Spot Instances through an instance fleet with several instance types.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **Task nodes** run compute only and store no HDFS data, so losing one just means Spark reruns its tasks. Spot's discount lets you add **more compute for the same budget**, which shortens the job. **Instance fleets** that allow several instance types draw on more Spot capacity pools, which lowers the chance of interruption. EMR managed scaling can adjust the task fleet automatically.

**Why not the others:**
- **A**: ⚠️ Trap: If the **primary** node is interrupted, the whole cluster is lost. If **core** nodes are interrupted, HDFS data is lost, which can fail the job.
- **B**: This is the wrong way round: the nodes holding HDFS data become the interruptible ones.
- **C**: One instance can't scale out, and it's a single point of failure. It's unlikely to be faster than a properly sized cluster.

**Task statement:** 3.2 · **Review:** [Data, analytics & migration: EMR](../notes/09-data-analytics-migration.md)
</details>

---

## Answer summary

<details>
<summary>Show all answers</summary>

| Q | Answer | Topic | Task |
|---|---|---|---|
| 1 | B, D | S3 multipart upload + Transfer Acceleration | 3.1 |
| 2 | C | io2 Block Express for >80,000 IOPS | 3.1 |
| 3 | A | FSx for Lustre linked to S3 | 3.1 |
| 4 | D | FSx for Windows File Server (SMB, AD) | 3.1 |
| 5 | A | Cluster placement group + EFA | 3.2 |
| 6 | B | Lambda memory = CPU | 3.2 |
| 7 | D | CloudFront with S3 (OAC) + ALB origins | 3.4 |
| 8 | C | DynamoDB for key-value at massive scale | 3.3 |
| 9 | B | DAX for microsecond reads | 3.3 |
| 10 | A, E | ElastiCache lazy loading + TTL | 3.3 |
| 11 | A | Global Accelerator for UDP + static IPs | 3.4 |
| 12 | B | Kinesis Data Streams, enhanced fan-out, replay | 3.5 |
| 13 | C | Firehose record format conversion to Parquet | 3.5 |
| 14 | C, E | Glue crawler + Athena | 3.5 |
| 15 | D | EMR task nodes on Spot | 3.2 |

</details>

**Scoring:** 13+/15 strong · 10–12 review misses · <10 re-read the module
