# AWS SAP-C02 High-Yield Exam Scenario & Distractor Breakdown Dossier

The AWS Certified Solutions Architect - Professional (SAP-C02) exam is not a test of service definitions; it is an endurance examination in **evaluating architectural trade-offs under rigid constraints**. A single question often spans two paragraphs of multi-account, hybrid-network, and regulatory requirements, followed by four complex multi-sentence choices.

In exam preparation dumps (such as ExamTopics), **up to 25% of the "official" default answers are provably wrong, outdated, or reference deprecated services** (e.g., suggesting AWS Server Migration Service [SMS] instead of AWS MGN, or CloudWatch Events instead of EventBridge). The true architectural answer is revealed in the community discussion debates through rigorous architectural citations.

This dossier breaks down **10 canonical, recurring SAP-C02 scenario archetypes** across all 4 exam domains. Each entry details:
1. **The Question Scenario & Requirements**
2. **The 4 Architecture Choices**
3. **The Flawed Default Answer Trap (Where Dumps Fail)**
4. **The Canonical Architectural Answer & Community Consensus**
5. **Exhaustive Distractor Deconstruction (Why Options Fail)**
6. **Staff+ Exam Keyword Triggers**

---

## Scenario 1: Multi-Account Egress Traffic Deep Packet Inspection (Domain 1)

### The Scenario
A global enterprise operates 80 AWS accounts across `us-east-1` organized under AWS Organizations. Each account owns multiple VPCs hosting production microservices. Security compliance mandates that all outbound internet traffic from all VPCs across all accounts must undergo stateful deep packet inspection (DPI) through a cluster of third-party firewall appliances. The solution must support auto-scaling of the firewall fleet, preserve the original client source IP address without performing Source NAT (SNAT), and ensure that return traffic flows symmetrically back to the same firewall instance that inspected the initial request.

Which combination of steps meets these requirements with the LEAST operational overhead?

### The Options
- **A**: Deploy an Application Load Balancer (ALB) in each spoke VPC. Forward outbound traffic to the ALB. Configure the ALB to route to the third-party firewall instances across VPC Peering connections.
- **B**: Create a central Inspection VPC containing a Gateway Load Balancer (GWLB) backed by an Auto Scaling group of third-party firewalls. Create a transit gateway in the central network account and share it via AWS RAM. Attach all spoke VPCs and the Inspection VPC to the transit gateway. Enable Appliance Mode on the Inspection VPC transit gateway attachment. Route outbound traffic (`0.0.0.0/0`) from spoke VPC route tables to the transit gateway.
- **C**: Deploy an AWS Network Firewall in each spoke VPC in a dedicated firewall subnet. Route `0.0.0.0/0` from application subnets to the firewall endpoint. Configure stateful 5-tuple inspection rules.
- **D**: Create a central Inspection VPC with a Network Load Balancer (NLB) in front of the firewalls. Attach all spoke VPCs to a transit gateway. Route spoke internet traffic to the transit gateway, into the NLB, through the firewalls, and out through a central NAT Gateway.

---

### The Flawed Answer Trap
- *Flawed Dump Answer*: Often marks **Option D** (NLB with NAT Gateway).
- *Why it is Wrong*: An NLB does not natively preserve client source IPs for intermediate transparent routing without modifying packets or requiring proxy protocol support. Furthermore, standard TGW routing without Appliance Mode causes **asymmetric routing** (request enters Firewall A in AZ-1; response returns to Firewall B in AZ-2), causing the stateful firewall to drop the TCP session via `RST`.

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: ALBs operate at Layer 7 and perform reverse-proxy termination with Source NAT (SNAT). The original client source IP is replaced with the ALB's private IP (relegating client IP to `X-Forwarded-For` headers, which cannot be inspected by Layer 3/4 stateful firewalls). Furthermore, managing 80+ VPC peering connections violates the "least operational overhead" constraint.
2. **Option C is disqualified**: While AWS Network Firewall supports stateful inspection, deploying dedicated endpoints and firewall instances across hundreds of VPCs in 80 accounts multiplies licensing costs and management overhead exponentially compared to a centralized inspection hub.
3. **Option D is disqualified**: Fails to enable **Transit Gateway Appliance Mode**. Without Appliance Mode, bidirectional traffic hashes across different AZ attachments, breaking stateful firewall inspection.
4. **Option B is the Staff+ Winner**:
   - **Gateway Load Balancer (GWLB)** uses the **GENEVE protocol (UDP port 6081)** to transparently encapsulate packets, preserving the original IP headers without SNAT.
   - **TGW Appliance Mode** guarantees that request and response flows route symmetrically to the exact same firewall instance in the same AZ.
   - Centralizing in one Inspection VPC shared via AWS RAM minimizes firewall licensing costs and management effort.

> [!TIP]
> **Exam Keyword Trigger**: `"Inspect outbound traffic without SNAT"` + `"Third-party firewall auto-scaling"` + `"Stateful symmetric routing"` $\implies$ **Gateway Load Balancer (GWLB) + TGW with Appliance Mode**.

---

## Scenario 2: Legacy Oracle Database Heterogeneous Modernization (Domain 4)

### The Scenario
A financial services firm is migrating a mission-critical on-premises Oracle 19c database (12TB) to AWS. The leadership team wants to eliminate commercial database licensing costs and modernize onto an open-source compatible engine on Amazon Aurora. The database contains over 300 stored procedures, complex PL/SQL packages, foreign keys, and material views. The migration must complete with less than 15 minutes of cutover downtime.

Which architectural strategy should the Solutions Architect execute?

### The Options
- **A**: Use the AWS Application Migration Service (MGN) to perform continuous block-level replication of the Oracle operating system to an Amazon EC2 instance. After cutover, run an automated script to export data to Amazon Aurora MySQL.
- **B**: Use the AWS Schema Conversion Tool (SCT) to convert the Oracle database schema, views, and PL/SQL code to Amazon Aurora PostgreSQL. Deploy an AWS Database Migration Service (DMS) replication instance to perform a Full Load plus ongoing Change Data Capture (CDC). Once replication lag reaches zero, schedule a 15-minute maintenance window to stop on-prem writes and point applications to the Aurora PostgreSQL cluster.
- **C**: Take an Oracle Recovery Manager (RMAN) backup of the on-premises database. Upload the backup files to Amazon S3 using AWS DataSync. Restore the RMAN backup directly into an Amazon Aurora PostgreSQL database.
- **D**: Use AWS DMS to migrate the schema, tables, and PL/SQL stored procedures directly from on-premises Oracle to Amazon Aurora MySQL using an ongoing replication task.

---

### The Flawed Answer Trap
- *Flawed Dump Answer*: Sometimes marks **Option D**, assuming AWS DMS converts stored procedures and schemas automatically.
- *Why it is Wrong*: **AWS DMS DOES NOT convert PL/SQL stored procedures, triggers, or views!** DMS only migrates table structures and data rows.

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: AWS MGN performs server-level rehosting to EC2, which retains Oracle and does not achieve the goal of eliminating commercial licensing fees or moving to Aurora.
2. **Option C is disqualified**: An Oracle RMAN binary backup is proprietary to Oracle. It cannot be restored into Amazon Aurora PostgreSQL!
3. **Option D is disqualified**: AWS DMS does not translate heterogeneous database objects (stored procedures, functions, packages). Attempting to run DMS without SCT will fail to convert the 300+ PL/SQL packages.
4. **Option B is the Staff+ Winner**:
   - **AWS SCT** translates the proprietary Oracle PL/SQL schema and procedures into PostgreSQL syntax (PL/pgSQL).
   - **AWS DMS (Full Load + CDC)** streams data and changes continuously via Oracle LogMiner/redo logs.
   - The final cutover takes under 5 minutes (stopping on-prem writes, verifying zero CDC lag, and updating DNS).

> [!TIP]
> **Exam Keyword Trigger**: `"Heterogeneous migration (Oracle -> Aurora)"` + `"Stored procedures / PL/SQL"` $\implies$ **AWS SCT (for schema/code) + AWS DMS (Full Load + CDC for data)**.

---

## Scenario 3: Hybrid DNS Resolution for Cross-Cloud Workloads (Domain 1)

### The Scenario
A company has a multi-account AWS environment connected to an on-premises data center via a 10 Gbps AWS Direct Connect Dedicated Connection with a Transit VIF.
- On-premises servers query an on-premises Active Directory DNS server (`192.168.10.2`) to resolve internal domains (`corp.example.com`).
- AWS EC2 instances in private VPCs must resolve records in Route 53 Private Hosted Zones (`prod.aws.internal`).
- Simultaneously, on-premises developers must resolve AWS private services (`*.prod.aws.internal`), and AWS applications must resolve on-premises databases (`db.corp.example.com`).

How should the architect implement this with the LEAST administrative complexity?

### The Options
- **A**: Deploy two BIND DNS forwarder EC2 instances in each spoke VPC. Configure conditional forwarding on the BIND servers to point `corp.example.com` to `192.168.10.2`. Configure on-premises DNS to forward `prod.aws.internal` to the BIND EC2 instances.
- **B**: In the central network account, create Route 53 Resolver Inbound Endpoints and Outbound Endpoints in multi-AZ subnets. Configure on-premises DNS to conditionally forward queries for `prod.aws.internal` to the Inbound Endpoint IP addresses. Create a Route 53 Resolver Forwarding Rule for `corp.example.com` pointing to `192.168.10.2` using the Outbound Endpoints. Share the rule with all AWS accounts via AWS RAM.
- **C**: Create an AWS Managed Microsoft AD in the AWS cloud. Establish a two-way forest trust between the AWS Managed AD and the on-premises Active Directory over the Direct Connect connection.
- **D**: Use Route 53 Public Hosted Zones for `prod.aws.internal` with split-horizon routing policies enabled, using geolocation routing to detect whether queries originate on-premises.

---

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: Deploying and maintaining self-managed BIND DNS servers across dozens of spoke VPCs incurs massive operational overhead, patching burdens, and lacks high availability.
2. **Option C is disqualified**: Establishing an Active Directory forest trust solves Windows authentication/Kerberos, but is massive overkill and does not resolve Route 53 Private Hosted Zones for non-domain-joined Linux instances.
3. **Option D is disqualified**: Public Hosted Zones cannot resolve RFC 1918 private IP addresses securely and expose internal domain namespaces to the public internet.
4. **Option B is the Staff+ Winner**:
   - **Inbound Endpoints**: Allows on-prem DNS to query Route 53 Private Hosted Zones.
   - **Outbound Endpoints + Forwarding Rules**: Allows AWS VPCs to resolve on-premises Active Directory names.
   - **AWS RAM Sharing**: Allows creating the rule once in the central network account and sharing it with all 80 accounts seamlessly.

> [!TIP]
> **Exam Keyword Trigger**: `"Hybrid DNS resolution (On-prem <-> AWS)"` $\implies$ **Route 53 Resolver Inbound Endpoint (On-Prem -> AWS) + Outbound Endpoint & Forwarding Rules (AWS -> On-Prem) shared via RAM**.

---

## Scenario 4: Multi-Region Active-Active with Near-Zero RPO/RTO (Domain 3)

### The Scenario
An e-commerce platform requires a multi-region disaster recovery architecture across `us-east-1` and `us-west-2`. The application consists of a stateless containerized API running on Amazon ECS and a high-velocity transactional database handling shopping cart checkouts. The business mandates an **RPO < 1 second** and an **RTO of near zero (sub-minute)**. If a regional cloud failure occurs, user requests must be redirected immediately without waiting for DNS cache expiration.

Which architecture satisfies these requirements?

### The Options
- **A**: Deploy ECS in `us-east-1` (Primary) and keep a scaled-down ECS fleet in `us-west-2` (Warm Standby). Use Amazon Aurora Global Database with an unassisted failover script. Use Route 53 with Failover Routing and a 60-second TTL.
- **B**: Deploy ECS tasks in both `us-east-1` and `us-west-2`. Use **AWS Global Accelerator** with endpoints pointing to Application Load Balancers in both regions. Use **Amazon DynamoDB Global Tables** for the shopping cart database. Use Route 53 Application Recovery Controller (ARC) routing controls to manage traffic distribution.
- **C**: Deploy ECS in both regions. Use an Amazon RDS Multi-AZ DB cluster in `us-east-1` with a cross-region read replica in `us-west-2`. Use Route 53 Geoproximity routing with traffic bias.
- **D**: Deploy ECS in `us-east-1` only. Back up all data to an S3 bucket with S3 Cross-Region Replication enabled. In the event of a disaster, use AWS CloudFormation StackSets to deploy the ECS fleet in `us-west-2`.

---

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: Standard Route 53 DNS failover is subject to client and ISP DNS caching. Even with a 60-second TTL, client resolvers frequently cache records for 10–15 minutes, violating the sub-minute RTO mandate. Furthermore, Warm Standby requires time to scale up tasks.
2. **Option C is disqualified**: Standard RDS cross-region read replicas use asynchronous replication with replication lag that can exceed seconds to minutes under write bursts. In addition, standard RDS read replicas cannot accept writes without manual promotion.
3. **Option D is disqualified**: This is a classic Backup & Restore strategy with an RTO of hours, completely violating the near-zero requirement.
4. **Option B is the Staff+ Winner**:
   - **AWS Global Accelerator**: Provides static Anycast IP addresses. Traffic is routed over the private AWS global backbone; health checks detect regional failure and reroute traffic in **under 30 seconds**, completely bypassing DNS TTL caching!
   - **DynamoDB Global Tables**: Delivers active-active multi-region writes with sub-second replication lag ($RPO < 1\text{s}$).
   - **Route 53 ARC**: Provides deterministic, cell-based routing controls to isolate failing regions instantly.

> [!TIP]
> **Exam Keyword Trigger**: `"Instantaneous failover bypassing DNS TTL caching"` + `"Active-Active Multi-Region"` $\implies$ **AWS Global Accelerator + DynamoDB Global Tables + Route 53 ARC**.

---

## Scenario 5: Ransomware-Proof Data Archival & WORM Compliance (Domain 3)

### The Scenario
A healthcare compliance officer requires that electronic medical records (EMR) uploaded to Amazon S3 be retained for exactly 7 years. The data must be protected against accidental deletion, deliberate tampering, or ransomware attacks—even if an attacker gains access to the AWS account's administrative IAM credentials or the AWS Organizations Root user credentials. Furthermore, storage costs must be minimized because records are rarely accessed after 90 days.

Which combination of steps achieves this?

### The Options
- **A**: Store the records in S3 Standard. Enable S3 Versioning and S3 MFA Delete on the bucket. Configure a lifecycle rule to transition objects to S3 Glacier Flexible Retrieval after 90 days.
- **B**: Store the records in an S3 bucket with **S3 Object Lock enabled in Compliance Mode** with a retention period of 7 years. Attach an S3 Lifecycle policy to transition objects to **S3 Glacier Flexible Retrieval** after 90 days. Restrict root account access with an SCP.
- **C**: Create an AWS Backup vault with **Vault Lock enabled in Governance Mode**. Configure backup policies to take daily snapshots of EMR file systems and retain them for 7 years.
- **D**: Store the records in S3 Standard. Enable Client-Side Encryption using an AWS KMS Customer Managed Key (CMK). Deny the `kms:ScheduleKeyDeletion` permission across all IAM roles.

---

### The Flawed Answer Trap
- *Flawed Dump Answer*: Recommends **Option A** (S3 MFA Delete).
- *Why it is Wrong*: **MFA Delete can still be disabled by the AWS Root Account!** If an attacker compromises root credentials or an administrator goes rogue, MFA Delete does not prevent the root user from disabling MFA Delete and deleting the bucket.

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: MFA Delete does not provide WORM compliance and does not prevent the root account from overriding settings.
2. **Option C is disqualified**: **Governance Mode** allows users with specific IAM permissions to override or delete retention settings. Only **Compliance Mode** prevents administrative override.
3. **Option D is disqualified**: KMS key deletion protections prevent deleting the key, but do not prevent someone from deleting the underlying S3 objects!
4. **Option B is the Staff+ Winner**:
   - **S3 Object Lock in Compliance Mode**: Enforces SEC Rule 17a-4 WORM compliance. **No user—including the AWS account root user—can overwrite or delete the object during the 7-year retention period**.
   - **S3 Glacier Flexible Retrieval**: S3 Object Lock retention settings persist even after objects transition to Glacier, achieving maximum storage cost reduction while maintaining legal immutability.

> [!TIP]
> **Exam Keyword Trigger**: `"Protected against deletion even by Root / Admins"` + `"WORM compliance"` $\implies$ **S3 Object Lock / AWS Backup Vault Lock in COMPLIANCE Mode (Never Governance Mode)**.

---

## Scenario 6: High-Throughput Microservice Bursting & Cold Start Elimination (Domain 2)

### The Scenario
A financial trading platform deploys a payment authorization API on AWS Lambda integrated with an Amazon API Gateway HTTP API. During market open (9:30 AM EST), traffic spikes instantly from 50 requests/sec to 8,000 requests/sec within 10 seconds. During this surge, customers experience latency spikes exceeding 3 seconds, and some API calls return `HTTP 500` and `HTTP 429` errors. The application code is written in Java 21 Spring Boot.

Which set of architectural modifications will eliminate latency spikes and throttling during market open with the LEAST cost?

### The Options
- **A**: Migrate the entire application from AWS Lambda to an Amazon ECS on Fargate cluster. Configure an Auto Scaling policy based on CPU utilization with a target of 50%.
- **B**: Configure **Provisioned Concurrency** on the Lambda function with an Application Auto Scaling scheduled action to pre-warm 4,000 execution environments at 9:15 AM EST. Set the function's memory to 3,008 MB. Request a regional Lambda concurrency service quota increase from AWS Support.
- **C**: Increase the Lambda function timeout from 30 seconds to 15 minutes. Deploy an Amazon ElastiCache Redis cluster to cache payment authorization tokens.
- **D**: Replace API Gateway HTTP API with an Application Load Balancer. Enable Lambda Reserved Concurrency set to 100 to ensure the database is not overwhelmed.

---

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: ECS Fargate task scaling takes 45 to 90 seconds to launch new containers. An instantaneous spike in 10 seconds will overwhelm the initial containers before auto-scaling can respond. Furthermore, migrating a serverless app to ECS introduces container management overhead.
2. **Option C is disqualified**: Increasing the timeout does not solve cold starts or concurrency throttling; in fact, long timeouts cause concurrency slots to remain occupied longer, exacerbating `429` throttling!
3. **Option D is disqualified**: Setting Reserved Concurrency to 100 caps the function at 100 concurrent requests; an 8,000 req/sec surge will result in massive `429` throttling drops!
4. **Option B is the Staff+ Winner**:
   - Java Spring Boot cold starts are notorious for multi-second initialization latency.
   - **Provisioned Concurrency** pre-warms the JVM runtime, completing `Init` before traffic hits.
   - **Scheduled Scaling** at 9:15 AM ensures execution environments are warm 15 minutes before market open.
   - Requesting a quota increase guarantees the regional pool can handle the 8,000 TPS surge.

> [!TIP]
> **Exam Keyword Trigger**: `"Predictable sudden spike"` + `"Java Lambda latency / cold starts"` $\implies$ **Provisioned Concurrency + Scheduled Auto Scaling**.

---

## Scenario 7: Offline PB-Scale Data Center Evacuation (Domain 4)

### The Scenario
A media conglomerate must shut down an on-premises data center within 45 days. The facility contains **600 TB of raw video footage** stored on local NAS appliances that must be migrated into Amazon S3 Standard. The data center has a dedicated 100 Mbps internet connection, and the corporate security policy restricts cloud migration network bandwidth consumption to a maximum of 50% of the available uplink.

Which migration strategy will successfully transfer the data before the deadline?

### The Options
- **A**: Install the AWS DataSync agent on an on-premises VM. Schedule continuous synchronization to Amazon S3 with bandwidth throttling set to 50 Mbps.
- **B**: Order 8 **AWS Snowball Edge Storage Optimized** devices (80 TB usable each). Copy data locally from the NAS to the Snowball devices over 10 GbE interfaces. Ship the devices back to AWS for ingestion into S3.
- **C**: Order an AWS Snowmobile truck to park outside the data center. Connect via optical fiber to extract the 600 TB in a single transfer.
- **D**: Upgrade the internet connection to 1 Gbps. Use `rclone` with multipart upload directly into an S3 bucket with S3 Transfer Acceleration enabled.

---

### Canonical Correct Answer: **Option B**

### Mathematical Verification & Distractor Deconstruction
1. **Mathematical Proof against Option A**:
   - Available bandwidth = $50\text{ Mbps} = 6.25\text{ MB/sec}$.
   - Total data = $600\text{ TB} = 600,000,000\text{ MB}$.
   - Transfer time:
     $$\text{Time} = \frac{600,000,000\text{ MB}}{6.25\text{ MB/s}} = 96,000,000\text{ seconds} \approx \mathbf{1,111\text{ Days (Over 3 Years!)}}$$
   - Option A will miss the 45-day deadline by more than 1,000 days!
2. **Option C is disqualified**: AWS Snowmobile is reserved for **exabyte-scale datasets (typically > 10 PB)**. AWS will reject a Snowmobile request for 600 TB.
3. **Option D is disqualified**: Upgrading an enterprise physical fiber connection typically takes 60–90 days for telecommunications providers to dig and provision, exceeding the 45-day facility closure deadline.
4. **Option B is the Staff+ Winner**:
   - 600 TB divided by 80 TB usable per Snowball Edge = **8 Snowball Edge devices**.
   - Copying 80 TB over 10 GbE local network takes under **24 hours per device**.
   - Total shipping and ingestion cycle takes **under 10 to 14 days**, easily beating the 45-day deadline.

> [!TIP]
> **Exam Keyword Trigger**: Always compute network math! If $\text{Transfer Days} > 7\text{--}10\text{ days}$ $\implies$ **AWS Snowball Edge Storage Optimized**.

---

## Scenario 8: Centralized Secrets Management with Automated Cross-Account Rotation (Domain 3)

### The Scenario
An enterprise deploys a multi-account architecture with an RDS PostgreSQL database running in a central Shared Data account (`Account-A`). Over 20 microservices running in separate Workload accounts (`Account-B`, `Account-C`, `Account-D`) require access to the database. Compliance requires database credentials to rotate every 30 days automatically without application downtime or hard-coded secrets.

Which solution meets these requirements with the LEAST operational overhead?

### The Options
- **A**: Store credentials in AWS Systems Manager Parameter Store in `Account-A` as `SecureString` parameters. Write a custom cron job on an EC2 instance that runs monthly to update the RDS password and parameter value.
- **B**: Store database credentials in **AWS Secrets Manager in `Account-A`**. Enable automatic rotation using the pre-built RDS PostgreSQL rotation Lambda function. Configure a resource-based policy on the secret in `Account-A` allowing `secretsmanager:GetSecretValue` to the IAM roles in `Account-B`, `Account-C`, and `Account-D`. Grant KMS `kms:Decrypt` permissions on the CMK used to encrypt the secret to the external accounts.
- **C**: Deploy an AWS Secrets Manager secret in each individual workload account (`Account-B`, `Account-C`, `Account-D`). Use EventBridge to synchronize secret rotations across all accounts simultaneously.
- **D**: Hardcode a single master database user and password in AWS AppConfig. Deploy an IAM policy allowing workload accounts to query AppConfig.

---

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: SSM Parameter Store does not natively support automated database credential rotation; writing custom cron scripts on EC2 creates unmanaged operational overhead.
2. **Option C is disqualified**: Duplicating secrets across 20 accounts creates synchronization race conditions where one account rotates the password, instantly locking out the other 19 accounts!
3. **Option D is disqualified**: Hardcoding credentials violates basic security compliance and does not support rotation.
4. **Option B is the Staff+ Winner**:
   - **AWS Secrets Manager** supports native cross-account access via **Resource-Based Policies** on the secret itself.
   - Pre-built Lambda rotation handles the 4-step rotation protocol and dual-user switching without downtime.
   - Workload accounts retrieve the latest credentials dynamically via `GetSecretValue` using STS.

> [!TIP]
> **Exam Keyword Trigger**: `"Automated DB credential rotation"` + `"Cross-account access"` $\implies$ **AWS Secrets Manager with Resource-Based Policy + KMS cross-account decrypt**.

---

## Scenario 9: Real-Time Event Filtering & Direct 3rd-Party API Ingestion (Domain 2)

### The Scenario
A logistics company ingests 20,000 package delivery events per second. Certain high-priority events (e.g., `DeliveryFailed` with `packageValue > 1000`) must be forwarded immediately to a third-party CRM webhook (Zendesk API). The third-party API enforces a strict rate limit of **500 requests per second** and requires OAuth2 client credentials authentication. The remaining standard delivery events must be stored in an Amazon S3 data lake for end-of-day analytics.

Which design satisfies these constraints with the LEAST custom code?

### The Options
- **A**: Send all events to Amazon Kinesis Data Streams. Write an AWS Lambda consumer that parses each record. If the record matches the high-priority criteria, make an HTTPS POST request to Zendesk. Write a second Lambda consumer to write batch records to S3.
- **B**: Publish all events to an **Amazon EventBridge custom event bus**.
  1. Create Rule 1 with a content-based pattern matching `DeliveryFailed` and numeric comparison `{"packageValue": [{"numeric": [">", 1000]}]}`. Target an **EventBridge API Destination** configured with Zendesk's endpoint, OAuth2 connection credentials, and an invocation rate limit of 500 requests/sec.
  2. Create Rule 2 to forward all events to an **Amazon Kinesis Data Firehose** delivery stream buffering data into Amazon S3.
- **C**: Publish events to an Amazon SNS topic. Create two SQS queues subscribed to the topic with SNS subscription filter policies. Use an EC2 worker fleet to read from the high-priority queue and send requests to Zendesk with an in-memory token bucket rate limiter.
- **D**: Ingest events via Amazon API Gateway REST API. Configure a Velocity Template Language (VTL) mapping template to post to Zendesk directly.

---

### Canonical Correct Answer: **Option B**

### Distractor Deconstruction
1. **Option A is disqualified**: Writing custom Lambda code to parse payloads, manage OAuth token refreshes, and handle rate-limiting retries against third-party APIs introduces significant operational overhead and failure risk.
2. **Option C is disqualified**: Managing an EC2 worker fleet and writing custom token-bucket rate limiters requires manual scaling and infrastructure management.
3. **Option D is disqualified**: API Gateway cannot perform payload-based conditional routing to external APIs while managing OAuth credentials and rate limiting.
4. **Option B is the Staff+ Winner**:
   - **EventBridge Content-Based Filtering**: Natively filters JSON payloads without running any compute code.
   - **EventBridge API Destinations**: Manages OAuth2 token refreshes natively via AWS Secrets Manager and enforces built-in **rate limiting (500 req/sec)**, queuing bursts automatically.
   - **Kinesis Data Firehose**: Serverless zero-code ingestion into Amazon S3.

> [!TIP]
> **Exam Keyword Trigger**: `"Route events to 3rd party webhook with rate limit & OAuth"` $\implies$ **Amazon EventBridge API Destinations**.

---

## Scenario 10: Multi-Protocol Enterprise File Storage Migration (Domain 4)

### The Scenario
An oil and gas enterprise operates an on-premises NetApp ONTAP storage cluster hosting 150 TB of geophysical seismic data. The environment requires simultaneous, concurrent access to the same files over **both NFS and SMB protocols** from Linux analytics clusters and Windows desktop applications. The company wants to migrate this storage to AWS with zero modification to client application mount configurations, while taking advantage of inline data deduplication and block-level incremental snapshot replication.

Which AWS service should the architect select?

### The Options
- **A**: Amazon EFS with multi-AZ enabled, using Samba servers on EC2 to export SMB shares to Windows clients.
- **B**: Amazon FSx for Windows File Server configured with Multi-AZ deployment.
- **C**: **Amazon FSx for NetApp ONTAP**. Use **NetApp SnapMirror** to replicate the on-premises volumes to AWS asynchronously.
- **D**: AWS S3 File Gateway configured with both NFS and SMB endpoints pointing to the same S3 bucket.

---

### Canonical Correct Answer: **Option C**

### Distractor Deconstruction
1. **Option A is disqualified**: EFS natively supports NFS only; deploying custom Samba proxy servers on EC2 introduces an unmanaged single point of failure and high maintenance overhead.
2. **Option B is disqualified**: FSx for Windows File Server natively supports SMB only; it does not support native multi-protocol concurrent NFS access for Linux clusters.
3. **Option D is disqualified**: S3 File Gateway does not support concurrent multi-protocol file locking across NFS and SMB on the same underlying files, nor does it support block-level SnapMirror replication.
4. **Option C is the Staff+ Winner**:
   - **FSx for NetApp ONTAP** natively supports **concurrent multi-protocol access (NFS, SMB, and iSCSI)** against the exact same data volumes.
   - **NetApp SnapMirror** replicates data at the block level directly from on-premises NetApp hardware into AWS with zero code changes and minimal cutover time.
   - Delivers built-in NetApp deduplication, compression, and snapshotting.

> [!TIP]
> **Exam Keyword Trigger**: `"NetApp on-premises"` OR `"Concurrent NFS and SMB multi-protocol access"` $\implies$ **Amazon FSx for NetApp ONTAP + NetApp SnapMirror**.

---

## 7. Master SAP-C02 Exam Strategy & Trap Evasion Rules

1. **Beware the "Cheapest" vs "Least Operational Overhead" Constraint**:
   - If the prompt demands **"Least operational overhead"**, prefer managed serverless solutions (EventBridge API Destinations, GWLB, S3 Intelligent-Tiering, FSx ONTAP) over self-managed EC2 scripts, even if EC2 might appear cheaper in raw compute.
   - If the prompt demands **"Most cost-effective"**, evaluate Compute Savings Plans, Spot instances with `price-capacity-optimized` ASG, and S3 Glacier Instant Retrieval.
2. **The Deprecated Service Trap in Dumps**:
   - Dumps frequently reference **AWS SMS (Server Migration Service)** or **CloudEndure** $\implies$ The correct modern answer is **AWS Application Migration Service (MGN)**.
   - Dumps reference **CloudWatch Events** $\implies$ The correct modern answer is **Amazon EventBridge**.
   - Dumps reference **AWS SSO** $\implies$ The modern identity service is **AWS IAM Identity Center**.
3. **Verify the Community Discussion Votes on ExamTopics**:
   - Never trust the initial displayed letter on ExamTopics without reading the top-voted community comments. In tricky questions (like GWLB Appliance Mode or KMS Multi-Region CMKs), the community discussion consistently corrects flawed answer keys with direct AWS documentation quotes.
