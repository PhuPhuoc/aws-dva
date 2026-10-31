# Section 8 - Kiến thức mở rộng: AWS CloudFormation

---

## 1. CloudFormation - Cơ chế hoạt động chi tiết

### Vòng đời (Lifecycle) của CloudFormation Template

#### Giải thích cốt lõi
CloudFormation hoạt động theo cơ chế **declarative** (khai báo) chứ không phải **imperative** (mệnh lệnh). Khi bạn tải lên (upload) template, CloudFormation:

1. **Giai đoạn phân tích (parse) template** → So sánh trạng thái mong muốn (desired state) trong template với trạng thái hiện tại (current state) trong AWS
2. **Tính toán các thay đổi (changes) cần thiết** → Ví dụ: tạo mới EC2, cập nhật (update) rule của security group, không thay đổi RDS
3. **Thực thi các thay đổi theo đúng thứ tự**:
   - Tạo VPC → Subnets → Security Groups → EC2 instances → Load Balancer
   - Phụ thuộc được tự động phát hiện (qua `!Ref` và `!GetAtt`)
4. **Trạng thái Stack (Stack states):**
   - `CREATE_IN_PROGRESS` → `CREATE_COMPLETE` (hoặc `CREATE_FAILED` → `ROLLBACK_COMPLETE`)
   - `UPDATE_IN_PROGRESS` → `UPDATE_COMPLETE` (hoặc `UPDATE_FAILED` → `UPDATE_ROLLBACK_COMPLETE`)
   - `DELETE_IN_PROGRESS` → `DELETE_COMPLETE` (hoặc `DELETE_FAILED`)

**So sánh với các phương án khác (alternatives):**

| Công cụ | Cách tiếp cận (Approach) | Quản lý State (State Management) | Phù hợp nhất (Best For) |
|---------|----------|------------------|----------|
| **CloudFormation** | Declarative (khai báo) | AWS tự quản lý state | Chỉ dùng AWS, hệ thống ổn định (mature) |
| **Terraform** | Declarative (khai báo) | File state riêng | Multi-cloud (nhiều nhà cung cấp) |
| **CDK** | Imperative (mệnh lệnh, biên dịch thành CF) | AWS tự quản lý state | Đội ngũ kỹ sư (developers) thích viết code |
| **Pulumi** | Imperative (dùng các ngôn ngữ thông dụng) | File state riêng | Multi-cloud + viết code |
| **Ansible** | Imperative (mệnh lệnh) | Không theo dõi state | Quản lý cấu hình (config management) |

#### NÊN làm
- **Bắt đầu với CloudFormation khi infrastructure là AWS-only (chỉ AWS)** → Vì native (gốc), miễn phí (chỉ trả tiền cho resources), tích hợp (integrate) tốt với AWS services, có **drift detection**. So với Terraform: cần quản lý state file riêng, tốn thêm learning curve (chậm quen công cụ).
- **Dùng YAML thay vì JSON** → Vì YAML hỗ trợ ghi chú (comments), chuỗi nhiều dòng (multi-line strings), ít dấu ngoặc. JSON "thật khó dùng với CloudFormation (CF)" vì dài dòng (verbose), không có comments, dễ sai cú pháp (syntax).
- **Gán nhãn (Tag) mọi resource bằng tên stack (stack name)** → Vì AWS tự động gán nhãn (tag) resources với `aws:cloudformation:stack-name`, giúp theo dõi chi phí (tracking cost), dọn dẹp (cleanup), gỡ lỗi (debugging) dễ dàng.

#### KHÔNG NÊN làm
- **Viết 1 template khổng lồ chứa toàn bộ infrastructure** → Vì khi 1 resource thất bại (fail), toàn bộ stack bị hủy (rollback) hết. NÊN: chia thành **nested stacks** (VPC stack, Network stack, App stack) → dễ quản lý, dễ tái sử dụng (reuse), cách ly lỗi (fail isolation).
- **Mã hóa sẵn (Hardcode) giá trị trong Resources** → Vì mỗi lần thay đổi phải tải lên lại (re-upload) template. NÊN: dùng **Parameters** cho giá trị có thể thay đổi (InstanceType, CIDR), **Mappings** cho giá trị theo Region/Environment.

---

## 2. CloudFormation Parameters

### Thực hành tốt (Best Practices) với Parameters

#### Giải thích cốt lõi
Parameters cho phép template có thể **tái sử dụng (reusable)** cho nhiều môi trường (environment)/đội nhóm (team). Có 2 loại Parameters chính:

**AWS-Specific Parameters** - AWS xác thực (validate) giá trị với các giá trị tồn tại sẵn (existing values):
- `AWS::EC2::VPC::Id` - chỉ chấp nhận VPC ID tồn tại trong account
- `AWS::EC2::Subnet::Id` - chỉ chấp nhận Subnet ID
- `AWS::EC2::SecurityGroup::Id`
- `List<AWS::EC2::VPC::Id>` - danh sách VPC IDs

**SSM Parameter Types** - Lấy giá trị (value) từ SSM Parameter Store:
- `AWS::SSM::Parameter::Value<String>`
- `AWS::SSM::Parameter::Value<AWS::EC2::VPC::Id>`
- Cho phép **quản lý cấu hình tập trung (centralized configuration management)**

**So sánh các approaches:**

| Cách làm (Approach) | Trường hợp dùng (Use Case) | Bảo mật (Security) |
|----------|----------|----------|
| Giá trị hardcode (Hardcoded values) | Môi trường (env) đơn lẻ, không cần thay đổi | ✅ N/A |
| Parameters | Nhiều env, cấu hình lúc runtime | ⚠️ Cần thận trọng với NoEcho |
| Mappings | Giá trị theo Region/AMI (region/AMI-specific values) | ✅ Hardcode, an toàn |
| SSM Parameter Store | Cấu hình dùng chung (shared config) giữa các stack | ✅ Được mã hóa, có phiên bản (versioned) |
| Secrets Manager | Dữ liệu nhạy cảm (password DB) | ✅ Tự động xoay (auto-rotation), có audit |

#### NÊN làm
- **Dùng AWS-Specific Parameters thay vì String cho IDs** → Vì AWS sẽ **xác thực (validate)** giá trị có tồn tại. Nếu user nhập sai VPC ID, stack thất bại (fail) ngay lúc xác thực (validate) chứ không phải lúc chạy (runtime).
- **Dùng `AllowedValues` để giới hạn các tùy chọn (options)** → Vì nó chống sai chính tả (typo) và tài liệu hóa (document) rõ ràng các tùy chọn (options). Ví dụ: `AllowedValues: [t3.micro, t3.small, t3.medium]`.
- **Dùng `NoEcho: true` cho các giá trị nhạy cảm (sensitive values)** → Vì mặc định các giá trị parameter (parameter values) hiển thị trong CloudFormation console/events. Với secrets, NÊN dùng **Secrets Manager** hoặc **SSM SecureString** thay vì parameter.

#### KHÔNG NÊN làm
- **Đặt giá trị mặc định (Default value) cho mọi parameter** → Vì user có thể bỏ qua (skip) bước nhập và dùng giá trị mặc định (default) không phù hợp. NÊN: chỉ đặt default cho các parameter tùy chọn (optional parameters), **không đặt default** cho các giá trị quan trọng (critical values: production instance type, CIDR).
- **Truyền database password qua Parameters** → Vì password hiển thị trong CloudFormation logs/events. NÊN: dùng `Secrets Manager` với dynamic reference: `{{resolve:secretsmanager:MySecret:SecretString:password}}`.

---

## 3. CloudFormation Mappings

### Mappings chi tiết

#### Giải thích cốt lõi
Mappings là **các bảng tra mã hóa sẵn (hardcoded lookup tables)** trong template. Khác với Parameters (do user nhập), Mappings là các giá trị **do developer định nghĩa (developer-defined)**, thường dùng cho:

1. **Giá trị theo region (region-specific values)** (AMI IDs theo từng region)
2. **Cấu hình theo môi trường (environment-specific configs)** (dev/staging/prod sizes)
3. **Giá trị theo account (account-specific values)** (ARNs, account IDs)
4. **Cấu hình có điều kiện (conditional configurations)** theo một biến (variable) có thể xác định (determine) lúc triển khai (deploy)

**Ví dụ thực tế - AMI mapping:**

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0ff8a91504f7b2c5c
      AZ1: us-east-1a
      AZ2: us-east-1b
    us-west-2:
      AMI: ami-0bdb828fd58c52435
      AZ1: us-west-2a
      AZ2: us-west-2b

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, AMI]
      AvailabilityZone: !FindInMap [RegionMap, !Ref AWS::Region, AZ1]
```

#### NÊN làm
- **Dùng Mappings cho AMI IDs theo từng region** → Vì AMI là **đặc thù theo region (region-specific)** (không dùng được AMI us-east-1 ở eu-west-1). Hardcode mapping → không cần cập nhật (update) mỗi khi AWS ra AMI mới (trừ khi dùng SSM Parameter).
- **Dùng Mappings kết hợp với `!Ref AWS::Region`** → Vì tự động chọn đúng cấu hình (config) cho region đang triển khai (deploy), không cần user nhập.

#### KHÔNG NÊN làm
- **Dùng Mappings cho các giá trị mà user cần thay đổi thường xuyên** → Vì mỗi lần thay đổi phải chỉnh sửa (edit) template và upload lại (re-upload). NÊN: dùng **Parameters** cho giá trị riêng theo từng user, dùng **SSM Parameter Store** cho cấu hình dùng chung (shared config) giữa các stack.
- **Dùng Mappings cho dữ liệu (data) quá lớn (>100 entries)** → Vì template sẽ khó đọc. NÊN: chuyển dữ liệu (data) lên S3 hoặc DynamoDB, dùng Custom Resource để đọc (fetch).

---

## 4. CloudFormation Outputs & Cross-Stack References

### Mẫu tham chiếu cross-stack (Cross-Stack Reference Pattern)

#### Giải thích cốt lõi
**Outputs** cho phép một stack công khai (expose) các giá trị để các stack khác sử dụng (consume). Có 2 cách:

1. **Outputs không có Export** - chỉ để xem (qua CLI/Console)
2. **Outputs có Export** - dùng để import giữa các stack (cross-stack, qua `Fn::ImportValue`)

**Ví dụ:**

```yaml
# Network Stack
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${AWS::StackName}-VPC-ID"
  PublicSubnet1:
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub "${AWS::StackName}-PublicSubnet1"

# App Stack
Resources:
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue NetworkStack-VPC-ID
      GroupDescription: "App SG"
```

**Quy tắc quan trọng:**
- **Tên Export (Export names) phải DU NHẤT (UNIQUE) trong account/region**
- **Không thể xóa stack gốc** nếu vẫn còn stack khác đang import từ nó
- Nếu muốn xóa stack có exports → phải xóa tất cả các stack phụ thuộc (dependent stacks) trước

**So sánh với các phương án khác (alternatives):**

| Mẫu (Pattern) | Ưu điểm (Pros) | Nhược điểm (Cons) |
|---------|------|------|
| **Outputs + ImportValue** | Native trong CF, không phát sinh chi phí | Liên kết chặt với việc xóa stack (deletion coupling) |
| **SSM Parameter Store** | Liên kết lỏng (loose coupling), dễ xóa hơn | Có chi phí, chỉ đồng nhất cuối cùng (eventual consistency) |
| **DynamoDB lookup** | Rất linh hoạt | Cần custom resource, phức tạp hơn |
| **Nested Stacks** | Liên kết chặt (strong coupling), vòng đời dễ quản lý | Gộp tất cả vào một (all-in-one) template |

#### NÊN làm
- **Dùng `!Sub "${AWS::StackName}-VPC-ID"` làm tên Export** → Vì stack name là duy nhất (unique) → export name cũng duy nhất. Tránh hardcode tên export.
- **Dùng Outputs cho hạ tầng dùng chung (shared infrastructure, VPC, Subnet, SG IDs)** → Vì nhiều stacks cần dùng chung, **nguồn dữ liệu duy nhất (single source of truth)**.

#### KHÔNG NÊN làm
- **Export quá nhiều giá trị (values) không cần thiết** → Vì mỗi export tạo "kẻ khóa (lock)" - khó xóa stack. NÊN: chỉ export những gì thực sự cần dùng chung giữa các stack (cross-stack).
- **Dùng cross-stack references cho các bản cập nhật tần suất cao (high-frequency updates)** → Vì khi cập nhật (update) giá trị ở stack gốc, tất cả các stack phụ thuộc (dependent stacks) bị ảnh hưởng (affected). NÊN: dùng **SSM Parameter Store** cho các giá trị cần thay đổi lúc runtime.

---

## 5. CloudFormation Conditions

### Conditions chi tiết

#### Giải thích cốt lõi
Điều kiện (Conditions) cho phép **tạo resources có điều kiện (conditionally create resources)** dựa trên:
- Giá trị parameter (Parameter values: Environment, Region)
- Pseudo parameters
- Các điều kiện khác (Other conditions)

**Các intrinsic functions cho Conditions:**

| Function | Logic | Example |
|----------|-------|---------|
| `Fn::Equals` | A == B | `!Equals [!Ref Env, prod]` |
| `Fn::And` | Tất cả điều kiện đều true | `!And [!Equals, !Equals]` |
| `Fn::Or` | Ít nhất một điều kiện true | `!Or [IsProd, IsStaging]` |
| `Fn::Not` | Phủ định (negate) | `!Not [!Equals [!Ref Env, dev]]` |
| `Fn::If` | If/else | `!If [IsProd, prod-value, dev-value]` |

**Ví dụ thực tế:**

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  CreateBackup: !Or [!Equals [!Ref Environment, prod], !Equals [!Ref Environment, staging]]

Resources:
  # Tạo EIP chỉ cho production
  ProdEIP:
    Type: AWS::EC2::EIP
    Condition: IsProduction
    Properties:
      Domain: vpc
  
  # Tạo backup vault cho prod/staging
  BackupVault:
    Type: AWS::Backup::BackupVault
    Condition: CreateBackup
    Properties:
      BackupVaultName: !Ref Environment
```

#### NÊN làm
- **Dùng Conditions để tạo resources theo môi trường (environment-specific)** → Vì 1 template cho nhiều môi trường (dev, staging, prod) nhưng vẫn tùy chỉnh (customize) resources theo từng env. So với việc có 3 templates riêng: dễ bị lệch (drift), khó bảo trì (maintain).
- **Dùng `!If` để chuyển đổi (switch) giữa các giá trị** → Vì tránh trùng lặp (duplicate) resources. Ví dụ: `InstanceType: !If [IsProd, m5.large, t3.micro]`.

#### KHÔNG NÊN làm
- **Dùng Conditions thay vì Nested Stacks cho logic quá phức tạp** → Vì Conditions chỉ true/false, không thể diễn đạt được logic "với mỗi X thì tạo Y" (for each X, create Y). NÊN: dùng **Nested Stacks** cho multi-environment, dùng **Modules** (CloudFormation Modules) cho các component có thể tái sử dụng (reusable components).
- **Dùng Conditions với tham chiếu vòng (circular references)** → Vì CloudFormation không phát hiện (detect) được → triển khai (deploy) thất bại (fail) với lỗi (error) khó hiểu. NÊN: kiểm thử (test) kỹ với mọi tổ hợp (combination) giá trị.

---

## 6. CloudFormation Intrinsic Functions

### Functions chi tiết

#### Giải thích cốt lõi
**BẮT BUỘC PHẢI THUỘC (MUST KNOW, in đậm trong slide - "Pink"):**
- `Ref`
- `Fn::Base64`
- `Fn::GetAtt`
- `Fn::Cidr`
- `Fn::FindInMap`
- `Fn::GetAZs`
- `Fn::ImportValue`
- `Fn::Select`
- `Fn::Join`
- `Fn::Split`
- `Fn::Sub`
- `Fn::Transform`
- `Fn::ForEach`
- `Fn::Length`
- `Fn::ToJsonString`
- Các hàm điều kiện (Condition Functions: Fn::If, Fn::Not, Fn::Equals, v.v.)

**`!Ref` vs `!GetAtt` - Sự khác biệt quan trọng:**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro

# !Ref - returns primary identifier (physical ID)
!Ref MyInstance  # → "i-0123456789abcdef0"

# !GetAtt - returns specific attribute
!GetAtt MyInstance.AvailabilityZone  # → "us-east-1a"
!GetAtt MyInstance.PrivateIp  # → "10.0.1.5"
!GetAtt MyInstance.PublicDnsName  # → "ec2-1-2-3-4.compute-1.amazonaws.com"
```

**Các attributes có sẵn** tùy thuộc vào loại resource (resource type) - xem tài liệu (docs) của AWS cho mỗi resource.

**`!Sub` - String substitution mạnh mẽ:**

```yaml
# Simple substitution
!Sub "arn:aws:s3:::${BucketName}"

# Với function calls
!Sub "https://${Domain}.s3.${AWS::Region}.amazonaws.com"

# Multi-line với template
!Sub |
  Hello ${User}
  Your account is ${AccountId}
  Stack: ${AWS::StackName}
```

#### NÊN làm
- **Dùng `!Sub` thay vì `!Join` khi có thể** → Vì `!Sub` ngắn gọn hơn, dễ đọc hơn. `!Sub "arn:${Partition}:s3:::${Bucket}"` ngắn hơn `!Join ["", ["arn:", !Ref Partition, ":s3:::", !Ref Bucket]]`.
- **Dùng `!GetAtt` thay vì hardcode giá trị attribute** → Vì attributes có thể thay đổi (private IP sau khi stop/start EC2). Hardcode sẽ sai.
- **Dùng `Fn::Cidr` để tạo subnet CIDRs tự động** → Vì tránh tính toán thủ công (manual calculation), dễ mở rộng (scale). Ví dụ: từ VPC `/16`, tạo 4 subnets `/18`: `!Cidr [!Ref VPCCidr, 4, 18]`.

#### KHÔNG NÊN làm
- **Dùng `!Sub` cho logic phức tạp (thay thế lồng nhau - nested substitutions)** → Vì template trở nên khó đọc. NÊN: dùng `!Join` cho các phép ghép chuỗi phức tạp (concatenations), dùng `!Select` để chỉ mục trong list (list indexing).
- **Dùng `Fn::Base64` cho dữ liệu nhạy cảm** → Vì Base64 chỉ là mã hóa dạng (encoding), không phải mã hóa bảo mật (encryption). NÊN: dùng `Secrets Manager` hoặc `SSM SecureString`.

---

## 7. CloudFormation Rollbacks

### Rollback chi tiết

#### Giải thích cốt lõi
CloudFormation có 3 kịch bản (scenarios) rollback:

**1. Tạo Stack thất bại (Stack Creation Fails):**
- Hành vi mặc định (Default behavior): **ROLLBACK** - xóa tất cả resources đã tạo
- Vô hiệu hóa rollback (Disable rollback): chỉnh `DisableRollback: true` → giữ lại các resources để debug
- Sau khi fix, xóa thủ công (manually delete) hoặc `ContinueUpdateRollback`

**2. Cập nhật Stack thất bại (Stack Update Fails):**
- Tự động rollback (auto rollback) về **trạng thái hoạt động tốt trước đó (previous known working state)**
- Có thể xem logs để biết lý do thất bại (fail)
- Trạng thái stack (Stack status): `UPDATE_ROLLBACK_COMPLETE` hoặc `UPDATE_ROLLBACK_FAILED`

**3. Rollback thất bại (Rollback Failure):**
- Các resources bị "kẹt" (stuck) ở trạng thái không nhất quán (inconsistent state)
- Sửa thủ công (Manual fix) từng resource
- Sau đó thực hiện (issue) API `continue-update-rollback`

**So sánh với các phương án khác (alternatives):**

| Kịch bản (Scenario) | CloudFormation | Terraform |
|----------|----------------|-----------|
| Tạo thất bại (Create fail) | Auto rollback (xóa tất cả, delete all) | Dừng lại, giữ state (Stop, keep state) |
| Cập nhật thất bại (Update fail) | Rollback về trạng thái tốt cuối (Rollback to last good state) | Rollback về trạng thái tốt cuối |
| Rollback thất bại (Rollback fail) | Sửa thủ công + `continue-update-rollback` | Làm mới state (refresh) + apply |

#### NÊN làm
- **Vô hiệu hóa rollback cho lần triển khai đầu (first deployment) của production stack** → Vì muốn **xem các resources đã tạo** để debug, không muốn bị xóa hết. Sau khi triển khai (deploy) thành công, bật lại rollback (enable rollback).
- **Dùng Change Sets cho production updates** → Vì xem trước (preview) các thay đổi trước khi áp dụng (apply), tránh xóa nhầm (accidental deletes). Change Sets hiển thị rõ resource nào sẽ bị thay thế (replaced), sửa đổi (modified), gỡ bỏ (removed).

#### KHÔNG NÊN làm
- **Tắt rollback vĩnh viễn cho production** → Vì nếu cập nhật (update) thất bại, stack ở trạng thái không nhất quán (inconsistent). NÊN: bật rollback cho production, chỉ vô hiệu hóa cho các kịch bản debug (debugging scenarios).
- **Bỏ qua (Ignore) trạng thái `UPDATE_ROLLBACK_FAILED`** → Vì stack bị "kẹt" (stuck), không thể cập nhật/xóa (update/delete) bình thường. NÊN: kiểm tra (investigate) ngay, sửa resources thủ công, thực hiện (issue) `continue-update-rollback`.

---

## 8. CloudFormation Service Role

### Thực hành tốt (Best Practices) với Service Role

#### Giải thích cốt lõi
CloudFormation cần IAM permissions để create/update/delete resources. Hai cách cấp permissions:

**Cách 1: User có đủ (full) permissions trực tiếp**
- Đơn giản nhưng **vi phạm (violates) nguyên tắc đặc quyền tối thiểu (least privilege)**
- Người dùng (User) phải có quyền đối với mọi resource trong template

**Cách 2: CloudFormation Service Role (được khuyến nghị - Recommended)**
- User chỉ cần `cloudformation:*` permissions
- Service Role có quyền tạo (create) resources
- User phải có **`iam:PassRole`** để chuyển (pass) Service Role cho CloudFormation

**Ví dụ IAM Policy cho Service Role:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSecurityGroup",
        "ec2:RunInstances",
        "ec2:CreateVpc"
      ],
      "Resource": "*"
    }
  ]
}
```

**Chính sách (Policy) người dùng (user) cần có:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["cloudformation:*"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::123456789012:role/CloudFormationServiceRole"
    }
  ]
}
```

#### NÊN làm
- **Dùng Service Role cho production** → Vì áp dụng **nguyên tắc đặc quyền tối thiểu (least privilege)**. Người dùng (Users) chỉ cần quyền triển khai (deploy) stacks, không cần quyền tạo trực tiếp EC2/RDS/etc.
- **Tạo nhiều Service Roles cho các trường hợp sử dụng (use cases) khác nhau** → Vì giới hạn phạm vi ảnh hưởng (blast radius). Ví dụ: `CFRole-Dev` (đủ quyền, full permissions), `CFRole-Prod-ReadOnly` (chỉ cho phép cập nhật, read-only cho updates).

#### KHÔNG NÊN làm
- **Cho user full AdministratorAccess chỉ để triển khai (deploy) stacks** → Vì vi phạm (violates) nguyên tắc đặc quyền tối thiểu (least privilege), user có thể tạo/xóa bất cứ gì. NÊN: tạo Service Role thu gọn phạm vi (scoped) theo resources cần thiết.
- **Quên (Forget) phân quyền `iam:PassRole`** → Vì CloudFormation sẽ thất bại (fail) với error "User is not authorized to perform iam:PassRole". NÊN: thêm `iam:PassRole` vào policy của user (user policy) khi dùng Service Role.

---

## 9. CloudFormation DeletionPolicy

### DeletionPolicy chi tiết

#### Giải thích cốt lõi
**DeletionPolicy** kiểm soát điều gì xảy ra khi:
- Stack bị xóa
- Resource bị xóa khỏi template

**3 loại:**

| Policy | Hành vi (Behavior) | Trường hợp dùng (Use Case) |
|--------|----------|----------|
| `Delete` (mặc định) | Xóa resource | Resources không có trạng thái (stateless, EC2, Security Group) |
| `Retain` | Giữ lại resource | Dữ liệu có state, log files, resources quản lý thủ công |
| `Snapshot` | Tạo snapshot trước khi xóa | EBS, RDS, ElastiCache, Redshift |

**Ví dụ:**

```yaml
Resources:
  # EC2 - không quan trọng, xóa luôn
  WebServer:
    Type: AWS::EC2::Instance
    DeletionPolicy: Delete  # default

  # EBS Volume có data - giữ lại
  DataVolume:
    Type: AWS::EC2::Volume
    DeletionPolicy: Retain

  # RDS Database - snapshot trước khi xóa
  Database:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot
    Properties:
      Engine: mysql
      DBInstanceClass: db.t3.micro
```

**Lưu ý quan trọng:**
- `Delete` không xóa được **S3 bucket không trống (non-empty S3 bucket)**
- `Snapshot` chỉ hỗ trợ (support) một số loại resource (resource types, xem slide 426)
- `Retain` cần dọn dẹp thủ công (manual cleanup) nếu muốn xóa sau

#### NÊN làm
- **Dùng `Snapshot` cho RDS databases** → Vì có backup tự động trước khi xóa, có thể khôi phục (restore) nếu cần. So với `Delete`: mất dữ liệu (data) vĩnh viễn.
- **Dùng `Retain` cho S3 buckets có data quan trọng** → Vì không lỡ tay xóa bucket khi xóa (delete) stack, dọn dẹp thủ công (manual cleanup) khi cần.

#### KHÔNG NÊN làm
- **Dùng `Delete` cho databases/EBS có data production** → Vì dữ liệu (data) mất vĩnh viễn. NÊN: `Snapshot` cho databases, `Retain` cho EBS volumes.
- **Dùng `Retain` cho tất cả resources** → Vì tạo ra các "rơi rớt" (orphaned) resources, tốn tiền, khó dọn dẹp (cleanup). NÊN: `Delete` cho resources không có trạng thái (stateless: EC2, ASG, ELB), `Retain`/`Snapshot` cho có trạng thái (stateful).

---

## 10. CloudFormation Stack Policies

### Stack Policies chi tiết

#### Giải thích cốt lõi
**Stack Policy** bảo vệ resources khỏi **các bản cập nhật không cố ý (unintentional updates)** trong stack update.

**Hành vi mặc định (Default behavior, không có Stack Policy):**
- Stack update có thể sửa/xóa (modify/delete) **bất kỳ resource** nào
- Nguy hiểm: cập nhật (update) template vô tình xóa production database

**Với Stack Policy:**
- Tất cả resources **được bảo vệ theo mặc định (protected by default)**, từ chối (deny) tất cả updates
- Phải cho phép (ALLOW) một cách tường minh (explicit) cho các resources muốn cập nhật
- Policy là một **tài liệu JSON (JSON document)**

**Ví dụ Stack Policy:**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/WebServer"
    },
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/Database"
    }
  ]
}
```

**Các actions có thể allow/deny:**
- `Update:*` - mọi update (Modify, Replace, etc.)
- `Update:Replace` - chỉ replace
- `Update:Delete` - chỉ delete
- `Update:Modify` - chỉ modify properties

#### NÊN làm
- **Dùng Stack Policy cho production databases** → Vì bảo vệ khỏi thay thế/xóa (replacement/deletion) không cẩn thận (accidental) khi cập nhật (update) template. Nếu cần cập nhật (update) DB, tạm thời sửa (modify) Stack Policy.
- **Dùng Stack Policy cho các resource quan trọng (critical resources: Encryption keys, IAM Roles)** → Vì những resources này khó tạo lại (recreate), ảnh hưởng lớn nếu bị thay thế (replace).

#### KHÔNG NÊN làm
- **Dùng Stack Policy chỉ với `Allow: *` cho mọi resource** → Vì tương đương với không có policy, không có sự bảo vệ (protection) nào. NÊN: từ chối (Deny) tường minh (explicit) cho các resource quan trọng (critical), chỉ cho phép (Allow) cho các resource muốn được cập nhật (update) được.
- **Quên (Forget) rằng Stack Policy áp dụng cho cả việc xóa (delete) stack** → Vì nếu muốn xóa (delete) stack, phải xóa Stack Policy trước (nếu có Deny cho resources). NÊN: lưu ý khi thiết kế.

---

## 11. CloudFormation Custom Resources

### Custom Resources chi tiết

#### Giải thích cốt lõi
**Custom Resources** mở rộng CloudFormation để:
1. Tạo các resource **chưa được hỗ trợ** (new AWS services)
2. Chạy (Run) **logic tùy chỉnh (custom logic)** trong các thao tác CFN (operations)
3. Quản lý (Manage) **các resource bên ngoài (external resources)** (on-premises, bên thứ 3 - 3rd party)

**Cấu trúc:**

```yaml
Resources:
  EmptyS3Bucket:
    Type: Custom::EmptyS3Bucket
    Properties:
      ServiceToken: !GetAtt EmptyS3Function.Arn
      BucketName: !Ref MyBucket

  EmptyS3Function:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.handler
      Role: !GetAtt LambdaRole.Arn
      Code:
        ZipFile: |
          import boto3
          import cfnresponse
          
          def handler(event, context):
              try:
                  if event['RequestType'] == 'Delete':
                      s3 = boto3.client('s3')
                      bucket = event['ResourceProperties']['BucketName']
                      # Delete all objects
                      objects = s3.list_objects_v2(Bucket=bucket)
                      if 'Contents' in objects:
                          for obj in objects['Contents']:
                              s3.delete_object(Bucket=bucket, Key=obj['Key'])
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {})
              except Exception as e:
                  cfnresponse.send(event, context, cfnresponse.FAILED, {})
```

**Sự kiện vòng đời (Lifecycle events):**
- `Create` - khi resource được tạo
- `Update` - khi properties thay đổi
- `Delete` - khi resource/stack bị xóa

#### NÊN làm
- **Dùng Custom Resource để làm trống (empty) S3 bucket trước khi xóa** → Vì CloudFormation không thể xóa S3 bucket không trống (non-empty bucket). Lambda function này chạy trước khi bucket bị xóa.
- **Dùng Custom Resource cho các resource bên thứ ba (third-party)** → Vì tích hợp Datadog, MongoDB Atlas, resources on-premises vào CloudFormation.
- **Lưu (cache) values trong Custom Resource** → Vì tránh gọi API nhiều lần. Ví dụ: tạo IAM Role, cache Role ARN.

#### KHÔNG NÊN làm
- **Dùng Custom Resource cho các resource mà AWS đã hỗ trợ sẵn** → Vì phức tạp, khó bảo trì (maintain). NÊN: dùng resources native của CloudFormation khi có thể.
- **Không xử lý lỗi (handle errors) trong Lambda** → Vì nếu Lambda thất bại (fail), stack update cũng thất bại (fail). NÊN: dùng try/except, gửi `cfnresponse.FAILED` kèm thông báo lỗi (error message) rõ ràng.
- **Quên tính nguyên ảnh (idempotency)** → Vì CF có thể gọi lại Lambda với cùng input. NÊN: thiết kế (design) Lambda để đạt nguyên ảnh (idempotent) - chạy nhiều lần vẫn cho cùng kết quả.

---

## 12. CloudFormation StackSets

### StackSets chi tiết

#### Giải thích cốt lõi
**StackSets** cho phép tạo/cập nhật/xóa (create/update/delete) stacks trên **nhiều (multiple) accounts và regions** với một template duy nhất (single template).

**Các trường hợp sử dụng (Use cases):**
- Triển khai (Deploy) **security baseline** (CloudTrail, Config) cho tất cả các accounts trong Organization
- Triển khai (Deploy) **hạ tầng ghi log (logging infrastructure)** ở nhiều regions
- Áp dụng (Enforce) **compliance policies** trên tất cả các accounts (across accounts)

**Các khái niệm (Concepts):**

| Thuật ngữ (Term) | Định nghĩa (Definition) |
|------|------------|
| **StackSet** | Template + parameters + capabilities |
| **Stack Instance** | Stack trong target account/region |
| **Administrator Account** | Account tạo/quản lý StackSet |
| **Target Account** | Account chứa các stack instance |
| **Service-managed permissions** | Tự động trust giữa các accounts trong Organization |
| **Self-managed permissions** | Thiết lập thủ công (manual setup) các IAM roles |

**Quy trình:**
1. Administrator account tạo StackSet
2. Chọn các target account (manual hoặc Organizational Units)
3. Chọn các region
4. Các stack instance được tạo tại mỗi target account/region
5. Cập nhật (Update) StackSet → tất cả instances được cập nhật (updated)

**So sánh với các phương án khác (alternatives):**

| Cách làm (Approach) | Trường hợp dùng (Use Case) | Độ phức tạp (Complexity) |
|----------|----------|------------|
| **StackSets** | Nhiều account, nhiều region (Multi-account, multi-region) | Trung bình - AWS tự quản lý (manages) |
| **Nested Stacks + cross-region** | Một (single) account, nhiều region | Cao - phải sao chép thủ công (manual replication) |
| **Custom automation (Lambda)** | Logic tùy chỉnh (Custom logic) | Cao - tự làm (DIY) |
| **AWS Organizations SCP** | Giới hạn phân quyền (Permission boundaries) | Thấp - nhưng chỉ về phân quyền |

#### NÊN làm
- **Dùng StackSets với service-managed permissions khi trong AWS Organization** → Vì AWS tự động tạo IAM roles, quan hệ tin cậy (trust relationships). Đơn giản hơn self-managed.
- **Dùng StackSets cho baseline infrastructure** (CloudTrail, Config, GuardDuty) → Vì mọi account đều cần có, tự động triển khai (deploy) khi thêm account mới.

#### KHÔNG NÊN làm
- **Dùng StackSets cho các stack ứng dụng cụ thể (application-specific)** → Vì mỗi account/region có ứng dụng (app) khác nhau, khó cấu hình (config). NÊN: dùng nested stacks hoặc CI/CD pipeline cho việc triển khai ứng dụng (application deployment).
- **Dùng StackSets mà không kiểm thử (test) trên ít account trước** → Vì một lỗi có thể ảnh hưởng hàng trăm accounts. NÊN: test trên 1-2 accounts, sau đó mới triển khai toàn bộ (rollout) Organization.

---

## 13. CloudFormation Best Practices tổng hợp

### Tổng hợp Best Practices

#### Giải thích cốt lõi
Dựa trên kinh nghiệm triển khai thực tế, có một số mẫu (patterns) giúp CloudFormation templates dễ bảo trì (maintain) hơn:

**1. Tổ chức Template (Template Organization):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Application stack - Web tier'

# 1. Parameters trước
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

# 2. Mappings
Mappings:
  ConfigMap:
    dev:
      InstanceType: t3.micro
    prod:
      InstanceType: m5.large

# 3. Conditions
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

# 4. Resources (theo thứ tự: Network → Security → Compute)
Resources:
  # Network
  VPC:
    Type: AWS::EC2::VPC
    ...
  # Security
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    ...
  # Compute
  Instance:
    Type: AWS::EC2::Instance
    ...
# 5. Outputs cuối
Outputs:
  InstanceId:
    Value: !Ref Instance
```

**2. Quy ước đặt tên (Naming Conventions):**
- Resource Logical IDs: PascalCase, có ý nghĩa (`WebServer` chứ không phải `Instance1`)
- Parameters: PascalCase với prefix theo nhóm (category) (`DbPassword`, `VpcCIDR`)

**3. Bảo mật (Security):**
- Không hardcode secrets
- Dùng `Secrets Manager` với dynamic reference
- Bật (Enable) mã hóa (encryption) theo mặc định

#### NÊN làm
- **Dùng Nested Stacks cho infrastructure lớn** → Vì chia nhỏ template → dễ đọc, dễ kiểm thử (test), dễ tái sử dụng (reuse). So với 1 template lớn: khó xem lại (review), sai 1 chỗ (fail) thì rollback tất cả.
- **Tag mọi resources với Environment, Project, Owner** → Vì theo dõi chi phí (tracking cost), dọn dẹp tự động (automation cleanup) dễ dàng.
- **Dùng Change Sets cho production updates** → Vì xem trước (preview) các thay đổi trước khi áp dụng (apply), tránh thay thế (replacement) resource nhầm (accidental).
- **Lưu (Store) templates trong Git** → Vì quản lý phiên bản (version control), xem lại (review) code, có dấu vết kiểm toán (audit trail).
- **Bật (Enable) CloudFormation Drift Detection** → Vì phát hiện các resources bị sửa đổi (modify) ngoài CloudFormation (thay đổi thủ công - manual changes qua console).

#### KHÔNG NÊN làm
- **Viết 1 template > 500 lines** → Vì khó đọc, khó xem lại (review), dễ sai nhiều (fail). NÊN: chia thành nested stacks, mỗi stack < 300 lines.
- **Tự sửa (Manual edit) resources trong console sau khi triển khai (deploy)** → Vì CloudFormation không biết → drift detection sẽ cảnh báo. NÊN: cập nhật (update) template + áp dụng (apply) qua CloudFormation.
- **Hardcode AMI IDs cụ thể** → Vì AMI bị ngừng hỗ trợ (deprecate) theo thời gian. NÊN: dùng `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` để lấy AMI mới nhất (latest).
- **Bỏ qua (Skip) kiểm thử (testing) cho production stacks** → Vì một lỗi template có thể xóa dữ liệu (data). NÊN: kiểm thử (test) trên dev account trước, dùng Change Sets để xem lại (review).

---

## 14. CloudFormation Drift Detection

### Drift Detection chi tiết

#### Giải thích cốt lõi
**Drift** (lệch trạng thái) xảy ra khi resources trong AWS **bị sửa đổi (modified)** ngoài CloudFormation (qua console, CLI, hoặc service khác). Khi đó, state của CloudFormation không khớp với trạng thái thực tế (actual state).

**Ví dụ:**
- Template tạo Security Group với rule SSH (port 22)
- Người dùng (User) tự thêm (manual add) rule RDP (port 3389) qua console
- CloudFormation không biết về rule này
- Nếu cập nhật (update) stack, có thể **ghi đè** rule RDP

**Phát hiện drift:**

```bash
# Detect drift cho 1 stack
aws cloudformation detect-stack-drift --stack-name my-stack

# Xem chi tiết drift
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

**Trạng thái drift (Drift status):**
- `IN_SYNC` - khớp với template
- `MODIFIED` - bị thay đổi
- `DELETED` - bị xóa
- `NOT_CHECKED` - chưa kiểm tra

#### NÊN làm
- **Bật (Enable) drift detection định kỳ (tuần/tháng)** → Vì phát hiện các thay đổi thủ công (manual changes) sớm, tránh bất ngờ (surprises) khi cập nhật (update) stack. Có thể cài đặt (setup) CloudWatch Event rule tự động.
- **Dùng IAM permissions để hạn chế (restrict) các thay đổi thủ công** → Vì ngăn người dùng (users) chỉnh sửa (edit) resources qua console khi resources được quản lý bởi CloudFormation. Ví dụ: từ chối (deny) `ec2:CreateSecurityGroup` cho các user không phải admin (non-admin users).

#### KHÔNG NÊN làm
- **Bỏ qua (Ignore) drift detection** → Vì stack update có thể **ghi đè (overwrite)** các thay đổi thủ công, gây mất cấu hình. NÊN: kiểm tra (investigate) drift, cập nhật (update) template nếu cần, hoặc hoàn tác (revert) các thay đổi thủ công.
- **Chạy (Run) drift detection liên tục (mỗi phút)** → Vì tốn API calls, có thể chạm (hit) giới hạn tốc độ (rate limits). NÊN: lên lịch (schedule) theo tuần hoặc tháng, kết hợp với CloudWatch Events.

---

## 15. CloudFormation vs CDK vs Terraform

### So sánh chi tiết

#### Giải thích cốt lõi

| Tính năng (Feature) | CloudFormation | CDK | Terraform |
|---------|----------------|-----|-----------|
| **Syntax** | YAML/JSON | TypeScript, Python, Java, Go, .NET | HCL (HashiCorp Language) |
| **State Management** | AWS tự quản lý (AWS-managed) | AWS tự quản lý (biên dịch thành CF) | File state (local hoặc S3) |
| **Multi-cloud** | ❌ Chỉ AWS (AWS only) | ❌ Chỉ AWS (AWS only, chính) | ✅ AWS, Azure, GCP, v.v. |
| **Learning Curve** | Trung bình (YAML, intrinsic functions) | Cao (các khái niệm lập trình) | Trung bình (cú pháp HCL) |
| **Community** | Native của AWS | Native của AWS | Rất lớn, multi-cloud |
| **Cost** | Miễn phí (chỉ trả tiền resources) | Miễn phí | Miễn phí (Terraform Cloud có phí) |
| **Drift Detection** | ✅ Có sẵn (Built-in) | ✅ Có sẵn (qua CF) | ✅ Có sẵn |
| **Rollback** | ✅ Auto rollback | ✅ Auto rollback (qua CF) | ⚠️ Thủ công (manual) hoặc tự động |

**Khi nào dùng cái nào:**

| Trường hợp dùng (Use Case) | Nên dùng (Recommended) |
|----------|-------------|
| **Chỉ AWS (AWS-only), team thích YAML** | CloudFormation |
| **Chỉ AWS, team là developers** | CDK |
| **Multi-cloud** | Terraform |
| **Compliance cao, cần AWS-managed state** | CloudFormation |
| **Hệ sinh thái mã nguồn mở (Open-source ecosystem)** | Terraform |

#### NÊN làm
- **Chọn CloudFormation khi chỉ dùng AWS (AWS-only) + cần AWS-managed state** → Vì không cần hạ tầng phụ (zero infrastructure) để quản lý state, tích hợp (integrate) tốt với các AWS services (CodePipeline, Service Catalog).
- **Dùng CDK khi team là developers và code nhiều logic** → Vì CDK cho phép viết loops, conditions, abstractions dễ dàng hơn YAML.

#### KHÔNG NÊN làm
- **Trộn (Mix) CloudFormation và Terraform cho cùng một resources** → Vì hai tools sẽ "tranh giành" (fight) nhau, drift detection của cả hai cùng báo cáo (report). NÊN: chọn 1 tool cho infrastructure.
- **Dùng CloudFormation cho chiến lược multi-cloud** → Vì giới hạn chỉ trong AWS (AWS only). NÊN: Terraform hoặc Pulumi cho multi-cloud.

---

## 16. Exam Tips cho CloudFormation

### Các điểm trọng tâm

#### Giải thích cốt lõi
Dựa trên exam guide và câu hỏi DVA-C02, CloudFormation tập trung vào:

**1. Intrinsic Functions (Hay xuất hiện nhất):**
- `!Ref` vs `!GetAtt` - khi nào dùng cái nào
- `!Sub` cho thay chuỗi (string substitution)
- `!Join` cho ghép chuỗi phức tạp (concatenations)
- `!FindInMap` cho giá trị theo region (region-specific values)
- `!ImportValue` cho tham chiếu cross-stack (cross-stack references)

**2. Quản lý Stack (Stack Management):**
- Hành vi Rollback (khi Tạo, Cập nhật, Thất bại)
- DeletionPolicy (Delete, Retain, Snapshot)
- Stack Policies (bảo vệ resources)
- Termination Protection
- Drift Detection

**3. Tính năng nâng cao (Advanced Features):**
- Custom Resources (dựa trên Lambda)
- StackSets (nhiều account, nhiều region)
- Nested Stacks
- Change Sets
- Service Roles

**4. Parameters vs Mappings:**
- Parameters: đầu vào của user (user input)
- Mappings: bảng tra mã hóa sẵn (hardcoded lookup)

**5. Outputs:**
- Tham chiếu cross-stack (cross-stack references)
- Tên Export phải duy nhất (unique)

#### NÊN làm
- **Học thuộc cú pháp rút gọn (shorthand) của YAML** → Vì thi (exam) cho cả JSON và YAML. YAML: `!Ref`, `!Sub`, `!GetAtt`. JSON: `{"Ref": "...", "Fn::Sub": "..."}`.
- **Hiểu rõ các hành vi (behaviors) của DeletionPolicy** → Vì hay có câu hỏi: "RDS bị xóa khi delete stack - làm sao giữ data?" → Đáp án (Answer): `DeletionPolicy: Snapshot`.
- **Biết khi nào dùng Custom Resources** → Vì câu hỏi: "Cần xóa non-empty S3 bucket" → Đáp án (Answer): Custom Resource dựa trên Lambda.

#### KHÔNG NÊN làm
- **Học thuộc lòng 700+ resource types** → Vì không khả thi và không cần thiết. NÊN: hiểu cấu trúc (structure) `service-provider::service-name::data-type-name`, biết cách tra cứu tài liệu (docs).
- **Bỏ qua Drift Detection** → Vì là chủ đề (topic) mới trong kỳ thi (exam), AWS hay hỏi về "phát hiện resources bị thay đổi ngoài CloudFormation".

---

## Tổng kết Section 8

### Các khái niệm cần nhớ:

| Khái niệm (Concept) | Điểm chính (Key Point) |
|---------|-----------|
| **Declarative IaC** | Khai báo trạng thái mong muốn (desired state), AWS lo phần chi tiết |
| **Parameters** | Đầu vào của user (user inputs), giá trị động (dynamic values) |
| **Mappings** | Bảng tra mã hóa sẵn (hardcoded lookups: Region, AMI) |
| **Outputs** | Tham chiếu cross-stack (cross-stack references) qua `!ImportValue` |
| **Conditions** | Tạo resource có điều kiện (conditional resource creation) |
| **Intrinsic Functions** | `!Ref`, `!GetAtt`, `!Sub`, `!FindInMap`, `!ImportValue` |
| **Rollback** | Tự động rollback (auto-rollback) khi tạo/cập nhật thất bại (create/update fail) |
| **Service Role** | IAM role để CloudFormation thực hiện thay mặt (on behalf of) user |
| **DeletionPolicy** | Delete (mặc định), Retain, Snapshot |
| **Stack Policy** | Bảo vệ resources khỏi cập nhật nhầm (accidental updates) |
| **Custom Resources** | Mở rộng CF bằng logic tùy chỉnh (custom logic) dựa trên Lambda |
| **StackSets** | Triển khai (deployment) trên nhiều account, nhiều region |
| **Drift Detection** | Phát hiện (detect) các thay đổi thủ công (manual changes) ngoài CloudFormation |
