# Câu hỏi ôn tập DVA-C02 - Section 1

> *Các câu hỏi mẫu từ đề thi AWS Developer Associate. Câu hỏi bằng tiếng Anh, giải thích bằng tiếng Việt.*

---

## AWS Global Infrastructure - Choosing a Region (Source Page 11)

### Question:

A company is launching an application for customers in Asia. It must meet **data governance and legal requirements**, provide a required AWS service, and keep latency and cost reasonable. How should the company choose its AWS Region?

**A)** Choose the lowest-priced Region, even if it is far from customers or does not offer the required service.
**B)** First confirm legal/data requirements and service availability, then compare customer proximity and pricing.
**C)** Always choose the Region with the most Availability Zones.
**D)** Always choose the geographically closest Region, even if it violates legal/data requirements or lacks the required service.

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B:**
- Trang 11 nêu bốn yếu tố chọn Region: tuân thủ yêu cầu quản trị dữ liệu/pháp lý, khoảng cách đến khách hàng, dịch vụ hiện có và giá.
- Xác nhận ràng buộc pháp lý và dịch vụ bắt buộc trước; sau đó cân nhắc độ trễ và chi phí trong các Region phù hợp.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| A | Giá thấp không bù được việc vi phạm yêu cầu lưu trú dữ liệu hoặc thiếu dịch vụ cần thiết. |
| C | Số lượng AZ không phải tiêu chí duy nhất; Region phải đáp ứng yêu cầu dữ liệu, dịch vụ, độ trễ và chi phí. |
| D | Không thể chỉ chọn theo vị trí địa lý; cần thỏa mãn yêu cầu pháp lý và dịch vụ trước khi cân nhắc độ trễ. |

### 📝 Phân tích từ vựng:
- **Data governance**: yêu cầu quản trị và kiểm soát vị trí/xử lý dữ liệu.
- **Proximity to customers**: mức độ gần khách hàng, ảnh hưởng đến độ trễ.
- **Available services**: các dịch vụ AWS đang được cung cấp trong Region.

---

## IAM Policies - Từ chối tường minh

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

Trong AWS IAM, **Explicit Deny** luôn được ưu tiên hơn **Explicit Allow**. Khi có cả Allow và Deny trên cùng resource/action → **Deny thắng**.

AWS đánh giá policies theo thứ tự:
1. Kiểm tra Explicit Deny → **DENY** (dừng lại, không xét tiếp).
2. Kiểm tra Explicit Allow → **ALLOW**.
3. Mặc định → **DENY** (implicit deny).

Trong trường hợp này:
- **Policy của Developers Group:** Allow `s3:GetObject`.
- **Policy gắn trực tiếp vào user:** Deny `s3:*` (bao gồm cả GetObject).

→ Explicit Deny → **Bị từ chối truy cập**.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **B** | User policy không ghi đè group policy. AWS kết hợp tất cả policies, nhưng Deny luôn thắng. Đây gọi là **policy aggregation**. |
| **C** | Cú pháp JSON hợp lệ, không có lỗi. |
| **D** | `s3:GetObject` dùng để tải object. Không cần `ListBucket`, vì action đó dùng để liệt kê objects trong bucket, không liên quan đến việc lấy object. |

### 📝 Phân tích từ vựng:

- **Explicit Deny**: từ chối rõ ràng, không mơ hồ.
- **Override**: ghi đè, được ưu tiên hơn.
- **Priority**: thứ tự ưu tiên.
- **Syntax error**: lỗi cú pháp, sai định dạng.
- **Policy aggregation**: tổng hợp policy từ nhiều nguồn.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu bỏ deny policy** → User sẽ truy cập được (Allow từ group có hiệu lực).
- **Nếu muốn cho phép dù có deny** → Không thể. Không có cách nào ghi đè explicit deny.
- **Nếu chỉ có Allow policy** → Truy cập được.

---

## EC2 Security Groups - Tham chiếu bằng SG

### Question:

A web application runs on **EC2 instances** and needs to connect to an **RDS database**. The current **security group configuration** is:

- **EC2 Security Group (sg-web):** Inbound: `0.0.0.0/0` port 80; outbound: allow all (default)
- **RDS Security Group (sg-db):** Inbound: port 3306 from `sg-admin` only

The application cannot connect to the database. What is the best way to fix this while maintaining **security best practices**?

**A)** Allow inbound port 3306 to the RDS SG from `0.0.0.0/0`
**B)** Add an inbound port 3306 rule to the EC2 SG from `sg-db`
**C)** Add an inbound port 3306 rule to the RDS SG with source `sg-web`
**D)** Add an outbound port 3306 rule to the EC2 SG with destination `sg-db`

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C:**

Outbound rule mặc định của `sg-web` đã cho phép traffic đi ra. `sg-db` chỉ nhận kết nối port 3306 từ `sg-admin`, nên cần cho phép inbound port 3306 từ đúng **Security Group** của ứng dụng (`sg-web`).

```text
RDS SG Inbound:
  - Type: MySQL/Aurora (port 3306)
  - Source: sg-web (EC2 Security Group)
```

**Thực hành tốt nhất:** Tham chiếu Security Group thay vì địa chỉ IP; các EC2 instances gắn `sg-web` được phép kết nối mà không cần cập nhật rule khi IP thay đổi.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A** | Mở database cho mọi địa chỉ Internet là quá rộng và không cần thiết; chỉ cho phép `sg-web`. |
| **B** | Inbound của EC2 kiểm soát traffic đi vào EC2, không mở quyền truy cập vào RDS. |
| **D** | Outbound của EC2 đã cho phép traffic đi ra; thiếu quyền là inbound của `sg-db`. |

### 📝 Phân tích từ vựng:

- **Tham chiếu bằng Security Group**: dùng SG ID làm nguồn/đích thay vì địa chỉ IP.
- **Inbound rule**: quy tắc kiểm soát lưu lượng đi vào.
- **Outbound rule**: quy tắc kiểm soát lưu lượng đi ra.
- **Least privilege**: chỉ cho phép đúng nguồn và cổng cần thiết.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu RDS SG đã cho phép `0.0.0.0/0`** → Kết nối vẫn có thể lỗi do:
  - RDS không public (cần bật khả năng truy cập công khai).
  - Network ACL chặn lưu lượng.
  - Thông tin xác thực database sai.

---

## Lựa chọn hình thức mua EC2 - Reserved và Savings Plans

### Question:

A company runs **24/7 EC2 instances** in the **M5 family** in **us-east-1**. It wants a term-based discount that **automatically applies without exchanging a reservation** when it changes instance **size**, **operating system**, or **tenancy** within that family and Region. Which option best matches these requirements?

**A)** Spot Instances
**B)** Convertible Reserved Instances (3-year)
**C)** EC2 Instance Savings Plan (3-year)
**D)** Standard Reserved Instances (3-year)

### ✅ Answer: **C**

### 📖 Giải thích:

**Tại sao chọn C (EC2 Instance Savings Plan):**

EC2 Instance Savings Plans áp dụng cho mức sử dụng trong một **instance family** và **Region** đã chọn, đồng thời linh hoạt giữa các instance size, hệ điều hành và tenancy trong phạm vi đó. Cam kết chi tiêu theo giờ giúp giảm giá so với On-Demand khi workload chạy ổn định.

Không thể khẳng định trước một mức tiết kiệm hay chi phí năm cụ thể chỉ từ các lựa chọn này: giá phụ thuộc Region, family, thời hạn, phương thức thanh toán và mức sử dụng thực tế. Hãy so sánh giá hiện hành cho workload cụ thể.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (Spot)** | Spot có thể bị gián đoạn và không phù hợp với yêu cầu chạy ổn định 24/7. |
| **B (Convertible RI)** | Có thể trao đổi Reserved Instance theo các điều kiện áp dụng, nhưng việc đổi reservation không tự động áp dụng cho các thay đổi về size/OS/tenancy như EC2 Instance Savings Plan. |
| **D (Standard RI)** | Gắn với cấu hình đặt trước hẹp hơn, không phù hợp với các thay đổi nhóm dự kiến. |

### 📝 Phân tích từ vựng:

- **Steady workload**: workload ổn định, chạy đều.
- **Savings Plan**: cam kết một mức chi tiêu theo giờ để nhận mức giá chiết khấu.
- **Instance family**: họ instance, ví dụ M5.
- **Tenancy**: cách instance sử dụng phần cứng shared hoặc dedicated.
- **Convertible Reserved Instance**: RI có thể trao đổi theo các điều kiện của AWS.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu cần linh hoạt giữa các instance family hoặc Region** → Xem xét Compute Savings Plan.
- **Nếu cần capacity reservation trong một AZ cụ thể** → Xem xét Zonal Reserved Instance hoặc Capacity Reservation; Savings Plans tự thân không dự trữ capacity.

---



## IAM Role và Access Key

### Question:

A **Lambda function** needs to access an **S3 bucket** to read files. The developer is **hard-coding IAM access keys** in the Lambda code. What is the **best way to improve security**?

**A)** Store access key in Lambda **environment variables**
**B)** Use an **IAM Role** attached to the Lambda function
**C)** Use **KMS** to encrypt the access key
**D)** Move access key to **Parameter Store**

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (IAM Role):**

Lambda chạy trên hạ tầng AWS → có thể **assume IAM Role** để tự động nhận temporary credentials.

**Lợi ích của IAM Role cho Lambda:**

| Lợi ích | Giải thích |
|----------|------------|
| **Không phải quản lý credentials** | AWS tự động cung cấp temporary credentials |
| **Tự động luân chuyển** | Credentials được rotate tự động, developer không cần quản lý |
| **Audit trail** | CloudTrail ghi lại ai truy cập và vào thời điểm nào |
| **Least privilege** | Role chỉ có các quyền function cần |

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

Gắn policy này vào **execution role** của Lambda.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (Environment variables)** | Vẫn là static credentials; Lambda có thể đọc chúng khi chạy, nhưng AWS không tự quản lý hoặc luân chuyển chúng như temporary credentials của IAM Role. Chúng không tự động được ghi vào CloudWatch Logs. |
| **C (KMS encrypt)** | Mã hóa key không bỏ nhu cầu quản lý/giải mã static credentials; IAM Role tránh dùng access key cho quyền truy cập AWS. |
| **D (Parameter Store)** | Parameter Store có thể lưu secret, nhưng Lambda vẫn không cần access key để đọc S3; dùng execution role thay vì lưu static credentials. |

### 📝 Phân tích từ vựng:

- **Hard-coding**: ghi cố định giá trị vào code.
- **IAM Role**: vai trò IAM, cho phép nhận temporary credentials.
- **Execution role**: role thực thi (dùng cho Lambda).
- **Least privilege**: nguyên tắc chỉ cấp quyền vừa đủ.
- **Audit trail**: nhật ký theo dõi ai đã làm gì.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu Lambda cần truy cập external API** → Vẫn cần credentials nhưng nên lưu trong **Secrets Manager**.
- **Nếu Lambda cần truy cập giữa các accounts** → Dùng IAM Role với **STS assume role**.
- **Nếu EC2 cần truy cập S3** → Cũng nên dùng IAM Role thay vì Access Key.

---

## AWS CLI - Nhiều tài khoản

### Question:

A developer needs to configure **AWS CLI** to access **multiple AWS accounts** (dev and prod). They need to switch between accounts frequently. What is the **best practice** to set this up without reconfiguring each time?

**A)** Use `--region` flag in each command
**B)** Create **named profiles** for each account
**C)** Set **environment variables** for each account
**D)** Use the same credentials for both accounts

### ✅ Answer: **B**

### 📖 Giải thích:

**Tại sao chọn B (Named Profiles):**

AWS CLI hỗ trợ **profiles có tên**, cho phép lưu nhiều bộ credentials và cấu hình:

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
| **A** | Phải nhớ thêm flag mỗi lần → dễ sai; không lưu được credentials hoặc Region mặc định. |
| **C** | Mỗi lần phải đặt/hủy environment variables → bất tiện và dễ quên. |
| **D** | Không bảo mật, không phân biệt được môi trường (dev và prod), rất nguy hiểm. |

### 📝 Phân tích từ vựng:

- **Named profile**: profile có tên, lưu cấu hình riêng.
- **Environment variables**: biến môi trường.
- **Credentials**: thông tin xác thực (Access Key).
- **Configure**: cấu hình AWS CLI.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu muốn tự động chuyển profile** → Có thể dùng `aws-vault` hoặc `aws-sso`.
- **Nếu dùng CI/CD** → Dùng cơ chế IAM Role thay cho credentials tĩnh.
- **Nếu muốn đặt profile mặc định** → Đặt biến môi trường `AWS_PROFILE`.

---

## EC2 User Data - Chỉ chạy một lần

### Question:

A developer wants to **run a bootstrap script on every boot** of a Linux EC2 instance. The instance uses the default **User Data** behavior, and the script is:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
```

The script ran during the instance's **initial boot**, but did not run again when the instance was rebooted. Which statement explains this behavior?

**A)** By default, User Data runs only during the instance's **first boot**
**B)** The `yum` command has incorrect syntax
**C)** The `systemctl` command requires `sudo`
**D)** The HTTP port is not open in the Security Group

### ✅ Answer: **A**

### 📖 Giải thích:

**Tại sao chọn A:**

EC2 User Data script có một đặc điểm quan trọng:

| Thời điểm | User Data có chạy không? |
|------------|-----------------|
| Instance **launch lần đầu** | ✅ Có |
| Instance **restart/reboot** | ❌ Không |
| Instance **stop rồi start** | ❌ Không |

User Data được thiết kế để **bootstrapping** - chỉ chạy một lần khi instance được tạo.

**Để dịch vụ tự khởi động sau reboot:**

User Data không cần chạy lại để khởi động service đã cài đặt. Với Apache, bật service để systemd tự khởi động khi boot:

```bash
sudo systemctl enable --now httpd
```

Nếu cần chạy tác vụ mỗi lần boot, cấu hình **systemd service** hoặc cloud-init chạy mỗi boot; `runcmd` thông thường chỉ chạy một lần cho mỗi instance.

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **B** | Cú pháp `yum` hợp lệ trên Amazon Linux. |
| **C** | User Data chạy với quyền root theo mặc định, nên script không cần `sudo`. |
| **D** | Security Group có thể ảnh hưởng kết nối HTTP, nhưng không quyết định User Data có chạy lại sau reboot hay không. |

### 📝 Phân tích từ vựng:

- **Bootstrapping**: khởi tạo, cấu hình ban đầu khi khởi động.
- **User Data**: dữ liệu người dùng, script chạy khi launch.
- **Restart vs Stop/Start**: restart = reboot; stop/start = tắt rồi khởi động lại.
- **cloud-init**: công cụ khởi tạo cloud.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu cần chạy service mỗi boot** → Dùng systemd service hoặc cấu hình cloud-init chạy ở mỗi boot; `runcmd` mặc định chỉ chạy một lần.
- **Nếu cần cài package trên từng instance mới** → Đặt bootstrap script trong User Data của Launch Template hoặc tạo AMI đã tùy chỉnh.
- **Nếu muốn quản lý cấu hình** → Dùng SSM State Manager hoặc Ansible/Chef.

---

## IAM Password Policy - Tính năng tích hợp sẵn

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

**IAM Password Policy** là tính năng tích hợp sẵn cho IAM users:

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
| **B (Secrets Manager)** | Dùng để lưu secrets (API keys, mật khẩu DB, certificates), không dùng để quản lý mật khẩu IAM user. |
| **C (Parameter Store)** | Dùng để lưu parameters/cấu hình (như database URLs, feature flags), không liên quan đến password policy. |
| **D (Cognito)** | Dùng cho xác thực người dùng cuối (ứng dụng web/di động), không dành cho IAM users trong AWS account. |

### 📝 Phân tích từ vựng:

- **Password Policy**: chính sách mật khẩu.
- **Built-in feature**: tính năng tích hợp sẵn.
- **Password reuse prevention**: ngăn sử dụng lại mật khẩu.
- **Password expiration**: thời hạn mật khẩu.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu cần password policy cho người dùng cuối (ứng dụng web)** → Dùng Cognito User Pool với password policy.
- **Nếu cần lưu mật khẩu database** → Dùng Secrets Manager.
- **Nếu cần lưu cấu hình ứng dụng** → Dùng Parameter Store.

---

## Chọn EC2 Instance Type - Cơ sở dữ liệu

### Question:

A developer needs to select an **EC2 instance** for a MySQL database backed by EBS, with:
- **32GB RAM**
- **20,000+ EBS IOPS**
- Persistent storage with at least **99.99% durability**

The EBS volume and instance-level EBS bandwidth can be configured to meet the storage requirements. Which instance type provides the required memory?


**A)** t3.large
**B)** r5.xlarge
**C)** i3.2xlarge
**D)** c5.2xlarge

### ✅ Answer: **B**

### 📖 Giải thích:

**Phân tích từng instance:**

| Instance | Họ | Trường hợp sử dụng | RAM | Lưu trữ |
|----------|--------|----------|-----|---------|
| t3.large | Mục đích chung (có khả năng burst) | Web, phát triển/kiểm thử | 8GB | Chỉ EBS |
| **r5.xlarge** | **Tối ưu cho bộ nhớ** | **Cơ sở dữ liệu, cache** | **32GB** | **Tối ưu cho EBS** |
| i3.2xlarge | Tối ưu cho lưu trữ | NoSQL, kho dữ liệu | 61GB | NVMe SSD |
| c5.2xlarge | Tối ưu cho tính toán | Xử lý theo lô, ML | 16GB | Chỉ EBS |
| Yêu cầu | r5.xlarge có đáp ứng? |
|----------|---------------------|
| 32GB RAM | ✅ Chính xác 32GB |
| IOPS và độ bền | Cần volume EBS phù hợp; xác nhận IOPS/băng thông mà volume và instance hỗ trợ cùng đáp ứng yêu cầu. |

**Tại sao không chọn các đáp án khác:**

| Đáp án | Lý do loại bỏ |
|--------|----------------|
| **A (t3.large)** | Chỉ có 8GB RAM → không đủ. Đây là burstable instance, không phù hợp cho cơ sở dữ liệu production 24/7. |
| **C (i3.2xlarge)** | Tối ưu cho lưu trữ với NVMe, nhưng có 61GB RAM → dư so với yêu cầu 32GB. NVMe không phải lựa chọn tốt nhất cho cơ sở dữ liệu cần độ bền cao. |
| **D (c5.2xlarge)** | Tối ưu cho tính toán → thiếu RAM (chỉ 16GB), không phù hợp cho cơ sở dữ liệu. |

### 📝 Phân tích từ vựng:

- **Tối ưu cho bộ nhớ (Memory Optimized)**: dành cho khối lượng công việc cần nhiều RAM.
- **Tối ưu cho lưu trữ (Storage Optimized)**: dành cho khối lượng công việc cần hiệu năng lưu trữ cao.
- **Tối ưu cho tính toán (Compute Optimized)**: dành cho khối lượng công việc cần CPU.
- **EBS optimized**: instance có băng thông mạng cao cho EBS.
- **Durability**: độ bền, khả năng không mất dữ liệu.

### 🔄 Thay đổi ngữ cảnh:

- **Nếu thực sự cần độ bền 99.99%** → Nên dùng **RDS** thay vì EC2 với EBS, vì RDS cung cấp Multi-AZ, automated backups và khôi phục về một thời điểm cụ thể.
- **Nếu cần NVMe cho tempdb** → i3 phù hợp cho temp tables nhưng không phải lựa chọn tốt nhất làm primary storage.
- **Nếu workload chủ yếu đọc** → Có thể cân nhắc dùng ElastiCache (Redis) làm lớp cache.

---

## Tóm tắt các mẫu câu hỏi DVA Exam

### Câu hỏi IAM:
- **Explicit Deny** luôn thắng Explicit Allow.
- IAM Role cho Lambda/EC2 tốt hơn Access Keys.
- Nguyên tắc least privilege.

### Câu hỏi EC2:
- Security Group: tham chiếu bằng SG thay vì IP.
- User Data chỉ chạy một lần khi launch.
- Chọn đúng Instance Type theo workload.

### Hình thức mua:
- Workload ổn định → Reserved / Savings Plans.
- Workload linh hoạt → Savings Plans.
- Workload có thể bị gián đoạn → Spot.
