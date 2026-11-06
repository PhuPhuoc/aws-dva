# Section 1 - Bắt đầu với AWS | IAM & AWS CLI | Cơ bản về EC2

---

## 1. Bắt đầu với AWS (Trang 8-16)

### AWS - Dẫn đầu thị trường đám mây (Trang 9)

*(Hình ảnh: AWS dẫn đầu thị trường đám mây.)*

### Hạ tầng toàn cầu AWS (Trang 10)

- AWS Regions
- AWS Availability Zone
- AWS Data Centers
- AWS Edge Locations / Points of Presence

### Các AWS Region (Trang 11)

- **Region** là một cụm trung tâm dữ liệu.
- **Chọn AWS Region dựa trên:**
  - **Tuân thủ quy định quản trị dữ liệu và yêu cầu pháp lý:** dữ liệu không rời khỏi Region nếu bạn chưa cho phép rõ ràng.
  - **Khoảng cách tới khách hàng:** giảm độ trễ.
  - **Các dịch vụ hiện có trong Region:** dịch vụ và tính năng mới không có ở mọi Region.
  - **Giá:** khác nhau giữa các Region và được công bố trên trang định giá của dịch vụ.

> *Biểu đồ AWS Region: minh họa các Region trên thế giới và phạm vi phủ sóng.*

### AWS Availability Zone (Trang 12-14)

- Mỗi Region có nhiều **Availability Zone** (AZ):
  - Thông thường có **3**, tối thiểu **3**, tối đa **6**.
  - Ví dụ: `ap-southeast-2a`, `ap-southeast-2b`, `ap-southeast-2c`.

- Mỗi AZ gồm một hoặc nhiều trung tâm dữ liệu riêng biệt, có:
  - Nguồn điện dự phòng
  - Kết nối mạng dự phòng
  - Kết nối riêng biệt

- Các AZ **tách biệt** để hạn chế ảnh hưởng lẫn nhau khi xảy ra thảm họa.
- Các AZ kết nối bằng mạng **băng thông cao, độ trễ cực thấp**.

> *Sơ đồ Availability Zone: ba AZ trong cùng một Region được kết nối với nhau.*

### AWS Edge Locations (Points of Presence) (Trang 15-16)

- Mạng lưới edge toàn cầu hiện có:
  - **400+ Edge Locations**
  - **13 Regional Caches**
  - **90+ thành phố** tại **48+ quốc gia**

- **Mục đích:** phân phối nội dung đến người dùng cuối với **độ trễ thấp hơn**.

> *Sơ đồ Edge Locations: minh họa mạng lưới edge toàn cầu.*

---

## 2. IAM & AWS CLI (Trang 17-32)

### AWS Identity and Access Management (AWS IAM) (Trang 17)


### IAM: Users & Groups (Trang 18)
- IAM = Quản lý danh tính và quyền truy cập.
- Là **dịch vụ toàn cầu** (không cần chọn Region).

- **Root account** được tạo mặc định, **không nên dùng hoặc chia sẻ**.
- **Users** là những người trong tổ chức và có thể được nhóm lại.
- **Groups** chỉ chứa users, không chứa groups khác.
- Users không bắt buộc thuộc một group; một user có thể thuộc nhiều groups.

> *Sơ đồ IAM Users & Groups: Root → Groups (Admin, Dev) → Users.*

### IAM: Permissions (Trang 19)

- Users hoặc Groups có thể được gán các tài liệu **JSON** gọi là **policies**.
- Policies xác định quyền của users.
- AWS áp dụng nguyên tắc **least privilege**: không cấp nhiều quyền hơn mức user cần.

### Tính kế thừa IAM Policies (Trang 20)

> *Sơ đồ kế thừa policy: Users trong Groups thừa hưởng policies từ group.*

### Cấu trúc IAM Policies (Trang 21)

Policy gồm:

| Thành phần | Mô tả |
|------------|--------|
| **Version** | Phiên bản ngôn ngữ policy |
| **Id** | Mã định danh policy (không bắt buộc) |
| **Statement** | Một hoặc nhiều statement (bắt buộc) |

**Statement gồm:**
- **Sid**: mã định danh statement (không bắt buộc)
- **Effect**: `Allow` hoặc `Deny`
- **Principal**: thực thể được policy chỉ định (account/user/role); chỉ xuất hiện trong resource-based policy, không có trong identity-based policy
- **Action**: danh sách hành động được cho phép hoặc từ chối
- **Resource**: danh sách tài nguyên áp dụng các hành động
- **Condition**: điều kiện để policy có hiệu lực (không bắt buộc)

### IAM - Chính sách mật khẩu (Trang 22)

- Mật khẩu mạnh hơn giúp tăng cường bảo mật.
- Trong AWS có thể thiết lập chính sách mật khẩu:
  - Đặt độ dài tối thiểu cho mật khẩu.
  - Yêu cầu các loại ký tự cụ thể.
  - Cho phép mọi IAM user đổi mật khẩu của chính họ.
  - Yêu cầu users đổi mật khẩu sau một khoảng thời gian (hết hạn mật khẩu).
  - Ngăn sử dụng lại mật khẩu cũ.

### Xác thực đa yếu tố (MFA) (Trang 23)

- Bảo vệ **Root Account** và IAM users.
- MFA kết hợp **mật khẩu bạn biết** với **thiết bị bảo mật bạn sở hữu**.

### Các thiết bị MFA trong AWS (Trang 24)

| Loại thiết bị | Ví dụ |
|-------------|-------|
| **Virtual MFA device** | Google Authenticator (chỉ trên điện thoại), Authy (chỉ trên điện thoại) |
| **U2F Security Key** | YubiKey của Yubico (bên thứ ba) |

### Người dùng có thể truy cập AWS bằng cách nào? (Trang 25)

**Có ba cách truy cập AWS:**

| Phương thức | Được bảo vệ bằng |
|-------------|------------------|
| **AWS Management Console** | Mật khẩu + MFA |
| **AWS CLI** | Access Keys |
| **AWS SDK** | Access Keys |

- Access Keys được tạo qua AWS Console.
- Access Keys là bí mật như mật khẩu, không chia sẻ.
- **Access Key ID** tương tự tên người dùng.
- **Secret Access Key** tương tự mật khẩu.

### AWS CLI là gì? (Trang 26)

- Công cụ tương tác với các dịch vụ AWS bằng lệnh trong shell dòng lệnh.
- Cho phép truy cập trực tiếp các API công khai của các dịch vụ AWS.
- Mã nguồn mở: https://github.com/aws/aws-cli

### AWS SDK (Trang 27)

- AWS SDK = Bộ công cụ phát triển phần mềm AWS.
- Cung cấp **API theo ngôn ngữ lập trình** (tập hợp thư viện).
- Cho phép truy cập và quản lý các dịch vụ AWS theo cách lập trình.
- Được nhúng trong ứng dụng.

**Ngôn ngữ/nền tảng được hỗ trợ:**
- SDK: JS, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++.
- SDK di động: Android, iOS.
- SDK thiết bị IoT: Embedded C, Arduino.

> *AWS CLI được xây dựng trên AWS SDK for Python.*

### IAM Roles cho các dịch vụ (Trang 28)

- Một số dịch vụ AWS cần thực hiện hành động thay mặt bạn.
- Gán quyền cho các dịch vụ AWS bằng **IAM Roles**.

**Các role thường dùng:**
- EC2 Instance Roles
- Lambda Function Roles
- Roles dành cho CloudFormation

### Công cụ bảo mật IAM (Trang 29)

| Công cụ | Cấp độ | Mô tả |
|------|--------|--------|
| **IAM Credentials Report** | Cấp account | Báo cáo liệt kê tất cả users và trạng thái credentials của họ |
| **IAM Access Advisor** | Cấp user | Hiển thị quyền đối với dịch vụ và thời điểm gần nhất dịch vụ được truy cập |

### Hướng dẫn và thực hành tốt nhất về IAM (Trang 30)

- ✅ Không dùng root account ngoài việc thiết lập AWS account.
- ✅ Một người dùng thực tế = Một AWS user.
- ✅ Gán users vào groups và gán quyền cho groups.
- ✅ Tạo chính sách mật khẩu mạnh.
- ✅ Dùng và bắt buộc sử dụng MFA.
- ✅ Tạo và dùng Roles để cấp quyền cho các dịch vụ AWS.
- ✅ Dùng Access Keys cho truy cập lập trình (CLI/SDK).
- ✅ Kiểm tra quyền bằng IAM Credentials Report và IAM Access Advisor.
- ✅ Không chia sẻ IAM users hoặc Access Keys.

### Mô hình trách nhiệm chung cho IAM (Trang 31)

| Trách nhiệm của AWS | Trách nhiệm của người dùng |
|--------------------|----------------------------|
| Hạ tầng (bảo mật mạng toàn cầu) | Quản lý Users, Groups, Roles, Policies |
| Phân tích cấu hình và lỗ hổng | Bật MFA cho mọi account |
| Xác thực tuân thủ | Thường xuyên luân chuyển keys |
| | Dùng công cụ IAM để áp dụng quyền phù hợp |
| | Phân tích mẫu truy cập và rà soát quyền |

### Tóm tắt phần IAM (Trang 32)

- **Users**: gắn với một người dùng thực tế, có mật khẩu cho AWS Console.
- **Groups**: chỉ chứa users.
- **Policies**: tài liệu JSON mô tả quyền cho users hoặc groups.
- **Roles**: dành cho EC2 instances hoặc các dịch vụ AWS.
- **Bảo mật**: MFA + chính sách mật khẩu.
- **AWS CLI**: quản lý dịch vụ AWS bằng dòng lệnh.
- **AWS SDK**: quản lý dịch vụ AWS bằng ngôn ngữ lập trình.
- **Access Keys**: truy cập AWS bằng CLI hoặc SDK.
- **Kiểm tra quyền**: IAM Credentials Report và IAM Access Advisor.

---

## 3. Cơ bản về EC2 (Trang 33-60)

### Amazon EC2 - Cơ bản (Trang 33)

- EC2 là một trong những dịch vụ phổ biến nhất của AWS.
- EC2 = **Elastic Compute Cloud** = hạ tầng như một dịch vụ (IaaS).

**EC2 chủ yếu cung cấp khả năng:**
- Thuê máy ảo (EC2).
- Lưu trữ dữ liệu trên ổ đĩa ảo (EBS).
- Phân phối tải giữa các máy (ELB).
- Mở rộng dịch vụ bằng Auto Scaling Group (ASG).

> *EC2 là nền tảng quan trọng để hiểu cách Cloud hoạt động.*

### EC2 - Các tùy chọn định cỡ và cấu hình (Trang 34)

| Tùy chọn | Các lựa chọn |
|--------|--------------|
| **Hệ điều hành (OS)** | Linux, Windows, macOS |
| **Năng lực tính toán và số lõi (CPU)** | Tùy Instance Type |
| **RAM** | Tùy Instance Type |
| **Lưu trữ** | Gắn qua mạng (EBS/EFS) hoặc ổ cứng cục bộ (EC2 Instance Store) |
| **Giao diện mạng** | Tốc độ, địa chỉ IP công khai |
| **Quy tắc tường lửa** | Security Group |
| **Script khởi tạo** | EC2 User Data |

### EC2 - User Data (Trang 35)

- Có thể dùng **EC2 User Data script** để khởi tạo instance.
- Bootstrapping là chạy các lệnh khi máy khởi động.
- Script **chỉ chạy một lần** khi instance khởi động lần đầu.

**EC2 User Data dùng để:**
- Cài đặt bản cập nhật.
- Cài đặt phần mềm.
- Tải tệp dùng chung từ Internet.
- Thực hiện tác vụ khởi tạo khác theo nhu cầu.

> *EC2 User Data Script chạy với quyền root.*

> *Thực hành: khởi chạy EC2 instance chạy Linux - Trang 36.*

### EC2 - Instance Types (Trang 37)

- Có nhiều Instance Type được tối ưu cho các trường hợp sử dụng khác nhau.

**Quy ước đặt tên EC2:**
```
m5.2xlarge
│ │ └──── kích thước
│ └────── thế hệ
└──────── họ instance
```

### EC2 Instance Types - Mục đích chung (Trang 38)

**Cung cấp sự cân bằng giữa:**
- Năng lực tính toán
- Bộ nhớ
- Mạng

**Phù hợp cho:**
- Máy chủ web
- Kho lưu trữ mã nguồn

### EC2 Instance Types - Tối ưu cho tính toán (Trang 39)

**Phù hợp với ứng dụng bị giới hạn bởi năng lực tính toán:**
- Ứng dụng cần nhiều năng lực tính toán
- Tác vụ xử lý theo lô
- Chuyển mã nội dung đa phương tiện
- Máy chủ web hiệu năng cao và tính toán hiệu năng cao (HPC)
- Mô hình khoa học, học máy
- Máy chủ trò chơi chuyên dụng và công cụ quảng cáo

### EC2 Instance Types - Tối ưu cho bộ nhớ (Trang 40)

**Hiệu suất cao cho khối lượng công việc xử lý tập dữ liệu lớn trong bộ nhớ:**
- Cơ sở dữ liệu quan hệ/phi quan hệ
- Bộ nhớ đệm phân tán quy mô web
- Cơ sở dữ liệu trong bộ nhớ tối ưu cho Business Intelligence (BI)
- Ứng dụng xử lý dữ liệu lớn phi cấu trúc theo thời gian thực

### EC2 Instance Types - Tối ưu cho lưu trữ (Trang 41)

**Đọc/ghi tuần tự với thông lượng cao trên tập dữ liệu rất lớn trong lưu trữ cục bộ:**

**Phù hợp cho:**
- Hệ thống OLTP tần suất cao
- Cơ sở dữ liệu quan hệ và NoSQL
- Bộ nhớ đệm cho cơ sở dữ liệu trong bộ nhớ (ví dụ: Redis)
- Kho dữ liệu
- Hệ thống tệp phân tán

### EC2 - Security Groups (Trang 42-44)

- Kiểm soát lưu lượng vào/ra EC2 (tường lửa).
- Có thể gắn vào **nhiều instances**.
- **Mặc định:** mọi inbound traffic bị chặn.
- **Mặc định:** mọi outbound traffic được cho phép.
- Security Groups chỉ chứa các quy tắc **Allow**.
- Rules có thể tham chiếu địa chỉ **IP** hoặc **Security Group**.

> *Sơ đồ Security Groups: Bastion SG → Web server SG → quy tắc Inbound/Outbound.*

**Ma trận Security Group minh họa trên slide (Trang 44):**

| Security Group | Hướng | Loại | Giao thức | Dải cổng | Nguồn/Đích |
|----------------|-------|------|----------|------------|--------------------|
| **Bastion SG** | Inbound | SSH | TCP | 22 | `0.0.0.0/0` |
| **Bastion SG** | Outbound | All traffic | All | All | `0.0.0.0/0` |
| **Web server SG** | Inbound | SSH | TCP | 22 | SG của bastion host |
| **Web server SG** | Outbound | All traffic | All | All | `0.0.0.0/0` |

### Các cổng phổ biến cần biết (Trang 45)

| Cổng | Giao thức | Mô tả |
|------|----------|--------|
| **22** | SSH | Đăng nhập vào instance Linux |
| **21** | FTP | Tải tệp lên vùng chia sẻ tệp |
| **22** | SFTP | Tải tệp bằng SSH |
| **80** | HTTP | Truy cập website không mã hóa |
| **443** | HTTPS | Truy cập website bảo vệ bằng TLS |
| **3389** | RDP | Đăng nhập vào instance Windows |

### EC2 - Kết nối tới EC2 instance (Trang 46)

> *Ma trận phương thức kết nối trên slide (Trang 46): dấu ✓ cho biết cách kết nối được hỗ trợ.*

| Hệ điều hành máy khách | SSH | PuTTY | EC2 Instance Connect |
|-------------------------|-----|-------|----------------------|
| Mac | ✓ | — | ✓ |
| Linux | ✓ | — | ✓ |
| Windows < 10 | — | ✓ | ✓ |
| Windows ≥ 10 | ✓ | ✓ | ✓ |

### EC2 Instance Connect (Trang 47)

- Kết nối tới EC2 instance **trong trình duyệt**
- **Không cần** tệp khóa
- Cần đảm bảo cổng 22 vẫn mở
- Chỉ hoạt động trên **Amazon Linux** hoặc **Ubuntu**

### Các lựa chọn mua EC2 (Trang 48)

| Loại | Mô tả |
|------|--------|
| **On-Demand Instances** | Khối lượng công việc ngắn hạn, giá dễ dự đoán, trả theo giây |
| **Reserved Instances (1 & 3 năm)** | Khối lượng công việc dài hạn - Reserved Instances và Convertible Reserved Instances |
| **Savings Plans (1 & 3 năm)** | Cam kết một mức sử dụng |
| **Spot Instances** | Khối lượng công việc ngắn hạn, giá thấp, có thể mất instance |
| **Dedicated Hosts** | Thuê trọn một máy chủ vật lý |
| **Dedicated Instances** | Phần cứng không dùng chung với khách hàng khác |
| **Capacity Reservations** | Dự trữ capacity trong một AZ cụ thể |

### EC2 On-Demand (Trang 49)

| Ưu điểm | Nhược điểm |
|----------|-------------|
| Không cần trả trước | Chi phí cao nhất |
| Trả theo mức sử dụng | |
| Không cần cam kết dài hạn | |

**Trường hợp sử dụng:**
- Môi trường phát triển/đánh giá/kiểm thử.
- Tác vụ xử lý theo lô quan trọng.
- Hệ thống chỉ chạy giờ hành chính (8:00-17:00).
- Hệ thống thường xuyên thay đổi quy mô.

### EC2 Reserved Instances (Trang 50-51)

**Giảm đến 72%** so với On-Demand.

| Thuộc tính | Lựa chọn |
|------------|---------|
| **Thời hạn đặt trước** | 1 năm (+ chiết khấu) hoặc 3 năm (+++ chiết khấu) |
| **Phương thức thanh toán** | Không trả trước (+), trả trước một phần (++), trả trước toàn bộ (+++) |
| **Phạm vi** | Regional hoặc Zonal (dự trữ capacity trong AZ) |

**Convertible Reserved Instances:**
- Có thể thay đổi type, family, OS.
- Giảm đến **66%**.

**Trường hợp sử dụng:**
- Hệ thống chạy 24/7 trong thời gian dài.
- Khối lượng công việc ổn định trong thời gian dài.

### EC2 Savings Plans (Trang 52-53)

- Cam kết một mức sử dụng ổn định, tính bằng USD/giờ.
- Giảm đến **72%** so với On-Demand.
- Gắn với **instance family** và **AWS Region** cụ thể.

**Linh hoạt theo:**
- Kích thước instance (m5.xlarge, m5.2xlarge).
- Hệ điều hành (Linux, Windows).
- Tenancy (Host, Dedicated, Default).

**Trường hợp sử dụng:**
- Hệ thống chạy 24/7 trong thời gian dài.
- Khối lượng công việc cần thay đổi **INSTANCE TYPE** về lâu dài.

### EC2 Spot Instances (Trang 54)

- Giảm đến **90%** so với On-Demand.
- Là loại instance **tiết kiệm chi phí nhất** của AWS.
- Nếu Spot price vượt max price, bạn có thể dừng hoặc kết thúc instance trong thời gian báo trước 2 phút.

**Spot Block:** giữ instance Spot trong 1-6 giờ mà không bị gián đoạn.

**Trường hợp sử dụng (khối lượng công việc có thể bị gián đoạn):**
- Tác vụ xử lý theo lô.
- Phân tích dữ liệu.
- Khối lượng công việc có thời điểm bắt đầu và kết thúc linh hoạt.

### EC2 Dedicated Hosts (Trang 55)

- Máy chủ vật lý có toàn bộ capacity của EC2 instances dành riêng cho bạn.
- Cho phép dùng giấy phép phần mềm máy chủ hiện có (theo socket, core hoặc VM).

### EC2 Dedicated Instances (Trang 56)

- Instances chạy trên phần cứng dành riêng cho bạn.
- Có thể **dùng chung phần cứng** với instances khác trong cùng account.
- **Không kiểm soát vị trí** đặt instance.

**Trường hợp sử dụng:**
- Hệ thống cần chính sách nghiêm ngặt và chạy trong môi trường cô lập.
  - Ví dụ: hệ thống ngân hàng, bệnh viện.

### Tìm hiểu AWS Tenancy (Trang 57)

> *Sơ đồ: Shared, Dedicated Host và Dedicated Instance.*

### EC2 Capacity Reservations (Trang 58)

- Dự trữ capacity On-Demand trong **AZ cụ thể**.
- Đảm bảo luôn có quyền truy cập EC2 capacity khi cần.
- **Không cam kết thời hạn** (có thể tạo/hủy bất kỳ lúc nào).
- **Không có chiết khấu**; bị tính phí theo mức On-Demand dù không chạy instances.

**Trường hợp sử dụng:** khối lượng công việc ngắn hạn, không bị gián đoạn và cần chạy trong AZ cụ thể.

### Lựa chọn hình thức mua nào phù hợp? (Trang 59)

| Phép so sánh | Hình thức mua |
|---------|-------------------|
| Khu nghỉ dưỡng - trả đủ giá, có thể đến ở bất cứ lúc nào | On-Demand |
| Khu nghỉ dưỡng - đặt trước lâu dài để được giảm giá | Reserved / Savings Plans |
| Khu nghỉ dưỡng - thuê phòng theo giờ nhưng có thể bị mời rời đi | Spot Instances |
| Khu nghỉ dưỡng - thuê trọn một tòa nhà | Dedicated Hosts |
| Khu nghỉ dưỡng - đặt phòng nhưng vẫn trả tiền dù không ở | Capacity Reservations |

### AWS tính phí địa chỉ IPv4 (Trang 60)

- Từ **ngày 1 tháng 2 năm 2024**, AWS tính phí cho Public IPv4 được tạo trong account.
- **$0.005/giờ** cho mỗi Public IPv4 (khoảng $3.6/tháng).

**Free Tier cho EC2 (12 tháng đầu):**
- 750 giờ Public IPv4 mỗi tháng.

**Không có Free Tier:**
- Load Balancer (một Public IPv4 cho mỗi AZ).
- RDS Database (một Public IPv4).

---

## Tổng kết Section 1

### Hạ tầng toàn cầu AWS
- **Region**: cụm trung tâm dữ liệu.
- **AZ**: các trung tâm dữ liệu riêng biệt trong Region (3-6 AZ).
- **Edge Locations**: hơn 400 điểm phân phối nội dung.

### IAM
- Users, Groups, Policies, Roles.
- MFA, chính sách mật khẩu.
- AWS CLI, SDK, Access Keys.

### EC2
- Instance Types: mục đích chung, tối ưu tính toán, bộ nhớ và lưu trữ.
- Security Groups: tường lửa kiểm soát lưu lượng.
- Các hình thức mua: On-Demand, Reserved, Spot, Dedicated.
