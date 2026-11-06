# Câu hỏi ôn tập DVA-C02 - Section 2

> *Các câu hỏi mẫu từ đề thi AWS Developer Associate. Câu hỏi bằng tiếng Anh, giải thích bằng tiếng Việt.*

---

## Các loại EBS Volume

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
| 50,000 IOPS | ✅ Tối đa 256,000 | ✅ Tối đa 80,000 (Regional) | ❌ Tối đa 16,000 |
| Sub-millisecond latency | ✅ Có | ❌ Không | ❌ Không |
| 99.999% durability | ✅ Có | ❌ 99.8-99.9% | ❌ 99.8-99.9% |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (gp3)** | gp3 Regional có thể đạt đến 80,000 IOPS, nên riêng yêu cầu IOPS không loại volume này; nhưng durability 99.8-99.9% và latency không đáp ứng các yêu cầu còn lại. |
| **B (gp2)** | IOPS tăng theo dung lượng nhưng tối đa 16,000; gp2 không đạt 50,000 IOPS và chưa bị ngừng hỗ trợ. |
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

## Mã hóa EBS

### Question:

A developer needs to **encrypt an existing unencrypted EBS volume** without data loss. What are the correct steps?

**A)** Enable encryption on the volume directly → Volume is encrypted in place
**B)** Create snapshot → Copy snapshot with encryption enabled → Create volume from encrypted snapshot → Attach to instance
**C)** Detach volume → Enable encryption → Re-attach volume
**D)** Create AMI from instance → Enable encryption in AMI settings → Launch new instance

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

EBS không hỗ trợ mã hóa tại chỗ. Cần thực hiện qua snapshot:

**Các bước đúng:**
1. **Tạo snapshot** của volume chưa mã hóa.
2. **Sao chép snapshot** và bật mã hóa → snapshot mới sẽ được mã hóa.
3. **Tạo volume** từ snapshot đã mã hóa.
4. **Gắn** volume đã mã hóa vào instance.

**Vì sao đáp án B đúng:**
- Snapshot gốc vẫn được giữ nguyên.
- Volume mới được tạo sẽ được mã hóa.
- Có thể chuyển sang bản đã mã hóa mà không mất dữ liệu.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | EBS không hỗ trợ bật mã hóa trực tiếp trên volume hiện có. |
| **C** | Không thể bật mã hóa bằng cách tháo rồi gắn lại volume. |
| **D** | Tạo AMI sẽ tạo snapshot, nhưng cần sao chép snapshot và bật mã hóa. |

### 📝 Phân tích từ vựng:

- **Encrypt in-place**: mã hóa tại chỗ, không cần tạo bản sao
- **Copy with encryption**: tạo bản sao đã được mã hóa
- **Snapshot**: bản chụp trạng thái volume tại một thời điểm

### 🔄 Thay đổi context:

- **Nếu muốn mã hóa root volume** → Trước tiên dừng instance, tháo root volume rồi thực hiện theo các bước trên.
- **Encryption by default** là setting của account theo từng Region, không phụ thuộc Instance Type. Khi bật trong Region, volume mới và snapshot copy mới ở Region đó được mã hóa; tài nguyên hiện có không tự đổi.

---

## EBS và Instance Store

### Question:

A company is building a **Redis cache cluster** on an EC2 instance type that provides **local NVMe Instance Store**. The workload needs:
- **Ultra-low latency** storage
- **Data can be lost** if the instance stops (acceptable)
- **High IOPS** for cache operations

Which storage option is **most appropriate**?

**A)** EBS gp3
**B)** EBS io2 Block Express
**C)** Instance Store
**D)** EFS

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C (Instance Store):**

Instance Store trên Instance Type đã nêu cung cấp:
- NVMe SSD cục bộ, không đi qua mạng EBS.
- Hiệu năng cao cho dữ liệu cache có thể tạo lại.
- Không tính phí EBS volume riêng (EC2 instance vẫn có chi phí).

**Đặc biệt phù hợp với Redis/Cache vì:**
- Dữ liệu cache có thể tạo lại → mất dữ liệu không nghiêm trọng.
- Độ trễ thấp là ưu tiên hàng đầu.
- Hiệu năng cao hơn EBS.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (gp3)** | EBS có độ trễ mạng nên cao hơn Instance Store. |
| **B (io2 Block Express)** | Đắt hơn, độ trễ cao hơn và không cần tính bền vững dữ liệu trong tình huống này. |
| **D (EFS)** | Hệ thống tệp qua mạng có độ trễ cao, không phù hợp cho cache. |

### 📝 Phân tích từ vựng:

- **Instance Store**: lưu trữ gắn trực tiếp vào instance, có tính tạm thời (mất khi dừng).
- **Ephemeral storage**: lưu trữ tạm thời, không bền vững.
- **Local NVMe**: NVMe cục bộ, giao diện lưu trữ tốc độ cao.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu dữ liệu cần được lưu bền vững** → Dùng EBS (gp3 hoặc io2).
- **Nếu cần lưu trữ chia sẻ giữa nhiều instances** → Dùng EFS.
- **Nếu cần tính sẵn sàng cao qua nhiều AZ** → EBS là chưa đủ; cần EFS hoặc kiến trúc phân tán.

---

## AMI và EBS Snapshot

### Question:

A developer wants to **backup an entire EC2 instance** so they can **launch new instances** from it. Should they create an **EBS snapshot** or an **AMI**?

**A)** EBS Snapshot, because it includes all volumes
**B)** AMI, because it includes the full instance configuration and can launch new instances
**C)** Either would work, they are the same
**D)** Neither, EC2 instances cannot be backed up

### ✅ Answer: **B**

### 📖 Giải thích:

**AMI bao gồm:**
- Toàn bộ cấu hình EC2 instance.
- Tất cả EBS volumes (root + data volumes).
- EBS snapshots của các volumes.
- Quyền khởi chạy.
- Metadata (kernel ID, RAM disk ID, v.v.).

**EBS Snapshot chỉ bao gồm:**
- Một volume cụ thể.
- Không thể khởi chạy instance trực tiếp từ snapshot.

| Đặc điểm | AMI | EBS Snapshot |
|---------|-----|-------------|
| Khởi chạy instances mới | ✅ | ❌ |
| Bao gồm mọi volumes | ✅ | ❌ |
| Bao gồm cấu hình instance | ✅ | ❌ |
| Dùng để sao lưu dữ liệu | ✅ | ✅ |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | EBS Snapshot không gồm mọi volume, chỉ có một volume cụ thể. |
| **C** | Hai loại này khác nhau: AMI dùng để khởi chạy, Snapshot dùng để sao lưu dữ liệu. |
| **D** | Sai - EC2 có thể được sao lưu bằng AMI. |

### 📝 Phân tích từ vựng:

- **AMI (Amazon Machine Image)**: template để khởi chạy instance, gồm OS, ứng dụng và cấu hình.
- **Khởi chạy từ AMI**: tạo instance mới từ image đã có.
- **Cấu hình instance**: các thiết lập của instance (Instance Type, Security Groups, v.v.).

### 🔄 Thay đổi ngữ cảnh:

- **Nếu chỉ muốn sao lưu dữ liệu của ổ D:** → EBS Snapshot là đủ.
- **Nếu muốn sao chép toàn bộ hệ thống** → Cần AMI.
- **Nếu cần sao lưu qua Region** → Có thể sao chép cả hai.

---

## Chọn Load Balancer

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
- **Layer 4 (TCP/UDP)** → không phân tích HTTP headers.
- **Hàng triệu request mỗi giây** → phù hợp với thông lượng cao.
- **Độ trễ khoảng 100ms** → thấp hơn ALB (~400ms).
- **Static IP cho mỗi AZ** → hỗ trợ Elastic IP để allowlist IP.
- **Hiệu năng rất cao** → không có overhead của Layer 7.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (ALB)** | Hoạt động ở Layer 7 (HTTP/HTTPS), có overhead cao hơn và không phù hợp với TCP thuần túy. |
| **C (GWLB)** | Dùng cho network virtual appliances của bên thứ ba (firewalls), không dành cho ứng dụng thông thường. |
| **D (CLB)** | Là loại cũ, AWS không khuyến nghị cho kiến trúc mới và có ít tính năng hơn. |

### 📝 Phân tích từ vựng:

- **Layer 4 (Transport Layer)**: tầng truyền tải, hoạt động với TCP/UDP.
- **Layer 7 (Application Layer)**: tầng ứng dụng, hoạt động với HTTP/HTTPS.
- **Whitelisting**: chỉ cho phép truy cập từ danh sách IP xác định.
- **Elastic IP**: public IP tĩnh có thể gán lại.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu cần định tuyến HTTP hoặc theo path** → Dùng ALB.
- **Nếu cần SSL termination** → Dùng ALB (NLB cũng hỗ trợ nhưng cấu hình phức tạp hơn).
- **Nếu cần tích hợp với WAF** → Dùng ALB.

---

## Định tuyến ALB

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

ALB hỗ trợ **định tuyến theo path**, một trong những tính năng mạnh nhất của dịch vụ:

**Cấu hình phù hợp:**
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
- Một ALB duy nhất quản lý tất cả services.
- Mỗi target group có health check riêng.
- Dễ dàng mở rộng từng service độc lập.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Quá phức tạp - không cần Lambda khi ALB đã có path routing. |
| **C** | Lãng phí tài nguyên và chi phí vì không cần nhiều Load Balancers. |
| **D** | NLB không hỗ trợ định tuyến theo path. |

### 📝 Phân tích từ vựng:

- **Path-based routing**: định tuyến dựa trên URL path.
- **Target group**: nhóm instances nhận lưu lượng.
- **Listener rule**: quy tắc định tuyến request đến target groups.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu cần định tuyến theo hostname** → ALB cũng hỗ trợ host-based routing.
- **Nếu cần định tuyến theo query string** → ALB hỗ trợ query string routing.
- **Nếu dùng containers** → ALB hỗ trợ dynamic port mapping với ECS.

---

## Cấu hình Health Check

### Question:

An ALB target group's health check is configured with:
- **Health check path**: `/`
- **Healthy threshold**: 2
- **Unhealthy threshold**: 2
- **Timeout**: 2 seconds
- **Interval**: 15 seconds

The `/` path returns HTTP 200, but the application takes **8 seconds** to respond. Why are instances being marked as **unhealthy**?

**A)** Health check interval is too short
**B)** **Timeout** is shorter than response time
**C)** Health check path should be `/health`
**D)** Only one threshold should be used

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**

**Vấn đề:** Health check timeout là 2 giây, ngắn hơn thời gian phản hồi 8 giây của ứng dụng. Vì vậy mỗi lần kiểm tra đều timeout; sau 2 lần thất bại liên tiếp (unhealthy threshold = 2), target bị đánh dấu unhealthy.

**Để khắc phục:**
- Chọn endpoint health check nhẹ, phản hồi nhanh nếu có thể.
- Nếu endpoint hợp lệ thực sự cần 8 giây, tăng Timeout lên hơn 8 giây và tăng Interval phù hợp (Timeout phải nhỏ hơn Interval).

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Interval 15 giây không ngắn hơn thời gian phản hồi; vấn đề trực tiếp là Timeout 2 giây. |
| **C** | `/` trả HTTP 200 nên path không phải nguyên nhân trong tình huống này. |
| **D** | Threshold = 2 hợp lệ; timeout xảy ra trước khi threshold được áp dụng. |

### 📝 Phân tích từ vựng:

- **Health check**: kiểm tra trạng thái của instance.
- **Timeout**: thời gian tối đa chờ response.
- **Healthy threshold**: số lần thành công cần có để đánh dấu healthy.
- **Unhealthy threshold**: số lần thất bại cần có để đánh dấu unhealthy.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu ứng dụng xử lý nặng** → Tăng timeout và interval.
- **Nếu cần failover nhanh hơn** → Giảm unhealthy threshold nhưng vẫn phải đặt timeout đủ dài.
- **Nếu health check quá nhạy** → Tăng thresholds để tránh false positives.

---

## Chính sách mở rộng của Auto Scaling Group

### Question:

An ASG has a **target tracking scaling policy** configured with:
- **Metric type**: ASGAverageCPUUtilization
- **Target value**: 50%

After deployment, the application experiences **high latency**. Investigation shows CPU is at **80%**, and the group is already running at its configured **maximum size**. Why is the ASG not scaling out further?

**A)** Target value is too low
**B)** **Cooldown period** has not elapsed
**C)** Maximum size is set too low
**D)** Health check is failing

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C:**

**Lý do:** ASG đã đạt **maximum capacity** → không thể scale out thêm dù CPU cao.

**Kiểm tra giới hạn của ASG:**
```bash
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name <name>
```

Kết quả hiển thị:
- MinSize
- MaxSize
- DesiredCapacity
- Instances (số lượng hiện tại)

**Nếu:**
- Số instances hiện tại = MaxSize
- CPU = 80%
→ ASG muốn scale out nhưng **không thể** vì đã đạt MaxSize.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Target 50% hợp lý - đặt thấp hơn sẽ scale out nhiều hơn, không phải nguyên nhân. |
| **B** | Cooldown period chỉ trì hoãn hành động scale; không ngăn scale khi đã đạt threshold. |
| **D** | Health check thất bại sẽ terminate instances, không ngăn scale out. |

### 📝 Phân tích từ vựng:

- **Target tracking policy**: tự động scale theo metric và target value.
- **Cooldown period**: thời gian chờ sau một lần scale trước khi scale tiếp.
- **Maximum size**: giới hạn số lượng instances tối đa.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu MaxSize đã hợp lý** → Kiểm tra Launch Template có đúng không.
- **Nếu cần scale thêm** → Tăng MaxSize.
- **Nếu độ trễ do ứng dụng** → Có thể cần tối ưu code thay vì scale.

---

## Cân bằng tải liên AZ

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

Traffic được phân phối **riêng trong từng AZ**:

| AZ | Instances | Lưu lượng |
|----|----------|---------|
| AZ-1 | 5 | 100% request của AZ-1 → 5 instances |
| AZ-2 | 2 | 100% request của AZ-2 → 2 instances |
| AZ-3 | 1 | 100% request của AZ-3 → 1 instance |

**Kết quả:**
- Mỗi instance ở AZ-1 nhận ít lưu lượng hơn.
- Instance duy nhất ở AZ-3 có thể quá tải vì phải xử lý toàn bộ lưu lượng của AZ-3.

**Khi bật Cross-Zone Load Balancing:**

Tất cả 8 instances nhận lưu lượng chia đều (khoảng 12.5% mỗi instance).

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Chỉ đúng khi Cross-Zone được bật, nhưng câu hỏi nêu rằng tính năng đang tắt. |
| **C** | Lưu lượng không dồn hết vào AZ có nhiều instances nhất. |
| **D** | Load Balancer phân phối theo quy tắc, không phải ngẫu nhiên. |

### 📝 Phân tích từ vựng:

- **Cross-zone load balancing**: phân phối lưu lượng giữa các AZ.
- **Intra-zone distribution**: phân phối lưu lượng chỉ trong AZ tương ứng.
- **Inter-AZ data transfer**: truyền dữ liệu giữa các AZ (NLB có tính phí).

### 🔄 Thay đổi ngữ cảnh:

- **ALB**: Cross-zone được bật mặc định (miễn phí).
- **NLB**: Cross-zone tắt mặc định (có phí inter-AZ).
- **Nếu muốn tiết kiệm** → Giữ tính năng tắt với NLB nhưng phải bảo đảm instances được phân bố đều giữa các AZ.

---

## EFS và EBS

### Question:

A company needs to store **shared files** that will be accessed by **50 EC2 instances** across **multiple Availability Zones**. The data is **content management data** that needs to be accessed frequently. Which storage solution is **most appropriate**?

**A)** EBS volumes attached to each instance
**B)** **EFS (Elastic File System)**
**C)** Instance Store on each instance
**D)** S3 bucket with Mountpoint

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (EFS):**

**Trường hợp sử dụng phù hợp với EFS:**
- **Nhiều instances** cần truy cập cùng dữ liệu.
- Triển khai **Multi-AZ**.
- Dữ liệu **được truy cập thường xuyên** (không phải lưu trữ lạnh).
- Hệ thống tệp **tuân thủ POSIX** (Linux).

**Đặc điểm của EFS:**
- NFS (Network File System) được quản lý.
- Tự động mở rộng.
- **Regional EFS** hoạt động trên nhiều AZ; **One Zone EFS** chỉ lưu dữ liệu trong một AZ và không chống chịu được việc mất AZ đó.
- Trả phí theo mức sử dụng.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (EBS)** | EBS chỉ gắn được với một instance (ngoại trừ io1/io2 Multi-Attach - phức tạp và đắt). |
| **C (Instance Store)** | Không thể chia sẻ giữa các instances, có tính tạm thời (mất dữ liệu). |
| **D (S3 with Mountpoint)** | Mountpoint for S3 cung cấp giao diện tệp cho object storage, nhưng không phải hệ thống tệp POSIX dùng chung đa năng; với ứng dụng cần ngữ nghĩa hệ thống tệp chia sẻ giữa nhiều instances, EFS phù hợp hơn. |

### 📝 Phân tích từ vựng:

- **Shared file storage**: lưu trữ tệp có thể truy cập từ nhiều instances.
- **NFS (Network File System)**: giao thức chia sẻ hệ thống tệp qua network.
- **POSIX-compliant**: tuân thủ chuẩn hệ thống tệp của Unix/Linux.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu chỉ có một instance** → EBS đủ dùng và rẻ hơn.
- **Nếu cần tương thích với Windows** → EFS không hỗ trợ; cần FSx for Windows.
- **Nếu cần lưu trữ cực rẻ** → Dùng S3 với lifecycle policies.

---

## Tóm tắt các mẫu câu hỏi DVA Exam - Phần 2

### Câu hỏi EBS:
- **io2 Block Express** cho cơ sở dữ liệu cần độ bền 99.999% và IOPS cao.
- **gp3** là lựa chọn mặc định cho hầu hết trường hợp sử dụng.
- Mã hóa cần qua các bước **snapshot → copy → volume mới**.
- **Instance Store** dùng cho cache và dữ liệu tạm thời.

### AMI và Snapshot:
- **AMI** = toàn bộ instance (có thể khởi chạy).
- **Snapshot** = một volume (sao lưu dữ liệu).

### Load Balancers:
- **ALB** = Layer 7, HTTP/HTTPS, định tuyến theo path.
- **NLB** = Layer 4, TCP/UDP, hiệu năng cao, static IP.
- **GWLB** = Layer 3, network appliances của bên thứ ba.

### ASG:
- Dùng **Target Tracking** để tự động mở rộng.
- Giới hạn **MaxSize** có thể ngăn scale out.
- Dùng **Health checks** từ Load Balancer.

### EFS:
- Lưu trữ chia sẻ giữa nhiều instances.
- Mặc định hoạt động trên nhiều AZ.
- Không hỗ trợ Windows.
