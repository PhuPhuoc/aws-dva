# Section 4 - Amazon S3, S3 Advanced, S3 Security, Amazon VPC

---

## Phần 1: Amazon S3 (Trang 188-211)

### Amazon S3 Overview (Trang 189-191)

**S3 = Simple Storage Service**

- **Building block** (viên gạch nền tảng) chính của AWS
- Được quảng cáo là storage có khả năng **"infinitely scaling"** (mở rộng vô hạn)
- **Use cases (trên thực tế):**
  - Backup và lưu trữ (Backup and storage)
  - Disaster Recovery (khôi phục thảm họa)
  - Archive (lưu trữ dài hạn)
  - Hybrid Cloud storage (lưu trữ hybrid cloud)
  - Application hosting (chủ trì ứng dụng)
  - Media hosting (chủ trì media)
  - Data lakes & big data analytics (hồ dữ liệu & phân tích big data)
  - Software delivery (phân phối phần mềm)
  - Static website (website tĩnh)

**Lợi ích (Benefits):**
- **Scalability** (tính mở rộng)
- **Durability**: Loại storage bền nhất trên cloud
- **Availability**: Dẫn đầu ngành (industry leading)
- **Security**: An toàn, riêng tư, mã hóa (encrypted) theo mặc định
- **Nhiều storage class** với hiệu suất/chi phí (price performance) tốt nhất
- **Resiliency, flexibility, latency, throughput** (độ bền, linh hoạt, độ trễ, thông lượng)

### Amazon S3 - Buckets (Trang 192)

- S3 cho phép lưu **objects (files)** trong **"buckets" (directories)**
- Buckets phải có **tên unique toàn cầu** (globally unique, across all regions, all accounts)
- Buckets được định nghĩa ở **region level**
- S3 nhìn như một **global service** (dịch vụ toàn cầu) nhưng các bucket được tạo trong một region cụ thể

**Quy tắc đặt tên (Naming convention):**
- Không dùng chữ in hoa (uppercase), không dùng dấu gạch dưới (underscore)
- 3-63 ký tự
- Không được là địa chỉ IP
- Phải bắt đầu bằng **chữ thường hoặc số** (lowercase letter hoặc number)
- Không bắt đầu bằng prefix `xn-`
- Không kết thúc bằng suffix `-s3alias`

### Amazon S3 - Objects (Trang 193)

- **Objects** có một **Key**
- Key = **FULL path:**
  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder1/another_folder/my_file.txt`
- **Key** được tạo bằng **prefix + tên object** (Key composed of prefix + object name)

**Thuộc tính Object (Object properties):**
- **Kích thước tối đa Object (Max Object Size)**: 5TB (5000GB)
- **Metadata**: Bộ các cặp name-value (Set of name-value pairs)
- **Tags**: Cặp key/value kiểu Unicode (tối đa 10)
- **Version ID**: Khi versioning được bật (If versioning enabled)

### Amazon S3 - Security (Trang 194-195)

**Dựa theo user (User-Based):**
- **IAM Policies** - cho phép (allowed) theo từng specific user từ IAM

**Dựa theo resource (Resource-Based):**
- **Bucket Policies** - rules áp dụng cho cả bucket (JSON)
- **Object Access Control List (ACL)** - granular hơn (có thể disable)
- **Bucket Access Control List (ACL)** - ít dùng hơn (có thể disable)

**Quy tắc Access:**
- IAM principal có thể access S3 object nếu:
  - User IAM permissions ALLOW **HOẶC** resource policy ALLOWS
  - **VÀ** không có explicit DENY nào

**Encryption (Mã hóa):** Mã hóa (Encrypt) các object bằng encryption keys

### S3 Bucket Policies (Trang 195)

**Policies dạng JSON (JSON based policies):**
- **Effect**: Allow / Deny
- **Principal**: Account hoặc user để apply policy
- **Actions**: Bộ các API để Allow hoặc Deny
- **Resources**: Các Buckets và objects

**Use cases (trên thực tế):**
- Cấp quyền công khai (public access) cho bucket
- Buộc objects phải được mã hóa (encrypted) khi upload
- Cấp quyền truy cập cho account khác (Cross Account)

### Bucket settings của Block Public Access (Trang 200)

- Được tạo để **chống rò rỉ dữ liệu công ty** (prevent company data leaks)
- Nếu bucket **không bao giờ** nên public, hãy để settings này ON
- Có thể set ở **account level**

### Amazon S3 - Static Website Hosting (Trang 201)

- S3 có thể host **static websites** (website tĩnh)
- Website có thể truy cập từ Internet (accessible on the Internet)
- **Nếu gặp lỗi 403 Forbidden:**
  - Block all Public Access: **OFF**
  - Bucket policy phải cho phép public read (allows public reads)

**URL format:**
```
http://bucket-name.s3-website-aws-region.amazonaws.com
http://bucket-name.s3-website.aws-region.amazonaws.com
```

### Amazon S3 - Versioning (Trang 202)

- Có thể **version files** (gán phiên bản) trong S3
- Bật (Enabled) ở **bucket level**
- Overwrite cùng 1 key sẽ thay đổi "version": 1, 2, 3...
- **Best practice** (thực hành tốt) là nên bật versioning cho các bucket

**Lợi ích (Benefits):**
- Giữ lại, truy xuất và khôi phục (Preserve, retrieve, and restore) **mọi version**
- Khôi phục (Recover) các object bị **xóa nhầm hoặc ghi đè** (accidental deletion or overwrite)

**Ghi chú (Notes):**
- File không versioned trước khi enable sẽ có version **"null"**
- Tạm dừng (Suspending) versioning **không xóa** các previous versions

### Amazon S3 - Replication (CRR & SRR) (Trang 203-204)

**Yêu cầu (Requirements):**
- Phải bật **Versioning** ở cả source lẫn destination buckets

**Các loại (Types):**
- **Cross-Region Replication (CRR)**
- **Same-Region Replication (SRR)**
- Các Buckets có thể thuộc **nhiều AWS account khác nhau**
- Copy dữ liệu theo **asynchronous** (bất đồng bộ)
- Phải cấp **IAM permissions** (quyền) đúng

**Use cases (trên thực tế):**
- **CRR**: Tuân thủ (Compliance), giảm độ trễ truy cập (lower latency), nhân bản xuyên nhiều accounts
- **SRR**: Gom log (Log aggregation), replication trực tiếp giữa production và test accounts

**Lưu ý quan trọng (Important Notes):**
- Sau khi enable replication, chỉ **new objects** (object mới) được replicated
- Dùng **S3 Batch Replication** để replicate các existing objects (object đã có sẵn)
- **DELETE operations**:
  - Có thể replicate các delete markers (tuỳ chọn)
  - Deletion có kèm version ID **không được replicate** (để tránh malicious deletes - xoá ác ý)
- **Không được chaining** (chuỗi) replication

### S3 Storage Classes Overview (Trang 205-206)

| Storage Class | Mục đích (Purpose) |
|---------------|---------|
| **S3 Standard** | Dành cho mục đích chung (General Purpose) |
| **S3 Standard-IA** | Truy cập không thường xuyên (Infrequent Access) |
| **S3 One Zone-IA** | Truy cập không thường xuyên trong 1 Availability Zone (One Zone) |
| **S3 Glacier Instant Retrieval** | Lưu trữ dài hạn, truy xuất cấp mili-giây (ms) |
| **S3 Glacier Flexible Retrieval** | Lưu trữ dài hạn, truy xuất 1-12 giờ |
| **S3 Glacier Deep Archive** | Lưu trữ dài hạn, thời gian dài (Long-term archive) |
| **S3 Intelligent-Tiering** | Tự động phân tầng (Auto-tiering) |

**Durability & Availability:**
- **Durability (Độ bền)**: 99.999999999% (11 chữ 9) - giống nhau ở tất cả các class
- **Availability (Độ sẵn sàng)**: Khác nhau tùy class

### S3 Standard - General Purpose (Trang 207)

- **99.99% Availability** (độ sẵn sàng)
- Dùng cho **data được truy cập thường xuyên** (frequently accessed data)
- **Độ trễ thấp** (Low latency) và **thông lượng cao** (high throughput)
- Chịu được **2 facility failures đồng thời** (sustain 2 concurrent facility failures)
- **Use Cases (Trên thực tế)**: Big Data analytics, mobile & gaming apps, phân phối nội dung (content distribution)

### S3 Storage Classes - Infrequent Access (Trang 208)

**S3 Standard-Infrequent Access (S3 Standard-IA):**
- **99.9% Availability** (độ sẵn sàng)
- Rẻ hơn S3 Standard (lower cost than S3 Standard)
- **Use cases (trên thực tế)**: Disaster Recovery (khôi phục thảm họa), backups (sao lưu)

**S3 One Zone-Infrequent Access (S3 One Zone-IA):**
- Độ bền cao (11 chữ 9) nhưng chỉ trong **một AZ** (single AZ)
- Mất data khi AZ bị hỏng (data lost if the AZ is destroyed)
- **99.5% Availability** (độ sẵn sàng)
- **Use Cases (Trên thực tế)**: Bản sao backup thứ cấp (Secondary backup copies), data có thể tạo lại (recreate được)

### Amazon S3 Glacier Storage Classes (Trang 209)

**Giá (Pricing):** phí lưu trữ + phí truy xuất object (object retrieval cost)

**S3 Glacier Instant Retrieval:**
- **Truy xuất cấp mili-giây (Millisecond retrieval)**
- Data được truy cập **một lần/quý** (once a quarter)
- **Thời gian lưu trữ tối thiểu (Minimum storage duration)**: 90 ngày

**S3 Glacier Flexible Retrieval:**
- Expedited (nhanh): **1-5 phút** (có phí)
- Standard (chuẩn): **3-5 giờ** (miễn phí)
- Bulk (khối lượng lớn): **5-12 giờ** (miễn phí)
- **Thời gian lưu trữ tối thiểu (Minimum storage duration)**: 90 ngày

**S3 Glacier Deep Archive:**
- Standard (chuẩn): **12 giờ**
- Bulk (khối lượng lớn): **48 giờ**
- **Thời gian lưu trữ tối thiểu (Minimum storage duration)**: 180 ngày

### S3 Intelligent-Tiering (Trang 210)

- Có phí giám sát hàng tháng nhỏ (small monthly monitoring) và phí auto-tiering
- **Không thu phí truy xuất** (No retrieval charges)
- Tự động di chuyển objects giữa các Access Tiers dựa trên mức sử dụng (usage)

**Các Access Tier (Phân tầng truy cập):**
| Tier | Điều kiện kích hoạt (Trigger) |
|------|---------|
| Frequent Access (thường xuyên) | Mặc định (tự động) |
| Infrequent Access (hiếm khi) | 30 ngày không bị truy cập |
| Archive Instant Access | 90 ngày không bị truy cập |
| Archive Access | Tùy chọn: 90-700+ ngày |
| Deep Archive Access | Tùy chọn: 180-700+ ngày |

---

## Phần 2: Amazon S3 - Advanced (Trang 212-220)

### Amazon S3 - Moving between Storage Classes (Trang 213)

- Có thể transition objects giữa các storage classes
- **Lifecycle Rules** tự động hóa transitions

### Amazon S3 - Lifecycle Rules (Trang 214-216)

**Transition Actions (Thao tác chuyển đổi):**
- Cấu hình (Configure) objects để chuyển (transition) sang storage class khác
- Ví dụ:
  - Chuyển objects sang Standard IA **60 ngày** sau khi tạo
  - Chuyển sang Glacier để lưu trữ (archiving) **sau 6 tháng**

**Expiration Actions (Thao tác hết hạn):**
- Cấu hình (Configure) objects để hết hạn (expire - xóa) sau một thời gian
- Ví dụ:
  - Xóa file access log sau **365 ngày**
  - Xóa các version cũ (nếu versioning đang bật)
  - Xóa các upload Multi-Part chưa hoàn tất (incomplete)

**Các rule nhắm tới (Rules targeting):**
- **Prefix**: `s3://mybucket/mp3/*`
- **Tags**: `Department: Finance`

### Amazon S3 Analytics - Storage Class Analysis (Trang 217)

- Giúp quyết định **khi nào nên chuyển đổi (transition)** các object
- Có **gợi ý (Recommendations)** cho S3 Standard và Standard-IA
- **Không hoạt động** cho One-Zone IA hoặc Glacier
- Báo cáo được cập nhật **hàng ngày (daily)**
- Cần 24-48 giờ để bắt đầu có dữ liệu (24-48 hours to start seeing data)

### S3 Event Notifications (Trang 218-219)

**Các loại event (Event types):**
- `S3:ObjectCreated`
- `S3:ObjectRemoved`
- `S3:ObjectRestore`
- `S3:Replication`

**Tính năng (Features):**
- Lọc theo tên object (Object name filtering) (`*.jpg`)
- Điểm đến (Destinations): SNS, SQS, Lambda
- Giao trong vòng vài giây, có thể lâu hơn (Deliver in seconds)

### S3 Event Notifications with Amazon EventBridge (Trang 220)

**Lợi thế (Advantages):**
- Lọc nâng cao (Advanced filtering) với **JSON rules** (metadata, kích thước object, tên)
- Nhiều điểm đến (Multiple Destinations): Step Functions, Kinesis
- Khả năng của EventBridge: lưu trữ (Archive), phát lại (Replay), giao tin cậy (Reliable delivery)

---

## Phần 3: Amazon S3 - Performance & Security (Trang 221-243)

### S3 Baseline Performance (Trang 221)

- **Tự động mở rộng (Auto-scales)** theo lượng request lớn
- **Độ trễ (Latency)**: 100-200 ms
- **Mỗi prefix: 3,500 PUT/COPY/POST/DELETE** hoặc **5,500 GET/HEAD** mỗi giây

**Ví dụ prefix (Example prefixes):**
```
bucket/folder1/sub1/file => /folder1/sub1/
bucket/folder1/sub2/file => /folder1/sub2/
bucket/1/file => /1/
bucket/2/file => /2/
```
→ Phân tán (spread) các đọc (reads) trên 4 prefixes = **22,000 request/giây**

### S3 Performance Optimizations (Trang 222)

**Multi-Part upload:**
- Khuyến nghị (Recommended) dùng cho files **> 100MB**
- Bắt buộc (Must use) cho files **> 5GB**
- Chạy song song (Parallelize) các upload

**S3 Transfer Acceleration:**
- Tăng tốc bằng cách transfer qua **điểm edge của AWS (AWS edge location)**
- Chuyển tiếp (forward) data tới S3 bucket ở target region
- Tương thích (compatible) với multi-part upload

### Amazon S3 - Object Encryption (Trang 224-229)

**4 phương pháp mã hóa (4 methods of encryption):**

| Loại (Type) | Mô tả (Description) | Quản lý Key (Key Management) |
|------|------------|----------------|
| **SSE-S3** | Mã hóa phía server (Server-Side Encryption) với key do S3 quản lý | AWS quản lý |
| **SSE-KMS** | Mã hóa phía server (Server-Side Encryption) với KMS Key | AWS KMS |
| **SSE-C** | Mã hóa phía server (Server-Side Encryption) với key khách hàng cung cấp | Khách hàng quản lý |
| **Client-Side Encryption** | Client tự mã hóa trước khi upload | Khách hàng quản lý |

### SSE-S3 (Trang 225)

- Key do **AWS xử lý, quản lý và sở hữu** (handled, managed, owned by AWS)
- Loại mã hóa: **AES-256**
- Header: `"x-amz-server-side-encryption": "AES256"`
- **Bật theo mặc định (Enabled by default)** cho các bucket & object mới

### SSE-KMS (Trang 226)

- Sử dụng (Leverage) **AWS KMS** để quản lý key
- **Lợi thế (Advantages)**: Người dùng tự kiểm soát + theo dõi (audit) việc dùng key qua CloudTrail
- Header: `"x-amz-server-side-encryption": "aws:kms"`

**Hạn chế (Limitations):**
- Được tính vào **quota KMS trên giây (KMS quota per second)**
- Quota: 5500, 10000, 30000 req/s (tùy region)
- Có thể yêu cầu tăng quota (request quota increase)

### SSE-C (Trang 228)

- Khách hàng (Customer) quản lý keys **bên ngoài AWS** (outside of AWS)
- S3 **KHÔNG lưu trữ (does NOT store)** encryption key
- **PHẢI DÙNG HTTPS (HTTPS must be used)**
- Key phải đưa vào (provided in) **HTTP headers** cho mọi request

### Client-Side Encryption (Trang 229)

- Client tự mã hóa (encrypt) **TRƯỚC KHI GỬI** (sending) lên S3
- Client tự giải mã (decrypt) **KHI LẤY VỀ** (retrieving)
- Khách hàng (Customer) toàn quyền quản lý keys và vòng đời mã hóa (encryption cycle)

### Amazon S3 - Encryption in Transit (Trang 230)

**Hai endpoint (Two endpoints):**
- **HTTP Endpoint** - không mã hóa (non encrypted)
- **HTTPS Endpoint** - mã hóa khi truyền (encryption in flight) (khuyến nghị / recommended)

**Lưu ý:**
- HTTPS là **bắt buộc (mandatory)** với SSE-C
- Đa số client dùng HTTPS theo mặc định (Most clients use HTTPS by default)

### S3 - Force Encryption in Transit (Trang 231)

- Sử dụng **Bucket Policy** với `aws:SecureTransport`

### S3 - Default Encryption vs. Bucket Policies (Trang 232)

- **SSE-S3** được tự động áp dụng (applied) cho object mới (new objects)
- Có thể **ép mã hóa (force encryption)** bằng Bucket Policy
- **Bucket Policies được đánh giá TRƯỚC "Default Encryption"** (evaluated BEFORE)

### CORS là gì? (What is CORS? - Trang 233-234)

- **CORS** = chia sẻ tài nguyên khác nguồn (Cross-Origin Resource Sharing)
- **Origin** = scheme + host + port
  - Ví dụ: `https://www.example.com`
- Cơ chế (mechanism) của trình duyệt web cho phép request tới **các origin khác (other origins)**
- Cùng origin (Same origin): `http://example.com/app1` & `http://example.com/app2`
- Khác origin (Different origins): `http://www.example.com` & `http://other.example.com`

### Amazon S3 - CORS (Trang 235)

- Bật **correct CORS headers** khi client thực hiện (makes) request khác nguồn (cross-origin)
- Cho phép (Allow) origin cụ thể hoặc `*` (tất cả origin / all origins)

### Amazon S3 - MFA Delete (Trang 236)

**Các thao tác BẮT BUỘC MFA (MFA required for):**
- **Xóa vĩnh viễn** (permanently delete) một version object
- **Tạm dừng Versioning (Suspend Versioning)** trên bucket

**Các thao tác KHÔNG cần MFA (MFA NOT required for):**
- Bật Versioning (Enable Versioning)
- Liệt kê (List) các version đã xóa

**Yêu cầu (Requirements):**
- Phải bật (enable) Versioning trước
- Chỉ **bucket owner (root account)** mới bật/tắt (enable/disable) được

### S3 Access Logs (Trang 237-238)

- Ghi log (Log) mọi truy cập (all access) để **mục đích kiểm toán (audit purpose)**
- Log vào **một S3 bucket khác (another S3 bucket)**
- Bucket nhận log (Target logging bucket) phải **ở cùng AWS region (in same AWS region)**

**⚠️ Cảnh báo (Warning):**
- **KHÔNG NÊN đặt logging bucket chính là bucket cần giám sát** (monitored bucket)
- Sẽ tạo **vòng lặp log (logging loop)**

### Amazon S3 - Pre-Signed URLs (Trang 239)

**Generate bằng:**
- S3 Console
- AWS CLI
- SDK

**Thời gian hết hạn URL (URL Expiration):**
| Cách tạo (Method) | Thời gian (Expiration) |
|--------|-----------|
| S3 Console | 1 phút - 720 phút (12 giờ) |
| AWS CLI | mặc định 3600 giây, tối đa 604800 giây (~168 giờ) |

**Người dùng kế thừa (inherit) quyền (permissions)** của user tạo ra URL (generated URL)

**Use cases (trên thực tế):**
- Cho phép (Allow) user đã đăng nhập tải xuống (download) video premium
- Cho phép (Allow) đổi danh sách user được tải file (download files)
- Cho phép (Allow) tải lên (upload) tạm thời (temporarily) vào S3 bucket

### S3 Object Lock (Trang 240)

**Áp dụng mô hình WORM (Adopt WORM model)** (Write Once Read Many - ghi một lần, đọc nhiều lần)

**Các chế độ (Modes):**

| Chế độ (Mode) | Mô tả (Description) |
|------|-------------|
| **Compliance (Tuân thủ)** | Không ai overwrite/delete được, kể cả root |
| **Governance (Quản trị)** | Đa số user không được (can't), một số có quyền đặc biệt (special permissions) |

**Legal Hold (Phong toả pháp lý):**
- Bảo vệ (Protect) object **vĩnh viễn (indefinitely)**
- Có thể đặt/bỏ (placed/removed) với quyền IAM `s3:PutObjectLegalHold`

### S3 Access Points (Trang 241-242)

**Đơn giản hoá quản lý bảo mật (Simplify security management):**
- Mỗi Access Point có (Each Access Point has):
  - **Tên DNS riêng (Own DNS name)**
  - **Policy cho access point (Access point policy)**

**Nguồn từ VPC (VPC Origin):**
- Chỉ truy cập được **từ bên trong VPC (only from within VPC)**
- Phải tạo **VPC Endpoint** (Gateway hoặc Interface)

### S3 Object Lambda (Trang 243)

- Dùng **Lambda Functions** để chỉnh sửa (change) object trước khi truy xuất (retrieve)

**Use cases (trên thực tế):**
- Che thông tin định danh cá nhân (Redacting PII) phục vụ phân tích
- Chuyển đổi định dạng dữ liệu (Converting data formats) (XML → JSON)
- Thu nhỏ và thêm dấu mờ (Resizing và watermarking)

---

## Phần 4: Amazon VPC - Virtual Private Cloud (Trang 246-260+)

### VPC Overview (Trang 246-247)

**VPC Components Diagram:**
```
Internet → Internet Gateway
         → Route Tables
         → NACL
         → Security Groups
         
Public Subnet ←→ NAT Gateway
Private Subnet ← EC2 Instances

VPC Peering Connection
VPC Endpoints (S3, DynamoDB)
VPC Flow Logs → CloudWatch
```

### Hiểu về CIDR - IPv4 (Understanding CIDR - IPv4, Trang 248-252)

**CIDR = Classless Inter-Domain Routing** (tuyến tính liên miền không phân lớp)

**Ví dụ:** `172.22.241.128/25`

**Cách tính dải IP (IP Range calculation):**
- `/24` = 256 IPs (254 usable)
- `/25` = 128 IPs (126 usable)
- `/26` = 64 IPs (62 usable)
- `/27` = 32 IPs (30 usable)
- `/28` = 16 IPs (14 usable)

### VPC trong AWS - IPv4 (Trang 254)

- **VPC = Virtual Private Cloud** (đám mây riêng ảo)
- Mỗi AWS region có nhiều VPC (Multiple VPCs per region) (tối đa 5 - soft limit, giới hạn mềm)
- Số CIDR tối đa (Max CIDR) cho mỗi VPC là **5**
  - Kích thước nhỏ nhất (Min size): **/28** (16 IP)
  - Kích thước lớn nhất (Max size): **/16** (65536 IP)

**Khuyến nghị (Recommendation):**
- Dùng **các dải IPv4 riêng tư (private IPv4 ranges)** (RFC 1918):
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`

### VPC Mặc định (Default VPC, Trang 253)

- Mọi (All) AWS account mới đều có **Default VPC**
- EC2 instances mới khởi tạo (launched) vào Default VPC nếu không chỉ định (specify) subnet
- Default VPC có **kết nối Internet (Internet connectivity)** và instances có **địa chỉ IPv4 công cộng (public IPv4 addresses)**

### VPC - Subnet (Trang 257)

**AWS giữ lại (reserves) 5 địa chỉ IP** trong mỗi subnet:
1. `x.x.x.0` - Địa chỉ mạng (Network Address)
2. `x.x.x.1` - Bộ định tuyến VPC (VPC Router)
3. `x.x.x.2` - DNS của Amazon
4. `x.x.x.3` - Dành cho tương lai (Future use)
5. `x.x.x.255` - Trùng phát (Broadcast)

**Mẹo làm bài (Exam Tip):**
- Cần **29 IP** cho EC2 → Không dùng được `/27` (32 IP - 5 = 27)
- Cần dùng `/26` (64 IP - 5 = 59)

### Internet Gateway (IGW) (Trang 258-259)

- Cho phép các resources trong VPC **kết nối Internet (connect to Internet)**
- Phải **tạo riêng (created separately)** so với VPC
- Một VPC chỉ gắn (attach) được **một IGW**

**Lưu ý:**
- IGW đơn lẻ **KHÔNG cho phép** (does not allow) truy cập Internet
- **Route tables** (bảng định tuyến) cũng phải được sửa (edited)

### Chỉnh sửa Route Tables (Editing Route Tables, Trang 260)

- Route tables (bảng định tuyến) định hướng traffic (lưu lượng)
- Public subnet (subnet công cộng) cần một route (đường dẫn) đến IGW
- Private subnet (subnet riêng tư) KHÔNG có route trực tiếp (direct route) tới IGW
