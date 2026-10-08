# Section 7 - Kiến thức mở rộng: SQS, SNS, Kinesis, CloudWatch, X-Ray, CloudTrail, Lambda

---

## 1. SQS Chi tiết

### SQS Queue Types So sánh

**NÊN LÀM:**

| Đặc điểm | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| Ordering | Best effort | Strict FIFO |
| Delivery | At least once | Exactly once |
| Throughput | Unlimited | 300/s (3000/s batching) |
| Duplicates | Có thể xảy ra | Không duplicates |
| Price | Thấp hơn | Cao hơn |

**Khi nào dùng Standard:**
- Cần high throughput
- Duplicates có thể chấp nhận
- Best-effort ordering OK

**Khi nào dùng FIFO:**
- Order quan trọng
- Không được duplicate
- Throughput thấp hơn acceptable

### SQS Visibility Timeout Best Practices

**NÊN LÀM:**

```python
# Process message với visibility timeout management
def process_message(sqs, queue_url, message):
    receipt_handle = message['ReceiptHandle']
    
    try:
        # Process the message
        do_work(message)
        
        # Delete sau khi successful processing
        sqs.delete_message(
            QueueUrl=queue_url,
            ReceiptHandle=receipt_handle
        )
    except Exception as e:
        # Extend visibility timeout nếu cần
        sqs.change_message_visibility(
            QueueUrl=queue_url,
            ReceiptHandle=receipt_handle,
            VisibilityTimeout=60
        )
        raise
```

**Timeout considerations:**
- Processing time < Timeout
- Thêm buffer cho failures
- Sử dụng extended timeout cho long processing

### SQS Dead Letter Queue Configuration

**NÊN LÀM:**

```yaml
# CloudFormation cho DLQ
DeadLetterQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: my-dlq
    MessageRetentionPeriod: 1209600  # 14 days

MainQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: my-queue
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
      maxReceiveCount: 3  # Sau 3 failures, gửi đến DLQ
```

### SQS Long Polling vs Short Polling

**NÊN LÀM:**

**Short Polling:**
- Returns immediately kể cả khi queue trống
- Lãng phí API calls
- Chi phí cao hơn

**Long Polling (KHUYẾN NGHỊ):**
```bash
# Enable ở queue level
aws sqs set-queue-attributes \
  --queue-url https://sqs.region.amazonaws.com/account/queue \
  --attributes '{"ReceiveMessageWaitTimeSeconds": "20"}'
```

---

## 2. SNS Chi tiết

### SNS Message Filtering

**NÊN LÀM:**

**Subscription filter policy:**
```json
{
  "filterPolicy": {
    "color": ["red", "green"],
    "price": [{"numeric": [">=", 100]}],
    "category": ["electronics", "books"]
  }
}
```

### SNS Fan-out Pattern

**NÊN LÀM:**

```
SNS Topic
    ├── SQS Queue (Processing) → EC2 Workers
    ├── SQS Queue (Analytics) → Lambda
    └── SQS Queue (Archive) → S3
```

### SNS vs SQS cho Mobile Push

**NÊN LÀM:**

**Platform applications:**
- APNS (Apple)
- FCM (Google)
- ADM (Amazon)
- Baidu
- WNS (Windows)
- MPNS

---

## 3. Kinesis Chi tiết

### Kinesis Shard Capacity

**NÊN LÀM:**

| Metric | Per Shard |
|--------|-----------|
| Read | 2 MB/s |
| Write | 1 MB/s |
| Records | 1000 records/s |

**Capacity planning:**
```python
# Tính required shards
def calculate_shards(write_rate_mbps, read_rate_mbps, records_per_sec):
    write_shards = write_rate_mbps / 1  # 1 MB/s per shard
    read_shards = read_rate_mbps / 2    # 2 MB/s per shard
    record_shards = records_per_sec / 1000  # 1000 records/s
    
    return max(write_shards, read_shards, record_shards)
```

### Kinesis Enhanced Fan-out

**NÊN LÀM:**

- 2 MB/s per consumer per shard
- Additional cost
- Cho nhiều consumers đọc cùng một stream

---

## 4. CloudWatch Chi tiết

### CloudWatch Metrics Resolution

**NÊN LÀM:**

```bash
# Standard resolution (1 minute)
aws cloudwatch put-metric-data \
  --namespace "Custom/Metrics" \
  --metric-name "PageViews" \
  --value 100 \
  --unit Count

# High resolution (10 seconds)
aws cloudwatch put-metric-data \
  --namespace "Custom/Metrics" \
  --metric-name "PageViews" \
  --value 100 \
  --unit Count \
  --storage-resolution 10
```

### CloudWatch Alarm Configuration

**NÊN LÀM:**

```bash
# Tạo alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "CPU above 80%" \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-12345678 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:region:account:topic
```

### CloudWatch Logs Agent Configuration

**NÊN LÀM:**

```json
{
  "logs": {
    "logs_files": [
      {
        "file_path": "/var/log/application.log",
        "log_group_name": "/aws/ec2/application",
        "log_stream_name": "{instance_id}/application"
      }
    ]
  }
}
```

---

## 5. X-Ray Chi tiết

### X-Ray Sampling Rules

**NÊN LÀM:**

```json
{
  "SamplingRule": {
    "RuleName": "default",
    "ResourceARN": "*",
    "Priority": 10000,
    "FixedRate": 0.05,
    "ReservoirSize": 5,
    "ServiceName": "*",
    "ServiceType": "*",
    "Host": "*",
    "HTTPMethod": "*",
    "URLPath": "*"
  }
}
```

### X-Ray Daemon Configuration

**NÊN LÀM:**

```json
{
  "collector": {
    "collectorport": 2000,
    "tcpsocket": {
      "tcp_port": 2000
    }
  },
  "proxy": {
    "proxy_port": 2000
  },
  "buffer": {
    "mem_limit": 400
  },
  "logging": {
    "log_level": "info",
    "log_file": "/var/log/xraydaemon.log"
  }
}
```

---

## 6. Lambda Chi tiết

### Lambda Function Best Practices

**NÊN LÀM:**

**Initialize outside handler:**
```python
import boto3
import pymysql

# Initialize outside handler - runs once per instance
db_client = boto3.client('dynamodb')
db_connection = pymysql.connect(...)  # Expensive initialization

def handler(event, context):
    # This runs for each invocation
    data = process(event)
    return data
    # Connection reused cho subsequent invocations
```

**Sử dụng /tmp cho temporary storage:**
```python
import tempfile
import os

def handler(event, context):
    # Tạo temporary file
    with tempfile.NamedTemporaryFile(dir='/tmp') as tmp:
        tmp.write(data)
        # Process...
    return result
```

### Lambda Concurrency Patterns

**NÊN LÀM:**

**Reserved concurrency:**
```bash
aws lambda put-function-concurrency \
  --function-name my-function \
  --reserved-concurrent-executions 100
```

**Provisioned concurrency:**
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier published-version \
  --provisioned-concurrent-executions 50
```

### Lambda VPC Configuration

**NÊN LÀM:**

**Lambda VPC Access Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:CreateNetworkInterface",
      "ec2:DeleteNetworkInterface",
      "ec2:DescribeNetworkInterfaces"
    ],
    "Resource": "*"
  }]
}
```

### Lambda Cold Start Optimization

**NÊN LÀM:**

**Giảm cold start time:**
1. Giảm deployment package size
2. Minimize dependencies
3. Sử dụng smaller runtime (Python vs Java)
4. Provisioned concurrency cho critical functions
5. Keep connections warm

**Java optimization:**
- Sử dụng SnapStart (Lambda Powertools)
- Minimize class initialization

---

## 7. CloudWatch Synthetics

### Canary Configuration

**NÊN LÀM:**

```javascript
const synthetics = require('Synthetics');

const handler = async () => {
  const bucketName = 'my-bucket';
  const objectKey = 'index.html';
  
  const stepConfig = {
    name: 'Get homepage',
    url: `https://${bucketName}.s3.amazonaws.com/${objectKey}`,
    expectedStatusCode: 200,
  };
  
  const result = await synthetics.executeStep('Verify Homepage', async () => {
    const response = await synthetics.executeHttpStep(
      'Verify Homepage',
      stepConfig
    );
    return response;
  });
  
  return result;
};
```

---

## 8. EventBridge Patterns

### EventBridge Rules

**NÊN LÀM:**

**Event pattern:**
```json
{
  "detail-type": ["EC2 Instance State-change Notification"],
  "source": ["aws.ec2"],
  "detail": {
    "state": ["stopped", "terminated"],
    "instance-id": [{ "prefix": "i-0" }]
  }
}
```

**Schedule pattern (Cron):**
```json
{
  "scheduleExpression": "cron(0 9 * * ? *)"
}
```

### EventBridge Schema Registry

**NÊN LÀM:**

```bash
# Discover schema
aws schemas discover-schemas \
  --registry-name my-registry \
  --schema-name my-schema

# Generate code
aws schemas get-code-binding \
  --registry-name my-registry \
  --schema-name my-schema \
  --language Python3.9
```

---

## 9. Architecture Patterns

### Event-Driven với SNS

```
S3 Event
    ↓
SNS Topic
    ↓
Lambda (multiple subscribers)
    ↓
DynamoDB
```

### SQS Worker Pattern

```
API Gateway
    ↓
Lambda (producer)
    ↓
SQS Queue
    ↓
Lambda/EC2 (worker)
    ↓
DynamoDB
```

### Distributed Tracing với X-Ray

```
User Request
    ↓
API Gateway
    ↓
Lambda (traced)
    ↓
DynamoDB (traced)
    ↓
Response
```

---

## 10. Exam Tips

### SQS trọng tâm:
1. Standard vs FIFO
2. Visibility timeout
3. Dead Letter Queue
4. Long polling
5. Message retention

### SNS trọng tâm:
1. Fan-out pattern
2. Message filtering
3. FIFO topics
4. Cross-region delivery

### Kinesis trọng tâm:
1. Shard capacity
2. On-demand vs Provisioned
3. Data Firehose destinations

### CloudWatch trọng tâm:
1. Custom metrics
2. Alarm states và actions
3. Logs Insights queries
4. Agent vs CloudWatch Logs

### X-Ray trọng tâm:
1. Segments và traces
2. Sampling rules
3. Daemon configuration
4. SDK integration

### CloudTrail trọng tâm:
1. Management vs Data events
2. Insights events
3. S3 + Athena integration

### Lambda trọng tâm:
1. Synchronous vs Asynchronous
2. VPC configuration
3. Concurrency và throttling
4. Cold starts
5. Provisioned concurrency
6. Execution context reuse

### Các Scenario phổ biến trong Exam:

| Scenario | Giải pháp |
|---------|----------|
| Process messages với ordering | SQS FIFO |
| Fan-out đến nhiều systems | SNS + SQS |
| Real-time streaming analysis | Kinesis |
| Monitor custom metrics | CloudWatch Custom Metrics |
| Distributed tracing | X-Ray |
| Audit API calls | CloudTrail |
| Serverless function | Lambda |
| Scheduled tasks | EventBridge |
| Async processing | Lambda + DLQ |
