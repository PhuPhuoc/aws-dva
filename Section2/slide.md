# Section 2 - Lưu trữ trên EC2 | Tính sẵn sàng cao và khả năng mở rộng: ELB & ASG

---

## Phần 1: Lưu trữ trên EC2 (Trang 63-87)

### EBS Volume (Trang 64-68)

#### EBS Volume là gì? (Trang 64)

- **EBS (Elastic Block Store) Volume** là ổ đĩa qua mạng có thể gắn vào instances đang chạy.
- Cho phép instances **lưu dữ liệu bền vững** ngay cả sau khi bị kết thúc.
- Chỉ có thể gắn vào **một instance tại một thời điểm** (ngoại trừ io1/io2).
- **Gắn với một Availability Zone (AZ)**.
- Tương tự như một "USB qua mạng".

> *Sơ đồ: EC2 Instance kết nối với EBS Volume trong cùng AZ*

#### Root EBS Volume (Trang 65)

- Mặc định, khi tạo EC2 Instance sẽ tự động gắn một EBS Volume gọi là **Root EBS Volume**.
- Volume này chứa OS và các tệp hệ thống.
- Có thể thêm các volumes khác nếu cần.

> *Hình minh họa: EC2 Windows có thêm ổ D: và E:.*

#### EBS Multi-Attach (Trang 66-67)

- Chỉ **họ io1/io2** hỗ trợ Multi-Attach.
- Có thể gắn cùng một EBS volume vào **tối đa 16 EC2 instances** trong cùng AZ.

> *Sơ đồ: 16 EC2 instances cùng gắn một EBS volume io2.*

#### Xóa khi kết thúc (Delete on Termination) (Trang 68)

| Loại volume | Mặc định |
|-------------|-----------|
| **Root EBS Volume** | Bị **xóa** khi terminate (thuộc tính được bật) |
| **Non-root EBS Volume** | **Không bị xóa** (thuộc tính bị tắt) |

> *Trường hợp sử dụng: giữ lại root volume khi instance bị kết thúc.*

---

### EBS Snapshots (Trang 69)

- Tạo bản snapshot sao lưu EBS volume tại một thời điểm.
- Không bắt buộc tháo volume để tạo snapshot, nhưng **nên tháo**.
- Có thể **sao chép snapshots** qua AZ hoặc Region.

---

### Tổng quan về AMI (Trang 70-71)

#### AMI là gì?

- **AMI = Amazon Machine Image**.
- AMI là cấu hình tùy chỉnh của EC2 instance.
- Có thể thêm phần mềm, cấu hình, hệ điều hành, công cụ giám sát...
- **Khởi động nhanh hơn** vì phần mềm đã được đóng gói sẵn.
- AMI được tạo cho **Region cụ thể** (có thể sao chép sang Region khác).

**3 loại AMI:**

| Loại | Mô tả |
|-------|--------|
| **Public AMI** | AWS cung cấp sẵn |
| **Own AMI** | Tự tạo và bảo trì |
| **AWS Marketplace AMI** | AMI do bên khác tạo (có thể bán) |

#### Quy trình tạo AMI (Trang 71)

```
1. Khởi chạy EC2 instance → tùy chỉnh
2. Dừng instance (để bảo đảm tính toàn vẹn dữ liệu)
3. Tạo AMI → thao tác này cũng tạo EBS snapshots
4. Khởi chạy instances từ AMI
```

---

### EBS Snapshot và AMI (Trang 72-73)

| | **EBS Snapshot** | **AMI** |
|---|-------------------|---------|
| **Là gì** | Bản sao của một volume cụ thể | Bản sao lưu toàn bộ EC2 instance (bao gồm EBS snapshots) |
| **Khởi chạy EC2?** | ❌ Không thể | ✅ Có thể |
| **Trường hợp sử dụng** | Sao lưu dữ liệu của một volume (D:, E:) | Sao lưu toàn bộ instance, tạo nhiều instances khi mở rộng hệ thống |

---

### Tính năng của EBS Snapshots (Trang 74)

| Tính năng | Mô tả |
|---------|--------|
| **EBS Snapshot Archive** | Chuyển snapshot sang "archive tier", giảm 75% chi phí. Khôi phục mất 24-72 giờ |
| **Recycle Bin for EBS Snapshots** | Thiết lập rule giữ lại snapshots đã xóa để khôi phục khi vô tình xóa. Thời hạn: 1 ngày đến 1 năm |
| **Fast Snapshot Restore (FSR)** | Buộc khởi tạo đầy đủ để tránh độ trễ ở lần dùng đầu tiên (chi phí cao) |

---

### Các loại EBS Volume (Trang 75-80)

#### Tổng quan 6 loại Volume

| Loại | Đặc điểm |
|------|----------------|
| **gp2 / gp3** | SSD - Mục đích chung |
| **io1 / io2 Block Express** | SSD - IOPS được cấp phát |
| **st1** | HDD - Tối ưu thông lượng |
| **sc1** | HDD - Lưu trữ lạnh |

> *EBS Volumes được đặc trưng bởi: dung lượng | thông lượng | IOPS.*
> *Chỉ gp2/gp3 và io1/io2 Block Express có thể dùng làm boot volume.*

#### SSD mục đích chung (Trang 76)

| Loại | Mô tả |
|------|--------|
| **gp3** | Mức cơ sở 3,000 IOPS, 125 MiB/s. Có thể tăng IOPS đến 16,000 và thông lượng đến 1,000 MiB/s **độc lập** |
| **gp2** | Volume gp2 nhỏ có thể tăng đột biến lên 3,000 IOPS. Dung lượng và IOPS **liên kết**. IOPS tối đa: 16,000. 3 IOPS mỗi GB |

> *gp3 cho phép tăng IOPS và thông lượng độc lập với dung lượng volume.*

#### SSD IOPS được cấp phát (PIOPS) (Trang 77)

**io1 (4 GiB - 16 TiB):**
- PIOPS tối đa: 64,000 trên Nitro instances, 32,000 trên các loại khác.
- Có thể tăng PIOPS **độc lập** với dung lượng lưu trữ.

**io2 Block Express (4 GiB - 64 TiB):**
- Độ trễ dưới một mili giây.
- PIOPS tối đa: **256,000**, với tỷ lệ IOPS:GiB là 1,000:1.
- Hỗ trợ **EBS Multi-Attach**.

#### Ổ đĩa cứng (HDD) (Trang 78)

| Loại | Trường hợp sử dụng | Thông lượng tối đa | IOPS tối đa |
|------|----------------|----------------|----------|
| **st1** (tối ưu thông lượng) | Big Data, kho dữ liệu, xử lý nhật ký | 500 MiB/s | 500 |
| **sc1** (HDD lạnh) | Dữ liệu ít truy cập, chi phí thấp nhất | 250 MiB/s | 250 |

#### Tóm tắt các loại EBS Volume (Trang 79-80)

**Volumes SSD:**

| Loại volume | Độ bền | Trường hợp sử dụng | IOPS tối đa | Boot volume |
|-------------|------------|-----------|----------|-------------|
| **gp3** | 99.8-99.9% | Khối lượng công việc giao dịch, máy tính để bàn ảo | 16,000 | ✅ |
| **gp2** | 99.8-99.9% | Boot volumes, môi trường phát triển/kiểm thử | 16,000 | ✅ |
| **io2 Block Express** | **99.999%** | >64,000 IOPS, cơ sở dữ liệu nhiều thao tác I/O | 256,000 | ✅ |
| **io1** | 99.8-99.9% | IOPS ổn định, >16,000 IOPS | 64,000 | ✅ |

**Volumes HDD:**

| Loại volume | Độ bền | Trường hợp sử dụng | Thông lượng tối đa | Boot volume |
|-------------|------------|-----------|---------------|-------------|
| **st1** | 99.8-99.9% | Dữ liệu lớn, kho dữ liệu | 500 MiB/s | ❌ |
| **sc1** | 99.8-99.9% | Lưu trữ lạnh, chi phí thấp nhất | 250 MiB/s | ❌ |

---

### EBS Multi-Attach (Trang 81)

- Gắn cùng một EBS volume vào **nhiều EC2 instances** trong cùng AZ.
- **Trường hợp sử dụng:** tăng tính sẵn sàng cho ứng dụng Linux dạng cluster (ví dụ: Teradata).
- Ứng dụng phải **quản lý các thao tác ghi đồng thời**.
- Hỗ trợ **tối đa 16 EC2 instances**.

---

### Mã hóa EBS (Trang 82-83)

Khi tạo EBS volume đã mã hóa:
- Dữ liệu lưu trữ (at rest) được mã hóa trong volume.
- Dữ liệu truyền tải (in flight) giữa instance và volume được mã hóa.
- Tất cả **snapshots đều được mã hóa**.
- Tất cả **volumes tạo từ snapshot đều được mã hóa**.
- Mã hóa có **ảnh hưởng tối thiểu đến độ trễ**.
- EBS Encryption dùng các khóa **KMS (AES-256)**.

#### Mã hóa EBS volume chưa được mã hóa (Trang 83)

```
1. Tạo EBS snapshot của volume
2. Mã hóa snapshot (bằng thao tác sao chép)
3. Tạo EBS volume mới từ snapshot (volume sẽ được mã hóa)
4. Gắn encrypted volume vào instance
```

---

### EC2 Instance Store (Trang 84)

- **Ổ đĩa phần cứng hiệu năng cao**.
- Hiệu năng I/O **tốt hơn** EBS.
- **Nguy cơ mất dữ liệu** khi EC2 bị dừng (lưu trữ tạm thời).
- **Trường hợp sử dụng:** buffer, cache, dữ liệu tạm, nội dung nhất thời.
- Một số Instance Type không hỗ trợ Instance Store.

> *Instance Store là lưu trữ tạm thời, EBS là lưu trữ bền vững.*

---

### Amazon EFS - Elastic File System (Trang 85-86)

**EFS là gì:**
- Hệ thống tệp mạng **NFS** được quản lý.
- Có thể gắn vào nhiều EC2 instances.
- **Regional EFS** hoạt động với EC2 instances trên nhiều AZ và có tính sẵn sàng cao; chi phí cao (3x gp2 theo slide).
- **One Zone EFS** lưu dữ liệu trong một AZ, nên không chống chịu được việc mất AZ đó.
- **Trường hợp sử dụng:** quản lý nội dung, phục vụ web, chia sẻ dữ liệu.
- Dùng giao thức **NFSv4.1**.
- **Chỉ tương thích với Linux** (không phải Windows).
- Hệ thống tệp POSIX (tương tự Linux).

#### Các lớp lưu trữ EFS (Trang 86)

| Lớp lưu trữ | Mô tả | Độ trễ byte đầu tiên | Độ bền |
|---------------|--------|-------------------|------------|
| **EFS Standard** | Dữ liệu đang hoạt động, cần độ trễ dưới mili giây | Dưới một mili giây | 99.999999999% |
| **EFS IA** | Dữ liệu ít hoạt động, truy cập vài lần mỗi quý | Hàng chục ms | 99.999999999% |
| **EFS Archive** | Dữ liệu ít hoạt động, truy cập vài lần mỗi năm | Hàng chục ms | 99.999999999% |

---

### EBS, EFS và Instance Store (Trang 87)

| Đặc điểm | **EBS** | **EFS** | **Instance Store** |
|---------|---------|---------|-------------------|
| **Kết nối** | Một instance | Nhiều instances | Gắn cục bộ với instance |
| **AZ** | Một AZ | Nhiều AZ | Gắn với instance |
| **Lưu trữ** | ✅ Bền vững | ✅ Bền vững | ❌ Tạm thời |
| **Chi phí** | $/GB | 3x gp2 | Đã gồm trong giá instance (nhưng giá instance cao hơn) |
| **Trường hợp sử dụng** | Cơ sở dữ liệu, boot | Chia sẻ tệp | Cache, dữ liệu tạm |

---

## Phần 2: Tính sẵn sàng cao và khả năng mở rộng: ELB & ASG (Trang 88-129)

### Khả năng mở rộng và tính sẵn sàng cao (Trang 89)

**Khả năng mở rộng (Scalability):**
- Ứng dụng/hệ thống có thể xử lý tải lớn hơn bằng cách thích ứng.
- Có hai loại:
  - **Mở rộng theo chiều dọc** (scale up/down)
  - **Mở rộng theo chiều ngang** (scale out/in), còn gọi là elasticity

> *Khả năng mở rộng khác với tính sẵn sàng cao nhưng có liên quan.*

---

### Mở rộng theo chiều dọc (Vertical Scalability) (Trang 90)

- Mở rộng theo chiều dọc (scale up/down) là tăng kích thước instance.
- Thường dùng cho **hệ thống không phân tán**, chẳng hạn cơ sở dữ liệu.
- RDS và ElastiCache có thể mở rộng theo chiều dọc.

```
T2.micro (1 vCPU, 1 GB RAM)
        ↓ tăng quy mô
M5.large (2 vCPU, 8 GB RAM)
```
---

### Mở rộng theo chiều ngang (Horizontal Scalability) (Trang 91)

- Mở rộng theo chiều ngang (scale out/in) là tăng số lượng instances.
- Phù hợp với **hệ thống phân tán**.
- Thường dùng cho ứng dụng web và ứng dụng hiện đại.

```
Auto Scaling Group
├── EC2 Instance
├── EC2 Instance
├── EC2 Instance
└── EC2 Instance
```

---

### Tính sẵn sàng cao (High Availability) (Trang 92)

- Tính sẵn sàng cao thường đi cùng với mở rộng theo chiều ngang.
- Chạy ứng dụng trên **ít nhất hai trung tâm dữ liệu (AZs)**.
- **Mục tiêu:** tiếp tục hoạt động khi một trung tâm dữ liệu gặp sự cố.

| Loại | Mô tả | Ví dụ |
|------|--------|-------|
| **HA thụ động** | Dự phòng thụ động | RDS Multi-AZ |
| **HA chủ động** | Dự phòng chủ động | Mở rộng theo chiều ngang với ASG |

---

### Tính sẵn sàng cao và khả năng mở rộng cho EC2 (Trang 93-94)

| Phương pháp | Mô tả |
|----------|--------|
| **Mở rộng theo chiều dọc** | Tăng kích thước instance (t2.nano: 1 vCPU, 0.5 GB → u-12tb1.metal: 448 vCPU, 12.3 TB RAM) |
| **Mở rộng theo chiều ngang** | Tăng số lượng instances (ASG + Load Balancer) |
| **Tính sẵn sàng cao** | Chạy instances trên nhiều AZs (ASG và Load Balancer multi-AZ) |

---

### Cân bằng tải là gì? (Trang 95)

- Load Balancer là máy chủ chuyển tiếp lưu lượng đến nhiều máy chủ phía sau (EC2 instances).

> *Sơ đồ: Client → Load Balancer → nhiều EC2 instances.*

---

### Vì sao cần Load Balancer? (Trang 96)

| Lợi ích | Mô tả |
|----------|--------|
| **Phân phối tải** | Phân phối tải giữa nhiều instances |
| **Một điểm truy cập** | Một DNS endpoint cho ứng dụng |
| **Xử lý lỗi** | Tự động xử lý lỗi của instance phía sau |
| **Kiểm tra tình trạng** | Kiểm tra định kỳ tình trạng của instances |
| **Kết thúc TLS** | Xử lý HTTPS cho website |
| **Gắn kết phiên** | Duy trì session bằng cookies |
| **Tính sẵn sàng cao** | Triển khai qua nhiều AZs |
| **Tách luồng lưu lượng** | Lưu lượng công khai và riêng tư |

---

### Elastic Load Balancer (Trang 97)

- Load Balancer được quản lý; AWS đảm bảo tính sẵn sàng.
- AWS xử lý nâng cấp, bảo trì và tính sẵn sàng cao.
- **Tích hợp với:** EC2, EC2 Auto Scaling Groups, Amazon ECS, ACM, CloudWatch, Route 53, WAF, Global Accelerator.

---

### Kiểm tra tình trạng (Health Checks) (Trang 98)

- Health Checks rất quan trọng với Load Balancers.
- Chúng cho Load Balancer biết instance có sẵn sàng phản hồi request được chuyển đến hay không.
- Health check được thực hiện trên **port và route** (`/health` là đường dẫn phổ biến).
- Nếu response không phải **200 (OK)**, instance được đánh dấu **unhealthy**.

---

### Security Groups cho Load Balancer (Trang 99)

```
Load Balancer SG:
  - Inbound: 0.0.0.0/0 (port 80, 443)

Application SG:
  - Inbound: Chỉ từ Load Balancer SG
```

> *Sơ đồ: luồng Security Group từ Internet → LB → EC2.*

---

### Các loại Load Balancer trên AWS (Trang 100)

| Đặc điểm | **Application LB** | **Network LB** | **Gateway LB** | **Classic LB** |
|---------|-------------------|----------------|---------------|---------------|
| **Lớp** | Layer 7 | Layer 4 | Layer 3 Gateway + Layer 4 Load Balancing | Layer 4/7 |
| **Loại đích** | IP, Instance, Lambda | IP, Instance, ALB | IP, Instance | EC2 instance |
| **Giao thức** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP | TCP, SSL/TLS, HTTP, HTTPS |

---

### Application Load Balancer (ALB) (Trang 101-106)

**Đặc điểm:**
- Hoạt động ở **Layer 7 (HTTP/HTTPS)**.
- Cân bằng tải cho **nhiều ứng dụng HTTP** trên các máy khác nhau (target groups).
- Cân bằng tải cho **nhiều ứng dụng trên cùng một máy** (containers).
- Hỗ trợ **HTTP/2 và WebSocket**.
- Hỗ trợ **redirects** (ví dụ: HTTP → HTTPS).

**Tùy chọn định tuyến:**
- Đường dẫn trong URL (`/users`, `/posts`).
- Tên host (`one.example.com`, `other.example.com`).
- Query String và Headers (`?id=123&order=false`).

**Target Groups:**
- EC2 instances (có thể được ASG quản lý).
- ECS tasks.
- Lambda functions.
- IP Addresses (phải là private IPs).

**Thông tin cần biết:**
- Tên host cố định (`XXX.region.elb.amazonaws.com`).
- IP thực của client nằm trong header **X-Forwarded-For**.

---

### Network Load Balancer (NLB) (Trang 107-109)

**Đặc điểm:**
- Hoạt động ở **Layer 4 (TCP/UDP)**.
- Xử lý **hàng triệu request mỗi giây**.
- **Độ trễ thấp hơn**, khoảng 100 ms (so với 400 ms của ALB).
- Mỗi AZ có **1 static IP**, hỗ trợ **Elastic IP** (để allowlist IP cụ thể).
- **Không nằm trong AWS Free Tier**.

**Target Groups:**
- EC2 instances.
- IP Addresses (private IPs).
- Application Load Balancer.
- **Health Checks:** hỗ trợ giao thức TCP, HTTP và HTTPS.

---

### Gateway Load Balancer (GWLB) (Trang 110-111)

- Triển khai, mở rộng và quản lý đội ngũ **network virtual appliances của bên thứ ba**.
- **Trường hợp sử dụng:** firewalls, IDS/IPS, Deep Packet Inspection, chỉnh sửa payload.
- Hoạt động ở **Layer 3** (gói IP).
- Kết hợp: Transparent Network Gateway + Load Balancer.
- Dùng **GENEVE protocol** trên port 6081.
- **Target Groups:** EC2 instances và IP addresses (phải là private IPs).

---

### Phiên gắn kết (Sticky Sessions) (Trang 112)

- **Stickiness** nghĩa là cùng một client luôn được chuyển đến cùng một instance phía sau Load Balancer.
- Hoạt động với CLB, ALB, NLB.
- Với CLB và ALB, cookie dùng cho stickiness có thời hạn do bạn kiểm soát.
- **Trường hợp sử dụng:** giữ session data của người dùng.
- **Lưu ý:** có thể làm tải phân bố mất cân bằng giữa các EC2 instances phía sau.

---

### Cân bằng tải liên AZ (Cross-Zone Load Balancing) (Trang 113-114)

| Loại LB | Mặc định | Dữ liệu liên AZ |
|---------|---------|----------------|
| **Application LB** | ✅ Bật (có thể tắt) | Miễn phí |
| **Network LB** | ❌ Tắt (có thể bật) | **Có tính phí** |
| **Gateway LB** | ❌ Tắt (có thể bật) | **Có tính phí** |

---

### Cơ bản về SSL/TLS (Trang 115)

- **SSL Certificate** mã hóa lưu lượng giữa clients và Load Balancer.
- **SSL** = Secure Sockets Layer.
- **TLS** = Transport Layer Security (phiên bản mới hơn).
- Chứng chỉ SSL công khai do **Certificate Authorities (CA)** cấp.
- Chứng chỉ có ngày hết hạn và phải được gia hạn.

---

### Chứng chỉ SSL của Load Balancer (Trang 116)

- LB dùng **X.509 certificate**.
- Quản lý certificates bằng **ACM (AWS Certificate Manager)**.
- HTTPS listener yêu cầu:
  - Một certificate mặc định.
  - Có thể thêm danh sách certificates cho nhiều domain.
- Clients dùng **SNI (Server Name Indication)** để chỉ định hostname.

---

### SNI - Server Name Indication (Trang 117)

- SNI giải quyết vấn đề gắn **nhiều SSL certificates** lên một web server.
- **Hoạt động với:** ALB, NLB (thế hệ mới), CloudFront.
- **Không hoạt động với:** CLB (thế hệ cũ).

---

### Ngừng kết nối (Connection Draining) (Trang 118)

| Tên tính năng | Load Balancer |
|---------|--------------|
| **Connection Draining** | CLB |
| **Deregistration Delay** | ALB & NLB |

- Thời gian hoàn tất các request đang xử lý (in-flight) khi instance đang bị deregister hoặc unhealthy.
- Ngừng gửi request mới đến instance đang deregister.
- Phạm vi: **1-3600 giây** (mặc định: 300).
- Có thể tắt (đặt = 0).
- Nên đặt giá trị thấp nếu request ngắn.

---

### Auto Scaling Group (ASG) (Trang 119-120)

**Trong thực tế:** tải trên website/ứng dụng có thể thay đổi.

**Mục tiêu của ASG:**
- **Scale out:** thêm EC2 instances để đáp ứng tải tăng.
- **Scale in:** xóa EC2 instances khi tải giảm.
- Đảm bảo có số lượng instances tối thiểu và tối đa.
- Tự động đăng ký instances mới với Load Balancer.
- Tạo lại EC2 instance nếu instance trước bị kết thúc (ví dụ: unhealthy).

> *ASG miễn phí (chỉ trả tiền cho EC2 instances bên dưới).*

---


### Auto Scaling Group với Load Balancer (Trang 121)

> *Sơ đồ: Auto Scaling Group đăng ký các EC2 instances với Load Balancer.*

---

### Các thuộc tính của Auto Scaling Group (Trang 122)

- **Launch Template** (Launch Configurations cũ đã deprecated).
- AMI và Instance Type.
- EC2 User Data.
- EBS Volumes.
- Security Groups.
- SSH Key Pair.
- IAM Roles cho EC2 instances.
- Thông tin Network và Subnet.
- Thông tin Load Balancer.
- Min Size / Max Size / Initial Capacity.
- Scaling Policies.

---

### Auto Scaling, CloudWatch Alarms và mở rộng (Trang 123)

- Có thể mở rộng ASG dựa trên **CloudWatch alarms**.
- Alarm theo dõi một metric, ví dụ CPU trung bình hoặc custom metric; CPU trung bình được tính trên các instances trong ASG.
- Tùy alarm, tạo policy **scale-out** (tăng số instances) hoặc **scale-in** (giảm số instances).

---

### Các chính sách mở rộng của Auto Scaling Groups (Trang 124)

- **Dynamic Scaling:**
  - **Target Tracking Scaling** (được khuyến nghị mạnh): điều chỉnh capacity để giữ metric CloudWatch quanh target; ví dụ CPU trung bình của ASG ở mức khoảng 40%.
  - **Simple / Step Scaling:** phản ứng với CloudWatch alarm; ví dụ CPU >70% thì thêm 2 instances, CPU <30% thì bỏ 1 instance.
- **Scheduled Scaling:** mở rộng theo lịch sử dụng đã biết; ví dụ tăng min capacity lên 10 vào 5 giờ chiều thứ Sáu.

---

### Auto Scaling Groups - Mở rộng dự đoán (Trang 125)

- Dùng học máy để dự đoán nhu cầu capacity dựa trên dữ liệu lịch sử từ CloudWatch.

---

### Metrics phù hợp để mở rộng (Trang 126)

- **CPUUtilization:** mức sử dụng CPU trung bình của các instances.
- **RequestCountPerTarget:** giữ số request trên mỗi EC2 instance ổn định.
- **Average Network In / Out:** phù hợp nếu ứng dụng bị giới hạn bởi network.
- Custom metric được gửi lên CloudWatch.

---

### Thời gian chờ giữa các lần mở rộng của Auto Scaling Groups (Trang 127)

- Sau khi có hoạt động scaling, ASG đi vào **cooldown period** (mặc định 300 giây).
- Trong khoảng này, ASG không launch hoặc terminate thêm instances để metric ổn định.
- Lời khuyên của slide: dùng AMI sẵn sàng để giảm thời gian cấu hình, sớm phục vụ request và giảm cooldown period.

---

### Auto Scaling - Instance Refresh (Trang 128)

- **Instance Refresh** dùng để thay thế các EC2 instances sau khi cập nhật Launch Template.
- Cấu hình **minimum healthy percentage** và **warm-up time** (thời gian instance cần để sẵn sàng phục vụ).

### Bài tập (Trang 129)
- Application Load Balancer.
- Application Load Balancer kết hợp EC2 Auto Scaling Group.

---
## Tổng kết Section 2

### Lưu trữ trên EC2

| Loại lưu trữ | Đặc điểm |
|---------|-----------|
| **EBS** | Ổ đĩa qua mạng, bền vững, một AZ, một instance (trừ io1/io2) |
| **EFS** | Regional: chia sẻ nhiều AZ; One Zone: một AZ; hệ thống tệp chia sẻ, chỉ Linux |
| **Instance Store** | Ổ đĩa cục bộ, hiệu năng cao, tạm thời (mất dữ liệu khi dừng) |

### Các loại EBS Volume

| Loại | Trường hợp sử dụng | IOPS | Chi phí |
|------|----------|------|------|
| **gp3** | Mục đích chung, boot | 16,000 | $ |
| **io2 Block Express** | Cơ sở dữ liệu quan trọng | 256,000 | $$$ |
| **st1** | Big data, xử lý nhật ký | 500 | $$ |
| **sc1** | Lưu trữ lạnh | 250 | $ |

### Load Balancers

| Loại | Lớp | Trường hợp sử dụng |
|------|-------|----------|
| **ALB** | L7 | HTTP/HTTPS, microservices |
| **NLB** | L4 | TCP/UDP, hiệu năng cao |
| **GWLB** | L3 | Network virtual appliances của bên thứ ba |

### ASG

- Tự động scale out/in.
- Kết hợp với Load Balancer.
- Minimum, Maximum, Desired capacity.
- Kiểm tra tình trạng và tự động thay thế instances.
