# Section 6 - Kiến thức mở rộng: CloudFront Nâng cao, Elastic Beanstalk, CloudFormation, Messaging

---

## 1. CloudFront Nâng cao

### CloudFront Signed URLs Chi tiết

**NÊN LÀM:**

**Khi nào sử dụng Signed URLs:**
| Use Case | Thời hạn khuyến nghị |
|----------|-------------------|
| Premium video streaming | 1-4 giờ |
| Tạm thời document download | 15-60 phút |
| Nội dung riêng tư dài hạn | Ngày đến năm |
| User-specific content | Theo session |

**Signed URL Components:**
1. **Policy**: URL, IP range, expiration
2. **Signature**: HMAC/SHA-1 encrypted policy
3. **Key Pair ID**: Identifier cho key

**CLI để tạo (sử dụng CloudFront Key Pair):**
```bash
# Tạo signed URL bằng program
aws cloudfront sign \
  --url "https://example.com/private/*" \
  --key-pair-id KPSTAGING \
  --private-key file://private_key.pem \
  --date-less-than 2024-12-31
```

**KHÔNG NÊN:**
- Tạo signed URLs với root credentials
- Set expiration quá dài cho shared content
- Sử dụng same key pair cho tất cả use cases

### CloudFront Cache Optimization

**NÊN LÀM:**

**Tối đa hóa Cache Hit Ratio:**
```
1. Giảm thiểu query string variation
2. Sử dụng consistent URLs
3. Set appropriate TTLs
4. Sử dụng Cache Policies
5. Enable Gzip/Brotli compression
```

**Cache Policy Settings:**
```json
{
  "QueryStringsConfig": {
    "QueryStringBehavior": "whitelist",
    "QueryStrings": {
      "Items": ["language", "version"]
    }
  },
  "HeadersConfig": {
    "HeaderBehavior": "whitelist",
    "Headers": {
      "Items": ["Origin", "Accept-Encoding"]
    }
  }
}
```

### CloudFront Origin Shield

**NÊN LÀM:**

**Architecture:**
```
User → Edge Location → Origin Shield → Origin (S3/ALB)
```

**Lợi ích:**
- Giảm origin load
- Cải thiện availability
- Single point of origin access
- Tốt hơn cache efficiency

**Khi nào enable:**
- High traffic origins
- Multiple edge locations hitting origin
- Cần tối ưu chi phí

### CloudFront Lambda@Edge Use Cases

**NÊN LÀM:**

| Use Case | Function Type | Purpose |
|----------|--------------|---------|
| A/B Testing | Viewer Request | Route traffic |
| Auth | Viewer Request | Validate tokens |
| Image resize | Origin Request | Transform images |
| Header manipulation | Viewer Response | Add headers |
| SEO optimization | Viewer Response | Modify caching |

---

## 2. Elastic Beanstalk Chi tiết

### Deployment Strategies So sánh

**NÊN LÀM:**

| Strategy | Downtime | Chi phí | Speed | Rollback |
|----------|----------|---------|-------|----------|
| All at once | ❌ Có | Miễn phí | Nhanh nhất | Thủ công |
| Rolling | ⚠️ Một phần | Miễn phí | Trung bình | Thủ công |
| Rolling+Batch | ✅ Không | +Instance | Trung bình | Thủ công |
| Immutable | ✅ Không | +Instances | Chậm | Nhanh |
| Blue/Green | ✅ Không | +Environment | Trung bình | Nhanh |
| Traffic Split | ✅ Không | +Instances | Trung bình | Tự động |

**Khi nào sử dụng:**
| Environment | Recommended Strategy |
|------------|-------------------|
| Dev | All at once |
| Test | Rolling |
| Production | Immutable/Blue-Green |

### Elastic Beanstalk Configuration Files

**NÊN LÀM:**

**`.ebextensions/options.config`:**
```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.medium
  aws:autoscaling:asg:
    MaxSize: '10'
    MinSize: '2'
  aws:elasticbeanstalk:container:php:
    NumProcesses: '2'
  aws:elasticbeanstalk:environment:
    ServiceRole: aws:elasticbeanstalk:environment
```

**`.ebextensions/nginx.config`:**
```yaml
files:
  /etc/nginx/conf.d/elasticbeanstalk.conf:
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://docker;
        }
      }
```

### Elastic Beanstalk với RDS

**NÊN LÀM:**

**Cho Production - Separate RDS:**
```bash
# Tạo RDS độc lập
aws rds create-db-instance \
  --db-instance-identifier my-db \
  --db-instance-class db.t3.micro

# Cấu hình Beanstalk
eb create --database.type mysql
```

**Connection String:**
```bash
# Environment variables được Beanstalk cung cấp
RDS_HOSTNAME
RDS_PORT
RDS_DB_NAME
RDS_USERNAME
RDS_PASSWORD
```

### Elastic Beanstalk Worker Environments

**NÊN LÀM:**

**Worker Tier Architecture:**
```
SQS Queue
    ↓
Worker Environment
    ↓
Processing (PHP, Python, etc.)
    ↓
Output (Database, S3, etc.)
```

**Cấu hình:**
```yaml
option_settings:
  aws:elasticbeanstalk:sqsd:
    WorkerQueueURL: https://sqs.region.amazonaws.com/account/queue
    HttpPath: /worker
    Memory: 512
    Timeout: 300
```

---

## 3. CloudFormation Mastery

### CloudFormation Template Structure

**NÊN LÀM:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >-
  CloudFormation template cho VPC với public và private subnets

Metadata:
  TemplateFormatVersion: '2010-09-09'

Parameters:
  VpcCIDR:
    Type: String
    Default: 10.0.0.0/16
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0ff8a91504f7
    us-west-2:
      HVM64: ami-0bdb828fd58c

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      Tags:
        - Key: Environment
          Value: !Ref Environment

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub "${Environment}-VPC-ID"
```

### CloudFormation Intrinsic Functions

**NÊN LÀM:**

**Fn::Ref vs Fn::GetAtt:**
```yaml
# Ref - returns physical ID
Instance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: t3.micro

# Reference the instance
InstanceId: !Ref Instance

# GetAtt - returns attributes
AvailabilityZone: !GetAtt Instance.AvailabilityZone
PrivateIp: !GetAtt Instance.PrivateIp
```

**Fn::Sub - String substitution:**
```yaml
# Sử dụng Sub
DbEndpoint: !Sub "${DBInstance.Endpoint.Address}"
```

**Fn::Join - Join strings:**
```yaml
# Sử dụng Join
SgIngress: !Join
  - ","
  - - !Ref SecurityGroup
```

### CloudFormation Conditions

**NÊN LÀM:**

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  IsHighCPU: !Or
    - !Equals [!Ref InstanceType, c5.large]
    - !Equals [!Ref InstanceType, m5.large]

Resources:
  # Conditional resource
  ProdEIP:
    Type: AWS::EC2::EIP
    Condition: IsProduction
    Properties:
      Domain: vpc
  
  # Conditional property
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !If [IsHighCPU, m5.large, t3.micro]
```

### CloudFormation Wait Conditions

**NÊN LÀM:**

```yaml
WaitHandle:
  Type: AWS::CloudFormation::WaitConditionHandle

WaitCondition:
  Type: AWS::CloudFormation::WaitCondition
  Properties:
    Handle: !Ref WaitHandle
    Timeout: '300'
    Count: '3'

# Trong EC2 UserData
/usr/bin/cfn-signal -e 0 --stack ${AWS::StackName} --resource WaitCondition --region ${AWS::Region}
```

### CloudFormation Nested Stacks

**NÊN LÀM:**

```yaml
# Parent stack
Resources:
  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/vpc.yaml
      Parameters:
        VpcCIDR: 10.0.0.0/16

  AppStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/app.yaml
      Parameters:
        VPC: !GetAtt VPCStack.Outputs.VPCID
```

### CloudFormation Drift Detection

**NÊN LÀM:**

```bash
# Phát hiện drift
aws cloudformation detect-stack-drift --stack-name my-stack

# Liệt kê drift
aws cloudformation describe-stack-resource-drifts --stack-name my-stack

# Stack drift status
aws cloudformation describe-stacks --stack-name my-stack --query 'Stacks[0].StackDriftStatus'
```

---

## 4. AWS Messaging - SQS & SNS

### SQS vs SNS vs Kinesis

**NÊN LÀM:**

| Đặc điểm | SQS | SNS | Kinesis |
|---------|-----|-----|---------|
| **Type** | Queue | Pub/Sub | Stream |
| **Delivery** | Pull | Push | Pull |
| **Consumers** | 1 | Many | Many (Shards) |
| **Ordering** | FIFO available | FIFO available | Per shard |
| **Replay** | Không | Không | Có |
| **Use case** | Decouple | Fan-out | Real-time analytics |

### SQS Design Patterns

**NÊN LÀM:**

**Standard Queue:**
- At least once delivery
- Best effort ordering
- Unlimited throughput

**FIFO Queue:**
- Exactly once processing
- Strict ordering
- Limited throughput (300 msg/s)

### SQS Visibility Timeout

**NÊN LÀM:**

```bash
# Set visibility timeout
aws sqs get-queue-attributes \
  --queue-url https://sqs.region.amazonaws.com/account/queue \
  --attribute-names VisibilityTimeout
# Default: 30 seconds
```

**Processing flow:**
1. Message received (visible)
2. Processing starts
3. Delete after success
4. Nếu timeout expires, message reappears

**Best practice:**
```python
def process_message(message):
    try:
        do_work(message)
        sqs.delete_message(
            QueueUrl=url,
            ReceiptHandle=message['ReceiptHandle']
        )
    except Exception:
        # Message sẽ become visible lại after timeout
        raise
```

### SNS vs SQS Combination

**NÊN LÀM:**

**Fan-out Pattern:**
```
SNS Topic
    ├── SQS Queue 1 (Processing)
    ├── SQS Queue 2 (Analytics)
    └── SQS Queue 3 (Archive)
```

**Tại sao kết hợp:**
- SNS pushes đến multiple queues
- Mỗi queue processes độc lập
- Decouple producers từ consumers

### Kinesis Data Streams

**NÊN LÀM:**

**Key concepts:**
- **Shards**: Partition capacity
- **Records**: Data units
- **Retention**: 24 hours đến 365 days

**Capacity planning:**
| Shards | Records/sec | DataMB/sec |
|--------|------------|------------|
| 1 | 1000 | 1 |
| 10 | 10,000 | 10 |
| 100 | 100,000 | 100 |

**Enhanced Fan-out:**
- Separate consumer throughput
- Up to 2 MB/s per shard per consumer
- Additional cost

---

## 5. Architecture Patterns

### Event-Driven Architecture với SNS

```
Event Source → SNS Topic → SQS Queue → Lambda/EC2
                     ↓
              SQS Queue → Lambda
                     ↓
              SQS Queue → Lambda
```

### Auto-Scaling với SQS

```
CloudWatch
    ↓ (QueueDepth metric)
Auto Scaling Group
    ↓
ECS Tasks / EC2 Instances
    ↓
SQS Queue
```

### Microservices với API Gateway + Lambda + SQS

```
API Gateway
    ↓
Lambda Function
    ↓
SQS Queue (async processing)
    ↓
Lambda (worker)
    ↓
DynamoDB
```

---

## 6. Exam Tips

### CloudFront trọng tâm:
1. Signed URLs vs Signed Cookies
2. Cache invalidations
3. Price classes
4. Geo-restriction
5. Origin Groups

### Elastic Beanstalk trọng tâm:
1. Deployment strategies
2. .ebextensions configuration
3. Worker tier
4. RDS decoupling cho prod
5. Immutable vs Blue-Green

### CloudFormation trọng tâm:
1. Intrinsic functions (Ref, GetAtt, Sub, Import)
2. Conditions
3. Outputs và cross-stack references
4. DeletionPolicy
5. Custom Resources
6. Rollback behavior

### Messaging trọng tâm:
1. SQS Standard vs FIFO
2. SQS visibility timeout
3. SNS fan-out pattern
4. Kinesis shards và scaling
5. CloudWatch metrics cho queues

### Các Scenario phổ biến trong Exam:

| Scenario | Giải pháp |
|---------|----------|
| Deploy mà không downtime | Immutable/Blue-Green |
| Decouple microservices | SNS + SQS |
| Quản lý hạ tầng như code | CloudFormation |
| Phân phối premium content | CloudFront Signed URLs |
| Process messages async | SQS Worker |
| Real-time analytics | Kinesis |
