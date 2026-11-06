# Phần 8 - Kiến thức mở rộng: AWS CloudFormation

---

## 1. CloudFormation - Cơ chế hoạt động chi tiết

### Vòng đời template CloudFormation

#### Giải thích cốt lõi
CloudFormation hoạt động theo cơ chế **declarative** (khai báo), không phải **imperative** (mệnh lệnh). Khi tải template lên, CloudFormation:

1. **Phân tích template** → So sánh trạng thái mong muốn trong template với trạng thái hiện tại trên AWS.
2. **Tính toán các thay đổi cần thiết** → Ví dụ: tạo EC2 mới, cập nhật rule của security group, giữ nguyên RDS.
3. **Thực thi thay đổi theo đúng thứ tự**:
   - Tạo VPC → Subnet → Security Group → EC2 instance → Load Balancer.
   - Tự động phát hiện các quan hệ phụ thuộc (qua `!Ref` và `!GetAtt`).
4. **Trạng thái stack:**
   - `CREATE_IN_PROGRESS` → `CREATE_COMPLETE` (hoặc `CREATE_FAILED` → `ROLLBACK_COMPLETE`).
   - `UPDATE_IN_PROGRESS` → `UPDATE_COMPLETE` (hoặc `UPDATE_FAILED` → `UPDATE_ROLLBACK_COMPLETE`).
   - `DELETE_IN_PROGRESS` → `DELETE_COMPLETE` (hoặc `DELETE_FAILED`).

**So sánh với các công cụ khác:**

| Công cụ | Cách tiếp cận | Quản lý trạng thái | Phù hợp nhất |
|---------|----------|------------------|----------|
| **CloudFormation** | Declarative (khai báo) | AWS tự quản lý trạng thái | Chỉ dùng AWS, hệ thống đã ổn định |
| **Terraform** | Declarative (khai báo) | Tệp trạng thái riêng | Nhiều nhà cung cấp cloud |
| **CDK** | Imperative (mệnh lệnh, biên dịch thành CloudFormation) | AWS tự quản lý trạng thái | Đội ngũ phát triển thích viết mã |
| **Pulumi** | Imperative (dùng ngôn ngữ lập trình phổ biến) | Tệp trạng thái riêng | Nhiều cloud và viết mã |
| **Ansible** | Imperative (mệnh lệnh) | Không theo dõi trạng thái | Quản lý cấu hình |

#### NÊN làm (DO)
- **Bắt đầu với CloudFormation khi hạ tầng chỉ dùng AWS** → Vì đây là công cụ gốc, không tính phí dịch vụ (chỉ trả tiền cho resource), tích hợp tốt với dịch vụ AWS và có **drift detection**. So với Terraform, không cần tự quản lý tệp trạng thái nhưng sẽ phải làm quen với công cụ riêng.
- **Dùng YAML thay vì JSON** → YAML hỗ trợ chú thích, chuỗi nhiều dòng và ít dấu ngoặc hơn. JSON dài dòng, không có chú thích và dễ sai cú pháp.
- **Gắn thẻ mọi resource bằng tên stack** → AWS tự gắn thẻ resource với `aws:cloudformation:stack-name`, giúp theo dõi chi phí, dọn dẹp và gỡ lỗi.

#### KHÔNG NÊN làm (DON'T)
- **Viết một template khổng lồ chứa toàn bộ hạ tầng** → Khó đọc, khó tái sử dụng và khó xác định resource lỗi; tạo nested stacks để chia trách nhiệm và quản lý từng thành phần. Nested stack vẫn có thể làm stack cha rollback khi thất bại, nên đây không phải cơ chế cô lập rollback.
- **Ghi cố định giá trị trong Resources** → Mỗi lần thay đổi sẽ phải tải template lên lại. NÊN: dùng **Parameters** cho giá trị có thể đổi (InstanceType, CIDR), **Mappings** cho giá trị theo Region/Environment.

---

## 2. CloudFormation - Parameters

### Thực hành tốt với Parameters

#### Giải thích cốt lõi
Parameters giúp tái sử dụng template cho nhiều môi trường và đội nhóm. Có hai loại Parameters chính:

**AWS-Specific Parameters** — AWS xác thực giá trị với các giá trị đang tồn tại:
- `AWS::EC2::VPC::Id` — chỉ chấp nhận VPC ID có trong account.
- `AWS::EC2::Subnet::Id` — chỉ chấp nhận Subnet ID.
- `AWS::EC2::SecurityGroup::Id`
- `List<AWS::EC2::VPC::Id>` — danh sách VPC ID.

**SSM Parameter Types** — lấy giá trị từ SSM Parameter Store:
- `AWS::SSM::Parameter::Value<String>`
- `AWS::SSM::Parameter::Value<AWS::EC2::VPC::Id>`
- Cho phép quản lý cấu hình tập trung.

**So sánh các cách làm:**

| Cách làm | Trường hợp sử dụng | Bảo mật |
|----------|----------|----------|
| Giá trị ghi cố định | Một môi trường đơn lẻ, không cần thay đổi | ✅ Không áp dụng |
| Parameters | Nhiều môi trường, cấu hình lúc chạy | ⚠️ Cần thận trọng với NoEcho |
| Mappings | Giá trị theo Region/AMI | ✅ Ghi sẵn, an toàn |
| SSM Parameter Store | Cấu hình dùng chung giữa các stack | `SecureString` được mã hóa bằng KMS; `String` và `StringList` không phải loại lưu bí mật |
| Secrets Manager | Dữ liệu nhạy cảm (mật khẩu database) | ✅ Tự động xoay vòng, có kiểm tra |

#### NÊN làm (DO)
- **Dùng AWS-Specific Parameters thay vì String cho ID** → AWS xác thực ID có tồn tại hay không. Nếu người dùng nhập sai VPC ID, stack thất bại ở bước xác thực thay vì lúc chạy.
- **Dùng `AllowedValues` để giới hạn tùy chọn** → Ngăn lỗi chính tả và liệt kê rõ các lựa chọn. Ví dụ: `AllowedValues: [t3.micro, t3.small, t3.medium]`.
- **Dùng `NoEcho: true` cho giá trị nhạy cảm** → Mặc định, giá trị parameter có thể hiển thị trong CloudFormation Console/events. Với secret, NÊN dùng **Secrets Manager** hoặc **SSM SecureString** thay vì parameter thông thường.

#### KHÔNG NÊN làm (DON'T)
- **Đặt giá trị mặc định cho mọi parameter** → Người dùng có thể bỏ qua bước nhập và dùng giá trị mặc định không phù hợp. NÊN: chỉ đặt default cho parameter tùy chọn; không đặt default cho giá trị quan trọng (loại instance production, CIDR).
- **Truyền mật khẩu database qua Parameters** → Mật khẩu có thể xuất hiện trong CloudFormation log/events. NÊN: dùng `Secrets Manager` với dynamic reference: `{{resolve:secretsmanager:MySecret:SecretString:password}}`.

---

## 3. CloudFormation Mappings

### Chi tiết về Mappings

#### Giải thích cốt lõi
Mappings là các **bảng tra cứu có giá trị ghi sẵn** trong template. Khác với Parameters (do người dùng nhập), Mappings chứa các giá trị **do nhà phát triển định nghĩa**, thường dùng cho:

1. **Giá trị theo Region** (AMI ID cho từng Region).
2. **Cấu hình theo môi trường** (kích cỡ cho dev/staging/prod).
3. **Giá trị theo account** (ARN, account ID).
4. **Cấu hình có điều kiện** dựa trên biến có thể xác định lúc triển khai.

**Ví dụ thực tế - Mapping AMI:**

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

#### NÊN làm (DO)
- **Dùng Mappings cho AMI ID theo Region khi cần kiểm soát phiên bản** → AMI ID là giá trị tĩnh; khi chuẩn hóa AMI mới, cần cập nhật mapping. So với public SSM parameter, mapping cho phép ghim phiên bản nhưng đòi hỏi quy trình bảo trì.
- **Kết hợp Mappings với `!Ref AWS::Region`** → Tự động chọn đúng cấu hình cho Region đang triển khai, không cần người dùng nhập.

#### KHÔNG NÊN làm (DON'T)
- **Dùng Mappings cho giá trị người dùng cần thay đổi thường xuyên** → Mỗi lần thay đổi phải chỉnh sửa và tải lại template. NÊN: dùng **Parameters** cho giá trị riêng theo từng người dùng, dùng **SSM Parameter Store** cho cấu hình dùng chung giữa các stack.
- **Dùng Mappings cho tập dữ liệu quá lớn (>100 entries)** → Template sẽ khó đọc. NÊN: chuyển dữ liệu lên S3 hoặc DynamoDB, dùng Custom Resource để lấy dữ liệu.

---

## 4. CloudFormation - Outputs và tham chiếu giữa các stack

### Mẫu tham chiếu giữa các stack

#### Giải thích cốt lõi
**Outputs** cho phép stack công khai giá trị để các stack khác sử dụng. Có hai cách:

1. **Outputs không có Export** — chỉ để xem qua CLI/Console.
2. **Outputs có Export** — dùng `Fn::ImportValue` để tham chiếu giữa các stack.
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
- **Tên Export phải DUY NHẤT trong account/Region.**
- **Không thể xóa stack xuất hoặc thay đổi/xóa giá trị Export** khi stack khác còn import giá trị đó.
- Để xóa stack có Export, trước đó phải xóa mọi stack phụ thuộc.

**So sánh với các cách khác:**

| Mẫu | Ưu điểm | Nhược điểm |
|---------|------|------|
| **Outputs + ImportValue** | Tích hợp sẵn trong CloudFormation, không phát sinh chi phí | Gắn chặt vòng đời xóa stack |
| **SSM Parameter Store** | Liên kết lỏng, dễ xóa hơn | Có chi phí, nhất quán cuối cùng |
| **DynamoDB lookup** | Linh hoạt | Cần custom resource, phức tạp hơn |
| **Nested Stacks** | Liên kết chặt, dễ quản lý vòng đời | Gộp các thành phần vào một template |

#### NÊN làm (DO)
- **Dùng `!Sub "${AWS::StackName}-VPC-ID"` làm tên Export** → Tên stack là duy nhất nên tên Export cũng duy nhất. Tránh ghi cố định tên Export.
- **Dùng Outputs cho hạ tầng dùng chung (VPC, Subnet, SG ID)** → Nhiều stack cần dùng chung các giá trị này, tạo thành **nguồn dữ liệu duy nhất**.

#### KHÔNG NÊN làm (DON'T)
- **Export quá nhiều giá trị không cần thiết** → Mỗi Export tạo một ràng buộc khiến stack khó xóa. NÊN: chỉ export giá trị thực sự cần dùng chung giữa các stack.
- **Dùng tham chiếu giữa các stack cho giá trị thay đổi thường xuyên** → Khi giá trị ở stack gốc đổi, mọi stack phụ thuộc đều bị ảnh hưởng. NÊN: dùng **SSM Parameter Store** cho giá trị cần thay đổi lúc chạy.

---

## 5. CloudFormation - Conditions

### Chi tiết về Conditions

#### Giải thích cốt lõi
Conditions cho phép **tạo resource có điều kiện** dựa trên:
- Giá trị parameter (Environment, Region).
- Pseudo parameters.
- Các condition khác.

**Các hàm nội tại dùng cho Conditions:**

| Hàm | Ý nghĩa | Ví dụ |
|----------|-------|---------|
| `Fn::Equals` | So sánh hai giá trị | `!Equals [!Ref Environment, prod]` |
| `Fn::And` | Mọi điều kiện đều đúng | `!And [!Condition IsProduction, !Condition CreateBackup]` |
| `Fn::Or` | Ít nhất một điều kiện đúng | `!Or [!Condition IsProduction, !Condition IsStaging]` |
| `Fn::Not` | Phủ định điều kiện | `!Not [!Condition IsProduction]` |
| `Fn::If` | Chọn một trong hai giá trị | `!If [IsProduction, prod-value, dev-value]` |

**Ví dụ:**

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  IsStaging: !Equals [!Ref Environment, staging]
  CreateBackup: !Or
    - !Condition IsProduction
    - !Condition IsStaging

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

#### NÊN làm (DO)
- **Dùng Conditions để tạo resource theo môi trường** → Một template có thể phục vụ dev, staging và prod nhưng vẫn tùy chỉnh resource theo từng môi trường. So với ba template riêng, giảm nguy cơ lệch cấu hình và công sức bảo trì.
- **Dùng `!If` để chọn giữa các giá trị** → Tránh lặp lại resource. Ví dụ: `InstanceType: !If [IsProd, m5.large, t3.micro]`.

#### KHÔNG NÊN làm (DON'T)
- **Dùng Conditions thay cho Nested Stacks khi logic quá phức tạp** → Conditions chỉ biểu diễn đúng/sai, không thể diễn tả “với mỗi X thì tạo Y”. NÊN: dùng **Nested Stacks** cho nhiều môi trường, dùng **Modules** cho thành phần có thể tái sử dụng.
- **Tạo tham chiếu vòng giữa Conditions** → Không có thứ tự đánh giá nhất quán và template có thể không triển khai được. NÊN: chỉ tham chiếu theo một chiều và kiểm tra các tổ hợp giá trị.

---

## 6. CloudFormation - Hàm nội tại

### Chi tiết về các hàm

#### Giải thích cốt lõi
**Các hàm được tô hồng — cần nắm vững theo trang trình chiếu:**
- `Ref`
- `Fn::GetAtt`
- `Fn::FindInMap`
- `Fn::ImportValue`
- `Fn::Base64`
- `Condition Functions` (`Fn::If`, `Fn::Not`, `Fn::Equals`, v.v.)

**`!Ref` và `!GetAtt` - Khác biệt quan trọng:**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro

# !Ref - trả về định danh chính (physical ID)
!Ref MyInstance  # → "i-0123456789abcdef0"

# !GetAtt - trả về thuộc tính cụ thể
!GetAtt MyInstance.AvailabilityZone  # → "us-east-1a"
!GetAtt MyInstance.PrivateIp  # → "10.0.1.5"
!GetAtt MyInstance.PublicDnsName  # → "ec2-1-2-3-4.compute-1.amazonaws.com"
```

**Các thuộc tính có sẵn** tùy theo loại resource; tra tài liệu AWS của từng resource.

**`!Sub` - Thay thế chuỗi linh hoạt:**

```yaml
# Thay thế đơn giản
!Sub "arn:aws:s3:::${BucketName}"

# Dùng kèm lệnh gọi hàm
!Sub "https://${Domain}.s3.${AWS::Region}.amazonaws.com"

# Nhiều dòng trong template
!Sub |
  Hello ${User}
  Your account is ${AccountId}
  Stack: ${AWS::StackName}
```

#### NÊN làm (DO)
- **Dùng `!Sub` thay vì `!Join` khi phù hợp** → `!Sub` ngắn gọn và dễ đọc hơn. `!Sub "arn:${Partition}:s3:::${Bucket}"` ngắn hơn `!Join ["", ["arn:", !Ref Partition, ":s3:::", !Ref Bucket]]`.
- **Dùng `!GetAtt` thay vì ghi cố định giá trị thuộc tính** → Thuộc tính có thể thay đổi (private IP sau khi dừng/khởi động lại EC2); giá trị ghi cố định sẽ sai.
- **Dùng `Fn::Cidr` để tự động tạo CIDR cho subnet** → Tránh tính toán thủ công và dễ mở rộng. Ví dụ: từ VPC `/16`, tạo 4 subnet `/18`: `!Cidr [!Ref VPCCidr, 4, 18]`.

#### KHÔNG NÊN làm (DON'T)
- **Dùng `!Sub` cho logic thay thế lồng nhau phức tạp** → Template sẽ khó đọc. NÊN: dùng `!Join` cho phép nối chuỗi phức tạp, dùng `!Select` để chọn phần tử theo chỉ mục trong danh sách.
- **Dùng `Fn::Base64` cho dữ liệu nhạy cảm** → Base64 chỉ là cách biểu diễn dữ liệu, không phải mã hóa bảo mật. NÊN: dùng `Secrets Manager` hoặc `SSM SecureString`.

---

## 7. CloudFormation - Rollback

### Chi tiết về Rollback

#### Giải thích cốt lõi
CloudFormation có ba tình huống rollback:

**1. Tạo stack thất bại:**
- Hành vi mặc định: **ROLLBACK** — xóa mọi resource đã tạo.
- Tắt rollback bằng `DisableRollback: true` → giữ lại resource để gỡ lỗi.
- Sau khi sửa lỗi, xóa thủ công hoặc gọi `ContinueUpdateRollback`.

**2. Cập nhật stack thất bại:**
- Tự động rollback về **trạng thái hoạt động tốt trước đó**.
- Có thể xem log để biết nguyên nhân thất bại.
- Trạng thái stack: `UPDATE_ROLLBACK_COMPLETE` hoặc `UPDATE_ROLLBACK_FAILED`.

**3. Rollback thất bại:**
- Resource có thể bị kẹt ở trạng thái không nhất quán.
- Sửa từng resource thủ công.
- Sau đó gọi API `continue-update-rollback`.

**So sánh với công cụ khác:**

| Tình huống | CloudFormation | Terraform |
|----------|----------------|-----------|
| Tạo thất bại | Tự động rollback (xóa các resource đã tạo) | Dừng apply và ghi các thay đổi đã hoàn tất vào state; hạ tầng có thể ở trạng thái dở dang |
| Cập nhật thất bại | Rollback về trạng thái hoạt động tốt trước đó | Có thể để lại thay đổi đã thực hiện; Terraform không tự rollback partial apply |
| Tiếp tục sau thất bại | Sửa thủ công và gọi `continue-update-rollback` khi rollback của stack lỗi | Sửa lỗi rồi tạo plan/apply mới; không có thao tác rollback tự động tương đương |

[Tài liệu Terraform về lỗi trong lúc apply](https://developer.hashicorp.com/terraform/cli/commands/apply#errors-during-an-apply-operation).

#### NÊN làm (DO)
- **Tắt rollback cho lần triển khai đầu của production stack** → Có thể xem các resource đã tạo để gỡ lỗi mà không bị xóa hết. Sau khi triển khai thành công, bật lại rollback.
- **Dùng Change Sets cho cập nhật production** → Xem trước thay đổi trước khi áp dụng, tránh xóa nhầm. Change Sets nêu rõ resource nào sẽ bị thay thế, sửa đổi hoặc gỡ bỏ.

#### KHÔNG NÊN làm (DON'T)
- **Tắt rollback vĩnh viễn cho production** → Nếu cập nhật thất bại, stack có thể ở trạng thái không nhất quán. NÊN: bật rollback cho production, chỉ tắt trong tình huống gỡ lỗi.
- **Bỏ qua trạng thái `UPDATE_ROLLBACK_FAILED`** → Stack bị kẹt, không thể cập nhật hoặc xóa bình thường. NÊN: kiểm tra ngay, sửa resource thủ công rồi gọi `continue-update-rollback`.

---

## 8. CloudFormation Service Role

### Thực hành tốt với Service Role

#### Giải thích cốt lõi
CloudFormation cần quyền IAM để tạo, cập nhật và xóa resource. Có hai cách cấp quyền:

**Cách 1: Người dùng có đủ quyền trực tiếp**
- Đơn giản nhưng **vi phạm nguyên tắc đặc quyền tối thiểu**.
- Người dùng phải có quyền đối với mọi resource trong template.

**Cách 2: CloudFormation Service Role (được khuyến nghị)**
- Người dùng chỉ cần quyền `cloudformation:*`.
- Service Role có quyền tạo resource.
- Người dùng cần **`iam:PassRole`** để chuyển Service Role cho CloudFormation.

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

**Chính sách người dùng cần có:**

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

#### NÊN làm (DO)
- **Dùng Service Role cho production** → Áp dụng **nguyên tắc đặc quyền tối thiểu**. Người dùng chỉ cần quyền triển khai stack, không cần quyền trực tiếp tạo EC2/RDS, v.v.
- **Tạo nhiều Service Role cho các trường hợp khác nhau** → Giới hạn phạm vi ảnh hưởng. Ví dụ: `CFRole-Dev` (đủ quyền), `CFRole-Prod-ReadOnly` (chỉ cho phép đọc khi cập nhật).

#### KHÔNG NÊN làm (DON'T)
- **Cấp `AdministratorAccess` đầy đủ cho người dùng chỉ để triển khai stack** → Vi phạm nguyên tắc đặc quyền tối thiểu; người dùng có thể tạo hoặc xóa mọi thứ. NÊN: tạo Service Role có phạm vi quyền giới hạn theo resource cần thiết.
- **Quên cấp quyền `iam:PassRole`** → CloudFormation sẽ thất bại với lỗi “User is not authorized to perform iam:PassRole”. NÊN: thêm `iam:PassRole` vào policy của người dùng khi dùng Service Role.

---

## 9. CloudFormation DeletionPolicy
### Chi tiết về DeletionPolicy
#### Giải thích cốt lõi
**DeletionPolicy** kiểm soát điều gì xảy ra khi:
- Xóa stack.
- Xóa resource khỏi template.

**Ba loại:**

| Chính sách | Hành vi | Trường hợp sử dụng |
|--------|----------|----------|
| `Delete` (mặc định) | Xóa resource | Resource không lưu trạng thái (EC2, Security Group) |
| `Retain` | Giữ lại resource | Dữ liệu có trạng thái, tệp log, resource quản lý thủ công |
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
- `Delete` không xóa được **S3 bucket còn object**.
- `Snapshot` chỉ hỗ trợ một số loại resource (xem trang trình chiếu 426).
- Với `Retain`, cần dọn dẹp thủ công nếu muốn xóa resource sau đó.

#### NÊN làm (DO)
- **Dùng `Snapshot` cho RDS database** → Tạo bản sao lưu trước khi xóa để có thể khôi phục nếu cần. So với `Delete`, tránh mất dữ liệu vĩnh viễn.
- **Dùng `Retain` cho S3 bucket chứa dữ liệu quan trọng** → Tránh vô tình xóa bucket khi xóa stack; có thể dọn dẹp thủ công khi cần.

#### KHÔNG NÊN làm (DON'T)
- **Dùng `Delete` cho database/EBS chứa dữ liệu production** → Dữ liệu sẽ mất vĩnh viễn. NÊN: dùng `Snapshot` cho database, `Retain` cho EBS volume.
- **Dùng `Retain` cho mọi resource** → Tạo resource mồ côi, tốn tiền và khó dọn dẹp. NÊN: dùng `Delete` cho resource không lưu trạng thái (EC2, ASG, ELB), `Retain`/`Snapshot` cho resource có trạng thái.

---

## 10. CloudFormation Stack Policy

### Chi tiết về Stack Policy

#### Giải thích cốt lõi
**Stack Policy** bảo vệ resource khỏi **các cập nhật ngoài ý muốn** khi cập nhật stack.

**Hành vi mặc định khi không có Stack Policy:**
- Có thể sửa hoặc xóa **bất kỳ resource nào** khi cập nhật stack.
- Nguy cơ: vô tình xóa database production khi cập nhật template.

**Khi có Stack Policy:**
- Mặc định mọi resource **được bảo vệ**, từ chối mọi thao tác cập nhật.
- Phải chỉ định rõ **ALLOW** cho resource được phép cập nhật.
- Policy là một **tài liệu JSON**.

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

**Các thao tác có thể cho phép/từ chối:**
- `Update:*` — mọi thao tác cập nhật (Modify, Replace, v.v.).
- `Update:Replace` — chỉ thay thế.
- `Update:Delete` — chỉ xóa.
- `Update:Modify` — chỉ sửa thuộc tính.

#### NÊN làm (DO)
- **Dùng Stack Policy cho database production** → Bảo vệ khỏi thay thế hoặc xóa nhầm khi cập nhật template. Nếu cần cập nhật database, tạm sửa Stack Policy.
- **Dùng Stack Policy cho resource quan trọng (khóa mã hóa, IAM Role)** → Các resource này khó tạo lại và việc thay thế có ảnh hưởng lớn.

#### KHÔNG NÊN làm (DON'T)
- **Dùng Stack Policy chỉ có `Allow: *` cho mọi resource** → Tương đương không có policy bảo vệ. NÊN: từ chối rõ ràng thao tác với resource quan trọng, chỉ cho phép cập nhật resource cần thiết.
- **Cho rằng Stack Policy ngăn xóa toàn bộ stack** → Stack Policy chỉ áp dụng trong lúc cập nhật, không ngăn `DeleteStack`. NÊN: bật Termination Protection để ngăn xóa nhầm stack; dùng Stack Policy để bảo vệ resource khỏi cập nhật ngoài ý muốn.

---

## 11. CloudFormation Custom Resources

### Chi tiết về Custom Resources

#### Giải thích cốt lõi
**Custom Resources** mở rộng CloudFormation để:
1. Tạo resource **chưa được hỗ trợ** (ví dụ dịch vụ AWS mới).
2. Chạy **logic tùy chỉnh** trong các thao tác CloudFormation.
3. Quản lý **resource bên ngoài** (tài nguyên tại chỗ hoặc của bên thứ ba).
CloudFormation gửi các sự kiện vòng đời đến provider được chỉ định bằng `ServiceToken`; Lambda hoặc SNS thực hiện logic tùy chỉnh rồi phản hồi kết quả. Khác với resource gốc mà CloudFormation trực tiếp quản lý, Custom Resource phù hợp khi cần xử lý bên ngoài khả năng sẵn có (chẳng hạn làm trống bucket trước khi xóa).

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
              bucket = event["ResourceProperties"]["BucketName"]
              try:
                  if event["RequestType"] == "Delete":
                      s3 = boto3.client("s3")
                      versioning = s3.get_bucket_versioning(Bucket=bucket).get("Status")
                      if versioning in ("Enabled", "Suspended"):
                          paginator = s3.get_paginator("list_object_versions")
                          for page in paginator.paginate(Bucket=bucket):
                              objects = page.get("Versions", []) + page.get("DeleteMarkers", [])
                              if objects:
                                  batch = [{"Key": obj["Key"], "VersionId": obj["VersionId"]} for obj in objects]
                                  result = s3.delete_objects(Bucket=bucket, Delete={"Objects": batch, "Quiet": True})
                                  if result.get("Errors"):
                                      raise RuntimeError(str(result["Errors"]))
                      else:
                          paginator = s3.get_paginator("list_objects_v2")
                          for page in paginator.paginate(Bucket=bucket):
                              objects = page.get("Contents", [])
                              if objects:
                                  batch = [{"Key": obj["Key"]} for obj in objects]
                                  result = s3.delete_objects(Bucket=bucket, Delete={"Objects": batch, "Quiet": True})
                                  if result.get("Errors"):
                                      raise RuntimeError(str(result["Errors"]))
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {}, physicalResourceId=bucket)
              except Exception as exc:
                  cfnresponse.send(event, context, cfnresponse.FAILED, {"Error": str(exc)}, physicalResourceId=bucket, reason=str(exc))
```

Execution role cần có các quyền `s3:GetBucketVersioning`, `s3:ListBucket`, `s3:ListBucketVersions`, `s3:DeleteObject` và `s3:DeleteObjectVersion` phù hợp với bucket; xử lý từng trang trước khi Lambda trả kết quả để không vượt giới hạn 1.000 key của một lệnh `DeleteObjects`.


**Sự kiện vòng đời:**
- `Create` — khi tạo resource.
- `Update` — khi thay đổi properties.
- `Delete` — khi xóa resource/stack.

#### NÊN làm (DO)
- **Dùng Custom Resource để làm trống S3 bucket trước khi xóa** → CloudFormation không thể xóa bucket còn object. Lambda function này chạy trước khi xóa bucket.
- **Dùng Custom Resource cho resource bên thứ ba** → Có thể tích hợp Datadog, MongoDB Atlas và tài nguyên tại chỗ vào CloudFormation.
- **Lưu giá trị vào cache trong Custom Resource** → Tránh gọi API nhiều lần. Ví dụ: tạo IAM Role rồi cache Role ARN.

#### KHÔNG NÊN làm (DON'T)
- **Dùng Custom Resource cho resource AWS đã được hỗ trợ sẵn** → Phức tạp và khó bảo trì. NÊN: dùng resource CloudFormation gốc nếu có.
- **Quên thiết kế thao tác có tính lũy đẳng (idempotent)** → CloudFormation có thể gọi lại Lambda với cùng input. NÊN: thiết kế để gọi lặp vẫn đưa hệ thống về cùng trạng thái.

---

## 12. CloudFormation StackSets

### Chi tiết về StackSets

#### Giải thích cốt lõi
**StackSets** cho phép tạo, cập nhật và xóa stack trên **nhiều account và Region** bằng một template duy nhất.

**Trường hợp sử dụng:**
- Triển khai **nền tảng bảo mật cơ bản** (CloudTrail, Config) cho mọi account trong Organization.
- Triển khai **hạ tầng ghi log** ở nhiều Region.
- Áp dụng **chính sách tuân thủ** trên mọi account.

**Khái niệm:**

| Thuật ngữ | Định nghĩa |
|------|------------|
| **StackSet** | Template + parameters + capabilities |
| **Stack Instance** | Stack trong account/Region đích |
| **Administrator Account** | Account tạo/quản lý StackSet |
| **Target Account** | Account chứa stack instance |
| **Service-managed permissions** | Tự động thiết lập quan hệ tin cậy giữa các account trong Organization |
| **Self-managed permissions** | Tự thiết lập IAM Role |

**Quy trình:**
1. Administrator account tạo StackSet.
2. Chọn target account (thủ công hoặc Organizational Unit).
3. Chọn Region.
4. Tạo stack instance trong từng account/Region đích.
5. Cập nhật StackSet → cập nhật mọi instance.

**So sánh với các cách khác:**

| Cách làm | Trường hợp sử dụng | Độ phức tạp |
|----------|----------|------------|
| **StackSets** | Nhiều account, nhiều Region | Trung bình — AWS tự quản lý |
| **Nested Stacks + cross-region** | Một account, nhiều Region | Cao — phải tự sao chép |
| **Tự động hóa tùy chỉnh (Lambda)** | Logic tùy chỉnh | Cao — tự triển khai |
| **AWS Organizations SCP** | Giới hạn quyền | Thấp — chỉ giới hạn quyền |
#### NÊN làm (DO)
- **Dùng StackSets với service-managed permissions trong AWS Organization** → AWS tự tạo IAM Role và quan hệ tin cậy; đơn giản hơn so với tự quản lý.
- **Dùng StackSets cho hạ tầng nền** (CloudTrail, Config, GuardDuty) → Mọi account đều cần; có thể tự triển khai khi thêm account mới.

#### KHÔNG NÊN làm (DON'T)
- **Dùng StackSets cho stack ứng dụng riêng biệt** → Mỗi account/Region có thể chạy ứng dụng khác nhau nên khó cấu hình. NÊN: dùng nested stack hoặc CI/CD pipeline để triển khai ứng dụng.
- **Dùng StackSets mà chưa kiểm thử trên một vài account** → Một lỗi có thể ảnh hưởng hàng trăm account. NÊN: kiểm thử trên 1–2 account rồi mới triển khai toàn Organization.
## 13. Tổng hợp thực hành tốt với CloudFormation

### Các thực hành tốt tổng hợp

#### Giải thích cốt lõi
Dựa trên kinh nghiệm triển khai thực tế, một số mẫu thiết kế giúp template CloudFormation dễ bảo trì hơn:

**1. Cách tổ chức template:**
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
  # Mạng
  VPC:
    Type: AWS::EC2::VPC
    ...
  # Bảo mật
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    ...
  # Tính toán
  Instance:
    Type: AWS::EC2::Instance
    ...
# 5. Outputs ở cuối
Outputs:
  InstanceId:
    Value: !Ref Instance
```

**2. Quy ước đặt tên:**
- Resource Logical ID: dùng PascalCase và tên có ý nghĩa (`WebServer` thay vì `Instance1`).
- Parameters: dùng PascalCase với tiền tố theo nhóm (`DbPassword`, `VpcCIDR`).

**3. Bảo mật:**
- Không ghi cố định secret trong template.
- Dùng `Secrets Manager` với dynamic reference.
- Bật mã hóa theo mặc định.

#### NÊN làm (DO)
- **Dùng Nested Stacks cho hạ tầng lớn** → Chia template thành phần nhỏ giúp dễ đọc, kiểm thử và tái sử dụng. So với một template lớn, cách này dễ xem xét hơn; tuy nhiên, lỗi ở Nested Stack vẫn có thể khiến stack cha rollback, nên đây không phải cơ chế cô lập rollback.
- **Gắn thẻ mọi resource bằng Environment, Project, Owner** → Dễ theo dõi chi phí và tự động dọn dẹp.
- **Dùng Change Sets cho cập nhật production** → Xem trước thay đổi trước khi áp dụng, tránh vô tình thay thế resource.
- **Lưu template trong Git** → Có quản lý phiên bản, khả năng xem xét mã và dấu vết kiểm toán.
- **Bật CloudFormation Drift Detection** → Phát hiện resource bị thay đổi ngoài CloudFormation (ví dụ thao tác thủ công qua Console).

#### KHÔNG NÊN làm (DON'T)
- **Viết một template dài hơn 500 dòng** → Khó đọc, khó xem xét và dễ lỗi. NÊN: chia thành nested stack, mỗi stack dưới 300 dòng.
- **Ghi cố định AMI ID mà không có quy trình cập nhật** → AMI có thể bị ngừng hỗ trợ hoặc khác nhau theo Region. NÊN: dùng SSM parameter phù hợp hoặc mapping được duy trì.
- **Bỏ qua kiểm thử cho production stack** → Lỗi template có thể xóa dữ liệu. NÊN: kiểm thử trước trên dev account và dùng Change Sets để xem xét thay đổi.

---

## 14. CloudFormation - Drift Detection

### Chi tiết về Drift Detection

#### Giải thích cốt lõi
**Drift** (lệch trạng thái) xảy ra khi resource trên AWS bị sửa đổi bên ngoài CloudFormation (qua Console, CLI hoặc dịch vụ khác). Khi đó, trạng thái CloudFormation không khớp với trạng thái thực tế.

**Ví dụ:**
- Template tạo Security Group có rule SSH (port 22).
- Người dùng tự thêm rule RDP (port 3389) qua Console.
- CloudFormation không biết rule này.
- Nếu cập nhật stack, CloudFormation có thể **ghi đè** rule RDP.
Với resource type được hỗ trợ, drift detection so sánh thuộc tính thực tế với template và báo sai khác; thao tác này chỉ phát hiện, không tự sửa resource. Khác với cập nhật stack, để xử lý drift cần sửa template rồi cập nhật stack hoặc hoàn tác thay đổi ngoài CloudFormation.

**Phát hiện drift:**

```bash
# Phát hiện drift cho một stack
aws cloudformation detect-stack-drift --stack-name my-stack

# Xem chi tiết drift
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

**Trạng thái drift:**
- `IN_SYNC` — khớp với template.
- `MODIFIED` — đã bị thay đổi.
- `DELETED` — đã bị xóa.
- `NOT_CHECKED` — chưa được kiểm tra; trạng thái này không xác nhận resource khớp template.

#### NÊN làm (DO)
- **Chạy Drift Detection định kỳ (hàng tuần/hàng tháng)** → Phát hiện sớm thay đổi thủ công, tránh bất ngờ khi cập nhật stack. Có thể tự động hóa bằng CloudWatch Events rule.
- **Dùng quyền IAM để hạn chế thay đổi thủ công** → Ngăn người dùng sửa resource qua Console khi CloudFormation đang quản lý resource đó. Ví dụ: từ chối `ec2:CreateSecurityGroup` với người dùng không phải quản trị viên.

#### KHÔNG NÊN làm (DON'T)
- **Bỏ qua Drift Detection** → Khi cập nhật stack, các thay đổi thủ công có thể bị ghi đè và làm mất cấu hình. NÊN: điều tra drift, cập nhật template nếu cần hoặc hoàn tác thay đổi thủ công.
- **Chạy Drift Detection liên tục (mỗi phút)** → Tốn API call và có thể chạm giới hạn tốc độ. NÊN: lên lịch hàng tuần/hàng tháng, kết hợp với CloudWatch Events.

---

## 15. So sánh CloudFormation, CDK và Terraform

### So sánh chi tiết

#### Giải thích cốt lõi

| Tính năng | CloudFormation | CDK | Terraform |
|---------|----------------|-----|-----------|
| **Cú pháp** | YAML/JSON | TypeScript, Python, Java, Go, .NET | HCL (HashiCorp Language) |
| **Quản lý trạng thái** | AWS tự quản lý | AWS tự quản lý (biên dịch thành CloudFormation) | Tệp trạng thái (local hoặc S3) |
| **Nhiều cloud** | ❌ Chỉ AWS | ❌ Chủ yếu AWS | ✅ AWS, Azure, GCP, v.v. |
| **Thời gian làm quen** | Trung bình (YAML, hàm nội tại) | Cao (khái niệm lập trình) | Trung bình (cú pháp HCL) |
| **Cộng đồng** | Tích hợp gốc của AWS | Tích hợp gốc của AWS | Rất lớn, hỗ trợ nhiều cloud |
| **Chi phí** | Miễn phí (chỉ trả tiền resource) | Miễn phí | Miễn phí (Terraform Cloud có phí) |
| **Drift Detection** | ✅ Có sẵn | ✅ Có sẵn (qua CloudFormation) | ✅ Có sẵn |
| **Rollback** | ✅ Tự động | ✅ Tự động (qua CloudFormation) | ⚠️ Thủ công hoặc tự động |

**Nên dùng công cụ nào:**

| Trường hợp sử dụng | Công cụ nên dùng |
|----------|-------------|
| **Chỉ dùng AWS, nhóm thích YAML** | CloudFormation |
| **Chỉ dùng AWS, nhóm là nhà phát triển** | CDK |
| **Nhiều cloud** | Terraform |
| **Tuân thủ cao, cần AWS quản lý trạng thái** | CloudFormation |
| **Cần hệ sinh thái mã nguồn mở** | Terraform |

#### NÊN làm (DO)
- **Chọn CloudFormation khi chỉ dùng AWS và cần AWS quản lý trạng thái** → Không cần hạ tầng phụ để quản lý state; tích hợp tốt với dịch vụ AWS như CodePipeline, Service Catalog.
- **Dùng CDK khi nhóm là nhà phát triển và cần nhiều logic trong mã** → CDK hỗ trợ vòng lặp, điều kiện và cấu trúc trừu tượng dễ hơn YAML.

#### KHÔNG NÊN làm (DON'T)
- **Trộn CloudFormation và Terraform để quản lý cùng một resource** → Hai công cụ sẽ tranh quyền quản lý và cả hai cùng báo drift. NÊN: chọn một công cụ cho hạ tầng.
- **Dùng CloudFormation cho hạ tầng nhiều cloud** → CloudFormation chỉ hỗ trợ AWS. NÊN: dùng Terraform hoặc Pulumi.

---

## 16. Điểm cần lưu ý khi thi về CloudFormation

### Các điểm trọng tâm

#### Giải thích cốt lõi
Dựa trên đề cương thi và câu hỏi DVA-C02, CloudFormation tập trung vào:

**1. Hàm nội tại (thường gặp nhất):**
- `!Ref` và `!GetAtt` — khi nào dùng mỗi hàm.
- `!Sub` để thay thế chuỗi.
- `!Join` để nối chuỗi phức tạp.
- `!FindInMap` để lấy giá trị theo Region.
- `!ImportValue` để tham chiếu giữa các stack.

**2. Quản lý stack:**
- Hành vi rollback khi tạo, cập nhật hoặc thất bại.
- DeletionPolicy (`Delete`, `Retain`, `Snapshot`).
- Stack Policy (bảo vệ resource).
- Termination Protection.
- Drift Detection.

**3. Tính năng nâng cao:**
- Custom Resources (dựa trên Lambda).
- StackSets (nhiều account, nhiều Region).
- Nested Stacks.
- Change Sets.
- Service Roles.

**4. Parameters và Mappings:**
- Parameters: giá trị đầu vào của người dùng.
- Mappings: bảng tra cứu có giá trị ghi sẵn.

**5. Outputs:**
- Tham chiếu giữa các stack.
- Tên Export phải duy nhất.

#### NÊN làm (DO)
- **Học cú pháp rút gọn của YAML** → Kỳ thi có cả JSON và YAML. YAML: `!Ref`, `!Sub`, `!GetAtt`; JSON: `{"Ref": "...", "Fn::Sub": "..."}`.
- **Hiểu hành vi của DeletionPolicy** → Câu hỏi giữ dữ liệu RDS khi xóa stack có thể cần `Snapshot` nếu resource hỗ trợ; nếu không, cân nhắc `Retain`.
- **Biết khi nào cần Custom Resources** → Để xóa S3 bucket còn object, cần làm trống bucket trước.

#### KHÔNG NÊN làm (DON'T)
- **Học thuộc hơn 700 loại resource** → Không khả thi và không cần thiết. NÊN: hiểu cấu trúc `service-provider::service-name::data-type-name` và biết tra tài liệu.
- **Bỏ qua Drift Detection** → Kỳ thi có thể hỏi cách phát hiện resource bị thay đổi bên ngoài CloudFormation.

---

## Tổng kết Phần 8

### Các khái niệm cần nhớ:

| Khái niệm | Điểm chính |
|---------|-----------|
| **Hạ tầng dưới dạng mã khai báo (IaC)** | Khai báo trạng thái mong muốn; AWS xử lý các bước còn lại |
| **Parameters** | Đầu vào của người dùng, giá trị động |
| **Mappings** | Bảng tra cứu ghi sẵn (Region, AMI) |
| **Outputs** | Tham chiếu giữa các stack qua `!ImportValue` |
| **Conditions** | Tạo resource có điều kiện |
| **Intrinsic Functions** | `!Ref`, `!GetAtt`, `!Sub`, `!FindInMap`, `!ImportValue` |
| **Rollback** | Tự động rollback khi tạo/cập nhật thất bại |
| **Service Role** | IAM Role để CloudFormation thực hiện thao tác thay mặt người dùng |
| **DeletionPolicy** | Delete (mặc định), Retain, Snapshot |
| **Stack Policy** | Bảo vệ resource khỏi cập nhật nhầm |
| **Custom Resources** | Mở rộng CloudFormation bằng logic tùy chỉnh dựa trên Lambda |
| **StackSets** | Triển khai trên nhiều account và Region |
| **Drift Detection** | Phát hiện thay đổi thủ công bên ngoài CloudFormation |

## 17. Cập nhật về cú pháp, bí mật và vòng đời resource

> Cập nhật theo tài liệu AWS chính thức ngày 2026-11-06; phần này bổ sung cho trang trình chiếu, không thay thế nội dung gốc.

### Hàm nội tại và `AWS::LanguageExtensions`

#### Giải thích cốt lõi

`Fn::Length` không phải hàm mặc định có thể dùng độc lập: cần khai báo transform `AWS::LanguageExtensions`. Transform này cũng cung cấp `Fn::ForEach` và `Fn::ToJsonString`. Trong YAML, khai báo điều kiện ở cấp cao nhất bằng `Conditions:`; gắn condition vào resource bằng thuộc tính `Condition: LogicalId`, còn các hàm nội tại như `!Equals` biểu diễn logic. Tham khảo [transform AWS::LanguageExtensions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/transform-aws-languageextensions.html) và [Conditions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html).

#### NÊN làm (DO)

- **Khai báo `Transform: AWS::LanguageExtensions` khi dùng `Fn::Length`/`Fn::ForEach`** → Nếu thiếu transform, template không nhận các hàm mở rộng; đây là yêu cầu bổ sung so với cú pháp CloudFormation lõi.
- **Dùng `Conditions:` ở cấp cao nhất và tham chiếu Logical ID bằng thuộc tính `Condition`** → Tách biểu thức điều kiện khỏi resource được kiểm soát.

#### KHÔNG NÊN làm (DON'T)

- **Dùng `Fn::Length` như hàm nội tại có sẵn mà không khai báo transform** → Template có thể không được xử lý như mong muốn. NÊN: khai báo transform và kiểm tra template theo tài liệu hiện hành.

### SecureString, mapping AMI và import giữa các stack

#### Giải thích cốt lõi

SSM Parameter Store `SecureString` được mã hóa bằng KMS, nhưng `AWS::SSM::Parameter::Value<SecureString>` không phải loại CloudFormation parameter được hỗ trợ như các loại parameter thông thường. Với dynamic reference, dùng `{{resolve:ssm-secure:parameter-name:version}}` để lấy SecureString trong thuộc tính resource được hỗ trợ; tránh đưa secret vào output/log. AMI ID phụ thuộc Region: mapping tĩnh cần được cập nhật khi duy trì AMI mới, còn public SSM parameter có thể cung cấp AMI hiện hành. `Fn::ImportValue` tham chiếu Export giữa các stack trong cùng Region; không thể xóa stack xuất khi còn stack đang import. Xem [dynamic reference SSM SecureString](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references-ssm-secure-strings.html), [ví dụ parameter AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html#stable-ami-ids), [Fn::ImportValue](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-importvalue.html).
`Ref` trả về giá trị tùy loại resource: thường là tên resource (ví dụ S3 bucket name), còn EC2 instance trả về instance ID. Tra mục giá trị trả về của `Ref` cho từng resource; không mặc định mọi `Ref` đều trả physical ID. [Tài liệu `Ref`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-ref.html).


#### NÊN làm (DO)

- **Dùng dynamic reference `ssm-secure` cho SecureString khi resource/property hỗ trợ** → CloudFormation không ghi giá trị bí mật vào template; so với `NoEcho`, đây là cách tham chiếu giá trị được mã hóa trong Parameter Store.
- **Dùng `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` hoặc mapping AMI có quy trình cập nhật** → Chọn AMI theo Region, tránh tham chiếu image không tồn tại ở Region khác.
- **Gỡ mọi stack đang import trước khi xóa stack xuất** → Cross-stack import ràng buộc vòng đời; cần xóa theo đúng thứ tự, khác với tham chiếu lỏng.

#### KHÔNG NÊN làm (DON'T)

- **Coi `NoEcho` là nơi lưu secret hoặc truyền SecureString trực tiếp bằng parameter type** → `NoEcho` không thay kho secret, và CloudFormation không hỗ trợ SSM SecureString như loại parameter thông thường. NÊN: dùng dynamic reference `ssm-secure` hoặc Secrets Manager.
- **Dùng AMI ID cố định mà không kiểm tra vòng đời/Region** → AMI có thể không còn phù hợp hoặc không tồn tại ở Region khác. NÊN: dùng mapping được cập nhật có kiểm soát hoặc public SSM parameter.

### Dọn dẹp S3, DeletionPolicy và drift

#### Giải thích cốt lõi

Không thể xóa S3 bucket khi còn object; custom resource cần xử lý theo từng trang và riêng trường hợp bucket bật/tạm ngưng versioning bằng cách xóa mọi version và delete marker. Nếu không khai báo `DeletionPolicy`, CloudFormation thường xóa resource; ngoại lệ mặc định `Snapshot` áp dụng cho `AWS::RDS::DBCluster` và `AWS::RDS::DBInstance` không có `DBClusterIdentifier`. `Snapshot` chỉ áp dụng cho loại resource được hỗ trợ; `NOT_CHECKED` nghĩa là chưa kiểm tra drift, không phải `IN_SYNC`. Tài liệu: [DeletionPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html), [mã trạng thái drift](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html), [Custom Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html).

#### NÊN làm (DO)

- **Thiết kế thao tác xóa của custom resource có tính lũy đẳng (idempotent)** → CloudFormation có thể gửi lại yêu cầu vòng đời; gọi lặp với cùng yêu cầu phải đưa hệ thống về cùng trạng thái, không báo lỗi nếu bucket đã trống.
- **Kiểm tra loại resource có hỗ trợ `Snapshot` trước khi chọn DeletionPolicy** → Nếu không hỗ trợ, stack không thể tạo snapshot như dự kiến. NÊN: dùng `Retain` hoặc cơ chế sao lưu riêng.
- **Coi `NOT_CHECKED` là chưa có kết luận về drift** → Chạy drift detection để xác minh khi cần; chỉ `IN_SYNC` mới cho biết resource đã được kiểm tra và khớp cấu hình.

#### KHÔNG NÊN làm (DON'T)

- **Dịch idempotency thành “nguyên ảnh”** → Cụm từ này không diễn đạt nghĩa kỹ thuật. NÊN: dùng “tính lũy đẳng (idempotent): gọi lặp lại vẫn cho cùng kết quả/trạng thái”.
- **Cho rằng mọi DeletionPolicy Snapshot đều hợp lệ hoặc `NOT_CHECKED` nghĩa là không drift** → Có thể mất dữ liệu hoặc bỏ sót lệch cấu hình. NÊN: đối chiếu loại resource được hỗ trợ và chủ động chạy kiểm tra.

## 18. Điểm cần nhớ và bài tập

#### Giải thích cốt lõi

Các điểm trọng tâm trên trang trình chiếu gồm thành phần template, Parameters/Mappings/Outputs/Conditions, hàm nội tại, rollback, DeletionPolicy, Custom Resources, StackSets và drift. Với các hàm trên trang trình chiếu, cần nhớ `Fn::Length`, `Fn::ForEach` và `Fn::ToJsonString` phải dùng cùng `AWS::LanguageExtensions`; không nhầm chúng với các hàm nội tại cơ bản.

#### NÊN làm (DO)

- **Nhớ AMI mapping theo Region, import/export trong cùng Region, S3 bucket phải rỗng trước khi xóa và DeletionPolicy Snapshot chỉ dùng với resource được hỗ trợ** → Đây là các điều kiện ảnh hưởng đến đáp án.
- **Thực hành bài tập VPC, Subnet, ALB và EC2 trên trang trình chiếu** → Có thể áp dụng `Ref`, `GetAtt`, mapping và tham chiếu giữa các resource.

#### KHÔNG NÊN làm (DON'T)

- **Chỉ học thuộc tên hàm mà bỏ qua yêu cầu transform hoặc điều kiện vòng đời** → Có thể chọn cú pháp không hợp lệ. NÊN: nhớ cả trường hợp sử dụng và giới hạn.
