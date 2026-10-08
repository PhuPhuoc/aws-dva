# Section 7 - AWS Integration & Messaging (SQS, SNS, Kinesis), Monitoring (CloudWatch, X-Ray, CloudTrail), AWS Lambda

---

## Phần 1: Amazon SQS - Simple Queue Service (Trang 441-457)

### SQS - Consuming Messages (Trang 441-442)

**Consumer workflow - quy trình consumer:**
1. Poll SQS for messages (nhận up to **10 messages** tại một lần)
2. Process messages (ví dụ: insert vào RDS)
3. Delete messages sử dụng **DeleteMessage API**

**Parallel Processing - xử lý song song:**
- Multiple EC2 instances có thể consume song song
- **At least once delivery** - ít nhất một lần
- **Best-effort message ordering** - cố gắng giữ thứ tự
- Scale horizontally để cải thiện throughput

### Amazon SQS - Security (Trang 445-446)

**Mã hóa - Encryption:**
- **In-flight**: HTTPS API
- **At-rest**: KMS keys
- **Client-side**: Customer tự quản lý

**Access Controls - kiểm soát truy cập:**
- **IAM policies**: Điều chỉnh SQS API access
- **SQS Access Policies**: Cross-account access, cho phép SNS/S3 ghi

### SQS - Message Visibility Timeout (Trang 447-448)

- Sau khi được poll, message trở nên **invisible** với các consumer khác
- Default: **30 seconds**
- Message có 30 giây để process
- Nếu không delete, message **xuất hiện lại** trong queue
- Có thể dùng **ChangeMessageVisibility API** để thêm thời gian

**Lưu ý:**
- Timeout quá ngắn → duplicates
- Timeout quá dài → slow re-processing on crash

### Amazon SQS - Dead Letter Queue (DLQ) (Trang 449-450)

- Nếu message fail (exceed visibility timeout), quay lại queue
- **MaximumReceives** threshold - ngưỡng
- Sau khi exceed → message đến **DLQ**
- DLQ phải match queue type:
  - FIFO queue → FIFO DLQ
  - Standard queue → Standard DLQ
- Set retention lên đến **14 days**

### SQS DLQ - Redrive to Source (Trang 450)

- Tính năng để consume DLQ messages
- Hiểu vấn đề
- Fix code
- **Redrive** messages trở về source queue

### Amazon SQS - Delay Queue (Trang 451)

- Delay message (invisible) lên đến **15 minutes**
- Default: 0 seconds (immediately available)
- Set ở queue level hoặc per message sử dụng **DelaySeconds**

### Amazon SQS - Long Polling (Trang 452)

- Consumer có thể **wait** cho messages nếu queue trống
- Giảm API calls, cải thiện efficiency
- Wait time: **1-20 seconds** (20 sec preferable)
- Enabled ở queue level hoặc sử dụng **WaitTimeSeconds**

### SQS Extended Client (Trang 453)

- Message size limit: **256KB**
- Cho messages lớn hơn (ví dụ: 1GB): Sử dụng SQS Extended Client (Java library)

### SQS Must Know API (Trang 454)

| API | Mô tả |
|-----|-------------|
| `CreateQueue` | Tạo queue (MessageRetentionPeriod) |
| `DeleteQueue` | Xóa queue |
| `PurgeQueue` | Xóa tất cả messages |
| `SendMessage` | Gửi (DelaySeconds) |
| `ReceiveMessage` | Nhận messages (max 10) |
| `DeleteMessage` | Xóa sau khi process |
| `ChangeMessageVisibility` | Thay đổi timeout |

### Amazon SQS - FIFO Queue (Trang 455-457)

**FIFO = First In First Out**

**Thuộc tính:**
- Limited throughput: **300 msg/s** (without batching)
- **3000 msg/s** (with batching)
- **Exactly-once** send capability
- Messages processed **in order**

**Deduplication - loại bỏ trùng lặp:**
- Interval: **5 minutes**
- Methods:
  - **Content-based**: SHA-256 hash của message body
  - **Explicit**: Cung cấp Message Deduplication ID

**Message Grouping - nhóm messages:**
- Same **MessageGroupID** → một consumer, ordered
- Different MessageGroupID → parallel processing, order within group

---

## Phần 2: Amazon SNS - Simple Notification Service (Trang 458-468)

### Amazon SNS Overview (Trang 459)

**Architecture:**
- **Producer** gửi đến một SNS topic
- **Multiple subscribers** nhận tất cả messages
- Up to **12,500,000 subscriptions** per topic
- **100,000 topics** limit

**Subscriber types:**
- SQS, Lambda, Kinesis Data Firehose
- Emails, SMS, HTTP(S)
- Mobile Notifications

### SNS Integrations (Trang 460)

**AWS Services có thể gửi đến SNS:**
- CloudWatch Alarms
- AWS Budgets
- Auto Scaling Group
- S3 Bucket Events
- CloudFormation State Changes
- AWS DMS
- Lambda, DynamoDB, RDS Events

### Amazon SNS - How to Publish (Trang 461)

**Topic Publish:**
1. Tạo một topic
2. Tạo subscriptions
3. Publish đến topic

**Direct Publish (cho mobile apps):**
1. Tạo platform application
2. Tạo platform endpoint
3. Publish đến platform endpoint

### Amazon SNS - Security (Trang 462)

**Mã hóa:**
- In-flight: HTTPS
- At-rest: KMS keys
- Client-side: Customer tự quản lý

**Access Controls:**
- IAM policies
- SNS Access Policies (cross-account)

### SNS + SQS: Fan Out (Trang 463)

**Pattern - mô hình:**
```
SNS Topic → SQS Queue 1 → Consumer 1
         → SQS Queue 2 → Consumer 2
         → SQS Queue 3 → Consumer 3
```

**Lợi ích:**
- Push một lần, nhận ở tất cả queues
- Fully decoupled, no data loss - tách biệt hoàn toàn
- SQS cung cấp persistence, retries
- Cross-Region delivery supported

### Amazon SNS - FIFO Topic (Trang 466)

- FIFO ordering
- Giống SQS FIFO:
  - Message Group ID cho ordering
  - Deduplication
- SQS Standard và FIFO queues as subscribers

### SNS - Message Filtering (Trang 468)

- JSON policy để filter messages
- Subscription không có filter → nhận tất cả messages

---

## Phần 3: Amazon Kinesis (Trang 469-475)

### Kinesis Data Streams (Trang 469-470)

**Tính năng:**
- Collect and store streaming data **real-time** - thu thập và lưu trữ dữ liệu streaming thời gian thực
- Retention: **1-365 days**
- Ability to **replay** data - khả năng phát lại dữ liệu
- Data up to **1MB**
- Ordering guarantee cho same Partition ID
- At-rest KMS encryption

### Kinesis Data Streams - Capacity Modes (Trang 471)

**Provisioned Mode:**
- Choose number of **shards**
- Each shard: **1MB/s in**, **2MB/s out**
- Scale manually
- Pay per shard per hour

**On-Demand Mode:**
- No provisioning needed - không cần provision
- Default capacity: **4 MB/s**
- Auto-scales based on peak
- Pay per stream + data in/out

### Amazon Data Firehose (Trang 472-473)

- Load streaming data đến các destinations
- Destinations:
  - S3, Redshift, OpenSearch
  - 3rd party (Splunk, MongoDB, Datadog)
  - Custom HTTP
- **Fully managed**, serverless
- Near real-time
- Supports CSV, JSON, Parquet, Avro
- Custom transformations via Lambda

### Kinesis Data Streams vs Firehose (Trang 474)

| Đặc điểm | Kinesis Data Streams | Firehose |
|---------|---------------------|----------|
| **Type** | Streaming collection | Load to destinations |
| **Code** | Producer & Consumer | Fully managed |
| **Real-time** | Có | Near real-time |
| **Capacity** | Provisioned/On-demand | Auto scaling |
| **Replay** | Có | Không |
| **Storage** | Up to 365 days | Không |

---

## Phần 4: SQS vs SNS vs Kinesis (Trang 476)

| Đặc điểm | SQS | SNS | Kinesis |
|---------|-----|-----|---------|
| **Type** | Queue | Pub/Sub | Stream |
| **Delivery** | Pull | Push | Pull |
| **Consumers** | 1 | Many | Many (shards) |
| **Ordering** | FIFO available | FIFO available | Per shard |
| **Replay** | Không | Không | Có |
| **Use case** | Decouple | Fan-out | Real-time analytics |

---

## Phần 5: Amazon CloudWatch (Trang 481-504)

### Amazon CloudWatch Metrics (Trang 481-482)

**Khái niệm:**
- **Namespace**: Container cho metrics (ví dụ: AWS/EC2)
- **Dimension**: Attribute của metric (up to 30)
- **Metric**: Variable để monitor (CPUUtilization, NetworkIn)
- **Timestamp**: Thời gian đo lường

**EC2 Metrics:**
- Standard: CPU, Disk, Network
- **Không thể monitor memory** theo default
- Custom metrics cho RAM, disk usage

### EC2 Detailed Monitoring (Trang 484)

- Standard: **5 minutes**
- Detailed: **1 minute**
- Free Tier: 10 detailed monitoring metrics
- Use cho faster ASG scaling

### CloudWatch Custom Metrics (Trang 485)

**Hai cách:**
- **PutMetricData API** (SDK, CLI)
- **CloudWatch Agent** (recommended cho EC2)

**Resolution - độ phân giải:**
- Standard: **1 minute**
- High Resolution: **1/5/10/30 seconds** (higher cost)

### CloudWatch Logs (Trang 486-488)

**Khái niệm:**
- **Log groups**: Container cho related logs
- **Log stream**: Individual log sequence
- Expiration: Never expire đến 10 years

**Sources - các nguồn:**
- SDK, CloudWatch Logs Agent, Unified Agent
- Elastic Beanstalk, ECS, Lambda
- VPC Flow Logs, API Gateway
- CloudTrail, Route53

### CloudWatch Logs Insights (Trang 489-490)

- Purpose-built query language
- Auto-discovers fields từ JSON logs
- Calculate statistics, filter, sort
- Save queries, add to dashboards
- Query multiple accounts/regions

### CloudWatch Logs for EC2 (Trang 495-496)

**Yêu cầu:**
1. Run CloudWatch agent trên EC2
2. IAM permissions đúng
3. Configuration file cho metrics/logs

**Unified Agent thu thập:**
- CPU, Disk metrics, RAM
- Netstat, Processes, Swap Space

### CloudWatch Logs Metric Filter (Trang 498)

- Filter expressions cho logs
- Trigger CloudWatch Alarms
- **Does not retroactively filter** - không lọc dữ liệu cũ

### CloudWatch Alarms (Trang 499-503)

**States - các trạng thái:**
- **OK**
- **INSUFFICIENT_DATA**
- **ALARM**

**Targets - các mục tiêu:**
- Stop, Terminate, Reboot, Recover EC2
- Trigger Auto Scaling
- Send SNS notification

**EC2 Instance Recovery:**
- **Instance status**: Check EC2 VM
- **System status**: Check hardware
- Recovery restores same IP, metadata

**Composite Alarms:**
- Monitor multiple alarms
- AND/OR conditions
- Reduce alarm noise - giảm tiếng ồn alarm

### CloudWatch Synthetics Canary (Trang 504)

- Configurable scripts monitor APIs, URLs, Websites
- Reproduce customer behavior - tái tạo hành vi khách hàng
- Check availability and latency
- Integration with CloudWatch Alarms
- Scripts: Node.js or Python

---

## Phần 6: Amazon EventBridge (Trang 505-509)

### Amazon EventBridge Overview (Trang 505)

**Hai patterns - hai mô hình:**
- **Schedule**: Cron jobs (scheduled scripts)
- **Event Pattern**: React to service events - phản ứng với các sự kiện

**Targets - các mục tiêu:**
- Lambda functions
- SQS/SNS messages

### Amazon EventBridge Features (Trang 507-509)

**Event Buses:**
- Default event bus cho AWS services
- Partner event buses cho SaaS
- Custom event buses

**Archive & Replay:**
- Archive events (all or filtered)
- Replay archived events

**Schema Registry:**
- Infer schema từ events
- Generate code với known structure
- Versioning support

**Resource-based Policies:**
- Cross-account access
- Aggregate events from Organization

---

## Phần 7: AWS X-Ray (Trang 510-528)

### AWS X-Ray Overview (Trang 510-512)

**X-Ray làm gì:**
- Trace & analyze requests - theo dõi và phân tích requests
- Visualize request flow across services - trực quan hóa luồng request
- Identify bottlenecks - nhận diện điểm nghẽn
- Troubleshoot performance issues - khắc phục vấn đề hiệu suất

**Lợi ích:**
- Troubleshooting performance
- Understand microservice dependencies
- Pinpoint service issues
- Review request behavior
- Find errors and exceptions
- SLA monitoring

### X-Ray Compatibility (Trang 513)

- AWS Lambda, Elastic Beanstalk, ECS
- ELB, API Gateway
- EC2 Instances, on-premises

### AWS X-Ray - How it Works (Trang 514-516)

**Components - các thành phần:**
1. **SDK** in application code
2. **X-Ray Daemon** (UDP packet interceptor)

**SDK captures - SDK bắt:**
- Calls to AWS services
- HTTP/HTTPS requests
- Database calls
- Queue calls

### X-Ray Concepts (Trang 519)

| Khái niệm | Mô tả |
|---------|-------------|
| **Segment** | Data from application/service |
| **Subsegment** | More details in segment |
| **Trace** | Collection of segments |
| **Sampling** | Reduce data recorded - giảm dữ liệu ghi |
| **Annotations** | Key-value pairs, indexed |
| **Metadata** | Key-value pairs, not indexed |

**Default sampling:**
- First request each second
- 5% of additional requests

### AWS Distro for Open Telemetry (Trang 527-528)

- AWS-supported distribution of OpenTelemetry
- Single set of APIs, libraries, agents
- Collect traces and metrics
- Send to multiple destinations (X-Ray, CloudWatch, Prometheus)

---

## Phần 8: AWS CloudTrail (Trang 529-535)

### AWS CloudTrail Overview (Trang 529-530)

**CloudTrail làm gì:**
- Governance, compliance, audit - quản trị, tuân thủ, kiểm toán
- **Enabled by default** - được bật theo mặc định
- History of API calls - lịch sử các API calls

**Sources - các nguồn:**
- Console, SDK, CLI, AWS Services

**Integrations:**
- CloudWatch Logs
- S3

### CloudTrail Events (Trang 531-532)

**Management Events:**
- Operations on resources
- Configure security, routing, logging
- Read vs Write events

**Data Events:**
- S3 object-level (GetObject, DeleteObject, PutObject)
- Lambda function execution
- Not logged by default (high volume)

**CloudTrail Insights:**
- Detect unusual activity - phát hiện hoạt động bất thường
- Inaccurate provisioning
- Hitting limits - chạm giới hạn
- Bursts of IAM actions

### CloudTrail Events Retention (Trang 533)

- Events stored for **90 days**
- Log to S3 + Athena cho longer retention

### CloudTrail vs CloudWatch vs X-Ray (Trang 536)

| Service | Purpose - Mục đích |
|---------|---------|
| **CloudTrail** | Audit API calls, detect unauthorized access - kiểm toán API calls |
| **CloudWatch** | Metrics over time, logs, alarms |
| **X-Ray** | Distributed tracing, latency analysis |

---

## Phần 9: AWS Lambda (Trang 539-590)

### What's Serverless? (Trang 540)

- No servers to manage - không cần quản lý servers
- Deploy code/functions
- FaaS (Function as a Service)
- Databases, messaging, storage managed

### Why AWS Lambda (Trang 542)

**vs EC2:**

| EC2 | Lambda |
|-----|--------|
| Virtual servers | Virtual functions |
| Limited by RAM/CPU | Limited by time |
| Continuously running | Run on-demand |
| Manual scaling | Automated scaling |

### AWS Lambda Benefits (Trang 543)

- **Easy Pricing**: Pay per request and compute time
- **Free Tier**: 1,000,000 requests + 400,000 GB-seconds
- Integrated với AWS services
- Multiple languages support
- Easy monitoring (CloudWatch)
- Up to **10GB RAM**

### AWS Lambda Language Support (Trang 544)

- Node.js, Python, Java
- C#, PowerShell, Ruby
- Custom Runtime API
- Container Image (Lambda Runtime API)

### Lambda Pricing (Trang 548)

**Pay per calls:**
- First 1M requests: **Free**
- After: $0.20 per 1M requests ($0.0000002 per request)

**Pay per duration:**
- 400,000 GB-seconds free per month
- After: $1.00 per 600,000 GB-seconds

### Lambda - Synchronous Invocations (Trang 549-550)

**Services - các dịch vụ:**
- CLI, SDK, API Gateway, ALB
- CloudFront (Lambda@Edge)
- S3 Batch
- Cognito, Step Functions
- Lex, Alexa, Kinesis Data Firehose

**Error handling:** Client-side retries

### Lambda - Asynchronous Invocations (Trang 556)

**Services:**
- S3, SNS, CloudWatch Events/EventBridge
- CodeCommit, CodePipeline
- CloudWatch Logs, SES, CloudFormation

**Retry behavior:**
- 3 tries total - thử 3 lần
- 1 minute wait → 2 minutes wait
- **Idempotent processing required** - cần xử lý idempotent
- DLQ cho failed processing

### Lambda - Event Source Mapping (Trang 561)

**Sources - các nguồn:**
- Kinesis Data Streams
- SQS & SQS FIFO
- DynamoDB Streams

**Lambda invoked synchronously**

### Lambda - Destinations (Trang 565)

**Async invocations:**
- Successful: SQS, SNS, Lambda, EventBridge
- Failed: DLQ

**Event source mapping:**
- Discarded batches: SQS, SNS

### Lambda Execution Role (Trang 566)

**Managed policies:**
- `AWSLambdaBasicExecutionRole` - CloudWatch Logs
- `AWSLambdaKinesisExecutionRole` - Kinesis
- `AWSLambdaDynamoDBExecutionRole` - DynamoDB Streams
- `AWSLambdaSQSQueueExecutionRole` - SQS
- `AWSLambdaVPCAccessExecutionRole` - VPC
- `AWSXRayDaemonWriteAccess` - X-Ray

### Lambda Resource Based Policies (Trang 567)

- Give accounts/services permission to use Lambda
- Similar to S3 bucket policies

### Lambda Logging & Monitoring (Trang 569)

**CloudWatch Logs:**
- Execution logs stored
- IAM policy cho writes

**CloudWatch Metrics:**
- Invocations, Durations
- Concurrent Executions
- Error count, Success Rates
- Throttles, Iterator Age

### Lambda Tracing with X-Ray (Trang 570)

- Enable Active Tracing
- IAM: `AWSXRayDaemonWriteAccess`
- Environment variables cho X-Ray

### CloudFront Functions & Lambda@Edge (Trang 571-576)

**CloudFront Functions:**
- JavaScript only
- Sub-ms startup, millions req/s
- Viewer Request/Response
- Cache key normalization, header manipulation

**Lambda@Edge:**
- Node.js or Python
- 1000s req/s
- 4 phases: Viewer/Origin Request/Response
- Author in us-east-1, replicated globally

### Lambda in VPC (Trang 577-579)

**Theo mặc định:** Lambda runs outside your VPC

**Để truy cập VPC resources:**
1. Define VPC ID, Subnets, Security Groups
2. Lambda creates ENI in subnets
3. Requires `AWSLambdaVPCAccessExecutionRole`

**Internet Access - truy cập Internet:**
- Public subnet: No internet access
- Private subnet: Internet via NAT Gateway
- **VPC Endpoints**: Private access to AWS services

### Lambda Function Configuration (Trang 580)

**RAM:**
- 128MB đến **10GB** (1MB increments)
- More RAM = more vCPU
- At 1792MB = 1 full vCPU

**Timeout:** Default 3 seconds, max **900 seconds (15 minutes)**

### Lambda Execution Context (Trang 581)

- Temporary runtime environment
- Initialize external dependencies
- **Re-used cho subsequent invocations**
- Includes `/tmp` directory

### Lambda /tmp Space (Trang 583)

- Up to **10GB** storage
- Content remains when frozen
- Use cho transient cache
- For permanent storage: S3
- Encrypt với KMS Data Keys

### Lambda Layers (Trang 584)

- Externalize dependencies
- Custom runtimes (C++, Rust)
- Reuse across functions

### Lambda File Systems (Trang 585)

- Access EFS in VPC
- Configure EFS Access Points
- Watch EFS connection limits

### Lambda Concurrency (Trang 587-588)

- **1000 concurrent executions** default limit
- **Reserved concurrency**: Limit function
- Throttle behavior:
  - Sync: Return ThrottleError (429)
  - Async: Retry → DLQ

### Cold Starts & Provisioned Concurrency (Trang 590)

**Cold Start:**
- New instance: Load code, run init
- First request có higher latency

**Provisioned Concurrency:**
- Allocated before invocation
- No cold starts
- Low latency all the time
- Application Auto Scaling support
