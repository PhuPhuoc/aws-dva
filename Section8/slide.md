# Section 8 - AWS CloudFormation (Trang 385-432)

---

## Phần 1: CloudFormation - Tổng quan (Trang 385-391)

### AWS CloudFormation - Giới thiệu (Trang 387)

**CloudFormation là gì?**
- Là **cách tiếp cận khai báo (declarative)** để định nghĩa hạ tầng AWS
- Hỗ trợ hầu hết các dịch vụ (resources) của AWS
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
- **Chiến lược tiết kiệm (Savings strategy)**: Tự động xóa templates lúc 5 PM và tạo lại lúc 8 AM (môi trường Dev)

### Lợi ích của AWS CloudFormation (2/2) (Trang 390)

**Năng suất (Productivity):**
- Khả năng phá hủy và tạo lại hạ tầng trên cloud một cách nhanh chóng
- **Tự động sinh sơ đồ (diagram)** cho templates
- **Lập trình declarative** - không cần lo về thứ tự (ordering) và điều phối (orchestration)

**Phân tách trách nhiệm (Separation of Concern):**
- Tạo nhiều stacks cho nhiều apps, nhiều layers
- Ví dụ: VPC stacks, Network stacks, App stacks

**Không cần phát minh lại bánh xe (don't re-invent the wheel):**
- Tận dụng các templates có sẵn trên web
- Tận dụng tài liệu (documentation) của AWS

### CloudFormation hoạt động như thế nào (Trang 391)

- Template phải được **upload lên S3** rồi mới được tham chiếu (reference) trong CloudFormation
- Để cập nhật (update) template, **không thể chỉnh sửa (edit) trực tiếp** - phải upload lại (re-upload) phiên bản (version) mới
- Các Stack được định danh bằng **tên (name)**
- **Xóa một stack** → xóa toàn bộ artifacts đã được tạo bởi CloudFormation

---

## Phần 2: CloudFormation Building Blocks (Trang 392-399)

### CloudFormation - Building Blocks (Trang 393)

**Các thành phần chính của Template (Template's Components):**

| Component | Mô tả |
|-----------|-------------|
| `AWSTemplateFormatVersion` | Xác định phiên bản/khả năng của template (vd: "2010-09-09") |
| `Description` | Ghi chú (comments) về template |
| `Resources` (**MANDATORY**) | AWS resources được khai báo trong template |
| `Parameters` | Các giá trị đầu vào động (dynamic inputs) cho template |
| `Mappings` | Static variables cho template |
| `Outputs` | References đến những gì đã được tạo |
| `Conditionals` | Danh sách điều kiện (list of conditions) để thực hiện việc tạo resource |

**Trợ giúp của Template (Template's Helpers):**
- **References**
- **Functions**

### CloudFormation - Resources (Trang 396)

- **Resources** là core của CloudFormation template (**BẮT BUỘC**)
- Đại diện cho các AWS Components sẽ được tạo và cấu hình
- Resources được khai báo và có thể reference lẫn nhau
- AWS tự xử lý việc tạo (creation), cập nhật (updates) và xóa (deletes)
- **Hơn 700 loại (types)** resources
- Định danh (identifier) resource type có format: `service-provider::service-name::data-type-name`
  - Ví dụ: `AWS::EC2::Instance`

### Tìm tài liệu về Resources (Trang 397)

- Không thể học hết hơn 700 resources, nhưng có thể học cách sử dụng chúng
- Tất cả resources nằm tại: AWS CloudFormation User Guide
- Đọc tài liệu (documentation) của từng resource
- Ví dụ: tài liệu của AWS::EC2::Instance

### Hỏi thường gặp về Resources của CloudFormation (Trang 399)

**Có thể tạo số lượng resources động (dynamic) không?**
- Có, sử dụng **CloudFormation Macros** và **Transform**
- Ngoài phạm vi của khóa học

**Mọi dịch vụ (service) của AWS đều được hỗ trợ không?**
- Gần như tất cả
- Một số dịch vụ ngách (niche services) chưa có → giải pháp thay thế (workaround) bằng **CloudFormation Custom Resources**

---

## Phần 3: CloudFormation - Parameters (Trang 400-405)

### CloudFormation - Parameters (Trang 400)

**Parameters là gì?**
- Cung cấp các giá trị đầu vào (inputs) cho CloudFormation template
- Quan trọng khi nào:
  - Muốn **tái sử dụng (reuse) templates** trên toàn công ty
  - Một số giá trị đầu vào (inputs) không thể xác định trước
- Parameters rất mạnh mẽ, được kiểm soát và ngăn ngừa lỗi (errors) nhờ **types**

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
- `AWS-Specific Parameter` (giúp phát hiện các giá trị không hợp lệ (invalid values) - phải khớp (match) với các giá trị tồn tại sẵn (existing values) trong AWS account)
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

### Cách tham chiếu (reference) một Parameter? (Trang 404)

- Hàm `Fn::Ref` được dùng để tham chiếu (reference) các parameters
- Parameters có thể được sử dụng ở bất kỳ đâu trong template
- Cú pháp rút gọn (shorthand) trong YAML: `!Ref`
- Hàm này cũng có thể tham chiếu (reference) các thành phần (elements) khác trong template

### CloudFormation - Pseudo Parameters (Trang 405)

- AWS cung cấp sẵn **Pseudo Parameters** trong mọi CloudFormation template
- Có thể sử dụng bất kỳ lúc nào (bật sẵn, enabled by default)
- Pseudo parameters quan trọng:
  - `AWS::Region`
  - `AWS::StackName`
  - `AWS::StackId`
  - `AWS::AccountId`
  - `AWS::NoValue`

---

## Phần 4: CloudFormation - Mappings (Trang 406-408)

### CloudFormation - Mappings (Trang 406)

- Mappings là các **biến cố định (fixed variables)** trong CloudFormation template
- Rất hữu ích để phân biệt giữa:
  - Các environments (dev vs prod)
  - Regions (AWS regions)
  - AMI types
- Tất cả các giá trị (values) đều được **hardcode sẵn** trong template

### Accessing Mapping Values - Fn::FindInMap (Trang 407)

- Sử dụng `Fn::FindInMap` để lấy named value từ một key cụ thể
- Format: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`
- Mappings rất phù hợp với AMIs vì AMIs là region-specific

### Khi nào dùng Mappings vs. Parameters? (Trang 408)

**Mappings phù hợp khi:**
- Biết trước tất cả các giá trị (values) có thể có
- Có thể suy ra từ: Region, Availability Zone, AWS Account, Environment (dev vs prod)
- Cho phép kiểm soát template an toàn hơn

**Sử dụng Parameters khi:**
- Values thực sự user-specific

---

## Phần 5: CloudFormation - Outputs (Trang 409-411)

### CloudFormation - Outputs (Trang 409-410)

- **Phần Outputs (Outputs section)** khai báo các giá trị đầu ra (output values) tùy chọn
- Có thể **đưa vào (import) các stacks khác** (nếu đã export trước)
- Có thể xem các output qua AWS Console hoặc AWS CLI
- Rất hữu ích khi:
  - Định nghĩa mạng (network) bằng CloudFormation
  - Xuất (output) các biến (variables) như **VPC ID**, **Subnet IDs**
- **Cách tốt nhất** để thực hiện cross-stack collaboration (mỗi team quản lý stack riêng)

### CloudFormation - Tham chiếu Cross-Stack cho Outputs (Trang 411)

- Tạo template thứ hai tận dụng security group từ template đầu tiên
- Sử dụng hàm `Fn::ImportValue`
- **Không thể xóa stack gốc** cho đến khi mọi tham chiếu (references) bị xóa

---

## Phần 6: CloudFormation - Conditions (Trang 412-414)

### CloudFormation - Điều kiện (Conditions) (Trang 412)

- **Điều kiện (Conditions)** được sử dụng để kiểm soát việc tạo resources hoặc các output dựa trên một điều kiện
- Các điều kiện (Conditions) có thể là bất kỳ điều gì bạn muốn, các loại phổ biến:
  - **Environment** (dev / test / prod)
  - **AWS Region**
  - Bất kỳ parameter value nào
- Mỗi condition có thể tham chiếu (reference):
  - Một condition khác
  - Giá trị parameter
  - Mapping

### Cách định nghĩa một Condition (Trang 413)

- **Logical ID** do bạn tự chọn - đó là cách bạn đặt tên cho condition
- **Hàm nội tại (intrinsic function)** (logical) có thể là:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

### Cách sử dụng một Condition (Trang 414)

- Các Condition có thể được áp dụng cho:
  - Resources
  - Outputs
  - Và nhiều thành phần khác

---

## Phần 7: CloudFormation - Intrinsic Functions (Trang 415-421)

### CloudFormation - Intrinsic Functions (Trang 415)

| Function | Mô tả |
|----------|-------------|
| `Ref` | Tham chiếu (reference) parameters/resources |
| `Fn::Base64` | Chuyển String sang Base64 |
| `Fn::GetAtt` | Lấy attributes của resource |
| `Fn::Cidr` | Các phép toán CIDR |
| `Fn::FindInMap` | Lấy giá trị từ Mappings |
| `Fn::GetAZs` | Lấy các Availability Zone |
| `Fn::ImportValue` | Import từ các stacks khác |
| `Fn::Select` | Chọn giá trị từ list |
| `Fn::Join` | Nối các chuỗi (strings) |
| `Fn::Split` | Tách chuỗi (string) |
| `Fn::Sub` | Thay chuỗi (string substitution) |
| `Fn::Transform` | Biến đổi (transform) template |
| `Fn::ForEach` | Lặp (loop) qua các giá trị |
| `Fn::Length` | Độ dài của list/string |
| `Fn::ToJsonString` | Chuyển sang JSON string |
| **Condition Functions** | `Fn::If`, `Fn::Not`, `Fn::Equals`, etc. |

> *Lưu ý: Các hàm in đậm (Pink) là "must know" - bắt buộc phải thuộc*

### Intrinsic Functions - Fn::Ref (Trang 416)

- Hàm `Fn::Ref` được sử dụng để tham chiếu (reference):
  - **Parameters** - trả về value của parameter
  - **Resources** - trả về **physical ID** của underlying resource (vd: EC2 ID)
- Cú pháp rút gọn (shorthand) trong YAML: `!Ref`

### Intrinsic Functions - Fn::GetAtt (Trang 417)

- **Attributes** được gắn cho bất kỳ resource nào bạn tạo
- Để biết attributes của resource, xem **tài liệu (documentation)**
- Ví dụ: Availability Zone của một EC2 instance

### Intrinsic Functions - Fn::FindInMap (Trang 418)

- Sử dụng `Fn::FindInMap` để lấy named value từ một key cụ thể
- Format: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

### Intrinsic Functions - Fn::ImportValue (Trang 419)

- Import các giá trị (values) đã được **export** ở các stacks khác
- Sử dụng hàm `Fn::ImportValue`

### Intrinsic Functions - Fn::Base64 (Trang 420)

- Chuyển String sang **dạng Base64 (Base64 representation)**
- Ví dụ: truyền (pass) dữ liệu đã mã hóa đến **tính chất UserData (UserData property)** của EC2 Instance

### Intrinsic Functions - Condition Functions (Trang 421)

- **Logical ID** do bạn tự chọn
- Hàm nội tại (intrinsic function, logical) có thể là:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

---

## Phần 8: CloudFormation - Rollbacks (Trang 422)

### CloudFormation - Rollbacks (Trang 422)

**Stack Creation Fails - Tạo stack thất bại:**
- **Mặc định (default)**: mọi thứ bị roll back (bị xóa) → xem logs
- Có tùy chọn **vô hiệu hóa rollback (disable rollback)** để khảo sát (troubleshoot) vấn đề

**Stack Update Fails - Update stack thất bại:**
- Stack tự động **rollback về previous known working state**
- Có thể xem logs và thông báo lỗi (error messages)

**Rollback Failure - Rollback thất bại:**
- Sửa (fix) các resources thủ công
- Sau đó thực hiện (issue) API `ContinueUpdateRollback` từ Console
- Hoặc từ CLI sử dụng lệnh gọi API (API call) `continue-update-rollback`

---

## Phần 9: CloudFormation - Service Role (Trang 423)

### CloudFormation - Service Role (Trang 423)

- **IAM Role** cho phép CloudFormation tạo/cập nhật/xóa (create/update/delete) các resources của stack thay mặt bạn
- Cho phép người dùng (users) tạo/cập nhật/xóa (create/update/delete) các resources của stack ngay cả khi họ **không có permissions** trực tiếp với các resources trong stack

**Các trường hợp sử dụng (Use cases):**
- Muốn áp dụng **nguyên tắc đặc quyền tối thiểu (least privilege principle)**
- Nhưng không muốn cấp cho user tất cả permissions cần thiết để tạo các resources của stack

**Yêu cầu:**
- User phải có **phân quyền `iam:PassRole`**

---

## Phần 10: CloudFormation - DeletionPolicy (Trang 424-426)

### DeletionPolicy: Delete (Trang 424)

- **DeletionPolicy**: Kiểm soát điều gì xảy ra khi CloudFormation template bị xóa hoặc khi resource bị xóa khỏi template
- Là biện pháp bảo vệ bổ sung để giữ lại (preserve) và sao lưu (backup) resources
- **DeletionPolicy mặc định (Default DeletionPolicy) = Delete**
- `Delete` sẽ **không hoạt động** trên S3 bucket nếu bucket không rỗng

### DeletionPolicy: Retain (Trang 425)

- **`DeletionPolicy = Retain`**:
- Chỉ định các resources để giữ lại (preserve) trong trường hợp CloudFormation xóa (delete)
- Hoạt động với **bất kỳ resource nào**

### DeletionPolicy: Snapshot (Trang 426)

- **`DeletionPolicy = Snapshot`**:
- Tạo một **snapshot cuối cùng (final snapshot)** trước khi xóa resource
- Các resources được hỗ trợ:
  - **EBS Volume**
  - **ElastiCache Cluster**, **ElastiCache ReplicationGroup**
  - **RDS DBInstance**, **RDS DBCluster**
  - **Redshift Cluster**
  - **Neptune DBCluster**
  - **DocumentDB DBCluster**

---

## Phần 11: CloudFormation - Stack Policies (Trang 427)

### CloudFormation - Stack Policies (Trang 427)

- Trong CloudFormation Stack update, tất cả các hành động cập nhật (update actions) được phép trên mọi resources (mặc định)
- **Stack Policy** là một **tài liệu JSON (JSON document)** định nghĩa các hành động cập nhật (update actions) được phép trên những resources cụ thể (specific) trong Stack updates
- **Bảo vệ resources** khỏi các bản cập nhật không cố ý
- Khi thiết lập (set) Stack Policy, **tất cả resources** trong Stack được bảo vệ theo mặc định
- Chỉ định **ALLOW** một cách tường minh (explicit) cho những resources bạn muốn cho phép cập nhật

---

## Phần 12: CloudFormation - Termination Protection (Trang 428)

### CloudFormation - Termination Protection (Trang 428)

- Để ngăn chặn **việc xóa nhầm (accidental deletes)** các CloudFormation Stacks, sử dụng **TerminationProtection**

---

## Phần 13: CloudFormation - Custom Resources (Trang 429-431)

### CloudFormation - Custom Resources (Trang 429)

**Custom Resources được sử dụng để:**
- Định nghĩa resources **chưa được hỗ trợ** bởi CloudFormation
- Định nghĩa **custom provisioning logic** cho resources bên ngoài CloudFormation (on-premises resources, 3rd party resources)
- Chạy **các scripts tùy chỉnh (custom scripts)** trong quá trình tạo / cập nhật / xóa (create / update / delete) thông qua Lambda functions (ví dụ: chạy Lambda function để làm trống (empty) S3 bucket trước khi bucket bị xóa)

**Cách định nghĩa:**
- Sử dụng `AWS::CloudFormation::CustomResource` hoặc `Custom::MyCustomResourceTypeName` (khuyến nghị)
- Backed bởi **Lambda function** (phổ biến nhất) hoặc **SNS topic**

### Cách định nghĩa một Custom Resource? (Trang 430)

- **ServiceToken** chỉ định nơi CloudFormation gửi các requests đến, ví dụ: Lambda ARN hoặc SNS ARN (**bắt buộc & phải cùng region**)
- Các parameters dữ liệu đầu vào (input data parameters, tùy chọn)

### Use Case - Xóa nội dung S3 bucket (Trang 431)

- Không thể xóa một **non-empty S3 bucket**
- Để xóa S3 bucket không trống (non-empty S3 bucket), phải xóa hết objects bên trong trước
- Sử dụng **custom resource** để làm trống (empty) S3 bucket trước khi nó bị xóa bởi CloudFormation

---

## Phần 14: CloudFormation - StackSets (Trang 432)

### CloudFormation - StackSets (Trang 432)

- Tạo, cập nhật (update), hoặc xóa (delete) stacks trên **nhiều accounts và regions** với một thao tác duy nhất (operation)/template
- Chọn (target) các account để tạo, cập nhật (update), xóa (delete) stack instances từ StackSets
- Khi cập nhật (update) một stack set, toàn bộ các stack instances liên quan (associated) được cập nhật (update) trên tất cả các account và region
- Có thể áp dụng (apply) cho **tất cả các account** của một **AWS Organization**
- Chỉ **Administrator account** (hoặc **Delegated Administrator**) mới có thể tạo StackSets

---

## Bài tập (Trang 433)

- Tạo CloudFormation Stack với VPC, Subnet, ALB, EC2, etc.
