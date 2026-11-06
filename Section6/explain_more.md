# Section 6 - Kiến thức mở rộng: ECS, ECR, Fargate và EKS

---
## 1. Mẫu kiến trúc ECS

### Thực hành tốt với ECS Task Definition

#### Giải thích cốt lõi
Task Definition là bản mô tả có phiên bản về cách ECS chạy một hoặc nhiều containers: image, CPU/bộ nhớ, ports, biến môi trường, role và logs. Task là lần chạy thực tế theo bản mô tả đó; so với cấu hình thủ công trên từng máy, Task Definition giúp tạo lại tasks nhất quán.

#### NÊN làm (DO)

- Khai báo log và cấu hình secret reference trong Task Definition → Giúp chẩn đoán và tránh ghi credentials trực tiếp vào image; so với biến môi trường chứa giá trị bí mật rõ, dùng `secrets` để lấy giá trị từ SSM Parameter Store/Secrets Manager.

```json
{
  "containerDefinitions": [{
    "name": "web-app",
    "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/web:latest",
    "memory": 512,
    "cpu": 256,
    "essential": true,
    "portMappings": [{
      "containerPort": 80,
      "protocol": "tcp"
    }],
    "environment": [
      {"name": "ENV", "value": "production"}
    ],
    "secrets": [
      {"name": "API_KEY", "valueFrom": "arn:aws:ssm:us-east-1:123456789012:parameter/ecs/API_KEY"}
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/web-app",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }],
  "family": "web-app-task"
}
```

#### KHÔNG NÊN làm (DON'T)
- **Ghi API key hoặc password trực tiếp vào `environment` hay image** → Giá trị bí mật có thể lộ khi xem cấu hình hoặc image. NÊN: tham chiếu secret bằng `secrets` và cấp quyền tối thiểu cho role phù hợp.


### Bảo mật ECS - IAM Roles

#### Giải thích cốt lõi
- **EC2 Instance Profile (container instance role)** được ECS Agent dùng trên EC2 container instances để giao tiếp với ECS.
- **Task Execution Role** được ECS/Fargate Agent dùng khi khởi chạy task, chẳng hạn tải Docker image từ ECR, ghi logs vào CloudWatch Logs và lấy secrets từ Secrets Manager hoặc SSM Parameter Store để đưa vào container.
- **Task Role** cấp AWS credentials cho mã ứng dụng chạy trong container.

| IAM Role | Bên sử dụng | Mục đích |
|----------|-------------|----------|
| EC2 Instance Profile | ECS Agent trên EC2 | Giao tiếp với ECS và quản lý instance chạy containers |
| Task Execution Role | ECS/Fargate Agent khi khởi chạy task | Tải image từ ECR, ghi logs, đọc secrets |
| Task Role | Ứng dụng trong container | Gọi AWS services thay mặt ứng dụng |
- Ví dụ giới hạn `dynamodb:Query` vào đúng bảng bằng ARN `arn:aws:dynamodb:us-east-1:123456789012:table/my-table`; với `s3:GetObject`, tài nguyên có dạng `arn:aws:s3:::my-bucket/*`.

#### NÊN làm (DO)
- **Cấp role riêng với quyền tối thiểu và khai báo đúng trường** → Tách quyền agent lúc khởi chạy khỏi quyền của ứng dụng; so với dùng EC2 Instance Profile cho mọi quyền, Task Role giới hạn ảnh hưởng khi một container bị xâm nhập.

#### KHÔNG NÊN làm (DON'T)
- Dùng EC2 Instance Profile để cấp quyền nghiệp vụ cho mọi container trên host; dùng Task Role cho quyền của ứng dụng.
- Nhầm Task Role với Task Execution Role: một role dành cho mã ứng dụng, role còn lại dành cho thao tác khởi chạy task.

Tài liệu AWS: [IAM Role cho ECS Task](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) · [IAM Role thực thi ECS Task](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

### Các chế độ mạng của ECS

#### Giải thích cốt lõi
Network mode quyết định cách container/task dùng network stack và nhận địa chỉ IP/port. `awsvpc` cấp ENI/IP riêng cho task; `bridge` ánh xạ qua host; `host` chia sẻ network stack của host. Fargate yêu cầu `awsvpc`; trên EC2, chọn mode theo mức cô lập và nhu cầu hiệu năng.

#### NÊN làm (DO)

- Dùng `awsvpc` cho Fargate hoặc khi cần IP/Security Group riêng theo task; trên EC2 chỉ chọn `bridge`/`host` khi yêu cầu ánh xạ port hay hiệu năng phù hợp với đánh đổi cô lập.

| Chế độ | Mô tả | Trường hợp sử dụng |
|--------|-------|--------------------|
| **awsvpc** | Mỗi task có ENI và private IP | Fargate, ứng dụng hiện đại |
| **bridge** | Docker bridge trên host | EC2, ánh xạ port |
| **host** | Container dùng network stack của host | Ứng dụng cần hiệu năng cao |
| **none** | Không cấu hình network | Tác vụ cô lập |

**awsvpc trên Fargate:**
- ENI được gắn vào task.
- Security Group được áp dụng cho task.
- Tránh nhầm lẫn port.

#### KHÔNG NÊN làm (DON'T)
- **Dùng `host` mode khi tasks cần cùng port hoặc cô lập mạng riêng** → Các containers dùng chung network stack và có thể xung đột port. NÊN: chọn `awsvpc` để mỗi task có ENI và Security Group riêng, hoặc `bridge` khi phù hợp với ánh xạ port.

### Tự động mở rộng ECS Service

#### Giải thích cốt lõi
ECS Service Auto Scaling thay đổi số tasks mong muốn theo CloudWatch metrics. Target Tracking giữ metric gần mục tiêu; Step Scaling phản ứng theo mức alarm; Scheduled Scaling phù hợp với tải dự đoán trước. Scaling tasks không tự đồng nghĩa với tăng số EC2 container instances.

#### NÊN làm (DO)

**Chính sách Target Tracking:**
```json
{
  "TargetTrackingScalingPolicyConfiguration": {
    "TargetValue": 60.0,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    }
  }
}
```
- Chọn metric phản ánh nút thắt (CPU, memory hoặc ALB requests per target) và Target Tracking cho tải biến động; so với lịch cố định, metric giúp phản ứng theo nhu cầu thực tế.


**💡 Mẹo: Chỉ số dùng để mở rộng**
| Chỉ số | Phù hợp với |
|--------|-------------|
| CPU | Ứng dụng web thông thường |
| Memory | Ứng dụng cần nhiều bộ nhớ |
| ALB Request Count | Ứng dụng cân bằng tải |

#### KHÔNG NÊN làm (DON'T)
- **Chỉ scale tasks mà không bảo đảm EC2 capacity cho EC2 launch type** → Tasks có thể ở trạng thái không chạy vì thiếu CPU/bộ nhớ. NÊN: kết hợp Capacity Provider/Auto Scaling Group hoặc chọn Fargate khi muốn AWS quản lý capacity.
- **Dùng Scheduled Scaling cho tải khó dự đoán** → Service có thể thiếu hoặc dư tasks giữa các khung giờ. NÊN: dùng Target Tracking hoặc Step Scaling theo metric/alarm.

### Cập nhật rolling cho ECS

#### Giải thích cốt lõi
ECS rolling deployment thay tasks theo tỷ lệ Minimum healthy percent và Maximum percent: cấu hình đặt giới hạn số task giữ khỏe và số task có thể tồn tại trong lúc triển khai. So với thay toàn bộ một lần, rolling chia nhỏ rủi ro nhưng cần đủ capacity để chạy phiên bản mới.

#### NÊN làm (DO)

- Chọn Minimum healthy percent và Maximum percent theo SLA/khả năng chạy song song → So với thay toàn bộ tasks cùng lúc, rolling giữ số task khỏe trong giới hạn đã tính trước.

**Cấu hình:**
- **Minimum healthy percent**: 100% nghĩa là không giảm số tasks trong lúc cập nhật.
- **Maximum percent**: 200% cho phép tăng số tasks lên gấp đôi trong lúc cập nhật.

**Ví dụ chiến lược:**
```
Min: 100%, Max: 200%
- Khởi chạy 4 tasks
- Khởi chạy 4 tasks mới dùng v2
- Chờ các tasks mới đạt health check
- Dừng 4 tasks cũ dùng v1
```

#### KHÔNG NÊN làm (DON'T)
- **Đặt Maximum percent thấp đến mức không còn chỗ khởi chạy task mới trong khi vẫn giữ Minimum healthy percent** → Deployment có thể không tiến triển. NÊN: tính trước số task và capacity khả dụng, rồi chọn tỷ lệ cho phép thay task.

---

## 2. Thực hành tốt với ECR

### Chính sách kho lưu trữ của ECR

#### Giải thích cốt lõi
ECR repository policy cấp quyền trực tiếp lên repository, hữu ích khi chia sẻ image giữa AWS accounts; identity-based IAM policy vẫn cần phù hợp ở phía caller. So với public repository, repository policy giới hạn principal và thao tác được phép.

#### NÊN làm (DO)

- Chỉ cho phép account/role cần thiết và các action push/pull phù hợp → Hạn chế người có thể tải hoặc sửa image so với policy mở rộng.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PushPull",
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": [
      "ecr:GetDownloadUrlForLayer",
      "ecr:BatchGetImage",
      "ecr:BatchCheckLayerAvailability",
      "ecr:PutImage",
      "ecr:UploadLayerPart",
      "ecr:InitiateLayerUpload",
      "ecr:CompleteLayerUpload"
    ]
  }]
}
```

#### KHÔNG NÊN làm (DON'T)
- **Cho phép `Principal: "*"` hoặc mọi action trên repository riêng** → Bên ngoài có thể đọc/ghi image ngoài ý muốn. NÊN: giới hạn principal theo account/role và action tối thiểu.

### Quản lý vòng đời image trong ECR

#### Giải thích cốt lõi
ECR Lifecycle Policy chọn image theo tag status, prefix hoặc tuổi/số lượng rồi tự hết hạn các image khớp rule. So với xóa thủ công, policy giữ repository gọn và kiểm soát dung lượng, nhưng rule sai có thể xóa image còn cần.

#### NÊN làm (DO)

- Kiểm tra selection và thời gian giữ trước khi áp dụng; ưu tiên thử rule trên image mẫu → Xác nhận image hiện hành/rollback vẫn còn, thay vì xóa theo một số ngày không kiểm chứng.

```json
{
  "rules": [{
    "rulePriority": 1,
    "description": "Expire untagged images",
    "selection": {
      "tagStatus": "untagged",
      "countType": "sinceImagePushed",
      "countUnit": "days",
      "countNumber": 7
    },
    "action": {
      "type": "expire"
    }
  }]
}
```

#### KHÔNG NÊN làm (DON'T)
- **Đặt rule hết hạn image có tag đang được production dùng mà không có bản thay thế** → Task mới có thể không tải được image hoặc rollback thất bại. NÊN: dùng tag/version rõ ràng, giữ image đang triển khai và thử rule trước.

---

## 3. Chi tiết Fargate

### Phân bổ tài nguyên Fargate

#### Giải thích cốt lõi
Fargate chỉ nhận các cặp CPU/bộ nhớ được nền tảng và hệ điều hành hỗ trợ; dung lượng ephemeral storage cũng có mức mặc định và giới hạn cấu hình. So với EC2, người dùng không chọn host instance type mà phải chọn task size tương thích.

#### NÊN làm (DO)

| CPU | Tùy chọn bộ nhớ |
|-----|-----------------|
| 0.25 vCPU | 0.5, 1, 2 GB (Linux) |
| 0.5 vCPU | 1, 2, 3, 4 GB (Linux) |
| 1 vCPU | 2-8 GB (Linux/Windows) |
| 2 vCPU | 4-16 GB, bước 1 GB (Linux/Windows) |
| 4 vCPU | 8-30 GB, bước 1 GB (Linux/Windows) |
| 8 vCPU | 16-60 GB, bước 4 GB (Linux platform 1.4+) |
| 16 vCPU | 32-120 GB, bước 8 GB (Linux platform 1.4+) |
| 32 vCPU | 60, 120 hoặc 244 GB (Linux platform 1.4+) |

Cập nhật 06/11/2026: các cặp CPU/bộ nhớ của Fargate phụ thuộc hệ điều hành và platform version; Linux platform 1.4+ hỗ trợ task 8, 16 và 32 vCPU với giới hạn bộ nhớ tương ứng trong bảng. Kiểm tra bảng tương thích trước khi đăng ký Task Definition. [AWS Fargate task size](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#task_size).
- Xác thực cặp CPU/bộ nhớ và platform version trước khi đăng ký Task Definition → Tránh lỗi cấu hình; so với đoán theo dung lượng mong muốn, bảng hỗ trợ của Fargate xác định các tổ hợp hợp lệ.

**Lưu trữ của task:**
- Ephemeral storage 20 GiB theo mặc định.
- Có thể tăng lên đến 200 GiB bằng cấu hình bổ sung.
- Dùng EFS cho lưu trữ bền vững.

#### KHÔNG NÊN làm (DON'T)
- **Giả định mọi cặp CPU/bộ nhớ đều hợp lệ hoặc ephemeral storage là lưu trữ bền vững** → Task có thể không khởi chạy hoặc mất dữ liệu khi kết thúc. NÊN: kiểm tra compatibility matrix và dùng EFS cho dữ liệu cần tồn tại lâu dài.

### Fargate và EC2 Launch Type

#### Giải thích cốt lõi
Fargate chạy tasks mà không cần người dùng quản lý EC2 hosts; EC2 Launch Type cho phép kiểm soát host, AMI, capacity và cấu hình instance. Đổi lại, Fargate giới hạn quyền kiểm soát host và tính phí theo task, còn EC2 yêu cầu vận hành hạ tầng nhưng có thể phù hợp workload cần tùy chỉnh.

#### NÊN làm (DO)

| Yếu tố | Fargate | EC2 Launch Type |
|--------|---------|-----------------|
| **Quản lý** | AWS quản lý hoàn toàn | Bạn quản lý |
| **Chi phí** | Trả theo task | Trả cho EC2 instances |
| **Mở rộng** | Nhanh | Mất thời gian hơn |
| **Khả năng tùy chỉnh** | Hạn chế | Toàn quyền kiểm soát |
| **Spot** | Fargate Spot qua capacity provider | EC2 Spot Instances |
| **Reserved Instances** | Không áp dụng cho Fargate | Có |
| **Savings Plans** | Compute Savings Plans có thể áp dụng | Compute/EC2 Instance Savings Plans |

Cập nhật 06/11/2026: Fargate Spot sử dụng dung lượng dự phòng với mức giá thấp hơn; task có thể nhận thông báo gián đoạn trước hai phút, vì vậy chỉ dùng cho khối lượng công việc chịu được gián đoạn. Chọn Fargate Spot qua capacity provider. [Fargate capacity providers](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-capacity-providers.html).
- Chọn Fargate khi ưu tiên giảm vận hành; chọn EC2 khi cần AMI/host customization hoặc tối ưu capacity bằng Spot. So với chọn theo thói quen, đối chiếu yêu cầu kiểm soát, tính gián đoạn và mô hình chi phí.

**Khi nào dùng Fargate:**
- Tasks chạy trong thời gian ngắn.
- Khối lượng công việc khó dự đoán.
- Microservices.
- Khi muốn giảm công việc quản lý hạ tầng.

**Khi nào dùng EC2:**
- Cần các kết nối duy trì lâu.
- Cần tiết kiệm chi phí bằng Spot/RI.
- Cần AMIs tùy chỉnh.
- Cần lưu trữ cục bộ bền vững.

#### KHÔNG NÊN làm (DON'T)
- **Cho rằng Fargate luôn rẻ hơn hoặc phù hợp mọi workload** → Chi phí và giới hạn host có thể không khớp nhu cầu. NÊN: ước tính theo task usage và so sánh với EC2 capacity cho workload ổn định, cần tùy chỉnh.

---

## 4. Kiến trúc EKS

### Nhóm node EKS và Fargate

#### Giải thích cốt lõi
EKS hỗ trợ Managed Node Groups (AWS quản lý EC2 node group), Self-Managed Nodes (người dùng quản lý nodes/ASG) và Fargate (không quản lý node). Node group cho phép dùng EC2/EBS và kiểm soát host; Fargate giảm vận hành nhưng không hỗ trợ mọi loại storage/workload. Chọn theo nhu cầu quản trị và storage.

#### NÊN làm (DO)

| Đặc điểm | Managed Node Group | Fargate |
|----------|-------------------|---------|
| Mức quản lý | Một phần | Không |
| Truy cập SSH | Có | Không |
| Lưu trữ bền vững | EBS, EFS | Chỉ EFS |
| Spot Instances | Có | Không |
| Yêu cầu VPC | Thông thường | Cần nhiều ENIs hơn |
- **Chọn Managed Node Groups khi cần EC2 nhưng muốn AWS quản lý vòng đời node; chọn Self-Managed khi cần kiểm soát AMI/cấu hình; chọn Fargate khi không cần node riêng** → So sánh mức kiểm soát và công vận hành để tránh quản lý thừa hoặc thiếu khả năng tùy chỉnh.

#### KHÔNG NÊN làm (DON'T)
- **Chọn Fargate khi workload phụ thuộc storage hoặc cấu hình node không được hỗ trợ** → Pod có thể không được lập lịch hoặc không gắn được volume. NÊN: kiểm tra yêu cầu workload; dùng EBS/EFS và node group phù hợp.

### Lưu trữ EKS với CSI

#### Giải thích cốt lõi
CSI (Container Storage Interface) chuẩn hóa cách Kubernetes yêu cầu storage driver cấp volume. EBS CSI Driver cung cấp block volume gắn vào EC2 node; EFS phù hợp dữ liệu chia sẻ qua nhiều AZ và được hỗ trợ cho Fargate. So với ephemeral storage, volume CSI có thể tồn tại theo vòng đời của storage resource.

#### NÊN làm (DO)

- Chọn EBS cho workload cần block storage gắn với node/AZ; chọn EFS khi cần file system dùng chung → So với ephemeral storage, CSI volume phù hợp hơn cho dữ liệu cần tồn tại theo vòng đời storage resource.

**Cài đặt EBS CSI Driver:**
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster my-cluster
```

**StorageClass:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
```

#### KHÔNG NÊN làm (DON'T)
- **Dùng EBS như volume chia sẻ đồng thời giữa các AZ hoặc kỳ vọng ephemeral storage bền vững** → Pod không truy cập được volume theo cách mong muốn hoặc dữ liệu mất cùng task. NÊN: dùng EFS khi cần file system chia sẻ và dùng driver CSI phù hợp với storage class.

---
## 5. Cập nhật AWS (2026-11-06)

### Worker ECS tiêu thụ SQS

#### Giải thích cốt lõi
Với mô hình xử lý liên tục, các tasks worker trong ECS Service thăm dò SQS, xử lý thông điệp rồi xóa sau khi xử lý thành công. ECS Service Auto Scaling điều chỉnh số lượng tasks theo tải hàng đợi; SQS không tự gọi một ECS Service task cho mỗi thông điệp.

#### NÊN làm (DO)
- **Mở rộng theo lượng công việc tồn trên mỗi task** → Chia `ApproximateNumberOfMessagesVisible` cho số tasks đang chạy để đo lượng công việc trung bình mỗi worker. So với chỉ dùng độ sâu hàng đợi, chỉ số này xét cả năng suất worker và độ trễ chấp nhận được.
- **Bật CloudWatch Container Insights nếu chính sách cần `RunningTaskCount`** → Chỉ số này làm mẫu số khi tính lượng công việc tồn trên mỗi task; nếu thiếu dữ liệu, phép tính mở rộng sẽ không hoạt động.
- **Bảo vệ task đang xử lý khi giảm số lượng tasks nếu công việc kéo dài hoặc không lũy đẳng** → Tránh dừng task giữa chừng khiến thông điệp phải xử lý lại.

#### KHÔNG NÊN làm (DON'T)
- **Giả định SQS tự khởi chạy một ECS task cho mỗi thông điệp** → Worker có thể không thăm dò hàng đợi liên tục và cách mở rộng không phản ánh lượng công việc tồn. NÊN: chạy worker trong ECS Service, để mỗi task thăm dò SQS và mở rộng service theo tải.
- **Chỉ mở rộng theo độ sâu hàng đợi mà không xét số worker hoặc thời gian xử lý** → Có thể mở rộng quá mức và tăng chi phí. NÊN: dùng lượng công việc tồn trên mỗi task với target phù hợp SLA.

Tài liệu AWS: [Mở rộng ECS Service theo hàng đợi SQS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-queue.html).

### Target Tracking cho ECS Service

#### Giải thích cốt lõi
Target Tracking điều chỉnh số tasks để duy trì CloudWatch metric gần giá trị mục tiêu. Với CPU, `ECSServiceAverageCPUUtilization` là predefined metric; policy phải gắn vào scalable target của ECS Service. So với Step Scaling theo alarm, Target Tracking tự điều chỉnh theo sai lệch metric.

#### NÊN làm (DO)
- Gắn policy vào đúng scalable target và chọn target phù hợp với hành vi workload → Tránh điều chỉnh nhầm tài nguyên; theo dõi metric để cân bằng độ trễ và số tasks.

#### KHÔNG NÊN làm (DON'T)
- **Dùng metric không phản ánh tải hoặc bỏ qua scalable target** → Policy không scale đúng service hoặc không xử lý nút thắt. NÊN: kiểm tra resource ID, dimension và metric trước khi bật policy.

Tài liệu AWS: [Tạo chính sách target tracking cho tự động mở rộng ECS Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/target-tracking-create-policy.html).
