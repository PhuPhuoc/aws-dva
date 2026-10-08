# Section 2 - EC2 Instance Storage | High Availability & Scalability: ELB & ASG

---

## Phần 1: EC2 Instance Storage (Trang 63-87)

### EBS Volume (Trang 64-68)

#### What's an EBS Volume? (Trang 64)

- **EBS (Elastic Block Store) Volume** là network drive có thể attach vào instances khi đang chạy
- Cho phép instances **persist data** (lưu trữ dữ liệu) ngay cả sau khi terminated
- Chỉ có thể mount vào **một instance tại một thời điểm** (trừ: io1/io2)
- **Locked vào một Availability Zone (AZ)**
- Tương tự như "network USB stick"

> *Sơ đồ: EC2 Instance kết nối với EBS Volume trong cùng AZ*

#### Root EBS Volume (Trang 65)

- Mặc định, khi tạo EC2 Instance sẽ tự động attach một EBS Volume gọi là **Root EBS Volume**
- Volume này chứa OS và system files
- Có thể thêm additional volumes nếu cần

> *Hình minh họa: Windows EC2 với thêm D: và E: drives*

#### EBS Multi-Attach (Trang 66-67)

- **Chỉ io1/io2 family** hỗ trợ Multi-Attach
- Có thể attach cùng một EBS volume đến **tối đa 16 EC2 Instances** trong cùng AZ

> *Sơ đồ: 16 EC2 Instances cùng attach vào 1 EBS Volume io2*

#### Delete on Termination (Trang 68)

| Loại Volume | Mặc định |
|-------------|-----------|
| **Root EBS Volume** | Bị **xóa** khi terminate (attribute enabled) |
| **Non-root EBS Volume** | **Không bị xóa** (attribute disabled) |

> *Use case: preserve root volume when instance is terminated*

---

### EBS Snapshots (Trang 69)

- Tạo **backup snapshot** của EBS volume tại một thời điểm
- Không bắt buộc phải detach volume để làm snapshot, nhưng **nên detach**
- Có thể **copy snapshots** across AZ hoặc Region

---

### AMI Overview (Trang 70-71)

#### AMI là gì?

- **AMI = Amazon Machine Image**
- AMI là customization của EC2 instance
- Thêm software, configuration, operating system, monitoring...
- **Boot nhanh hơn** vì software đã được pre-packaged
- AMI được build cho **specific region** (có thể copy across regions)

**3 loại AMI:**

| Loại | Mô tả |
|-------|--------|
| **Public AMI** | AWS cung cấp sẵn |
| **Own AMI** | Tự tạo và bảo trì |
| **AWS Marketplace AMI** | AMI người khác tạo (có thể bán) |

#### AMI Process (Trang 71)

```
1. Start EC2 instance → customize it
2. Stop the instance (đảm bảo data integrity)
3. Build an AMI → sẽ tạo EBS snapshots
4. Launch instances từ AMI
```

---

### EBS Snapshots vs AMI (Trang 72-73)

| | **EBS Snapshot** | **AMI** |
|---|-------------------|---------|
| **Là gì** | Copy của một volume cụ thể | Backup toàn bộ EC2 instance (bao gồm EBS snapshots) |
| **Launch EC2?** | ❌ Không thể | ✅ Có thể |
| **Use case** | Backup data cho một volume (D:, E:) | Backup entire instance, scale ra nhiều instances |

---

### EBS Snapshots Features (Trang 74)

| Feature | Mô tả |
|---------|--------|
| **EBS Snapshot Archive** | Chuyển snapshot sang "archive tier" giảm 75% chi phí. Restore mất 24-72 giờ |
| **Recycle Bin for EBS Snapshots** | Setup rules để retain deleted snapshots, recover sau khi accidental deletion. Retention: 1 ngày đến 1 năm |
| **Fast Snapshot Restore (FSR)** | Force full initialization để không có latency lần đầu sử dụng (tốn tiền $ $ $) |

---

### EBS Volume Types (Trang 75-80)

#### Tổng quan 6 loại Volume

| Loại | Characteristic |
|------|----------------|
| **gp2 / gp3** | SSD - General Purpose |
| **io1 / io2 Block Express** | SSD - Provisioned IOPS |
| **st1** | HDD - Throughput Optimized |
| **sc1** | HDD - Cold Storage |

> *EBS Volumes được đặc trưng bởi: Size | Throughput | IOPS*
> *Chỉ gp2/gp3 và io1/io2 Block Express có thể dùng làm boot volumes*

#### General Purpose SSD (Trang 76)

| Type | Mô tả |
|------|--------|
| **gp3** | Baseline 3,000 IOPS, 125 MiB/s. Có thể tăng IOPS đến 16,000 và throughput đến 1,000 MiB/s **independently** |
| **gp2** | Small gp2 volumes có thể burst IOPS đến 3,000. Size và IOPS **linked**. Max IOPS: 16,000. 3 IOPS per GB |

> *gp3 cho phép tăng IOPS và throughput độc lập với volume size*

#### Provisioned IOPS (PIOPS) SSD (Trang 77)

**io1 (4 GiB - 16 TiB):**
- Max PIOPS: 64,000 (Nitro instances), 32,000 (others)
- Có thể tăng PIOPS **independently** từ storage size

**io2 Block Express (4 GiB - 64 TiB):**
- Sub-millisecond latency
- Max PIOPS: **256,000** với IOPS:GiB ratio 1,000:1
- Hỗ trợ **EBS Multi-attach**

#### Hard Disk Drives (HDD) (Trang 78)

| Type | Use Case | Max Throughput | Max IOPS |
|------|----------|----------------|----------|
| **st1** (Throughput Optimized) | Big Data, Data Warehouses, Log Processing | 500 MiB/s | 500 |
| **sc1** (Cold HDD) | Infrequently accessed data, lowest cost | 250 MiB/s | 250 |

#### EBS Volume Types Summary (Trang 79-80)

**SSD Volumes:**

| Volume type | Durability | Use cases | Max IOPS | Boot volume |
|-------------|------------|-----------|----------|-------------|
| **gp3** | 99.8-99.9% | Transactional workloads, Virtual desktops | 16,000 | ✅ |
| **gp2** | 99.8-99.9% | Boot volumes, Dev/test | 16,000 | ✅ |
| **io2 Block Express** | **99.999%** | >64,000 IOPS, I/O-intensive DB | 256,000 | ✅ |
| **io1** | 99.8-99.9% | Sustained IOPS, >16,000 IOPS | 64,000 | ✅ |

**HDD Volumes:**

| Volume type | Durability | Use cases | Max Throughput | Boot volume |
|-------------|------------|-----------|---------------|-------------|
| **st1** | 99.8-99.9% | Big data, Data warehouses | 500 MiB/s | ❌ |
| **sc1** | 99.8-99.9% | Cold storage, lowest cost | 250 MiB/s | ❌ |

---

### EBS Multi-Attach (Trang 81)

- Attach cùng EBS volume đến **nhiều EC2 instances** trong cùng AZ
- **Use case:** Higher application availability trong clustered Linux applications (VD: Teradata)
- Applications phải **manage concurrent write operations**
- Hỗ trợ **tối đa 16 EC2 Instances**

---

### EBS Encryption (Trang 82-83)

Khi tạo encrypted EBS volume:
- Data at rest được encrypt trong volume
- Data in flight (giữa instance và volume) được encrypt
- Tất cả **snapshots đều encrypted**
- Tất cả **volumes từ snapshot đều encrypted**
- Encryption có **minimal impact on latency**
- EBS Encryption sử dụng **KMS (AES-256)**

#### Encrypt unencrypted EBS volume (Trang 83)

```
1. Tạo EBS snapshot của volume
2. Encrypt snapshot (dùng copy)
3. Tạo EBS volume mới từ snapshot (sẽ được encrypted)
4. Attach encrypted volume vào instance
```

---

### EC2 Instance Store (Trang 84)

- **High-performance hardware disk**
- **Better I/O performance** so với EBS
- **Rủi ro: data loss** khi EC2 stopped (ephemeral storage)
- **Use cases:** Buffer, cache, scratch data, temporary content
- Một số instance types không support instance store

> *Instance Store là temporary, EBS là persistent*

---

### Amazon EFS - Elastic File System (Trang 85-86)

**EFS là gì:**
- **Managed NFS** (network file system)
- Có thể mount trên **nhiều EC2**
- Hoạt động với EC2 instances trong **Multi-AZ**
- **Highly available, scalable**, nhưng **expensive (3x gp2)**
- **Use cases:** Content management, web serving, data sharing
- Sử dụng **NFSv4.1 protocol**
- **Chỉ tương thích với Linux** (không Windows)
- POSIX file system (~Linux)

#### EFS Storage Classes (Trang 86)

| Storage Class | Mô tả | First Byte Latency | Durability |
|---------------|--------|-------------------|------------|
| **EFS Standard** | Active data, fast sub-millisecond | Sub-millisecond | 99.999999999% |
| **EFS IA** | Inactive data, accessed few times/quarter | Tens of ms | 99.999999999% |
| **EFS Archive** | Inactive data, accessed few times/year | Tens of ms | 99.999999999% |

---

### EBS vs EFS vs Instance Store (Trang 87)

| Feature | **EBS** | **EFS** | **Instance Store** |
|---------|---------|---------|-------------------|
| **Connection** | Single instance | Multiple instances | Local to instance |
| **AZ** | Single AZ | Multi-AZ | Instance bound |
| **Persistence** | ✅ Persistent | ✅ Persistent | ❌ Ephemeral |
| **Cost** | $/GB | 3x gp2 | Included (but instance cost higher) |
| **Use case** | Database, Boot | Shared file storage | Cache, temp data |

---

## Phần 2: High Availability & Scalability (Trang 88-120)

### Scalability & High Availability (Trang 89)

**Scalability:**
- Application/system có thể handle greater loads bằng cách adapting
- Có 2 loại:
  - **Vertical Scalability** (Scale up/down)
  - **Horizontal Scalability** (Scale out/in) = elasticity

> *Scalability khác với High Availability nhưng liên quan*

---

### Vertical Scalability (Trang 90)

- Vertical scalability (= **scale up/down**) = tăng kích thước instance
- Phổ biến cho **non-distributed systems** như database
- RDS, ElastiCache là services có thể scale vertically

```
t2.micro (1 vCPU, 1 GB RAM)
        ↓
u-12tb1.metal (448 vCPU, 12.3 TB RAM)
```

---

### Horizontal Scalability (Trang 91)

- Horizontal scalability (= **scale out/in**) = tăng số lượng instances
- Cần **distributed systems**
- Phổ biến cho **web applications/modern applications**

```
Auto Scaling Group
├── EC2 Instance
├── EC2 Instance
├── EC2 Instance
└── EC2 Instance
```

---

### High Availability (Trang 92)

- High Availability thường đi cùng với horizontal scaling
- Chạy application trong **ít nhất 2 data centers (AZs)**
- **Goal:** Survive a data center loss

| Loại | Mô tả | Ví dụ |
|------|--------|-------|
| **Passive HA** | Passive redundancy | RDS Multi-AZ |
| **Active HA** | Active redundancy | Horizontal scaling với ASG |

---

### High Availability & Scalability cho EC2 (Trang 93-94)

| Approach | Mô tả |
|----------|--------|
| **Vertical Scaling** | Tăng instance size (t2.micro → u-12tb1.metal) |
| **Horizontal Scaling** | Tăng số instances (ASG + Load Balancer) |
| **High Availability** | Chạy instances across multi AZ (ASG multi AZ + Load Balancer multi AZ) |

---

### What is Load Balancing? (Trang 95)

- Load Balancers là servers **forward traffic** đến nhiều servers (EC2 instances) downstream

> *Sơ đồ: Client → Load Balancer → Multiple EC2 Instances*

---

### Why use a Load Balancer? (Trang 96)

| Lợi ích | Mô tả |
|----------|--------|
| **Spread load** | Phân phối load across multiple instances |
| **Single point of access** | Một DNS endpoint cho application |
| **Handle failures** | Tự động handle downstream instance failures |
| **Health checks** | Kiểm tra health của instances định kỳ |
| **SSL termination** | HTTPS cho websites |
| **Stickiness** | Duy trì session với cookies |
| **High availability** | Across zones |
| **Separate traffic** | Public vs private traffic |

---

### Elastic Load Balancer (Trang 97)

- **Managed load balancer** - AWS guarantee availability
- AWS lo mọi thứ: upgrades, maintenance, high availability
- **Integrated với:** EC2, EC2 Auto Scaling Groups, Amazon ECS, ACM, CloudWatch, Route 53, WAF, Global Accelerator

---

### Health Checks (Trang 98)

- Health Checks **quan trọng** cho Load Balancers
- LB biết instance có available để reply requests không
- Health check trên **port và route** (`/health` là common)
- Nếu response không phải **200 (OK)** → instance là **unhealthy**

---

### Load Balancer Security Groups (Trang 99)

```
Load Balancer SG:
  - Inbound: 0.0.0.0/0 (port 80, 443)

Application SG:
  - Inbound: Chỉ từ Load Balancer SG
```

> *Sơ đồ: Security Group flow từ Internet → LB → EC2*

---

### Types of Load Balancer on AWS (Trang 100)

| Feature | **Application LB** | **Network LB** | **Gateway LB** | **Classic LB** |
|---------|-------------------|----------------|---------------|---------------|
| **Layer** | Layer 7 | Layer 4 | Layer 3+4 | Layer 4/7 |
| **Target type** | IP, Instance, Lambda | IP, Instance, ALB | IP | IP, Instance |
| **Protocol** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | GENEVE | TCP, SSL, HTTP |

---

### Application Load Balancer (ALB) (Trang 101-106)

**Đặc điểm:**
- **Layer 7 (HTTP/HTTPS)**
- Load balance đến **multiple HTTP applications** across machines (target groups)
- Load balance đến **multiple applications trên cùng machine** (containers)
- Hỗ trợ **HTTP/2 và WebSocket**
- Hỗ trợ **redirects** (HTTP → HTTPS)

**Routing options:**
- Path in URL (`/users`, `/posts`)
- Hostname (`one.example.com`, `other.example.com`)
- Query String, Headers (`?id=123&order=false`)

**Target Groups:**
- EC2 instances (có thể managed by ASG)
- ECS tasks
- Lambda functions
- IP Addresses (phải là private IPs)

**Good to know:**
- Fixed hostname (`XXX.region.elb.amazonaws.com`)
- Client IP thực trong header: **X-Forwarded-For**

---

### Network Load Balancer (NLB) (Trang 107-109)

**Đặc điểm:**
- **Layer 4 (TCP/UDP)**
- Handle **millions requests per second**
- **Less latency** ~100 ms (so với 400 ms của ALB)
- Mỗi AZ có **1 static IP**, hỗ trợ **Elastic IP** (whitelisting)
- **Không included trong AWS free tier**

**Target Groups:**
- EC2 instances
- IP Addresses (private IPs)
- Application Load Balancer

---

### Gateway Load Balancer (Trang 110-111)

- Deploy, scale, manage **3rd party network virtual appliances**
- **Use cases:** Firewalls, IDS/IPS, Deep Packet Inspection, payload manipulation
- **Layer 3** (IP Packets)
- Kết hợp: Transparent Network Gateway + Load Balancer
- Uses **GENEVE protocol** on port 6081

---

### Sticky Sessions (Trang 112)

- **Stickiness** = same client luôn được redirect đến same instance
- Hoạt động với CLB, ALB, NLB
- Sử dụng **cookie** với expiration date
- **Use case:** User không mất session data
- **Warning:** Có thể gây load imbalance

---

### Cross-Zone Load Balancing (Trang 113-114)

| Loại LB | Default | Inter-AZ Data |
|---------|---------|----------------|
| **Application LB** | ✅ Enabled (có thể disable) | Miễn phí |
| **Network LB** | ❌ Disabled (có thể enable) | **Phải trả tiền** |
| **Gateway LB** | ❌ Disabled (có thể enable) | **Phải trả tiền** |

---

### SSL/TLS Basics (Trang 115)

- **SSL Certificate** encrypts traffic giữa clients và load balancer
- **SSL** = Secure Sockets Layer
- **TLS** = Transport Layer Security (version mới hơn)
- Public SSL certificates được issue bởi **Certificate Authorities (CA)**
- Certificates có expiration date, phải **renew**

---

### Load Balancer - SSL Certificates (Trang 116)

- LB sử dụng **X.509 certificate**
- Quản lý certificates bằng **ACM (AWS Certificate Manager)**
- HTTPS listener yêu cầu:
  - Default certificate
  - Optional list of certs cho multiple domains
- Clients dùng **SNI (Server Name Indication)** để specify hostname

---

### SNI - Server Name Indication (Trang 117)

- SNI giải quyết vấn đề load **multiple SSL certificates** lên một web server
- **Hoạt động với:** ALB, NLB (newer), CloudFront
- **Không hoạt động với:** CLB (older)

---

### Connection Draining (Trang 118)

| Tên gọi | Load Balancer |
|---------|--------------|
| **Connection Draining** | CLB |
| **Deregistration Delay** | ALB & NLB |

- Time để complete "in-flight requests" trong khi instance deregistering/unhealthy
- **Stops sending new requests** đến instance đang deregistering
- Range: **1 - 3600 seconds** (default: 300)
- Có thể disable (set = 0)
- Nên set low value nếu requests ngắn

---

### Auto Scaling Group (ASG) (Trang 119-120)

**Trong thực tế:** Load trên websites/application thay đổi

**Mục tiêu của ASG:**
- **Scale out:** Thêm EC2 instances để match increased load
- **Scale in:** Xóa EC2 instances để match decreased load
- Đảm bảo **minimum và maximum** instances đang chạy
- **Auto register** new instances đến load balancer
- **Re-create** EC2 instance nếu instance trước bị terminated (ví dụ: unhealthy)

> *ASG là free (chỉ trả tiền cho EC2 instances bên dưới)*

---

## Tổng kết Section 2

### EC2 Storage

| Storage | Đặc điểm |
|---------|-----------|
| **EBS** | Network drive, persistent, single AZ, single instance (trừ io1/io2) |
| **EFS** | Shared file system, multi-AZ, Linux only, 3x gp2 cost |
| **Instance Store** | Local disk, high performance, ephemeral (mất data khi stop) |

### EBS Volume Types

| Type | Use Case | IOPS | Cost |
|------|----------|------|------|
| **gp3** | General purpose, boot | 16,000 | $ |
| **io2 Block Express** | Mission-critical DB | 256,000 | $$$ |
| **st1** | Big data, log processing | 500 | $$ |
| **sc1** | Cold storage | 250 | $ |

### Load Balancers

| Type | Layer | Use Case |
|------|-------|----------|
| **ALB** | L7 | HTTP/HTTPS, microservices |
| **NLB** | L4 | TCP/UDP, high performance |
| **GWLB** | L3 | 3rd party virtual appliances |

### ASG

- Scale out/in tự động
- Kết hợp với Load Balancer
- Minimum, Maximum, Desired capacity
- Health checks và replacement tự động
