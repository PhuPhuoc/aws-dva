# Section 8 - DVA Practice Questions: AWS CloudFormation

---

## 1. CloudFormation Fundamentals

### Question 1:
A development team needs to deploy and manage AWS infrastructure using code. Which AWS service allows them to define infrastructure declaratively using templates?

```
A. AWS CloudFormation
B. AWS Systems Manager
C. AWS Config
D. AWS CodeDeploy
```

### ✅ Answer: **A. AWS CloudFormation**

### 📖 Explanation (Vietnamese):

**Why A is correct:**
- **CloudFormation** là AWS service cho phép định nghĩa infrastructure **declaratively** (khai báo)
- Sử dụng templates (YAML/JSON) để mô tả desired state
- AWS tự động tạo, update, delete resources theo template

**Cơ chế hoạt động:**
```
Template (YAML/JSON) → Upload lên S3 → CloudFormation → AWS Resources
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B. Systems Manager | Quản lý EC2 instances, không phải infrastructure provisioning |
| C. Config | Tracking và compliance, không tạo resources |
| D. CodeDeploy | Deploy application code, không phải infrastructure |

### 📝 Vocabulary Analysis:
- **declaratively**: khai báo (không cần chỉ định cách làm, chỉ cần mô tả kết quả mong muốn)
- **template**: file định nghĩa infrastructure (YAML hoặc JSON)
- **provisioning**: cung cấp, tạo ra (resources)

### 🔄 Context Variations:
- **If team thích lập trình** → Consider **AWS CDK** (compiles to CloudFormation)
- **If multi-cloud** → Consider **Terraform** (HashiCorp)

---

### Question 2:
A CloudFormation template has multiple sections. Which section is **MANDATORY** in every CloudFormation template?

```
A. Parameters
B. Mappings
C. Resources
D. Outputs
```

### ✅ Answer: **C. Resources**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **`Resources`** section là **BẮT BUỘC** trong mọi CloudFormation template
- Khai báo AWS resources sẽ được tạo (EC2, S3, RDS, etc.)
- Không có Resources → stack không thể tạo được gì

**Các section khác (optional):**

| Section | Required? | Purpose |
|---------|-----------|---------|
| `AWSTemplateFormatVersion` | Optional | Template version |
| `Description` | Optional | Comments về template |
| `Resources` | **MANDATORY** | AWS resources |
| `Parameters` | Optional | Dynamic inputs |
| `Mappings` | Optional | Static variables |
| `Outputs` | Optional | Reference values |
| `Conditions` | Optional | Conditional creation |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Parameters | Optional, dùng cho dynamic inputs |
| B. Mappings | Optional, dùng cho hardcoded lookups |
| D. Outputs | Optional, dùng cho cross-stack references |

### 📝 Vocabulary Analysis:
- **MANDATORY**: bắt buộc, không thể thiếu
- **Resources**: AWS components (EC2, S3, etc.)

---

## 2. CloudFormation Parameters

### Question 3:
A company wants to reuse the same CloudFormation template across dev, staging, and production environments with different instance types. What should they use?

```
A. Mappings
B. Parameters
C. Outputs
D. Conditions
```

### ✅ Answer: **B. Parameters**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Parameters** cho phép user cung cấp inputs khi tạo/update stack
- Mỗi environment có thể truyền giá trị khác nhau
- Không cần edit template cho mỗi environment

**Ví dụ:**
```yaml
Parameters:
  EnvironmentName:
    Type: String
    AllowedValues: [dev, staging, prod]
  InstanceType:
    Type: String
    Default: t3.micro

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Mappings | Hardcoded values, không linh hoạt bằng Parameters |
| C. Outputs | Chỉ để hiển thị/export values, không phải input |
| D. Conditions | Chỉ true/false, không thay đổi được values |

### 📝 Vocabulary Analysis:
- **reuse**: tái sử dụng
- **inputs**: đầu vào, giá trị truyền vào

### 🔄 Context Variations:
- **If values là region-specific** (vd: AMI IDs) → Dùng **Mappings**
- **If values thay đổi thường xuyên** → Dùng **SSM Parameter Store** + dynamic reference

---

### Question 4:
A developer needs to ensure that parameter values are validated against existing AWS resources. Which parameter type should they use?

```
A. String
B. Number
C. AWS::EC2::VPC::Id
D. CommaDelimitedList
```

### ✅ Answer: **C. AWS::EC2::VPC::Id**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **AWS-Specific Parameter** validate giá trị với existing AWS resources
- `AWS::EC2::VPC::Id` chỉ chấp nhận VPC ID **tồn tại trong account/region**
- Nếu user nhập sai → stack fail ngay lúc validation

**Các AWS-Specific Parameter types:**
- `AWS::EC2::VPC::Id` - VPC ID
- `AWS::EC2::Subnet::Id` - Subnet ID
- `AWS::EC2::SecurityGroup::Id` - Security Group ID
- `AWS::EC2::KeyPair::KeyName` - Key Pair name
- `List<AWS::EC2::VPC::Id>` - List of VPC IDs

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. String | Không validate, chấp nhận bất kỳ string |
| B. Number | Chỉ validate là số, không check AWS resources |
| D. CommaDelimitedList | List of strings, không validate AWS |

### 📝 Vocabulary Analysis:
- **validated**: xác thực, kiểm tra tính hợp lệ
- **existing**: hiện có, đang tồn tại

---

## 3. CloudFormation Mappings

### Question 5:
A company deploys the same CloudFormation template to multiple AWS regions. Each region has different AMI IDs. What is the BEST way to handle this?

```
A. Use Parameters for AMI IDs
B. Use Mappings for AMI IDs
C. Use Outputs for AMI IDs
D. Use Conditions for AMI IDs
```

### ✅ Answer: **B. Use Mappings for AMI IDs**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Mappings** rất phù hợp cho **region-specific** values
- AMIs là region-specific (không dùng được AMI us-east-1 ở eu-west-1)
- Mappings cho phép hardcode AMI IDs cho từng region

**Ví dụ:**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0ff8a91504f7b2c5c
    eu-west-1:
      AMI: ami-0bdb828fd58c52435

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, AMI]
```

**Kết hợp với `!Ref AWS::Region`** → tự động chọn đúng AMI cho region.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Parameters | User phải nhập AMI ID mỗi lần, dễ sai |
| C. Outputs | Không thể dùng để khai báo values trong Resources |
| D. Conditions | Chỉ true/false, không thể map values |

### 📝 Vocabulary Analysis:
- **Mappings**: bảng ánh xạ, tra cứu giá trị
- **region-specific**: riêng biệt cho từng region

---

## 4. CloudFormation Outputs

### Question 6:
Network team creates a VPC stack. Application team needs to use the VPC ID in their app stack. What is the BEST way to share the VPC ID between stacks?

```
A. Use Parameters to pass VPC ID
B. Use Outputs with Export and !ImportValue
C. Use SSM Parameter Store
D. Use a manual copy-paste
```

### ✅ Answer: **B. Use Outputs with Export and !ImportValue**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Outputs with Export** cho phép cross-stack references
- App stack dùng `Fn::ImportValue` để lấy VPC ID
- **Single source of truth** - network team quản lý VPC, app team consume

**Ví dụ:**

Network Stack:
```yaml
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${AWS::StackName}-VPC-ID"
```

App Stack:
```yaml
Resources:
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue NetworkStack-VPC-ID
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Parameters | App team phải biết VPC ID trước khi deploy |
| C. SSM Parameter Store | Workaround tốt nhưng tốn thêm setup |
| D. Manual copy-paste | Error-prone, không tự động |

### 📝 Vocabulary Analysis:
- **cross-stack reference**: tham chiếu giữa các stacks
- **single source of truth**: một nguồn dữ liệu duy nhất (chân lý duy nhất)

### 🔄 Context Variations:
- **If values thay đổi runtime** → Dùng **SSM Parameter Store** thay vì Outputs
- **If cần xóa stack gốc** → Phải xóa tất cả dependent stacks trước (vì import)

---

## 5. CloudFormation Intrinsic Functions

### Question 7:
A CloudFormation template needs to reference the Availability Zone of an EC2 instance. Which intrinsic function should be used?

```
A. !Ref
B. !GetAtt
C. !Sub
D. !FindInMap
```

### ✅ Answer: **B. !GetAtt**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- `!GetAtt` lấy **specific attributes** của resource
- Mỗi resource có nhiều attributes (AZ, PrivateIp, PublicDnsName, etc.)
- `!GetAtt MyInstance.AvailabilityZone` trả về AZ của instance

**Cú pháp:**
```yaml
AvailabilityZone: !GetAtt MyInstance.AvailabilityZone
PrivateIp: !GetAtt MyInstance.PrivateIp
```

**`!Ref` vs `!GetAtt`:**

| Function | Returns | Example |
|----------|---------|---------|
| `!Ref MyInstance` | Physical ID | `i-0123456789abcdef0` |
| `!GetAtt MyInstance.AvailabilityZone` | Specific attribute | `us-east-1a` |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. !Ref | Trả về Physical ID, không phải AZ |
| C. !Sub | String substitution, không lấy attributes |
| D. !FindInMap | Lấy values từ Mappings, không phải từ Resources |

### 📝 Vocabulary Analysis:
- **attributes**: thuộc tính (của resource)
- **Physical ID**: ID thực tế của resource trong AWS (vd: EC2 instance ID)

---

### Question 8:
A template needs to create an ARN string combining multiple values. Which intrinsic function is the BEST choice?

```
A. !Ref
B. !Join
C. !Sub
D. !Cidr
```

### ✅ Answer: **C. !Sub**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- `!Sub` (String substitution) cho phép embed values trong string
- Ngắn gọn và dễ đọc hơn `!Join`
- Hỗ trợ cả `!Ref` và các intrinsic functions khác

**Ví dụ:**
```yaml
# !Sub - ngắn gọn
BucketArn: !Sub "arn:aws:s3:::${BucketName}"

# !Join - verbose hơn
BucketArn: !Join ["", ["arn:aws:s3:::", !Ref BucketName]]
```

**Kết quả giống nhau**, nhưng `!Sub` dễ đọc hơn nhiều.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. !Ref | Chỉ reference một value, không concatenate |
| B. !Join | Work nhưng verbose hơn `!Sub` |
| D. !Cidr | CIDR operations, không phải string operations |

### 📝 Vocabulary Analysis:
- **ARN (Amazon Resource Name)**: định danh duy nhất cho AWS resource
- **embed**: nhúng, chèn vào trong

### 🔄 Context Variations:
- **If cần concatenate nhiều values với separator** → `!Join` với delimiter
- **If cần conditional values** → `!Sub` với `!If` bên trong

---

## 6. CloudFormation Rollbacks

### Question 9:
A CloudFormation stack creation fails midway. What happens by default?

```
A. The stack continues creating the remaining resources
B. All created resources are deleted (rolled back)
C. The stack is paused for manual intervention
D. The stack enters UPDATE_ROLLBACK_FAILED state
```

### ✅ Answer: **B. All created resources are deleted (rolled back)**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Default behavior**: khi stack creation fail, **tất cả resources đã tạo bị xóa** (rollback)
- Stack status: `CREATE_FAILED` → `ROLLBACK_COMPLETE`
- AWS đảm bảo stack không ở trạng thái "nửa vời"

**Có 3 loại rollback scenarios:**

| Scenario | Behavior |
|----------|----------|
| **Stack Creation Fails** | Default: rollback (xóa hết) |
| **Stack Update Fails** | Auto rollback về previous working state |
| **Rollback Failure** | Manual fix + `continue-update-rollback` API |

**Disable rollback:** Đặt `DisableRollback: true` → giữ resources để debug.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Continues creating | Không đúng, AWS rollback để tránh inconsistent state |
| C. Paused for manual | Không tự động, AWS rollback ngay |
| D. UPDATE_ROLLBACK_FAILED | Chỉ cho update fail, không phải create fail |

### 📝 Vocabulary Analysis:
- **midway**: giữa chừng
- **rolled back**: hoàn tác, quay lại trạng thái trước
- **consistent state**: trạng thái nhất quán

---

## 7. CloudFormation DeletionPolicy

### Question 10:
A production RDS database is managed by CloudFormation. The team wants to ensure the database is **backed up** before being deleted when the stack is removed. What should be configured?

```
A. DeletionPolicy: Delete
B. DeletionPolicy: Retain
C. DeletionPolicy: Snapshot
D. No DeletionPolicy needed
```

### ✅ Answer: **C. DeletionPolicy: Snapshot**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **`DeletionPolicy: Snapshot`** tạo một **final snapshot** trước khi xóa resource
- Snapshot có thể được restore nếu cần
- Phù hợp cho databases có data quan trọng

**3 loại DeletionPolicy:**

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `Delete` (default) | Xóa resource | Stateless resources (EC2) |
| `Retain` | Giữ lại resource | S3 buckets với data quan trọng |
| `Snapshot` | Tạo snapshot trước khi xóa | RDS, EBS, ElastiCache |

**Resources hỗ trợ Snapshot:**
- RDS DBInstance, DBCluster
- EBS Volume
- ElastiCache Cluster, ReplicationGroup
- Redshift Cluster
- Neptune DBCluster
- DocumentDB DBCluster

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Delete | Mất data vĩnh viễn, không có backup |
| B. Retain | Giữ lại DB, nhưng không tạo snapshot để restore |
| D. Default (Delete) | Mất data, không an toàn cho production |

### 📝 Vocabulary Analysis:
- **snapshot**: ảnh chụp nhanh (backup tại một thời điểm)
- **restored**: khôi phục từ backup

### 🔄 Context Variations:
- **If S3 bucket với data** → `DeletionPolicy: Retain` (vì S3 không tạo snapshot)
- **If EC2 instance stateless** → Default `Delete` (không cần backup)

---

## 8. CloudFormation Stack Policies

### Question 11:
A company wants to **prevent accidental replacement** of a production RDS database during stack updates. What should they configure?

```
A. Termination Protection
B. Stack Policy with Deny on the database
C. DeletionPolicy: Snapshot
D. IAM Policy restricting RDS access
```

### ✅ Answer: **B. Stack Policy with Deny on the database**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Stack Policy** bảo vệ resources khỏi **accidental updates**
- Mặc định, Stack Policy **deny tất cả updates**
- Phải explicit `Allow` cho resources muốn update
- Có thể `Deny` specific resource (như database) để protect

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

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Termination Protection | Chỉ ngăn **delete stack**, không ngăn update resources |
| C. DeletionPolicy: Snapshot | Chỉ áp dụng khi xóa, không ngăn update |
| D. IAM Policy | Kiểm soát user permissions, không phải stack update behavior |

### 📝 Vocabulary Analysis:
- **accidental replacement**: thay thế ngoài ý muốn
- **Stack Policy**: chính sách bảo vệ stack khi update

### 🔄 Context Variations:
- **If muốn update DB sau** → Tạm thời modify Stack Policy (change Deny → Allow), update, sau đó Deny lại
- **If muốn ngăn delete toàn bộ stack** → Dùng **Termination Protection**

---

## 9. CloudFormation Custom Resources

### Question 12:
A CloudFormation stack includes an S3 bucket. The team wants to ensure the bucket is **emptied before deletion** (because CloudFormation cannot delete non-empty buckets). What is the BEST solution?

```
A. DeletionPolicy: Snapshot
B. Custom Resource backed by Lambda
C. Manually empty the bucket before deletion
D. Use S3 Lifecycle Policy
```

### ✅ Answer: **B. Custom Resource backed by Lambda**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **Custom Resource** cho phép run custom logic trong CloudFormation lifecycle
- Lambda function xóa tất cả objects trong S3 bucket
- Trigger tự động trước khi CloudFormation xóa bucket

**Ví dụ Custom Resource:**
```yaml
Resources:
  EmptyBucketFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.handler
      Code:
        ZipFile: |
          import boto3, cfnresponse
          def handler(event, context):
              if event['RequestType'] == 'Delete':
                  s3 = boto3.client('s3')
                  # Delete all objects
                  ...
              cfnresponse.send(event, context, cfnresponse.SUCCESS, {})
  
  EmptyBucket:
    Type: Custom::EmptyBucket
    Properties:
      ServiceToken: !GetAtt EmptyBucketFunction.Arn
      BucketName: !Ref MyBucket
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. DeletionPolicy: Snapshot | S3 không support Snapshot policy |
| C. Manually empty | Error-prone, không tự động, dễ quên |
| D. S3 Lifecycle Policy | Chỉ xóa objects theo thời gian, không tức thì |

### 📝 Vocabulary Analysis:
- **non-empty bucket**: bucket có chứa objects
- **lifecycle**: vòng đời (create, update, delete)
- **backed by Lambda**: sử dụng Lambda function làm backend

---

## 10. CloudFormation StackSets

### Question 13:
A company has 50 AWS accounts in an AWS Organization. They need to deploy CloudTrail to ALL accounts for compliance. What is the BEST approach?

```
A. Manually deploy CloudTrail in each account
B. Use CloudFormation StackSets with AWS Organization
C. Use a Lambda function to deploy to all accounts
D. Use AWS CLI scripts with for loop
```

### ✅ Answer: **B. Use CloudFormation StackSets with AWS Organization**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **StackSets** cho phép tạo/update/delete stacks trên **multiple accounts và regions** với single template
- Với **service-managed permissions** + AWS Organization, AWS tự động tạo IAM roles
- Khi thêm account mới vào Organization → StackSet có thể tự động deploy

**StackSets features:**

| Feature | Description |
|---------|-------------|
| **Multi-account** | Deploy across nhiều accounts |
| **Multi-region** | Deploy across nhiều regions |
| **AWS Organization integration** | Auto-deploy tới all OUs |
| **Service-managed permissions** | AWS quản lý IAM roles |
| **Only Administrator account** | Chỉ admin account tạo StackSets |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Manually deploy | Lặp lại 50 lần, error-prone, không scale |
| C. Lambda function | Phức tạp, tự maintain, reinvent wheel |
| D. AWS CLI scripts | Lặp lại thủ công, không có rollback tracking |

### 📝 Vocabulary Analysis:
- **compliance**: tuân thủ (quy định, tiêu chuẩn)
- **AWS Organization**: tổ chức AWS (quản lý nhiều accounts)
- **scale**: mở rộng quy mô

### 🔄 Context Variations:
- **If không có AWS Organization** → Dùng StackSets với self-managed permissions
- **If cần deploy application-specific** → Dùng CI/CD pipeline, không phải StackSets

---

## 11. CloudFormation Service Role

### Question 14:
A company wants to apply the **principle of least privilege**. Users should be able to deploy CloudFormation stacks without having direct permissions to create AWS resources. What should be configured?

```
A. Grant users AdministratorAccess
B. Use a CloudFormation Service Role with users having iam:PassRole
C. Use IAM Role for each user manually
D. Use AWS Organizations SCP
```

### ✅ Answer: **B. Use a CloudFormation Service Role with users having iam:PassRole**

### 📖 Explanation (Vietnamese):

**Why B is correct:**
- **CloudFormation Service Role** cho phép CloudFormation create resources thay mặt user
- Users chỉ cần quyền `cloudformation:*` và `iam:PassRole`
- **Least privilege**: user không cần quyền trực tiếp với EC2, RDS, etc.

**Architecture:**
```
User (chỉ có cloudformation:* + iam:PassRole)
    ↓
CloudFormation (assume Service Role)
    ↓
Service Role (có quyền tạo EC2, RDS, etc.)
    ↓
AWS Resources
```

**User policy cần:**
```json
{
  "Effect": "Allow",
  "Action": ["cloudformation:*", "iam:PassRole"],
  "Resource": "*"
}
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. AdministratorAccess | Vi phạm least privilege, quá nhiều quyền |
| C. IAM Role cho mỗi user | Phức tạp, không tận dụng được CF's service role |
| D. AWS Organizations SCP | Chỉ là permission boundaries, không phải solution cho CF |

### 📝 Vocabulary Analysis:
- **principle of least privilege**: nguyên tắc đặc quyền tối thiểu
- **on behalf of**: thay mặt
- **iam:PassRole**: quyền pass IAM role cho AWS services

---

## 12. CloudFormation Termination Protection

### Question 15:
A team accidentally deleted a production CloudFormation stack, causing significant downtime. What should be enabled to **prevent accidental stack deletion** in the future?

```
A. IAM Policy
B. Stack Policy
C. Termination Protection
D. DeletionPolicy: Retain
```

### ✅ Answer: **C. Termination Protection**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Termination Protection** ngăn chặn **accidental deletion** của CloudFormation stacks
- Khi enabled, stack không thể bị xóa qua Console, CLI, hoặc API
- Phải **disable Termination Protection trước** mới có thể xóa

**Enable qua CLI:**
```bash
aws cloudformation update-termination-protection \
  --stack-name production-stack \
  --enable-termination-protection
```

**Disable để xóa:**
```bash
# 1. Disable termination protection
aws cloudformation update-termination-protection \
  --stack-name production-stack \
  --no-enable-termination-protection

# 2. Sau đó mới delete được
aws cloudformation delete-stack --stack-name production-stack
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. IAM Policy | Kiểm soát user permissions, không ngăn delete stack |
| B. Stack Policy | Bảo vệ resources trong update, không ngăn delete stack |
| D. DeletionPolicy: Retain | Giữ lại resources khi xóa, không ngăn xóa stack |

### 📝 Vocabulary Analysis:
- **accidental deletion**: xóa ngoài ý muốn
- **downtime**: thời gian ngừng hoạt động
- **Termination Protection**: bảo vệ stack khỏi bị xóa

### 🔄 Context Variations:
- **If cần update resources trong stack** → Termination Protection **không ảnh hưởng** (chỉ ngăn delete)
- **If muốn protect resources trong update** → Dùng **Stack Policy**

---

## 13. CloudFormation Drift Detection

### Question 16:
A security team suspects that someone manually modified resources through the AWS Console. They want to detect any differences between the CloudFormation template and actual AWS resources. What should they use?

```
A. CloudTrail
B. CloudWatch Alarms
C. CloudFormation Drift Detection
D. AWS Config Rules
```

### ✅ Answer: **C. CloudFormation Drift Detection**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- **Drift Detection** phát hiện resources bị modified **ngoài CloudFormation**
- So sánh actual state (AWS) với expected state (template)
- Hiển thị rõ resource nào bị `MODIFIED`, `DELETED`, hoặc `IN_SYNC`

**Cách sử dụng:**

```bash
# Detect drift cho 1 stack
aws cloudformation detect-stack-drift --stack-name my-stack

# Xem chi tiết drift
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

**Drift Status:**

| Status | Meaning |
|--------|---------|
| `IN_SYNC` | Khớp với template |
| `MODIFIED` | Bị thay đổi |
| `DELETED` | Bị xóa |
| `NOT_CHECKED` | Chưa kiểm tra |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. CloudTrail | Track API calls, không so sánh với template |
| B. CloudWatch Alarms | Monitoring metrics, không liên quan đến drift |
| D. AWS Config Rules | Compliance checks, không so sánh với CF template |

### 📝 Vocabulary Analysis:
- **drift**: sai lệch, chệch hướng
- **manually modified**: thay đổi thủ công
- **detect**: phát hiện

### 🔄 Context Variations:
- **If muốn auto-detect mỗi tuần** → Setup CloudWatch Event rule + Lambda
- **If muốn prevent drift** → Restrict IAM permissions, chỉ cho phép thay đổi qua CF

---

## Tổng hợp câu hỏi

| # | Topic | Key Point |
|---|-------|-----------|
| 1 | CloudFormation Basics | Declarative IaC service |
| 2 | Template Structure | Resources is MANDATORY |
| 3 | Parameters | User inputs, dynamic values |
| 4 | AWS-Specific Parameters | Validate against existing AWS resources |
| 5 | Mappings | Region-specific hardcoded values |
| 6 | Outputs | Cross-stack references via !ImportValue |
| 7 | !GetAtt | Get specific resource attributes |
| 8 | !Sub | String substitution (shorter than !Join) |
| 9 | Rollback | Default: delete all on create fail |
| 10 | DeletionPolicy: Snapshot | Backup before delete (RDS, EBS) |
| 11 | Stack Policy | Protect resources from accidental updates |
| 12 | Custom Resources | Lambda-backed for custom logic |
| 13 | StackSets | Multi-account, multi-region deployment |
| 14 | Service Role | Least privilege for users |
| 15 | Termination Protection | Prevent accidental stack deletion |
| 16 | Drift Detection | Detect manual changes outside CF |

---

## Exam Tips cho Section 8

### Các chủ đề hay xuất hiện:
1. **Intrinsic Functions** (`!Ref`, `!GetAtt`, `!Sub`, `!FindInMap`, `!ImportValue`)
2. **Parameters vs Mappings vs Conditions** (khi nào dùng cái nào)
3. **DeletionPolicy** (Delete, Retain, Snapshot)
4. **Rollback behavior** (creation, update, failure)
5. **Cross-stack references** (Outputs + Export + ImportValue)
6. **StackSets** (multi-account, multi-region)
7. **Custom Resources** (use cases, Lambda-backed)
8. **Service Role** (least privilege)
9. **Drift Detection** (phát hiện manual changes)

### Câu hỏi "trick" thường gặp:
- "Cần share values giữa stacks" → Outputs + Export, **KHÔNG** phải SSM Parameter Store
- "Cần xóa non-empty S3 bucket" → Custom Resource với Lambda, **KHÔNG** phải DeletionPolicy
- "Cần backup RDS trước khi xóa" → DeletionPolicy: Snapshot, **KHÔNG** phải Retain
- "Cần ngăn update resources" → Stack Policy, **KHÔNG** phải Termination Protection
- "Cần ngăn xóa stack" → Termination Protection, **KHÔNG** phải Stack Policy
