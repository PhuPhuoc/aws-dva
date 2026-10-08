# Section 5 - VPC Nâng Cao, ECS, ECR & Fargate, EKS, CloudFront

---

## Phần 1: VPC Nâng Cao (Trang 261-281)

### Bastion Hosts (Trang 261)

- **Bastion Host** được sử dụng để SSH vào các EC2 instance private
- Bastion nằm trong **public subnet** và kết nối đến các private subnets khác
- **Security Group của Bastion**:
  - Allow inbound từ Internet on port 22
  - Restrict CIDR (ví dụ: corporate public CIDR)

- **Security Group của EC2 Instances**:
  - Allow traffic từ Bastion Security Group
  - Hoặc allow từ private IP của Bastion

### NAT Gateway (Trang 262-263)

**Đặc điểm:**
- **AWS-managed** NAT - do AWS quản lý
- Higher bandwidth, high availability - băng thông cao, độ sẵn sàng cao
- **Không cần quản lý**
- Pay per hour cho usage và bandwidth
- NAT Gateway được tạo trong một **specific AZ**
- Uses an **Elastic IP**
- Requires an **IGW**

**Thông số kỹ thuật:**
- **5 Gbps** bandwidth, auto scaling up to **100 Gbps**
- **Không có Security Groups** để quản lý

### Network Access Control List (NACL) (Trang 264-266)

**Đặc điểm NACL:**
- Firewall kiểm soát traffic **từ và đến subnets**
- **Một NACL per subnet**
- New subnets được assign **Default NACL**
- Có thể có **ALLOW và DENY rules**
- Rules chỉ bao gồm **IP addresses**

**Default NACL:**
- Accepts **everything** inbound/outbound - chấp nhận tất cả
- **DON'T modify Default NACL** → Tạo custom NACLs thay thế

### Security Group vs NACLs (Trang 267)

| Đặc điểm | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance | Subnet |
| **Rules** | Chỉ Allow | Allow và Deny |
| **State** | **Stateful** | **Stateless** |
| **Rule evaluation** | Tất cả rules | Theo thứ tự (thấp nhất thắng) |
| **Default** | Deny inbound, Allow outbound | Allow all |

### VPC Peering (Trang 268-269)

- **Privately connect** hai VPCs sử dụng AWS network
- Behave như **same network** - hoạt động như cùng một mạng
- **Yêu cầu:**
  - **Must not have overlapping CIDRs** - không được trùng CIDR
  - **Non-transitive** - A-B connection, B-C connection ≠ A-C
  - Must update **route tables** in both VPCs - phải cập nhật route tables ở cả hai VPC

### VPC Endpoints (Trang 270-272)

**Hai loại:**

| Loại | Services | Chi phí | Security |
|------|----------|-------|----------|
| **Gateway** | S3, DynamoDB | **Free** | Route table target |
| **Interface** | Most AWS services | $ per hour + GB | ENI + SG |

**Lợi ích:**
- Private access đến AWS services
- Không cần IGW, NATGW
- Powered by **AWS PrivateLink**
- Redundant và scale horizontally - dự phòng và mở rộng ngang

### VPC Flow Logs (Trang 273-276)

**Thu thập thông tin:**
- IP traffic vào interfaces
- Levels: **VPC, Subnet, ENI**
- Destinations: **S3, CloudWatch Logs, Kinesis Data Firehose**

**Log fields - các trường:**
- `srcaddr`, `dstaddr` - identify problematic IPs - nhận diện IP có vấn đề
- `srcport`, `dstport` - identify problematic ports - nhận diện port có vấn đề
- `action` - ACCEPT/REJECT

### Site-to-Site VPN (Trang 277-279)

- Connect **on-premises VPN** đến AWS
- Connection tự động **encrypted** - mã hóa
- Goes over the **public internet**

### Direct Connect (Trang 278)

- Establish **physical connection** giữa on-premises và AWS
- Connection **private, secure, fast** - riêng tư, bảo mật, nhanh
- Goes over **private network**
- Takes **at least a month** to establish - mất ít nhất một tháng để thiết lập

---

## Phần 2: ECS, ECR & Fargate (Trang 285-320)

### Docker là gì? (Trang 286)

- **Docker** là nền tảng phát triển phần mềm để deploy ứng dụng
- Apps được đóng gói trong **containers**
- Apps chạy giống nhau, bất kể chạy ở đâu
- **Lợi ích:**
  - Any machine, any OS - bất kỳ máy nào, hệ điều hành nào
  - No compatibility issues - không có vấn đề tương thích
  - Predictable behavior - hành vi có thể dự đoán
  - Less work - ít công việc hơn
  - Easier to maintain và deploy - dễ bảo trì và deploy hơn
  - Works with any language - hoạt động với mọi ngôn ngữ

### Docker Repositories (Trang 288)

**Docker Hub:**
- **Public repository** - kho công khai
- Base images cho nhiều công nghệ
- Ví dụ: Ubuntu, MySQL

**Amazon ECR (Elastic Container Registry):**
- **Private repository** - kho riêng tư
- Public repository (Amazon ECR Public Gallery)
- Tích hợp với AWS services

### Docker Containers Management on AWS (Trang 291)

**Các lựa chọn:**

| Service | Mô tả |
|---------|-------------|
| **Amazon ECS** | Nền tảng container của AWS |
| **Amazon EKS** | Kubernetes được quản lý |
| **AWS Fargate** | Nền tảng container serverless |
| **Amazon ECR** | Lưu trữ container images |

### ECS Cluster (Trang 292)

- **Cluster** = nhóm các services và tasks
- **Phải tạo cluster trước!**

### ECS Task Definition (Trang 293-295)

- **Task Definition** = metadata (JSON) để tell ECS cách run Docker container

**Chứa:**
- Image Name - tên image
- Port Binding (Container & Host)
- Memory và CPU required - yêu cầu bộ nhớ và CPU
- Environment variables - biến môi trường
- Networking information - thông tin mạng
- **IAM Role**
- Logging configuration (CloudWatch)
- Up to **10 containers** per Task Definition

### Amazon ECS - EC2 Launch Type (Trang 297)

**Đặc điểm:**
- Tasks được khởi chạy trong **EC2 instances** của bạn
- **Bạn phải provision & maintain** infrastructure
- Mỗi EC2 Instance phải chạy **ECS Agent**
- AWS xử lý starting/stopping containers

### ECS Agent (Trang 298)

- ECS Agent được cài đặt trên mỗi EC2 Instance
- Xử lý các yêu cầu deploy container
- Xử lý vòng đời container
- Pre-installed on **ECS AMI**

### Amazon ECS - IAM Roles (Trang 299)

**1. EC2 Instance Profile (chỉ EC2 Launch Type):**
- Được ECS agent sử dụng
- Gọi API đến ECS service
- Gửi logs đến CloudWatch
- Pull images từ ECR
- Truy cập secrets từ Secrets Manager/SSM Parameter Store

**2. ECS Task Role:**
- Mỗi task có specific role - vai trò cụ thể
- Được định nghĩa trong task definition
- Cho phép các roles khác nhau cho các ECS Services khác nhau

### Amazon ECS - Fargate Launch Type (Trang 300)

**Đặc điểm:**
- Tasks được khởi chạy đến **Fargate** (AWS quản lý)
- **Không có EC2 instances** để quản lý
- **Serverless!**
- Chỉ cần tạo task definitions
- AWS chạy tasks dựa trên **CPU/RAM** cần thiết
- Scale bằng cách tăng **number of tasks**

### ECS Service (Trang 301)

- **Service** giám sát tasks
- Giữ tasks chạy (desired count)
- Launch/terminate instances
- Expose tasks ra bên ngoài
- Direct traffic đến đúng host và port

### Amazon ECS - Load Balancer Integrations (Trang 302)

| LB Type | Khuyến nghị |
|---------|---------------|
| **Application Load Balancer** | ✅ Hỗ trợ, hoạt động cho hầu hết use cases |
| **Network Load Balancer** | Cho high throughput hoặc PrivateLink |
| **Classic Load Balancer** | Không khuyến khích (không hỗ trợ Fargate) |

### Amazon ECS - Data Volumes (Trang 303-304)

**EFS File Systems:**
- Mount EFS vào ECS tasks
- Hoạt động cho **cả EC2 và Fargate**
- Tasks ở bất kỳ AZ nào **chia sẻ cùng dữ liệu**
- Fargate + EFS = **Serverless**
- **Use case:** persistent multi-AZ shared storage

**Bind Mounts:**
- Chia sẻ dữ liệu giữa **multiple containers** trong cùng task
- **EC2 Tasks:** Dữ liệu gắn với vòng đời EC2 instance
- **Fargate Tasks:** Ephemeral storage (20-200 GiB)
- **Use case:** Sidecar pattern (logs/metrics)

### ECS Service Auto Scaling (Trang 305)

**Amazon ECS Auto Scaling:**
- Tự động tăng/giảm ECS tasks

**Metrics - các chỉ số:**
- ECS Service Average **CPU Utilization**
- ECS Service Average **Memory Utilization**
- ALB Request Count Per Target

**Scaling Policies - các chính sách:**

| Policy | Mô tả |
|--------|-------------|
| **Target Tracking** | Scale dựa trên giá trị mục tiêu |
| **Step Scaling** | Scale dựa trên CloudWatch Alarm |
| **Scheduled Scaling** | Scale dựa trên ngày/giờ |

### ECS Rolling Updates (Trang 308-310)

**Parameters - các tham số:**
- **Minimum healthy percent:** Số task tối thiểu phải chạy
- **Maximum percent:** Số task tối đa được tạo

### ECS Environment Variables (Trang 317)

**Các nguồn:**

| Source | Use Case |
|--------|----------|
| **Hardcoded** | URLs |
| **SSM Parameter Store** | Sensitive configs (API keys) |
| **Secrets Manager** | Sensitive data (DB passwords) |
| **S3 Environment Files** | Bulk configs |

### Amazon ECR (Trang 318-319)

**Tính năng:**
- **Lưu trữ và quản lý Docker images** trên AWS
- Private và Public repository
- **Tích hợp hoàn toàn với ECS**
- Backed by **Amazon S3**
- Access controlled through **IAM**
- Hỗ trợ:
  - Image vulnerability scanning - quét lỗ hổng
  - Versioning - phiên bản
  - Image tags - nhãn
  - Image lifecycle - vòng đời

### AWS Copilot (Trang 320)

- **CLI tool** để build, release, và vận hành containerized apps
- Chạy trên **AppRunner, ECS, Fargate**
- Tự động provision infrastructure
- Automated deployments với CodePipeline
- Deploy đến nhiều môi trường

---

## Phần 3: Amazon EKS (Trang 321-324)

### Amazon EKS Overview (Trang 321)

- **EKS** = Elastic Kubernetes Service
- Khởi chạy managed **Kubernetes clusters** trên AWS
- **Kubernetes**: Open-source system cho deployment, scaling, quản lý containerized apps
- Alternative to ECS, **cùng mục tiêu, khác API**
- Hỗ trợ **EC2** (worker nodes) hoặc **Fargate**

**Use case:**
- Company đã dùng Kubernetes on-premises/cloud
- Muốn migrate sang AWS sử dụng Kubernetes
- **Kubernetes is cloud-agnostic** - hoạt động trên mọi cloud

### EKS Node Types (Trang 323)

| Loại | Mô tả |
|------|-------------|
| **Managed Node Groups** | AWS tạo/quản lý EC2 instances trong ASG |
| **Self-Managed Nodes** | Bạn tạo, đăng ký với cluster, quản lý ASG |
| **AWS Fargate** | Không có nodes để quản lý |

### EKS Data Volumes (Trang 324)

**Storage Classes - các lớp lưu trữ:**
- **Amazon EBS**
- **Amazon EFS** (hoạt động với Fargate)
- **Amazon FSx for Lustre**
- **Amazon FSx for NetApp ONTAP**

---

## Phần 4: Amazon CloudFront (Trang 338-350)

### Amazon CloudFront Overview (Trang 339)

- **CDN** = Content Delivery Network
- **Lợi ích:**
  - Cải thiện read performance - cải thiện hiệu suất đọc (cached at edge)
  - 600+ **Edge Locations** trên toàn cầu
  - DDoS protection - bảo vệ DDoS
  - Tích hợp với **Shield, WAF**

### CloudFront - Origins (Trang 340)

**S3 Bucket:**
- Phân phối files
- **Origin Access Control (OAC)** - bảo mật nâng cao
- Có thể sử dụng như **ingress** (upload to S3)

**Custom Origin (HTTP):**
- Application Load Balancer
- EC2 instance
- S3 website (static hosting)
- Any HTTP backend

### CloudFront vs S3 Cross Region Replication (Trang 344)

| Đặc điểm | CloudFront | S3 CRR |
|---------|------------|---------|
| Scope | Toàn cầu | Per-region setup |
| Updates | Cached for TTL | Near real-time |
| Read/Write | Chỉ đọc | Chỉ đọc |
| Use case | Static content ở mọi nơi | Dynamic ở vài regions |

### CloudFront - Caching (Trang 345-346)

**Cache Key:**
- Unique identifier cho mỗi object trong cache
- Default: hostname + resource portion of URL
- Có thể thêm **headers, cookies, query strings** qua Cache Policies

**Cache Behaviors:**
- Các settings khác nhau cho URL path patterns
- `/images/*`, `/api/*`, `/*` (default)

### CloudFront - Cache Invalidations (Trang 347)

- Khi origin update, CloudFront không biết
- Có thể force cache refresh qua **Invalidation**
- Invalidate all files (`/*`) hoặc specific path (`/images/*`)

### CloudFront - Origin Groups (Trang 343)

- Tăng **high availability** và **failover**
- **Primary** và **secondary** origin
- Auto failover khi primary fails

### CloudFront - VPC Origins (Trang 349-350)

- Phân phối content từ **private subnets** (VPC)
- Không cần expose trên Internet
- Hỗ trợ:
  - Application Load Balancer
  - Network Load Balancer
  - EC2 Instances
