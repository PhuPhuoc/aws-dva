# Section 6 - CloudFront Nâng Cao, AWS Elastic Beanstalk, AWS CloudFormation, AWS Integration & Messaging

---

## Phần 1: CloudFront Nâng Cao (Trang 351-358)

### CloudFront Geo Restriction (Trang 351)

- Hạn chế ai có thể truy cập distribution
- **Allowlist**: Chỉ users trong các approved countries được access
- **Blocklist**: Ngăn users trong các banned countries được access
- Country được xác định bằng **3rd party Geo-IP database**
- **Use case**: Luật Copyright để kiểm soát quyền truy cập nội dung

### CloudFront Signed URLs / Signed Cookies (Trang 352-353)

**Khi nào sử dụng:**
- Phân phối **nội dung trả phí** cho người dùng premium

**Policy bao gồm:**
- URL expiration - thời hạn URL
- IP ranges để truy cập
- Trusted signers - các AWS accounts có thể tạo signed URLs

**Thời hạn URL bao lâu?**
| Loại nội dung | Thời hạn |
|-------------|----------|
| Nội dung chia sẻ (phim, nhạc) | Vài phút |
| Nội dung riêng tư | Lên đến nhiều năm |

**Signed URL vs Signed Cookies:**
- **Signed URL**: Truy cập **từng file** (một URL cho một file)
- **Signed Cookies**: Truy cập **nhiều files** (một cookie cho nhiều files)

### CloudFront Field Level Encryption (Trang 354)

- Bảo vệ **thông tin nhạy cảm của người dùng**
- Thêm một lớp bảo mật bổ sung bên cạnh HTTPS
- **Mã hóa tại edge** gần người dùng
- Sử dụng **asymmetric encryption** - mã hóa bất đối xứng

**Cách sử dụng:**
- Chỉ định các trường trong POST requests muốn mã hóa (tối đa 10 trường)
- Chỉ định **public key** để mã hóa

### CloudFront Real-Time Logs (Trang 355)

- Nhận real-time requests gửi đến **Kinesis Data Streams**
- Monitor, phân tích content delivery performance
- Có thể chọn:
  - **Sampling Rate** - phần trăm requests muốn nhận
  - Specific fields và Cache Behaviors

### CloudFront - Price Classes (Trang 357-358)

**Ba price classes:**

| Class | Mô tả |
|-------|-------------|
| **Price Class All** | Tất cả regions - hiệu suất tốt nhất |
| **Price Class 200** | Hầu hết regions, loại trừ regions đắt nhất |
| **Price Class 100** | Chỉ regions ít đắt nhất |

---

## Phần 2: AWS Elastic Beanstalk (Trang 359-383)

### Developer Problems on AWS (Trang 361)

- Managing infrastructure - Quản lý hạ tầng
- Deploying Code - Triển khai code
- Configuring databases, load balancers, etc. - Cấu hình databases, load balancers
- Scaling concerns - Vấn đề về scaling
- Most web apps have same architecture (ALB + ASG) - Hầu hết web apps có cùng kiến trúc
- Developers just want **code to run!** - Developers chỉ muốn code chạy thôi!

### Elastic Beanstalk - Overview (Trang 362)

**Elastic Beanstalk là gì?**
- Cách tiếp cận lấy developer làm trung tâm để deploy ứng dụng trên AWS
- Sử dụng các components: **EC2, ASG, ELB, RDS, ...**
- **Managed service** - dịch vụ được quản lý
- Tự động xử lý:
  - Capacity provisioning - cung cấp capacity
  - Load balancing - cân bằng tải
  - Scaling
  - Application health monitoring - giám sát sức khỏe ứng dụng
  - Instance configuration - cấu hình instance
- **Miễn phí** - chỉ trả tiền cho underlying instances

### Elastic Beanstalk - Components (Trang 363)

| Component | Mô tả |
|-----------|-------------|
| **Application** | Tập hợp các thành phần của Beanstalk |
| **Application Version** | Một phiên bản lặp của ứng dụng |
| **Environment** | Tập hợp AWS resources đang chạy một app version (một version tại một thời điểm) |
| **Environment Tiers** | Web Server Environment Tier & Worker Environment Tier |

### Elastic Beanstalk - Supported Platforms (Trang 364)

- Go, Java SE, Java with Tomcat
- .NET Core on Linux, .NET on Windows Server
- Node.js, PHP, Python, Ruby
- Packer Builder
- Single Container Docker, Multi-container Docker
- Preconfigured Docker

### Web Server Tier vs. Worker Tier (Trang 365)

**Web Server Tier:**
- Xử lý HTTP requests
- Hướng đến public

**Worker Tier:**
- Xử lý các tác vụ nền
- Pulls từ SQS queue
- Tách biệt async processing

### Elastic Beanstalk Deployment Modes (Trang 367-373)

| Deployment Mode | Downtime | Chi phí thêm | Use Case |
|---------------|----------|-----------------|----------|
| **All at once** | Có | Không | Dev |
| **Rolling** | Một phần | Không | Prod |
| **Rolling with additional batches** | Không | Nhỏ | Prod |
| **Immutable** | Không | Cao | Prod |
| **Blue/Green** | Không | Trung bình | Prod |
| **Traffic Splitting** | Không | Trung bình | Canary testing |

### Beanstalk Deployment Options Details

**1. All at once (Trang 368):**
- Deploy nhanh nhất
- **Application có downtime**
- Phù hợp cho quick iterations trong development

**2. Rolling (Trang 369):**
- Update instances **từng ít một**
- App chạy **dưới capacity**
- Cả hai versions đều chạy đồng thời
- Không tốn thêm chi phí

**3. Rolling with additional batches (Trang 370):**
- App chạy **ở full capacity**
- New instances được tạo cho deployment
- Chi phí thêm nhỏ
- Phù hợp cho production

**4. Immutable (Trang 371):**
- **Không downtime**
- New instances trong **temporary ASG**
- Chi phí cao, **double capacity**
- Rollback nhanh (terminate new ASG)
- Rất tốt cho production

**5. Blue/Green (Trang 372):**
- Tạo new "stage" environment
- Deploy v2 ở đó
- Route 53 weighted policies
- **Swap URLs** khi sẵn sàng
- Không downtime

**6. Traffic Splitting (Trang 373):**
- **Canary testing**
- Một % nhỏ traffic đến version mới
- Monitor deployment health
- **Automated rollback** khi có lỗi

### Elastic Beanstalk CLI (Trang 375)

```bash
eb create          # Tạo environment
eb status          # Kiểm tra status
eb health          # Xem health
eb events          # Xem events
eb logs            # Lấy logs
eb open            # Mở app
eb deploy          # Deploy
eb config          # Cấu hình
eb terminate       # Xóa environment
```

### Elastic Beanstalk Deployment Process (Trang 376)

1. Mô tả dependencies (`requirements.txt`, `package.json`)
2. Package code thành **zip**
3. Console: upload zip → tạo version → deploy
4. CLI: tạo version → deploy

### Beanstalk Lifecycle Policy (Trang 377)

- Lưu trữ tối đa **1000 application versions**
- Lifecycle policy dựa trên:
  - **Time** (old versions được remove)
  - **Space** (khi có quá nhiều versions)

### Elastic Beanstalk Extensions (Trang 378)

**Yêu cầu:**
- `.ebextensions/` directory trong root
- **YAML/JSON format**
- `.config` extension
- `option_settings` để modify defaults

**Có thể thêm resources:**
- RDS, ElastiCache, DynamoDB, etc.

### Elastic Beanstalk Cloning (Trang 380)

- Clone environment với cùng configuration
- Hữu ích cho **test environment**
- Được preserve:
  - Load Balancer type và configuration
  - RDS database type (data không được preserve)
  - Environment variables

### RDS with Elastic Beanstalk (Trang 382)

**Dev/Test:**
- Provision RDS với Beanstalk ✓

**Production:**
- Tạo RDS **riêng biệt**
- Cung cấp connection string cho EB app
- **Đừng gắn database lifecycle với Beanstalk**

---

## Phần 3: AWS CloudFormation (Trang 386-432)

### AWS CloudFormation Overview (Trang 387)

**CloudFormation là gì?**
- Cách **khai báo** để định nghĩa AWS Infrastructure
- Infrastructure as Code - Hạ tầng như code
- Ví dụ:
  - Security group
  - Hai EC2 instances
  - Hai Elastic IPs
  - S3 bucket
  - Load Balancer

**CloudFormation tạo:**
- Resources theo **đúng thứ tự**
- Với **cấu hình chính xác** như đã khai báo

### Lợi ích của CloudFormation (Trang 389-390)

**Infrastructure as Code - Hạ tầng như code:**
- Không cần tạo resources thủ công
- Version control (Git)
- Thay đổi được review qua code

**Chi phí:**
- Tag resources để theo dõi chi phí
- Ước tính chi phí qua template
- **Tự động hóa**: Xóa lúc 5 PM, tạo lại lúc 8 AM (Dev)

**Năng suất:**
- Destroy và re-create infrastructure nhanh chóng
- Tự động tạo diagram cho templates
- **Declarative programming** - lập trình khai báo

**Separation of Concern - Phân tách trách nhiệm:**
- Nhiều stacks cho nhiều apps/layers
- Ví dụ: VPC stacks, Network stacks, App stacks

### How CloudFormation Works (Trang 391)

1. Upload template lên **S3**
2. Reference trong CloudFormation
3. Để update: **re-upload new version**
4. Stacks được identify bằng **name**
5. **Deleting stack** xóa tất cả artifacts

### CloudFormation Building Blocks (Trang 393)

**Template Components:**

| Component | Mô tả |
|-----------|-------------|
| `AWSTemplateFormatVersion` | Template capabilities |
| `Description` | Comments - ghi chú |
| `Resources` | **BẮT BUỘC** - AWS resources |
| `Parameters` | Dynamic inputs - đầu vào động |
| `Mappings` | Static variables - biến tĩnh |
| `Outputs` | References đến những gì đã tạo |
| `Conditions` | Điều kiện để tạo resources |

### CloudFormation - Resources (Trang 396)

- Core của template (**BẮT BUỘC**)
- Đại diện cho các AWS Components
- **Hơn 700 types của resources**
- Format: `service-provider::service-name::data-type-name`
- Ví dụ: `AWS::EC2::Instance`

### CloudFormation - Parameters (Trang 400-403)

**Khi nào sử dụng:**
- Tái sử dụng templates across company
- Một số inputs không thể xác định trước

**Parameter Settings - các thiết lập:**

| Setting | Mô tả |
|---------|-------------|
| `Type` | String, Number, CommaDelimitedList, AWS-Specific |
| `Default` | Giá trị mặc định |
| `AllowedValues` | Array của các giá trị cho phép |
| `AllowedPattern` | Regex pattern |
| `NoEcho` | Boolean (ẩn giá trị) |

### CloudFormation - Pseudo Parameters (Trang 405)

Có sẵn trong mọi template:
- `AWS::Region`
- `AWS::StackName`
- `AWS::StackId`
- `AWS::AccountId`
- `AWS::NoValue`

### CloudFormation - Mappings (Trang 406-408)

**Sử dụng Mappings khi:**
- Values đã biết trước
- Có thể suy ra từ: Region, AZ, Account, Environment

**vs Parameters:**

| Mappings | Parameters |
|----------|------------|
| Giá trị hardcoded | User inputs |
| Kiểm soát an toàn hơn | Linh hoạt hơn |

### CloudFormation - Outputs (Trang 409-410)

- Khai báo các output values tùy chọn
- Có thể **import vào các stacks khác**
- Hữu ích cho: **VPC ID, Subnet IDs**
- Tốt nhất cho **cross-stack collaboration**

### CloudFormation - Conditions (Trang 412-414)

**Các conditions phổ biến:**
- Environment (dev/test/prod)
- AWS Region
- Parameter values

**Intrinsic functions - các hàm:**
- `Fn::And`
- `Fn::Equals`
- `Fn::If`
- `Fn::Not`
- `Fn::Or`

### CloudFormation - Intrinsic Functions (Trang 415-421)

| Function | Mô tả |
|----------|-------------|
| `Ref` | Reference parameters/resources |
| `Fn::GetAtt` | Lấy resource attributes |
| `Fn::FindInMap` | Lấy giá trị từ mapping |
| `Fn::ImportValue` | Import từ các stacks khác |
| `Fn::Base64` | Convert sang Base64 |
| `Fn::Sub` | String substitution |
| `Fn::Join` | Nối strings |
| `Fn::Select` | Chọn từ list |
| `Fn::Cidr` | CIDR operations |

### CloudFormation - Rollbacks (Trang 422)

**Stack Creation Fails - Tạo stack thất bại:**
- Default: **mọi thứ rollback** (được xóa)
- Có tùy chọn disable rollback

**Stack Update Fails - Update stack thất bại:**
- Auto rollback về previous working state
- Có thể xem logs cho errors

**Rollback Failure - Rollback thất bại:**
- Fix resources thủ công
- Sau đó dùng `continue-update-rollback` API

### CloudFormation - Service Role (Trang 423)

- IAM role cho phép CloudFormation create/update/delete stack resources thay mặt bạn
- Cho phép users create/update/delete stacks ngay cả khi không có permissions trực tiếp
- User phải có `iam:PassRole` permissions

### CloudFormation - DeletionPolicy (Trang 424-426)

| Policy | Mô tả |
|--------|-------------|
| `Delete` (default) | Xóa resource |
| `Retain` | Giữ lại resource |
| `Snapshot` | Tạo snapshot trước khi xóa |

**Snapshot hỗ trợ:**
- EBS Volume
- ElastiCache Cluster
- RDS DBInstance/DBCluster
- Redshift Cluster

### CloudFormation - Stack Policies (Trang 427)

- JSON document định nghĩa update actions
- **Bảo vệ resources** khỏi các updates không cố ý
- Tất cả resources được bảo vệ theo mặc định
- Chỉ định explicit **ALLOW** cho những resources được phép update

### CloudFormation - Termination Protection (Trang 428)

- Ngăn chặn accidental deletes của CloudFormation Stacks

### CloudFormation - Custom Resources (Trang 429-431)

**Sử dụng cho:**
- Resources chưa được hỗ trợ bởi CloudFormation
- Custom provisioning logic cho resources bên ngoài CloudFormation
- Custom scripts chạy during create/update/delete qua Lambda functions

**Backed by:**
- **Lambda function** (phổ biến nhất)
- SNS topic

**Use case:**
- Xóa nội dung S3 bucket trước khi bucket được xóa bởi CloudFormation

### CloudFormation - StackSets (Trang 432)

- Create, update, hoặc delete stacks trên **nhiều accounts và regions** với một operation/template
- Khi update một stack set, tất cả associated stack instances được update khắp tất cả accounts và regions
- Có thể apply vào tất cả accounts của một **AWS Organization**
- Chỉ Administrator account (hoặc Delegated Administrator) có thể create StackSets

---

## Phần 4: AWS Integration & Messaging - Giới thiệu (Trang 435-440)

### Section Introduction (Trang 436-437)

**Hai patterns của application communication:**

1. **Synchronous** (trực tiếp)
2. **Asynchronous/Decoupled** (dựa trên queue)

**Vấn đề với Synchronous:**
- Có vấn đề nếu có sudden spikes - đột ngột tăng đột biến
- Ví dụ: Cần encode 1000 videos nhưng thông thường chỉ 10

**Giải pháp - Decouple applications:**
- **SQS**: Queue model - mô hình hàng đợi
- **SNS**: Pub/sub model - mô hình publish/subscribe
- **Kinesis**: Real-time streaming model - mô hình streaming thời gian thực

### Amazon SQS - Standard Queue (Trang 439-440)

**Thuộc tính:**

| Thuộc tính | Giá trị |
|-----------|-------|
| Throughput | **Unlimited** |
| Message retention | **4 ngày mặc định, tối đa 14 ngày** |
| Latency | **<10 ms** |
| Message size | **256KB** |
| Duplicates | **Có thể xảy ra** (at least once delivery) |
| Ordering | **Best effort** (có thể không đúng thứ tự) |
