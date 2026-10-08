# Section 7 - DVA Practice Questions: SQS, SNS, Kinesis, CloudWatch, X-Ray, CloudTrail, Lambda

---

## 1. Amazon SQS

### Question 1:
A message in SQS fails to process and exceeds the visibility timeout. What happens?

```
A. Message is deleted
B. Message is sent to DLQ
C. Message becomes visible in queue again
D. Message is archived
```

### ✅ Answer: **C. Message becomes visible in queue again**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Visibility Timeout**:
  - Message becomes invisible after being received
  - Default: **30 seconds**
  - If not deleted within timeout, message **reappears** in queue
  - Can be processed by another consumer

**Flow:**
```
1. Receive message (visible)
2. Processing starts...
3. Timeout expires (30 seconds):
   → Message reappears in queue
4. Another consumer can process it
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Deleted | Only if explicitly deleted by consumer |
| B. DLQ | Only after max receive count exceeded |
| D. Archived | No automatic archiving |

---

### Question 2:
An application needs guaranteed message ordering and exactly-once processing. Which SQS queue type should be used?

```
A. Standard Queue
B. FIFO Queue
C. Delay Queue
D. Priority Queue
```

### ✅ Answer: **B. FIFO Queue**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **FIFO Queue**:
  - **Strict ordering** (First In, First Out)
  - **Exactly-once processing**
  - No duplicate messages
  - Throughput: 300 msg/s (3000 with batching)

**Standard vs FIFO:**
| Feature | Standard | FIFO |
|---------|----------|-------|
| Ordering | Best effort | **Strict** |
| Delivery | At least once | **Exactly once** |
| Duplicates | Can occur | **No** |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Standard | At least once, best effort |
| C. Delay Queue | Visibility delay, not ordering |
| D. Priority | Not an SQS type |

---

### Question 3:
A company needs to prevent the same message from being processed after maximum retries. What should be configured?

```
A. Visibility Timeout
B. Delay Queue
C. Dead Letter Queue
D. Long Polling
```

### ✅ Answer: **C. Dead Letter Queue**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Dead Letter Queue (DLQ)**:
  - Failed messages sent to DLQ after max receives
  - For debugging and reprocessing
  - Prevents infinite retry loops

**Configuration:**
- Set **MaximumReceives** threshold
- After exceeded → message goes to DLQ
- DLQ types must match source queue type

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Visibility Timeout | Time to process, not retry limit |
| B. Delay Queue | Delay before visible, not retry |
| D. Long Polling | Waiting for messages, not retry |

---

### Question 4:
Which SQS API should be used to reduce the number of API calls when a queue is empty?

```
A. ReceiveMessage with MaxNumberOfMessages
B. ChangeMessageVisibility
C. ReceiveMessage with WaitTimeSeconds (Long Polling)
D. SendMessage with DelaySeconds
```

### ✅ Answer: **C. ReceiveMessage with WaitTimeSeconds (Long Polling)**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Long Polling**:
  - Wait for messages if queue empty
  - Reduces API calls
  - Wait time: **1-20 seconds** (20 preferable)
  - Enabled with `WaitTimeSeconds`

**Short Polling vs Long Polling:**
| Feature | Short | Long |
|---------|-------|------|
| API calls | High | Reduced |
| Latency | Higher | Lower |
| Cost | Higher | Lower |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. MaxNumberOfMessages | Gets multiple messages, doesn't wait |
| B. ChangeMessageVisibility | Changes timeout, doesn't poll |
| D. DelaySeconds | Delays message visibility |

---

## 2. Amazon SNS

### Question 5:
A company needs to send one message to multiple downstream systems. Which AWS service should be used?

```
A. SQS Queue
B. SNS Topic with multiple subscriptions
C. Kinesis Data Streams
D. Lambda Function
```

### ✅ Answer: **B. SNS Topic with multiple subscriptions**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **SNS Fan-out Pattern**:
  - One message → multiple subscribers
  - Each subscriber receives all messages
  - Subscribers: SQS, Lambda, HTTP, Email, SMS

**Architecture:**
```
Publisher → SNS Topic
              ├── SQS Queue 1 → Consumer 1
              ├── SQS Queue 2 → Consumer 2
              └── Lambda → Processing
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. SQS | Single consumer per queue |
| C. Kinesis | Streaming, not pub/sub |
| D. Lambda | Function, not messaging |

---

### Question 6:
An SNS topic needs to send messages only to subscribers with specific attributes. What feature enables this?

```
A. SNS FIFO
B. SNS Message Filtering
C. SNS Delivery Policies
D. SNS Access Policies
```

### ✅ Answer: **B. SNS Message Filtering**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **SNS Message Filtering**:
  - JSON policy to filter messages
  - Subscribers only receive matching messages
  - No filter → receives all messages

**Example filter:**
```json
{
  "filterPolicy": {
    "color": ["red", "blue"],
    "price": [{"numeric": [">=", 100]}]
  }
}
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. SNS FIFO | Ordering, not filtering |
| C. Delivery Policies | Retry behavior, not filtering |
| D. Access Policies | Permissions, not filtering |

---

## 3. Amazon Kinesis

### Question 7:
A company needs to analyze streaming data with the ability to replay past data. Which service is appropriate?

```
A. Amazon SQS
B. Amazon SNS
C. Amazon Kinesis Data Streams
D. Amazon Kinesis Data Firehose
```

### ✅ Answer: **C. Amazon Kinesis Data Streams**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Kinesis Data Streams**:
  - Collect and store streaming data
  - **Retention: 1-365 days**
  - **Ability to replay** data
  - Consumer pulls data

**Kinesis Data Streams vs Firehose:**
| Feature | Data Streams | Firehose |
|---------|-------------|----------|
| **Replay** | ✅ Yes | ❌ No |
| Storage | ✅ Yes | ❌ No |
| Real-time | ✅ Yes | Near real-time |
| Consumers | Custom | Managed |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. SQS | Queue, no replay capability |
| B. SNS | Pub/sub, no replay |
| D. Firehose | No replay, loads to destinations |

---

## 4. Amazon CloudWatch

### Question 8:
EC2 instances cannot report memory utilization to CloudWatch by default. How can this be collected?

```
A. Use EC2 detailed monitoring
B. Install CloudWatch Agent
C. Enable CloudWatch Logs
D. Create SNS subscription
```

### ✅ Answer: **B. Install CloudWatch Agent**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudWatch Agent**:
  - Collect **custom metrics** from EC2
  - Memory utilization
  - Disk space
  - Application metrics
  - Requires IAM permissions

**EC2 Default Metrics (without agent):**
- CPU utilization
- Network
- Disk (not memory!)

**Custom Metrics (with agent):**
- Memory usage
- Disk used percentage
- Custom application metrics

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Detailed monitoring | Only 1-minute intervals, not memory |
| C. CloudWatch Logs | Log files, not metrics |
| D. SNS subscription | Notifications, not metrics |

---

### Question 9:
A CloudWatch alarm is in ALARM state. What actions can be triggered?

```
A. Stop, Terminate, Reboot, Recover EC2
B. Start new EC2 instances only
C. Send emails only
D. Delete CloudWatch metrics
```

### ✅ Answer: **A. Stop, Terminate, Reboot, Recover EC2**

### 📖 Explanation (Vietnamese):

**Why A is correct:**
- **CloudWatch Alarm Actions**:
  - **Stop** EC2 instance
  - **Terminate** EC2 instance
  - **Reboot** EC2 instance
  - **Recover** EC2 instance
  - Trigger Auto Scaling
  - Send **SNS notification**

**EC2 Recovery:**
- Restores same Private/Public IP
- Same Elastic IP
- Same metadata

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B. Start new only | Not a valid action |
| C. Send emails only | Emails via SNS, not only |
| D. Delete metrics | Not an alarm action |

---

### Question 10:
Log data needs to be queried with a purpose-built query language. Which service should be used?

```
A. CloudWatch Logs
B. CloudWatch Logs Insights
C. CloudWatch Metrics
D. CloudWatch Synthetics
```

### ✅ Answer: **B. CloudWatch Logs Insights**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudWatch Logs Insights**:
  - Purpose-built query language
  - Query log data
  - Calculate statistics
  - Filter and sort
  - Save queries to dashboards

**CloudWatch Logs vs Logs Insights:**
| Feature | Logs | Logs Insights |
|---------|------|--------------|
| Store | ✅ | ✅ |
| Query | Basic | **Purpose-built** |
| Statistics | No | **Yes** |
| Multi-account | No | **Yes** |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. CloudWatch Logs | Storage, not advanced queries |
| C. CloudWatch Metrics | For metrics, not logs |
| D. CloudWatch Synthetics | For canary scripts, not logs |

---

## 5. AWS X-Ray

### Question 11:
A distributed application needs to visualize request flow across multiple services. Which service provides this?

```
A. CloudWatch Metrics
B. CloudTrail
C. AWS X-Ray
D. CloudWatch Logs
```

### ✅ Answer: **C. AWS X-Ray**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **AWS X-Ray**:
  - Trace & analyze requests
  - Visualize request flow
  - Service map visualization
  - Identify bottlenecks
  - Latency analysis

**X-Ray Concepts:**
| Concept | Description |
|---------|-------------|
| Segment | Data from service |
| Trace | Collection of segments |
| Service Map | Visual of all services |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. CloudWatch Metrics | Single service metrics |
| B. CloudTrail | Audit API calls |
| D. CloudWatch Logs | Log storage |

---

## 6. AWS CloudTrail

### Question 12:
Which CloudTrail event type logs S3 object-level operations like GetObject and PutObject?

```
A. Management Events
B. Data Events
C. Insights Events
D. Read Events
```

### ✅ Answer: **B. Data Events**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Data Events**:
  - S3 object-level activity
  - **GetObject, DeleteObject, PutObject**
  - Lambda function execution
  - **Not logged by default** (high volume)

**CloudTrail Event Types:**
| Type | Description | Default |
|------|-------------|---------|
| **Management** | Configure security, routing | ✅ Logged |
| **Data** | S3 object ops, Lambda | ❌ Not logged |
| **Insights** | Unusual activity detection | Optional |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Management | Configure resources, not object ops |
| C. Insights | Anomaly detection, not normal ops |
| D. Read Events | Subset of management, not data |

---

## 7. AWS Lambda

### Question 13:
A Lambda function needs to access resources in a VPC (RDS). What must be configured?

```
A. Enable VPC Peering
B. Define VPC ID, Subnets, and Security Groups
C. Create Internet Gateway
D. Add NAT Gateway
```

### ✅ Answer: **B. Define VPC ID, Subnets, and Security Groups**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Lambda in VPC**:
  - Define VPC ID, Subnets, Security Groups
  - Lambda creates ENI in subnets
  - Requires IAM role: `AWSLambdaVPCAccessExecutionRole`

**Lambda VPC Access:**
```
Lambda → ENI → VPC Subnet → RDS
```

**⚠️ Important:**
- Lambda in VPC has **no internet access**
- Use NAT Gateway for internet
- Use VPC Endpoints for AWS services

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. VPC Peering | Connect VPCs, not Lambda VPC access |
| C. Internet Gateway | For public instances, not Lambda in VPC |
| D. NAT Gateway | For internet, not VPC access |

---

### Question 14:
A Lambda function is invoked asynchronously and fails. How many times does Lambda retry?

```
A. 1 time
B. 3 times
C. 5 times
D. Unlimited times
```

### ✅ Answer: **B. 3 times**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Lambda Asynchronous Invocation Retry**:
  - **3 tries total**
  - 1st retry: after 1 minute
  - 2nd retry: after 2 minutes
  - Then → **DLQ** (if configured)

**Retry behavior:**
```
Invoke → Fail → Wait 1 min → Retry 1
                        → Fail → Wait 2 min → Retry 2
                                          → Fail → DLQ
```

**Requirements:**
- Function must be **idempotent**
- Configure **DLQ** for failed processing

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. 1 time | Lambda retries 3 times |
| C. 5 times | Not the default |
| D. Unlimited | No, max 3 retries |

---

### Question 15:
A Lambda function needs to run for more than the default 3 seconds. What is the maximum timeout?

```
A. 30 seconds
B. 60 seconds
C. 300 seconds (5 minutes)
D. 900 seconds (15 minutes)
```

### ✅ Answer: **D. 900 seconds (15 minutes)**

### 📖 Explanation (Vietnamese):

**Why D is correct:**
- **Lambda Timeout**:
  - Default: **3 seconds**
  - Maximum: **900 seconds (15 minutes)**
  - Configurable per function

**Lambda Limits:**
| Resource | Limit |
|----------|-------|
| Timeout | Up to 15 minutes |
| Memory | 128 MB - 10 GB |
| Concurrent executions | 1000 (default) |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. 30 seconds | Not the max |
| B. 60 seconds | Not the max |
| C. 300 seconds | Only 5 minutes, not max |

---

### Question 16:
A Lambda function has sudden traffic spikes causing throttling. How can this be prevented?

```
A. Increase timeout
B. Reserve concurrency
C. Add more memory
D. Use SQS
```

### ✅ Answer: **B. Reserve concurrency**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Lambda Concurrency**:
  - Default limit: **1000 concurrent executions**
  - **Reserved concurrency**: Guarantee capacity for function
  - Prevents throttling
  - Other functions can't use reserved amount

**Throttle behavior:**
- Sync invocation: Returns **ThrottleError (429)**
- Async invocation: Retries → DLQ

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Increase timeout | For long-running, not concurrency |
| C. Add more memory | For CPU-intensive, not concurrency |
| D. Use SQS | Queue, doesn't prevent throttling |

---

## 8. Combined Scenarios

### Question 17:
A company needs to build a serverless application that processes S3 events and stores data in DynamoDB. Which Lambda invocation type is used?

```
A. Synchronous
B. Asynchronous
C. Event source mapping
D. Stream mapping
```

### ✅ Answer: **C. Event source mapping**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Lambda Event Source Mapping**:
  - Used for: **Kinesis, SQS, DynamoDB Streams**
  - Lambda **polls** source
  - Lambda invoked **synchronously**

**When to use:**
- Kinesis Data Streams
- SQS & SQS FIFO
- DynamoDB Streams

**S3 Event Notifications:**
- S3 → **Async invocation** → Lambda
- S3 → EventBridge → Lambda

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Synchronous | For API Gateway, ALB |
| B. Asynchronous | For S3, SNS, EventBridge |
| D. Stream mapping | Not a Lambda invocation type |

---

### Question 18:
A Lambda function in a VPC needs to access S3 without going through the Internet. What is needed?

```
A. NAT Gateway
B. Internet Gateway
C. VPC Endpoint for S3
D. Lambda Layer
```

### ✅ Answer: **C. VPC Endpoint for S3**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **VPC Endpoint (Gateway)**:
  - Private access to S3
  - No Internet required
  - Traffic stays within AWS
  - **Free** for S3 Gateway Endpoint

**Lambda in VPC + S3:**
```
Lambda → VPC Endpoint (Gateway) → S3
         (No Internet needed)
```

**Options:**
| Need | Solution |
|------|----------|
| S3 from VPC | **VPC Endpoint** |
| Internet from VPC | NAT Gateway |
| AWS services from VPC | VPC Endpoints |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. NAT Gateway | For Internet, not S3 specifically |
| B. IGW | For public instances, doesn't help Lambda |
| D. Lambda Layer | For code dependencies, not networking |

---

### Question 19:
A company needs to ensure Lambda functions process messages in order within the same group. What should be configured?

```
A. SQS Standard Queue
B. SQS FIFO Queue with MessageGroupID
C. SNS Topic with filtering
D. Kinesis with multiple shards
```

### ✅ Answer: **B. SQS FIFO Queue with MessageGroupID**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **SQS FIFO with Message Grouping**:
  - Same **MessageGroupID** → Ordered
  - Different MessageGroupID → Parallel processing
  - Each group processed in order

**Use case:**
```
Order Messages → Same GroupID → Processed in Order
```

**Comparison:**
| Feature | Standard | FIFO (no grouping) | FIFO (grouping) |
|---------|----------|-------------------|----------------|
| Ordering | Best | Strict | Per group |
| Parallel | Yes | No | Yes (different groups) |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Standard Queue | Best effort, no guarantee |
| C. SNS | Fan-out, not queue-based |
| D. Kinesis | Different concept, shards for parallelism |

---

### Question 20:
A Lambda function has a cold start latency issue. Which feature allocates capacity before invocation to eliminate cold starts?

```
A. Reserved concurrency
B. Provisioned concurrency
C. Increased timeout
D. Larger memory
```

### ✅ Answer: **B. Provisioned concurrency**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Provisioned Concurrency**:
  - Allocates **before invocation**
  - **No cold starts**
  - Low latency for all invocations
  - Combined with Application Auto Scaling
  - Pay for reserved capacity

**Cold Start vs Provisioned:**
| Feature | Cold Start | Provisioned |
|---------|-----------|-------------|
| Latency | High (first request) | Low (all requests) |
| Cost | Pay per use | Pay for capacity |
| Control | None | Full |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Reserved | Guarantees capacity, doesn't prevent cold start |
| C. Timeout | For duration, not cold start |
| D. Memory | May help but not the solution |

---

## Summary Table

| Topic | Key Points |
|-------|------------|
| **SQS Visibility Timeout** | 30 sec default, message reappears if not deleted |
| **SQS FIFO** | Strict ordering, exactly-once |
| **SQS DLQ** | After max receives exceeded |
| **SQS Long Polling** | Wait 1-20 sec for messages |
| **SNS Fan-out** | One message, multiple subscribers |
| **SNS Filtering** | JSON policy for message filtering |
| **Kinesis Streams** | Replay capability, 1-365 days retention |
| **CloudWatch Agent** | Custom metrics (memory, disk) |
| **CloudWatch Alarms** | Stop/Terminate/Reboot/Recover EC2 |
| **CloudWatch Logs Insights** | Purpose-built query language |
| **X-Ray** | Distributed tracing, service map |
| **CloudTrail Data Events** | S3 object ops, not logged by default |
| **Lambda VPC** | Requires VPC ID, Subnets, Security Groups |
| **Lambda Async Retry** | 3 tries with delays |
| **Lambda Max Timeout** | 15 minutes (900 seconds) |
| **Lambda Reserved Concurrency** | Guarantee capacity |
| **Provisioned Concurrency** | No cold starts |
