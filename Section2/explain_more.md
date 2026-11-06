# Giải thích bổ sung - Section 2

> *Phần này chứa kiến thức thực tế từ kinh nghiệm triển khai AWS. Mỗi topic có giải thích cốt lõi, NÊN/KHÔNG NÊN với lý do cụ thể.*

---

## 1. Các loại EBS Volume

### Chọn loại EBS Volume - So sánh

#### Giải thích cốt lõi

EBS có 6 loại volume, chia thành 2 nhóm chính:

**Các loại dựa trên SSD (4 loại):**

| Loại | IOPS tối đa | Thông lượng tối đa | Độ bền | Boot volume |
|------|----------|----------------|------------|-------------|
| **gp3 (Regional)** | 80,000 | 2,000 MiB/s | 99.8-99.9% | ✅ |
| **gp2** | 16,000 | 250 MiB/s | 99.8-99.9% | ✅ |
| **io2 Block Express** | 256,000 | 4,000 MiB/s | **99.999%** | ✅ |
| **io1** | 64,000 | 1,000 MiB/s | 99.8-99.9% | ✅ |

**Cập nhật 2026-11-06:** Mức tối đa của gp3 tiêu chuẩn là 80,000 IOPS, 2,000 MiB/s và 64 TiB; trên **Outposts** là 16,000 IOPS, 1,000 MiB/s và 16 TiB. Mức tối đa thực tế còn phụ thuộc vào kích thước volume và tỉ lệ cấp phát IOPS/thông lượng. [AWS gp3 limits](https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html)

**Các loại dựa trên HDD (2 loại):**

| Loại | Thông lượng tối đa | IOPS tối đa | Boot volume |
|------|---------------|----------|-------------|
| **st1** | 500 MiB/s | 500 | ❌ |
| **sc1** | 250 MiB/s | 250 | ❌ |

**Khác biệt quan trọng giữa gp2 và gp3:**

| Đặc điểm | gp2 | gp3 |
|---------|-----|-----|
| **IOPS so với dung lượng** | Liên kết (3 IOPS/GB) | **Độc lập** |
| **IOPS cơ sở** | Tỷ lệ theo dung lượng | Cố định 3,000 |
| **IOPS cấp phát riêng** | Không | ✅ Có thể tăng |
| **Thông lượng** | Tối đa 250 MiB/s | Tối đa 2,000 MiB/s (Regional); 1,000 MiB/s (Outposts) |
| **Chi phí** | Đắt hơn gp3 | **Rẻ hơn** |

#### NÊN làm (DO):
- **Dùng gp3 làm mặc định** → Vì gp3 có giá thấp hơn gp2 trên mỗi GiB và cho phép cấp IOPS/thông lượng độc lập với dung lượng; xác định đúng giới hạn theo Region hay Outposts.
- **Dùng io2 Block Express khi cần độ bền 99.999%, độ trễ dưới mili giây hoặc hiệu năng vượt giới hạn gp3** → gp3 có độ bền 99.8-99.9%, dù mức tối đa Regional hiện đã lên tới 80,000 IOPS.

#### KHÔNG NÊN làm (DON'T):
- **Ưu tiên gp3 cho khối lượng công việc mới nếu đáp ứng yêu cầu** → gp2 vẫn được hỗ trợ, nhưng IOPS gắn với dung lượng (3 IOPS/GiB, tối đa 16,000 ở 5,334 GiB); gp3 tách IOPS/thông lượng khỏi dung lượng và thường có giá thấp hơn. NÊN: chọn loại theo hiệu năng và giá cần thiết, không vì cho rằng gp2 đã deprecated.
- **Dùng st1/sc1 làm boot volume** → Vì volume dựa trên HDD không hỗ trợ boot. NÊN: dùng gp3 hoặc gp2 cho hệ điều hành.

---

### Dung lượng EBS Volume

#### Giải thích cốt lõi

EBS có hai thao tác chính với dung lượng:

**Tăng dung lượng (upsize):**
- ✅ Có thể sửa đổi volume khi đang chạy → **không cần downtime**.
- ✅ Không cần dừng instance.
- ✅ Có hiệu lực ngay.

**Giảm dung lượng (downsize):**
- ❌ **Không có thao tác trực tiếp**.
- Cần thực hiện thủ công:
  ```
  1. Dừng instance (downtime)
  2. Tạo snapshot (sao lưu để phòng mất dữ liệu)
  3. Tạo volume mới có dung lượng nhỏ hơn
  4. Gắn volume mới
  5. Mount → sao chép dữ liệu từ volume cũ
  6. Umount → tháo volume cũ
  7. Xóa volume cũ (nếu không cần nữa)
  ```
- Mất thời gian, cần downtime và có nguy cơ mất dữ liệu nếu thao tác sai.

**So sánh với các lựa chọn khác:**

| Loại lưu trữ | Tăng dung lượng | Giảm dung lượng | Lưu trữ bền vững | Chi phí |
|---------|--------|----------|-------------|------|
| **EBS** | ✅ Khi đang chạy | ❌ Thủ công | ✅ | $/GB |
| **Instance Store** | ❌ | ❌ | ❌ Tạm thời | Đã bao gồm |
| **EFS** | ✅ Tự động | ✅ Tự động | ✅ | Tùy Region, storage class và throughput/access |

#### NÊN làm (DO):
- **Bắt đầu với dung lượng vừa đủ rồi tăng dần** → Vì tăng dung lượng chỉ cần một thao tác trên console, có hiệu lực ngay và không cần downtime. Giảm dung lượng phải thực hiện thủ công, phức tạp; so với cấp dư ngay từ đầu rồi không dùng hết thì cách này tránh lãng phí.

#### KHÔNG NÊN làm (DON'T):
- **Cấp dư EBS "cho chắc" ngay từ đầu** → Vì nếu sau này dữ liệu ít hơn, không thể giảm trực tiếp. Phải tạo volume mới, sao chép dữ liệu rồi chuyển đổi → mất thời gian và có rủi ro. NÊN: ước tính nhu cầu ban đầu cộng một phần dự phòng nhỏ, rồi resize khi có dữ liệu thực tế.

---

### Mã hóa EBS

#### Giải thích cốt lõi

EBS encryption sử dụng **AES-256** thông qua **AWS KMS**:

**Mã hóa được áp dụng cho:**
- Dữ liệu lưu trữ (at rest) trong volume.
- Dữ liệu truyền tải (in flight) giữa instance và volume.
- Tất cả snapshots tạo từ volume đã mã hóa.
- Tất cả volumes tạo từ snapshot đã mã hóa.

**Quy trình mã hóa volume chưa được mã hóa:**

```
Volume chưa mã hóa
    ↓ Tạo Snapshot
Snapshot chưa mã hóa
    ↓ Sao chép (bật mã hóa)
Snapshot đã mã hóa ← (snapshot mới đã được mã hóa)
    ↓ Tạo Volume
Volume đã mã hóa
    ↓ Gắn vào
Instance
```

**Điểm quan trọng:**
- Mã hóa có **ảnh hưởng tối thiểu đến độ trễ** (thường <3%).
- Không thể mã hóa tại chỗ.
- Không thể tắt mã hóa trên volume đã mã hóa.

#### NÊN làm (DO):
- **Bật EBS encryption by default riêng cho từng Region cần dùng** → Đây là setting cấp account theo Region, không tự bật mọi nơi. Khi bật, volume mới và snapshot copy mới được mã hóa; tài nguyên đã tồn tại không bị thay đổi. [AWS EBS encryption by default](https://docs.aws.amazon.com/ebs/latest/userguide/encryption-by-default.html) (đã kiểm tra 2026-11-06).
- **Dùng KMS keys có rotation** → Customer managed keys cho phép tự động luân chuyển key hằng năm, tăng cường bảo mật.

#### KHÔNG NÊN làm (DON'T):
- **Tắt encryption để "cải thiện performance"** → Vì encryption chỉ ảnh hưởng khoảng 3% độ trễ. Trong hầu hết trường hợp, đây là sự đánh đổi hợp lý để tăng bảo mật. Với khối lượng công việc nhạy cảm về hiệu năng, hãy kiểm thử trước.

---

### Sao lưu EBS bằng Snapshots

#### Giải thích cốt lõi

**Cơ chế EBS Snapshot:**
- Snapshot là **incremental** → chỉ sao lưu những blocks đã thay đổi.
- Lần đầu sao lưu toàn bộ; các lần sau chỉ sao lưu phần thay đổi → tiết kiệm storage.
- Có thể sao chép **qua AZ** hoặc **qua Region**.

**Các lớp lưu trữ của snapshots:**

| Lớp | Chi phí tham khảo (USD/GB-tháng) | Thời gian khôi phục | Trường hợp sử dụng |
|------|------|--------------|----------|
| **Standard** | $0.05/GB | Ngay lập tức | Bản sao lưu gần đây |
| **Archive** | $0.0125/GB (giảm 75%) | 24-72 giờ | Lưu giữ dài hạn |
| **Recycle Bin** | Không thu thêm phí; snapshot tính theo mức thông thường | Ngay lập tức | Phòng xóa nhầm |
Các mức giá Standard/Archive là số tham khảo, có thể khác theo Region; Recycle Bin không thu thêm phí riêng nhưng snapshot bên trong vẫn tính theo mức snapshot thông thường. [AWS EBS pricing](https://aws.amazon.com/ebs/pricing/) · [Recycle Bin pricing](https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin.html)

**Recycle Bin:**
- Thiết lập rules để giữ lại snapshots đã xóa.
- Thời hạn lưu: 1 ngày → 1 năm.
- Có thể khôi phục sau khi đã xóa.

#### NÊN làm (DO):
- **Dùng Lifecycle Policy để tự động sao lưu** → Bảo đảm sao lưu đều đặn mà không cần thao tác thủ công. Có thể thiết lập lịch hằng ngày/hằng tuần cùng retention policy.
- **Thiết lập Recycle Bin với thời hạn 1-7 ngày** → Giúp phòng xóa nhầm; vẫn có thời gian khôi phục nếu ai đó xóa nhầm.

#### KHÔNG NÊN làm (DON'T):
- **Không có chiến lược sao lưu** → EBS không bảo đảm độ bền 100%. Dù độ bền cao (99.8-99.999%), khi quy mô lớn vẫn có nguy cơ mất dữ liệu.
- **Giữ snapshots lâu hơn mức cần thiết** → Tốn chi phí. NÊN: dùng Archive tier cho snapshots hơn 30 ngày không cần truy cập thường xuyên.

---

## 2. EC2 Instance Store

### Khi nào dùng Instance Store

#### Giải thích cốt lõi

**Instance Store là gì:**
- Lưu trữ cục bộ gắn với instance; thiết bị có thể là NVMe SSD, loại SSD khác hoặc HDD tùy Instance Type.
- Dữ liệu vẫn còn sau **reboot**, nhưng mất khi instance **stopped, hibernated hoặc terminated**; lỗi ổ đĩa nền tảng cũng có thể gây mất dữ liệu.
- Hiệu năng phụ thuộc Instance Type, thiết bị và khối lượng công việc; không có mức IOPS/độ trễ chung cho mọi Instance Store.

**So sánh với EBS:**

| Đặc điểm | Instance Store | EBS |
|---------|---------------|-----|
| **Gắn kết** | Cục bộ với instance; không thể chuyển volume sang instance khác | Block storage qua mạng; có thể attach/detach theo giới hạn của volume |
| **Lưu trữ bền vững** | Giữ dữ liệu khi reboot; mất khi stop/hibernate/terminate | Bền vững, độc lập với vòng đời instance |
| **Hiệu năng** | Phụ thuộc Instance Type và phần cứng cục bộ | Phụ thuộc loại volume và hiệu năng được cấp phát |
| **Thay đổi kích thước** | Không thể resize volume trực tiếp | Có thể sửa đổi volume bằng Elastic Volumes |

Instance Store thường có lợi thế truy cập cục bộ, không qua network như EBS; hãy so sánh trên Instance Type và khối lượng công việc cụ thể thay vì dựa vào con số IOPS/độ trễ chung.

#### NÊN làm (DO):
- **Dùng cho cache, buffer, scratch data** → Dữ liệu có thể tái tạo; xác nhận Instance Type cung cấp thiết bị phù hợp (NVMe chỉ có trên những loại hỗ trợ).

#### KHÔNG NÊN làm (DON'T):
- **Dùng làm nơi duy nhất lưu dữ liệu cần giữ** → Dữ liệu mất khi instance dừng, ngủ đông hoặc kết thúc; sao chép dữ liệu cần giữ sang EBS, EFS hoặc S3. [Độ bền dữ liệu của EC2 Instance Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-store-lifetime.html) (đã kiểm tra 2026-11-06).
## 3. EFS - Elastic File System

### Khi nào nên dùng EFS

#### Giải thích cốt lõi

**EFS là gì:**
- **NFS v4.1** được quản lý → hệ thống tệp tuân thủ POSIX.
- Có thể gắn cùng lúc vào **hàng trăm instances**.
- **Regional EFS** lưu dữ liệu dư thừa trên nhiều Availability Zones, nên có tính sẵn sàng cao.
- **One Zone EFS** lưu dữ liệu trong một Availability Zone; nếu AZ đó gặp sự cố lớn, dữ liệu có thể mất. [AWS EFS file system types](https://docs.aws.amazon.com/efs/latest/ug/features.html)
- Tự động mở rộng → không cần ước tính capacity trước.

**So sánh với EBS và S3:**

| Đặc điểm | EFS | EBS | S3 |
|---------|-----|-----|-----|
| **Truy cập** | Nhiều instances | Một instance | Nhiều clients qua API |
| **Giao thức** | NFS (hệ thống tệp) | Block | Object |
| **Lưu trữ bền vững** | ✅ | ✅ | ✅ |
| **Gắn với AZ** | Regional: nhiều AZ; One Zone: một AZ | Một AZ | S3 không gắn với một AZ |
| **Chi phí** | Tùy Region, storage class, throughput và access | $/GB và hiệu năng cấp phát | $/GB + requests |
| **Windows** | ❌ | ✅ | ✅ |

Chi phí EFS không có tỷ lệ cố định so với gp2: giá phụ thuộc Region, storage class, throughput mode và access. IA/Archive có giá lưu trữ thấp hơn nhưng có thể có phí truy cập/tiering; hãy so sánh khối lượng công việc trên [định giá EFS](https://aws.amazon.com/efs/pricing/) thay vì dùng một hệ số chung (kiểm tra 2026-11-06).

**Các chế độ hiệu năng EFS:**

| Chế độ | Mô tả | Trường hợp sử dụng |
|------|------------|----------|
| **General Purpose** | Độ trễ thấp, burst | Máy chủ web, CMS, phát triển |
| **Max I/O** | Thông lượng cao hơn, độ trễ cao hơn | Big data, ML, genomics |

**Các lớp lưu trữ EFS:**

| Lớp | Mẫu truy cập | Chi phí lưu trữ | Độ trễ đọc byte đầu tiên |
|-------|---------------|----------------|--------------------------|
| **Standard** | Truy cập thường xuyên | Cao nhất | Dưới 1 ms |
| **IA (Infrequent Access)** | Truy cập vài lần mỗi quý | Thấp hơn Standard đến 95% | Hàng chục ms |
| **Archive** | Truy cập vài lần mỗi năm hoặc ít hơn | Thấp hơn IA đến 50% | Hàng chục ms |

EFS Archive vẫn là một lớp lưu trữ trong cùng file system; file được truy cập trực tiếp, không cần restore/rehydrate như EBS Snapshot Archive. [Các lớp lưu trữ EFS](https://aws.amazon.com/efs/storage-classes/) (đã kiểm tra 2026-11-06).

#### NÊN làm (DO):
- **Dùng EFS để chia sẻ tệp giữa nhiều instances** → EBS chỉ gắn được với một instance (ngoại trừ io1/io2 Multi-Attach - phức tạp). EFS cho phép hàng trăm instances cùng gắn qua NFS.
- **Dùng Lifecycle policies (EFS IA)** → Tự động chuyển files không được truy cập hơn 90 ngày sang lớp rẻ hơn; tiết kiệm 90% chi phí mà không cần quản lý thủ công.

#### KHÔNG NÊN làm (DON'T):
- **Không chọn EFS cho một instance nếu không cần hệ thống tệp chia sẻ** → EFS có thể tốn kém hơn block storage tùy Region, storage class và throughput/access; so sánh giá theo khối lượng công việc. Nếu cần block storage cho một host, EBS thường phù hợp hơn.
- **Dùng EFS cho yêu cầu độ trễ cực thấp** → EFS đi qua network (NFS), độ trễ cao hơn EBS hoặc Instance Store. NÊN: dùng Instance Store cho cache, EBS cho database.

---

## 4. AMI

### Tạo và quản lý AMI

#### Giải thích cốt lõi

**AMI và EBS Snapshot:**

| | AMI | EBS Snapshot |
|---|-----|-------------|
| **Là gì** | Template để khởi chạy instance | Bản sao lưu của một volume |
| **Có thể khởi chạy?** | ✅ Có | ❌ Không |
| **Bao gồm** | Cấu hình instance + mọi volumes | Chỉ một volume |
| **Trường hợp sử dụng** | Triển khai các instances giống nhau | Sao lưu dữ liệu |

**Quy trình tạo AMI:**
```
1. Khởi chạy instance → tùy chỉnh (cài phần mềm, cấu hình)
2. Dừng instance → bảo đảm tính nhất quán dữ liệu
3. Tạo Image → AWS tạo AMI + EBS snapshots
4. Khởi chạy instances từ AMI → các bản sao giống nhau
```

**AMI có thể:**
- Sao chép **qua các Regions**.
- Chia sẻ **giữa các accounts** (có permissions).
- Công khai **public** (nếu muốn).

#### NÊN làm (DO):
- **Dừng instance trước khi tạo AMI** → Bảo đảm tính toàn vẹn dữ liệu. Nếu tạo AMI khi đang có thao tác ghi, snapshot có thể không nhất quán.
- **Dùng Launch Template thay cho Launch Configuration** → Launch Template hỗ trợ versioning, có thể update mà không tạo mới và tích hợp tốt hơn với ASG và SSM.

#### KHÔNG NÊN làm (DON'T):
- **Ghi cứng credentials trong AMI** → Credentials sẽ được snapshot và có thể bị lộ. NÊN: dùng IAM Role, User Data scripts hoặc Secrets Manager để thêm credentials khi khởi chạy.

---

## 5. Cân bằng tải (Load Balancers)

### Chọn loại Load Balancer

#### Giải thích cốt lõi

**Ba loại Load Balancer chính:**

| Đặc điểm | ALB | NLB | GWLB |
|---------|-----|-----|------|
| **Lớp OSI** | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) | Layer 3 (IP) |
| **Độ trễ** | ~400ms | ~100ms | Thay đổi |
| **Thông lượng** | Trung bình | Rất cao (hàng triệu RPS) | Thay đổi |
| **Định tuyến** | Path, Host, Query | IP/Port | Packet |
| **Static IP** | ❌ | ✅ (1 cho mỗi AZ) | ✅ |
| **Kết thúc SSL** | ✅ Dễ | ✅ Phức tạp | ❌ |
| **Chi phí** | $/LCU | $/NLBCU | $/GB |

**Khả năng định tuyến của ALB:**
```
Listener
    ├── Rule: /api/* → Target Group A
    ├── Rule: /web/* → Target Group B
    ├── Rule: *.example.com → Target Group C
    └── Rule: ?user=vip → Target Group D
```

#### NÊN làm (DO):
- **Dùng ALB cho ứng dụng web (HTTP/HTTPS)** → Hỗ trợ định tuyến theo path và host để tách microservices. So với NLB, ALB cấu hình SSL termination dễ hơn và có nhiều loại routing rules hơn.
- **Dùng NLB cho ứng dụng TCP/UDP cần hiệu năng cao** → Độ trễ thấp hơn ALB (~100ms so với ~400ms), throughput cao hơn và hỗ trợ static IP để allowlist.

#### KHÔNG NÊN làm (DON'T):
- **Không chọn CLB (Classic Load Balancer) cho hệ thống mới nếu ALB/NLB phù hợp** → CLB là thế hệ cũ với ít khả năng định tuyến hơn; AWS vẫn liệt kê CLB là loại Load Balancer được hỗ trợ, nên không gọi dịch vụ này là deprecated. [So sánh Load Balancer của AWS](https://aws.amazon.com/elasticloadbalancing/features/) (đã kiểm tra 2026-11-06).
- **Dùng NLB cho ứng dụng HTTP** → NLB không hỗ trợ định tuyến theo path, SSL termination phức tạp hơn. ALB phù hợp hơn với HTTP/HTTPS.

---

### Security Groups cho Load Balancer

#### Giải thích cốt lõi

**Kiến trúc phù hợp:**

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

**Vì sao EC2 không cần public IP:**
- **Bảo mật:** chỉ LB được phơi ra Internet.
- **Chi phí:** Public IP bị tính phí từ tháng 2 năm 2024.
- **Security:** SG của EC2 chỉ cần cho phép từ LB.

#### NÊN làm (DO):
- **EC2 Security Group chỉ cho phép lưu lượng TỪ Load Balancer SG** → Khi LB mở rộng, mọi instances đều tự động được cho phép. So với cho phép theo IP, cách này không cần cập nhật rules khi IP của nhiều instances thay đổi.

#### KHÔNG NÊN làm (DON'T):
- **Để EC2 truy cập trực tiếp từ Internet (public IP + SG 0.0.0.0/0)** → Bất kỳ ai cũng có thể truy cập, vi phạm nguyên tắc bảo mật. NÊN: dùng Load Balancer và chỉ cho phép EC2 nhận lưu lượng từ LB.

---

### Cấu hình Health Checks

#### Giải thích cốt lõi

**Cơ chế health check:**

```
Load Balancer
    ↓ health checks định kỳ
Instance (port + path)
    ├── Response 200 → Healthy
    └── Response ≠ 200 → Unhealthy (Load Balancer ngừng gửi traffic mới)

Nếu ASG được cấu hình dùng ELB health checks, ASG có thể thay thế instance không khỏe; nếu không, health check tự nó chỉ đánh dấu target là unhealthy.

**Timeout và Interval:**
- **Timeout**: thời gian tối đa chờ response.
- **Interval**: tần suất gửi health check.
- **Threshold**: số lần thất bại/thành công trước khi đổi trạng thái.

**Ví dụ cấu hình:**
```yaml
HealthCheckIntervalSeconds: 30   # Gửi mỗi 30s
HealthCheckTimeoutSeconds: 5    # Chờ response 5s
HealthyThresholdCount: 2        # 2 lần success → healthy
UnhealthyThresholdCount: 3     # 3 lần fail → unhealthy
```

#### NÊN làm (DO):
- **Dùng path health check riêng (/health hoặc /status)** → Endpoint nên trả về 200 ngay và không yêu cầu xác thực. So với kiểm tra root `/`, endpoint riêng tránh phát sinh health check từ request của người dùng thật.

#### KHÔNG NÊN làm (DON'T):
- **Dùng logic ứng dụng nặng cho health check** → Nếu ứng dụng xử lý request mất 5 giây nhưng timeout chỉ 2 giây thì target bị đánh dấu unhealthy nhầm. NÊN: dùng endpoint health check đơn giản, trả về 200 ngay.

---

## 6. Auto Scaling Group (ASG)

### Các chính sách mở rộng

#### Giải thích cốt lõi

**Ba loại Scaling Policies:**

| Loại | Mô tả | Trường hợp sử dụng |
|------|-------------|----------|
| **Target Tracking** | Tự động mở rộng theo metric mục tiêu | CPU 50%, Memory 70% |
| **Simple/Step** | Mở rộng dựa trên CloudWatch alarm | Custom metrics |
| **Scheduled** | Mở rộng theo lịch | Cao điểm hằng ngày |

**Ví dụ Target Tracking:**
```yaml
ScalingPolicy:
  Type: TargetTrackingConfiguration
  TargetValue: 50.0  # Target CPU 50%
  PredefinedMetricSpecification:
    PredefinedMetricType: ASGAverageCPUUtilization
```

**Cooldown Period:**
- Áp dụng cho **Simple Scaling**: sau một hành động scale, ASG chờ hết cooldown trước khi bắt đầu hành động tiếp theo từ simple scaling policies.
- Mặc định là 300 giây; có thể cấu hình lại.
- **Target Tracking** và **Step Scaling** có thể scale-out mà không chờ default cooldown; các instance mới có thời gian warm-up riêng. ASG cũng không chờ cooldown để thay instance unhealthy.
- Nên ưu tiên Target Tracking hoặc Step Scaling hơn Simple Scaling theo hướng dẫn AWS hiện hành. [AWS scaling cooldowns](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-scaling-cooldowns.html)

#### NÊN làm (DO):
- **Dùng Target Tracking cho auto scaling** → Đơn giản, AWS tự quản lý. Chỉ cần chỉ định target value, AWS tính số instances cần thiết. So với Simple Scaling, Target Tracking điều chỉnh mượt hơn.

#### KHÔNG NÊN làm (DON'T):
- **Đặt MaxSize quá thấp** → Khi đạt MaxSize, ASG không thể scale out dù CPU/RAM cao, khiến ứng dụng quá tải. NÊN: ước tính tải đỉnh cộng 50% dự phòng.

---

### Launch Template và Launch Configuration

#### Giải thích cốt lõi

**Launch Template (được khuyến nghị):**
- ✅ Hỗ trợ **versioning**.
- ✅ Tích hợp **SSM**, **Spot Fleet**.
- ✅ Hỗ trợ **mixed instances** (On-Demand + Spot).
- ✅ Là thế hệ mới nhất của AWS.

**Launch Configuration (deprecated):**
- ❌ Không có versioning.
- ❌ Không hỗ trợ nhiều tính năng mới.
- ❌ AWS khuyến nghị dùng Launch Template.

#### NÊN làm (DO):
- **Dùng Launch Template có versioning** → Có thể rollback nếu version mới gặp vấn đề. Cập nhật hạ tầng bằng cách cập nhật version, không cần tạo mới.
#### KHÔNG NÊN làm (DON'T):
- **Không dùng Launch Configuration cho môi trường mới** → thiếu versioning và không hỗ trợ nhiều tính năng mới; thay bằng Launch Template.

---

## Tóm tắt các điểm cần nhớ

### EBS
| | Khuyến nghị | Lý do |
|--|----------------|-----|
| **Loại volume** | gp3 mặc định, io2 Block Express cho DB | gp3 rẻ và linh hoạt hơn gp2 |
| **Dung lượng** | Bắt đầu nhỏ, resize khi đang chạy | Tăng dung lượng dễ; giảm cần thao tác thủ công |
| **Encryption** | Bật theo từng Region trong account settings | Volume mới/snapshot copy mới được mã hóa khi setting Region bật; dữ liệu hiện có không đổi |

### EFS và EBS
| | Trường hợp sử dụng | Lý do |
|--|---------|-----|
| **EFS** | Lưu trữ chia sẻ, nhiều instances | Nhiều instances có thể gắn qua NFS |
| **EBS** | Một instance, cơ sở dữ liệu | Chi phí thấp hơn, độ trễ tốt hơn |

### Load Balancers
| | Trường hợp sử dụng | Lý do |
|--|---------|-----|
| **ALB** | HTTP/HTTPS, ứng dụng web | Định tuyến Layer 7, SSL termination dễ |
| **NLB** | TCP/UDP, hiệu năng cao | Độ trễ cực thấp, static IP |

### ASG
| | Khuyến nghị | Lý do |
|--|-------------|-----|
| **Policy** | Target Tracking | Đơn giản, AWS quản lý |
| **Template** | Launch Template | Versioning, tính năng hiện đại |
| **MaxSize** | Đặt đủ cao | Tránh chặn scale-out |
