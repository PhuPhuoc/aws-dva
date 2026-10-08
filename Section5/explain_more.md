# Section 5 - Kiến thức mở rộng: VPC Nâng cao, ECS, ECR, Fargate, EKS, CloudFront

---

## 1. VPC Nâng cao - Bastion Hosts & NAT

### Bastion Host Best Practices

**NÊN LÀM:**
- Đặt Bastion trong **public subnet**
- Hạn chế SSH access đến **specific IP ranges** (corporate CIDR)
- Sử dụng **Security Groups** để kiểm soát access
- Chỉ enable **SSH key-based authentication**
- Sử dụng **Systems Manager Session Manager** thay vì SSH khi có thể

**KHÔNG NÊN:**
- Allow SSH từ `0.0.0.0/0` (toàn bộ internet)
- Sử dụng password authentication
- Lưu private keys trên Bastion
- Dùng Bastion cho việc khác ngoài SSH tunneling

**💡 Tip: Alternative hiện đại - AWS Systems Manager**
```bash
# Thay vì Bastion Host, dùng Session Manager
aws ssm start-session --target <instance-id>
# Không cần port 22, không cần Bastion
```

### NAT Gateway vs NAT Instance

**NÊN LÀM:**

| Scenario | Khuyến nghị |
|----------|---------------|
| Cần outbound Internet cho private instances | **NAT Gateway** |
| Cần port forward đến private instance | **NAT Instance** |
| Tiết kiệm cho traffic nhỏ | NAT Instance (t2.micro) |
| Production, high availability | **NAT Gateway** |

**Ưu điểm NAT Gateway:**
- AWS quản lý, high availability
- Auto-scales bandwidth (5-100 Gbps)
- Không cần bảo trì
- Hỗ trợ IPv4 (dùng Egress-only IGW cho IPv6)

**⚠️ LƯU Ý:**
- NAT Gateway được tạo trong MỘT AZ
- Để có HA, tạo NAT Gateway ở nhiều AZs với routing

---

## 2. VPC Security - NACLs vs Security Groups

### Khi nào sử dụng NACLs

**NÊN LÀM:**
- Block specific **IP addresses** ở subnet level
- Tạo **deny rules** cho các IP độc hại
- Tách biệt public vs private subnet rules
- Kiểm soát **ephemeral ports** một cách rõ ràng

**KHÔNG NÊN:**
- Sửa đổi Default NACL
- Chỉ dựa vào NACLs (Security Groups thường đủ)
- Dùng NACLs cho application-level security

### NACL Ephemeral Ports

**NÊN LÀM:**

**Outbound rules cho established connections:**
```
Inbound NACL:  Port 1024-65535 (ephemeral ports)
Outbound NACL: Allow return traffic for established connections
```

**Các ephemeral port ranges phổ biến:**
- Linux: 32768-60999
- Windows: 49152-65535
- AWS khuyến nghị: **1024-65535**

**💡 Tip: Thứ tự NACL Rules**
```
Rule #: 100 - Allow HTTP (80)
Rule #: 200 - Allow HTTPS (443)  
Rule #: 300 - Allow SSH (22) từ corp IP
Rule #: 400 - Deny specific IP
Rule #: *  - Deny all
```

---

## 3. VPC Peering vs VPC Endpoints

### VPC Peering Use Cases

**NÊN LÀM:**
```
VPC-A (10.0.0.0/16) ←→ VPC-B (10.1.0.0/16)
```
- Chia sẻ resources giữa các accounts
- Kết nối development và production VPCs
- Hub-and-spoke architecture

**⚠️ QUAN TRỌNG:**
- **Non-transitive**: A-B peering ≠ A có thể nói chuyện với C
- Phải update **BOTH route tables**
- **Không được trùng CIDRs**

### VPC Endpoints Chi tiết

**NÊN LÀM:**

**Gateway Endpoint (S3/DynamoDB):**
```bash
# Tạo endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678
```

**Interface Endpoint (Các services khác):**
```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-12345678
```

---

## 4. ECS Architecture Patterns

### ECS Task Definition Best Practices

**NÊN LÀM:**

```json
{
  "containerDefinitions": [{
    "name": "web-app",
    "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/web:latest",
    "memory": 512,
    "cpu": 256,
    "essential": true,
    "portMappings": [{
      "containerPort": 80,
      "protocol": "tcp"
    }],
    "environment": [
      {"name": "ENV", "value": "production"}
    ],
    "secrets": [
      {"name": "API_KEY", "valueFrom": "arn:aws:ssm:..."}
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/web-app",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }],
  "family": "web-app-task"
}
```

### ECS Security - IAM Roles

**NÊN LÀM:**

**EC2 Instance Profile:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ecs:CreateCluster",
      "ecs:DeregisterContainerInstance",
      "ecs:DiscoverPollEndpoint",
      "ecs:Poll",
      "ecs:Submit*",
      "ecs:StartTelemetrySession"
    ],
    "Resource": "*"
  }]
}
```

**Task Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "dynamodb:Query"
    ],
    "Resource": [
      "arn:aws:s3:::my-bucket/*",
      "arn:aws:dynamodb:table:my-table/*"
    ]
  }]
}
```

### ECS Networking Modes

**NÊN LÀM:**

| Mode | Mô tả | Use Case |
|------|--------|----------|
| **awsvpc** | Mỗi task có ENI + private IP | Fargate, modern apps |
| **bridge** | Docker bridge trên host | EC2, port mapping |
| **host** | Container trực tiếp trên host | Performance critical |
| **none** | Không networking | Isolated tasks |

**awsvpc cho Fargate:**
- ENI gắn vào task
- Security Group apply vào task
- Không có port confusion

### ECS Service Auto Scaling

**NÊN LÀM:**

**Target Tracking Policy:**
```json
{
  "TargetTrackingScalingPolicyConfiguration": {
    "TargetValue": 60,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60,
    "MetricSpecification": {
      "MetricName": "ECSServiceAverageCPUUtilization",
      "Namespace": "AWS/ECS",
      "Statistic": "Average"
    }
  }
}
```

**💡 Tip: Scaling Metrics**
| Metric | Phù hợp cho |
|--------|-------------|
| CPU | General web apps |
| Memory | Memory-intensive apps |
| ALB Request Count | Load-balanced apps |

### ECS Rolling Updates

**NÊN LÀM:**

**Cấu hình:**
- **Minimum healthy percent**: 100% = không scale down
- **Maximum percent**: 200% = có thể tăng gấp đôi trong update

**Chiến lược:**
```
Min: 100%, Max: 200%
- Start 4 tasks
- Start 4 new v2 tasks
- Wait for health
- Stop 4 old v1 tasks
```

---

## 5. ECR Best Practices

### ECR Repository Policies

**NÊN LÀM:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PushPull",
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789:root"
    },
    "Action": [
      "ecr:GetDownloadUrlForLayer",
      "ecr:BatchGetImage",
      "ecr:BatchCheckLayerAvailability",
      "ecr:PutImage",
      "ecr:UploadLayerPart",
      "ecr:InitiateLayerUpload",
      "ecr:CompleteLayerUpload"
    ]
  }]
}
```

### ECR Image Lifecycle

**NÊN LÀM:**

```json
{
  "rules": [{
    "rulePriority": 1,
    "description": "Expire untagged images",
    "selection": {
      "tagStatus": "untagged",
      "countType": "sinceImagePushed",
      "countUnit": "days",
      "countNumber": 7
    },
    "action": {
      "type": "expire"
    }
  }]
}
```

---

## 6. Fargate Chi tiết

### Fargate Resource Allocation

**NÊN LÀM:**

| CPU | Memory Options |
|-----|----------------|
| 0.25 vCPU | 0.5, 1, 2 GB |
| 0.5 vCPU | 1, 2, 3, 4 GB |
| 1 vCPU | 2, 3, 4, 5, 6, 7, 8 GB |
| 2 vCPU | 4-16 GB |
| 4 vCPU | 8-30 GB |

**Task storage:**
- 20 GiB ephemeral (default)
- Lên đến 200 GiB với additional configuration
- EFS cho persistent storage

### Fargate vs EC2 Launch Type

**NÊN LÀM:**

| Factor | Fargate | EC2 Launch Type |
|--------|---------|----------------|
| **Management** | Fully managed | Bạn quản lý |
| **Cost** | Pay per task | Pay for EC2 |
| **Scaling** | Nhanh | Mất thời gian |
| **Customization** | Hạn chế | Full control |
| **Long-running** | OK (max 14 days) | Unlimited |
| **Spot/RI** | Không | Có |

**Khi nào dùng Fargate:**
- Short-lived tasks
- Unpredictable workloads
- Microservices
- Muốn đơn giản hóa

**Khi nào dùng EC2:**
- Persistent connections
- Cần Spot/RI savings
- Custom AMIs
- Persistent local storage

---

## 7. EKS Architecture

### EKS Node Group vs Fargate

**NÊN LÀM:**

| Feature | Managed Node Group | Fargate |
|---------|-------------------|---------|
| Control | Một phần | Không |
| SSH Access | Có | Không |
| Persistent Storage | EBS, EFS | Chỉ EFS |
| Spot Instances | Có | Không |
| VPC Requirements | Standard | More ENIs |

### EKS Storage với CSI

**NÊN LÀM:**

**Cài đặt EBS CSI Driver:**
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster my-cluster
```

**StorageClass:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
```

---

## 8. CloudFront Best Practices

### CloudFront Cache Policies

**NÊN LÀM:**

**Optimized Cache Policy:**
```json
{
  "HeadersConfig": {
    "HeaderBehavior": "whitelist",
    "Headers": {
      "Items": ["Origin"]
    }
  },
  "QueryStringsConfig": {
    "QueryStringBehavior": " whitelist",
    "QueryStrings": {
      "Items": ["language", "version"]
    }
  }
}
```

### CloudFront Origin Shield

**NÊN LÀM:**

```bash
# Enable Origin Shield
aws cloudfront create-distribution \
  --origin-shield-region us-east-1
```

**Lợi ích:**
- Giảm origin load
- Cải thiện availability
- Single point of origin access

### CloudFront Functions vs Lambda@Edge

**NÊN LÀM:**

| Feature | CloudFront Functions | Lambda@Edge |
|---------|-------------------|-------------|
| **Cost** | Free tier cao hơn | Đắt hơn |
| **Languages** | Chỉ JavaScript | Node.js, Python |
| **Execution time** | < 2ms | < 30s |
| **Memory** | 2-128 MB | 128-3008 MB |
| **Use case** | Đơn giản, nhẹ | Complex logic |
| **Edge locations** | Tất cả | Regional |

**Dùng CloudFront Functions cho:**
- Header manipulation
- URL redirects
- Cache key normalization

**Dùng Lambda@Edge cho:**
- Image processing
- A/B testing
- Authentication

### CloudFront Security

**NÊN LÀM:**

**Enable WAF Integration:**
```bash
# Tạo Web ACL
aws wafv2 create-web-acl \
  --name my-web-acl \
  --scope CLOUDFRONT \
  --rules file://rules.json
```

**Signed URLs cho Private Content:**
```bash
# Generate signed URL
aws cloudfront sign \
  --url "https://example.com/private/*" \
  --key-pair-id KDPKSTAGING \
  --private-key file://private_key.pem \
  --date-less-than 2024-12-31
```

---

## 9. Architecture Patterns

### ECS Fargate Web Application

```
Internet
    ↓
CloudFront
    ↓
ALB (Multi-AZ)
    ↓
ECS Fargate (Auto Scaling)
    ↓
EFS (Shared Storage)
    ↓
RDS Aurora (Multi-AZ)
```

### ECS với Dynamic Port Mapping

```
ALB (port 80)
    ↓
Security Group: Allow from ALB SG
    ↓
EC2 Instance (multiple tasks)
    ↓
Tasks on dynamic ports (32768+)
```

---

## 10. Exam Tips

### VPC High-Yield Topics:
1. Security Groups vs NACLs (stateful/stateless)
2. NAT Gateway vs NAT Instance
3. VPC Endpoints (Gateway vs Interface)
4. Bastion Host configuration
5. VPC Peering (non-transitive)
6. VPC Flow Logs (ACCEPT/REJECT, Athena queries)

### ECS High-Yield Topics:
1. Task Definition components
2. IAM Roles (Instance Profile vs Task Role)
3. EC2 vs Fargate Launch Types
4. Service Auto Scaling
5. Rolling Updates
6. Environment variables (SSM, Secrets Manager)
7. ECR CLI commands

### CloudFront High-Yield Topics:
1. Origins (S3 with OAC, ALB)
2. Cache behaviors and invalidations
3. CloudFront Functions vs Lambda@Edge
4. Origin Groups cho failover
5. Signed URLs/Cookies

### Common Exam Scenarios:

| Scenario | Solution |
|---------|----------|
| Private EC2 cần S3 access | VPC Endpoint (Gateway) |
| Block malicious IP | NACL với DENY rule |
| SSH đến private instance | Bastion Host |
| Container không quản lý servers | ECS Fargate |
| Lưu Docker images trên AWS | ECR |
| Global content delivery | CloudFront |
| Cache invalidation | CloudFront Invalidation |

### CLI Commands cần nhớ:

```bash
# ECR Login
aws ecr get-login-password | docker login --username AWS --password-stdin $ACCOUNT.dkr.ecr.$REGION.amazonaws.com

# VPC Endpoint
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx --service-name com.amazonaws.us-east-1.s3 --route-table-ids rtb-xxx

# ECS Task Definition
aws ecs register-task-definition --cli-input-json file://task.json

# CloudFront Invalidation
aws cloudfront create-invalidation --distribution-id EXXXX --paths "/*"
```
