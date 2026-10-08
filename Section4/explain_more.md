# Section 4 - Kiến thức mở rộng: Amazon S3, S3 Nâng cao, S3 Security, Amazon VPC

---

## 1. S3 Best Practices

### S3 Bucket Naming

**NÊN LÀM:**
- Sử dụng **tên mô tả, duy nhất**
- Bao gồm environment (dev, staging, prod)
- Bao gồm region hoặc application name
- Sử dụng chữ thường

**KHÔNG NÊN:**
- Sử dụng chữ in hoa
- Sử dụng dấu gạch dưới (underscores)
- Bắt đầu bằng `xn-` hoặc kết thúc bằng `-s3alias`
- Sử dụng IP addresses

### S3 Object Key Design

**NÊN LÀM:**
```
# Cấu trúc tốt
my-bucket/
  images/
    users/
      user123/
        profile.jpg
    products/
      sku456/
        thumbnail.png
  logs/
    2024/
      01/
        access.log
```

**Lợi ích của cấu trúc phân cấp:**
- Tổ chức tốt hơn
- Sử dụng prefix-based lifecycle rules
- Cải thiện hiệu suất (phân tán across prefixes)

**KHÔNG NÊN:**
```
# Xấu - tất cả objects ở root
my-bucket/user123_profile.jpg
my-bucket/sku456_thumbnail.png
```

### S3 Storage Classes Selection

**NÊN LÀM:**

| Loại dữ liệu | Storage Class khuyến nghị |
|-----------|-------------------|
| Truy cập thường xuyên | S3 Standard |
| Truy cập ít (hàng tháng) | S3 Standard-IA |
| Backup, có thể tạo lại | S3 One Zone-IA |
| Archive, một lần/quý | S3 Glacier Instant |
| Archive dài hạn | S3 Glacier Flexible/Deep |
| Chưa biết pattern | S3 Intelligent-Tiering |

**💡 Tip: Tối ưu chi phí**
- S3 Standard-IA: Phí retrieval per-GB cao hơn
- S3 Glacier: Tối thiểu 90 ngày + phí retrieval
- Tính tổng chi phí (storage + retrieval) trước khi chọn

### S3 Lifecycle Rules

**NÊN LÀM:**

```json
{
  "Rules": [
    {
      "ID": "Move-to-IA-after-30days",
      "Status": "Enabled",
      "Filter": {"Prefix": "documents/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"}
      ]
    },
    {
      "ID": "Delete-old-versions",
      "Status": "Enabled",
      "Filter": {},
      "Expiration": {"Days": 365}
    }
  ]
}
```

**Các patterns phổ biến:**
- Current versions: Standard → Standard-IA (30d) → Glacier (90d)
- Previous versions: Standard-IA (30d) → Glacier (90d) → Delete (365d)
- Logs: Xóa sau 30-90 ngày
- Incomplete uploads: Xóa sau 7 ngày

### S3 Replication Strategy

**NÊN LÀM:**

**CRR (Cross-Region Replication):**
- Yêu cầu tuân thủ (data residency)
- Giảm latency cho người dùng toàn cầu
- Disaster Recovery ở region khác
- Không có chaining của replications

**SRR (Same-Region Replication):**
- Log aggregation từ nhiều accounts
- Production → Test environment sync
- Backup trong cùng region

**KHÔNG NÊN:**
- Chain replications (A→B→C) - objects sẽ không đến C
- Enable replication mà không có IAM permissions đúng
- Quên rằng chỉ NEW objects được replicate

### S3 Versioning Best Practices

**NÊN LÀM:**
- Enable versioning cho TẤT CẢ buckets (best practice)
- Sử dụng lifecycle rules để quản lý noncurrent versions
- Set up MFA Delete cho sensitive buckets
- Hiểu rằng suspend không xóa versions

**⚠️ LƯU Ý:**
- Versioning tăng chi phí storage
- Mỗi version đều tính vào giá
- Kết hợp với lifecycle rules để dọn old versions

---

## 2. S3 Security Chi tiết

### S3 Access Control Layers

```
┌─────────────────────────────────────────────┐
│           IAM Policies                       │
│  (User/Role/Group permissions)              │
├─────────────────────────────────────────────┤
│           Bucket Policies                    │
│  (Account-level or cross-account access)    │
├─────────────────────────────────────────────┤
│           ACLs (Object & Bucket)            │
│  (Legacy, granular control)                 │
├─────────────────────────────────────────────┤
│           S3 Access Points                  │
│  (Simplified management)                   │
└─────────────────────────────────────────────┘
```

**Access Decision Logic:**
1. Explicit DENY = DENY
2. User IAM ALLOW OR Resource ALLOW = ALLOW
3. Default = Implicit DENY

### S3 Bucket Policy Examples

**1. Force HTTPS (Encryption in Transit):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ForceHTTPS",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": ["arn:aws:s3:::my-bucket/*"],
      "Condition": {
        "Bool": {"aws:SecureTransport": "false"}
      }
    }
  ]
}
```

**2. Force SSE-KMS Encryption:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ForceKMSEncryption",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

**3. Cross-Account Access:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CrossAccountAccess",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### S3 Encryption Comparison

| Đặc điểm | SSE-S3 | SSE-KMS | SSE-C | Client-Side |
|---------|--------|---------|-------|-------------|
| Key Management | AWS | AWS KMS | Bạn | Bạn |
| Key Rotation | Tự động | Có thể cấu hình | Thủ công | Thủ công |
| Audit Trail | Hạn chế | CloudTrail | Không | Không |
| Performance | Nhanh nhất | Chậm hơn | Chậm hơn | Chậm nhất |
| HTTPS Required | Tùy chọn | Tùy chọn | **Bắt buộc** | Tùy chọn |

**NÊN LÀM:**
- Sử dụng **SSE-S3** cho hầu hết cases (default, hiệu quả)
- Sử dụng **SSE-KMS** khi cần audit trail hoặc customer-managed keys
- Sử dụng **SSE-C** khi phải quản lý keys bên ngoài AWS
- Sử dụng **Client-side** cho mã hóa end-to-end

**⚠️ QUAN TRỌNG:**
- SSE-KMS có giới hạn request rate (5500-30000/s)
- Tăng quota nếu cần
- SSE-C yêu cầu HTTPS - không hoạt động nếu không có

### S3 CORS Configuration

**NÊN LÀM:**

```json
[
  {
    "AllowedOrigin": "https://myapp.example.com",
    "AllowedMethod": ["GET", "PUT", "POST"],
    "AllowedHeader": ["*"],
    "ExposeHeader": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

**Scenario phổ biến:**
- Frontend (CloudFront) → S3 bucket cho static assets
- API calls đến S3 từ origin khác
- Enable CORS trên S3 bucket

### S3 Pre-Signed URLs

**NÊN LÀM:**
- Tạo short-lived URLs cho downloads
- Set expiration phù hợp
- Sử dụng IAM roles cho việc tạo (bảo mật hơn)

**Use cases:**
```bash
# Download (GET)
aws s3 presign s3://bucket/file.txt --expires-in 3600

# Upload (PUT)
aws s3 presign s3://bucket/upload.txt --expires-in 3600
```

**KHÔNG NÊN:**
- Tạo pre-signed URLs với root credentials
- Set expiration quá dài
- Chia sẻ URLs công khai cho sensitive data

### S3 Object Lock vs. Versioning

**Object Lock Modes:**

| Mode | Overwrite | Delete | Change Retention |
|------|-----------|--------|------------------|
| **Compliance** | ❌ (kể cả root) | ❌ | ❌ |
| **Governance** | ❌ (most users) | ❌ (most users) | ✅ (some users) |
| **Legal Hold** | ❌ | ❌ | ❌ (cho đến khi removed) |

**Khi nào sử dụng:**
- **Compliance**: Ngành được quản lý, yêu cầu pháp lý
- **Governance**: Kiểm soát nội bộ, ngăn accidental deletion
- **Legal Hold**: Kiện tụng, điều tra

---

## 3. S3 Performance Optimization

### S3 Performance Guidelines

**NÊN LÀM:**
- **Phân tán GET requests across prefixes**
  - 1 prefix = 5,500 GET/HEAD requests/s
  - 4 prefixes = 22,000 GET/HEAD requests/s
- Sử dụng **CloudFront** CDN đằng trước S3
- Sử dụng **S3 Transfer Acceleration** cho global uploads

**Performance by operation:**

| Operation | Baseline per Prefix |
|-----------|--------------------|
| GET/HEAD | 5,500 requests/s |
| PUT/COPY/POST/DELETE | 3,500 requests/s |

### Multi-Part Upload Strategy

**NÊN LÀM:**
```python
# Cho files > 100MB, luôn sử dụng multipart
# Cho files > 5GB, bắt buộc phải có

import boto3

s3 = boto3.client('s3')

# Automatic multipart cho large files
s3.upload_file('largefile.zip', 'my-bucket', 'largefile.zip')
```

**Cấu hình:**
```python
config = TransferConfig(
    multipart_threshold=1024 * 1024 * 100,  # 100MB
    max_concurrency=10,
    multipart_chunksize=1024 * 1024 * 50   # 50MB chunks
)
```

### S3 Select

**NÊN LÀM:**
- Sử dụng S3 Select để filter CSV/JSON/Parquet
- Retrieve chỉ data cần thiết
- Giảm data transfer và chi phí

```python
response = s3.select_object_content(
    Bucket='my-bucket',
    Key='data.csv',
    ExpressionType='SQL',
    Expression="SELECT * FROM s3object WHERE age > 25",
    InputSerialization={'CSV': {}},
    OutputSerialization={'JSON': {}}
)
```

---

## 4. Amazon VPC Fundamentals

### VPC Sizing

**NÊN LÀM:**

| VPC Size | Usable IPs | Common Use |
|----------|------------|------------|
| /28 | 11 | Small, isolated |
| /27 | 27 | Single app |
| /26 | 59 | Small department |
| /24 | 243 | Medium app |
| /16 | 65,531 | Large, multi-tier |

**RFC 1918 Private Ranges:**
- `10.0.0.0/8` - Large networks
- `172.16.0.0/12` - Medium networks (AWS default VPC)
- `192.168.0.0/16` - Small networks

**KHÔNG NÊN:**
- Sử dụng overlapping CIDRs
- Sử dụng public IP ranges trong VPC
- Tạo subnets quá nhỏ (nhớ -5 IPs)

### Subnet Design

**NÊN LÀM:**

```
VPC: 10.0.0.0/16
├── Public Subnet 1A (AZ 1): 10.0.1.0/24
│   └── NAT Gateway, Load Balancer
├── Private Subnet 1A (AZ 1): 10.0.2.0/24
│   └── Application servers
├── Private Subnet 2A (AZ 2): 10.0.3.0/24
│   └── Databases
└── Reserved: 10.0.4.0/22 (future)
```

**Best Practices:**
- Tách biệt public (Internet-facing) và private subnets
- Deploy across multiple AZs cho HA
- /24 subnet size là phổ biến

### Internet Gateway vs. NAT Gateway

**Internet Gateway (IGW):**
- Cho phép instances với **public IPs** truy cập Internet
- **Outbound AND Inbound** traffic
- Được sử dụng bởi instances trong **public subnets**

**NAT Gateway:**
- Cho phép instances với **private IPs** truy cập Internet
- **Chỉ Outbound** (không có inbound từ Internet)
- Được sử dụng bởi instances trong **private subnets**
- Nằm trong **public subnet**
- Được quản lý bởi AWS (không cần quản lý)

**Khi nào sử dụng:**
| Nhu cầu | Giải pháp |
|------|---------|
| Instance với public IP truy cập Internet | IGW |
| Private instance truy cập Internet (chỉ outbound) | NAT Gateway |
| Không cần Internet access | Không cần |

### Security Groups vs NACLs

**Security Groups (Stateful):**
- Instance level
- **Stateful**: Return traffic tự động được cho phép
- **Default**: Deny all inbound, Allow all outbound
- Evaluate TẤT CẢ rules
- Hỗ trợ referencing other security groups

**Network ACLs (Stateless):**
- Subnet level
- **Stateless**: Phải explicitly allow return traffic
- **Default**: Allow all inbound/outbound
- Evaluate rules theo THỨ TỰ (thấp đến cao)
- Không referencing other security groups

**NÊN LÀM:**

```
Architecture:
Internet → IGW → Route Table → Public Subnet → NACL → Security Group → EC2
```

| Layer | Function | Type |
|-------|----------|------|
| NACL | Subnet-level filtering | Stateless |
| Security Group | Instance-level filtering | Stateful |

### VPC Endpoints

**NÊN LÀM:**

**Gateway Endpoints (S3, DynamoDB):**
- Miễn phí
- Sử dụng route table entries
- Chỉ cho S3 và DynamoDB

```bash
# Tạo Gateway Endpoint cho S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678
```

**Interface Endpoints:**
- Chi phí per hour + data processing
- ENI trong subnet
- Cho các AWS services khác

### VPC Flow Logs

**NÊN LÀM:**

```bash
# Tạo Flow Log
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flow-logs
```

**Log fields:**
- srcaddr, dstaddr
- srcport, dstport
- action (ACCEPT/REJECT)
- packets, bytes

**KHÔNG NÊN:**
- Log vào cùng VPC (tạo logging loop)
- Quên tạo IAM role cho Flow Logs

---

## 5. Architecture Patterns

### Static Website on S3

```
User → CloudFront → S3 Bucket (Static Website)
                ↓
         Route 53 (Alias)
```

**Cấu hình:**
1. Enable Static Website Hosting trên S3
2. Set index.html làm index document
3. Bucket policy: Allow public read
4. Block Public Access: OFF
5. Route 53: Tạo A record (Alias to S3)

### S3 + CloudFront Distribution

**Lợi ích:**
- Global edge locations
- Giảm latency
- DDoS protection
- Caching at edge

**Cache invalidation:**
```bash
# Invalidate single file
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/images/*.jpg"

# Invalidate everything
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

### Multi-Tier VPC Architecture

```
Internet
    ↓
Internet Gateway
    ↓
Route Table
    ↓
┌─────────────────────────────────────┐
│ Public Subnet (AZ1)                 │
│ - NAT Gateway                       │
│ - Application Load Balancer          │
├─────────────────────────────────────┤
│ Private Subnet (AZ1)                │
│ - EC2 Instances (Application)       │
│ - Auto Scaling Group                 │
├─────────────────────────────────────┤
│ Private Subnet (AZ2)                │
│ - RDS Multi-AZ                       │
│ - ElastiCache                        │
└─────────────────────────────────────┘
    ↓
VPC Endpoint (S3, DynamoDB)
```

---

## 6. Exam Tips

### S3 Topics trọng tâm:
1. Storage classes và tối ưu chi phí
2. Lifecycle rules
3. Encryption (SSE-S3, SSE-KMS, SSE-C)
4. Bucket policies (HTTPS enforcement, cross-account)
5. Replication (CRR/SRR)
6. Pre-signed URLs
7. CORS configuration

### VPC Topics trọng tâm:
1. CIDR calculations
2. IGW vs NAT Gateway
3. Security Groups vs NACLs
4. VPC Endpoints
5. Public vs Private subnets
6. Route Tables

### Các Scenario phổ biến trong Exam:
- "Cần host static website" → S3 + Route 53
- "Cần ngăn accidental deletion" → S3 Versioning + Object Lock
- "Cần cross-account access" → Bucket Policy
- "Cần private instances truy cập S3" → VPC Endpoint (Gateway)
- "Cần tính usable IPs" → Subnet size - 5

### VPC CIDR Quick Reference:

| CIDR | Total IPs | Usable IPs |
|------|-----------|------------|
| /28 | 16 | 11 |
| /27 | 32 | 27 |
| /26 | 64 | 59 |
| /25 | 128 | 123 |
| /24 | 256 | 251 |
| /23 | 512 | 507 |
| /22 | 1024 | 1019 |
| /21 | 2048 | 2043 |
| /20 | 4096 | 4091 |
| /19 | 8192 | 8187 |
| /18 | 16384 | 16379 |
| /17 | 32768 | 32763 |
| /16 | 65536 | 65531 |
