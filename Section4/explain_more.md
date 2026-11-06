# Section 4 - Kiến thức mở rộng: Amazon S3

---

### Ghi chú cập nhật theo tài liệu AWS (2026-11-06)

- **Giới hạn kích thước object:** slide nguồn p.193 ghi 5 TB; tài liệu S3 hiện hành nêu tối đa **50 TB cho mỗi object**. Dùng con số mới khi trả lời câu hỏi hiện hành, nhưng giữ số liệu gốc trong phần dịch slide. [Amazon S3 objects overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingObjects.html)
- **Mã hóa mặc định:** từ 2023-01-05, SSE-S3 tự động áp dụng cho object mới được tải lên bucket mới và bucket hiện có chưa cấu hình SSE khác; object cũ không được mã hóa hồi tố và không phát sinh phí SSE-S3 riêng. [Default encryption FAQ](https://docs.aws.amazon.com/AmazonS3/latest/userguide/default-encryption-faq.html)
- **S3 server access logging:** chuyển log theo cơ chế best-effort; tính đầy đủ và thời gian giao không được bảo đảm, có thể thiếu hoặc trùng bản ghi. Không dùng nó như sổ kiểm kê đầy đủ mọi request. [Best-effort server log delivery](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html#LogDeliveryBestEffort)
- **CloudTrail data events:** không được ghi mặc định và có thể phát sinh phí bổ sung; giới hạn event/resource bằng advanced event selectors để kiểm soát phạm vi và chi phí. [Logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)


## 1. Thực hành tốt với S3

### Đặt tên S3 bucket
#### Giải thích cốt lõi
Bucket name là định danh toàn cục, không đổi được sau khi tạo; tên phải tuân theo quy tắc của S3 và nên tránh chứa dữ liệu nhạy cảm.

#### NÊN làm (DO)
- Chọn tên ngắn, mô tả rõ mục đích và duy nhất → giảm nhầm lẫn giữa bucket và tránh xung đột tên; thêm environment/Region khi cần phân biệt môi trường.
- Dùng chữ thường và ký tự hợp lệ → giảm lỗi khi tạo bucket và cải thiện tương thích với công cụ.

#### KHÔNG NÊN làm (DON'T)
- Không dùng chữ hoa, dấu gạch dưới, IP address hoặc reserved prefix/suffix → tên không hợp lệ và thao tác tạo bucket bị từ chối; dùng chữ thường, số và dấu gạch nối theo quy tắc S3.
- Không đưa thông tin nhạy cảm vào tên → bucket name xuất hiện trong URL; dùng nhãn mô tả không tiết lộ dữ liệu.

### Thiết kế S3 Object Key
#### Giải thích cốt lõi
Object key là chuỗi tên duy nhất của object trong bucket; dấu `/` chỉ tạo quy ước prefix chứ không tạo thư mục thật. Prefix giúp nhóm dữ liệu và lọc Lifecycle Rules, nhưng không thay thế quyền truy cập hay cách thiết kế lưu trữ.

#### NÊN làm (DO)
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

**Lợi ích của cấu trúc theo cấp bậc:**
- Dễ tổ chức hơn
- Dễ áp dụng Lifecycle Rules theo prefix
- Có thể cải thiện hiệu năng bằng cách phân tán request qua nhiều prefix

#### KHÔNG NÊN làm (DON'T)
```
# Xấu - tất cả objects ở root
my-bucket/user123_profile.jpg
my-bucket/sku456_thumbnail.png
```
- Không dồn object vào một tên phẳng khó phân biệt → khó lọc và áp dụng Lifecycle Rules; tổ chức key bằng prefix theo loại dữ liệu hoặc tenant.

### Chọn S3 Storage Classes
#### Giải thích cốt lõi
Storage Class cân bằng chi phí lưu trữ, thời gian truy xuất, phí truy xuất và độ sẵn sàng. S3 Standard phù hợp truy cập thường xuyên; IA giảm giá lưu trữ nhưng có phí truy xuất/minimum duration; Glacier phù hợp lưu trữ lâu hơn với độ trễ lấy dữ liệu cao hơn.

#### NÊN làm (DO)
- Chọn lớp theo tần suất truy cập, thời gian khôi phục và tổng phí → IA/Glacier có thể giảm phí lưu trữ nhưng phát sinh retrieval/minimum-duration constraints; so với Standard, chỉ chuyển khi mẫu truy cập phù hợp.

| Loại dữ liệu | Storage Class khuyến nghị |
|-----------|-------------------|
| Truy cập thường xuyên | S3 Standard |
| Truy cập ít (hàng tháng) | S3 Standard-IA |
| Dữ liệu tái tạo được, ít truy cập | S3 One Zone-IA (chỉ dùng làm bản sao phụ, không phải bản sao lưu duy nhất) |
| Lưu trữ dài hạn, truy cập mỗi quý | S3 Glacier Instant |
| Lưu trữ dài hạn | S3 Glacier Flexible/Deep |
| Chưa rõ pattern truy cập | S3 Intelligent-Tiering |
#### KHÔNG NÊN làm (DON'T)
- Không chọn lớp chỉ theo giá lưu trữ → phí truy xuất, thời gian chờ và thời hạn tối thiểu có thể làm tổng chi phí hoặc thời gian khôi phục cao hơn; ước lượng cả tần suất truy cập và yêu cầu phục hồi rồi so sánh các lớp.

**💡 Mẹo tối ưu chi phí:**
- S3 Standard-IA có phí truy xuất theo GB cao hơn
- S3 Glacier có thời gian lưu trữ tối thiểu 90 ngày và phí truy xuất
- Tính tổng chi phí lưu trữ và truy xuất trước khi chọn

### S3 Lifecycle Rules

**Cơ chế:** Lifecycle configuration có thể chuyển current versions sang Storage Class khác, hết hạn current versions và quản lý riêng noncurrent versions. Khi bật Versioning, `Expiration` cho current version thường tạo delete marker; muốn dọn dữ liệu cũ cần cấu hình action cho noncurrent versions.
So với Intelligent-Tiering, Lifecycle Rules dùng mốc thời gian do người vận hành đặt, còn Intelligent-Tiering tự chuyển tier theo mẫu truy cập và có phí giám sát.

#### NÊN làm (DO)
- Dùng Lifecycle để tự động chuyển/xóa dữ liệu theo retention và access pattern → giảm thao tác thủ công; so với Intelligent-Tiering, Lifecycle phù hợp khi mốc thời gian đã biết.

```json
{
  "Rules": [
    {
      "ID": "Transition-current-versions",
      "Status": "Enabled",
      "Filter": {"Prefix": "documents/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"}
      ]
    },
    {
      "ID": "Manage-noncurrent-versions",
      "Status": "Enabled",
      "Filter": {"Prefix": "documents/"},
      "NoncurrentVersionTransitions": [
        {"NoncurrentDays": 30, "StorageClass": "STANDARD_IA"},
        {"NoncurrentDays": 90, "StorageClass": "GLACIER"}
      ],
      "NoncurrentVersionExpiration": {"NoncurrentDays": 365}
    },
    {
      "ID": "Abort-incomplete-multipart-uploads",
      "Status": "Enabled",
      "Filter": {"Prefix": "documents/"},
      "AbortIncompleteMultipartUpload": {"DaysAfterInitiation": 7}
    }
  ]
}
```

**Các quy tắc cần phân biệt:**
- **Current version:** dùng `Transitions` và `Expiration`.
- **Noncurrent version:** dùng `NoncurrentVersionTransitions` và `NoncurrentVersionExpiration`; `Expiration` thông thường không xóa vật lý các noncurrent versions.
- Chọn mốc thời gian đáp ứng minimum storage duration và yêu cầu khôi phục dữ liệu.
- Dùng `AbortIncompleteMultipartUpload` để xóa các phần tải lên chưa hoàn tất; nếu không dọn, các part đã tải lên vẫn phát sinh phí lưu trữ.

#### KHÔNG NÊN làm (DON'T)
- Dùng `Expiration` như thể nó xóa ngay noncurrent versions trong bucket đã bật Versioning; hãy dùng `NoncurrentVersionExpiration`.
- Không chuyển object sang Glacier mà chưa tính retrieval time, minimum storage duration và phí retrieval → dùng Standard/IA hoặc Glacier Instant Retrieval khi cần truy cập nhanh hơn.

### Chiến lược S3 Replication
#### Giải thích cốt lõi
CRR sao chép bất đồng bộ sang Region khác, còn SRR sao chép trong cùng Region; replication tạo bản sao độc lập chứ không thay thế backup hoặc tự sao chép ngược các object cũ.

#### NÊN làm (DO)

- Dùng CRR cho DR hoặc giảm latency ở Region khác → dữ liệu được sao chép khỏi Region nguồn; dùng khi cần tuân thủ vị trí lưu dữ liệu hoặc phục vụ người dùng phân tán.
- Dùng SRR để tập trung log hoặc đồng bộ production/test trong cùng Region → đơn giản hóa việc chia sẻ bản sao; replication không thay thế backup.

#### KHÔNG NÊN làm (DON'T)
- Không giả định replication là backup hoặc tự bảo vệ mọi object cũ → xóa nhầm/ghi đè và object có trước có thể không được xử lý; bật Versioning và dùng S3 Batch Replication khi cần backfill.
- Không nối chuỗi replication hoặc bỏ qua IAM permissions → object có thể không đến bucket đích; cấu hình quyền S3 và thiết kế từng nguồn/đích rõ ràng.

### Thực hành tốt về S3 Versioning
#### Giải thích cốt lõi
Versioning giữ từng phiên bản object theo cùng key; ghi đè tạo version mới, còn thao tác xóa thông thường tạo delete marker. Nó hỗ trợ khôi phục nhưng làm tăng dung lượng lưu trữ vì các version cũ vẫn được tính phí.

#### NÊN làm (DO)
- Bật Versioning để khôi phục object bị ghi đè/xóa nhầm; thêm Lifecycle Rules cho noncurrent versions để giới hạn phí lưu trữ.
- Bật MFA Delete cho bucket nhạy cảm khi cần thêm lớp xác nhận cho thao tác xóa version.
- Tạm dừng Versioning không xóa các version đã có, nên kiểm tra noncurrent-version retention riêng.
#### KHÔNG NÊN làm (DON'T)
- Không bật Versioning rồi bỏ mặc noncurrent versions → chi phí tăng theo thời gian; đặt Lifecycle Rules để chuyển hoặc xóa version cũ theo yêu cầu lưu giữ.
- Không coi Versioning là biện pháp WORM → người có quyền vẫn có thể xóa version; dùng Object Lock khi cần bất biến dữ liệu.

**⚠️ LƯU Ý:**
- Versioning làm tăng chi phí storage vì mỗi version đều được tính phí
- Kết hợp với Lifecycle Rules để dọn các version cũ

---

## 2. Chi tiết về bảo mật S3

### Các lớp kiểm soát truy cập S3
#### Giải thích cốt lõi
S3 xác định quyền từ identity-based và resource-based policies cùng các ACL còn bật; explicit Deny chặn quyền Allow, còn Block Public Access ngăn các cấu hình public. Các lớp này giải quyết phạm vi quyền khác nhau nên cần kiểm tra cùng nhau.

```
┌─────────────────────────────────────────────┐
│           IAM Policies                      │
│  (Quyền của user/role/group)                │
├─────────────────────────────────────────────┤
│           Bucket Policies                   │
│  (Truy cập cấp account hoặc cross-account)  │
├─────────────────────────────────────────────┤
│           ACLs (Object & Bucket)            │
│  (Cơ chế cũ, kiểm soát chi tiết)             │
├─────────────────────────────────────────────┤
│           S3 Access Points                  │
│  (Đơn giản hóa quản lý)                      │
└─────────────────────────────────────────────┘
```

**Logic quyết định quyền truy cập:**
1. **DENY** tường minh = **DENY**
2. Trong cùng account, IAM policy hoặc resource policy **ALLOW** có thể cấp quyền; truy cập **cross-account** cần cả identity-based policy của caller và resource-based policy của bucket owner cùng **ALLOW**.
3. Mặc định (**Default**) = **Implicit DENY**
#### NÊN làm (DO)
- Dùng IAM Policies cho quyền của principal và Bucket Policies cho quyền ở bucket/cross-account → đặt quyền ở đúng phạm vi; bật Block Public Access khi không cần public để chặn truy cập nhầm thay vì chỉ dựa vào rà soát policy.

#### KHÔNG NÊN làm (DON'T)
- Không mở ACL hoặc public policy khi chưa có yêu cầu → object có thể bị lộ; ưu tiên quyền IAM/bucket hẹp và kiểm tra Block Public Access.
### Ví dụ S3 Bucket Policy
#### Giải thích cốt lõi
Bucket Policy là resource policy JSON áp dụng tại bucket; mỗi statement ghép Principal, Action, Resource và Condition để cho phép hoặc từ chối request. So với IAM Policy, nó thuận tiện cho quyền cấp bucket và truy cập cross-account, nhưng cần tránh cấp public rộng.

#### NÊN làm (DO)
- Giới hạn Principal, Action và Resource đúng nhu cầu; dùng explicit Deny cho điều kiện bắt buộc như HTTPS hoặc encryption → giảm phạm vi ảnh hưởng khi policy bị dùng sai.

#### KHÔNG NÊN làm (DON'T)
- Không dùng `Principal: "*"` với Allow rộng nếu không chủ ý public → dữ liệu có thể bị truy cập ngoài dự kiến; thêm điều kiện, giới hạn action/resource hoặc giữ Block Public Access bật.

**1. Buộc dùng HTTPS (mã hóa khi truyền):**
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

**2. Buộc dùng mã hóa SSE-KMS:**
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

**3. Truy cập cross-account:**
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

Chính sách này chỉ là phía bucket. Principal trong caller account cũng cần identity-based policy cho phép các thao tác tương ứng trên ARN của object; trong cross-account access, cả hai account policies phải Allow và explicit Deny vẫn chặn. [AWS cross-account policy evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html)

**Identity-based policy ở caller account (gắn vào IAM user/role cần truy cập):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

### So sánh các phương pháp mã hóa S3
#### Giải thích cốt lõi
SSE mã hóa object ở phía server, còn Client-Side Encryption mã hóa trước khi gửi lên S3. SSE-S3 để AWS quản lý key, SSE-KMS bổ sung kiểm soát/audit qua KMS, SSE-C yêu cầu khách hàng gửi key cho mỗi request; chọn theo mức kiểm soát và yêu cầu audit.

| Đặc điểm | SSE-S3 | SSE-KMS | SSE-C | Client-Side |
|---------|--------|---------|-------|-------------|
| Quản lý key | AWS | AWS KMS | Khách hàng | Khách hàng |
| Xoay key | Tự động | Có thể cấu hình | Thủ công | Thủ công |
| Audit trail | Hạn chế | CloudTrail | Không | Không |
| Hiệu năng | Nhanh nhất | Chậm hơn | Chậm hơn | Chậm nhất |
| Bắt buộc HTTPS | Tùy chọn | Tùy chọn | **Bắt buộc** | Tùy chọn |

#### NÊN làm (DO)
- Dùng **SSE-S3** cho đa số object → AWS tự quản lý key, phù hợp khi không cần kiểm soát KMS riêng.
- Dùng **SSE-KMS** khi cần quyền kiểm soát/audit key usage → CloudTrail ghi nhận hoạt động sử dụng KMS key.
- Dùng **SSE-C** khi bắt buộc tự quản lý key ngoài AWS và có thể gửi key an toàn qua HTTPS.
- Dùng **Client-Side Encryption** khi ứng dụng cần kiểm soát mã hóa trước khi upload và giải mã sau khi tải.

**⚠️ QUAN TRỌNG:**
- SSE-KMS có giới hạn request rate (5.500-30.000/giây)
- Yêu cầu tăng quota nếu cần
- SSE-C bắt buộc dùng HTTPS
#### KHÔNG NÊN làm (DON'T)
- Không dùng SSE-C nếu client không thể bảo vệ và gửi key an toàn trên mọi request → mất key khiến không giải mã được object; dùng SSE-KMS hoặc SSE-S3 nếu phù hợp.
- Không dùng SSE-KMS mà bỏ qua quota/cost KMS → tải cao có thể bị throttling và phát sinh phí; ước lượng request rate, chọn key/configuration và theo dõi quota.
### Cấu hình CORS cho S3
#### Giải thích cốt lõi
CORS là cấu hình response header để trình duyệt cho phép JavaScript đọc request cross-origin; nó không cấp quyền truy cập object thay IAM hoặc Bucket Policy. Khác với policy truy cập S3, CORS chỉ áp dụng cho hành vi browser.

#### NÊN làm (DO)
- Chỉ cho phép origin, method và header cần thiết → trình duyệt có thể đọc response đúng nhu cầu mà không mở cross-origin rộng hơn.

```json
[
  {
    "AllowedOrigins": ["https://myapp.example.com"],
    "AllowedMethods": ["GET", "PUT", "POST"],
    "AllowedHeaders": ["*"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

- Dùng đúng tên trường plural và giá trị dạng array (`AllowedOrigins`, `AllowedMethods`, `AllowedHeaders`, `ExposeHeaders`). CORS chỉ cho phép trình duyệt đọc response cross-origin; nó **không cấp quyền S3** thay cho IAM/Bucket Policy.
#### KHÔNG NÊN làm (DON'T)
- Không dùng CORS như cơ chế xác thực/phân quyền hoặc cho phép `*` khi cần giới hạn origin → CORS không chặn client ngoài browser và wildcard mở quyền đọc response cho mọi website; đặt origin/method tối thiểu và cấu hình IAM/Bucket Policy riêng.

**Tình huống thường gặp:**
- Frontend (CloudFront) gọi S3 để lấy tài nguyên tĩnh
- Các lời gọi API đến S3 từ origin khác
- Bật CORS trên S3 bucket

### S3 Pre-Signed URLs
#### Giải thích cốt lõi
Pre-signed URL là request được ký bằng credentials của principal tạo URL; người nhận chỉ có thể thực hiện thao tác đã ký trong thời hạn còn hiệu lực và không được vượt quyền của principal đó.

#### NÊN làm (DO)
- Tạo URL có thời hạn ngắn cho thao tác cụ thể → giảm khoảng thời gian URL bị lộ còn dùng được.
- Dùng principal least-privilege để ký URL → người nhận chỉ thừa hưởng quyền cần thiết của principal đó.
- Dùng URL GET cho tải xuống và SDK ký `put_object` cho upload → thao tác được ký khớp với mục đích.

**Ví dụ:**
```bash
# AWS CLI tạo URL cho HTTP GET
aws s3 presign s3://bucket/file.txt --expires-in 3600
```

```python
# Dùng SDK để tạo URL cho HTTP PUT
url = s3.generate_presigned_url(
    ClientMethod="put_object",
    Params={"Bucket": "bucket", "Key": "upload.txt"},
    ExpiresIn=3600,
)
```

- Lệnh `aws s3 presign` của AWS CLI tạo URL **GET** ([tài liệu AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/s3/presign.html)); dùng SDK để ký thao tác `put_object` khi cần upload
- URL chỉ thực hiện được thao tác mà principal ký có quyền; URL dùng temporary credentials hết hạn khi credentials hết hạn, kể cả khi thời hạn URL dài hơn

#### KHÔNG NÊN làm (DON'T)
- Không dùng root credentials để ký URL → rủi ro lộ URL có thể cấp quyền quá rộng; dùng IAM Role riêng với least privilege.
- Không đặt expiry dài hơn nhu cầu hoặc chia sẻ URL nhạy cảm công khai → bất kỳ ai có URL đều có thể dùng tới khi URL/credentials hết hạn; dùng thời hạn ngắn và gửi qua kênh phù hợp.

### S3 Object Lock và Versioning

Object Lock yêu cầu bật **Versioning** và bảo vệ từng object version theo mô hình **WORM (Write Once Read Many)**.

**Retention modes:**

| Chế độ | Quyền bảo vệ |
|---|---|
| **Compliance** | Không ai, kể cả root, có thể ghi đè hoặc xóa object version trong retention period; không thể đổi mode hay rút ngắn thời hạn |
| **Governance** | Người dùng thông thường không thể ghi đè/xóa object version hoặc đổi lock settings; principal có quyền đặc biệt có thể bypass theo policy |

- **Retention Period** bảo vệ object đến thời điểm xác định và có thể được gia hạn
- **Legal Hold không phải retention mode**: bảo vệ object vô thời hạn, độc lập với retention period; có thể đặt/gỡ bằng quyền IAM `s3:PutObjectLegalHold`

#### NÊN làm (DO)
- Dùng Compliance khi không được phép rút ngắn/xóa trong thời hạn → kể cả root không thể bỏ khóa; dùng Governance khi cần ngoại lệ có kiểm soát.

#### KHÔNG NÊN làm (DON'T)
- Không coi Versioning đơn lẻ là WORM hoặc nhầm Legal Hold với retention period cố định → người có quyền vẫn có thể xóa version; bật Object Lock và chọn retention/hold phù hợp.

---

## 3. Tối ưu hiệu năng S3

### Hướng dẫn hiệu năng S3
#### Giải thích cốt lõi
S3 tự mở rộng theo request rate và đạt mức hiệu năng cơ bản theo prefix; phân tán request qua nhiều prefix giúp tận dụng năng lực song song. CloudFront giảm truy cập lặp đến origin, còn Transfer Acceleration tối ưu đường upload từ xa.

#### NÊN làm (DO)
- Phân tán GET requests qua nhiều prefix khi tải cao → tăng khả năng xử lý song song theo baseline per-prefix; theo dõi request metrics trước khi thêm prefix.
- Đặt **CloudFront** trước S3 cho nội dung lặp lại → cache ở edge giảm lượt gọi origin và latency.
- Dùng **S3 Transfer Acceleration** cho upload từ xa khi đo thử cho thấy cải thiện → truyền qua edge có thể rút ngắn đường đến bucket ở Region đích.

**Hiệu năng theo thao tác:**

| Thao tác | Mức cơ bản trên mỗi prefix |
|-----------|--------------------|
| GET/HEAD | 5.500 request/giây |
| PUT/COPY/POST/DELETE | 3.500 request/giây |
#### KHÔNG NÊN làm (DON'T)
- Không tối ưu prefix trước khi đo tải thực tế → cấu trúc dư thừa làm tăng độ phức tạp; chỉ phân tán request khi cần throughput cao và theo dõi metric S3.
### Chiến lược Multi-Part Upload
#### Giải thích cốt lõi
Multi-Part Upload chia object lớn thành nhiều part có thể tải song song và chỉ hoàn tất khi ghép các part; nếu upload thất bại, các part chưa hoàn tất vẫn chiếm storage cho đến khi bị xóa hoặc lifecycle dọn.

#### NÊN làm (DO)
- Dùng multipart cho file lớn → tải part song song và retry part lỗi thay vì gửi lại toàn object; S3 yêu cầu multipart cho object trên 5 GB.
```python
# Dùng multipart cho file > 100 MB
# Bắt buộc dùng multipart cho file > 5 GB

import boto3

s3 = boto3.client('s3')

# SDK tự dùng multipart cho file lớn
s3.upload_file('largefile.zip', 'my-bucket', 'largefile.zip')
```

**Cấu hình:**
```python
config = TransferConfig(
    multipart_threshold=1024 * 1024 * 100,  # ngưỡng 100 MB
    max_concurrency=10,
    multipart_chunksize=1024 * 1024 * 50   # mỗi phần 50 MB
)
```
#### KHÔNG NÊN làm (DON'T)
- Không để multipart upload dang dở hoặc chọn part/configuration không phù hợp → phát sinh phí và có thể giảm hiệu quả upload; đặt lifecycle abort cho upload chưa hoàn tất và dùng ngưỡng/chunk size phù hợp.


### S3 Select
#### Giải thích cốt lõi
S3 Select lọc một phần dữ liệu có cấu trúc trong object bằng biểu thức SQL phía dịch vụ, giảm lượng dữ liệu trả về so với tải toàn bộ object rồi lọc tại ứng dụng.

#### NÊN làm (DO)
- Dùng S3 Select để lọc CSV/JSON/Parquet
- Chỉ truy xuất dữ liệu cần thiết
- Giảm lượng dữ liệu truyền và chi phí

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
#### KHÔNG NÊN làm (DON'T)
- Không dùng S3 Select cho format hoặc phép xử lý không được hỗ trợ → request thất bại hoặc không đáp ứng nhu cầu; kiểm tra giới hạn định dạng/cú pháp và dùng Athena hay xử lý ứng dụng khi cần truy vấn rộng.

---

## 4. Mẫu kiến trúc S3

### Website tĩnh trên S3
#### Giải thích cốt lõi
S3 Static Website Hosting phục vụ nội dung tĩnh qua website endpoint HTTP; bucket cần cho phép public read nếu truy cập trực tiếp. CloudFront có thể đứng trước để cache và cung cấp HTTPS/custom domain mà không phải công khai bucket theo cùng cách.

```
Người dùng → CloudFront → S3 Bucket (Website tĩnh)
                         ↓
                  Route 53 (Alias)
```

#### NÊN làm (DO)
- Dùng Static Website Hosting cho nội dung tĩnh, đặt index document và cấu hình DNS phù hợp; với website công khai, dùng CloudFront khi cần HTTPS, cache hoặc giảm lộ origin.
**Cấu hình:**
1. Bật Static Website Hosting trên S3
2. Đặt `index.html` làm index document
3. Bucket Policy: cho phép public read
4. Tắt Block Public Access
5. Route 53: tạo A record (Alias trỏ đến S3)
#### KHÔNG NÊN làm (DON'T)
- Không bật public access chỉ để phục vụ bucket chứa dữ liệu riêng → mọi object được phép có thể bị tải công khai; tách bucket public và private, hoặc dùng CloudFront với origin access control.

### S3 và CloudFront Distribution
#### Giải thích cốt lõi
CloudFront cache object tại edge locations, giảm lượt đọc origin và độ trễ cho client; invalidation yêu cầu edge làm mới nội dung sớm hơn TTL. So với truy cập trực tiếp S3, CDN cải thiện phân phối toàn cầu nhưng cần cân nhắc cache policy và chi phí.

#### NÊN làm (DO)
- Đặt CloudFront trước S3 cho nội dung phân phối rộng và invalidation chỉ cho object đã thay đổi → tận dụng cache, hạn chế gửi lại toàn bộ nội dung.

#### KHÔNG NÊN làm (DON'T)
- Không giả định invalidation làm mới nội dung origin hoặc vô hiệu hóa mọi cache ngoài CloudFront → người dùng có thể vẫn thấy bản cũ; cập nhật object/version và kiểm tra cache policy, TTL.

**Lợi ích:**
- Có các edge locations trên toàn cầu
- Giảm latency
- Bảo vệ khỏi DDoS
- Cache tại các edge locations

**Xóa cache (invalidation):**
```bash
# Xóa cache của một file
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/images/*.jpg"

# Xóa toàn bộ cache
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

## 5. Mẹo làm bài thi S3

### Các chủ đề S3 trọng tâm
1. Storage Classes và tối ưu chi phí
2. Lifecycle Rules
3. Mã hóa (SSE-S3, SSE-KMS, SSE-C)
4. Bucket Policies (bắt buộc HTTPS, truy cập cross-account)
5. Replication (CRR/SRR)
6. Pre-Signed URLs
7. Cấu hình CORS

### Các tình huống thường gặp trong kỳ thi
- Cần lưu trữ website tĩnh → S3 + Route 53
- Cần ngăn xóa nhầm → S3 Versioning + Object Lock
- Cần truy cập cross-account → Bucket Policy
- Cần private instance truy cập S3 → VPC Endpoint (Gateway)

