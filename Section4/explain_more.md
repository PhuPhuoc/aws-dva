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
- Sử dụng địa chỉ IP (IP addresses)

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
- Sử dụng lifecycle rules theo prefix (prefix-based)
- Cải thiện hiệu suất (phân tán across các prefix)

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
- Disaster Recovery (khôi phục thảm họa) ở Region khác
- Không có chaining của các replications

**SRR (Same-Region Replication):**
- Tổng hợp log (Log aggregation) từ nhiều accounts
- Production → đồng bộ (sync) môi trường Test
- Backup trong cùng Region

**KHÔNG NÊN:**
- Chaining (chuỗi) các replications (A→B→C) - objects sẽ không đến C
- Bật (Enable) replication mà không có IAM permissions đúng
- Quên rằng chỉ NEW objects (object mới) được replicate

### S3 Versioning Best Practices

**NÊN LÀM:**
- Bật (Enable) versioning cho TẤT CẢ buckets (best practice)
- Sử dụng lifecycle rules để quản lý các noncurrent versions (version không hiện hành)
- Thiết lập (Set up) MFA Delete cho các bucket sensitive (nhạy cảm)
- Hiểu rằng suspend (tạm dừng) không xóa các versions

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

**Logic ra quyết định truy cập (Access Decision Logic):**
1. Explicit DENY = DENY (từ chối rõ ràng = từ chối)
2. User IAM ALLOW HOẶC (OR) Resource ALLOW = ALLOW
3. Mặc định (Default) = Implicit DENY (từ chối ngầm)

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
| Quản lý Key (Key Management) | AWS | AWS KMS | Bạn | Bạn |
| Xoay Key (Key Rotation) | Tự động | Có thể cấu hình | Thủ công | Thủ công |
| Theo dõi (Audit Trail) | Hạn chế | CloudTrail | Không | Không |
| Hiệu năng (Performance) | Nhanh nhất | Chậm hơn | Chậm hơn | Chậm nhất |
| HTTPS bắt buộc (HTTPS Required) | Tùy chọn | Tùy chọn | **Bắt buộc** | Tùy chọn |

**NÊN LÀM:**
- Dùng **SSE-S3** cho hầu hết các case (mặc định, hiệu quả)
- Dùng **SSE-KMS** khi cần audit trail (theo dõi) hoặc customer-managed keys (key do khách hàng quản lý)
- Dùng **SSE-C** khi phải quản lý keys bên ngoài AWS
- Dùng **Client-side** cho mã hóa end-to-end (từ đầu đến cuối)

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

**Kịch bản phổ biến (Scenario phổ biến):**
- Frontend (CloudFront) → S3 bucket cho static assets (tài nguyên tĩnh)
- Các API calls (gọi API) đến S3 từ origin (nguồn) khác
- Bật (Enable) CORS trên S3 bucket

### S3 Pre-Signed URLs

**NÊN LÀM:**
- Tạo URL short-lived (hết hạn nhanh) cho downloads (tải xuống)
- Đặt thời gian hết hạn (expiration) phù hợp
- Dùng IAM roles cho việc tạo (an toàn hơn)

**Use cases:**
```bash
# Download (GET)
aws s3 presign s3://bucket/file.txt --expires-in 3600

# Upload (PUT)
aws s3 presign s3://bucket/upload.txt --expires-in 3600
```

**KHÔNG NÊN:**
- Tạo pre-signed URL bằng root credentials
- Đặt thời gian hết hạn (expiration) quá dài
- Chia sẻ URL công khai (public) cho dữ liệu nhạy cảm (sensitive data)

### S3 Object Lock vs. Versioning

**Object Lock Modes:**

| Mode | Overwrite | Delete | Change Retention |
|------|-----------|--------|------------------|
| **Compliance** | ❌ (kể cả root) | ❌ | ❌ |
| **Governance** | ❌ (most users) | ❌ (most users) | ✅ (some users) |
| **Legal Hold** | ❌ | ❌ | ❌ (cho đến khi removed) |

**Khi nào sử dụng:**
- **Compliance (Tuân thủ)**: Ngành được quản lý (regulated), yêu cầu pháp lý
- **Governance (Quản trị)**: Kiểm soát nội bộ, ngăn việc xóa nhầm (accidental deletion)
- **Legal Hold (Phong tỏa pháp lý)**: Tranh tụng (Kiện tụng), điều tra

---

## 3. S3 Performance Optimization

### S3 Performance Guidelines

**NÊN LÀM:**
- **Phân tán GET requests theo các prefix** (across prefixes)
  - 1 prefix = 5,500 request GET/HEAD/s
  - 4 prefixes = 22,000 request GET/HEAD/s
- Dùng **CloudFront** CDN phía trước S3
- Dùng **S3 Transfer Acceleration** cho các upload toàn cầu (global)

**Hiệu năng theo thao tác (Performance by operation):**

| Thao tác (Operation) | Baseline theo Prefix |
|-----------|--------------------|
| GET/HEAD | 5,500 request/s |
| PUT/COPY/POST/DELETE | 3,500 request/s |

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

| VPC Size | Số IP khả dụng (Usable IPs) | Mục đích dùng (Common Use) |
|----------|------------|------------|
| /28 | 11 | Nhỏ, cách ly (isolated) |
| /27 | 27 | Một ứng dụng (single app) |
| /26 | 59 | Phòng ban nhỏ (small department) |
| /24 | 243 | Ứng dụng vừa (medium app) |
| /16 | 65,531 | Lớn, nhiều tầng (large, multi-tier) |

**Các dải riêng tư RFC 1918 (RFC 1918 Private Ranges):**
- `10.0.0.0/8` - Mạng lớn (large networks)
- `172.16.0.0/12` - Mạng vừa (medium networks, AWS default VPC)
- `192.168.0.0/16` - Mạng nhỏ (small networks)

**KHÔNG NÊN:**
- Sử dụng các CIDR trùng (overlapping)
- Sử dụng dải IP công cộng (public IP ranges) trong VPC
- Tạo subnets quá nhỏ (nhớ công thức: khả dụng = tổng số - 5 IP)

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

**Thực hành tốt (Best Practices):**
- Tách biệt subnet public (Internet-facing) và subnet private
- Triển khai (Deploy) trên nhiều AZ cho HA (High Availability - cao khả dụng)
- Kích thước subnet /24 là phổ biến

### Internet Gateway vs. NAT Gateway

**Internet Gateway (IGW):**
- Cho phép instances có **IP public** (IP công cộng) truy cập Internet
- Cả **Outbound và Inbound** (đi ra và đi vào) traffic
- Được dùng bởi các instances trong **subnet public** (public subnets)

**NAT Gateway:**
- Cho phép instances có **IP private** (IP riêng tư) truy cập Internet
- **Chỉ Outbound** (đi ra) (không có inbound đi vào từ Internet)
- Được dùng bởi các instances trong **subnet private** (private subnets)
- Được đặt trong **subnet public** (public subnet)
- Được AWS quản lý (không cần tự quản lý)

**Khi nào sử dụng:**
| Nhu cầu | Giải pháp |
|------|---------|
| Instance có IP public truy cập Internet | IGW |
| Instance private truy cập Internet (chỉ outbound) | NAT Gateway |
| Không cần truy cập Internet (Internet access) | Không cần |

### Security Groups vs NACLs

**Security Groups (Stateful):**
- Cấp instance (instance level)
- **Stateful (hữu trạng thái)**: Return traffic (lưu lượng phản hồi) được tự động cho phép
- **Mặc định (Default)**: Deny (từ chối) mọi inbound (đi vào), Allow (cho phép) mọi outbound (đi ra)
- Đánh giá (Evaluate) TẤT CẢ rules
- Hỗ trợ tham chiếu (referencing) các security group khác

**Network ACLs (Stateless):**
- Cấp subnet (subnet level)
- **Stateless (vô trạng thái)**: Phải explicitly allow (cho phép rõ) return traffic
- **Mặc định (Default)**: Allow (cho phép) mọi inbound/outbound
- Đánh giá (Evaluate) rules theo THỨ TỰ (từ thấp đến cao)
- Không tham chiếu (referencing) các security group khác

**NÊN LÀM:**

```
Architecture:
Internet → IGW → Route Table → Public Subnet → NACL → Security Group → EC2
```

| Lớp (Layer) | Chức năng (Function) | Loại (Type) |
|-------|----------|------|
| NACL | Lọc (filtering) ở cấp subnet | Stateless (vô trạng thái) |
| Security Group | Lọc (filtering) ở cấp instance | Stateful (hữu trạng thái) |

### VPC Endpoints

**NÊN LÀM:**

**Gateway Endpoints (S3, DynamoDB):**
- Miễn phí (không thu phí)
- Dùng route table entries (các entry trong bảng định tuyến)
- Chỉ dùng cho S3 và DynamoDB

```bash
# Tạo Gateway Endpoint cho S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678
```

**Interface Endpoints:**
- Có chi phí theo giờ (per hour) + phí xử lý data (data processing)
- Có ENI (Elastic Network Interface - giao diện mạng ảo) trong subnet
- Dùng cho các AWS services khác (ngoài S3, DynamoDB)

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

**Các trường log (Log fields):**
- srcaddr (địa chỉ nguồn), dstaddr (địa chỉ đích)
- srcport (cổng nguồn), dstport (cổng đích)
- action (hành động: ACCEPT/REJECT)
- packets (số gói), bytes (số byte)

**KHÔNG NÊN:**
- Log vào cùng VPC (tạo vòng lặp log - logging loop)
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
1. Bật (Enable) Static Website Hosting trên S3
2. Đặt (Set) index.html làm index document
3. Bucket policy: cho phép đọc công khai (Allow public read)
4. Block Public Access (Chặn truy cập công khai): TẮT (OFF)
5. Route 53: Tạo A record (trỏ Alias tới S3)

### S3 + CloudFront Distribution (Phân phối S3 + CloudFront)

**Lợi ích:**
- Các vị trí edge toàn cầu (Global edge locations)
- Giảm độ trễ (latency)
- Bảo vệ DDoS (DDoS protection)
- Caching tại các điểm edge (Caching at edge)

**Invalidation cache (loại bỏ cache):**
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

### Các chủ đề S3 trọng tâm (S3 Topics):
1. Storage classes và tối ưu chi phí
2. Lifecycle rules (quy tắc vòng đời)
3. Encryption (SSE-S3, SSE-KMS, SSE-C)
4. Bucket policies (bắt buộc HTTPS - HTTPS enforcement, cross-account)
5. Replication (nhân bản: CRR/SRR)
6. Pre-signed URLs (URL ký trước)
7. Cấu hình CORS (CORS configuration)

### Các chủ đề VPC trọng tâm (VPC Topics):
1. Tính toán CIDR (CIDR calculations)
2. IGW vs NAT Gateway
3. Security Groups vs NACLs
4. VPC Endpoints
5. Subnet Public so với Subnet Private (Public vs Private subnets)
6. Route Tables (bảng định tuyến)

### Các Scenario phổ biến trong Exam:
- "Cần host (chủ trì) static website" → S3 + Route 53
- "Cần ngăn (prevent) accidental deletion (xóa nhầm)" → S3 Versioning + Object Lock
- "Cần cross-account access (truy cập xuyên account)" → Bucket Policy
- "Cần private instances truy cập S3" → VPC Endpoint (Gateway)
- "Cần tính usable IPs (số IP khả dụng)" → Subnet size - 5

### Tra cứu nhanh CIDR cho VPC (VPC CIDR Quick Reference):

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
