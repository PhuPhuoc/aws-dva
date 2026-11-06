# Section 4 - Amazon S3 (Trang 188-244)

---

## Phần 1: Amazon S3 (Trang 188-211)

### Tổng quan Amazon S3 (Trang 189-191)

**S3 = Simple Storage Service**

- Một trong những thành phần nền tảng chính của AWS
- Được quảng bá là dịch vụ lưu trữ có khả năng **"mở rộng vô hạn"**
- **Trường hợp sử dụng:**
  - Sao lưu và lưu trữ
  - Khôi phục sau thảm họa (**Disaster Recovery**)
  - Lưu trữ dài hạn (**Archive**)
  - Lưu trữ **Hybrid Cloud**
  - Lưu trữ ứng dụng
  - Lưu trữ nội dung đa phương tiện
  - **Data Lakes** và phân tích **Big Data**
  - Phân phối phần mềm
  - Website tĩnh

**Lợi ích:**
- **Scalability** (khả năng mở rộng)
- **Durability**: độ bền lưu trữ hàng đầu trên cloud
- **Availability**: tính sẵn sàng hàng đầu ngành
- **Security**: an toàn, riêng tư và được mã hóa theo mặc định
- Nhiều **Storage Classes** với tỷ lệ hiệu năng/chi phí phù hợp cho workload và khả năng tự động quản lý vòng đời dữ liệu
- **Resiliency, flexibility, latency, throughput** (khả năng chống chịu, tính linh hoạt, độ trễ và thông lượng)

### Amazon S3 - Buckets (Trang 192)

- S3 lưu **objects (files)** trong **buckets (directories)**
- Bucket phải có tên **duy nhất trên toàn cầu** (giữa mọi Region và account)
- Bucket được định nghĩa ở cấp **Region**
- S3 trông như dịch vụ toàn cầu, nhưng mỗi bucket được tạo trong một Region cụ thể

**Quy tắc đặt tên:**
- Không dùng chữ in hoa hoặc dấu gạch dưới
- Tên dài từ 3 đến 63 ký tự
- Không dùng địa chỉ IP làm tên
- Phải bắt đầu bằng chữ thường hoặc số
- Không bắt đầu bằng tiền tố `xn--`
- Không kết thúc bằng hậu tố `-s3alias`

### Amazon S3 - Objects (Trang 193)

- Mỗi **object** có một **Key**
- Key là **đường dẫn đầy đủ**:
  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder1/another_folder/my_file.txt`
- **Key** gồm **prefix + tên object**

**Thuộc tính của object:**
- **Kích thước tối đa của object**: 5 TB (5.000 GB)
- **Metadata**: tập hợp các cặp tên-giá trị chứa thông tin về object
- **Tags**: cặp key/value Unicode, tối đa 10; hữu ích cho bảo mật và lifecycle
- **Version ID**: có khi bật Versioning

### Amazon S3 - Bảo mật (Trang 194-195)

**Dựa trên user:**
- **IAM Policies** quy định quyền cho một user cụ thể từ IAM

**Dựa trên resource:**
- **Bucket Policies** đặt quy tắc áp dụng cho toàn bucket (JSON)
- **Object ACL** kiểm soát chi tiết hơn (có thể tắt)
- **Bucket ACL** ít được dùng hơn (có thể tắt)

**Quy tắc truy cập:**
- Khi caller và resource cùng account: identity policy hoặc resource policy có thể cấp **ALLOW**, nếu không có **DENY** tường minh.
- Với truy cập **cross-account**: identity-based policy của caller account và resource-based policy của bucket owner account đều phải cho phép hành động; mọi explicit **DENY** vẫn chặn truy cập.

**Encryption:** mã hóa object trong S3 bằng encryption keys

### S3 Bucket Policies (Trang 195)

**Policy có định dạng JSON:**
- **Effect**: Allow / Deny
- **Principal**: account hoặc user được áp dụng policy
- **Actions**: tập hợp API được Allow hoặc Deny
- **Resources**: bucket và object

**Trường hợp sử dụng:**
- Cấp quyền public cho bucket
- Bắt buộc mã hóa object khi upload
- Cấp quyền truy cập cho account khác (**Cross Account**)

### Cài đặt Block Public Access cho bucket (Trang 200)

- Các cài đặt này được tạo ra để ngăn rò rỉ dữ liệu của công ty
- Nếu bucket tuyệt đối không được public, hãy bật các cài đặt này
- Có thể đặt ở cấp account

### Amazon S3 - Lưu trữ website tĩnh (Trang 201)

- S3 có thể lưu trữ website tĩnh và cung cấp quyền truy cập từ Internet
- Nếu gặp lỗi **403 Forbidden**, hãy kiểm tra bucket policy có cho phép public read không

**Định dạng URL:**
```
http://bucket-name.s3-website-aws-region.amazonaws.com
http://bucket-name.s3-website.aws-region.amazonaws.com
```

### Amazon S3 - Versioning (Trang 202)

- Có thể bật Versioning cho files trong S3
- Bật ở cấp bucket
- Khi ghi đè cùng một key, S3 tạo version mới (1, 2, 3...)
- Thực hành tốt là bật Versioning cho bucket

**Lợi ích:**
- Giữ lại, truy xuất và khôi phục mọi version của object
- Khôi phục object bị xóa nhầm hoặc ghi đè

**Ghi chú:**
- File chưa được version trước khi bật Versioning sẽ có version ID `null`
- Tạm dừng Versioning không xóa các version trước đó

### Amazon S3 - Replication (CRR & SRR) (Trang 203-204)

**Yêu cầu:**
- Phải bật Versioning ở cả bucket nguồn và bucket đích

**Các loại:**
- **Cross-Region Replication (CRR)**
- **Same-Region Replication (SRR)**
- Bucket nguồn và đích có thể thuộc các AWS account khác nhau
- Dữ liệu được sao chép bất đồng bộ
- Cần cấp đúng IAM permissions cho S3

**Trường hợp sử dụng:**
- **CRR**: tuân thủ yêu cầu, giảm độ trễ truy cập, sao chép giữa các account
- **SRR**: tập trung log, sao chép trực tiếp giữa account production và test

**Ghi chú quan trọng:**
- Sau khi bật replication, chỉ object mới được sao chép
- Dùng **S3 Batch Replication** để sao chép object đã tồn tại
- **DELETE operations:**
  - Có thể tùy chọn sao chép delete marker từ nguồn sang đích
  - Lệnh xóa kèm version ID không được sao chép (để tránh xóa ác ý)
- Không hỗ trợ nối chuỗi replication

### Tổng quan về S3 Storage Classes (Trang 205-206)

| Storage Class | Mục đích |
|---------------|---------|
| **S3 Standard** | Mục đích sử dụng chung |
| **S3 Standard-IA** | Truy cập không thường xuyên |
| **S3 One Zone-IA** | Truy cập không thường xuyên, lưu trong một Availability Zone |
| **S3 Glacier Instant Retrieval** | Lưu trữ dài hạn, truy xuất trong mili-giây |
| **S3 Glacier Flexible Retrieval** | Lưu trữ dài hạn, truy xuất trong 1-12 giờ |
| **S3 Glacier Deep Archive** | Lưu trữ dài hạn, truy xuất chậm |
| **S3 Intelligent-Tiering** | Tự động chuyển tầng lưu trữ |

**Durability và Availability:**
- **Durability**: S3 nêu mức 99.999999999% (11 chữ 9); **S3 One Zone-IA** lưu dữ liệu trong một AZ nên không chống chịu được việc mất AZ đó
- **Availability** khác nhau tùy Storage Class

### S3 Standard - Mục đích chung (Trang 207)

- Availability **99.99%**
- Dùng cho dữ liệu được truy cập thường xuyên
- Độ trễ thấp và thông lượng cao
- Chịu được hai sự cố cơ sở vật chất đồng thời
- **Trường hợp sử dụng:** phân tích Big Data, ứng dụng di động và trò chơi, phân phối nội dung

### S3 Storage Classes - Truy cập không thường xuyên (Trang 208)

**S3 Standard-Infrequent Access (S3 Standard-IA):**
- Availability **99.9%**
- Chi phí thấp hơn S3 Standard
- **Trường hợp sử dụng:** khôi phục sau thảm họa và sao lưu

**S3 One Zone-Infrequent Access (S3 One Zone-IA):**
- Độ bền cao (11 chữ 9) nhưng chỉ trong **một AZ**
- Không chống chịu được việc mất AZ; không dùng làm bản sao duy nhất cho dữ liệu quan trọng
- Availability **99.5%**
- **Trường hợp sử dụng:** bản sao lưu thứ cấp, dữ liệu có thể tạo lại


### Amazon S3 Glacier Storage Classes (Trang 209)

**Định giá:** gồm phí lưu trữ và phí truy xuất object

**S3 Glacier Instant Retrieval:**
- Truy xuất trong mili-giây
- Phù hợp với dữ liệu được truy cập mỗi quý một lần
- Thời gian lưu trữ tối thiểu: **90 ngày**

**S3 Glacier Flexible Retrieval:**
- Expedited: **1-5 phút** (có phí)
- Standard: **3-5 giờ** (miễn phí)
- Bulk: **5-12 giờ** (miễn phí)
- Thời gian lưu trữ tối thiểu: **90 ngày**

**S3 Glacier Deep Archive:**
- Standard: **12 giờ**
- Bulk: **48 giờ**
- Thời gian lưu trữ tối thiểu: **180 ngày**

### S3 Intelligent-Tiering (Trang 210)

- Có phí giám sát và tự động phân tầng hàng tháng ở mức thấp
- Không thu phí truy xuất
- Tự động di chuyển object giữa các Access Tiers dựa trên mức sử dụng

**Các Access Tier:**
| Tier | Điều kiện kích hoạt |
|------|---------|
| Frequent Access | Tầng mặc định, tự động |
| Infrequent Access | Không được truy cập trong 30 ngày |
| Archive Instant Access | Không được truy cập trong 90 ngày |
| Archive Access | Tùy chọn: 90-700+ ngày |
| Deep Archive Access | Tùy chọn: 180-700+ ngày |

---

### S3 - Bảng so sánh Storage Classes (Trang 211)

| Storage Class | Phù hợp với | AZs | Thời gian lưu trữ tối thiểu | Kích thước object tính phí tối thiểu | Lưu ý khác |
|---|---|---:|---:|---:|---|
| **S3 Standard** | Dữ liệu truy cập thường xuyên (hơn một lần mỗi tháng), truy xuất mili-giây | ≥ 3 | Không có | Không có | Phân tích Big Data, ứng dụng di động/trò chơi, phân phối nội dung |
| **S3 Standard-IA** | Dữ liệu lưu lâu, ít truy cập (mỗi tháng một lần), truy xuất mili-giây | ≥ 3 | 30 ngày | 128 KB | Có phí truy xuất theo GB; khôi phục sau thảm họa, sao lưu |
| **S3 Intelligent-Tiering** | Dữ liệu có pattern truy cập không rõ, thay đổi hoặc khó dự đoán | ≥ 3 | Không có | Không có | Phí giám sát và tự động hóa theo object; không có phí truy xuất; tự chuyển giữa các Access Tiers theo mức sử dụng |
| **S3 One Zone-IA** | Dữ liệu có thể tạo lại, ít truy cập (mỗi tháng một lần), truy xuất mili-giây | 1 | 30 ngày | 128 KB | Có phí truy xuất theo GB; không chống chịu được sự cố mất AZ; bản sao lưu dữ liệu tại chỗ |
| **S3 Glacier Instant Retrieval** | Dữ liệu lưu trữ dài hạn, truy cập mỗi quý một lần, truy xuất mili-giây | ≥ 3 | 90 ngày | 128 KB | Phí lưu trữ và truy xuất object |
| **S3 Glacier Flexible Retrieval** | Dữ liệu lưu trữ dài hạn, truy cập mỗi năm một lần; Expedited 1-5 phút, Standard 3-5 giờ, Bulk 5-12 giờ | ≥ 3 | 90 ngày | NA* | Lưu trữ dài hạn/sao lưu chi phí thấp; tính phí lưu trữ và truy xuất object |
| **S3 Glacier Deep Archive** | Dữ liệu lưu trữ dài hạn, truy cập ít hơn một lần mỗi năm; Standard 12 giờ, Bulk 48 giờ | ≥ 3 | 180 ngày | NA** | Lưu trữ dài hạn/sao lưu chi phí thấp; tính phí lưu trữ và truy xuất object |

> *Giữ nguyên ký hiệu NA* và NA** như trong bảng nguồn.*

---

## Phần 2: S3 nâng cao (Trang 212-220)

### Amazon S3 - Di chuyển giữa các Storage Classes (Trang 213)

- Có thể chuyển object giữa các Storage Classes
- **Lifecycle Rules** tự động hóa quá trình chuyển đổi

### Amazon S3 - Lifecycle Rules (Trang 214-216)

**Transition Actions:**
- Cấu hình object để chuyển sang Storage Class khác
- Ví dụ:
  - Chuyển object sang Standard-IA sau **60 ngày** kể từ khi tạo
  - Chuyển object sang Glacier để lưu trữ sau **6 tháng**

**Expiration Actions:**
- Cấu hình object để hết hạn (xóa) sau một khoảng thời gian
- Ví dụ:
  - Xóa access log sau **365 ngày**
  - Xóa version cũ (nếu đã bật Versioning)
  - Xóa các phần tải lên Multi-Part chưa hoàn tất

**Có thể nhắm tới object theo:**
- **Prefix**: `s3://mybucket/mp3/*`
- **Tags**: `Department: Finance`

### Amazon S3 Analytics - Phân tích Storage Class (Trang 217)

- Giúp quyết định khi nào nên chuyển object sang Storage Class phù hợp
- Đưa ra gợi ý cho S3 Standard và Standard-IA
- Không hỗ trợ One Zone-IA hoặc Glacier
- Báo cáo được cập nhật hằng ngày
- Cần 24-48 giờ trước khi bắt đầu có dữ liệu phân tích

### S3 Event Notifications (Trang 218-219)

**Các loại event:**
- `S3:ObjectCreated`
- `S3:ObjectRemoved`
- `S3:ObjectRestore`
- `S3:Replication`

**Tính năng:**
- Có thể lọc theo tên object (ví dụ `*.jpg`)
- Điểm đến: SNS, SQS hoặc Lambda
- Event thường được gửi trong vài giây, nhưng đôi khi có thể mất một phút hoặc lâu hơn

### S3 Event Notifications với Amazon EventBridge (Trang 220)

**Ưu điểm:**
- Lọc nâng cao bằng JSON rules (metadata, kích thước object, tên, v.v.)
- Hỗ trợ nhiều điểm đến, ví dụ Step Functions và Kinesis
- EventBridge hỗ trợ lưu trữ (Archive), phát lại (Replay) và chuyển giao tin cậy


---

## Phần 3: Hiệu năng và bảo mật Amazon S3 (Trang 221-243)

### Hiệu năng cơ bản của S3 (Trang 221)

- S3 tự mở rộng để xử lý tốc độ request cao; độ trễ 100-200 ms
- Mỗi prefix trong bucket có thể đạt ít nhất 3.500 PUT/COPY/POST/DELETE hoặc 5.500 GET/HEAD mỗi giây

**Ví dụ về prefix:**
```
bucket/folder1/sub1/file => /folder1/sub1/
bucket/folder1/sub2/file => /folder1/sub2/
bucket/1/file => /1/
bucket/2/file => /2/
```
→ Phân tán thao tác đọc đều trên 4 prefix có thể đạt **22.000 GET/HEAD request mỗi giây**

### Tối ưu hiệu năng S3 (Trang 222)

**Tải lên Multi-Part:**
- Khuyến nghị với file **> 100 MB**
- Bắt buộc với file **> 5 GB**
- Có thể tải các phần song song để tăng tốc

**S3 Transfer Acceleration:**
- Tăng tốc bằng cách gửi file đến AWS edge location, sau đó chuyển tiếp dữ liệu đến bucket S3 ở Region đích
- Tương thích với Multi-Part upload

### Amazon S3 - Mã hóa object (Trang 224-229)

**Có bốn phương pháp mã hóa:**

| Loại | Mô tả | Quản lý key |
|------|------------|----------------|
| **SSE-S3** | Mã hóa phía server bằng key do S3 quản lý | AWS quản lý |
| **SSE-KMS** | Mã hóa phía server bằng KMS key | AWS KMS |
| **SSE-C** | Mã hóa phía server bằng key do khách hàng cung cấp | Khách hàng quản lý |
| **Client-Side Encryption** | Client mã hóa trước khi tải lên | Khách hàng quản lý |

### SSE-S3 (Trang 225)

- AWS xử lý, quản lý và sở hữu key
- Thuật toán mã hóa: **AES-256**
- Header: `"x-amz-server-side-encryption": "AES256"`
- Được bật mặc định cho bucket và object mới

### SSE-KMS (Trang 226)

- Dùng **AWS KMS** để quản lý key
- Ưu điểm: người dùng kiểm soát key và kiểm toán việc sử dụng key qua CloudTrail
- Header: `"x-amz-server-side-encryption": "aws:kms"`

**Hạn chế:**
- Request được tính vào quota KMS mỗi giây
- Quota: 5.500, 10.000 hoặc 30.000 request/giây (tùy Region)
- Có thể yêu cầu tăng quota

### SSE-C (Trang 228)

- Khách hàng quản lý key bên ngoài AWS
- S3 **không lưu trữ** encryption key do khách hàng cung cấp
- **Bắt buộc dùng HTTPS**
- Phải gửi key trong HTTP headers của mỗi request

### Client-Side Encryption (Trang 229)

- Client tự mã hóa dữ liệu trước khi gửi lên S3
- Client tự giải mã dữ liệu khi tải về
- Khách hàng toàn quyền quản lý key và quy trình mã hóa

### Amazon S3 - Mã hóa khi truyền (SSL/TLS) (Trang 230)

**S3 cung cấp hai endpoint:**
- **HTTP Endpoint**: không mã hóa
- **HTTPS Endpoint**: mã hóa dữ liệu khi truyền (được khuyến nghị)

**Lưu ý:**
- HTTPS bắt buộc khi dùng SSE-C
- Hầu hết client mặc định dùng HTTPS Endpoint

### S3 - Bắt buộc mã hóa khi truyền (Trang 231)

- Dùng **Bucket Policy** với `aws:SecureTransport`

### S3 - Mã hóa mặc định và Bucket Policies (Trang 232)

- SSE-S3 tự động được áp dụng cho object mới trong bucket
- Có thể dùng Bucket Policy để buộc mã hóa và từ chối API PUT object không có encryption headers (SSE-KMS hoặc SSE-C)
- **Bucket Policies được đánh giá trước "Default Encryption"**

### CORS là gì? (Trang 233-234)

- **CORS** là viết tắt của Cross-Origin Resource Sharing (chia sẻ tài nguyên khác nguồn)
- **Origin** gồm scheme (protocol), host (domain) và port; ví dụ `https://www.example.com` (HTTPS mặc định dùng cổng 443, HTTP dùng cổng 80)
- CORS là cơ chế của trình duyệt cho phép trang web gửi request đến origin khác; origin đích phải cho phép request bằng response headers CORS (ví dụ `Access-Control-Allow-Origin`)
- Cùng origin: `http://example.com/app1` và `http://example.com/app2`; khác origin: `http://www.example.com` và `http://other.example.com`


### Amazon S3 - CORS (Trang 235)

- Khi client gửi cross-origin request đến bucket S3, cần cấu hình đúng CORS headers
- Có thể cho phép một origin cụ thể hoặc `*` (mọi origin)

### Amazon S3 - MFA Delete (Trang 236)

**Các thao tác bắt buộc MFA:**
- Xóa vĩnh viễn một version của object
- Tạm dừng Versioning trên bucket

**Các thao tác không cần MFA:**
- Bật Versioning
- Liệt kê các version đã xóa

**Yêu cầu:**
- Bucket phải bật Versioning trước
- Chỉ bucket owner (root account) mới có thể bật/tắt MFA Delete

### S3 Access Logs (Trang 237-238)

- Dùng để ghi nhận truy cập vào bucket phục vụ kiểm toán
- Log được lưu trong một S3 bucket khác
- Bucket đích phải ở cùng AWS Region

**⚠️ Cảnh báo:**
- Không đặt bucket nhận log làm chính bucket đang được giám sát
- Làm như vậy sẽ tạo vòng lặp log khiến bucket tăng dung lượng liên tục

### Amazon S3 - Pre-Signed URLs (Trang 239)

**Có thể tạo bằng:**
- S3 Console
- AWS CLI
- SDK

**Thời hạn URL:**
| Cách tạo | Thời hạn |
|--------|-----------|
| S3 Console | 1-720 phút (12 giờ) |
| AWS CLI | Mặc định 3.600 giây, tối đa 604.800 giây (khoảng 168 giờ) |

Người nhận URL dùng quyền của principal đã tạo URL; quyền truy cập còn bị giới hạn bởi thao tác/object được ký và thời hạn URL.

**Trường hợp sử dụng:**
- Cho phép user đã đăng nhập tải video premium
- Tạo URL động để danh sách user được tải file có thể thay đổi
- Cho phép tải tạm thời file lên vị trí cụ thể trong bucket S3

### S3 Object Lock (Trang 240)

**Áp dụng mô hình WORM (Write Once Read Many - ghi một lần, đọc nhiều lần):**
- Bucket phải bật Versioning
- Chặn xóa một object version trong khoảng thời gian được chỉ định

**Retention modes:**

| Chế độ | Nội dung |
|---|---|
| **Compliance** | Không ai, kể cả root, có thể ghi đè hoặc xóa object version; không thể đổi mode hay rút ngắn retention period |
| **Governance** | Hầu hết người dùng không thể ghi đè/xóa object version hoặc đổi lock settings; người có quyền đặc biệt có thể đổi retention hoặc xóa object |

- **Retention Period** bảo vệ object trong một khoảng thời gian cố định và có thể được gia hạn
- **Legal Hold** bảo vệ object vô thời hạn, độc lập với retention period; có thể đặt/gỡ bằng quyền IAM `s3:PutObjectLegalHold`

### S3 Access Points (Trang 241-242)

**Đơn giản hóa việc quản lý bảo mật:**
- Mỗi Access Point có:
  - Tên DNS riêng
  - Policy riêng cho Access Point

**VPC Origin:**
- Có thể giới hạn Access Point chỉ cho phép truy cập từ bên trong VPC
- Cần tạo **VPC Endpoint** (Gateway hoặc Interface)
- **VPC Endpoint Policy** phải cho phép truy cập đến cả bucket đích và Access Point.

### S3 Object Lambda (Trang 243)

- Dùng **Lambda Functions** để chỉnh sửa object trước khi ứng dụng gọi nhận object

**Trường hợp sử dụng:**
- Che thông tin định danh cá nhân (PII) phục vụ phân tích
- Chuyển đổi định dạng dữ liệu, ví dụ XML sang JSON
- Thay đổi kích thước và thêm watermark

### Bài thực hành: Tạo website tĩnh bằng Amazon S3 (Trang 244)



