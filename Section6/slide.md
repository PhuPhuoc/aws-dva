# Section 6 - Containers trên AWS: ECS, ECR, Fargate, Docker và EKS

---

## Phần 1: Docker và các dịch vụ container trên AWS (Trang 285-291)

### Containers trên AWS (Trang 285)

- Các chủ đề trong phần này: ECS, Fargate, ECR và EKS.

### Docker là gì? (Trang 286)

- **Docker** là nền tảng phát triển phần mềm để triển khai ứng dụng.
- Ứng dụng được đóng gói trong **containers**, có thể chạy trên nhiều hệ điều hành.
- Ứng dụng hoạt động nhất quán dù chạy ở đâu:
  - Chạy trên nhiều máy.
  - Giảm vấn đề tương thích.
  - Hành vi có thể dự đoán.
  - Giảm công sức vận hành.
  - Dễ bảo trì và triển khai.
  - Hỗ trợ nhiều ngôn ngữ, hệ điều hành và công nghệ.
- Trường hợp sử dụng: kiến trúc microservices, chuyển ứng dụng từ môi trường tại chỗ lên AWS.

> *Sơ đồ (Trang 287): Docker hoạt động trên một hệ điều hành.*

### Nơi lưu trữ Docker images (Trang 288)

- Docker images được lưu trong các kho lưu trữ Docker.
- **Docker Hub** là kho công khai, có image nền cho nhiều công nghệ và hệ điều hành, như Ubuntu và MySQL.
- **Amazon ECR** có kho riêng tư và kho công khai tại [Amazon ECR Public Gallery](https://gallery.ecr.aws).

### Docker và máy ảo (Trang 289)

- Docker có một số đặc điểm giống công nghệ ảo hóa nhưng không hoàn toàn là máy ảo.
- Containers chia sẻ tài nguyên với host, vì vậy một máy chủ có thể chạy nhiều containers.

> *Sơ đồ (Trang 290): Hướng dẫn bắt đầu sử dụng Docker.*

### Quản lý containers trên AWS (Trang 291)

| Dịch vụ | Vai trò |
|---------|---------|
| **Amazon ECS** | Nền tảng container của AWS |
| **Amazon EKS** | Kubernetes được AWS quản lý |
| **AWS Fargate** | Nền tảng container serverless, dùng được với ECS và EKS |
| **Amazon ECR** | Lưu trữ container images |

---

## Phần 2: Amazon ECS, ECR và Fargate (Trang 292-320)

### ECS Cluster và Task (Trang 292-293)

- **Cluster** là nhóm các dịch vụ và tasks; cần tạo cluster trước.
- **Task** là đơn vị công việc cơ bản của ECS, có thể chứa một hoặc nhiều containers.
- Task chạy containers theo cấu hình trong **Task Definition**; một mẫu sử dụng phổ biến là container sidecar.

> *Sơ đồ (Trang 294): So sánh EC2 với ECS.*

### ECS Task Definition (Trang 295-296)

- **Task Definition** là siêu dữ liệu dạng JSON, hướng dẫn ECS cách chạy Docker container.
- Các thông tin chính gồm:
  - Tên image.
  - Ánh xạ port giữa container và host.
  - Bộ nhớ và CPU cần thiết.
  - Biến môi trường và thông tin mạng.
  - IAM Role.
  - Cấu hình ghi logs, ví dụ CloudWatch.
- Một Task Definition hỗ trợ tối đa **10 containers**.

> *Sơ đồ (Trang 296): IAM Role gắn với ECS Task Definition.*

### Amazon ECS - EC2 Launch Type (Trang 297-298)

- Tasks được khởi chạy trên các EC2 instances của bạn.
- Bạn phải cung cấp và duy trì hạ tầng EC2.
- Mỗi EC2 instance phải chạy **ECS Agent** để đăng ký vào ECS Cluster.
- AWS phụ trách khởi chạy và dừng containers.
- ECS Agent được cài trên mỗi EC2 container instance; nhận yêu cầu triển khai và quản lý vòng đời container.
- ECS Agent được cài sẵn khi dùng ECS AMI.

### Amazon ECS - IAM Roles (Trang 299)

**EC2 Instance Profile (chỉ dành cho EC2 Launch Type):**
- ECS Agent trên EC2 container instance dùng role này để gọi API của ECS.

**Task Execution Role:**
- ECS/Fargate Agent dùng role này khi khởi chạy task: tải Docker image từ ECR, ghi log vào CloudWatch Logs và lấy dữ liệu bí mật từ Secrets Manager hoặc SSM Parameter Store.

**ECS Task Role:**
- Cấp thông tin xác thực AWS cho mã ứng dụng trong container để gọi các dịch vụ AWS.
- Có thể khai báo role riêng cho từng task trong Task Definition.

> *Hiệu chỉnh theo mô hình ECS hiện hành so với slide gốc: quyền tải image từ ECR, ghi log vào CloudWatch Logs và truy xuất dữ liệu bí mật (secrets) thuộc Task Execution Role, không phải EC2 Instance Profile ([AWS docs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)).*

### Amazon ECS - Fargate Launch Type (Trang 300)

- Tasks được chạy trên Fargate, hạ tầng do AWS quản lý.
- Không cần cung cấp hạ tầng EC2; đây là mô hình serverless.
- Chỉ cần tạo Task Definitions; AWS chạy tasks theo CPU/RAM đã chỉ định.
- Mở rộng bằng cách tăng số lượng tasks.

### ECS Service (Trang 301)

- **Service** giám sát tasks và duy trì số lượng tasks mong muốn.
- Service khởi chạy tasks để duy trì chiến lược lập lịch.
- Cung cấp tasks cho bên ngoài truy cập và định tuyến lưu lượng đến đúng host, port.

### Tích hợp Load Balancer với ECS (Trang 302)

| Loại Load Balancer | Khuyến nghị trong slide |
|--------------------|--------------------------|
| **Application Load Balancer** | Được hỗ trợ, phù hợp với phần lớn trường hợp sử dụng |
| **Network Load Balancer** | Dành cho thông lượng/hiệu năng cao hoặc kết hợp với AWS PrivateLink |
| **Classic Load Balancer** | Được hỗ trợ nhưng không khuyến nghị; thiếu tính năng nâng cao và không dùng được với Fargate |

### Volumes dữ liệu trong ECS (Trang 303-304)

**EFS:**
- Gắn hệ thống tệp EFS vào ECS tasks; dùng được với EC2 và Fargate.
- Tasks ở các AZ khác nhau có thể chia sẻ cùng dữ liệu trên EFS.
- Fargate kết hợp EFS vẫn là mô hình serverless.
- Trường hợp sử dụng: lưu trữ bền vững dùng chung qua nhiều AZ.
- S3 không thể gắn như một hệ thống tệp.

**Bind Mounts:**
- Chia sẻ dữ liệu giữa nhiều containers trong cùng Task Definition; dùng được với EC2 và Fargate.
- Với EC2, dữ liệu gắn với vòng đời EC2 instance.
- Với Fargate, dùng ephemeral storage; mặc định 20 GiB, có thể cấu hình 20-200 GiB. Dữ liệu gắn với container sử dụng volume.
- Trường hợp sử dụng: chia sẻ dữ liệu tạm và sidecar gửi metrics/logs đến đích khác.

### Tự động mở rộng ECS Service (Trang 305-307)

- Tự động tăng/giảm số lượng ECS tasks mong muốn bằng Application Auto Scaling.
- Chỉ số: ECS Service Average CPU Utilization, ECS Service Average Memory Utilization và ALB Request Count Per Target.
- **Target Tracking** mở rộng dựa trên giá trị mục tiêu của CloudWatch metric.
- **Step Scaling** mở rộng theo CloudWatch Alarm.
- **Scheduled Scaling** mở rộng theo ngày/giờ đã định.
- ECS Service Auto Scaling thay đổi số lượng tasks; EC2 Auto Scaling thay đổi số lượng EC2 instances.
- Thiết lập tự động mở rộng trên Fargate đơn giản hơn vì không phải quản lý EC2 instances.
- Với EC2 Launch Type, Auto Scaling Group hoặc ECS Cluster Capacity Provider có thể thêm EC2 instances để đáp ứng nhu cầu tasks. Capacity Provider kết hợp với Auto Scaling Group và thêm instances khi thiếu CPU/RAM.

> *Sơ đồ (Trang 307): ECS Service được mở rộng dựa trên mức sử dụng CPU.*

### Cập nhật rolling cho ECS (Trang 308-310)

- Khi nâng cấp từ v1 lên v2, có thể kiểm soát số tasks bắt đầu/dừng và thứ tự thực hiện.
- Trang 309 minh họa cấu hình **Minimum healthy percent 50%**, **Maximum percent 100%**, với 4 tasks ban đầu.
- Trang 310 minh họa cấu hình **Minimum healthy percent 100%**, **Maximum percent 150%**, với 4 tasks ban đầu.

### ECS Tasks được gọi bằng EventBridge (Trang 311-312)

> *Sơ đồ (Trang 311): EventBridge gọi ECS Tasks.*
>
> *Sơ đồ (Trang 312): EventBridge gọi ECS Tasks theo lịch.*

### ECS và SQS (Trang 313)

> *Sơ đồ: SQS Queue chứa thông điệp; các tasks trong ECS Service lần lượt thăm dò queue để nhận thông điệp. ECS Service Auto Scaling điều chỉnh số lượng tasks.*

### Theo dõi tasks bị dừng bằng EventBridge (Trang 314)

> *Sơ đồ: Dùng EventBridge để nhận diện các ECS tasks đã dừng.*

### Cân bằng tải cho ECS (Trang 315-316)

**EC2 Launch Type:**
- Nếu chỉ khai báo container port trong Task Definition, ECS dùng ánh xạ host port động.
- ALB tìm đúng port trên EC2 instance.
- Security Group của EC2 instance cần cho phép lưu lượng từ Security Group của ALB đến các port được dùng.

**Fargate:**
- Mỗi task có private IP riêng; chỉ khai báo container port, không áp dụng host port.
- ECS ENI Security Group cho phép port 80 từ ALB.
- ALB Security Group cho phép port 80/443 từ các client web.

### Biến môi trường trong ECS (Trang 317)

- URL có thể được khai báo trực tiếp.
- SSM Parameter Store dùng cho biến nhạy cảm như API keys hoặc cấu hình dùng chung.
- Secrets Manager dùng cho dữ liệu nhạy cảm như DB passwords.
- Environment Files chứa nhiều cấu hình có thể được lưu ở S3.

### Amazon ECR (Trang 318-319)

- **ECR** là Elastic Container Registry, dùng để lưu trữ và quản lý Docker images trên AWS.
- Có kho riêng tư và kho công khai; tích hợp với ECS và được hỗ trợ bởi Amazon S3.
- Quyền truy cập được kiểm soát bằng IAM.
- Hỗ trợ quét lỗ hổng bảo mật của image, versioning, image tags và image lifecycle.
- Đăng nhập Docker bằng AWS CLI v2:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

- Dùng `docker push` để đẩy image và `docker pull` để tải image.
- Nếu EC2 instance hoặc người dùng không tải được image, hãy kiểm tra IAM permissions.

### AWS Copilot (Trang 320)

- **Công cụ CLI** để xây dựng, phát hành và vận hành ứng dụng containers sẵn sàng cho production.
- Chạy ứng dụng trên AppRunner, ECS và Fargate.
- Cấp phát hạ tầng cần thiết cho ứng dụng containers, như ECS, VPC, ELB và ECR.
- Hỗ trợ triển khai tự động bằng một lệnh qua CodePipeline, triển khai đến nhiều môi trường, xử lý sự cố, xem logs và trạng thái health.

---

## Phần 3: Amazon EKS (Trang 321-324)

### Tổng quan Amazon EKS (Trang 321-322)

- **Amazon EKS** (Amazon Elastic Kubernetes Service) cung cấp Kubernetes clusters được quản lý trên AWS.
- **Kubernetes** là hệ thống mã nguồn mở để tự động triển khai, mở rộng và quản lý ứng dụng containers.
- EKS là lựa chọn thay thế ECS với mục tiêu tương tự nhưng API khác.
- EKS hỗ trợ worker nodes trên EC2 hoặc containers serverless trên Fargate.
- Trường hợp sử dụng: công ty đang dùng Kubernetes tại chỗ hoặc trên cloud khác và muốn chuyển sang AWS mà vẫn dùng Kubernetes.
- Kubernetes không phụ thuộc vào một cloud cụ thể.
- Khi triển khai nhiều Regions, tạo một EKS Cluster cho mỗi Region.
- Thu thập logs/metrics bằng CloudWatch Container Insights.

> *Sơ đồ (Trang 322): Amazon EKS Cluster.*

### Các loại node của EKS (Trang 323)

- **Managed Node Groups:** AWS tạo và quản lý các EC2 nodes; nodes thuộc ASG do EKS quản lý; hỗ trợ On-Demand hoặc Spot Instances.
- **Self-Managed Nodes:** Bạn tạo nodes, đăng ký chúng với cluster và quản lý bằng ASG; có thể dùng Amazon EKS Optimized AMI; hỗ trợ On-Demand hoặc Spot Instances.
- **AWS Fargate:** Không cần bảo trì hoặc quản lý nodes.

### Volumes dữ liệu trong EKS (Trang 324)

- Cần khai báo manifest Storage Class trên EKS Cluster và dùng driver tuân thủ Container Storage Interface (CSI).
- Các loại storage được hỗ trợ trong slide:
  - Amazon EBS.
  - Amazon EFS (hoạt động với Fargate).
  - Amazon FSx for Lustre.
  - Amazon FSx for NetApp ONTAP.

---

## Phần 4: Bài thực hành triển khai ứng dụng trên ECS (Trang 325-335)

### Sơ đồ kiến trúc (Trang 325-326)

> *Sơ đồ: Người dùng truy cập ứng dụng qua Route 53 và ALB trong VPC trải rộng hai AZ. ECR lưu image; VPC có các public subnets với Bastion/NAT Gateway và private subnets chứa EC2 instances. ECS Service chạy các tasks theo Task Definition.*

### Các bước triển khai (Trang 327-335)

1. Tạo VPC với public và private subnets ở hai AZ (Trang 327).
2. Tạo Bastion host (Trang 328).
3. Tạo ECR và xây dựng Docker image (Trang 329).
4. Tạo ECS Cluster và EC2 instances (Trang 330).
5. Tạo Task Definition (Trang 331).
6. Tạo Target Group và ALB (Trang 332).
7. Tạo ECS Service và các tasks (Trang 333).
8. Kiểm tra ứng dụng (Trang 334).
9. Cấu hình Route 53 (Trang 335).
