# Section 4 - DVA Questions: Amazon S3, S3 Advanced, S3 Security

---

## 1. Amazon S3 Fundamentals

### Question 1:
A company needs to store 10 million objects in S3. What is the S3 durability rating?

```
A. 99.9% (3 9's)
B. 99.99% (4 9's)
C. 99.9999999% (7 9's)
D. 99.999999999% (11 9's)
```

### ✅ Answer: **D. 99.999999999% (11 9's)**

### 📖 Explanation (Vietnamese):

**Why D is correct:**
- S3 có durability **99.999999999% (11 9's)**; slide nêu mức này cho các storage classes
- Nhiều storage classes lưu dữ liệu trên ít nhất 3 AZ, nhưng **S3 One Zone-IA chỉ lưu trong một AZ** và không chống chịu được việc mất AZ đó

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | 99.9% là availability của S3 Standard-IA; S3 One Zone-IA có 99.5% availability |
| B | 99.99% là availability của S3 Standard |
| C | 7 9's không phải specification của S3 |

### 📝 Vocabulary Analysis:
- **Durability**: Khả năng object không bị mất theo thời gian
- **Availability**: Tỷ lệ thời gian service có thể truy cập được
- **11 9's**: 99.999999999%

---

### Question 2:
Which S3 storage class has the **LOWEST storage cost** for long-term archive data that is rarely accessed?

```
A. S3 Standard
B. S3 Glacier Deep Archive
C. S3 Glacier Flexible Retrieval
D. S3 Intelligent-Tiering
```

### ✅ Answer: **B. S3 Glacier Deep Archive**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **S3 Glacier Deep Archive** có storage cost **thấp nhất**
- Designed cho **long-term retention** (legal compliance, tape replacement)
- Minimum storage duration: **180 days**
- Retrieval: Standard (12h) hoặc Bulk (48h)

**Storage cost comparison (approximate):**
| Class | Cost (per GB/month) |
|-------|---------------------|
| S3 Standard | ~$0.023 |
| S3 Glacier Flexible | ~$0.004 |
| **S3 Glacier Deep Archive** | **~$0.00099** |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. S3 Standard | Đắt nhất, cho frequently accessed data |
| C. S3 Glacier Flexible | Rẻ hơn Standard nhưng đắt hơn Deep Archive |
| D. S3 Intelligent-Tiering | Monitoring fee, không phải lowest cost |

### 📝 Vocabulary Analysis:
- **Archive storage**: Lưu trữ dài hạn cho data không thường xuyên truy cập
- **Minimum storage duration**: Thời gian lưu trữ tối thiểu trước khi xóa
- **Retrieval time**: Thời gian để lấy lại data từ Glacier

---

### Question 3:
An application needs **millisecond retrieval** for archive data accessed **once a quarter**. Which S3 Glacier class should be used?

```
A. S3 Glacier Deep Archive
B. S3 Glacier Flexible Retrieval
C. S3 Glacier Instant Retrieval
D. S3 Standard-IA
```

### ✅ Answer: **C. S3 Glacier Instant Retrieval**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **S3 Glacier Instant Retrieval** cung cấp **millisecond retrieval**
- Perfect cho data accessed **once a quarter**
- Minimum storage duration: **90 days**
- Cost thấp hơn S3 Standard nhưng cao hơn Flexible/Deep Archive

**Retrieval time comparison:**
| Glacier Class | Retrieval Time |
|---------------|---------------|
| Instant Retrieval | Milliseconds |
| Flexible (Expedited) | 1-5 minutes |
| Flexible (Standard) | 3-5 hours |
| Flexible (Bulk) | 5-12 hours |
| Deep Archive (Standard) | 12 hours |
| Deep Archive (Bulk) | 48 hours |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Deep Archive | Retrieval 12-48 hours, không phải milliseconds |
| B. Flexible Retrieval | Expedited 1-5 phút, không phải milliseconds |
| D. Standard-IA | Không phải Glacier class |

---

## 2. S3 Lifecycle & Versioning

### Question 4:
A company wants to automatically move objects to cheaper storage after 30 days and to Glacier after 90 days. Which feature should be configured?

```
A. S3 Replication
B. S3 Lifecycle Rules
C. S3 Intelligent-Tiering
D. S3 Storage Classes Analysis
```

### ✅ Answer: **B. S3 Lifecycle Rules**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Lifecycle Rules** cho phép tự động:
  - **Transition** objects sang storage class khác
  - **Expire (delete)** objects sau một thời gian
- Có thể configure:
  - Transition to Standard-IA sau 30 days
  - Transition to Glacier sau 90 days

**Types of actions:**
- **Transition Actions**: Di chuyển sang storage class khác
- **Expiration Actions**: Xóa objects

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Replication | Sao chép objects sang bucket khác, không di chuyển |
| C. Intelligent-Tiering | Tự động di chuyển dựa trên access patterns, không dựa trên thời gian cố định |
| D. Analytics | Chỉ phân tích và đề xuất, không tự động thực hiện |

### 📝 Vocabulary Analysis:
- **Lifecycle Rules**: Quy tắc tự động quản lý object lifecycle
- **Transition**: Di chuyển sang storage class khác
- **Expiration**: Xóa object sau một thời gian

---

### Question 5:
Versioning is enabled on an S3 bucket. What happens when an object with the same key is uploaded?

```
A. The original object is replaced
B. A new version is created with a new version ID
C. The upload fails
D. The object is deleted
```

### ✅ Answer: **B. A new version is created with a new version ID**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- Khi versioning **enabled**:
  - Upload với same key tạo **new version**
  - Original version được **giữ lại**
  - Mỗi version có **unique version ID**
- Có thể:
  - Retrieve any previous version
  - Restore deleted objects (bằng cách delete delete marker)
  - Recover từ accidental overwrites

**Key points:**
- Version không được đánh số 1, 2, 3 mà dùng **version ID**
- File chưa versioned trước đó sẽ có version ID = **"null"**
- Suspending versioning **không xóa** previous versions

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Object không bị replaced, được giữ lại với version mới |
| C | Upload không fail |
| D | Object không bị delete |

---

## 3. S3 Encryption

### Question 6:
A company needs to encrypt S3 objects and **manage encryption keys outside of AWS**. Which **server-side encryption (SSE) method** uses a key the customer provides with each request and Amazon S3 does not store?

```
A. SSE-S3 (S3-Managed Keys)
B. SSE-KMS (KMS-Managed Keys)
C. SSE-C (Customer-Managed Keys)
D. Client-Side Encryption
```

### ✅ Answer: **C. SSE-C (Customer-Managed Keys)**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **SSE-C** = Server-Side Encryption với Customer-Provided Keys
- Customer **hoàn toàn quản lý keys outside of AWS**
- S3 **does NOT store** encryption key
- **HTTPS required** for all requests

**How SSE-C works:**
1. Client gửi encryption key trong HTTP headers
2. S3 encrypt data sử dụng key đó
3. S3 không lưu key - chỉ encrypt và return

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. SSE-S3 | Keys được quản lý bởi AWS S3 |
| B. SSE-KMS | Keys được quản lý trong AWS KMS |
| D. Client-Side | Encryption xảy ra trước khi upload, không phải SSE |

### 📝 Vocabulary Analysis:
- **SSE-C**: Server-Side Encryption với Customer-provided keys
- **Encryption key**: Khóa mã hóa để bảo vệ data
- **HTTPS required**: Bắt buộc sử dụng HTTPS khi dùng SSE-C

---

### Question 7:
Which S3 encryption method is **enabled by default** for all new buckets?

```
A. SSE-S3
B. SSE-KMS
C. SSE-C
D. Client-Side Encryption
```

### ✅ Answer: **A. SSE-S3**

### 📖 Explanation (Vietnamese):

**Why A is correct:**
- **SSE-S3** được **enabled by default** cho:
  - New buckets
  - New objects
- Sử dụng **AES-256** encryption
- Keys được **handled, managed, owned by AWS**
- Header: `"x-amz-server-side-encryption": "AES256"`

**Comparison:**
| Method | Key Management | Default |
|--------|---------------|---------|
| SSE-S3 | AWS S3 | ✅ Yes |
| SSE-KMS | AWS KMS | No |
| SSE-C | Customer | No |
| Client-Side | Customer | No |

---

### Question 8:
A bucket policy denies any PUT request without the `x-amz-server-side-encryption` header. What is this enforcing?

```
A. Default encryption
B. Bucket policy-based encryption enforcement
C. Object lock
D. Versioning
```

### ✅ Answer: **B. Bucket policy-based encryption enforcement**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- Bucket policy này **forces encryption** bằng cách:
  - Deny PUT requests **without** encryption header
  - Yêu cầu header `x-amz-server-side-encryption: AES256` hoặc `aws:kms`
- Đây là **enforcement** qua bucket policy

**Note:**
- **Default Encryption** là một tính năng khác - tự động encrypt new objects
- Bucket policy evaluated **BEFORE** default encryption
- Cả hai có thể dùng cùng lúc

**⚠️ IMPORTANT:**
- Default Encryption: S3 tự động encrypt
- Bucket Policy: Refuse requests không có encryption

---

## 4. S3 Security & Access

### Question 9:
An application needs to allow users to **upload files to S3 for a limited time**. What is the best solution?

```
A. IAM User with restricted permissions
B. S3 Pre-Signed URL
C. S3 Access Point
D. S3 Bucket Policy
```

### ✅ Answer: **B. S3 Pre-Signed URL**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Pre-Signed URL** cho phép:
  - Grant temporary access đến S3 object
  - User inherit permissions của người tạo URL
  - Có thể set **expiration time**
- Use cases:
  - Temporary upload permissions
  - Time-limited downloads
  - User-specific access

**Thời hạn tối đa:**
| Cách tạo | Thời hạn tối đa |
|--------|----------------|
| S3 Console | 12 giờ |
| AWS CLI | 168 giờ (7 ngày) |
| SDK | 168 giờ (7 ngày) |

URL dùng temporary credentials sẽ hết hạn khi credentials hết hạn, kể cả khi thời hạn URL được đặt dài hơn. [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. IAM User | Dài hạn, không phải temporary |
| C. Access Point | Quản lý access, không phải temporary URL |
| D. Bucket Policy | Dài hạn, không expire |

---

### Question 10:
A web application hosted on `https://example.com` needs to load images from an S3 bucket at `https://mybucket.s3.amazonaws.com`. What must be configured on the S3 bucket?

```
A. Bucket Policy
B. CORS Configuration
C. Lifecycle Rules
D. Versioning
```

### ✅ Answer: **B. CORS Configuration**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CORS** = Cross-Origin Resource Sharing
- Web browsers **block requests** từ different origins
- CORS headers cho phép requests từ allowed origins
- Cần configure trên S3 bucket để:
  - Allow requests từ `example.com`
  - Specify allowed methods (GET, PUT, etc.)

**Ví dụ cấu hình CORS:**
```json
[
  {
    "AllowedOrigins": ["https://example.com"],
    "AllowedMethods": ["GET", "PUT"],
    "AllowedHeaders": ["*"]
  }
]
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Bucket Policy | Cho phép access control, không phải CORS headers |
| C. Lifecycle Rules | Di chuyển/xóa objects tự động |
| D. Versioning | Quản lý object versions |

### 📝 Vocabulary Analysis:
- **Cross-Origin**: Request từ domain khác
- **CORS headers**: Headers cho phép cross-origin requests
- **Origin**: Protocol + host + port

---

### Question 11:
A company wants to ensure that objects cannot be deleted for 1 year. Which feature should be used?

```
A. S3 Versioning
B. S3 Object Lock with Compliance mode
C. S3 Object Lock with Governance mode
D. S3 Lifecycle Rules
```

### ✅ Answer: **B. S3 Object Lock with Compliance mode**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **S3 Object Lock** với **Compliance mode** ngăn ghi đè/xóa object version trong retention period, kể cả bởi root
- Bucket phải bật **Versioning**; phải cấu hình retention period, ví dụ một năm
- Dùng cho yêu cầu **regulatory compliance** không cho phép rút ngắn kỳ hạn bảo vệ

**Khác biệt giữa retention mode và Legal Hold:**
| Chế độ | Ghi đè/xóa object version | Thay đổi thời hạn |
|--------|---------------------------|------------------|
| **Compliance** | Không ai, kể cả root | Không thể đổi mode hoặc rút ngắn retention period |
| **Governance** | Người dùng thường không được; người có quyền đặc biệt có thể bypass | Người có quyền đặc biệt có thể đổi retention settings |

**Legal Hold** là lớp bảo vệ riêng, không phải retention mode; nó giữ object vô thời hạn cho đến khi được gỡ, độc lập với retention period.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Versioning | Không prevent deletion |
| C. Governance | Some users có thể bypass |
| D. Lifecycle Rules | Chỉ tự động xóa, không prevent deletion |

---

## 5. S3 Replication

### Question 12:
Cross-Region Replication (CRR) is enabled on an S3 bucket. What happens to **existing objects** in the bucket?

```
A. They are automatically replicated immediately
B. They are not replicated - only new objects are replicated
C. They are replicated after enabling S3 Batch Replication
D. They are deleted from source bucket
```

### ✅ Answer: **B. They are not replicated - only new objects are replicated**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- Khi enable CRR/SRR:
  - Chỉ **new objects** được replicated
  - Objects tồn tại trước khi enable **không được replicated**
- Để replicate existing objects:
  - Sử dụng **S3 Batch Replication**
  - Batch Replication also handles failed replications

**Important Notes:**
- Replication là **async** (không đồng bộ)
- **DELETE operations**:
  - Delete markers có thể replicate (optional)
  - Deletes with version ID **không replicate** (bảo vệ against malicious deletes)
- **No chaining**: A→B→C không hoạt động

---

## 6. Kịch bản ôn tập S3

### Question 13:
A company needs to host a **static website** with images stored in S3. The website should be accessible via a custom domain. What configuration is needed?

```
A. S3 bucket with public access enabled, Route 53 A record
B. S3 bucket with static website hosting, Route 53 Alias record
C. CloudFront distribution only
D. EC2 instance with S3 bucket
```

### ✅ Answer: **B. S3 bucket with static website hosting, Route 53 Alias record**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
1. Enable **Static Website Hosting** on S3 bucket
2. Set **index document** (index.html)
3. **Block Public Access**: OFF
4. **Bucket Policy**: Allow public read
5. **Route 53**: Create **Alias record** pointing to S3 website endpoint

**Configuration steps:**
```
S3 Bucket
  ├── Enable Static Website Hosting
  ├── Set index.html
  ├── Block Public Access: OFF
  └── Bucket Policy (public read)

Route 53
  └── A Record (Alias) → S3 website endpoint
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A | Cần enable Static Website Hosting, không chỉ public access |
| C | CloudFront không required cho static website, nhưng có thể improve performance |
| D | Không cần EC2 cho static website |

---

### Question 14:
An audit team needs detailed records of **object-level S3 API activity** for a specific bucket, including reads and writes. Which feature should it enable?

```
A. S3 Versioning
B. S3 server access logging
C. CloudTrail data events for the bucket
D. CloudTrail management events only
```

### ✅ Answer: **C. CloudTrail data events for the bucket**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- CloudTrail data events có thể ghi lại hoạt động API ở cấp object như `GetObject`, `PutObject` và `DeleteObject` trong bucket được chọn.
- S3 data events không được ghi mặc định và có thể phát sinh thêm phí; dùng event selectors để giới hạn tài nguyên và loại event cần ghi.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| A. S3 Versioning | Giữ lại các version của object, không ghi lại ai đã gửi từng API request ở cấp object. |
| B. S3 server access logging | Việc chuyển log theo cơ chế best-effort nên có thể bị trễ, trùng hoặc thiếu; không phải bản ghi kiểm toán đầy đủ. |
| D. CloudTrail management events only | Management events không bao gồm data events ở cấp object như `GetObject` và `PutObject`. |

**Lưu ý vận hành:** Nếu bật S3 server access logging, hãy dùng một bucket đích riêng; ghi log bucket vào chính nó có thể tạo vòng lặp log.

---

### Question 15:
A company needs to **prevent public access** to an S3 bucket. Which settings should be reviewed?

```
A. Bucket Policy only
B. Block Public Access settings only
C. Both Bucket Policy and Block Public Access settings
D. S3 Versioning
```

### ✅ Answer: **C. Both Bucket Policy and Block Public Access settings**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Block Public Access settings**:
  - Account-level hoặc bucket-level
  - Override Bucket Policies cho public access
  - 4 settings có thể configure

- **Bucket Policy**:
  - JSON policy attached to bucket
  - Có thể Allow hoặc Deny access
  - Có thể Allow public access

- **Block Public Access** có thể đặt ở cấp account, bucket, access point hoặc organization; S3 áp dụng cấu hình hạn chế nhất và từ chối request bị cấm.
- Bucket Policy và ACL có thể cấp public access; cần rà soát chúng cùng với các Block Public Access settings đang có hiệu lực.
- Để chặn public access, bật các Block Public Access settings phù hợp và kiểm tra policies/ACLs; Block Public Access không thay thế việc rà soát quyền IAM.

AWS mô tả rằng Block Public Access overrides các policy/permission public và kết hợp mức cấu hình áp dụng để chọn giới hạn nghiêm ngặt nhất. [AWS Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)

---


## Summary Table

| Topic | Key Points |
|-------|------------|
| **S3 Durability** | 99.999999999% (11 9's) |
| **S3 Storage Classes** | Standard, IA, One Zone-IA, Glacier, Intelligent-Tiering |
| **Lifecycle Rules** | Auto transition/expiration |
| **S3 Encryption** | SSE-S3 (default), SSE-KMS, SSE-C, Client-side |
| **S3 Security** | Bucket Policy, Block Public Access, CORS |
| **Pre-Signed URLs** | Temporary access |
| **Object Lock** | WORM model, Compliance/Governance |
| **Replication** | CRR/SRR, only new objects |
