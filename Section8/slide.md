# Phần 8 - AWS CloudFormation (Trang 385-433)

---

## Phần 1: Tổng quan về CloudFormation (Trang 385-392)

### Giới thiệu CloudFormation (Trang 387)

**CloudFormation là gì?**
- Là cách tiếp cận khai báo để định nghĩa hạ tầng AWS
- Hầu hết tài nguyên AWS đều có thể được khai báo và quản lý bằng CloudFormation.
- Bạn khai báo trong template:
  - Security group
  - Hai EC2 instance dùng chung security group
  - Hai Elastic IP gắn với các EC2 instance
  - S3 bucket
  - Load Balancer (ELB) đứng trước các EC2 instance

**CloudFormation tự động:**
- Tạo resource theo **đúng thứ tự**
- Với **cấu hình chính xác** như đã khai báo

> *Sơ đồ (Trang 388): Template YAML được biểu diễn cùng sơ đồ Infrastructure Composer, gồm MySQLDatabase, LaunchConfig, ALBListener, ApplicationLoadBalancer và ALBTargetGroup.*

### Lợi ích của AWS CloudFormation (1/2) (Trang 389)

**Infrastructure as Code - Hạ tầng dưới dạng mã:**
- Không tạo resource thủ công — nhờ đó kiểm soát tốt hơn
- Có thể quản lý phiên bản mã bằng Git
- Xem xét thay đổi hạ tầng thông qua mã

**Chi phí:**
- Mỗi resource trong stack được gắn thẻ với một định danh → dễ theo dõi chi phí
- Có thể ước tính chi phí qua template CloudFormation
- **Chiến lược tiết kiệm:** tự động xóa template lúc 5 giờ chiều và tạo lại lúc 8 giờ sáng (môi trường Dev)

### Lợi ích của AWS CloudFormation (2/2) (Trang 390)

**Năng suất:**
- Có thể nhanh chóng xóa rồi tạo lại hạ tầng trên cloud
- Tự động tạo sơ đồ cho template
- Lập trình theo cách khai báo — không cần lo về thứ tự và điều phối

**Phân tách trách nhiệm:**
- Tạo nhiều stack cho nhiều ứng dụng và nhiều tầng
- Ví dụ: stack VPC, stack Network, stack App

**Không cần phát minh lại từ đầu:**
- Tận dụng template có sẵn trên web
- Tận dụng tài liệu AWS

### CloudFormation hoạt động như thế nào (Trang 391)

- Theo slide, tải template lên S3 rồi tham chiếu bằng URL trong CloudFormation.
> *Lưu ý hiện hành: không bắt buộc với mọi stack. Console nhận tệp cục bộ tối đa 1 MB và tự tải lên S3; API/CLI có thể nhận `TemplateBody` trực tiếp tối đa 51.200 bytes. S3 URL hỗ trợ object tối đa 1 MB và thường dùng khi template vượt giới hạn truyền trực tiếp hoặc có nested stacks ([tạo stack từ Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html), [quota template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)).*
- Khi cập nhật stack, cung cấp template đã chỉnh sửa; nếu tham chiếu qua S3, cập nhật hoặc tải lại object rồi cung cấp URL tương ứng.
- Stack được định danh bằng tên.
- **Xóa stack** → CloudFormation xóa resource theo `DeletionPolicy`: mặc định `Delete`; `Retain` giữ resource, còn `Snapshot` tạo snapshot cuối trước khi xóa resource được hỗ trợ.

### Triển khai template CloudFormation (Trang 392)

- **Thủ công:** chỉnh template bằng Application Composer hoặc trình soạn thảo; dùng Console để nhập parameter.
- **Tự động:** chỉnh tệp YAML, dùng AWS CLI hoặc công cụ Continuous Delivery để triển khai; đây là cách được khuyến nghị khi tự động hóa toàn bộ quy trình.

---

## Phần 2: Các khối cấu thành CloudFormation (Trang 393-399)

### Các khối cấu thành CloudFormation (Trang 393)

**Các thành phần của template:**

| Thành phần | Mô tả |
|-----------|-------------|
| `AWSTemplateFormatVersion` | Xác định phiên bản/khả năng của template (ví dụ: `2010-09-09`) |
| `Description` | Ghi chú về template |
| `Resources` (**BẮT BUỘC**) | Các AWS resource được khai báo trong template |
| `Parameters` | Giá trị đầu vào động cho template |
| `Mappings` | Biến tĩnh cho template |
| `Outputs` | Tham chiếu đến những gì đã được tạo |
| `Conditions` | Danh sách điều kiện để tạo resource |

**Các tiện ích hỗ trợ template:**
- **References**
- **Functions**

> *Ví dụ (Trang 394): Tạo EC2 instance đơn giản và gắn Security Group; phần này tập trung vào cấu trúc template trước khi đi sâu vào cú pháp.*

- Mã nguồn ví dụ: [Bài tập CloudFormation EC2](https://github.com/phongaws/aws-dva-code-2023/blob/main/cloudformation/Exercise/1.simple.ec2.yaml).

### Làm quen với YAML (Trang 395)

- CloudFormation hỗ trợ YAML và JSON; theo trang trình chiếu, JSON dài dòng hơn còn YAML thuận tiện hơn cho template.
- YAML hỗ trợ cặp khóa-giá trị, đối tượng lồng nhau, mảng, chuỗi nhiều dòng và chú thích.

### CloudFormation - Tài nguyên (Trang 396)

- **Resources** là thành phần cốt lõi của template CloudFormation (**BẮT BUỘC**).
- Đại diện cho các thành phần AWS sẽ được tạo và cấu hình.
- Các resource được khai báo và có thể tham chiếu lẫn nhau.
- AWS tự xử lý việc tạo, cập nhật và xóa.
- Có hơn 700 loại resource.
- Định danh loại resource có định dạng: `service-provider::service-name::data-type-name`.
  - Ví dụ: `AWS::EC2::Instance`.

### Tìm tài liệu về resource (Trang 397)

- Không thể học hết hơn 700 resource, nhưng có thể học cách sử dụng chúng.
- Tài liệu của mọi resource nằm trong AWS CloudFormation User Guide.
- Đọc tài liệu của từng resource.
- Ví dụ: tài liệu về `AWS::EC2::Instance`.

### Phân tích template CloudFormation (Trang 398)

- Xem lại ví dụ EC2 instance, Security Group và Elastic IP để hiểu lý do template được viết theo cấu trúc đó.
- Tra tài liệu CloudFormation cho từng resource, ví dụ `AWS::EC2::Instance`, `AWS::EC2::SecurityGroup` và `AWS::EC2::EIP`.

### Câu hỏi thường gặp về resource của CloudFormation (Trang 399)

**Có thể tạo số lượng resource động không?**
- Có, sử dụng **CloudFormation Macros** và **Transform**.
- Nội dung này nằm ngoài phạm vi khóa học.

**Mọi dịch vụ AWS đều được hỗ trợ không?**
- Gần như tất cả.
- Một số dịch vụ chuyên biệt chưa được hỗ trợ → có thể dùng **CloudFormation Custom Resources** làm giải pháp thay thế.

---

## Phần 3: CloudFormation - Parameters (Trang 400-405)

### CloudFormation - Parameters (Trang 400)

**Parameters là gì?**
- Cung cấp giá trị đầu vào cho template CloudFormation.
- Hữu ích khi:
  - Muốn tái sử dụng template trên toàn công ty.
  - Một số giá trị đầu vào chưa thể xác định trước.
- Parameters rất linh hoạt; type giúp kiểm soát giá trị và ngăn lỗi.

### Khi nào nên dùng Parameter? (Trang 401)

- Tự hỏi: “Cấu hình này có thể thay đổi trong tương lai không?”
- Nếu **có** → chuyển giá trị đó thành parameter.
- Lợi ích: không cần tải lại template để thay đổi giá trị.

### Cài đặt Parameters của CloudFormation (Trang 402)

**Parameters được kiểm soát bằng:**

**Type:**
- `String`
- `Number`
- `CommaDelimitedList`
- `List<Number>`
- `AWS-Specific Parameter` (giúp phát hiện giá trị không hợp lệ — giá trị phải khớp với giá trị có sẵn trong AWS account).
- `List<AWS-Specific Parameter>`

**Các cài đặt khác:**
- `Description` (String)
- `ConstraintDescription` (String)
- `Min/MaxLength`
- `Min/MaxValue`
- `Default`
- `AllowedValues` (mảng)
- `AllowedPattern` (biểu thức chính quy)
- `NoEcho` (Boolean)

**SSM Parameter:**
- Lấy giá trị parameter từ **SSM Parameter Store**.

> *Ví dụ (Trang 403): Truyền Parameters vào template CloudFormation.*

### Cách tham chiếu Parameter (Trang 404)

- Dùng hàm `Ref` để tham chiếu Parameters.
- Có thể sử dụng Parameters ở bất kỳ đâu trong template.
- Cú pháp rút gọn trong YAML: `!Ref`.
- Hàm này cũng có thể tham chiếu các thành phần khác trong template.

### Pseudo Parameters của CloudFormation (Trang 405)

- AWS cung cấp sẵn **Pseudo Parameters** trong mọi template CloudFormation.
- Có thể dùng bất kỳ lúc nào; chúng được bật mặc định.
- Các Pseudo Parameter quan trọng:
  - `AWS::Region`
  - `AWS::StackName`
  - `AWS::StackId`
  - `AWS::AccountId`
  - `AWS::NoValue`

---

## Phần 4: CloudFormation - Mappings (Trang 406-408)

### CloudFormation - Mappings (Trang 406)

- Mappings là các biến cố định trong template CloudFormation.
- Hữu ích để phân biệt giữa:
  - Các môi trường (dev và prod).
  - Các Region của AWS.
  - Các loại AMI.
- Tất cả giá trị đều được ghi sẵn trong template.

### Truy cập giá trị trong Mapping - Fn::FindInMap (Trang 407)

- Dùng `Fn::FindInMap` để lấy giá trị có tên từ một key cụ thể.
- Định dạng: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`.
- Mappings phù hợp với AMI vì AMI khác nhau theo Region.

### Khi nào dùng Mappings thay vì Parameters? (Trang 408)

**Mappings phù hợp khi:**
- Biết trước mọi giá trị có thể có.
- Có thể suy ra giá trị từ Region, Availability Zone, AWS Account hoặc môi trường (dev/prod).
- Muốn kiểm soát template an toàn hơn.

**Dùng Parameters khi:**
- Giá trị thực sự phụ thuộc vào từng người dùng.

---

## Phần 5: CloudFormation - Outputs (Trang 409-411)

### CloudFormation - Outputs (Trang 409-410)

- Phần **Outputs** khai báo các giá trị đầu ra tùy chọn.
- Có thể xuất giá trị để các stack khác import (nếu đã export trước đó).
- Có thể xem Outputs qua AWS Console hoặc AWS CLI.
- Hữu ích khi:
  - Định nghĩa mạng bằng CloudFormation.
  - Xuất các giá trị như **VPC ID**, **Subnet ID**.
- Đây là cách tốt để các nhóm phối hợp qua nhiều stack (mỗi nhóm quản lý stack riêng).

> *Sơ đồ (Trang 410): Stack tạo SSH Security Group và khai báo giá trị đầu ra tham chiếu đến Security Group đó.*

### Tham chiếu Outputs giữa các stack trong CloudFormation (Trang 411)

- Tạo template thứ hai sử dụng Security Group từ template thứ nhất.
- Dùng hàm `Fn::ImportValue`.
- **Không thể xóa stack gốc** cho đến khi xóa mọi tham chiếu đến stack đó.

---

## Phần 6: CloudFormation - Conditions (Trang 412-414)

### CloudFormation - Điều kiện (Trang 412)

- **Conditions** dùng để kiểm soát việc tạo resource hoặc output dựa trên một điều kiện.
- Điều kiện có thể là bất kỳ biểu thức nào; các trường hợp thường gặp:
  - **Environment** (dev / test / prod)
  - **AWS Region**
  - Giá trị parameter bất kỳ
- Mỗi condition có thể tham chiếu đến:
  - Một condition khác
  - Giá trị parameter
  - Mapping

### Cách định nghĩa Condition (Trang 413)

- **Logical ID** do bạn tự chọn để đặt tên cho condition.
- Có thể dùng các hàm logic nội tại sau:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

### Cách sử dụng Condition (Trang 414)

- Có thể áp dụng Condition cho:
  - Resource
  - Output
  - Và nhiều thành phần khác

---

## Phần 7: CloudFormation - Hàm nội tại (Trang 415-421)

### CloudFormation - Hàm nội tại (Trang 415)

| Hàm | Mô tả |
|----------|-------------|
| **`Ref`** | Tham chiếu parameters/resources |
| **`Fn::Base64`** | Chuyển chuỗi sang Base64 |
| **`Fn::GetAtt`** | Lấy thuộc tính của resource |
| **`Fn::Cidr`** | Phép toán CIDR |
| **`Fn::FindInMap`** | Lấy giá trị từ Mappings |
| **`Fn::GetAZs`** | Lấy các Availability Zone |
| **`Fn::ImportValue`** | Import từ các stack khác |
| **`Fn::Select`** | Chọn giá trị từ danh sách |
| **`Fn::Join`** | Nối chuỗi |
| **`Fn::Split`** | Tách chuỗi |
| **`Fn::Sub`** | Thay thế chuỗi |
| **`Fn::Transform`** | Biến đổi template |
| **`Fn::ForEach`** | Lặp qua các giá trị |
| **`Fn::Length`** | Đếm số phần tử trong một mảng |
| **`Fn::ToJsonString`** | Chuyển thành chuỗi JSON |
| **Condition Functions** | `Fn::If`, `Fn::Not`, `Fn::Equals`, v.v. |

> *Lưu ý: Các hàm được tô màu hồng trên trang trình chiếu là những hàm cần nắm vững.*

### Hàm nội tại `Ref` (Trang 416)

- Hàm `Ref` dùng để tham chiếu:
  - **Parameters** — trả về giá trị của parameter.
  - **Resources** — trả về giá trị phụ thuộc vào loại resource (ví dụ: instance ID với EC2 instance).
- Cú pháp rút gọn trong YAML: `!Ref`.

### Hàm nội tại `Fn::GetAtt` (Trang 417)

- Mỗi resource bạn tạo có thể có các thuộc tính.
- Tra tài liệu để biết thuộc tính của resource.
- Ví dụ: Availability Zone của một EC2 instance.

> *Ví dụ (Trang 417): Lấy thuộc tính Availability Zone của EC2 instance bằng `Fn::GetAtt`.*

### Hàm nội tại `Fn::FindInMap` (Trang 418)

- Dùng `Fn::FindInMap` để lấy giá trị có tên từ một key cụ thể.
- Định dạng: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`.

### Hàm nội tại `Fn::ImportValue` (Trang 419)

- Import các giá trị đã export ở những stack khác.
- Dùng hàm `Fn::ImportValue`.

### Hàm nội tại `Fn::Base64` (Trang 420)

- Chuyển chuỗi sang dạng Base64.
- Ví dụ: truyền dữ liệu đã mã hóa đến thuộc tính **UserData** của EC2 instance.

### Hàm nội tại về Condition (Trang 421)

- **Logical ID** do bạn tự chọn.
- Có thể dùng các hàm logic nội tại sau:
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn::Not`
  - `Fn::Or`

---

## Phần 8: CloudFormation - Rollback (Trang 422)

### CloudFormation - Rollback (Trang 422)

**Tạo stack thất bại:**
- Mặc định: mọi thứ được rollback (xóa) → xem log.
- Có thể tắt rollback để khảo sát sự cố.

**Cập nhật stack thất bại:**
- Stack tự động rollback về trạng thái hoạt động tốt trước đó.
- Có thể xem log và thông báo lỗi.

**Rollback thất bại:**
- Sửa resource thủ công.
- Sau đó gọi API `ContinueUpdateRollback` từ Console.
- Hoặc gọi API từ CLI bằng lệnh `continue-update-rollback`.

> *Sơ đồ (Trang 422): Luồng rollback khi tạo hoặc cập nhật stack thất bại.*
---

## Phần 9: CloudFormation - Service Role (Trang 423)

### CloudFormation - Service Role (Trang 423)

- **IAM Role** cho phép CloudFormation tạo, cập nhật và xóa resource của stack thay mặt bạn.
- Nhờ đó, người dùng có thể tạo, cập nhật và xóa resource của stack dù họ **không có quyền trực tiếp** trên các resource đó.

**Trường hợp sử dụng:**
- Muốn áp dụng **nguyên tắc đặc quyền tối thiểu**.
- Không muốn cấp cho người dùng mọi quyền cần thiết để tự tạo resource của stack.

**Yêu cầu:**
- Người dùng phải có quyền `iam:PassRole`.

---

## Phần 10: CloudFormation - DeletionPolicy (Trang 424-426)

### DeletionPolicy: Delete (Trang 424)

- **DeletionPolicy** kiểm soát điều gì xảy ra khi xóa template CloudFormation hoặc xóa resource khỏi template.
- Đây là lớp bảo vệ bổ sung để giữ lại hoặc sao lưu resource.
- Mặc định, DeletionPolicy là `Delete`.
- `Delete` không thể xóa S3 bucket nếu bucket vẫn còn object.

### DeletionPolicy: Retain (Trang 425)

- **`DeletionPolicy = Retain`**:
- Chỉ định resource cần giữ lại khi CloudFormation xóa.

### DeletionPolicy: Snapshot (Trang 426)

- **`DeletionPolicy = Snapshot`**:
- Tạo **snapshot cuối cùng** trước khi xóa resource.
- Các resource được hỗ trợ:
  - **EBS Volume**
  - **ElastiCache Cluster**, **ElastiCache ReplicationGroup**
  - **RDS DBInstance**, **RDS DBCluster**
  - **Redshift Cluster**
  - **Neptune DBCluster**
  - **DocumentDB DBCluster**

---

## Phần 11: CloudFormation - Stack Policy (Trang 427)

### CloudFormation - Stack Policy (Trang 427)

- Khi cập nhật CloudFormation stack, mặc định mọi thao tác cập nhật đều được phép trên tất cả resource.
- **Stack Policy** là tài liệu JSON định nghĩa các thao tác cập nhật được phép trên những resource cụ thể trong stack.
- **Bảo vệ resource** khỏi các cập nhật ngoài ý muốn.
- Khi thiết lập Stack Policy, mặc định tất cả resource trong stack đều được bảo vệ.
- Chỉ định rõ **ALLOW** cho những resource được phép cập nhật.

---

## Phần 12: CloudFormation - Bảo vệ khỏi xóa stack (Trang 428)

### Bảo vệ stack khỏi xóa (Trang 428)

- Để ngăn **xóa nhầm** CloudFormation stack, hãy dùng **Termination Protection**.

---

## Phần 13: CloudFormation - Custom Resources (Trang 429-431)

### CloudFormation - Custom Resources (Trang 429)

**Custom Resources được dùng để:**
- Định nghĩa resource chưa được CloudFormation hỗ trợ.
- Định nghĩa logic cấp phát tùy chỉnh cho resource bên ngoài CloudFormation (tài nguyên tại chỗ hoặc tài nguyên bên thứ ba).
- Trang trình chiếu nêu ví dụ dùng custom resource để làm trống S3 bucket trước khi CloudFormation xóa bucket.

**Cách định nghĩa:**
- Dùng `AWS::CloudFormation::CustomResource` hoặc `Custom::MyCustomResourceTypeName` (được khuyến nghị).
- Được hỗ trợ bởi **Lambda function** (phổ biến nhất) hoặc **SNS topic**.

### Cách định nghĩa Custom Resource (Trang 430)

- **ServiceToken** chỉ định đích nhận request từ CloudFormation, ví dụ Lambda ARN hoặc SNS ARN (**bắt buộc và phải cùng Region**).
- Có thể truyền thêm các parameter đầu vào.

### Trường hợp sử dụng - Xóa nội dung S3 bucket (Trang 431)

- Không thể xóa **S3 bucket còn object**; cần xóa hết object trước.
- Ví dụ trên trang trình chiếu: dùng **custom resource** để làm trống bucket trước khi CloudFormation xóa.

---

## Phần 14: CloudFormation - StackSets (Trang 432)

### CloudFormation - StackSets (Trang 432)

- Tạo, cập nhật hoặc xóa stack trên **nhiều account và Region** bằng một thao tác/template.
- Chọn các account để tạo, cập nhật hoặc xóa stack instance từ StackSets.
- Khi cập nhật stack set, mọi stack instance liên quan được cập nhật trên tất cả account và Region.
- Có thể áp dụng cho **tất cả account** trong một **AWS Organization**.
- Chỉ **Administrator account** (hoặc **Delegated Administrator**) mới có thể tạo StackSets.

---

## Bài tập (Trang 433)

- Tạo CloudFormation Stack gồm VPC, Subnet, ALB, EC2, v.v.

> *Sơ đồ bài tập (Trang 433): VPC `10.0.0.0/16` trải trên hai AZ, mỗi AZ có public và private subnet; Internet Gateway đưa lưu lượng tới ALB rồi phân phối đến hai EC2 web server.*
