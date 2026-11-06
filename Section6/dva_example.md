# Section 6 - DVA Practice Questions: ECS, ECR, Fargate and EKS

---
## 1. Amazon ECS

### Question 1:
What is the basic unit of work in ECS?

```
A. Cluster
B. Service
C. Task Definition
D. Task
```

### ✅ Answer: **D. Task**

### 📖 Explanation (Vietnamese):

**Tại sao chọn D:**
- **Task** là đơn vị công việc đang chạy trong ECS và có thể chứa một hoặc nhiều containers.
- Task Definition là bản mô tả/cấu hình JSON; Task là lần chạy thực tế theo cấu hình đó.

**Phân cấp ECS:**
```
Cluster
  └── Service (duy trì các tasks)
        └── Tasks (các lần chạy theo Task Definition)
```

**Tại sao loại các lựa chọn khác:**

| Lựa chọn | Lý do |
|----------|-------|
| A. Cluster | Nhóm các services và tasks, không phải đơn vị chạy container |
| B. Service | Duy trì số lượng tasks mong muốn |
| C. Task Definition | Bản mô tả cách chạy task, không phải task đang chạy |
---

### Question 2:
In ECS, which IAM role is used by the ECS agent on EC2 instances?

```
A. ECS Task Role
B. EC2 Instance Profile
C. Service Role
D. Task Execution Role
```

### ✅ Answer: **B. EC2 Instance Profile**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Trên EC2 Launch Type, **EC2 Instance Profile** (container instance role) được ECS Agent sử dụng để giao tiếp với ECS.
- **Task execution role** cho phép ECS/Fargate Agent thực hiện thao tác khởi chạy như tải image từ ECR, gửi logs và đọc secrets được khai báo trong Task Definition.
- **Task role** cấp quyền AWS cho mã ứng dụng chạy bên trong container.

| Role | Bên sử dụng | Mục đích |
|------|-------------|----------|
| EC2 Instance Profile | ECS Agent trên EC2 | Giao tiếp giữa container instance và ECS |
| Task Execution Role | ECS/Fargate Agent khi khởi chạy task | ECR, logging, secrets injection |
| ECS Task Role | Mã ứng dụng trong container | Gọi AWS services thay mặt ứng dụng |

**Tại sao loại các lựa chọn khác:**
- A. ECS Task Role cấp quyền cho mã ứng dụng, không phải ECS Agent trên EC2.
- C. Service Role không phải role mà ECS Agent dùng để đăng ký/giao tiếp với ECS.
- D. Task Execution Role phục vụ thao tác khởi chạy, khác với container instance role.

---

### Question 3:
What are the two launch types supported by ECS?

```
A. EC2 and Fargate
B. EC2 and Lambda
C. Fargate and Lambda
D. EC2 and EKS
```

### ✅ Answer: **A. EC2 and Fargate**

### 📖 Explanation (Vietnamese):

**Tại sao chọn A:**
- ECS hỗ trợ **EC2 Launch Type** (bạn quản lý EC2 instances) và **Fargate Launch Type** (AWS quản lý hạ tầng).

| Đặc điểm | EC2 Launch Type | Fargate Launch Type |
|----------|-----------------|---------------------|
| Hạ tầng | Bạn quản lý | AWS quản lý |
| Mở rộng | EC2/ASG và tasks | Tăng số tasks |
| Security Group | Áp dụng theo cấu hình mạng; với `awsvpc` là theo task | Theo task |

**Tại sao loại các lựa chọn khác:**
- B và C. Lambda là dịch vụ compute khác, không phải ECS launch type.
- D. EKS là dịch vụ Kubernetes, không phải ECS launch type.
---

## 2. ECS Volumes and Fargate

### Question 4:
An ECS task's application container writes temporary scratch files that a sidecar container must read. The data should be shared only within this task, not stored in a persistent network file system. Which volume type should the task use?

```
A. EFS volumes
B. Bind Mounts
C. EBS volumes
D. S3 buckets
```

### ✅ Answer: **B. Bind Mounts**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- **Bind Mounts** chia sẻ dữ liệu giữa nhiều container trong cùng task.
- Với EC2, dữ liệu nằm trên ổ lưu trữ của EC2 instance; với Fargate, dữ liệu nằm trên vùng lưu trữ tạm thời (ephemeral storage).
- Mẫu sử dụng điển hình là sidecar chia sẻ nhật ký và chỉ số (metrics) với tiến trình khác.

| Loại volume | Dạng lưu trữ | Trường hợp sử dụng |
|-------------|-------------|---------------------|
| Bind Mounts | Ổ cục bộ hoặc lưu trữ tạm (ephemeral storage) | Chia sẻ dữ liệu tạm giữa containers |
| EFS | Hệ thống tệp qua mạng | Lưu trữ bền vững dùng chung (Regional hoặc One Zone) |
| EBS | Lưu trữ dạng khối | Volume bền vững gắn với EC2 instance |
| S3 | Lưu trữ dạng đối tượng | Truy cập object qua API, không gắn như hệ thống tệp |

**Tại sao loại các lựa chọn khác:**
- A. EFS cũng có thể được gắn vào nhiều containers, nhưng đây là hệ thống tệp dùng chung qua mạng, lưu trữ bền vững; đề bài yêu cầu dữ liệu tạm chỉ trong phạm vi một task.
- C. EBS là lưu trữ dạng khối; D. S3 là lưu trữ dạng đối tượng.

---

### Question 5:
A company wants to run containers without managing any EC2 instances. Which ECS launch type should be used?

```
A. EC2 Launch Type
B. Fargate Launch Type
C. Both EC2 and Fargate
D. EKS with Fargate
```

### ✅ Answer: **B. Fargate Launch Type**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- **Fargate Launch Type** chạy containers mà không cần quản lý EC2 instances.
- Bạn chỉ định CPU/RAM; AWS quản lý hạ tầng bên dưới.
- Mở rộng bằng cách tăng số tasks.

**Đặc điểm:** không cần provision EC2 instances; tính phí theo tài nguyên task sử dụng; phù hợp khi ưu tiên giảm công việc quản lý hạ tầng.

**Tại sao loại các lựa chọn khác:**
- A. EC2 Launch Type yêu cầu quản lý EC2 instances.
- C. Lựa chọn cả hai không đáp ứng yêu cầu không quản lý EC2.
- D. EKS là Kubernetes service, không phải ECS launch type.

---

## 3. Amazon ECR

### Question 6:
A developer needs to push a Docker image to Amazon ECR. Which command authenticates Docker to ECR?

```
A. aws ecr login
B. aws ecr get-login-password | docker login
C. docker login -u AWS
D. aws ecr authorize
```

### ✅ Answer: **B. aws ecr get-login-password | docker login**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- AWS CLI v2 lấy mật khẩu đăng nhập ECR; Docker nhận mật khẩu qua `--password-stdin`:
```bash
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com
```
- Sau khi đăng nhập, dùng `docker push` để đẩy image hoặc `docker pull` để tải image.

**Tại sao loại các lựa chọn khác:**
- A. `aws ecr login` và D. `aws ecr authorize` không phải lệnh đăng nhập ECR.
- C. Cần truyền mật khẩu lấy từ AWS CLI bằng `--password-stdin`.

---

### Question 7:
Which storage backend does Amazon ECR use?

```
A. Amazon EBS
B. Amazon S3
C. Amazon EFS
D. Amazon DynamoDB
```

### ✅ Answer: **B. Amazon S3**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- ECR lưu trữ và quản lý Docker images; Amazon S3 là lớp lưu trữ hỗ trợ phía sau.
- Image gồm các layers, manifest và metadata.

**Tại sao loại các lựa chọn khác:**
- A. EBS là block storage; C. EFS là file system; D. DynamoDB là NoSQL database. Chúng không phải backend storage được slide nêu cho ECR.

---
## 4. Amazon EKS

### Question 8:
Which AWS service is used to launch managed Kubernetes clusters on AWS?

```
A. Amazon ECS
B. Amazon EKS
C. Amazon ECR
D. AWS Fargate
```

### ✅ Answer: **B. Amazon EKS**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- **Amazon EKS** cung cấp Kubernetes clusters được AWS quản lý.
- **Kubernetes** là hệ thống mã nguồn mở để triển khai, mở rộng và quản lý ứng dụng containers.
- EKS là lựa chọn thay thế ECS với mục tiêu tương tự nhưng API khác.

| Dịch vụ | Vai trò |
|---------|---------|
| ECS | Nền tảng container của AWS |
| EKS | Kubernetes được quản lý |
| Fargate | Compute serverless cho containers |
| ECR | Registry lưu trữ container images |

**Tại sao loại các lựa chọn khác:**
- A. ECS là nền tảng container của AWS, không phải Kubernetes.
- C. ECR lưu trữ images, không điều phối containers.
- D. Fargate cung cấp compute serverless, không phải Kubernetes service.

---
## 5. ECS workers consuming SQS

### Question 9:
A team processes SQS messages with container workers and needs to scale the number of workers as the queue backlog changes. Which design is appropriate?

```
A. Use EventBridge Scheduler to start a worker task at a fixed time, regardless of queue backlog.
B. Run workers in an ECS Service; each task polls SQS, and scale the service using queue backlog per task.
C. Put the SQS queue URL in the Task Definition so SQS pushes messages to containers.
D. Use an ALB to distribute messages from SQS to ECS tasks.
```

### ✅ Answer: **B. Run workers in an ECS Service and scale from queue backlog per task**

### 📖 Explanation (Vietnamese):

**Tại sao chọn B:**
- Mỗi task trong ECS Service chạy worker code để nhận messages từ SQS, xử lý thành công rồi xóa messages.
- ECS Service Auto Scaling tăng/giảm số worker tasks khi lượng công việc thay đổi.
- Nên dựa trên backlog trên mỗi task; chỉ dùng queue depth có thể làm tăng số tasks quá mức so với nhu cầu.

**Kiến trúc:**
```
Producer → SQS Queue ← workers trong ECS Service (poll)
                         ↑
        Service Auto Scaling theo backlog/task
```

**Tại sao loại các lựa chọn khác:**
- A. EventBridge Scheduler chạy theo lịch định sẵn; nó không điều chỉnh số worker tasks theo queue backlog.
- C. Task Definition không đăng ký consumer hoặc khiến SQS tự đẩy messages vào container.
- D. ALB định tuyến HTTP requests, không phân phối SQS messages.

**Tài liệu AWS:** [Scale Amazon ECS services based on an Amazon SQS queue](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-queue.html).

## 6. ECS Secrets
### Question 10:
An application needs to store sensitive configuration data (API keys) and make it available to ECS tasks. Which service should be used?

```
A. Amazon S3
B. Amazon DynamoDB
C. AWS Secrets Manager
D. Amazon CloudWatch
```

### ✅ Answer: **C. AWS Secrets Manager**

### 📖 Explanation (Vietnamese):

**Tại sao chọn C:**
- **Secrets Manager** lưu trữ dữ liệu nhạy cảm như API keys và DB passwords.
- Có thể khai báo secret trong Task Definition để ECS đưa giá trị vào container khi khởi chạy task.
- Dùng SSM Parameter Store khi phù hợp với loại cấu hình; dùng secrets thay vì hardcode credentials.

| Lựa chọn | Lý do loại trừ |
|----------|----------------|
| A. S3 | Lưu trữ object, không phải dịch vụ quản lý secret |
| B. DynamoDB | Cơ sở dữ liệu, không phải nơi quản lý secret |
| D. CloudWatch | Dịch vụ giám sát và ghi logs |

---
