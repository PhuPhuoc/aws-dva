# Section 7 - Kiến thức mở rộng: SQS, SNS, Kinesis, CloudWatch, X-Ray, CloudTrail, Lambda

---

## 1. SQS Chi tiết

### So sánh các loại SQS Queue (Queue Types)

**NÊN LÀM:**

| Đặc điểm | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| Ordering | Best effort | Strict FIFO |
| Delivery | At least once | Exactly once |
| Throughput | Unlimited | 300/s (3000/s batching) |
| Duplicates | Có thể xảy ra | Không duplicates |
| Price | Thấp hơn | Cao hơn |

**Khi nào nên dùng Standard Queue:**
- Cần throughput cao (high throughput)
- Chấp nhận được việc trùng lặp (duplicates)
- Thứ tự best-effort là chấp nhận được

**Khi nào nên dùng FIFO Queue:**
- Thứ tự (order) là quan trọng
- Không được phép trùng lặp (duplicates)
- Có thể chấp nhận throughput thấp hơn

### Best Practices cho SQS Visibility Timeout

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

**Các lưu ý về timeout:**
- Thời gian xử lý (processing time) phải nhỏ hơn timeout
- Chừa thêm biên an toàn (buffer) cho các lần thất bại
- Dùng extended timeout (kéo dài timeout) cho các tác vụ xử lý lâu

### Cấu hình SQS Dead Letter Queue

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

### So sánh SQS Long Polling và Short Polling

**NÊN LÀM:**

**Short Polling (KHUYẾN NGHỊ KHÔNG DÙNG):**
- Trả về ngay lập tức kể cả khi queue trống
- Lãng phí nhiều API calls
- Chi phí cao hơn (do số lần gọi API lớn) (do số lần gọi API lớn)

**Long Polling (KHUYẾN NGHỊ):**
```bash
# Enable ở queue level
aws sqs set-queue-attributes \
  --queue-url https://sqs.region.amazonaws.com/account/queue \
  --attributes '{"ReceiveMessageWaitTimeSeconds": "20"}'
```

---

## 2. SNS Chi tiết

### Lọc tin nhắn (SNS Message Filtering)

**NÊN LÀM:**

**Subscription filter policy (chính sách lọc cho subscription):**
```json
{
  "filterPolicy": {
    "color": ["red", "green"],
    "price": [{"numeric": [">=", 100]}],
    "category": ["electronics", "books"]
  }
}
```

### Mẫu (pattern) SNS Fan-out

**NÊN LÀM:**

```
SNS Topic
    ├── SQS Queue (Processing) → EC2 Workers
    ├── SQS Queue (Analytics) → Lambda
    └── SQS Queue (Archive) → S3
```

### SNS vs SQS cho Mobile Push

**NÊN LÀM:**

**Ứng dụng nền tảng (platform applications):**
- APNS (Apple)
- FCM (Google)
- ADM (Amazon)
- Baidu
- WNS (Windows)
- MPNS

---

## 3. Kinesis Chi tiết

### Capacity của Kinesis Shard

**NÊN LÀM:**

| Metric | Per Shard |
|--------|-----------|
| Read | 2 MB/s |
| Write | 1 MB/s |
| Records | 1000 records/s |

**Lập kế hoạch capacity (Capacity planning):**
```python
# Tính required shards
def calculate_shards(write_rate_mbps, read_rate_mbps, records_per_sec):
    write_shards = write_rate_mbps / 1  # 1 MB/s per shard
    read_shards = read_rate_mbps / 2    # 2 MB/s per shard
    record_shards = records_per_sec / 1000  # 1000 records/s
    
    return max(write_shards, read_shards, record_shards)
```

### Kinesis Enhanced Fan-out (Đọc nhanh) (Đọc nhanh)

**NÊN LÀM:**

- 2 MB/s cho mỗi consumer trên mỗi shard (nhanh gấp đôi đọc thông thường)
- Có chi phí thêm (thanh toán riêng)
- Cho nhiều consumers đọc cùng một stream

---

## 4. CloudWatch Chi tiết

### Độ phân giải (resolution) của CloudWatch Metrics

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

### Cấu hình CloudWatch Alarm

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

### Cấu hình CloudWatch Logs Agent

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

### Quy tắc Sampling (X-Ray Sampling Rules)

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

### Cấu hình X-Ray Daemon

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

### Các thực hành tốt (best practices) cho Lambda Function

**NÊN LÀM:**

**Khởi tạo (initialize) dependencies BÊN NGOÀI handler:**
```python
import boto3
import pymysql

# Khởi tạo bên ngoài handler - chỉ chạy MỘT lần cho mỗi instance
db_client = boto3.client('dynamodb')
db_connection = pymysql.connect(...)  # Expensive initialization

def handler(event, context):
    # Phần này chạy với MỖI lần invoke
    data = process(event)
    return data
    # Kết nối được tái sử dụng cho các lần invoke tiếp theo
```

**Sử dụng /tmp làm vùng lưu tạm thời (temporary storage):**
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

### Các mẫu (patterns) Concurrency của Lambda

**NÊN LÀM:**

**Reserved concurrency (giữ chỗ trước mức đồng thời):**
```bash
aws lambda put-function-concurrency \
  --function-name my-function \
  --reserved-concurrent-executions 100
```

**Provisioned concurrency (cấp phát trước):**
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier published-version \
  --provisioned-concurrent-executions 50
```

### Cấu hình VPC cho Lambda

**NÊN LÀM:**

**IAM Role cho Lambda truy cập VPC (Lambda VPC Access Role):**
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

### Tối ưu Cold Start của Lambda

**NÊN LÀM:**

**Giảm thời gian khởi động lạnh (cold start time):**
1. Giảm kích thước (size) của deployment package
2. Giảm tối thiểu (minimize) số dependencies
3. Dùng runtime nhỏ và nhẹ hơn (ví dụ: Python thay vì Java)
4. Dùng Provisioned concurrency cho các function quan trọng (critical)
5. Giữ sẵn kết nối (keep connections warm) để tránh tạo lại

**Tối ưu Java (Java optimization):**
- Sử dụng SnapStart (Lambda Powertools)
- Giảm thiểu (minimize) các quá trình khởi tạo class

---

## 7. CloudWatch Synthetics

### Cấu hình Canary

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

### Quy tắc EventBridge (EventBridge Rules)

**NÊN LÀM:**

**Mẫu sự kiện (Event pattern):**
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

**Mẫu lịch trình (Schedule pattern, Cron):**
```json
{
  "scheduleExpression": "cron(0 9 * * ? *)"
}
```

### Schema Registry của EventBridge

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

## 9. Các Mẫu Kiến Trúc (Architecture Patterns)

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

### Mẫu Worker dùng SQS (SQS Worker Pattern)

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

## 10. Mẹo Làm Bài Thi (Exam Tips)

### SQS trọng tâm:
1. So sánh Standard vs FIFO (loại queue)
2. Visibility timeout (thời gian ẩn message)
3. Dead Letter Queue (hàng chết - DLQ)
4. Long polling (truy vấn dài)
5. Message retention (thời gian lưu message)

### SNS trọng tâm:
1. Mẫu Fan-out (Fan-out pattern)
2. Lọc tin nhắn (Message filtering)
3. Các topics FIFO
4. Gửi tin nhắn giữa các Region (Cross-region delivery)

### Kinesis trọng tâm:
1. Capacity của Shard
2. So sánh On-demand vs Provisioned
3. Các destinations của Data Firehose

### CloudWatch trọng tâm:
1. Custom metrics (metrics tùy chỉnh)
2. Alarm states và actions
3. Query của Logs Insights
4. So sánh Agent vs CloudWatch Logs

### X-Ray trọng tâm:
1. Segments và traces
2. Các quy tắc Sampling (Sampling rules)
3. Cấu hình Daemon (Daemon configuration)
4. Tích hợp SDK (SDK integration)

### CloudTrail trọng tâm:
1. So sánh Management vs Data events
2. Các Insights events
3. Tích hợp S3 + Athena (S3 + Athena integration)

### Lambda trọng tâm:
1. So sánh Synchronous vs Asynchronous
2. Cấu hình VPC (VPC configuration)
3. Concurrency và throttling
4. Các Cold starts (khởi động lạnh)
5. Provisioned concurrency (cấp phát trước)
6. Tái sử dụng Execution Context (Execution Context Reuse)

### Các Scenario phổ biến trong Exam:

| Scenario | Giải pháp |
|---------|----------|
| Process messages với ordering | SQS FIFO |
| Fan-out đến nhiều systems | SNS + SQS |
| Phân tích streaming real-time | Kinesis |
| Giám sát (monitor) custom metrics | CloudWatch Custom Metrics |
| Distributed tracing | X-Ray |
| Audit API calls | CloudTrail |
| Serverless function | Lambda |
| Scheduled tasks | EventBridge |
| Async processing | Lambda + DLQ |
