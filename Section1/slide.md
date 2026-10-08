# Section 1 - Getting started with AWS | IAM & AWS CLI | EC2 Fundamentals

---

## 1. Getting started with AWS (Trang 8-16)

### AWS Global Infrastructure (Trang 8)

- AWS Regions
- AWS Availability Zone
- AWS Data Centers
- AWS Edge Locations / Points of Presence

### AWS - The leading cloud (Trang 9)

*(Hình ảnh: AWS dẫn đầu thị trường cloud)*

### AWS Regions (Trang 10-11)

- **Region** là một cụm các data centers
- **Chọn AWS Region dựa trên:**
  - **Compliance với data governance và yêu cầu pháp lý:** data không bao giờ rời khỏi region mà không có sự cho phép rõ ràng của bạn
  - **Proximity đến khách hàng:** giảm latency
  - **Available services trong Region:** services và features mới không có ở mọi Region
  - **Pricing:** giá cả khác nhau theo region và được công khai trên service pricing page

> *Biểu đồ AWS Regions: Minh họa các region trên thế giới với các điểm coverage*

### AWS Availability Zones (Trang 12-14)

- Mỗi region có nhiều **Availability Zones** (AZ):
  - Thông thường là **3**, tối thiểu **3**, tối đa **6**
  - Ví dụ: `ap-southeast-2a`, `ap-southeast-2b`, `ap-southeast-2c`

- Mỗi AZ là một hoặc nhiều data centers riêng biệt với:
  - Nguồn điện dự phòng
  - Kết nối mạng dự phòng
  - Kết nối riêng

- Các AZ **tách biệt nhau** để tránh ảnh hưởng khi có disaster
- Các AZ được kết nối bằng mạng **bandwidth cao, latency cực thấp**

> *Sơ đồ Availability Zones: Hiển thị 3 AZ trong một Region, kết nối với nhau*

### AWS Edge Locations (Points of Presence) (Trang 15-16)

- Mạng lưới edge toàn cầu hiện có:
  - **400+ Edge Locations**
  - **13 Regional Caches**
  - **90+ thành phố** across **48+ quốc gia**

- **Mục đích:** Content được deliver đến end users với **latency thấp hơn**

> *Sơ đồ Edge Locations: Minh họa mạng lưới edge toàn cầu*

---

## 2. IAM & AWS CLI (Trang 17-32)

### AWS Identity and Access Management (AWS IAM) (Trang 17)

- IAM = Identity and Access Management
- Là **Global service** (không cần chọn region)

### IAM: Users & Groups (Trang 18)

- **Root account** được tạo mặc định, **không nên sử dụng hoặc chia sẻ**
- **Users** là người trong tổ chức, có thể được nhóm lại
- **Groups** chỉ chứa users, không chứa groups khác
- Users không bắt buộc phải thuộc một group, và một user có thể thuộc nhiều groups

> *Sơ đồ IAM Users & Groups: Root -> Groups (Admin, Dev) -> Users*

### IAM: Permissions (Trang 19)

- Users hoặc Groups có thể được gán các **JSON documents** gọi là **policies**
- Policies định nghĩa permissions của users
- Trong AWS áp dụng nguyên tắc **least privilege principle**: không cấp permissions nhiều hơn mức user cần

### IAM Policies inheritance (Trang 20)

> *Sơ đồ Policy Inheritance: Users trong Groups thừa hưởng policies từ group*

### IAM Policies Structure (Trang 21)

Policy gồm:

| Thành phần | Mô tả |
|------------|--------|
| **Version** | Policy language version |
| **Id** | Identifier cho policy (optional) |
| **Statement** | Một hoặc nhiều statements (required) |

**Statement gồm:**
- **Sid**: Identifier cho statement (optional)
- **Effect**: `Allow` hoặc `Deny`
- **Principal**: account/user/role
- **Action**: Danh sách actions được allow hoặc deny
- **Resource**: Danh sách resources mà actions áp dụng
- **Condition**: Điều kiện để policy có hiệu lực (optional)

### IAM - Password Policy (Trang 22)

- Strong passwords = higher security
- Trong AWS có thể setup password policy:
  - Set minimum password length
  - Require specific character types
  - Allow all IAM users thay đổi password của chính họ
  - Require users thay đổi password sau một khoảng thời gian (password expiration)
  - Prevent password re-use

### Multi Factor Authentication - MFA (Trang 23)

- Bảo vệ **Root Accounts** và **IAM users**
- MFA = **password bạn biết** + **security device bạn sở hữu**

### MFA devices options in AWS (Trang 24)

| Loại Device | Ví dụ |
|-------------|-------|
| **Virtual MFA device** | Google Authenticator (phone only), Authy (phone only) |
| **U2F Security Key** | YubiKey by Yubico (3rd party) |

### How can users access AWS? (Trang 25)

**3 cách truy cập AWS:**

| Phương thức | Bảo vệ bởi |
|-------------|-------------|
| **AWS Management Console** | Password + MFA |
| **AWS CLI** | Access Keys |
| **AWS SDK** | Access Keys |

- Access Keys được tạo qua AWS Console
- Access Keys là secret, không chia sẻ
- **Access Key ID** ~= username
- **Secret Access Key** ~= password

### What's the AWS CLI? (Trang 26)

- Tool cho phép tương tác với AWS services bằng commands trong command-line shell
- Direct access đến public APIs của AWS services
- Open-source: https://github.com/aws/aws-cli

### AWS SDK (Trang 27)

- AWS SDK = Software Development Kit
- **Language-specific APIs** (set of libraries)
- Cho phép truy cập và quản lý AWS services theo cách programmatic
- **Embedded trong application**

**Hỗ trợ:**
- SDKs: JS, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++
- Mobile SDKs: Android, iOS
- IoT Device SDKs: Embedded C, Arduino

> *AWS CLI được build trên AWS SDK for Python*

### IAM Roles for Services (Trang 28)

- Một số AWS service cần thực hiện actions thay mặt bạn
- Gán permissions cho AWS services bằng **IAM Roles**

**Common roles:**
- EC2 Instance Roles
- Lambda Function Roles
- Roles for CloudFormation

### IAM Security Tools (Trang 29)

| Tool | Cấp độ | Mô tả |
|------|--------|--------|
| **IAM Credentials Report** | Account-level | Report liệt kê tất cả users và status của credentials |
| **IAM Access Advisor** | User-level | Shows service permissions và thời điểm services được truy cập gần nhất |

### IAM Guidelines & Best Practices (Trang 30)

- ✅ Không sử dụng root account ngoài việc setup AWS account
- ✅ Một user vật lý = Một AWS user
- ✅ Gán users vào groups, gán permissions cho groups
- ✅ Tạo strong password policy
- ✅ Sử dụng và enforce MFA
- ✅ Tạo và sử dụng Roles để cấp permissions cho AWS services
- ✅ Sử dụng Access Keys cho Programmatic Access (CLI/SDK)
- ✅ Audit permissions bằng IAM Credentials Report & IAM Access Advisor
- ✅ Không chia sẻ IAM users & Access Keys

### Shared Responsibility Model for IAM (Trang 31)

| AWS responsibility | User responsibility |
|--------------------|---------------------|
| Infrastructure (global network security) | Users, Groups, Roles, Policies management |
| Configuration and vulnerability analysis | Enable MFA on all accounts |
| Compliance validation | Rotate all keys often |
| | Use IAM tools để apply permissions |
| | Analyze access patterns & review permissions |

### IAM Section - Summary (Trang 32)

- **Users**: mapped đến user vật lý, có password cho AWS Console
- **Groups**: chỉ chứa users
- **Policies**: JSON document định nghĩa permissions
- **Roles**: cho EC2 instances hoặc AWS services
- **Security**: MFA + Password Policy
- **AWS CLI**: quản lý AWS services bằng command-line
- **AWS SDK**: quản lý AWS services bằng programming language
- **Access Keys**: truy cập AWS bằng CLI hoặc SDK
- **Audit**: IAM Credential Reports & IAM Access Advisor

---

## 3. EC2 Fundamentals (Trang 33-60)

### Amazon EC2 - Basics (Trang 33)

- EC2 là một trong những offering phổ biến nhất của AWS
- EC2 = **Elastic Compute Cloud** = Infrastructure as a Service

**EC2 chủ yếu cung cấp khả năng:**
- Rent virtual machines (EC2)
- Store data trên virtual drives (EBS)
- Distribute load across machines (ELB)
- Scale services sử dụng Auto-Scaling Group (ASG)

> *EC2 là nền tảng quan trọng để hiểu Cloud hoạt động như thế nào*

### EC2 - Sizing & Configuration Options (Trang 34)

| Option | Các lựa chọn |
|--------|--------------|
| **Operating System (OS)** | Linux, Windows, Mac OS |
| **Compute power & cores (CPU)** | Tùy instance type |
| **RAM** | Tùy instance type |
| **Storage** | Network-attached (EBS & EFS) hoặc hardware (EC2 Instance Store) |
| **Network card** | Speed, Public IP address |
| **Firewall rules** | Security Group |
| **Bootstrap script** | EC2 User Data |

### EC2 - User Data (Trang 35)

- Có thể **bootstrapping** instances bằng **EC2 User Data script**
- Bootstrapping = launch commands khi machine start
- Script **chỉ chạy một lần** khi instance start lần đầu

**EC2 User Data được dùng để:**
- Install updates
- Install software
- Download common files từ internet
- Bất cứ điều gì bạn muốn

> *EC2 User Data Script chạy với quyền root user*

> *Demo: Launching an EC2 Instance running Linux - Trang 36*

### EC2 - Instance Types (Trang 37)

- Có nhiều loại EC2 instances được tối ưu cho các use cases khác nhau

**AWS naming convention:**
```
m5.2xlarge
│ │  │    │
│ │  │    └── size (kích thước)
│ │  └─────── generation (AWS cải thiện theo thời gian)
│ └────────── instance class
└───────────── instance family
```

### EC2 Instance Types - General Purpose (Trang 38)

**Cung cấp balance của:**
- Compute
- Memory
- Networking

**Phù hợp cho:**
- Web servers
- Code repositories

### EC2 Instance Types - Compute Optimized (Trang 39)

**Phù hợp cho compute-bound applications:**
- Compute intensive applications
- Batch processing workloads
- Media transcoding
- High performance web servers, HPC
- Scientific modeling, machine learning
- Dedicated gaming servers và ad server engines

### EC2 Instance Types - Memory Optimized (Trang 40)

**Hiệu suất cao cho workloads xử lý large data sets in memory:**
- Relational/non-relational databases
- Distributed web scale cache stores
- In-memory databases optimized for BI
- Real-time processing của big unstructured data

### EC2 Instance Types - Storage Optimized (Trang 41)

**High, sequential read/write access đến very large data sets trên local storage:**

**Phù hợp cho:**
- High frequency OLTP systems
- Relational & NoSQL databases
- Cache for in-memory databases (Redis)
- Data warehousing applications
- Distributed file systems

### EC2 - Security Groups (Trang 42-44)

- Kiểm soát traffic vào/ra EC2 (**firewall**)
- Có thể attach đến **nhiều instances**
- **Mặc định:** Tất cả inbound traffic bị **block**
- **Mặc định:** Tất cả outbound traffic được **authorize**
- Security groups chỉ chứa **allow rules**
- Rules có thể reference bằng **IP** hoặc **security group**

> *Sơ đồ Security Groups: Bastion SG -> Web server SG -> Inbound/Outbound rules*

### Classic Ports to know (Trang 45)

| Port | Protocol | Mô tả |
|------|----------|--------|
| **22** | SSH | Log into Linux instance |
| **21** | FTP | Upload files vào file share |
| **22** | SFTP | Upload files sử dụng SSH |
| **80** | HTTP | Access unsecured websites |
| **443** | HTTPS | Access secured websites |
| **3389** | RDP | Log into Windows instance |

### EC2 - Connect to your EC2 instance (Trang 46)

> *Hình ảnh: Các cách kết nối đến EC2*

### EC2 Instance Connect (Trang 47)

- Kết nối đến EC2 instance **trong browser**
- **Không cần** key file
- Cần đảm bảo port 22 vẫn mở
- Chỉ hoạt động trên **Amazon Linux** hoặc **Ubuntu**

### EC2 Instances Purchasing Options (Trang 48)

| Loại | Mô tả |
|------|--------|
| **On-Demand Instances** | Short workload, predictable pricing, pay by second |
| **Reserved Instances (1 & 3 years)** | Long workloads - Reserve Instances & Convertible Reserved Instances |
| **Savings Plans (1 & 3 years)** | Commit một amount of usage |
| **Spot Instances** | Short workloads, cheap, có thể mất instance |
| **Dedicated Hosts** | Book entire physical server |
| **Dedicated Instances** | Không share hardware với customers khác |
| **Capacity Reservations** | Reserve capacity trong specific AZ |

### EC2 On Demand (Trang 49)

| Ưu điểm | Nhược điểm |
|----------|-------------|
| Không có upfront payment | Chi phí cao nhất |
| Pay for what you use | |
| No long-term commitment | |

**Use cases:**
- Dev/stg/test environment
- Critical batch job
- System chạy theo giờ hành chính (8am - 5pm)
- System scale frequently

### EC2 Reserved Instances (Trang 50-51)

**Giảm đến 72%** so với On-demand

| Thuộc tính | Options |
|------------|---------|
| **Reservation Period** | 1 year (+discount) hoặc 3 years (+++discount) |
| **Payment Options** | No Upfront (+), Partial Upfront (++), All Upfront (+++) |
| **Scope** | Regional hoặc Zonal (reserve capacity trong AZ) |

**Convertible Reserved Instances:**
- Có thể thay đổi type, family, OS
- Giảm đến **66%**

**Use cases:**
- System chạy 24/7 trong thời gian dài
- Workload không thay đổi trong thời gian dài

### EC2 Savings Plans (Trang 52-53)

- Commit một consistent amount of usage, tính theo USD/hour
- Giảm đến **72%** so với On-demand
- Locked vào **specific instance family** & **AWS region**

**Flexible across:**
- Instance Size (m5.xlarge, m5.2xlarge)
- OS (Linux, Windows)
- Tenancy (Host, Dedicated, Default)

**Use cases:**
- System chạy 24/7 trong thời gian dài
- Workload cần thay đổi **INSTANCE TYPE** về lâu dài

### EC2 Spot Instances (Trang 54)

- Giảm đến **90%** so với On-demand
- **MOST cost-efficient** instances trong AWS
- Nếu spot price > max price → có thể stop/terminate trong 2 minutes grace period

**Spot Block:** "Block" spot instance trong 1-6 giờ mà không bị interrupt

**Use cases (workloads có thể bị interrupt):**
- Batch jobs
- Data analysis
- Workloads với flexible start và end time

### EC2 Dedicated Hosts (Trang 55)

- Physical server với EC2 instance capacity **hoàn toàn dedicated** cho bạn
- Sử dụng **existing server-bound software licenses** (per-socket, per-core, per-VM)

### EC2 Dedicated Instances (Trang 56)

- Instances chạy trên hardware dedicated cho bạn
- Có thể **share hardware** với instances khác trong cùng account
- **Không control** instance placement

**Use cases:**
- System cần strict policy chạy trong isolated environment
  - Ví dụ: Bank system, Hospital system

### Understanding AWS Tenancy (Trang 57)

> *Sơ đồ: Shared, Dedicated Host, Dedicated Instance*

### EC2 Capacity Reservations (Trang 58)

- Reserve On-Demand instances capacity trong **specific AZ**
- Luôn có quyền truy cập EC2 capacity khi cần
- **Không có time commitment** (create/cancel anytime)
- **Không có billing discounts**
- Bị charge theo On-Demand rate cho dù có chạy instances hay không

**Use cases:** Short-term, uninterrupted workloads cần ở specific AZ

### Which purchasing option is right for me? (Trang 59)

| Analogy | Purchasing Option |
|---------|-------------------|
| Resort - trả giá đầy đủ, ở bất kỳ lúc nào | On Demand |
| Resort - đặt trước lâu dài, được giảm giá | Reserved / Savings Plans |
| Resort - đặt phòng theo giờ, có thể bị đuổi ra | Spot Instances |
| Resort - book entire building | Dedicated Hosts |
| Resort - đặt phòng nhưng không ở, vẫn trả tiền | Capacity Reservations |

### AWS charges for IPv4 addresses (Trang 60)

- Từ **February 1, 2024**, có phí cho tất cả Public IPv4 được tạo trong account
- **$0.005/giờ** Public IPv4 (~$3.6/tháng)

**Free tier cho EC2 (12 tháng đầu):**
- 750 giờ Public IPv4/tháng

**Không có free tier:**
- Load Balancer (1 Public IPv4 mỗi AZ)
- RDS Database (1 Public IPv4)

---

## Tổng kết Section 1

### AWS Global Infrastructure
- **Region**: Cụm data centers
- **AZ**: Data centers riêng biệt trong region (3-6 AZ)
- **Edge Locations**: 400+ điểm cho content delivery

### IAM
- Users, Groups, Policies, Roles
- MFA, Password Policy
- AWS CLI, SDK, Access Keys

### EC2
- Instance Types: General, Compute, Memory, Storage Optimized
- Security Groups: Firewall kiểm soát traffic
- Purchasing Options: On-Demand, Reserved, Spot, Dedicated
