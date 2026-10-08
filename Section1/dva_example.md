# Câu hỏi ôn tập DVA-C02 - Section 1

> *Các câu hỏi mẫu từ đề thi AWS Developer Associate. Câu hỏi bằng tiếng Anh, giải thích bằng tiếng Việt.*

---

## IAM Policies - Explicit Deny

### Question:

A **developer** has been created as an IAM user and added to the **Developers group**. The group has a policy that allows `s3:GetObject` on the `app-data` bucket. However, the developer cannot download files from S3. The IAM user also has a **deny policy** attached directly that denies `s3:*` on the resource `arn:aws:s3:::app-data/*`. Why is the access denied?

```
Developers Group Policy:
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::app-data/*"
}

User Policy (directly attached):
{
  "Effect": "Deny",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::app-data/*"
}
```

**A)** A Deny policy always has higher priority than an Allow policy
**B)** User policy overrides group policy
**C)** The policy JSON has a syntax error
**D)** `s3:ListBucket` permission needs to be added

### ✅ Answer: **A**

### 📖 Giải thích:

**Tại sao chọn A:**

Trong AWS IAM, **Explicit Deny** ln override (ghi đè) **Explicit Allow**. Khi có cả Allow và Deny trên cùng resource/action → **Deny thắng**.

AWS evaluates policies theo thứ tự:
1. Check explicit Deny → **DENY** (dừng lại, không xét tiếp)
2. Check explicit Allow → ALLOW
3. Default → DENY (implicit deny)

Trong trường hợp này:
- Developers Group: Allow `s3:GetObject`
- User Policy: Deny `s3:*` (bao gồm cả GetObject)

→ Explicit Deny → **Access denied**

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **B** | User policy không override group policy. AWS kết hợp tất cả policies, nhưng Deny thắng. Khái niệm này gọi là **policy aggregation** |
| **C** | JSON syntax không có lỗi - cú pháp hoàn toàn đúng |
| **D** | `s3:GetObject` là action để download object. Không cần `ListBucket` vì đó là action để list objects trong bucket, không liên quan đến việc get object |

### 📝 Phân tích từ vựng:

- **Explicit Deny**: từ chối rõ ràng, không ambiguous
- **Override**: ghi đè, có trọng lượng hơn
- **Priority**: thứ tự ưu tiên
- **Syntax error**: lỗi cú pháp, sai format
- **Policy aggregation**: tổng hợp policy từ nhiều nguồn

### 🔄 Thay đổi context:

- **Nếu bỏ deny policy** → User sẽ access được (Allow từ group có hiệu lực)
- **Nếu muốn allow dù có deny** → Không thể. Không có cách nào override explicit deny
- **Nếu chỉ có Allow policy** → Access được

---

## EC2 Security Groups - Reference by SG

### Question:

A web application runs on **EC2 instances** and needs to connect to an **RDS database**. The current **security group configuration** is:

- **EC2 Security Group (sg-web):** Inbound: 0.0.0.0/0 port 80
- **RDS Security Group (sg-db):** Inbound: 0.0.0.0/0 port 3306

The developer reports that the application cannot connect to the database. What is the best way to fix this while maintaining **security best practices**?

**A)** Change RDS SG inbound rule to reference sg-web
**B)** Change EC2 SG outbound rule to reference sg-db
**C)** Add a rule to RDS SG allowing port 3306 from sg-web
**D)** Add a rule to EC2 SG allowing port 3306 to sg-db

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C:**

RDS chạy trong VPC (thường là private subnet). Để EC2 kết nối đến RDS:

1. EC2 SG check **outbound** → default allow all ✅
2. RDS SG check **inbound** → phải allow traffic từ EC2

**Best practice cho Security Groups:**
- Thay vì dùng IP address (0.0.0.0/0), nên reference **security group khác**
- Khi EC2 scale, mọi instances có SG `sg-web` sẽ tự động được allow vào RDS

**Cách implement đúng:**
```
RDS SG Inbound:
  - Type: MySQL/Aurora (port 3306)
  - Source: sg-web (EC2 Security Group)
```

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Không có option "change to sg-web" - phải add rule mới, không phải change |
| **B** | Outbound của EC2 đã allow all default. Vấn đề nằm ở RDS inbound |
| **D** | EC2 cần outbound đến database, nhưng outbound đã allow all. SG của EC2 không cần thay đổi |

### 📝 Phân tích từ vựng:

- **Reference by security group**: thay vì dùng IP, dùng SG ID để allow
- **Inbound rule**: rule kiểm soát traffic đi vào
- **Outbound rule**: rule kiểm soát traffic đi ra
- **Best practice**: phương pháp tốt nhất được khuyến nghị

### 🔄 Thay đổi context:

- **Nếu RDS SG đã allow 0.0.0.0/0** → Vẫn không work có thể do:
  - RDS không publicly accessible (cần enable)
  - Network ACL blocking
  - Database credentials sai

---

## EC2 Purchasing Options - Reserved vs Spot

### Question:

A company has an application running **24/7** with **steady workload**. The backend uses 4 **EC2 instances** (m5.large) running continuously. The team wants to **optimize costs**. They are currently using **On-Demand**. Which option provides the **most cost savings**?

**A)** Spot Instances with 4 instances
**B)** Convertible Reserved Instances 1-year, All Upfront
**C)** Savings Plans 3-year, All Upfront
**D)** Standard Reserved Instances 3-year, All Upfront

### ✅ Answer: **D**

### 📖 Giải thích:

**Tính toán tiết kiệm (approximate):**

| Option | Discount so với On-Demand | Chi phí/year (est.) |
|--------|---------------------------|---------------------|
| On-Demand | 0% | ~$2,800 |
| Convertible RI 1yr | ~54% | ~$1,288 |
| Savings Plans 3yr | ~72% | ~$784 |
| **Standard RI 3yr** | **~72%** | **~$784** |

**Tại sao Standard RI 3-year All Upfront là best choice:**

1. **72% discount** - giống Savings Plans nhưng thường có giá tốt hơn cho steady workload
2. **3-year commitment** - discount cao nhất
3. **All Upfront** - thanh toán toàn bộ → discount cao nhất trong các payment options
4. **Standard RI** - được thiết kế cho steady state, không cần flexibility như Convertible

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (Spot)** | Ứng dụng chạy 24/7, steady workload → Spot không phù hợp vì có thể bị **interrupted** (terminated) bất cứ lúc nào. Database không nên dùng Spot |
| **B (Convertible 1yr)** | Discount thấp hơn (54% vs 72%), chỉ commit 1 year |
| **C (Savings Plans 3yr)** | Discount tương đương nhưng Savings Plans linh hoạt hơn (có thể change instance type/family). Với steady workload không đổi, Standard RI có giá tốt hơn |

### 📝 Phân tích từ vựng:

- **Steady workload**: workload ổn định, không dao động nhiều
- **On-Demand**: theo yêu cầu, trả tiền theo giờ sử dụng
- **Reserved Instance**: đặt trước, cam kết thời gian để được giảm giá
- **Savings Plans**: cam kết chi tiêu theo giờ
- **All Upfront**: thanh toán toàn bộ trước
- **Convertible**: có thể chuyển đổi sang instance type khác
- **Interrupted**: bị ngắt giữa chừng (Spot Instance)

### 🔄 Thay đổi context:

- **Nếu workload có thể thay đổi instance type** → Savings Plans tốt hơn vì flexible
- **Nếu cần guarantee capacity** → Reserved Instance (Standard hoặc Zonal)
- **Nếu startup chỉ có budget hàng tháng** → All Upfront không khả thi → Partial Upfront
- **Nếu workload có thể bị interrupt** → Spot Instances là lựa chọn tiết kiệm nhất

---

## IAM Role vs Access Key

### Question:

A **Lambda function** needs to access an **S3 bucket** to read files. The developer is **hard-coding IAM access keys** in the Lambda code. What is the **best way to improve security**?

**A)** Store access key in Lambda **environment variables**
**B)** Use an **IAM Role** attached to the Lambda function
**C)** Use **KMS** to encrypt the access key
**D)** Move access key to **Parameter Store**

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (IAM Role):**

Lambda chạy trên AWS infrastructure → có thể **assume IAM Role** để nhận temporary credentials tự động.

**Benefits của IAM Role cho Lambda:**

| Lợi ích | Giải thích |
|----------|------------|
| **No credentials management** | AWS tự động provide temporary credentials |
| **Auto-rotation** | Credentials được rotate tự động, không cần developer quản lý |
| **Audit trail** | CloudTrail log ai đã access, khi nào |
| **Least privilege** | Role có permissions vừa đủ cho function đó |

**Cách implement đúng:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

Attach policy này vào Lambda's **execution role**.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (Environment variables)** | Vẫn là static credentials, có thể bị leak qua **CloudWatch logs** (logs có thể ghi lại environment variables) |
| **C (KMS encrypt)** | Phức tạp không cần thiết. Access key vẫn cần decrypt mỗi lần chạy |
| **D (Parameter Store)** | Vẫn cần credentials để access Parameter Store → vấn đề nằm ở chỗ **hard-code** chưa được giải quyết triệt để |

### 📝 Phân tích từ vựng:

- **Hard-coding**: viết cố định giá trị vào code
- **IAM Role**: vai trò IAM, cho phép nhận temporary credentials
- **Execution role**: vai trò thực thi (dùng cho Lambda)
- **Least privilege**: nguyên tắc chỉ cấp quyền vừa đủ
- **Audit trail**: nhật ký kiểm tra, theo dõi ai làm gì

### 🔄 Thay đổi context:

- **Nếu Lambda cần access external API** → Vẫn cần credentials nhưng nên store trong **Secrets Manager**
- **Nếu Lambda cần cross-account access** → Use IAM Role với **STS assume role**
- **Nếu EC2 cần access S3** → Cũng nên dùng IAM Role thay vì Access Key

---

## AWS CLI - Multiple Accounts

### Question:

A developer needs to configure **AWS CLI** to access **multiple AWS accounts** (dev and prod). They need to switch between accounts frequently. What is the **best practice** to set this up without reconfiguring each time?

**A)** Use `--region` flag in each command
**B)** Create **named profiles** for each account
**C)** Set **environment variables** for each account
**D)** Use the same credentials for both accounts

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (Named Profiles):**

AWS CLI hỗ trợ **named profiles** - cho phép lưu nhiều bộ credentials và configuration:

```bash
# Configure dev profile
aws configure --profile dev
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region: ap-southeast-1

# Configure prod profile  
aws configure --profile prod
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region: ap-southeast-1
```

**Sử dụng:**
```bash
# List S3 buckets trong dev
aws s3 ls --profile dev

# List S3 buckets trong prod
aws s3 ls --profile prod
```

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Phải nhớ thêm flag mỗi lần → dễ sai, không lưu được credentials, không lưu được default region |
| **C** | Environment variables cần set/unset mỗi lần → không tiện lợi, dễ quên |
| **D** | Không bảo mật, không phân biệt được environment (dev vs prod), rất nguy hiểm |

### 📝 Phân tích từ vựng:

- **Named profile**: hồ sơ đặt tên, lưu cấu hình riêng
- **Environment variables**: biến môi trường
- **Credentials**: thông tin xác thực (Access Key)
- **Configure**: cấu hình AWS CLI

### 🔄 Thay đổi context:

- **Nếu muốn tự động switch profile** → Có thể dùng `aws-vault` hoặc `aws-sso`
- **Nếu dùng CI/CD** → Use IAM Role assumption thay vì static credentials
- **Nếu muốn set default profile** → Set `AWS_PROFILE` environment variable

---

## EC2 User Data - Only Runs Once

### Question:

A developer wants to **automate** the installation of Apache web server each time an EC2 instance launches. They use the following **User Data script**:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
```

The script works when the instance first launches. However, after the instance is **restarted**, Apache is not running. Why?

**A)** User Data only runs when an instance is **first launched**
**B)** yum command has incorrect syntax
**C)** systemctl command requires sudo
**D)** HTTP port is not open in Security Group

### ✅ Answer: **A**

### 📖 Giải thích:

**Tại sao chọn A:**

EC2 User Data script có đặc điểm quan trọng:

| Thời điểm | Chạy User Data? |
|------------|-----------------|
| Instance **launch lần đầu** | ✅ Có |
| Instance **restart/reboot** | ❌ Không |
| Instance **stop và start** | ❌ Không |
| Instance **launch từ stopped state** | ❌ Không |

User Data được thiết kế cho **bootstrapping** - chỉ chạy một lần khi instance được tạo.

**Để re-run script sau restart:**

| Phương pháp | Mô tả |
|-------------|--------|
| **cloud-init** | Dùng `runcmd` block để chạy mỗi boot |
| **SSM State Manager** | Apply configuration định kỳ |
| **Systems Manager Run Command** | Chạy command on-demand |

**Ví dụ với cloud-init:**
```yaml
#cloud-config
runcmd:
  - [systemctl, start, httpd]
  - [systemctl, enable, httpd]
```

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **B** | yum syntax đúng cho Amazon Linux |
| **C** | User Data chạy với quyền root → không cần sudo |
| **D** | Đúng là cần open port nhưng đó không phải lý do script không chạy |

### 📝 Phân tích từ vựng:

- **Bootstrapping**: khởi tạo, cấu hình ban đầu khi khởi động
- **User Data**: dữ liệu người dùng, script chạy khi launch
- **Restart vs Stop/Start**: restart = reboot, stop/start = shutdown rồi khởi động lại
- **cloud-init**: công cụ cloud initialization

### 🔄 Thay đổi context:

- **Nếu cần chạy script mỗi boot** → Dùng cloud-init hoặc systemd service
- **Nếu cần install packages mỗi boot** → Nên dùng **Launch Template** với bootstrap script hoặc **AMI** đã custom
- **Nếu muốn manage configuration** → Dùng SSM State Manager hoặc Ansible/Chef

---

## IAM Password Policy - Built-in Feature

### Question:

A company needs to enforce the following **password policy**:
- Minimum **12 characters**
- Require **uppercase**, **lowercase**, **numbers**, **symbols**
- **No password reuse** (5 previous passwords)
- Password **expires after 90 days**

Which AWS service should they use?

**A)** IAM Password Policy
**B)** AWS Secrets Manager
**C)** AWS SSM Parameter Store
**D)** AWS Cognito

### ✅ Answer: **A**

### 📖 Giải thích:

**IAM Password Policy** là **built-in feature** (tính năng có sẵn) cho IAM users:

```json
{
  "MinimumPasswordLength": 12,
  "RequireSymbols": true,
  "RequireNumbers": true,
  "RequireUppercaseCharacters": true,
  "RequireLowercaseCharacters": true,
  "PasswordReusePrevention": 5,
  "ExpirePasswords": true,
  "MaxPasswordAge": 90
}
```

**Tìm ở đâu:** IAM Dashboard → Account Settings → Password Policy

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **B (Secrets Manager)** | Dùng để store secrets (API keys, DB passwords, certificates), không phải cho IAM user passwords |
| **C (Parameter Store)** | Dùng để store parameters/configuration (như database URLs, feature flags), không liên quan đến password policy |
| **D (Cognito)** | Dùng cho **end-user authentication** (web/mobile apps), không phải cho IAM users trong AWS account |

### 📝 Phân tích từ vựng:

- **Password Policy**: chính sách mật khẩu
- **Built-in feature**: tính năng có sẵn, tích hợp sẵn
- **Password reuse prevention**: ngăn chặn tái sử dụng mật khẩu
- **Password expiration**: thời hạn mật khẩu

### 🔄 Thay đổi context:

- **Nếu cần password policy cho end-users (web app)** → Dùng Cognito User Pool với password policy
- **Nếu cần store database passwords** → Dùng Secrets Manager
- **Nếu cần store application configuration** → Dùng Parameter Store

---

## EC2 Instance Type Selection - Database

### Question:

A developer needs to select an **EC2 instance** for a **MySQL database** with:
- **32GB RAM**
- **High IOPS** (20,000+)
- Need **99.99% durability**

Which instance is the **best fit**?

**A)** t3.large
**B)** r5.xlarge
**C)** i3.2xlarge
**D)** c5.2xlarge

### ✅ Answer: **B**

### 📖 Giải thích:

**Phân tích từng instance:**

| Instance | Family | Use Case | RAM | Storage |
|----------|--------|----------|-----|---------|
| t3.large | General (Burstable) | Web, Dev/Test | 8GB | EBS only |
| **r5.xlarge** | **Memory Optimized** | **Database, Cache** | **32GB** | **EBS optimized** |
| i3.2xlarge | Storage Optimized | NoSQL, Data warehousing | 61GB | NVMe SSD |
| c5.2xlarge | Compute Optimized | Batch, ML | 16GB | EBS only |

**Tại sao r5.xlarge:**

| Yêu cầu | r5.xlarge đáp ứng? |
|----------|---------------------|
| 32GB RAM | ✅ Chính xác 32GB |
| High IOPS | ✅ EBS optimized, hỗ trợ io2 với 256000 IOPS |
| 99.99% durability | ✅ Kết hợp với EBS io2 Block Express |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (t3.large)** | Chỉ có 8GB RAM → không đủ. Là burstable instance → không phù hợp cho production database 24/7 |
| **C (i3.2xlarge)** | Storage optimized với NVMe, nhưng 61GB RAM → overkill cho 32GB requirement. NVMe không phải best choice cho database có yêu cầu durability cao |
| **D (c5.2xlarge)** | Compute optimized → thiếu RAM (chỉ 16GB), không phù hợp cho database |

### 📝 Phân tích từ vựng:

- **Memory Optimized**: được tối ưu cho workload cần nhiều RAM
- **Storage Optimized**: được tối ưu cho workload cần storage performance
- **Compute Optimized**: được tối ưu cho workload cần CPU
- **EBS optimized**: instance có network bandwidth cao cho EBS
- **Durability**: độ bền, khả năng không mất dữ liệu

### 🔄 Thay đổi context:

- **Nếu cần 99.99% durability thực sự** → Nên dùng **RDS** thay vì EC2 với EBS, vì RDS cung cấp Multi-AZ, automated backups, point-in-time recovery
- **Nếu cần NVMe cho tempdb** → i3 suitable cho temp tables nhưng không phải best cho primary storage
- **Nếu workload là read-heavy** → Có thể consider dùng ElastiCache (Redis) làm cache layer

---

## Tóm tắt Patterns trong DVA Exam

### IAM Questions:
- **Explicit Deny** ln thắng Explicit Allow
- IAM Role cho Lambda/EC2 tốt hơn Access Keys
- Least privilege principle

### EC2 Questions:
- Security Group: Reference by SG thay vì IP
- User Data chỉ chạy 1 lần khi launch
- Chọn đúng instance type theo workload

### Purchasing:
- Steady workload → Reserved / Savings Plans
- Flexible workload → Savings Plans
- Interruptible workload → Spot
