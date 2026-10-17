# Section 8 - AWS CloudFormation (Trang 385-432)

---

## Phần 1: CloudFormation - Tổng quan (Trang 385-391)

### AWS CloudFormation - Giới thiệu (Trang 387)

**CloudFormation là gì?**
- **Declarative way** (cách tiếp cận khai báo) để định nghĩa AWS Infrastructure
- Hỗ trợ hầu hết các AWS resources
- Bạn khai báo trong template:
  - Security group
  - Hai EC2 instances sử dụng security group
  - Hai Elastic IPs cho các EC2 instances
  - S3 bucket
  - Load Balancer (ELB) trước các EC2 instances

**CloudFormation tự động:**
- Tạo resources theo **đúng thứ tự**
- Với **cấu hình chính xác** như đã khai báo

### Lợi ích của AWS CloudFormation (1/2) (Trang 389)

**Infrastructure as Code - Hạ tầng như code:**
- Không tạo resources thủ công - kiểm soát tốt hơn
- Code có thể được **version control** (Git)
- Thay đổi infrastructure được review qua code

**Chi phí:**
- Mỗi resource trong stack được tag với một identifier → dễ theo dõi chi phí
- Có thể ước tính chi phí qua CloudFormation template
- **Savings strategy**: Tự động xóa templates lúc 5 PM và tạo lại lúc 8 AM (môi trường Dev)

### Lợi ích của AWS CloudFormation (2/2) (Trang 390)

**Productivity - Năng suất:**
- Khả năng destroy và re-create infrastructure trên cloud một cách nhanh chóng
- **Auto-generate Diagram** cho templates
- **Declarative programming** - không cần lo về ordering và orchestration

**Separation of Concern - Phân tách trách nhiệm:**
- Tạo nhiều stacks cho nhiều apps, nhiều layers
- Ví dụ: VPC stacks, Network stacks, App stacks

**Don't re-invent the wheel:**
- Tận dụng existing templates trên web
- Tận dụng documentation

### How CloudFormation Works (Trang 391)

- Templates phải được **upload lên S3** rồi mới reference trong CloudFormation
- Để update template, **không thể edit trực tiếp** - phải re-upload version mới
- Stacks được identify bằng **name**
- **Deleting a stack** → xóa toàn bộ artifacts đã được tạo bởi CloudFormation

---

## Phần 2: CloudFormation Building Blocks (Trang 392-399)

### CloudFormation - Building Blocks (Trang 393)

**Template's Components - Các thành phần chính:**

| Component | Mô tả |
|-----------|-------------|
| `AWSTemplateFormatVersion` | Identifies capabilities của template (vd: "2010-09-09") |
| `Description` | Comments về template |
| `Resources` (**MANDATORY**) | AWS resources được khai báo trong template |
| `Parameters` | Dynamic inputs cho template |
| `Mappings` | Static variables cho template |
| `Outputs` | References đến những gì đã được tạo |
| `Conditionals` | List of conditions để thực hiện việc tạo resource |

**Template's Helpers:**
- **References**
- **Functions**

### CloudFormation - Resources (Trang 396)

- **Resources** là core của CloudFormation template (**BẮT BUỘC**)
- Đại diện cho các AWS Components sẽ được tạo và cấu hình
- Resources được khai báo và có thể reference lẫn nhau
- AWS tự xử lý creation, updates và deletes
- **Hơn 700 types** của resources
- Resource type identifier có format: `service-provider::service-name::data-type-name`
  - Ví dụ: `AWS::EC2::Instance`

### Tìm Resources Documentation (Trang 397)

- Không thể học hết 700+ resources, nhưng có thể học cách sử dụng
- Tất cả resources tại: AWS CloudFormation User Guide
- Đọc docs cho từng resource
- Ví dụ: AWS::EC2::Instance documentation

### CloudFormation - Resources FAQ (Trang 399)

**Có thể tạo dynamic number of resources không?**
- Có, sử dụng **CloudFormation Macros** và **Transform**
- Ngoài phạm vi khóa học

**Mọi AWS Service đều được hỗ trợ?**
- Gần như tất cả
- Một số niche services chưa có → workaround bằng **CloudFormation Custom Resources**

---

## Phần 3: CloudFormation - Parameters (Trang 400-405)

### CloudFormation - Parameters (Trang 400)

**Parameters là gì?**
- Cung cấp inputs cho CloudFormation template
- Quan trọng khi:
  - Muốn **reuse templates** across company
  - Một số inputs không thể xác định trước
- Parameters rất powerful, controlled, và ngăn ngừa errors nhờ **types**

### Khi nào sử dụng Parameter? (Trang 401)

- Tự hỏi: "Configuration này có khả năng thay đổi trong tương lai không?"
- Nếu **CÓ** → làm nó thành parameter
- Lợi ích: Không cần re-upload template để thay đổi nội dung

### CloudFormation - Parameters Settings (Trang 402)

**Parameters được kiểm soát bởi:**

**Type:**
- `String`
- `Number`
- `CommaDelimitedList`
- `List<Number>`
- `AWS-Specific Parameter` (giúp bắt invalid values - match với existing values trong AWS account)
- `List<AWS-Specific Parameter>`

**Các settings khác:**
- `Description` (String)
- `ConstraintDescription` (String)
- `Min/MaxLength`
- `Min/MaxValue`
- `Default`
- `AllowedValues` (array)
- `AllowedPattern` (regex)
- `NoEcho` (Boolean)

**SSM Parameter:**
- Lấy parameter value từ **SSM Parameter Store**

### How to Reference a Parameter? (Trang 404)

- Hàm `Fn::Ref` được dùng để reference parameters
- Parameters có thể được sử dụng ở bất kỳ đâu trong template
- Shorthand trong YAML: `!Ref`
- Hàm này cũng có thể reference các elements khác trong template

### CloudFormation - Pseudo Parameters (Trang 405)

- AWS cung cấp **Pseudo Parameters** trong mọi CloudFormation template
- Có thể sử dụng bất kỳ lúc nào, enabled by default
- Pseudo parameters quan trọng:
  - `AWS::Region`
  - `AWS::StackName`
  - `AWS::StackId`
  - `AWS::AccountId`
  - `AWS::NoValue`

---

## Phần 4: CloudFormation - Mappings (Trang 406-408)

### CloudFormation - Mappings (Trang 406)

- Mappings là **fixed variables** trong CloudFormation template
- Rất hữu ích để phân biệt giữa:
  - Các environments (dev vs prod)
  - Regions (AWS regions)
  - AMI types
- Tất cả values được **hardcoded** trong template

### Accessing Mapping Values - Fn::FindInMap (Trang 407)

- Sử dụng `Fn::FindInMap` để lấy named value từ một key cụ thể
- Format: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`
- Mappings rất phù hợp với AMIs vì AMIs là region-specific

### Khi nào dùng Mappings vs. Parameters? (Trang 408)

**Mappings phù hợp khi:**
- Biết trước tất cả các values có thể có
- Có thể suy ra từ: Region, Availability Zone, AWS Account, Environment (dev vs prod)
- Cho phép kiểm soát template an toàn hơn

**Sử dụng Parameters khi:**
- Values thực sự user-specific

---

## Phần 5: CloudFormation - Outputs (Trang 409-411)

### CloudFormation - Outputs (Trang 409-410)

- **Outputs section** khai báo các output values tùy chọn
- Có thể **import vào các stacks khác** (nếu export trước)
- Có thể xem outputs trong AWS Console hoặc AWS CLI
- Rất hữu ích khi:
  - Định nghĩa network CloudFormation
  - Output các variables như **VPC ID**, **Subnet IDs**
- **Cách tốt nhất** để thực hiện cross-stack collaboration (mỗi team quản lý stack riêng)

### CloudFormation - Outputs Cross-Stack Reference (Trang 411)

- Tạo template thứ hai tận dụng security group từ template đầu
- Sử dụng hàm `Fn::ImportValue`
- **Không thể xóa stack gốc** cho đến khi tất cả references bị xóa

---

## Phần 6: CloudFormation - Conditions (Trang 412-414)

### CloudFormation - Conditions (Trang 412)

- **Conditions** được sử dụng để kiểm soát việc tạo resources hoặc outputs dựa trên một điều kiện
- Conditions có thể là bất cứ điều gì bạn muốn, các loại phổ biến:
  - **Environment** (dev / test / prod)
  - **AWS Region**
  - Bất kỳ parameter value nào
- Mỗi condition có thể reference:
  - Condition khác
  - Parameter value
  - Mapping

### How to define a Condition (Trang 413)

- **Logical ID** do bạn tự chọn - đó là cách bạn đặt tên condition
- **Intrinsic function** (logical) có thể là:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

### How to use a Condition (Trang 414)

- Conditions có thể được áp dụng cho:
  - Resources
  - Outputs
  - Và nhiều thành phần khác

---

## Phần 7: CloudFormation - Intrinsic Functions (Trang 415-421)

### CloudFormation - Intrinsic Functions (Trang 415)

| Function | Mô tả |
|----------|-------------|
| `Ref` | Reference parameters/resources |
| `Fn::Base64` | Convert String sang Base64 |
| `Fn::GetAtt` | Lấy resource attributes |
| `Fn::Cidr` | CIDR operations |
| `Fn::FindInMap` | Lấy giá trị từ mapping |
| `Fn::GetAZs` | Lấy Availability Zones |
| `Fn::ImportValue` | Import từ stacks khác |
| `Fn::Select` | Chọn từ list |
| `Fn::Join` | Nối strings |
| `Fn::Split` | Tách strings |
| `Fn::Sub` | String substitution |
| `Fn::Transform` | Transform template |
| `Fn::ForEach` | Loop over values |
| `Fn::Length` | Độ dài của list/string |
| `Fn::ToJsonString` | Convert sang JSON string |
| **Condition Functions** | `Fn::If`, `Fn::Not`, `Fn::Equals`, etc. |

> *Lưu ý: Các hàm in đậm (Pink) là "must know" - cần phải thuộc*

### Intrinsic Functions - Fn::Ref (Trang 416)

- Hàm `Fn::Ref` được sử dụng để reference:
  - **Parameters** - trả về value của parameter
  - **Resources** - trả về **physical ID** của underlying resource (vd: EC2 ID)
- Shorthand trong YAML: `!Ref`

### Intrinsic Functions - Fn::GetAtt (Trang 417)

- **Attributes** được gắn vào bất kỳ resource nào bạn tạo
- Để biết attributes của resource, xem **documentation**
- Ví dụ: Availability Zone của một EC2 instance

### Intrinsic Functions - Fn::FindInMap (Trang 418)

- Sử dụng `Fn::FindInMap` để lấy named value từ một key cụ thể
- Format: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

### Intrinsic Functions - Fn::ImportValue (Trang 419)

- Import values đã được **exported** ở stacks khác
- Sử dụng hàm `Fn::ImportValue`

### Intrinsic Functions - Fn::Base64 (Trang 420)

- Convert String sang **Base64 representation**
- Ví dụ: pass encoded data đến EC2 Instance's **UserData property**

### Intrinsic Functions - Condition Functions (Trang 421)

- **Logical ID** do bạn tự chọn
- Intrinsic function (logical) có thể là:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

---

## Phần 8: CloudFormation - Rollbacks (Trang 422)

### CloudFormation - Rollbacks (Trang 422)

**Stack Creation Fails - Tạo stack thất bại:**
- **Default**: tất cả roll back (bị xóa) → xem logs
- Option để **disable rollback** và troubleshoot vấn đề

**Stack Update Fails - Update stack thất bại:**
- Stack tự động **rollback về previous known working state**
- Có thể xem logs và error messages

**Rollback Failure - Rollback thất bại:**
- Fix resources thủ công
- Sau đó issue `ContinueUpdateRollback` API từ Console
- Hoặc từ CLI sử dụng `continue-update-rollback` API call

---

## Phần 9: CloudFormation - Service Role (Trang 423)

### CloudFormation - Service Role (Trang 423)

- **IAM role** cho phép CloudFormation create/update/delete stack resources thay mặt bạn
- Cho phép users create/update/delete stack resources ngay cả khi họ **không có permissions** trực tiếp với resources trong stack

**Use cases:**
- Muốn áp dụng **least privilege principle**
- Nhưng không muốn cấp cho user tất cả permissions cần thiết để tạo stack resources

**Yêu cầu:**
- User phải có **`iam:PassRole`** permissions

---

## Phần 10: CloudFormation - DeletionPolicy (Trang 424-426)

### DeletionPolicy: Delete (Trang 424)

- **DeletionPolicy**: Kiểm soát điều gì xảy ra khi CloudFormation template bị xóa hoặc khi resource bị xóa khỏi template
- Biện pháp bảo vệ bổ sung để preserve và backup resources
- **Default DeletionPolicy = Delete**
- `Delete` sẽ **không hoạt động** trên S3 bucket nếu bucket không rỗng

### DeletionPolicy: Retain (Trang 425)

- **`DeletionPolicy = Retain`**:
- Chỉ định resources để preserve trong trường hợp CloudFormation deletes
- Hoạt động với **bất kỳ resources** nào

### DeletionPolicy: Snapshot (Trang 426)

- **`DeletionPolicy = Snapshot`**:
- Tạo một **final snapshot** trước khi xóa resource
- Resources hỗ trợ:
  - **EBS Volume**
  - **ElastiCache Cluster**, **ElastiCache ReplicationGroup**
  - **RDS DBInstance**, **RDS DBCluster**
  - **Redshift Cluster**
  - **Neptune DBCluster**
  - **DocumentDB DBCluster**

---

## Phần 11: CloudFormation - Stack Policies (Trang 427)

### CloudFormation - Stack Policies (Trang 427)

- Trong CloudFormation Stack update, tất cả update actions được phép trên tất cả resources (default)
- **Stack Policy** là một **JSON document** định nghĩa update actions được phép trên specific resources trong Stack updates
- **Bảo vệ resources** khỏi các updates không cố ý
- Khi set Stack Policy, **tất cả resources** trong Stack được bảo vệ theo mặc định
- Chỉ định explicit **ALLOW** cho resources bạn muốn cho phép update

---

## Phần 12: CloudFormation - Termination Protection (Trang 428)

### CloudFormation - Termination Protection (Trang 428)

- Để ngăn chặn **accidental deletes** của CloudFormation Stacks, sử dụng **TerminationProtection**

---

## Phần 13: CloudFormation - Custom Resources (Trang 429-431)

### CloudFormation - Custom Resources (Trang 429)

**Custom Resources được sử dụng để:**
- Định nghĩa resources **chưa được hỗ trợ** bởi CloudFormation
- Định nghĩa **custom provisioning logic** cho resources bên ngoài CloudFormation (on-premises resources, 3rd party resources)
- Chạy **custom scripts** trong quá trình create / update / delete thông qua Lambda functions (ví dụ: chạy Lambda function để empty S3 bucket trước khi bucket bị xóa)

**Cách định nghĩa:**
- Sử dụng `AWS::CloudFormation::CustomResource` hoặc `Custom::MyCustomResourceTypeName` (khuyến nghị)
- Backed bởi **Lambda function** (phổ biến nhất) hoặc **SNS topic**

### How to define a Custom Resource? (Trang 430)

- **ServiceToken** chỉ định nơi CloudFormation gửi requests đến, ví dụ: Lambda ARN hoặc SNS ARN (**bắt buộc & phải cùng region**)
- Input data parameters (tùy chọn)

### Use Case - Xóa nội dung S3 bucket (Trang 431)

- Không thể xóa một **non-empty S3 bucket**
- Để xóa non-empty S3 bucket, phải xóa hết objects bên trong trước
- Sử dụng **custom resource** để empty S3 bucket trước khi nó bị xóa bởi CloudFormation

---

## Phần 14: CloudFormation - StackSets (Trang 432)

### CloudFormation - StackSets (Trang 432)

- Tạo, update, hoặc delete stacks trên **nhiều accounts và regions** với một operation/template
- Target accounts để tạo, update, delete stack instances từ StackSets
- Khi update một stack set, tất cả associated stack instances được update trên tất cả accounts và regions
- Có thể apply cho **tất cả accounts** của một **AWS Organization**
- Chỉ **Administrator account** (hoặc **Delegated Administrator**) mới có thể tạo StackSets

---

## Bài tập (Trang 433)

- Tạo CloudFormation Stack với VPC, Subnet, ALB, EC2, etc.
