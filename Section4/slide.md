# Section 4 - Amazon S3, S3 Advanced, S3 Security, Amazon VPC

---

## Phần 1: Amazon S3 (Trang 188-211)

### Amazon S3 Overview (Trang 189-191)

**S3 = Simple Storage Service**

- **Building block** chính của AWS
- Được quảng cáo là **"infinitely scaling"** storage
- **Use cases:**
  - Backup and storage
  - Disaster Recovery
  - Archive
  - Hybrid Cloud storage
  - Application hosting
  - Media hosting
  - Data lakes & big data analytics
  - Software delivery
  - Static website

**Benefits:**
- **Scalability**
- **Durability**: Most durable storage in the cloud
- **Availability**: Industry leading
- **Security**: Secure, private, encrypted by default
- **Multiple storage classes** với price performance tốt nhất
- **Resiliency, flexibility, latency, throughput**

### Amazon S3 - Buckets (Trang 192)

- S3 cho phép lưu **objects (files)** trong **"buckets" (directories)**
- Buckets phải có **globally unique name** (across all regions, all accounts)
- Buckets được định nghĩa ở **region level**
- S3 looks like a **global service** nhưng buckets được tạo trong một region

**Naming convention:**
- Không uppercase, không underscore
- 3-63 characters
- Không phải IP
- Phải bắt đầu bằng **lowercase letter hoặc number**
- Không bắt đầu bằng prefix `xn-`
- Không kết thúc bằng suffix `-s3alias`

### Amazon S3 - Objects (Trang 193)

- **Objects** có một **Key**
- Key = **FULL path:**
  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder1/another_folder/my_file.txt`
- Key composed of **prefix + object name**

**Object properties:**
- **Max Object Size**: 5TB (5000GB)
- **Metadata**: Set of name-value pairs
- **Tags**: Unicode key/value pair (up to 10)
- **Version ID**: If versioning enabled

### Amazon S3 - Security (Trang 194-195)

**User-Based:**
- **IAM Policies** - allowed cho specific user từ IAM

**Resource-Based:**
- **Bucket Policies** - bucket-wide rules (JSON)
- **Object Access Control List (ACL)** - finer grain (có thể disable)
- **Bucket Access Control List (ACL)** - less common (có thể disable)

**Quy tắc Access:**
- IAM principal có thể access S3 object nếu:
  - User IAM permissions ALLOW **OR** resource policy ALLOWS
  - **AND** there's no explicit DENY

**Encryption:** Encrypt objects sử dụng encryption keys

### S3 Bucket Policies (Trang 195)

**JSON based policies:**
- **Effect**: Allow / Deny
- **Principal**: Account hoặc user để apply policy
- **Actions**: Set of API to Allow hoặc Deny
- **Resources**: Buckets và objects

**Use cases:**
- Grant public access to bucket
- Force objects to be encrypted at upload
- Grant access to another account (Cross Account)

### Bucket settings for Block Public Access (Trang 200)

- Được tạo để **prevent company data leaks**
- Nếu bucket **không bao giờ** nên public, hãy để settings này ON
- Có thể set ở **account level**

### Amazon S3 - Static Website Hosting (Trang 201)

- S3 có thể host **static websites**
- Website accessible on the Internet
- **Nếu get 403 Forbidden error:**
  - Block all Public Access: **OFF**
  - Bucket policy allows public reads

**URL format:**
```
http://bucket-name.s3-website-aws-region.amazonaws.com
http://bucket-name.s3-website.aws-region.amazonaws.com
```

### Amazon S3 - Versioning (Trang 202)

- Có thể **version files** trong S3
- Enabled ở **bucket level**
- Same key overwrite sẽ thay đổi "version": 1, 2, 3...
- **Best practice** để version buckets

**Benefits:**
- Preserve, retrieve, and restore **every version**
- Recover objects từ **accidental deletion or overwrite**

**Notes:**
- File không versioned trước khi enable sẽ có version **"null"**
- Suspending versioning **không delete** previous versions

### Amazon S3 - Replication (CRR & SRR) (Trang 203-204)

**Requirements:**
- Must enable **Versioning** ở source và destination buckets

**Types:**
- **Cross-Region Replication (CRR)**
- **Same-Region Replication (SRR)**
- Buckets có thể ở **different AWS accounts**
- Copying là **asynchronous**
- Must give proper **IAM permissions**

**Use cases:**
- **CRR**: Compliance, lower latency access, replication across accounts
- **SRR**: Log aggregation, live replication between production và test accounts

**Important Notes:**
- Sau khi enable replication, chỉ **new objects** được replicated
- Sử dụng **S3 Batch Replication** để replicate existing objects
- **DELETE operations**:
  - Có thể replicate delete markers (optional)
  - Deletions with version ID **không replicated** (tránh malicious deletes)
- **No chaining** of replication

### S3 Storage Classes Overview (Trang 205-206)

| Storage Class | Purpose |
|---------------|---------|
| **S3 Standard** | General Purpose |
| **S3 Standard-IA** | Infrequent Access |
| **S3 One Zone-IA** | One Zone Infrequent Access |
| **S3 Glacier Instant Retrieval** | Archive, ms retrieval |
| **S3 Glacier Flexible Retrieval** | Archive, 1-12 hours retrieval |
| **S3 Glacier Deep Archive** | Long-term archive |
| **S3 Intelligent-Tiering** | Auto-tiering |

**Durability & Availability:**
- **Durability**: 99.999999999% (11 9's) - same for all classes
- **Availability**: Khác nhau tùy class

### S3 Standard - General Purpose (Trang 207)

- **99.99% Availability**
- Used for **frequently accessed data**
- **Low latency** và **high throughput**
- Sustain **2 concurrent facility failures**
- **Use Cases**: Big Data analytics, mobile & gaming apps, content distribution

### S3 Storage Classes - Infrequent Access (Trang 208)

**S3 Standard-Infrequent Access (S3 Standard-IA):**
- **99.9% Availability**
- Lower cost than S3 Standard
- **Use cases**: Disaster Recovery, backups

**S3 One Zone-Infrequent Access (S3 One Zone-IA):**
- High durability (11 9's) trong **single AZ**
- Data lost khi AZ bị destroy
- **99.5% Availability**
- **Use Cases**: Secondary backup copies, data có thể recreate

### Amazon S3 Glacier Storage Classes (Trang 209)

**Pricing:** price for storage + object retrieval cost

**S3 Glacier Instant Retrieval:**
- **Millisecond retrieval**
- Data accessed **once a quarter**
- **Minimum storage duration**: 90 days

**S3 Glacier Flexible Retrieval:**
- Expedited: **1-5 minutes** (có phí)
- Standard: **3-5 hours** (free)
- Bulk: **5-12 hours** (free)
- **Minimum storage duration**: 90 days

**S3 Glacier Deep Archive:**
- Standard: **12 hours**
- Bulk: **48 hours**
- **Minimum storage duration**: 180 days

### S3 Intelligent-Tiering (Trang 210)

- **Small monthly monitoring** và auto-tiering fee
- **No retrieval charges**
- Tự động di chuyển objects giữa các Access Tiers dựa trên usage

**Access Tiers:**
| Tier | Trigger |
|------|---------|
| Frequent Access | Default (automatic) |
| Infrequent Access | Not accessed for 30 days |
| Archive Instant Access | Not accessed for 90 days |
| Archive Access | Optional: 90-700+ days |
| Deep Archive Access | Optional: 180-700+ days |

---

## Phần 2: Amazon S3 - Advanced (Trang 212-220)

### Amazon S3 - Moving between Storage Classes (Trang 213)

- Có thể transition objects giữa các storage classes
- **Lifecycle Rules** tự động hóa transitions

### Amazon S3 - Lifecycle Rules (Trang 214-216)

**Transition Actions:**
- Configure objects để transition sang storage class khác
- Ví dụ:
  - Move objects to Standard IA **60 days** sau khi tạo
  - Move to Glacier for archiving **after 6 months**

**Expiration Actions:**
- Configure objects để expire (delete) sau some time
- Ví dụ:
  - Access log files delete after **365 days**
  - Delete old versions (nếu versioning enabled)
  - Delete incomplete Multi-Part uploads

**Rules targeting:**
- **Prefix**: `s3://mybucket/mp3/*`
- **Tags**: `Department: Finance`

### Amazon S3 Analytics - Storage Class Analysis (Trang 217)

- Help decide **when to transition** objects
- **Recommendations** cho Standard và Standard IA
- **Không work** cho One-Zone IA hoặc Glacier
- Report updated **daily**
- 24-48 hours để start seeing data

### S3 Event Notifications (Trang 218-219)

**Event types:**
- `S3:ObjectCreated`
- `S3:ObjectRemoved`
- `S3:ObjectRestore`
- `S3:Replication`

**Features:**
- Object name filtering (`*.jpg`)
- Destinations: SNS, SQS, Lambda
- Deliver trong seconds, có thể lâu hơn

### S3 Event Notifications with Amazon EventBridge (Trang 220)

**Advantages:**
- Advanced filtering với **JSON rules** (metadata, object size, name)
- Multiple Destinations (Step Functions, Kinesis)
- EventBridge Capabilities: Archive, Replay, Reliable delivery

---

## Phần 3: Amazon S3 - Performance & Security (Trang 221-243)

### S3 Baseline Performance (Trang 221)

- **Auto-scales** to high request rates
- **Latency**: 100-200 ms
- **3,500 PUT/COPY/POST/DELETE** hoặc **5,500 GET/HEAD** per second per prefix

**Example prefixes:**
```
bucket/folder1/sub1/file => /folder1/sub1/
bucket/folder1/sub2/file => /folder1/sub2/
bucket/1/file => /1/
bucket/2/file => /2/
```
→ Spread reads across 4 prefixes = **22,000 requests/second**

### S3 Performance Optimizations (Trang 222)

**Multi-Part upload:**
- Recommended cho files **> 100MB**
- Must use cho files **> 5GB**
- Parallelize uploads

**S3 Transfer Acceleration:**
- Tăng speed bằng cách transfer qua **AWS edge location**
- Forward data đến S3 bucket ở target region
- Compatible với multi-part upload

### Amazon S3 - Object Encryption (Trang 224-229)

**4 methods of encryption:**

| Type | Description | Key Management |
|------|------------|----------------|
| **SSE-S3** | Server-Side Encryption với S3-Managed Keys | AWS managed |
| **SSE-KMS** | Server-Side Encryption với KMS Keys | AWS KMS |
| **SSE-C** | Server-Side Encryption với Customer-Provided Keys | Customer managed |
| **Client-Side Encryption** | Client encrypts trước khi upload | Customer managed |

### SSE-S3 (Trang 225)

- Keys **handled, managed, owned by AWS**
- Encryption type: **AES-256**
- Header: `"x-amz-server-side-encryption": "AES256"`
- **Enabled by default** cho new buckets & objects

### SSE-KMS (Trang 226)

- Leverages **AWS KMS** để manage keys
- **Advantages**: User control + audit key usage via CloudTrail
- Header: `"x-amz-server-side-encryption": "aws:kms"`

**Limitations:**
- Count towards **KMS quota per second**
- Quotas: 5500, 10000, 30000 req/s (based on region)
- Có thể request quota increase

### SSE-C (Trang 228)

- Customer manages keys **outside of AWS**
- S3 **does NOT store** encryption key
- **HTTPS must be used**
- Key phải provided in **HTTP headers** for every request

### Client-Side Encryption (Trang 229)

- Clients encrypt **trước khi sending** to S3
- Clients decrypt **khi retrieving**
- Customer fully manages keys và encryption cycle

### Amazon S3 - Encryption in Transit (Trang 230)

**Two endpoints:**
- **HTTP Endpoint** - non encrypted
- **HTTPS Endpoint** - encryption in flight (recommended)

**Lưu ý:**
- HTTPS is **mandatory** cho SSE-C
- Most clients use HTTPS by default

### S3 - Force Encryption in Transit (Trang 231)

- Sử dụng **Bucket Policy** với `aws:SecureTransport`

### S3 - Default Encryption vs. Bucket Policies (Trang 232)

- **SSE-S3** tự động applied cho new objects
- Có thể **force encryption** bằng Bucket Policy
- **Bucket Policies evaluated BEFORE "Default Encryption"**

### What is CORS? (Trang 233-234)

- **CORS** = Cross-Origin Resource Sharing
- **Origin** = scheme + host + port
  - Ví dụ: `https://www.example.com`
- Web Browser mechanism cho phép requests đến **other origins**
- Same origin: `http://example.com/app1` & `http://example.com/app2`
- Different origins: `http://www.example.com` & `http://other.example.com`

### Amazon S3 - CORS (Trang 235)

- Enable **correct CORS headers** khi client makes cross-origin request
- Allow specific origin hoặc `*` (all origins)

### Amazon S3 - MFA Delete (Trang 236)

**MFA required for:**
- **Permanently delete** an object version
- **Suspend Versioning** on bucket

**MFA NOT required for:**
- Enable Versioning
- List deleted versions

**Requirements:**
- Versioning must be enabled
- Only **bucket owner (root account)** can enable/disable

### S3 Access Logs (Trang 237-238)

- Log all access cho **audit purpose**
- Log vào **another S3 bucket**
- Target logging bucket phải **in same AWS region**

**⚠️ Warning:**
- **Không set logging bucket là monitored bucket**
- Will create **logging loop**

### Amazon S3 - Pre-Signed URLs (Trang 239)

**Generate bằng:**
- S3 Console
- AWS CLI
- SDK

**URL Expiration:**
| Method | Expiration |
|--------|-----------|
| S3 Console | 1 min - 720 mins (12 hours) |
| AWS CLI | default 3600 secs, max 604800 secs (~168 hours) |

**Users inherit permissions** của user generated URL

**Use cases:**
- Allow logged-in users download premium video
- Allow changing list of users download files
- Allow temporarily upload to S3 bucket

### S3 Object Lock (Trang 240)

**Adopt WORM model** (Write Once Read Many)

**Modes:**

| Mode | Description |
|------|-------------|
| **Compliance** | Không ai overwrite/delete được, kể cả root |
| **Governance** | Most users can't, some có special permissions |

**Legal Hold:**
- Protect object **indefinitely**
- Can be placed/removed với `s3:PutObjectLegalHold` IAM permission

### S3 Access Points (Trang 241-242)

**Simplify security management:**
- Each Access Point có:
  - **Own DNS name**
  - **Access point policy**

**VPC Origin:**
- Accessible **only from within VPC**
- Must create **VPC Endpoint** (Gateway hoặc Interface)

### S3 Object Lambda (Trang 243)

- Use **Lambda Functions** để change object trước khi retrieve

**Use cases:**
- Redacting PII for analytics
- Converting data formats (XML → JSON)
- Resizing và watermarking

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

### Understanding CIDR - IPv4 (Trang 248-252)

**CIDR = Classless Inter-Domain Routing**

**Ví dụ:** `172.22.241.128/25`

**IP Range calculation:**
- `/24` = 256 IPs (254 usable)
- `/25` = 128 IPs (126 usable)
- `/26` = 64 IPs (62 usable)
- `/27` = 32 IPs (30 usable)
- `/28` = 16 IPs (14 usable)

### VPC in AWS - IPv4 (Trang 254)

- **VPC = Virtual Private Cloud**
- Multiple VPCs per AWS region (max 5 - soft limit)
- Max CIDR per VPC là **5**
  - Min size: **/28** (16 IPs)
  - Max size: **/16** (65536 IPs)

**Recommendation:**
- Sử dụng **private IPv4 ranges** (RFC 1918):
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`

### Default VPC (Trang 253)

- All new AWS accounts có **Default VPC**
- New EC2 instances launched vào Default VPC nếu không specify subnet
- Default VPC có **Internet connectivity** và instances có **public IPv4 addresses**

### VPC - Subnet (Trang 257)

**AWS reserves 5 IP addresses** trong mỗi subnet:
1. `x.x.x.0` - Network Address
2. `x.x.x.1` - VPC Router
3. `x.x.x.2` - Amazon DNS
4. `x.x.x.3` - Future use
5. `x.x.x.255` - Broadcast

**Exam Tip:**
- Need **29 IPs** for EC2 → Can't use `/27` (32 IPs - 5 = 27)
- Need use `/26` (64 IPs - 5 = 59)

### Internet Gateway (IGW) (Trang 258-259)

- Cho phép resources trong VPC **connect to Internet**
- Must be **created separately** từ VPC
- Một VPC chỉ attach được **một IGW**

**Lưu ý:**
- IGW alone **does not allow** Internet access
- **Route tables** must also be edited

### Editing Route Tables (Trang 260)

- Route tables định hướng traffic
- Public subnet cần route đến IGW
- Private subnet không có direct route to IGW
