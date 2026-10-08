# Giải thích bổ sung - Section 2

> *Phần này chứa kiến thức thực tế từ kinh nghiệm triển khai AWS. Mỗi topic có giải thích cốt lõi, NÊN/KHÔNG NÊN với lý do cụ thể.*

---

## 1. EBS Volume Types

### Chọn EBS Volume Type - So sánh

#### Giải thích cốt lõi

EBS có 6 loại volume, chia thành 2 nhóm chính:

**SSD-based (3 loại):**

| Type | IOPS Max | Throughput Max | Durability | Boot Volume |
|------|----------|----------------|------------|-------------|
| **gp3** | 16,000 | 1,000 MiB/s | 99.8-99.9% | ✅ |
| **gp2** | 16,000 | 250 MiB/s | 99.8-99.9% | ✅ |
| **io2 Block Express** | 256,000 | 4,000 MiB/s | **99.999%** | ✅ |

**HDD-based (2 loại):**

| Type | Throughput Max | IOPS Max | Boot Volume |
|------|---------------|----------|-------------|
| **st1** | 500 MiB/s | 500 | ❌ |
| **sc1** | 250 MiB/s | 250 | ❌ |

**Điểm khác biệt quan trọng giữa gp2 và gp3:**

| Feature | gp2 | gp3 |
|---------|-----|-----|
| **IOPS vs Size** | Linked (3 IOPS/GB) | **Independent** |
| **IOPS base** | Tỷ lệ với size | Fixed 3,000 |
| **Provisioned IOPS** | Không | ✅ Có thể increase |
| **Throughput** | Max 250 MiB/s | Max 1,000 MiB/s |
| **Cost** | Đắt hơn gp3 | **Rẻ hơn** |

#### NÊN làm:
- **Dùng gp3 làm default** → Vì gp3 rẻ hơn gp2, cho phép tăng IOPS và throughput độc lập với size. Đây là lựa chọn tối ưu cho hầu hết workloads.
- **Dùng io2 Block Express cho mission-critical database** → Vì durability 99.999% (vs 99.9% của gp3), IOPS lên đến 256,000, và sub-millisecond latency. So với gp3 chỉ đủ 16,000 IOPS, io2 phù hợp cho database cần nhiều hơn.

#### KHÔNG NÊN làm:
- **Dùng gp2 cho database mới** → Vì gp2 có IOPS linked với size (3 IOPS/GB). Muốn 9,000 IOPS → phải có 3,000 GB volume. Trong khi gp3 có thể provision 9,000 IOPS ngay với 100 GB. NÊN: dùng gp3 với provisioned IOPS.
- **Dùng st1/sc1 làm boot volume** → Vì HDD-based volumes không support boot volume. NÊN: dùng gp3 hoặc gp2 cho OS.

---

### EBS Volume Size

#### Giải thích cốt lõi

EBS có 2 operations chính với size:

**Upsize (tăng size):**
- ✅ Modify volume online → **không downtime**
- ✅ Không cần stop instance
- ✅ Immediate effect

**Downsize (giảm size):**
- ❌ **Không có direct operation**
- Phải làm thủ công:
  ```
  1. Stop instance (downtime)
  2. Create snapshot (backup phòng data loss)
  3. Create new volume (size nhỏ hơn)
  4. Attach new volume
  5. Mount → copy data từ volume cũ
  6. Umount → detach volume cũ
  7. Delete volume cũ (nếu không cần)
  ```
- Tốn thời gian, cần downtime, risk data loss nếu làm sai

**So sánh với alternatives:**

| Storage | Upsize | Downsize | Persistence | Cost |
|---------|--------|----------|-------------|------|
| **EBS** | ✅ Online | ❌ Manual | ✅ | $/GB |
| **Instance Store** | ❌ | ❌ | ❌ Ephemeral | Included |
| **EFS** | ✅ Auto | ✅ Auto | ✅ | 3x gp2 |

#### NÊN làm:
- **Bắt đầu với size nhỏ vừa đủ, tăng dần** → Vì upsizing chỉ cần 1 click trên console, effect immediate, không downtime. Downsizing phải manual process phức tạp. So với việc overprovision từ đầu rồi không dùng hết → lãng phí tiền.

#### KHÔNG NÊN làm:
- **Overprovision EBS size từ đầu "cho chắc"** → Vì nếu sau này data thật sự cần ít hơn, bạn không thể giảm. Phải tạo volume mới, copy data, swap → mất thời gian và có risk. NÊN: estimate dựa trên initial requirements + buffer nhỏ, sau đó resize khi có data thật.

---

### EBS Encryption

#### Giải thích cốt lõi

EBS encryption sử dụng **AES-256** thông qua **AWS KMS**:

**Encryption được applied ở đâu:**
- Data at rest (trong volume)
- Data in flight (giữa instance và volume)
- All snapshots từ encrypted volume
- All volumes từ encrypted snapshot

**Quy trình encrypt unencrypted volume:**

```
Unencrypted Volume
    ↓ Create Snapshot
Unencrypted Snapshot
    ↓ Copy (with encryption enabled)
Encrypted Snapshot ← (snapshot mới đã encrypt)
    ↓ Create Volume
Encrypted Volume
    ↓ Attach
Instance
```

**Điểm quan trọng:**
- Encryption có **minimal impact on latency** (thường <3%)
- Không thể encrypt in-place
- Encryption không thể disable trên encrypted volume

#### NÊN làm:
- **Để encryption enabled by default** → Vì AWS tự enable khi tạo volume mới. Encryption không ảnh hưởng performance đáng kể nhưng bảo vệ data nếu disk bị leak.
- **Dùng KMS keys có rotation** → Vì customer managed keys cho phép automatic rotation (hàng năm), tăng security.

#### KHÔNG NÊN làm:
- **Tắt encryption để "cải thiện performance"** → Vì encryption chỉ ảnh hưởng ~3% latency. Trong hầu hết cases, đây là acceptable tradeoff cho security. Performance-sensitive cases nên test trước.

---

### EBS Backup với Snapshots

#### Giải thích cốt lõi

**EBS Snapshot mechanism:**
- Snapshot là **incremental** → chỉ backup những blocks đã thay đổi
- Lần đầu full backup, các lần sau chỉ incremental → tiết kiệm storage
- Có thể copy **across AZ** hoặc **across Regions**

**Storage tiers cho snapshots:**

| Tier | Cost | Restore Time | Use Case |
|------|------|--------------|----------|
| **Standard** | $0.05/GB | Immediate | Recent backups |
| **Archive** | $0.0125/GB (75% off) | 24-72 hours | Long-term retention |
| **Recycle Bin** | $0.03/GB | Immediate | Prevent accidental deletion |

**Recycle Bin (Bộ rác):**
- Setup rules để retain deleted snapshots
- Retention: 1 ngày → 1 năm
- Restore được sau khi "delete"

#### NÊN làm:
- **Dùng Lifecycle Policy cho automated backup** → Vì đảm bảo backup đều đặn mà không cần manual. Có thể setup daily/weekly backup với retention policy tự động.
- **Setup Recycle Bin với retention 1-7 ngày** → Vì prevent accidental deletion. Nếu ai đó xóa nhầm, vẫn có thời gian recover.

#### KHÔNG NÊN làm:
- **Không có backup strategy** → Vì EBS không đảm bảo 100% durability. Mặc dù durability cao (99.8-99.999%), trong scale lớn vẫn có risk mất data.
- **Lưu snapshots quá lâu không cần** → Vì tốn tiền. NÊN: dùng Archive tier cho snapshots >30 ngày không cần frequent access.

---

## 2. EC2 Instance Store

### Khi nào dùng Instance Store

#### Giải thích cốt lõi

**Instance Store là gì:**
- Storage được attach **trực tiếp vào instance** (không qua network)
- NVMe SSD → **ultra-high performance**
- **Ephemeral** → data mất khi instance stop/terminate

**So sánh với EBS:**

| Feature | Instance Store | EBS |
|---------|---------------|-----|
| **Latency** | ~0.1ms (local NVMe) | ~1-3ms (network) |
| **IOPS** | Up to millions | Max 256,000 (io2) |
| **Persistence** | ❌ Mất khi stop | ✅ Persistent |
| **Cost** | Included in instance | Extra $/GB |
| **Resize** | ❌ Không thể | ✅ Online resize |

**Tại sao Instance Store nhanh hơn:**
```
Instance Store: CPU ↔ NVMe (direct attach)
                ↓ ~0.1ms

EBS: CPU → Network → EBS Service → NVMe
                ↓ ~1-3ms
```

#### NÊN làm:
- **Dùng cho cache, buffer, scratch data** → Vì data có thể rebuild được. Performance cao quan trọng hơn persistence. Ví dụ: Redis cache, temporary processing files.

#### KHÔNG NÊN làm:
- **Dùng cho persistent data (database, files)** → Vì data mất khi instance stop. Không có backup mechanism. NÊN: dùng EBS hoặc EFS cho persistent data.

---

## 3. EFS - Elastic File System

### Khi nào nên dùng EFS

#### Giải thích cốt lõi

**EFS là gì:**
- **Managed NFS v4.1** → POSIX-compliant filesystem
- Mount được trên **hàng trăm instances** cùng lúc
- **Multi-AZ** → highly available by default
- Auto-scaling → không cần estimate capacity trước

**So sánh với EBS và S3:**

| Feature | EFS | EBS | S3 |
|---------|-----|-----|-----|
| **Access** | Multiple instances | Single instance | Multiple via API |
| **Protocol** | NFS (filesystem) | Block | Object |
| **Persistence** | ✅ | ✅ | ✅ |
| **AZ-bound** | Multi-AZ | Single AZ | Global |
| **Cost** | 3x gp2 | $/GB | $/GB + requests |
| **Windows** | ❌ | ✅ | ✅ |

**EFS Performance Modes:**

| Mode | Description | Use Case |
|------|------------|----------|
| **General Purpose** | Low latency, burst | Web servers, CMS, development |
| **Max I/O** | Higher throughput, higher latency | Big data, ML, genomics |

**EFS Storage Classes:**

| Class | Access Pattern | Cost | Restore |
|-------|---------------|------|---------|
| **Standard** | Frequent | Highest | Immediate |
| **IA (Infrequent Access)** | Few times/quarter | ~90% cheaper | Tens of ms |
| **Archive** | Few times/year | ~95% cheaper | Hours |

#### NÊN làm:
- **Dùng EFS cho shared file storage giữa nhiều instances** → Vì EBS chỉ attach được 1 instance (trừ io1/io2 Multi-Attach - phức tạp). EFS cho phép hàng trăm instances mount cùng lúc qua NFS.
- **Dùng Lifecycle policies (EFS IA)** → Vì tự động chuyển files không accessed >90 ngày sang tier rẻ hơn. Tiết kiệm 90% cost mà không cần manual management.

#### KHÔNG NÊN làm:
- **Dùng EFS cho single instance** → Vì EFS đắt hơn EBS ~3x. Nếu chỉ 1 instance, dùng EBS đủ và rẻ hơn.
- **Dùng EFS cho ultra-low latency requirement** → Vì EFS đi qua network (NFS), latency cao hơn EBS hoặc Instance Store. NÊN: dùng Instance Store cho cache, EBS cho database.

---

## 4. AMI

### Tạo và quản lý AMI

#### Giải thích cốt lõi

**AMI vs EBS Snapshot:**

| | AMI | EBS Snapshot |
|---|-----|-------------|
| **Là gì** | Template để launch instance | Backup của 1 volume |
| **Launch được?** | ✅ Có | ❌ Không |
| **Bao gồm** | Instance config + all volumes | Chỉ 1 volume |
| **Use case** | Deploy identical instances | Backup data |

**AMI creation process:**
```
1. Start instance → customize (install software, config)
2. Stop instance → đảm bảo data consistency
3. Create Image → AWS tạo AMI + EBS snapshots
4. Launch instances từ AMI → identical copies
```

**AMI có thể:**
- Copy **across regions**
- Share **across accounts** (với permissions)
- Make **public** (nếu muốn)

#### NÊN làm:
- **Stop instance trước khi tạo AMI** → Vì đảm bảo data integrity. Nếu tạo AMI khi có writes đang xảy ra, snapshot có thể không nhất quán.
- **Dùng Launch Template thay vì Launch Configuration** → Vì Launch Template hỗ trợ versioning, có thể update mà không tạo mới, tích hợp tốt hơn với ASG và SSM.

#### KHÔNG NÊN làm:
- **Hard-code credentials trong AMI** → Vì credentials sẽ được snapshot và có thể bị leak. NÊN: dùng IAM Role, User Data scripts, hoặc Secrets Manager để inject credentials khi launch.

---

## 5. Load Balancers

### Chọn Load Balancer Type

#### Giải thích cốt lõi

**3 loại Load Balancer chính:**

| Feature | ALB | NLB | GWLB |
|---------|-----|-----|------|
| **OSI Layer** | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) | Layer 3 (IP) |
| **Latency** | ~400ms | ~100ms | Varies |
| **Throughput** | Medium | Extreme (millions RPS) | Varies |
| **Routing** | Path, Host, Query | IP/Port | Packet |
| **Static IP** | ❌ | ✅ (1 per AZ) | ✅ |
| **SSL Termination** | ✅ Easy | ✅ Complex | ❌ |
| **Cost** | $/LCU | $/NLBCU | $/GB |

**ALB routing capabilities:**
```
Listener
    ├── Rule: /api/* → Target Group A
    ├── Rule: /web/* → Target Group B
    ├── Rule: *.example.com → Target Group C
    └── Rule: ?user=vip → Target Group D
```

#### NÊN làm:
- **Dùng ALB cho web applications (HTTP/HTTPS)** → Vì hỗ trợ path-based routing, host-based routing, giúp tách microservices. So với NLB, ALB dễ config SSL termination hơn, và có nhiều routing rules phong phú.
- **Dùng NLB cho TCP/UDP applications cần high performance** → Vì latency thấp hơn ALB (~100ms vs ~400ms), throughput cao hơn, hỗ trợ static IP cho whitelisting.

#### KHÔNG NÊN làm:
- **Dùng CLB (Classic Load Balancer)** → Vì legacy, ít features, đang deprecated. NÊN: dùng ALB hoặc NLB.
- **Dùng NLB cho HTTP applications** → Vì NLB không hỗ trợ path-based routing, SSL termination phức tạp hơn. ALB là lựa chọn tốt hơn cho HTTP/HTTPS.

---

### Security Groups cho Load Balancer

#### Giải thích cốt lõi

**Architecture đúng:**

```
Internet (0.0.0.0/0)
    ↓
Load Balancer (public IP)
    ├── Inbound: 0.0.0.0/0 (port 80, 443)
    └── Outbound: instances
    ↓
EC2 Instances (private IP)
    ├── Inbound: Chỉ từ LB SG
    └── Outbound: internet (qua NAT/S3 endpoint)
    ↓
Database
    └── Inbound: Chỉ từ App SG
```

**Tại sao EC2 không có public IP:**
- Security Layer: Only LB exposed to internet
- Cost: Public IPs có phí từ Feb 2024
- Security: SG của EC2 chỉ cần allow từ LB

#### NÊN làm:
- **EC2 Security Group chỉ allow traffic TỪ Load Balancer SG** → Vì khi LB scale, mọi instances đều được allow tự động. So với allow by IP, khi LB có nhiều instances, IP thay đổi → phải update rules.

#### KHÔNG NÊN làm:
- **Expose EC2 directly to internet (public IP + SG 0.0.0.0/0)** → Vì vi phạm security principle. Bất kỳ ai cũng có thể truy cập. NÊN: dùng Load Balancer, EC2 chỉ allow từ LB.

---

### Health Checks Configuration

#### Giải thích cốt lõi

**Health check mechanism:**

```
Load Balancer
    ↓ periodic health checks
Instance (port + path)
    ├── Response 200 → Healthy
    └── Response ≠ 200 → Unhealthy
        ↓ after N failures
    Terminate & Replace
```

**Timeout vs Interval:**
- **Timeout**: Thời gian chờ response tối đa
- **Interval**: Tần suất gửi health check
- **Threshold**: Số lần fail/success để change status

**Ví dụ config:**
```yaml
HealthCheckIntervalSeconds: 30   # Gửi mỗi 30s
HealthCheckTimeoutSeconds: 5    # Chờ response 5s
HealthyThresholdCount: 2        # 2 lần success → healthy
UnhealthyThresholdCount: 3     # 3 lần fail → unhealthy
```

#### NÊN làm:
- **Health check path riêng (/health hoặc /status)** → Vì endpoint này nên return 200 ngay lập tức mà không cần authenticate. So với check root `/`, endpoint riêng tránh triggering health check từ real user requests.

#### KHÔNG NÊN làm:
- **Health check trên application logic nặng** → Vì nếu application mất 5s để process request nhưng timeout chỉ 2s → false unhealthy. NÊN: health check endpoint đơn giản, return 200 ngay lập tức.

---

## 6. Auto Scaling Group (ASG)

### Scaling Policies

#### Giải thích cốt lõi

**3 loại Scaling Policies:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Target Tracking** | Tự động scale theo metric target | CPU 50%, Memory 70% |
| **Simple/Step** | Scale dựa trên CloudWatch alarm | Custom metrics |
| **Scheduled** | Scale theo schedule | Daily peaks |

**Target Tracking Example:**
```yaml
ScalingPolicy:
  Type: TargetTrackingConfiguration
  TargetValue: 50.0  # Target CPU 50%
  PredefinedMetricSpecification:
    PredefinedMetricType: ASGAverageCPUUtilization
```

**Cooldown Period:**
- Thời gian chờ SAU khi scale action
- Prevents rapid scale up/down cycles
- Default: 300 seconds (5 minutes)

#### NÊN làm:
- **Dùng Target Tracking cho auto scaling** → Vì đơn giản, AWS tự quản lý. Chỉ cần specify target value, AWS calculate số instances cần thiết. So với Simple Scaling, Target Tracking mượt mà hơn.

#### KHÔNG NÊN làm:
- **Set MaxSize quá thấp** → Vì khi đạt MaxSize, ASG không thể scale out dù CPU/RAM cao → application overloaded. NÊN: estimate peak load + buffer 50%.

---

### Launch Template vs Launch Configuration

#### Giải thích cốt lõi

**Launch Template (recommended):**
- ✅ Hỗ trợ **versioning**
- ✅ Tích hợp **SSM**, **Spot Fleet**
- ✅ **Mixed instances** (On-Demand + Spot)
- ✅ **Latest generation** của AWS

**Launch Configuration (deprecated):**
- ❌ Không có versioning
- ❌ Không hỗ trợ nhiều features mới
- ❌ AWS khuyến khích dùng Launch Template

#### NÊN làm:
- **Dùng Launch Template với versioning** → Vì có thể rollback nếu version mới có vấn đề. Update infrastructure bằng cách update version, không cần tạo mới.

---

## Tóm tắt Key Takeaways

### EBS
| | Recommendation | Why |
|--|----------------|-----|
| **Volume type** | gp3 default, io2 Block Express cho DB | gp3 rẻ và flexible hơn gp2 |
| **Size** | Start small, resize online | Upsize easy, downsize requires manual process |
| **Encryption** | Enable by default | Minimal performance impact, high security value |

### EFS vs EBS
| | Use Case | Why |
|--|---------|-----|
| **EFS** | Shared storage, multi-instance | Multiple instances mount via NFS |
| **EBS** | Single instance, database | Lower cost, better latency |

### Load Balancers
| | Use Case | Why |
|--|---------|-----|
| **ALB** | HTTP/HTTPS web apps | Layer 7 routing, SSL termination easy |
| **NLB** | TCP/UDP high performance | Ultra-low latency, static IP |

### ASG
| | Recommendation | Why |
|--|----------------|-----|
| **Policy** | Target Tracking | Simple, AWS-managed |
| **Template** | Launch Template | Versioning, modern features |
| **MaxSize** | Set high enough | Prevent blocking scale-out |
