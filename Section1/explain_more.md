# Giải thích bổ sung - Section 1

> *Phần này chứa kiến thức thực tế từ kinh nghiệm triển khai AWS. Đánh dấu **NÊN** hoặc **KHÔNG NÊN** để dễ nhớ.*

---

## 1. AWS Global Infrastructure

### Region Selection

**NÊN:**
- Chọn region **gần end users nhất** để giảm latency
- Ở Việt Nam thường dùng: **Singapore (ap-southeast-1)** hoặc **Sydney (ap-southeast-2)**
- Kiểm tra services có sẵn ở region trước khi design architecture

**KHÔNG NÊN:**
- Chọn region chỉ vì giá rẻ nếu end users ở xa → latency cao
- Hard-code region name trong code → khó migrate

### Availability Zones

**Thực tế:**
- Các AZ trong cùng region cách nhau **vài km** nhưng đủ xa để tránh cùng disaster
- Kết nối giữa AZ có **latency ~1-2ms** → vẫn nhanh nhưng cần consider cho distributed systems

**NÊN:**
- Luôn deploy resources spanning **ít nhất 2 AZ** cho high availability
- Thiết kế application có khả năng chịu lỗi 1 AZ mà không ảnh hưởng toàn bộ

**LƯU Ý QUAN TRỌNG:**
- Không phải region nào cũng có 3 AZ. Một số region mới chỉ có 2 AZ
- Kiểm tra: `aws ec2 describe-availability-zones --region ap-southeast-1`

---

## 2. IAM

### Users & Groups

**NÊN:**
- Tạo user riêng cho **mỗi người**, không dùng chung account
- Sử dụng **Group** để quản lý permissions theo role (Admin, Developer, Support)
- Áp dụng **least privilege** từ đầu, sau đó mở rộng nếu cần

**KHÔNG NÊN:**
- Attach policy trực tiếp vào user → khó quản lý
- Dùng AdministratorAccess cho mọi thứ → vi phạm security principle

### Password Policy

**NÊN:**
```json
{
  "MinimumPasswordLength": 14,
  "RequireSymbols": true,
  "RequireNumbers": true,
  "RequireUppercaseCharacters": true,
  "RequireLowercaseCharacters": true,
  "PasswordReusePrevention": 24,
  "ExpirePasswords": true,
  "MaxPasswordAge": 90
}
```

### MFA

**BẮT BUỘC cho:**
- Root account (càng sớm càng tốt)
- Tài khoản có quyền Admin
- Production access

**NÊN dùng:**
- **Hardware Key (YubiKey)** cho developers có budget
- **Authy** cho multi-device support (điện thoại + tablet)

**KHÔNG NÊN:**
- SMS MFA → có thể bị SIM swap attack
- Virtual MFA trên cùng device với password → mất cả 2 cùng lúc

### Access Keys

**NÊN:**
- Sử dụng **IAM Role** thay vì Access Key cho EC2, Lambda
- Rotation Access Key **90 ngày/lần**
- Lưu Access Key ở **AWS Secrets Manager** hoặc **Parameter Store** nếu cần

**KHÔNG NÊN:**
- Hard-code Access Key trong code → lộ key
- Lưu Access Key trong Git → git history sẽ lưu mãi mãi
- Dùng root account Access Key → hoàn toàn không an toàn

### IAM Roles

**Sử dụng khi nào:**
- EC2 instance cần access AWS services
- Lambda function cần permissions
- ECS task cần AWS resources
- Cross-account access

**NÊN:**
- Mỗi Lambda function nên có **role riêng** với permissions vừa đủ
- Sử dụng **IAM Role** thay vì Access Key cho ứng dụng chạy trên AWS

### IAM Best Practices Summary

| Thực hành | Priority |
|-----------|----------|
| Enable MFA everywhere | Critical |
| Không dùng root account | Critical |
| One user per person | High |
| Use groups | High |
| Least privilege | High |
| Regular audit | Medium |
| Access Key rotation | Medium |

---

## 3. EC2

### Instance Types - Khi nào dùng gì?

| Use Case | NÊN DÙNG | KHÔNG NÊN |
|----------|----------|-----------|
| Web server nhỏ | t3.micro, t3.small | c5.large (overkill) |
| Web server lớn | m5.large, m5.xlarge | t3.micro (yếu) |
| Batch processing | c5.2xlarge, c5.4xlarge | r5.large (lãng phí RAM) |
| Database | r5.large, r5.xlarge | t3.micro (không đủ RAM) |
| Cache (Redis) | r5b, im4gn (NVMe SSD) | c5 (không có local SSD) |
| File storage | i3 (local NVMe) | r5 (RAM không cần thiết) |

### General Purpose (T, M series)

**T series (Burstable):**
- Phù hợp: dev/test, web nhỏ, database nhỏ
- **Credit system**: có CPU credits để burst khi cần
- **NÊN**: dùng cho workload không liên tục, có baseline thấp
- **KHÔNG NÊN**: dùng cho production với workload ổn định 24/7 → CPU credits hết → throttled

**M series:**
- Balance compute/memory
- Phù hợp: general purpose web apps, databases vừa

### Compute Optimized (C series)

- Phù hợp: batch processing, video encoding, ML inference
- **LƯU Ý**: High performance nhưng **không có local storage** → phải dùng EBS

### Memory Optimized (R, X series)

- Phù hợp: Database lớn, In-memory cache, SAP HANA
- **R6i, R5**: DDR4 RAM
- **R6b**: DDR5 RAM (mới hơn, nhanh hơn)

### Storage Optimized (I, D, H series)

| Series | Storage Type | Use Case |
|--------|-------------|----------|
| **I3** | NVMe SSD | Database, NoSQL, Cache |
| **I4i** | NVMe SSD | Database với better performance |
| **D2** | HDD | HDFS, MapReduce |
| **H1** | HDD | Big data, data warehousing |

**NÊN:** Dùng I series cho database với high IOPS requirements

### User Data Script

**Sử dụng khi nào:**
- Install packages khi launch
- Setup application đầu tiên
- Pull code từ GitHub/S3

**Ví dụ:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd php php-mysql
systemctl start httpd
systemctl enable httpd
```

**LƯU Ý:**
- Script chỉ chạy **1 lần** khi instance được create
- Không chạy lại khi restart
- Nếu cần re-run → dùng cloud-init hoặc SSM Run Command

### Security Groups

**NGUYÊN TẮC VÀNG:**

1. **Chỉ mở port cần thiết**
   - SSH (22): Chỉ từ IP của bạn hoặc bastion host
   - HTTP (80): 0.0.0.0/0 nếu cần public web
   - HTTPS (443): 0.0.0.0/0 nếu cần public web
   - Database ports (3306, 5432): Chỉ từ app servers

2. **Reference Security Group thay vì IP**
   - Thay vì: `0.0.0.0/0` → dùng: `sg-0123456789abcdef0` (app server SG)
   - Khi app server IP thay đổi, SG tự động update

3. **Layered Security Groups**
   ```
   Internet → ALB Security Group (port 80, 443)
   ALB → Web Server SG (chỉ từ ALB SG)
   Web Server → App SG (chỉ từ Web SG)
   App Server → Database SG (chỉ từ App SG)
   ```

**NÊN:**
- Tạo **Security Group riêng** cho mỗi tier (web, app, db)
- Name convention rõ ràng: `sg-webservers-prod`, `sg-databases-prod`

**KHÔNG NÊN:**
- Mở SSH từ 0.0.0.0/0 → nguy hiểm
- Attach nhiều SG giống nhau vào nhiều resources → khó quản lý

### Classic Ports

| Port | Security Group Rule |
|------|-------------------|
| 22 (SSH) | Chỉ từ your IP hoặc bastion |
| 80 (HTTP) | 0.0.0.0/0 (nếu public web) |
| 443 (HTTPS) | 0.0.0.0/0 (nếu public web) |
| 3306 (MySQL) | Chỉ từ app server SG |
| 5432 (PostgreSQL) | Chỉ từ app server SG |
| 6379 (Redis) | Chỉ từ app server SG (private) |
| 27017 (MongoDB) | Chỉ từ app server SG (private) |

### EC2 Instance Connect

**Ưu điểm:**
- Không cần key pair
- Browser-based terminal
- Audit được ai đã SSH

**Điều kiện:**
- Instance phải có **public IP** hoặc qua **bastion**
- Security Group phải allow port 22
- IAM permission cần thiết

### Purchasing Options - So sánh thực tế

#### On-Demand

**Khi nào dùng:**
- Development/Testing environment
- Short-term unpredictable workloads
- First-time deployment để estimate resource needs

**Nhược điểm:**
- **Đắt nhất** ($/hour)
- Không có cam kết

#### Reserved Instances

**Khi nào dùng:**
- Production database chạy 24/7
- Steady-state applications
- Khi đã estimate được resource needs

**TIẾT KIỆM:** 72% so với On-Demand (3 years, All Upfront)

**LƯU Ý QUAN TRỌNG:**
- RI không đảm bảo capacity → có thể không có instance available
- **Zonal RI** → reserve capacity trong specific AZ
- **Regional RI** → không reserve capacity, chỉ giảm price

#### Savings Plans

**Khác với Reserved:**
- Flexible về instance size trong cùng family
- Ví dụ: M5.2xlarge có thể chạy M5.xlarge (tự động)

**NÊN dùng khi:**
- Workload có thể thay đổi size trong tương lai
- Muốn tiết kiệm mà vẫn linh hoạt

#### Spot Instances

**Ưu điểm:**
- Giảm đến 90%
- Tuyệt vời cho batch jobs, data processing

**NHƯỢC ĐIỂM:**
- **Có thể bị terminated bất cứ lúc nào** với 2 phút notice
- Không đảm bảo availability

**NÊN dùng cho:**
- Batch processing (retry được)
- Spark/Hadoop clusters
- CI/CD build agents
- Machine learning training (checkpoint để resume)

**KHÔNG NÊN dùng cho:**
- Database (data loss risk)
- Production web servers
- Anything requires 24/7 uptime

**Chiến lược Spot:**
```yaml
# Ví dụ: Auto Scaling Group với Spot
instanceTypes:
  - m5.large
  - m5.xlarge
  - m4.large
SpotAllocationStrategy: "lowest-price"
```

#### Dedicated Hosts

**Khi nào dùng:**
- Compliance requirements (some organizations cannot share hardware)
- Existing per-socket/per-core licenses
- regulatory requirements

**Nhược điểm:**
- **Đắt nhất**
- Hard to get availability

#### Capacity Reservations

**Khi nào dùng:**
- Cần guarantee capacity trong specific AZ
- Short-term critical workloads
- BackupDR scenarios

**LƯU Ý:** Không tiết kiệm gì so với On-Demand

### EBS Storage - Kinh nghiệm thực tế

#### Volume Types

| Type | Use Case | NÊN/KHÔNG |
|------|----------|-----------|
| **gp3** | General purpose (default) | ✅ NÊN - giá rẻ nhất |
| **gp2** | Legacy, đang deprecated | ⚠️ Migrate sang gp3 |
| **io2** | High performance database | ✅ NÊN cho mission-critical |
| **io2 Block Express** | Ultra high performance | ✅ SAP, Oracle |
| **st1** | Throughput-optimized HDD | ⚠️ Boot volume không dùng |
| **sc1** | Cold storage | ⚠️ Rất chậm |

#### EBS Best Practices

**NÊN:**
- Dùng **gp3** làm default cho hầu hết workloads
- Enable **EBS encryption** (default khi tạo mới)
- Attach nhiều volumes thay vì 1 volume lớn → dễ manage
- Snapshot regular để backup

**KHÔNG NÊN:**
- Cấu hình **EBS lớn từ đầu** → sau này muốn giảm rất khó
- Dùng EBS cho **temporary data** → nên dùng Instance Store
- Tắt **Delete on Termination** nếu muốn giữ data sau khi terminate

**VỀ KÍCH THƯỚC EBS:**
```
❌ KHÔNG NÊN: 1000 GB gp3 từ đầu
✅ NÊN: 50 GB gp3, sau đó resize (dễ dàng online)
```

**VỀ IOPS:**
- gp3: 3000 IOPS base, có thể increase đến 16000
- io2: 99.999% durability, 256000 IOPS max

#### Backup EBS

**Phương pháp:**

1. **EBS Snapshots (khuyến nghị):**
   - Automation: Data Lifecycle Manager (DLM)
   - Hoặc AWS Backup
   - Cross-region copy cho DR

2. **Automated Snapshots:**
```bash
# Tạo snapshot
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "Daily backup"
```

3. **Với Database:**
   - Dùng **RDS** thay vì EC2 + EBS cho database
   - RDS tự động backup với point-in-time recovery
   - Multi-AZ deployment cho real-time redundancy

### SSH và Bastion Host

**Vấn đề bảo mật thực tế:**

**❌ KHÔNG NÊN:**
```bash
# SSH trực tiếp vào production EC2
ssh -i key.pem ec2-user@public-ip
```

**✅ NÊN - Sử dụng Bastion Host:**
```
Developer Laptop 
    ↓ (SSH)
Bastion Host (1 instance, public IP, strict SG)
    ↓ (SSH via Systems Manager Session Manager)
Private EC2 (no public IP)
```

**Hoặc tốt hơn - Systems Manager Session Manager:**
- Không cần bastion host
- Không cần port 22 mở
- Có audit logs
- Không cần key pair

```bash
# Kết nối qua SSM
aws ssm start-session --target instance-id
```

### EC2 Placement Groups

| Type | Mô tả | Use Case |
|------|--------|----------|
| **Cluster** | Instances trong cùng AZ | HPC, ML training (low latency) |
| **Partition** | Spread across partitions | HDFS, HBase, Cassandra |
| **Spread** | Spread across hardware | Critical applications |

**LƯU Ý:**
- Cluster: Rủi ro - nếu hardware fail, fail hết
- Spread: Đảm bảo high availability

---

## 4. AWS CLI & SDK

### AWS CLI Best Practices

**Cài đặt:**
```bash
# Linux/macOS
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify
aws --version
```

**Configuration:**
```bash
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region name: ap-southeast-1
# Default output format: json
```

**Sử dụng Profile:**
```bash
# Tạo named profile
aws configure --profile production

# Sử dụng
aws s3 ls --profile production
```

### AWS SDK

**Python (Boto3) - Phổ biến nhất:**
```python
import boto3

# Sử dụng default credentials
s3 = boto3.client('s3')
s3.list_buckets()

# Với specific credentials
dynamodb = boto3.client(
    'dynamodb',
    region_name='ap-southeast-1',
    aws_access_key_id='...',
    aws_secret_access_key='...'
)
```

**KHÔNG NÊN:**
- Hard-code credentials trong code
- Commit credentials vào Git

**✅ NÊN:**
- Sử dụng IAM Role cho EC2/Lambda
- Sử dụng AWS Secrets Manager cho credentials
- Sử dụng environment variables

---

## Tóm tắt Key Takeaways

### Phải làm ngay:
1. ✅ Enable MFA cho root account
2. ✅ Tạo IAM user riêng cho bản thân
3. ✅ Không dùng root account cho công việc hàng ngày
4. ✅ Sử dụng Security Groups đúng cách
5. ✅ Backup EBS volumes

### Không nên:
1. ❌ Mở SSH từ 0.0.0.0/0
2. ❌ Hard-code credentials
3. ❌ Dùng quá nhiều permissions
4. ❌ Dùng Spot cho database
5. ❌ Cấu hình EBS quá lớn từ đầu
