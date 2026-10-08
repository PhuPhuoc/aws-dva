# Câu hỏi ôn tập DVA-C02 - Section 2

> *Các câu hỏi mẫu từ đề thi AWS Developer Associate. Câu hỏi bằng tiếng Anh, giải thích bằng tiếng Việt.*

---

## EBS Volume Types

### Question:

A company is running a **high-performance database** on an EC2 instance that requires:
- **Sustained IOPS**: 50,000
- **Low latency**: sub-millisecond
- **99.999% durability**

Which **EBS volume type** should they use?

**A)** gp3
**B)** gp2
**C)** io2 Block Express
**D)** st1

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C (io2 Block Express):**

io2 Block Express là volume type duy nhất đáp ứng được cả 3 yêu cầu:

| Yêu cầu | io2 Block Express | gp3 | gp2 |
|----------|------------------|-----|-----|
| 50,000 IOPS | ✅ 256,000 max | ❌ Max 16,000 | ❌ Max 16,000 |
| Sub-millisecond latency | ✅ Có | ❌ Không | ❌ Không |
| 99.999% durability | ✅ Có | ❌ 99.8-99.9% | ❌ 99.8-99.9% |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (gp3)** | Max IOPS chỉ 16,000 → không đủ 50,000. Durability chỉ 99.8-99.9% |
| **B (gp2)** | IOPS giới hạn bởi size (3 IOPS/GB) → cần ~17TB để đạt 50,000 IOPS. Đang deprecated |
| **D (st1)** | HDD-based, chỉ 500 IOPS max → không phù hợp cho database |

### 📝 Phân tích từ vựng:

- **Sustained IOPS**: IOPS liên tục, không phải burst
- **Sub-millisecond latency**: độ trễ dưới 1 mili giây
- **Durability**: độ bền, khả năng không mất dữ liệu
- **Provisioned IOPS**: IOPS được cấu hình riêng, không phụ thuộc size

### 🔄 Thay đổi context:

- **Nếu chỉ cần 10,000 IOPS** → gp3 là lựa chọn tiết kiệm hơn
- **Nếu cần tiết kiệm chi phí** → gp3 với provisioned IOPS 16,000
- **Nếu là batch processing** → st1 có thể phù hợp với throughput cao

---

## EBS Encryption

### Question:

A developer needs to **encrypt an existing unencrypted EBS volume** without data loss. What are the correct steps?

**A)** Enable encryption on the volume directly → Volume is encrypted in place
**B)** Create snapshot → Copy snapshot with encryption enabled → Create volume from encrypted snapshot → Attach to instance
**C)** Detach volume → Enable encryption → Re-attach volume
**D)** Create AMI from instance → Enable encryption in AMI settings → Launch new instance

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

EBS không hỗ trợ encrypt in-place (mã hóa tại chỗ). Phải thông qua snapshot:

**Các bước đúng:**
1. **Create snapshot** của unencrypted volume
2. **Copy snapshot** với encryption enabled → snapshot mới sẽ được encrypted
3. **Create volume** từ encrypted snapshot
4. **Attach** encrypted volume vào instance

**Phương pháp B đúng vì:**
- Snapshot gốc không bị mất
- Volume mới được tạo ra là encrypted
- Có thể switch qua encrypted version mà không mất data

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | EBS không support enable encryption trực tiếp trên existing volume |
| **C** | Không thể enable encryption khi detach/attach |
| **D** | Tạo AMI sẽ tạo snapshot, nhưng phải copy với encryption enabled mới được |

### 📝 Phân tích từ vựng:

- **Encrypt in-place**: mã hóa tại chỗ, không cần tạo bản sao
- **Copy with encryption**: tạo bản sao đã được mã hóa
- **Snapshot**: bản chụp trạng thái volume tại một thời điểm

### 🔄 Thay đổi context:

- **Nếu muốn encrypt root volume** → Phải stop instance trước, detach root volume, làm theo steps trên
- **Nếu dùng newer instances** → Encryption có thể được enable mặc định khi tạo volume

---

## EBS vs Instance Store

### Question:

A company is building a **Redis cache cluster** that needs:
- **Ultra-low latency** storage
- **Data can be lost** if instance stops (acceptable)
- **High IOPS** for in-memory operations

Which storage option is **most appropriate**?

**A)** EBS gp3
**B)** EBS io2 Block Express
**C)** Instance Store
**D)** EFS

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C (Instance Store):**

Instance Store cung cấp:
- **Highest performance**: local NVMe SSD attached trực tiếp
- **Lowest latency**: không có network overhead như EBS
- **Không có cost riêng**: included trong instance price

**Đặc biệt phù hợp cho Redis/Cache vì:**
- Cache data có thể rebuild được → data loss không critical
- Latency cực thấp là ưu tiên hàng đầu
- Performance vượt trội so với EBS

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (gp3)** | EBS có network latency → cao hơn Instance Store |
| **B (io2 Block Express)** | Đắt hơn, latency cao hơn, không cần persistence |
| **D (EFS)** | Network filesystem → latency rất cao, không phù hợp cho cache |

### 📝 Phân tích từ vựng:

- **Instance Store**: storage được attach trực tiếp vào instance, ephemeral (mất khi stop)
- **Ephemeral storage**: storage tạm thời, không persistence
- **Local NVMe**: non-volatile memory express, giao diện storage rất nhanh

### 🔄 Thay đổi context:

- **Nếu data cần persistence** → Dùng EBS (gp3 hoặc io2)
- **Nếu cần shared storage giữa nhiều instances** → EFS
- **Nếu cần cross-AZ high availability** → EBS không đủ → cần EFS hoặc distributed architecture

---

## AMI vs EBS Snapshot

### Question:

A developer wants to **backup an entire EC2 instance** so they can **launch new instances** from it. Should they create an **EBS snapshot** or an **AMI**?

**A)** EBS Snapshot, because it includes all volumes
**B)** AMI, because it includes the full instance configuration and can launch new instances
**C)** Either would work, they are the same
**D)** Neither, EC2 instances cannot be backed up

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

**AMI bao gồm:**
- Toàn bộ EC2 instance configuration
- Tất cả EBS volumes (root + data volumes)
- EBS snapshots của các volumes
- Launch permissions
- Metadata (kernel ID, RAM disk ID, etc.)

**EBS Snapshot chỉ bao gồm:**
- Một volume cụ thể
- Không thể launch instance trực tiếp từ snapshot

| Feature | AMI | EBS Snapshot |
|---------|-----|-------------|
| Launch new instances | ✅ | ❌ |
| Include all volumes | ✅ | ❌ |
| Include instance config | ✅ | ❌ |
| Use for backup data | ✅ | ✅ |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | EBS Snapshot không bao gồm all volumes - chỉ một volume cụ thể |
| **C** | Chúng khác nhau - AMI dùng để launch, Snapshot dùng để backup data |
| **D** | Hoàn toàn sai - EC2 có thể backup bằng AMI |

### 📝 Phân tích từ vựng:

- **AMI (Amazon Machine Image)**: template để launch instance, bao gồm OS, apps, configs
- **Launch from AMI**: tạo instance mới từ image đã có
- **Instance configuration**: cấu hình của instance (instance type, security groups, etc.)

### 🔄 Thay đổi context:

- **Nếu chỉ muốn backup data của D: drive** → EBS Snapshot là đủ
- **Nếu muốn replicate entire system** → AMI bắt buộc
- **Nếu cần cross-region backup** → Cả 2 đều có thể copy

---

## Load Balancer Selection

### Question:

A company needs to distribute **TCP traffic** for a **gaming application** that requires:
- **Millions of requests per second**
- **Ultra-low latency** (~100ms)
- **Static IP address** for whitelisting by partners

Which Load Balancer type is **most appropriate**?

**A)** Application Load Balancer (ALB)
**B)** Network Load Balancer (NLB)
**C)** Gateway Load Balancer (GWLB)
**D)** Classic Load Balancer (CLB)

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (Network Load Balancer):**

NLB được thiết kế cho:
- **Layer 4 (TCP/UDP)** → không parse HTTP headers
- **Millions of requests/second** → phù hợp cho high-throughput
- **~100ms latency** → thấp hơn ALB (~400ms)
- **Static IP per AZ** → hỗ trợ Elastic IP cho whitelisting
- **Extreme performance** → không có overhead của Layer 7

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (ALB)** | Layer 7 (HTTP/HTTPS) → overhead cao hơn, không phù hợp cho TCP thuần túy |
| **C (GWLB)** | Dùng cho 3rd party virtual appliances (firewalls), không phải cho applications |
| **D (CLB)** | Legacy, không recommended cho architectures mới, ít features |

### 📝 Phân tích từ vựng:

- **Layer 4 (Transport Layer)**: hoạt động ở tầng transport, TCP/UDP
- **Layer 7 (Application Layer)**: hoạt động ở tầng application, HTTP/HTTPS
- **Whitelisting**: cho phép truy cập từ danh sách IP cố định
- **Elastic IP**: static public IP có thể reassign

### 🔄 Thay đổi context:

- **Nếu cần HTTP routing, path-based routing** → ALB
- **Nếu cần SSL termination** → ALB (NLB cũng có nhưng phức tạp hơn)
- **Nếu cần integrate với WAF** → ALB

---

## ALB Routing

### Question:

A company runs a **microservices architecture** with separate services for:
- `/api/users` - User service
- `/api/products` - Product service
- `/api/orders` - Order service

How should they configure the **Application Load Balancer** to route traffic to the correct service?

**A)** Create one target group for all services, use Lambda to route
**B)** Create separate target groups for each service, use **path-based routing rules**
**C)** Create separate Load Balancers for each service
**D)** Use Network Load Balancer with port-based routing

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

ALB hỗ trợ **path-based routing** - một trong những features mạnh nhất của nó:

**Cấu hình đúng:**
```json
{
  "Rules": [
    {
      "PathPattern": "/api/users*",
      "TargetGroup": "tg-users"
    },
    {
      "PathPattern": "/api/products*",
      "TargetGroup": "tg-products"
    },
    {
      "PathPattern": "/api/orders*",
      "TargetGroup": "tg-orders"
    }
  ]
}
```

**Lợi ích:**
- Một ALB duy nhất quản lý tất cả services
- Mỗi target group có health check riêng
- Dễ dàng scale từng service độc lập

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Unnecessary complexity - Lambda không cần thiết khi ALB có sẵn path routing |
| **C** | Lãng phí resources và chi phí - nhiều LBs không cần thiết |
| **D** | NLB không hỗ trợ path-based routing |

### 📝 Phân tích từ vựng:

- **Path-based routing**: định tuyến dựa trên URL path
- **Target group**: nhóm instances để nhận traffic
- **Listener rule**: quy tắt để route traffic đến target groups

### 🔄 Thay đổi context:

- **Nếu cần routing dựa trên hostname** → ALB cũng hỗ trợ host-based routing
- **Nếu cần routing dựa trên query string** → ALB hỗ trợ query string routing
- **Nếu dùng containers** → ALB có tính năng dynamic port mapping với ECS

---

## Health Check Configuration

### Question:

An ALB target group's health check is configured with:
- **Health check path**: `/`
- **Healthy threshold**: 1
- **Unhealthy threshold**: 1
- **Timeout**: 2 seconds
- **Interval**: 5 seconds

The application takes **8 seconds** to respond. Why are instances being marked as **unhealthy**?

**A)** Health check interval is too short
**B)** **Timeout** is shorter than response time
**C)** Health check path should be `/health`
**D)** Only one threshold should be used

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

**Vấn đề:** Timeout = 2 giây, nhưng application mất 8 giây để respond.

**Khi nào instance bị mark unhealthy:**
1. LB gửi request đến health check path
2. Chờ response trong **timeout period** (2 giây)
3. Nếu không nhận được response → lần này là **failed**
4. Sau **unhealthy threshold** (1) failures → instance = **unhealthy**

**Với cấu hình hiện tại:**
- Mỗi health check → timeout sau 2s → failed
- Unhealthy threshold = 1 → immediately unhealthy

**Để fix:**
- Tăng **Timeout** lên ít nhất 10 giây
- Hoặc giảm **Interval** và tăng thresholds

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Interval ngắn không phải vấn đề - timeout mới là vấn đề |
| **C** | `/` hay `/health` đều được - không ảnh hưởng đến việc timeout |
| **D** | Cấu hình threshold không liên quan đến timeout |

### 📝 Phân tích từ vựng:

- **Health check**: kiểm tra trạng thái của instance
- **Timeout**: thời gian tối đa chờ response
- **Healthy threshold**: số lần thành công để mark healthy
- **Unhealthy threshold**: số lần thất bại để mark unhealthy

### 🔄 Thay đổi context:

- **Nếu application chạy nặng** → Tăng timeout và interval
- **Nếu cần faster failover** → Giảm unhealthy threshold nhưng vẫn phải đủ timeout
- **Nếu health check quá aggressive** → Tăng thresholds để tránh false positives

---

## Auto Scaling Group Scaling Policy

### Question:

An ASG has a **target tracking scaling policy** configured with:
- **Metric type**: ASGAverageCPUUtilization
- **Target value**: 50%

After deployment, the application experiences **high latency**. Investigation shows CPU is at **80%** but ASG is not scaling out. What is the most likely cause?

**A)** Target value is too low
**B)** **Cooldown period** has not elapsed
**C)** Maximum size is set too low
**D)** Health check is failing

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C:**

**Lý do:** ASG đã reach **maximum capacity** → không thể scale out thêm dù CPU cao.

**Kiểm tra ASG limits:**
```bash
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name <name>
```

Output sẽ show:
- MinSize
- MaxSize
- DesiredCapacity
- Instances (current count)

**Nếu:**
- Current instances = MaxSize
- CPU = 80%
→ ASG muốn scale out nhưng **không thể** vì đã đạt MaxSize

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Target 50% là hợp lý - thấp hơn sẽ scale out nhiều hơn, không phải nguyên nhân |
| **B** | Cooldown period chỉ delay scale actions - không ngăn cản scale khi đã đạt threshold |
| **D** | Health check failing sẽ terminate instances, không ngăn scale out |

### 📝 Phân tích từ vựng:

- **Target tracking policy**: tự động scale dựa trên metric và target value
- **Cooldown period**: thời gian chờ sau khi scale trước khi scale tiếp
- **Maximum size**: giới hạn số lượng instances tối đa

### 🔄 Thay đổi context:

- **Nếu MaxSize đã đúng** → Kiểm tra launch template có đúng không
- **Nếu cần scale more** → Tăng MaxSize
- **Nếu latency do application** → Có thể cần optimize code, không phải scale

---

## Cross-Zone Load Balancing

### Question:

A company has:
- **AZ-1**: 5 instances
- **AZ-2**: 2 instances
- **AZ-3**: 1 instance

**Cross-zone load balancing** is disabled on the NLB. How will traffic be distributed?

**A)** Evenly across all 8 instances
**B)** **Only within each AZ** - requests in each AZ distributed to instances in that AZ only
**C)** Only to instances in AZ-1 (has most instances)
**D)** Randomly to any instance

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

**Khi Cross-Zone Load Balancing DISABLED:**

Traffic được distribute **trong mỗi AZ riêng biệt**:

| AZ | Instances | Traffic |
|----|----------|---------|
| AZ-1 | 5 | 100% của AZ-1's requests → 5 instances |
| AZ-2 | 2 | 100% của AZ-2's requests → 2 instances |
| AZ-3 | 1 | 100% của AZ-3's requests → 1 instance |

**Kết quả:**
- AZ-1 instances: nhận ít traffic hơn mỗi instance
- AZ-3 instance: overload vì phải handle tất cả traffic của AZ-3

**Khi Cross-Zone Load Balancing ENABLED:**

8 instances đều nhận equal distribution (~12.5% mỗi instance)

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Đúng nếu Cross-Zone ENABLED, nhưng question nói DISABLED |
| **C** | Traffic không đi tất cả vào AZ có nhiều instances nhất |
| **D** | Load balancer distribute có rule, không phải random |

### 📝 Phân tích từ vựng:

- **Cross-zone load balancing**: phân phối traffic đều giữa các AZ
- **Intra-zone distribution**: phân phối traffic chỉ trong AZ của nó
- **Inter-AZ data transfer**: data transfer giữa các AZ (có phí với NLB)

### 🔄 Thay đổi context:

- **ALB**: Cross-zone enabled by default (free)
- **NLB**: Cross-zone disabled by default (có phí inter-AZ)
- **Nếu muốn tiết kiệm** → Keep disabled với NLB nhưng phải đảm bảo instances đều giữa các AZ

---

## EFS vs EBS

### Question:

A company needs to store **shared files** that will be accessed by **50 EC2 instances** across **multiple Availability Zones**. The data is **content management data** that needs to be accessed frequently. Which storage solution is **most appropriate**?

**A)** EBS volumes attached to each instance
**B)** **EFS (Elastic File System)**
**C)** Instance Store on each instance
**D)** S3 bucket with Mountpoint

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (EFS):**

**Use case phù hợp với EFS:**
- **Multiple instances** cần access cùng data
- **Multi-AZ** deployment
- **Frequent access** (không phải cold storage)
- **POSIX-compliant** filesystem (Linux)

**EFS characteristics:**
- Managed NFS (Network File System)
- Scale automatically
- Multi-AZ by default
- Pay per use

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (EBS)** | EBS chỉ attach được 1 instance (trừ io1/io2 Multi-Attach - phức tạp và đắt) |
| **C (Instance Store)** | Không shared được giữa các instances, ephemeral (mất data) |
| **D (S3 with Mountpoint)** | S3 là object storage, không phải filesystem. Mountpoint for S3 không phù hợp cho frequent writes |

### 📝 Phân tích từ vựng:

- **Shared file storage**: lưu trữ file có thể access từ nhiều instances
- **NFS (Network File System)**: giao thức để share filesystem qua network
- **POSIX-compliant**: tuân theo chuẩn file system của Unix/Linux

### 🔄 Thay đổi context:

- **Nếu chỉ 1 instance** → EBS đủ và rẻ hơn
- **Nếu cần Windows compatibility** → EFS không hỗ trợ → cần FSx for Windows
- **Nếu cần ultra-cheap storage** → S3 với lifecycle policies

---

## Tóm tắt Patterns trong DVA Exam - Section 2

### EBS Questions:
- **io2 Block Express** cho database cần 99.999% durability và high IOPS
- **gp3** là lựa chọn default cho hầu hết use cases
- Encryption phải qua **snapshot → copy → new volume**
- **Instance Store** cho cache, ephemeral data

### AMI vs Snapshot:
- **AMI** = entire instance (launch được)
- **Snapshot** = single volume (backup data)

### Load Balancers:
- **ALB** = Layer 7, HTTP/HTTPS, path-based routing
- **NLB** = Layer 4, TCP/UDP, high performance, static IP
- **GWLB** = Layer 3, 3rd party appliances

### ASG:
- **Target Tracking** cho auto scaling
- **MaxSize** limit có thể ngăn scale out
- **Health checks** từ Load Balancer

### EFS:
- **Shared storage** giữa multiple instances
- **Multi-AZ** by default
- Không hỗ trợ Windows
