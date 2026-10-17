# Section 8 - Kiến thức mở rộng: AWS CloudFormation

---

## 1. CloudFormation - Cơ chế hoạt động chi tiết

### CloudFormation Template Lifecycle

#### Giải thích cốt lõi
CloudFormation hoạt động theo cơ chế **declarative** (khai báo) chứ không phải **imperative** (mệnh lệnh). Khi bạn upload template, CloudFormation:

1. **Parse template** → So sánh desired state (trong template) với current state (trong AWS)
2. **Tính toán changes cần thiết** → Ví dụ: tạo mới EC2, update security group rule, không thay đổi RDS
3. **Thực thi changes theo thứ tự**:
   - Tạo VPC → Subnets → Security Groups → EC2 instances → Load Balancer
   - Phụ thuộc được tự động phát hiện (qua `!Ref` và `!GetAtt`)
4. **Stack states**:
   - `CREATE_IN_PROGRESS` → `CREATE_COMPLETE` (hoặc `CREATE_FAILED` → `ROLLBACK_COMPLETE`)
   - `UPDATE_IN_PROGRESS` → `UPDATE_COMPLETE` (hoặc `UPDATE_FAILED` → `UPDATE_ROLLBACK_COMPLETE`)
   - `DELETE_IN_PROGRESS` → `DELETE_COMPLETE` (hoặc `DELETE_FAILED`)

**So sánh với alternatives:**

| Công cụ | Approach | State Management | Best For |
|---------|----------|------------------|----------|
| **CloudFormation** | Declarative | AWS tự quản lý state | AWS-only, mature |
| **Terraform** | Declarative | State file riêng | Multi-cloud |
| **CDK** | Imperative (compiles to CF) | AWS tự quản lý state | Developers thích code |
| **Pulumi** | Imperative (general languages) | State file riêng | Multi-cloud + code |
| **Ansible** | Imperative | Không có state tracking | Config management |

#### NÊN làm
- **Bắt đầu với CloudFormation khi infrastructure là AWS-only** → Vì native, free (chỉ trả tiền resources), integrate tốt với AWS services, có **drift detection**. So với Terraform: cần quản lý state file riêng, tốn thêm learning curve.
- **Dùng YAML thay vì JSON** → Vì YAML hỗ trợ comments, multi-line strings, ít dấu ngoặc. JSON "horrible for CF" vì verbose, không có comments, dễ sai syntax.
- **Tag mọi resource bằng stack name** → Vì AWS tự động tag resources với `aws:cloudformation:stack-name`, giúp tracking cost, cleanup, debugging dễ dàng.

#### KHÔNG NÊN làm
- **Viết 1 template khổng lồ chứa toàn bộ infrastructure** → Vì khi fail 1 resource, rollback toàn bộ stack. NÊN: chia thành **nested stacks** (VPC stack, Network stack, App stack) → dễ quản lý, dễ reuse, fail isolation.
- **Hardcode giá trị trong Resources** → Vì mỗi lần thay đổi phải re-upload template. NÊN: dùng **Parameters** cho values thay đổi được (InstanceType, CIDR), **Mappings** cho values theo Region/Environment.

---

## 2. CloudFormation Parameters

### Parameters Best Practices

#### Giải thích cốt lõi
Parameters cho phép template trở nên **reusable** cho nhiều environments/teams. Có 2 loại Parameters chính:

**AWS-Specific Parameters** - AWS validate giá trị với existing values:
- `AWS::EC2::VPC::Id` - chỉ chấp nhận VPC ID tồn tại trong account
- `AWS::EC2::Subnet::Id` - chỉ chấp nhận Subnet ID
- `AWS::EC2::SecurityGroup::Id`
- `List<AWS::EC2::VPC::Id>` - danh sách VPC IDs

**SSM Parameter Types** - Lấy value từ SSM Parameter Store:
- `AWS::SSM::Parameter::Value<String>`
- `AWS::SSM::Parameter::Value<AWS::EC2::VPC::Id>`
- Cho phép **centralized configuration management**

**So sánh các approaches:**

| Approach | Use Case | Security |
|----------|----------|----------|
| Hardcoded values | Single env, không cần thay đổi | ✅ N/A |
| Parameters | Multi-env, runtime config | ⚠️ Cần careful với NoEcho |
| Mappings | Region/AMI-specific values | ✅ Hardcoded, safe |
| SSM Parameter Store | Shared config across stacks | ✅ Encrypted, versioned |
| Secrets Manager | Sensitive data (DB passwords) | ✅ Auto-rotation, audit |

#### NÊN làm
- **Dùng AWS-Specific Parameters thay vì String cho IDs** → Vì AWS sẽ **validate** giá trị có tồn tại. Nếu user nhập sai VPC ID, stack fail ngay lúc validate chứ không phải lúc runtime.
- **Dùng `AllowedValues` để giới hạn options** → Vì nó chống typo và document rõ ràng options. Ví dụ: `AllowedValues: [t3.micro, t3.small, t3.medium]`.
- **Dùng `NoEcho: true` cho sensitive values** → Vì mặc định parameter values hiển thị trong CloudFormation console/events. Với secrets, NÊN dùng **Secrets Manager** hoặc **SSM SecureString** thay vì parameter.

#### KHÔNG NÊN làm
- **Đặt Default value cho mọi parameter** → Vì user có thể skip nhập và dùng default không phù hợp. NÊN: chỉ default cho optional parameters, **không default** cho critical values (production instance type, CIDR).
- **Truyền database password qua Parameters** → Vì password hiển thị trong CloudFormation logs/events. NÊN: dùng `Secrets Manager` với dynamic reference: `{{resolve:secretsmanager:MySecret:SecretString:password}}`.

---

## 3. CloudFormation Mappings

### Mappings chi tiết

#### Giải thích cốt lõi
Mappings là **hardcoded lookup tables** trong template. Khác với Parameters (user input), Mappings là **developer-defined** values, thường dùng cho:

1. **Region-specific values** (AMI IDs per region)
2. **Environment-specific configs** (dev/staging/prod sizes)
3. **Account-specific values** (ARNs, account IDs)
4. **Conditional configurations** theo một variable có thể determine lúc deploy

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
- **Dùng Mappings cho AMI IDs per region** → Vì AMIs là **region-specific** (không dùng được AMI us-east-1 ở eu-west-1). Hardcode mapping → không cần update mỗi khi AWS ra AMI mới (trừ khi dùng SSM Parameter).
- **Dùng Mappings kết hợp với `!Ref AWS::Region`** → Vì tự động chọn đúng config cho region đang deploy, không cần user nhập.

#### KHÔNG NÊN làm
- **Dùng Mappings cho values user cần thay đổi thường xuyên** → Vì mỗi lần thay đổi phải edit template và re-upload. NÊN: dùng **Parameters** cho user-specific values, dùng **SSM Parameter Store** cho shared config across stacks.
- **Dùng Mappings cho data quá lớn (>100 entries)** → Vì template sẽ khó đọc. NÊN: chuyển data lên S3 hoặc DynamoDB, dùng Custom Resource để fetch.

---

## 4. CloudFormation Outputs & Cross-Stack References

### Cross-Stack Reference Pattern

#### Giải thích cốt lõi
**Outputs** cho phép một stack expose values để các stacks khác consume. Có 2 cách:

1. **Outputs without Export** - chỉ để xem (CLI/Console)
2. **Outputs with Export** - để cross-stack import (qua `Fn::ImportValue`)

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
- **Export names phải UNIQUE trong account/region**
- **Không thể xóa stack gốc** nếu vẫn còn stack khác đang import từ nó
- Nếu muốn xóa stack có exports → phải xóa tất cả dependent stacks trước

**So sánh với alternatives:**

| Pattern | Pros | Cons |
|---------|------|------|
| **Outputs + ImportValue** | Native CF, no extra cost | Stack deletion coupling |
| **SSM Parameter Store** | Loose coupling, easier deletion | Cost, eventual consistency |
| **DynamoDB lookup** | Very flexible | Need custom resource, more complex |
| **Nested Stacks** | Strong coupling, easy lifecycle | All-in-one template |

#### NÊN làm
- **Dùng `!Sub "${AWS::StackName}-VPC-ID"` làm Export name** → Vì stack name là unique → export name cũng unique. Tránh hardcode export name.
- **Dùng Outputs cho shared infrastructure (VPC, Subnet, SG IDs)** → Vì nhiều stacks cần consume, **single source of truth**.

#### KHÔNG NÊN làm
- **Export quá nhiều values không cần thiết** → Vì mỗi export tạo "lock" - khó xóa stack. NÊN: chỉ export những gì thực sự cross-stack cần.
- **Dùng cross-stack references cho high-frequency updates** → Vì khi update giá trị ở stack gốc, tất cả dependent stacks bị affected. NÊN: dùng **SSM Parameter Store** cho values cần thay đổi runtime.

---

## 5. CloudFormation Conditions

### Conditions chi tiết

#### Giải thích cốt lõi
Conditions cho phép **conditionally create resources** dựa trên:
- Parameter values (Environment, Region)
- Pseudo parameters
- Other conditions

**Các intrinsic functions cho Conditions:**

| Function | Logic | Example |
|----------|-------|---------|
| `Fn::Equals` | A == B | `!Equals [!Ref Env, prod]` |
| `Fn::And` | All true | `!And [!Equals, !Equals]` |
| `Fn::Or` | Any true | `!Or [IsProd, IsStaging]` |
| `Fn::Not` | Negate | `!Not [!Equals [!Ref Env, dev]]` |
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
- **Dùng Conditions để tạo environment-specific resources** → Vì template 1 cho nhiều môi trường (dev, staging, prod) nhưng vẫn customize resources theo từng env. So với việc có 3 templates riêng: dễ bị drift, khó maintain.
- **Dùng `!If` để switch giữa các giá trị** → Vì tránh duplicate resources. Ví dụ: `InstanceType: !If [IsProd, m5.large, t3.micro]`.

#### KHÔNG NÊN làm
- **Dùng Conditions thay vì Nested Stacks cho logic quá phức tạp** → Vì Conditions chỉ true/false, không thể express "for each X, create Y". NÊN: dùng **Nested Stacks** cho multi-environment, dùng **Modules** (CloudFormation Modules) cho reusable components.
- **Dùng Conditions với circular references** → Vì CloudFormation không detect được → deploy fail với error khó hiểu. NÊN: test kỹ với mọi combination.

---

## 6. CloudFormation Intrinsic Functions

### Functions chi tiết

#### Giải thích cốt lõi
**MUST KNOW (in đậm trong slide - "Pink"):**
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
- Condition Functions (Fn::If, Fn::Not, Fn::Equals, etc.)

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

**Các attributes có sẵn** tùy thuộc vào resource type - xem AWS docs cho mỗi resource.

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
- **Dùng `!GetAtt` thay vì hardcode attribute values** → Vì attributes có thể thay đổi (private IP sau khi stop/start EC2). Hardcode sẽ sai.
- **Dùng `Fn::Cidr` để tạo subnet CIDRs tự động** → Vì tránh manual calculation, dễ scale. Ví dụ: từ VPC `/16`, tạo 4 subnets `/18`: `!Cidr [!Ref VPCCidr, 4, 18]`.

#### KHÔNG NÊN làm
- **Dùng `!Sub` cho logic phức tạp (nested substitutions)** → Vì template trở nên khó đọc. NÊN: dùng `!Join` cho concatenations phức tạp, dùng `!Select` cho list indexing.
- **Dùng `Fn::Base64` cho dữ liệu nhạy cảm** → Vì Base64 chỉ là encoding, không phải encryption. NÊN: dùng `Secrets Manager` hoặc `SSM SecureString`.

---

## 7. CloudFormation Rollbacks

### Rollback chi tiết

#### Giải thích cốt lõi
CloudFormation có 3 loại rollback scenarios:

**1. Stack Creation Fails:**
- Default behavior: **ROLLBACK** - xóa tất cả resources đã tạo
- Disable rollback: chỉnh `DisableRollback: true` → giữ resources để debug
- Sau khi fix, manually delete hoặc `ContinueUpdateRollback`

**2. Stack Update Fails:**
- Auto rollback về **previous known working state**
- Có thể xem logs để biết lý do fail
- Stack status: `UPDATE_ROLLBACK_COMPLETE` hoặc `UPDATE_ROLLBACK_FAILED`

**3. Rollback Failure:**
- Resources bị stuck ở inconsistent state
- Manual fix từng resource
- Sau đó issue `continue-update-rollback` API

**So sánh với alternatives:**

| Scenario | CloudFormation | Terraform |
|----------|----------------|-----------|
| Create fail | Auto rollback (delete all) | Stop, keep state |
| Update fail | Rollback to last good state | Rollback to last good state |
| Rollback fail | Manual fix + continue-update-rollback | Refresh state + apply |

#### NÊN làm
- **Disable rollback cho production stack first deployment** → Vì muốn **xem resources đã tạo** để debug, không muốn bị xóa hết. Sau khi deploy thành công, enable rollback lại.
- **Dùng Change Sets cho production updates** → Vì preview changes trước khi apply, tránh accidental deletes. Change Sets hiển thị rõ resource nào sẽ be replaced, modified, removed.

#### KHÔNG NÊN làm
- **Tắt rollback vĩnh viễn cho production** → Vì nếu update fail, stack ở trạng thái inconsistent. NÊN: enable rollback cho prod, disable chỉ cho debugging scenarios.
- **Ignore `UPDATE_ROLLBACK_FAILED` status** → Vì stack bị stuck, không thể update/delete bình thường. NÊN: investigate ngay, fix resources thủ công, issue `continue-update-rollback`.

---

## 8. CloudFormation Service Role

### Service Role Best Practices

#### Giải thích cốt lõi
CloudFormation cần IAM permissions để create/update/delete resources. Hai cách cấp permissions:

**Cách 1: User có full permissions trực tiếp**
- Đơn giản nhưng **violates least privilege**
- User phải có quyền đối với mọi resource trong template

**Cách 2: CloudFormation Service Role (Recommended)**
- User chỉ cần `cloudformation:*` permissions
- Service Role có quyền create resources
- User phải có **`iam:PassRole`** để pass Service Role cho CloudFormation

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

**User policy cần:**

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
- **Dùng Service Role cho production** → Vì áp dụng **least privilege** principle. Users chỉ cần quyền deploy stacks, không cần quyền tạo trực tiếp EC2/RDS/etc.
- **Tạo nhiều Service Roles cho các use cases khác nhau** → Vì giới hạn blast radius. Ví dụ: `CFRole-Dev` (full perms), `CFRole-Prod-ReadOnly` (chỉ updates).

#### KHÔNG NÊN làm
- **Cho user full AdministratorAccess chỉ để deploy stacks** → Vì violates least privilege, user có thể tạo/xóa bất cứ gì. NÊN: tạo Service Role scoped theo resources cần thiết.
- **Forget `iam:PassRole` permission** → Vì CloudFormation sẽ fail với error "User is not authorized to perform iam:PassRole". NÊN: thêm `iam:PassRole` vào user policy khi dùng Service Role.

---

## 9. CloudFormation DeletionPolicy

### DeletionPolicy chi tiết

#### Giải thích cốt lõi
**DeletionPolicy** kiểm soát điều gì xảy ra khi:
- Stack bị xóa
- Resource bị xóa khỏi template

**3 loại:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `Delete` (default) | Xóa resource | Stateless resources (EC2, Security Group) |
| `Retain` | Giữ lại resource | State data, log files, manually-managed resources |
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
- `Delete` không xóa được **non-empty S3 bucket**
- `Snapshot` chỉ support một số resource types (xem slide 426)
- `Retain` cần manual cleanup nếu muốn xóa sau

#### NÊN làm
- **Dùng `Snapshot` cho RDS databases** → Vì có backup tự động trước khi xóa, có thể restore nếu cần. So với `Delete`: mất data vĩnh viễn.
- **Dùng `Retain` cho S3 buckets có data quan trọng** → Vì không lỡ tay xóa bucket khi delete stack, manual cleanup khi cần.

#### KHÔNG NÊN làm
- **Dùng `Delete` cho databases/EBS có data production** → Vì data mất vĩnh viễn. NÊN: `Snapshot` cho databases, `Retain` cho EBS volumes.
- **Dùng `Retain` cho tất cả resources** → Vì tạo "orphaned" resources, tốn tiền, khó cleanup. NÊN: `Delete` cho stateless resources (EC2, ASG, ELB), `Retain`/`Snapshot` cho stateful.

---

## 10. CloudFormation Stack Policies

### Stack Policies chi tiết

#### Giải thích cốt lõi
**Stack Policy** bảo vệ resources khỏi **unintentional updates** trong stack update.

**Default behavior** (không có Stack Policy):
- Stack update có thể modify/delete **bất kỳ resource** nào
- Nguy hiểm: update template vô tình xóa production database

**Với Stack Policy:**
- Tất cả resources **protected by default** (deny tất cả updates)
- Phải explicit **ALLOW** cho resources muốn update
- Policy là **JSON document**

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
- **Dùng Stack Policy cho production databases** → Vì bảo vệ khỏi accidental replacement/deletion khi update template. Nếu cần update DB, tạm thời modify Stack Policy.
- **Dùng Stack Policy cho critical resources (Encryption keys, IAM Roles)** → Vì những resources này khó recreate, ảnh hưởng lớn nếu bị replace.

#### KHÔNG NÊN làm
- **Dùng Stack Policy chỉ với `Allow: *` cho mọi resource** → Vì tương đương với không có policy, không có protection. NÊN: explicit Deny cho critical resources, chỉ Allow cho resources muốn update được.
- **Forget rằng Stack Policy áp dụng cho cả stack delete** → Vì nếu muốn delete stack, phải delete Stack Policy trước (nếu có Deny cho resources). NÊN: lưu ý khi thiết kế.

---

## 11. CloudFormation Custom Resources

### Custom Resources chi tiết

#### Giải thích cốt lõi
**Custom Resources** mở rộng CloudFormation để:
1. Tạo resources **chưa được hỗ trợ** (new AWS services)
2. Run **custom logic** trong quá trình CFN operations
3. Manage **external resources** (on-premises, 3rd party)

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

**Lifecycle events:**
- `Create` - khi resource được tạo
- `Update` - khi properties thay đổi
- `Delete` - khi resource/stack bị xóa

#### NÊN làm
- **Dùng Custom Resource để empty S3 bucket trước khi xóa** → Vì CloudFormation không thể xóa non-empty bucket. Lambda function này chạy trước khi bucket bị xóa.
- **Dùng Custom Resource cho third-party resources** → Vì tích hợp Datadog, MongoDB Atlas, on-premises resources vào CloudFormation.
- **Cache values trong Custom Resource** → Vì tránh gọi API nhiều lần. Ví dụ: tạo IAM Role, cache Role ARN.

#### KHÔNG NÊN làm
- **Dùng Custom Resource cho resources AWS đã hỗ trợ** → Vì phức tạp, khó maintain. NÊN: dùng native CloudFormation resources khi có thể.
- **Không handle errors trong Lambda** → Vì nếu Lambda fail, stack update fail. NÊN: try/except, gửi `cfnresponse.FAILED` với error message rõ ràng.
- **Quên idempotency** → Vì CF có thể gọi lại Lambda với cùng input. NÊN: design Lambda để idempotent - chạy nhiều lần cũng cho cùng kết quả.

---

## 12. CloudFormation StackSets

### StackSets chi tiết

#### Giải thích cốt lõi
**StackSets** cho phép tạo/update/delete stacks trên **multiple accounts và regions** với một single template.

**Use cases:**
- Deploy **security baseline** (CloudTrail, Config) cho tất cả accounts trong Organization
- Deploy **logging infrastructure** ở nhiều regions
- Enforce **compliance policies** across accounts

**Concepts:**

| Term | Definition |
|------|------------|
| **StackSet** | Template + parameters + capabilities |
| **Stack Instance** | Stack trong target account/region |
| **Administrator Account** | Account tạo/quản lý StackSet |
| **Target Account** | Account chứa stack instances |
| **Service-managed permissions** | Tự động trust giữa accounts trong Organization |
| **Self-managed permissions** | Manual setup IAM roles |

**Quy trình:**
1. Administrator account tạo StackSet
2. Chọn target accounts (manual hoặc Organizational Units)
3. Chọn regions
4. Stack instances được tạo tại mỗi target account/region
5. Update StackSet → tất cả instances updated

**So sánh với alternatives:**

| Approach | Use Case | Complexity |
|----------|----------|------------|
| **StackSets** | Multi-account, multi-region | Medium - AWS manages |
| **Nested Stacks + cross-region** | Single account, multi-region | High - manual replication |
| **Custom automation (Lambda)** | Custom logic | High - DIY |
| **AWS Organizations SCP** | Permission boundaries | Low - but only permissions |

#### NÊN làm
- **Dùng StackSets với service-managed permissions khi trong AWS Organization** → Vì AWS tự động tạo IAM roles, trust relationships. Đơn giản hơn self-managed.
- **Dùng StackSets cho baseline infrastructure** (CloudTrail, Config, GuardDuty) → Vì mọi account cần có, tự động deploy khi thêm account mới.

#### KHÔNG NÊN làm
- **Dùng StackSets cho application-specific stacks** → Vì mỗi account/region có app khác nhau, khó config. NÊN: dùng nested stacks hoặc CI/CD pipeline cho application deployment.
- **Dùng StackSets mà không test trên ít account trước** → Vì một lỗi có thể ảnh hưởng hàng trăm accounts. NÊN: test trên 1-2 accounts, sau đó mới rollout toàn bộ Organization.

---

## 13. CloudFormation Best Practices tổng hợp

### Tổng hợp Best Practices

#### Giải thích cốt lõi
Dựa trên kinh nghiệm triển khai thực tế, có một số patterns giúp CloudFormation templates dễ maintain hơn:

**1. Template Organization:**
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

**2. Naming Conventions:**
- Resource Logical IDs: PascalCase, có ý nghĩa (`WebServer` chứ không phải `Instance1`)
- Parameters: PascalCase với prefix theo category (`DbPassword`, `VpcCIDR`)

**3. Security:**
- Không hardcode secrets
- Dùng `Secrets Manager` với dynamic reference
- Enable encryption by default

#### NÊN làm
- **Dùng Nested Stacks cho infrastructure lớn** → Vì chia nhỏ template → dễ đọc, dễ test, dễ reuse. So với 1 template lớn: khó review, fail 1 chỗ rollback tất cả.
- **Tag mọi resources với Environment, Project, Owner** → Vì tracking cost, automation cleanup dễ dàng.
- **Dùng Change Sets cho production updates** → Vì preview changes trước khi apply, tránh accidental resource replacement.
- **Store templates trong Git** → Vì version control, code review, audit trail.
- **Enable CloudFormation Drift Detection** → Vì phát hiện resources bị modify ngoài CloudFormation (manual changes qua console).

#### KHÔNG NÊN làm
- **Viết 1 template > 500 lines** → Vì khó đọc, khó review, fail nhiều. NÊN: chia thành nested stacks, mỗi stack < 300 lines.
- **Manual edit resources trong console sau khi deploy** → Vì CloudFormation không biết → drift detection sẽ cảnh báo. NÊN: update template + apply qua CloudFormation.
- **Hardcode AMI IDs cụ thể** → Vì AMI bị deprecate theo thời gian. NÊN: dùng `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` để lấy latest AMI.
- **Skip testing cho production stacks** → Vì một lỗi template có thể xóa data. NÊN: test trên dev account trước, dùng Change Sets để review.

---

## 14. CloudFormation Drift Detection

### Drift Detection chi tiết

#### Giải thích cốt lõi
**Drift** xảy ra khi resources trong AWS **bị modified** ngoài CloudFormation (qua console, CLI, hoặc service khác). Khi đó, CloudFormation state không khớp với actual state.

**Ví dụ:**
- Template tạo Security Group với rule SSH (port 22)
- User manual add rule RDP (port 3389) qua console
- CloudFormation không biết về rule này
- Nếu update stack, có thể **ghi đè** rule RDP

**Phát hiện drift:**

```bash
# Detect drift cho 1 stack
aws cloudformation detect-stack-drift --stack-name my-stack

# Xem chi tiết drift
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

**Drift status:**
- `IN_SYNC` - khớp với template
- `MODIFIED` - bị thay đổi
- `DELETED` - bị xóa
- `NOT_CHECKED` - chưa kiểm tra

#### NÊN làm
- **Enable drift detection định kỳ (weekly/monthly)** → Vì phát hiện manual changes sớm, tránh surprises khi update stack. Có thể setup CloudWatch Event rule tự động.
- **Dùng IAM permissions để restrict manual changes** → Vì ngăn users edit resources qua console khi resources được quản lý bởi CloudFormation. Ví dụ: deny `ec2:CreateSecurityGroup` cho non-admin users.

#### KHÔNG NÊN làm
- **Ignore drift detection** → Vì stack update có thể **overwrite** manual changes, gây mất cấu hình. NÊN: investigate drift, update template nếu cần, hoặc revert manual changes.
- **Run drift detection liên tục (mỗi phút)** → Vì tốn API calls, có thể hit rate limits. NÊN: schedule weekly hoặc monthly, kết hợp với CloudWatch Events.

---

## 15. CloudFormation vs CDK vs Terraform

### So sánh chi tiết

#### Giải thích cốt lõi

| Feature | CloudFormation | CDK | Terraform |
|---------|----------------|-----|-----------|
| **Syntax** | YAML/JSON | TypeScript, Python, Java, Go, .NET | HCL (HashiCorp Language) |
| **State Management** | AWS-managed | AWS-managed (compiles to CF) | State file (local hoặc S3) |
| **Multi-cloud** | ❌ AWS only | ❌ AWS only (chính) | ✅ AWS, Azure, GCP, etc. |
| **Learning Curve** | Medium (YAML, intrinsic functions) | High (programming concepts) | Medium (HCL syntax) |
| **Community** | AWS native | AWS native | Very large, multi-cloud |
| **Cost** | Free (chỉ trả resources) | Free | Free (Terraform Cloud có phí) |
| **Drift Detection** | ✅ Built-in | ✅ Built-in (qua CF) | ✅ Built-in |
| **Rollback** | ✅ Auto rollback | ✅ Auto rollback (qua CF) | ⚠️ Manual hoặc auto |

**Khi nào dùng cái nào:**

| Use Case | Recommended |
|----------|-------------|
| **AWS-only, team thích YAML** | CloudFormation |
| **AWS-only, team là developers** | CDK |
| **Multi-cloud** | Terraform |
| **Compliance cao, cần AWS-managed state** | CloudFormation |
| **Open-source ecosystem** | Terraform |

#### NÊN làm
- **Chọn CloudFormation khi AWS-only + cần AWS-managed state** → Vì zero infrastructure để quản lý state, integrate tốt với AWS services (CodePipeline, Service Catalog).
- **Dùng CDK khi team là developers và code nhiều logic** → Vì CDK cho phép viết loops, conditions, abstractions dễ dàng hơn YAML.

#### KHÔNG NÊN làm
- **Mix CloudFormation và Terraform cho cùng resources** → Vì hai tools sẽ fight nhau, drift detection cả hai cùng report. NÊN: chọn 1 tool cho infrastructure.
- **Dùng CloudFormation cho multi-cloud strategy** → Vì giới hạn AWS only. NÊN: Terraform hoặc Pulumi cho multi-cloud.

---

## 16. Exam Tips cho CloudFormation

### Các điểm trọng tâm

#### Giải thích cốt lõi
Dựa trên exam guide và câu hỏi DVA-C02, CloudFormation tập trung vào:

**1. Intrinsic Functions (Hay xuất hiện nhất):**
- `!Ref` vs `!GetAtt` - khi nào dùng cái nào
- `!Sub` cho string substitution
- `!Join` cho concatenations phức tạp
- `!FindInMap` cho region-specific values
- `!ImportValue` cho cross-stack references

**2. Stack Management:**
- Rollback behavior (Creation, Update, Failure)
- DeletionPolicy (Delete, Retain, Snapshot)
- Stack Policies (protect resources)
- Termination Protection
- Drift Detection

**3. Advanced Features:**
- Custom Resources (Lambda-backed)
- StackSets (multi-account, multi-region)
- Nested Stacks
- Change Sets
- Service Roles

**4. Parameters vs Mappings:**
- Parameters: user input
- Mappings: hardcoded lookup

**5. Outputs:**
- Cross-stack references
- Export names unique

#### NÊN làm
- **Học thuộc syntax YAML shorthand** → Vì exam cho cả JSON và YAML. YAML: `!Ref`, `!Sub`, `!GetAtt`. JSON: `{"Ref": "...", "Fn::Sub": "..."}`.
- **Hiểu rõ DeletionPolicy behaviors** → Vì hay có câu hỏi: "RDS bị xóa khi delete stack - làm sao giữ data?" → Answer: `DeletionPolicy: Snapshot`.
- **Biết khi nào dùng Custom Resources** → Vì câu hỏi: "Cần xóa non-empty S3 bucket" → Answer: Custom Resource backed by Lambda.

#### KHÔNG NÊN làm
- **Học thuộc lòng 700+ resource types** → Vì không khả thi và không cần thiết. NÊN: hiểu structure `service-provider::service-name::data-type-name`, biết cách tra docs.
- **Bỏ qua Drift Detection** → Vì là topic mới trong exam, AWS hay hỏi về "phát hiện resources bị thay đổi ngoài CloudFormation".

---

## Tổng kết Section 8

### Các khái niệm cần nhớ:

| Concept | Key Point |
|---------|-----------|
| **Declarative IaC** | Khai báo desired state, AWS lo chi tiết |
| **Parameters** | User inputs, dynamic values |
| **Mappings** | Hardcoded lookups (Region, AMI) |
| **Outputs** | Cross-stack references via `!ImportValue` |
| **Conditions** | Conditional resource creation |
| **Intrinsic Functions** | `!Ref`, `!GetAtt`, `!Sub`, `!FindInMap`, `!ImportValue` |
| **Rollback** | Auto-rollback on create/update fail |
| **Service Role** | IAM role for CloudFormation to act on user's behalf |
| **DeletionPolicy** | Delete (default), Retain, Snapshot |
| **Stack Policy** | Protect resources from accidental updates |
| **Custom Resources** | Extend CF with Lambda-backed custom logic |
| **StackSets** | Multi-account, multi-region deployment |
| **Drift Detection** | Detect manual changes outside CloudFormation |
