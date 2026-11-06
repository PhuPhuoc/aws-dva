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
- **CloudFormation** là dịch vụ AWS cho phép định nghĩa hạ tầng theo cách khai báo.
- Dùng template để mô tả trạng thái hạ tầng mong muốn.
- AWS tự động tạo, cập nhật hoặc xóa các resource theo template.

**Cơ chế hoạt động:**
```
Template (YAML/JSON) → Tải lên S3 → CloudFormation → Tài nguyên AWS
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| B. Systems Manager | Quản lý EC2 instances, không chuyên cung cấp hạ tầng bằng template |
| C. Config | Theo dõi cấu hình và việc tuân thủ, không tạo resources |
| D. CodeDeploy | Triển khai mã ứng dụng, không cấp phát hạ tầng |

### 📝 Vocabulary Analysis:
- **declaratively**: khai báo (không cần chỉ định cách làm, chỉ cần mô tả kết quả mong muốn)
- **template**: tệp định nghĩa hạ tầng (YAML hoặc JSON)
- **provisioning**: cấp phát hoặc tạo resource

### 🔄 Context Variations:
- **Nếu nhóm thích viết mã** → Cân nhắc **AWS CDK** (biên dịch thành CloudFormation).
- **Nếu cần hỗ trợ nhiều cloud** → Cân nhắc **Terraform** (HashiCorp).

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
- Phần **`Resources`** là **BẮT BUỘC** trong mọi CloudFormation template.
- Phần này khai báo các AWS resources cần tạo (EC2, S3, RDS, v.v.).
- Không có `Resources` thì stack không khai báo resource để tạo.

**Các section khác (optional):**

| Section | Bắt buộc? | Mục đích |
|---------|-----------|----------|
| `AWSTemplateFormatVersion` | Không | Phiên bản template |
| `Description` | Không | Ghi chú về template |
| `Resources` | **BẮT BUỘC** | Khai báo AWS resources |
| `Parameters` | Không | Giá trị đầu vào động |
| `Mappings` | Không | Bảng tra cứu giá trị tĩnh |
| `Outputs` | Không | Công bố giá trị đầu ra |
| `Conditions` | Không | Tạo/cấu hình có điều kiện |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Parameters | Không bắt buộc; dùng cho giá trị đầu vào động |
| B. Mappings | Không bắt buộc; dùng làm bảng tra cứu tĩnh |
| D. Outputs | Không bắt buộc; dùng công bố giá trị đầu ra |

### 📝 Vocabulary Analysis:
- **MANDATORY**: bắt buộc, không thể thiếu
- **Resources**: các thành phần AWS được khai báo trong template (EC2, S3, v.v.)

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
- **Parameters** cho phép người dùng truyền giá trị đầu vào khi tạo hoặc cập nhật stack.
- Mỗi môi trường có thể nhận giá trị khác nhau.
- Không cần chỉnh sửa template để đổi các giá trị này.

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
| A. Mappings | Chứa giá trị ghi sẵn, kém linh hoạt hơn Parameters cho đầu vào theo lần triển khai |
| C. Outputs | Công bố/xuất giá trị; không phải đầu vào |
| D. Conditions | Chọn đúng/sai; không thay thế giá trị đầu vào |

### 📝 Vocabulary Analysis:
- **reuse**: tái sử dụng
- **inputs**: đầu vào, giá trị truyền vào

### 🔄 Context Variations:
- **Nếu giá trị phụ thuộc Region** (ví dụ AMI ID) → Dùng **Mappings**.
- **Nếu cần cấu hình dùng chung, có thể thay đổi độc lập** → Cân nhắc **SSM Parameter Store** và dynamic reference.

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
- **AWS-Specific Parameter** xác thực giá trị với resource AWS hiện có.
- `AWS::EC2::VPC::Id` chỉ chấp nhận VPC ID tồn tại trong account/Region.
- Nếu nhập sai, stack thất bại ngay ở bước xác thực parameter.

**Các AWS-Specific Parameter types:**
- `AWS::EC2::VPC::Id` - VPC ID
- `AWS::EC2::Subnet::Id` - Subnet ID
- `AWS::EC2::SecurityGroup::Id` - Security Group ID
- `AWS::EC2::KeyPair::KeyName` - Key Pair name
- `List<AWS::EC2::VPC::Id>` - List of VPC IDs

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. String | Không xác thực resource; có thể nhận bất kỳ chuỗi nào |
| B. Number | Chỉ xác thực giá trị là số, không kiểm tra AWS resources |
| D. CommaDelimitedList | Danh sách chuỗi, không xác thực AWS resources |

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
- **Mappings** phù hợp với các giá trị phụ thuộc Region.
- AMI thuộc từng Region; không dùng trực tiếp AMI của `us-east-1` trong `eu-west-1`.
- Có thể ghi sẵn AMI ID cho từng Region trong Mappings.

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
| A. Parameters | Người dùng phải nhập AMI ID mỗi lần triển khai, dễ nhập sai |
| C. Outputs | Không dùng để khai báo giá trị trong phần Resources |
| D. Conditions | Chỉ biểu diễn điều kiện đúng/sai, không tra cứu giá trị |

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
- **Outputs với Export** cho phép tham chiếu trực tiếp giữa các stacks.
- App stack dùng `Fn::ImportValue` để lấy VPC ID.
- Nhóm mạng quản lý VPC; nhóm ứng dụng dùng lại cùng một giá trị thay vì sao chép thủ công.

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
| A. Parameters | Nhóm ứng dụng phải biết và tự truyền VPC ID trước khi triển khai |
| C. SSM Parameter Store | Có thể dùng khi cần liên kết lỏng, nhưng không phải tham chiếu CloudFormation trực tiếp |
| D. Sao chép thủ công | Dễ nhập sai và không tự đồng bộ khi giá trị đổi |

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
- `!GetAtt` lấy các thuộc tính cụ thể của resource.
- Mỗi resource có các thuộc tính khác nhau (AZ, PrivateIp, PublicDnsName, v.v.).
- `!GetAtt MyInstance.AvailabilityZone` trả về AZ của instance.

**Cú pháp:**
```yaml
AvailabilityZone: !GetAtt MyInstance.AvailabilityZone
PrivateIp: !GetAtt MyInstance.PrivateIp
```

**`!Ref` vs `!GetAtt`:**

| Hàm | Giá trị trả về | Ví dụ |
|----------|---------|---------|
| `!Ref MyInstance` | Physical ID | `i-0123456789abcdef0` |
| `!GetAtt MyInstance.AvailabilityZone` | Thuộc tính cụ thể | `us-east-1a` |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. !Ref | Trả về Physical ID, không phải AZ |
| C. !Sub | Thay thế chuỗi, không lấy thuộc tính resource |
| D. !FindInMap | Lấy giá trị từ Mappings, không lấy thuộc tính resource |

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
- `!Sub` thay thế các thành phần `${...}` bằng giá trị của parameter, resource hoặc thuộc tính tương ứng.
- Ngắn gọn và dễ đọc hơn `!Join` khi ghép nhiều giá trị thành chuỗi.
- Khi cần giá trị thay thế tùy chỉnh, có thể khai báo map trong `Fn::Sub`.

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
| A. !Ref | Chỉ tham chiếu một giá trị, không ghép chuỗi |
| B. !Join | Có thể ghép chuỗi nhưng dài dòng hơn `!Sub` |
| D. !Cidr | Thực hiện phép toán CIDR, không xử lý chuỗi |

### 📝 Vocabulary Analysis:
- **ARN (Amazon Resource Name)**: định danh duy nhất cho AWS resource
- **embed**: nhúng, chèn vào trong

### 🔄 Context Variations:
- **Nếu cần nối nhiều giá trị bằng dấu phân cách** → Dùng `!Join` với delimiter.
- **Nếu cần giá trị có điều kiện** → Dùng `!If`; có thể đưa kết quả vào chuỗi mà `!Sub` tạo.

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
- **Hành vi mặc định:** khi tạo stack thất bại, các resources đã tạo sẽ bị xóa (rollback).
- Trạng thái stack: `CREATE_FAILED` → `ROLLBACK_COMPLETE`.
- Nếu rollback hoàn tất, CloudFormation xóa các resources đã tạo; nếu rollback thất bại, stack có thể còn ở trạng thái lỗi cần xử lý.

**Ba tình huống rollback:**

| Tình huống | Hành vi |
|----------|----------|
| **Tạo stack thất bại** | Mặc định rollback (xóa resources đã tạo) |
| **Cập nhật stack thất bại** | Tự rollback về trạng thái hoạt động tốt trước đó |
| **Update rollback thất bại** | Sửa thủ công rồi gọi API `continue-update-rollback` |

**Tắt rollback:** Đặt `DisableRollback: true` để giữ lại resources nhằm gỡ lỗi.

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Tiếp tục tạo | Không đúng; AWS rollback theo hành vi mặc định |
| C. Tạm dừng chờ thao tác | Không đúng; thao tác rollback diễn ra tự động |
| D. `UPDATE_ROLLBACK_FAILED` | Trạng thái này áp dụng cho trường hợp cập nhật thất bại, không phải tạo stack thất bại |

### 📝 Vocabulary Analysis:
- **midway**: giữa chừng
- **rolled back**: hoàn tác, quay lại trạng thái trước
- **consistent state**: trạng thái nhất quán

---

## 7. CloudFormation DeletionPolicy

### Question 10:
An existing CloudFormation template explicitly sets `DeletionPolicy: Delete` on a standalone `AWS::RDS::DBInstance` (no `DBClusterIdentifier`). The team wants CloudFormation to create a snapshot before deleting the database when the stack is removed. What should they configure?

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
| `Delete` (default for most resources) | Xóa resource | Hầu hết resources không cần giữ lại |
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
| D. No DeletionPolicy needed | Template đang đặt tường minh `Delete`; không có thay đổi nên không tạo snapshot. RDS chỉ mặc định `Snapshot` khi không có policy và resource thuộc trường hợp được AWS nêu. |

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
- Nếu **không gắn Stack Policy**, CloudFormation mặc định cho phép cập nhật tất cả resources.
- Khi áp dụng policy, các hành động không được `Allow` tường minh bị từ chối; `Deny` trên Database chặn các lần update/replace của resource đó.
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
- **Custom Resource** cho phép chạy logic tùy chỉnh trong vòng đời CloudFormation.
- Lambda function có thể xóa các objects trong S3 bucket.
- Custom Resource được gọi trước khi CloudFormation xóa bucket nếu có quan hệ phụ thuộc phù hợp.

**Ví dụ tham chiếu Custom Resource** (giả sử `EmptyBucketFunction` và `MyBucket` đã được khai báo trong stack):
```yaml
Resources:
  EmptyBucket:
    Type: Custom::EmptyBucket
    Properties:
      ServiceToken: !GetAtt EmptyBucketFunction.Arn
      BucketName: !Ref MyBucket
```
Lambda được gọi qua `ServiceToken` cần liệt kê mọi trang kết quả; nếu bucket bật hoặc tạm ngưng versioning, cần xóa mọi object version và delete marker trước khi trả `SUCCESS` (xem ví dụ trong `explain_more.md`).

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. DeletionPolicy: Snapshot | S3 không hỗ trợ Snapshot policy |
| C. Tự làm trống bucket | Dễ quên, dễ sai và không được tự động hóa trong stack |
| D. S3 Lifecycle Policy | Xóa objects theo quy tắc/vòng đời, không làm trống bucket ngay trong quy trình xóa stack |

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
- **StackSets** cho phép triển khai, cập nhật hoặc xóa stacks trên nhiều AWS accounts và Regions bằng cùng một template.
- Với **service-managed permissions** và AWS Organization, AWS tự thiết lập các IAM roles cần thiết giữa các tài khoản.
- Khi thêm tài khoản mới vào Organization, StackSet có thể tự động triển khai đến đó nếu bật tính năng tự động triển khai.

**Các tính năng của StackSets:**

| Tính năng | Mô tả |
|---------|-------------|
| **Nhiều tài khoản** | Triển khai trên nhiều AWS accounts |
| **Nhiều Region** | Triển khai trên nhiều Regions |
| **Tích hợp AWS Organization** | Tự động triển khai đến các OU được chọn |
| **Service-managed permissions** | AWS quản lý IAM roles |
| **Quyền quản trị** | Administrator account hoặc Delegated Administrator có thể tạo StackSets |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. Triển khai thủ công | Phải lặp lại nhiều lần, dễ sai và khó mở rộng |
| C. Lambda function | Phức tạp, phải tự vận hành và trùng lặp chức năng StackSets |
| D. AWS CLI scripts | Phải tự lặp thao tác; khó theo dõi trạng thái/rollback tập trung |

### 📝 Vocabulary Analysis:
- **compliance**: tuân thủ (quy định, tiêu chuẩn)
- **AWS Organization**: tổ chức AWS (quản lý nhiều accounts)
- **scale**: mở rộng quy mô

### 🔄 Context Variations:
- **Nếu không dùng AWS Organization** → Dùng StackSets với self-managed permissions.
- **Nếu triển khai ứng dụng riêng cho từng môi trường** → Cân nhắc CI/CD pipeline.

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
- **CloudFormation Service Role** cho phép CloudFormation tạo resources thay mặt người dùng.
- Người dùng cần quyền CloudFormation phù hợp với stack được phép quản lý và `iam:PassRole` trên đúng Service Role.
- **Đặc quyền tối thiểu**: người dùng không cần quyền trực tiếp để tạo EC2, RDS, v.v.

**Architecture:**
```
User (CloudFormation actions scoped to allowed stacks + iam:PassRole on this role)
    ↓
CloudFormation (assume Service Role)
    ↓
Service Role (có quyền tạo EC2, RDS, etc.)
    ↓
AWS Resources
```

**Policy người dùng cần:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:CreateStack",
        "cloudformation:UpdateStack",
        "cloudformation:DescribeStacks",
        "cloudformation:DescribeStackEvents",
        "cloudformation:ListStackResources"
      ],
      "Resource": "arn:aws:cloudformation:us-east-1:123456789012:stack/production-*/*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::123456789012:role/CloudFormationServiceRole",
      "Condition": {
        "StringEquals": {
          "iam:PassedToService": "cloudformation.amazonaws.com"
        }
      }
    }
  ]
}
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. AdministratorAccess | Vi phạm least privilege, quá nhiều quyền |
| C. IAM Role cho mỗi user | Phức tạp, không tận dụng được CF's service role |
| D. AWS Organizations SCP | SCP giới hạn quyền tối đa trong Organization; không thay thế CloudFormation Service Role. |

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
- **Termination Protection** ngăn xóa nhầm CloudFormation stack.
- Khi bật, không thể xóa stack qua Console, CLI hoặc API.
- Phải **tắt Termination Protection trước** khi xóa stack.

**Bật qua CLI:**
```bash
aws cloudformation update-termination-protection \
  --stack-name production-stack \
  --enable-termination-protection
```

**Tắt bảo vệ trước khi xóa:**
```bash
# 1. Tắt Termination Protection
aws cloudformation update-termination-protection \
  --stack-name production-stack \
  --no-enable-termination-protection

# 2. Sau đó mới xóa stack
aws cloudformation delete-stack --stack-name production-stack
```

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. IAM Policy | Kiểm soát quyền của người dùng, nhưng không tự ngăn xóa stack nếu policy vẫn cho phép |
| B. Stack Policy | Bảo vệ resources khi cập nhật, không ngăn xóa toàn bộ stack |
| D. DeletionPolicy: Retain | Giữ resource theo chính sách khi xóa stack, nhưng không ngăn thao tác xóa stack |

### 📝 Vocabulary Analysis:
- **accidental deletion**: xóa ngoài ý muốn
- **downtime**: thời gian ngừng hoạt động
- **Termination Protection**: bảo vệ stack khỏi bị xóa

### 🔄 Context Variations:
- **Nếu cần cập nhật resources trong stack** → Termination Protection không ảnh hưởng đến thao tác cập nhật.
- **Nếu muốn bảo vệ resources trong lúc cập nhật** → Dùng **Stack Policy**.

---

## 13. CloudFormation Drift Detection

### Question 16:
A security team suspects that someone manually modified a CloudFormation stack. They want to compare the actual values of **supported resources and properties** with the template and identify which resources were evaluated. What should they use?

```
A. CloudTrail
B. CloudWatch Alarms
C. CloudFormation Drift Detection
D. AWS Config Rules
```

### ✅ Answer: **C. CloudFormation Drift Detection**

### 📖 Explanation (Vietnamese):

**Why C is correct:**
- Drift Detection so sánh cấu hình thực tế với template đối với những resource/property được hỗ trợ.
- Kết quả có thể gồm `MODIFIED`, `DELETED`, `IN_SYNC` hoặc `NOT_CHECKED`; `NOT_CHECKED` nghĩa là chưa đánh giá, không khẳng định resource không bị drift.

**Cách sử dụng:**

```bash
# Phát hiện drift cho một stack
aws cloudformation detect-stack-drift --stack-name my-stack

# Xem chi tiết drift
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

**Drift Status:**

| Trạng thái | Ý nghĩa |
|--------|---------|
| `IN_SYNC` | Khớp với template |
| `MODIFIED` | Bị thay đổi |
| `DELETED` | Bị xóa |
| `NOT_CHECKED` | Chưa kiểm tra |

**Why other options were eliminated:**

| Option | Reason for Elimination |
|--------|------------------------|
| A. CloudTrail | Ghi nhận API calls, không so sánh cấu hình với template |
| B. CloudWatch Alarms | Giám sát metrics, không phát hiện drift trong stack |
| D. AWS Config Rules | Kiểm tra compliance, không so sánh trực tiếp với CloudFormation template |

### 📝 Vocabulary Analysis:
- **drift**: sai lệch, chệch hướng
- **manually modified**: thay đổi thủ công
- **detect**: phát hiện

### 🔄 Context Variations:
- **Nếu muốn tự động kiểm tra hằng tuần** → Dùng EventBridge rule và Lambda để gọi drift detection.
- **Nếu muốn hạn chế drift** → Giới hạn IAM permissions và cho phép thay đổi qua CloudFormation.

---

## Tổng hợp câu hỏi

| # | Chủ đề | Điểm chính |
|---|-------|-----------|
| 1 | Nền tảng CloudFormation | Dịch vụ IaC khai báo |
| 2 | Cấu trúc template | `Resources` là bắt buộc |
| 3 | Parameters | Đầu vào động do người dùng cung cấp |
| 4 | AWS-Specific Parameters | Xác thực giá trị với resources AWS hiện có |
| 5 | Mappings | Bảng giá trị cố định theo Region |
| 6 | Outputs | Tham chiếu giữa stacks qua `!ImportValue` |
| 7 | `!GetAtt` | Lấy thuộc tính cụ thể của resource |
| 8 | `!Sub` | Thay thế chuỗi, thường gọn hơn `!Join` |
| 9 | Rollback | Mặc định xóa resources đã tạo nếu tạo stack thất bại |
| 10 | `DeletionPolicy: Snapshot` | Tạo snapshot trước khi xóa resource được hỗ trợ |
| 11 | Stack Policy | Bảo vệ resources khỏi cập nhật ngoài ý muốn |
| 12 | Custom Resources | Chạy logic tùy chỉnh, thường được hỗ trợ bởi Lambda |
| 13 | StackSets | Triển khai trên nhiều accounts và Regions |
| 14 | Service Role | Cho phép CloudFormation thao tác thay người dùng theo quyền được cấp |
| 15 | Termination Protection | Ngăn xóa nhầm stack |
| 16 | Drift Detection | Phát hiện thay đổi ngoài CloudFormation |

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
- "Cần chia sẻ giá trị trực tiếp giữa các stacks bằng tham chiếu CloudFormation" → Dùng Outputs + Export; **SSM Parameter Store** cũng phù hợp khi cần liên kết lỏng hoặc giá trị thay đổi độc lập.
- "Cần xóa non-empty S3 bucket" → Custom Resource với Lambda, **KHÔNG** phải DeletionPolicy
- "Cần backup RDS trước khi xóa" → DeletionPolicy: Snapshot, **KHÔNG** phải Retain
- "Cần ngăn update resources" → Stack Policy, **KHÔNG** phải Termination Protection
- "Cần ngăn xóa stack" → Termination Protection, **KHÔNG** phải Stack Policy
