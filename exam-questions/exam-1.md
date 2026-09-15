# Exam 1

## Difficult questions to review

No questions marked for review yet.

## Questions

### Question 01: EBS encryption and KMS key rotation

#### Question

A company uses Amazon EC2 instances and stores data on Amazon Elastic Block Store (Amazon EBS) volumes. The company must ensure that all data is encrypted at rest by using AWS Key Management Service (AWS KMS). The company must be able to control rotation of the encryption keys.

Which solution will meet these requirements with the LEAST operational overhead?

- **A.** Create a customer managed key. Use the key to encrypt the EBS volumes.
- **B.** Use an AWS managed key to encrypt the EBS volumes. Use the key to configure automatic key rotation.
- **C.** Create an external KMS key with imported key material. Use the key to encrypt the EBS volumes.
- **D.** Use an AWS owned key to encrypt the EBS volumes.

#### Correct answer: A

Use a symmetric customer managed KMS key with AWS-generated key material for EBS encryption, and enable automatic rotation. This gives the company control over rotation while KMS handles the recurring work.

#### Understand the terms

- **EC2 instance:** A virtual server.
- **EBS volume:** Block storage used as a disk by an EC2 instance.
- **Encryption at rest:** Encryption of stored data.
- **AWS KMS:** The AWS service used to create and manage encryption keys.
- **Key material:** The cryptographic secret associated with a key.
- **Operational overhead:** The ongoing work needed to operate and maintain a solution.

EBS supports an AWS managed key (`aws/ebs`) or a symmetric customer managed key. EBS encrypts volume data with a data key protected by the selected KMS key. [AWS: How EBS encryption works](https://docs.aws.amazon.com/ebs/latest/userguide/how-ebs-encryption-works.html)

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: Customer managed key** | **Correct.** The company configures rotation; AWS KMS performs it automatically for eligible AWS-generated keys. |
| **B: AWS managed key** | AWS controls rotation. The company cannot configure it, so this fails the control requirement. |
| **C: Imported key material** | Adds responsibility for generating and importing key material. It does not support automatic rotation, although symmetric imported keys support on-demand rotation. More overhead than A. |
| **D: AWS owned key** | The company does not control rotation. It is also not one of the documented selectable EBS key types. |

Rotation behavior: [AWS: Rotate KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html). Key ownership: [AWS: KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html).

#### How to reason through the question

1. **"Encrypted at rest ... using AWS KMS"** identifies EBS encryption integrated with KMS.
2. **"Control rotation"** eliminates B and D.
3. **"LEAST operational overhead"** favors A over managing imported material in C.

Choose the simplest option that meets every requirement. A low-maintenance option that fails a requirement is still incorrect.

#### Common exam trap

**Customer managed does not mean manually rotated.** You control the settings while AWS performs automatic rotation. Option A enables that capability even though its wording does not explicitly mention enabling rotation.

#### What to remember

**Control rotation + low operational overhead = customer managed KMS key with AWS-generated material and automatic rotation.**

### Question 02: Resilient migration of a web tier and MySQL database

#### Question

A company is migrating its multi-tier on-premises application to AWS. The application consists of a single-node MySQL database and a multi-node web tier. The company must minimize changes to the application during migration. The company wants to improve application resiliency after the migration.

Which combination of steps will meet these requirements? (Choose two.)

- **A.** Migrate the web tier to Amazon EC2 instances in an Auto Scaling group behind an Application Load Balancer.
- **B.** Migrate the database to Amazon EC2 instances in an Auto Scaling group behind a Network Load Balancer.
- **C.** Migrate the database to an Amazon RDS Multi-AZ deployment.
- **D.** Migrate the web tier to an AWS Lambda function.
- **E.** Migrate the database to an Amazon DynamoDB table.

#### Correct answers: A and C

Keep the web application running on servers using EC2, and keep MySQL using Amazon RDS for MySQL. Improve availability at both tiers without redesigning the application around a different execution model or database type.

#### Understand the terms

- **Multi-tier:** The application has separate layers, here a web layer and a database layer.
- **Multi-node web tier:** Multiple servers run the web application.
- **Resiliency:** The ability to withstand or recover from failures.
- **Availability Zone (AZ):** An isolated location within an AWS Region.
- **Auto Scaling group (ASG):** Maintains the desired number of EC2 instances, replaces unhealthy instances, and can adjust capacity.
- **Application Load Balancer (ALB):** Distributes HTTP/HTTPS requests among web application targets.
- **RDS Multi-AZ:** A managed relational database deployment spanning Availability Zones for availability and failover.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: EC2 + ASG + ALB for the web tier** | **Correct.** Preserves the server-based application. The ALB distributes requests, and the ASG maintains capacity and replaces unhealthy instances. Deploy the instances across multiple AZs for resilience to an AZ outage. |
| **B: EC2 + ASG + NLB for MySQL** | **Incorrect.** Launching additional database servers does not automatically replicate their data or coordinate the primary database and failover. The NLB distributes connections; it does not provide MySQL replication. Building those mechanisms adds work and complexity. |
| **C: RDS Multi-AZ for MySQL** | **Correct.** Keeps the MySQL engine while adding managed failover across AZs. This minimizes application changes compared with switching database models. Connection settings and compatibility still need checking. |
| **D: Lambda for the web tier** | **Incorrect.** Moving a conventional server-based web tier into a Lambda function generally requires adapting the application to Lambda's execution model. It conflicts with minimizing changes. |
| **E: DynamoDB for the database** | **Incorrect.** DynamoDB is a NoSQL database. Moving from MySQL would require changes to the data model and database access logic, conflicting with minimal application changes. |

For A, enable Elastic Load Balancing health checks in the ASG if it should replace instances that fail application-level load balancer checks. [AWS: Auto Scaling with Elastic Load Balancing](https://docs.aws.amazon.com/autoscaling/ec2/userguide/autoscaling-load-balancer.html)

For C, a standard Multi-AZ DB instance deployment uses a standby in another AZ. RDS automatically fails over when appropriate; applications must reconnect after the interruption. [AWS: RDS Multi-AZ failover](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.Failover.html)

#### How to reason through the question

1. **"Minimize changes"** means preserve the current application and database technologies where possible. Eliminate D and E because they change the execution model or database model.
2. **"Multi-node web tier"** fits multiple EC2 instances behind an ALB. Add an ASG to maintain capacity: choose A.
3. **"Single-node MySQL database"** identifies a single point of failure. Use RDS for MySQL with Multi-AZ to add managed failover: choose C.
4. **"Improve application resiliency"** means address both tiers. B provides infrastructure components without the database replication and failover mechanisms needed.
5. **"Choose two"** is satisfied by A for the web tier and C for the database tier.

#### Common exam trap

**An Auto Scaling group does not automatically make a database highly available.** Web servers can often serve interchangeable requests; database servers must coordinate persistent data and writes. More EC2 instances plus a load balancer do not establish that coordination.

Also, serverless services are not automatically the best migration choice when the question prioritizes minimal changes.

For a standard RDS Multi-AZ DB instance deployment, the standby supports failover and does not serve reads. Do not confuse this with a Multi-AZ DB cluster, whose reader instances can serve read traffic. [AWS: RDS Multi-AZ deployment types](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

#### What to remember

**Minimal-change resilient migration: web tier → EC2 + ASG + ALB across AZs; MySQL → RDS for MySQL Multi-AZ.**

**Load balancing distributes traffic; database replication and failover protect database availability.**

### Question 03: SMB file access and archiving after seven days

#### Question

A company runs an SMB file server in its data center. The file server stores large files that the company frequently accesses for up to 7 days after the file creation date. After 7 days, the company needs to be able to access the files with a maximum retrieval time of 24 hours.

Which solution will meet these requirements?

- **A.** Use AWS DataSync to copy data that is older than 7 days from the SMB file server to AWS.
- **B.** Create an Amazon S3 File Gateway to increase the company's storage space. Create an S3 Lifecycle policy to transition the data to S3 Glacier Deep Archive after 7 days.
- **C.** Create an Amazon FSx File Gateway to increase the company's storage space. Create an Amazon S3 Lifecycle policy to transition the data after 7 days.
- **D.** Configure access to Amazon S3 for each user. Create an S3 Lifecycle policy to transition the data to S3 Glacier Flexible Retrieval after 7 days.

#### Correct answer: B

S3 File Gateway provides SMB access to files stored as S3 objects. Use an online storage class such as S3 Standard initially, then a Lifecycle rule to archive the objects after seven days. Use Deep Archive Standard retrieval when an archived file is needed.

The intended exam interpretation is that the existing SMB access model should be retained. B addresses both this integration and the archive requirement.

#### Understand the terms

- **SMB (Server Message Block):** A network file-sharing protocol commonly used for Windows shared folders.
- **S3 File Gateway:** Presents an SMB or NFS file share backed by S3, with a local cache for low-latency access.
- **S3 Lifecycle policy:** Rules that transition S3 objects between storage classes or expire them.
- **S3 Glacier Deep Archive:** An archive storage class requiring a restore before archived data can be read.

File Gateway supports file protocols and S3 lifecycle management. [AWS: S3 File Gateway](https://docs.aws.amazon.com/filegateway/latest/files3/what-is-file-s3.html)

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: DataSync** | Incomplete as written. Copies data, but does not specify an archive destination, retrieval workflow, or ongoing SMB access to the cloud copy. DataSync can be part of a larger solution, but this option does not describe that solution. |
| **B: S3 File Gateway + Deep Archive lifecycle** | **Correct.** Combines SMB access with S3-backed storage and age-based archiving. Deep Archive Standard retrieval typically completes within 12 hours, fitting the exam's 24-hour requirement. |
| **C: FSx File Gateway + S3 lifecycle** | Incorrect service pairing. FSx File Gateway accesses FSx for Windows File Server, not S3 objects to which an S3 Lifecycle policy can be applied. |
| **D: Direct S3 access + Flexible Retrieval** | The retrieval speed can fit, but direct S3 access does not provide the existing SMB file-share interface. B better matches the stated file-server environment. |

FSx backend: [AWS: FSx File Gateway](https://docs.aws.amazon.com/filegateway/latest/filefsxw/what-is-file-fsxw.html). Restore timing: [AWS: Archive retrieval options](https://docs.aws.amazon.com/AmazonS3/latest/userguide/restoring-objects-retrieval-options.html).

#### How to reason through the question

1. **"SMB file server in its data center"** points toward a gateway that preserves file-share access.
2. **"Frequently accesses ... up to 7 days"** means keep recent data online; a gateway cache helps with frequently used files.
3. **"After 7 days"** identifies an age-based transition suitable for S3 Lifecycle.
4. **"Maximum retrieval time of 24 hours"** allows archival storage with a suitable restore tier. Deep Archive Standard retrieval typically takes up to 12 hours; Bulk can take 48 hours and is unsuitable here.
5. Combine those requirements: **S3 File Gateway + Lifecycle + Deep Archive, using Standard restores = B.**

#### Common exam trap

**Deep Archive does not always mean a 48-hour retrieval.** The retrieval tier matters: choose Standard for this question, not Bulk. AWS describes typical completion times, not an unconditional end-to-end 24-hour guarantee.

**S3 File Gateway and FSx File Gateway have different backends.** S3 Lifecycle manages S3 objects, not files on an FSx file system.

**Archiving is automatic; restoring is a separate operation.** Opening an archived file through SMB does not automatically initiate a restore. Restore the object in S3 before reading it through the gateway; otherwise access can produce an I/O error. [AWS: Gateway storage classes](https://docs.aws.amazon.com/filegateway/latest/files3/storage-classes.html), [AWS: Gateway troubleshooting](https://docs.aws.amazon.com/filegateway/latest/files3/troubleshooting-file-gateway-issues.html).

Implementation detail: lifecycle age is based on the S3 object's creation time, not automatically the original file's on-premises creation timestamp. Timely ingestion makes the question's seven-day policy align with file age; migrating old files requires accounting for this difference.

#### What to remember

**SMB + S3-backed storage = S3 File Gateway. Age-based archiving = S3 Lifecycle. Deep Archive Standard restore ≈ 12 hours; Bulk ≈ 48 hours.**

**Check both the access protocol and the retrieval tier before choosing an archive solution.**

### Question 04: Continuously copy changed S3 objects to EFS and another S3 bucket

#### Question

A solutions architect needs to copy files from an Amazon S3 bucket to an Amazon Elastic File System (Amazon EFS) file system and another S3 bucket. The files must be copied continuously. New files are added to the original S3 bucket consistently. The copied files should be overwritten only if the source file changes.

Which solution will meet these requirements with the LEAST operational overhead?

- **A.** Create an AWS DataSync location for both the destination S3 bucket and the EFS file system. Create a task for the destination S3 bucket and the EFS file system. Set the transfer mode to transfer only data that has changed.
- **B.** Create an AWS Lambda function. Mount the file system to the function. Set up an S3 event notification to invoke the function when files are created and changed in Amazon S3. Configure the function to copy files to the file system and the destination S3 bucket.
- **C.** Create an AWS DataSync location for both the destination S3 bucket and the EFS file system. Create a task for the destination S3 bucket and the EFS file system. Set the transfer mode to transfer all data.
- **D.** Launch an Amazon EC2 instance in the same VPC as the file system. Mount the file system. Create a script to routinely synchronize all objects that changed in the origin S3 bucket to the destination S3 bucket and the mounted file system.

#### Correct answer: A

Use AWS DataSync with the original S3 bucket as the source and create two recurring transfer tasks: one from the source S3 location to the destination S3 location, and one from the source S3 location to the EFS location. Set both tasks to **transfer only data that has changed** (`CHANGED`) and allow changed destination files to be overwritten.

This is the intended interpretation of option A. A DataSync task has exactly one source location and one destination location, so a single task cannot copy to both destinations. The wording “a task for the destination S3 bucket and the EFS file system” should be understood as configuring the necessary task for each destination.

AWS DataSync supports S3-to-S3 and S3-to-EFS transfers without a DataSync agent. Scheduled DataSync tasks run periodically, with a minimum interval of one hour. Therefore, “continuously” means recurring synchronization in this question, not real-time replication.

#### Understand the terms

- **DataSync location:** A DataSync configuration representing storage that acts as a task's source or destination.
- **DataSync task:** A transfer definition containing one source, one destination, transfer settings, and optionally a schedule.
- **Transfer only data that has changed (`CHANGED`):** The initial execution copies the data; later executions compare the source and destination and copy only differences.
- **Overwrite mode:** Determines whether DataSync replaces destination data when the corresponding source data or metadata has changed.
- **Operational overhead:** The ongoing work required to build, monitor, patch, and maintain the solution.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: DataSync with changed-data mode** | **Correct.** DataSync is a managed transfer service that supports both required destinations. Recurring tasks with `CHANGED` mode copy new and modified source data and avoid transferring unchanged data again. It requires less custom operation than Lambda or EC2. |
| **B: Lambda triggered by S3 events** | **Incorrect for least overhead.** It can be engineered to copy objects when events occur, but the company must write, test, monitor, retry, and maintain custom code. The function also needs networking and EFS mounting configuration. DataSync already provides the managed transfer capability. |
| **C: DataSync with transfer-all mode** | **Incorrect.** `ALL` mode copies everything from the source on every task execution without comparing the source and destination. It does not meet the requirement to overwrite copied files only when the source changes and creates unnecessary transfer work. |
| **D: EC2 synchronization script** | **Incorrect for least overhead.** The company must operate the EC2 instance and maintain scheduling, synchronization logic, retries, monitoring, and patching. DataSync removes most of that work. |

AWS references: [supported DataSync locations and transfers](https://docs.aws.amazon.com/datasync/latest/userguide/working-with-locations.html), [changed versus all transfer modes and overwrite behavior](https://docs.aws.amazon.com/datasync/latest/userguide/configure-metadata.html), and [DataSync task scheduling](https://docs.aws.amazon.com/datasync/latest/userguide/task-scheduling.html).

#### How to reason through the question

1. **“S3 bucket to EFS and another S3 bucket”** identifies a managed service that supports both S3-to-EFS and S3-to-S3 transfers: DataSync.
2. **“Files must be copied continuously”** means configure recurring DataSync task executions for each destination.
3. **“Overwritten only if the source file changes”** directly matches DataSync's **transfer only data that has changed** mode.
4. **“LEAST operational overhead”** eliminates the custom Lambda solution and the self-managed EC2 script.
5. Between A and C, **A** uses the required changed-data behavior; C repeatedly transfers all source data.

#### Common exam trap

**“Continuously” does not always mean event-driven or real time.** In a least-overhead data-transfer question, it can mean a managed task that runs repeatedly on a schedule. Native DataSync schedules have a minimum interval of one hour.

**A DataSync task has one source and one destination.** Copying one source to two destinations requires two tasks, even though the answer option compresses this into singular wording.

**`CHANGED` is not the same as copying only newly created objects.** After the initial transfer, DataSync copies new objects and source data or metadata that differs from the destination. With overwriting enabled, it replaces the destination copy only when a difference is detected.

#### What to remember

**Recurring managed copies between supported AWS storage services = AWS DataSync.**

**One source and two destinations = two DataSync tasks. Use `CHANGED` mode to copy only new or modified data after the initial transfer.**

### Question 05: Full control of encryption keys for sensitive S3 data

#### Question

A company stores sensitive data in Amazon S3. A solutions architect needs to create an encryption solution. The company needs to fully control the ability of users to create, rotate, and disable encryption keys with minimal effort for any data that must be encrypted.

Which solution will meet these requirements?

- **A.** Use default server-side encryption with Amazon S3 managed encryption keys (SSE-S3) to store the sensitive data.
- **B.** Create a customer managed key by using AWS Key Management Service (AWS KMS). Use the new key to encrypt the S3 objects by using server-side encryption with AWS KMS keys (SSE-KMS).
- **C.** Create an AWS managed key by using AWS Key Management Service (AWS KMS). Use the new key to encrypt the S3 objects by using server-side encryption with AWS KMS keys (SSE-KMS).
- **D.** Download S3 objects to an Amazon EC2 instance. Encrypt the objects by using customer managed keys. Upload the encrypted objects back into Amazon S3.

#### Correct answer: B

Create a symmetric customer managed KMS key and configure Amazon S3 to use it with SSE-KMS. A customer managed key gives the company control over who can administer and use the key through KMS key policies, IAM policies, and grants. Authorized administrators can enable or disable the key, configure automatic rotation, perform eligible on-demand rotations, and schedule deletion.

S3 performs the encryption and decryption operations, so the company receives this control without building and operating a separate encryption process. The KMS key must be in the same AWS Region as the S3 bucket.

#### Understand the terms

- **SSE-S3:** Server-side encryption in which Amazon S3 owns and manages the encryption keys.
- **SSE-KMS:** Server-side encryption in which Amazon S3 uses an AWS KMS key to protect object data.
- **Customer managed KMS key:** A KMS key created in the company's AWS account whose policy, access, state, rotation settings, and lifecycle the company controls.
- **AWS managed key:** A KMS key created and managed by an AWS service for use in the customer's account, such as `aws/s3`. Customers can view and use it but cannot manage its key policy, rotation, or enabled state.
- **Key policy:** The primary resource policy that controls access to a KMS key.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: SSE-S3** | **Incorrect.** S3 encrypts the data at rest, but Amazon S3 owns and manages the keys. The company cannot control who creates, rotates, or disables those keys. |
| **B: Customer managed KMS key + SSE-KMS** | **Correct.** The company controls the key policy and administrative permissions, can enable or disable the key, and controls eligible rotation settings. S3 handles object encryption and decryption, satisfying the minimal-effort requirement. |
| **C: AWS managed KMS key + SSE-KMS** | **Incorrect.** Customers do not create AWS managed keys; the integrated AWS service creates them when needed. AWS controls their policy and lifecycle. AWS rotates AWS managed keys automatically, and customers cannot enable or disable their rotation or disable the keys. |
| **D: Download, encrypt, and re-upload objects** | **Incorrect.** A custom client-side workflow could use company-controlled keys, but it adds unnecessary data movement, code, compute, monitoring, and key-handling work. It does not satisfy the minimal-effort requirement as well as managed SSE-KMS. |

AWS references: [S3 server-side encryption with AWS KMS keys](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html), [SSE-S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html), [enabling and disabling KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/enabling-keys.html), and [KMS key rotation](https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html).

#### How to reason through the question

1. **“Sensitive data in Amazon S3”** requires encryption at rest. Several options provide encryption, so this alone does not decide the answer.
2. **“Fully control”** is the decisive phrase. It requires a **customer managed KMS key**, whose administrators and users the company controls.
3. **“Create, rotate, and disable encryption keys”** eliminates SSE-S3 and AWS managed KMS keys because AWS manages those capabilities.
4. **“Minimal effort”** favors server-side encryption through the native S3 and KMS integration instead of a custom EC2 encryption workflow.
5. Therefore, choose **B: customer managed KMS key with SSE-KMS**.

#### Common exam trap

**“AWS managed key” and “customer managed key in AWS KMS” are not the same thing.** Both reside in KMS, but only a customer managed key gives the company administrative control over its policy, enabled state, rotation configuration, and lifecycle.

**SSE-S3 provides encryption but not customer control of the keys.** When a question only requires encryption at rest with the least cost or simplest defaults, SSE-S3 may be enough. When it asks for control, auditing, or specific key permissions, look for SSE-KMS with a customer managed key.

Disabling the KMS key does not erase encrypted S3 objects. It prevents KMS from decrypting the protected data keys, so the objects become inaccessible until the key is re-enabled. This makes permission to disable a key highly sensitive.

#### What to remember

**S3 encryption plus full customer control of key access, rotation, and state = SSE-KMS with a customer managed KMS key.**

**SSE-S3: S3 manages the keys. AWS managed KMS key: AWS manages the key. Customer managed KMS key: the customer controls administration and access.**

### Question 06: Reduce heavy read load on an RDS for PostgreSQL database

#### Question

A company runs a web application on Amazon EC2 instances in an Auto Scaling group. The application uses a database that runs on an Amazon RDS for PostgreSQL DB instance. The application performs slowly when traffic increases. The database experiences a heavy read load during periods of high traffic.

Which actions should a solutions architect take to resolve these performance issues? (Choose two.)

- **A.** Turn on auto scaling for the DB instance.
- **B.** Create a read replica for the DB instance. Configure the application to send read traffic to the read replica.
- **C.** Convert the DB instance to a Multi-AZ DB instance deployment. Configure the application to send read traffic to the standby DB instance.
- **D.** Create an Amazon ElastiCache cluster. Configure the application to cache query results in the ElastiCache cluster.
- **E.** Configure the Auto Scaling group subnets to ensure that the EC2 instances are provisioned in the same Availability Zone as the DB instance.

#### Correct answers: B and D

Create an RDS for PostgreSQL read replica and route eligible read queries to it. Also cache frequently requested query results in Amazon ElastiCache. These actions reduce the number of reads handled by the primary DB instance and improve response times during traffic spikes.

The two solutions complement each other:

- The **read replica** adds database capacity for read-only queries.
- **ElastiCache** prevents repeated queries from reaching the database at all when a valid cached result is available.

#### Understand the terms

- **Read replica:** A read-only copy of an RDS DB instance that is updated asynchronously from the source database and has its own endpoint.
- **Read scaling:** Distributing read-only queries across additional database capacity.
- **ElastiCache:** A managed, in-memory cache service that can return frequently requested data faster than querying a relational database.
- **Cache hit:** The requested value is found in the cache, so the application does not query the database.
- **Multi-AZ DB instance deployment:** A high-availability configuration with a synchronous standby in another Availability Zone for failover.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: Turn on auto scaling for the DB instance** | **Incorrect.** A standard RDS for PostgreSQL DB instance does not automatically add or remove DB compute instances in response to read traffic. RDS storage autoscaling can grow storage, but the problem is a heavy read workload rather than insufficient storage. RDS also does not automatically scale the number of DB instance read replicas. |
| **B: Create and use a read replica** | **Correct.** RDS for PostgreSQL read replicas are designed to scale read-heavy workloads. Routing read-only queries to the replica reduces work on the primary DB instance. Writes must still go to the primary. |
| **C: Send reads to a Multi-AZ standby** | **Incorrect.** A standard Multi-AZ DB instance standby exists for high availability and automatic failover. It does not accept application connections and cannot serve read traffic. |
| **D: Cache query results in ElastiCache** | **Correct.** Frequently repeated queries can be served from fast, in-memory cache instead of the database. This reduces database load and application latency, especially during high-concurrency traffic. |
| **E: Place all EC2 instances in the database's AZ** | **Incorrect.** This does not add database read capacity or prevent repeated queries. It also reduces web-tier resilience by concentrating instances in one Availability Zone. |

AWS references: [RDS for PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html), [RDS read-replica use cases](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html), [RDS Multi-AZ DB instance standbys](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html), and [caching database query results with ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/caching-database-query-results.html).

#### How to reason through the question

1. Identify the bottleneck: the prompt explicitly says **“the database experiences a heavy read load.”** Focus on reducing or distributing database reads.
2. **B distributes reads:** a read replica handles read-only queries separately from the primary database.
3. **D removes repeated reads:** ElastiCache serves reusable query results without contacting PostgreSQL.
4. Eliminate **C** because a standard Multi-AZ standby is a failover target, not a readable replica.
5. Eliminate **A** because the option describes DB instance auto scaling that standard RDS PostgreSQL does not provide; storage autoscaling would not address read compute load.
6. Eliminate **E** because changing AZ placement does not solve database saturation and would weaken availability.

#### Common exam trap

**Multi-AZ and read replicas solve different primary problems.**

- **Multi-AZ DB instance:** high availability and failover; its standby cannot serve reads.
- **Read replica:** read scaling; the application must connect to the replica endpoint for read-only queries.

Do not confuse **RDS storage autoscaling** with automatic scaling of database compute or read replicas. Storage autoscaling increases allocated storage when needed; it does not solve high CPU or query pressure caused by heavy reads.

Caching also requires application logic and a suitable cache strategy. Frequently repeated, relatively stable results are good candidates. Queries requiring strong read-after-write consistency might need to bypass the cache or use careful invalidation.

#### What to remember

**Heavy relational-database read load = add read replicas and cache frequently requested results.**

**Read replica scales reads. ElastiCache avoids repeated reads. Standard Multi-AZ standby provides failover and cannot serve read traffic.**

### Question 07: Low-latency application deployment outside the parent AWS Region

#### Question

A company wants to migrate its web applications from on premises to AWS. The company is located close to the eu-central-1 Region. Because of regulations, the company cannot launch some of its applications in eu-central-1. The company wants to achieve single-digit millisecond latency.

Which solution will meet these requirements?

- **A.** Deploy the applications in eu-central-1. Extend the company’s VPC from eu-central-1 to an edge location in Amazon CloudFront.
- **B.** Deploy the applications in AWS Local Zones by extending the company's VPC from eu-central-1 to the chosen Local Zone.
- **C.** Deploy the applications in eu-central-1. Extend the company’s VPC from eu-central-1 to the regional edge caches in Amazon CloudFront.
- **D.** Deploy the applications in AWS Wavelength Zones by extending the company’s VPC from eu-central-1 to the chosen Wavelength Zone.

#### Correct answer: B

Enable an appropriate AWS Local Zone associated with the parent Region, extend the VPC by creating a subnet in that Local Zone, and launch the supported application resources in the Local Zone subnet. A Local Zone places compute and other supported resources closer to a specific population or industry center, which supports single-digit millisecond latency and can help satisfy local data-residency requirements.

The company must confirm that the selected Local Zone's physical location, supported services, and data flows meet its exact regulation. The exam question implies that an eligible Local Zone exists outside the prohibited eu-central-1 deployment location.

#### Understand the terms

- **AWS Region:** A separate geographic area containing multiple Availability Zones.
- **AWS Local Zone:** AWS infrastructure placed near a population or industry center and connected to a parent Region. It supports selected AWS services close to local users.
- **Local Zone subnet:** A VPC subnet whose resources are physically placed in the Local Zone.
- **CloudFront edge location:** A point of presence that caches and delivers content close to viewers; it is not a general-purpose VPC deployment location.
- **Regional edge cache:** A larger CloudFront caching layer between edge locations and an application's origin.
- **AWS Wavelength Zone:** AWS compute and storage infrastructure placed at the edge of a communications service provider's network, especially for low-latency traffic from mobile and 5G devices.

#### How a Local Zone deployment works

1. The company selects and enables a Local Zone associated with the parent AWS Region.
2. It extends its existing VPC by creating a subnet in that Local Zone.
3. It launches supported resources, such as EC2 instances, in the Local Zone subnet.
4. Local users reach those resources through a shorter network path, reducing latency.
5. The Local Zone resources can still communicate with services in the parent Region, but that traffic has regional latency and may affect data-residency decisions.

For example, an interactive rendering application can run its latency-sensitive compute nodes in a nearby Local Zone while keeping less latency-sensitive management services in the parent Region. If regulations prohibit application data from entering the parent Region, the design must verify that every dependency and data path remains compliant rather than assuming the Local Zone alone solves compliance.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: eu-central-1 plus a CloudFront edge location** | **Incorrect.** It explicitly launches the applications in the prohibited Region. CloudFront edge locations cache and deliver content; customers do not extend a VPC to an edge location or deploy an ordinary web application stack there. |
| **B: Extend the VPC to an AWS Local Zone** | **Correct.** Local Zones run supported compute and storage resources close to users, are designed for low-latency applications, and can support local residency needs. The application resources are launched in a Local Zone subnet instead of an eu-central-1 Availability Zone. |
| **C: eu-central-1 plus a CloudFront regional edge cache** | **Incorrect.** It still deploys the applications in the prohibited Region. A regional edge cache is an internal CloudFront caching layer, not a VPC extension or a place where the company launches its application resources. |
| **D: Extend the VPC to an AWS Wavelength Zone** | **Incorrect for this scenario.** Wavelength is intended primarily for workloads that need ultra-low-latency access from mobile devices through a communications service provider's network. The question describes general web applications near a geographic location and does not mention 5G or carrier-network traffic, so Local Zones are the appropriate choice. |

AWS references: [how AWS Local Zones work](https://docs.aws.amazon.com/local-zones/latest/ug/how-local-zones-work.html), [extending VPCs to Local and Wavelength Zones](https://docs.aws.amazon.com/vpc/latest/userguide/Extend_VPCs.html), [how CloudFront edge locations and regional edge caches work](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html), and [Wavelength Zone subnets and use cases](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-wavelength.html).

#### How to reason through the question

1. **“Cannot launch some applications in eu-central-1”** immediately eliminates A and C because both deploy the applications in that Region.
2. **“Single-digit millisecond latency”** requires application compute close to the company, not merely cached static content close to viewers.
3. The remaining choices are Local Zones and Wavelength Zones.
4. **General web applications near a geographic location** match AWS Local Zones.
5. Wavelength would become the stronger choice if the prompt emphasized **5G mobile devices, carrier networks, or mobile-edge computing**.
6. Therefore, choose **B**.

#### Common exam trap

**CloudFront edge locations and regional edge caches are not places where you extend a VPC and launch a normal application stack.** CloudFront routes HTTP requests and caches content fetched from an origin. It can reduce content-delivery latency, but it does not relocate the application's general-purpose compute resources or satisfy a requirement that prohibits deploying those applications in the Region.

**Local Zones and Wavelength Zones both extend a VPC, but their access patterns differ:**

- Choose **Local Zones** for compute and storage close to a city, local users, or an on-premises site.
- Choose **Wavelength Zones** when the decisive requirement is ultra-low latency from devices through a supported telecommunications provider's mobile network.

“Associated with eu-central-1” does not mean a Local Zone is an Availability Zone physically located inside the eu-central-1 Region. A Local Zone is an extension of its parent Region in another geographic location. However, service control planes and regional dependencies still matter, so real compliance must be validated against the exact regulation and architecture.

#### What to remember

**Run supported AWS resources close to a city or local users with single-digit millisecond latency = AWS Local Zones.**

**CloudFront caches and delivers content. Local Zones host application resources. Wavelength places resources at the telecommunications edge for mobile-network workloads.**

#### Simple practice: choose the correct edge option

Try this five-minute exercise without looking at the answers. For each scenario, choose **CloudFront**, **AWS Local Zones**, or **AWS Wavelength Zones**, and explain which clue made the choice clear.

1. A company runs an interactive virtual workstation application. Employees in one city need single-digit millisecond access to application compute running on EC2.
2. A worldwide news site wants images and videos cached near internet users. The application origin can remain in its AWS Region.
3. A multiplayer augmented-reality application needs ultra-low-latency communication between 5G phones and EC2 compute inside a telecommunications provider's network.
4. A company places its web servers in a Local Zone, but every request queries a database in the parent Region. Predict why the application might still miss its latency target.

<details>
<summary>Suggested answers</summary>

1. **AWS Local Zones.** The workload needs general-purpose compute close to users in a specific city.
2. **CloudFront.** The requirement is global HTTP content caching; the application itself does not need to move.
3. **AWS Wavelength Zones.** The decisive clues are 5G phones and the telecommunications provider's network.
4. The web tier is nearby, but every request still crosses the network to the parent Region for database work. End-to-end latency includes every dependency, so moving only one tier may not achieve single-digit milliseconds.

</details>

#### Optional read-only AWS CLI practice

If the AWS CLI is configured, list the Local Zones visible to the account without enabling a zone or launching resources:

```powershell
aws ec2 describe-availability-zones `
  --all-availability-zones `
  --filters Name=zone-type,Values=local-zone `
  --query "AvailabilityZones[].{Zone:ZoneName,Parent:ParentZoneName,Status:OptInStatus}" `
  --output table
```

Then answer these questions from the output:

1. Which parent Region is associated with each Local Zone?
2. Which zones are opted in, opted out, or unavailable to the account?
3. If an application uses a Local Zone for EC2 but a regional database, which part of the request path still travels to the parent Region?

This command only describes zone availability. Enabling a Local Zone or launching resources is unnecessary for this exercise.

### Question 08: Detect and remediate unencrypted EBS volumes

#### Question

A company needs a solution to enforce data encryption at rest on Amazon EC2 instances. The solution must automatically identify noncompliant resources and enforce compliance policies on findings.

Which solution will meet these requirements with the LEAST administrative overhead?

- **A.** Use an IAM policy that allows users to create only encrypted Amazon Elastic Block Store (Amazon EBS) volumes. Use AWS Config and AWS Systems Manager to automate the detection and remediation of unencrypted EBS volumes.
- **B.** Use AWS Key Management Service (AWS KMS) to manage access to encrypted Amazon Elastic Block Store (Amazon EBS) volumes. Use AWS Lambda and Amazon EventBridge to automate the detection and remediation of unencrypted EBS volumes.
- **C.** Use Amazon Macie to detect unencrypted Amazon Elastic Block Store (Amazon EBS) volumes. Use AWS Systems Manager Automation rules to automatically encrypt existing and new EBS volumes.
- **D.** Use Amazon Inspector to detect unencrypted Amazon Elastic Block Store (Amazon EBS) volumes. Use AWS Systems Manager Automation rules to automatically encrypt existing and new EBS volumes.

#### Correct answer: A

Use an IAM policy to prevent users from creating unencrypted EBS volumes. Use the AWS Config managed rule `encrypted-volumes` to evaluate attached EBS volumes and mark unencrypted volumes as noncompliant. Associate an AWS Systems Manager Automation runbook with the Config rule to remediate findings automatically.

This solution combines three types of control:

- **Preventive control — IAM:** Denies or does not allow requests that create unencrypted volumes.
- **Detective control — AWS Config:** Continuously evaluates recorded EBS volume configurations against the encryption rule.
- **Corrective control — Systems Manager Automation:** Runs a managed or custom remediation workflow for noncompliant resources.

AWS Config and Systems Manager provide native compliance and remediation integration, which reduces the custom code and operational work required by a Lambda-based solution.

#### Understand the terms

- **Encryption at rest:** Protecting stored data so that it is unreadable without the appropriate cryptographic key.
- **EBS volume:** Persistent block storage commonly attached to an EC2 instance as a disk.
- **Noncompliant resource:** A resource whose configuration violates a defined rule or policy.
- **AWS Config rule:** A rule that evaluates recorded AWS resource configurations and reports `COMPLIANT` or `NON_COMPLIANT` status.
- **Systems Manager Automation runbook:** A document containing automated operational steps that can remediate a resource.
- **IAM policy condition:** A rule within an access policy that allows or denies an action based on request attributes, such as whether an EBS volume is encrypted.

#### How the solution works

1. A user or automation requests a new EBS volume.
2. The IAM policy permits the request only when encryption is enabled, preventing new violations through those identities.
3. AWS Config records EBS volume configuration changes.
4. The `encrypted-volumes` managed rule checks attached volumes. It reports a volume as noncompliant if it is unencrypted or, when configured with a KMS key ID, if it does not use the required key.
5. AWS Config invokes the associated Systems Manager Automation remediation for the finding.
6. The remediation workflow creates encrypted replacement storage as required by its runbook and the company validates or completes the safe volume replacement process.

EBS encryption cannot simply be switched on for an existing unencrypted volume in place. A practical remediation normally involves creating a snapshot, copying it with encryption, creating an encrypted volume from that snapshot, and replacing the old volume. This workflow can affect attached workloads, so the Automation runbook must handle application-specific stop, detach, attach, validation, and rollback requirements safely.

#### Why each option is right or wrong

| Option | Assessment |
|---|---|
| **A: IAM + AWS Config + Systems Manager** | **Correct.** IAM prevents users from creating new unencrypted volumes. AWS Config provides a managed EBS encryption compliance rule, and its native Systems Manager Automation integration can remediate noncompliant findings with less custom administration. |
| **B: KMS + Lambda + EventBridge** | **Incorrect for least overhead.** KMS manages encryption keys and access to them; it does not by itself evaluate whether every EBS volume is encrypted. Lambda and EventBridge could support a custom solution, but the company would need to design, code, test, monitor, and maintain the detection and remediation workflow. |
| **C: Macie + Systems Manager** | **Incorrect.** Amazon Macie discovers and helps protect sensitive data in Amazon S3. It is not the service for evaluating whether EBS volumes are encrypted. |
| **D: Inspector + Systems Manager** | **Incorrect.** Amazon Inspector scans supported workloads for software vulnerabilities and unintended network exposure. It is not an EBS configuration-compliance service and does not identify unencrypted EBS volumes for this use case. |

AWS references: [AWS Config `encrypted-volumes` managed rule](https://docs.aws.amazon.com/config/latest/developerguide/encrypted-volumes.html), [remediating noncompliant resources with AWS Config and Systems Manager Automation](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html), [setting up automatic AWS Config remediation](https://docs.aws.amazon.com/config/latest/developerguide/setup-autoremediation.html), and [EC2 IAM condition keys, including `ec2:Encrypted`](https://docs.aws.amazon.com/service-authorization/latest/reference/list_ec2.html).

#### How to reason through the question

1. **“Encryption at rest on EC2 instances”** points to the attached persistent storage: EBS volumes.
2. **“Automatically identify noncompliant resources”** points to AWS Config, which evaluates resource configurations against compliance rules.
3. **“Enforce compliance policies on findings”** points to AWS Config automatic remediation through Systems Manager Automation.
4. **“Enforce” for new resources** also requires prevention. The IAM policy stops permitted users from requesting unencrypted EBS volumes.
5. **“LEAST administrative overhead”** favors the AWS managed Config rule and native remediation integration over a custom Lambda and EventBridge workflow.
6. Eliminate C and D because Macie and Inspector do not evaluate EBS encryption configuration.

#### Common exam trap

**KMS manages keys; AWS Config evaluates resource configuration.** Seeing the word “encryption” can make KMS appear to be the entire answer, but KMS does not inventory EBS volumes and report which ones violate an encryption policy.

**Prevention, detection, and remediation are different controls:**

- IAM can prevent authorized identities from creating an unencrypted volume.
- AWS Config can find existing or newly changed resources that are noncompliant.
- Systems Manager Automation can run the corrective workflow.

An IAM restriction alone does not repair existing volumes and might not cover resource creation through every principal unless policies are applied comprehensively. Conversely, automatic remediation alone is weaker than preventing the invalid configuration in the first place.

Also remember that existing EBS volumes are not encrypted in place. Remediation generally creates encrypted replacement resources, so automation must account for workload interruption and safe replacement.

#### What to remember

**AWS resource configuration compliance = AWS Config. Automatic correction of Config findings = Systems Manager Automation.**

**Use IAM for prevention, AWS Config for detection, and Systems Manager for remediation. Macie is for sensitive data in S3; Inspector is for vulnerability management.**

